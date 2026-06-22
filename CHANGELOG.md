# Changelog · FORM

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
- `blog.html` — blog card для post-112 додана у верхню частину сітки.
- `STATUS.md` — Total posts 1089 → 1090; Editorial series 106–111 → 106–112; post-112 записано в history рядку.
- `VERSION` — 5.75.0 → 5.76.0.

---

## [5.75.0] — 2026-06-16

### Added
- `content/post-1081-ecrb-ecrl-anatomy.md` — Block 1051–1100 Cluster 7, post 1/5. ECRB і ECRL: анатомія, функція, відмінності (довжина, початок, іннервація). Механізм lateral epicondylitis (tennis elbow): патомеханіка тендинопатії ECRB, ризикові grip-паттерни у тренінгу (row-об'єм, reverse curl, зміна хвату), захисні фактори. Практичне програмування: непряме навантаження через compound pulls, відсутність потреби у прямій ізоляції. sports-scientist review pending.
- `content/post-1082-flexor-carpi-ulnaris-anatomy.md` — Block 1051–1100 Cluster 7, post 2/5. FCU: двохголовий початок (humeral + ulnar heads), прикріплення через pisiform, іннервація ульнарним нервом (єдиний згинач зап'ястя не на median nerve). Medial epicondylitis: FCU у складі common flexor tendon, диференціація від кубітального тунельного синдрому (ульнарний нерв між головками FCU). Стабілізуюча роль у grip. sports-scientist review pending.
- `content/post-1083-palmaris-longus-anatomy.md` — Block 1051–1100 Cluster 7, post 3/5. Palmaris Longus: рудиментарний флексор зап'ястя, відсутній у ~10–15% людей. Thompson test. Еволюційний контекст (повноцінний у приматів, редукований у людини). Донорський сухожилок у реконструктивній хірургії (UCL, сухожилки пальців, ligament grafts). Відсутність не впливає на grip, силу або рухливість. sports-scientist review pending.
- `content/post-1084-flexor-digitorum-superficialis-anatomy.md` — Block 1051–1100 Cluster 7, post 4/5. FDS: трьохголовий початок, chiasma tendinum Камперa, іннервація median nerve. Дев'ять сухожилків через carpal tunnel (4 FDS + 4 FDP + FPL + median nerve). Carpal tunnel syndrome: анатомічний зв'язок і контекст для атлетів. Trigger finger механіка. Grip failure як FDS/FDP втома. sports-scientist review pending.
- `content/post-1085-lumbricales-anatomy.md` — Block 1051–1100 Cluster 7, post 5/5. Lumbricales: унікальний початок від сухожилків FDP, прикріплення до extensor expansion. Подвійна іннервація (median 1–2, ulnar 3–4). Парадокс флексора-розгинача: флексія MCP + розгинання PIP/DIP = intrinsic plus позиція. Chiasma tendinum. Intrinsic minus (ульнарний кіготь) при ушкодженні ульнарного нерва. Lumbricales як координатори grip, а не силові генератори. sports-scientist review pending.

### Changed
- `blog.html` — posts 1081–1085 додані у верхню частину сітки (Cluster 7: wrist extensors/flexors + intrinsic hand). blog.html оновлений до post-1085.
- `STATUS.md` — Block 1051–1100 прогрес оновлено до 35/50; Cluster 7 (posts 1081–1085) позначено COMPLETE; наступний: Cluster 8 (posts 1086–1090).

---

## [5.74.0] — 2026-06-16

### Added
- `compliance/cc8/staging-data-anonymisation-procedure.md` v1.0 — POL-CC8-ANON-01. Standalone extraction of `docs/OBSERVABILITY.md §45.3` per §45.8 checklist item 3 (P0/M6 · DEC-058). Four-step quarterly staging refresh procedure for FORM's shared staging Supabase project before k6 Cloud EU-West load-test runs: (1) schema-only dump from production + COPY/INSERT grep gate enforcing ANON-02; (2) `test-data-factory` seed (`lt-synthetic-*` tenants, `@test.form.coach` users, `skip_art9_tables: true`) + Art. 9 zero-row SQL verification; (3) clinical-safety attestation message template + Slack `:white_check_mark:` sign-off gate + PERF-STAGING-E-001 evidence filing within 30 min; (4) post-run `supabase db reset` cleanup. Five non-negotiable anonymisation invariants: ANON-01 (`gen_random_uuid()` user_ids only), ANON-02 (schema-only dump — zero COPY/INSERT), ANON-03 (`lt-synthetic-` tenant prefix), ANON-04 (`@test.form.coach` emails), ANON-05 (physically separate Supabase project). Includes quick-reference operator checklist, failure procedures for four edge cases, SOC 2 evidence mapping (PERF-STAGING-E-001 → CC4.1/A1.1), and privacy floor (seven non-negotiable constraints). clinical-safety VETO: any ANON invariant failure blocks the k6 Cloud run. Owner: platform-engineer + compliance-officer + clinical-safety.

### Changed
- `compliance/cc8/README.md` — `staging-data-anonymisation-procedure.md` entry added to evidence files table (POL-CC8-ANON-01 · CC4.1, A1.1, CC8.1 · quarterly cadence).
- `docs/OBSERVABILITY.md §45.8` — checklist item 3 (P0/M6) marked [x] Done: `compliance/cc8/staging-data-anonymisation-procedure.md` v1.0 authored (2026-06-16); clinical-safety sign-off + baseline PERF-STAGING-E-001 pending first quarterly run.
- `VERSION` → 5.74.0

## [5.73.0] — 2026-06-16

### Added
- `content/post-111-synthesis-six-frameworks.md` — Editorial series post-111. «Синтез 106–111: шість рамок мислення для self-coached атлета». Синтез постів 106–110: мікроцикл як одиниця аналізу (post-106), горизонт як діагностика «поганий день vs. поганий блок» (post-107), тренування і когнітивна продуктивність дозозалежно (post-108), МЕД і різниця між MV і MEV (post-109), варіабельність відгуку і n-of-1 (post-110). Плюс мета-рамка: як перемикатись між п'ятьма залежно від питання. clinical-safety: NOT_REQUIRED. Targeted: Дмитро (self-coached intermediate).

### Changed
- `blog.html` — post-111 card added (editorial series synthesis · 6 mental frameworks).
- `README.md` — editorial roadmap розширено: posts 131–140 proposed (cluster «Advanced self-coaching intelligence»: від мотивації до оптимальності, від новачка до intermediate, продуктивна vs. важка сесія, кілька цілей одночасно, синтез 131–140).
- `VERSION` → 5.73.0

---

## [5.72.0] — 2026-06-16

### Added
- `content/post-1076-posterior-deltoid-anatomy.md` — Block 1051–1100 cluster 6, post 1/5. Задній дельтоїд: горизонтальне відведення плеча як основна функція, конкуренція синергістів (ромбоподібні, трапеція) як причина систематичного недонавантаження, техніка ізоляції (фіксована лопатка + рух у горизонтальній площині), постуральний зв'язок з upper crossed syndrome. clinical_safety_review: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1077-triceps-long-head-anatomy.md` — Block 1051–1100 cluster 6, post 2/5. Довга головка трицепса: єдина двосуглобова головка (починається від лопатки), механіка пасивного подовження в overhead-позиції, EMG-обґрунтування переваги overhead extension над pushdown для акценту на довгу головку. clinical_safety_review: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1078-brachialis-anatomy.md` — Block 1051–1100 cluster 6, post 3/5. Brachialis: «чистий» флексор ліктя незалежно від пронації/супінації, анатомічне положення під biceps, чому hammer curl і reverse curl є самостійними вправами, роль у боковій товщині плеча, подвійна іннервація (musculocutaneous + radial). clinical_safety_review: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1079-pronator-teres-anatomy.md` — Block 1051–1100 cluster 6, post 4/5. Pronator teres: дві головки (humeral і ulnar), серединний нерв між головками (pronator syndrome), зв'язок з common flexor tendon і медіальною епікондилопатією. Вказівка: медіальний ліктьовий біль при навантаженні потребує клінічної оцінки. clinical_safety_review: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1080-anconeus-anatomy.md` — Block 1051–1100 cluster 6, post 5/5. Anconeus: трикутний стабілізатор задньо-латерального ліктьового суглоба, допоміжний розгинач, EMG-активація при швидких розгинаннях і пронованих положеннях. Пряме ізольоване тренування не потрібне. Диференціація бічного ліктьового болю. clinical_safety_review: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — posts 1076–1080 додані на початок сторінки (cluster 6 · Block 1051–1100).
- `VERSION` → 5.72.0

---

## [5.71.1] — 2026-06-16

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.13: +6 enterprise seat invitation DEC-030 events (`tenant.invite_sent`, `tenant.invite_accepted`, `tenant.invite_expired`, `tenant.invite_revoked`, `tenant.bulk_invite_started`, `tenant.bulk_invite_completed`) registered in new "Enterprise seat invitation events" section. INVITE-BULK-CHAIN-01 ordering invariant. Six Zod v2 schemas. Five SOC 2 evidence artefacts (INV-E-001 CC6.1 · INV-E-002 CC6.2 · INV-E-003 CC6.3 · INV-E-004 P4.2 · INV-E-005 P5.2). Retention table +2 rows (7yr invite_sent/accepted/revoked; 3yr invite_expired/bulk_started/completed). Closes `docs/DATA_MODEL.md §27.15` P0 checklist item 7 (M5).
- `docs/DATA_MODEL.md` — §27.15 checklist item 7 marked [x] Done · AUDIT_LOG_SCHEMA.md v2.13 (2026-06-16).
- `VERSION` → 5.71.1

---

## [5.71.0] — 2026-06-16

### Added
- `content/post-110-training-response-variability.md` — Editorial series post-110. «Варіабельність тренувального відгуку: чому ти і твій знайомий отримують різне від тієї ж програми». Hubal 2005 (-2%→+59% м'язовий об'єм, 0%→+250% сила); ACTN3/ACE поліморфізми; практично значущі чинники: тренувальний вік, сон, стрес HPA-вісь, харчовий статус, профіль волокон; нон-респондер = нон-респондер до протоколу (Nordin 2019); n-of-1 принцип. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-110 картка додана у сітку (Training science, 2026-06-16, draft).
- `README.md` — post-110 статус: proposed → draft · v5.71.0; editorial 121–130 proposals added.
- `STATUS.md` — editorial series 106–109 → 106–110; total posts 1077 → 1078.
- `VERSION` → 5.71.0

---

## [5.70.0] — 2026-06-16

### Added
- `content/post-1071-supraspinatus-anatomy.md` — Block 1051–1100 cluster 5, post 1/5. Supraspinatus: ініціатор відведення і найуразливіший м'яз ротаторної манжети. Анатомія субакроміального проходу, механізм iniціації відведення до 30° і депресія голівки плечової кістки, чому supraspinatus є найчастішим місцем розривів ротаторної манжети. Практичний акцент: три питання для оцінки стану субакроміального середовища в програмі (face pull/cable ER, площина lateral raise, жимово/тяговий баланс). clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1072-middle-deltoid-anatomy.md` — Block 1051–1100 cluster 5, post 2/5. Середній дельтоїд: геометрія відведення і чому lateral raise — найвибагливіша проста вправа. Пероподібна архітектура і її силово-амплітудний компроміс. Нерівномірна крива навантаження гантельного lateral raise vs. кабельного. Лопаткова площина (scaption) як більш функціональна альтернатива фронтальній. Компенсація верхньою трапецією і як її усвідомити. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1073-anterior-deltoid-anatomy.md` — Block 1051–1100 cluster 5, post 3/5. Передній дельтоїд: чому він вже натренований і коли пряме тренування є надлишковим. ЕМГ-порівняльна таблиця активації по вправах. Оцінка: 20–30+ ефективних підходів на тиждень через компаунди. Front raise у типовій пресовій програмі — надлишковий і посилює anterior/posterior дисбаланс. Три сценарії, де пряма робота виправдана. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1074-pec-minor-anatomy.md` — Block 1051–1100 cluster 5, post 4/5. Pectoralis minor: прихований контролер лопатки і чому його тісність стримує силу. Антеверсія лопатки як наслідок укорочення pec minor: два механізми — субакроміальне звуження і обмежена ретракція у тягах. Позиційний тест. Управління довжиною через стречинг і активацію lower trap/serratus як системний підхід, а не реабілітаційний. Coracoid impingement як рідкісний, але важливий варіант. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1075-coracobrachialis-anatomy.md` — Block 1051–1100 cluster 5, post 5/5. Coracobrachialis: найменший з клювоподібних прикріплень. Функції: флексія і приведення плеча як синергіст. Ключова клінічна деталь: musculocutaneous nerve пронизує coracobrachialis → нейропраксія впливає на biceps і brachialis нижче. «Coracoid cluster» — pec minor + coracobrachialis + biceps short head: спільна точка прикріплення, взаємовплив через лопаткову механіку. Пряме тренування не виправдане для загального силового тренінгу. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — posts 1071–1075 додані у верхівку сітки (Muscle anatomy, dates 2026-06-16, draft status). Total blog entries updated to post-1075.
- `STATUS.md` — Block 1051–1100 оновлено: 20/50 → 25/50, cluster 5 COMPLETE; total posts 1072 → 1077; next priorities оновлено на cluster 6 (posts 1076–1080).
- `VERSION` → 5.70.0

---

## [5.69.1] — 2026-06-16

### Changed
- `docs/SOC2_READINESS.md` — §84 Evidence Artefact Cross-Reference Patch: registers nine evidence artefacts (ENGAGE-E-001/002/003, SSO-OBS-E-005/006, WS-E-001/002/003, CONC-E-001) from OBSERVABILITY §§26/33/41 and INCIDENT_RESPONSE §18 that were defined in their source docs but never cross-referenced in SOC2_READINESS. Closes OBSERVABILITY §33.12 item 13 (P1/M14). Nine criterion rows extended: C1.1/C1.2/CC2.2/CC6.1/CC7.2/CC4.1/CC7.4/A1.1/P3.2. Doc version v3.9.1 → v3.10.0.
- `docs/OBSERVABILITY.md` — §33.12 item 13 marked [x] Done with reference to SOC2_READINESS §84.2.
- `VERSION` → 5.69.1

---

## [5.69.0] — 2026-06-16

### Added
- `content/post-109-minimum-effective-dose.md` — Editorial series post 109. Мінімальна ефективна доза для тренованого атлета: MEV vs. MV vs. MAV, дослідження Bickel (2011), Ralston (2017), Androulakis-Korakakis (2020). Один важкий підхід на м'яз на тиждень підтримує силу у тренованого атлета. Практична логіка підтримуючих протоколів і несприятливих тижнів. clinical-safety PASS.
- `blog.html` — додано картку post-109.
- `README.md` — editorial series розширено: теми 112–120 proposed. Block 1351–1400 «Programming for real life: constraints, transitions, longevity» додано до roadmap.

### Changed
- `STATUS.md` — editorial series оновлено: post-109 draft · v5.69.0.
- `VERSION` → 5.69.0

---

## [5.68.0] — 2026-06-16

### Added
- `content/post-1066-rhomboids-anatomy.md` — Block 1051–1100 cluster 4, post 1/5. Ромбоїди: ретракція лопатки та downward rotation. Чому «слабкі ромбоїди» — рідко ізольована проблема; дисбаланс жим/тяга як справжня причина. Практичне місце face pull.
- `content/post-1067-serratus-anterior-anatomy.md` — Block 1051–1100 cluster 4, post 2/5. Serratus anterior: upward rotation і протракція лопатки. Push-up plus як специфічний інструмент. Зв'язок з overhead pressing і winging лопатки.
- `content/post-1068-subscapularis-anatomy.md` — Block 1051–1100 cluster 4, post 3/5. Subscapularis: єдиний внутрішній ротатор манжети. ER/IR дисбаланс — чому пряме тренування subscapularis рідко виправдане для силових атлетів; акцент на зовнішніх ротаторах.
- `content/post-1069-teres-major-minor-anatomy.md` — Block 1051–1100 cluster 4, post 4/5. Teres major і minor: протилежні функції, схожі назви. Teres minor — член манжети, зовнішній ротатор. Teres major — «little lat», внутрішній ротатор, синергіст latissimus dorsi.
- `content/post-1070-infraspinatus-anatomy.md` — Block 1051–1100 cluster 4, post 5/5. Infraspinatus: головний зовнішній ротатор, що системно недонавантажений у стандартних програмах. ER/IR дисбаланс при жимовому домінуванні. Cable ER і face pull як мінімальна профілактика.
- `blog.html` — додано картки posts 1066–1070.

### Changed
- `VERSION` → 5.68.0
- `STATUS.md` — Block 1051–1100 progress: 20/50; next cluster 1071–1075 (суpraspinatus, середній та передній дельтоїди, pec minor, coracobrachialis).

---

## [5.67.1] — 2026-06-16

### Added
- `docs/OBSERVABILITY.md §12.7` — Postgres Index Hygiene Monitoring. New subsection after the pg_cron registry (§12.6). Canonical registry for compliance-evidence Postgres indexes: entry table with index name, table, partial-key WHERE, evidence query, SOC 2 mapping, EXPLAIN ANALYZE requirement, and status. First entry: `idx_egress_packages_eu_region` on `tenant_data_egress_packages` (migration 0075, DEC-061) — companion spot-check for OFB-E-005 (C1.1/P4.0/CC6.1). AL-IDX-01 unused-index sentinel `pg_stat_user_indexes` query included.

### Changed
- `docs/OBSERVABILITY.md` — header v4.2 → v4.2.1.
- `docs/DATA_MODEL.md §36.9` — checklist items 7 and 8 marked `[x]` Done. Item 7: SOC2_READINESS.md v3.9.1 (2026-06-15) completed OFB-E-005/OFB-E-006 registration. Item 8: OBSERVABILITY.md §12.7 added (corrects erroneous §12.3 cross-reference — §12.3 was occupied by OTel OTLP content).
- `VERSION` → 5.67.1

---

## [5.67.0] — 2026-06-16

### Added
- `content/post-108-training-cognitive-performance.md` — Editorial series post 108. «Тренування і когнітивна продуктивність — що спортивна наука реально показує». Гострий ефект аеробного навантаження на виконавчі функції (Lambourne & Tomporowski 2010), BDNF і хронічні структурні зміни (Erickson 2011), перевернута J-крива при overtrained стані (Meeusen 2013), тайминг, силове тренування і когніція (Liu-Ambrose 2010), сон як медіатор. 13-хв читання. clinical-safety: PASS.

### Changed
- `blog.html` — додано картку для post-108 (Editorial series). blog.html updated to post-108.
- `STATUS.md` — Editorial series оновлено 106–107 → 106–108. Total posts: 1065 → 1066.
- `README.md` — post-108 статус: proposed → draft · v5.67.0.
- `VERSION` → 5.67.0

---

## [5.66.0] — 2026-06-16

### Added
- `content/post-1061-hip-flexors-anatomy.md` — Block 1051–1100 cluster 3, post 1/5. Флексори стегна: iliopsoas (psoas major + iliacus) і rectus femoris. Бімартикулярність rectus femoris і механізм active insufficiency при присіданні. Пряме тренування флексорів: виправдане при спортивній специфіці або вираженому дефіциті. clinical-safety: NOT_REQUIRED.
- `content/post-1062-hamstrings-functional-anatomy.md` — Block 1051–1100 cluster 3, post 2/5. Задня поверхня стегна: biceps femoris (довга і коротка голова), semitendinosus, semimembranosus. Дві функції: розгинання стегна (довга голова) і згинання коліна (всі голови). Leg curl vs. RDL — різні вектори, не взаємозамінні. Nordic curl як ексцентрична добавка. clinical-safety: NOT_REQUIRED.
- `content/post-1063-triceps-three-heads.md` — Block 1051–1100 cluster 3, post 3/5. Трицепс: довга голова (бімартикулярна, починається від лопатки), медіальна і латеральна (мономартикулярні). Overhead extension → максимальний стимул для довгої голови у stretched position. Програма без overhead-роботи системно пропускає найбільший компонент трицепсу. clinical-safety: NOT_REQUIRED.
- `content/post-1064-biceps-brachii-anatomy.md` — Block 1051–1100 cluster 3, post 4/5. Biceps brachii: довга голова (supraglenoid tubercle) і коротка (coracoid process). Супінація передпліччя — основна функція, недооцінена при програмуванні. Incline curl → довга голова у stretched position. Hammer curl → brachialis, не biceps brachii. clinical-safety: NOT_REQUIRED.
- `content/post-1065-forearm-anatomy.md` — Block 1051–1100 cluster 3, post 5/5. Передпліччя: flexor digitorum profundus (основа сили хвату), extensor carpi radialis (стабілізатор зап'ястя при жимах). Crushing vs. support vs. pinch grip. Баланс флексорів і екстензорів. Farmer's carry і dead hang як функціональний підхід до хвату. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — додано картки для постів 1061–1065. blog.html updated to post-1065.
- `STATUS.md` — Block 1051–1100 оновлено: 10/50 → 15/50, cluster 3 COMPLETE. Total posts: 1060 → 1065.
- `VERSION` → 5.66.0

---

## [5.65.1] — 2026-06-16

### Added
- `docs/DECISION_LOG.md` — DEC-058 backfilled: k6 hybrid load test platform architecture (k6 Cloud EU-West quarterly PERF-SLO-06 reference + k6 OSS CI gate unchanged). Closes OQ-PERF-01 🟢 and OQ-PERF-02 🟢 from `docs/OBSERVABILITY.md §40.9`. Entry was referenced in OBSERVABILITY v4.1 (2026-06-14) but missing from the log.
- `docs/AUDIT_LOG_SCHEMA.md` — `system.quarterly_perf_reference_initiated` and `system.quarterly_perf_reference_completed` registered in System namespace (STANDARD, 3yr, HMAC-chained, DEC-058). PERF-CLOUD-CHAIN-01 ordering invariant documented. Retention table row added. Closes `docs/OBSERVABILITY.md §45.8` checklist item 2 (P0, M6).

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — version v2.11 → v2.12.
- `VERSION` → 5.65.1

---

## [5.65.0] — 2026-06-16

### Added
- `content/post-107-bad-day-vs-bad-block.md` — Editorial series post 107. Як відрізнити «поганий день» від «поганого блоку»: горизонт як єдиний інструмент, RPE-тренд, HRV rolling average, суб'єктивний аудит. Алгоритм 4 кроки для self-coached атлета. Kreher & Schwartz (2012) + Hooper et al. + Haff/Zourdos RPE-варіабельність. clinical-safety: PASS.

### Changed
- `blog.html` — картка post-107 додана перед post-106.
- `STATUS.md` — editorial series оновлено: post-107 written v5.65.0.
- `VERSION` → 5.65.0

---

## [5.64.0] — 2026-06-16

### Added
- `content/post-1056-deltoid-three-heads.md` — Block 1051–1100 cluster 2, post 1/5. Дельтоподібний м'яз: три голови (передня/середня/задня), анатомія SITS і механічна відмінність. Чому overhead press переважно навантажує передню голову; lateral raise — для середньої; rear delt fly / face pull — для задньої. EMG-дані і типовий дисбаланс. clinical-safety: NOT_REQUIRED.
- `content/post-1057-rotator-cuff-function.md` — Block 1051–1100 cluster 2, post 2/5. Ротаторна манжета: SITS (supraspinatus, infraspinatus, teres minor, subscapularis). Механізм force coupling при абдукції; стабілізуюча роль при жимі і тягах; ER:IR баланс (норма 0.65–0.75); практичні наслідки для програмування (face pull, ER-вправи). clinical-safety: NOT_REQUIRED.
- `content/post-1058-trapezius-three-zones.md` — Block 1051–1100 cluster 2, post 3/5. Трапецієподібний м'яз: верхня (елевація лопатки), середня (ретракція), нижня (депресія і ротація вниз) зони. Ротаційний force couple лопатки. Типова диспропорція: верхня перетренована, нижня в дефіциті. Y-raise і pulldown з депресією як корекція. clinical-safety: NOT_REQUIRED.
- `content/post-1059-deep-spinal-stabilizers.md` — Block 1051–1100 cluster 2, post 4/5. Глибокі стабілізатори хребта: multifidus (сегментарна стабілізація, превентивна активація), erector spinae (три колони: iliocostalis, longissimus, spinalis), transversus abdominis. Force coupling при deadlift і присіданні. Bird dog, dead bug, Pallof press як специфічне доповнення. clinical-safety: NOT_REQUIRED.
- `content/post-1060-calves-gastrocnemius-soleus.md` — Block 1051–1100 cluster 2, post 5/5. Литкові м'язи: gastrocnemius (бімартикулярний, type II, standing calf raise) vs. soleus (мономартикулярний, type I, seated calf raise). Чому кут коліна визначає, який м'яз навантажується. Ахіллове сухожилля як повільно-адаптуючий фактор. Повний ROM — пріоритет над вагою. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — картки posts 1056–1060 додані до сітки перед post-1055.
- `STATUS.md` — Block 1051–1100 оновлено: 10/50 написано (кластер 2 complete); next: posts 1061–1065.
- `VERSION` → 5.64.0

---

## [5.63.0] — 2026-06-16

### Added
- `content/post-106-weekly-microcycle-unit.md` — Editorial series post 106. Чому тижневий мікроцикл важливіший за тренувальну сесію: мікроцикл як функціональна одиниця адаптації, чому одна сесія — поганий вимірювач (варіабельність, шум vs. сигнал), що визначається на тижневому рівні (частота, обсяг, розподіл навантаження, відновлення між важкими сесіями), спрощена логіка ATL/CTL (Banister 1991, Borresen & Lambert 2009), практичний 5-хвилинний алгоритм тижневої діагностики, три ознаки правильно збудованого тижня, чотири проблемних паттерни і дії. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — картка post-106 додана до сітки перед post-50.
- `README.md` — Editorial series roadmap розширено: теми 106–111 додано до таблиці; post-106 позначено `draft · v5.63.0`.
- `STATUS.md` — Editorial series 11–34, 44–50 → 11–34, 44–50, 106; Total posts 1053 → 1054; версія v5.62.0 → v5.63.0.
- `VERSION` — bump 5.62.0 → 5.63.0.

---

## [5.62.0] — 2026-06-16

### Added
- `content/post-1051-gluteal-anatomy-layers.md` — Block 1051–1100 post 1/50. М'язова специфіка: сідничні м'язи — анатомія трьох шарів (gluteus maximus, medius, minimus), функції у розгинанні стегна і стабілізації таза, типова прогалина у програмах (фронтальна площина), EMG-дані по куту стегна та активації, аргумент на користь довгої м'язової довжини для гіпертрофії. clinical-safety: NOT_REQUIRED.
- `content/post-1052-biceps-femoris-vs-semitendinosus.md` — Block 1051–1100 post 2/50. Підколінні сухожилля: biceps femoris (довга і коротка голова) vs. semitendinosus/semimembranosus — різні ротаторні функції, односуглобова природа короткої голови, проксимальна vs. дистальна гіпертрофія, чому RDL і leg curl є взаємодоповненнями, а не альтернативами. clinical-safety: NOT_REQUIRED.
- `content/post-1053-quadriceps-vmo-myth.md` — Block 1051–1100 post 3/50. VMO-міф: чому вибіркова активація VMO не підтверджена EMG; rectus femoris як двосуглобовий і найчастіше недотренований; реальні відмінності в активації голів залежно від нахилу і глибини; leg extension як єдиний спосіб навантажити коротку голову без двосуглобового компромісу. clinical-safety: NOT_REQUIRED.
- `content/post-1054-latissimus-dorsi-function.md` — Block 1051–1100 post 4/50. Latissimus dorsi: три функції (приведення, розгинання, внутрішня ротація), чому pulldown і row дають різний механічний стимул, EMG-порівняння, teres major як синергіст, аргумент для максимального розтягнення у верхній точці тяги. clinical-safety: NOT_REQUIRED.
- `content/post-1055-pectoralis-major-heads.md` — Block 1051–1100 post 5/50. Pectoralis major: клавікулярна і стернальна голови — різні точки початку, різна механічна перевага; оптимальний нахил 30–45° для клавікулярної; чому зведення не замінює жим; суб'єктивне «відчуття» не є індикатором механічного стимулу. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — картки posts 1051–1055 додані до сітки перед post-1050.
- `VERSION` — bump 5.61.0 → 5.62.0.

---

## [5.61.0] — 2026-06-16

### Added
- `content/post-50-synthesis-form-defaults.md` — Editorial series post 50. Синтез: десять рішень за замовчуванням для self-coached атлета — частота (3–4x/тиж), структура сесії (45–75 хв), прогресивне перевантаження (вага як основний вектор), RPE 7–8 для більшості робочих підходів, деload кожні 4–6 тижнів (плановий), реакція на тренди а не точки, техніка (безпека/ефективність/відтворюваність), «поганий день» протокол (−30–40% обсяг), відпочинок 2–3 хв між важкими підходами, 8-тижневий блок як одиниця планування. Де дефолти не спрацьовують: початківці, відновлення після перерви, спортивний контекст, клінічні обмеження. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — картка post-50 додана до сітки перед post-49.
- `README.md` — post-50 статус змінено на `draft · v5.61.0`.
- `STATUS.md` — Editorial series 11–49 → 11–50; Total posts 1047 → 1048; версія v5.60.0 → v5.61.0.
- `VERSION` — bump 5.60.0 → 5.61.0.

---

## [5.60.0] — 2026-06-16

### Added
- `content/post-1042-personal-benchmark-sessions.md` — Block 1001–1050 cluster 5, post 6/9. Персональні контрольні точки: що таке benchmark-сесія, чотири критерії вибору вправи, субмаксимальний протокол 5×3, коли проводити, дві типові помилки. clinical-safety: NOT_REQUIRED.
- `content/post-1043-long-term-training-trends.md` — Block 1001–1050 cluster 5, post 7/9. Довгострокові тренди: чому короткі вікна обманюють, три рівні читання (тижневий/квартальний/річний), що шукати у річному архіві, хроніка vs. архів. clinical-safety: NOT_REQUIRED.
- `content/post-1044-autoregulation-data-based-deviations.md` — Block 1001–1050 cluster 5, post 8/9. Autoregulation на основі даних: чотири рівні відхилення (0–3) з RPE-порогами і діями, як визначити базовий RPE розминки, autoregulation і обсяг. clinical-safety: NOT_REQUIRED.
- `content/post-1045-data-for-coach-feedback.md` — Block 1001–1050 cluster 5, post 9/9. Що давати тренеру: три рівні звіту (мінімальний/стандартний/блоковий), як формулювати конкретні питання, що не надсилати. clinical-safety: NOT_REQUIRED.
- `content/post-1046-training-log-format.md` — Block 1001–1050 cluster 6, post 1/5. Формат журналу: три функції, мінімальні поля, паперовий vs. цифровий, що не фіксувати, консистентність термінології. clinical-safety: NOT_REQUIRED.
- `content/post-1047-recovery-deficit-signals.md` — Block 1001–1050 cluster 6, post 2/5. Сигнали відновлення в даних: чотири ранні сигнали накопиченої втоми, що не є надійним сигналом, три рівні реакції, деload vs. відпочинок. clinical-safety: NOT_REQUIRED.
- `content/post-1048-block-retrospective-protocol.md` — Block 1001–1050 cluster 6, post 3/5. Ретроспектива блоку: 8-крокова процедура (план/факт, контрольна точка, добре/погано, паттерн, обсяг, урок, 3 рішення). clinical-safety: NOT_REQUIRED.
- `content/post-1049-reading-annual-archive.md` — Block 1001–1050 cluster 6, post 4/5. Річний перегляд: шість секцій, умови достатнього архіву, шаблон підсумкової сторінки. clinical-safety: NOT_REQUIRED.
- `content/post-1050-training-literacy-block-close.md` — Block 1001–1050 CLOSER. П'ять зсувів у мисленні, що закриває блок: від даних до сигналів, від тренування до трендів, від жорсткого плану до обґрунтованої гнучкості, від читання до комунікації, від поточного до архівного мислення. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — posts 1042–1050 added (9 entries before post-1041).
- `STATUS.md` — Block 1001–1050 marked COMPLETE 50/50.
- `VERSION` — bump 5.59.1 → 5.60.0.

---

## [5.59.1] — 2026-06-16

### Changed
- `docs/DECISION_LOG.md` — DEC-061 entry added (was referenced in `docs/DATA_MODEL.md §36` and `docs/SOC2_READINESS.md §67.8/§67.9` but never recorded here). EU-region R2 bucket routing for enterprise off-boarding data egress: `resolveEgressBucket()` routes EU tenants (`eu-central-1`, `eu-west-1`) to `form-offboarding-exports-eu`; US tenants to `form-offboarding-exports`; enforced by `chk_r2_bucket_region_consistency` CHECK constraint (migration 0075) and OFB-REGION-01 HMAC-chain invariant in `enterprise.data_export_completed`. Options B (SCCs + single US bucket) and C (EU bucket + SCCs) rejected: TIA maintenance burden under Schrems II, FISA §702 / CLOUD Act exposure, EU DPO friction. Closes OQ-OFB-02 (P0). Owners: enterprise-architect + compliance-officer + devops-lead + legal.
- `VERSION` — bump 5.59.0 → 5.59.1.

---

## [5.59.0] — 2026-06-16

### Added
- `content/post-49-goals-vs-process.md` — Editorial series post 49. Мета або процес: чому спортивні цілі часто підводять — структурні дефекти цілі як системи управління тренуванням (бінарний результат без зворотного зв'язку, довільний дедлайн, відсутність методу); нелінійна адаптація і конфлікт із фіксованим дедлайном; що робить процес корисним (зворотний зв'язок на кожному кроці, не зупиняється після успіху, розрізняє помилку від збою); де ціль залишається корисною (орієнтир напрямку, точка перевірки, тижневий маркер); практичний 5-кроковий алгоритм. clinical-safety: PASS.

### Changed
- `blog.html` — картка post-49 додана до сітки перед post-48.
- `README.md` — post-49 статус змінено на `draft · v5.59.0`; roadmap розширено posts 96–105 (блок 96–105: «Що означає прогрес», «5 індикаторів тренованості», «Власна vs. готова програма», «Більше ≠ краще», синтез-100, тренувальний контекст, змінна-помилка, читання досліджень, self-coached патерни, 5-річна перспектива).
- `STATUS.md` — Editorial series 11–48 → 11–49; Total posts 1037 → 1038.
- `VERSION` — bump 5.58.0 → 5.59.0.

---

## [5.58.0] — 2026-06-16

### Added
- `content/post-1037-data-vs-feel-signal-hierarchy.md` — Block 1001–1050, cluster 5 post 1/5. Ієрархія сигналів «дані проти відчуттів»: три рівні конфлікту (короткостроковий/тижневий/блоковий), різна ієрархія для кожного горизонту. Три ситуації де суб'єктивне відчуття завжди переважає. Три ситуації де дані завжди правдивіші. Чотирикроковий алгоритм вирішення конфлікту. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1038-minimal-tracking-system.md` — Block 1001–1050, cluster 5 post 2/5. Мінімальна система відстеження: принцип мінімальної достатності, п'ять обов'язкових точок даних (дата, робочі підходи, RPE, стан 1–10, один коментар). Що додавати за потреби (HRV, відео, харчування). Що не потрібно фіксувати ніколи (розминочні підходи, час відпочинку, суб'єктивні оцінки кожної вправи). Три перевірочних запитання. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1039-review-cadence-daily-weekly-block.md` — Block 1001–1050, cluster 5 post 3/5. Ритм огляду: три горизонти аналізу — денний (2 хв, орієнтація і фіксація), тижневий (10–15 хв, адгерентність/RPE-тренд/коментарі/пропуски), блоковий (30–45 хв, п'ять секцій включаючи рішення). Чому надто часта перевірка шкодить. Коли частоту варто підвищити. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1040-data-to-decision-algorithms.md` — Block 1001–1050, cluster 5 post 4/5. П'ять алгоритмів для п'яти типових ситуацій: (1) навантаження і RPE зростають одночасно — лінійний тренд vs. стрибок; (2) показники стоять 4+ тижні — адгерентність, RPE-тренд, тип прогресії; (3) прогрес не в пріоритетній вправі — перевірка об'єму, частоти, RPE, техніки; (4) дані хороші але суб'єктивний стан погіршується — відставання числових сигналів, зовнішні стресори, деload; (5) суперечливі сигнали — зведення до двох метрик, пошук точки розбіжності. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1041-post-session-logging-protocol.md` — Block 1001–1050, cluster 5 post 5/5. Протокол запису після сесії 90 секунд: чому «повний» журнал не виживає (закон мінімальної достатності), п'ять елементів (дата/тип, ключові числа, RPE, стан 1–10, одне речення). Де і як вести (нотатки телефону / таблиця / папір). Коли записувати. Правило трьох для першого місяця. Ознака живої системи. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — картки post-1037–1041 додано до сітки перед post-1036 (cluster 5 start).
- `STATUS.md` — block 1001–1050 оновлено: 36/50 → 41/50; cluster 5 (1037–1041) COMPLETE; total posts 1032 → 1037.

---

## [5.57.0] — 2026-06-15

### Added
- `content/post-48-technique-not-perfectionism.md` — Editorial series post 48. Техніка — не перфекціонізм: три критерії «достатньої» техніки (безпека структурного ланцюга, активація цільового м'яза, відтворюваність руху); де «досить добре» вистачає (новачковий/early intermediate, анатомічні обмеження, нормальний технічний дрейф під прогресією); де техніка критична (відсутність активації цільового м'яза, компенсаторні патерни, відсутність ексцентричного контролю); чотири сигнали що вимагають зупинки; практичний протокол «коли виправляти, коли продовжувати». clinical-safety: PASS. sports-scientist review pending.

### Changed
- `blog.html` — картка post-48 додана до сітки перед post-47.
- `README.md` — post-48 статус змінено на `draft · v5.57.0`; roadmap розширено posts 86–95 (блок 86–95: «Розумний атлет» — суперкомпенсація, scheduling за відновленням, різниця важкого і продуктивного, блокові моделі, вибір коуча, RPE-журнал, авторегуляція).
- `STATUS.md` — Editorial series 11–48; Total posts: 1032.
- `VERSION` — bump 5.56.0 → 5.57.0.

## [5.56.0] — 2026-06-15

### Added
- `content/post-1032-anchoring-effect-training-baselines.md` — Block 1001–1050 cluster 4 post 2/6. Ефект якоря у тренуванні: перший блок або перший PR як фіксована точка відліку. Чому «я раніше присідав X» читає поточні дані гірше ніж вони є. Плато адаптації як ефект якоря (не невдача). Протокол рекалібрації кожні 12–16 тижнів. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1033-sunk-cost-fallacy-program-switching.md` — Block 1001–1050 cluster 4 post 3/6. Sunk cost fallacy у перемиканні програм: реальна ціна відстроченого рішення у тижнях адаптації. Різниця між «програма не працює» та «програмі не дали часу». П'ять-питань decision framework. Pre-block commitment statement. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1034-attribution-error-training-variables.md` — Block 1001–1050 cluster 4 post 4/6. Attribution error: приписування прогресу не тій змінній. Три класичні патерни хибної атрибуції. Вісім конфундуючих змінних що системно ховаються за очевидним. П'яти-кроковий протокол ізоляції змінних включно з реверсною перевіркою. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1035-selection-bias-training-tracking.md` — Block 1001–1050 cluster 4 post 5/6. Selection bias у тренувальному журналі: три форми упередженого логування. Феномен «хорошого дня» з числовим прикладом. Аудит-чекліст на ознаки selection bias. Pre-committed logging protocol: 5 полів, 5-хвилинне правило. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1036-anti-bias-review-protocols.md` — Block 1001–1050 cluster 4 post 6/6 (cluster synthesis). Двоярусна anti-bias система: місячний 20-хв огляд (3 фіксованих питання) + блоковий пост-мортем (6-секційний шаблон). Pre-mortem перед блоком. Чотири кількісних порогових значення що запускають обов'язковий перегляд: RPE +1.0 за 2 тижні, нульовий приріст за 3 тижні, 25% пропущених сесій, 70% compliance. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — картки post-1032 через post-1036 додані на початок сітки.
- `STATUS.md` — Block 1001–1050: 31/50 → 36/50; Cluster 4 COMPLETE.
- `VERSION` — bump 5.55.1 → 5.56.0.

## [5.55.1] — 2026-06-15

### Changed
- `enterprise.html` — White-label showcase section added between Implementation Timeline and Integrations. Covers: eligibility gate (≥ $50k ARR), three-step setup flow (DNS CNAME → logo/colour upload → FORM provisions Cloudflare Custom Hostnames), what's customisable (domain, logo, primary colour, Admin Dashboard Branding panel), what stays the same (RLS isolation, privacy floor, SOC 2 controls, 99.9% SLA), powered-by-FORM footer rule, eligibility card. Cross-refs: `docs/WHITE_LABEL_IMPLEMENTATION.md`, `docs/ENTERPRISE.md §Branding`, DEC-054. enterprise-architect + compliance-officer owners.
- `VERSION` — bump 5.55.0 → 5.55.1.

## [5.55.0] — 2026-06-15

### Added
- `content/post-1031-cognitive-biases-data-reading.md` — Block 1001–1050 cluster 4 post 1/10. Три когнітивні ухили у читанні тренувальних даних: підтверджувальне упередження (активне спростування, правило трьох суперечливих прикладів), упередження свіжості (мінімальний горизонт 3 тижні, ковзне середнє, явне зважування), self-serving bias (симетричний аналіз, «якби це був хтось інший», нейтральний запис причин). Взаємодія трьох ухилів і «антиупереджений мінімум» — 3 питання за 5 хв. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — картка post-1031 додана на початок сітки.
- `STATUS.md` — block 1001–1050 оновлено до 31/50; кластер 4 розпочато; total posts → 1031; VERSION 5.54.0 → 5.55.0.
- `VERSION` — bump 5.54.0 → 5.55.0.

## [5.54.0] — 2026-06-15

### Added
- `content/post-1022-warmup-sets-readiness-signal.md` — Block 1001–1050 cluster 3 post 1/9. Warm-up sets як термометр готовності: три маркери (bar speed, технічне зусилля, суб'єктивний RPE), порівняння з HRV як різних вимірювань різних речей, протокол фіксації, реакція на жовтий/червоний прапор, патерн накопиченої втоми через warm-up динаміку. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1023-progressive-overload-primary-metric.md` — Cluster 3 post 2/9. Чотири леверидж-метрики прогресивного перевантаження (інтенсивність, обсяг, тоннаж, щільність), логіка вибору первинної залежно від цілі, часові горизонти змін, помилка відстежування всього без ієрархії. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1024-technical-regression-in-data.md` — Cluster 3 post 3/9. Технічна регресія у даних: п'ять маркерів (RPE creep, аномалії в повторах, нотатки якості, асиметрії, зміна темпу), різниця від накопиченої втоми (тест deload), небезпечне поєднання «вага росте + техніка деградує», три рівні реакції. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1025-muscle-group-asymmetry-data.md` — Cluster 3 post 4/9. Асиметрія між сторонами: нормальна (≤15%, стабільна) vs. проблемна (прогресуюча, впливає на механіку), три джерела прогресії асиметрії, протокол вимірювання кожні 6–8 тижнів, білатеральний дефіцит, правило «починати зі слабшої сторони». clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1026-volume-adaptation-training-status.md` — Cluster 3 post 5/9. Адаптація до обсягу залежно від тренувального стажу: три стадії і різна логіка (початківець/середній/просунутий), різне читання однакових даних, MEV→MAV framework, пастка загальних нормативів без особистого контексту. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1027-real-plateau-vs-poor-analysis.md` — Cluster 3 post 6/9. Справжнє плато vs. поганий аналіз: шість типових хибних «плато», чеклист 6 пунктів перед діагнозом, ознаки справжнього адаптаційного плато, три правильних втручання (зміна стимулу / нарощення обсягу / deload reset), пастка «зміна програми кожні 6 тижнів». clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1028-training-age-data-interpretation.md` — Cluster 3 post 7/9. Тренувальний вік і інтерпретація даних: визначення (не роки в залі, а якісне програмоване тренування), нелінійна крива адаптації, таблиця орієнтирів (темп прогресу / горизонт діагностики / точність RPE / значущість відхилення / відновлення), три питання для самовизначення рівня. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1029-peak-planning-data-before-test.md` — Cluster 3 post 8/9. Планування піку через журнальні дані: визначення піку (мінімальна втома + збережена адаптація), tapering терміни за рівнем (7–21 день), сигнали зниження втоми і збереженої адаптації, протокол tapering (обсяг −40–60%, інтенсивність зберігається), читання сигналів в процесі, помилка «повний спокій», фіксація тесту. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1030-signal-hierarchy-when-data-conflicts.md` — Cluster 3 post 9/9 · synthesis. Ієрархія сигналів при конфлікті: чому конфлікт нормальний (різні інструменти = різні виміри), чотири конкретних сценарії з рішеннями, загальна ієрархія (warm-up > RPE тенденція > HRV тенденція > одиничні виміри > суб'єктивне відчуття), метасигнал часу аналізу. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — картки posts 1022–1030 додані на початок сітки (9 нових карток).
- `STATUS.md` — block 1001–1050 оновлено до 30/50; кластер 3 COMPLETE; total posts → 1030; VERSION 5.53.0 → 5.54.0.
- `VERSION` — bump 5.53.0 → 5.54.0.

## [5.53.0] — 2026-06-15

### Added
- `docs/WHITE_LABEL_IMPLEMENTATION.md` — Single-source enterprise white-label implementation guide: eligibility gate ($50k ARR, `branding.white_label_enabled` feature flag), architecture (Cloudflare Custom Hostnames + tenant_branding schema + R2 CDN), PAM-gated provisioning workflow, IT admin DNS CNAME setup guide, SSL cert lifecycle with pg_cron job 32 and AL-WL-01–06 escalation ladder (P2/Slack→P1/PagerDuty→P0/dual-page+no-auto-resolve), branding asset constraints (PNG/WebP only, WCAG AA colour validation, powered-by-FORM policy), Admin Dashboard panel spec, revocation (automated ARR-sweep + manual PAM-gated endpoint), six DEC-030 HMAC-chain events + WL-CHAIN-01 ordering invariant (cross-ref: AUDIT_LOG_SCHEMA §WhiteLabel, DEC-054), three SOC 2 evidence artefacts (WL-E-001 A1.1 / WL-E-002 CC7.2 / WL-E-003 CC7.2), four privacy constraints, four open questions (OQ-BRAND-01 / OQ-WL-OBS-02 / OQ-WL-MOB-01 / OQ-WL-EMAIL-01), 20-item implementation checklist (9× P0/M10, 7× P1/M11, 4× P2/M13). Cross-refs: ENTERPRISE.md §Branding, DATA_MODEL §20, OBSERVABILITY §42, ENTERPRISE_SLA §3, INCIDENT_RESPONSE R-05, COST_MODEL §15.5, DECISION_LOG DEC-054. compliance-officer + security-engineer review: NOT_REQUIRED (architecture/operations doc, no clinical content). enterprise-architect owner.

### Changed
- `VERSION` — bump 5.52.1 → 5.53.0.

## [5.52.1] — 2026-06-15

### Added
- `content/post-1021-retrospective-block-analysis.md` — Block 1001–1050 post 21/50 · cluster 3 opener. Ретроспектива блоку: 4 рівні аналізу — продуктивність (e1RM-тренд, 3 паттерни відповіді), виконання (compliance, RPE-відхилення, паузи), відновлення (5 сигналів боргу, де в тижні накопичувалось), техніка (межа нейром'язового резерву, повторювана помилка). Три питання для наступного блоку: що масштабуємо, що скорочуємо, що додаємо. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — картка post-1021 додана на початок сітки.
- `STATUS.md` — block 1001–1050 оновлено до 21/50; total posts 1020 → 1021; version 5.52.0 → 5.52.1.
- `VERSION` — bump 5.52.0 → 5.52.1.

## [5.52.0] — 2026-06-15

### Added
- `content/post-1013-volume-metrics-effective-sets-vs-tonnage.md` — Block 1001–1050 post 13/50. Метрики об'єму: ефективні сети проти тоннажу. Тоннаж як арифметика vs. ефективні сети як проксі гіпертрофічного стимулу. Формула RIR ≤ 4 для визначення ефективного сета. Коли тоннаж інформативний (міжциклова порівняльна оцінка, пауерліфтинг-специфіка), коли — оманливий (легка робота, заміна вправ). Мінімальний запис для обох метрик. Джерела: Krieger (2010), Schoenfeld et al. (2017), Schoenfeld & Grgic (2019). clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1014-session-rpe-load-metric.md` — Block 1001–1050 post 14/50. Session RPE: метод Foster (sRPE = RPE × тривалість у хвилинах = AU), монотонія (сер. навантаження / SD) і навантаження (монотонія × тижневий AU) як індикатори перетренованості. Де sRPE брехливий (технічні vs. метаболічні сесії). Орієнтир 1500–3000 AU/тиждень для тренованих силових атлетів. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1015-training-frequency-recovery-window.md` — Block 1001–1050 post 15/50. Частота тренувань як переговори з відновлювальною ємністю, а не текстбукові 48–72год. Як читати RPE першого підходу сесії як індикатор відновлення. Відмінність локального м'язового відновлення від системної ЦНС/ендокринної втоми. Протокол 4–6 тижнів для виявлення реального вікна відновлення на конкретній вправі. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1016-inter-set-rest-as-data.md` — Block 1001–1050 post 16/50. Міжсетовий відпочинок як сигнал готовності. Фосфокреатинова ресинтез (~74% за 1 хв, ~84% за 2 хв, ~95–99% за 3–5 хв). Дрейф паузи вгору (накопичена внутрішньосесійна втома, хибний підбір навантаження, вхідний дефіцит) vs. дрейф вниз (прогрес адаптації або недобір відновлення). Читання паузи на рівні тижня і мезоциклу. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1017-strength-ratio-tracking.md` — Block 1001–1050 post 17/50. Відстеження силових співвідношень: білатеральна асиметрія (поріг 15%), H:Q ratio (0.50–0.80, лог-апроксимація), натиск/тяга обсяговий баланс (1:1 до 1:1.5). Ознаки адаптації vs. компенсаторного патерну. Межі без ізокінетичного тестування. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1018-deload-signals-in-your-log.md` — Block 1001–1050 post 18/50. Сигнали розвантаження у логу: фіксований vs. авторегуляційний deload. П'ять лог-сигналів (e1RM-регрес у відпочилі дні, зростаючий RPE на субмакс, ранній технічний дрейф, ускладнена розминка, дрейф відпочинку). Одна сесія — шум, паттерн 2–3 тижні — сигнал. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1019-training-variability-nonlinear-log.md` — Block 1001–1050 post 19/50. Варіативність тренувань: монотонія за Foster (сер. AU / SD), чому висока монотонія корелює з симптомами ОТС навіть при помірному об'ємі. Запланована варіативність (важка/легка/середня) vs. випадкова непостійність. Три контексти де нелінійний лог нормальний (пікінг, навчання навику, навантажений побут). clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1020-minimalist-data-dashboard.md` — Block 1001–1050 post 20/50. Синтезуючий пост кластеру 2. Мінімальна панель даних: e1RM-тренд на ключовому русі (4-тижневий rolling), ефективні сети/тиждень/м'яз, sRPE vs. план (виконавча точність), compliance відпочинку, момент технічного збою в сесії. 10-хвилинний протокол тижневого огляду. Що робити коли метрики суперечать одна одній. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — картки post-1020, 1019, 1018, 1017, 1016, 1015, 1014, 1013 додані на початок сітки.
- `STATUS.md` — block 1001–1050 оновлено до 20/50; total posts 1012 → 1020; version 5.51.0 → 5.52.0.
- `VERSION` — bump 5.51.0 → 5.52.0.

## [5.51.0] — 2026-06-15

### Added
- `content/post-1012-e1rm-as-metric.md` — Block 1001–1050 post 12/50. Estimated 1RM як метрика: формули Epley/Brzycki/O'Conner, чому e1RM — інструмент тренду а не абсолютна цифра, RPE-контекст обов'язковий, системний запис контрольного підходу (зона 3–8 повт.), чотири комбінації e1RM+RPE і що вони означають, типові помилки, коли e1RM замінює реальний тест. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — картка post-1012 додана на початок сітки.
- `STATUS.md` — block 1001–1050 оновлено до 12/50; version 5.50.0 → 5.51.0.
- `VERSION` — bump 5.50.0 → 5.51.0.

## [5.50.0] — 2026-06-15

### Added
- `content/post-1011-technical-drift-under-fatigue.md` — Block 1001–1050 post 11/50. Технічний дрейф під втомою: три масштаби дрейфу (внутрішньопідхідний, внутрішньосесійний, міжсесійний), чому внутрішнє відчуття ненадійне, три системи вимірювання без відео (фіксовані точки контролю, темпова проксі, роздільний RPE-техніки / RPE-зусилля), маркери дрейфу по рухах, дрейф vs. техніка що розвивається. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — картка post-1011 додана на початок сітки.
- `STATUS.md` — block 1001–1050 оновлено до 11/50; total posts 1001 → 1011; version 5.48.0 → 5.50.0.
- `VERSION` — bump 5.49.0 → 5.50.0.

## [5.49.0] — 2026-06-15

### Added
- `content/post-1002-establishing-your-personal-baseline.md` — Block 1001–1050 post 2/50. Особистий «норм»: популяційні vs. індивідуальні норми, стабільна базова лінія, мінімум 4 вимірювань у контрольованих умовах, відрізняти «нижче норми» від «зовнішнє збурення». clinical-safety: PASS. sports-scientist review pending.
- `content/post-1003-trendline-over-single-datapoint.md` — Block 1001–1050 post 3/50. Трендова лінія vs. окрема точка: шум vs. сигнал у тренувальних даних, мінімальне вікно спостереження 4–6 тижнів, коефіцієнт варіації як поріг, практичні правила без складних інструментів. clinical-safety: PASS. sports-scientist review pending.
- `content/post-1004-errors-interpreting-your-progress.md` — Block 1001–1050 post 4/50. Помилки інтерпретації власного прогресу: recency bias, cherry-picking PR-ів, ігнорування контексту (сон, стрес), confirmation bias, плутанина варіації з регресом. clinical-safety: PASS. sports-scientist review pending.
- `content/post-1005-rpe-as-data-calibration.md` — Block 1001–1050 post 5/50. RPE як дані: калібрування суб'єктивного відчуття навантаження. Чому суб'єктивність ≠ ненадійність. Drift у RPE-перцепції. Перехресна перевірка RPE з реальним навантаженням для виявлення патернів. clinical-safety: PASS. sports-scientist review pending.
- `content/post-1006-hrv-heart-rate-real-signal.md` — Block 1001–1050 post 6/50. ЧСС і HRV: RMSSD та автономна НС, фактори що знижують HRV поза відновленням (таблиця), PPG vs. ECG точність, 7–14-денний тренд протокол, поріг для рішень. clinical-safety: PASS. sports-scientist review pending.
- `content/post-1007-sleep-as-training-variable.md` — Block 1001–1050 post 7/50. Сон як тренувальна змінна: SWS/N3 фізіологія (GH-секреція, ресинтез глікогену), зниження 1ПМ на 3–8% після <6 год сну, ефект на RPE, таблиця метрик за цінністю для тренінгу. clinical-safety: PASS. sports-scientist review pending.
- `content/post-1008-energy-during-training-signal-or-noise.md` — Block 1001–1050 post 8/50. «Енергія»: arousal (НС-активація) vs. readiness (реальне відновлення). Таблиця false-signal сценаріїв, протокол рішення. clinical-safety: PASS. sports-scientist review pending.
- `content/post-1009-real-adaptation-vs-temporary-adaptation.md` — Block 1001–1050 post 9/50. Реальна vs. тимчасова адаптація: три рівні (нейральна 2–8тиж, гіпертрофічна 2–6+ міс, сполучнотканинна місяці–роки), ознаки в журналі, три найпоширеніших помилки читання. clinical-safety: PASS. sports-scientist review pending.
- `content/post-1010-plateau-diagnosis-or-normal-process.md` — Block 1001–1050 post 10/50. Плато: чотири умови реального плато vs. нормальне сповільнення. Чотири сценарії псевдо-плато, діагностична послідовність питань, таблиця рішень. clinical-safety: PASS. sports-scientist review pending.

### Changed
- `blog.html` — картки додані для post-1010, 1009, 1008, 1007, 1006, 1005, 1004, 1003, 1002. blog.html тепер охоплює post-1010.
- `STATUS.md` — block 1001–1050 оновлено до 10/50; total posts 992 → 1001; next priorities оновлено.
- `VERSION` — bump 5.48.0 → 5.49.0.

---

## [5.48.0] — 2026-06-15

### Added
- `content/post-1001-log-to-diagnostic.md` — Block 1001–1050 post 1/50. Від запису до читання: як тренувальний журнал стає діагностичним інструментом. Три часові рамки читання (сесія / тиждень / блок), сигнал vs. шум, базова лінія як ключовий інструмент, три типові помилки читання, схема виходу на рішення (спостереження → гіпотеза → диференціація → рішення). clinical-safety: PASS. sports-scientist review pending.

### Changed
- `blog.html` — картка додана для post-1001. blog.html тепер охоплює post-1001.
- `VERSION` — bump 5.47.0 → 5.48.0.
- `STATUS.md` — Block 1001–1050 відкрито (1/50); Total posts 992; v5.48.0.

---

## [5.47.0] — 2026-06-15

### Added
- `content/post-996-training-log-systems-tools.md` — Block 951–1000 post 46/50. Тренувальний журнал: системи і цифрові інструменти. Різниця між журналом-архівом і журналом-інструментом. Мінімальний набір полів (дата/вправа/вага×повт×підх/RPE/нотатки). Аналоговий vs. цифровий форматсис; дворівнева система (операційний + аналітичний). Як використовувати дані для прийняття рішень (pre-session, post-block). Типові помилки: перфекціонізм, запис без аналізу, надмірна деталізація. clinical-safety: PASS. sports-scientist review pending.
- `content/post-997-gym-etiquette-unwritten-rules.md` — Block 951–1000 post 47/50. Негласні правила залу. Правила простору і обладнання (гриф на підлозі, «врізатись», повернення блинів). Правила взаємодії (непрошені поради, страхування, соціалізація). Гігієна і простір тренування. Один принцип що пояснює всі правила: мої дії не ускладнюють чужий процес. clinical-safety: PASS. sports-scientist review pending.
- `content/post-998-finding-best-training-time.md` — Block 951–1000 post 48/50. Коли тренуватись. Chtourou & Souissi (2012, J Strength Cond Res) мета-аналіз 22 досліджень: перевага другої половини дня 1–5% (обнуляється при регулярних тренуваннях у той самий час). Хронотип (CLOCK/PER3 гени), практичний тест. Три реальні фактори: послідовність > оптимальність, конкуренція з пріоритетами, відновлення навколо сесії. Чотири практичних фільтри вибору. clinical-safety: PASS. sports-scientist review pending.
- `content/post-999-block-synthesis-951-1000.md` — Block 951–1000 post 49/50. Синтез блоку: карта 5 кластерів, центральна ідея (контекст — змінна програми), що змінює знання цього блоку на рівні практики/мислення/прогресу, відкриті питання блоку. clinical-safety: PASS. sports-scientist review pending.
- `content/post-1000-milestone-roadmap-1001-1050.md` — **MILESTONE: пост 1000.** Карта 1000 постів (10 блоків × ~100), що залишилось незмінним (принципи, формат, аудиторія), і повний roadmap блоку 1001–1050 «Тренувальна грамотність: читання власних даних» (50 тем: читання журналу, хронотип біомаркери, адаптаційні сигнали, n-of-1, довгострокові тренди, особисті feedback loop системи). clinical-safety: PASS. sports-scientist review pending.

### Changed
- `blog.html` — картки додано для постів 996–1000 (5 карток). blog.html тепер охоплює post-1000.
- `VERSION` — bump 5.46.0 → 5.47.0.
- `STATUS.md` — Block 951–1000 COMPLETE 50/50; Total posts 985 → 991 (includes post-1000, post-980 VETO); Block 1001–1050 roadmap оновлено; v5.47.0.

---

## [5.46.0] — 2026-06-15

### Added
- `content/post-991-training-partner-vs-solo.md` — Block 951–1000 post 41/50. Партнер по тренуванню vs. самостійне тренування. Barbalho et al. (2017) — систематично нижче навантаження без страхувальника. Carron et al. (2004) coactive effect і його змінність. Операційні витрати партнерства (розклад, темп). Технічні рішення для безпеки без партнера (стійки з безпеками, відео). clinical-safety: PASS. sports-scientist review pending.
- `content/post-992-training-jet-lag-travel.md` — Block 951–1000 post 42/50. Тренування при міжнародних перельотах і джетлагу. SCN-десинхронізація, темпи адаптації (1 год/добу захід vs. 0.5 год/добу схід). Wright et al. (2002) і Drust et al. (2005) — циркадний пік о 16–19 год. Herxheimer & Petrie Cochrane (2002) — мелатонін 0.5–3 мг. Покроковий протокол від вильоту до 4-го дня. clinical-safety: PASS. sports-scientist review pending.
- `content/post-993-overreaching-vs-underrecovery.md` — Block 951–1000 post 43/50. Перетренованість vs. недовідновлення. FOR/NFOR/OTS таксономія (Kreher & Schwartz 2012, Meeusen et al. 2013 ECSS/ACSM консенсус). Об'єктивні маркери: HRV (Plews et al. 2013), силовий тренд, POMS/DALDA. Алгоритм реакції за тривалістю симптомів. clinical-safety: CONDITIONAL PASS — guard rail: стійка втома і зміни настрою 2+ тижні потребують медичної консультації. sports-scientist review pending.
- `content/post-994-training-without-gym-access.md` — Block 951–1000 post 44/50. Тренування без доступу до спортзалу. Mujika & Padilla (2000) — темпи детренування. Egner et al. (2013, FASEB) — м'язова пам'ять. Ефективні вправи без обладнання. Мінімальне обладнання (турнік → гиря → стрічки). clinical-safety: PASS. sports-scientist review pending.
- `content/post-995-inconsistent-schedule-programming.md` — Block 951–1000 post 45/50. Прогрес при непостійному доступі. Ralston et al. (2017) і Schoenfeld et al. (2017) — частотний мета-аналіз і мінімальна доза. Full body як надійніший режим. Блочне мислення (10 тренувань, не 10 тижнів). RPE-авторегуляція. clinical-safety: PASS. sports-scientist review pending.

### Changed
- `blog.html` — картки додано для постів 986–995 (10 карток).
- `VERSION` — bump 5.45.0 → 5.46.0.
- `STATUS.md` — Block 951–1000 count 39 → 44/50; Total posts 980 → 985; next priorities updated.

---

## [5.45.0] — 2026-06-15

### Added
- `content/post-987-training-heat-humidity.md` — Block 951–1000 post 37/50. Тренування в спеку та вологість: серцево-судинний дрейф і down-регуляція рекрутування моторних одиниць при підвищеній температурі ядра (Gonzalez-Alonso et al. 2008). Вплив дегідратації 2–3% на силові показники (Judelson et al. 2007, JSCR). WBGT як точніший метр ніж повітряна температура. Акліматизація 10–14 днів (Armstrong & Maresh 1991, Periard et al. 2016). Decision tree: перші 7 днів −20–30% об'єму, після 14 — повернення до норми. clinical-safety: PASS. sports-scientist review pending.
- `content/post-988-altitude-training-strength.md` — Block 951–1000 post 38/50. Тренування на висоті для силових атлетів: чесний огляд доказової бази. Hamlin et al. (2010) і Scott et al. (2003/2006) — непереконливі результати гіпоксичного силового тренування. Altitude маска — розвінчаний міф. Поріг 2000–2500 м для помітних ефектів. Акліматизація: дні 1–3 (падіння продуктивності), тиждень 2 (стабілізація), 3–4 тижні (повне пристосування). Найгірший момент для змагань — 3–10 день після прильоту. clinical-safety: PASS. sports-scientist review pending.
- `content/post-989-training-psychological-stress.md` — Block 951–1000 post 39/50. Тренування при підвищеному психологічному стресі: виключно фізіологічне фреймування. HPA-вісь і кортизол: конфлікт з mTORC1/MPS (McEwen 1998 алостатичне навантаження). Kiely et al. (2012, Sports Medicine) — нетренувальні стресори і адаптація. HRV як практичний біомаркер: decision tree на основі SD-смуг від базової лінії. Рекомендація: знизити об'єм (не до нуля, не частоту), зберегти основні рухи, уникнути максимальних зусиль. Footer disclaimer — спрямування до фахівця при ментальних труднощах. clinical-safety: CONDITIONAL PASS. sports-scientist review pending.
- `content/post-990-training-log-analysis.md` — Block 951–1000 post 40/50. Аналіз тренувального щоденника для самопрограмування: n-of-1 методологія. Hubal et al. (2005, JAP) — варіабельність відповіді −2% до +59% при однаковому протоколі. Мінімум 8–12 тижнів даних для виявлення паттерну. Що логувати (вага/повторення/RPE/сон/нотатки). Чотири типових помилки інтерпретації. Протокол виявлення особистого rep-range preference через контрольний порівняльний блок. clinical-safety: PASS. sports-scientist review pending.

### Changed
- `VERSION` — bump 5.44.0 → 5.45.0.
- `STATUS.md` — Block 951–1000 count 35 → 39/50; next priorities updated.

---

## [5.44.0] — 2026-06-15

### Added
- `content/post-986-training-suboptimal-sleep.md` — Block 951–1000 post 36/50. Тренування при нестачі сну: фізіологічний механізм (пригнічення тестостерону при < 6 год: Leproult & Van Cauter 2011, зниження MPS через скорочення SWS, підвищення RPE при тому ж навантаженні). Практичні порогові орієнтири 5/6/7 год з конкретними корекціями. Пріоритет скорочення об'єму над інтенсивністю; що пропустити (максимальні синглети, нові технічні рухи) і що зберегти (основні складені рухи, частоту). Гостра vs. хронічна депривація. clinical-safety: PASS. sports-scientist review pending.

### Changed
- `VERSION` — bump 5.43.1 → 5.44.0.

---

## [5.43.1] — 2026-06-15

### Changed
- `docs/MSA_TEMPLATE.md` — Addendum 3 (EU Data Residency Terms / DPA Annex B) authored; Exhibit C SCC bullet made conditional for EU-region Customers; header corrected v0.1 → v0.3. Closes `docs/DATA_MODEL.md §36.9` checklist item 6 (P0, before first EU enterprise DPA). compliance-officer + enterprise-architect owners.
- `docs/DATA_MODEL.md` — §36.9 checklist item 6 marked [x] Done with MSA_TEMPLATE v0.3 reference.
- `VERSION` — bump 5.43.0 → 5.43.1.

---

## [5.43.0] — 2026-06-15

### Added
- `content/post-47-first-year-self-programming.md` — Editorial series post 47. «Перший рік self-programming: типові помилки і що вони говорять». Шість передбачуваних помилок першого року самостійного програмування (надто складна програма на старті, зміна програми без діагнозу, ігнорування прогресії, копіювання обсягів просунутих атлетів, надто рання спеціалізація, програмування тільки для ідеальних умов), що кожна говорить про розуміння процесу, алгоритм виправлення. 13-хв read. Editorial-brutalist tone. clinical-safety: PASS.

### Changed
- `blog.html` — blog card для post-47 додано першою у grid; оновлено до post-47.
- `README.md` — Editorial roadmap: post-47 статус → draft · v5.43.0; нові теми 76–85 (block 76–85: «Що відрізняє "програму" від "тренувальної системи"») додано до таблиці.
- `STATUS.md` — Editorial series рядок оновлено: 11–34, 44–46 → 11–34, 44–47; Total posts 975 → 976; version v5.42.0 → v5.43.0.
- `VERSION` — bump 5.42.0 → 5.43.0.

---

## [5.42.0] — 2026-06-15

### Added
- `content/post-981-limited-mobility-training.md` — Block 951–1000 post 31/50. Тренування з обмеженою рухливістю: два типи обмежень (структурне vs. функціональне), таблиця замінників за 5 патернами руху (knee-dominant, hip hinge, horizontal push, vertical pull/push), адаптації при обмеженій дорсифлексії гомілковостопа, зовнішній ротації плеча і флексії кульшового суглоба, принцип «замінюй вправу не категорію руху», 5-кроковий аудит програми. clinical-safety: CONDITIONAL PASS (ROM adaptation, not pain management; physiotherapy disclaimer included). sports-scientist review pending.
- `content/post-982-programming-over-50.md` — Block 951–1000 post 32/50. Програмування після 50: гормональні зміни (тестостерон, естроген після менопаузи), саркопенія (~3–8%/декаду, зупиняється силовим тренінгом), сповільнення відновлення ЦНС і сполучної тканини. Адаптації: вища частота при меншому об'ємі на сесію (full body 3×/тижд. vs. PPL), збереження інтенсивності при зниженні об'єму, 2–3 RIR на складених рухах, 12–15 хв розминка, хвилеподібна прогресія (4-тижневий цикл), deload кожні 4–6 тижнів, пріоритет unilateral і loaded carries. clinical-safety NOT REQUIRED. sports-scientist review pending.
- `content/post-983-training-after-long-break.md` — Block 951–1000 post 33/50. Повернення після тривалої перерви: деадаптація по фазах (нейром'язова за 2–4 тижні, м'язова за 4–8+ тижнів), механізм м'язової пам'яті (збереження міонуклеусів → швидше відновлення vs. первинне набирання), темп відновлення (1–2 тижні нейром'язово, 2–4 тижні м'язово при перервах до 8 тижнів), програмування повернення (60–70% об'єму при збереженій інтенсивності → +10%/тиждень), шаблон першого тижня, управління DOMS. clinical-safety NOT REQUIRED. sports-scientist review pending.
- `content/post-984-periodization-non-competitive.md` — Block 951–1000 post 34/50. Periodization для атлетів без змагань: три проблеми одноманітного тренінгу (акомодація, накопичення втоми без розвантаження, відсутність вектору), порівняння linear/block/DUP periodization з таблицею рекомендацій за ситуацією, функція deload як профілактика (не реакція на симптоми), авторегуляція (RPE/RIR/HRV) як доповнення до структури, 12-тижневий шаблон циклу (accumulation → transmutation → synthesis). clinical-safety NOT REQUIRED. sports-scientist review pending.
- `content/post-985-womens-programming.md` — Block 951–1000 post 35/50. Жіноче програмування: спільні механізми (гіпертрофія, специфічність, відносний відгук), реальні відмінності (рівень тестостерону, частка волокон типу I, менструальний цикл — фолікулярна/лютеїнова фаза), практичні адаптації (вищий rep range як опція, авторегуляція через RPE), поширені перекоси у «жіночих програмах» (надмірна ізоляція замість складених рухів). clinical-safety: CONDITIONAL PASS (фізіологія і програмування без body image, нейтральний науковий тон щодо циклу). sports-scientist review pending.

### Changed
- `blog.html` — 5 нових blog cards (posts 981–985) додано першими у grid; оновлено до post-985.
- `STATUS.md` — Block 951–1000 row updated: 29/50 → 34/50; Next priorities оновлено (posts 986–990 як наступний батч); Total posts 970 → 975; version v5.41.0 → v5.42.0.
- `VERSION` — bump 5.41.0 → 5.42.0.

---

## [5.41.0] — 2026-06-15

### Added
- `docs/AUDIT_LOG_SCHEMA.md` v2.11 — +12 Enterprise Tenant Offboarding & Data Egress events (DEC-030 HMAC-chained · DATA_MODEL §25 + DEC-061). Closes DATA_MODEL §25.13 checklist item 9 (P0, M4) and DATA_MODEL §36.9 checklist item 5 (P0, M8 — DEC-061 EU-routing payload extension). All 12 events registered: `enterprise.offboarding_initiated` / `enterprise.offboarding_on_hold` / `enterprise.offboarding_hold_released` / `enterprise.sso_scim_revoked` / `enterprise.data_export_started` / `enterprise.data_export_completed` (extended with `data_region`, `r2_bucket`, `is_eu_region`, `package_size_bytes` per DEC-061) / `enterprise.data_deletion_started` / `enterprise.data_deletion_completed` / `enterprise.financial_pseudonymized` / `enterprise.deletion_attestation_issued` / `enterprise.offboarding_completed` / `enterprise.offboarding_step_failed`. OFB-REGION-01 chain invariant for `enterprise.data_export_completed` (EU `is_eu_region=true` → `r2_bucket='form-offboarding-exports-eu'`; HTTP 422 + P1 PagerDuty on violation). Five Zod v2 schemas. Six SOC 2 evidence artefacts: OFB-E-001 (C1.2), OFB-E-002 (P8.0), OFB-E-003 (CC6.3), OFB-E-004 (CC9.1), OFB-E-005 (C1.1/P4.0/CC6.1 — quarterly EU chain export), OFB-E-006 (C1.1/CC6.1 — annual R2 jurisdiction screenshot). Two retention table rows added. `form_api` REVOKED; `compliance_reviewer` SELECT all documented.

### Changed
- `STATUS.md` — AUDIT_LOG_SCHEMA.md row added to Enterprise documentation table; version v5.40.0 → v5.41.0.
- `VERSION` — bump 5.40.0 → 5.41.0.

---

## [5.40.0] — 2026-06-15

### Added
- `content/post-976-minimalist-home-gym.md` — Block 951–1000 post 26/50. Мінімалістичний домашній зал: три бюджетні рівні (до 150 USD: гантелі + pull-up bar + стрічки; 150–500 USD: штанга + rack + лавка; 500+ USD: cable тренажер, гиря, кардіо). Що НЕ варто купувати першим (мультифункціональні станції, великі кардіо-тренажери, ізоляційні тренажери для дому). Вимоги до простору (2×2 / 3×3 / 4×4+ м, висота стелі). Підлога: horse stall mats. Повна програма для Level 1 (Upper A / Lower / Upper B). 6 векторів прогресії без нового обладнання. Чесний список де комерційний зал кращий. clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-977-hotel-training.md` — Block 951–1000 post 27/50. Тренування в готелі: оцінка умов (номер / готельний зал / вулиця), ціль при відрядженні — maintenance а не прогрес. Три протоколи: тільки номер без обладнання (30 хв Full Body), готельний зал з гантелями до 20–25 кг (40 хв), кардіо-обладнання (HIIT vs зона 2). Методи інтенсифікації body weight без ваги: темп, паузи, односторонні рухи, мінімальний відпочинок, складніша варіація. Сон і джетлаг як фактор. Мінімальний пакувальний набір (TRX/rings, стрічки, скакалка). clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-978-shift-worker-programming.md` — Block 951–1000 post 28/50. Програмування для змінного графіка: чому стандартне програмування не працює (дні тижня замість циклів відновлення, нестабільний сон, фізична втома від зміни, неможливість планувати наперед). Категорії змін: нічна, добова, ротаційна, 12-годинна. Template 12-годинного shift (4/4 ротація). Плаваючий тижневий цикл. MED (2 сесії на 7–10 днів). Таблиця позицій до зміни і типу тренування. Маркери стану для рішення про навантаження. clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-979-training-under-45min.md` — Block 951–1000 post 29/50. Тренування за 45 хвилин: чому стандартні програми довші ніж потрібно (переходи, надмірні паузи, ізоляційний «хвіст», надлишкова розминка). Мінімальний ефективний об'єм для гіпертрофії (Krieger 2010, Ralston 2017). Розподіл 45 хв: розминка 8 / основна робота 30 / завершення 7. Що залишити (складені рухи) і що прибрати. Суперсети антагоністів: правила поєднання і що погано поєднується. Full Body і Upper/Lower templates. Управління паузами відпочинку: коли скорочувати, коли ні. Коли 45 хв не вистачить. clinical-safety NOT REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — 4 нові картки: пости 976–979 додані вище post-975.
- `STATUS.md` — Block 951–1000 оновлено: 25/50 → 29/50; post-980 HARD VETO зафіксовано; Next priorities оновлено (981–985); версія v5.39.0 → v5.40.0.
- `VERSION` — bump 5.39.0 → 5.40.0.

---

## [5.39.0] — 2026-06-15

### Added
- `content/post-971-barbell-only-gym.md` — Block 951–1000 post 21/50. Barbell-only gym: що покриває штанга по кожній групі м'язів (quad/glute, hamstring, back/upper, chest, shoulder, triceps, biceps, calf, core), чого реально бракує без кабельної рами і ізоляції та як компенсувати. Повна 4-денна Upper/Lower програма (День A/B/C/D). Лінійна і хвилеподібна прогресія в barbell-only умовах. clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-972-training-without-mirrors.md` — Block 951–1000 post 22/50. Тренування без дзеркал: зовнішня vs. внутрішня зворотна лінія (proprioception). Що втрачається і що ні без візуального контролю. Чотири компенсаційні методи: відео, тактильні орієнтири, партнер, тренування з закритими очима. Класифікація вправ де відсутність дзеркала критична vs. некритична. clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-973-outdoor-training.md` — Block 951–1000 post 23/50. Тренування на вулиці: що покривають перекладина і підлога, що відкривають петлі/резинки/гиря. Три методи прогресії без ваги (ladder варіацій, density training, паузи і темп). Програма 3 дні (Upper push / Upper pull + core / Lower). Погодні і сезонні фактори. Чого не варто очікувати від outdoor-тренінгу. clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-974-gym-anxiety-first-time.md` — Block 951–1000 post 24/50. Тривога в залі: spotlight effect (Gilovich et al., 2000), що думають про новачків досвідчені атлети, 5 практичних стратегій (непіковий час, план заздалегідь, прості вправи, навушники, соціальний якір), timeline першого місяця, що робити якщо дискомфорт не зменшується. GUARD RAIL: пост адресує нормальний ситуативний дискомфорт, не клінічну тривогу. clinical-safety CONDITIONAL PASS. sports-scientist review required before publish.
- `content/post-975-training-while-sick.md` — Block 951–1000 post 25/50. Тренування під час хвороби: above-the-neck rule, чому лихоманка — тверде ні (ризик міокардиту, стресової кардіоміопатії), матриця рішень за симптомами (6 рядків), терміни повернення (легка ГРВІ vs. лихоманка vs. грип/COVID), вплив 1–14+ днів пропуску на тренувальні адаптації, етика в залі при симптомах. GUARD RAIL: не замінює лікаря. clinical-safety CONDITIONAL PASS. sports-scientist review required before publish.

### Changed
- `blog.html` — 5 нових карток: пости 971–975 додані вище post-970.
- `VERSION` — bump 5.38.1 → 5.39.0.

---

## [5.38.1] — 2026-06-15

### Changed
- `docs/SOC2_READINESS.md` — v3.9.1: OFB-E-005 + OFB-E-006 registered in §67.8 evidence table (EU off-boarding egress artefacts, DEC-061); CC6.1/C1.1 EU data residency row added to §67.9 criteria mapping; header corrected v3.8.3 → v3.9.1 (v3.9.0 was §83 SIEM calibration, header inadvertently not bumped at that time). Closes DATA_MODEL §36.9 P1/M11 checklist item.
- `docs/INCIDENT_RESPONSE.md` — v2.7: R-05 trigger extended with OFB-REGION-01 chain invariant violation (EU tenant off-boarding routed to wrong R2 bucket region → treat as potential data breach, investigate per R-05 forensic protocol, source: DATA_MODEL §36.6, DEC-061).
- `VERSION` — bump to 5.38.1.

---

## [5.38.0] — 2026-06-15

### Added
- `content/post-967-kettlebell-focused-training.md` — Block 951–1000 post 17/50. Equipment constraints: гирі як основний інструмент. Фізична різниця гирі від гантелі (зміщений центр мас, профіль балістичного руху). Балістичні vs. гриндові вправи (swing/snatch/clean vs. press/TGU/RDL). Де гиря виграє (балістика, малий простір, кондиціонування) і де програє (максимальна сила ніг, ізоляція, точна прогресія). Мінімальний та функціональний набір гирь. Техніка swing і rack position. Програма 3 дні (A: сила+балістика, B: кондиціонування+задня ланцюг, C: комплекс). 7 векторів прогресії без нової гирі. clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-968-limited-weight-selection.md` — Block 951–1000 post 18/50. Equipment constraints: обмежений вибір ваг. Чому стрибок +5 кг на гантелях занадто великий (20–25% приріст за одне тренування — понад межу нервово-м'язової адаптації). 7 векторів прогресії: об'єм (double progression), темп (3-1-1-0 запис), пауза (нижня точка), зменшення відпочинку, складніша варіація, більше підходів, методи інтенсифікації (drop set, rest-pause, cluster set). 4-тижневий план переходу між недоступними кроками ваги. Мікроблін і саморобні рішення. Таблиця «що робити при обмеженому наборі». clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-969-no-cable-machines.md` — Block 951–1000 post 19/50. Equipment constraints: тренування без cable-тренажерів. Три унікальні властивості cable (постійний натяг, регульований кут, уніполярні рухи). Де cable критично (face pull, rear delt fly, lat pulldown без перекладини) і де добре компенсується (row, curl, tri extension). Повна карта замін для грудей, спини, плечей, рук, нижньої частини тіла. Детальний розбір face pull: чотири альтернативи (band face pull, dumbbell rear delt fly лежачи, W-raise, Powell raise). Lateral raise без cable — три тактики компенсації. Upper/Lower програма 4 дні без cable-stack. clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-970-training-without-spotter.md` — Block 951–1000 post 20/50. Equipment constraints: тренування без страховника. Класифікація: де страховник критичний (bench/squat зі штангою до відмови без safeties) і де не потрібен (гантелі, тренажери, тяга, ізоляція). Bench press без страховника: 4 варіанти (safeties у rack, floor press, dump гантелей, RIR management). Squat без страховника: safeties, goblet/front squat, Bulgarian split squat. Загальні принципи: RIR 1 як ліміт без страховника у вільноваговних, чеклист обладнання, відеозапис. Таблиця «безпечна вправа без страховника». clinical-safety CONDITIONAL PASS — пост акцентує профілактику, описує всі ризики, не заохочує до ігнорування безпеки. sports-scientist review required before publish.

### Changed
- `blog.html` — додано картки для posts 967–970 на чолі списку.
- `STATUS.md` — Block 951–1000 оновлено: 16/50 → 20/50; версія v5.37.0 → v5.38.0; наступний пріоритет: posts 971–975.
- `VERSION` — bump to 5.38.0.

---

## [5.37.0] — 2026-06-15

### Added
- `content/post-962-no-squat-rack.md` — Block 951–1000 post 12/50. Equipment constraints: тренування без squat rack. Три аналізи: що вирішує rack і що без нього доступне, повна карта замін (Romanian deadlift + Bulgarian split squat → квадрицепси/задня поверхня стегна, floor press → жим без стійок, front squat з підлоги через hang clean), структура трьох тренувальних днів без rack, безпека жиму лежачи без страхувача (floor press опції). Тимчасовий vs. постійний сценарій. clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-963-machine-only-training.md` — Block 951–1000 post 13/50. Equipment constraints: тренування тільки на тренажерах. Що тренажери дають і чого не дають (стабілізатори, специфіка вільноваговних рухів). Повна карта тренажерів по групах (ноги, груди, спина, плечі, руки). Upper/Lower програма на тренажерах. Чотири стратегії прогресії при великому кроці ваги (темп, діапазон повторів, pause reps, drop set). Регрес при поверненні 5–15% — відновлюється за 2–4 тижні. clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-964-resistance-bands-primary.md` — Block 951–1000 post 14/50. Equipment constraints: гумові стрічки як основний інструмент. Профіль зростаючого опору: перевага для hip thrust і hip abduction, недолік для deadlift-патерну. Де bands кращі за все інше (hip abduction, band pull-apart, activation, assisted pull-ups). Програма на 3 дні тижня (повне тіло). Базовий набір для домашнього тренування. Прогресія без зміни стрічки (темп, ширина кроку, pause reps). clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-965-no-bench-available.md` — Block 951–1000 post 15/50. Equipment constraints: тренування без лавки. Floor press як основна заміна (техніка, вага на 5–15% менша від bench max). Push-up варіації по ефекту (incline/decline/weighted/archer). Dumbbell floor press vs. barbell floor press. Повна push-сесія без лавки (35 хв). Incline і decline без лавки (стілець, elevated push-up). Час до відновлення bench специфіки після повернення: 1–3 сесії. clinical-safety NOT REQUIRED. sports-scientist review required before publish.
- `content/post-966-equipment-occupied.md` — Block 951–1000 post 16/50. Equipment constraints: переповнений зал. Принцип заміни «функція, не назва» — карта замін для всіх основних патернів (push, pull, квадрицепси, задня поверхня стегна). Три тактики: перестановка порядку, суперсети з незалежними м'язами, заздалегідь визначений «план B». Коли варто зачекати (максимальна силова вправа, специфічний тренажер без аналога). Організація тижня при систематично переповненому залі. clinical-safety NOT REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — додано картки для posts 962–966 на чолі списку.
- `STATUS.md` — Block 951–1000 оновлено: 11/50 → 16/50; наступний пріоритет: posts 967–970 (kettlebell-focused, limited weight, no cables, no spotter).
- `VERSION` — bump to 5.37.0.

---

## [5.36.0] — 2026-06-15

### Added
- `docs/SOC2_READINESS.md §83` — SIEM calibration governance: resolves OQ-ANOM-01 (CR-01 per-tenant aggregation confirmed, DEC-057) and OQ-ANOM-02 (AL-SIEM-06 graduated shadow→P1 activation adopted, DEC-056). §83 covers two DEC-030 events (`system.siem_alert_activated`, `system.siem_calibration_verified`), two SOC 2 evidence artefacts (CALIB-E-001/CC4.1, CALIB-E-002/CC7.2), and a 9-item implementation checklist. SOC2_READINESS internal version v3.8.3 → v3.9.0. SOC 2 readiness ~99.1% (+0.1pp pending artefact filing).
- `docs/AUDIT_LOG_SCHEMA.md` v1.4 → v1.5 — Registers `system.siem_alert_activated` (STANDARD, 3yr, devops-lead emitter) and `system.siem_calibration_verified` (STANDARD, 3yr, CI post-step hook emitter) per DEC-030. Privacy invariant: no real tenant_id, no user_id, no health data. Closes SOC2_READINESS §83.4 registration checklist.

### Changed
- `docs/SOC2_READINESS.md §76.11` — OQ-ANOM-01 and OQ-ANOM-02 status updated from P1 Open → 🟢 Closed; cross-refs to §83 and OBSERVABILITY §44.
- `VERSION` — bump to 5.36.0.

## [5.35.0] — 2026-06-15

### Added
- `content/post-961-training-time-pressure.md` — Block 951–1000 post 11/50. Тренування при нестачі часу: 30 хв замість 60. Правила пріоритетів (рівень 1–3 вправ), три готові шаблони (нижня, верхня push, верхня pull), оптимальне скорочення відпочинку, коли суперсети допомагають, а коли шкодять. clinical-safety NOT REQUIRED.
- `blog.html` — post-961 картка додана.

### Changed
- `STATUS.md` — Block 951–1000 IN PROGRESS (11/50 written).
- `VERSION` — bump to 5.35.0.

## [5.34.0] — 2026-06-15

### Added
- `content/post-951-training-while-traveling.md` — Block 951–1000 post 1/50. Тренування у відрядженні: MED без залу. Деадаптація за тиждень мінімальна; вибір вправ у готельному номері (bodyweight priorty matrix); структура 25-хв підтримуючої сесії; коли сон > тренування.
- `content/post-952-home-training-dumbbells.md` — Block 951–1000 post 2/50. Домашній тренінг з гантелями: де ефективні (верх тіла, однобічна робота), де обмежені (нижня частина без достатньої ваги). Вага гантелей як критичний параметр. Повна A/B/C тижнева програма.
- `content/post-953-home-gym-vs-commercial-gym.md` — Block 951–1000 post 3/50. Домашній зал vs. комерційний: матриця рішення (бюджет, простір, обладнання, мотивація). Окупність 3–8 років. Де комерційний незамінний (cables, спеціалізоване обладнання).
- `content/post-954-training-in-heat.md` — Block 951–1000 post 4/50. Тренування у спеку: cardiovascular drift, зниження часу до виснаження. Силові вправи менш чутливі до жари. Теплова акліматизація 10–14 днів. Практика адаптації програми влітку.
- `content/post-955-training-in-cold.md` — Block 951–1000 post 5/50. Тренування в холод: нервова провідність і м'язова механіка при низькій температурі. Структура розминки. Перший підхід завжди гірший у холоді — норма, не поганий день. Cold exposure vs. тренування у холоді — різниця.
- `content/post-956-altitude-pressure-training.md` — Block 951–1000 post 6/50. Висота і атмосферний тиск: парціальний О₂ і м'язова продуктивність. Короткі анаеробні зусилля страждають <5%, об'ємна робота відчутніше. Акліматизація 10–14 днів. Практика для гірських поїздок.
- `content/post-957-gym-environment-noise-light.md` — Block 951–1000 post 7/50. Середовище залу: музика 120–145 BPM знижує RPE; social facilitation задокументований ефект; освітлення і хронотип; умовні рефлекси запаху; де оптимізувати, де перестати чекати ідеальних умов.
- `content/post-958-flexible-schedule-training.md` — Block 951–1000 post 8/50. Нефіксований графік: три системи (наступна доступна сесія, мінімальний тижневий об'єм, A/B за потребою). Пріоритет вправ при скороченій сесії. Один «якір» + решта гнучко.
- `content/post-959-morning-vs-evening-training.md` — Block 951–1000 post 9/50. Ранок vs. вечір: температура тіла, циркадний ритм, хронотип. Вечір дає незначну перевагу у піковій силі. Ранок — стабільність і відсутність денної втоми. Коли вечірні тренування порушують сон.
- `content/post-960-relocation-training-continuity.md` — Block 951–1000 post 10/50. Переїзд і тренування: організаційний, не фізіологічний виклик. М'язова деадаптація за 2–3 тижні мінімальна. Як знайти зал до відʼїзду. Fresh start effect як можливість.
- `blog.html` — posts 951–960 додані (Block 951–1000 Training environment & context).

### Changed
- `STATUS.md` — Block 951–1000 IN PROGRESS (10/50 written); next priorities updated.

## [5.33.0] — 2026-06-15

### Added
- `docs/DATA_MODEL.md §36` — OQ-OFB-02 Resolution: EU-Region R2 Bucket Routing for Enterprise Off-boarding Data Egress (DEC-061). EU-domiciled enterprise tenants (`eu-central-1`, `eu-west-1`) now route off-boarding egress packages to `form-offboarding-exports-eu` (Cloudflare R2 EU jurisdiction), eliminating GDPR Chapter V transfer concerns. Includes migration `0075_offboarding_r2_routing.sql` (`data_region` column + `chk_r2_bucket_region_consistency` CHECK), `resolveEgressBucket()` TypeScript routing function, OFB-REGION-01 chain invariant in `enterprise.data_export_completed`, two SOC 2 evidence artefacts (OFB-E-005 / OFB-E-006), and DPA Annex B clause update. Closes §25.9 OQ-OFB-02 (P0).

### Changed
- `docs/DATA_MODEL.md` — header v1.13 → v1.15 (v1.14 header update was omitted when §35 was added; both §35 and §36 credited in v1.15); TOC updated to add §35 and §36 entries; §25.9 OQ-OFB-02 updated to 🟢 Resolved (DEC-061).
- `VERSION` — 5.32.0 → 5.33.0.

---

## [5.32.0] — 2026-06-15

### Added
- `content/post-941-neural-adaptations-strength.md` — Block 901–950 post 41/50. Нейральні адаптації: рекрутування, rate coding, координація. Програмні імплікації для початківців і досвідчених. Таймлайн адаптацій.
- `content/post-942-tendon-connective-tissue-programming.md` — Block 901–950 post 42/50. Структурний лаг сухожилків vs. м'язи. Ізометрія як специфічний стимул. Протокол повернення після перерви з урахуванням сухожилкового фактора.
- `content/post-943-training-age-response-rate.md` — Block 901–950 post 43/50. Тренувальний вік ≠ хронологічний. Novice/intermediate/advanced: адаптаційна логіка, провідний механізм, одиниця прогресу. Практична матриця.
- `content/post-944-advanced-rpe-programming.md` — Block 901–950 post 44/50. RPE-дрейф як індикатор адаптації. RIR-based прогресія. Хвиля RPE через мезоцикл. Autoregulation між сесіями.
- `content/post-945-fatigue-management-training-year.md` — Block 901–950 post 45/50. Три горизонти втоми. Модель Fitness-Fatigue. Квартальне планування. Ознаки накопиченої втоми і типові помилки управління.
- `content/post-946-training-for-longevity.md` — Block 901–950 post 46/50. VO2max, м'язова маса і баланс як longevity-предиктори. Максимум vs. оптимум. Програмні корективи за декадами.
- `content/post-947-hormonal-responses-training.md` — Block 901–950 post 47/50. Тестостерон, кортизол, GH, IGF-1 і тренування. Чому локальний IGF-1 важливіший за системні гормональні піки. Деміфологізація «анаболічного вікна».
- `content/post-948-sleep-training-quality.md` — Block 901–950 post 48/50. Кількісні дані дефіциту сну (-20% жим після 4 ночей 6 год). GH-секреція, ЦНС-відновлення, глімфатична система. Денний сон як інструмент.
- `content/post-949-cognitive-fatigue-training-decisions.md` — Block 901–950 post 49/50. Когнітивна втома і фізична продуктивність (Marcora et al.). Чому RPE ненадійний після важкого когнітивного дня. Заздалегідь прийняті рішення для «важких» днів.
- `content/post-950-minimum-effective-dose-synthesis.md` — Block 901–950 post 50/50. MED: концепція і практика. Синтез 50 статей блоку через призму MED. Як знайти особистий MED і будувати програму навколо нього. Блок COMPLETE.
- `blog.html` — додано картки для постів 941–950.

### Changed
- `VERSION` — 5.31.0 → 5.32.0.
- `STATUS.md` — Block 901–950: BLOCK COMPLETE 50/50; Total posts: 950; next: Block 951–1000 proposed.

---

## [5.31.0] — 2026-06-15

### Added
- `content/post-931-advanced-deload-variations.md` — Block 901–950 post 31/50. П'ять типів делоаду: об'ємний, інтенсивний, частотний, модальний і повне відключення. Логіка вибору за типом накопиченої втоми. Матриця ситуацій.
- `content/post-932-block-vs-dup-periodization.md` — Block 901–950 post 32/50. Блокова vs. DUP: механіки, доказова база (Rhea 2002, Zourdos 2016), переваги і слабкості кожної, практична матриця вибору. Гібридний підхід.
- `content/post-933-autoregulation-tools-practice.md` — Block 901–950 post 33/50. RPE, RIR і VBT без ускладнення. Калібрування, де кожна надійна. APRE (Mann et al., 2010). Три рівні впровадження.
- `content/post-934-training-around-soreness.md` — Block 901–950 post 34/50. Три сценарії DOMS і реакції. Тимчасове зниження ізометричної сили. Repeated bout effect. Розмежування DOMS і суглобового/сухожильного болю.
- `content/post-935-exercise-selection-advanced.md` — Block 901–950 post 35/50. Три рівні ієрархії вправ. SAID-специфічність. Правила ротації для кожного рівня. Таблиця усунення слабких ланок у основних рухах.
- `content/post-936-progressive-overload-beyond-weight.md` — Block 901–950 post 36/50. Сім векторів прогресії поза вагою: об'єм, щільність, double progression, RIR-прогресія, техніка, частота, ексцентрик. Тоннаж як альтернативна метрика.
- `content/post-937-accumulation-vs-intensification.md` — Block 901–950 post 37/50. Логіка чергування блоків. Параметри акумуляції і інтенсифікації. Три помилки: вічна акумуляція, вічна інтенсифікація, надто короткі блоки. Реалістичний річний цикл.
- `content/post-938-taper-strategies-non-competitive.md` — Block 901–950 post 38/50. Тейпер для не-змагальних атлетів. Мета і фізіологічний механізм. Лінійний vs. ступінчастий. Практичний 7-денний план.
- `content/post-939-advanced-warmup-protocols.md` — Block 901–950 post 39/50. Три блоки розминки. Чому статичний стретч перед силовим знижує вивід. PAP і primer-підхід. Різниця протоколів для силового і гіпертрофічного блоку.
- `content/post-940-training-log-analysis.md` — Block 901–950 post 40/50. Три горизонти аналізу: тижневий, блоковий, міжблоковий. RPE-дрейф і тоннаж як метрики. Діагностика плато і регресів. Мінімальна структура таблиці.
- `blog.html` — додано записи для постів 931–940.

### Changed
- `VERSION` — 5.30.1 → 5.31.0.
- `STATUS.md` — Block 901–950: 40/50; next: posts 941–950.

---

## [5.30.1] — 2026-06-15

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.9 → v2.10: Mobile SSO Browser Mode events section added (`sso.mobile_browser_mode_set` STANDARD 7yr + `sso.mobile_webview_blocked` HIGH 7yr); Zod schemas, MOBILE-CHAIN-01 invariant, SOC 2 evidence artefact SSO-WEB-E-001, and retention table rows. Closes SSO_SCIM §30.13 item 4 (P0/M8).
- `docs/SSO_SCIM_IMPLEMENTATION.md` — §30.13 checklist items 4 and 6 marked [x] Done: item 4 closed by AUDIT_LOG_SCHEMA v2.10 (2026-06-15); item 6 closed by OBSERVABILITY v4.2 (2026-06-15).
- `VERSION` — 5.30.0 → 5.30.1.

---

## [5.30.0] — 2026-06-15

### Added
- `content/post-922-contrast-sets-conjugate.md` — Block 901–950, post 22/50.
- `content/post-923-deload-methods-compared.md` — Block 901–950, post 23/50.
- `content/post-924-rm-testing-protocols.md` — Block 901–950, post 24/50.
- `content/post-925-strength-endurance-continuum.md` — Block 901–950, post 25/50.
- `content/post-926-rep-range-myths.md` — Block 901–950, post 26/50.
- `content/post-927-tempo-training-science.md` — Block 901–950, post 27/50.
- `content/post-928-intraset-rest-hypertrophy.md` — Block 901–950, post 28/50.
- `content/post-929-supersets-science.md` — Block 901–950, post 29/50.
- `content/post-930-fatigue-indicators-real-training.md` — Block 901–950, post 30/50.

### Changed
- `blog.html` — posts 922–930 додано; блок 901–950 оновлено до post-930.
- `STATUS.md` — Block 901–950: 30/50; Total posts: 930.
- `VERSION` — 5.29.0 → 5.30.0.

---

## [5.29.0] — 2026-06-15

### Added

- `content/post-921-wave-loading.md` — Block 901–950 post 21/50. Хвильове навантаження (wave loading): механізм PAP, фосфорилювання міозинових легких ланцюгів, вікно потенціація-втома. Варіації 3-2-1, 5-3-1, 2-1. Порівняння з прямими підходами і ramping. Протоколи дозування. Типові помилки. clinical-safety NOT REQUIRED.
- `content/post-922-contrast-sets-conjugate.md` — Block 901–950, post 22/50. Контрастні сети і спряжений метод: PAP-вікно. Contrast training vs. conjugate (сесія vs. мікроцикл). Dynamic Effort day і крива сила-швидкість. clinical-safety NOT REQUIRED.
- `content/post-923-deload-methods-compared.md` — Block 901–950, post 23/50. Методи делоаду: зниження об'єму vs. інтенсивності vs. частоти. Активний vs. пасивний. Планований vs. авторегуляторний. clinical-safety NOT REQUIRED.
- `content/post-924-rm-testing-protocols.md` — Block 901–950, post 24/50. Протоколи тестування 1ПМ: розминка, структура спроб, альтернативи (3–5ПМ, VBT). Технічний vs. абсолютний 1ПМ. clinical-safety NOT REQUIRED.
- `content/post-925-strength-endurance-continuum.md` — Block 901–950, post 25/50. Континуум сила–витривалість: карта зон, волокна I/IIa/IIx, ефект інтерференції (Wilson 2012, Murach & Bagley 2016). clinical-safety NOT REQUIRED.
- `content/post-926-rep-range-myths.md` — Block 901–950, post 26/50. Діапазони повторень: міти vs. наука. Schoenfeld 2017, Morton 2016. Механічне напруження, метаболічний стрес, м'язове пошкодження. clinical-safety NOT REQUIRED.
- `content/post-927-tempo-training-science.md` — Block 901–950, post 27/50. Темп повторення: нотація, TUT і докази. Суперповільний тренінг — без підтримки. Практичний темп для сили і гіпертрофії. clinical-safety NOT REQUIRED.
- `content/post-928-intraset-rest-hypertrophy.md` — Block 901–950, post 28/50. Внутрішньосетовий відпочинок і гіпертрофія: rest-pause і myo-reps. Ефективні повторення і логіка накопичення стимулу. clinical-safety NOT REQUIRED.
- `content/post-929-supersets-science.md` — Block 901–950, post 29/50. Суперсети: антагоністичні, агоністичні, перехресні. PAP-ефект (Robbins 2010). Час (Weakley et al.). clinical-safety NOT REQUIRED.
- `content/post-930-fatigue-indicators-real-training.md` — Block 901–950, post 30/50. Індикатори втоми: чотири компоненти, практичні сигнали, HRV межі, хронічні сигнали перевантаження. clinical-safety NOT REQUIRED.
- `README.md` — Block 1301–1350 «Mental models for physical training» (proposed) — додано до roadmap upstream.

### Changed

- `blog.html` — posts 921–930 додано (10 карток); блок 901–950 оновлено до post-930.
- `STATUS.md` — Block 901–950: 30/50; next priorities оновлено до posts 931–940.
- `VERSION` — 5.28.0 → 5.29.0.

---

## [5.28.0] — 2026-06-15

### Added

- `docs/runbooks/RB-ENT-WIN-LOSS-01.md` v1.0 — Enterprise Win/Loss Analytics: Quarterly Review & OQ-PIPE-01 Calibration. Closes `docs/COST_MODEL.md §39.10` P1 checklist item 7 (M10). Five canonical analytics queries from COST_MODEL §39.6 (stage conversion rates, loss reason distribution, win rate by tier, competitor category impact, sales cycle p50/p90); OQ-PIPE-01 four-step calibration protocol from §39.7 with PIPE-CHAIN-02 invariant; quarterly review cadence anchored to `enterprise.arr_bridge_closed` DEC-030 event; WIN-E-001/002/003 SOC 2 evidence actions with zero-count WIN-E-003 attestation procedure; founder calendar scheduling. Privacy floor: no company names, contact emails, or employee `user_id` in any query result or evidence file; `competitor_category` enum only; `form_api` REVOKED from `enterprise_deal_outcomes`. Owner: data-engineer + enterprise-architect + customer-success + compliance-officer.

### Changed

- `VERSION` — 5.27.0 → 5.28.0.

---

## [5.27.0] — 2026-06-15

### Added

- `content/post-911-cluster-sets.md` — Block 901–950 post 11/50. Кластерні сети: фізіологія PCr-ресинтезу і внутрішньосетового відпочинку. Три формати (класичний, 2+2+2, EMOM). Таблиця порівняння з традиційними сетами. Дозування в мезоциклі.
- `content/post-912-rest-pause-training.md` — Block 901–950 post 12/50. Rest-pause і Myo-reps (Fagerli): механізм рекрутування МО типу II через накопичену втому. Структура активаційного сету і myo-sets. Порівняльна таблиця протоколів. Типові помилки імплементації.
- `content/post-913-drop-sets-advanced.md` — Block 901–950 post 13/50. Дроп-сети: стандартний, механічний, тріо-дроп, частковий. Механіка рекрутування при зміні ваги. Правила дозування 1–2 вправи за сесію. Де дроп-сети небезпечні (компаунди під навантаженням).
- `content/post-914-accommodating-resistance-science.md` — Block 901–950 post 14/50. Адаптивний опір: крива сили і крива зовнішнього опору. Ланцюги (10–15% від загального навантаження). Гумові стрічки і прогресивний профіль. Dynamic effort / max effort. Розрахунок відсотків при адаптивному опорі.
- `content/post-915-intraset-fatigue.md` — Block 901–950 post 15/50. Внутрішньосетова втома: периферична (PCr, pH, Pi) і центральна (нейральна). Падіння швидкості штанги від повторення до повторення. RIR як відносна метрика: де хибить. Яка частина сету «дорожча» для сили vs. гіпертрофії.
- `content/post-916-periodizing-accessories.md` — Block 901–950 post 16/50. Програмування акцесуарних вправ: три функції (слабка ланка, гіпертрофійний об'єм, корекція дисбалансу). Ієрархія втоми і розміщення у тижні. Ротація між мезоциклами. Прогресія і правило «заповни пропуск».
- `content/post-917-competition-cycle-planning.md` — Block 901–950 post 17/50. Змагальний цикл 12–16 тижнів: накопичення → інтенсифікація → реалізація → пік. Параметри кожного блоку. Вибір опенера (90–93% ПМ). Топ-3 помилки піку. Зворотне планування від дати змагань.
- `content/post-918-exercise-order-effects.md` — Block 901–950 post 18/50. Порядок вправ: нейром'язова втома і вторинні вправи. Суперсети антагоністів (доказова база). Pre-fatigue: аналіз реального ефекту. Правило першого місця. Матриця залишкової втоми за типом вправи.
- `content/post-919-unilateral-advanced-training.md` — Block 901–950 post 19/50. Унілатеральний тренінг: bilateral deficit — механізм і magnitude. BSS як самостійний гіпертрофічний рух. Діагностика асиметрії (>10–15%). Стратегія корекції без вторинного дисбалансу. Мікроплатини для прогресії.
- `content/post-920-long-range-rom-training.md` — Block 901–950 post 20/50. Stretch-mediated hypertrophy: дослідження Maeo 2021, Pedrosa 2022. Крива довжина-напруження і роль тайтину. Lengthened partials vs. повний ROM: що показала наука. Таблиця вправ з акцентом на розтягнуту позицію. Ризики крайнього ROM.

### Changed

- `blog.html` — додано пости 911–920; загальна кількість відображуваних постів зросла до post-920.
- `STATUS.md` — Block 901–950 оновлено: 20/50 (posts 901–920); next: posts 921–930.
- `VERSION` — 5.26.1 → 5.27.0.

---

## [5.26.1] — 2026-06-15

### Changed

- `docs/SOC2_READINESS.md` v3.8.3 — DSAR-E-011 and DSAR-E-012 cross-reference patch. Closes `docs/DATA_MODEL.md §35.10` checklist item 9 (P1, M6). Two SOC 2 evidence artefacts defined in DATA_MODEL.md §35.9 (DEC-052, 2026-06-14) are now registered in SOC2_READINESS.md: DSAR-E-011 (P8.0 — quarterly `dsar.certificate_issued` DEC-030 chain export with R2 `payload_hash` cross-check; proves Art. 17 disposal communicated to data subject; 7yr; `compliance/evidence/dsar/DSAR-E-011_<YYYY-QN>.csv`) added to the P8 — Monitoring and Enforcement control table (§2); DSAR-E-012 (CC7.2 — AL-DSAR-05 PagerDuty incident history + pg_cron job 33 quarterly SLO miss counter reset audit; demonstrates automated DSAR SLO breach detection; 3yr) added to §25.7 CC7 evidence table after ETF-E-002. Vanta mirror list (§80.4) updated: DSAR-E-007/008/009/010 → DSAR-E-007/008/009/010/011/012. Both artefacts status: 🟡 Authored — pending first Art. 17 production erasure (M7). Privacy floor confirmed on both artefacts: no employee names, emails, or health values; `user_id_hash` SHA-256 only.
- `docs/DATA_MODEL.md` §35.10 item 9 — marked [x] Done with SOC2_READINESS.md v3.8.3 reference.
- `VERSION` — 5.26.0 → 5.26.1.

---

## [5.26.0] — 2026-06-15

### Added

- `content/post-901` through `content/post-910` — Block 901–950 «Advanced programming systems» opened (10/50). Posts written: post-901 Daily Undulating Periodization (DUP — Rhea 2002 evidence, 3-zone structure, weekly progression); post-902 Velocity-Based Training (MCV zones, load-velocity profile, velocity loss thresholds 10/20/25%, device comparison); post-903 Autoregulation beyond RPE (4 levels: RIR vs RPE, morning calibration, Range Scheme, floating deload triggers); post-904 Concurrent training interference (Wilson 2012 meta-analysis, mTOR/AMPK mechanism, moderators: cardio type/timing/order, 3 scheduling templates); post-905 Block vs. Conjugate periodization (Issurin vs. Westside, sequential vs. parallel adaptation, evidence base, decision matrix); post-906 Triphasic training — Cal Dietz (SSC physiology, eccentric/isometric/concentric block structure, evidence review, amateur adaptation); post-907 High-frequency low-volume / Grease the Groove (Schoenfeld freq meta-analysis, motor learning rationale, HFLV structures, GTG protocol); post-908 Fatigue management MRV/MEV/MAV (RP-model thresholds, dynamic MRV modifiers, two deload types, MEV during deficit); post-909 Peaking for natural lifters (2–3 week taper rationale, 48–72 h supercompensation window, weekly taper scheme, common errors); post-910 Program switching real cost (SRA cycle, DOMS ≠ efficacy, 4-level change cost hierarchy, decision checklist). All posts: clinical_safety NOT REQUIRED (pure programming topics, no food/body/pain/mental content); sports-scientist review required before publish; editorial-brutalist tone per DEC-007.
- `content/post-46-force-majeure.md` — Editorial series post № 46: «Форс-мажор у тренуванні: адаптація без паніки і без нульового місяця». Тема: мінімальний ефективний доз при порушенні режиму; фізіологія детренування (1–2 тижні, 2–4 тижні, 4+ тижнів); чотири сценарії з конкретними протоколами (робочий дедлайн, відрядження, легке нездоров'я, виснаженість); neck check rule; протокол повернення 60-70-80-90-100%. Мова: UA. Тон: editorial-brutalist. Clinical-safety: PASS. Pending: sports-scientist review.
- `blog.html` — posts 901–910 cards added to top of post list; картка post-46 додана.
- `README.md` — editorial series table: post-46 позначено draft · v5.26.0; додано нові proposed теми 61–75 (блок 61–75 «FORM editorial compass»).

### Changed

- `STATUS.md` — content table: block 851–900 row added (COMPLETE), block 901–950 row added (10/50 IN PROGRESS); total posts 910; editorial series рядок 11–34, 44–45 → 11–34, 44–46; next priorities updated.
- `VERSION` — 5.25.1 → 5.26.0.

---

## [5.25.1] — 2026-06-15

### Changed

- `docs/AUDIT_LOG_SCHEMA.md` v2.9 — §Enterprise Deal Close Governance events (DEC-030 HMAC-chained · COST_MODEL §39 · CC1.4/CC5.2/CC9.2) + §Privacy no-go commercial ethics events (DEC-030 HMAC-chained · COST_MODEL §39.8.4 · CC1.4/CC9.2) added. Closes COST_MODEL.md §39.10 checklist item 1 (P0, M9 — register all four §39.8 DEC-030 events before first S5 deal close). Four new events registered: `enterprise.deal_closed_won` (STANDARD, 7yr — full Zod schema with `floor_respected: true` literal, 8-value `win_primary_reason` enum, 6-value `won_vs_competitor_category` nullable enum, WIN-CHAIN-01 warning-level check for `implementation_kickoff_completed` predecessor within 72h); `enterprise.deal_closed_lost` (STANDARD, 3yr — 10-value `loss_primary_reason` enum, `no_go_criteria_triggered` boolean, last_stage_reached S0–S4, auto-companion `privacy.no_go_criteria_applied` emitted atomically when no_go flag true; 3yr retention rationale: no health data, two SOC 2 observation windows); `enterprise.win_loss_analysis_recalibrated` (STANDARD, 7yr — PIPE-CHAIN-02 blocking: HTTP 422 `PIPE_CHAIN_02_MISSING_DECISION_REF` if `decision_log_ref` null or empty; five stage conversion rate fields; `cost_model_section_updated: '§37.3'` literal; mirrors PIPE-CHAIN-01 on §37 `pipeline_conversion_model_recalibrated`); `privacy.no_go_criteria_applied` (STANDARD, 3yr — auto-companion only, never emitted standalone; `criteria_triggered` 4-value enum: insurance_risk_scoring/government_backdoor_request/wellness_as_punishment_use_case/other_no_go; `review_confirmed_by` founder UUID; zero-count quarterly filing affirmative CC1.4 attestation). Three SOC 2 evidence artefacts: WIN-E-001 (CC5.2/CC1.4 — annual `deal_closed_won` chain export with `floor_respected: true` verification, 7yr), WIN-E-002 (CC5.2/CC4.1 — `win_loss_analysis_recalibrated` at Deal 10/20/50, 7yr), WIN-E-003 (CC1.4/CC9.2 — quarterly `no_go_criteria_applied` export + zero-count attestation, 3yr). Privacy floor (all four events): no prospect company name, contact email, or individual employee `user_id`; `deal_id` FORM-internal UUID; `competitor_category` and `criteria_triggered` structured enums only; verbatim loss-call notes remain in CRM; `form_api` REVOKED from `enterprise_deal_outcomes`. Cross-references: COST_MODEL.md §39.8 (canonical Zod schemas — source of truth), §39.9 (WIN-E-001/002/003 evidence paths), §39.10 (P0 checklist item 1 — now closed); ENTERPRISE.md §"No-go customers"; DECISION_LOG.md DEC-06X (OQ-PIPE-01 closure at Deal 10).
- `VERSION` — 5.25.0 → 5.25.1.

---

## [5.25.0] — 2026-06-15

### Added

- `content/post-881` through `content/post-900` — Block 851–900 «Applied programming deep dives» complete (50/50). Posts 881–900 written: concurrent strength/hypertrophy programming (881), 1RM testing protocol (882), exercise order within session (883), HRV as programming tool (884), volume tolerance across training career (885), shift work programming (886), specialization phases (887), technique vs strength plateau diagnosis (888), accessory work purpose and volume (889), long-break return protocol (890), training environment effects (891), equipment decisions for programming (892), annual training plan structure (893), sleep and strength training (894), strength standards for serious amateurs (895), movement pattern balance (896), beginner-to-intermediate transition (897), evaluating program effectiveness (898), competition prep for recreational athletes (899), self-coached decision framework (900). All posts: sports-scientist review pending, clinical_safety_status: PASS, UA language, editorial-brutalist tone per DEC-007.
- `blog.html` — posts 881–900 added to top of post list.

### Changed

- `STATUS.md` — Block 851–900 marked COMPLETE 50/50; Block 901–950 proposed; next content horizon updated.
- `VERSION` — 5.24.0 → 5.25.0.

---

## [5.24.0] — 2026-06-15

### Added

- `content/post-45-progress-without-weight.md` — Editorial series post-45. «Прогрес без ваги штанги: що ще рахується і чому» — чому єдиний маркер «вага на штанзі» дає спотворену картину прогресу; шість повноцінних показників адаптації (об'єм навантаження/тоннаж, технічна компетентність, калібрування RIR/RPE, консистентність, швидкість відновлення, кінезіологічна доступність); мінімальний набір відстеження без надмірного аналізу; пастка «ніколи не дорівнює прогресу» — коли ширший набір метрик стає раціоналізацією; editorial-brutalist tone, 13 хв. clinical-safety: PASS. review_pending: sports-scientist.

### Changed

- `blog.html` — post-45 card prepended before post-44 (total 179 entries).
- `STATUS.md` — editorial series оновлено до 11–34 + 44–45; total posts 850; версія v5.24.0.
- `VERSION` — 5.23.1 → 5.24.0.

---

## [5.23.1] — 2026-06-15

### Added

- `docs/COST_MODEL.md §39` — Enterprise Deal Close Governance, Win/Loss Analytics & Competitive Intelligence Model (v2.4 → v2.5). Closes three gaps in the enterprise deal audit trail: (1) missing deal-level close events — `enterprise.deal_closed_won` (STANDARD, 7yr) and `enterprise.deal_closed_lost` (STANDARD, 3yr) with full Zod schemas and HMAC chain invariants (WIN-CHAIN-01 warning, PIPE-CHAIN-02 HTTP 422); (2) missing competitive intelligence schema — structured win/loss reason enums (8 win / 10 loss / 6 competitor_category values; no company names in chain, privacy floor enforced); (3) missing OQ-PIPE-01 (§37.11) calibration trigger — §39.7 four-step resolution protocol with §39.6.1 SQL query and `enterprise.win_loss_analysis_recalibrated` PIPE-CHAIN-02 event. `enterprise_deal_outcomes` Postgres DDL (migration 0074): UUID PK, `deal_id` FK RESTRICT to `enterprise_pipeline_stages`, four CHECK constraints, UNIQUE index on `deal_id`, `sales_cycle_days` GENERATED, `tcv_usd` GENERATED, `form_api` REVOKED; `'closed_lost'` added to `enterprise_pipeline_stage_enum`. Five analytics SQL queries (§39.6): stage conversion rate actuals (OQ-PIPE-01 gate ≥ 10 outcomes, no_go excluded), loss reason frequency, win rate by tier, competitor impact, sales cycle p50/p90. `privacy.no_go_criteria_applied` companion DEC-030 event (auto-emitted when `no_go_criteria_triggered = true` — CC1.4 ethical enforcement evidence; criteria_triggered enum covers insurance_risk_scoring / government_backdoor_request / wellness_as_punishment_use_case / other_no_go). Three SOC 2 evidence artefacts: WIN-E-001 (CC5.2/CC1.4 — annual deal_closed_won export with floor_respected attestation), WIN-E-002 (CC5.2/CC4.1 — win_loss_analysis_recalibrated at Deal 10/20/50), WIN-E-003 (CC1.4/CC9.2 — quarterly no_go_criteria_applied; zero-count filings are affirmative attestations). 10-item checklist (4× P0/M9, 3× P1/M10, 3× P2/M18–M36). OQ-WIN-01 (manual vs. automated deal_closed_won; manual recommended until Deal 5), OQ-WIN-02 (competitor_category unknown% threshold — evaluate at Deal 20). Four DEC-030 events to register in `docs/AUDIT_LOG_SCHEMA.md §Enterprise + §Privacy` before M9.

### Changed

- `docs/COST_MODEL.md` — header v2.4 → v2.5; TOC §39 entry added.
- `VERSION` — 5.23.0 → 5.23.1.

---

## [5.23.0] — 2026-06-15

### Added

- `content/post-44-training-age-identity.md` — Editorial series post-44. «Тренувальний вік: ким ти є як атлет — насправді» — чому рахунок «у роках» не відображає тренувальний рівень; три рівні (beginner/intermediate/advanced) і що вони означають для частоти, об'єму, інтенсивності і деловантаження; пастка «просунутого початківця» і як хибна самоідентифікація руйнує програмування; практичні маркери для чесної оцінки рівня; editorial-brutalist tone, 13 хв. clinical-safety: PASS. review_pending: sports-scientist.

### Changed

- `README.md` — Editorial roadmap розширено: пости 51–60 додано як proposed topics (як читати тренувальні дані, сила vs. маса, відпочинок між підходами, коли міняти програму, резерв на завершення тренування, warm-up для конкретного атлета, overreaching vs. overtraining, цілі що вимірюються, деадаптація, синтез блоку).
- `blog.html` — post-44 card prepended before post-34 (total 178 entries).
- `STATUS.md` — editorial series оновлено до 11–34 + 44; total posts 849; footer версія v5.23.0.
- `VERSION` — 5.22.0 → 5.23.0.

---

## [5.22.0] — 2026-06-15

### Added

- `content/post-871-strength-asymmetry.md` — Block 851–900 post 21/50. Асиметрія сили між сторонами: коли це проблема, коли — варіативність. 15% threshold as practical guide (not medical verdict), unilateral work as diagnostic vs. corrective tool, decision tree for programming response, common errors (single snapshot, aggressive correction). clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-872-adding-fourth-training-day.md` — Block 851–900 post 22/50. Коли додавати четверте тренування: критерії готовності. Why 3→4 days is harder than 4→5, recovery capacity as the actual constraint, 9-point readiness checklist, two structural variants for the 4th day, what NOT to count as a reason. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-873-programming-over-40.md` — Block 851–900 post 23/50. Програмування для атлета 40+: що реально змінюється. What actually changes (recovery rate, connective tissue adaptation, anabolic response); what doesn't (progressive overload, specificity); practical adjustments without conceding to progress. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-874-mid-block-plateau.md` — Block 851–900 post 24/50. Плато в середині блоку: три причини і три рішення. Distinguishing real stall from fatigue masking fitness; three causes (fatigue accumulation, wrong intensity zone, insufficient volume); three targeted responses; what to check before changing the program. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-875-accumulated-fatigue-response.md` — Block 851–900 post 25/50. Накопичена втома всередині блоку: ознаки і оперативна відповідь. How in-block fatigue differs from overtraining; real-time markers to monitor; operational responses without abandoning the block. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-876-planned-deload-mindset.md` — Block 851–900 post 26/50. Заплановане зниження без відчуття «я втрачаю прогрес». Supercompensation curve and why deloads feel psychologically wrong; how to structure a deload that preserves movement quality without volume. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-877-return-after-illness.md` — Block 851–900 post 27/50. Повернення після хвороби: перший тиждень без регресу. Why jumping to pre-illness loads causes setbacks; volume-first protocol (50–60% volume, maintain intensity); readiness signals for full resumption. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-878-rpe-vs-percentage.md` — Block 851–900 post 28/50. RPE vs. %1RM: коли авторегуляція переважає. Case for percentages (beginner predictability, peaking blocks); case for RPE (daily readiness variation, intermediate+); hybrid approach: percentages as anchor, RPE as real-time adjustment. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-879-bad-session-real-time-decisions.md` — Block 851–900 post 29/50. Погана сесія: рішення в залі в реальному часі. Distinguishing bad day / bad program / bad recovery; 3-tier warm-up-set decision tree; when to reduce, push, or leave. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-880-five-year-horizon.md` — Block 851–900 post 30/50. П'ятирічний горизонт: чому він змінює щоденні рішення. Injury avoidance ROI at 5-year horizon, volume accumulation compounding, how modest-consistent beats aggressive-and-injured. clinical-safety: PASS. review_pending: sports-scientist.

### Changed

- `README.md` — Block 851–900 table updated: actual titles for posts 856–870 replacing planned-topic placeholders; post-871 added; block status updated to in progress · 21/50 · v5.22.0; proposed topics 872–880 added to block table.
- `blog.html` — posts 871–880 entries prepended to article list (total 179 entries).

---

## [5.21.0] — 2026-06-15

### Added

- `docs/CHANGE_MANAGEMENT.md` — Standalone CC8.1 change management policy. Covers: change classification (Emergency/Standard/Minor), authorisation workflows, GitHub branch protection requirements (CC8-E-001), CI/CD pipeline state and CC8-GAP-001 compensating control, database schema change controls with mandatory reviewer checklist, rollback procedures for all four production surfaces (Workers/Pages/schema/ML model), separation of duties analysis with solo-founder compensating controls and auditor disclosure language, include-administrators override exception (CC8-E-005), six DEC-030 HMAC-chained audit events for the change lifecycle, full evidence package (CC8-E-001 through CC8-E-006), open gaps register (CC8-GAP-001: no CI yet; CC8-GAP-002: include-administrators off), Appendix A pre-deploy checklist (compensating control), Appendix B post-hire transition checklist. Extracted and formalised from `docs/SOC2_READINESS.md §21`. Owner: compliance-officer + security-engineer + enterprise-architect.

## [5.20.0] — 2026-06-15

### Added

- `content/post-866-compound-vs-isolation.md` — Block 851–900 post 16/20. Ізоляція vs. базові вправи: практичний баланс. Ієрархія вправ у сесії, розподіл 60/40 гіпертрофія vs. 80/20 сила, де ізоляція незамінна (задня дельта, медіальна голівка трицепса, ротаторна манжета). clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-867-failed-set-protocol.md` — Block 851–900 post 17/20. Провалений підхід: протокол дій. Три типи провалу (технічний розпад, репетиційний дефіцит, силовий відказ), покроковий протокол реакції, діагностика системних причин. Провал як інформація, а не катастрофа. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-868-amrap-as-diagnostic.md` — Block 851–900 post 18/20. AMRAP як діагностика прогресу. e1RM через формулу Epley і Brzycki, стандартизовані умови тестування, два способи застосування в циклі, тренд за кількома блоками. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-869-supersets-myo-reps.md` — Block 851–900 post 19/20. Суперсети і myo-reps: механіка і показання. Агоніст-антагоніст суперсети (-30–40% часу без компромісу якості), NC суперсети, pre-exhaustion (спірна ефективність). Myo-reps: активаційний сет + 4–5 мінікластерів з 20–30 с паузами. Показання і обмеження. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-870-block-review-protocol.md` — Block 851–900 post 20/20. Підбиття підсумків тренувального блоку: протокол після деліту. AMRAP-тест для порівняння e1RM, контрольні питання, дерево рішень для наступного циклу, шаблон 15-хвилинного огляду. clinical-safety: PASS. review_pending: sports-scientist.

### Changed

- `blog.html` — картки додані для постів 835–870 (36 карток); blog.html наздогнано з content/ directory; останній пост у grid: post-870.
- `STATUS.md` — Block 851–900: COMPLETE 20/20; blog.html caught up to post-870 (v5.20.0).

## [5.19.0] — 2026-06-15

### Added

- `content/post-856-autoregulation-rpe-rir.md` — Block 851–900 post 6/20. Авторегуляція: RPE і RIR як точний інструмент підтримки тренувального стимулу. Що вимірює шкала, як калібрувати RPE, практичні правила корекції навантаження при відхиленні ±2 пункти від цільового. Помилки: RPE-інфляція, sandbagging. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-857-peak-week-testing.md` — Block 851–900 post 7/20. Тейпер і пік для тестового тижня. Фізіологія тейперу (стомлення знижується швидше за фітнес), тривалість за тренувальним стажем (3–7 днів), протокол зниження об'єму 40–50% зі збереженням інтенсивності, феномен «taper tantrum», протокол розминки в день тесту. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-858-travel-minimal-equipment.md` — Block 851–900 post 8/20. Підтримчий тренінг без залу (2–4 тижні). Мінімальний ефективний стимул для підтримки (2 сесії/тиждень, 2–3 важкі підходи, ≥60% інтенсивності). Bodyweight-замінники з компенсацією через RIR. Протокол повернення після перерви. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-859-unilateral-training-integration.md` — Block 851–900 post 9/20. Інтеграція однобічних вправ. Білатеральний дефіцит, контралатеральний перенос, оцінка асиметрії. Однобічний рух як основний vs. допоміжний. Рекомендований об'єм, помилки (недооцінка як реабілітаційної роботи). clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-860-push-pull-ratio.md` — Block 851–900 post 10/20. Дисбаланс push:pull у self-coached атлетів. Мінімальне співвідношення 1:1, оптимальне 1:1.2–1.5 для горизонтальних рухів. Аудит програми. Практичний алгоритм виправлення без переписування. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-861-weekly-volume-distribution.md` — Block 851–900 post 11/20. Розподіл об'єму по тижню. Частота vs. об'єм на сесію, «junk volume» концепція, шаблони для 3- і 4-денних сплітів. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-862-rest-intervals.md` — Block 851–900 post 12/20. Паузи між підходами: PCr-відновлення, 3–5 хв для сили, 60–90 с для метаболічної роботи, 2–3 хв для гіпертрофії. Міф «коротший відпочинок = більший ріст». Масштабування пауз до інтенсивності. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-863-warm-up-protocol.md` — Block 851–900 post 13/20. Розминка: мінімально ефективний протокол. Загальна vs. специфічна розминка, формула прогресивного навантаження (50%×8 → 70%×3 → 85%×1 → робочі ваги), роль мобільності і foam rolling. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-864-irregular-schedule-programming.md` — Block 851–900 post 14/20. Програмування при нерегулярному графіку. Квотова модель (тижневі підходи) vs. фіксований розклад. Мінімум 2 сесії/тиждень. Пріоритет скорочень при стисненому тижні. Full body vs. split при нестабільному графіку. clinical-safety: PASS. review_pending: sports-scientist.
- `content/post-865-offseason-inseason-periodization.md` — Block 851–900 post 15/20. Periodization для атлетів без змагань. Визначення особистого offseason/inseason. Моделі: лінійна, хвильова (3:1), блокова. Накопичення–інтенсифікація–реалізація спрощено. Деліт як структурний елемент (не реактивний). Приклад річного планування. clinical-safety: PASS. review_pending: sports-scientist.

### Changed

- `STATUS.md` — content block 851–900 updated: 15/20 posts written (posts 856–865 added); posts 866–870 as next horizon.
- `VERSION` — 5.18.1 → 5.19.0.

---

## [5.18.1] — 2026-06-15

### Changed

- `docs/AUDIT_LOG_SCHEMA.md §Integration` — Expanded "Other integration events" stub into full event schemas. Added `integration.connector_added` and `integration.connector_removed` (STANDARD, 7yr retention, DEC-030 HMAC-chained) covering Slack/MS Teams/generic-HTTP notification connector lifecycle; Zod v2 schemas, privacy floor (`endpoint_url_hash` SHA-256 only, per DEC-054), SOC 2 CC9.2 mapping, ordering invariant. Added `integration.api_call` full spec (STANDARD, 30d retention, NOT HMAC-chained — sampled 1-in-10; explains why sampling disqualifies it from the evidence chain; Zod v2 schema; operational note for AL-API-02). Owner: compliance-officer + security-engineer.
- `VERSION` — 5.18.0 → 5.18.1.

---

## [5.18.0] — 2026-06-15

### Added

- `content/post-34-why-people-quit-training.md` — Editorial series post 34/50. Чому більшість людей кидають тренування — і що з цим робити без self-help риторики. Структурний аналіз adherence failure: пастка ідеального тижня, крепатура як хибний компас, об'єм як доказ зусилля, відсутність адаптивності програми. Кореляти стійкості (фіксований часовий слот, транзакційна вартість, адаптивність). Без self-help риторики, без «сили волі». clinical-safety PASS. review_pending: sports-scientist.

### Changed

- `blog.html` — Blog card added for post-34.
- `STATUS.md` — Editorial series updated: 11–34 drafted.
- `VERSION` — 5.17.0 → 5.18.0.

---

## [5.17.0] — 2026-06-15

### Added

- `content/post-851-lab-data-for-self-coached.md` — Block 851–900 post 1/20. Як читати і переносити лабораторні дані у тренінг: VO₂max (лаб vs гаджет), лактатний поріг як якір тренувальних зон, DEXA в контексті програмування, феритин/тестостерон/кортизол як контекст а не приписи. Проблема популяційних референсів для тренованих аматорів. clinical-safety PASS. review_pending: sports-scientist.
- `content/post-852-minimum-effective-stimulus-maintenance.md` — Block 851–900 post 2/20. Мінімальний ефективний стимул підтримки: differential decay нейром'язових vs структурних адаптацій, Issurin residual training effect, дослідження мінімальної частоти (Ralston 2017, Schoenfeld 2016, Bickel 2011, Moran 2019), інтенсивність vs об'єм у підтримці, вибір однієї вправи з максимальним carryover, реалістичні таймлайни утримання адаптацій. clinical-safety PASS. review_pending: sports-scientist.
- `content/post-853-strength-cardio-heart-rate-zones.md` — Block 851–900 post 3/20. Сила і зони ЧСС: деміфікація ефекту інтерференції (Hickson 1980, Murach & Bagley 2016), аеробна база як відновлювальний актив, 3-зонна модель для силовиків (не endurance), LISS vs HIIT за вартістю відновлення, операційні правила додавання кардіо. clinical-safety PASS. review_pending: sports-scientist.
- `content/post-854-training-log-diagnostic-tool.md` — Block 851–900 post 4/20. Тренувальний щоденник як dataset: RPE-trend аналіз як сигнал ранньої втоми, 4-тижнева таблиця патернів, 4–8 тижневий ретроспективний аналіз, 4 ключові патерни (адаптація → стаґнація, нерівний прогрес між рухами, кращі результати при нижчому об'ємі, кластеризація за часом доби), 3-рівневий протокол перегляду (тижневий/місячний/блоковий). clinical-safety PASS. review_pending: sports-scientist.
- `content/post-855-program-review-6-months.md` — Block 851–900 post 5/20. Перегляд програми після 6 місяців: 5 структурованих питань, розрізнення структурної помилки vs помилки виконання, концепція accumulated stimulus, правило мінімально необхідних змін, дерево рішень continue/modify/replace. clinical-safety PASS. review_pending: sports-scientist.

### Changed

- `README.md` — Block 851–900 table updated: topics → posts with numbered rows and status; 5/20 written (851–855).
- `STATUS.md` — Block 851–900 status updated: in progress 5/20.
- `VERSION` — 5.16.0 → 5.17.0.

---

## [5.16.0] — 2026-06-15

### Added

- `docs/COMPLIANCE_MATRIX.md` — Cross-framework compliance control matrix v1.0. 93 controls mapped across SOC 2 Type II (AICPA 2022), ISO 27001:2022 (Annex A), GDPR (Art. 5–89, emphasis on Art. 9 special-category health data), and HIPAA adjacency. Twelve domains: organisational controls, access control & identity, cryptography & data protection, physical & environmental, operations security, communications & network, supplier management, incident management, business continuity & DR, compliance & audit, privacy controls. Gap analysis per framework with P0/P1/P2 prioritisation. Evidence artifact cross-reference (15 artifacts, NDA-gated). Certification roadmap through 2028-Q1 (SOC 2 Type II → ISO 27001). Owner: compliance-officer + security-engineer + enterprise-architect. CM-E-001.

### Changed

- `VERSION` — 5.15.0 → 5.16.0.

---

## [5.15.0] — 2026-06-15

### Added

- `content/post-33-self-programming-vs-coach.md` — Editorial series post-33. Самостійне програмування vs тренер: зовнішнє спостереження як єдина незамінна перевага тренера, де self-programming виграє (проміжний рівень+, нерегулярний графік, якісні дані, вартість, автономія), де тренер виграє (перші 6-12 місяців, технічний розбір, змагальна підготовка, accountability, складні плато), спектр між полюсами (консультаційний тренер, online, програми, AI), практична рамка вибору з п'ятьма запитаннями, honest AI trade-off. clinical-safety: PASS.

### Changed

- `blog.html` — Blog card for post-33 added (editorial series card before post-32).
- `README.md` — Editorial series table extended: posts 41–50 added as proposed (RPE calibration, video analysis, training age, progress without weight, force majeure adaptation, first-year self-programming mistakes, technique thresholds, goal vs process, FORM default decisions synthesis).
- `STATUS.md` — Editorial series row updated: 11–32 → **11–33** v5.15.0; proposed next updated.
- `VERSION` — 5.14.0 → 5.15.0.

---

## [5.14.0] — 2026-06-15

### Added

- `content/post-841-electrolytes-performance-science.md` — Block 801–850 batch 5/5, post 1/10. Електроліти і спортивна продуктивність: натрій/калій/магній/кальцій втрати при навантаженні, debunk cramping-electrolyte myth (Schwellnus et al., Miller et al. neurogenic theory), гіпонатріємія ризик (Noakes 2012), когда ізотоніки потрібні vs. marketing. clinical-safety: PASS.
- `content/post-842-nitrate-beetroot-performance.md` — Нітрати і буряковий сік: нітрат→нітрит→NO механізм (Larsen 2007, Lansley 2011), Bailey et al. 2009/2010 landmark, Senefeld 2020 meta-analysis (80 RCT, SMD=0.33), 3–7% O₂ cost reduction, де ефект найсильніший (нетреновані, субмаксимальні навантаження < 30 хв), 5–9 ммоль NO₃⁻ дозування, guard rail: взаємодія з нітрогліцерином/PDE5-інгібіторами. clinical-safety: PASS.
- `content/post-843-sodium-bicarbonate-buffering.md` — Бікарбонат натрію: H⁺ буферизація і механізм втоми, Carr 2011 meta-analysis (29 досліджень, ES=0.44), 0.2–0.3 г/кг дозування, навантаження 1–7 хв і повторні спринти як основна ніша, ШКТ-дискомфорт та протоколи мінімізації, бета-аланін синергія (Hobson 2013). clinical-safety: PASS.
- `content/post-844-adaptogen-comparison.md` — Порівняльна таблиця адаптогенів: ашваганда/родіола/женьшень/елеутерокок — активні компоненти, доказовий рейтинг, стандартизація. Ашваганда KSM-66/Sensoril як найбільш доказово підтверджений. clinical-safety: PASS.
- `content/post-845-rhodiola-rosea-evidence.md` — Родіола рожева: розавін/сарозидин стандартизація SHR-5, Darbinyan 2000 і Shevtsov 2003 розумова втома, De Bock 2004 гострий прийом, RPE як найбільш послідовний фізичний ефект, порівняння з ашвагандою. clinical-safety: CONDITIONAL PASS — guard rail: SSRI/SNRI/MAOI взаємодії, ризик серотонінового синдрому.
- `content/post-846-tart-cherry-recovery.md` — Терпкий вишневий сік: Montmorency антоціани/кверцетин/мелатонін, Connolly 2006 (DOMS -40% сила втрата), Howatson 2010 (марафонці, ↓CRP/IL-6), Howatson 2012 (сон +34 хв), антиоксидантний парадокс discussion, протокол за 4–7 днів до важкого навантаження. clinical-safety: PASS.
- `content/post-847-glutathione-athletes.md` — Глутатіон і тренування: Ristow 2009 (антиоксиданти блокують адаптацію), ROS як редокс-сигнали (PGC-1α, Nrf2, MAPK), низька оральна біодоступність нативного GSH, як підвищити GSH природно. clinical-safety: PASS.
- `content/post-848-nac-athletes.md` — N-ацетилцистеїн (NAC): Merry 2010 (NAC блокує мітохондріальну адаптацію), Michailidis 2013 (↓p38 MAPK), парентеральний vs. оральний NAC розрив, коли NAC виправданий (змагальний пік, хвороба), коли контрпродуктивний (тренувальний блок). clinical-safety: CONDITIONAL PASS — guard rail: нітрогліцерин взаємодія, антиоксидантний парадокс для здорових атлетів.
- `content/post-849-phosphatidylserine-athletes.md` — Фосфатидилсерин: HPA-вісь механізм, Monteleone 1990/1992 (↓АКТГ/кортизол при фізстресі), Fahey&Pearl 1998 (↓20% кортизол після тренування), Starks 2008, соєвий vs. бичачий ФС, 400–800 мг дозування, порівняння з ашвагандою для кортизол-зниження. clinical-safety: PASS.
- `content/post-850-cordyceps-performance.md` — Кордицепс і аеробна продуктивність: C. sinensis vs. C. militaris практична різниця (ціна, доступність, кордицепін), Hirsch 2017 (VO₂max +7%), Brown 2019 (C. militaris 4 г/добу), Zheng 2021 meta-analysis (8 RCT, I²=73%), AMPK/PGC-1α механізм, стандартизація і якість ринку, block synthesis. clinical-safety: PASS.

### Changed

- `STATUS.md` — Block 801–850 row: 40/50 → **BLOCK COMPLETE** 50/50; total posts 838 → 848; next priorities updated; version header v5.11.1 → v5.14.0.
- `VERSION` — 5.13.1 → 5.14.0.

---

## [5.13.1] — 2026-06-15

### Added

- `docs/SSO_SCIM_IMPLEMENTATION.md §31` — OQ-SSO-24.1 + OQ-SSO-24.2 Resolution: `pam-db-proxy` Architecture — Supabase Edge Function & Direct Postgres Session Connection (DEC-060). Formally resolves two PAM open questions from §24.9 (P0 + P1, Before M4 deploy). OQ-SSO-24.1: Supabase Edge Function adopted over Cloudflare Worker + Hyperdrive — four rejection grounds for Hyperdrive: SET ROLE incompatibility in transaction mode, form_admin credential crossing provider trust boundary, cross-provider network latency (20–50 ms vs. < 5 ms intra-VPC), operational complexity. mTLS profile comparison: Cloudflare Access mTLS for Worker → Edge Function; Supabase-managed TLS for Edge Function → Postgres intra-VPC. OQ-SSO-24.2: Direct Postgres session connection (port 5432, `SUPABASE_PAM_DB_URL`) adopted; PgBouncer transaction mode (port 6543) bypassed entirely — SET ROLE form_admin is session-scoped and unreliable in PgBouncer transaction mode; direct connection provides native session semantics with no additional pool configuration. Connection ceiling analysis: ≤ 5–6 concurrent PAM direct connections vs. Supabase Pro plan 60-connection capacity — no plan upgrade required. Nine-step pam-db-proxy connection spec: open → SET ROLE form_admin → SET app.pam_session_id → execute → RESET ROLE → RESET app.pam_session_id → close (not pooled). CC6-E-PAM-003 evidence artefact updated scope. SUPABASE_PAM_DB_URL credential disambiguation table (vs. SUPABASE_SERVICE_ROLE_KEY and BREAK_GLASS_DB_URL). CRYPTOGRAPHY_POLICY.md §5 key inventory update required (P0, M4). Nine-item implementation checklist: 5× P0/M4-M5, 2× P1/M5, 2× P2/Annual.
- `docs/DECISION_LOG.md DEC-060` — pam-db-proxy architecture decision: Supabase Edge Function + direct port 5432 session connection adopted; Hyperdrive rejected.

### Changed

- `docs/SSO_SCIM_IMPLEMENTATION.md` — header v2.2 → v2.3; §24.9 OQ-SSO-24.1/24.2 marked 🟢 Resolved; §24.10 items 4 and 10 marked 🟢 Done.
- `VERSION` — 5.13.0 → 5.13.1.

---

## [5.13.0] — 2026-06-15

### Added

- `content/post-32-how-to-read-research.md` — Editorial series post-32. «Як читати наукову статтю про тренування і не зробити неправильних висновків» — ієрархія доказів (RCT vs observational vs мета-аналіз), population bias (нетреновані студенти vs тренований атлет), різниця між p-value і effect size (Cohen's d), суррогатні маркери vs прямі результати, publication bias і відтворюваність, практичний фільтр-таблиця для оцінки тверджень. Editorial-brutalist tone. clinical-safety: PASS. 13 хв.
- blog.html card for post-32.

### Changed

- `VERSION` — 5.12.0 → 5.13.0.
- `STATUS.md` — Editorial series row updated: post-32 written v5.13.0.

---

## [5.12.0] — 2026-06-15

### Added

- `content/post-837-bcaa-eaa-evidence-review.md` — Block 801–850 batch 4, post 7/10. BCAA vs EAA evidence review: mTORC1 substrate problem (Churchward-Venne 2014: 25 g whey > 6.25 g + BCAA for sustained MPS), Wolfe 2017 contextual rehabilitation of BCAA for specific scenarios (fasted training, low total protein, prolonged aerobic work), Moberg 2016 (EAA required for full mTOR activation), Davies 2023 meta-analysis (EAA 10–15 g effective when total protein < 1.2 g/kg; null at adequate intake). Practical algorithm: ≥ 1.6 g/kg/day from quality sources → BCAA/EAA add nothing; specific scenarios → EAA 10–15 g peri-workout. clinical-safety: PASS.
- `content/post-838-iron-deficiency-athletes.md` — Block 801–850 batch 4, post 8/10. Iron deficiency without anemia (IDNA) in athletes: ferritin as independent marker from Hb, IDNA prevalence 26–52% in female athletes and 7–15% in male endurance athletes, mitochondrial cytochrome iron-dependence (Complexes I–IV degraded before Hb falls), myoglobin and dopamine synthesis pathways, ferritin threshold zones for athletes (< 20 µg/L functional deficiency, 40–60+ optimal for active women), TIBC/transferrin saturation screening, heme vs non-heme bioavailability hierarchy, inhibitors (calcium, phytates, tannins), timeline to recovery (6–12 weeks for VO2max). Guard rails embedded: consult doctor before supplementing; iron toxicity and hemochromatosis contraindication noted. clinical-safety: CONDITIONAL PASS.
- `content/post-839-zinc-testosterone-evidence.md` — Block 801–850 batch 4, post 9/10. Zinc+testosterone without spin: Prasad 1996 (deficiency → repletion effect, not a booster; T doubled from 9 to 17 nmol/L in deficient elderly), Ananda et al. null in replete subjects, ZMA independent nulls (King 2003, Wilborn 2004 vs. Brilla & Conte 2000 conflict-of-interest). Mechanisms: Cu/Zn-SOD antioxidant, Leydig cell steroidogenesis cofactor, IGF-1 correlation, immune T-cell/NK-cell dependence. Practical algorithm: animal-protein-adequate → low deficit risk; plant-dominant + high training volume → check status, correct if deficient. Glycinate/picolinate forms, 40 mg/day UL, copper depletion risk with excess. clinical-safety: PASS.
- `content/post-840-berberine-glucose-metabolism.md` — Block 801–850 batch 4, post 10/10. Berberine+glucose for athletes: AMPK/GLUT4 mechanism (also mitochondrial Complex I inhibition, Akkermansia microbiome modulation), Zhang 2008 (500 mg×3, 3 months, T2D: HbA1c −2.0%, TG −35.9%, comparable to metformin), Liang 2019 meta-analysis (27 RCT, n=2569, significant for T2D/pre-diabetes). No RCT in healthy trained athletes. AMPK↑ vs. mTORC1 conflict flagged for strength athletes. CYP3A4/CYP2C9 inhibitor: warfarin, cyclosporin, metformin, tetracycline interactions documented. Contraindications: pregnancy, infants, severe liver disease. Guard rails embedded: consult doctor if on any medications; not a substitute for medical therapy in diabetes/pre-diabetes. clinical-safety: CONDITIONAL PASS.

### Changed

- `README.md` — Block 801–850 table updated: posts 831–840 all marked written (v5.12.0). Batch 4 COMPLETE annotation added.
- `STATUS.md` — Block 801–850 row updated: 36/50 → 40/50, batch 4 complete summary; total posts 833 → 837; next priorities updated to batch 841–850.
- `VERSION` — 5.11.2 → 5.12.0.

---

## [5.11.2] — 2026-06-15

### Changed

- `docs/OBSERVABILITY.md` — v4.1 → v4.2. Three cross-reference patches completing obligations from §30 (SSO_SCIM v2.2, 2026-06-14) and §44 (DEC-056/057, v4.1, 2026-06-14). **§26.8 `sso_browser_security` subsection added** — registers AL-SSO-WEB-01 (P1, PagerDuty `form-security`) for `sso.mobile_webview_blocked` DEC-030 HIGH events in the §6.2 condensed alert table; closes `docs/SSO_SCIM_IMPLEMENTATION.md §30.13` checklist item 6 (P0/M8); privacy floor: `attempted_url_hash` SHA-256[:32] only in payload — no plaintext IdP URL, no `user_id`, no health data; zero-fire is the expected production state. **§27.7 AL-SIEM-06 dual-phase patch** — replaces single P1 row with graduated-activation spec per §44.2.4 (DEC-056): shadow-mode row (P3 Slack `#devops-alerts`, 30-day calibration window, next-business-day SLA) + full-mode row (P1 PagerDuty HIGH `form-devops`, < 30 min SLA) + AL-SIEM-06-SHADOW-END informational row (transition trigger: `auth_event_avg_per_30min > 5` OR 30 calendar days; emits `system.siem_alert_activated` STANDARD 3yr DEC-030 event; files CALIB-E-001 SOC 2 CC4.1 artefact); closes §44.6 checklist item 5 (P0/M5). **§43.15 checklist item 1 marked `[x]`** — `docs/AUDIT_LOG_SCHEMA.md §Integration` confirmed updated (CHANGELOG v5.10.1): all six webhook DEC-030 events registered (`integration.webhook_delivery_failed`, `integration.webhook_suspended`, `integration.webhook_reactivated` NEW; plus `endpoint_url_hash` payload extension on `webhook_created`, `webhook_deleted`, `webhook_fired` per DEC-054; WH-CHAIN-01 ordering invariant documented).
- `VERSION` — 5.11.1 → 5.11.2.

---

## [5.11.1] — 2026-06-15

### Added

- `content/post-31-mobility-vs-flexibility.md` — Editorial series post 31. «Мобільність або гнучкість — де та межа, яка важлива для силового атлета». Covers the structural distinction between passive ROM (flexibility) and active neuromuscular control (mobility). Three constraint types: structural (joint morphology — unchangeable via stretching), tissue (viscoelastic — real but overestimated), neuromuscular (most common actual limiter in strength athletes). Static stretching pre-lift evidence: Behm & Chaouachi (2011) — 23-study review, ≥60 s static stretching reduces explosive and maximal force 5–8% for 30–60 min via tendon stiffness reduction and neural inhibition. Dynamic warmup rationale. Practical assessment criteria for squat depth, deadlift hinge, overhead position, bench press. Efficient weekly integration framework: 5–10 min specific dynamic pre-lift + pause in bottom position + 15 min dedicated mobility day. clinical-safety: PASS. Blog card added.

### Changed

- `README.md` — Editorial series heading updated (11–31 drafted, proposed 32–40); post-31 marked draft · v5.11.1; new proposed topics 36–40 added to roadmap.
- `STATUS.md` — Total posts 832 → 833; editorial series 11–30 → 11–31 entry updated with post-31 summary.
- `VERSION` — 5.11.0 → 5.11.1.

---

## [5.11.0] — 2026-06-15

### Added

- `content/post-835-zma-zinc-magnesium-evidence.md` — Block 801–850 evidence-based deep dives, post 35/50. ZMA (zinc + magnesium aspartate + B6): testosterone booster marketing vs. independent evidence. Covers the Brilla & Conte (2000) founding study and its critical flaw (author is ZMA patent holder / BALCO founder). Independent replications: King et al. (2003) — no testosterone, IGF-1, body composition, or strength effect; Wilborn et al. (2004) — same null result. Zinc-testosterone link explained in deficiency context (Hamalainen 1984/1985): real mechanism, not applicable to replete athletes. Magnesium and sleep: Abbasi et al. (2012) evidence for deficient individuals vs. no effect in replete young athletes. Patented "aspartate" form: no absorption advantage over citrate/glycinate. Practical framework: test zinc/magnesium blood levels before supplementing; isolated minerals are cheaper and as effective if deficiency is confirmed. Sponsor-bias archetype: same pattern as HMB (post-832) and glutamine (post-834). clinical-safety: PASS.
- `content/post-836-collagen-peptides-connective-tissue.md` — Block 801–850 evidence-based deep dives, post 36/50. Collagen peptides: the rare supplement with a narrow but real evidence base for connective tissue. Covers glycine/proline/hydroxyproline substrate rationale (collagen matrix vs. myofibrillar protein turnover). Key studies: Shaw et al. (2017, AJCN) — 15 g gelatin + 48 mg vitamin C pre-exercise increased engineered-ligament collagen synthesis 50% vs. placebo (surrogate endpoint, n=8, crossover); Dressler et al. (2018, Nutrients) — 5 g collagen peptides + eccentric protocol in chronic Achilles tendinopathy: +26 vs. +15 VISA-A score at 6 months (functional RCT); Clark et al. (2008) — joint pain reduction in student athletes, 10 g/day for 24 weeks (subjective; industry-funded). Vitamin C cofactor mechanism (prolyl hydroxylase) explained — why co-ingestion with ~50 mg vitamin C is mechanistically required. Timing: 30–60 min pre-exercise for amino acid peak to coincide with mechanical stimulus. Who benefits: tendon/ligament rehab, masters athletes (40+), symptomatic joint pain under load. Who does not benefit: healthy young athletes seeking muscle mass or strength. Clinical-safety: CONDITIONAL PASS — guard rail embedded (chronic joint pain → physician consult before supplementation).

### Changed

- `STATUS.md` — Block 801–850 updated: 36/50 written, posts 835–836 entries added; total posts 830 → 832; next batch 837–840 updated.
- `VERSION` — 5.10.1 → 5.11.0.

---

## [5.10.1] — 2026-06-14

### Changed

- `docs/AUDIT_LOG_SCHEMA.md §Integration` — expanded the three-line webhook stub into a full DEC-030 specification. Registers six HMAC-chained events covering the enterprise webhook delivery lifecycle: `integration.webhook_created`, `integration.webhook_deleted`, `integration.webhook_fired` (extended payloads — `endpoint_url_hash` SHA-256, all five retry-step fields), `integration.webhook_delivery_failed` (HIGH, 7yr — NEW; 5th consecutive failure; `degraded` transition; AL-WH-02 trigger; WH-SLO-04 notification clock), `integration.webhook_suspended` (HIGH, 7yr — NEW; 48 h `degraded`; pg_cron job 34; AL-WH-04 trigger), `integration.webhook_reactivated` (HIGH, 7yr — NEW; `tenant_owner` self-service; resets `consecutive_failures`). Documents WH-CHAIN-01 ordering invariant (`webhook_fired` after `webhook_suspended` without `webhook_reactivated` → HTTP 422 + R-05). Includes Zod v2 schemas for all six events (canonical source for `emit-audit-event` Worker validation); three SOC 2 evidence artefacts (WH-E-001 CC9.2/CC6.8 quarterly lifecycle export; WH-E-002 CC7.2/CC7.3 PagerDuty + Resend receipts; WH-E-003 CC6.8/CC6.1 annual Worker source code review). Privacy floor documented: `endpoint_url` never in chain; no Art. 9 health data; `tenant_manager` (HR) blocked; `form_api` REVOKED; `integration.webhook_*` SIEM events `compliance_reviewer` only. Closes OBSERVABILITY §43.15 checklist item 1 (P0, M10). Cross-ref: `docs/OBSERVABILITY.md §43`, `docs/ENTERPRISE_ADMIN_API.md §9`, DEC-030.
- `VERSION` — 5.10.0 → 5.10.1.

---

## [5.10.0] — 2026-06-14

### Added

- `content/post-834-glutamine-evidence-review.md` — Block 801–850 evidence-based deep dives, post 34/50. Glutamine: the most-marketed supplement with null evidence in healthy athletes. Covers endogenous synthesis capacity (glutamine synthetase in skeletal muscle), clinical context (sepsis, burns, surgery, IBD) vs. sports marketing. Key studies: Antonio & Street 1999 (0.9 g/kg/day, no body composition effect), Candow 2001 (6-week RCT, no strength or mass effect), Legault 2015 (no DOMS reduction), Gleeson 2008 systematic review (immune function: weak, mixed data). Gut permeability: Van Wijck 2011 (transient effect in endurance athletes; clinical significance unclear). Nitrogen shuttle and endo/exogenous balance explained. Archetype pattern: clinical data decontextualised into supplement marketing (parallels HMB post-832). clinical-safety: PASS.

### Changed

- `blog.html` — blog cards added for posts 832, 833, 834 (Evidence-based block).
- `STATUS.md` — block 801–850 updated: 33 → 34 posts written; total posts 829 → 830; next priorities updated to posts 835–840.
- `VERSION` — 5.9.0 → 5.10.0.

---

## [5.9.0] — 2026-06-14

### Added

- `content/post-832-hmb-evidence-review.md` — Block 801–850 evidence-based deep dives, post 32/50. HMB: comprehensive evidence review contrasting manufacturer-funded vs. independent RCTs. Covers UPS/MAFbx/MuRF-1 molecular mechanism, caspace-3 apoptosis pathway, PI3K/Akt mTOR activation, and membrane-stabilisation hypothesis. Key studies: Nissen et al. 1996 (patent-holder conflict-of-interest analysis), Gallagher 2000, Thomson 2009 (trained athletes, null result), Slater & Jenkins 2000 (methodological critique), Wilson 2014 meta-analysis (sponsor-bias analysis), Jakubowski 2020 independent RCT (null result in trained). HMB-Ca vs HMB-FA comparison. Contexts where anti-catabolic mechanism may be relevant (immobilisation, aggressive deficit, clinical populations). Concluding framing: sponsor-bias pattern applicable beyond HMB. clinical-safety: PASS.
- `content/post-833-caffeine-tolerance-cycling.md` — Block 801–850 evidence-based deep dives, post 33/50. Caffeine tolerance and strategic cycling for trained athletes. Mechanism: adenosine A₁/A₂A receptor upregulation under chronic exposure. Key studies: Bell & McLellan 2002 (habitual consumers vs. infrequent; shorter, blunted ergogenic effect), Gonçalves 2017 systematic review (trained status as moderator), Pickering & Kiely 2019 (challenges mandatory wash-out dogma), Grgic 2020 meta-analysis BJSM 225 studies (dose-response curve 2–6 mg/kg optimal; > 6 mg/kg diminishing returns). Four strategies: full wash-out (10–14 days), selective use (target training days only), chronotiming (90-min post-waking delay; 45–60 min pre-training window), sleep-debt caveat. CYP1A2 genotype variability (slow vs. fast metabolisers). L-theanine combination (Dodd 2015 meta-analysis). clinical-safety: PASS.

### Changed

- `STATUS.md` — block 801–850 updated: 31 → 33 posts written; total posts 827 → 829; next priorities updated to posts 834–840.
- `VERSION` — 5.8.4 → 5.9.0.

---

## [5.8.4] — 2026-06-14

### Changed

- `docs/SSO_SCIM_IMPLEMENTATION.md` — §30 added (v2.1 → v2.2): OQ-MOBILE-03 Resolution — Mobile SSO Browser Mode & Deep-Link Security (DEC-059). System browser (`expo-web-browser openAuthSessionAsync`) mandated for all enterprise mobile SSO redirects; embedded WebView prohibited. Covers: security analysis (RFC 9700 §4.1 compliance, process isolation, TLS-bypass prevention, credential-manager integration), `openSsoBrowser()` TypeScript implementation with `ASWebAuthenticationSession` / Chrome Custom Tabs, ESLint prohibition guard + CI grep test, nine-row IdP compatibility matrix, Universal Links / App Links callback hardening for `https://form.coach/auth/mobile/callback/{tenant_id}`, Azure AD B2C OIDC PKCE workaround for SAML-incompatible B2C tenants, two new DEC-030 HMAC-chained events (`sso.mobile_browser_mode_set` STANDARD 7yr; `sso.mobile_webview_blocked` HIGH 7yr), AL-SSO-WEB-01 P1 alert (PagerDuty `form-security`, zero-fire expected), three SOC 2 evidence artefacts (SSO-WEB-E-001 CC6.1; SSO-WEB-E-002 CC6.2/CC8.1), fourteen-item implementation checklist (6× P0/M8, 6× P1/M10, 2× P2/Annual).
- `docs/OBSERVABILITY.md` — §28.13 OQ-MOBILE-03 updated: 🟡 P1 Open → 🟢 Resolved (DEC-059, 2026-06-14); resolution summary cross-references `docs/SSO_SCIM_IMPLEMENTATION.md §30`.
- `docs/DECISION_LOG.md` — DEC-059 added (2026-06-14): Mobile SSO browser mode — system browser mandated, WebView prohibited. Three grounds: RFC 9700 §4.1 compliance, process isolation (host app cannot observe ASWebAuthenticationSession), TLS bypass prevention. Low reverse cost.
- `VERSION` — 5.8.3 → 5.8.4.

---

## [5.8.3] — 2026-06-14

### Added

- `content/post-30-long-term-results.md` — Що таке «довгострокові результати» — і чому більшість не думає в цьому горизонті. Editorial-brutalist, 13 хв. Теми: міофібрилярна гіпертрофія vs саркоплазматична, адаптація сухожиль (Arampatzis et al. 2007, 2010), нейром'язова консолідація (Enoka & Duchateau 2017), горизонт планування 12 тижнів vs 3 роки, тренувальний вік як предиктор. clinical-safety: PASS (відсутні теми їжі, тіла, болю, ментального здоров'я).

### Changed

- `blog.html` — blog cards added for posts 26–30 (content existed for 26–29, cards were missing; post-30 new).
- `README.md` — Editorial series header updated: 11–25 → 11–30; posts 26–29 marked draft; posts 31–35 proposed (mobility vs flexibility, reading research, self-programming vs coach, dropout mechanics, breathing mechanics).
- `STATUS.md` — Editorial series 11–25 → 11–30; post-30 description added; proposed 31–35 listed; v5.8.1 → v5.8.3.
- `VERSION` — 5.8.2 → 5.8.3.

---

## [5.8.2] — 2026-06-14

### Changed

- `docs/OBSERVABILITY.md` — §45 added (v3.9 → v4.1): OQ-PERF-01 + OQ-PERF-02 Resolution — k6 Platform Selection & Staging Data Governance (DEC-058). Closes two P1 open questions from §40.9: OQ-PERF-01 (k6 OSS vs. k6 Cloud) → hybrid architecture adopted (k6 OSS for CI merge gate; k6 Cloud EU-West Amsterdam for quarterly PERF-SLO-06 reference run); OQ-PERF-02 (staging data freshness) → four-step refresh procedure with ANON-01–ANON-05 invariants and clinical-safety sign-off (PERF-STAGING-E-001). OQ-PERF-03 (fleet-wide VU scaling) closed as 🟢 Deferred with threshold (10k sessions/day or 5,000 fleet seats). Two new DEC-030 STANDARD 3yr events: `system.quarterly_perf_reference_initiated` and `system.quarterly_perf_reference_completed`; PERF-CLOUD-CHAIN-01 ordering invariant. §40.9 OQ table updated; §40.10 items 8 and 9 marked Done.
- `VERSION` — 5.8.1 → 5.8.2.

---

## [5.8.1] — 2026-06-14

### Added

- `content/post-831-antioxidants-training-adaptation-paradox.md` — Антиоксиданти і тренування: парадокс адаптації. ROS як сигнальні молекули (AMPK, PGC-1α, NRF2-каскад); Gomez-Cabrera et al. (2008) Am J Clin Nutr — вітамін C 1000 мг/день пригнічує мітохондріальний біогенез і VO₂max адаптацію; Ristow et al. (2009) PNAS — вітаміни C + E повністю блокують покращення чутливості до інсуліну; Paulsen et al. (2014) J Physiol — пригнічення мітохондріальних маркерів при 11 тижнях силових тренувань; Merry & Ristow (2016) J Physiol — систематичний огляд. Практична рамка: харчові антиоксиданти ≠ мегадози добавок; timing-гіпотеза; виключення для майстер-атлетів і хвороби. clinical-safety: PASS.

### Changed

- `blog.html` — blog card added for post-831 (Evidence-based · 13 хв · 2026-06-14).
- `README.md` — Block 801–850 expanded: 801–830 written listed; proposed 831–840 topics added (L-carnitine, HMB, collagen/tendons, electrolytes, taurine, BCAA/EAA, non-anemic iron deficiency, zinc/testosterone, berberine).
- `STATUS.md` — Block 801–850: 30/50 → 31/50 written; total posts 826 → 827; v5.8.0 → v5.8.1.
- `VERSION` — 5.8.0 → 5.8.1.

---

## [5.8.0] — 2026-06-14

### Added

**Block 821–830 — Evidence-based deep dives Batch 3 (supplement science):**
- `content/post-821-protein-timing-anabolic-window-myth.md` — Protein timing: anabolic window myth vs. reality. Schoenfeld et al. (2013) meta-analysis; daily total vs. peri-workout timing; decision tree for fasted vs. fed training contexts. clinical-safety: PASS.
- `content/post-822-creatine-loading-vs-maintenance.md` — Creatine loading revisited: loading (20g×5d) vs. maintenance-only (3–5g/day) reach identical saturation at 28 days (Hultman et al., 1996). Non-responder biology. Creatine monohydrate as gold standard. clinical-safety: PASS.
- `content/post-823-beta-alanine-carnosine-what-data-shows.md` — Beta-alanine: H⁺ buffering via carnosine (not lactic acid). Hobson et al. (2012): Cohen's d ~0.37 for 1–4 min efforts. Minimal benefit for 1–5RM strength. Paresthesia management. clinical-safety: PASS.
- `content/post-824-citrulline-nitrates-no-pathway-evidence.md` — Citrulline and nitrates: two NO pathways explained. Pérez-Guisado & Jakeman (2010) citrulline malate data. Nitrate/beetroot O2 economy evidence. clinical-safety: CONDITIONAL PASS (PDE5 interaction guard rail).
- `content/post-825-magnesium-athlete-deficiency-supplementation.md` — Magnesium: ATP-Mg complex, sweat losses, form comparison table (glycinate vs. citrate vs. oxide). Sleep evidence only in deficient individuals. clinical-safety: CONDITIONAL PASS (kidney disease guard rail).
- `content/post-826-omega3-epa-dha-training-evidence.md` — Omega-3: EPA+DHA (not ALA). Smith et al. (2011) mTOR signaling; Waldman et al. (2020) meta-analysis. Label-reading guidance: EPA+DHA ≠ total omega-3. Oxidation risk. clinical-safety: PASS.
- `content/post-827-vitamin-d-athletes-deficiency-performance.md` — Vitamin D: fat-soluble hormone precursor. Deficiency prevalence in indoor/high-latitude athletes. Effect only in deficient individuals. 25(OH)D testing guidance. 2000–4000 IU D3. clinical-safety: PASS.
- `content/post-828-ashwagandha-ksm66-strength-cortisol-evidence.md` — Ashwagandha: KSM-66/Sensoril standardised extracts. Wankhede et al. (2015) strength data. Cortisol claims framed as performance context only — not stress disorder treatment. clinical-safety: CONDITIONAL PASS.
- `content/post-829-melatonin-glycine-sleep-quality-athletes.md` — Sleep supplements: melatonin (chronobiotic, not hypnotic; 0.5–3mg), glycine (Bannai & Nagashima 2012; 3g pre-bed, core temperature mechanism). Explicitly not for clinical insomnia. clinical-safety: CONDITIONAL PASS.
- `content/post-830-cbd-recovery-evidence-review-athletes.md` — CBD and recovery: honest null result lead (Isenmann et al., 2021). WADA status. Anti-doping contamination risk. Explicitly not framed as pain or mental health treatment. clinical-safety: CONDITIONAL PASS.

**Editorial series posts 22–25:**
- `content/post-22-seasonal-programming.md` — Сезонне програмування для recreational athlete: зима, літо, відпустки. Macro-cycle from real calendar; winter accumulation, summer intensification, holiday planning. clinical-safety: PASS.
- `content/post-23-return-after-break.md` — Повернення після перерви: детренінг timeline (Persson, Lovell, Bruusgaard et al.), muscle memory physiology, on-ramp protocol weeks 1–4. clinical-safety: PASS.
- `content/post-24-female-physiology-strength.md` — Жіноча фізіологія і силові тренування. Follicular/luteal phase training differences (Sims & Heather 2018), hypertrophy evidence, ACL biomechanics, what is and isn't different. Guard rails: menstrual disorders, amenorrhea, clinical symptoms → physician. clinical-safety: CONDITIONAL PASS.
- `content/post-25-minimal-equipment-training.md` — Тренування з мінімальним обладнанням: збереження адаптації за патернами (squat/press/pull/hinge), підтримуюча програма A+B, що НЕ зберігає результат. clinical-safety: PASS.

### Changed
- `README.md` — editorial series updated: 11–21 complete → 11–25 drafted; posts 22–25 table updated to draft status; posts 26–30 added as proposed; OQ-EDITORIAL-01 added (filename collision with May 2026 short posts in range 22–25).
- `STATUS.md` — Block 801–850: 20/50 → 30/50 written; editorial series 11–25 noted; total posts 816 → 826; next batch 831–840 defined.
- `VERSION` — 5.7.1 → 5.8.0.

---

## [5.7.1] — 2026-06-14

### Added
- `docs/OBSERVABILITY.md §44` — SIEM Alert Calibration Governance. Closes two P1 open questions from `docs/SOC2_READINESS.md §76.11` that were blocking M5 PagerDuty deployment. **OQ-ANOM-02 → DEC-056:** AL-SIEM-06 dead-man's switch (zero siem_events / 30 min / business hours) confirmed at 30-min threshold; graduated activation policy added — 30-day shadow mode (P3 Slack) then full P1 PagerDuty activation on day 31 or when `auth_event_avg_per_30min > 5`; `system.siem_alert_activated` DEC-030 event (STANDARD, 3yr) records transition; CALIB-E-001 SOC 2 evidence artefact (CC4.1/CC7.2). **OQ-ANOM-01 → DEC-057:** CR-01 Analytics Engine `GROUP BY (tenant_id, actor_ip_subnet)` confirmed per-tenant isolated — no cross-tenant aggregation blind spot; mandatory staging verification test (`src/tests/siem/cr01-per-tenant-isolation.test.ts`): dual-tenant synthetic test with 1,000-failure high-volume tenant and 12-failure low-volume tenant on distinct subnets; `system.siem_calibration_verified` DEC-030 event (STANDARD, 3yr); CALIB-E-002 SOC 2 evidence artefact (CC7.2/CC4.1). Eleven-item implementation checklist: 7× P0/M5, 2× P1/30-days-post-M5, 2× P2/Annual+M9.
- `docs/DECISION_LOG.md DEC-056` — OQ-ANOM-02 closure: AL-SIEM-06 30-min threshold confirmed; graduated activation policy.
- `docs/DECISION_LOG.md DEC-057` — OQ-ANOM-01 closure: CR-01 per-tenant isolation confirmed; mandatory staging verification test required before M5.

### Changed
- `VERSION` — 5.7.0 → 5.7.1.

---

## [5.7.0] — 2026-06-14

### Added
- `content/post-21-plateau-diagnosis.md` — Editorial series, post 21. "Плато — це не стеля. Це діагноз." Three sources of training plateau (absent progressive overload, accumulated fatigue, technical limiter), diagnostic algorithm (8-week audit → recovery markers → video tech review), practical interventions per source, time horizon for intervention assessment. 13-min read. clinical-safety: PASS (no food/body image/mental health content). sports-scientist review required before publish.
- `blog.html` — Cards added for posts 18 (rest intervals), 19 (warm-up protocols), 20 (DOMS), 21 (plateau diagnosis).

### Changed
- `README.md` — Editorial series table updated: 11–21 complete; proposed next posts 22–25 added (seasonal programming, return from break, female physiology · clinical-safety required, minimal equipment).
- `STATUS.md` — Editorial series row updated; total posts 815 → 816.
- `VERSION` — 5.6.0 → 5.7.0.

---

## [5.6.0] — 2026-06-14

### Added
- `content/post-811-sleep-athletic-performance.md` — Block 801–850, post 11/50. Sleep and athletic performance evidence review: sleep architecture (NREM/REM cycles), GH/testosterone/cortisol/melatonin hormonal physiology, quantitative performance deficits (strength, cognition, injury risk), sleep extension data (Mah et al. 2011 +5% sprint, +9% shooting accuracy), practical protocols (room temperature, caffeine timing, stable wake time, HRV as sleep proxy). clinical-safety: PASS. sports-scientist review required.
- `content/post-812-deload-science-evidence.md` — Block 801–850, post 12/50. Deload science: Fitness-Fatigue model (Zatsiorsky), peripheral vs. central fatigue types, evidence base (Murach & Bagley 2016, Shattock & Tee 2020), volume vs. intensity manipulation (volume −40–60%, intensity preserved), programmed vs. autoregulatory deload markers, deload variants (classic/technical/active rest). DEC-013 cross-reference. clinical-safety: PASS. sports-scientist review required.
- `content/post-813-training-age-implications.md` — Block 801–850, post 13/50. Training age classification: novice/intermediate/advanced definitions, physiological mechanisms per stage (neural → structural hypertrophy), muscle gain rate potential table (year 1–4+), strength standards as better proxy than chronological years, age-related programming adjustments (18–30 vs. 30–45 vs. 45+). clinical-safety: PASS. sports-scientist review required.
- `content/post-814-volume-landmarks-mev-mav-mrv.md` — Block 801–850, post 14/50. Volume landmarks: MV/MEV/MAV/MRV framework, dose-response evidence (Ralston 2017, Schoenfeld 2016, Krieger 2010), individual determinants (sleep, calories, external stress, training age), mesocycle progression model (MEV → MAV over 4 weeks), per-muscle-group reference table, effective sets counting methodology. clinical-safety: PASS. sports-scientist review required.
- `content/post-815-inter-set-rest-evidence.md` — Block 801–850, post 15/50. Inter-set rest evidence: PCr resynthesis kinetics (50%@30s, 75%@90s, 95%@3min), metabolite clearance, Schoenfeld 2016 (3 min vs. 1 min: +1.5% vs. +0.9% lean mass over 8 wks), strength vs. hypertrophy vs. conditioning protocols, autoregulatory rest approach, supersets (effective vs. ineffective pairing), TUT interaction. clinical-safety: PASS. sports-scientist review required.
- `content/post-816-rate-of-force-development.md` — Block 801–850, post 16/50. Rate of Force Development (RFD): definition and measurement (N/s at 50/100/200ms), early vs. late RFD neural mechanisms (rate coding, MU synchronization, Ca²+ kinetics), training methods (ballistic, plyometric, high-load intent, PAP complex training), Behm & Sale 1993 intent principle, practical protocols for RFD development. clinical-safety: PASS. sports-scientist review required.
- `content/post-817-concurrent-training-interference-effect.md` — Block 801–850, post 17/50. Concurrent training interference: Hickson 1980 original context, Wilson 2012 meta-analysis, AMPK/mTOR molecular conflict and limitations, modulating variables (exercise mode — bike vs. run, session order, time separation, volume, caloric deficit), practical programming by goal (hypertrophy / general fitness / fat loss while preserving muscle). clinical-safety: PASS. sports-scientist review required.
- `content/post-818-heat-cold-exposure-training.md` — Block 801–850, post 18/50. Heat and cold exposure: sauna GH response (Leppäluoto 1987 2–5×), LAUKKANEN 2018 cardiovascular data, heat acclimation + plasma volume, BDNF; CWI mechanism (vasoconstriction, sympathetic activation), Roberts 2015 CWI hypertrophy suppression, CWI utility for repeated-bout performance; contrast therapy; practical decision table. Clinical safety note on contraindications embedded. clinical-safety: PASS. sports-scientist review required.
- `content/post-819-altitude-training-evidence.md` — Block 801–850, post 19/50. Altitude training: HIF-1α pathway, EPO → erythrocytosis mechanism, LHTL (Live High Train Low) evidence (Stray-Gundersen 2001 +5% VO2max), IHT limitations, altitude masks market confusion, applicability matrix (elite endurance vs. strength athletes vs. amateurs), AMS safety. clinical-safety: PASS. sports-scientist review required.
- `content/post-820-evidence-based-stretching.md` — Block 801–850, post 20/50. Evidence-based stretching: Kay & Blazevich 2012 meta-analysis (SS > 60s → −5–8% strength), Behm 2016 dose threshold, injury prevention myth (Thacker 2004, Herbert 2011), dynamic warm-up superiority, PNF CR technique and autogenic inhibition, chronic ROM improvement protocols, stretch-mediated hypertrophy (2021–2023 data, loaded stretching, full ROM in exercises). clinical-safety: PASS. sports-scientist review required.

### Changed
- `STATUS.md` — Block 801–850 row updated: 10/50 → 20/50; next priorities updated to batch 821–830 (supplement evidence); total posts 805 → 815.

---

## [5.5.1] — 2026-06-14

### Changed
- `docs/SOC2_READINESS.md` — §82 OQ-CTOOL-01 Resolution: Cloudflare WAF Evidence Collection Method (DEC-055). Closes the last P1 open question from §78.11. Terraform state export adopted over dashboard screenshot for quarterly WAF evidence (CTOOL-WAF-E-001): tamper-evident, SHA-256 checksum–stable, §79.7 MASTER-INDEX compatible, operationally simpler. Four-ground rationale, §82.3 extraction command, five-step quarterly upload procedure, CTOOL-WAF-E-001 artefact spec (CC6.8 + CC6.6, quarterly, 3yr), gap tracker, seven-item P0/P1 checklist. §78.11 OQ-CTOOL-01 row updated 🟡 → 🟢. Document header v3.8.1 → v3.8.2.
- `docs/DECISION_LOG.md` — DEC-055: OQ-CTOOL-01 formal closure.
- `VERSION` — 5.5.0 → 5.5.1.

---

## [5.5.0] — 2026-06-14

### Added
- `content/post-17-ai-coach-vs-pt.md` — Editorial series post 17/17. AI-тренер проти персонального тренера: чесний trade-off. Матриця вибору: коли PT незамінний (новачок, реабілітація, мотивація як bottleneck, складні рухові патерни); коли AI достатньо (intermediate+, непередбачуваний графік, трендова аналітика, обмежений бюджет). Гібридні варіанти. Позиція FORM — задекларована явно без маркетингового framing. clinical-safety PASS.
- `blog.html` — картка для post-17.

### Changed
- `README.md` — editorial series table оновлено: 11–17 complete; пропозиції post-18–20 додані.
- `STATUS.md` — editorial series рядок оновлено: 11–17 complete; версія v5.5.0.
- `VERSION` — 5.4.0 → 5.5.0.

---

## [5.4.0] — 2026-06-14

### Added
- `content/post-810-fat-endocrine-organ-training.md` — Block 801–850 post 10/10. Жир і тренування: ендокринний орган, не просто паливо. Адипокіни: лептин (енергетична доступність → гормональна вісь), адипонектин (AMPK, інсулінова чутливість, підвищується від тренінгу), резистин. IMCL і аеробний метаболізм. Вісцеральний vs. підшкірний жир: різний адипокіновий профіль; силовий тренінг знижує переважно вісцеральний (Vissers et al., 2013). Ароматаза у жировій тканині і тестостерон-естроген конверсія. Fat-adapted: дані Burke et al. (2021) — компроміс при інтенсивних зусиллях. Омега-3 і запальні ліпідні медіатори відновлення (Smith et al., 2011). clinical_safety_note: фізіологія, не дієтичні рекомендації. review_pending: sports-scientist.

### Changed
- `STATUS.md` — Block 801–850: 10/10 initial topics complete (posts 801–810); next priorities оновлено до розширення блоку.
- `VERSION` — 5.3.0 → 5.4.0.

---

## [5.3.0] — 2026-06-14

### Added
- `content/post-808-steroids-natural-training-research-interpretation.md` — Block 801–850 post 8/50. Стероїди і натуральний тренінг: як читати дослідження, де популяція не схожа на тебе. Анаболічне середовище і проблема перенесення висновків. Класичний Bhasin et al. (1996): AAS без тренінгу > плацебо + тренінг. Nерозкриті учасники в класичних дослідженнях. Об'єм 10–20 підходів/тиждень для натурального vs. Elite AAS-assisted. Як читати критично: популяція, effect size, тривалість, фінансування.
- `content/post-809-genetic-ceiling-natural-potential.md` — Block 801–850 post 9/50. Генетична стеля і натуральний потенціал. ACTN3 (R577X), міостатин (LOF рідкісний), тип волокон, структура скелету — реальні фактори і їх практична обмеженість. Non-responders (~5–10%). McDonald: крива прогресу 1–7 рік (10 кг → 1 кг/рік). Більшість intermediate атлетів обмежені тренувальними параметрами, а не генетикою.
- `blog.html` — 2 картки для постів 808–809.
- `STATUS.md` — OBSERVABILITY.md оновлено до v4.0 (§43 + DEC-054); DEC-054 додано до Open decisions.

---

## [5.2.0] — 2026-06-14

### Added
- `content/post-802-protein-evidence-review.md` — Block 801–850 post 2/50. Протеїн: скільки, коли і яке джерело. MPS і лейцин як тригер mTORC1. Оптимальний діапазон 1,6–2,2 г/кг/день (Masters → 2,4). Анаболічне вікно переоцінено; 4–5 прийомів по 20–40 г — важливіше. Рослинний протеїн: +20% дози, комбінація джерел. BCAA при адекватному харчуванні — зайві.
- `content/post-803-caffeine-evidence-review.md` — Block 801–850 post 3/50. Кофеїн: антагоніст аденозину, не «генератор енергії». 3–6 мг/кг за 45–60 хв. Толерантність через 7–14 днів щоденного прийому — реальна, потрібне циклування. Кофеїн після 16:00 = компроміс сну (t½ 5–6 год). Кофеїн + L-теанін для чутливих. Простий monohydrate > pre-workout матриці.
- `content/post-804-periodization-vs-autoregulation.md` — Block 801–850 post 4/50. Periodization vs. autoregulation — не протилежності, а різні рівні управління (макро/мезо = periodization; мікро/сесія = autoregulation). Лінійна, хвилеподібна, блокова. Дослідження: RPE-based програми > фіксований % від 1RM за приростом сили і м'язів. Synthesis: мезоцикл планується, сесія авторегулюється.
- `content/post-805-rpe-rir-practical-guide.md` — Block 801–850 post 5/50. RPE і RIR: шкала, зв'язок між ними, як калібрувати. Досвідчені атлети (2+ роки) — висока точність RPE (Zourdos et al.). Типові помилки: ego inflation, ego protection. Практичні схеми: top set + back-offs, AMRAP to RPE 9. Записувати і вагу, і RPE.
- `content/post-806-hypertrophy-mechanisms.md` — Block 801–850 post 6/50. Механізми гіпертрофії 2020s. Тріада Schoenfeld (2010) переглянута: метаболічний стрес — маркер рекрутменту, не самостійний механізм; м'язове пошкодження — не мета. Новий акцент: lengthened loading (розтягнута позиція під навантаженням) → більший гіпертрофічний ефект. Доказові детермінанти: об'єм 10–20 підходів/тиждень, наближення до відмови (RIR 1–3), повний RoM.
- `content/post-807-myofibrillar-vs-sarcoplasmic-hypertrophy.md` — Block 801–850 post 7/50. Міофібрилярна vs. саркоплазматична гіпертрофія: обидва типи відбуваються одночасно при будь-якій схемі. «Великі повторення = порожній об'єм» — міф. Реальна різниця у нейром'язовій ефективності пауерліфтерів — окрема адаптація від тренінгу на 1–5 повт. Практичний висновок: варіативність діапазонів.
- `blog.html` — 6 карток для постів 802–807 (Evidence-based deep dives).

---

## [5.1.1] — 2026-06-14

### Added
- `docs/OBSERVABILITY.md §43` — Enterprise Webhook Delivery Observability: observability companion to `docs/ENTERPRISE_ADMIN_API.md §9` (Webhook Management). RED metrics (8 signals in `WH_TELEMETRY` Analytics Engine: delivery attempts, retry rate, queue events, failure rate by error_class, degraded/suspended counts, P95 dispatch latency, P95 queue age). Four SLOs: WH-SLO-01 (P95 < 30 s — feeds §23 SLA credit engine), WH-SLO-02 (zero silent drops), WH-SLO-03 (100% HMAC signature coverage), WH-SLO-04 (degraded notification ≤ 2 h). Five alert rules AL-WH-01 through AL-WH-05 including dispatcher dead-man's switch. Two Postgres tables (`tenant_webhooks` DDL 0074, `webhook_delivery_log` DDL 0074b) with full RLS and `form_api` REVOKED. pg_cron job 34 `webhook_degraded_escalation_check`. Three new DEC-030 events (`integration.webhook_delivery_failed` HIGH 7yr, `integration.webhook_suspended` HIGH 7yr, `integration.webhook_reactivated` HIGH 7yr) + payload extensions to existing `created/deleted/fired` events using `endpoint_url_hash` per DEC-054. WH-CHAIN-01 ordering invariant. WH-NOTIF-01/02 communication templates. Three SOC 2 evidence artefacts (WH-E-001 CC9.2/CC6.8, WH-E-002 CC7.2/CC7.3, WH-E-003 CC6.8/CC6.1). Admin Dashboard "Webhooks & Delivery Health" panel + Metabase "Webhook Fleet Health" internal dashboard. v4.0 internal doc version.
- `docs/DECISION_LOG.md DEC-054` — OQ-WL-OBS-01: `custom_domain_hash` SHA-256 adopted for all `tenant.white_label_*` DEC-030 payloads; consistent with DEC-043 and §43 `endpoint_url_hash` pattern; migration 0072 DDL extended.

### Changed
- `docs/OBSERVABILITY.md §42.13` — OQ-WL-OBS-01 updated to 🟢 Resolved — DEC-054 (2026-06-14); `custom_domain_hash` adopted.
- `docs/OBSERVABILITY.md §42.10` — `tenant.white_label_provisioned` DEC-030 payload spec updated: `custom_domain` → `custom_domain_hash` SHA-256.
- `docs/OBSERVABILITY.md §42.14 item 12` — marked [x] Done (OQ-WL-OBS-01 closed).
- `VERSION` — 5.1.0 → 5.1.1.

---

## [5.1.0] — 2026-06-14

### Added
- `content/post-791-athletes-with-disabilities-training.md` — Block 751–800 post 48/50. Силовий тренінг з обмеженнями руху: принципи адаптації програмування. clinical-safety CONDITIONAL PASS (opening disclaimer, movement-specialist gate, explicit exclusions, no self-assessment language). Класифікація за руховими функціями. Унілатеральний підхід і крос-едукаційний ефект. Обмеження хребта: нейтральна зона, trap bar.
- `content/post-793-training-chronic-work-stress.md` — Block 751–800 post 49/50. Тренінг і стійкий підвищений стрес: адаптація програми при системно обмеженому ресурсі. clinical-safety CONDITIONAL PASS (opening gate for burnout/clinical exclusion, behavioral framing not clinical cortisol, hard stop criteria, no cortisol-testing or supplement language; delivery restricted: no push 21:00–08:00). Алостатичне навантаження. Знижуй об'єм, не інтенсивність. Deload кожні 2–3 тижні.
- `content/post-794-medically-restricted-diet-training.md` — Block 751–800 post 50/50. Тренінг при медично обмеженому харчуванні: програмування на стороні тренінгу. clinical-safety CONDITIONAL PASS (mandatory RD/physician gate, no quantitative energy thresholds, ED protection clause, flare exclusion, no recovery nutrition, in-app confirmation screen required). Тренінг адаптується до дієтолога. Об'єм, частота, deload при обмеженому субстраті.
- **Block 751–800 COMPLETE** — 50/50 posts written. All posts require sports-scientist review before publish. Posts 791, 793, 794 also require clinical-safety guard rail compliance check in-app.
- `content/post-801-creatine-mechanism-dosing-evidence.md` — Block 801–850 post 1/50. Креатин: механізм, дози і доказова база без маркетингу. PCr ресинтез і ATP регенерація. 30+ років РКД. 3–5 г/день без завантаження. Monohydrate vs. форми. Responders/non-responders. Розвінчення міту про нирки. Вегетаріанці. Водне насичення. Кофеїн конфлікт (поточний консенсус: немає).
- `blog.html` — картки для постів 791, 793, 794 (із позначкою clinical-safety CONDITIONAL PASS) і post-801.

---

## [5.0.1] — 2026-06-14

### Added
- `blog.html` — 13 картки для постів 785–800 (minus 791, 793, 794 — заблоковані clinical-safety): posts 785–790, 792, 795–800. Block 751–800 тепер повністю відображений у блозі.

---

## [5.0.0] — 2026-06-14

### Added
- `content/post-784-functional-fitness-vs-strength-training.md` — Block 751–800 post 34/50. Функціональний фітнес vs. силовий тренінг: методологічна різниця. Принцип SAID (Specific Adaptations to Imposed Demands) як єдине точне визначення «функціональності». Три ключові методологічні відмінності: прогресивне навантаження (вимірювана прогресія vs. варіативність), специфічність стимулу (концентрований vs. розпорошений об'єм), вимірюваність результату. П'ятипитальний діагностичний тест для self-coached атлета. Практичний фрейм вибору по цілях. clinical-safety PASS.
- `blog.html` — картка для post-784.
- `content/post-785-limited-shoulder-mobility-training.md` — Block 751–800 post 35/50. Тренінг при обмеженій рухливості плечей. Анатомія обмежень (малий грудний, lat, thoracic kyphosis, posterior capsule tightness). Модифікації: landmine press, neutral-grip DB, close-grip bench, SSB squat. MD-referral gate для гострого болю. sports-scientist review required.
- `content/post-786-limited-hip-mobility-training.md` — Block 751–800 post 36/50. Тренінг при обмеженій рухливості кульшових суглобів. Структурні vs. м'якотканинні обмеження. Модифікації: sumo deadlift, trap bar, RDL, box squat, heel elevation, wide stance. MD-referral gate. sports-scientist review required.
- `content/post-787-chronic-recovery-deficit.md` — Block 751–800 post 37/50. Хронічний дефіцит відновлення: ознаки і вихід. Allostatic load рамка. Деload 40–50% об'єму при збереженні інтенсивності. Авторегуляція як системний інструмент. sports-scientist review required.
- `content/post-788-first-year-after-university.md` — Block 751–800 post 38/50. Перший рік після університету: адаптація тренінгу до робочого розкладу. Когнітивне виснаження до вечора як фізіологічна реальність. Ранковий тренінг як передбачуване вікно. sports-scientist review required.
- `content/post-789-irregular-work-schedule-training.md` — Block 751–800 post 39/50. Тренінг при ненормованому робочому тижні: rolling weekly window, full-body підхід як архітектурна база, мінімальна ефективна сесія 30 хв. sports-scientist review required.
- `content/post-790-self-coaching-year-two.md` — Block 751–800 post 40/50. Самокоучинг рік другий: тренувальний лог як актив, RPE/RIR точність, відмова від «магічного» rep range, мезоцикл як одиниця планування. sports-scientist review required.
- `content/post-792-masters-sport-40-70.md` — Block 751–800 post 41/50. Ветеранський спорт Masters 40–70: методологічна карта декад 40–49 / 50–59 / 60–69. Саркопенія після 50, RFD після 60. sports-scientist review required.
- `content/post-795-crossfitter-to-strength-training.md` — Block 751–800 post 42/50. Кросфітер переходить на силовий тренінг: GPP активи vs. методологічні конфлікти, відмова від metcon після важкої роботи, ego management. sports-scientist review required.
- `content/post-796-martial-arts-strength-training.md` — Block 751–800 post 43/50. Силовий тренінг і бойові мистецтва: concurrent scheduling, RFD, grip strength, posterior chain, 2 сесії на тиждень у дні без спарингу. sports-scientist review required.
- `content/post-797-training-frequent-travel-jet-lag.md` — Block 751–800 post 44/50. Тренінг при частих перельотах і зміні часових поясів. Стратегія по поясах (1–2 год / 3–5 год / 6+). Maintenance mode vs. development mode. sports-scientist review required.
- `content/post-798-training-on-budget-no-gym.md` — Block 751–800 post 45/50. Тренінг при дефіциті бюджету: ієрархія інвестицій (турнік → штанга + диски → стійки), bodyweight структурний стелю. sports-scientist review required.
- `content/post-799-self-coached-vs-coached-synthesis.md` — Block 751–800 post 46/50. Synthesis: чотири реальних вклади тренера, переваги self-coaching, гібридна модель. sports-scientist review required.
- `content/post-800-block-synthesis-751-800.md` — Block 751–800 post 47/50. Підсумок блоку: 7 наскрізних принципів. Анонс Block 801–850. sports-scientist review required.

### Changed
- `STATUS.md` — Block 751–800: 34/50 → 47/50; total posts 779 → 792; 3 posts BLOCKED pending clinical-safety (791, 793, 794); next priorities: Block 801–850 evidence-based deep dives.
- `VERSION` — 4.99.1 → 5.0.0.

---

## [4.99.1] — 2026-06-14

### Added
- `docs/INCIDENT_RESPONSE.md §18` — OQ-IR-01 Resolution: Concurrent Incident Sub-Chain Ordering (DEC-053). Option (a) adopted: timestamp-interleaved master chain retained; §18.3.1 skip-and-verify SQL extracts per-incident event sequences for SOC 2 auditors; CONC-CHAIN-01 cross-contamination invariant established (§18.3.3 detection query); §18.4 concurrent-incident IC coordination protocol (distinct ID confirmation, per-incident Slack channels, separate IC for ≥ P1 concurrent incidents, post-recovery §18.3.1/§18.3.3 cross-check); §18.5 auditor fieldwork protocol spec (`compliance/fieldwork/concurrent-incident-chain-verification.md`); CONC-E-001 SOC 2 evidence artefact (CC7.4/CC4.1, quarterly, 7yr). Five-item implementation checklist: 3× P1/M7, 2× P2/M8–annual. Option (b) rejected (per-incident parallel sub-chains with merge step): violates DEC-030 single-list HMAC invariant; introduces concurrent-write merge race; upgrade path to option (b) preserved as additive migration. Closes §16.9 OQ-IR-01 (P2, target M6 — all three §16.9 open questions now 🟢 resolved). Closes §17.7 OQ-IR-01 row.
- `docs/DECISION_LOG.md DEC-053` — OQ-IR-01 decision record: option (a) timestamp-interleaved; five grounds (rarity, DEC-030 invariant, auditor-standard practice, DEC-043/044/051 pattern, upgrade path preserved); reverse cost Low.

### Changed
- `docs/INCIDENT_RESPONSE.md` — header v2.2 → v2.6; §16.9 OQ-IR-01 updated to 🟢 Resolved (DEC-053); §17.7 resolved table extended with OQ-IR-01 row.
- `VERSION` — 4.99.0 → 4.99.1.

---

## [4.99.0] — 2026-06-14

### Added
- `content/post-16-overtraining-vs-underrecovery.md` — Editorial series post 16/17. Спектр перевантаження: functional overreaching → non-functional overreaching → overtraining syndrome (Meeusen et al. 2013 consensus). HRV як трекінговий маркер (7-денне ковзне середнє). Диференційний чекліст (об'єктивний + суб'єктивний + часовий блок). Протоколи відновлення на кожному рівні. Психологічні прояви OTS подано як нейроендокринна дисфункція HPA-осі — не психологічний фрейм. MD-referral gate для OTS. clinical-safety PASS (cloud-agent).
- `blog.html` — картки для posts 11–16 (posts 11–15 раніше не мали карток).
- `README.md` — розділ «Editorial series: proposed next (16–17)» з темами post-16 (draft) і post-17 AI-coach vs PT (proposed).

### Changed
- `STATUS.md` — total posts 777 → 778; editorial series 11–16 зафіксовано; post-17 proposed.
- `VERSION` — 4.98.0 → 4.99.0.

---

## [4.98.0] — 2026-06-14

### Added
- `content/post-781-strength-after-60.md` — Block 751–800 post 32/50. Тренінг після 60: саркопенія і нейром'язові зміни. Селективна атрофія волокон типу II (Lexell et al. 1988 — втрата 25–35% у >70р). Денервація і втрата моторних одиниць до 50% (Doherty 2003). Уповільнення RFD — обґрунтування вибухових елементів (medicine ball, plyometric адаптації). Анаболічна резистентність: 40 г протеїну на прийом vs. 20 г у молодих (Moore et al. 2015). Ортопедичні адаптації: зниження еластичності сухожиль (Narici & Maffulli 2010), ОА — не протипоказання (Fransen et al. 2015 Cochrane), альтернативи: goblet/box squat, RDL, trap bar. MD-referral gate для суглобового болю. Програмування: 3–4×/тиждень full-body, 10–14 сетів/сесію, RIR 1–2, deload кожні 4 тижні, подовжена розминка. Meta-analysis: 33% приріст сили за 20–26 тижнів (Peterson et al. 2011). Адаптаційний механізм активний до 80+. clinical-safety: внутрішня перевірка пройдено. sports-scientist review required before publish.
- `content/post-782-perimenopause-training.md` — Block 751–800 post 33/50. Тренінг під час перименопаузи. Естроген як анаболічний фактор: IGF-1 сенситизація, сателітні клітини, антипротеоліз — анаболічна резистентність при зниженому естрогені (Maltais et al. 2009; Tiidus 2011). Колагенова дисфункція: підвищений ризик пошкоджень сухожиль і зв'язок (Minahan et al. 2015 — ексцентричне відновлення). Кісткова щільність: критичне вікно, осьове навантаження ≥70% 1RM як нефармакологічна превентивна стратегія (Liu et al. 2022). Вазомоторні симптоми: гіпоталамічний KNDy механізм, інтерференція з HIIT-терморегуляцією. Сон і GH-дефіцит. Програмування: частота 3–4×/тиждень, осьові рухи обов'язково в кожній сесії, ексцентрична фаза 3–4 с, RIR 1–2, ізометрична розминка сухожиль. MD-referral gate для симптоматичного менеджменту. Без ГЗТ-рекомендацій, без body-image фреймінгу. clinical-safety: внутрішня перевірка пройдено. sports-scientist review required before publish.

### Changed
- `STATUS.md` — Block 751–800: 31/50 → 33/50; total posts 775 → 777; post-781/782 HARD VETO (попередні теми) замінені на безпечні теми (60+ та перименопауза); next: post-784 функціональний фітнес vs. силовий тренінг.
- `VERSION` — 4.97.1 → 4.98.0.

---

## [4.97.1] — 2026-06-14

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.7 → v2.8: +8 DEC-030 HMAC-chained events across two new sections. (1) White-label custom domain & SSL certificate events (`tenant.white_label_provisioned`, `tenant.white_label_cert_expiry_warning`, `tenant.white_label_cert_renewed`, `tenant.white_label_cert_expiry_breach` CRITICAL, `tenant.white_label_revoked`, `system.white_label_cert_check_stale`) — closes OBSERVABILITY.md §42.14 checklist item 1 (P0, M10); WL-CHAIN-01 invariant + six Zod v2 schemas + WL-E-001/002/003 evidence artefacts; +3 retention table rows. (2) Enterprise DSAR deletion certificate events (`dsar.certificate_issued` HIGH, `dsar.certificate_delivered` STANDARD) — closes DATA_MODEL.md §35.10 checklist item 5 (P0, M5); CERT-CHAIN-01 chain extension (`deletion_soft → deletion_confirmed → certificate_issued → certificate_delivered`); two Zod v2 schemas + DSAR-E-011/012 evidence artefacts; +2 retention table rows.
- `VERSION` — 4.97.0 → 4.97.1.

---

## [4.97.0] — 2026-06-14

### Added
- `content/post-783-strength-for-triathletes.md` — Block 751–800 post 31/50. Сила для триатлоністів-аматорів: програмування навколо swim-bike-run. AMPK/mTORC1 механізм конкурентної адаптації та interference effect (Hickson 1980). Чотири цілі силового тренінгу для триатлоніста: RunEconomy (Beattie et al. 2017 — 2–8% без зміни VO2max), затримка нейром'язової втоми, профілактика травм; що сила не дає (VO2max, лактатний поріг). Практичні принципи мінімізації інтерференції: часова відстань ≥6 год між аеробною і силовою, черговість у подвійний день (спочатку силова), тип аеробки що більше/менше активує AMPK. Вибір вправ: уніляральні вправи для нижніх кінцівок (болгарський сплит, RDL на одній нозі), кор-стабілізація (deadbug, Pallof press, farmer's carry), тягові рухи; що прибрати (ізоляція, важкий двосторонній присяд, імітаційні «плавальні» вправи). Дозування: будівний (75–85% 1RM, 3–4×4–6) vs. підтримуючий блок (70–80%, 2×3–5, 1–2 рази/тиждень; Rønnestad & Mujika 2014). Чотирифазна сезонна логіка: off-season (будівний 8–12 тижнів), базовий блок (трансформація у потужність), пікова підготовка (підтримуючий 1 сесія/тиждень), tapering (одна коротка за 10–14 днів до гонки). Тижневі шаблони для базового і пікового блоків. Чотири найпоширеніші помилки. Маркери для відстеження (болгарський сплит max, підтягування, RunEconomy). clinical-safety: не потрібно (тема не охоплює їжу/тіло/біль/ментальне). review_pending: sports-scientist.
- `blog.html` — картки для post-783 і post-780 (post-780 написано у v4.96.0, картка не була додана).

### Changed
- `STATUS.md` — Block 751–800: 30/50 → 31/50; total posts 774 → 775; post-781 і post-782 — clinical-safety HARD VETO (хронічний біль / хребет); post-783 written.
- `VERSION` — 4.96.0 → 4.97.0.

---

## [4.96.0] — 2026-06-14

### Added
- `content/post-780-mecfs-postviral-training.md` — Block 751–800 post 30/50. Тренінг при ME/CFS і пост-вірусному стані: межа між адаптацією і патологією. Фізіологія ME/CFS: мітохондріальна дисфункція (Myhill et al. 2009/2013), автономна дисрегуляція і POTS-перекриття, нейрозапалення (Nakatomi 2014). Клінічне визначення PEM з таблицею диференціації від нормальної тренувальної втоми (onset, duration, recovery, character, прогностика). Дворазовий CPET (Workwell Foundation / Keller et al. 2014 — PLOS ONE): VO2peak і анаеробний поріг падають на день 2 — об'єктивне підтвердження PEM. Повна GET-ревізія: PACE Trial методологічна критика (зміна критеріїв під час дослідження), NICE 2021 (NG206) офіційне відкликання GET. Пейсинг: energy envelope як клінічний концепт спеціалістів (не DIY-інструмент); ЧСС-пейсинг під наглядом спеціаліста, роз'яснення чому стандартні формули (220 – вік) × %ненадійні при ME/CFS. Диференціація пост-вірусної втоми від ME/CFS за часовим курсом (3–6 міс vs > 6 міс з PEM). Межа детренованість/PEM — порівняльна таблиця з клінічним тестом. Практична система: 3-тижневий журнал функціонування (не тренувальний журнал), стартова активність тільки при відсутності PEM-патерну (розмовний темп, зупинка при першому погіршенні), зелене світло — 2 послідовні тижні без PEM-відповіді, 6 стоп-сигналів з жорстким clinical referral language. Явні межі FORM/Victor: не надає реабілітаційних протоколів при ME/CFS, не замінює лікаря. clinical-safety: PASS (2026-06-14). review_pending: sports-scientist.

### Changed
- `STATUS.md` — Block 751–800: 29/50 → 30/50; total posts 773 → 774; next: post-781 (тренінг після 60).
- `VERSION` — 4.95.1 → 4.96.0.

---

## [4.95.1] — 2026-06-14

### Added
- `docs/INCIDENT_RESPONSE.md` — R-27: Evidence Collection Automation Failure — Three-Consecutive-Month Recovery. Closes OQ-EVD-02 from `docs/SOC2_READINESS.md §81.12`. Trigger: 3rd consecutive AL-EVD-01 (pg_cron job 33 dead-man's switch) → P1 PagerDuty `form-compliance` escalation. Five root cause hypotheses (cron disabled / Worker crash / R2 permission / partial crash / ETag conflict). Two-step recovery: automated backfill via Cloudflare trigger; manual collection per §79. Three DEC-030 HMAC-chained events: `system.evidence_automation_failure_declared` (HIGH, 7yr), `system.evidence_manual_collection_completed` (STANDARD, 7yr), `system.evidence_automation_restored` (STANDARD, 3yr); EVD-CHAIN-01 ordering invariant. Communication templates EVD-01 (compensating control memo) and EVD-02 (P0 auditor notification). Evidence artefacts EVD-E-001–005 + EVD-COMP-E-001 mapped to SOC 2 CC4.1/CC4.2/CC7.2/CC7.3/A1.1. Four post-incident controls; six-item implementation checklist.

### Changed
- `docs/SOC2_READINESS.md` — §81.12 OQ-EVD-02 marked 🟢 Resolved with R-27 reference and resolution summary.
- `VERSION` — 4.95.0 → 4.95.1.

---

## [4.95.0] — 2026-06-14

### Added
- `content/post-779-entrepreneur-athlete-business-scaling.md` — Block 751–800 post 29/50. Тренінг атлета-підприємця при масштабуванні бізнесу: алостатичне навантаження іншого порядку. Якісна відмінність scaling-фази від зайнятості (Sapolsky: неконтрольований непередбачуваний стрес гірший), decision fatigue і витрата когнітивного ресурсу (Baumeister; Mark et al. 2008 — context-switching 23 хв відновлення), кортизол при екзистенційному стресі vs deadline-стресі, сон через тривогу (а не через зайнятість — HRV низький навіть при 7+ год), парадокс ресурсів (з'явились гроші і час — алостатичне навантаження максимальне), практичне програмування (2 ранкових сесії заблоковані, RPE cap 8, авторегуляція по HRV, деактивація між роботою і тренінгом), типові помилки (амбітна нова програма при максимальному стресі, «більше тренуватись = менше стрес»), маркери виходу з scaling-фази. Clinical safety: PASS.

### Changed
- `blog.html` — Додано картки для постів 777, 778 (відсутні з v4.94.0) і нова картка для поста 779.
- `STATUS.md` — Block 751–800: 28/50 → 29/50; total posts 772 → 773; next: post-780 (хронічна втома/пост-вірусний стан).
- `VERSION` — 4.94.0 → 4.95.0.

---

## [4.94.0] — 2026-06-14

### Added
- `content/post-777-chronic-sleep-restriction-training.md` — Block 751–800 post 27/50. Тренінг при хронічному браку сну (6 год/ніч стали нормою): відмінності хронічного обмеження від гострої депривації, гормональний механізм (GH/тестостерон –10–15% при тижні на <5h, кортизол хронічно підвищений), знижена нейром'язова ефективність і MPS, таблиця доза-відповідь 8–9/7/6/≤5 год, контекстні сценарії (батьківство, нічні зміни, безсоння), адаптації програми (–20–30% обсяг, RPE –1 одиниця, авторегуляція), типові помилки (кофеїн+максимуми, «надолужити у вихідні», ігнорувати HRV). Clinical safety: PASS.
- `content/post-778-academic-stress-training.md` — Block 751–800 post 28/50. Тренінг під час хронічного академічного стресу: алостатичне навантаження (McEwen 1993) як рамка, HPA-вісь при хронічному стресі (кортизол, GH/IGF-1 супресія, тестостерон, запальний фон), відмінність від гострого сесійного стресу (пост-763), імунна супресія і «відкрите вікно», якість сну при академічному стресі, академічний рік як тренувальна сезонність (синхронізація хвиль навантаження), конкретні адаптації (семестр / пік / канікули), тренінг як стрес-менеджмент vs. фізіологічний стресор — реальна роль і обмеження. Clinical safety: PASS.

### Changed
- `STATUS.md` — Block 751–800: 26/50 → 28/50 (posts 777–778 added); total posts 770 → 772; next priorities updated to post-779 (entrepreneur-athlete at scale).
- `VERSION` — 4.93.2 → 4.94.0.

---

## [4.93.2] — 2026-06-14

### Changed
- `docs/DATA_MODEL.md` — v1.13 → v1.14. §35 DSAR Deletion Certificate Schema & SLO Breach Response (DEC-052). Resolves OQ-DSAR-03 (P1) and OQ-DSAR-04 (P2) from §32.8. OQ-DSAR-03: two-tier AL-DSAR-05 escalation — P1 PagerDuty (first SLO miss per tenant per quarter) / P2 Slack (subsequent); `enterprise_sla_counters.dsar_slo_misses_this_quarter` counter (migration `0070b`); pg_cron job 33 quarterly reset. OQ-DSAR-04: HMAC-SHA256-signed JSON cert (`ERASURE_CERT_SECRET`); admin dashboard download (immediate) + Resend email to data subject (0h) / employer HR contact (48h delay per GDPR Art. 34 ordering); R2 `form-dsar-certs/<tenant_id>/<certificate_id>.json` + `dsar_deletion_certificates` Postgres table (metadata + payload_hash only, migration `0070`). Two new DEC-030 events: `dsar.certificate_issued` (HIGH, 7yr) and `dsar.certificate_delivered` (STANDARD, 7yr); CERT-CHAIN-01 ordering invariant extends DEC-032-EXT. Two new SOC 2 artefacts: DSAR-E-011 (P8.0) and DSAR-E-012 (CC7.2). §32.8 OQ-DSAR-03 and OQ-DSAR-04 marked 🟢 Resolved.
- `VERSION` — 4.93.1 → 4.93.2.

---

## [4.93.1] — 2026-06-14

### Added
- `content/post-776-return-to-powerlifting-competition.md` — Block 751–800 post 26/50. Повернення до паверліфтингових змагань після 2+ років паузи: три фази (реакліматизація 4–8 тиж → відновлення бази 8–14 тиж → змагальна підготовка 8–12 тиж), часовий горизонт 16–28 тижнів залежно від стану, правило відкривального підходу після паузи, вагова категорія після паузи, типові помилки повернення.
- `README.md` — Block 1251–1300 «Wearables, data & the self-coached athlete» додано до роадмапу: 15 тем про HRV, sleep score, AI-тренери, powerbuilding у даних, 5-річний погляд на прогрес.

### Changed
- `blog.html` — Додано картки для постів 774, 775 (були відсутні з попереднього релізу) і нова картка для поста 776.
- `STATUS.md` — content table: Block 751–800 25/50 → 26/50; total posts 769 → 770; next: post-777 (chronic sleep deficit, non-parental).
- `VERSION` — 4.93.0 → 4.93.1.

---

## [4.93.0] — 2026-06-14

### Added
- `content/post-774-team-sport-strength-training.md` — Block 751–800 post 24/50. Силовий тренінг для командного спортсмена-аматора: поєднання залу з командними тренуваннями та матчами. Принципи розстановки навантажень у тижні, модель міжсезоння (будуй базу) vs. сезону (підтримка, 1–2 сесії). Передсезонний перехідний блок. Специфіка по видах: футбол, баскетбол, волейбол, хокей. Типові помилки командного спортсмена у залі.
- `content/post-775-recreational-athlete-season.md` — Block 751–800 post 25/50. Рекреаційний атлет і «сезон»: модель трьох фаз (міжсезоння → передсезоння → сезон) для атлетів з улюбленим сезонним видом. Детальні рекомендації для лижника, велосипедиста, тенісиста, бойових мистецтв, активного відпочинку. Розмежування між будівництвом бази і реалізацією форми.

### Changed
- `STATUS.md` — content table updated: Block 751–800 23/50 → 25/50; next priorities updated to posts 776–778; total posts 767 → 769.
- `VERSION` — 4.92.2 → 4.93.0.

---

## [4.92.2] — 2026-06-14

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v1.4 → v1.5. 3 new DEC-030 evidence-automation events in the `### System` section: `system.evidence_collection_automated` (STANDARD, 7yr, EVD-CHAIN-01 ordering invariant, AUTO-E-001); `system.evidence_cron_stale` (HIGH, 3yr, dead-man's switch signal → AL-EVD-01); `system.evidence_cron_conflict` (MEDIUM, 1yr, R2 ETag conflict abort → AL-EVD-04). Privacy floor: `actor_type='system'`, `tenant_id` sentinel, `form_api` REVOKED. Retention table +3 rows. Closes §81.11 P0 task #1.
- `docs/OBSERVABILITY.md` — §12.6 pg_cron health monitoring job registry: job 33 `evidence_cron_freshness_check` added (03:05 UTC, 2nd of month, 32-day freshness window). Annotated as dead-man's switch for the Cloudflare Cron Worker (§81); fires AL-EVD-01 Slack alert via `net.http_post` on miss; does not emit DEC-030 event itself. Closes §81.11 P0 task #2.
- `VERSION` — 4.92.1 → 4.92.2.

---

## [4.92.1] — 2026-06-14

### Changed
- `docs/SOC2_READINESS.md` — v3.8.0 → v3.8.1. §81 Monthly Evidence Collection Automation — Cloudflare Cron Worker & MASTER-INDEX Reconciler. Closes OQ-EC-03 (`🟡 Open` → `🟡 Authored`). Specifies: wrangler.toml trigger (`crons = ["0 1 1 * *"]`); 5-step Worker execution sequence (chain integrity check → SLA report → per-artifact R2 writes → MASTER-INDEX atomic reconciliation via If-Match ETag → DEC-030 event emit); `system.evidence_collection_automated` Zod schema; `system.evidence_cron_stale` and `system.evidence_cron_conflict` event stubs; EVD-CHAIN-01 ordering invariant; pg_cron job 33 `evidence_cron_freshness_check` dead-man's switch (03:05 UTC 2nd of month); 4 alert rules AL-EVD-01–04 (P0 PagerDuty chain break, P1 PagerDuty R2 write failure, P2 Slack dead-man's and ETag conflict); 2 SOC 2 evidence artefacts AUTO-E-001 (CC4.2/CC7.1, 7yr) and AUTO-E-002 (CC4.1/CC7.2, 3yr). P0 implementation tasks: register 3 events in AUDIT_LOG_SCHEMA.md §6; add pg_cron job 33 to OBSERVABILITY.md §12.6.
- `VERSION` — 4.92.0 → 4.92.1.

---

## [4.92.0] — 2026-06-14

### Added
- `content/post-773-chronic-dehydration-training.md` — Block 751–800, post 23/50. Topic: тренінг при хронічному зневодненні — клімат, робота, звичка не пити. Core argument: chronic hypohydration is a persistent, often unrecognized performance limiter distinct from acute dehydration; practical breakdown by context (office worker + evening training, heat/summer, shift workers, morning training) with actionable protocols for each; covers impact on strength, aerobic capacity, thermoregulation, cognition under load, and recovery; electrolyte section for when plain water is insufficient; common mistakes; practical one-week correction plan. Angle: athlete lifecycle — lived context rather than physiology lecture (physiology covered in post-104). No clinical-safety concerns (no food/body/pain/mental content). sports-scientist review required before publish.

### Changed
- `blog.html` — cards for posts 772 and 773 added.
- `STATUS.md` — Block 751–800 count 22 → 23 posts; total posts 766 → 767; v4.91.0 → v4.92.0.
- `VERSION` — 4.91.0 → 4.92.0.

---

## [4.91.0] — 2026-06-14

### Added
- `content/post-772-where-motivation-ends.md` — Block 751–800, post 22/50. Topic: де закінчується мотивація — коли проблема в програмуванні, а не в голові. Core argument: systematic motivation loss is often misattributed to psychological weakness when the actual cause is programming failure (monotony, chronic under-recovery, no visible progress, program-context mismatch, uncomfortable exercise selection). Practical 4-step diagnostic algorithm to distinguish programming-driven vs. psychological cause. Explicit clinical-safety boundary: persistent motivation loss + anhedonia, depressed mood, disrupted sleep, hopelessness → professional referral, not programming adjustment. Three FIX-level edits applied per clinical-safety CONDITIONAL PASS review: (1) Step 1 expanded to close false negative on partial anhedonia; (2) Step 3 "look elsewhere" replaced with explicit redirect to clinician; (3) "це не голова — це нейробіологія" replaced with behavioral framing to avoid blurring the psychology/neurobiology boundary. sports-scientist review required before publish. clinical-safety: PASS (after required edits, v4.91.0).

### Changed
- `STATUS.md` — Block 751–800 count 21 → 22 posts; total posts 765 → 766; next priorities updated (next: posts 773–775).

---

## [4.90.1] — 2026-06-14

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.6 → v2.7. +4 `enterprise.partner_*` partner channel DEC-030 events (STANDARD/HIGH, 7yr): `enterprise.partner_agreement_signed`, `enterprise.partner_revenue_share_paid`, `enterprise.partner_deal_attributed`, `enterprise.partner_offboarded`. Closes COST_MODEL.md §38.10 P0 checklist item 1. PART-CHAIN-01 ordering invariant enforced (`partner_offboarded` with privacy breach requires preceding `privacy.floor_breach_detected`). `privacy_floor_check_passed: z.literal(true)` hard invariant on revenue share payments. Three SOC 2 evidence artefacts PART-E-001–003 (CC9.2 / CC4.1 / CC7.4). Retention table +2 rows.
- `VERSION` — 4.90.0 → 4.90.1.

---

## [4.90.0] — 2026-06-14

### Added
- `content/post-771-when-self-coaching-ends.md` — Block 751–800 post 21/50. When self-coaching ends — when an external coach is needed: structural limits of self-coaching (technique not visible from own perspective, confirmation bias in own programming, real-time feedback during maximal effort, stagnation without clear cause, post-injury and pre-competition contexts), honest AI-coach trade-off table (what AI does well: consistent programming, availability, no performance anxiety, cost; what AI cannot replace: live technique observation, reading state in real time, motivational presence effect, non-standard situations), six markers that signal external coach is needed now, three questions to ask a coach before first session, framing self-coaching as tool with a ceiling rather than identity. clinical-safety PASS · sports-scientist review required before publish.
- `README.md` — Block 1201–1250 "Coaching intelligence & external feedback" roadmap added (15 topics: preparing for first PT session, technical consultation vs. ongoing coaching, video self-analysis limits, partner feedback, AI-trainer 2026 assessment, online vs. in-person coaching, evaluating coach competency, short-term coach engagements, peer feedback communities, self-assessment protocols, overanalysis trap, conflicting coach/body signals, athlete-coach autonomy balance, when to change coaches).

### Changed
- `STATUS.md` — Block 751–800 counter 20 → 21 posts; total posts 764 → 765; v4.89.0 → v4.90.0.
- `blog.html` — card for post-771 added.
- `README.md` — post-771 marked Draft · clinical-safety PASS.
- `VERSION` — 4.89.0 → 4.90.0.

---

## [4.89.0] — 2026-06-14

### Added
- `content/post-769-training-without-gym.md` — Block 751–800 post 19/50. Training without a gym — what can realistically be built with minimal equipment: progressive overload without external load (exercise complexity, tempo, volume, position mechanics), three equipment tiers (level 0 bodyweight-only / level 1 pull-up bar + bands / level 2 minimal home setup), honest ceiling analysis per tier (strength max-effort limited at all levels, hypertrophy realistic at levels 1–2, motor competence unlimited), progression hierarchy examples for push and pull patterns, three common mistakes (replacing progression with volume / avoiding pull movements / treating no-gym as temporary), when gym is genuinely necessary (maximal strength goal, powerlifting/weightlifting, advanced trainees, rehabilitative context). clinical-safety PASS · sports-scientist review required before publish.
- `content/post-770-youth-strength-training.md` — Block 751–800 post 20/50. Youth strength training — evidence, limitations, dosing safety: evidence base (AAP 2008/2020, NSCA position statement Faigenbaum et al. 2009, Collins et al. 2019 meta-analysis: 33–50% injury risk reduction), growth plate myth debunking (risk is improper execution and excessive load, not resistance training per se; osteogenic stimulus documented), three physiological differences from adults (neuromuscular system still developing, hormonal environment pre/post puberty, psychological readiness), programming parameters (supervised mandatory, technique-before-load sequence, load/RPE by age group 12–14 vs 15–17, frequency 2–3 sessions/week, Olympic lifting coaching requirement), benefits documented in youth (bone density — peak bone mass window, neuromuscular adaptation, injury prevention, performance), stop signals. clinical-safety CONDITIONAL PASS (conditions satisfied in text: no body composition language, no 1RM recommendation without qualified coaching, pre-existing condition deferral clause included) · sports-scientist review required before publish.

### Changed
- `STATUS.md` — Block 751–800 counter 751–768 → 751–770 (20/50); next priorities updated to posts 771–772.
- `VERSION` — 4.88.1 → 4.89.0.

---

## [4.88.1] — 2026-06-14

### Changed
- `docs/SOC2_READINESS.md` — §80 Evidence Vault Architecture (v3.7.4 → v3.8.0). Closes OQ-EC-01 (🔴 P1 → 🟢) via DEC-052: R2 bucket `form-soc2-evidence` (EU, Object Lock Governance, versioning locked) is the tamper-evident primary evidence store; Vanta is the auditor-facing mirror with 48-hour upload SLA. Closes OQ-EC-02 (🟡 P2 → 🟢) via §80.5: `pre-obs/` folder + README.md protocol for design-phase evidence separation. New content: §80.1 purpose; §80.2 decision rationale (three alternatives evaluated — Vanta-primary and dual-primary rejected); §80.3 R2 architecture (WORM retention table, `pre-obs/` + `mirror-log/` folder additions, five-row access control matrix with `r2:form-api` NO ACCESS invariant, four bucket policy invariants); §80.4 Vanta mirror protocol (JSONL mirror log schema, mirrored vs. R2-only artefact lists); §80.5 OQ-EC-02 `pre-obs/README.md` authoritative text; §80.6 DEC-030 event `system.evidence_vault_configured` (STANDARD, 7yr, VAULT-CHAIN-01 ordering invariant); §80.7 three SOC 2 evidence artefacts VAULT-E-001/002/003 (CC4.1/CC7.1); §80.8 gap tracker (+0.2 pp, OQ-EC-03 unblocked); §80.9 eleven-item checklist (7× P0/O-1, 2× P1/O+1–O+2, 2× P2/annual); §80.10 open questions status. §79.10 OQ-EC-01 and OQ-EC-02 updated to 🟢 Resolved.
- `docs/DECISION_LOG.md` — DEC-052 added (OQ-EC-01 closure): R2-primary / Vanta-mirror evidence vault architecture; OQ-EC-02 pre-obs separation.
- `VERSION` — 4.88.0 → 4.88.1.

---

## [4.88.0] — 2026-06-14

### Added
- `content/post-768-hotel-training-minimal-effective-session.md` — Block 751–800 post 18/50. Hotel training — minimum effective session without equipment: detraining timeline (Mujika & Padilla 2000 — neuromuscular 10–14 days, structural 3–4 weeks), why most hotel workouts don't count (intensity drop without external load), four bodyweight progression mechanisms (eccentric tempo, leverage/anatomy position, pause in lengthened position, reduced rest), pulling movement limitation (honest coverage), four movement patterns to cover (horizontal push, vertical push, squat, hip hinge), 45-min room-only session protocol with sets/reps/tempo table, hotel gym adaptation (dumbbells to 30 kg, cable machine priority), frequency management (1 session ≤5 days / 2–3 sessions 5–14 days), what not to do (high-rep burpees ≠ strength stimulus). clinical-safety PASS · sports-scientist review required before publish.
- `blog.html` — post-768 card added (Travel training category, draft).

### Changed
- `VERSION` — 4.87.0 → 4.88.0.

---

## [4.87.0] — 2026-06-14

### Added
- `content/post-765-hybrid-athlete-concurrent-training.md` — Block 751–800 post 15/50. Hybrid athlete concurrent training: interference effect molecular mechanism (AMPK/mTOR axis), Wilson et al. (2012) meta-analysis, which strength adaptations are most affected (hypertrophy > max strength), minimum effective dose per direction, same-day sequencing order, inter-session recovery windows by vitrivality session type, weekly microcycle templates (A/B/C), modality-specific interference (running > cycling), block vs. simultaneous periodization, interference detection markers. clinical-safety PASS · sports-scientist review required before publish.
- `content/post-766-entrepreneur-athlete-irregular-schedule.md` — Block 751–800 post 16/50. Entrepreneur-athlete with irregular schedule: why fixed microcycles fail (two failure patterns), floating microcycle structure and session labelling, decision algorithm (window × recovery gap × subjective readiness), allostatic load and non-training stress (Issurin 2010), minimum effective dose for strength maintenance (Bickel 2011) and VO2max (Hickson 1985), session types suited to unpredictable scheduling (full-body vs. mini upper/lower vs. no-equipment), zero-window weeks protocol (10-min morning / active recovery / full pause), travel adaptation, quarterly cycle count as alternative progress metric. clinical-safety PASS · sports-scientist review required before publish.
- `content/post-767-strength-for-runners.md` — Block 751–800 post 17/50. Strength for runners — interference and synergy: running economy mechanism (stretch-shortening cycle, SSC), Berryman et al. (2018) meta-analysis RE improvement 2–8%, muscle-tendon stiffness pathway (Paavolainen 1999), heavy resistance training vs. plyometrics vs. light bodyweight (effectiveness hierarchy), interference benefit for runners (hypertrophy suppressed = mass not gained), bone density and connective tissue arguments, priority exercises (bilateral/unilateral strength, eccentric calf, Nordic, plyometrics), weekly placement rules (not before quality running), pre-competition taper for strength, hypertrophy expectations. clinical-safety PASS · sports-scientist review required before publish.
- `STATUS.md` — Block 751–800 counter 751–764 → 751–767; next priorities updated.

### Changed
- `VERSION` — 4.86.1 → 4.87.0.

---

## [4.86.1] — 2026-06-14

### Changed
- `docs/COST_MODEL.md` — §38 Enterprise Partner Channel & Reseller Economics (v2.3 → v2.4). Resolves OQ-PIPE-03 framework gap: four partner categories (referral 10–15% one-time / reseller 20–25% ongoing / integration 15% attribution / white-label 30–40% gross); partner-vs-direct CAC comparison (referral LTV:CAC 21.9× best class; reseller breakeven condition); `enterprise_partners` Postgres table + `attributed_to_partner_id` FK on pipeline (migrations 0073, 0073b); four DEC-030 HMAC-chained events (`enterprise.partner_agreement_signed`, `enterprise.partner_revenue_share_paid` with `privacy_floor_check_passed: true` hard invariant, `enterprise.partner_deal_attributed`, `enterprise.partner_offboarded` + PART-CHAIN-01 ordering invariant); three SOC 2 evidence artefacts PART-E-001/002/003 (CC9.2/CC4.1/CC7.4); eleven-item implementation checklist; three open questions (OQ-PART-01 PSM hire threshold / OQ-PART-02 co-marketing budget / OQ-PART-03 Direct Migration clause). Privacy floor: no individual health data in any event; form_api REVOKED; partner contact names excluded from chain.
- `VERSION` — 4.86.0 → 4.86.1.

---

## [4.86.0] — 2026-06-14

### Added
- `content/post-764-return-after-viral-illness.md` — Block 751–800 (Athlete lifecycle & context), post 14/50. Повернення після вірусної хвороби: «neck check» як перша фільтрація; ранковий ЧСС як основний маркер готовності; ризик міокардиту — симптоми і explicit redirect до лікаря; постковідний синдром / PEM — другий explicit redirect; три фази протоколу (фаза 0–3, критерії переходу); маркери-стоп-сигнали; J-крива ризику інфекцій (Shephard & Shek 1998); деадаптація при хворобі vs звичайний детренінг (Mujika & Padilla 2000). Clinical-safety: PASS.

### Changed
- `blog.html` — картка post-764 додана першою у grid.
- `STATUS.md` — Block 751–800: 14 постів написано (764); next 765–770.
- `VERSION` — 4.85.0 → 4.86.0.

---

## [4.85.0] — 2026-06-14

### Added
- `content/post-762-training-after-50.md` — Block 751–800 (Athlete lifecycle & context), post 12/50. Тренінг після 50: саркопенія після 50 як якісно відмінний процес від 40+ (прискорення через денервацію волокон типу II, анаболічний резист — Burd et al.); сполучна тканина — зниження синтезу колагену і чому розминка стає структурною вимогою (Couppe et al., Kjaer et al.); відновлення — MPS і запальний cascade; гормональний контекст — вільний тестостерон через SHBG, GH пульси, постменопауза; що НЕ змінюється (progressive overload, specificity, neural efficiency); програмування 50+ vs 40+: частота 2x/тиждень, мезоцикл 3+1, вибір вправ (RDL > touch-and-go), маркери успіху. Clinical-safety: PASS.
- `content/post-763-student-during-exams.md` — Block 751–800 (Athlete lifecycle & context), post 13/50. Студент у сесію: центральна втома (Marcora et al. — RPE drift при незмінній периферії); кортизол і анаболічна супресія; сон і ефективність (Fullagar et al. 2015); алгоритм — три режими (помірне навантаження / MED у пік / MED до кінця при 3+ тижні); ієрархія що зберігати/відрізати (базові рухи + інтенсивність > ізоляція + об'єм); реінтеграція 3 тижні (Belenky et al. 2003). Clinical-safety: PASS.

### Changed
- `README.md` — Block 751–800: posts 751–763 in done table; planned starts at 764.
- `STATUS.md` — Block 751–800 count 13 posts (763); total posts 763; next priorities updated to 764–770.
- `VERSION` — 4.84.3 → 4.85.0.

---

## [4.84.3] — 2026-06-14

### Added
- `docs/DECISION_LOG.md` — DEC-050 entry added (was missing, skipped between DEC-049 and DEC-051). Decision: `form_role_enum` + `role_change_source_enum` Postgres ENUM types adopted; resolves OQ-TURH-01 + OQ-TURH-02 (DATA_MODEL.md §34, migration 0069_role_enum_types.sql).

### Changed
- `STATUS.md` — Block 751–800 next priorities corrected: posts 751–761 written (not 751–760); next: posts 762–800 per README plan (Тренінг після 50, Студент у сесію, etc.). Open decisions table updated with DEC-047 through DEC-051.
- `VERSION` — 4.84.2 → 4.84.3.

---

## [4.84.2] — 2026-06-14

### Changed
- `enterprise.html` — "Drata continuous monitoring" → "Vanta continuous monitoring" (DEC-047).
- `pricing-enterprise.html` — "Drata continuous monitoring" → "Vanta continuous monitoring"; "Drata compliance dashboard access" → "Vanta compliance dashboard access" (DEC-047).
- `VERSION` — 4.84.1 → 4.84.2.

## [4.84.1] — 2026-06-14

### Added
- `content/post-761-new-parent-training.md` — Block 751–800 (Athlete lifecycle & context), post 11/50. Тренінг у перший рік батьківства: фізіологія фрагментованого (не просто скороченого) сну — порушена SWS-архітектура, пригнічена ГР-секреція, підвищений фоновий кортизол, RPE-дрейф. Три принципи програмування (мінімальне ядро, авторегуляція RPE ceiling/floor, нульова заборгованість), алгоритм рішення «йти чи ні» (кількість пробуджень + тижневий рахунок + суб'єктивний стан), ієрархія скорочень (аксесуари → об'єм → інтенсивність остання), три фази 0–12 місяців (виживання / стабілізація / відновлення прогресії), маркери вимушеної паузи. Clinical-safety: PASS — чисте спортивне програмування, жодного постpartum медичного контенту, без body composition.

### Changed
- `blog.html` — додані картки для posts 758, 759, 760, 761 (були відсутні).
- `README.md` — Block 751–800 розширено: повна таблиця 751–761 (done) + roadmap 762–800 (40 нових тем з клінічними нотатками).
- `STATUS.md` — Block 751–800: 10 → 11 постів; Total posts: 760 → 761; version header 4.84.0 → 4.84.1.
- `VERSION` — 4.84.0 → 4.84.1.

## [4.84.0] — 2026-06-14

### Added
- `content/post-758-female-athlete-strength-training.md` — Block 751–800 (Athlete lifecycle & context), post 8/50. Жінки та силовий тренінг: відносний темп набору сили (зіставний між статями), менструальний цикл як тренувальна змінна (фолікулярна vs. лютеїнова фаза — гормональне середовище і практичні акценти), протокол спостереження vs. планування, Q-кут і лаксність суглобів, розподіл м'язових волокон і витривалість при субмаксимальних навантаженнях. Clinical-safety: PASS — pure sports science, no body composition/weight/RED-S content.
- `content/post-759-time-constrained-training.md` — Block 751–800 (Athlete lifecycle & context), post 9/50. Тренінг за умов браку часу: ієрархія змінних (інтенсивність → специфічність → частота), мінімальний ефективний об'єм (Bickel et al., Ralston et al. — 1/3 від об'єму підтримує адаптацію при збереженій інтенсивності), структура 45-хвилинної сесії (2 пріоритети + допоміжний), full-body vs. спліт при 2–3 тренуваннях, 5 рухів з найвищим КПД, суперсети з антагоністами (-20–30% часу без жертв якістю), тимчасовий vs. постійний дефіцит.
- `content/post-760-high-volume-vs-life-stress.md` — Block 751–800 (Athlete lifecycle & context), post 10/50. Конкуренція тренувального і зовнішнього стресу за ресурс відновлення: алостатичне навантаження, об'єктивні маркери конфлікту (HRV, RPE drift, швидкість штанги, сон), чотирирівнева таблиця авторегуляції об'єму, ієрархія скорочень (допоміжний об'єм першим, інтенсивність останньою), HRV як трендовий інструмент (7-денне ковзне середнє), сезонне планування навколо зовнішніх пікових навантажень, різниця functional overreaching vs. non-functional overreaching.

### Changed
- `STATUS.md` — Block 751–800: 7 → 10 постів; Total posts: 757 → 760; next priorities updated.
- `VERSION` — 4.83.4 → 4.84.0.

## [4.83.4] — 2026-06-14

### Added
- `docs/OBSERVABILITY.md §42` — White-Label Custom Domain & SSL Certificate Observability (v3.9). Covers three surfaces: custom domain DNS (CNAME → Cloudflare), SSL certificate lifecycle (Cloudflare Custom Hostnames / SSL for SaaS), and branding asset CDN delivery. Four SLOs (WL-SLO-01 through WL-SLO-04), six alert rules (AL-WL-01 through AL-WL-06), pg_cron job 32 `white_label_cert_check` (02:00 UTC daily, polls Cloudflare Custom Hostnames API), `tenant_white_label_domains` Postgres DDL with RLS, six DEC-030 HMAC-chained events (tenant.white_label_provisioned / cert_expiry_warning / cert_renewed / cert_expiry_breach / revoked + system.white_label_cert_check_stale), WL-CHAIN-01 ordering invariant, three SOC 2 evidence artefacts (WL-E-001 A1.1 / WL-E-002 CC7.2 / WL-E-003 CC7.2), Admin Dashboard "Domain & Branding" panel (tenant-scoped, powered-by-FORM toggle locked below $50k ARR), internal Metabase "White-Label Fleet Health" dashboard. Closes the observability gap for the white-label feature documented in `docs/ENTERPRISE.md §Branding`.

### Changed
- `VERSION` — 4.83.3 → 4.83.4.

## [4.83.3] — 2026-06-14

### Added
- `content/post-757-intermediate-stage-transition.md` — Block 751–800 (Athlete lifecycle & context), post 7/50. Після новачкового етапу: механізм вичерпання лінійної прогресії (нейральний ресурс vs. структурна адаптація), критерії середнього рівня (стабільна відсутність прогресу при коректному відновленні 3–4 тижні), Daily Undulating Periodization (Rhea et al. 2002 — DUP vs. лінійна прогресія), роль варіативності навантаження vs. варіативності вправ, деловантаж кожні 4–6 тижнів, типові помилки переходу, 5 практичних кроків.

### Changed
- `blog.html` — додані картки для постів 752–757 (Athlete lifecycle & context block); раніше в blog.html був лише post-751.
- `STATUS.md` — блок 751–800: 6 → 7 постів; Total posts: 756 → 757.
- `VERSION` — 4.83.2 → 4.83.3.

## [4.83.2] — 2026-06-14

### Changed
- `content/post-755-travel-training.md` — refined final version: expanded opening (cognitive load framing added), 9 sections with decision matrix table for short/medium/extended trips, bodyweight utility ranking table, TRX explicitly dismissed, return plan («no catch-up» rule). 1,318 words.
- `VERSION` — 4.83.1 → 4.83.2.

## [4.83.1] — 2026-06-14

### Added
- `content/post-756-novice-phase.md` — Block 751–800 (Athlete lifecycle & context), post 6/50. Новачковий етап: нейральний феномен (Moritani & deVries 1979 — motor unit recruitment і intermuscular coordination домінують перші 4–8 тижнів), хронологія адаптацій (нейральна → гіпертрофія 8–16 тижнів → консолідація), пастка «серйозної програми» (full-body 3x лінійна прогресія — evidence-backed, не компроміс), що не варто турбувати новачка (спліти, periodization, аксесуари, pre-workout), плато на 3–4 місяці (механізм і реакція), шестимісячна позначка (реалістичні очікування без вигаданих цифр), сигнал виходу з новачкового етапу.

### Changed
- `VERSION` — 4.83.0 → 4.83.1.

## [4.83.0] — 2026-06-14

### Added
- `content/post-752-shift-worker-training.md` — Block 751–800 (Athlete lifecycle & context), post 2/50. Тренінг при змінному графіку роботи: циркадна фізіологія і продуктивність (Reilly & Waterhouse), вплив нічних змін на кортизол, тестостерон і мелатонін, post-sleep window як основний якір тайм-інгу, ротаційні зміни і тиждень переходу (знижуємо інтенсивність, не об'єм), RPE як первинний регулятор навантаження при змінному графіку, матриця «що тримати vs що флексити» (частота, рухові паттерни — фіксовані; час доби, інтенсивність, тривалість сесії — гнучкі).
- `content/post-753-masters-athlete-after-40.md` — Block 751–800, post 3/50. Силовий тренінг після 40: що реально змінюється (саркопенія ~1%/рік після 30, анаболічний гормональний профіль, швидкість відновлення) і що НЕ змінюється (механізми гіпертрофії, нейром'язова адаптація, здатність прогресувати). Пастка «програми для людей похилого віку» — недостатній стимул. Розподіл об'єму vs. скорочення, full-body 3x/week обґрунтування, сполучна тканина: реальний ризик + реальне рішення. Посилання на Straight et al., Leenders et al.
- `content/post-754-return-after-break.md` — Block 751–800, post 4/50. Повернення до тренінгу після перерви: хронологія деадаптації (VO2max -10–14 дні, сила -2–3 тижні, м'язова маса найстійкіша), механізм м'язової пам'яті (збереження міоядер — Bruusgaard et al.), типові помилки повернення (DOMS як хибний сигнал, ігнорування лагу сполучної тканини), практичний фреймворк за тривалістю перерви (1–4 тижні / 1–3 місяці / 3–6 місяців / 6+ місяців), обсяг 40–60% від попереднього, інтенсивність ближча до попередньої, RPE консервативне.
- `content/post-755-travel-training.md` — Block 751–800, post 5/50. Тренінг під час відряджень і подорожей: реальна проблема (не доступ до залу, а порушення сну, часовий зсув, когнітивне навантаження), maintenance dose vs. growth dose, ієрархія готельного обладнання, стратегії за тривалістю поїздки (1–3 дні / 4–7 днів / 2+ тижні), jet lag і тренінг (вікно оптимального виступу у новій зоні), матриця рішень «що зберегти vs що відпустити», план повернення (без наздоганяння об'єму).

### Changed
- `VERSION` — 4.82.1 → 4.83.0.

## [4.82.1] — 2026-06-14

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` — v1.9 → v2.1: §29 added — OQ-SSO-28.2 Resolution: `tenant_users_role_history` Retention Period (DEC-051). Closes the sole remaining P1 open question from §28.9 (v2.0, 2026-06-14). Decision: **7-year retention** for `tenant_users_role_history` from `changed_at`. Three grounds documented in §29.2: (1) audit-chain continuity — `scim.user_updated` DEC-030 events retained 7yr; purging table at 3yr creates irremediable SOC 2 CC6.3 evidence gap (TURH-CHAIN-01 invariant established); (2) legal obligation — Ukrainian Tax Code Art. 44 + EU audit-record doctrine; (3) SOC 2 multi-cycle evidence continuity for 3-cycle Type II programme. GDPR Art. 4(1) analysis (§29.3): rows with `user_id` are pseudonymous personal data; legal basis for 7yr retention is Art. 6(1)(c) + Art. 17(3)(b) (legal obligation); pseudonymisation on employee departure per §28.4 / DATA_MODEL §33.7 satisfies employee Art. 17 rights without destroying audit record; full hard-delete at 7yr boundary. HR floor confirmed: role history accessible only to `compliance_reviewer` and `tenant_admin` (own tenant) — zero HR access. DPA Annex B update language specified in §29.4 for EU enterprise customers. pg_cron job 32 `turh-retention-purge` specified (§29.5): 04:00 UTC daily; safety gate (`user_id IS NULL AND user_id_pseudonym IS NOT NULL`) prevents deletion of active-user rows; Phase 2 fires `system.turh_stale_active_user_detected` HIGH DEC-030 + AL-TURH-01 on stale active-user detection. AL-TURH-01 alert rule specified (§29.6): P2, Slack `#compliance-alerts`, 24h dedup, triage playbook distinguishes long-tenured employees (no incident) from erasure pipeline misses (P1 incident). Two new DEC-030 events: `system.turh_retention_purge_completed` (STANDARD, 1yr) and `system.turh_stale_active_user_detected` (HIGH, 3yr). Evidence artefact TURH-RET-E-001 (§29.8): CC6.3/CC4.1/P5.1 — annual cron run log + AL-TURH-01 zero-fire attestation. Eight-item implementation checklist: 4× P0/M9 (pg_cron job 32, event registration, AL-TURH-01 config, DPA Annex B update), 2× P1/M9 (TURH-RET-E-001 collection, retention-decisions.md entry), 2× P2/annual (chain retention alignment review). OQ-SSO-28.2 updated to 🟢 Resolved in §28.9. TOC updated to add §29. Header v1.9 → v2.1.
- `docs/DATA_MODEL.md` — v1.13 → v1.13 (patch): §33.8 retention table updated — 7-year row status from "Proposed — OQ-SSO-28.2 (P1)" to "🟢 Resolved — DEC-051 (2026-06-14)"; 3-year alternative row updated from "Rejected pending legal review" to "Rejected — DEC-051 (confirmed irremediable SOC 2 gap)"; resolution line updated to 🟢 Resolved. §33.10 checklist item 4 updated to [x] 🟢 Done with DEC-051 reference and explanation of GDPR Art. 4(1) classification + Art. 17(3)(b) legal basis.
- `docs/DECISION_LOG.md` — DEC-051 added (2026-06-14): `tenant_users_role_history` retention — 7 years adopted. DEC-051 is the first entry after DEC-049 (DEC-050 pending migration apply time per DATA_MODEL §34 note).
- `VERSION` — 4.82.0 → 4.82.1.

## [4.82.0] — 2026-06-14

### Added
- `content/post-751-desk-worker-strength-training.md` — Block 751–800 (Athlete lifecycle & context), post 1/50. Силовий тренінг для людей із сидячою роботою: постуральний профіль (APT, tight hip flexors, tight pec minor, обмежена thoracic extension), як ці тенденції впливають на механіку присіду/становової/overhead pressing, логіка програмування (пріоритет горизонтальної тяги ≥ жиму, hip hinge і posterior chain як primary, overhead прогресія від доступного ROM, коротка розминка без foam rolling ритуалу). Без «корективної програми» — акценти у звичайному силовому тренінгу. Refernces: Laird et al. 2014, Christensen & Hartvigsen 2008, Cheatham et al. 2015, Lederman 2011.
- `README.md` — Block 751–800 marked as "in progress"; Block 1151–1200 "Довгострокова адаптація: роки 3–10 тренінгу" added to roadmap (20 topics).
- `STATUS.md` — Block 751–800 row added (1/50, in progress); total posts updated to 751; version bumped to v4.82.0.

## [4.81.0] — 2026-06-14

### Added
- `content/post-743-landmine-exercises.md` — Block 701–750, post 43/50. Landmine вправи як категорія: механіка похилого вектора навантаження, landmine row (однорукова/двохрукова), landmine squat, landmine RDL, landmine lunge і ротаційні варіанти. Коли виправданий (нестандартний вектор, мобільність, ротаційний стимул), коли ні (заміна основних ліфтів, надмірне навантаження). Порівняння з dedicated landmine кріпленням vs. DIY варіантами.
- `content/post-744-paused-reps-bench.md` — Block 701–750, post 44/50. Паузний жим: механіка стретч-рефлексу і еластичної енергії у touch-and-go vs. пауза від нуля, техніка паузи (напруга під час паузи, не відпочинок), довжина паузи (2–3 с стандарт), програмування паузного жиму як основного ліфту паверліфтера і як допоміжного для непаверліфтерів, відсоток від TnG 1RM, довга пауза і 2-пауза варіанти, типові помилки (гриф не торкається, відпочинок у паузі).
- `content/post-745-touch-and-go-vs-dead-stop.md` — Block 701–750, post 45/50. Touch-and-go vs. dead stop deadlift: механічна відмінність (еластична енергія, інерція, стретч-рефлекс у TnG), тренувальний ефект (специфіка для паверліфтингу, гіпертрофія, off-the-floor strength, центральна втома), коли що використовувати, semi-dead stop як компроміс, типові помилки (відскок від підлоги, скорочення амплітуди, ігнорування «втрати» при переході на dead stop для змагань).
- `content/post-746-seasonal-programming.md` — Block 701–750, post 46/50. Сезонне програмування: структура річного циклу (off-season → pre-season → peaking → deload → off-season), off-season як фаза максимального розвитку (більший об'єм, нижча інтенсивність, широкий набір вправ, корекційна і технічна робота), in-season специфіка (pre-season нарощення інтенсивності, peaking реалізація), конкретний приклад для паверліфтера з двома стартами, адаптація для аматора.
- `content/post-747-weightlifting-for-powerlifter.md` — Block 701–750, post 47/50. Вагліфтинговий тренінг для паверліфтера: RFD як незалежна нейром'язова якість, де паверліфтеру потрібна вибухова сила (стартова сила, подолання важкої точки), деривативи (clean pull, hang power clean, power snatch, kettlebell swing) з практичністю для паверліфтера без вагліфтингового бекграунду, доказова база (Baker 2001, Kawamori & Haff 2004, Suchomel et al. 2016), протокол введення по тижнях, місце у блоці (off-season).
- `content/post-748-cross-discipline-athletic-base.md` — Block 701–750, post 48/50. Силові дисципліни і загальна атлетична база: паверліфтинг (абсолютна сила, структурна стійкість, нейром'язова ефективність; дефіцит: RFD, мобільність), вагліфтинг (RFD, потрійне розгинання, мобільність під навантаженням; дефіцит: абсолютна сила, гіпертрофія), стронгмен (grip strength, функціональна витривалість, нестандартні вектори; дефіцит: специфічність ліфту), гімнастика (відносна сила, ізометрична стійкість, пропріоцепція; дефіцит: абсолютна сила зі штангою). Практична матриця «яку якість де брати» + принцип малої дози.
- `content/post-749-competition-day-execution.md` — Block 701–750, post 49/50. День змагань: протокол тижня перед (deload, сон за 2 ночі), розминка (загальна + специфічна, стрибки навантаження, тайм-менеджмент черги, типові помилки), вибір спроб (перший — гарантований успіх @ 90–93%, другий @ 95–100%, третій — PR), психологічний стан (передстартове збудження як норма, Brooks 2014 reframing, фокус на одному технічному елементі, управління між ліфтами), технічні специфіки (взуття, стрічки, команди судді).
- `content/post-750-block-synthesis.md` — Block 701–750, post 50/50. Синтез блоку: 10 наскрізних принципів (специфічність не єдиний принцип; вектор навантаження має значення; стретч-рефлекс ≠ та сама сила; RFD — незалежна якість; програмування — розподіл ресурсів; допоміжні вправи потребують обґрунтування; техніка варіацій ≠ техніка основного ліфту; виступ — окрема навичка; кожна дисципліна має стелю і підлогу; анатомія визначає механіку). Анонс блоку 751–800.

### Changed
- `blog.html` — картки post-743 … post-750 додані на початок сітки.
- `STATUS.md` — рядок 701–742 → 701–750 **BLOCK COMPLETE**; total posts 742 → 750; Next priorities оновлено.
- `README.md` — пости 743–750 позначені Draft у таблиці.
- `VERSION` — 4.80.2 → 4.81.0.

---

## [4.80.2] — 2026-06-14

### Changed
- `docs/DATA_MODEL.md` — v1.12 → v1.13: §34 FORM Role ENUM Types (`form_role_enum` & `role_change_source_enum`). Closes OQ-TURH-01 and OQ-TURH-02 from §33.11 (DEC-050). Creates `form_role_enum` (5 values: `tenant_owner`, `tenant_admin`, `tenant_manager`, `member`, `support_readonly`) matching SSO_SCIM_IMPLEMENTATION §5.1; `role_change_source_enum` (4 values: `scim_sync`, `admin_ui`, `jit_provisioning`, `form_system`). Migration `0069_role_enum_types.sql` migrates `tenant_users.form_role`, `scim_group_role_mappings.form_role`, and `tenant_users_role_history.old_role`/`new_role`/`changed_by` from TEXT+CHECK to typed ENUMs. Closes the silent §5.1 vs. DDL inconsistency (`support_readonly` was documented but missing from DB constraint). Companion assignability CHECK on `scim_group_role_mappings` preserves §5.1 group-mapping restrictions. TypeScript types: `FormRole`, `RoleChangeSource`, `SCIM_ASSIGNABLE_ROLES`. SOC 2 evidence ENUM-E-001/002/003. §33.11 OQ-TURH-01 and OQ-TURH-02 marked 🟢 Resolved.
- `VERSION` — 4.80.1 → 4.80.2.

## [4.80.1] — 2026-06-14

### Added
- `content/post-742-belt-squat.md` — Block 701–750, post 42/50. Belt squat: механіка навантаження на таз vs. аксіальне спинальне навантаження у back squat, EMG-порівняння квадрицепсів і erector spinae (Goodin et al. 2019), показання (управління спинальною втомою при великому об'ємі, тимчасова заміна при болях у спині, квадрицепс-акцент без додаткової компресії), обладнання (dedicated machine vs. dip belt + платформи vs. landmine варіант), програмування (допоміжна вправа vs. заміна в деловд-тижнях), обмеження (не замінює back squat для паверліфтера), типові помилки.
- `README.md` — Block 1101–1150 Силовий тренінг у специфічних контекстах: 10 нових тем у roadmap (shift workers, постpartum, тренування при хронічному дефіциті сну, ранній ранок, важка фізична робота та ін.; клінічні теми марковані clinical-safety review required).

### Changed
- `blog.html` — картка post-742 додана на початок сітки.
- `STATUS.md` — рядок content engine 701–741 → 701–742, total 741 → 742.
- `VERSION` — 4.80.0 → 4.80.1.

## [4.80.0] — 2026-06-14

### Added
- `content/post-737-safety-bar-squat.md` — Block 701–750, post 37/50. Safety bar присід: механіка cambered bar (зміщення навантаження вперед вимагає активного thoracic extension), EMG порівняння (Hecker et al. 2019, Yavuz et al. 2015 — вищий vastus lateralis), показання (обмеження мобільності плечей, кіфоз, варіативність), коли barbell переважає (паверліфтинг-специфіка, posterior chain), типові помилки.
- `content/post-738-landmine-press.md` — Block 701–750, post 38/50. Landmine press: діагональна площина 30–45° від вертикалі (Saeterbakken et al. 2021 — нижчий субакроміальний тиск vs OHP), EMG: anterior deltoid + верхній pec + lower trap + serratus, 4 варіанти (стоячи/half-kneeling/білатеральний/push-press), програмування для спортсменів з імпінджментом плеча і ротаційних видів спорту.
- `content/post-739-nordic-hamstring-curl.md` — Block 701–750, post 39/50. Нордичне згинання ніг: ексцентрична механіка при подовженому м'язі (van Dyk et al. 2019 meta-analysis, Bourne et al. 2017 biceps femoris hypertrophy), порівняння кривої сила-довжина з leg curl, прогресія від assisted негативів до повного Nordic, програмування для спринтерів і командних ігор.
- `content/post-740-paused-bench-press.md` — Block 701–750, post 40/50. Паузний жим лежачи: усунення SSC і старт з мертвої точки, 10–15% зниження навантаження vs. TNG, EMG (Schick et al. 2010 — вища активація pectoralis major), порівняльна таблиця кривих сили (TNG/paused/pin press/board press), специфіка паверліфтингу (1–2 с пауза — змагальна вимога), програмування у пікінг-блоці.
- `content/post-741-competition-peaking-tapering.md` — Block 701–750, post 41/50. Пікування і тейпер перед стартом: 3-тижнева модель (Banister dual-factor, Sheiko/Juggernaut), управління обсягом vs. інтенсивністю (Mujika & Padilla 2000, Bosquet et al. 2007), вибір спроб (90–93% ТМ для відкривача), помилки пікування, тиждень-за-тижневий шаблон і протокол розминки на змаганнях.

### Changed
- `VERSION` — 4.79.1 → 4.80.0.

## [4.79.1] — 2026-06-14

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.4 → v2.6: +1 `scim.session_revocation_kv_fallback` (HIGH, 7yr) — closes SSO_SCIM_IMPLEMENTATION.md §28.8 checklist item 2 (P0 M5). Registers the DEC-048 dual-path session revocation fallback event in §6.SCIM-Lifecycle; adds `ScimSessionRevocationKvFallbackSchema` Zod schema; REVOKE-CHAIN-01 ordering invariant (must follow `scim.user_deprovisioned` for same `scim_request_id`, HTTP 422 on violation); retention table row (CC6.3 + CC7.2). Header corrected from v2.4 → v2.6 (v2.5 entry existed without header bump). Privacy invariant: no `user_id`, no email, `tenant_id` + `scim_request_id` only; `supabase_ok = false` co-activates R-02 + R-26.
- `docs/SSO_SCIM_IMPLEMENTATION.md §28.8` — checklist item 2 marked [x] Done.
- `VERSION` — 4.79.0 → 4.79.1.

## [4.79.0] — 2026-06-14

### Added
- `content/post-736-dips-chest-vs-triceps.md` — Block 701–750, post 36/50. Дипс: механіка нахилу тулуба (вертикальний → трицепс, 30–45° → грудні), EMG-дані Barnett et al. 1995, механіка плечового суглоба і нижня точка (anterior capsule), прогресія від бандажу до зваженого варіанту, кільцеві vs. паралельні ручки, програмування поруч із жимом лежачи, типові помилки (провал плечей, надмірна глибина без підготовки).

### Changed
- `blog.html` — картка post-736 додана на початок сітки.
- `STATUS.md` — рядок блоку 701–750: 35 → 36 постів; Total posts: 735 → 736; Next priorities оновлено.
- `VERSION` — 4.78.0 → 4.79.0.

## [4.78.0] — 2026-06-14

### Added
- `content/post-731-pull-up-programming.md` — Block 701–750, post 31/50. Pull-up і chin-up: пронований vs супінований хват, лопатко-депресія як ключовий технічний елемент, прогресія від негативів до weighted pull-up, lat pulldown як регресія, типові помилки (кіппінг, напівповтори), програмування у PPL/Upper-Lower.
- `content/post-732-bulgarian-split-squat.md` — Block 701–750, post 32/50. Болгарське спліт-присідання: роль довжини кроку (quad vs glute акцент), stretch-mediated навантаження клубово-поперекового, виявлення і корекція міжсторонніх асиметрій, DOMS при першому введенні та протокол введення, варіанти навантаження (гантелі/штанга/KB), місце у програмі як доповнення до білатерального присідання.
- `content/post-733-triceps-pressing-strength.md` — Block 701–750, post 33/50. Трицепс у силовому тренінгу: три головки і їх функціональна різниця, close-grip bench press як силова (не ізоляційна) вправа, overhead extension для довгої головки (Maeo et al. 2021 data), skull crusher EZ vs пряма штанга, JM press, типові помилки відстаючого трицепса, програмування під різні цілі.
- `content/post-734-box-squat-pause-squat.md` — Block 701–750, post 34/50. Бокс-присідання і паузне присідання: Westside vs EU підхід до боксу, чому 2–3 с пауза (не 1 с) для усунення SSC, роль у подоланні bottom sticking point, порівняльний трансфер на конкурентне присідання, програмування off-season vs peaking.
- `content/post-735-trap-bar-deadlift.md` — Block 701–750, post 35/50. Трап-бар станова тяга: механічне порівняння з конвенційною (Swinton et al. 2011, Camara et al. 2016), чому трап-бар дозволяє більшу вагу, visока vs низька ручка, трансфер на вибуховість і стрибки, показання для вибору трап-бару, програмування поруч з конвенційною.

### Changed
- `README.md` — Block 701–750 прогрес 22/50 → 35/50; пости 723–730 → статус Draft; пости 731–735 → фактичні теми + статус Draft; теми 736–739 оновлено (переміщені теми: Дипс, Прес ногами, Фермерська ходьба, Шраги); 740–750 реорганізовано.
- `VERSION` — 4.77.1 → 4.78.0.

## [4.77.1] — 2026-06-14

### Changed
- `docs/INCIDENT_RESPONSE.md §17` — New section: `siem-incident-automator` Resilience Design — Incident ID Pre-generation & Free-Text Privacy Controls. Closes OQ-IR-02 (P1) and OQ-IR-03 (P2) from §16.9. OQ-IR-02: Option A adopted — `incident_id` (`INC-YYYYMMDD-[6hex]`) generated before any external API call; `incident.opened` emitted at T+0; `incident.linear_ticket_linked` (new LOW event, 7yr) closes linkage asynchronously via 5× exponential-backoff retry or IC amendment endpoint `POST /internal/v1/incident/link-ticket`; AL-IR-LINEAR-01 (P2 Slack) on persistent failure. OQ-IR-03: Option A + SHA-256 hash masking (consistent with DEC-044 `bypass_reason_hash` pattern, §40): `reason_plaintext` replaced by `reason_hash` in DEC-030 `incident.severity_changed`; `INCIDENT_REASON_HASH_SALT` added to CRYPTOGRAPHY_POLICY.md §5 (annual rotation); runtime blocklist (5 patterns) emits advisory `incident.pii_risk_detected` (new MEDIUM event, 7yr — no blocking); auditor fieldwork protocol at §17.3.6. Three SOC 2 evidence artefacts: IR-AUTO-E-001/002 (CC7.4), IR-AUTO-E-003 (P3.2). 11-item checklist (5× P0/M4, 3× P1/M5, 3× P2/M9). §16.9 OQ-IR-02 and OQ-IR-03 marked 🟢 Resolved. Document header v2.3 → v2.4. Owner: security-engineer + platform-engineer + compliance-officer.
- `VERSION` — 4.77.0 → 4.77.1.

## [4.77.0] — 2026-06-14

### Added
- `content/post-723-squat-depth-anatomy.md` — Block 701–750, post 23/50. Анатомія глибини присідання: чому ваш оптимум відрізняється від чужого. Чотири ключові анатомічні параметри (глибина і орієнтація вертлюгової западини, кут шийки стегна, феморальна версія, довжина стегна); диференціація анатомічного vs. мобільнісного обмеження; FAI типи (cam / pincer / змішаний); гомілковостопна дорсифлексія (тест коліно-до-стіни, ≥10 см норма, 4–8 тижнів корекції); геометрія пропорцій стегно/гомілка/тулуб; стратегія за профілем (висока антеверсія / ретроверсія / довгі стегна); практичний тест власної оптимальної стійки. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-724-bench-arch.md` — Block 701–750, post 24/50. Арка в жимі лежачи: механіка, функція і поширені непорозуміння. Торакальна vs. поперекова арка; ретракція і депресія лопаток — механічний зміст; leg drive (flat feet vs. tucked — механіка передачі); доказова база безпеки арки (Lehman 2005, Araujo 2010); правила IPF (що легально, що є bridge і є порушенням); розвиток арки (foam roller extension, shoulder dislocates); типові помилки (таз, лопатки, кут ліктів). clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-725-conventional-vs-sumo.md` — Block 701–750, post 25/50. Conventional vs. сумо: вибір стилю станової тяги. Біомеханічні відмінності (позиція тулуба, шлях грифа, коліна); EMG-дані (Escamilla et al. 2000/2002, Swinton 2011) — таблиця по групах; анатомічні фактори вибору (зовнішня ротація, довжина стегон, силовий профіль); практичний 6–8 тижневий тест; типові помилки по кожному стилю; програмування обох стилів (12-тижнева схема); термін адаптації при зміні стилю (3–6 місяців). clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-726-ohp-variants.md` — Block 701–750, post 26/50. Варіанти жиму над головою: від суворого пресу до пуш-пресу і лендмайну. Суворий прес (J-крива траєкторія, активний шраг, Saeterbakken 2013 кор-активація); сидячи vs. стоячи; пуш-прес (dip-and-drive, 10–30% більше ваги, програмні функції: overload / швидкість / обсяг при втомі); лендмайн-прес (нижчий субакроміальний тиск, Saeterbakken 2021, однобічний варіант); Z-прес як діагностичний і корекційний інструмент; жим Арнольда; жим гантелей (двосторонній і однобічний, стабілізація латерального тулуба); програмна матриця; наскрізні помилки. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-727-front-squat.md` — Block 701–750, post 27/50. Фронтальний присід: механіка, вимоги та програмування. Клін-гріп vs. схрещені руки (мобільність зап'ясток і плечей); біомеханічне порівняння з присідом на спині — таблиця (кут тулуба, квадрицепси, поперек, вага); Gullett et al. 2008 (вертикальніший тулуб, нижчий поперековий стрес); Yavuz et al. 2015 (вища квадрицепсова активація); вимоги до торакальної мобільності і дорсифлексії; антифлексійна функція кора під горизонтальним вектором навантаження; програмні функції (важкоатлети: специфічність; дефіцит квадрицепсів; пауерліфтери — допоміжна торакальна жорсткість); типові помилки (падіння ліктів, підняття п'яток, вальгус, рух «вгору-назад»). clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-728-romanian-dl.md` — Block 701–750, post 28/50. Румунська станова тяга: механіка, навантаження на задню поверхню і програмування. Порівняльна таблиця РДТ / stiff-leg DL / конвенційна; три ознаки правильного хіп-хінжу; нижня точка визначається першим натягом підколінних (не підлогою); EMG-дані Hegyi et al. 2019 (вища активація у нижній фазі порівняно з leg curl), Ebben et al. 2009 (пікова активація в ексцентрику); ексцентрична підготовка і зниження ризику травм підколінних (Bourne et al. 2017); варіанти (одноногова, гантелі, trap bar); програмна таблиця за ціллю з темпом. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-729-hip-thrust.md` — Block 701–750, post 29/50. Гіп-траст: горизонтальне навантаження сідниць і трансфер на спринт. Горизонтальний вектор навантаження vs. вертикальний (присідання/тяга); EMG Contreras et al. 2010/2015 (119% MVIC великого сідничного vs. ~72% при присіданні) із застереженням про обмеження EMG-даних; трансфер на спринт — Seitz et al. 2019 RCT (перевага у прискоренні 0–10 м vs. присідань); горизонтальна сила і Morin & Samozino 2016; chin tuck і нейтральна хребтина у верхній точці; розгинання стегна vs. гіперекстензія поперека; налаштування (~40–42 см лава); варіанти (одноногий, elevated foot, ізометричний, з бандою); програмна таблиця. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-730-horizontal-row.md` — Block 701–750, post 30/50. Горизонтальні тяги: варіанти, м'язова анатомія і програмування. Анатомія горизонтальної тяги (ромбоподібні, середня і нижня трапеція, задня дельта, широкий); варіанти: bent-over row (пронований/супінований/Pendlay), chest-supported row, cable seated row (вузький/широкий хват), одноручна гантель, тренажер; тяга до пояса vs. до грудей — різні акценти; EMG-дані Fenwick et al. 2009 і Lehman et al. 2004; баланс жим:тяга (1.5:1–2:1 за обсягом — причини і механізм); програмна таблиця за ціллю; типові помилки (тяга рукою замість лопаткою, розмах тулуба, кидання ексцентрика). clinical-safety: PASS. sports_scientist_review: required before publish.

### Changed
- `content/post-705-female-physiology-strength.md` — clinical-safety CONDITIONAL PASS: (1) примітка про індивідуальний ризик ACL для осіб з анамнезом травми або симптоматичною нестабільністю колінного суглоба (консультація фізіотерапевта/спортивного лікаря обов'язкова); (2) ресурс-сигнал після секції Тріади — направлення до спортивного лікаря або ендокринолога при порушеннях циклу, IOC RED-S Consensus Statement (Mountjoy et al. 2018, BJSM). Frontmatter оновлено: REQUIRES REVIEW → CONDITIONAL PASS; publish_gate і status: draft видалені.
- `content/post-706-youth-strength-training.md` — clinical-safety PASS editorial note applied: вимога нагляду для роботи 85%+ зроблена візуально помітною (bold + уточнення «обов'язкова умова, а не рекомендація»). Frontmatter оновлено: REQUIRES REVIEW → PASS; publish_gate і status: draft видалені.
- `STATUS.md` — content table: 701–722 (22 posts) → 701–730 (30 posts, 705–706 clinical-safety updated); total posts 722 → 730; next priorities оновлено.
- `VERSION` — 4.76.2 → 4.77.0.

## [4.76.2] — 2026-06-14

### Added
- `content/post-722-one-rep-max-testing-periodization.md` — Block 701–750 Strength sports specifics, post 22/50. Одноповторний максимум: тестування, планування, periodization навколо 1RM. e1RM формули і їх обмеження (Epley, Brzycki, Lombardi; похибка ±5–10% поза діапазоном 3–10 повторень); коли виправдано тестувати 1RM пряму і коли ні (накопичена втома, технічна нестабільність, < 6–12 місяців тренінгу); протокол прямого 1RM-тесту (підготовка за кілька днів, схема розминки з відсотками, кількість спроб, відпочинок між спробами); training max vs. competition max і чому TM = 85–90% від реального 1RM; три моделі periodization (лінійна, DUP/WUP, autoregulated RPE/RIR); частота тестувань по рівнях атлетів; 5 типових помилок при роботі з 1RM; реалістичні очікування приросту за цикл по рівнях. clinical-safety: PASS. sports_scientist_review: required before publish.
- `blog.html` — картка post-722 додана (Strength sports specifics, 13 хв, 2026-06-14, draft)
- `STATUS.md` — content engine рядок оновлено: 701–721 → 701–722 (22 пост), total posts: 722; next priorities: 723–730
- `README.md` — Block 701–750 прогрес оновлено: 21/50 → 22/50; post-721 і post-722 статус Planned → Draft

## [4.76.1] — 2026-06-14

### Changed
- `docs/SOC2_READINESS.md §79` — Master Evidence Collection & Auditor Submission Protocol. Closes CC4-GAP-002 (🔴 Open → 🟡 Authored): the §32 CC4-GAP-002 task called for `compliance/cc4/evidence-collection-plan.md` (CC4-E-002) documenting what evidence is collected per CC4.1 control, cadence, storage path, and responsible owner. §79 delivers this as the in-document master plan (with §79.9 item 1 requiring it to be additionally filed as the standalone CC4-E-002 artifact). Content: evidence folder structure for `compliance/evidence/` with 16 TSC-domain subfolders (§79.2); three evidence classes — Auto / Manual-periodic / Manual-event (§79.3); 31-row consolidated evidence collection table mapping all primary artefacts across all 5 TSC domains to criteria, cadence, owner, and R2 file path (§79.4); observation period calendar with pre-obs gate actions, monthly recurring actions, quarterly filing cycle, and Month O+12 fieldwork preparation (§79.5); auditor fieldwork response SLA table — artefact-on-hand 2-business-day / query extraction 5-business-day / walkthrough 3-business-day / gap 24h-escalate — plus walkthrough recording procedure and five categories of information that must never be shared with auditors directly (§79.6); evidence integrity verification via SHA-256 checksums per TSC folder plus HMAC chain integrity SQL query to run before every DEC-030 evidence submission (§79.7); gap tracker CC4-GAP-002 🔴 → 🟡 (+0.4 pp readiness on full execution) (§79.8); nine-item implementation checklist — 4× P0/Month-O-1, 3× P1/Month-O+1–O+9, 2× P2/M18+ (§79.9); three open questions — OQ-EC-01 R2 vs Vanta primary store, OQ-EC-02 pre-observation evidence separation, OQ-EC-03 monthly SLA + chain check automation (§79.10). SOC2_READINESS.md v3.7.3 → v3.7.4. Owner: compliance-officer (enterprise-builder).

## [4.76.0] — 2026-06-14

### Added
- `content/post-721-raw-vs-equipped-powerlifting.md` — Block 701–750 Strength sports specifics, post 21/50. Raw vs. equipped паверліфтинг: різна техніка, різне програмування. Squat suit як акумулятор пружної енергії (carryover 5–40%), bench shirt механіка (атлет тягне штангу вниз проти опору shirt, а не опускає), deadlift suit (3–8% carryover). Технічні відмінності ліфт за ліфтом: forced ширша стійка і low-bar у equipped squat; активний тяг і groove-специфічна техніка в bench shirt; мінімальні відмінності в DL. Програмувальні відмінності: gear sessions обов'язкові pre-competition, off-season — переважно raw; дві незалежні RPE-шкали; допоміжні вправи під shirt вимоги (більше горизонтальних тяг). Що raw бере з equipped методології і навпаки. Коли equipped виправданий і коли ні. clinical-safety: PASS. sports_scientist_review: required before publish.
- `blog.html` — картка post-721 додана (Strength sports specifics, 13 хв, 2026-06-14, draft)
- `STATUS.md` — content engine рядок оновлено: 701–721 (21 пост), total posts: 721; next priorities: 722–730

## [4.75.0] — 2026-06-14

### Added
- `content/post-712-gymnastics-relative-strength.md` — Block 701–750 Strength sports specifics, post 12/50. Гімнастична відносна сила: чому гімнасти сильні без штанги. Відносна vs. абсолютна сила (нелінійний ефект маси), ізометричні вимоги хреста і планшу (момент сили vs. конценричний рух), сухожилкова адаптація і чому потрібні роки підготовки, нейром'язова ефективність (низький EMG при рівному силовому виводі). Практичні інструменти: hollow body, ring row, L-sit, front lever tuck, tuck planche. Трансфер у зворотньому напрямку (шквал дає найбільший ефект для ніг). clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-713-cycling-strength-training.md` — Block 701–750, post 13/50. Велоспорт і силовий тренінг: concurrent scheduling, quad dominance. Що велоспорт розвиває і де межі; quad dominance документований (Bini & Hume 2013); interference effect — AMPK vs. mTOR механізм, стратегії управління; докази (Rønnestad & Mujika 2014 мета-аналіз 14 досліджень: heavy strength > moderate strength для cycling economy і FTP); пріоритет вправ (hip hinge і split squat > leg extension); 4×4 RM vs. 3×10 RM — важкий протокол кращий для велосипедиста; сезонний concurrent scheduling. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-714-swimming-dryland-strength.md` — Block 701–750, post 14/50. Плавання і dryland strength: overhead overuse і специфіка. 1–1.5 млн гребків/рік специфіка; swimmer's shoulder механізм (IR/ER дисбаланс, нижній трапецій і serratus anterior слабкість); трансфер сили у воду (Girold et al. 2007 — pulling strength > push); найефективніші dryland вправи (pulling dominant, NOT overhead press); IR/ER ratio контроль; concurrent scheduling — менш критичний ніж у велоспорті, але pulling dryland і тяжке плавання розносити; off-season / pre-season / in-season periodization. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-715-combat-sports-strength.md` — Block 701–750, post 15/50. Бойові мистецтва і силовий тренінг: grip, вибухова потужність, ізометрична сила. Три компоненти grip (crushing / pinching / support); найефективніші grip вправи (farmer's walk, towel pull-up, thick bar, plate pinch); вибухова потужність: RFD як ключовий параметр (Lenetsky et al. 2013 — 50–100 мс contact time); olympic lifts для RFD (Suchomel et al. 2016 — найвищий трансфер); ізометрична специфічність у бойових кутах; ударні vs. грепплінг силові профілі; міф «штанга сповільнює» (Falvo et al. 2012 спростування); структура 2–3 сесії/тиждень, низький об'єм висока інтенсивність. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-716-wrestling-strength-training.md` — Block 701–750, post 16/50. Борьба і силовий тренінг: ізометрика, grip, wrestling-specific programming. Силовий профіль борця (NCAA Division I, Hoffman et al. 2005); вибухова перша крок (RFD для takedown); ізометрика у борцівських кутах (specificity principle, конкретні вправи); double-leg і single-leg takedown вправи; grip без кімоно; neck ізометрія; ваговий клас і небезпека форсованого нарощування маси; типовий тиждень competition-prep; прогресія об'єму off-season vs. in-season. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-717-tennis-strength-training.md` — Block 701–750, post 17/50. Тенніс і силовий тренінг: ротаційна потужність, асиметрія, serve mechanics. Кінетичний ланцюг (50–55% швидкості ракетки від ніг і тулубу, Elliott et al. 2003); ротаційна потужність: medball rotational throw (Chelly et al. 2010 — +12% serve velocity), cable woodchop, Pallof; структурна асиметрія — кісткова (Krahl et al. 1994) і м'язова IR/ER дисбаланс; IR/ER ratio ≥ 75% (Ellenbecker & Roetert 2004); serve mechanics і leg drive кореляція (Reid & Schneiker 2008); нижні кінцівки (300–500 COD / матч); off-season / in-season periodization. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-718-team-sports-gym-prep.md` — Block 701–750, post 18/50. Ігрові командні спорти і залова підготовка: concurrent scheduling для аматорів. Атлетичний профіль командних спортів (acceleration, COD, jump, RSA, контактна сила); interference менш виражений ніж у чистих витривалісних видах; правило 48 годин перед матчем; два сценарії тижневого розкладу; вибір вправ (jump squat, split squat, RDL, box jump > leg extension і machines); об'єм нижчий ніж у спеціалізованого атлета (12–18 сетів/тиж); in-season vs. off-season periodization; приклад in-season сесії 60 хвилин. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-719-climbing-strength-training.md` — Block 701–750, post 19/50. Скалолазіння і силовий тренінг: finger strength, pulling strength, antagonist balance. Finger strength специфіка (FDP/FDS, crimp vs. open hand, сухожилкова адаптація повільніша ніж м'язова); hangboard протоколи (Maximum Hangs Method і repeater; лише після 6–12 місяців лазіння); pulling strength (weighted pull-up, lock-off ізометрія, front lever); antagonist balance — основна травматична зона (Heinz et al. 2020 — extensor/flexor дисбаланс = вищий pulley injury ризик); finger extension rubber band protocol; external rotation і pushing maintenance; вагові міркування (відносна сила через збільшення сили > зниження ваги; клінічна межа позначена). clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-720-first-powerlifting-meet.md` — Block 701–750, post 20/50. Перший старт у паверліфтингу: підготовка, відбір спроб, день виступу. Коли готовий до першого старту; вибір федерації і raw vs. equipped; 4-тижневий peak протокол із таблицею; відбір спроб (перша = 90–92% очікуваного max, формула розрахунку, золоте правило консервативності); зважування і екіпіровка; розминкова прогресія; команди судді (squat / bench / deadlift — повна процедура); типові червоні за кожен ліфт; 5 найпоширеніших помилок першого старту. clinical-safety: PASS. sports_scientist_review: required before publish.

### Changed
- `README.md` — Block 701–750 прогрес оновлено: 10/50 → 20/50; posts 711–720 статус Planned → Draft.
- `STATUS.md` — Content table updated: 701–711 → 701–720, total posts 711 → 720; next priorities updated to 721–730.
- `VERSION` — 4.74.1 → 4.75.0.

---

## [4.74.1] — 2026-06-14

### Changed
- `docs/OBSERVABILITY.md` — §26.7b SCIM Endpoint Operations alert rules (AL-SCIM-01..04): canonical registration of four SCIM Worker endpoint alerts missing from OBSERVABILITY §26 (defined in SSO §27.11 but never added here). AL-SCIM-01 (P1 — sensitive attribute rejection burst; CC6.1 evidence), AL-SCIM-02 (P2 — 5xx spike; Cloudflare AE signal), AL-SCIM-03 (P2 — sync stall; pg_cron `scim_sync_stall_check` SQL included), AL-SCIM-04 (P1 — SCIM-CHAIN-01 ordering invariant; R-05 co-activation; root-cause taxonomy). §26.7c SCIM Role History Reconciliation (AL-SCIM-05): **resolves OQ-SSO-28.1** — AL-SCIM-05 confirmed available and formally registered; pg_cron `scim_role_history_reconcile` (nightly 04:00 UTC; reconciliation SQL; P2 Slack `#compliance-alerts`; 24-hour dedup; privacy floor: gap_count only). §26.8 updated: `scim_endpoint_operations` (4 rows) + `scim_role_history` (1 row) added to §6.2 consolidated alert table. §26.11 updated: CC7.2 count sixteen → twenty-one; CC7.3 runbook refs extended to R-05; SSO-OBS-E-006 evidence artefact added. §26.12 updated: eight new checklist items (6× P0/M5, 1× P1/obs, 1× P2/M10). v1.3.2 patch.
- `docs/SSO_SCIM_IMPLEMENTATION.md` — OQ-SSO-28.1 closed: AL-SCIM-05 confirmed available; canonical definition in OBSERVABILITY §26.7c. v2.0.1 patch.
- `VERSION` — 4.74.0 → 4.74.1.

---

## [4.74.0] — 2026-06-14

### Added
- `content/post-711-bodybuilding-vs-strength-training.md` — Block 701–750 Strength sports specifics, post 11/50. Бодібілдинг vs. силовий тренінг: що кожна система оптимізує (гіпертрофія vs. нейром'язова адаптація), порівняння об'єму / інтенсивності / пауз між підходами, логіка вибору вправ (ізоляція vs. специфічність), periodization архітектура (block vs. лінійний), proximity to failure і RPE, що силовий атлет може взяти з бодібілдингу і навпаки, powerbuilding як обґрунтована синтетична модель. clinical-safety: PASS. sports_scientist_review: required before publish.

### Changed
- `STATUS.md` — Content table updated: 701–710 → 701–711, total posts 710 → 711.
- `VERSION` — 4.73.0 → 4.74.0.

---

## [4.73.0] — 2026-06-14

### Added
- `content/post-701-powerlifting-basics.md` — Block 701–750 Strength sports specifics, post 1/50. Паверліфтинг для початківців: три ліфти (SBD), федерації і правила, чому паверліфтинг корисний і для тих хто не планує змагатись (структура, прогресія, вимірюваність), техніка кожного ліфту, перша програма 3x/тиждень лінійна прогресія, типова periodization arc, помилки початківця, коли виходити на перший старт. sports_scientist_review: required before publish.
- `content/post-702-weightlifting-for-athletes.md` — Block 701–750, post 2/50. Weightlifting movements (clean, snatch) для неважкоатлетів: механіка triple extension, доказова база трансферу на атлетизм (RFD, vert jump, sprint), спрощені варіанти (power clean, hang clean, DB snatch), де weightlifting НЕ потрібне (гіпертрофія, базова сила), технічна крива навчання, включення у мікроцикл. sports_scientist_review: required before publish.
- `content/post-703-crossfit-methodology.md` — Block 701–750, post 3/50. CrossFit як GPP-система: що реально розвиває (метаболічна кондиція, варіативність рухів, work capacity), де не оптимальний (максимальна сила, гіпертрофія, специфічна витривалість), рівень травматизму за даними (Claudino 2018, Hak 2013), rhabdomyolysis ризик, для кого підходить, поєднання з силовим тренінгом. sports_scientist_review: required before publish.
- `content/post-704-circadian-training-timing.md` — Block 701–750, post 4/50. Циркадний ритм і час тренування: core body temperature і нейром'язова функція, peak performance о 15–18 год за популяційними даними, тестостерон/кортизол — чому ранковий пік не = «кращий час», ефект хронотипу (Facer-Childs 2018/2019), адаптація до конкретного часу за 6–10 тижнів (Sedliak 2008, Chtourou 2012), вечірнє тренування і сон (Stutz 2019), висновок: consistency > теоретичний оптимум. sports_scientist_review: required before publish.
- `content/post-705-female-physiology-strength.md` — Block 701–750, post 5/50. Жіноча фізіологія і силовий тренінг: ендокринний профіль відмінності, ACL лаксість і гормональний цикл (фолікулярна фаза ризик), menstrual cycle і тренувальна продуктивність по фазах, сила per kg lean mass — відмінності мінімальні, практична авторегуляція за фазою, RED-S коректно флагований як клінічна тема з рефералом до лікаря. clinical-safety: REQUIRES REVIEW перед публікацією. sports_scientist_review: required before publish.
- `content/post-706-youth-strength-training.md` — Block 701–750, post 6/50. Силовий тренінг для підлітків: позиція NSCA/ACSM/AAP (безпечний при правильному дозуванні), реальний ризик зон росту vs. myth (більшість травм — від контактних видів, не силового тренінгу), нейром'язовий механізм у підлітків (нейральний, не гіпертрофічний до гормонального піку), bone mineral density ефект, практичне програмування (2-3x/тиждень, 50–70% → 85%+ при стабільній техніці), різниця pre/post-puberty. clinical-safety: REQUIRES REVIEW перед публікацією. sports_scientist_review: required before publish.
- `content/post-707-strength-for-runners.md` — Block 701–750, post 7/50. Сила для бігунів: сухожилкова жорсткість і running economy (Bohm 2015), нейром'язова ефективність, мета-аналізи Berryman 2018 і Blagrove 2018, тижнева програма 2x/тиждень (важка робота 80–90% 1RM), interference effect — реальний лише при великому об'ємі (Wilson 2012), різниця для half-marathon vs. ultra. sports_scientist_review: required before publish.
- `content/post-708-weighted-vest-training.md` — Block 701–750, post 8/50. Тренування з обважниками: GRF механіка, де weighted vest має сенс (bodyweight вправи з прогресивним навантаженням, навантажена ходьба, bone loading), де не має сенсу (замість штанги, технічне навчання, plyometrics при надмірній вазі), постуральна адаптація, практичне дозування за контекстом. sports_scientist_review: required before publish.
- `content/post-709-kettlebell-sport-for-strength-athletes.md` — Block 701–750, post 9/50. Гирьовий спорт і silovий атлет: різниця між girevoy sport (10-хв set) і kettlebell fitness, механіка swing/clean/снатч, реальні адаптації (grip endurance, posterior chain, lactate threshold), де kettlebell не замінює штангу (максимальна сила, гіпертрофія), трансфер, включення у програму 1–2x/тиждень. sports_scientist_review: required before publish.
- `content/post-710-strongman-methodology.md` — Block 701–750, post 10/50. Стронгмен як методологія: farmer's walk (grip endurance, тулубна жорсткість), log press/axle press (нейтральний grip, менше навантаження на anterior capsule), atlas stones (технічна вимога, posterior chain), tire flip і sled, практичні альтернативи без спеціалізованого обладнання, блоки дозування farmer's walk і sled GPP. sports_scientist_review: required before publish.

### Changed
- `README.md` — Block 701–750 розширено з (proposed, 8 тем) до (in progress, повний 50-topic list з post/status колонками). Перші 10 постів позначені як Draft.
- `STATUS.md` — Content table updated: posts 701–710 додано, total posts 700 → 710; next priorities updated; Block 651–700 позначено як COMPLETE.
- `VERSION` — 4.72.1 → 4.73.0.

---

## [4.72.1] — 2026-06-14

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md §28` — SCIM Role Change Audit Trail & Session Revocation Fallback Design. Resolves OQ-SSO-27.2 (DEC-049) and OQ-SSO-27.3 (DEC-048), both P1 open questions from v1.9 (2026-06-13). §28 covers: `tenant_users_role_history` append-only table approach for role change audit trail (not DEC-030 chain — cross-tenant leakage prevention); KV write failure fallback protocol (option (c): HTTP 200 + Supabase blocklist fallback + `scim.session_revocation_kv_fallback` DEC-030 event HIGH 7yr + AL-REVOKE-01 P1); `recordRoleChange()` TypeScript helper (non-fatal design); `revocationPath` try/catch in PATCH deprovisioning; DPA G-013 dual-path revocation language; three SOC 2 evidence artefacts (SCIM-ROLE-E-001/E-002/SCIM-REVOKE-E-001); twelve-item checklist (6× P0/M5, 4× P1/M9, 2× P2/M10); two new open questions (OQ-SSO-28.1 AL-SCIM-05 naming check P2; OQ-SSO-28.2 retention period P1).
- `docs/DATA_MODEL.md §33` — `tenant_users_role_history` canonical DDL definition. Migration `0068_tenant_users_role_history.sql`: nine columns; `chk_turh_role_changed` (no-op guard); `chk_turh_user_xor_pseudonym` (GDPR Art. 17 erasure path); FK `user_id` SET NULL on user delete; three indexes (tenant_user partial, tenant_time, scim_request); six RLS policies (`compliance_reviewer` full read, `tenant_admin` own-tenant read, `form_system` INSERT, `form_api` zero access); four-step GDPR Art. 17 erasure path with `gdpr.erasure_role_history_pseudonymised` DEC-030 event; 7-year retention proposal (OQ-SSO-28.2); SOC 2 CC6.3/CC6.6/PI1.1 mapping; five-item checklist; two open questions (OQ-TURH-01 changed_by ENUM P1; OQ-TURH-02 role column type alignment P1).
- `docs/DECISION_LOG.md DEC-048` — OQ-SSO-27.3 resolution: SCIM KV failure option (c) adopted; AL-REVOKE-01 added; DPA G-013 dual-path revocation language specified.
- `docs/DECISION_LOG.md DEC-049` — OQ-SSO-27.2 resolution: `tenant_users_role_history` table adopted for SCIM role change audit trail; role values excluded from DEC-030 chain to prevent cross-tenant leakage in auditor exports.

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md §27.15` — OQ-SSO-27.2 and OQ-SSO-27.3 status updated to 🟢 Resolved (DEC-049 and DEC-048 respectively). TOC updated to add §28.
- `VERSION` — 4.72.0 → 4.72.1.

## [4.72.0] — 2026-06-14

### Added
- `content/post-683-proprioception-strength-training.md` — Biomechanics block, post 33/50. Mechanoreceptors (muscle spindles, Golgi tendon organs, joint receptors), joint position sense, proprioceptive training effect, technique preservation under fatigue.
- `content/post-684-muscle-synergies-coordination.md` — Biomechanics block, post 34/50. Motor synergies (Bernstein's DOF problem), CNS coordination strategies in compound lifts, inter-muscular vs. intra-muscular coordination, synergy refinement through practice.
- `content/post-685-elbow-mechanics-pulling.md` — Biomechanics block, post 35/50. Humeroulnar and radioulnar joints, carrying angle variability, pronation/supination in rows and pull-ups, flexor recruitment patterns, grip width downstream effects.
- `content/post-686-foot-arch-mechanics-lifting.md` — Biomechanics block, post 36/50. Three foot arches, windlass mechanism, pronation/supination under load, force transmission to knee and hip, footwear implications for squat and deadlift.
- `content/post-687-rotator-cuff-glenohumeral-centering.md` — Biomechanics block, post 37/50. Rotator cuff anatomy, force couple concept, humeral head centering against deltoid's superior pull, compression vs. shear loads, practical warm-up and activation for pressing/overhead.
- `content/post-688-thoracolumbar-fascia-force-transfer.md` — Biomechanics block, post 38/50. TLF layers and anatomy, passive tension transfer from lats and glutes, role in spinal stiffening, implications for deadlift and bent-over row mechanics.
- `content/post-689-rib-cage-mechanics-pressing.md` — Biomechanics block, post 39/50. Costovertebral joints, rib expansion in breathing, rib flare and lumbar position, arch vs. flat back in bench press, thoracic mobility for overhead press.
- `content/post-690-grip-mechanics-force-transfer.md` — Biomechanics block, post 40/50. Carpal anatomy and wrist alignment under axial load, grip type comparisons (hook/double-overhand/mixed), grip width downstream effects, knurling and wrist wrap mechanics.
- `content/post-691-intra-abdominal-pressure-mechanisms.md` — Biomechanics block, post 41/50. IAP four-element system, hydraulic cylinder mechanism, IAP magnitudes in heavy lifting, Valsalva vs. bracing, belt mechanics (+20–40% IAP), practical protocols by rep range.
- `content/post-692-fatigue-compensation-patterns.md` — Biomechanics block, post 42/50. Neuromuscular vs. metabolic fatigue effects on technique, proximal-to-distal compensation patterns in squat/deadlift/press, technique degradation as diagnostic signal, set termination criteria.
- `content/post-693-load-tissue-capacity-model.md` — Biomechanics block, post 43/50. Tissue capacity as dynamic variable, cumulative load model, ACWR with numeric risk zones, spike vs. absolute load risk, recovery as capacity restoration, 5–10%/week ramp rate.
- `content/post-694-movement-variability-motor-control.md` — Biomechanics block, post 44/50. Bernstein's motor variability, "repetition without repetition," cycle-to-cycle kinematic variability as healthy motor control, uncontrolled manifold hypothesis, micro-variation practice for connective tissue stress distribution.
- `content/post-695-reactive-strength-plyometrics.md` — Biomechanics block, post 45/50. RSI, slow vs. fast SSC (250 ms threshold), tendon elastic energy storage, why strength athletes benefit from plyometric exposure, dosing table and 12-week entry sequence.
- `content/post-696-asymmetry-when-to-address.md` — Biomechanics block, post 46/50. Asymmetry prevalence and taxonomy (structural/functional/acquired), LSI thresholds, five functional field tests, decision tree (observe → assess → address or accept), unilateral training as diagnostic tool.
- `content/post-697-eccentric-mechanics-structural.md` — Biomechanics block, post 47/50. Force-velocity and force-length in eccentric, cross-bridge detachment mechanics, sarcomerogenesis, titin adaptations, collagen remodeling, HSR tempo prescriptions, accentuated eccentrics at 105–120% 1RM.
- `content/post-698-tissue-stiffness-movement-patterns.md` — Biomechanics block, post 48/50. Young's modulus, passive vs. active stiffness, pre-activation as active stiffness mechanism, joint stiffness continuum, consequences of too-soft/too-stiff states, programming balance for mobility and stability.
- `content/post-699-cervical-spine-head-mechanics.md` — Biomechanics block, post 49/50. Cervical compressive load, tonic neck reflex, head-spine linkage, neutral head position per lift, gaze direction and motor pattern effects, cueing table, boundary between self-directed and clinical referral.
- `content/post-700-biomechanics-block-synthesis.md` — Biomechanics block, post 50/50. **BLOCK COMPLETE.** Eight synthesizing principles: joint stack/force chain, anatomy as constraint, structural lag timeline, load-vs-capacity model, movement variability, principle-to-programming decision table, field self-assessment toolkit, clinical referral boundary.

### Changed
- `STATUS.md` — block 651–700 marked COMPLETE (50/50 posts); total posts updated to 700; version bumped to v4.72.0.
- `VERSION` — 4.71.1 → 4.72.0.

## [4.71.1] — 2026-06-14

### Changed
- `docs/CRYPTOGRAPHY_POLICY.md` v1.2 — `SCIM_TOKEN_HASH_SECRET` registered: §5 Key Rotation Schedule (180-day cycle; rotation requires re-hashing all active `tenant_scim_tokens.token_hash` rows during maintenance window), §6 Key Custody (Cloudflare Workers Secret scoped to `scim-worker` only; 1Password Operations vault backup), §12 Document History updated. Closes SSO_SCIM_IMPLEMENTATION §27.14 checklist item 3 (P0 M4). Cross-ref: SSO_SCIM_IMPLEMENTATION §27.3 (HMAC-SHA256 bearer-token auth pipeline); DATA_MODEL §4.2 (`tenant_scim_tokens.token_hash`).
- `docs/SOC2_READINESS.md` — §56.3 Key Inventory: `SCIM_TOKEN_HASH_SECRET` row added (HMAC-SHA256; Cloudflare Workers Secret `scim-worker`; 180-day rotation; security-engineer owner). Nine-key inventory now covers all production secrets including both HMAC keying secrets (`API_KEY_HASH_SECRET` and `SCIM_TOKEN_HASH_SECRET`).
- `docs/SSO_SCIM_IMPLEMENTATION.md` — §27.14 checklist item 3 marked `[x]` ✅ 2026-06-14.

## [4.71.0] — 2026-06-14

### Added
- `content/post-682-hip-anatomy-variability-squat.md` — Biomechanics block, post 32/50. Hip joint anatomy variability and its effect on squat mechanics: acetabular version (anteversion/retroversion), femoral neck angle (coxa valga/vara), femoral version (anteversion/retroversion). Distinguishes soft-tissue restriction (responds to mobility work) from bony impingement (requires technique adaptation). Butt wink as anatomical vs. mobility phenomenon. Functional assessment without imaging (passive hip rotation test, natural stance test). Practical implications for squat stance, deadlift variant, and unilateral exercises. Brief FAI scope note: normal anatomical variation ≠ clinical FAI. clinical-safety PASS (anatomy education only; explicit disclaimer that symptomatic hip pain requires medical evaluation; no injury triage). sports-scientist review pending.
- `README.md` — Block 1051–1100 «М'язова специфіка та функціональна анатомія» added to content roadmap: 10 topics covering individual muscle group anatomy, moment arms, multi-joint behavior, programming implications. Glutes, hamstrings, quads, lats, pec major, deltoids, rotator cuff, traps, deep spinal stabilizers, calves.

## [4.70.0] — 2026-06-14

### Added
- `content/post-659-training-discomfort-vs-pain-neurobiology.md` — Biomechanics block, post 27/50. Neurobiology of training sensations: nociception vs. proprioception, DOMS (peripheral sensitization via inflammatory mediators, Z-disc microtrauma), metabolic burn (Group III/IV metaboreceptors), central pain modulation (DNIC, endogenous opioids, expectation effects). Educational framing only — no symptom triage, no "train through it" guidance. clinical-safety CONDITIONAL PASS (disclaimer required, no decision tree, no injury identification by sensation). sports-scientist review pending.
- `content/post-660-return-after-break-load-progression.md` — Biomechanics block, post 28/50. Return-to-training principles after lifestyle breaks (travel, schedule — not illness/injury). Detraining timeline asymmetry: cardiorespiratory (VO₂max drops 10–14 days) vs. neuromuscular (patterns persist weeks) vs. connective tissue (slowest). Nuclear and neural muscle memory mechanisms. Conservative load return orientations (70–80% post 2–6wk break). Explicit scope: healthy athletes, lifestyle break only. clinical-safety CONDITIONAL PASS (illness/injury explicitly scoped out, medical clearance disclaimer). sports-scientist review pending.
- `content/post-679-tendon-adaptation-collagen.md` — Biomechanics block, post 29/50. Tendon adaptation: collagen type I structure, mechanosensitivity of tenocytes (3–8% strain window), collagen synthesis timeline (peak 24–72h, full remodeling 6–12+ weeks per Kjaer/Magnusson isotope research). Stiffness adaptation vs. hypertrophy. Load-rate mismatch: muscle adapts faster than connective tissue — core periodization implication. Eccentric loading and collagen synthesis. clinical-safety PASS. sports-scientist review pending.
- `content/post-680-stretch-shortening-cycle.md` — Biomechanics block, post 30/50. SSC (Stretch-Shortening Cycle) mechanics: three mechanisms (elastic energy storage, stretch reflex via Ia afferents, titin-mediated potentiation). Short SSC (<250ms) vs. long SSC (>250ms) and stiffness optimum. Touch-and-go vs. paused lifting as SSC manipulation tool. Controlled vs. uncontrolled rebound and knee joint loading implications. clinical-safety PASS. sports-scientist review pending.
- `content/post-681-rate-of-force-development.md` — Biomechanics block, post 31/50. RFD (Rate of Force Development, ΔF/Δt in N/s): early RFD 0–50ms (doublet discharges, motor unit synchronization, descending neural drive) vs. late RFD (maximal strength, CSA, tendon stiffness). Relevance for strength sports: dead stop lifts, sticking point dynamics. Training methods: maximal intent at all intensities, dynamic effort (55–80% 1RM), plyometrics, near-maximal loads. Tendon stiffness as RFD transmission amplifier. clinical-safety PASS. sports-scientist review pending.

## [4.69.0] — 2026-06-13

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md` v1.9 — §27 SCIM v2.0 Endpoint Worker Implementation Design. Closes G-001 (Critical gap: 🔴 Not designed → 🟡 Design complete, implementation pending M4/M5). Full Cloudflare Worker architecture (`apps/scim-worker/`): six-file source layout, six bindings (SCIM_KV separate from SSO_KV, SCIM_TOKEN_HASH_SECRET, AUDIT_QUEUE, IP_HASH_SALT). Token authentication pipeline: HMAC-SHA256 hash → 5-minute KV cache → Supabase `tenant_scim_tokens` fallback; `ScimTokenCache` type; `enforceScimIpAllowlist()` shared from §25. Request routing: fourteen route patterns for Users + Groups + schema discovery. User handlers: POST with idempotency gate (201 create / 200 reactivate / 200 idempotent-replay via `(tenant_id, external_user_id)` UNIQUE index), GET by id, GET list with `eq` filter, PUT full replace, PATCH with deactivation → §22 KV revocation, DELETE soft-delete alias. Group handlers: POST mapping creation, PATCH with role re-evaluation via §19.3 view (higher-privilege-wins DEC-039), DELETE with role downgrade cascade; handles both FORM-UUID and `externalId` member value formats. Schema discovery: ServiceProviderConfig (bulk false, changePassword false), Schemas, ResourceTypes. RFC 7644 error format helper (seven scimType values; no tenant internals in detail strings). Idempotency contract: `(tenant_id, external_user_id)` UNIQUE as primary key; email fallback for legacy IdPs; `scim_request_id` dedup at emit-audit-event. Six DEC-030 HMAC-chained events: `scim.user_provisioned` (HIGH, 7yr), `scim.user_reactivated` (HIGH, 7yr), `scim.user_deprovisioned` (HIGH, 7yr — canonical name, replaces deprecated `scim.user_deactivated`), `scim.user_updated` (STANDARD, 7yr), `scim.group_synced` (STANDARD, 5yr), `scim.rejected_sensitive_attribute` (HIGH, 7yr). SCIM-CHAIN-01 ordering invariant. Four alerting rules AL-SCIM-01 through AL-SCIM-04. SOC 2 evidence mapping CC6.1/CC6.3/CC6.4/CC6.6/CC7.2/PI1.1; four artefacts SCIM-PROV-E-001 through SCIM-PROV-E-004. Seventeen-item implementation checklist (10× P0/M4–M5, 4× P1/M5–M6, 3× P2/M5–M12). Three open questions (OQ-SSO-27.1 SCIM-without-SSO P2; OQ-SSO-27.2 role-change audit diffing P1; OQ-SSO-27.3 async revocation KV failure mode P1). §9 G-001 row updated to 🟡 Authored. ToC updated.
- `docs/AUDIT_LOG_SCHEMA.md` — §SCIM-Lifecycle: six SCIM provisioning lifecycle events formally registered with Zod v2 schemas (`ScimUserProvisionedSchema`, `ScimUserReactivatedSchema`, `ScimUserDeprovisionedSchema`, `ScimUserUpdatedSchema`, `ScimGroupSyncedSchema`, `ScimRejectedSensitiveAttributeSchema`). `scim.user_deactivated` deprecated in favour of `scim.user_deprovisioned`. Retention table: 7yr for identity lifecycle events; 5yr for group sync. Privacy invariant: no name/email/health data in any event; `external_user_id` opaque IdP stable ID. Closes SSO_SCIM_IMPLEMENTATION.md §27.14 checklist item 1 (P0 M4).

## [4.68.0] — 2026-06-13

### Added
- `content/post-678-compression-vs-shear-spine-mechanics.md` — Biomechanics block, post 26/50. Compression vs. shear force mechanics at the spine: axial compression (vertebral tolerance >14 000 N per McGill lab data) vs. shear (lower threshold, ligament/disc risk). Trunk angle as primary determinant of shear-to-compression ratio. IAP (intra-abdominal pressure) as hydraulic stabilizer reducing lumbar compression by ~1 000–1 400 N. Spinal adaptation to loading (bone density, disc hydration cycle). Three evidence-based risk factors: shear under fatigue, dynamic flexion under load, chronic overload without recovery. clinical-safety PASS. sports-scientist review pending.
- `blog.html` — added card for post-678.
- `STATUS.md` — block 651–700 updated to 26/50; total posts 676; next priorities updated.

## [4.67.0] — 2026-06-13

### Added
- `content/post-670-knee-valgus-mechanics.md` — Biomechanics block, post 20/50. Dynamic valgus: three-component anatomical definition (femoral internal rotation + foot pronation + hip adduction), compensatory adaptation vs. technical fault differentiation, glute medius and abductor EMG data (Distefano 2009, Boren 2011), limitations of frontal-plane video analysis, muscle stimulus redistribution under valgus. clinical-safety PASS. sports-scientist review pending.
- `content/post-671-pelvic-tilt-compound-lifts.md` — Biomechanics block, post 21/50. Anterior vs. posterior pelvic tilt mechanics in compound lifts: lumbopelvic rhythm as a dynamic cycle, APT in squat (when adaptive vs. compensatory), PPT in deadlift (early loss of posterior chain efficiency), practical hip hinge test with dowel rod. clinical-safety PASS. sports-scientist review pending.
- `content/post-672-hamstring-mechanics.md` — Biomechanics block, post 22/50. Biarticular hamstring mechanics: active and passive insufficiency, length-tension relationship at long muscle lengths, why nordic curl is hardest at start (Herzog/titin eccentric force), RDL vs. lying leg curl as different vectors (proximal vs. distal hypertrophy, Bourne et al. 2017), deadlift vs. RDL distinction. clinical-safety PASS. sports-scientist review pending.
- `content/post-673-quad-dominance-posterior-chain.md` — Biomechanics block, post 23/50. Quad dominance as relative load distribution pattern, not absolute weakness: trunk lean angle and hip moment arm mechanics, when it is adaptive (powerlifter low-bar squat) vs. limiting, single-leg squat and single-leg RDL as diagnostic tests, vector-based programming solutions over isolation additions. clinical-safety PASS. sports-scientist review pending.
- `content/post-674-glute-medius-lateral-stability.md` — Biomechanics block, post 24/50. Gluteus medius anatomy and three roles (abduction, medial rotation, frontal-plane pelvic stabilisation), Trendelenburg as extreme manifestation, glute medius activity in bilateral squat and deadlift (EMG data Bolgla & Uhl 2005; Boudreau 2009), isolating vs. functional exercises (Distefano 2009 EMG comparison), practical single-leg stance and lateral band walk assessment. clinical-safety PASS. sports-scientist review pending.
- `content/post-675-cervical-spine-loading.md` — Biomechanics block, post 25/50. Cervical spine mechanics under load: cranio-cervical chain effect on dura mater tension, why "look up" in deadlift is mechanically questionable (C4–C7 compression, McGill 2010; Hales 2010), neutral cervical vs. packed neck cue differentiation, head position in high-bar vs. low-bar squat, overhead press "head through" mechanics. clinical-safety PASS. sports-scientist review pending.
- `content/post-676-bilateral-deficit.md` — Biomechanics block, post 26/50. Bilateral deficit: definition and measurement, interhemispheric inhibition mechanism (Howard & Enoka 1991), BLD magnitude by training level (Vandervoort, Secher), when BLD is sport-relevant vs. irrelevant (powerlifting specificity), unilateral training as complementary neural adaptation, 10% inter-limb asymmetry threshold. clinical-safety PASS. sports-scientist review pending.
- `content/post-677-neural-vs-metabolic-fatigue.md` — Biomechanics block, post 27/50. Two fatigue mechanisms: neural (central drive reduction, Gandevia 2001; Bigland-Ritchie) vs. metabolic (Pi, H+ accumulation, glycogen depletion), different recovery timescales, why heavy singles after high-rep volume perform sub-maximally despite metabolic recovery, intra-set vs. inter-session fatigue as distinct phenomena. clinical-safety PASS. sports-scientist review pending.
- `STATUS.md` — content table updated: block 651–700 count 17→25; total posts 667→675; next priorities updated.

---

## [4.66.1] — 2026-06-13

### Fixed
- `enterprise.html` — Tier seat ranges corrected to match calculator logic: Starter 50–199, Growth 200–999, Enterprise 1,000+. Previously "200–1,000" for Growth caused a $7,200/year discrepancy for 200-seat prospects (marketing showed Growth pricing, calculator charged Starter).
- `pricing-enterprise.html` — Calculator tier boundaries aligned with `docs/ENTERPRISE.md` (Growth 200–1,000; Enterprise 1,000+). `TIERS.starter.max` 200→199, `TIERS.growth.min` 201→200, `TIERS.growth.max` 1000→999, `TIERS.enterprise.min` 1001→1000. `REQ_MINS.csm` 201→200, `REQ_MINS.soc2` 1001→1000. Feature comparison table headers and Enterprise custom-pricing note updated to reflect 1,000+ boundary.
- `VERSION` — 4.66.0 → 4.66.1.

---

## [4.66.0] — 2026-06-13

### Added
- `content/post-669-scapular-mechanics-pressing-pulling.md` — Biomechanics block post 17/50. Scapular mechanics in pressing and pulling: scapula as a floating bone with no synovial joint to the thorax; six scapular movement vectors; bench press mechanics — retraction/depression at setup, natural protraction at lockout (why "keep shoulder blades pinched throughout" is a partial cue); overhead press — upward rotation required at lockout, the "shoulders away from ears" cue as context-dependent; horizontal row — full protraction at start required for full ROM, retraction + depression at peak (without elevation); vertical pull — scapular depression as first movement, not elbow drive; serratus anterior as an undertrained stabilizer; scapular push-up and wall slide as 2-minute pre-session activation. Three common errors invisible in a mirror. Clinical safety: pure biomechanics/mechanics, no pain/rehab content. sports-scientist review required before publish.
- `README.md` — Block 1001–1050 "Training literacy: reading your own data" added to roadmap. Ten proposed topics covering the meta-skill of pattern recognition in self-coaching data.

### Changed
- `blog.html` — post-669 card added (Biomechanics · 13 хв · draft).
- `STATUS.md` — block 651–700 row updated 16 → 17 posts, range note updated to 661–669; total posts 666 → 667; next priorities updated (670+; Block 1001–1050 proposed).
- `VERSION` — 4.65.3 → 4.66.0.

---

## [4.65.3] — 2026-06-13

### Added
- `content/post-667-lumbar-spine-under-load.md` — Biomechanics block post 17/50. Lumbar spine mechanics under load: neutral spine as a range not a fixed angle; butt wink dissected (femoral-acetabular impingement as structural constraint, ankle dorsiflexion as upstream cause); flexion-relaxation phenomenon at end-range; moment arm analysis comparing low-bar squat, high-bar squat, conventional and sumo deadlift; IAP — 360-degree bracing vs hollowing, Valsalva at 85%+; practical pre-lift checklist. Clinical safety: mechanics-only, no pain/rehab content. Sports-scientist review required before publish.
- `content/post-668-thoracic-spine-strength-training.md` — Biomechanics block post 18/50. Thoracic spine mechanics in strength training: rib cage as mobility limiter, 20–25° total extension range; how T-spine deficits manifest in overhead press (compensatory lumbar extension), bench press (scapular protraction), and rows (premature anterior tilt limits lat ROM); scapular mechanics — posterior tilt requires T-spine extension underneath it; postural vs structural kyphosis (passive extension test as screen); "chest up" cue explained biomechanically; foam roller + wall angel warm-up integration. Clinical safety: mechanics-only, no pain/rehab content. Sports-scientist review required before publish.
- `compliance/dpa/dpa-collection-checklist-posthog-2026-06-13.md` — PostHog DPA scope review (DEC-046 pre-condition #1 — compliance-officer). Full scope analysis: `readiness_bucket` confirmed within PostHog product analytics scope; Art. 9 health data analysis per DEC-046 clinical-safety ruling (three-bucket collapse not Art. 9 under EDPB guidance 03/2020); EU adequacy confirmed for EU Cloud Frankfurt (no SCC Module 2 required); mood_bucket/mood_score/readiness_score raw permanently prohibited. Click-through execution pending (founder action — PostHog Dashboard → Organization → Privacy & DPA). Three DEC-046 pre-conditions tabulated: review complete (#1), platform-engineer SDK allowlist (#2 pending), CI lint rule (#3 pending).

### Changed
- `STATUS.md` — block 651–700 row updated 14 → 16 posts; total posts 664 → 666; next priorities updated (newsletter-05 READY TO SEND; PostHog DPA review COMPLETE → click-through pending; Block 651–700 next: 669+).
- `VERSION` — 4.65.2 → 4.65.3.

---

## [4.65.2] — 2026-06-13

### Changed
- `content/newsletter-05.md` — brand-voice editorial pass complete. `tone_check` updated from `pending` to `complete`, status updated to `approved`. Editorial-brutalist tone confirmed throughout; PPG/ECG accuracy framing on-brand; closing line ("Your device is a useful tool. It's not the coach.") confirmed in register; Victor's voice consistent with issues 01–04. Placeholder `[посилання на підписку]` flagged — must be resolved before send. READY TO SEND pending subscription link substitution.

---

## [4.65.1] — 2026-06-13

### Changed
- `admin-dashboard.html` — Added DSAR status widget to Compliance screen. Closes DATA_MODEL.md §31 checklist item 10 (P1 M5). Widget shows: 4 summary stats (open requests, overdue count, closed-last-30-days, GDPR 30-day deadline clock); open-requests table (request ID, GDPR basis, status, submitted, deadline, days-remaining — colour-coded amber ≥14d / acid ≥21d); closed-last-30-days table with SLO outcome; privacy-floor footer noting excluded fields (`user_id`, `export_url`, `tables_erased`, `deletion_certificate_ref`) and Postgres RLS enforcement (`form_tenant_admin` policy per DATA_MODEL.md §31.4). Alert references: AL-DSAR-01 (P1, day-25 warn), AL-DSAR-02 (P0, overdue), AL-DSAR-04 (P1, DEC-032-EXT chain violation). HR admin cannot identify which employee filed a request — enforced at DB layer, not application code.
- `VERSION` — 4.65.0 → 4.65.1.

---

## [4.65.0] — 2026-06-13

### Added
- `content/post-665-hip-hinge-mechanics.md` — Biomechanics block post 15/50. Hip hinge as a distinct movement pattern from squat: anatom of tazo-femoral joint sharn, why most athletes cannot hinge (hamstring neural restriction, absent motor pattern, hip capsular limits). Wall drill, RDL, KB swing, good morning as teaching progression. Common faults: spine flexion instead of hinge, overdepth, compensatory hyperextension, locked knees. Posterior chain loading through lengthened position as hypertrophic stimulus. Programming placement. sports-scientist review required before publish.
- `content/post-666-wrist-elbow-pressing-mechanics.md` — Biomechanics block post 16/50. Wrist and elbow mechanics in pressing movements: neutral wrist position and bar placement in palm vs. fingers; grip width effects on elbow flare, ROM, and muscle recruitment; elbow angle range 30–60° rationale; overhead press rack position and stacking principle; wrist wraps — when they help vs. when they mask a technique problem; rowing elbow as driver vs. passive hook. Clinical safety: technique/mechanics only, no pain treatment content. sports-scientist review required before publish.
- `STATUS.md` — content table updated: 651–700 row updated to 14 posts (665–666 added); total posts 662 → 664; next priorities updated to 667+.

---

## [4.64.1] — 2026-06-13

### Fixed
- `docs/SOC2_READINESS.md §78` — DEC cross-reference correction: three occurrences of `DEC-031` in §78.2 (recommendation paragraph), §78.9 (gap tracker), and §78.10 checklist item 1 corrected to `DEC-047`. DEC-031 is "Agent team expanded from 14 → 24 agents"; the Vanta tool-selection decision is DEC-047 (2026-06-13). Checklist item 1 status updated `[ ]` → `[x]` — DEC-047 is now logged in DECISION_LOG.md. Version note v3.7.3 appended.

### Changed
- `VERSION` — 4.64.0 → 4.64.1.

## [4.64.0] — 2026-06-13

### Added
- `content/post-664-shoulder-complex-overhead-mechanics.md` — Block 651–700 post 664. Плечовий комплекс: механіка overhead руху і програмування для здоров'я плеча. Три суглоби плечового комплексу (GH, AC, scapulothoracic). Механіка лопатки: upward rotation, posterior tilt, зовнішня ротація як умова для subacromial space при підйомі руки. Scapulohumeral rhythm 2:1. Три причини порушеної лопаткової механіки: надмірна протракція, слабкість serratus anterior, обмеження торакального відділу. Push-pull ratio і відмінність горизонтальних vs вертикальних тяг. Зовнішньоротаційна робота як структурна частина програми. Overhead pressing: коли замінювати на neutral-grip або landmine. Торакальна рухливість як попередня умова. clinical_safety: PASS. sports_scientist_review: required before publish.

### Changed
- `blog.html` — додано картку post-664.
- `STATUS.md` — Block 651–700: 12 постів написано (664 додано); total posts 662.
- `VERSION` — 4.63.0 → 4.64.0.

## [4.63.0] — 2026-06-13

### Added
- `content/post-662-bar-path-mechanics.md` — Block 651–700 post 662. Траєкторія грифа: що насправді означає «вертикально». Чому траєкторія грифа у присіданні і становій тязі є кривою (J/S-крива у присіді, зворотна дуга у тязі), а не прямою лінією. Принцип «гриф над серединою стопи» як механічний орієнтир. Moment arm: як відхилення від оптимальної траєкторії збільшує вимогу до м'язів і навантаження на хребет. Методи самооцінки: відео збоку, крейда на грифі. clinical-safety: cleared. sports-scientist: cleared.
- `content/post-663-force-velocity-curve.md` — Block 651–700 post 663. Крива сила-швидкість: чому важко — це не все. Обернена залежність сили і швидкості скорочення м'язів. Три зони кривої: максимальна сила (повільно, >85% 1ПМ), гіпертрофія (помірно, 60–80%), потужність/швидкість (вибухово, <60%). Чому тренінг виключно з важкими вагами залишає нейром'язовий спектр неповним. Compensatory acceleration training (CAT) за Хатфілдом. Velocity-based training (VBT) — принцип без обладнання. clinical-safety: cleared. sports-scientist: cleared.

### Changed
- `STATUS.md` — Block 651–700: 11 постів написано (662, 663 додано); total posts 661.
- `VERSION` — 4.62.1 → 4.63.0.

## [4.62.1] — 2026-06-13

### Added
- `docs/OBSERVABILITY.md §37.12` — Enterprise Employer-Side DSAR Observability Extension. Closes the observability gap created by DATA_MODEL §32 (v4.60.0): five new employer-side DSAR events had no §37 monitoring coverage. New content: GDPR-SLO-06 (employer Art. 15 provision ≤ 24h, 100% per-event, ENTERPRISE_SLA §19.5 contractual); five RED metrics for employer DSAR pipeline (aggregate counts only — no user_id, no health data); three alert rules — AL-GDPR-07 (P2, offboarding export gap > 1h), AL-DSAR-04 (P1, DEC-032-EXT chain violation — deletion_confirmed without preceding deletion_soft), AL-DSAR-05 (P1, employer DSAR SLO breach, graduated response: first miss per tenant per quarter P1 PagerDuty, subsequent CSM Slack only); three monitoring queries (employer SLO status every 15 min, offboarding export gap every 10 min, DEC-032-EXT chain violation every 30 min via HMAC Worker); three SOC 2 evidence artefacts DSAR-E-011 through DSAR-E-013 (P5.0/A1.1, CC7.2/CC7.3, CC7.2); eight-item implementation checklist. Closes DATA_MODEL §32.8 OQ-DSAR-03 (P1 — how to handle dsar.data_provided SLO breach): AL-DSAR-05 graduated response is the decision.

### Fixed
- `docs/DATA_MODEL.md §32.5` (v1.12) — alert ID naming conflict: DATA_MODEL §32 (v1.11) incorrectly assigned the offboarding export gap alert `AL-GDPR-04`, which was already defined in OBSERVABILITY §37.5 as workout data purge job 26 staleness (P1 PagerDuty). Corrected to **AL-GDPR-07** across §32.4 chain note, §32.5 alert table (row renamed + correction note added), and §32.7 checklist item 5.

### Changed
- `docs/OBSERVABILITY.md` — v3.5 → v3.9 (header sync to match v3.8 footnote + §37.12 v3.9 addition).
- `docs/DATA_MODEL.md` — v1.11 → v1.12.
- `VERSION` — 4.62.0 → 4.62.1.

## [4.62.0] — 2026-06-13

### Added
- `content/post-661-lateral-asymmetry-strength-training.md` — Block 651–700 post 661. Симетрія і асиметрія в силовому тренінгу: типи латеральної асиметрії (силова, мобільна, нейром'язова), нормативний діапазон (10–15% при тестах нижніх кінцівок — нормальна варіабельність; >15% — поріг уваги за Maloney 2019, Bishop 2018). Limb Symmetry Index (LSI): wall ankle test для мобільної асиметрії, single-leg hop protocol, унілатеральний силовий тест. Три сценарії проблемної асиметрії: >15% без травми, після травми (LSI <90%), прогресуюча без причини. Логіка корекції: починати з мобільної причини; унілатеральний обсяг зі слабшої сторони; горизонт 3–6 місяців. clinical-safety: PASS. sports_scientist_review: required before publish.
- `README.md` — Block 951–1000 «Training environment & context» додано до роадмапу: 10 тем про тренінг поза ідеальними умовами (відрядження, домашній тренінг, середовищні фактори, нефіксований графік).
- `STATUS.md` — оновлено рядок Block 651–700: 9 постів написано (661 додано); total posts 659.

## [4.61.0] — 2026-06-13

### Added
- `content/post-656-deadlift-mechanics.md` — Block 651–700 post 656. Біомеханіка тяги: тяга як розгинання в кульшовому і колінному суглобах (не «підйом спиною»). Стартова позиція — гриф над серединою стопи, лопатки над грифом, стегна нижче плечей. «Відштовхнути підлогу», а не «тягнути штангу». Вертикальна траєкторія як ідеал. Роль широчайніх як прихованого стабілізатора. Conventional vs Sumo: механічні відмінності і анатомічна доцільність. Lockout через кульшовий суглоб. Три помилки з найбільшим ризиком: стегна першими, штанга від тіла, флексія хребта. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-657-core-stability-lifting.md` — Block 651–700 post 657. Стабільність кору: чотири компоненти (TVA, мультифідуси, діафрагма, тазове дно), IAP як головний захисний механізм хребта (150–200 mmHg при максимальних зусиллях). Bracing vs drawing-in — принципова різниця. Нейтральний хребет: рівномірний розподіл навантаження на диски. Три патерни розпаду стабілізації: флексія поперека, «розсипання» на останніх повтореннях, грудний колапс вперед. Антифлексійні, антиротаційні, антилатерофлексійні вправи і carry як ефективні інструменти. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-658-breathing-bracing-lifting.md` — Block 651–700 post 658. Дихання і bracing при підйомах: механіка modified Valsalva, IAP при трьох дихальних стратегіях (затримка, видих при підйомі, затримка до lockout). Коли виправданий Valsalva (≥80% 1ПМ, compound рухи), коли прийнятний видих при підйомі (< 60%, ізоляція, початківці). Серцево-судинні наслідки — факти без паніки. Покрокова механіка правильного bracing. Дихання між повтореннями: варіанти А і Б. Дві типових помилки: грудне дихання, brace після початку руху. clinical-safety: PASS. sports_scientist_review: required before publish.

### Changed
- `STATUS.md` — Block 651–700: 5 → 8 posts written (656–658 added). Total posts: 655 → 658. Next: 659–660 (обидва потребують clinical-safety review перед написанням). Версія: v4.60.0 → v4.61.0.
- `VERSION` — 4.60.0 → 4.61.0.

---

## [4.60.0] — 2026-06-13

### Added
- `docs/DATA_MODEL.md §32` — Enterprise DSAR Extensions: employer-side data lifecycle & offboarding export schema. Closes DEC-030 event gap created when `INCIDENT_RESPONSE.md` v2.3 registered five enterprise DSAR events (`dsar.data_provided`, `dsar.deletion_soft`, `dsar.deletion_confirmed`, `dsar.portability_export_completed`, `dsar.offboarding_export_available`) from `ENTERPRISE_SLA.md §19.5` without a corresponding DATA_MODEL.md schema section. New section covers: event registration table + Zod v2 schemas for all five events; DEC-032-EXT two-phase erasure chain ordering constraint; §25 offboarding chain interaction (1-hour SLO for export-available notification); two alert rules (AL-GDPR-04, AL-DSAR-04); four SOC 2 evidence artefacts DSAR-E-007–010 (P5.0, P8.0, P5.1, A1.1); eight-item implementation checklist; two open questions (OQ-DSAR-03, OQ-DSAR-04).

### Changed
- `docs/DATA_MODEL.md` — v1.10 → v1.11. TOC updated with §32 entry.
- `VERSION` — 4.59.0 → 4.60.0.

---

## [4.59.0] — 2026-06-13

### Added
- `content/post-655-bench-press-biomechanics.md` — Block 651–700 post 655. Біомеханіка жиму лежачи: плечовий суглоб як лімітуючий фактор (glenohumeral instability механізм), ширина хвату і абдукція плеча (45–65°), кут ліктів і вертикальність передпліч, реальна траєкторія штанги (дуга, не вертикаль), лопатка в ретракції і депресії як фундамент, latissimus dorsi як прихований стабілізатор, арка як механізм стабілізації (не шахрайство), leg drive і кінетична ланцюг від підлоги до плеча, нейтральне зап'ястя, фази сили (стернальна голівка → трицепс у локауті), три «тихих» патерни травм. clinical-safety: PASS. sports_scientist_review: required before publish.

### Changed
- `STATUS.md` — Block 651–700: 4 → 5 posts written (post-655 added). Total posts: 654 → 655. Next: 656–660. Версія: v4.58.0 → v4.59.0.
- `VERSION` — 4.58.0 → 4.59.0.

---

## [4.58.0] — 2026-06-13

### Added
- `content/post-652-mobility-vs-stability-joint-by-joint.md` — Block 651–700 post 652. Мобільність vs. стабільність: joint-by-joint підхід (Cook/Boyle). Чергування мобільних і стабільних суглобів у кінетичному ланцюгу. Різниця між пасивною гнучкістю і активною мобільністю. Різниця між жорсткістю і стабільністю. Механізм компенсації: як дефіцит мобільності «мобільного» суглобу руйнує сусідній «стабільний» (гомілкостоп→коліно, кульшовий→поперек, грудний→лопатка). Обмеження евристики. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-653-ankle-mobility-squat.md` — Block 651–700 post 653. Мобільність гомілкостопу і присідання: дорсифлексія як лімітуючий фактор. Норма 15–20° (до 38° для глибокого присідання). М'якотканинне vs. кісткове обмеження — практична різниця. Knee-to-wall тест з числовими орієнтирами (< 9 см — обмеження). Три компенсаційні механізми: підйом п'яти, надмірний нахил тулуба, вальгус коліна. Робота: soleus-специфічна розтяжка, навантажена мобільність (eccentric heel drop, knee-over-toe), banded mobilization. Роль підборної плити як перехідного інструменту. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-654-squat-anatomy-individual-stance.md` — Block 651–700 post 654. Анатомія присідання і варіації стійки: три кісткові фактори — версія стегнової кістки (антеверсія/ретроверсія), глибина і орієнтація вертлюжної западини, довжина стегна відносно тулуба. Практичний тест для визначення індивідуальної стійки. Деміфологізація правила «коліна за носками» (Fry 2003). Механіка butt wink: м'якотканинне vs. структурне обмеження. clinical-safety: PASS. sports_scientist_review: required before publish.

### Changed
- `STATUS.md` — Block 651–700: 1 → 4 posts written (652–654 added). Total posts: 651 → 654. Версія: v4.57.1 → v4.58.0.
- `VERSION` — 4.57.1 → 4.58.0.

---

## [4.57.1] — 2026-06-13

### Changed
- `docs/SOC2_READINESS.md` — v3.7.1 → v3.7.2. §78 added: Continuous Compliance Automation — Vanta Integration Design, Vendor Pre-Activation Review & Observation Period Evidence Bootstrapping (CC4.1/CC4.2/CC2.3). Covers tool selection rationale (Vanta over Drata, DEC-031), integration architecture (6 connectors), `vanta_readonly` Postgres role with explicit REVOKE on all Art. 9 health-data tables, CTOOL-E-001–E-006 evidence artefact series, 4-step DPA pre-activation procedure, control coverage map (7 automated, 2 manual upload), 16-item implementation checklist. Gap tracker: PRE-25 🔴 Open → 🟡 Partial. Readiness advances +0.3 pp on CTOOL-E-001/E-002 filing.
- `docs/AUDIT_LOG_SCHEMA.md` — v2.4 → v2.5. `system.compliance_tool_connected` event registered (STANDARD, 3yr, HMAC-chained). DPA-first compliance tooling activation record. Privacy invariant: no user_id, no health data. Zod v2 schema in SOC2_READINESS §78.7; filing path CTOOL-E-006. Retention table +1 row.
- `VERSION` — 4.57.0 → 4.57.1.

---

## [4.57.0] — 2026-06-13

### Added
- `content/post-651-kinetic-chain-foot-knee-hip.md` — Block 651–700 post 651. Кінетичний ланцюг: як пронація стопи або дефіцит дорсифлексії запускає ланцюг внутрішніх ротацій, який читається у коліні (вальгус) і стегні (внутрішня ротація стегнової кістки). Механіка замкнутого ланцюга під навантаженням. Дорсифлексія як ключовий параметр для присідання. Практичні наслідки для присідання, станового тязу і монопедальних вправ. Два самооцінювальні тести. Протилежний сценарій (жорстка стопа/супінація). Логіка взуття з клином. Порядок cue-ів: стопа → стегно → коліно. clinical-safety: PASS (техніка і механіка руху; без харчування/образ тіла/лікування болю/психіатрії; ремарка про звернення до фізіотерапевта при актуальному болю). sports_scientist_review: required before publish.

### Changed
- `README.md` — Додано Block 901–950 «Practical athlete intelligence» (proposed) до роадмапу: 10 тем про прийняття рішень у тренінгу без тренера.
- `blog.html` — Додано картку post-651 у верх сітки блогу.
- `STATUS.md` — Block 651–700: 0 → 1 (post-651 written). Total posts: 650 → 651. Версія: v4.55.1 → v4.57.0.
- `VERSION` — 4.56.0 → 4.57.0.

---

## [4.56.0] — 2026-06-13

### Added
- `content/post-647-compound-vs-isolation-order.md` — Block 601–650 post 647. Логіка вибору порядку вправ: compound-first як дефолт для сили і координації; три виправдані сценарії для isolation-first (pre-exhaust для пріоритетної групи, нейром'язова activation, реабілітаційний контекст); ключовий фреймворк: лімітуючий фактор у ключовому русі визначає порядок, а не правило. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-648-return-to-basics-after-specialization.md` — Block 601–650 post 648. Повернення до базових рухів після спеціалізованого блоку: що відбувається із балансом під час спеціалізованого блоку; чому відновлення базових рухів обов'язкове (баланс антагоністів, координаційна схема); практичний протокол тижень-за-тижнем; орієнтир «половина тривалості блоку» для відновлення балансу; типові помилки. clinical-safety: PASS. sports_scientist_review: required before publish.
- `content/post-649-sleep-deprived-athlete-programming.md` — Block 601–650 post 649. Програмування при хронічно скороченому сні через графік (не клінічний розлад). Scope обмежено чітко: змінна робота, дитина, ранні зміни — не медична проблема сну. Фізіологічні наслідки (синтез протеїну, відновлення, суб'єктивне зусилля, координація). Пʼять принципів адаптації програмування: знизити об'єм АБО інтенсивність (не обидва), вища частота/менше за сесію, RPE-based замість відсоткового, складні рухи першими, деload кожні 3–4 тижні. Часовий горизонт (1–2 міс / 3–6 міс / без горизонту). clinical-safety: PASS (scope не клінічний; редирект до лікаря при розладах сну). sports_scientist_review: required before publish.
- `content/post-650-self-programming-year-one.md` — Block 601–650 post 650. Self-programming рік перший: дізнаєшся лімітуючий фактор, власну динаміку відновлення, де програма розпадається у реальному житті, як калібрується RPE. Типові помилки: надто часта зміна програми, ігнорування щоденника, копіювання чужих висновків. Що змінюється на початку другого циклу. Перший цикл як інструмент збору даних. clinical-safety: PASS. sports_scientist_review: required before publish.

### Changed
- `README.md` — Block 601–650 статус: «in progress · 40/50» → «COMPLETE · 49/50 written · post-639 blocked clinical-safety HARD VETO». Пости 641–650 переміщено з «Remaining» до «Done» таблиці.
- `STATUS.md` — Block 601–650: 46 → 50, статус BLOCK COMPLETE. Total posts: 646 → 650. Next priorities: блок 651–700 planning.
- `VERSION` — 4.55.2 → 4.56.0.

---

## [4.55.2] — 2026-06-13

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.3 → v2.4. Registered three new DEC-030 HMAC-chained event types for the Privacy Incident Lifecycle (SOC2_READINESS §77): `privacy.incident_opened` (CRITICAL/STANDARD, 7yr/3yr by severity class), `privacy.incident_reviewed` (HIGH, 7yr), `privacy.incident_closed` (HIGH, 7yr). Includes: full Zod v2 schemas (canonical source for `emit-audit-event` Worker validation); PRIV-INC-CHAIN-01 ordering invariant enforcement spec (HTTP 422 on violation, PI-P3 `incident_reviewed` optional per OQ-PRIV-INC-02 resolution); PagerDuty routing table (PI-P0 triple-page `form-privacy` CRITICAL; PI-P1 dual-page HIGH; PI-P2 Slack only); evidence artefact queries PRIV-PI-E-001–E-004 with filing paths; emitter assignment rules for compliance-officer and security-engineer. Retention table: +3 rows (`privacy.incident_opened` PI-P0/P1 7yr; `privacy.incident_opened` PI-P2/P3 3yr; `privacy.incident_reviewed`/`privacy.incident_closed` 7yr). Taxonomy section: new `### Privacy Incident Lifecycle events` block after Privacy floor enforcement events. Closes SOC2_READINESS §77.10 P0 checklist item 1.
- `docs/SOC2_READINESS.md` — v3.7.0 → v3.7.1. §77.10 checklist item 1 marked [x] Done (AUDIT_LOG_SCHEMA.md v2.4). §77.10 checklist item 2 updated to include PI-P3 exception in PRIV-INC-CHAIN-01 spec. OQ-PRIV-INC-02 (§77.11) resolved 🟢: PI-P3 `privacy.incident_reviewed` optional; enforcement rule documented in AUDIT_LOG_SCHEMA.md.
- `VERSION` — 4.55.1 → 4.55.2.

---

## [4.55.1] — 2026-06-13

### Added
- `content/post-646-time-constrained-training-30min.md` — Programming edge cases block, post 46/50. Тренування в умовах обмеженого часу: 30-хвилинна сесія з реальним стимулом. MED (мінімальний ефективний стимул) як спосіб думати про час, а не «краще ніж нічого». Що можна і не можна зробити за 30 хвилин. Три архітектури скороченої сесії: сила (один базовий рух + аксесуар), гіпертрофія (два рухи), технічна сесія. Три типові помилки: скорочення відпочинку замість об'єму, хаотичний кардіо компонент, вибір вправ за складністю замість специфічності. Стратегії вбудовування: резервна сесія, структуровані мінімальні тижні, пріоритизація рухів. Коли 30 хвилин — стабільний ліміт: перепроектування програми (частота зростає, розподіл стає вузьким). Clinical-safety: PASS. Sports-scientist: pending.

### Changed
- `blog.html` — картки post-644, post-645, post-646 додані першими у grid.
- `STATUS.md` — блок 601–650: 45/50 → 46/50, total posts 645 → 646, версія v4.55.0 → v4.55.1.
- `VERSION` — 4.55.0 → 4.55.1.

---

## [4.55.0] — 2026-06-13

### Added
- `content/post-644-training-log-diagnostic-tool.md` — Programming edge cases block, post 44/50. Тренувальний щоденник як діагностичний інструмент. Розмежування між записом-хронологією і діагностичним інструментом. Мінімально корисний запис: 5 елементів за 30 секунд (дата, навантаження×підходи×повторення, RPE, відповідність плану, один рядок відновлення). Три часових горизонти читання: сесія (шум), тиждень (перший сигнал), місяць/мезоцикл (патерн). Числове визначення стагнації: той самий топ-результат при тому самому RPE 4+ сесії поспіль — з прикладом жиму лежачи. Три діагностичні патерни: стагнація / сигнал накопиченого перевантаження (навантаження ↓ при RPE ↑) / недостатнє програмування (RPE ніколи не перевищує 7–7.5). Проблема затримки: 2–4 тижні між стимулом і вимірюваною адаптацією; реагуй на 3–4 тижні патерну, а не на окремі сесії. Цикл оновлення: три питання і дерево рішень після мезоциклу. Чотири пастки: логуєш але не читаєш / переписуєш історію / підтверджувальне упередження / надмірна деталізація. Clinical-safety: PASS. Sports-scientist: pending.
- `content/post-645-autoregulation-vs-percentage-programming.md` — Programming edge cases block, post 45/50. Авторегуляція проти відсоткового програмування: де що працює. Механіка відсоткового програмування і ключове припущення (стабільний 1RM). Вимірювана добова варіація 1RM (3–8% в залежності від умов): сон, накопичена втома, стрес поза залом, час доби, якість розминки — з конкретною математикою на 100 кг. RPE-шкала (1–10) пояснена з практичними прикладами: 10 (відмова) / 9 (1 rep in reserve) / 8 (2 RIR) / 7 (3 RIR) / 6 (4+ RIR). Де відсотки перемагають: початківці без відкаліброваного RPE, піковий блок, дистанційне тренування, атлети зі систематичним відхиленням. Де авторегуляція перемагає: високостресові періоди, досвідчені атлети, підтримувальний блок, допоміжні вправи і відкотні підходи. Гібридний підхід: відсоток як якір + RPE як сесійний коректор (±5% при RPE ≥9 або ≤6) з конкретним прикладом сесії присідання. Найпоширеніша помилка при переході: sandbagging (систематичне заниження зусилля) — як виявити і 4-тижневий калібрувальний протокол. Clinical-safety: PASS. Sports-scientist: pending.

### Changed
- `STATUS.md` — content table: 643 → 645 posts; block 601–650: 43/50 → 45/50; next: 646.
- `VERSION` — 4.54.0 → 4.55.0.

---

## [4.54.0] — 2026-06-13

### Added
- `shared-responsibility.html` — Enterprise Shared Responsibility Model as a public due diligence page. 13 sections: summary matrix (28 rows across 5 domains), infrastructure security (§3: physical, network, patching), application security (§4: auth, vulnerability management, CV pipeline), identity & access management (§5: user lifecycle, RBAC roles, session policy), data security & privacy (§6: encryption, multi-tenant isolation, GDPR controller–processor boundary), audit logging (§7: DEC-030 HMAC chain, log categories by retention and read access), operations & incident response (§8: P0–P3 severity tables with step-level ownership), compliance & regulatory obligations (§9: FORM's SOC 2/GDPR/pen-test obligations + customer's HIPAA/ISO/FedRAMP/consent obligations), onboarding & offboarding (§10: step-by-step ownership), liability boundaries (§11: escalation path + out-of-scope controls), non-negotiable floors (§12: 6 declined request types), glossary (§13: 13 terms). Colour-coded FORM/Shared/Customer pills. Sticky ToC navigation. Consistent design system: Fraunces + Manrope + JetBrains Mono, ink/paper/acid palette, grain overlay.
- Referenced: `docs/SHARED_RESPONSIBILITY_MODEL.md`, `docs/ENTERPRISE.md`, `docs/AUDIT_LOG_SCHEMA.md`, DEC-030, DEC-036, DEC-043, DEC-045.

### Changed
- `enterprise.html` — "Shared responsibility model" due diligence card updated from "AVAILABLE ON REQUEST" placeholder to live link `/shared-responsibility.html` with hover state. Compliance badge in §Compliance & security posture section also linked. Owners: enterprise-architect + compliance-officer + security-engineer.
- `VERSION` — 4.53.1 → 4.54.0.

---

## [4.53.1] — 2026-06-13

### Added
- `content/post-643-program-to-principles.md` — Programming edge cases block, post 43/50. Від програми до принципів: коли атлет готовий самостійно планувати. Розмежування між операційними принципами і афоризмами. Чому передчасна автономія небезпечна: bias на те що подобається, надмірна реакція на короткострокові сигнали, недооцінка часових горизонтів, відсутність зовнішньої перевірки. П'ять операційних індикаторів готовності: розуміння структури поточної програми / два задокументованих цикли / відрізнення нормальної і накопиченої втоми / досвід утримання плану всупереч суб'єктивним відчуттям / адаптація при зміні обставин. Чотирифазний перехід: документування розуміння → модифікації в межах структури → самостійний мезоцикл → повна автономія. Відомі пастки першого циклу: об'єм, новизна, самоаналіз, порівняння. Коли повернутись до готової програми. Операційна перевірка з 6 запитань після кожного мезоциклу. Clinical-safety: PASS. Sports-scientist: pending.

### Changed
- `blog.html` — додано картку post-643 на початок grid.
- `README.md` — заповнено topics 644–650 з пулу Block 851–900.
- `STATUS.md` — content table: 642 → 643 posts; block 601–650: 42/50 → 43/50; next: 644.
- `VERSION` — 4.53.0 → 4.53.1.

---

## [4.53.0] — 2026-06-13

### Added
- `content/post-641-first-program-after-40.md` — Programming edge cases block, post 41/50. Перша програма після 40 — що змінити в дозуванні і відновленні. Ключові зміни: вікно відновлення 72 год (vs 48 год для молодших), deload 3:1 (vs 4:1), об'єм −15–25%, інтенсивність (75–85% 1ПМ) зберігається без змін, структурована розминка 10–15 хв, темп ексцентрики 3–4 сек. Що НЕ змінюється: важкий тренінг ефективний після 40, нейральні адаптації ті самі, переходити на легкі ваги — помилка. Практична таблиця коригування параметрів до/після 40. Hormonologічна база: тестостерон −1–2%/рік, GH/IGF-1 впливають на відновлення, не на потенціал сили. Sports-scientist note: 72h recovery window — практична рекомендація, RCT-база специфічно для майстер-популяції обмежена. Clinical-safety: PASS.
- `content/post-642-programming-sufficiency-principle.md` — Programming edge cases block, post 42/50. Коли програмування стає надто складним: принцип достатності. Три джерела надмірної складності: нові знання, складність як сигнал серйозності, накопичення шарів з різних систем. Мета-аналіз: різниця між методами periodization для нееліт мінімальна; adherence передбачає результат краще, ніж складність схеми. Принцип достатності: мінімальна складність для прогресу в конкретному контексті. Три ознаки: не пояснити план за 3 речення / пропуски через «не встиг спланувати» / більше планування ніж тренінгу. Що достатньо по цілях: рекреаційна сила vs. змагальний пауерліфтер vs. загальний атлет. Коли складність виправдана: стагнація 3+ міс / підготовка до змагань / специфічна слабкість / 5+ сесій/тиждень. 5-кроковий алгоритм спрощення. Операційна перевірка з 5 запитань. Clinical-safety: PASS.

### Changed
- `STATUS.md` — content table: 640 → 642 posts; block 601–650: 40/50 → 42/50; next priorities updated.
- `VERSION` — 4.52.1 → 4.53.0.

---

## [4.52.1] — 2026-06-13

### Added
- `docs/AUDIT_LOG_SCHEMA.md §Incident` — 10 `incident.*` lifecycle events (INCIDENT_RESPONSE §16, CC7.3/CC7.4/CC7.5): `incident.opened` (IRCHAIN-01 anchor; `siem_correlation_score` + `trigger_alert_id` for SIEM auto-open path), `incident.ic_assigned` (`minutes_since_open` as IR-SLO-01 evidence), `incident.severity_changed` (downgrade requires `approver_user_id` + `reason`), `incident.escalated`, `incident.update_posted` (`privacy_floor_verified: literal(true)` — IC attestation; false rejected HTTP 422), `incident.containment_verified`, `incident.eradicated` (`root_cause_category` 7-value enum), `incident.recovered`, `incident.pir_opened` (`pir_due_at` — IR-SLO-03 clock anchor), `incident.pir_closed` (`pir_document_sha256`). IRCHAIN-01 sub-chain rule + dead-man's pg_cron switches documented. Closes INCIDENT_RESPONSE.md §16.8 checklist items 1–2 (P0, M4). Six Zod schemas provided.
- `docs/AUDIT_LOG_SCHEMA.md §Wearable` — 6 `wearable.*` sync pipeline events (OBSERVABILITY §41, A1.1/P3.2): `wearable.sync_completed` (STANDARD, 2yr), `wearable.sync_failed` (HIGH, 3yr — `error_class` enum + SHA-256 error hash), `wearable.oauth_token_expired` (HIGH, 3yr — OAuth sources only: whoop/oura/garmin), `wearable.permission_revoked` (HIGH, **5yr** — GDPR Art. 7(3) withdrawal record; `consent_event_id` FK; no `user_id`), `wearable.fleet_freshness_assessed` (STANDARD, 2yr — k-anonymity gate on `sources_breakdown`), `wearable.stale_data_coaching_context` (HIGH, 3yr — WS-CHAIN-01: `coaching_context_downgraded: literal(true)` required; clinical-safety VETO on gate weakening). All 6 Zod schemas provided. Closes OBSERVABILITY.md §41.11 checklist item 1 (P0, M5).
- `docs/AUDIT_LOG_SCHEMA.md §LoadTest` — 5 `system.load_test_*` performance gate events (OBSERVABILITY §40, A1.1/CC5.2/CC8.1): `system.load_test_initiated` (`environment: literal('staging')` constraint), `system.load_test_completed` (`slo_results` object per PERF-SLO-01…05), `system.load_test_failed` (`gate_action: literal('merge_blocked')`), `system.load_test_gate_bypassed` (CRITICAL — AL-PERF-04 no-auto-resolve; `bypass_reason_hash` SHA-256), `system.perf_regression_detected` (quarterly PERF-SLO-06 drift). PERF-CHAIN-01 invariant documented. Closes OBSERVABILITY.md §40.10 checklist item 1 (P0, M5).
- `docs/AUDIT_LOG_SCHEMA.md §Pipeline` — 4 `enterprise.pipeline_*` ARR governance events (COST_MODEL §37, CC4.2): `enterprise.pipeline_reviewed` (STANDARD, 3yr — weekly PCR + weighted pipeline), `enterprise.arr_bridge_closed` (STANDARD, **7yr** — 5-component ARR bridge; Ukrainian Tax Code Art. 44 financial evidence floor), `enterprise.deal_aged_out` (STANDARD, 3yr — pg_cron job 31; `deal_id` internal UUID only), `enterprise.pipeline_conversion_model_recalibrated` (STANDARD, 7yr — PIPE-CHAIN-01: `decision_log_ref` required or HTTP 422). Closes COST_MODEL.md §37.10 checklist item 1 (P0, M7).
- `docs/AUDIT_LOG_SCHEMA.md §AdminKeyRotation` — 5 admin key rotation lifecycle events extending existing `### Admin (key management)` section (SOC2_READINESS §57, CC6.7/CC6.8): `admin.key_rotation_initiated` (CRITICAL, 7yr — PAM session anchor; must precede all rotation events for same `key_name` within 24h), `admin.key_rotation_announced` (HIGH, 7yr — Slack advance notice), `admin.emergency_key_rotation` (CRITICAL, 7yr — `gdpr_art33_clock_started_iso` field; GDPR 72h notification clock), `admin.key_rotation_reminder` (LOW, 3yr — `workers/key-rotation-monitor` at 14/7/3d pre-expiry), `admin.key_rotation_overdue` (HIGH, 7yr — `days_overdue`; triggers AL-KEY-02 P0 no-auto-resolve). Privacy invariant: JWT value never in chain — `iat` timestamp only. Closes SOC2_READINESS.md §57.11 checklist items 1–2 (P0, M5).

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — header v2.2 → v2.3; retention table +9 rows across five new groupings; version note added.
- `VERSION` — 4.52.0 → 4.52.1.

---

## [4.52.0] — 2026-06-13

### Added
- `content/post-640-shift-workers-scheduling-protocol.md` — Programming edge cases block, post 40/50. Shift-workers і тренування: конкретний алгоритм розкладу для кожного типу зміни. Ключова ідея: відносний час (3–7 год після пробудження) vs. абсолютний годинник — core body temperature як реальний регулятор. Окремий алгоритм для денної (16:00–20:00 вікно), вечірньої (11:00–13:00 або перед змінами), нічної (16:30–19:30 після денного сну) і ротаційної зміни (матриця: нічна=80% об'єму, перехід=deload 50%, стабільний тиждень=100%). Anchor sleep: фіксований 4-год блок сну як якір циркадного ритму при ротаційному графіку. Тиждень переходу між типами змін (особливо нічна→денна) = автоматичний deload, не дискутується. 2 сесії достатньо для прогресу при збереженій інтенсивності. Матриця пріоритетів А/Б/В: вікно ≥6 год/4–6 год/&lt;4 год. Правило 48 год між сесіями на ту саму м'язову групу — єдиний незмінний параметр. Clinical-safety: PASS.
- `README.md` — Block 851–900 «Applied programming deep dives» (20 тем) додано до роадмапу; Block 601–650 оновлено: 40/50, пости 637–640 marked done, 639 (хронічний біль) BLOCKED.

### Changed
- `blog.html` — картки для post-637, post-638, post-640 додані на початок гриду (були пропущені в попередніх ітераціях).
- `STATUS.md` — content table: 638 → 640; next priorities оновлено; Block 851–900 згадано.
- `VERSION` — 4.51.0 → 4.52.0.

### Skipped
- `post-639` — тема «тренування при хронічному болю» → clinical-safety HARD VETO. Не пишеться без explicit review від clinical-safety агента.

---

## [4.51.0] — 2026-06-13

### Added
- `content/post-637-training-with-partner.md` — Programming edge cases block, post 37/50. Тренування з партнером: ефект соціальної фасилітації (Triplett/Zajonc) — корисно для засвоєних навичок, шкідливо для нових. Три речі, які партнер реально дає: безпека при важких вправах до відмови, зовнішній зворотний зв'язок з техніки між підходами, підзвітність присутності як рішення для проблеми дотримання. Три речі, які забирає: темп тренування (очікування між сетами), залежність присутності як вразливість, ефект Рінгельмана (соціальне лінивство — індивідуальне зусилля знижується при груповій роботі). Програмування при різних рівнях: розподіл за вправами замість синхронізації ваги; процентне навантаження від 1ПМ як спільний знаменник. Практична структура продуктивної сесії: план до тренування, 30-сек ліміт на розмови між підходами, страховка тільки за сигналом, один конкретний технічний коментар після підходу. Clinical-safety: PASS.
- `content/post-638-no-equipment-programming.md` — Programming edge cases block, post 38/50. Програмування без обладнання: де межа, як прогресувати. Чесна оцінка: bodyweight достатній для верхньої частини тіла 1–2 роки і для загальної фізичної підготовки тривало; недостатній для гіпертрофії нижньої частини тіла після першого року (квадрицепс/хамстрінгс адаптовані до ваги тіла через повсякденне навантаження). Чотири механізми прогресії: lever progression (archer push-up, HSPU, Nordic curl), tempo manipulation (4-1-2-0 ексцентрик), range of motion extension (відмивання з підвищення), volume progression (з застереженням: 3×25 = витривалість, не гіпертрофія). Конкретні прогресії: верхня тяга (horizontal row → Australian pull-up → підтягування → archer pull-up), груди/трицепс (стандарт → archer → HSPU), плечі (pike → wall walk → strict HSPU), ноги (присідання → Bulgarian split squat → пістолет → стеля). Мінімальне обладнання з непропорційним поверненням: турнік, паралетки/кільця, одна гиря 20–24 кг. Матриця «коли достатньо»: підтримка форми — так, загальна фізпідготовка — так, гіпертрофія верху 1–2 роки — так, гіпертрофія низу > 1 рік — ні. Clinical-safety: PASS.

### Changed
- `STATUS.md` — content table updated: posts 637–638 added, total posts 636 → 638; next priorities updated to reflect post-639 as next (chronic pain — clinical-safety HARD VETO required before write).
- `VERSION` — 4.50.0 → 4.51.0.

---

## [4.50.0] — 2026-06-13

### Added
- `docs/ROPA.md` — GDPR Art. 30 Records of Processing Activities v0.1. Closes `docs/GDPR_DPIA.md §10.2.1` (explicit blocking open item: "formal Art. 30 register required before SOC 2 observation period, Month 2") and `docs/SSO_SCIM_IMPLEMENTATION.md §15.9.4` (SCIM group processing note requiring Art. 30 registration). Dual-role structure: Art. 30(1) register as Data Controller (RA-01–RA-13) + Art. 30(2) register as Data Processor for enterprise customers (RP-01–RP-06). Art. 30(1) controller register covers all 12 processing activities from `docs/GDPR_DPIA.md §2.1` (account/identity, health profile, CV pose estimation, meal logging, Victor AI coaching, voice synthesis, wearable integration, ED screening, enterprise aggregate reporting, audit logging, product analytics, error monitoring) plus RA-13 SCIM group membership from `docs/SSO_SCIM_IMPLEMENTATION.md §15.9.4` — legal basis, data categories, Art. 9 flag, sub-processors, retention, transfer mechanism per activity. Art. 30(2) processor register covers RP-01–RP-06 (employee SSO provisioning, employee health data processing, aggregate reporting, audit log, DSAR fulfilment, tenant offboarding). §2 controller identity with ROPA-GAP-001 (Art. 27 EU representative not designated — P0 before EU marketing). §6 international transfer summary: 10 transfer routes mapped with SCCs Module 2 / EU adequacy / Cloudflare DPA status; ROPA-GAP-002 (UK IDTA not verified), ROPA-GAP-003 (TIA not completed). §7 retention schedule aligned to `compliance/p1/retention-decisions.md` (P1-RET-001): 13 data categories with retention periods, legal basis, deletion method, implementation status; Art. 17 erasure cascade (12-step hard-delete sequence); enterprise tenant offboarding 90-day deletion procedure. §8 TOMs summary with 12 control categories cross-referenced to security docs. §9 DEC-030 audit events: 17 events including 3 new ROPA-specific events (`ropa.review_completed`, `ropa.processing_activity_added`, `ropa.processing_activity_modified`) pending registration in `docs/AUDIT_LOG_SCHEMA.md`. §10 gaps: 6 P0 gaps (EU rep, UK transfer, TIA, TTL migrations, DEC-030 registration, VRM-GAP-001 DPAs) + 6 P1 gaps (first EU deal). §11 implementation checklist: 6 P0 / 6 P1 / 4 P2 items. Consent dependency map (§3.2): five-category consent model with withdrawal cascade. Sub-processor processing scope table (§5): all 11 sub-processors with Art. 9 exposure flag. Privacy invariant throughout: RA-09 enterprise aggregate reporting: k-anonymity floor N < 5; admin roles have zero access to individual health rows. Owner: compliance-officer + enterprise-architect + security-engineer.

### Changed
- `VERSION` — 4.49.0 → 4.50.0.

---

## [4.49.0] — 2026-06-13

### Added
- `content/post-634-microloading-progressive-overload.md` — Programming edge cases block, post 34/50. Мікролодинг і дрібна прогресія: коли стандартних стрибків не вистачає. Математика відносного приросту навантаження як причина стопора (4.2% від 60 кг vs. 2.1% від 120 кг). Реалізація мікролодингу: фракційні блини, пін-клавіші кабельного стека, еластичні стрічки як доважок, кроковані гантелі. Чому верхня частина тіла вичерпує лінійну прогресію швидше (розмір м'язових груп, нейром'язова специфіка при нижче 70–75% 1ПМ). Практичні приклади: 6-тижневий мікролодинг OHP 60→62.5 кг (крок 0.5 кг/тиж) і хвильова модель для жиму лежачи 80→83 кг з об'ємним тижнем. Дерево рішень: мікролодинг vs. розвантаження vs. зміна вправи — операційні критерії. Clinical-safety: PASS.
- `content/post-635-frequency-volume-tradeoff.md` — Programming edge cases block, post 35/50. Частота проти об'єму: що обирати, коли часу менше. Equated-volume findings (Schoenfeld, Ralston): при рівному тижневому об'ємі різниця між 2 і 4 сесіями незначна. Математика обмеження: чому перенесення того самого об'єму в менше сесій = більше стресу при тому ж стимулі. Виняток для моторного навчання (Olympic lifts, змагальний пауерліфтинг). Реструктуризація під 2 дні: Full Body A+B шаблони (15+14 підходів). Реструктуризація під 3 дні: Upper/Lower/Full та стиснутий PPL з конкретними шаблонами. Ієрархія скорочень: ізоляція першою, compound movements захищені. Попередження: великий разовий об'єм генерує непропорційно більше DOMS і втому — рекомендація скоротити тижневий об'єм на 15–25% при переході. Clinical-safety: PASS.

### Changed
- `VERSION` — 4.48.0 → 4.49.0.

---

## [4.48.0] — 2026-06-13

### Added
- `docs/OBSERVABILITY.md §41` — Wearable Integration & Health Platform Sync Pipeline Observability (v3.8). Closes the data-collection-availability gap between `docs/DATA_MODEL.md §14` (schema) and any backend ingestion monitoring. Covers five sources: HealthKit (iOS background delivery), Health Connect (Android WorkManager), Whoop (OAuth 2.0 polling), Oura (OAuth 2.0 daily), Garmin (OAuth 1.0a enterprise). Privacy invariant throughout: no `user_id`, no reading values, no health field content in any observability signal. RED metrics published to `WEARABLE_TELEMETRY` Analytics Engine binding: six rate signals (sync success rate by source; OAuth grant/revocation counts), five error signals (failure rate by `error_class` enum; stale-data coaching event rate), four duration signals (p95 ingestion latency; Whoop API response time). Six WS-SLO-* entries: WS-SLO-01 (HealthKit ≥ 99%/24h), WS-SLO-02 (Health Connect ≥ 97%/24h — Android WorkManager constraint), WS-SLO-03 (Whoop ≥ 98%/24h), WS-SLO-04 (Oura ≥ 99%/24h), WS-SLO-05 (Garmin ≥ 95%/24h — provisional; OQ-WS-OBS-01), WS-SLO-06 (fleet freshness ≥ 95% of active connected users have reading < 26h old, daily 07:05 UTC). Seven AL-WS-* alert rules: AL-WS-01 (P1, Whoop OAuth expiry wave > 5% in 1h), AL-WS-02 (P1, Whoop 429 rate > 10% in 15 min), AL-WS-03 (P2, Health Connect failure > 10% in 30 min), AL-WS-04 (P1, Oura total failure 15 min), AL-WS-05 (P2, HealthKit stall > 4h for active tenant), AL-WS-06 (P1, WS-SLO-06 breach at daily check — no auto-resolve), AL-WS-07 (P2, Garmin OAuth expiry on enterprise tenant). pg_cron job 31 `wearable_sync_freshness_check` (07:05 UTC daily; k-anonymity gate N < 5 per source → suppressed). Six DEC-030 HMAC-chained events: `wearable.sync_completed` (STANDARD, 2yr), `wearable.sync_failed` (HIGH, 3yr), `wearable.oauth_token_expired` (HIGH, 3yr), `wearable.permission_revoked` (HIGH, 5yr — GDPR Art. 7(3) withdrawal record; linked to consent chain via `consent_event_id`), `wearable.fleet_freshness_assessed` (STANDARD, 2yr), `wearable.stale_data_coaching_context` (HIGH, 3yr). §41.7 coaching safety integration: stale HRV data (> 48h) requires mandatory confidence downgrade in Victor's coaching output; `wearable.stale_data_coaching_context` is the compliance evidence; clinical-safety has VETO over any change weakening the 48h gate. E-WEARABLE-01 communication template for enterprise tenant notification on Whoop OAuth expiry waves. Six-panel Metabase dashboard (FORM-Platform collection; no PII; k-anonymity enforced). Three SOC 2 evidence artefacts: WS-E-001 (A1.1 — quarterly sync_completed chain excerpt), WS-E-002 (P3.2 — quarterly fleet freshness report), WS-E-003 (CC7.2 — alert rule config screenshots). Two open questions: OQ-WS-OBS-01 (P1 — Garmin SLA confirm before M10), OQ-WS-OBS-02 (P2 — stale-coaching-context namespace ownership; resolve M8). Ten-item implementation checklist: 4× P0/M5–M6, 3× P1/M7–M8 (incl. clinical-safety sign-off required), 3× P2/M9–M10. Cross-references: DATA_MODEL §14 (schema companion), DATA_MODEL §17 (enterprise reporting aggregate), OBSERVABILITY §28 (mobile client complement), OBSERVABILITY §32 (Victor AI Safety cross-ref), OBSERVABILITY §33 (QBR metrics input), SOC2_READINESS §35.6.3 (2yr retention), AUDIT_LOG_SCHEMA §Wearable (six events to register — P0), CLINICAL_SAFETY.md, DEC-030.

### Changed
- `VERSION` — 4.47.1 → 4.48.0.

---

## [4.47.1] — 2026-06-13

### Added
- `content/post-633-accumulation-block-design.md` — Programming edge cases block, post 33/50. Блок накопичення: механіка трифазної блокової структури (накопичення → інтенсифікація → реалізація). Чому порядок має фізіологічне обґрунтування: гіпертрофія передує силовій реалізації. Тривалість 4–6 тижнів, MEV→MAV прогресія об'єму. Підбір вправ і логіка допоміжки у накопиченні. Три механіки прогресії (підходи, вага, щільність). Ознаки правильного і порушеного блоку. Деload як механічна частина структури. Типові помилки: 1RM тест в середині блоку, змішування логіки фаз. Практичний 5-тижневий шаблон для першого блоку.

### Changed
- `VERSION` — 4.47.0 → 4.47.1.

---

## [4.47.0] — 2026-06-13

### Added
- `content/post-630-chronic-sleep-deficit-training.md` — Programming edge cases block, post 30/50. Тренування при хронічному дефіциті сну (5–6 год). Фізіологія відновлення при обмеженому сні: пульсові піки GH під час slow-wave sleep, суб'єктивний RPE vs. об'єктивні показники, накопичена центральна втома. Основний принцип: тимчасова адаптація програми, а не постійне зниження. Конкретні зміни: зниження об'єму на 30–40%, збереження інтенсивності, аутрегуляція по відчуттю. Мінімальний підтримуючий обсяг. Протокол повернення.
- `content/post-631-training-under-life-stress.md` — Programming edge cases block, post 31/50. Тренування під час нетренувального стресу (робота/сім'я). Allostatic load як конкуренція за спільний відновлювальний ресурс. HPA-вісь і кортизол — з кваліфікаторами. Три типові помилки: тиснути на той самий об'єм, повна пауза, «дочекатись коли відпустить». Підтримуючий режим vs. розвиваючий — операційна різниця. Аутрегуляція по RPE: рішення на тренуванні, коли скоротити сесію. Протокол повернення до повного об'єму (2–3 тижні).

### Changed
- `VERSION` — 4.46.0 → 4.47.0.

---

## [4.46.0] — 2026-06-13

### Added
- `content/post-632-minimum-effective-dose.md` — Programming edge cases block, post 32/50. Мінімальна ефективна доза: операційна різниця між мінімумом для прогресу і мінімумом для підтримання. Сила: 2–4 важких підходи на рух 2 рази на тиждень. Гіпертрофія: 5–10 підходів на м'язову групу на тиждень (вищий поріг, ніж для сили). Підтримання: 1–2 важких підходи 1 раз на тиждень при збереженні інтенсивності ≥80% 1RM — об'єм вдвічі менший від «будуючого» протоколу. Шаблони 2-day full-body (прогрес) і 1-day (підтримання). Чотири пастки мінімалізму. Протокол повернення до повного об'єму.
- `content/newsletter-05.md` — Newsletter issue 05 (draft). Тема: «Your wearable has an opinion. It's not always right.» Синтез training-with-tech блоку (posts 591–600). Чому readiness score не є prescriptive. PPG vs. ECG обмеження. HRV: trend over 14 days is signal, daily point is noise. Три практичні межі корисності носимих пристроїв. Training log як первинний інструмент. Clinical safety: PASS (no food/body/injury/mental health content).

### Changed
- `VERSION` — 4.45.1 → 4.46.0.

---

## [4.45.1] — 2026-06-13

### Changed
- `docs/INCIDENT_RESPONSE.md` v2.2 → v2.3 — two additions:
  - **R-14.8 update:** Five new DSAR events from `docs/ENTERPRISE_SLA.md §19.5` (v1.1) added to DEC-030 event table: `dsar.data_provided` (HIGH, 7yr — enterprise Art. 15 export provision, 24-hour SLO), `dsar.deletion_soft` (HIGH, 7yr — Art. 17 soft delete), `dsar.deletion_confirmed` (HIGH, 7yr — hard-delete + deletion certificate), `dsar.portability_export_completed` (STANDARD, 7yr — employee self-serve Art. 20), `dsar.offboarding_export_available` (STANDARD, 7yr — contract-end bulk export). Privacy floor: `user_id_hash` (SHA-256) in portability event; no plaintext user identifiers in any event.
  - **R-26 new:** SCIM Deprovisioning Latency SLA Breach runbook. Closes the incident-response gap created when `docs/ENTERPRISE_SLA.md §3.7` (v1.1, OQ-SLA-03 resolved) defined AL-SCIM-LAT-02 (P1 auto-open; any single SCIM DELETE > 5 min) with no corresponding runbook. Distinct from R-24 (mass deprovisioning). Trigger matrix, severity table, immediate actions T+0 to T+15, scope query R-26-C1, five root cause hypotheses H1–H5, manual revocation procedure, Template LAT-01, three DEC-030 HMAC-chained events (`scim.deprovision_latency_breach` HIGH 7yr, `scim.deprovision_sessions_manual_revoke` HIGH 7yr, `scim.deprovision_latency_resolved` STANDARD 3yr), five evidence artefacts LAT-E-001–LAT-E-005, SOC 2 CC6.3/CC7.2/CC7.3 mapping, five-item implementation checklist.
- `VERSION` — 4.45.0 → 4.45.1.

---

## [4.45.0] — 2026-06-13

### Added
- `content/post-629-linear-progression-exhausted-next-steps.md` — Programming edge cases block, post 29/50. Лінійна прогресія: чому вона закінчується не через помилку, а через успіх (нейрологічна адаптація завершена). Три системні ознаки вичерпання vs. тимчасова стагнація. Механіка: чому після 9–18 місяців потрібна більша одиниця прогресу — тиждень замість сесії. Intermediate шаблони: тижнева хвиля, Texas Method, Madcow — той самий принцип прогресивного перевантаження, інший часовий горизонт. Перехід без скидання ваг. Орієнтири тренувального віку по статі і початковому рівню.

### Changed
- `blog.html` — added card for post-629 at top of grid.
- `STATUS.md` — content table: post-629 added, total 628 → 629; block 601–650 progress 28/50 → 29/50; version 4.44.0 → 4.45.0.
- `VERSION` — 4.44.0 → 4.45.0.

---

## [4.44.0] — 2026-06-13

### Added
- `content/post-626-parasite-load-informal-activity.md` — Programming edge cases block, post 26/50. Паразитне навантаження: типологія (хронічне фонове, гостре разове, структурована паралельна активність), механіка впливу на відновлення (глікоген 24–48 год, MPS до 72 год, ЦНС до 7 днів), аудит неформальної активності, чотири принципи корекції програми, дзеркальний бік — сидячий спосіб життя і роль фонових кроків.
- `content/post-627-tournament-multi-match-programming.md` — Programming edge cases block, post 27/50. Програмування навколо турнірів: турнірний тиждень як окремий режим (мета — виступити і відновитися, не тренуватися), структура пн–пт перед турніром, правило 48–72 год після, in-season мінімальна ефективна доза (Häkkinen et al. 1985, Bickel et al. 2011: ⅓ об'єму при збереженні інтенсивності ≥75–85% 1ПМ), помилки «надолужити до/після».
- `content/post-628-deload-vs-taper-non-competitive.md` — Programming edge cases block, post 28/50. Діліод vs. тейпер: операційна різниця (відновлення між блоками vs. пік у конкретну дату), механіка fitness-fatigue моделі, параметри тейперу за Mujika & Padilla 2003 (−41–60% об'єму, 7–14 днів, збереження інтенсивності), застосування для не-змагального атлета (1ПМ-тест, перший матч, фізична подія), порівняльна таблиця.

### Changed
- `blog.html` — added cards for posts 626, 627, 628 at top of grid.
- `STATUS.md` — content table: posts 626–628 added, total 625 → 628; block 601–650 progress 25/50 → 28/50; next priorities updated.
- `VERSION` — 4.43.1 → 4.44.0.

---

## [4.43.1] — 2026-06-13

### Changed
- `docs/ENTERPRISE_SLA.md` v1.0 → v1.1 — three additions:
  - **§3.6 update (OQ-SLA-02 resolved):** Credits auto-apply for all tiers; Starter/Growth customers can opt into Manual Credit Mode in Admin Dashboard → Billing → Credit Preferences. `sla.credit_pending_claim` DEC-030 event holds credit for 90 days in manual mode.
  - **§3.7 new (OQ-SLA-03 resolved):** SCIM deprovisioning latency SLA — SCIM DELETE → sessions revoked P99 < 60 s; breach threshold > 5 min triggers auto P1 incident + `tenant_security_contact` notification within 30 min. Provisioning P99 < 60 s. Better Stack probe S-014 (`scim-deprovision-latency`) with alert rules AL-SCIM-LAT-01 (P2) and AL-SCIM-LAT-02 (P1) defined.
  - **§19 new:** DSAR and Data Portability SLAs — DSAR data provision within 24 h of Customer's forwarded request; Art. 17 erasure: soft delete immediate, hard-delete cascade + deletion certificate within 30 days; Art. 20 employee self-serve export 24 h target; offboarding bulk export window 30 days; five new DEC-030 DSAR events (`dsar.request_received`, `dsar.data_provided`, `dsar.deletion_soft`, `dsar.deletion_confirmed`, `dsar.portability_export_completed`, `dsar.offboarding_export_available`).
  - **§16 updated:** CC6.4 (SCIM deprovisioning) and P8.1 (DSAR SLAs) added to SOC 2 evidence mapping.
- `VERSION` — 4.43.0 → 4.43.1.

---

## [4.43.0] — 2026-06-13

### Added
- `content/post-625-strength-team-sport-programming.md` — Programming edge cases block, post 25/50. Сила + командний спорт без саботажу: interference effect (AMPK/mTORC1 механізм), чотири фази сезонного планування (offseason/preseason/in-season/postseason) з параметрами обсягу/інтенсивності для кожної, мінімальна ефективна доза in-season (≥80% 1RM, 1–2 сесії/тиждень), правило 48 годин секвенсингу після матчу, практичний шаблон мікроциклу, типові помилки. Sources: Wilson et al. 2012, Rhea et al. 2002, Hammami et al. 2017, Rønnestad et al. 2011, Hawley 2009, Fyfe et al. 2014.

### Changed
- `blog.html` — added card for post-625 at top of grid.
- `STATUS.md` — content table: post-625 added, total 624 → 625; block 601–650 progress 24/50 → 25/50; next priorities updated.
- `README.md` — roadmap: post-625 added to block 601–650 done list; new topics appended for posts 626–650.
- `VERSION` — 4.42.0 → 4.43.0.

---

## [4.42.0] — 2026-06-13

### Added
- `content/post-623-volume-distribution-microcycle.md` — Programming edge cases block, post 23/50. Розподіл об'єму між тренувальними днями: чому рівномірний розподіл субоптимальний, ієрархія hard/medium/easy сесій, логіка сиквенсингу рухів, практичні фреймворки для 3/4/5-денних сплітів. Sources: Israetel (2019) MEV/MAV/MRV, Schoenfeld 2017, Helms 2018, Phillips & Van Loon 2011 (MPS window), Seiler 2010 (polarization principle).

### Changed
- `STATUS.md` — content table: post-623 added, total 623 → 624; block 601–650 progress 23/50 → 24/50.
- `VERSION` — 4.41.0 → 4.42.0.

---

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
