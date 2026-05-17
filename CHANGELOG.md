# Changelog · FORM

Усі помітні зміни в цьому проєкті документуються тут.
Формат базується на [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/).
Версіювання: **semver-light** — `MAJOR.MINOR.PATCH`.

| Bump | Коли |
|---|---|
| **PATCH** (`x.y.Z`) | Кожна cloud-ітерація. Один концепт = один bump. |
| **MINOR** (`x.Y.z`) | Нова фіча, новий розділ документації, помітна зміна. |

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
