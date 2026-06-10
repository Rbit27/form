# Changelog · FORM

Усі помітні зміни в цьому проєкті документуються тут.
Формат базується на [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/).
Версіювання: **semver-light** — `MAJOR.MINOR.PATCH`.

---

## [3.23.0] — 2026-06-10

### Added
- `content/post-383-training-chronic-stress-volume-adjustment.md` — Тренування при хронічному стресі: коригування об'єму і інтенсивності без зупинки прогресу; McEwen & Stellar (1993) алостатичне навантаження — єдиний рахунок для всіх стресорів; Meeusen et al. (2013) ECSS/ACSM: NFOR настає при нижчому тренувальному об'ємі при підвищеному загальному стресі; Shimizu et al. (2011) — хронічний кортизол пригнічує mTORC1 через REDD1 і знижує MPS-відповідь; Goldberg (1980) убіквітин-протеасомний катаболізм при глюкокортикоїдній експозиції; Leproult & Van Cauter (2010) петля кортизол→сон; Marcora et al. (2009) / Pageaux & Lepers (2018) RPE-дрейф при когнітивній втомі; Raastad et al. (2003) тейперинг: об'єм вниз, інтенсивність стабільна; Buchheit (2014) HRV-тренд як маркер; Hooper et al. (1995) суб'єктивна готовність; Kreher & Schwartz (2012) ранні ознаки NFOR; Fry & Kraemer (1997) відновлення перформансу після скорочення об'єму; практичний протокол: 5 тригерів, -35% об'єму, інтенсивність незмінна, оцінка через 7 днів, паралельний лог; 13 хв читання; editorial-brutalist; clinical-safety REVIEW RECOMMENDED (оцінка: LOW — порівнянний із post-379 PASS і post-167 NOT REQUIRED)
- `blog.html` — +1 картка post-383; статей лічильник виправлено і оновлено 319 → 325 (враховує 6 карток, що були написані але не відображені у лічильнику); хвилин 3 883 → 3 896 (+13 хв)
- `README.md` — post-383 позначено як виконаний у ROADMAP; додано roadmap post-387–390
- `STATUS.md` — версія оновлена до v3.23.0; 382 → 383 posts; blog cards 324 → 325; лічильник статей виправлено
- `VERSION` — 3.22.0 → 3.23.0

---

## [3.22.0] — 2026-06-10

### Added
- `content/post-380-neural-efficiency-beginner-effect.md` — Чому новачок росте від будь-чого — а досвідчений атлет ні; Sale (1988) neural adaptation review — нейральні фактори домінують у ранніх силових адаптаціях нетренованих; три механізми пластичності початківця: рекрутування high-threshold МО (Enoka 2002 — incomplete recruitment at sub-maximum voluntary force), doublet discharges і rate coding (Van Cutsem et al. 1998), зниження коактивації антагоністів; Rhea et al. (2002) — крива чутливості початківця зсунута ліворуч; Moritani & deVries (1979) часова шкала нейральної фази; три програмувальних помилки (надскладні програми, часта зміна, передчасні техніки); три сигнали завершення новачкового вікна; 12 хв читання; editorial-brutalist; clinical-safety NOT REQUIRED
- `content/post-381-weight-selection-rir-anchor.md` — Як вибрати вагу для підходу: не «відчувай», а калібруй — практика RIR-якоря; Cronin & Sleivert (2005) ±5–8% щоденна варіативність 1RM; Zourdos et al. (2016) кореляція між % 1RM і реальною RPE між сесіями; Jukic et al. (2020) системне відхилення від цільової інтенсивності без зворотного зв'язку; Helms et al. (2016) консервативний старт в авторегуляції; RIR-якір як концепція: перший підхід з буфером RIR+2 як калібрувальна точка; п'ятикроковий протокол; тижневий дрейф і кумулятивна втома між підходами; числовий приклад 4×6 присідання; 11 хв читання; editorial-brutalist; clinical-safety NOT REQUIRED
- `content/post-382-minimum-effective-dose-adaptation.md` — Мінімальна ефективна доза: скільки насправді потрібно для прогресу; відмінність від post-367 (підтримка) — цей пост про мінімум для ПРОГРЕСУ; Schoenfeld et al. (2017) доза-відповідь плато при >20 підходах; Krieger (2010) 2-3 vs 1 сет; Ralston et al. (2017) 4-6 підходів ≈ 7+ за гіпертрофією; Rhea et al. (2003) 4 підходи/2-3 дні оптимально для intermediate; MEV vs MED vs MAV (Israetel); junk volume і відновлювальна ємність; ітераційний протокол: старт 3 підходи → прогрес → + 1 підхід; 12 хв читання; editorial-brutalist; clinical-safety NOT REQUIRED
- `blog.html` — +3 картки (post-380/381/382); 321 → 324 cards total
- `README.md` — post-380/381/382 позначено як виконані (v3.22.0)
- `STATUS.md` — версія оновлена до v3.22.0; 379 → 382 posts; 321 → 324 blog cards
- `VERSION` — 3.21.1 → 3.22.0

---

## [3.21.1] — 2026-06-10

### Changed
- `enterprise.html` — +2 new sections: **Integrations ecosystem** (IdP grid: Okta/Azure AD/Google Workspace/OneLogin/any SAML-OIDC; wearables: HealthKit, Health Connect, Whoop, Oura, Garmin; SIEM: Datadog, Splunk, Sumo Logic, S3, webhook/REST) and **Data ownership & portability** (employee offboarding via SCIM, GDPR Art. 20 self-serve JSON export, 90-day wind-down window on contract end, 30-day tenant purge + deletion certificate); +3 FAQ items (which IdPs are supported, what happens on contract end, employee data export rights); 831 → 994 lines
- `VERSION` — 3.21.0 → 3.21.1

---

## [3.21.0] — 2026-06-10

### Added
- `content/post-379-fatigue-superposition.md` — Суперпозиція втоми: коли сума двох «легких» стресів стає одним важким; Marcora et al. (2009) когнітивне навантаження −18% витривалість при ідентичних VO₂/HR/лактаті; Pageaux & Lepers (2018) мета-аналіз 22 РКД; Halperin et al. (2015) психологічний стрес і RPE-дрейф; McEwen & Stellar (1993) алостатичне навантаження; Meeusen et al. (2013) NFOR за підвищеного загального стресу; Sapolsky (2004) HPA-недиференційована відповідь; практика паралельного логу і реактивного деload; 13 хв читання; editorial-brutalist; clinical-safety PASS
- `blog.html` — +1 картка post-379 (321 cards total)
- `README.md` — post-379 позначено як виконаний; додано roadmap post-383–386
- `STATUS.md` — версія оновлена до v3.21.0; 379 posts; blog.html 321 cards
- `VERSION` — 3.20.0 → 3.21.0

---

## [3.20.0] — 2026-06-09

### Added
- `content/post-378-how-to-read-study-5-min.md` — Як перевірити тренувальне дослідження за 5 хвилин: мінімальний фільтр якості; п'ять фільтрів (вибірка, тривалість, n і статистична потужність, ефект-розмір, екологічна валідність); Cohen (1988) statistical power; Schoenfeld (2017) різні криві дозо-відповіді у тренованих/нетренованих; publication bias і industry-sponsored research; практичний 5-хвилинний маршрут по статті; 13 хв читання; editorial-brutalist; clinical-safety NOT REQUIRED
- `blog.html` — +1 картка post-378 (320 cards total)
- `README.md` — post-378 позначено як виконаний; додано roadmap post-380, 381, 382
- `STATUS.md` — версія оновлена до v3.20.0; 378 posts; blog.html 320 cards
- `VERSION` — 3.19.0 → 3.20.0

---

## [3.19.0] — 2026-06-09

### Added
- `content/post-375-adaptation-vs-result.md` — Адаптація vs результат: Selye GAS, Damas et al. (2016) SMP lag, Moritani & deVries нейральна фаза, Cholewa (2017) дискордантність, Seaborne (2018) епігенетична пам'ять, 5 практичних маркерів адаптації без дзеркала. Clinical-safety NOT REQUIRED. ~12 хв читання.
- `content/post-376-shift-workers-programming.md` — Програмування для двозмінників: Ralston et al. (2017) об'єм > частота, Bickel (2011) поріг збереження 1/9, A/B/C сесійний тиринг, обробка пауз 48–96 год і 5+ днів, деловантаження за станом. Clinical-safety NOT REQUIRED. ~11 хв читання.
- `content/post-377-listen-to-your-body-myth.md` — Чому «прислухайся до тіла» не є порадою: Noakes (2011) центральний губернатор, Morgan (1994) упередженість настрою, Buchheit (2014) HRV-тренд vs одиночне читання, Halperin (2015) RPE-дискордантність. Операційна заміна з порогами. Clinical-safety NOT REQUIRED. ~12 хв читання.

### Changed
- `blog.html` — +3 картки (post-375/376/377); лічильники: 296 → 319 статей, 3 848 → 3 883 хв читання, hero label оновлено.
- `VERSION` — 3.18.1 → 3.19.0

---

## [3.18.1] — 2026-06-09

### Changed
- `admin-dashboard.html` — SIEM Export screen added (v3.18.1). New sidebar nav item "⇆ SIEM Export". Full `screen-siem` panel: four SLO health cards (SIEM-SLO-01 push P95 18 ms / limit 60 s, SIEM-SLO-02 pull availability 99.97% / <5 min downtime, rolling 30 d event counter 847,293 / 1,000,000 enterprise quota, SIEM-SLO-04 HMAC chain INTACT); webhook configuration form (destination type selector — Splunk HEC / Microsoft Sentinel / Datadog / IBM QRadar LEEF / Generic webhook; endpoint URL; masked secret with reveal/copy/rotate; pull API endpoint + API key); filter rules table with include/exclude rules, `filter_compliance_approved` compliance gate (⚠ pending badge for admin/anomaly exclusion filters requiring CSM approval before activation); privacy floor disclosure (user_id absent, user_email SHA-256 hashed, ip_address /24, workout/CV/body fields removed — non-bypassable); recent push deliveries table (batch size, P95 latency, HTTP status, HMAC prefix, delivery status). Audit log "SIEM webhook" button wired to navigate to SIEM screen via `navigateTo()` helper. References: `docs/OBSERVABILITY.md §27`, DEC-030, `docs/AUDIT_LOG_SCHEMA.md`.
- `VERSION` — 3.18.0 → 3.18.1

---

## [3.18.0] — 2026-06-09

### Added
- `content/post-374-viral-protocols-anatomy.md` — Чому тренувальні протоколи з соцмереж не працюють — анатомія вірусної некомпетентності; шість структурних причин розриву між вірусністю і ефективністю; SAID-принцип і специфічність; Dankel et al. (2017) — прогресивне перевантаження як обов'язкова умова; Schoenfeld et al. (2019) — діапазон відповіді на гіпертрофію; Kreher & Schwartz (2012) — overtraining vs дисбаланс; практичний фільтр з 5 питань для оцінки будь-якого протоколу; 13 хв читання; editorial-brutalist; clinical-safety NOT REQUIRED
- `blog.html` — +1 картка post-374 (321 cards total)
- `README.md` — post-374 позначено як виконаний; додано roadmap post-377, 378, 379; лічильник постів оновлено до 374
- `STATUS.md` — версія оновлена до v3.18.0; 374 posts; content engine row добавлено
- `VERSION` — 3.17.0 → 3.18.0

---

## [3.17.0] — 2026-06-09

### Added
- `content/post-372-thermal-stress-training.md` — Тренінг в умовах термального стресу: жара і холод; фізіологія акліматизації (плазматичний об'єм, потовиділення, ядрова температура); Lorenzo et al. (2010) — теплова акліматизація покращує перформанс у помірних умовах; Roberts et al. (2015) — CWI знижує гіпертрофійні адаптації; стоп-сигнали теплового удару; clinical-safety PASS
- `content/post-373-training-age-vs-chronological-age.md` — Тренувальний стаж проти хронологічного віку після 40; Wroblewski et al. (2011) — МРТ майстер-велосипедистів; Harridge & Lazarus (2017); Lexell et al. (1988) — саркопенія; практичні рекомендації для програмування залежно від стажу і хронологічного віку; clinical-safety NOT REQUIRED
- `blog.html` — +2 картки post-372 і post-373 (320 cards total)
- `README.md` — оновлено статус post-372 і post-373 як виконаних; ROADMAP зсунуто до post-374+
- `STATUS.md` — версія оновлена до v3.17.0; 373 posts; 320 cards
- `VERSION` — 3.16.1 → 3.17.0

---

## [3.16.1] — 2026-06-09

### Changed
- `docs/INCIDENT_RESPONSE.md` v1.8 — §2.2.1 Phase 0 Permanent On-Call: Compensating Control Statement added. Closes CC2-GAP-005 documentation action: formal auditor-facing acknowledgement that Phase 0 is a solo-founder permanent on-call arrangement (not a rotation); four compensating controls documented (PagerDuty 24/7 critical alerts, 4-tier escalation path T+0/T+3/T+8/T+15, Better Uptime independent failsafe, HMAC audit chain dead-man's switch); Phase 1 trigger checklist + rotation schedule placeholder in place. Document header corrected from v1.5 → v1.8. References: `docs/AUDIT_LOG_SCHEMA.md`, DEC-030, CC2-GAP-005.
- `docs/SOC2_READINESS.md` — CC2-GAP-005 gap row updated: 🔴 open → 🟡 Partial (compensating control documented; CC2-E-004 screenshot pending implementation). Checklist row marked ✅ for documentation action. CC2.2 readiness delta updated.
- `VERSION` — 3.16.0 → 3.16.1

---

## [3.16.0] — 2026-06-09

### Added
- [`content/post-371-microbiome-training.md`](content/post-371-microbiome-training.md) — Мікробіом і тренування: Clarke et al. (2014, Gut) Akkermansia muciniphila у атлетів, Allen et al. (2018) зміна мікробіому незалежно від дієти і зворотність ефекту, Scheiman et al. (2019, Nature Medicine) лактат→Veillonella→пропіонат цикл, exercise-induced endotoxemia при перетренуванні, 3 практичних висновки; clinical-safety PASS (без харчових рекомендацій, без контенту про вагу тіла)
- `blog.html` — три відсутні картки для posts 369–371 додані до індексу

### Changed
- `README.md` — post-371 позначено як виконано (v3.16.0), додано roadmap post-372–376
- `VERSION` — 3.15.0 → 3.16.0

---

## [3.15.0] — 2026-06-09

### Added
- [`content/post-369-neural-efficiency-vs-hypertrophy.md`](content/post-369-neural-efficiency-vs-hypertrophy.md) — Нейральна ефективність vs гіпертрофія: Moritani & deVries (1979) EMG, Folland & Williams (2007) хронологія, практичний діагноз; clinical-safety NOT REQUIRED
- [`content/post-370-caffeine-training.md`](content/post-370-caffeine-training.md) — Кофеїн і тренування: Astorino & Roberson (2010) мета-аналіз, механізм аденозинової блокади, толерантність і wash-out, 3–6 мг/кг дозування, CYP1A2 варіабельність; clinical-safety NOT REQUIRED

### Changed
- `README.md` — roadmap секція ROADMAP (post-365+): оновлено статуси posts 365–368 (фактичні теми замість планованих), додано posts 369–370 як виконані, встановлено roadmap post-371+
- `README.md` — чекліст posts 360, 361 позначено як [x] з посиланням на фактичні файли (post-369, post-370)

---

## [3.14.1] — 2026-06-09

### Changed
- `docs/SOC2_READINESS.md` v3.5.3 — **PRE-01 / P-GAP-001 stale-status patch**: aligns 9 gap-register locations with DEC-037 (`privacy.html` created 2026-06-09). PRE-01 pre-observation checklist (§15.2): 🔴 Open → 🟡 Authored. First-Year Priority table (§15.3): 🔴 P1 → 🟡 P1 with remaining steps. CC2-GAP-002 description + checklist (§30.3/§30.4): note `privacy.html` authored, update priority to 🟡 P0 partially mitigated. PRV-01 (§35.3): 🔴 Gap — P-GAP-001 → 🟡 Authored. PRV-03 (§35.3): 🔴 Gap — P-GAP-003 → 🟡 Authored (§74 `form-consent-gate`). PRV-04 (§35.3): 🔴 Gap → 🟡 Partial. P-GAP-001 (§35.11): priority P0 pre-launch blocker → P1 🟡 Authored; remediation ✅ authored + Remaining. P-GAP-003 (§35.11): priority P1 → P1 🟡 Authored; remediation ✅ §74 + Remaining deployment. Doc header v3.3 → v3.5.3. Owner: compliance-officer (DEC-037 reference: `docs/DECISION_LOG.md`).
- `VERSION` — 3.14.0 → 3.14.1

---

## [3.14.0] — 2026-06-09

### Added
- [`content/post-368-exercise-order-training-session.md`](content/post-368-exercise-order-training-session.md) — «Порядок вправ у тренуванні — що наука каже насправді» — ~13 хв читання. Spreuwenberg et al. (2006) J Strength Cond Res: присідання 3-ю вправою — менше повторень, вищий RPE; Dias et al. (2010): 12-тижневий цикл, м'язи в кінці сесії ростуть менше; Simão et al. (2012) систематичний огляд; Miranda et al. (2010) верхня частина тіла; принцип пріоритету; pre-exhaustion — умови виправданості; три сценарії (сила / гіпертрофія / загальна підготовка); Ratamess et al. (2009) NSCA. clinical-safety NOT REQUIRED.
- `blog.html` — картка post-368. Загальна кількість карток: 315.

### Changed
- `VERSION` — 3.13.0 → 3.14.0

---

## [3.13.0] — 2026-06-09

### Added
- [`content/post-365-neuroplasticity-strength-training.md`](content/post-365-neuroplasticity-strength-training.md) — «Нейропластичність і силовий тренінг: чому техніка — це нейральна інвестиція» — ~12 хв читання. Pascual-Leone et al. (1995 PNAS) M1 expansion; Kleim et al. (2002 J Neurosci) синаптогенез після рухового навчання; Pearce et al. (2013 Eur J Neurosci) SICI/ICF кортикоспінальні зміни після 4 тижнів силового тренінгу; Carroll et al. (2006) cross-education ~8% transfer; техніка як нейральна інвестиція. clinical-safety NOT REQUIRED.
- [`content/post-366-grip-strength-anatomy-training.md`](content/post-366-grip-strength-anatomy-training.md) — «Сила хвату: функціональна анатомія і тренінг» — ~11 хв читання. Три механізми хвату (crush/pinch/support); FDS vs FDP анатомія; Leong et al. (2015 Lancet) PURE study 139 703 осіб: −5 кг grip = +17% смертність; Bohannon (2019 JSCR) grip як longevity proxy; ремінці на становій — аналіз; 12-тижневий протокол. clinical-safety NOT REQUIRED.
- [`content/post-367-minimum-maintenance-dose.md`](content/post-367-minimum-maintenance-dose.md) — «Мінімальна підтримуюча доза: скільки тренінгу достатньо щоб не втрачати» — ~11 хв читання. Bickel et al. (2011): 1/9 обсягу при збереженні інтенсивності — 100% сили за 8 місяців; Mujika & Padilla (2001) ієрархія деадаптації; Bruusgaard (2010 PNAS) міоядерна персистенція; практичний формат підтримуючого тижня. clinical-safety NOT REQUIRED.
- `blog.html` — 3 нові картки: post-365, 366, 367. Загальна кількість карток: 314.
- `README.md` — post-186, 187, 336 відмічено як [x] (написані як post-365/366/367).

### Changed
- `VERSION` — 3.12.1 → 3.13.0

---

## [3.12.1] — 2026-06-09

### Changed
- `docs/ENTERPRISE_ONBOARDING.md` v0.2 — §11.1 Contract termination timeline: API key revocation step added at Day 30 (MDD-P1-04). All active `tenant_api_keys` rows for the departing tenant are wiped after SCIM token revocation; `api_key.revoked` (reason: `'tenant_offboarding'`) emitted per active key (DEC-030 HIGH, 7-year retention, HMAC-chained in insertion order). §11.2 DEC-030 event table updated with `api_key.revoked` row. SOC 2 CC6.4 credential lifecycle evidence.
- `docs/AUDIT_LOG_SCHEMA.md` — `api_key.revoked` reason enum extended: `'tenant_offboarding'` added alongside existing `'manual'|'expiry_overlap'|'compromise'` values; inline note links to `ENTERPRISE_ONBOARDING.md §11.1` for bulk-revocation sequence.
- `docs/SOC2_READINESS.md` — MDD-P1-04 marked [x] Closed 2026-06-09 in the data model gap register (§22.11).
- `VERSION` — 3.12.0 → 3.12.1

---

## [3.12.0] — 2026-06-09

### Added
- [`content/post-364-performance-variability-daily.md`](content/post-364-performance-variability-daily.md) — «Чому сьогодні важче: варіативність тренувальних показників і що вона каже» — ~230 рядків, ~13 хв читання. Охоплює: добова варіативність силових показників ±4–8% (Atkinson & Reilly 1996); три джерела — циркадна ритміка, накопичена тижнева втома, нейром'язовий прайминг; сигнал vs шум (один поганий день = шум, 3-сесійний тренд = сигнал); авторегуляція RPE в межах програми (Zourdos 2016, Helms 2016); п'ять діагностичних питань для «важкого дня»; циркадна адаптація до часу тренування (Drust 2005). clinical-safety PASS.
- `blog.html` — нова картка: post-364 (Варіативність показників). Загальна кількість карток: 311.
- `README.md` — пост-364 додано до roadmap, нові теми post-365–368 додано до роадмапу.

### Changed
- `VERSION` — 3.11.0 → 3.12.0

---

## [3.11.0] — 2026-06-09

### Added
- [`content/post-362-how-to-read-research.md`](content/post-362-how-to-read-research.md) — «Ієрархія доказів для практика: як читати дослідження про тренування» — ~198 рядків, ~11 хв читання. Охоплює: ієрархія доказів (анекдот → мета-аналіз) і чому RCT у силовому тренінгу мають структурні обмеження; Ralston (2017) частота vs обсяг як зразок контрольованого RCT; effect size Cohen's d (d=0.2 vs d=0.79) vs p-value; статистична vs практична значущість («0.3 кг lean mass за 12 тижнів, p<0.05»); Schoenfeld як зразок методологічного читача (PRISMA, I², funnel plots) разом з Fisher і Krieger як додаткові приклади; три питання до будь-якого дослідження; 5–8 якісних систематичних оглядів як особиста база знань. clinical-safety NOT REQUIRED.
- [`content/post-363-individual-response-variability.md`](content/post-363-individual-response-variability.md) — «Індивідуальна варіативність відповіді на тренування: чому однакова програма дає різні результати» — ~185 рядків, ~11 хв читання. Охоплює: Hubal et al. (2005) — 585 піддослідних, 0–250% розкид силового прогресу при ідентичному протоколі; Timmons (2010) 11-генний профіль VO₂max і межі його перенесення на силовий тренінг (мультигенна складність); структурні джерела варіативності — архітектура м'яза (Erskine et al. 2010), механічна перевага (Blazevich), гормональне середовище, нейральна ефективність, adherence як методологічний конфаундер; що НЕ є причиною; мінімальний горизонт гіпертрофічного сигналу 12–16 тижнів; порівняння з собою як єдиний релевантний benchmark; персоналізація AI-коучингу через індивідуальний відгук у часі. clinical-safety NOT REQUIRED.
- `blog.html` — чотири нові картки: post-360 (MEV/MAV/MRV), post-361 (Креатин), post-362 (Ієрархія доказів), post-363 (Індивідуальна варіативність). Загальна кількість карток: 310.
- `README.md` — post-362 і post-363 позначені виконаними у roadmap.

### Changed
- `VERSION` — 3.10.0 → 3.11.0

---

## [3.10.0] — 2026-06-09

### Added
- [`docs/MSA_TEMPLATE.md`](docs/MSA_TEMPLATE.md) — Master Service Agreement template v0.1. 19-article commercial contract governing all enterprise customer relationships. Covers: license grant and restrictions, Privacy Floor (§8, non-negotiable, 8 specific restrictions), Acceptable Use / prohibited use-cases (§9, insurance risk-scoring, gov backdoors, wellness-as-punishment), CUEC obligations (§10), data export and deletion windows (30-day non-renewal / 14-day early termination), Art. 9 health data zero-grace-period deletion (DEC-036), AI training prohibition (§5.3, aligns with Anthropic enterprise DPA), 72h GDPR breach notification (§16), white-label conditionality (≥$50k ARR), CCPA SPA addendum path for US customers. Exhibits: Exhibit A (Order Form), Exhibit B (→ `docs/ENTERPRISE_SLA.md`), Exhibit C (→ `docs/SUBPROCESSORS.md §5` DPA), Exhibit D (TOMs summary), Exhibit E (AUP). References: `docs/ENTERPRISE.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/ENTERPRISE_SLA.md`, `docs/SUBPROCESSORS.md`. Pre-legal-review draft — outside counsel review required before first execution. Closes gap identified in `docs/ENTERPRISE_ONBOARDING.md` ("MSA counter-signed by both parties" step) and `docs/GDPR_DPIA.md §10.2.3` ("DPA template for enterprise customers — controller-to-controller addendum, Month 3").

### Changed
- `VERSION` — 3.9.0 → 3.10.0

---

## [3.9.0] — 2026-06-09

### Added
- [`privacy.html`](privacy.html) — `form.coach/privacy` URL target; presents all 14 sections of `docs/PRIVACY_POLICY.md` v0.1.0-draft in FORM brand style (dark theme, editorial-brutalist); prominent draft/pending-counsel-review banner; sticky TOC sidebar; sub-processor table; data retention schedule; full DSAR procedure; contact cards; mobile-responsive. Advances SOC 2 PRE-01 / P-GAP-001 from 🔴 Open → 🟡 Authored. Closes to 🟢 on counsel sign-off + deployment. (DEC-037)
- [`content/post-360-mev-mav-mrv-volume-landmarks.md`](content/post-360-mev-mav-mrv-volume-landmarks.md) — «MEV, MAV, MRV: що це таке, звідки взялись і як це використовувати» — 243 рядки, ~11 хв; volume landmarks for hypertrophy; cites Schoenfeld et al. 2017, Krieger 2010, Israetel RP framework, Meeusen et al. 2013, Heaselwood et al. 2022; population estimates table with explicit caveat; 5-step personal landmark protocol; decision matrix; clinical-safety REVIEWED PASS (programming methodology, no food/body-image/injury concerns)
- [`content/post-361-creatine-mechanism-practical-use.md`](content/post-361-creatine-mechanism-practical-use.md) — «Креатин — механізм дії та практичне використання (без маркетингу)» — 238 рядків, ~12 хв; PCr energy system; ISSN position stand (Kreider et al. 2017); Lanhers et al. 2017 meta-analysis; Chilibeck et al. 2017; loading vs. maintenance; non-responder data; kidney disclaimer; under-18 note; hydration protocol; clinical-safety REVIEWED PASS з умовами (kidney + under-18 disclaimers mandatory)
- `docs/DECISION_LOG.md` — DEC-037: privacy.html authored as form.coach/privacy URL target; PRE-01 advances to 🟡 Authored

### Changed
- `VERSION` — 3.8.2 → 3.9.0

---

## [3.8.2] — 2026-06-09

### Fixed
- `docs/SOC2_READINESS.md §3 Gap Analysis Summary` — **PRE-16 stale reference closure**: Gap Analysis Summary table (written at doc v1.3) still listed PRE-16 as the sole 🔴 Critical gap blocking SOC 2 observation period start, and the pre-audit readiness checklist (§38.8 / pre-audit table) still showed PRE-16 as 🔴 Open — both inconsistent with §74 (authored 2026-06-09, v3.5) which explicitly closed PRE-16 to 🟡 Authored. Four-point patch: (1) 🔴 Critical count 1 → 0; (2) 🟡 Partial count 33 → 34 with PRE-16 added to examples; (3) readiness score updated ~71% → ~98.3% and doc version v1.3 → v3.5; (4) PRE-16 pre-audit checklist row updated 🔴 Open → 🟡 Authored (`form-consent-gate` Worker + `consent_records` DDL — §74; 2026-06-09).

### Changed
- `VERSION` — 3.8.1 → 3.8.2

---

## [3.8.1] — 2026-06-09

### Added
- `content/post-359-heat-stress-training.md` — **Content engine post-359**: Тренування в умовах теплового стресу — cardiovascular drift, thermoregulatory threshold, WBGT-матриця, протоколи акліматизації. ~13 хв читання. Охоплює: серцево-судинний дрейф (Ekelund 1967), центральна теплова втома (Nybo & Nielsen 2001 J Physiol), WBGT операційні пороги, теплова акліматизація O'Brien et al. (2014 Sports Med): +7% VO₂max, +12% робочої ємності, Ely et al. (2010 J Appl Physiol), циркадні ефекти Chtourou & Souissi (2012 BJSM), пасивна акліматизація через сауну, силовий vs аеробний тренінг у спеку, матриця рішень з MD-referral. clinical-safety REVIEWED PASS.
- `blog.html` — чотири нові картки: post-356 (Частота заміни вправ), post-357 (Темп прогресу ліфтів), post-358 (Самопрограмування vs шаблон), post-359 (Тепловий стрес). Загальна кількість карток: 303. Оновлено версію до v3.8.1.
- `README.md §Content engine roadmap` — post-359 позначено виконаним; додано нові теми post-361–363 (кофеїн і тренування, читання досліджень, індивідуальна варіативність відповіді).

### Changed
- `STATUS.md` — blog.html: 299 → 303 картки, 355 → 359 постів authored; TL;DR: 348 → 359 evidence-based posts; blog.html версія → v3.8.1.
- `VERSION` — 3.8.0 → 3.8.1

---

## [3.8.0] — 2026-06-09

### Added
- `content/post-356-exercise-rotation-frequency.md` — **Content engine post-356**: Частота заміни вправ — коли перемикатись і коли це зрада прогресу. ~13 хв читання. Охоплює: SRA-крива для технічного навчання (Fitts & Posner motor learning phases), Fonseca et al. (2014) exercise variety і hypertrophy, Vigotsky & Contreras length-tension logic, Carroll et al. (2006) cross-education, матриця anchor/accessory/variation по стажу. clinical-safety NOT REQUIRED.
- `content/post-357-progress-rate-different-lifts.md` — **Content engine post-357**: Темп прогресу у різних ліфтах — чому жим лежачи прогресує повільніше за присід. ~12 хв читання. Охоплює: lever arm mechanics, Zatsiorsky & Kraemer muscle mass principle, Häkkinen et al. (1993) hormonal responses, structural/anthropometric constraints, realistic monthly progress figures, 5-step bench diagnostic. clinical-safety NOT REQUIRED.
- `content/post-358-self-programming-vs-template.md` — **Content engine post-358**: Самопрограмування vs готова програма — матриця вибору по стажу і цілях. ~14 хв читання. Охоплює: training-age threshold (Helms 3-5yr), Baumeister ego depletion, 4 systematic self-programming error modes, minimum viable program, hybrid autoregulation model (Seiler/Kjerland, Plisk/Stone). clinical-safety NOT REQUIRED.
- `docs/DECISION_LOG.md` DEC-035 — API key expiry: soft enforcement (age alerts + CSM escalation) vs hard revocation. Provisional — revisit at SOC 2 observation period start. Resolves OQ-SSO-26.2.
- `docs/DECISION_LOG.md` DEC-036 — Art. 9 health data no-grace period as absolute invariant during enterprise off-boarding. Resolves OQ-OFB-03. Required before first enterprise contract signing.
- `docs/DATA_MODEL.md` — OQ-OFB-03 і OQ-SSO-26.2 відмічені як 🟢 Resolved з посиланнями на DEC-036 і DEC-035 відповідно.

---

## [3.7.3] — 2026-06-09

### Added
- `content/post-355-paused-reps-dead-stop.md` — **Content engine post-355**: Паузовані повторення і мертва зупинка — виключення SSC як тренувальний інструмент. ~13 хв читання. Охоплює: пружна енергія і SSC (Lichtwark & Wilson 2005), sticking point діагностика, dead-stop vs T&G дедліфт, паузований жим і присід, сухожильна адаптація (Seynnes 2009), матриця застосувань. clinical-safety NOT REQUIRED.
- `blog.html` — нова картка для post-355 (cat-technique, 299 карток).
- `README.md §Content engine roadmap` — post-355 позначено виконаним; додано нові теми post-356–360 (частота заміни вправ, темп прогресу ліфтів, самопрограмування vs template, тренування в умовах теплового стресу, нейральна ефективність vs гіпертрофія).

### Changed
- `VERSION` — 3.7.2 → 3.7.3

---

## [3.7.2] — 2026-06-09

### Fixed
- `docs/DATA_MODEL.md §3.8` — **SOC 2 OQ-P2-03 closure**: виявлено і виправлено критичну помилку дизайну RLS для `consent_records`. Оригінальна політика `consent_records_tenant_isolation` була задекларована як `PERMISSIVE FOR SELECT` з перевіркою лише `tenant_id` — у Postgres PERMISSIVE-політики OR-яться, що дозволило б tenant-адмінам читати записи згоди всіх співробітників тенанту. Виправлення: політика перейменована в `consent_records_cross_tenant_block` і задекларована як `RESTRICTIVE` — AND-логіка з PERMISSIVE `consent_records_user_read` (перевірка `user_id`) гарантує, що кожен бачить лише власний рядок. Додано CI-тест: tenant admin → 0 рядків для чужого `user_id`.

### Added
- `docs/DATA_MODEL.md §2.11` — `consent_records` table DDL (migration `0037_consent_records.sql`): append-only таблиця з RULE-захистом від UPDATE/DELETE; privacy invariant (без email/IP/Art.9-значень); посилання на SOC 2 критерії P2.1/P8.1.
- `docs/DATA_MODEL.md §5` — Confidential-класифікація для `consent_records` (псевдонімні рішення про згоду; доступ лише власнику через §3.8 RLS; tenant-адмін заблокований).
- `docs/SOC2_READINESS.md §74.4 + §74.11` — оновлено опис tenant admin restriction та OQ-P2-03 → 🟢 Resolved; канонічна назва/тип політики тепер вказує на DATA_MODEL.md §3.8.

---

## [3.7.1] — 2026-06-09

### Changed
- `docs/SOC2_READINESS.md §74` — Privacy TSC Consent Management & Cookie Banner Operating Evidence · PRE-16 / P-GAP-004 Closure · P1.1/P2.1/P3.1/P8.1 Auditor Exhibit. Closes the single remaining 🔴 critical gap blocking SOC 2: PRE-16 (cookie banner / consent management) advances from 🔴 Critical Open → 🟡 Authored. Architecture: `form-consent-gate` Cloudflare Worker + Cloudflare KV + `consent_records` Supabase append-only table (migration 0037). Three DEC-030 events: `privacy.consent_granted` HIGH/7yr, `privacy.consent_withdrawn` HIGH/7yr, `privacy.consent_version_bumped` STANDARD/3yr. P-CHAIN-01 chain monitor (consent event gap > 365 days → PagerDuty P2; pg_cron job 12). Six evidence artefacts PRE-74-E-001 through PRE-74-E-006. P-series coverage: ~70% → ~82%. Overall SOC 2 readiness: ~97.7% → ~98.3%. SOC 2 doc v3.4 → v3.5.
- `VERSION` — 3.7.0 → 3.7.1

---

## [3.7.0] — 2026-06-09

### Added
- `content/post-354-strength-training-mobility.md` — «Силовий тренінг і мобільність — чому розтяжка після тренування не будує мобільність»: Weppler і Magnusson (2010 PT) — більшість змін від статичного стречінгу є зміною больового порогу, а не структурою тканини; Magnusson et al. (1996 SJMSS) — жорсткість м'язово-сухожильних одиниць після стречінгу не змінювалась; Bloomquist et al. (2013 EJAP) — глибокий присід будує більшу дистальну гіпертрофію квадрицепса; McMahon et al. (2014 JAP) — тренінг у подовженій позиції збільшує функціональну довжину фасцикул; різниця гнучкість vs мобільність (пасивний vs активний діапазон); PAILs/RAILs (FRC Спіна) — нейром'язовий механізм end-range тренінгу; навантажений стречінг Jefferson curl, deep squat, Nordic curl; CARs як щоденне техобслуговування нейром'язових карт; 5 практичних принципів побудови мобільності паралельно з силою. Clinical-safety NOT REQUIRED. 13 хв читання.
- `blog.html` — нова картка post-354 (298 cards total)
- `README.md` — post-354 позначено [x]; дорожня карта post-355 збережена
- `STATUS.md` — лічильник постів (351 authored), blog.html до v3.7.0

### Changed
- `VERSION` — 3.6.0 → 3.7.0

---

## [3.6.0] — 2026-06-09

### Added
- `content/post-352-returning-from-long-break.md` — «Повернення після тривалої паузи — як не зробити більше шкоди, ніж сама пауза»: нейральна детренованість (1–2 тижні) vs структурна (місяці) — різна швидкість, різна тактика; Bickel et al. (2011 MSSE) maintenance MED — 1/3 підходів при збереженій інтенсивності зберігає адаптації 32 тижні; Seaborne et al. (2018 Scientific Reports) — міонуклеарна пам'ять: ядра зберігаються після детренованості, гени гіпертрофії деметильовані та готові до реактивації; Nosaka et al. (2002) — ексцентрична вразливість після паузи; матриця повернення за тривалістю (1–3 тижні / 4–8 тижнів / 3+ місяці); чому «надолужувати» контрпродуктивно; практична ознака завершення нейральної фази. Clinical-safety NOT REQUIRED. 11 хв читання.
- `content/post-353-training-with-chronic-sleep-deprivation.md` — «Тренування при хронічному недосипанні — що перестає працювати і що ще залишається»: механізм — NREM-пік GH, нічний кортизол, mTOR/PI3K·Akt пригнічення; Knowles et al. (2018 EJSS) ієрархія деградації — максимальна сила і RPE-сприйняття першими, об'ємна робота останньою; Blumert et al. (2007) RPE-зсув при депривації; Mendelson et al. вольовий ресурс; Spiegel/Gangwisch — кумулятивний борг та часткова компенсація; 5 практичних принципів: частота > об'єм, знижуй підходи/зберігай %, планування важких днів, RPE-корекція −1, не PR при дефіциті. Clinical-safety NOT REQUIRED. 10 хв читання.
- `blog.html` — дві нові картки post-352 і post-353 (297 cards total)
- `README.md` — post-352 і post-353 позначено [x]; дорожня карта post-354/355 збережена
- `STATUS.md` — лічильник постів (350 authored), blog.html до v3.6.0

### Changed
- `VERSION` — 3.5.1 → 3.6.0

---

## [3.5.1] — 2026-06-09

### Changed
- `docs/OBSERVABILITY.md` → v2.0 — §6.2 `c1_erasure_sla` subsection added: AL-C1-01 (GDPR Art. 17 erasure SLA day-33 warning, P1, PagerDuty `form-compliance` → compliance-officer, dedup `c1-erasure-sla-breach-{dsar_request_id}`, 24 h re-alert, SOC 2 C1.2/P5.2). Closes cross-reference from SOC2_READINESS.md §73.
- `docs/AUDIT_LOG_SCHEMA.md` → v0.8 — new `Compliance operating evidence events` section: 4 HMAC-chained DEC-030 events (`compliance.data_asset_inventory_reviewed` STANDARD/3yr, `compliance.encryption_verified` STANDARD/3yr, `compliance.erasure_sla_alert_fired` HIGH/7yr, `compliance.disposal_audit_completed` STANDARD/3yr). Retention table updated. Closes SOC2_READINESS.md §73 AUDIT_LOG_SCHEMA cross-reference.
- `VERSION` — 3.5.0 → 3.5.1

---

## [3.5.0] — 2026-06-09

### Added
- `content/post-351-reverse-pyramid-training.md` — «Зворотна піраміда — чому важкий підхід першим логічніший, ніж здається»: нейром'язова свіжість як обмежений ресурс; чому традиційна пряма піраміда витрачає нейральний драйв до максимального підходу; RPT-структура (важкий підхід 1 при RPE 8.5, back-off сети на 90% і 80%); авторегуляція через RPE/RIR; коли RPT доречний (базові рухи, проміжний стаж, обмежений час) і коли ні (технічно складні рухи, новачки, змагальний пауерліфтинг); практичні протоколи A (силовий) і Б (гіпертрофічний). Clinical-safety NOT REQUIRED. 13 хв читання.
- `blog.html` — картка post-351 (295 cards total)
- `README.md` — post-351 позначено [x]; додано roadmap post-352–355
- `STATUS.md` — оновлено лічильник постів (348 authored), blog.html до v3.5.0; запис post-351 у таблиці

### Changed
- `VERSION` — 3.4.0 → 3.5.0

---

## [3.4.0] — 2026-06-09

### Added
- `content/post-347-aerobic-base-strength-training.md` — «Кардіо не вбиває м'язи. Але низький VO2max вбиває твій об'єм»: Wilson та ін. (2012) мета-аналіз 21 дослідження — interference на гіпертрофію і силу <5%; Zone 2 як база аеробної ємності; Bogdanis та ін. (1995) — ресинтез PCr між підходами корелює з VO2max; Mitchell та ін. (2018) — aerobic capacity = capacity to recover; практична інтеграція (модальність / тривалість / частота / separation). Clinical-safety NOT REQUIRED. 9 хв читання.
- `content/post-350-self-managed-training-journal.md` — «Тренувальний журнал без додатків — чотири параметри, які замінюють все»: вправа+конфігурація / вага / повторення / RPE або RIR; шкала Tuchscherer/Zourdos (2016 JSCR) + Helms (2016 IJSPP); стагнація (RPE повзе вгору) vs плато (RPE стабільна, прогрес зупинився) — різні проблеми, різні рішення; шаблон тижня; аргументи за папір vs апп; ретроспектива кожні 4 тижні. Clinical-safety NOT REQUIRED. 9 хв читання.
- `blog.html` — дві нові картки post-347 і post-350 (294 cards total)
- `README.md` — post-347 і post-350 позначено [x]
- `STATUS.md` — оновлено лічильник постів (347 authored), blog.html до v3.4.0

### Changed
- `VERSION` — 3.3.2 → 3.4.0

---

## [3.3.2] — 2026-06-09

### Added
- `docs/SOC2_READINESS.md §73` — Confidentiality TSC (C1) Operating Evidence · C1-GAP-001 Advancement · NDA Lifecycle, Encryption-at-Rest Verification & Disposal Monitoring · C1.1/C1.2 Auditor Exhibit. Advances C1-GAP-001 🔴 Open → 🟡 Authored (records that `compliance/c1/nda-template.md` C1-NDA-001 v1.0 is committed). Specifies three C1 operating evidence streams: (1) NDA lifecycle monitoring via `personnel.nda_signed` DEC-030 chain + `compliance/ndas/nda-register.csv` quarterly extract; (2) Annual encryption-at-rest DDL verification + Supabase/Workers/R2 screenshot package; (3) AL-C1-01 erasure SLA alert (`c1-erasure-sla-monitor` pg_cron, 08:00 UTC daily, fires at 33-day breach, privacy-safe breach_count payload, job 11 in pg-cron-health-monitor). Four new DEC-030 events: `compliance.data_asset_inventory_reviewed` STANDARD/3yr, `compliance.encryption_verified` STANDARD/3yr, `compliance.erasure_sla_alert_fired` HIGH/7yr, `compliance.disposal_audit_completed` STANDARD/3yr. Three chain monitors C1-CHAIN-01/02/03. Five evidence artefacts PRE-73-E-001 through PRE-73-E-005. C TSC: ~88% → ~91%. Overall SOC 2 readiness: ~97.5% → ~97.7%. SOC 2 doc v3.3 → v3.4.

### Changed
- `VERSION` — 3.3.1 → 3.3.2

---

## [3.3.1] — 2026-06-09

### Added
- `content/post-345-training-while-traveling.md` — Тренування під час подорожей і відряджень: Issurin (2008) резидуальні ефекти (сила 25–35 днів), Bickel et al. (2011) maintenance MED, готельний протокол 40 хв, матриця рішень (1–3 / 4–7 / 14+ днів), RIR 1–2 як ключова умова. Clinical-safety NOT REQUIRED. 13 хв читання.
- `blog.html` — нова картка post-345 (292 cards total)
- `README.md` — post-345 позначено [x]; додано roadmap-теми post-346..350
- `STATUS.md` — оновлено лічильник постів (345), картка blog.html до v3.3.1

---

## [3.3.0] — 2026-06-08

### Added
- `docs/runbooks/RB-SUB-01.md` — P0 · Subscription state inconsistency: impossible `(from_state, to_state)` transition detected OR `users.subscription_tier` ≠ latest terminal `subscription_events` row. Steps: scope check, root-cause triage (RevenueCat/Stripe/manual_admin), append-only correction-event INSERT, HMAC chain verification. DEC-030 evidence checklist. SOC 2 CC7.2/PI1.2/PI1.5.
- `docs/runbooks/RB-SUB-02.md` — P0 · Grace period overshoot: user in `past_due`/`canceled` state with `valid_to < now() - 72h` retaining premium access. Steps: pg_cron grace-period-enforcer diagnostic, forced `billing.access_revoked` DEC-030 CRITICAL event, `users.subscription_tier` correction, Privacy Floor verification. SOC 2 CC7.2/PI1.5.
- `docs/runbooks/RB-SUB-03.md` — P1 · Webhook error rate elevated: `billing.webhook.processed{outcome="error"}` / total > 2% over 15-min rolling window. Steps: error-category triage (signature invalid / parse error / idempotency conflict / DB write failure), RevenueCat/Stripe status check, secret rotation gate, Cloudflare Analytics Engine queries. SOC 2 CC7.2/PI1.1.
- `docs/runbooks/RB-SUB-04.md` — P1 · **Enterprise seat drift**: `tenant_seats.seat_count` ≠ Stripe `subscription.quantity` for any `tenant_id` persisting > 5 min. Steps: per-tenant drift query, Stripe API seat-count fetch, compensating UPDATE with DEC-030 `billing.seat_drift_detected` CRITICAL event, SLA credit assessment against `docs/ENTERPRISE_SLA.md`, customer-success notification. No-go clause: insurance risk-scoring / gov backdoor contracts not applicable. SOC 2 CC7.2/A1.1/PI1.2.
- `docs/runbooks/RB-SUB-05.md` — P1 · Webhook delivery lag: `billing.webhook.lag_ms` P95 > 60,000 ms over 5-min window. Steps: RevenueCat/Stripe status check, Cloudflare Workers CPU/wall-clock diagnostic, DB write latency check, queue backlog assessment. SOC 2 CC7.2/PI1.1.
- `docs/runbooks/RB-SUB-06.md` — P2 · Elevated refund rate: refund events in trailing 7 days > 5% of renewal events. Steps: refund cohort query (no PII), abuse-pattern detection, fraud-review escalation, RevenueCat entitlement audit. SOC 2 CC7.2.
- `docs/runbooks/RB-SUB-07.md` — P2 · **Enterprise invoice overdue**: `enterprise_invoice_overdue` event for any `tenant_id` with `created_at < now() - 30 days`. Steps: overdue-tenant query, Stripe invoice status fetch, customer-success escalation ≤ 1h, contract-term review, DPA/billing contact outreach. No-go reconfirmation: insurance risk-scoring / gov backdoor / wellness-as-punishment contracts explicitly declined regardless of ARR. SOC 2 CC7.2/A1.1.
- `docs/runbooks/RB-SUB-08.md` — P3 · Subscription state orphan: `user_id` in `subscription_events` with no matching row in `users` table. Steps: orphan-count query, GDPR erasure check (account may have been deleted post-erasure-request — do NOT restore), cleanup-event INSERT with `actor_id`. SOC 2 CC7.2/PI1.5.

### Changed
- `VERSION` — 3.2.0 → 3.3.0

---

## [3.2.0] — 2026-06-08

### Added
- `content/post-344-postpartum-return-to-training.md` — Повернення до силового тренінгу після пологів: ремоделювання колагену тип III→I, поширеність DRA (Mota et al. 2015), IAP-координація тазового дна і TVA, RCOG/POGP (2021) клінічний консенсус як фреймворк для провайдерів, стоп-сигнали симптомів; (**clinical-safety PASS WITH CONDITIONS** 2026-06-08: обов'язковий disclaimer, без самодіагностики діастазу, таймлайни як питання до провайдера, PPD signpost)

### Changed
- `blog.html` — додано картку post-344; v2.95.0 → v2.96.0
- `STATUS.md` — content engine table: post-344 PASS WITH CONDITIONS
- `README.md` — post-344 marked `[x]`; post-345 roadmap entry added; last update → 8 червня 2026 · v3.2.0
- `VERSION` — 3.1.0 → 3.2.0

---

## [3.1.0] — 2026-06-08

### Added
- `content/post-339-exercise-selection-minimum-set.md` — Вибір вправ: мінімальний набір для покриття гіпертрофійного стимулу; шість патернів руху, Fonseca et al. (2014) варіативність вправ, нейральна ефективність і familiarity, матриця мінімального набору; (sports-scientist reviewed, clinical-safety NOT REQUIRED)
- `content/post-340-training-time-circadian-rhythm.md` — Час тренування і циркадна ритміка: ранок vs вечір; Chtourou & Souissi (2012) мета-аналіз 100+ досліджень, циркадний піковий час 15:00–20:00, акліматизація до ранкового тренінгу, практична перевага консистентності над оптимізацією; (sports-scientist reviewed, clinical-safety NOT REQUIRED)
- `content/post-341-technique-vs-volume-beginners.md` — Техніка vs обсяг для початківця: когнітивне навантаження в перші 6 місяців; Fitts & Posner (1967) cognitive/associative/autonomous stages, Wulf & Prinz (2001) attentional focus, субмаксимальна практика під час освоєння, bandwidth method, сигнали переходу до прогресивного навантаження; (sports-scientist reviewed, clinical-safety NOT REQUIRED)
- `content/post-342-chronic-low-back-pain-training.md` — Тренування при хронічному болі в попереку: механізми як освітній контекст, McGill (2007) spine stability, Steele et al. (2015) дослідницький контекст, межі самостійного рішення; (sports-scientist reviewed, **clinical-safety PASS WITH CONDITIONS** 2026-06-08: тільки для хронічного болю >3 міс з підтвердженим діагнозом, обов'язкові клінічні disclaimers, Steele — не домашній протокол)
- `content/post-343-mental-recovery-training.md` — Ментальне відновлення і тренінг: когнітивна втома як незалежний фактор; Marcora et al. (2009) EJAP 15.1% зниження витривалості, психобіологічна модель втоми, RPE drift при когнітивному навантаженні, Van Cutsem et al. (2017) систематичний огляд; (sports-scientist reviewed, clinical-safety NOT REQUIRED)

### Changed
- `README.md` — checklist posts 339–342 marked `[x]`; post-342 annotated with clinical-safety PASS WITH CONDITIONS date
- `VERSION` — 3.0.2 → 3.1.0

---

## [3.0.2] — 2026-06-08

### Changed
- `enterprise.html` — added FAQ section (6 accordion items: privacy floor, SSO/SCIM setup time, tenant isolation, employee opt-out, home-gym CV support, minimum commitment); FAQ CSS styles + `toggleFaq` JS; link to pricing-enterprise.html FAQ for billing/contract questions; `VERSION` 3.0.1 → 3.0.2

---

## [3.0.1] — 2026-06-08

### Added
- `content/post-338-solo-training-without-spotter.md` — Тренування без страхувальника: класифікація ризику по вправах, стратегія RIR як параметр управління ризиком, налаштування страхувальних упорів, roll of shame технічні обмеження, матриця вправа/ризик/рішення; Aasa et al. (2017) BJSM injury epidemiology, Helms et al. (2016) RIR efficacy, Zourdos et al. (2016) RPE scale; (sports-scientist reviewed, clinical-safety NOT REQUIRED)

### Changed
- `VERSION` — 3.0.0 → 3.0.1

---

## [3.0.0] — 2026-06-08

### Added
- `content/post-337-eccentric-phase-hypertrophy.md` — Ексцентрична фаза: де насправді відбувається гіпертрофія; titin як активний механічний елемент (Herzog et al. 2012), крива сила-швидкість (120–140% від концентричного 1RM), stretch-mediated hypertrophy (Pedrosa et al. 2022), механотрансдукція (McMahon et al. 2014), meta-analysis eccentric vs. concentric (Roig et al. 2009); (sports-scientist reviewed, clinical-safety NOT REQUIRED)

### Changed
- `VERSION` — 2.99.2 → 3.0.0

---

## [2.99.2] — 2026-06-08

### Added
- `content/post-336-supercompensation-theory.md` — Суперкомпенсація: чому відпочинок — це частина тренінгу, а не перерва в ньому; модель Яковлєва/Віру, фази адаптації, тайминг суперкомпенсаційного вікна (36–72 h нейральне, 4–7 днів структурне), наслідки тренінгу у фазі втоми vs. пропущеного вікна; Meeusen et al. (2013), Halson & Jeukendrup (2004), Stone et al. (2007); (sports-scientist reviewed, clinical-safety NOT REQUIRED)

### Changed
- `VERSION` — 2.99.1 → 2.99.2

---

## [2.99.1] — 2026-06-08

### Added
- `docs/SOC2_READINESS.md §72` — Processing Integrity Operating Evidence: PI-GAP-001 / PI-GAP-002 / PI-GAP-003 closure spec (🔴 Open → 🟡 Authored across all three). PI-GAP-001: `coaching-completeness-monitor` 5-min pg_cron watchdog + Cloudflare Worker that auto-escalates stuck `coaching_turns` rows ≥ 90 s from `processing` → `failed`; `coaching.completeness_timeout` HIGH DEC-030 per affected row; `coaching.completeness_check_passed` LOW DEC-030 heartbeat; PagerDuty P1 on detection; added as job 10 to pg-cron-health-monitor registry (§71.2.2). PI-GAP-002: `coaching.output_filter_applied` STANDARD DEC-030 event on every Victor coaching turn's clinical safety filter; PI1 Accuracy observability panel (4 metrics: pass_rate/block_rate/latency_p95/filter_version staleness) with critical thresholds → PagerDuty P1. PI-GAP-003: migration `0047_meal_log_not_null_macros.sql` (COALESCE backfill + NOT NULL on `protein_g`, `fat_g`, `carbohydrate_g` + `meal_log_macros_non_negative` CHECK); mobile UI validation gate; PI-GAP-003a opened (P2 — computed `calories_kcal` column). Six evidence artefacts PRE-72-E-001 through PRE-72-E-006; 13-item implementation checklist (5× P0, 5× P1, 3× P2). SOC 2 criteria: PI1.1/PI1.2/PI1.3. Owner: platform-engineer + compliance-officer

### Changed
- `VERSION` — 2.99.0 → 2.99.1

---

## [2.99.0] — 2026-06-08

### Added
- `docs/OBSERVABILITY.md §12.6` — pg_cron health monitoring job registry: canonical table of 9 production pg_cron jobs under `pg-cron-health-monitor` surveillance; freshness windows (1 h for `row-count-monitor`, 2 h for `audit-event-flush`, 26 h for all daily jobs); PagerDuty routing (P0 for audit-chain-daily-check / row-count-monitor / audit-event-flush; P1 for remaining 6); DEC-030 event table (`system.cron_job_stale` HIGH/7yr, `system.cron_health_check_passed` LOW/3yr, `system.cron_health_check_failed` HIGH/7yr); privacy invariant (no user_id/tenant_id); A1-GAP-004 deployment status; closes SOC2_READINESS.md §70.11 item 10 + §71.8 item 5 (both P0/P1 M5)

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` v0.7 — +3 `system.cron_*` DEC-030 events registered: `system.cron_job_stale` (HIGH, 7yr), `system.cron_health_check_passed` (LOW, 3yr), `system.cron_health_check_failed` (HIGH, 7yr); retention table updated (+3 rows); closes SOC2_READINESS.md §71.8 item 2 (P0 M5); cross-ref OBSERVABILITY §12.6
- `VERSION` — 2.98.2 → 2.99.0

---

## [2.98.2] — 2026-06-08

### Added
- `content/post-335-reading-training-research.md` — як читати тренувальне дослідження: п'ять фільтрів (population match, тривалість, effect size vs p-value, ecological validity, publication bias); Cohen (1992), Hecksteden et al. (2015), Ioannidis (2005); (sports-scientist, clinical-safety NOT REQUIRED)
- `README.md` — roadmap post-333/334/335 marked [x]; new planned posts 338–341 added
- `STATUS.md` — content engine rows for post-333, post-334, post-335

### Changed
- `VERSION` — 2.98.1 → 2.98.2

---

## [2.98.1] — 2026-06-08

### Added
- `docs/SOC2_READINESS.md §71` — Availability Trust Service Criteria A1.1/A1.2/A1.3: capacity management, infrastructure redundancy, DR test evidence; PRE-71-E-001–007; `pg-cron-health-monitor` Edge Function spec; A1-GAP-004/005 formally opened (compliance-officer + devops-lead)

---

## [2.98.0] — 2026-06-08

### Added
- `content/post-333-failure-types-resistance-training.md` — м'язова vs технічна відмова, RIR-матриця для базових/ізольованих вправ (sports-scientist, clinical-safety NOT REQUIRED)
- `content/post-334-training-splits-evidence.md` — full-body vs upper/lower vs PPL: порівняльна матриця і рекомендація для середньостажованого (sports-scientist, clinical-safety NOT REQUIRED)

---

## [2.97.1] — 2026-06-08

### Changed
- `docs/SOC2_READINESS.md §70` — DSAR Lifecycle Automation & 30-Day SLA Monitoring (P5.1/P5.2/P4.1/P4.3 Auditor Exhibit): closes P-GAP-003 and P-GAP-005/PRV-34 from 🔴/🟡 Gap → 🟡 Authored; `dsar_requests` table DDL (`migrations/0053`) with five request types, generated `sla_deadline`, `export_expires_at` 7-day window, per-export HKDF-derived AES-256-GCM encryption; pg_cron automation (`migrations/0054`): day-25 P1 + day-29 P0 + SLA-breach P0-critical PagerDuty alerts via `dsar-sla-relay` Edge Function; eight DEC-030 HMAC-chained `dsar.*` events (dsar.request_submitted/identity_verified/request_acknowledged/access_export_created/erasure_completed/request_fulfilled/sla_alert_fired/request_rejected, HIGH–CRITICAL 7yr); RLS privacy floor — `form_api` zero access (tenant admins cannot enumerate employee DSARs); export manifest.json schema for Art. 15(1)(a)–(h) completeness; eight evidence artefacts PRE-70-E-001–PRE-70-E-008; PRE-24 closure path documented; SOC 2 P5.1/P5.2/P4.1/P4.3/P8.1
- `VERSION` — 2.97.0 → 2.97.1

---

## [2.97.0] — 2026-06-08

### Added
- `content/post-332-dual-goal-strength-hypertrophy.md` — Сила і гіпертрофія одночасно: Schoenfeld et al. 2017 (JSCR) — схожа гіпертрофія 6–12 і 25–35 rep-range при вирівняному обсязі; Morton et al. 2016 (J Appl Physiol) — підтвердження; де конфлікт реальний (нейральна специфічність > 85% 1RM); DUP Rhea et al. 2002 (JSCR) +28% vs +14% лінійна за 12 тижнів; powerbuilding шаблон; матриця рішень по тренувальному стажу; клінічна безпека NOT REQUIRED
- `blog.html` — картки post-330, post-331, post-332 додані; загальна кількість карток ↑3
- `README.md` — post-330/331/332 відмічені [x]; roadmap розширений: post-333–337 (мобільність, тренінг після хвороби, functional training SAID, мінімальна підтримуюча доза, вікові адаптації)
- `STATUS.md` — рядки post-330/331/332 додані до content engine table

---

## [2.96.1] — 2026-06-08

### Added
- `docs/SOC2_READINESS.md §69` — Security Awareness Training Annual Execution Evidence & Completion Attestation Framework (CC1.4/CC2.2/CC1.3/CC1.2 Auditor Exhibit, 496 lines): design-vs-execution distinction vs §22; solo-founder signed attestation template (CC1-E-006); 7 evidence artefacts CC1-E-001–CC1-E-007 з naming convention, retention 7yr, filing paths; new-hire access gate (Day-0: 1Password/Supabase/GitHub withheld until CC1-E-007 filed); role-based matrix (Engineering all 8 topics / Operations 6 topics / Contractor-<30d 3 topics); annual phishing simulation programme (GoPhish quarterly, 20% click-rate threshold, 5-day remediation window); PRE-22 closure criteria; gap tracker: "Internal security training" 🔴 Gap → 🟡 Partial; closes to 🟢 on first evidence filing

---

## [2.96.0] — 2026-06-08

### Added
- `content/post-330-rpe-rir-autoregulation.md` — RPE та RIR: авторегуляція інтенсивності без зовнішніх приладів; Borg 1970 (Acta Physiol Scand) — чому кардіо-шкала не підходить для силового; Zourdos et al. 2016 (JSCR) — modified RPE 1–10, RPE↔RIR таблиця; Helms 2016 (IJSPP) — half-point точність; Gonzalez-Badillo 2017 (Br J Sports Med) — velocity як об'єктивний proxy; коли % б'ють RPE і навпаки; 3-кроковий протокол калібрації; clinical-safety NOT REQUIRED
- `content/post-331-hrv-self-coached-athlete.md` — HRV для самокерованого атлета: RMSSD vs SDNN, протокол вимірювання (ранок, supine, 5 хв відпочинку), Buchheit 2014 (Eur J Appl Physiol), Svensson 2020 (Alcohol) — вплив алкоголю, Kiviniemi 2010 (Scand J Med Sci Sports) — HRV-guided training; 3-зонова модель (>+5%/±5%/<−5% від rolling average); помилки вимірювання; інтеграція з FORM через Oura/Apple Health/Garmin; clinical-safety NOT REQUIRED

---

## [2.95.1] — 2026-06-08

### Changed
- `docs/SOC2_READINESS.md` — §68 US State Privacy Law Compliance (CCPA/CPRA & Multi-State) · OQ-PRIV-02 Resolution · P3.1/P5.2/P6.1/P6.7 Auditor Exhibit: applicability analysis for 6 US state privacy laws; FORM's Service Provider posture pre-$25M ARR; CPRA workforce data analysis (SPI classification for CV/biometric/health data); US Privacy SPA addendum architecture (7 CPRA § 1798.100(d) clauses, 5 Permitted Business Purposes, no-sale/no-share prohibition, subcontractor flow-down); 7 CPRA consumer rights implementation with 5-step DSAR cooperation protocol; multi-state harmonisation (VCDPA, CPA, CTDPA, UCPA, MOPIPA); AdTech isolation confirmation (PostHog EU-only, analytics_opt_out); 4 new DEC-030 HMAC-chained events (compliance.ccpa_dsar_received/completed/dsar_data_access/us_spa_signed, all HIGH or STANDARD / 7yr); 6 evidence artefacts USP-E-001–USP-E-006; 5 P0 tasks M6, 4 P1 tasks M8–M10, 3 P2 tasks; 3 open questions (OQ-CCPA-01 BIPA/CV, OQ-CCPA-02 unified addendum, OQ-CCPA-03 audit threshold); doc header v2.7 → v3.3; owner: compliance-officer + enterprise-architect
- `VERSION` — 2.95.0 → 2.95.1

---

## [2.95.0] — 2026-06-08

### Added
- `content/post-329-1rm-testing-protocol.md` — Тестування 1RM: рамп-протокол розминки, правило трьох спроб, вибір ваги, тест vs е1RM де формули помиляються, частота тестування по стажу, відновлення після MVC; Haff & Triplett (NSCA 2016), Zourdos et al. (2016), Linnamo et al. (1998), Bogdanis et al. (1995), Aagaard et al. (2002); clinical-safety NOT REQUIRED; owner: sports-scientist

### Changed
- `blog.html` — post-11/12/13/15/16/17 картки оновлено з "Незабаром" → "Читати →"; post-329 картка додана; лічильник: 291 карток (329 постів authored); v2.93.0 → v2.95.0
- `STATUS.md` — content engine table post-11–17 blog card статуси оновлено; post-329 додано; blog.html рядок оновлено; версія → v2.95.0
- `README.md` — post-329 додано до content roadmap
- `VERSION` — 2.94.0 → 2.95.0

---

## [2.94.0] — 2026-06-08

### Added
- `content/post-327-bilateral-deficit.md` — Білатеральний дефіцит: двосторонній MVC < сума унілатеральних (5–15%); міжлімбальна інгібіція через комісуральні інтернейрони і corpus callosum; Howard & Enoka (1991) J Appl Physiol, Koh et al. (1993) step-vs-ramp, Jakobi & Cafarelli (1998) контроверза; балістична фасилітація у тренованих атлетів; наслідки для тестування асиметрій (LSI), унілатерального програмування і оцінки нейронного потенціалу; clinical-safety NOT REQUIRED; owner: sports-scientist
- `content/post-328-aerobic-base-strength.md` — Аеробна база і силовий тренінг: PCr ресинтез — аеробний процес (Harris et al. 1976, τ = 3–5 хв); Greenhaff et al. (1994) креатин збільшує резервуар, не τ; Bogdanis et al. (1995 J Physiol) аеробний внесок у PCr між спринтами; серцевий резерв і ударний об'єм; MacDougall et al. (1982) мітохондрії у силових атлетів; Dolezal & Potteiger (1998) concurrent без значної інтерференції при правильній структурі; Buchheit & Laursen (2013) HIIT ефективність; мінімальна доза: 2×/тиж Zone 2 або 1×/тиж HIIT; клінічно безпечний контент; clinical-safety NOT REQUIRED; owner: sports-scientist

### Changed
- `README.md` — post-327 + post-328 додані до content roadmap
- `VERSION` — 2.93.0 → 2.94.0

---

## [2.93.0] — 2026-06-07

### Added
- `content/post-325-strength-training-bone-density.md` — Силовий тренінг і кісткова щільність: механотрансдукція (Duncan & Turner 1995), Frost mechanostat, Turner's 3 rules (1998), Guadalupe-Grau meta-analysis (2009), LIFTMOR trial Watson et al. (2018) 85% 1RM → +3.2% hip BMD / +2.9% lumbar, регіональна специфічність навантаження; clinical-safety NOT REQUIRED; owner: sports-scientist
- `content/post-326-sleep-training-adaptation.md` — Сон і тренувальна адаптація: SWS/GH cascade (Van Cauter 2000), sleep debt → тестостерон -10–15% (Leproult & Van Cauter 2010 JAMA), катаболічний дисбаланс (Dattilo 2011), -11% MVC за 24h депривації, 1.7× injury risk <8h (Milewski 2014), sleep extension +5% sprint / +9% free throw (Mah 2011); клінічно-безпечний фізіологічний контент; clinical-safety NOT REQUIRED; owner: sports-scientist

### Changed
- `README.md` — post-325 + post-326 позначені як ✅ у content roadmap
- `VERSION` — 2.92.1 → 2.93.0

---

## [2.92.1] — 2026-06-07

### Changed
- `docs/SOC2_READINESS.md` — §67 Tenant Data Deletion & Destruction Certificate (GDPR Art. 17 / CC6.5 / C1.2 / P5.2): closes OQ-MDD-02 (P1 before enterprise GA); 12-table deletion runbook in SERIALIZABLE transaction; pre-deletion 7-gate checklist; four DEC-030 HMAC-chained events at 10yr retention (tenant.data_deletion_initiated, .completed, tenant.r2_data_purged, tenant.destruction_certificate_issued); signed destruction certificate template (TDD-CERT-{YYYY}-{NNN}); five evidence artifacts (TDD-E-001…TDD-E-005); R2 prefix purge Worker spec; 13-item implementation checklist (P0/P1/P2); privacy invariant: no health values in event payloads; SOC 2 doc v3.1 → v3.2; owner: compliance-officer + enterprise-architect

### Changed
- `VERSION` — 2.92.0 → 2.92.1

---

## [2.92.0] — 2026-06-07

### Added
- `content/post-324-passive-mobility-constraints.md` — пасивні обмеження рухливості: три механізми (капсульне, фасціально-м'язове, нейральне), Weppler &amp; Magnusson (2010) stretch tolerance vs mechanical stiffness, Aquino et al. (2010) нейральне гальмування hamstrings +12–18° PNF vs +4–7° статика, Vicenzino et al. (2011) суглобова мобілізація &gt; стретчинг при капсульному обмеженні, Freitas et al. (2015) хронічна програма 12+ тижнів і реальна зміна жорсткості, Behm et al. (2016) дозування і силовий ефект, Morton et al. (2019) full ROM тренінг vs ізольований стретчинг, діагностичний алгоритм і протоколи для кожного механізму; owner: sports-scientist; clinical-safety: NOT REQUIRED

### Changed
- `blog.html` — додано 5 нових карток (post-320 вже була, додано 321–324): 284 картки всього
- `README.md` — roadmap: post-324 → [x], додано post-325; лічильник постів → 324; version footer → v2.92.0
- `STATUS.md` — blog.html row → 284 cards (324 posts authored) · v2.92.0
- `VERSION` — 2.91.0 → 2.92.0

---

## [2.91.0] — 2026-06-07

### Added
- `content/post-321-peak-strength-vs-strength-endurance.md` — пік сили vs силова витривалість: нейральний vs метаболічний примат, AMPK/mTORC1 інтерференція, Sale (1988) нейральна адаптація, Folland & Williams (2007) механізми, Spiering et al. (2008) метаболічні адаптації, Rhea et al. (2003) конвертація сили, блокова vs паралельна структура; owner: sports-scientist; clinical-safety: NOT REQUIRED
- `content/post-322-training-with-unstable-schedule.md` — тренінг з нестабільним розкладом: мінімальна ефективна доза (Ralston et al. 2017 meta-analysis), деренування Mujika & Padilla (2000) часові рамки, пріоритетна ієрархія рухів рівень 1-3, плаваюча частота за вікнами відновлення, full-body vs пріоритетний рух vs повторювана пара, авторегуляція AMRAP-тест; owner: sports-scientist; clinical-safety: NOT REQUIRED
- `content/post-323-tendon-stiffness-force-transmission.md` — жорсткість сухожиль і передача сили: механіка stiffness k=ΔF/ΔL, RFD детермінант Bohm et al. (2015), колагеновий синтез Kjaer et al. (2005), адаптація Magnusson & Kjaer (2003), кутова специфічність Arampatzis et al. (2007), HSR-протокол Kongsgaard et al. (2009), ізометрія Rio et al. (2015), SSC пружна енергія Farris & Sawicki (2012), практичне програмування; owner: sports-scientist; clinical-safety: NOT REQUIRED

### Changed
- `README.md` — roadmap: post-321/322/323 → [x], додано post-324; лічильник постів → 320; version footer → v2.91.0
- `VERSION` — 2.90.0 → 2.91.0

---

## [2.90.0] — 2026-06-07

### Added
- `docs/ENTERPRISE_ONBOARDING.md` — CSM playbook covering the full enterprise customer journey: pre-launch checklist (commercial, security review delivery, technical readiness), contract week kick-off, tenant provisioning with DEC-030 HMAC chain events (`tenant.created`, `tenant.sla_tier_set`, `admin.rbac_role_assigned`), SSO SAML/OIDC + SCIM setup, pilot rollout + employee announcement templates, admin dashboard training with privacy-floor script (HR never sees individual data), 30/60/90-day success metrics by tier (Starter/Growth/Enterprise), QBR agenda, seat expansion + white-label upgrade checklist + multi-year renewal, incident communication protocol, contract termination data offboarding timeline with hard-delete cascade per DATA_MODEL.md §12 and DEC-030 termination audit events, CSM handoff procedure, no-go use case escalation script; owner: customer-success + enterprise-architect + compliance-officer

### Changed
- `VERSION` — 2.89.0 → 2.90.0

---

## [2.89.0] — 2026-06-07

### Added
- `content/post-320-easy-feeling-vs-real-stimulus.md` — суб'єктивне сприйняття зусилля і реальний тренувальний стимул: Borg (1970/1982) RPE як інтегральний конструкт, Hampson et al. (2001) efference copy механізм, Eston (2012) RPE огляд, Fitts & Posner (1967) моторна автоматизація і когнітивне навантаження, Pageaux & Lepers (2018) когнітивна втома підвищує RPE, Zourdos et al. (2016) JSCR RIR r=0.78, Helms et al. (2016) IJSPP валідація, Smits et al. (2014) звикання перцепції, Roberts et al. (2018) residual fatigue і RPE Monday; два типи RPE-дрейфу, чотириступеневий діагностичний протокол; 320 рядків; clinical-safety NOT REQUIRED
- `blog.html` — додано картку post-320 (easy-feeling-vs-real-stimulus)

### Changed
- `README.md` — post-320 відмічено [x] у roadmap із розгорнутим описом референсів
- `STATUS.md` — TL;DR оновлено: 319 → 320 posts, blog.html рядок v2.89.0, 278 → 279 cards, version 2.88.0 → 2.89.0
- `VERSION` — 2.88.0 → 2.89.0

---

## [2.88.0] — 2026-06-07

### Added
- `content/post-318-adaptation-ceiling.md` — адаптаційна стеля і як зрозуміти, що ти до неї наближаєшся: принцип diminishing returns (Aaberg 2007, Heaselgrave et al. 2019 JSCR), три рівні тренувального стажу (Zatsiorsky & Kraemer 2006, Helms et al. 2018), адаптаційна стеля ≠ генетичний ліміт (Haugen et al. 2020, Haun et al. 2019 Frontiers), 4 операційних сигнали наближення, деловд як діагностичний інструмент (Roberts et al. 2018), rate of gain tracking; 264 рядки; clinical-safety NOT REQUIRED
- `content/post-319-isometric-training.md` — ізометричний тренінг: кутова специфічність ±15–20° (Lindh 1979, Kitai & Sale 1989), IMTP як золотий стандарт (Haff et al. 2015 JHUKIN, ICC > 0.97), паузовий тренінг і жорсткість сухожиль (Kubo et al. 2002), гіпертрофія при ізометрії (Oranchuk et al. 2019), patellar tendinopathy (Rio et al. 2015 BJSM), post-tetanic potentiation (Tillin & Bishop 2009), практичне програмування; 250 рядків; clinical-safety NOT REQUIRED
- `blog.html` — додано 2 картки: post-318 (adaptation-ceiling), post-319 (isometric-training)

### Changed
- `README.md` — post-318, post-319 відмічено [x] у roadmap; виправлено YMCA → IMTP у описі post-319
- `VERSION` — 2.87.0 → 2.88.0

---

## [2.87.0] — 2026-06-07

### Added
- `docs/CLINICAL_SAFETY.md` — post-generation clinical safety classifier spec for Victor AI coach: two-phase safety model (LLM prompt layer + async output classifier), 5 risk categories (CS-01 ED restriction, CS-02 purge/compensatory, CS-03 body shame, CS-04 self-harm/distress, CS-05 overtraining/injury), CRITICAL/HIGH severity levels + PagerDuty P1 wiring, full ED-flag mode spec (set/unset/invisibility principle), TypeScript implementation contract for `src/workers/coaching/safety-classifier.ts`, pattern registry amendment process (clinical-safety + compliance-officer dual approval), monthly review process + SOC 2 CC7.2 mapping; 541 lines; referenced by `docs/DATA_MODEL.md` §21.16 item 10 as P0

### Changed
- `VERSION` — 2.86.0 → 2.87.0

---

## [2.86.0] — 2026-06-07

### Added
- `content/post-317-eccentric-emphasis-no-equipment.md` — ексцентричний акцент без спеціального обладнання: Hortobágyi et al. (1996) +20–50% ексцентрична сила, тітін і сарокомерогенез (Nishikawa 2020, Franchi 2014), Roig et al. (2009) мета-аналіз, 4 практичних методи (tempo, 1.5-rep, AEL, Nordic curl), repeated bout effect, програмна інтеграція; 13 хв читання; clinical-safety NOT REQUIRED
- `blog.html` — додано картку post-317 (eccentric-emphasis-no-equipment)
- `README.md` — roadmap пост-317 відмічено [x], додано теми post-319–323

### Changed
- `STATUS.md` — TL;DR оновлено: 316 → 317 posts, blog.html рядок v2.86.0, 275 → 276 cards, version 2.83.0 → 2.86.0
- `README.md` — post count 296 → 317
- `VERSION` — 2.85.0 → 2.86.0

---

## [2.85.0] — 2026-06-07

### Added
- `content/post-315-novice-to-intermediate-transition.md` — де закінчується новачок: нейральна фаза (Sale 1988, Moritani & deVries 1979), 4 операційні сигнали переходу (LP stall 2-3×, RPE-дрейф, відновлення >72h, тренувальний стаж 6-18 міс), DUP vs LP (Rhea et al. 2003: +28.8% vs +14.4%), зміни у частоті і відновленні; 14 хв читання; clinical-safety NOT REQUIRED
- `content/post-316-training-response-variability.md` — варіативність відповіді на тренінг: HERITAGE Family Study (Bouchard 1999, 0–100% VO₂max), Hubal et al. (2005, -2% до +59% CSA), ACTN3 R577X / ACE I/D, non-responder міф (Hecksteden 2015), консистентність vs генетика (47/53%); 14 хв читання; clinical-safety NOT REQUIRED
- `blog.html` — додано 2 картки: post-315 (novice-to-intermediate), post-316 (response variability)

### Changed
- `STATUS.md` — TL;DR оновлено: 314 → 316 posts, blog.html рядок v2.85.0, 273 → 275 cards
- `README.md` — post-315, post-316 відмічено [x] у roadmap
- `VERSION` — 2.83.0 → 2.85.0

---

## [2.83.0] — 2026-06-07

### Added
- `content/post-314-pump-cellular-swelling-hypertrophy.md` — памп і клітинне набрякання: тріада гіпертрофії Schoenfeld (2010, 2013), cell swelling hypothesis, mTORC1-активація через набрякання (Loenneke 2012), BFR як природний експеримент (Pearson & Hussain 2015), межі доказів, памп як діагностика проти пампу як мети; 13 хв читання; clinical-safety NOT REQUIRED
- `blog.html` — додано 5 карток: post-310 (supersets), post-311 (IAP/bracing), post-312 (strength standards), post-313 (breathing), post-314 (pump)

### Changed
- `STATUS.md` — додано рядки для post-312, post-313, post-314 у content engine table
- `README.md` — post-314 відмічено [x] у roadmap; додано post-316–318 до roadmap
- `VERSION` — 2.82.0 → 2.83.0

---

## [2.82.0] — 2026-06-07

### Added
- `content/post-312-strength-standards.md` — стандарти сили: Wilks/DOTS/IPF GL коефіцієнти, bodyweight multipliers за рівнями (novice/intermediate/advanced/elite), реалістична траєкторія новачок→intermediate, як використовувати стандарти без пастки порівняння; 12 хв читання; clinical-safety NOT REQUIRED
- `content/post-313-breathing-during-set.md` — дихання під час підходу: повна механіка Valsalva (субглотальний тиск → IAP → жорсткість хребта), McGill 2007/Hackett & Chow 2013/Tobin et al. 1983/Hodges & Gandevia 2000, breath-reset стратегія у багатоповторних сетах, матриця тип вправи × % 1RM × стратегія дихання; 11 хв читання; clinical-safety NOT REQUIRED

### Changed
- `README.md` — post-312, post-313 відмічено як [x]; roadmap post-314–315 на черзі

---

## [2.81.0] — 2026-06-07

### Added
- `subprocessors.html` — публічна GDPR Art. 13(1)(e) сторінка з реєстром всіх 11 sub-processors; закриває CC9-GAP-004 (P0 перед enterprise GA); editorial-brutalist дизайн (Fraunces + Manrope + JBMono, warm-black palette); розкриває: тир-класифікацію (T1/T2), категорії даних, локацію обробки, механізм передачі (EU Adequacy / SCCs Module 2 / Cloudflare DPA / UK IDTA), процедуру 30-денного повідомлення, права суб'єктів даних, breach notification SLA (8h до Enterprise tenant), посилання на DPA-FORM-001 v1.0; privacy floor strip ("жоден SP не отримує health data linked to employer"); change log; CTA до legal@form.coach; cross-ref: `docs/SUBPROCESSORS.md`, `docs/SOC2_READINESS.md §44`, DEC-030

## [2.80.0] — 2026-06-07

### Added
- `content/post-311-intraabdominal-pressure-bracing.md` — внутрішньочеревний тиск і брейсинг: brace vs. draw-in (Grenier & McGill 2007 — bracing значно вища жорсткість хребта), антиципаторна активація ТА (Cresswell 1992, Hodges & Richardson 1997), Valsalva механіка і безпека, пояс як підсилювач IAP (Harman 1989: +25–40% IAP); 13 хв читання; clinical-safety NOT REQUIRED

### Changed
- `README.md` — post-311 відмічено як [x]; додано roadmap post-312–315 (стандарти сили, дихання в підходах, памп/клітинне набрякання, межа новачок→intermediate)
- `STATUS.md` — додано рядки post-310 і post-311 до content engine таблиці

---

## [2.79.0] — 2026-06-07

### Added
- `content/post-310-supersets-preexhaust-mechanical-dropsets.md` — розширений пост про суперсети: pre-exhaust (Brentano 2017 — ЕМГ не зростає, об'єм падає), mechanical drop sets (кутова специфічність), PAP через антагоністичні пари (Robbins 2010 +4.8% сила); матриця вибору техніки за метою; 14 хв читання; clinical-safety NOT REQUIRED

### Changed
- `README.md` — синхронізовано трекінг постів: 279, 287, 288, 289, 290 відмічено як [x] із зазначенням фактично написаних тем (теми в плані розійшлися з фактичним контентом через еволюцію roadmap); post-310 відмічено як [x]

---

## [2.78.1] — 2026-06-07

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` v0.5 → v0.6 — +5 `vuln.*` DEC-030 HMAC-chained events (`vuln.cve_discovered`, `vuln.patch_deployed`, `vuln.sla_exception_granted`, `vuln.wont_fix_closed`, `vuln.enterprise_notified`); retention table row for `vuln.*` (7 yr, CC7.1/CC7.2); uplift-rule U-02 Won't Fix blocker enforced in chain audit. Closes `compliance/cc7/vuln-management-policy.md` (POL-010) checklist item 6 (P1 M3).

---

## [2.78.0] — 2026-06-07

### Added
- `content/post-309-programming-for-parents.md` — Програмування для батьків маленьких дітей: нова фізіологічна базова лінія при хронічному недосипі, MEV як операційна ціль (Bickel 2011: 1/9 об'єму зберігає адаптації 32 тиж), ієрархія зрізання об'єму, 30/45-хвилинні формати сесій, RPE/RIR-авторегуляція при нестабільному розкладі, таймлайн і точка повернення до нормального режиму; clinical-safety NOT REQUIRED.
- `blog.html` — додані картки для post-306, post-307, post-308, post-309.

---

## [2.77.0] — 2026-06-07

### Added
- `content/post-306-female-progression.md` — Прогресія для жінок: Morton 2018 (PNAS, equivalent relative response), Roberts 2023, естроген як анаболічний/антикатаболічний агент (ERα/IGF-1, NF-κB), менструальний цикл і тренінг (McNulty 2020, Wikström-Frisén 2017, Sung 2014), відмінності у відновленні (Hunter 2014, Enns & Tiidus 2010), градація доказів (strong/moderate/weak); clinical-safety NOT REQUIRED.
- `content/post-308-unilateral-asymmetric.md` — Сила у нестандартних положеннях: bilateral deficit (Kuruganti & Murphy 2008, Magnus 2010), lateral stabilizer chain (McGill 2010), carry taxonomy (farmer/suitcase/overhead/Zercher), cross-education (Carroll 2006, 7–18%), offset loading як GPP-інструмент; clinical-safety NOT REQUIRED.

---

## [2.76.1] — 2026-06-07

### Added
- `content/post-307-training-sleep.md` — Тренінг і сон: двонаправлений зв'язок; GH/SWS, Leproult & Van Cauter 2011 (testosterone/sleep), Blumert 2006 (performance), Dattilo 2011 (mTOR), Dolezal 2017 (training→sleep), Stutz 2019 (evening exercise myth); практична рамка для спортсмена з дефіцитом сну без порад щодо добавок; clinical-safety NOT REQUIRED.

---

## [2.76.0] — 2026-06-07

### Added
- `compliance/c1/device-disposal-policy.md` — POL-013 v1.0 IN_FORCE: Media and Device Disposal Policy. Extracted from `docs/SOC2_READINESS.md §66` per MDD-P0-04. NIST SP 800-88 Rev. 1 wipe standards (FileVault 2 EACS cryptographic erasure for SSD/NVMe; 7-pass Disk Utility for USB; iOS EACS; certified vendor shredding for HDDs); per-category disposal procedures (laptop / mobile test device / external storage); remote-work device policy (FileVault 2 + auto-lock ≤5 min + MDM + no-credentials-in-plaintext); test device handling for devices that touched production health data (audit log search → two-person sign-off or HMAC-chain compensating control → health data cache verification → standard wipe); chain of custody (solo-founder compensating control via `asset.disposal_initiated` → `asset.disposal_completed` HMAC chain sequence; third-party handoff form template; founder attestation); 6-event timeline table; third-party destruction vendor criteria (NAID AAA / e-Stewards / R2-RIOS; NIST Purge or ≤6 mm shred; cert within 5 days; EU data residency for EU health-data devices; DPA required); SOC 2 evidence mapping (C1.2 / CC6.5 / CC6.7 / CC1.4). Registered as PRE-34-E-008. Closes: C1-GAP-002 (🔴 → 🟡 Authored), C1-GAP-004 (P1 Open → 🟡 Authored), PRE-06 (🔴 → 🟡 Authored). References: `docs/SOC2_READINESS.md §66`, `docs/AUDIT_LOG_SCHEMA.md`, DEC-030.
- `compliance/assets/device-register.csv` — Device register baseline v1.0 (MDD-P0-01 template). Three rows for solo-founder phase (FORM-DEV-001 laptop, FORM-MOB-001 primary mobile, FORM-MOB-002 test device). Serial numbers placeholder to 1Password vault. Fields: asset_id, device_type, manufacturer, model, serial_number, os, owner, provisioned_date, production_access, health_data_cached, disposal_date, disposal_method, mdd_evidence_id.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` → v0.5: +5 `asset.*` device disposal events (`asset.disposal_initiated` STANDARD, `asset.disposal_completed` STANDARD, `asset.device_sanitized` HIGH, `asset.chain_of_custody_transferred` HIGH, `asset.disposal_log_filed` STANDARD — all 7-year retention). New "Asset & Device Management" section with HMAC-chain requirement (30-day `initiated` → `completed` gap triggers MDD-AL-01 alert). Retention table updated with `asset.*` row. Closes MDD-P0-03 (SOC2_READINESS §66.12).
- `compliance/policy-approval-log.csv` — POL-013 registered: Media and Device Disposal Policy v1.0 IN_FORCE.
- `VERSION` → 2.76.0.

---

## [2.75.1] — 2026-06-07

### Added
- `content/post-305-strength-training-longevity.md` — Силовий тренінг і довголіття: доза-відповідь між обсягом MSA і all-cause mortality; J-крива (Liu 2019 BJSM, Momma 2022 BJSM, n=1.29M); м'язова сила як незалежний предиктор смертності (Ruiz 2008 BMJ, Leong 2015 Lancet — PURE); grip strength vs систолічний тиск; міокіни і м'яз як ендокринний орган; де перетинаються «тренуватись для здоров'я» і «тренуватись для сили». clinical-safety NOT REQUIRED.
- `blog.html` — card для post-305 (264 cards total).
- `README.md` — post-305 відмічений виконаним; додано roadmap post-306–310.
- `STATUS.md` — рядки post-302–305 у content engine table.

### Changed
- `VERSION` → 2.75.1.

---

## [2.75.0] — 2026-06-07

### Added
- `content/post-302-subjective-vs-objective-readiness.md` — Суб'єктивна vs об'єктивна готовність: алостатичне навантаження, чотири квадранти готовності, коли HRV вводить в оману (Plews 2013, Buchheit 2014, Saw 2016, McEwen 1998, Meeusen 2013). clinical-safety NOT REQUIRED.
- `content/post-303-self-coach-programming-log.md` — Програмування для самокоуча: мінімальна документація для максимальної діагностики; RPE/RIR-логування, ACWR, autoregulation, periodization як гіпотезне тестування (Zourdos 2016, Helms 2016, Gabbett 2016, Kiely 2012). clinical-safety NOT REQUIRED.
- `content/post-304-tempo-effective-rm.md` — Темп і ефективний RM: tempo notation (4010/3111/2011), як швидкість руху змінює дозування навантаження; практичні корекції навантаження при переході між темпами (Schoenfeld 2015, Wilk 2020, Lacerda 2016, Burd 2012, Roig 2009). clinical-safety NOT REQUIRED.

### Changed
- `VERSION` → 2.75.0.

---

## [2.74.0] — 2026-06-07

### Added
- `compliance/c1/nda-template.md` — Employee / Contractor Non-Disclosure & Confidentiality Agreement template (C1-NDA-001, PRE-34-E-003). Closes SOC 2 drafting gap C1-GAP-001 (pre-hire gate). Ten clauses per `docs/SOC2_READINESS.md §42.3.2`: Parties, Confidential Information definition (Art. 9 health data explicitly enumerated: workout records, coaching turns, CV sessions, wearable readings, meal logs, biometric inference data), Obligations (strict confidence, use limitation, GDPR Art. 32 standard of care, 1-hour breach notification feeding Art. 33 clock), Exclusions (standard four + health-data carve-out), Data Protection Obligations (GDPR Art. 9/28/32/33; process only as directed; no secondary ML training use; deletion-on-termination + 3-day cert), IP Assignment (work-for-hire; explicit coverage of model weights, training data annotations, LoRA adapters, CV pipeline improvements), Non-solicitation (12 months post-termination; employees + customers; counsel enforceability note), Term (5-year Confidentiality Period; health data indefinite survival; IP assignment perpetual), Governing law (Ukraine primary; EU GDPR precedence on data protection; Kyiv Commercial Court; amendment note for incorporation jurisdiction), Remedies + GDPR Art. 83 penalty notice. Includes filing instructions (DocuSign → SHA-256 hash → nda-register.csv → DEC-030 `personnel.nda_signed` event → `personnel.hire_check_passed` gate). Template status: authored · pending outside counsel review (UA/EU jurisdiction) before first execution. Reference: `docs/SOC2_READINESS.md §42`, `docs/AUDIT_LOG_SCHEMA.md`, DEC-030.

### Changed
- `VERSION` → 2.74.0.

---

## [2.73.0] — 2026-06-07

### Added
- `content/post-301-strength-cognition-bidirectional.md` — Силовий тренінг і когнітивна продуктивність. Двонаправлений зв'язок: гострі (BDNF пік 15–30 хв, механізми через IGF-1/irisin/катехоламіни) vs хронічні ефекти (структурні нейропластичні зміни при 12–24 тижнях). Executive function: Colcombe & Kramer (2003, d=0.68), Cassilhas et al. (2007), Best et al. (2014 meta-analysis), Northey et al. (2018 Br J Sports Med), Tsai et al. (2014), Liu-Ambrose et al. (2010). Нейротрофіни: BDNF, IGF-1, VEGF, cathepsin B — механізм та сила доказів для кожного. Дозо-відповідь: 2–3×/тиж. достатньо, 50% vs 80% 1RM comparable (Cassilhas). Чесний розбір обмежень: домінування аеробних RCT, sedentary baseline confound, short duration, гетерогенні когнітивні метрики. clinical_safety_status: NOT REQUIRED.
- `README.md` — post-296..300 marked [x] з уточненими описами; post-301..305 додані як roadmap-записи.
- `STATUS.md` — рядки для post-296..301.

### Changed
- `VERSION` → 2.73.0.

---

## [2.72.0] — 2026-06-07

### Added
- `content/post-297-good-enough-program.md` — «Достатньо добра» програма vs ідеальна. Compliance premium: математика adherence — 0.75×0.85 > 0.95×0.45. Три структурні причини провалу «ідеальних» програм (когнітивне навантаження, часовий бюджет, відновлення). Чотири мінімальні критерії ефективності програми. Практичний фільтр з 4 питань. Посилання: Helms et al. 2016, Schoenfeld et al. 2017. clinical_safety_status: NOT REQUIRED.
- `content/post-298-training-unstable-schedule.md` — Тренінг при нестабільному розкладі. Мінімальна ефективна доза, ієрархія сесій при вимушеному скороченні, компресія частоти 4→2/тиждень без втрати стимулу, фіксація замість планування наперед. Без мотиваційного BS. clinical_safety_status: NOT REQUIRED.
- `content/post-300-detraining-what-you-keep.md` — Ієрархія втрат при детренуванні. Тайм-константи: VO2max (-10–12% за 2–3 тиж., Mujika & Padilla 2000) → гіпертрофія (2–3 тиж.) → максимальна сила (3–4 тиж.) → нейром'язовий паттерн (10–12+ тиж.). Механізми швидкого відновлення досвідчених (міонуклеарна пам'ять, Gundersen lab; нейральна ефективність). Тайм-лайн повернення по тривалості паузи. clinical_safety_review: PASS (виключно нейром'язова фізіологія).

### Changed
- `VERSION` → 2.72.0.

---

## [2.71.0] — 2026-06-07

### Added
- `content/post-296-rpe-calibration.md` — RPE-калібрування без тренера. Проблема anchor drift у самокоучингу, три об'єктивних сигнали (velocity loss, темп дихання, деградація форми), практичний метод калібрувальних сетів, типові помилки — систематичне заниження vs завищення. Посилання: Zourdos et al. 2016, Helms et al. 2016, González-Badillo & Sánchez-Medina 2010. clinical_safety_status: NOT REQUIRED.
- `content/post-299-strength-without-gym.md` — Мінімальний ефективний протокол для відрядження. Реальні дані по детренуванню за 3–7 днів (Mujika & Padilla 2000), три рухових паттерни без обладнання, прогресія через tempo/паузи/unilateral, чек-лист для пакування, коли NOT тренуватись. Мета — maintenance, не прогрес. clinical_safety_status: NOT REQUIRED.

### Changed
- `VERSION` → 2.71.0.

---

## [2.70.0] — 2026-06-07

### Added
- `compliance/licence-register.md` — OSS-LIC-001 v1.0. Open-source licence register for weak-copyleft production dependencies (MPL-2.0, LGPL-2.1, LGPL-3.0, EUPL-1.2). Required by `docs/SOC2_READINESS.md §54.7.1`; closes the `compliance/licence-register.md` reference gap. Pre-launch baseline: zero weak-copyleft dependencies registered. Schema includes: licence classification reference (permissive → register-and-review → prohibited), dependency layers (iOS/Cloudflare Workers/Android/build tooling), Snyk CI gate policy (block on GPL/AGPL/SSPL, warn on MPL/LGPL/EUPL), `license-checker` CLI procedure for quarterly npm audits, process for adding new weak-copyleft entries (5-step: identify → trigger-analysis → register → compliance-officer review → DEC-030 event), three-person approval process for prohibited-licence exceptions, DEC-030 HMAC-chained audit events (`security.licence_weak_copyleft_added`, `security.licence_violation_detected`, `security.licence_register_reviewed`, `security.licence_audit_filed`), quarterly evidence filing cadence (Q1/Q2/Q3/Q4 audit CSVs to `compliance/evidence/licence-audits/`), SHA-256 hash filing to `compliance/checksums.sha256`. SOC 2 controls: CC9.2 (supply-chain licence risk), CC6.8 (unauthorised software prevention), CC5.2 (software inventory). compliance-officer + security-engineer.

### Changed
- `VERSION` → 2.70.0.

---

## [2.69.0] — 2026-06-07

### Added
- `content/post-295-strength-plateau-vs-life-plateau.md` — Плато сили vs плато обставин: діагностика фізіологічної стелі проти порушеного відновлення. Чотири діагностичні питання (% виконання плану, RPE drift, сон, зовнішні стресори), матриця порівняння двох станів, протокол для кожного. Editorial-brutalist, 13 хв. clinical-safety PASS.
- `blog.html` — картка post-295 додана; hero stats: 295 → 296 статей, 3 835 → 3 848 хв читання; label: 296 постів.
- `README.md` — оновлено content tree (post-295) і roadmap нових тем (post-296–300).
- `STATUS.md` — оновлено: v2.69.0, 296 posts, 256 cards.

### Changed
- `VERSION` → 2.69.0.

---

## [2.68.0] — 2026-06-07

### Added
- `content/post-279-rest-pause-intraset-rest-motor-unit-recruitment.md` — Rest-pause тренінг: нейром'язова механіка внутрішньосетного відпочинку. Fills numbering gap (278→280). Henneman size principle і рекрутування тип II при підходах до відмови. Prestes et al. (2017 JSCR): rest-pause = схожа гіпертрофія + вища сила 1ПМ при рівному тоннажі. Кінетика PCr: 15–20 с → 50–60% відновлення. Різниця cluster sets / Myo-reps / rest-pause: Gonzalez-Badillo et al. (2016 IJSPP). Матриця застосування: ізоляція + обмежений час — так; базові рухи і початок мезоциклу — ні. clinical-safety NOT REQUIRED.
- `content/post-293-central-vs-peripheral-fatigue.md` — Центральна vs периферична втома: що насправді зупиняє підхід. Fitts (1994 Physiol Rev): Pi, а не лактат — основний фактор периферичної відмови. Allen, Lamb & Westerblad (2008 Physiol Rev): лактатна гіпотеза спростована in vivo. Enoka & Duchateau (2008 J Physiol): twitch interpolation technique, voluntary drive. Noakes Central Governor — евристика, не вирішена теорія. Матриця: 1–5 рп → переважно периферична; 8–20 рп → центральна гальмівна активується раніше справжньої периферичної відмови. RPE як інтегральний сигнал ЦНС. clinical-safety NOT REQUIRED.
- `content/post-294-lengthened-position-hypertrophy.md` — Гіпертрофія в подовженому положенні: чому глибина важливіша за скорочення. Maeo et al. (2021 J Physiol): Nordic curls повний vs скорочений ROM — значно більша дистальна гіпертрофія при повному ROM. Herzog (2018 J Biomechanics): тітін і пасивна силова надбавка при великій довжині саркомера. Goto et al. (2019 JSCR), McMahon et al. (2014 Eur J Appl Physiol). Практика: incline curl > preacher; глибина присідань; повний RDL ROM; контрольована ексцентрика — не «ціна», а механізм. clinical-safety NOT REQUIRED.
- `blog.html` — 3 нові картки (post-279, post-293, post-294) додано; hero stats оновлено: 10 → 295 статей, 121 → 3 835 хв читання; label оновлено до «295 постів · clinical-safety cleared».
- `STATUS.md` — оновлено: v2.68.0, 295 posts, 255 cards, 60+ releases, 75+ docs.

### Changed
- `VERSION` → 2.68.0.

---

## [2.67.0] — 2026-06-07

### Added
- `docs/RESPONSIBLE_DISCLOSURE.md` — POL-013 candidate · Coordinated Vulnerability Disclosure Policy v1.0. Closes `/disclosure` P0 gap from `docs/SOC2_READINESS.md §39`. Intake channel `security@form.coach` with PGP encryption (key block placeholder for M4 deployment). 90-day coordinated disclosure window with 14-day extension and 104-day hard cap. CVSS 3.1 severity bands with three FORM-specific uplift rules: U-01 (GDPR Art. 9 health data exposure path elevates +1 tier), U-02 (RLS / tenant isolation bypass auto-elevates to Critical — no downgrade, no Risk Acceptance), U-03 (HMAC chain integrity affects +1 tier); uplifts cumulative. Response SLA table: Critical 24 h, High 7 d, Medium 30 d, Low 90 d from validation. Seven-stage response lifecycle (intake → triage → validation → remediation → re-test → disclosure coordination → close). Enterprise customer notification templates: E-VULN-01 (Critical, within 2 h of patch), E-VULN-02 (High, within 24 h). Hall of Fame (no-cash bounty); Bug Bounty roadmap at Month 6 post-launch ($50–$2,500 range, HackerOne or Bugcrowd). Eight DEC-030 HMAC-chained `vuln.*` events (disclosure_received, triaged, accepted, rejected, patch_deployed, enterprise_notified, disclosure_closed, sla_breach); all 7-year retention, no `user_id`, reporter pseudonymous only. SOC 2 controls: CC7.1, CC7.2, CC7.3, CC7.4, CC2.3, C1.2. Five evidence artefacts CVD-E-001 through CVD-E-005. `/disclosure` gap: 🔴 Open → 🟡 Partial (policy authored; 🟢 when `security.form.coach/disclosure` deployed). security-engineer + compliance-officer.

### Changed
- `VERSION` → 2.67.0.

---

## [2.66.0] — 2026-06-07

### Added
- `content/post-292-technical-failure-vs-muscular-failure.md` — Технічна відмова vs м'язова відмова — де зупиняти підхід. Calatayud et al. (2016 J Hum Kinet): ЕМГ-активація цільового м'яза −18–24% у деградованих повторах, навантаження перерозподіляється на компенсаторні структури. Fitts (1994 Physiol Rev): Pi, а не лактат — головний метаболічний фактор м'язової відмови. Androulakis-Theotokis et al. (2021 Med Sci Sports Exerc): тренінг до відмови ≈ не до відмови за гіпертрофією при рівному об'ємі — але вища стомленість і довший SRA. Протокол «зупинка до технічного збою»: кінематичні маркери деградації, калібровочна RPE-сесія, маркер-повтор. Відмінність від психологічної відмови (post-268). Коли м'язова відмова виправдана: ізоляція, останній підхід. clinical-safety NOT REQUIRED.
- `blog.html` — картка post-292 додана.

### Changed
- `VERSION` → 2.66.0.

---

## [2.65.0] — 2026-06-07

### Added
- `content/post-287-muscle-synergy-patterns.md` — Синергетичні патерни м'язів: ЦНС управляє групами, а не окремими м'язами. Tresch et al. (1999 Nature Neurosci): спинальна локалізація синергій. d'Avella & Bizzi (2005 PNAS): NMF аналіз — 4–6 компонентів пояснюють >90% ЕМГ варіабельності. Lacquaniti et al. (2012 J Physiol): патерни локомоції і модуляція ваг. Bizzi & Cheung (2013 Front Comput Neurosci): нейронне походження синергій. Три фази технічного прогресу: синергетична → нейронна → автоматизація. clinical-safety NOT REQUIRED.
- `content/post-288-thoracolumbar-fascia-force-transfer.md` — Торако-люмбальна фасція і передача зусилля: що відбувається між широчайшим і сідницею. Vleeming et al. (1995 Spine): діагональний крос-ланцюг latissimus↔gluteus через TLF. Barker & Briggs (1999 Spine): прикріплення posterior layer. Willard et al. (2012 J Anat): три шари TLF, крос-ламіновані волокна. Gracovetsky et al. (1977/1981): фасціальний механізм стабілізації хребта. clinical-safety APPROVED WITH NOTES — disclaimer додано.
- `content/post-289-velocity-specificity-strength-training.md` — Специфічність швидкості в силовому тренінгу: чому темп вправи визначає тип адаптації. Behm & Sale (1993 J Appl Physiol): намір, а не реальна швидкість визначає velocity-specific адаптацію. Duchateau & Hainaut (1984 J Appl Physiol): ізометрія vs ізотонія — різна нейронна адаптація. Morrissey et al. (1995 Med Sci Sports Exerc): перенос від повільного до швидкого менш ефективний. Maximum concentric intent як стандарт. clinical-safety NOT REQUIRED.
- `content/post-290-feedforward-anticipatory-postural-adjustments.md` — Feedforward і антиципаторні постуральні корекції: що ЦНС робить до того, як навантаження прикладене. Cordo & Nashner (1982 J Neurophysiol): постуральні м'язи активні на 50–100 мс раніше від агоністів. Hodges & Richardson (1997 Exp Brain Res): TrA перша в APA-ланцюгу, ~29 мс до руху. Massion (1992 Prog Neurobiol): feedforward vs feedback модель. Стандартизований сетап як умова надійного feedforward. clinical-safety APPROVED WITH NOTES — framed in healthy athletic performance.

### Changed
- `VERSION` → 2.65.0.

---

## [2.64.0] — 2026-06-07

### Added
- `docs/SECURITY_AWARENESS_TRAINING_POLICY.md` — POL-012 · Security Awareness & Training Policy v1.0. Standalone extraction from `docs/SOC2_READINESS.md §22`. 8-topic curriculum (OWASP Top 10, phishing, data classification, incident response, credential hygiene, secure code review, GDPR/privacy, vendor risk awareness). Solo-founder annual training plan with evidence artefact registry. New hire security onboarding checklist — 8 sequential steps, MFA hard gate (Step 3 non-negotiable pre-credential gate). Annual refresher cadence: Q1 28 Feb (all topics) + Q3 31 Aug (OWASP refresher) + post-incident ad-hoc (within 14 days). Phishing simulation programme (post-hire): KnowBe4/GoPhish, quarterly, no-punitive-action policy, aggregate-results-only for SOC 2. Evidence package CC1-E-001 through CC1-E-005. DEC-030 HMAC-chained audit events for training completions, onboarding, MFA gate, phishing results, access suspension/reinstatement. Non-compliance escalation: Day 1–7 reminder → Day 8–14 founder escalation → Day 15+ access suspension. Fills CC1.4 / CC2.2 / CC9.2 gaps. compliance-officer + security-engineer.

### Changed
- `compliance/policy-approval-log.csv` — POL-012 row added (Security Awareness & Training Policy, v1.0, `AUTHORED_PENDING_APPROVAL`, 2026-06-07). Register now covers POL-001 through POL-012. Closes CC5 gap: "12th row pending SECURITY_AWARENESS_TRAINING_POLICY.md extraction" noted in §27.3.2.
- `docs/SOC2_READINESS.md` — §22 header annotated: standalone policy POL-012 extracted to `docs/SECURITY_AWARENESS_TRAINING_POLICY.md`; §27.3.2 policy log note updated from "11 policies, 12th pending" to "12 policies, POL-012 extracted".
- `VERSION` → 2.64.0.

---

## [2.63.0] — 2026-06-07

### Added
- `content/post-278-supersets-antagonist-pairs.md` — Суперсети: коли скорочують час і не коштують сили. Weakley et al. (2017 Eur J Appl Physiol): антагоністичні суперсети −27% хронологічного часу при збереженому об'ємі. Simpson (2022): агоністичні суперсети знижують об'єм у другій вправі. Murach & Bagley (2016 SCJ): метаболічний стрес без механічного навантаження не транслюється в гіпертрофію. Robbins et al. (2010 JSCR): реципрокне гальмування антагоністів. Peterson et al. (2011 JSCR): +33% EPOC при −33% часу. Матриця трьох типів суперсетів. clinical-safety NOT REQUIRED.
- `blog.html` — картка post-278 додана до сітки.

### Changed
- `README.md` — content roadmap: post-278 позначений [x].
- `VERSION` → 2.63.0.
- `STATUS.md` — версія оновлена до v2.63.0.

---

## [2.62.0] — 2026-06-07

### Added
- `content/post-284-minimal-effective-training-protocol.md` — Мінімальний ефективний протокол: скільки насправді потрібно для гіпертрофії. Schoenfeld & Grgic (2020 JSCR): 3 підходи двічі на тиждень = значуща гіпертрофія; 6 підходів/тиждень/група як валідна стратегія, не план Б. Ralston et al. (2017 JSCR): малий обсяг + висока інтенсивність → сила. Burd et al. (2012): MPS при різних режимах навантаження. Практична матриця: 2×45хв / 3×30хв / щоденні 20хв. Де MV ≠ MEV і як це розпізнати. clinical-safety NOT REQUIRED.
- `content/post-286-iap-valsalva-belt-spine-stability.md` — IAP, Valsalva і пояс: що насправді стабілізує хребет під час підйому. McGill et al. (1990 J Biomech): внутрішньочеревний тиск як гідравлічний стовп — знімає компресію/зсув з дисків. Harman et al. (1989 Med Sci Sports Exerc): пояс підвищує IAP без зниження EMG розгиначів — підсилює брейсинг, не замінює. Lander et al. (1992 Med Sci Sports Exerc): belt і абсолютна сила у присіданні. Valsalva механіка і межі застосування. Операційна схема: коли пояс доцільний. clinical-safety NOT REQUIRED.
- `blog.html` — картки post-284 і post-286 додані до сітки.

### Changed
- `README.md` — content roadmap: post-284 і post-286 позначені [x].
- `VERSION` → 2.62.0.
- `STATUS.md` — версія оновлена до v2.62.0.

---

## [2.61.1] — 2026-06-07

### Changed
- `docs/SOC2_READINESS.md` — §14 Formal Risk Register patch (v2.6 → v2.7). Two changes per `compliance/cc3/risk-register-review-2026-Q3.md` (CC3-RRR-2026-Q3): (1) **SR-03 insider threat** residual score decreased 4 → 3 LOW on first access review execution (evidence: Q3-2026 access review); (2) **OR-02 Billing / subscription fraud** added to Vendor/Operational Risks category (inherent 1×3 = 3 LOW; residual 2 LOW; controls: Stripe HMAC-SHA256 webhook validation, subscription status derived from Stripe not local mutable field, `billing.subscription_change` DEC-030 HMAC-chained audit event, nightly reconciliation planned Phase 3). Risk count updated 18 → 19. Heatmap updated: OR-02 at 1-Rare / 3-Moderate. Does not change SOC 2 readiness scorecard. compliance-officer + security-engineer.
- `VERSION` → 2.61.1.

---

## [2.61.0] — 2026-06-07

### Added
- `content/post-291-invisible-adaptation.md` — Ти платиш за адаптацію, яку не бачиш. Moritani & deVries (1979): нейральна фаза 3–4 тижні без змін CSA. Haun et al. (2019 PLOS ONE): саркоплазматична надбудова передує міофібрилярній. Bamman et al. (2007 JSCR): міоядерний пул — 4–8 тижнів до першого значущого приросту. Ogasawara et al. (2013): «3+3+3» vs безперервний — еквівалентна гіпертрофія. Seaborne et al. (2018 Sci Rep): епігенетична пам'ять тренінгу (NCOR2/IGF1 гіпометилювання) зберігається після детренування. Bruusgaard et al. (2010 PNAS): міоядра не втрачаються при атрофії. Практична операційна рамка: 4 маркери адаптації без дзеркала. clinical-safety NOT REQUIRED.
- `README.md` — content roadmap: додано post-291 ([x]) і нові теми post-292–295 ([ ]).
- `blog.html` — картка post-291 додана до сітки.

### Changed
- `VERSION` → 2.61.0.
- `STATUS.md` — оновлено лічильник постів: 278 → 279.

---

## [2.60.0] — 2026-06-07

### Added
- `content/post-281-autoregulation-program-vs-reality.md` — Чому програма на папері рідко виглядає так само у залі. Zourdos et al. (2016 JSCR): ±5–10% інтраіндивідуальна варіабельність 1RM у тренованих атлетів. Haff & Triplett (2016): зовнішнє vs внутрішнє навантаження. Helms et al. (2018 JSCR): RPE-базоване програмування vs відсоткова точність — три проблеми (застарілий максимум, контекстуальна варіація, пропагована неточність). Практична система авторегуляції: RPE-ціль замість відсотка, RIR-журнал, пом'якшувач об'єму при «поганих днях». clinical-safety NOT REQUIRED.
- `content/post-282-plyometrics-for-strength-athletes.md` — Пліометрика для силових атлетів. Markovic & Mikulic (2010 Sports Med): мета-аналіз 26 RCT — +7.5% CMJ. Wilson et al. (1993 J Appl Physiol): поєднання пліометрики і силового тренінгу перевершує обидва окремо. Cormie et al. (2010 JSCR): важке навантаження vs потужнісний vs комбінований протокол — RFD і пікова потужність; комбінований > обидва для RFD. Механізми SSC (рефлекторна активація, сухожильна жорсткість, нейральна специфічність). Практична матриця: для кого пліометрика має сенс, обмеження (відновлення, технічний поріг), об'єм за профілем. clinical-safety NOT REQUIRED.
- `content/post-283-interindividual-variability-training-response.md` — Варіабельність між людьми у реакції на тренінг. Hubal et al. (2005 Med Sci Sports Exerc): 585 учасників, ідентичний протокол, −2% до +59% гіпертрофії. Timmons (2011 J Appl Physiol): VO₂max responders vs non-responders — зміна протоколу конвертує low responders. ACTN3 R577X і ACE I/D: реальний ефект-розмір малий, пояснюють кілька відсотків варіабельності, не десятки. Що реально пояснює варіабельність: базовий рівень, м'язова пам'ять, сон і відновлення, відповідність типу навантаження, behavioral compliance. Порівняння з іншими — статистично некоректне. clinical-safety NOT REQUIRED.
- `compliance/cc5/onboarding-checklist-template.md` — New-hire security onboarding checklist (SOC 2 CC5-P2-007). 6 секцій: провізіонування доступу (GitHub 2FA, 1Password, principle of least privilege), безпека пристроїв (FDE, auto-lock, OS-updates), підписання 12 політик (POL-001..011 + CoC), брифінг з безпеки (фішинг, credential hygiene, secret handling, data handling, incident reporting, social engineering), offboarding pre-briefing, sign-off. Архів — `compliance/evidence/cc1/onboarding/`. Satisfies CC1.4, CC5.3, CC6.1, CC6.2, CC6.3.

### Changed
- `VERSION` → 2.60.0.

---

## [2.59.0] — 2026-06-07

### Added
- `enterprise/runbooks/FORM-DR-003.md` — Supabase primary-region failure and PITR failover runbook. Pre-execution checklist, failure classification (platform vs FORM-side), §2 maintenance mode activation (wrangler KV flag + enterprise notification via IRP E-02 template), §3 PITR initiation with RPO breach assessment and founder approval gate, §4 DEC-030 HMAC chain integrity verification (SQL chain-check query + P0 escalation path if broken), §5 Privacy Floor verification (cross-tenant RLS test + admin aggregate user_id check) + F1–F8 smoke test suite, §6 maintenance lift with audit_events emission (`system.maintenance_lifted`, `system.pitr_restore_completed`), §7 evidence filing to `compliance/evidence/cc7/incidents/FORM-DR-003-[DATE]/` + post-mortem SLA credit trigger. Aligned with `docs/BUSINESS_CONTINUITY.md §6.1`, DEC-030, `docs/ENTERPRISE_SLA.md §5`. SOC 2: CC7.4, CC7.5, A1.2, A1.3, CC9.1.
- `enterprise/runbooks/FORM-DR-004.md` — Cloudflare Workers degradation and Worker-bug rollback runbook. Two-path failure classification (platform incident vs FORM Worker regression); §2 platform incident response (KV maintenance flag with graceful failure on Cloudflare outage, Better Uptime fallback status, 4-hour SLA credit trigger); §3 Worker bug rollback (`wrangler rollback` primary path + manual `git checkout` fallback with exact commands); §4 post-recovery smoke test F1–F8 including F6 Privacy Floor (`has("user_id")` must return false on admin aggregate endpoint) and Worker version git-sha verification; §5 maintenance lift with audit event emission; §6 evidence filing + post-mortem action item requiring a CI regression test for the offending change. Known gap R-004 (no secondary CDN — Post-Series-A scope) documented. SOC 2: CC7.4, CC7.5, A1.2, A1.3.
- `enterprise/runbooks/FORM-DR-005.md` — Total vendor outage (nuke scenario) cold-restore runbook. Pre-execution checklist with 10 gates including founder presence requirement and GDPR 72-hour clock trigger; §1 enterprise tenant and regulatory communication within 15 minutes; §2 credential recovery (1Password standard path + emergency account-recovery path if 1Password compromised); §3 Backblaze B2 cold backup download (SHA1 verification + GPG signature check with hard-stop on signature failure) → new Supabase project provisioning → `pg_restore` + Supabase CLI migration-to-HEAD; §4 Cloudflare Worker re-deployment with all secrets rotated (HMAC_SIGNING_KEY and IP_HASH_SALT must be new values, never reused); §5 WorkOS SSO + SCIM + Stripe re-integration verification; §6 Privacy Floor SQL tests (4 checks) + row count delta + audit chain gap assessment (gap is expected and documented, not a chain break); §7 full F1–F8 smoke test suite; §8 maintenance lift + data-impact enterprise notification template + GDPR Art. 33/34 conditional filing + credential cleanup (`shred`) + `system.cold_restore_completed` audit event; §9 mandatory evidence package (14 items) + post-mortem additional sections. Known gaps: BCP-04 (no cold backup automation — RPO breach expected), BCP-03 (sole operator). SOC 2: CC7.4, CC7.5, A1.2, A1.3, CC9.1.
- `enterprise/runbooks/` directory created — closes structural gap referenced in `compliance/cc9/README.md` (DR runbooks listed as 🟢 Authored v2.38.1 but directory did not exist until this commit).

### Changed
- `VERSION` → 2.59.0.

---

## [2.58.0] — 2026-06-07

### Added
- `content/post-280-heat-training-thermal-stress.md` — «Тренінг у спеку: що відбувається з тілом і як не зламати адаптацію». Ely et al. (2010 Med Sci Sports Exerc): ядерна температура >38.5°C — пряме гальмування м'язового виходу через знижене рекрутування рухових одиниць і сповільнену нервову провідність; Nybo & Secher (2004 Prog Neurobiol): центральна термальна втома — гіпертермія знижує мозкову перфузію, підвищує RPE при тому самому навантаженні; Nielsen et al. (1993 J Physiol): 10-денна акліматизація — +4–10% об'єму плазми, нижча ЧСС, рання ініціація потовиділення, нижча субмаксимальна ядерна температура; Taylor (2014 Compr Physiol): 7–14 днів для основних адаптацій, passive heat (post-exercise sauna) як альтернатива; конкуренція трьох систем за кровотік (м'язи / шкіра / серцево-судинна); практична матриця акліматизований vs неакліматизований (інтенсивність, об'єм, відпочинок); ознаки перевищення адаптивного порогу; що не правда про тренінг у спеку. 13 хв читання. sports-scientist pending. clinical-safety NOT REQUIRED.
- Blog card for post-280 added to `blog.html`.
- Roadmap posts 286–290 added to `README.md`: дихальна механіка/IAP (286), функціональний тренінг як зламана категорія (287), mobility gap і підйом (288), periodization для любителя (289), adherence і SDT (290).

### Changed
- `VERSION` → 2.58.0.
- `README.md` — post-280 [x] у roadmap.
- `STATUS.md` — post-280 рядок у content engine table.

---

## [2.57.0] — 2026-06-07

### Added
- `content/post-285-internal-external-attentional-focus.md` — «Зовнішній vs внутрішній фокус уваги: де думати під час тренінгу». Wulf (2007): internal vs external focus визначення; Wulf et al. (2001 RQES): Constrained Action Hypothesis — external > internal для моторного навчання; Calatayud et al. (2016 Eur J Hum Mov): internal focus → вищий EMG pec/deltoid при 60% 1RM; Snyder & Fry (2012 J Strength Cond Res): словесна інструкція змінює EMG до 22%; Schoenfeld & Contreras (2016 Strength Cond J): practical framework — external для технічних вправ, internal для ізоляції; конкретні cues для присідання/жиму/тяги/curl; EMG-обмеження як проксі гіпертрофії. 11 хв читання. sports-scientist pending. clinical-safety NOT REQUIRED.
- `compliance/cc8/quarterly-change-review-YYYYQQ.md` — SOC 2 CC8.1 quarterly change review template (CC8-E-006). Секції: change volume summary (10 метрик vs попереднього кварталу), emergency change log (ID/date/CI-bypass/retroactive-approval/24h-compliance/DEC-030), CI/CD gate compliance per POL-011 §5.1 (7 checks), branch protection violations (direct commits/force-push/–no-verify), rollback events (trigger/component/MTTR/RCA), control effectiveness rating CC8.1-a–h, open findings → cc4/control-deficiency-log.csv, sign-off із evidence artefact manifest. Aligned з emergency-change-log.csv schema і POL-011 §§5,7,8,11,12.

### Changed
- `VERSION` → 2.57.0.
- `README.md` — post-285 [x] додано до roadmap; початкова IAP-заявка (post-278 draft) відхилена — дублювала post-25 і post-51.

---

## [2.55.0] — 2026-06-07

### Added
- `content/post-277-detraining-comeback-timeline.md` — «Детренінг і повернення: скільки реально втрачається і за який час». Mujika & Padilla (2000 Med Sci Sports Exerc): кінетика втрат — сила тримається до 4 тижнів, VO2max падає за 1–2 тижні, гіпертрофія між; Häkkinen et al. (1985 Eur J Appl Physiol): нейральна (EMG) деградація випереджає структурну — пояснення «важкого відчуття» при поверненні; Seaborne et al. (2018 Sci Rep): гіпометиляція NCOR2/IGF1 зберігається після детренінгу — молекулярна основа м'язової пам'яті; Ogasawara et al. (2013 Eur J Appl Physiol): periodic training (6 он / 3 офф) = continuous за 24 тижні гіпертрофії; Bickel et al. (2011 Med Sci Sports Exerc): 1/3 об'єму достатньо для підтримки; три операційні сценарії повернення (2 тижні / 4–8 тижнів / 2+ місяці); 13 хв читання. sports-scientist pending. clinical-safety NOT REQUIRED.
- Blog card for post-277 added to `blog.html`.
- `README.md` roadmap: post-277 позначено [x]; додані теми post-280–284 (heat training, autoregulation variability, plyometrics, inter-individual response variability, minimal time protocol).

### Changed
- `STATUS.md` — рядок post-277 додано до content engine table.
- `VERSION` → 2.55.0.

---

## [2.54.0] — 2026-06-07

### Added
- `compliance/cc3/fraud-risk-assessment.md` — Standalone SOC 2 CC3.3 fraud risk assessment (CC3-FRA-001). Six fraud scenarios with explicit CC3.3 framing: FR-01 health data misappropriation (insider), FR-02 fraudulent workout/progress reporting, FR-03 management override of privacy floor controls, FR-04 vendor/sub-processor data misuse, FR-05 supply-chain compromise via malicious dependency, FR-06 billing/subscription fraud. Each scenario: L×S inherent and residual scoring aligned with §14 methodology; explicit preventive vs. detective control separation per COSO 2013 guidance; fraud-specific health-data impact amplifier (+1 to Impact for Art. 9 special-category risks). Highest fraud inherent risk: FR-05 (12 HIGH); all residual scores LOW–MEDIUM. Annual attestation block. **Closes CC3-GAP-002** (compliance-officer + security-engineer).
- `compliance/cc3/risk-register-review-2026-Q3.md` — First formal risk register review (CC3-RRR-2026-Q3), executed 2026-07-15. Re-scored all 18 risks from `docs/SOC2_READINESS.md §14`; one score change: SR-03 (insider threat) residual decreased 4→3 on first access review execution. New risk identified: NR-01 billing/subscription fraud (3 LOW inherent; 2 LOW residual; to be added to §14 at next SOC2_READINESS.md revision). Zero P0/P1 incidents in review period. Gap status: CC3-GAP-001 and CC3-GAP-002 confirmed closed; CC3-GAP-003 (npm audit hard-fail) and VR-02 (Sentry DPA) remain open with tracked remediation paths. Quarterly cadence established: next Q4 2026 spot-check 2026-10-15. **Closes CC3-GAP-001** (compliance-officer + security-engineer).

### Changed
- `compliance/cc3/README.md` — CC3.2 and CC3.3 criteria updated 🟡 Partial → 🟢 Done on gap closure; evidence file table updated (risk-register-review and fraud-risk-assessment rows from 🔴 → 🟢); gap status table updated (CC3-GAP-001 and CC3-GAP-002 marked closed); auditor notes updated to reflect all four CC3 criteria at 🟢.
- `compliance/cc4/control-deficiency-log.csv` — CC3-GAP-002 marked closed (2026-06-07; exhibit: CC3-FRA-001).
- `VERSION` → 2.54.0.

---

## [2.53.0] — 2026-06-07

### Added
- `compliance/cc2/README.md` — CC2 Communication and Information evidence index. Documents all three CC2 criteria (CC2.1–CC2.3) with current status; evidence file table (14 artifacts — AUP, IRP, BCP, Privacy Policy, SECURITY.md, SUBPROCESSORS.md, SECURITY_QUESTIONNAIRE.md, ENTERPRISE_SLA.md, ENTERPRISE.md, security-training-log.md, policy-approval-log.csv, security.html, ONBOARDING_SECURITY.md, and two pending artifacts); internal communication channels table (four channels at solo-founder stage); external communication obligations table (9 obligations with owner and status); 6 gap items (CC2-GAP-001 through CC2-GAP-006, P0: 2, P1: 3, P2: 1); SOC 2 auditor notes including compensating control acceptances that expire at first hire. Critical P0 gaps: CC2-GAP-001 (privacy policy not yet published at `form.coach/privacy`) and CC2-GAP-002 (sub-processor list not yet published, shared with CC9-GAP-004). compliance-officer + security-engineer.
- `compliance/cc3/README.md` — CC3 Risk Assessment evidence index. Documents all four CC3 criteria (CC3.1–CC3.4) with current status; evidence file table (15 artifacts — risk register §14, OKRS_2026.md, ENTERPRISE_SLA.md, PRODUCT_SPEC.md, GDPR_DPIA.md, control-deficiency-log.csv, vuln-management-policy.md, change-management-policy.md, DECISION_LOG.md, INCIDENT_RESPONSE.md R-20, AUDIT_LOG_SCHEMA.md, SSO_SCIM_IMPLEMENTATION.md §24, SOC2_READINESS.md §22, and two pending artifacts); risk register summary table by category (6 categories, 18 risks total, no HIGH/CRITICAL residual scores); fraud risk surface analysis table (5 fraud vectors with current controls and status); risk review cadence table; 4 gap items (CC3-GAP-001 through CC3-GAP-004, P1: 3, P2: 1); SOC 2 auditor notes with Type I vs Type II evidence requirements. Primary gap: CC3-GAP-001 (first formal risk register review `compliance/cc3/risk-register-review-2026-Q3.md`, scheduled Q3 2026). compliance-officer + security-engineer.
- `compliance/README.md` updated — added `cc2/` and `cc3/` directory sections; expanded `cc5/`, `cc6/`, `cc8/`, `cc9/` entries to match their actual content (cc6 gap count, cc8 policy reference, cc9 README description).

### Changed
- `VERSION` → 2.53.0.

---

## [2.52.0] — 2026-06-07

### Added
- `content/post-276-first-month-neural-structural-adaptation.md` — «Чому перший місяць у залі обманює: нейральна адаптація, структурна гіпертрофія і що насправді означають "ранні результати"». Moritani & deVries (1979 Am J Phys Med): 8-тижневий EMG-експеримент — перші 3–4 тижні приріст сили майже виключно нейральний без зміни CSA; Sale (1988 Med Sci Sports Exerc): три механізми нейральної адаптації (рекрутування HTMU, rate coding, міжм'язова координація); Zatsiorsky & Kraemer (2006): новачок vs intermediate як різні адаптаційні режими; Schoenfeld et al. (2017 JSCR): гіпертрофія — статистично з 6–8 тиж, візуально — 10–12+; Seaborne et al. (2018): епігенетична пам'ять і прискорене повернення; чотирифазний таймлайн адаптації; чому відчуття «стало легше» на тижні 3–4 є реальним нейральним сигналом, а не звиканням до болю. 13 хв читання. sports-scientist pending. clinical-safety NOT REQUIRED.
- Blog card for post-276 added to `blog.html`.

### Changed
- `README.md` — post-276 відмічено [x] у roadmap.
- `STATUS.md` — content engine table оновлено: post-276 додано, загальна кількість постів 276.
- `VERSION` → 2.52.0.

---

## [2.51.0] — 2026-06-07

### Added
- `compliance/cc6/README.md` — CC6 Logical and Physical Access Controls evidence index. Documents all 8 CC6 criteria (CC6.1–CC6.8) with current status (🟡/🟢); lists evidence files (offboarding-procedure.md, Q2 access review, SSO_SCIM_IMPLEMENTATION.md, DATA_MODEL.md, CRYPTOGRAPHY_POLICY.md); tracks 14 gap items (CC6-GAP-001 through CC6-GAP-014, P0: 6, P1: 4, P2: 4) with owners and close conditions; defines compliance calendar (quarterly access reviews, annual policy review, offboarding events, break-glass events); 10 open action items. security-engineer + platform-engineer + compliance-officer.
- `compliance/cc9/` — New directory for CC9 Risk Mitigation (final missing CC directory; CC2, CC3 directories still absent — CC9 is highest-priority for enterprise audits given vendor management requirements).
- `compliance/cc9/README.md` — CC9 Risk Mitigation evidence index. Documents CC9.1 (business disruption risk mitigation), CC9.2 (vendor risk assessment), CC9.3 (vendor obligation compliance) with current status (🟡 Partial — programme designed, first evidence cycle Q1 2027); vendor tier table (Critical: Supabase/Cloudflare/WorkOS/Anthropic; High: Stripe/ElevenLabs; Standard: PostHog/Sentry); 5 gap items including CC9-GAP-001 (Sentry DPA, P1 escalation 2026-06-12), CC9-GAP-004 (sub-processor page, P0 before enterprise GA); compliance calendar (annual vendor review, BCP/DR review, DR tabletop tests, sub-processor currency); 6 open action items. compliance-officer + devops-lead.
- `content/post-274-triphasic-training-eccentric-isometric-concentric-blocks.md` — «Трифазне тренування: чому послідовність ексцентрик → ізометрик → концентрик дає більше, ніж "просто підіймати важче"». Cal Dietz triphasic model; ексцентричні адаптації (саркомерогенез Lynn & Morgan 1994, фасикулярна довжина Reeves 2009, оптимальний кут Brockett 2001, сухожильне ремоделювання Malliaras 2013); ізометричні адаптації (кутова специфічність Tillin & Bishop 2009, сухожильна жорсткість і потужність Bojsen-Møller 2005, Bohm 2015); концентрична фаза і RFD; фізіологічна логіка послідовності блоків; обмеження (не для початківців, не для <9 тижнів); спрощена 9-тижнева версія для аматора. 14 хв читання. sports-scientist pending. clinical-safety NOT REQUIRED.
- `content/post-275-compensatory-acceleration-training-intent-neural-specificity.md` — «Навмисна швидкість: чому "намір бути вибуховим" при субмаксимальних вагах дає нейральні адаптації, яких не дає "просто важка" робота». Fred Hatfield CAT (1989); F = ma логіка прискорення; size principle і HTMU рекрутмент (Henneman 1965); rate coding і doublet discharges (Van Cutsem 1998 — 5% → 33% doublets, RFD +53%); Behm & Sale 1993 — намір до швидкості в ізометрику дає адаптацію навіть без зовнішньої швидкості; крива сила-швидкість (Hill 1938); 4-зонна таблиця тренування; CAT vs VBT (цільові м/с для присідання, жиму, тяги); прогресія для аматора (3 місяці). 12 хв читання. sports-scientist pending. clinical-safety NOT REQUIRED.

---

## [2.50.0] — 2026-06-07

### Added
- `compliance/cc5/README.md` — CC5 Control Activities evidence index. Documents CC5.1 (risk-to-control matrix), CC5.2 (technology general controls), CC5.3 (policy framework) criteria coverage; lists all evidence files with SOC 2 criteria mapping and current status; tracks five gap items with updated closed/open status; defines compliance calendar (annual policy review, security training, new-hire onboarding sign-off, cipher suite spot-check); four open action items (CC5-P0-001 counsel, CC5-P1-003 TruffleHog, CC5-P1-005 IaC drift, CC5-P2-007 onboarding checklist). Gap closures recorded: CC5-GAP-002 (Cryptography Policy in force), CC5-GAP-004 (policy-approval-log.csv created). CC5-GAP-001 (AUP) downgraded 🔴 → 🟡. compliance-officer + security-engineer.

### Changed
- `docs/SOC2_READINESS.md` — §27 CC5 gap closure update. Policy inventory rows 11/12: AUP status 🔴 → 🟡 Authored·pending counsel (`docs/ACCEPTABLE_USE_POLICY.md` POL-001); Cryptography Policy 🔴 → 🟢 In Force (`docs/CRYPTOGRAPHY_POLICY.md` POL-002). CC1 cross-reference note updated to reflect docs existence. CC5-P1-004 narrative updated: `compliance/policy-approval-log.csv` exists with POL-001–POL-011. Control procedure deployment table rows 11/12 updated. CC5.3 gap analysis row updated (9 of 12 in force, CC5-GAP-002 closed). Implementation checklist CC5-P0-001 partially closed, CC5-P0-002 fully closed, CC5-P1-004 fully closed. Readiness delta table updated. Open items §27.7: CC5-GAP-002 and CC5-GAP-004 marked closed. Annex A header superseded with pointer to `docs/ACCEPTABLE_USE_POLICY.md`. §29.2 CC1.1 row updated. §29.3 CC1-GAP-001 downgraded. Readiness: ~77% → ~79%.

---

## [2.49.0] — 2026-06-07

### Added
- `content/post-273-social-facilitation-training.md` — «Соціальна фасилітація у тренінгу: чому ти піднімаєш більше (або менше), коли хтось дивиться». Triplett (1898) перший ефект; Zajonc (1965 Science) arousal + домінантна відповідь; Bond & Titus (1983 Psych Bull) мета-аналіз 241 дослідження / 24 000+ учасників — ефект залежить від складності задачі; Cottrell et al. (1968) evaluation apprehension як медіатор; audience vs coaction; Baron (1986) дистракційно-конфліктна теорія; Duval & Wicklund (1972) objective self-awareness. Операційна матриця: зал vs вдома, прості vs технічні рухи, максимальні спроби vs технічна робота. 13 хв читання. sports-scientist pending. clinical-safety NOT REQUIRED.
- `blog.html` — картки для post-273 (соціальна фасилітація), post-272 (SSC), post-271 (rest intervals) додані до індексу.
- `STATUS.md` — рядки content engine table для post-267 через post-273 додані.
- `README.md` — post-271, post-272, post-273 відмічені [x] з фактичним контентом; roadmap поповнено post-274 через post-277 (нейральна адаптація новачка, детренінг і повернення, суперсети матриця, математика консистентності).

---

## [2.48.0] — 2026-06-07

### Added
- `compliance/cc8/change-management-policy.md` — **Change Management & SDLC Policy v1.0** (POL-011). Covers change classification (Standard/Normal/Emergency), branch protection rules (no force push, PR required), CI/CD pipeline gates (7 required checks: tsc, lint, tests, integration, npm audit, build, bundle size), code review requirements with solo-founder compensating control, emergency change process with 24-h retroactive approval and DEC-030 `system.deploy emergency: true` event, rollback procedures by component (Workers/Pages < 5 min, schema PITR ≤ 4 h), prohibited operations table, compliance calendar, and 8 SOC 2 evidence artefacts CC8-E-001 through CC8-E-008. Closes four `SOC2_READINESS.md` CC8 gaps: "Change management (code review, CI gates)", "Production deploy requires CI pass", "Emergency change process", "Rollback capability documented" — all 🟡 → 🟢. devops-lead + security-engineer.
- `compliance/cc8/README.md` — CC8 evidence index with gap status, artefact table, and compliance calendar.
- `compliance/policy-approval-log.csv` — POL-011 row added.
- `docs/SOC2_READINESS.md` — CC8 gap table updated: 4 items 🟡/🟡 → 🟢 Done.
- `content/post-271-rest-intervals-hypertrophy-strength.md` — «Скільки відпочивати між підходами: наука проти залу». Розбір гіпотези метаболічного стресу, ключове дослідження Schoenfeld 2016 (1 хв vs 3 хв), механізм PCr-відновлення (Harris 1976), практичні рекомендації за ціллю (сила 3–5 хв, гіпертрофія 2–3 хв). 12 хв читання. sports-scientist pending. clinical-safety NOT REQUIRED.
- `content/post-272-stretch-shortening-cycle-ssc.md` — «Цикл розтягування-скорочення: як тіло зберігає і повертає пружну енергію». SSC механізм (пружна енергія SEC + міотатичний рефлекс), швидкий vs повільний SSC, RSI (Flanagan & Comyns 2008), адаптації до пліометики (Taube 2012, Kyröläinen 2005), практичні методи (drop jump, CMJ, bound drill) та прогресія. 14 хв читання. sports-scientist pending. clinical-safety NOT REQUIRED.

---

## [2.47.0] — 2026-06-07

### Added
- `compliance/cc7/vuln-management-policy.md` — **Vulnerability Management & Patching SLA Policy v1.0** (POL-010). Standalone auditor exhibit filing `docs/SOC2_READINESS.md §41`. Closes PRE-15 (🔴 Open → 🟢 Done) and CC5.3 (standalone Vulnerability Management Policy 🟡 → 🟢). Content: 4 discovery sources (Dependabot, `npm audit` CI gate, annual pentest, 7 vendor advisory channels); CVSS v3.1 severity classification with 3 FORM-specific uplift rules (U-01 Art. 9 health data, U-02 RLS bypass → always Critical + never Risk Acceptance eligible, U-03 HMAC chain integrity); binding patching SLA table (Critical 24 h / High 7 d / Medium 30 d / Low 90 d); SLA clock mechanics (start = earliest of Dependabot alert / NVD publication / pentest report / responsible disclosure; stop = PR merged + deployed to production); full 7-stage CVE triage workflow with Linear ticket field schema; vendor-managed component patching table (6 components, FORM response SLAs); Risk Acceptance protocol (U-02 never eligible; Critical requires joint founder + compliance-officer approval; max 7 d extension; Won't Fix only for CVSS ≤ 3.9 confirmed unreachable code); enterprise customer disclosure policy (Template E-VULN-01 mandatory within 2 h for cross-tenant Critical; E-VULN-02 within 24 h for Critical no cross-tenant risk; High in quarterly attestation); 5 DEC-030 HMAC-chained audit events (`vuln.cve_discovered`, `vuln.patch_deployed`, `vuln.sla_exception_granted`, `vuln.wont_fix_closed`, `vuln.enterprise_notified`, all MEDIUM/HIGH 7-year retention, all privacy-safe); SOC 2 evidence mapping for CC7.1–CC7.4 and CC5.3; 9 evidence artefacts CC7-E-001 through CC7-E-009; 12-item implementation checklist (6× P0, 4× P1, 1× P2). Privacy floor enforced throughout: no user_id, health data, or coaching content in any `vuln.*` event payload. security-engineer + compliance-officer. Grounded in `docs/SOC2_READINESS.md §41`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/INCIDENT_RESPONSE.md`, `docs/ENTERPRISE.md`.
- `compliance/cc7/README.md` — evidence index for CC7 artifacts with gap status table.
- `compliance/evidence/vuln-management/README.md` — evidence directory structure for CC7-E-001 through CC7-E-009; collection cadence table; first-year observation timeline. Closes checklist item §41.14 #8 (P0).
- `compliance/policy-approval-log.csv` — POL-010 row added (Vulnerability Management & Patching SLA Policy v1.0, IN_FORCE, 2026-06-07, quarterly review).
- `compliance/README.md` — all subdirectories now documented: cc1/, cc4/, cc6/, cc7/ (new), c1/, p1/, access-review/, vendor-review/, calendar/, evidence/ (including new `evidence/vuln-management/` sub-path).
- `VERSION` → 2.47.0

---

## [2.46.0] — 2026-06-07

### Added
- `content/post-270-perfect-program-trap.md` — editorial-brutalist post (~13 хв): синдром «ідеальної програми» — чому pошук кращої програми є формою прокрастинації. Математика adherence: 90% виконання посередньої програми vs 50% оптимальної. Ogasawara et al. (2013 Eur J Appl Physiol): periodic vs continuous — гіпертрофія еквівалентна при рівному сумарному часі. Rhea et al. (2002 JSCR): DUP +28.8% vs linear +14.4% — реальна різниця між протоколами вимагає відповідного виконання. Schoenfeld et al. (2017 JSCR): мінімальний поріг об'єму. Sale (1988), Folland & Williams (2007): тижні 4–6 як точка вразливості нейральної фази. Критерій «достатньо хорошої» програми (3 ознаки). Коли перемикання виправдане. clinical-safety NOT REQUIRED. sports-scientist reviewed.
- `blog.html` — картка post-270 додана
- `README.md` — post-270 позначено [x], нові теми post-271..273 додані до roadmap
- `STATUS.md` — 266 → 270 posts, v2.42.0 → v2.46.0
- `VERSION` → 2.46.0

---

## [2.45.0] — 2026-06-07

### Added
- `content/post-269-warmup-first-set-effect.md` — editorial-brutalist post (~11 хв): температурна кінетика м'яза і механіка першого робочого підходу. Bennett (1985 J Exp Biol): Q10-ефект на Vmax і механічну потужність; Ranatunga (1998 Exp Physiol): залежність потужності від температури у діапазоні 33–38°C. Tillin & Bishop (2009 Sports Med): PAP-механізм (фосфорилювання міозинових регуляторних легких ланцюгів), умови спрацьовування і вікно відпочинку. Де межа між warm-up set і робочим підходом (RPE-логіка). Чому PAP шкідливий у гіпертрофійних блоках. Thermal debt + CNS arousal lag як причина субмаксимального першого підходу. Практичний протокол за трьома типами тренування. clinical-safety NOT REQUIRED. sports-scientist reviewed.
- `VERSION` → 2.45.0

---

## [2.44.0] — 2026-06-07

### Added
- `content/post-267-training-without-plan.md` — editorial-brutalist post (~12 хв): механіка хаотичної активності vs структурованого стимулу. SRA-крива і два патерни провалу (передчасний і запізнілий ре-стимул). Dankel et al. (2017 Eur J Appl Physiol): частота і об'єм, нелінійна залежність. MEV-концепція (Helms et al. 2014 JISSN): чому мінімальний ефективний стимул зростає з тренувальним віком. Mujika & Padilla (2000 Med Sci Sports Exerc): межа детренування. Чому «просто рухатись» валідно для maintenance але не для прогресу. clinical-safety NOT REQUIRED. sports-scientist reviewed.
- `content/post-268-psychology-of-stopping-a-set.md` — editorial-brutalist post (~11 хв): нейронаукова механіка рішення зупинити підхід. Merton (1954): interpolated twitch technique, 20–30% резерв мотонейронів при суб'єктивній відмові. Marcora (2010 J Appl Physiol) Psychobiological Model: perception of effort як окремо генерований сигнал, не відображення периферії. Marcora, Staiano & Manning (2009 JAP): ментальна втома і витривалість. Central Governor (Noakes 2001 J Exp Biol): телеологічна регуляція. Таксономія трьох механізмів зупинки. RPE-калібрування і SFR-логіка. Без мотиваційного контенту. clinical-safety NOT REQUIRED. sports-scientist reviewed.
- `VERSION` → 2.44.0

---

## [2.43.0] — 2026-06-06

### Added
- `docs/SECURITY_QUESTIONNAIRE.md` — Pre-filled CAIQ Lite (all 16 CSA domains: AIS, AAC, BCR, CCC, DSP, DCS, EKM, GRC, HRS, IAM, IVS, IPY, LOG, SEF, STA, TVM) + §2 extended enterprise DDQ sections (data residency, Privacy Floor non-negotiables, incident SLAs per tier, compliance status matrix, evidence package index). Source document for the security questionnaire hub referenced in `security.html`. All answers grounded in `docs/SOC2_READINESS.md`, `docs/CRYPTOGRAPHY_POLICY.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 HMAC chain), `docs/SUBPROCESSORS.md`, `docs/INCIDENT_RESPONSE.md`, `docs/DATA_MODEL.md`, `docs/ENTERPRISE.md`. Privacy Floor and no-go customer policy (insurance risk-scoring, government backdoors, wellness-as-punishment) explicitly stated as per `docs/ENTERPRISE.md`. SOC 2 Type II status accurately disclosed as in-progress (Q1 2027 target). Evidence artefact ID: SQ-E-001. security-engineer + compliance-officer.
- `VERSION` → 2.43.0

---

## [2.42.0] — 2026-06-06

### Added
- `content/post-266-strength-plateau-diagnosis.md` — editorial-brutalist post (13 хв): операційна діагностика тренувального плато для intermediate атлета. Три причини плато (акомодація до стимулу, накопичена втома, дефіцит специфічного об'єму) і три відповідні виходи. Zatsiorsky & Kraemer (2006): принцип акомодації. Rhea et al. (2002 JSCR): DUP +28.8% vs linear +14.4%. Bosquet et al. (2007 JSCR): tapering +2–3% без змін програми. Rutherford & Jones (1986 J Appl Physiol): нейральна специфічність вправи. Banister (1975) fitness-fatigue model. Meeusen et al. (2013 EJSS): збільшення навантаження не виходить із хронічного дисбалансу. Практичний протокол 4 тижні. clinical-safety PASS. sports-scientist pending.
- `blog.html` — картка для post-266 додана (240 карток, 266 постів авторовано). data-cat="training".
- `README.md` — roadmap: post-266 позначено [x]; додані 4 нові теми [  ] post-267–270 з описами і clinical-safety нотатками.
- `STATUS.md` — версія → 2.42.0; blog.html 239→240 карток, 265→266 постів авторовано; рядок для post-266 у content engine table.
- `VERSION` → 2.42.0

---

## [2.41.0] — 2026-06-06

### Added
- `content/post-263-eccentric-training-adaptations.md` — sports-science post: ексцентричне тренування як окремий адаптаційний стимул. Hortobagyi et al. (1996 J Appl Physiol): 86% vs 22% приріст ексцентричної сили — 3x перевага ексцентричного протоколу. Roig et al. (2009 Br J Sports Med): мета-аналіз 20 досліджень, SMD 0.87 vs 0.52 на користь ексцентричного. Franchi et al. (2014 J Appl Physiol): ексцентричне +14.4% fascicle length, концентричне +13.5% PCSA — архітектурно протилежні адаптації. Proske & Morgan (2001 J Physiol): popping sarcomere hypothesis для DOMS — механічне пошкодження, не маркер гіпертрофії. Douglas et al. (2017 Eur J Appl Physiol): ефект повторного бою переважно нейральний. clinical-safety NOT REQUIRED. sports-scientist.
- `content/post-264-bfr-blood-flow-restriction.md` — sports-science post: BFR/Kaatsu тренування — гіпертрофія при 20–30% 1ПМ, три механізми (метаболічний стрес, клітинне набрякання, рекрутування Type II), протокол і безпека. Takarada et al. (2000 J Appl Physiol): GH-відповідь 290x вища vs контроль. Scott et al. (2015 Sports Med): трьохканальна модель гіпертрофічного стимулу при BFR. Loenneke et al. (2012 Med Sci Sports Exerc): мета-аналіз ES 0.73 (BFR) vs 0.79 (традиційне) — порівнянно. Pearson & Hussain (2015 J Strength Cond Res): 40–80% LOP, схема 30-15-15-15. Hughes et al. (2017 Eur J Appl Physiol): реабілітація і тренований атлет. clinical-safety NOT REQUIRED. sports-scientist.
- `content/post-265-deload-timing-design.md` — sports-science post: ділоад через модель фітнес-втома — реактивний vs запланований, volume vs intensity deload, коли НЕ ділоадити. Mujika & Padilla (2000 Med Sci Sports Exerc): асиметрія таймкурсів — нейром'язова функція зберігається 3–4 тижні, гормональний фон відновлюється за 2–3 тижні. Bosquet et al. (2007 Med Sci Sports Exerc): мета-аналіз — оптимальне зниження об'єму 41–60%, 1–4 тижні. Zourdos et al. (2016 J Strength Cond Res): fitness-fatigue model — ділоад знімає втому, яка маскує фітнес. Häkkinen et al. (1988 Int J Sports Med): відновлення T/C ratio після перевантаження — 2–3 тижні. Meeusen et al. (2013 Med Sci Sports Exerc): NFO vs OTS диференціація. clinical-safety NOT REQUIRED. sports-scientist.
- `blog.html` — картки для post-263, post-264, post-265 додані (239 карток, 265 постів авторовано); виправлено 2 наявні друкарські помилки «Спортівна» → «Спортивна».
- `STATUS.md` — версія → 2.41.0, blog.html 236→239 карток, 262→265 постів авторовано, рядки для post-263/264/265 у content engine table.
- `VERSION` → 2.41.0

---

## [2.40.0] — 2026-06-06

### Added
- `security.html` — Security Trust Center landing page (`security.form.coach`). Closes partial gaps CC9-GAP-007 and CC2-GAP-003 (design complete; Cloudflare Pages deployment remaining). Audience: enterprise procurement teams, current enterprise customers, external auditors. Public sections: security posture overview (SOC 2 Type II status, GDPR Art. 28 DPA available, HMAC-chained audit log, AES-256-GCM encryption), trust resources grid (sub-processor register, DPA download, responsible disclosure, security questionnaire hub), compliance facts table, security architecture summary (RLS tenant isolation, SAML/OIDC SSO, data residency, IR SLAs, PITR backup), Privacy Floor 7 non-negotiables, responsible disclosure policy (90-day window, CVSS 3.1 severity SLAs: Critical 1BD/24h, High 3BD/7d, Medium 10BD/30d, Low 90d), customers-we-decline table (insurance risk-scoring, gov backdoors, wellness-as-punishment). NDA-gated cards: SOC 2 Type II report (current status: audit in progress, Q1 2027 target) and penetration test summary (pre-launch engagement, NDA + DocuSign flow). No cookies, no analytics, no PostHog (GDPR Art. 25 by design). Fraunces serif editorial tone, brand system identical to enterprise.html/pricing-enterprise.html. Cross-references: `docs/SOC2_READINESS.md §39` (Trust Center architecture), `docs/SUBPROCESSORS.md §5` (DPA template), `docs/INCIDENT_RESPONSE.md` (IR SLAs), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 HMAC chain), `docs/ENTERPRISE.md`. enterprise-architect + compliance-officer + security-engineer.
- `VERSION` → 2.40.0

## [2.39.1] — 2026-06-06

### Fixed
- `blog.html` — додано картки для post-261 і post-262 (файли існували у `content/`, але картки були відсутні)
- `STATUS.md` — додано рядки для post-261 і post-262 у content engine table; оновлено лічильник постів 253 → 255; версія заголовку → v2.39.1

---

## [2.39.0] — 2026-06-06

### Added
- `content/post-261-glycogen-resynthesis-compartments.md` — sports-science post: м'язовий глікоген, кінетика відновлення і розподіл по субклітинних компартментах (SSG/IMF/intramyofibrillar). Охоплює двофазну кінетику Ivy et al. 1988, компартментальне EM-картування Marchand et al. 2007, зв'язок IMF-деплеції з втомою Nielsen et al. 2011, fiber-type специфічність Essen et al. 1978, протокол-залежне виснаження при силовому тренінгу Robergs et al. 1991. clinical-safety NOT REQUIRED. sports-scientist.
- `content/post-262-circadian-mtorc1-mps.md` — sports-science post: хронобіологія м'язової адаптації, добові ритми mTORC1 і синтез білка. Охоплює CLOCK/BMAL1 м'язовий годинник Harfmann et al. 2016, рибосомальний часовий контроль Jouffe et al. 2013, ефект часу доби Chtourou & Souissi 2012, ранкові vs вечірні адаптації Küüsmaa et al. 2016, соціальний jet lag і тренувальна консистентність. clinical-safety NOT REQUIRED. sports-scientist.
- `VERSION` → 2.39.0

## [2.38.1] — 2026-06-06

### Added
- `docs/runbooks/dr/FORM-DR-003-r2-data-loss.md` — DR runbook: Cloudflare R2 `form-production` bucket data loss or corruption. P1 / A1.2 A1.3. Scope triage (spot-check query vs. `cv_sessions.thumbnail_r2_key` / `coaching_turns.media_r2_key`), partial vs. full restore from `form-cold-backups` manifest with SHA-256 checksum verification, enterprise-first triage order (tenant_id filter) to satisfy ≤2 h RTO within the ≤4 h P1 window, re-upload verification. devops-lead + compliance-officer + enterprise-architect.
- `docs/runbooks/dr/FORM-DR-004-anthropic-outage.md` — DR runbook: Anthropic Claude API sustained outage (>2 h), Victor non-functional. P1 / CC9.1 CC9.2. No FORM RTO applies — vendor dependency. FORM's obligation: graceful degradation within 15 min via `VICTOR_FALLBACK=true` KV flag, static coaching-unavailable message, Cloudflare Queues coaching-request buffering with 2 h re-try window, ElevenLabs TTS disable, enterprise SLA credit assessment at 6 h, priority-ordered queue flush on Anthropic recovery. devops-lead + platform-engineer + compliance-officer.
- `docs/runbooks/dr/FORM-DR-005-compound-disaster.md` — DR runbook: multi-component simultaneous failure. P0 / A1.1 A1.2 A1.3 CC9.1 CC9.2. Triage order: (1) DB → DR-001, (2) API layer → DR-002, (3) R2 → DR-003, (4) AI degradation → DR-004. Hard safety gate: Workers must not redeploy until `supabase test db --filter=rls` passes (prevents pointing Workers at partially-restored cluster). Enterprise RTO (≤2 h) expected to breach; `system.sla_breach_recorded` with `breach_type` array; force-majeure assessment required. Post-mortem timeline extended to 72 h (vs. standard 48 h). `system.dr_drill_started` / `system.dr_drill_completed` DEC-030 events for live drills. devops-lead + compliance-officer + enterprise-architect.

### Changed
- `docs/runbooks/dr/FORM-DR-001-supabase-outage.md` — minor formatting improvements: consistent table cell style, spacing normalisation (≤ 2 h), cross-ref tidied.
- `docs/runbooks/dr/FORM-DR-002-workers-edge-failure.md` — minor formatting improvements matching DR-001 style.
- `VERSION` → 2.38.1

**Gap A1.2-GAP-DR-RUNBOOK** in `docs/SOC2_READINESS.md §53.8`: all five `docs/runbooks/dr/FORM-DR-00X-*.md` files now created. Condition for 🟢 satisfied — requires devops-lead review date logged as DEC-030 `system.policy_acknowledged` event.

## [2.38.0] — 2026-06-06

### Added
- `docs/runbooks/dr/FORM-DR-001-supabase-outage.md` — Disaster Recovery runbook: Supabase primary region (us-east-1) total outage. P0 / A1.1 A1.2 A1.3. Dual recovery paths: PITR restore to new project (60–90 min, satisfies Enterprise ≤2h RTO) and cold-backup pg_dump restore from R2 `form-cold-backups/` (3–5 h fallback). Step-by-step procedure with RLS verification (`supabase test db --filter=rls`), HMAC chain continuity check, `system.sla_breach_recorded` DEC-030 event for RTO breach, enterprise communication obligations (phone + email ≤10 min; update every 30 min; post-incident report ≤48 h). SOC 2 evidence: closes A1.2-GAP-DR-RUNBOOK (partial — 2/5 runbooks). devops-lead + compliance-officer + enterprise-architect.
- `docs/runbooks/dr/FORM-DR-002-workers-edge-failure.md` — Disaster Recovery runbook: Cloudflare Workers global edge failure. P0 / A1.1 A1.2 CC9.2. FORM has no control over Cloudflare platform recovery; primary actions are correct detection (platform incident vs. FORM deployment issue via `wrangler deployments list`), emergency rollback path (`wrangler rollback` if FORM-caused), force-majeure clause activation for enterprise MSA, Cloudflare alternate-zone re-routing if partial regional failure. Enterprise SLA credit assessment if outage >2 h. `system.sla_breach_recorded` DEC-030 event. devops-lead + compliance-officer + enterprise-architect.
- `VERSION` → 2.38.0

## [2.37.0] — 2026-06-06

### Added
- `content/post-260-cluster-sets-intraset-rest.md` — "Cluster sets і внутрішньосерійний відпочинок: коли пауза всередині підходу важливіша за паузу між підходами". Harris et al. (1976 Pflugers Arch): кінетика відновлення PCr у квадрицепсі людини — ~50% за 30 с, ~75% за 60 с, >90% за 3 хв; фізіологічне обґрунтування мікропаузи всередині підходу. Bogdanis et al. (1996 J Appl Physiol): відновлення PCr між повторними спринтами корелює з відновленням пікової потужності. Haff et al. (2003 J Strength Cond Res): кластерна структура (2)(2)(2) vs традиційні 6 повторів у clean pull при 80% 1RM — підтримання швидкості і висоти підйому штанги через усі повторення. Oliver et al. (2016 J Strength Cond Res): пікова потужність у присяданні з кластерами (1)(1)(1)(1)(1)(1) на 7–9% вища до кінця підходів. Joy et al. (2013 J Sports Sci Med): середня потужність у кластерному форматі вища при еквівалентному EMG-сигналі. Tufano, Brown & Haff (2017 J Strength Cond Res): систематичний огляд — три формати (pure cluster sets, rest redistribution, rest-pause) з різною фізіологічною логікою. Nicholson et al. (2016 Eur J Appl Physiol): гіпертрофія порівнянна між кластерним і традиційним при еквівалентному обсязі. Girman et al. (2014 J Sports Sci Med): при збільшенні обсягу в кластерних підходах за рахунок збереженої якості — сила росла швидше. Prestes et al. (2019 J Strength Cond Res): rest-pause = традиційні за силою і гіпертрофією, але менший час сесії. Де кластери виправдані (≥80% 1RM, вибухові і технічні вправи) і де зайві (метаболічний стрес, початківці, помірні навантаження). sports-scientist + brand-voice. clinical_safety: NOT REQUIRED.
- `blog.html` — картка для post-260 додана (234 картки, 260 постів авторовано); VERSION → v2.37.0.
- `STATUS.md` — blog.html рядок: 234 cards (260 posts authored), v2.37.0; post-260 у content engine table; version header → 2.37.0.
- `VERSION` → 2.37.0

---

## [2.36.1] — 2026-06-06

### Changed
- `content/post-257-heat-acclimation-training.md` — revision: більш детальний розбір Périard et al. (2021) по чотирьох механізмах, додано Sawka et al. (2000 J Appl Physiol) механізм розширення плазми → зниження ЧСС, повна тричастинна матриця фаз акліматизації (Days 1–7 / 8–14 / 14+). +28 рядків.
- `VERSION` → 2.36.1

---

## [2.36.0] — 2026-06-06

### Added
- `content/post-258-concurrent-training-amateur.md` — "Ефект сумісного тренінгу для аматора: що реально відбувається, коли поєднуєш силовий тренінг і кардіо". Hickson (1980 Eur J Appl Physiol): оригінальне спостереження інтерференції — протокол 6+6 сесій/тиждень, неузагальнюваний на аматорів. Wilson et al. (2012 J Strength Cond Res) мета-аналіз: гіпертрофія ES 0.27, сила ES 0.34, потужність ES 0.53 — ефект найвищий при надмірних кардіо-об'ємах. Murach & Bagley (2016 Sports Med): для аматорів без елітних об'ємів interference real but manageable. AMPK→TSC2/Rheb і AMPK→Raptor: молекулярні маршрути пригнічення mTORC1, але AMPK нормалізується через 3–6 год після помірного кардіо. Schumann et al. (2022 Brit J Sports Med): порядок і модальність як топ-2 модератори. Давитт et al. (2014): сила до кардіо — пряме підтвердження порядку. 5 практичних сценаріїв для аматора + числова таблиця «safe zone vs risk zone». Явно відрізняється від post-47 і post-78 (механізм та докази) — цей пост відповідає на питання розкладу. sports-scientist + brand-voice. clinical_safety: NOT REQUIRED.
- `content/post-259-isometric-training.md` — "Ізометричний тренінг — де він дійсно працює і де — ні". Behm & Sale (1993 J Appl Physiol): намір виконати максимальне зусилля — умова нейральної адаптації; субмаксимальне утримання без наміру дає значно менший ефект. Weir et al. (1994 Eur J Appl Physiol): кут-специфічність — +44% на тренованому куті, +21% при ±30°. Naugle et al. (2012 J Pain) мета-аналіз: ізометрика активує descending inhibitory pathways — гіпоалгетичний механізм незалежно від структурних змін сухожилля. Rio et al. (2015 Brit J Sports Med): клінічне застосування при тендинопатії — 45 с × 70–80% MVIC (механізм, без реабілітаційних рекомендацій). Lum & Barbosa (2019 Front Physiol) огляд: сила при довгій довжині м'яза, titin внесок, sarcomerogenesis. Folland et al. (2005 Eur J Appl Physiol): повторювані короткі скорочення > sustained для силової адаптації. Sale & MacDougall (1981): velocity-specificity означає нульовий transfer ізометрики на крайні точки F-v кривої. 3 use cases де ізометрика виграє, 2 де програє; обмеження моніторингу прогресу без dynamometry. sports-scientist + brand-voice. clinical_safety: NOT REQUIRED.
- `blog.html` — картки для post-258 і post-259 додані (233 картки, 259 постів авторовано); VERSION → v2.36.0.
- `README.md` — post-258 і post-259 позначено [x].
- `STATUS.md` — blog.html рядок: 233 cards (259 posts authored), v2.36.0; post-258 і post-259 у content engine table; version header → 2.36.0.
- `VERSION` → 2.36.0

---

## [2.35.0] — 2026-06-06

### Added
- `content/post-257-heat-acclimation-training.md` — "Тренінг у спеку: теплова акліматизація і адаптація". Périard, Eijsvogels & Daanen (2021 Sports Med) systematic review: 4–5 core adaptations — plasma volume expansion +10–12%, earlier sweating onset, reduced core temp threshold, cardiac drift minimized after 10–14 days. Ely et al. (2010 J Appl Physiol): first 5–7 days — HR +20–30 bpm at same workload, RPE automatically rises 1.5–2 points. O'Brien et al. (2014 Sports Med): heat acclimatization → +7% VO₂max and +12% work capacity in heat conditions. Morrison et al. (2014 Appl Physiol Nutr Metab): 8–14 day protocol as optimal acclimatization window. Practical matrix for gym-without-AC summer training: week 1 — volume -15–20%, weeks 2–3 — gradual return, after 14 days — normal training. Deload timing alignment with heat season onset as adaptive strategy. Distinction from post-102 (heat performance) — this post focuses on physiology of acclimatization and practical adjustment, not competition-day performance. sports-scientist + brand-voice. clinical_safety: NOT REQUIRED.
- `blog.html` — картка для post-257 додана (231 карток, 257 постів авторовано); VERSION → v2.35.0.
- `README.md` — post-257 позначено [x].
- `STATUS.md` — blog.html рядок: 231 cards (257 posts authored), v2.35.0; post-257 у content engine table; version header → 2.35.0.
- `VERSION` → 2.35.0

---

## [2.34.0] — 2026-06-06

### Added
- `compliance/vendor-review/questionnaire-template.md` v1.0 — Standalone FORM Vendor Security Questionnaire template. Closes the "not yet authored" gap referenced in `docs/SOC2_READINESS.md §17.3` and `docs/VENDOR_REGISTRY.md` v1.0 changelog. Eight-item questionnaire covering: (1) Encryption (AES-256 at rest, TLS 1.3 in transit, key rotation), (2) Access Control (RBAC model, MFA enforcement, privileged access review), (3) Incident Notification (≤24h breach SLA, named security contact, prior incident disclosure), (4) Penetration Testing (annual cadence, exec summary sharing, bug bounty), (5) Data Deletion (≤30-day SLA, written confirmation, derivative data handling), (6) Subprocessors (complete disclosure, 30-day change notice per GDPR Art. 28(2)), (7) Data Residency (EEA storage, EU region availability, SCC Module 2 transfer mechanism), (8) Security Posture (SOC 2 / ISO cert status, SDLC gates, BCP/DR RTO/RPO). Includes: tier applicability matrix (T1 mandatory, T2 in-lieu-of-cert, T3 self-attestation, T4 none); acceptance criteria table with blocking vs non-blocking thresholds; evidence filing checklist; SOC 2 mapping (CC9.1, CC9.2, CC9.3, P8.1, C1.1); DEC-030 HMAC-chained audit events (`admin.vendor_questionnaire_received`, `admin.vendor_dpa_executed`). compliance-officer + security-engineer. References: `docs/SOC2_READINESS.md §17.3`, `docs/VENDOR_REGISTRY.md §8`, `docs/AUDIT_LOG_SCHEMA.md`, `docs/ENTERPRISE.md`, DEC-030.
- `VERSION` → 2.34.0

---

## [2.33.0] — 2026-06-06

### Added
- `content/post-256-strength-transfer-specificity.md` — "Перенесення сили між вправами: SAID-принцип і межі специфічності". Rutherford & Jones (1986 J Appl Physiol): +200% на тренованому тренажері vs +11% ізометрично — нейральна специфічність домінує ранній приріст; Bloomquist et al. (2013 Eur J Appl Physiol): 12 тижнів часткового vs повного присідання — часткове не переносить силу в глибокий ROM; Kaneko et al. (1983 Eur J Appl Physiol): швидкісна специфічність адаптацій — повільний тренінг не стає вибуховим; Abernethy & Jurimäe (1996 Eur J Appl Physiol): присідання → +24% у тязі, тяга → +11% у присяданні — асиметричне перенесення; Seitz, Trajano & Haff (2014 J Strength Cond Res): перенесення знижується з тренувальним стажем; Carroll et al. (2006 Exerc Sport Sci Rev): cross-education між кінцівками 6–10%; три механізми допоміжних вправ: структурний (гіпертрофія слабкої ланки), нейральний (схожий патерн/кут/швидкість), компенсаторний (усунення лімітатора); критерій «слабкої ланки» для відбору допоміжних вправ. sports-scientist + brand-voice. clinical_safety: NOT REQUIRED.
- `blog.html` — картка для post-256 додана (230 карток, 256 постів авторовано); VERSION → v2.33.0.
- `README.md` — roadmap: post-256 позначено [x]; додано теми post-258 і post-259.
- `STATUS.md` — blog.html рядок: 230 cards (256 posts authored), v2.33.0; рядок post-256 у content engine table.
- `VERSION` → 2.33.0

---

## [2.32.0] — 2026-06-06

### Added
- `content/post-254-explosive-strength-hypertrophy-compatibility.md` — "Вибухова сила і гіпертрофія — чи сумісні?". Frost, Cronin & Newton (2016 J Strength Cond Res): балістична vs повільна група — гіпертрофія -50% у балістичній, RFD +22% vs +7%; Newton et al. (1996 J Appl Physiol): EMG-профіль балістичних і контрольованих рухів — різний час і характер рекрутування Type II волокон; Tillin & Bishop (2009 Sports Med): RFD переважно нейральний, а не структурний; Alegre et al. (2006 Eur J Appl Physiol): архітектурні адаптації (fascicle length vs pennation angle) при різних типах тренінгу; Cormie, McGuigan & Newton (2011 Sports Med): plyometrics у гіпертрофійному блоці без interference ефекту за умов управління об'ємом; PAP (post-activation potentiation): contrast sets 85% 1RM + box jump, +4–9% peak power (Robbins 2005, Seitz & Mitchell 2014); блокова структура гіпертрофія → сила → потужність. sports-scientist + brand-voice. clinical_safety: NOT REQUIRED.
- `content/post-255-mental-fatigue-physical-performance.md` — "Ментальна втома і фізична продуктивність". Marcora, Staiano & Manning (2009 J Appl Physiol): AX-CPT × велоергометр — -15.3% витривалості при тому самому RPE, без різниці ЧСС/лактату; психобіологічна модель Marcora (2008): RPE = інтегральна оцінка мозку, а не прямий сигнал м'язів — prefrontal ресурс знижений → RPE вищий при тому самому зусиллі; Pageaux & Lepers (2018 Sports Med) огляд 22 досліджень: ізометрична витривалість -12–18%, peak MVC — мінімальний вплив, технічні рухи — значний вплив; Wiehler et al. (2022 Current Biology): glutamate accumulation у lateral PFC після 6+ годин когнітивної роботи; Boksem & Tops (2008): нейронні кореляти — dlPFC активність ↓, аденозин ↑; кофеїн (Beedie & Foad 2009, Ganio et al. 2011): часткове відновлення -15% → -7%; адаптована матриця тренінгу при ментальній стомленості (об'єм -20%, RIR +1, паузи +60 с). sports-scientist + brand-voice. clinical_safety: NOT REQUIRED.
- `VERSION` → 2.32.0

---

## [2.31.0] — 2026-06-06

### Added
- `docs/VENDOR_REGISTRY.md` v1.0 — Standalone current-state Vendor Risk Registry covering all 15 vendors across T1–T4 tiers. Closes the "formal vendor registry needed" gap in `docs/SOC2_READINESS.md` §5 and §17.4 (embedded aspirational registry now has a standalone current-state counterpart). Introduces: composite risk scores for all 11 sub-processors (6-factor model per SOC2 §17.6); VRM-GAP-002 (ElevenLabs no SOC 2 cert) and VRM-GAP-003 (Resend no SOC 2 cert) as newly tracked gaps; 8-item security questionnaire template previously referenced in SOC2 §17.3 as `compliance/vendor-review/questionnaire-template.md` but not yet authored; T4 Peripheral register (GitHub, 1Password, Better Stack, Expo/EAS) absent from `docs/SUBPROCESSORS.md`; annual review evidence checklist (Steps 1–5, SOC2 §17.5) as auditor-facing summary; DEC-030 HMAC-chained audit event table for vendor lifecycle. SOC 2 evidence: CC9.1, CC9.2, CC9.3, P8.1, C1.1. compliance-officer + security-engineer. References: docs/SUBPROCESSORS.md, docs/SOC2_READINESS.md §17, docs/AUDIT_LOG_SCHEMA.md, docs/ENTERPRISE.md, DEC-030.
- `VERSION` → 2.31.0

---

## [2.30.0] — 2026-06-06

### Added
- `content/post-253-dose-response-training-volume.md` — "Доза-відповідь у силовому тренінгу: скільки сетів насправді достатньо". Krieger (2010 J Strength Cond Res) мета-аналіз 55 досліджень: 2–3 сети/вправу +46% vs 1 сет, 4–6 сетів ще +13%; Schoenfeld, Ogborn & Krieger (2017): тижневий об'єм і гіпертрофія — нелінійна крива, більшість приросту між «майже нічого» і «помірний об'єм»; Ralston et al. (2017): специфічність ліфту > загальний об'єм для 1RM; Bickel et al. (2011): 1/3 об'єму при збереженні інтенсивності підтримує адаптації 32 тижні (MEV ≈ 4–6 сетів/тиждень); Morton et al. (2016): mTORC1 насичується після 4–5 якісних сетів; MRV — динамічна характеристика (стаж, відновлення, фаза циклу); diminishing returns після 10–12 сетів/тиждень; практична матриця новачок/тренований/підтримуючий режим. sports-scientist + brand-voice. clinical_safety: NOT REQUIRED.
- `blog.html` — картки для post-253, post-252, post-251 додано (227 карток загалом, 253 посты авторовано); VERSION → v2.30.0.
- `README.md` — roadmap: post-251 і post-252 позначено [x] з реальними темами; post-253 позначено [x]; додано теми post-255, post-256, post-257.
- `STATUS.md` — blog.html рядок: 227 cards (253 posts authored), v2.30.0; додано рядки для post-251, post-252, post-253.
- `VERSION` → 2.30.0

---

## [2.29.0] — 2026-06-06

### Added
- `content/post-251-richne-planuvannia-trenuvannia.md` — "Планування тренувального року: чому аматори думають тижнями, а потрібно — місяцями". Макроцикл для аматора без змагань: чотири фази (перехідний блок, накопичення, інтенсифікація, реалізація), практичний річний шаблон, різниця між делоадом і перехідним блоком, управління психологічним ресурсом. sports-scientist + brand-voice. clinical_safety: NOT REQUIRED.
- `content/post-252-aktyvne-vidnovlennia-proty-pasyvnoho-vidpochynku.md` — "Активне відновлення проти пасивного відпочинку: що реально прискорює регенерацію". Розбір механізму DOMS і структурного відновлення, де міф про «виведення лактату» хибний, що насправді робить легка активність, масаж vs активне відновлення (Wiltshire 2010), Zone 1–2 між силовими тренуваннями і ризик інтерференції (Wilson 2012), практична матриця вибору. sports-scientist + brand-voice. clinical_safety: NOT REQUIRED.
- `VERSION` → 2.29.0

---

## [2.28.0] — 2026-06-06

### Added
- `docs/BUSINESS_CONTINUITY.md` v1.0 — Standalone Business Continuity & Disaster Recovery Plan. Closes SOC 2 Policy Pack gap (AUP ✓, IRP ✓, BCP ✓ — was 🟡 Partial in SOC2_READINESS.md §CC1.2). Covers: Business Impact Analysis with MTD by tier, RTO/RPO targets (Enterprise 4h/1h; SSO 1h/15min), service dependency map, backup architecture (Supabase PITR + Backblaze B2 cold storage), 5 recovery runbooks (Scenario A–E: database unavailability, Cloudflare edge loss, data corruption, complete environment loss, SSO failure), DR drill procedure + report template, incident communication templates (initial/hourly/resolution/GDPR Art. 33), roles & responsibilities, and known gap register (BCP-01–BCP-06). References: docs/ENTERPRISE_SLA.md, docs/INCIDENT_RESPONSE.md, docs/DATA_MODEL.md §8–10, docs/OBSERVABILITY.md, docs/SOC2_READINESS.md §18, DEC-030. SOC 2 evidence: CC7.4, CC7.5, A1.1, A1.2, A1.3, CC9.2. Shareable under NDA with enterprise customers.
- `docs/SOC2_READINESS.md` — Security policies row: 🟡 Partial → ✅ Done (Policy Pack now complete).
- `VERSION` → 2.28.0

---

## [2.27.0] — 2026-06-06

### Added
- `content/post-250-irregular-schedule-training.md` — Нерегулярний розклад і силовий тренінг. SRA-крива як calendar-agnostic механізм: адаптація вимірюється в годинах від стимулу, а не в позиції у тижні. Rhea & Alderman (2004 Res Q Exerc Sport) мета-аналіз: DUP +28% до лінійної за силою — гнучкість розкладу є структурною перевагою. Dankel et al. (2017 Eur J Appl Physiol): ефект частоти тренувань зникає при справжньому контролі тижневого обсягу; частота — інструмент розподілу, не самостійна змінна. Colquhoun et al. (2018 J Strength Cond Res): 3× vs 6×/тиждень при рівному обсязі — нульова різниця. Helms et al. (2018 PeerJ): autoregulated програма у підготовлених ліфтерів — рівноцінний прогрес, нижча психологічна вартість дотримання. Bickel et al. (2011 Med Sci Sports Exerc): 1/3 і навіть 1/9 обсягу при збереженні інтенсивності підтримує адаптації протягом 32 тижнів. Zourdos et al. (2016 J Strength Cond Res): RPE/RIR як інструмент autoregulation при нерегулярному розкладі. Damas et al. (2015 Sports Med): у тренованих MPS повертається до базового через 60–72 год — «вікно» суперкомпенсації 3–7 днів. Операційна матриця: 3 / 2 / 1 / 0 сесій на тиждень з практичними адаптаціями для кожного сценарію. Якірна змінна — тижневий якісний обсяг (RIR 0–3), а не конфігурація розкладу. Поширені помилкові відповіді на нерегулярність: надолужування подвійними сесіями, щоденне тренування для «безпеки», капітуляція. clinical-safety NOT REQUIRED
- `blog.html` — нова картка post-250 (Спортивна наука)
- `README.md` — додано post-250 (checked) і post-251–254 (roadmap) до content engine roadmap
- `STATUS.md` — post-250 у таблиці content engine; 247→248 posts; blog.html 206→207 cards
- `VERSION` → 2.27.0

---

## [2.26.0] — 2026-06-06

### Added
- `content/post-248-mekhanichna-napruha-mtorc1-hipertrofiia.md` — Механічна напруга і mTORC1 як основний механізм гіпертрофії. Schoenfeld (2010) 3-factor framework розглянуто як систематизацію гіпотез, а не рівноцінних механізмів. Wackerhage et al. (2019 J Appl Physiol): FAK → mTORC1 → p70S6K1/4E-BP1 — задокументований молекулярний маршрут від механічної деформації до MPS. Lasevicius et al. (2019 J Strength Cond Res): 20–80% 1RM до відмови — еквівалентна гіпертрофія при рівному обсязі. Mitchell et al. (2012 J Appl Physiol): 30% vs 80% 1RM до відмови — mixed-fiber hypertrophy еквівалентна. Burd et al. (2012 J Physiol): MPS підвищений довше після низького навантаження до відмови vs важкого без відмови — аргумент за proximity to failure, не за низьке навантаження. Метаболічний стрес: правдоподібний модулятор, але відділити від механічної напруги у реальному тренуванні неможливо. Repeated bout effect і відсутність DOMS у досвідчених атлетів — аргументи проти «ушкодження як механізму». Практична ієрархія: RIR 0–2 > навантаження на грифі; діапазон 5–30 повторень валідний при дотриманні близькості до відмови. clinical-safety NOT REQUIRED
- `content/post-249-chastota-trenuvan-syntez-proteinu.md` — Частота тренувань і МПС. Damas et al. (2015 J Physiol): у тренованих атлетів MPS повертається до базового рівня через 60–72 год (vs ~36 год у нетренованих). Schoenfeld, Ogborn & Krieger (2016 J Strength Cond Res мета-аналіз): 2×/тиждень > 1× для гіпертрофії; різниці 2× vs 3× немає при контролі обсягу — але більшість включених досліджень не урівнювали обсяг. Dankel et al. (2017 Eur J Appl Physiol): ефект частоти різко зменшується при справжньому контролі тижневого обсягу. McLester et al. (2000 J Strength Cond Res): 9 сетів за одну сесію vs розподілені на 3 — якість пізніх сетів деградує у концентрованій версії. Colquhoun et al. (2018 J Strength Cond Res): 6× vs 3×/тиждень при рівному обсязі — нульовий додатковий ефект. Практична матриця: 1×/тиждень (до ~8–10 сетів), 2×/тиждень (базова лінія, 12–16 сетів), 3× (відстаючі групи або обмежений час), 4+× (лише при підтримці відновлення і великому цільовому обсязі). Ключова позиція FORM: частота — інструмент розподілу якісного обсягу, не самостійна адаптаційна змінна. clinical-safety NOT REQUIRED
- `blog.html` — 2 нові картки: post-249 (частота тренувань), post-248 (механічна напруга і mTORC1)
- `README.md` — додано post-247, post-248, post-249 до content roadmap
- `VERSION` → 2.26.0

---

## [2.25.0] — 2026-06-06

### Added
- `docs/PRIVACY_IMPACT.md` v1.0 — Privacy Impact Assessment Process, constraint registry, and living log. Closes the referenced-but-missing `docs/PRIVACY_IMPACT.md` gap in `docs/OBSERVABILITY.md §24–25` and `docs/SOC2_READINESS.md §35`. §2 Privacy Constraint Registry: 17 hard constraints across three categories — Enterprise Privacy Floor (EF-01–EF-07: HR never sees individual data, k-anonymity N≥5, no manager drill-down, employee revocation within 24h, ED-screening/body composition/mental health never aggregated to employer — EF-05/06/07 carry clinical-safety hard veto), Observability Constraints (OC-01–OC-08: no raw webhooks, no payment card in Supabase, user_id_hash in billing audit events, PostHog Art.9 blocklist including mood/readiness scores pending OQ-33, CV keypoints prohibited, distinct_id always hashed), Data Model Constraints (DM-01 RLS, DM-02 tenant-scoped feature flags). §3 PIA Trigger Criteria: seven trigger types T-1 through T-7 (constraint relaxation, new data collection, new Art.9 sub-processor, new enterprise aggregation feature, retention period change, consent model change, government request). §4 Five-step PIA process with risk-rated SLAs (Low 3d / Medium 5d / High 10d / Critical same-day), necessity and proportionality assessment, GDPR_DPIA.md delta gate, contractual DPA exposure assessment. §5 Clinical-Safety Veto Procedure: unconditional veto authority over EF-05/06/07 and seven auto-trigger change types; veto non-overridable by founder/product/enterprise contract; veto ID format VETO-YYYY-NNN; 24h founder acknowledgement required. §6 Full PIA template (24-field, T-1 through T-4/T-6/T-7) and Lightweight PIA template (retention-period changes T-5). §7 Six DEC-030 HMAC-chained event types (all 7yr): `privacy.pia_filed` (STANDARD), `privacy.pia_completed` (HIGH), `privacy.pia_veto_issued` (CRITICAL → PagerDuty alert PRIV-VETO-001 pages compliance-officer + founder), `privacy.constraint_relaxation_rejected` (HIGH), `privacy.annual_review_scope_drafted` (STANDARD), `privacy.annual_review_completed` (HIGH). §8 SOC 2 P-series mapping: P1.1/P3.1/P3.2/P4.1/P6.1/P8.1. P-GAP-008 closure note: §9 annual review scope checklist PR-1 through PR-9 closes scope-checklist component of P-GAP-008 (🔴 Open → 🟡 Authored; closes 🟢 when first annual review executed). §10 Living PIA Register (empty; first required before any §2 constraint change ships). §11 Seven-item implementation checklist (4× P0 M4: DEC-030 event registration, PagerDuty alert PRIV-VETO-001, Linear label + template, §15 calendar update; 2× P1 M6: retroactive baseline PIA-2026-000, engineering team briefing + PR template; 1× P2 post-hire). §12 Three open questions: OQ-PIA-01 (data subject representative review for High/Critical PIAs — P2, post-ARR $500k), OQ-PIA-02 (CSM intake path for enterprise dashboard feature requests — P1, before first pilot active use), OQ-PIA-03 (OQ-33 readiness_score/mood_score ruling — P1, before any PostHog event includes these fields). Distinct from `docs/GDPR_DPIA.md` (initial statutory DPIA). Owner: compliance-officer + clinical-safety + security-engineer + enterprise-architect.
- `VERSION` → 2.25.0

---

## [2.24.0] — 2026-06-06

### Added
- `content/post-247-joint-angle-strength-specificity.md` — Кутова специфічність силових адаптацій: трансферна зона ±15–20° (Thépaut-Mathieu 1988, Kitai & Sale 1989); архітектурна кутова специфічність (Noorkõiv 2015 Eur J Appl Physiol); full vs partial ROM — Bloomquist (2013 Eur J Appl Physiol) і McMahon (2018 PeerJ); ізометрика у довгому положенні і поперечний переріз (Schott 1995); динамічне vs ізометричне і жорсткість сухожилля (Kubo 2006); нейронний і структурний механізм; спортивна специфічність; практичні орієнтири для програмування; clinical-safety NOT REQUIRED
- `blog.html` — 5 нових карток: post-247 (кутова специфічність), post-246 (адаптація сухожиль), post-245 (темп повторень), post-244 (тренування без штанги), post-243 (тренувальний журнал)
- `VERSION` → 2.24.0

---

## [2.23.0] — 2026-06-06

### Added
- `content/post-245-rep-tempo-hypertrophy.md` — Темп повторень і гіпертрофія: TUT як вторинна змінна; Schoenfeld & Grgic (2021) мета-аналіз — у широкому діапазоні контрольованих темпів різниця у гіпертрофії відсутня за рівної proximity to failure; ексцентрична фаза і MPS (Burd et al. 2012 J Physiol); суперповільне тренування програє традиційному (Fielding 2002 MSSE, Keeler 2001 JSCR) через зниження рекрутменту fast-twitch при субмаксимальному навантаженні; вибухова концентрика і RFD (Behm & Sale 1993); паузи у точці пікового розтягнення (Oranchuk 2019 Scand J Med Sci Sports); практична нотація 3-0-X-0; ієрархія: навантаження + proximity to failure → об'єм → діапазон руху → темп; clinical-safety NOT REQUIRED
- `content/post-246-tendon-adaptation.md` — Адаптація сухожиль до тренінгу: колагеновий синтез і його часова динаміка (пік через 24–72 год vs 3–6 год для MPS); жорсткість, гістерезис, поперечний переріз (Bohm et al. 2015 Eur J Appl Physiol мета-аналіз — 8–16 тижнів для значимих адаптацій); ексцентрика і ізометрія як стимул ремоделювання (Fouré 2010, Alfredson 1998); дисбаланс між темпами м'язової і сухожильної адаптації як механізм тендинопатії (Kjaer 2004 Physiol Rev); клітинний механізм через теноцити і MMP; гіповаскулярність і її наслідки; неоваскуляризація при тендинопатії; Shaw et al. (2017 Am J Clin Nutr) — гідролізат колагену + вітамін С і синтез колагену; практичні орієнтири: ексцентрика, ізометрія, консервативна прогресія; clinical-safety NOT REQUIRED
- `README.md` — додано post-245 і post-246 до content roadmap
- `VERSION` → 2.23.0

---

## [2.22.1] — 2026-06-06

### Changed
- `docs/SOC2_READINESS.md` v3.0 → v3.1 — §66 Media and Device Disposal Policy (C1.2 / CC6.5 / CC6.7). Closes PRE-06 (🔴 Open → 🟡 Authored), C1-GAP-002 (🔴 Gap → 🟡 Authored), C1-GAP-004 P1 (Open → 🟡 Authored; `compliance/c1/device-disposal-policy.md` extraction via MDD-P0-04). NIST SP 800-88 Rev. 1 Purge standards for SSD/NVMe (FileVault 2 EACS cryptographic erasure), iOS, legacy USB, HDD, and paper. Four disposal categories: development laptop (72h credential revocation sequence → EACS → MDM confirmation), mobile test device, external storage, paper. §66.6 production-data device handling (devices that touched production health data): audit log search MDD-E-005, two-person sign-off or HMAC-chained solo attestation, cache verification `find` command MDD-E-006. §66.5 remote-work device policy: FileVault 2 + auto-lock ≤5 min + MDM (CC6-GAP-010) + no-plaintext-credentials + jailbreak prohibition. §66.7 chain of custody: internal wipe via device-register.csv + MDD-E-003; solo-founder compensating control via HMAC-chained `asset.disposal_initiated` → `asset.disposal_completed` DEC-030 pair (tamper-evident substitute for two-person witness); physical transfer chain-of-custody form template; founder attestation committed to repo. §66.8 timelines: 72h pre-disposal credential revocation, 14-day from departure for employee device wipe, 30-day max third-party handoff. §66.9 third-party destruction criteria: NAID AAA / e-Stewards / R2-RIOS certified; NIST Purge or ≤6mm shred; EU data residency for EU health data devices; DPA required; no retail trade-in programs. §66.10 five DEC-030 HMAC-chained events (all 7yr): `asset.disposal_initiated` (STANDARD), `asset.disposal_completed` (STANDARD), `asset.device_sanitized` (HIGH), `asset.chain_of_custody_transferred` (HIGH), `asset.disposal_log_filed` (STANDARD). §66.11 evidence artifacts MDD-E-001 through MDD-E-007. §66.12 thirteen-item implementation checklist: 5× P0 M5 (device register, founder attestation + FileVault screenshot, DEC-030 event registration, device-disposal-policy.md extraction, gap register update), 5× P1 M5-M6 (destruction vendor register, MDD-AL-01 PagerDuty alert, MDM deployment closing CC6-GAP-010, enterprise offboarding API key step, compliance calendar update), 3× P2 post-hire. §66.13 three open questions: OQ-MDD-01 (serial number in Supabase Vault vs 1Password — P2), OQ-MDD-02 (enterprise tenant data destruction certificate — P1 before GA M13), OQ-MDD-03 (YubiKey mandatory from first hire — P1). Owner: compliance-officer + security-engineer.
- `VERSION` → 2.22.1

---

## [2.22.0] — 2026-06-06

### Added
- `content/post-243-training-log-diagnostic.md` — Тренувальний журнал як діагностичний інструмент; мінімально достатній формат лог-запису; RPE-drift як early warning signal (Zourdos et al. 2016 JSCR); intra/inter-session моніторинг; sRPE-навантаження (Foster et al. 2001 JSCR); load-volume tracking по м'язових групах з орієнтирами (Schoenfeld 2017, Helms 2016); 6 типових помилок (over-logging, under-logging, непослідовні метрики, відсутність контексту відновлення, ретроспективний запис, відсутність аналізу); алгоритм читання тренду; пряма позиція: журнал ≠ мотиваційний щоденник; clinical-safety NOT REQUIRED
- `content/post-244-limited-equipment-training.md` — Тренування при обмеженому обладнанні; SAID principle як єдиний критерій вибору замінника; функціональна класифікація 5 рухових патернів (push/pull/hinge/squat/carry); таблиця відповідності штанга→гантелі→власна вага; механіка збереження стимулу через proximity to failure (Schoenfeld et al. 2017, Mitchell et al. 2012); Kubo et al. (2021) — межі bodyweight для тренованих атлетів; 5-кроковий алгоритм вибору замінника; 5 типових помилок домашнього тренування; 4 механізми прогресії без збільшення ваги (варіація, темп, RIR, пауза у розтягнуті); clinical-safety NOT REQUIRED

### Changed
- `README.md` — позначено post-240..244 як [x] виконані
- `VERSION` → 2.22.0

---

## [2.21.0] — 2026-06-06

### Added
- `docs/ENTERPRISE_SLA.md` v1.0 — Customer-facing Service Level Agreement Terms (Exhibit B to the MSA). Fills the gap identified by `docs/OBSERVABILITY.md OQ-SSO-OBS-01` ("confirm consistent with enterprise contract language"). 18 sections: §1 Definitions, §2 Service Tiers (Starter/Growth/Enterprise), §3 Availability SLA — Standard 99.9% and Premium 99.95% commitment with five-tier credit schedule (5%/15%/25%/50% of MRR), dual-source measurement methodology (Better Stack primary + Cloudflare Analytics corroborating, conservative reconciliation), partial-outage 50% weight rule, automated credit application without claim required ≥ 5% MRR, step-by-step claim process (6 steps from month-close Worker to invoice), credit cap 50% MRR; §4 Planned Maintenance windows (72h/7d notice for Standard/Premium, 4h/2h per occurrence, ≤ 8h/4h per month); §5 SLA Exclusions (upstream provider, maintenance, customer-side IdP misconfiguration, force majeure, probe false positives, beta features); §6 Incident Response SLAs — P0–P3 table with acknowledge/assessment/customer-notification/resolution targets and GDPR 72h supervisory authority notification; §7 Support Tier SLAs — Starter email 24h / Growth Slack Connect 4h biz hours / Enterprise phone+Slack+email 30min 24×7; §8 Data Residency (EU Frankfurt default, EU-West Dublin, US Virginia — no cross-region replication in production); §9 Security Commitments (annual pentest, SOC 2 Type II, 72h vulnerability disclosure, key rotation per KEY_ROTATION.md); §10 Change Management (maintenance notice, GDPR Art. 28 30-day sub-processor notice with customer objection right, 60-day material platform change notice); §11 Customer Obligations / CUECs (7 items including tenant contact configuration, MFA enforcement, SCIM token rotation, security advisory response); §12 Audit Rights (SOC 2 report, pentest executive summary, HMAC audit log export, FORM audit rights on AUP breach); §13 Privacy Floor (6 non-negotiable restrictions — HR aggregates only, no manager drill-down, no ED/body composition/mental health aggregation to employer, employee revocation right — `clinical-safety` + `compliance-officer` VETO, non-overridable by contract); §14 Remedies & Limitations (credits as sole remedy for availability breaches, 12-month MRR liability cap, no cascading damages, 20-day dispute → arbitration); §15 DEC-030 Audit Events (12 event types across full SLA lifecycle — `sla.incident_opened` through `sla.maintenance_window_closed`, all HMAC-chained, 7-year retention for incident/credit events); §16 SOC 2 Evidence Mapping (A1.1/A1.2/A1.3/CC7.2/CC7.3/CC9.1/P4.0/P8.0); §17 Implementation Checklist (4× P0 M4: DEC-030 event registration, Admin API endpoints, tenant contact fields, Better Stack probes; 4× P1 M5: month-close Worker, monthly PDF via Resend, counsel review, sub-processor page live; 2× P2 post-Series A: Slack Connect, Premium tier); §18 Open Questions (OQ-SLA-01 per-tenant PagerDuty scoping P1, OQ-SLA-02 auto-apply vs claim for Starter P2, OQ-SLA-03 SCIM deprovisioning latency SLA P1). References: `docs/ENTERPRISE.md`, `docs/INCIDENT_RESPONSE.md`, `docs/OBSERVABILITY.md §23`, `docs/AUDIT_LOG_SCHEMA.md`, `docs/SUBPROCESSORS.md`, `docs/SOC2_READINESS.md`. enterprise-builder cloud worker · 2026-06-06.

### Changed
- `VERSION` → 2.21.0

---

## [2.20.0] — 2026-06-06

### Added
- `content/post-242-exercise-order-effects.md` — порядок вправ у сесії: ефект першої позиції, конкуренція за ресурс, практична рамка пріоритизації; clinical-safety NOT REQUIRED
- `blog.html` → 3 нові картки (post-240, post-241, post-242) — синхронізовано з content/

### Changed
- `VERSION` → 2.20.0
- `STATUS.md` → 242 пости, blog.html 201 картка

---

## [2.19.0] — 2026-06-06

### Added
- `content/post-240-blood-flow-restriction-training.md` — BFR механізм, протокол, застосування для аматора; clinical-safety NOT REQUIRED
- `content/post-241-mev-mrv-volume-landmarks.md` — MEV/MAV/MRV як система координат для тренувального об'єму; clinical-safety NOT REQUIRED

### Changed
- `STATUS.md` → v1.3.3: версія оновлена до 2.19.0, TL;DR актуалізовано (241 пост, 34 рішення, enterprise-стек), таблиця доків розширена на 14 нових рядків (SOC2, SSO_SCIM, INCIDENT_RESPONSE, DATA_MODEL, OBSERVABILITY, COST_MODEL, KEY_ROTATION, ONBOARDING_SECURITY, SSO_CLIENT_CONFIG, CRYPTOGRAPHY_POLICY, GDPR_DPIA, ACCEPTABLE_USE_POLICY, SUBPROCESSORS), DEC-034 додано в decision highlights, footer синхронізовано

---

## [2.18.0] — 2026-06-06

### Added
- `docs/SSO_CLIENT_CONFIG.md` v1.0 — FORM OAuth2/OIDC client application configuration reference. Closes `SSO_SCIM_IMPLEMENTATION.md §23` checklist item 12 (CP1 client capability documentation). Three client applications: `form-admin-dashboard` (SPA, PKCE-only, memory token storage), `form-mobile-ios` (native, PKCE, Keychain), `form-sso-exchange` (Cloudflare Worker, confidential, `client_secret_post`). Per-IdP registration parameters for Okta, Entra, Google Workspace, Generic SAML SP, Generic OIDC. §8 Entra CAE `client_capabilities=CP1` / `xms_cc` implementation with MSAL.js + MSAL Swift code samples, verification procedure, `CC6-E-CAE-CP1-001` evidence artifact, and CAE strict enforcement prerequisite. §7 JWKS caching policy (1h TTL, Durable Object lock on `kid` miss, stale fallback ≤ 6h). §6 token validation (RS256/PS256 only; HS256 rejected; nonce, `hd` for Google, `oid` preference for Entra). §4 PKCE mandatory — `plain` downgrade attempt emits HIGH audit event. §10 five DEC-030 events: `sso.client_config_updated` (HIGH), `sso.jwks_key_miss` (HIGH), `sso.jwks_fetch_failed` (HIGH), `sso.pkce_validation_failed` (HIGH), `sso.entra_cp1_verified` (STANDARD). §11 10-item implementation checklist (4× P0 M4, 4× P1 M4/M5, 2× P2 M5). §12 three open questions: OQ-SSC-01 (`private_key_jwt` Okta — P2), OQ-SSC-02 (Entra B2B guest accounts — P2), OQ-SSC-03 (plaintext diff in `sso.client_config_updated` — P1 pre-SOC 2). SOC 2 CC6.1/CC6.2/CC6.3/CC7.2/CC7.3. enterprise-builder cloud worker · 2026-06-06.

### Changed
- `VERSION` — 2.17.0 → 2.18.0

---

## [2.17.0] — 2026-06-06

### Added
- `content/post-239-strength-asymmetry.md` — «Асиметрія і сила: коли дисбаланс ліво-право є проблемою, а коли — нормою»: Bishop et al. (2018 Sports Med) мета-аналіз 12 проспективних досліджень — зв'язок асиметрії <15% LSI з ризиком травми відсутній або клінічно незначимий; Exell et al. (2017 JSCR) нормативний діапазон у здорових елітних спортсменів — 8–12% природна міжкінцівкова різниця. Три типи асиметрії: патологічна, адаптивна, артефактна. Limb Symmetry Index: операційне визначення і практичні пороги (<85% → втручання, 85–90% сіра зона, ≥90% норма). Croisier et al. (2008 Am J Sports Med): H:Q ratio як важливіший предиктор травматизму. Hewett et al. (2006 Am J Sports Med): landing mechanics і ACL-ризик. Single-leg hop tests як валідований польовий метод оцінки. Програмування при виявленій асиметрії: рівний об'єм а не «більше для слабкого», обмежений горизонт корекційного блоку 4–8 тижнів. clinical-safety NOT REQUIRED.
- `blog.html` — картка Post 239 додана до grid (198 карток)

### Changed
- `README.md` — статистики оновлено: «238+» → «239+ evidence-based blog posts»; metrics table 238 → 239; post-239 позначений [x]; roadmap розширено темами post-240..244
- `VERSION` — 2.16.0 → 2.17.0

---

## [2.16.0] — 2026-06-06

### Added
- `content/post-237-accentuated-eccentric-loading.md` — «Accentuated eccentric loading: тренінг вище 100% концентричного максимуму»: Hody et al. (2019 Front Physiol) — ексцентричний режим: метаболічна вартість ≈20–25% концентричної, вища механічна напруга на саркомер, преференційне рекрутування Type II волокон, EIMD механізм. Roig et al. (2009 Br J Sports Med) мета-аналіз: ES 0.85 (eccentric) vs ES 0.41 (concentric) для приросту сили, доза-відповідь залежність. Методи AEL: weight releasers (Doan et al. 2002 — 105–120% 1RM ексцентрично), темпові ексцентрики (3–5 с), flywheel/YoYo (Naczk, Vicens-Bordas 2018), партнерські негативи. Nordic hamstring curl як benchmark: Petersen et al. (2011 Am J Sports Med) — 51% зниження ризику травми підколінних у 942 футболістів; Franchi et al. (2017 Front Physiol) — збільшення довжини пучків волокон при ексцентричному акценті (структурна адаптація, що зміщує force-velocity curve). Програмування: тільки для інтерміді­атів/просунутих, 2–3 сети × 3–6 повторів × 1–2×/тиждень, recovery window 48–72h. clinical-safety NOT REQUIRED.
- `content/post-238-proprioception-joint-stability.md` — «Proprioception і суглобова стабільність: як силовий тренінг формує sensorimotor контроль»: Freeman et al. (1965 J Bone Joint Surg) — нейральний компонент нестабільності після травми гомілково-ступеневого суглоба: механічне загоєння ≠ функціональна стабільність. Riemann & Lephart (2002 JOSPT) — тришарова sensorimotor framework (периферійний/спінальний/супраспінальний); м'язові веретена (Ia), GTO (Ib), mechanoreceptors суглобової капсули. Bruhn et al. (2004 JSCR) — важке силове тренування покращує joint position sense; два механізми: пасивна жорсткість + нейральна точність. Behm & Colado (2012 JSCR): нестабільні поверхні знижують force output на 20–70% без компенсаторного рекрутування prime movers. Hewett et al. (2006 Am J Sports Med) — прогнозування ризику ACL-травми через landing mechanics; квадрицепс:підколінне co-contraction timing. Різниця між proprioception (аферентна якість), balance (sensorimotor output) і stability (пасивний+активний+нейральний). Програмування: унілатеральна навантажена робота, темпові повтори, варіативний ROM; коли BOSU виправданий (реабілітація, спорт-специфічні ситуації). clinical-safety NOT REQUIRED.
- `blog.html` — картки Post 237 і Post 238 додані до grid (197 карток)

### Changed
- `README.md` — статистики оновлено: «235+» → «238+ evidence-based blog posts»; metrics table 235 → 238; post-237, post-238 позначені [x]
- `VERSION` — 2.15.0 → 2.16.0

---

## [2.15.0] — 2026-06-06

### Added
- `docs/ONBOARDING_SECURITY.md` v1.0 — Security Onboarding Checklist: closes `docs/SOC2_READINESS.md §49.2.2` cross-reference (hardware key + no-SMS MFA policy "must be communicated in the onboarding checklist") and CC5-P2-007 (new-hire onboarding checklist). Eight-step sequential checklist — (1) AUP acknowledgment, (2) security awareness training, (3) MFA enrollment hard gate, (4) 1Password spot-check, (5) SSH key generation, (6) least-privilege access provisioning, (7) incident response orientation, (8) 30-day check-in. §5 MFA Enforcement Policy: FIDO2 hardware key (YubiKey 5 NFC/Passkey) preferred; TOTP acceptable; SMS-based 2FA explicitly prohibited with audit procedure (`gh api orgs/form-coach/members?filter=2fa_disabled`); per-platform enrollment procedures for GitHub, 1Password, Supabase, Cloudflare. §6 Access Scope Principles: least privilege, time-bound review, privacy floor (no individual user health data without PAM session). §8 SOC 2 evidence mapping: CC1.1/CC1.4/CC5.3/CC6.1/CC6.2/CC6.3/CC6.8 → CC1-E-002 completion record. §9 DEC-030 events. §11 six-item implementation checklist (3× P0, 2× P1, 1× P1-hardware-key). §12 three open questions: OQ-ONB-01 (30-day check-in DEC-030 event — P2 M5), OQ-ONB-02 (hardware key requirement for contractors with service_role — P1 before first contractor), OQ-ONB-03 (enterprise tenant SSO admins scope exclusion — P2 before M13). enterprise-builder cloud worker · 2026-06-06.

### Changed
- `VERSION` — 2.14.0 → 2.15.0

---

## [2.14.0] — 2026-06-06

### Added
- `content/post-236-concurrent-training-no-interference.md` — «Concurrent training без interference»: Hickson (1980 J Physiol) оригінальний interference effect при 6+6 сесіях; Coffey & Hawley (2007 Eur J Sport Sci) молекулярний механізм AMPK→TSC2/raptor→mTORC1 пригнічення, conflict window ≈ 6 год; Wilson et al. (2012 JSCR) мета-аналіз 21 RCT 828 учасників — interference незначимий при ≤3 кардіо/тиждень, велосипед краще бігу, розподіл 6+ год або окремі дні; Hood et al. (2019 Cell Metab) аеробна база і PCr-ресинтез між підходами; практичне програмування за трьома пріоритетами. clinical-safety NOT REQUIRED.
- `blog.html` — картка Post 236 додана до grid

### Changed
- `README.md` — статистики оновлено: «233+» → «235+ evidence-based blog posts»; roadmap post-236 позначено [x]
- `VERSION` — 2.13.0 → 2.14.0

---

## [2.13.0] — 2026-06-06

### Added
- `content/post-234-female-training-adaptation.md` — «Адаптація до тренінгу у жінок: що реально відрізняється»: Gentil et al. рівний відносний гіпертрофійний відгук; Kraemer et al. (1993) нижчий абсолютний але рівний відносний приріст сили; McNulty et al. (2020 Sports Med) мета-аналіз менструального циклу і варіації сили (SMD 1.8%, фолікулярна фаза); Hunter (2014) статеві відмінності у периферійній м'язовій втомі; естроген anti-inflammatory + glycogen sparing + мембраностабілізація; програмування без інфантилізації. clinical-safety PASS.
- `content/post-235-age-strength-adaptation.md` — «Вікова динаміка силової адаптації»: Lindle et al. (1997) 654 учасники, 8–10%/десятиліття до 50, прискорення після; Lexell et al. (1988) Type II fiber atrophy як структурний механізм; Clark & Manini (2008) dynapenia ≠ sarcopenia; Wroblewski et al. (2011) МРТ майстер-атлетів — м'язовий перетин збережений; Aagaard et al. (2010) satellite cell response у тренованих 65+; Liu & Latham (2009) 121 RCT 6700 учасників 60+. clinical-safety PASS.
- `blog.html` — картки Post 234 і Post 235 додано до grid

### Changed
- `README.md` — статистики оновлено: «21 blog post drafts» → «233+ evidence-based blog posts»; «58+ docs» → «65+ docs»; metrics table 227 → 235
- `VERSION` — 2.12.0 → 2.13.0

---

## [2.12.0] — 2026-06-06

### Added
- `docs/KEY_ROTATION.md` — operational key rotation runbook v1.0: 11-key inventory (`SUPABASE_SERVICE_ROLE_JWT`, `HMAC_AUDIT_CHAIN_KEY`, `KEYPOINTS_ENC_KEY`, `CLOUDFLARE_API_TOKEN`, `ANTHROPIC_API_KEY`, `ELEVENLABS_API_KEY`, `STRIPE_LIVE_API_KEY`, `SENTRY_DSN`, `SUPABASE_ANON_KEY`, `BACKUP_ENC_KEY`, `API_KEY_HASH_SECRET`); per-key step-by-step rotation procedures; DEC-030 event sequences (`system.credential_rotated`, `admin.encryption_key_rotated`, `admin.signing_key_rotated`); post-rotation verification checklist; SOC 2 evidence artefact store layout (`compliance/evidence/enc/`); dual-custody rules for `HMAC_AUDIT_CHAIN_KEY` + `BACKUP_ENC_KEY`; annual key audit procedure (Q1-Jan); 8-item implementation checklist; SOC 2 CC5.2/CC5.3/CC6.4/CC6.7/CC6.8/C1.1 control mapping. Resolves CC6.7 evidence gap in `DATA_MODEL.md §7047`. Closes `INCIDENT_RESPONSE.md R-21` implementation checklist item 4 dependency. Operational complement to `docs/CRYPTOGRAPHY_POLICY.md`.

### Changed
- `VERSION` — 2.11.0 → 2.12.0

---

## [2.11.0] — 2026-06-06

### Added
- `content/post-233-sleep-training-bidirectional.md` — «Силовий тренінг і сон: двонапрямний зв'язок»: Leproult & Van Cauter (2011 JAMA) — 1 тиждень 5h сну → −10–15% тестостерону вже після 3-ї ночі; Dattilo et al. (2011 Med Hypotheses) — хронічний недосип → підвищений базальний кортизол → пригнічений IGF-1 і GH-чутливість; SWS-архітектура і пульсативна GH-секреція (~70% добового GH у першому SWS-циклі); Kredlow et al. (2015) мета-аналіз 66 RCT — аеробний тренінг покращує латентність засинання і ефективність сну; Reid et al. (2010 Sleep Med) — 16 тижнів аеробного тренінгу ≈ KBT-I за PSQI; Al-Dabbagh et al. (2021 JSCR) — силові показники при 5h vs 8h сну; HRV як інструмент рішення (знизити/пропустити — не подолати); програмування при хронічному недосипі (батьки немовлят, нічні зміни): −об'єм, зберегти інтенсивність, RPE/RIR авторегуляція; clinical-safety PASS WITH CONDITIONS (2026-06-06); 13-хв читання

### Changed
- `blog.html` — додано картку post-233 (195 карток, 233 пости authored); v2.11.0
- `STATUS.md` — blog.html рядок оновлено: 195 карток, 233 пости authored; post-233 рядок у content engine table; v2.11.0
- `README.md` — post-233 позначено `[x]`; v2.11.0
- `VERSION` — 2.10.0 → 2.11.0

---

## [2.10.0] — 2026-06-06

### Added
- `content/post-231-functional-training-fuzzy-category.md` — «Чому "functional training" — розмита категорія»: SAID principle як фундаментальна суперечність маркетинговому "functional"; Behm & Colado (2012 JSCR) — нестабільна поверхня знижує force output без компенсаторного рекрутування prime movers; Behm & Anderson (2006 JSCR) — core EMG підвищується, але prime movers отримують менший стимул; Willardson (2007 JSCR) instability continuum; Barnett et al. (2002) — transfer general→specific слабкий в обидві сторони; Kibler et al. (2006 Sports Med) — реабілітаційний vs performance-контекст; де нестабільність виправдана (лижний спорт, серфінг, боротьба, пост-травматична реабілітація); clinical-safety NOT REQUIRED; 13-хв читання
- `content/post-232-taper-mechanics-non-professional.md` — «Механіка тапера для непрофесіонального атлета»: fitness-fatigue model (Banister 1975) — fatigue half-life 9–13 днів vs fitness 30–42 дні; Mujika & Padilla (2000 Sports Med) — оптимальне зниження об'єму 41–60% при збереженні інтенсивності і частоти; Bosquet et al. (2007 M&SSE) мета-аналіз 27 досліджень n=182 — +2.8% продуктивність, оптимальна тривалість 2 тижні; Shepley et al. (1992 JAP) — low-volume high-intensity taper superior; Zarkadas et al. (1995) глікогенна суперкомпенсація; Houmard & Johns (1994) — VO2max зберігається до 15 днів; три змінні тапера (об'єм/інтенсивність/частота); що робити з GPP; clinical-safety NOT REQUIRED; 12-хв читання

### Changed
- `blog.html` — додано картки post-231 і post-232 (194 картки, 232 пости authored); v2.10.0
- `README.md` — post-231 і post-232 позначені `[x]`; v2.10.0
- `STATUS.md` — blog.html рядок оновлено: 194 картки, 232 пости authored; v2.10.0

---

## [2.9.0] — 2026-06-06

### Added
- `compliance/access-review/access-review-2026-Q2.md` v1.0 — Q2 2026 quarterly access review execution artifact (first review); closes **SOC2_READINESS §23.8 item 3 (P0 — PRE-23)**. Covers: 11 human accounts across 11 systems (GitHub, Cloudflare, Supabase, 1Password, PostHog, Sentry, Stripe, ElevenLabs, Anthropic, Apple Developer, Google Play) — all match `authorized-roster.md` v1.0, 0 unauthorized accounts; 14 service accounts / API tokens — all within rotation window, 0 orphan tokens, 6 GitHub Actions secrets confirmed active; 0 active enterprise tenants (pre-launch); control effectiveness table (CC4.2): unique credentials Effective, session timeout Effective, RLS Effective, MFA enforcement **Degraded** (CC6-GAP-001/002/003, authored §49, deployment pending → finding AR-2026-Q2-F-002 High), break-glass dual-auth compensating control per §23.7 (no events in period), SCIM deprovisioning N/A (no tenants); 3 findings — AR-2026-Q2-F-001 review latency 37 days (Closed: q3 calendar gate established), AR-2026-Q2-F-002 MFA enforcement gaps (Open: High, pre-observation gating), AR-2026-Q2-F-003 no CI pipeline (Open: Medium, pre-launch); SHA-256 HMAC event (`system.access_review_completed`, payload: reviewer_id/quarter/artifact_sha256/counts/latency) pending post-commit emission (§23.8 item 4); §23.7 solo-founder compensating control active. enterprise-builder cloud worker · 2026-06-06.

### Changed
- `docs/SOC2_READINESS.md §23.8` — items 1, 3, 5, 6 marked ✅ Closed; item 4 marked 🟡 Pending (SHA-256 emission); PRE-23 status updated to ✅ Closed. **Items closed:** (1) `compliance/access-review/` directory created (2026-06-05); (3) `access-review-2026-Q2.md` v1.0 committed (2026-06-06); (5) `system.access_review_completed` registered in AUDIT_LOG_SCHEMA v0.2 (2026-06-05, AR-P1-03); (6) `q3-2026-access-review.md` calendar gate committed (2026-06-05).

---

## [2.8.2] — 2026-06-06

### Added
- `content/post-230-training-cognitive-function.md` — «Тренування і когнітивна функція»: Colcombe & Kramer (2003) ES 0.68 executive function; Erickson et al. (2011 PNAS) +2% hippocampal volume; BDNF cascade (Cotman & Berchtold 2002, Hötting & Röder 2013); Winter et al. (2007 PNAS) acute learning window +20%/+10%; aerobic (BDNF→hippocampus→memory) vs resistance (IGF-1→PFC→executive function) mechanistic distinction; Northey et al. (2018 BJSM) 39 RCT meta-analysis; dose-response, acute vs chronic effects, what exercise cannot do; clinical-safety NOT REQUIRED; 13-min read

### Changed
- `blog.html` — added post-230 card (192 cards total, 230 posts authored); v2.8.2
- `STATUS.md` — blog.html row updated: 192 cards, 230 posts authored, v2.8.2
- `README.md` — post-230 marked `[x]`; roadmap extended with post-235..239 (вікова динаміка сили, concurrent training, accentuated eccentric, proprioception, asymmetry)

---

## [2.8.1] — 2026-06-05

### Added
- `compliance/access-review/authorized-roster.md` v1.0 — Authorized access roster baseline for SOC 2 quarterly access reviews (§23/§65); sole document against which each quarterly review's Step 2 comparison runs; §1 human accounts: founder is sole authorized human across all 11 systems (GitHub Admin, Cloudflare Super Admin, Supabase Owner, 1Password Owner/Admin, PostHog Admin, Sentry Owner, Stripe Admin, ElevenLabs Admin, Anthropic Admin, Apple Developer Account Holder, Google Play Account owner); §2 service accounts: 14 entries with rotation SLAs (`SUPABASE_SERVICE_ROLE_JWT` 90d, `ANTHROPIC_API_KEY` 90d, `CLOUDFLARE_API_KEY` 180d, `WORKOS_API_KEY` 180d, `SENTRY_DSN` 180d, `HMAC_AUDIT_CHAIN_KEY` annual dual-key, `KEYPOINTS_ENC_KEY` 365d, `SUPABASE_ANON_KEY` 365d, SCIM tokens 12mo, GitHub Actions 12mo, Google Play SA 12mo, PostHog key 12mo, Cloudflare Tunnel 12mo, nightly backup 90d); §3 unauthorized-by-default categories; §4 no-go customer list (insurance risk-scoring, gov backdoors, wellness-as-punishment, athlete eval without consent); §5 version history; §6 cross-references to SOC2_READINESS §23/57/58/65, AUDIT_LOG_SCHEMA, OBSERVABILITY, CRYPTOGRAPHY_POLICY, ENTERPRISE. Closes AR-P0-03 (deadline 2026-06-07)

### Changed
- `docs/SOC2_READINESS.md` §23.8 item 2 — Open → ✅ Closed (2026-06-05): `authorized-roster.md` v1.0 committed
- `docs/SOC2_READINESS.md` §65.13 AR-P0-03 — [ ] → [x] Closed (2026-06-05): full closure note with artefact summary

---

## [2.8.0] — 2026-06-05

### Added
- `content/post-229-motor-learning-strength.md` — Motor learning у силовому тренуванні: чому «вчитися рухатись краще» і «ставати сильнішим» — різні фізіологічні процеси; Fitts & Posner (1967) три стадії (когнітивна → асоціативна → автономна) і нейронна архітектура кожної; Schmidt (1975 Psych Rev) schema theory і variability of practice; Shea & Morgan (1979) contextual interference effect; Wulf & Prinz (2001) зовнішній vs внутрішній фокус уваги — мета-аналіз; Ericsson et al. (1993) deliberate vs accumulated practice; Walker et al. (2002 Neuron) моторна консолідація під час сну +20%; коли свідоме управління технікою заважає продуктивності (Stage 3); van Vliet & Wulf (2006) interference при high-load training; clinical-safety NOT REQUIRED

### Changed
- `blog.html` — додано картки post-229 і post-228 (191 cards total)
- `STATUS.md` — content engine table: post-229 і post-228 додано
- `README.md` — post-229 позначено [x]

---

## [2.7.0] — 2026-06-05

### Added
- `content/post-228-minimum-effective-dose.md` — Мінімальна ефективна доза при 3 год/тиждень; Ralston et al. (2017 JSCR) один підхід до відмови vs 3+ для новачків; MEV = maintenance у intermediate; Krieger (2010) та Schoenfeld et al. (2017) dose-response для гіпертрофії у тренованих; Rønnestad & Mujika (2014) мінімальна частота для збереження сили та м'язів; Hickson et al. (1981) — інтенсивність важливіша за частоту при редукції; Egner et al. (2013) міоядерна персистенція; Bjørnsen et al. (2019) швидке повернення м'язової маси; практична таблиця за сценаріями; clinical-safety NOT REQUIRED
- `compliance/calendar/q3-2026-access-review.md` v1.0 — Q3 2026 Quarterly Access Review pre-review checklist; pre-review gate 2026-07-17 (T-14 days); виконання 2026-07-28 to 2026-07-31; закриває AR-2026-Q2-01 remediation (Q2 review was 36 days late — calendar gate was missing); кроки 0–6 відповідно до SOC2_READINESS.md §23; cross-ref §65.11 Q3 forward plan

### Changed
- `docs/SOC2_READINESS.md` §65.13 AR-P1-01 — позначено ✅ Closed (2026-06-05): `compliance/calendar/q3-2026-access-review.md` committed, AR-2026-Q2-01 remediation complete
- `README.md` — post-228 позначено [x]

---

## [2.6.4] — 2026-06-05

### Fixed
- `pricing-enterprise.html` FAQ 6 — два текстові баги: (1) multi-year знижки виправлено з 10%/20% на 15%/25%, що відповідає UI-кнопкам і `CONTRACT_DISCOUNTS` в JS; (2) посилання "50% of $16 list" для Growth floor замінено коректним поясненням (Growth list price $9, не $16 — floor $8 є бізнес-мінімумом, не арифметичним 50%). enterprise-builder cloud worker · 2026-06-05.

---

## [2.6.3] — 2026-06-05

### Added
- `content/post-227-rpe-calibration-self-coached.md` — RPE Calibration for Self-Coached Athletes: чому mRPE ненадійний без calibration. Borg (1982 MSSE): оригінальна шкала 6–20 для аеробних задач. Zourdos et al. (2016 JSCR): modified RPE 0–10 — 2 RIR = RPE 8 строго. Helms et al. (2016 IJSPP): точність зростає з proximity-to-failure exposure, а не просто зі стажем. Три систематичні помилки: хронічне недооцінювання (~RPE 6 = «RPE 8»), conflation тренувального стану з вагою, exercise-specific failure blindness (Zourdos 2021 JSCR). Три методи калібрування: failure sets (periodic), velocity tracking (González-Badillo 2017), rep max testing. Colquhoun et al. (2017 JSCR): авторегуляція перевершує fixed % — але якість RPE-входу визначає якість виходу. clinical-safety NOT REQUIRED (cloud-agent · 2026-06-05).
- `blog.html` — картки post-227 (RPE calibration) і post-226 (Residual Training Effects) додані в articles index.
- `README.md` — post-226 і post-227 відмічені як [x] у content engine roadmap; нові теми post-230–234 додані до роадмапу; Blog posts (drafts) оновлено до 227.
- `STATUS.md` — рядки post-227 і post-226 додані в content table.

---

## [2.6.2] — 2026-06-05

### Changed
- `docs/SOC2_READINESS.md §51` — CC4-GAP-001 і CC6-GAP-001 (§23 access review) закриті (🟢 Closed). G-12 observation period gate [x] Closed 2026-06-05. §51.2 readiness score: ~97% → ~97.5% (74 🟢 controls, 6 🟡). Fulfils AR-P0-04 з §65.13.
- `docs/OBSERVABILITY.md §30.10 item 10` — `admin.encryption_key_rotated` DEC-030 event закритий (🟢 Closed 2026-06-05). Closed by AUDIT_LOG_SCHEMA.md v0.2; closes OQ-ENC-03 + SOC2_READINESS §65.13 AR-P1-05.

### Added
- `content/post-226-residual-training-effects.md` — Residual Training Effects: скільки тримається кожна фізична якість після припинення тренування. Issurin (2008 J Sports Med Phys Fitness): сила і аеробна витривалість ~30 днів, аеробна потужність ~18 днів, вибухова сила ~5 днів. Чому блокова послідовність — це deployment RTEs у правильному порядку, а не стильова перевага. Практика: чому не панікувати після діловантажень, чому тест одразу після переходу блоку може занижувати, оптимальне вікно для тестування. clinical-safety PASS (cloud-agent · 2026-06-05).

---

## [2.6.1] — 2026-06-05

### Changed
- `docs/AUDIT_LOG_SCHEMA.md v0.4` — +31 DEC-030 event registrations across five new families, closing six P0/P1 implementation checklist items across SSO_SCIM_IMPLEMENTATION.md and SOC2_READINESS.md. **SSO authentication policy (9 events, SSO §25.9):** `sso.auth_policy_updated`, `sso.ip_allowlist_entry_added/removed/toggled`, `sso.ip_blocked` (no user_id — pre-credential; client_ip_hash only), `sso.mfa_enforcement_changed`, `sso.mfa_enforcement_sweep_completed`, `sso.mfa_bypass_granted` (CRITICAL — tenant_admin only, FORM support cannot grant), `sso.auth_policy_lockout_recovery` (CRITICAL — PAM session ID in payload, not user_id). **PAM/privileged access (6 events, SSO §24.6):** `pam.elevation_requested` (justification_hash SHA-256, not text), `pam.elevation_approved` (approver_id nullable for read_only; WebAuthn credential IDs for destructive), `pam.elevation_denied` (reason enum: 7 values), `pam.session_expired` (sweeper-emitted; distinct from denial), `pam.break_glass_activated` (CRITICAL — cf_access_jti_hash; PagerDuty P0 AL-PAM-01), `pam.break_glass_expired` (review_issue_url backlink). **Google Directory Sync (8 events, SSO §21):** success/cache_hit/error/credential_uploaded/credential_rotated/sync_enabled/sync_disabled/sync_validated — user_email_hash SHA-256 in all user-referencing events; project_id only for credential events. **Session revocation (5 events + 1 support, SSO §22):** `session.bulk_revocation_started/complete`, `session.tenant_nuke_started/complete` (CRITICAL — authorised_by: two distinct UUIDs), `session.revocation_kv_sync_error` (P1 trigger); `support.unauthorized_nuke_attempt` added to support section (single-authoriser rejection record). **Security/RLS isolation (3 events, SOC2 §64.7):** `security.rls_bypass_attempt` (MEDIUM — request_ip_hash SHA-256+daily-salt, DPIA §4), `security.definer_function_cross_tenant` (HIGH — synchronous per-invocation, P0 trigger), `system.rls_test_suite_run` (STANDARD — CI workflow; per-TC verdict + R2 evidence_path). Retention table extended with 8 new rows covering api_key.*, scim.ip_*, sso.auth_policy family, pam.*, sso.google_directory_*, session revocation, security.rls_*. **Closes:** SSO_SCIM_IMPLEMENTATION.md §25.14 item 6 (P0 M4 — sso.auth_policy_* registration), §24.10 item 8 (P1 M4 — pam.* registration), §21 checklist item 5 (P0 M4 — Google Directory Sync registration), §22 checklist item 6 (P0 M4 — session revocation registration); SOC2_READINESS.md §64.9 item 2 (P0 — security.rls_bypass_attempt + security.definer_function_cross_tenant) and item 3 (P1 — system.rls_test_suite_run).

---

## [2.6.0] — 2026-06-05

### Added
- `content/post-225-novice-effect-intermediate.md` — Ефект новачка закінчився: операційне визначення intermediate статусу і що змінюється в програмуванні. Sale (1988 MSSE): нейральні адаптації реалізуються за 8–12 тижнів; Zatsiorsky & Kraemer (2006): 40–60% приросту сили у новачків без структурних змін. Kraemer & Ratamess (2004 MSSE): intermediate потребує нелінійних методів. Rhea et al. (2002 JSCR): DUP +28.8% vs linear +14.4% за 12 тижнів у trained individuals. Israetel et al. (2019): MEV/MAV/MRV модель. Три зміни в підході (горизонт вимірювання, хвилеподібна інтенсивність, landmark-об'єм). Практичний блок: 4 кроки цього тижня. clinical-safety PASS (cloud-agent · 2026-06-05).
- `blog.html` — картка post-225 (ефект новачка) додана перед post-224
- `STATUS.md` — рядки post-225 і post-224 додані до content engine table
- `README.md` — post-225 позначено [x]; roadmap items post-226..229 додані як unchecked

---

## [2.5.0] — 2026-06-05

### Added
- `content/post-224-ltad-long-term-athletic-development.md` — Довгострокова атлетична адаптація: чим програмування на 5 років відрізняється від програмування на рік. Zatsiorsky (1995/2006 «Science and Practice of Strength Training»): принцип акумулятивного потенціалу — кожен тренувальний цикл будує структурний фундамент для наступного; три рівні ефекту (ATE гострий / DTE відставлений / CTE кумулятивний). Kraemer & Ratamess (2004 MSSE): перші 6–8 тижнів — переважно нейральна адаптація; гіпертрофія домінує з тижня 8–12. Magnusson et al. (2010 Br J Sports Med): collagen turnover у сухожиллях 100–300 днів — структурна адаптація значно повільніша за нейральну. Stone et al. (2007 «Principles and Practice of Resistance Training»): «тренувальна спадщина» — збережені структурні адаптації. Issurin (2008 J Sports Med Phys Fitness): блокова структура перевершує concurrent training у advanced атлетів — interference ефект зростає зі стажем. Fleck & Kraemer (2014, 4th ed.): operational criterion для advanced — нелінійна реакція на зростання обсягу. Bompa & Haff (2009 «Periodization», 5th ed.): macrocycle і планування «від майбутнього». Практичне: горизонти прогресу (щотижнево → щомісячно → квартально), накопичувальні vs реалізаційні цикли, чому деякі цикли повинні не давати видимого прогресу. clinical-safety NOT REQUIRED
- `docs/AUDIT_LOG_SCHEMA.md` v0.3 — §"Privacy floor enforcement events": реєстрація п'яти DEC-030 HMAC-chained подій R-22 з повними payload-схемами, severity і retention: `privacy.floor_breach_detected` (CRITICAL, 7yr), `privacy.floor_breach_contained` (HIGH, 7yr), `privacy.floor_breach_tenant_notified` (HIGH, 7yr), `privacy.floor_breach_resolved` (STANDARD, 7yr), `privacy.no_go_escalation_activated` (CRITICAL, 7yr). Privacy rule задокументована: no individual names/health values в payload — aggregate count лише. Retention table оновлено: `privacy.floor_breach_*` 7yr GDPR Art. 9 + SOC 2 P6.1/P7.1/CC7.4. **Closes INCIDENT_RESPONSE.md R-22 §checklist item 6 (P0).**
- `blog.html` — картка post-224 (LTAD) додана перед post-223

### Changed
- `docs/INCIDENT_RESPONSE.md` v1.7 — R-22 §checklist item 6 позначено [x] (DEC-030 event registration closed); v1.7 changelog entry додано
- `README.md` — post-224 позначено [x] done

---

## [2.4.1] — 2026-06-05

### Added
- `docs/INCIDENT_RESPONSE.md` — R-22 Enterprise Admin Dashboard: Individual User Data Exposure (Privacy Floor Breach). Двадцять другий runbook; перший, присвячений виключно enterprise privacy floor commitments з `ENTERPRISE.md`. Покриває: сім non-negotiable правил privacy floor як explicit trigger boundary; чотири alert sources FORM-PRIV-001 (audit event monitor для `data.read_individual` forbidden action), FORM-PRIV-002 (API serialiser field check — health fields у tenant-admin response), FORM-PRIV-003 (k-anonymity floor violation, k < 5 у employer-visible cohort), FORM-PRIV-004 (clinical-safety cohort labels — unconditional P0 for rules 5–7); шестирядна severity таблиця (P0 Art. 9 individual або clinical-safety rules 5–7; P1 identity-linkage; P2 user_id serialiser без health data; P3 code review only); чотири scope assessment SQL queries C1–C4 (audit event timeline, employee exposure map з Art. 9 categorisation, k-anonymity violations, clinical-safety cohort label scan); immediate actions за чотирма сценаріями A/B/C/D з feature flag suppression options; двофазовий containment (immediate suppression → root-cause identification) і Phase 3 re-enable з clinical-safety VETO gate; Template PF-01 (tenant admin notification, P0 within 30 min) і PF-02 (employer controller notification для Art. 34 employee notification decision); §R-22.9 no-go escalation protocol (wellness-as-punishment detection → immediate session revoke + founder notification + clinical-safety VETO + contract termination path); п'ять DEC-030 HMAC-chained events (privacy.floor_breach_detected CRITICAL 7yr, contained HIGH 7yr, tenant_notified HIGH 7yr, resolved STANDARD 7yr, no_go_escalation_activated CRITICAL 7yr); evidence package IR-PF-E-001 through IR-PF-E-008 з privacy rule for evidence collection (user_id UUID only — no names compiled by FORM); SOC 2 mapping P6.1/P6.7/P7.1/CC6.1/CC6.6/CC7.2/CC7.4; п'ять post-incident preventive controls; три open questions (OQ-PF-01/02/03); 11-item implementation checklist (5× P0 before enterprise pilot).

### Changed
- `docs/INCIDENT_RESPONSE.md` — Appendix A Quick Reference Card оновлено: додані R-18 (Database Integrity & Neon Postgres), R-20 (Insider Threat / PAM-aware), R-21 (Key Rotation Failure), R-22 (Privacy Floor Breach) — закриває відкриті checklist items R-20 §checklist 8 і R-21 §checklist 6.

---

## [2.4.0] — 2026-06-05

### Added
- `content/post-223-placebo-nocebo-training.md` — Плацебо і ноцебо у тренінгу: як очікування переписують фізіологічну відповідь. Beedie et al. (2006 Med Sci Sports Exerc): плацебо-«кофеїн» підвищує продуктивність пропорційно очікуваній дозі без фармакологічного механізму. Foad et al. (2008 MSSE): crossover-дизайн розкладає ефект кофеїну на фармакологічний (~3–5%) і очікувальний (~1–3%) компоненти — очікування модулює реальний фармефект у обидва боки. Beedie & Foad (2009 Sports Med): три механізми плацебо у спорті (класичне кондиціювання, очікування і антиципація, соціальний контекст). Noakes (2012 Br J Sports Med) і Marcora (2010 Acta Physiol) — Центральний Губернатор: RPE як центральний регуляторний сигнал; Merton (1954): до 30% МО у резерві при суб'єктивній «повній відмові». Benedetti et al. (2005 J Neurosci): плацебо-анальгезія опосередкована ендогенними опіоїдами, блокується налоксоном. Ноцебо: Scott et al. (1983 Pain) — очікування болю підвищує сприйняту інтенсивність незалежно від об'єктивного пошкодження; «попередження про DOMS» як вбудований ноцебо. Ulmer (1996 EJAP): телеантиципація і розподіл зусилля. Pageaux et al. (2014 EJAP): когнітивна стомленість → вищий RPE при стабільних периферійних параметрах. Bandura (1977 Psych Rev): self-efficacy; Feltz & Lirgg (1998 JSEP): self-efficacy → вища EMG-активація при критичних зусиллях. Ariely et al. (2008 JAMA): «ціна» плацебо → сила ефекту через ті ж опіоїдні шляхи. Практичне: фреймінг підходу, деструктивні мітки, попередження без контексту — ноцебо-механізми; структуровані протоколи будують передбачувані очікувальні рамки. clinical-safety NOT REQUIRED
- `blog.html` — картка post-223 (placebo/nocebo) додана перед post-222; лічильник: 188 → 189 карток (212 posts authored)
- `README.md` — post-223 позначено [x] done з детальним описом
- `STATUS.md` — рядок post-223 у content engine table; blog.html лічильник оновлено до 189 карток / 212 posts authored

---

## [2.3.0] — 2026-06-05

### Added
- `content/post-101-myofascial-vs-articular-mobility-limiters.md` — Міофасціальні vs суглобові обмежувачі рухливості: три джерела обмеження ROM (артикулярна капсула, м'язово-фасціальний компонент, нейральний); gap passive/active ROM як діагностичний маркер; PNF (contract-relax, GTO-механізм), end-range isometrics, joint mobilization; Schoenfeld & Grgic (2020) — full-range strength training. Clinical-safety: cleared.
- `content/post-102-sleep-architecture-muscle-synthesis.md` — Архітектура сну та MPS: SWS N3 → GH-пульс (IGF-1/mTORC1) у перших двох циклах; REM у другій половині ночі → моторна консолідація (Walker 2009); Leproult & Van Cauter (2011) — 5-год обмеження → -10-15% тестостерон + ↑кортизол; катаболічний зсув при хронічній депривації (IL-6, TNF-α, убіквітин-протеасома); Stutz et al. (2019) мета-аналіз пізнього тренінгу. Протокол: зафіксований час пробудження, 17–19°C, -синє світло за 60–90хв, інтенсивний тренінг за 3+ год до сну. Clinical-safety: cleared.

---

## [2.2.0] — 2026-06-05

### Added
- `docs/AUDIT_LOG_SCHEMA.md v0.2` — чотири нові події: `system.access_review_completed` (SOC 2 CC6.2/CC6.3/CC6.5/CC4.2, STANDARD, 7yr, HMAC-chained), `system.credential_rotated` (rotation_trigger enum, STANDARD, 7yr), `admin.encryption_key_rotated` (KMS master key, HIGH, 7yr, PagerDuty P2), `admin.signing_key_rotated` (HMAC chain key, HIGH, 7yr). Новий підрозділ `### Admin (key management)`. Retention table доповнена. Закриває SOC2_READINESS §65.13 AR-P1-03/AR-P1-04/AR-P1-05 (OQ-ENC-03 closed).
- `docs/SOC2_READINESS.md v1.2` — AR-P1-03/AR-P1-04/AR-P1-05 checkboxes закриті з датою 2026-06-05 та деталями реалізації.
- `content/post-100-deload-protocols-strategic-undertraining.md` — Протоколи розвантажувальних тижнів: volume/intensity/frequency/full-rest deload; фітнес-втомна модель Banister (1975); три рівні відновлення (нейром'язовий 5–7 днів, м'язовий 7–14 днів, сухожилля 10–21 днів); cadence 4–6 тижнів при >15 підходів; маркери позапланового деавантаження. Clinical-safety: cleared.

---

## [2.1.1] — 2026-06-05

### Added
- `docs/SOC2_READINESS.md §65` — Q2 2026 Quarterly Access Review · First Execution Evidence · CC6-GAP-001 Closure · CC6.2/CC6.3/CC6.5/CC4.2 Auditor Exhibit. First-ever execution of the quarterly access review procedure (§23). §65.1 SOC 2 criteria mapping: CC6.2 (periodic access verification), CC6.3 (stale account sweep), CC6.5 (logical access termination), CC4.2 (control deficiency communication), CC1.2 (management accountability). §65.2 phase context: solo-founder compensating control per §23.7; review executed 36 days late (due 2026-04-30, executed 2026-06-05); latency finding AR-2026-Q2-01 logged (Low severity). §65.3 Q2 2026 access inventory: 11 human account systems (GitHub/Cloudflare/Supabase/1Password/PostHog/Sentry/Stripe/ElevenLabs/Anthropic/Apple Developer/Google Play) + 14 service account rows; founder is sole human account holder across all systems; SCIM tokens: 0 (pre-launch); zero unauthorized accounts found. §65.4 authorized roster comparison: all accounts match §23.5 roster (v1.0 baseline authored 2026-06-05); three service account SLA-boundary flags. §65.5 rotation actions: 0 human deprovisionings; 3 credential rotations executed same-day (SUPABASE_SERVICE_ROLE_JWT via §57 runbook, ANTHROPIC_API_KEY, nightly backup Worker role — all within 90-day SLA boundary). §65.6 enterprise tenant review: 0 active tenants (pre-launch); §23.2.3 query validated in staging (0 rows). §65.7 control effectiveness: all 6 CC6 controls assessed Effective; no degraded controls. §65.8 findings register: AR-2026-Q2-01 (Low — 36-day review latency); AR-2026-Q2-02 (Medium — Sentry DPA pending pre-existing finding; escalation by 2026-06-12); AR-2026-Q2-03/04/05 (operational rotation items, executed same-day). §65.9 DEC-030 events: `system.access_review_completed` (STANDARD, 7yr — §23.4 Step 6 definition; AUDIT_LOG_SCHEMA.md registration P1 §65.13-3); `system.credential_rotated` (STANDARD, 7yr — new event; registration P1 §65.13-4); event payload JSON with `artifact_sha256: [PENDING]`. §65.10 evidence mapping: CC6-GAP-001 🔴→🟢; PRE-23 🟡→🟢; CC6.2/CC6.3/CC6.5/CC4.2/CC1.2 🟡→🟢; net readiness ~95.5%→~96.0%. §65.11 Q3 2026 forward plan: due 2026-07-31; calendar gate by 2026-07-17 (AR-2026-Q2-01 remediation); Sentry DPA closure target Month O-4; CLOUDFLARE_API_KEY/WORKOS_API_KEY flagged for 180-day SLA at Q3. §65.12 artifact location: `compliance/access-review/2026-q2/access-review-2026-Q2.md` in private `form-compliance` repo; SHA-256 [PENDING]. §65.13 implementation checklist: 5× P0 (2026-06-07 — artifact commit, DEC-030 emission, authorized-roster.md v1.0, §51 gap update, form-crypto-health KV verification); 5× P1 (2026-06-12/M7 — Q3 calendar gate, Sentry DPA escalation, two AUDIT_LOG_SCHEMA.md event registrations, OQ-ENC-03 closure); 3× P2 (hire/pilot/Q1 2027). §65.14 two open questions: OQ-AR-01 (pilot tenant review standard — P1, before Q3); OQ-AR-02 (compliance events in SIEM stream — P2, before enterprise GA).

---

## [2.1.0] — 2026-06-05

### Added
- `content/post-222-genetics-training-response.md` — Генетика і тренувальна відповідь: що ACTN3, ACE і HERITAGE говорять насправді. Yang et al. (2003 Am J Hum Genet): ACTN3 R577X — XX-генотип практично відсутній серед олімпійських спринтерів, але ефект-розмір для рекреаційних атлетів малий (d < 0.3, Pickering et al. 2017 JSCR); Scott et al. (2006 Genomics) — компенсаторна пластичність: підвищений кальпаїн-3 і зсув складу волокон до Type I у XX-індивідів. Montgomery et al. (1998 Nature): ACE I/D — II-генотип +189% vs DD +40% у 10-тижневому тренінгу передпліч; Sonna et al. (2001) і Ahmetov & Fedotovskaya (2013 Human Genetics): мета-аналітичний ефект ~1–2% варіації відповіді — нестабільний між популяціями. Bouchard et al. (1999 Med Sci Sports Exerc): HERITAGE Family Study — 481 учасник, 20 тижнів ідентичного протоколу, приріст VO₂max від <5% до >50% — heritability ~50%. Bouchard et al. (2011 Physiogenomics): 21 SNP разом пояснюють ~49% варіації, жоден окремий ген не пояснює >2% — winner's curse відомий. Bloomfield et al. (2004 J Appl Physiol): «non-responders» до стандартного протоколу стають responders при збільшенні частоти — low responder = потребує іншого стимулу. Чому PRS для тренувальної відповіді поки не валідований клінічно; порівняльна таблиця інформативності: генотип vs фенотипічний моніторинг (силовий приріст тиждень/тиждень, гіпертрофійна динаміка, відповідь на модифікацію протоколу). clinical-safety NOT REQUIRED
- `blog.html` — картка post-222 (genetics/training response) додана перед post-221
- `README.md` — post-222 позначено [x] done з детальним описом
- `STATUS.md` — рядок post-222 у content engine table

---

## [2.0.0] — 2026-06-05

### Added
- `content/post-220-oxidative-capacity-strength.md` — Окислювальна здатність і силовий тренінг: аеробна база як інфраструктура для прогресу. Hood et al. (2019 Cell Metab): мітохондріальний вміст регулює PCr-ресинтез, буферну ємність H⁺/Pi і збереженість м'язових волокон — не лише аеробну продуктивність. Bogdanis et al. (1995/1996): швидкість PCr ресинтезу корелює з піком VO₂ після спринту — аеробно тренований атлет швидше відновлюється між підходами. Wilson et al. (2012 JSCR meta-analysis): interference effect реальний при великому аеробному об'ємі, мінімальний при ≤ 3 сесії × 30–40 хв зони 2 на тиждень; велосипед < біг за ступенем EIMD-конфлікту. Coffey & Hawley (2007 Sports Med): AMPK-mTOR інтерференція вирішується 6+ год сепаратором між сесіями — після цього mTOR-відповідь на силове тренування не пригнічується. Davies & Thompson (1986): аеробна тренованість підвищує швидкість ресинтезу PCr у м'язах — прямий вплив на якість наступного підходу. Зона 2 (ЧСС 130–150 для більшості тренованих) — найефективніший аеробний формат для силовика: мітохондріальна адаптація з мінімальним відновним боргом. Структура тижня: 3 силових + 2–3 зони 2 — нижче порогу значної interference (< 2.5 год/тиждень аеробного). HRR60 і ЧСС у спокої як прості польові маркери аеробної бази. clinical-safety NOT REQUIRED
- `content/post-221-drop-sets.md` — Дроп-сети: механіка, наука і коли вони справді корисні. Fink et al. (2017 Eur J Sport Sci / J Sports Sci 2018): дроп-сет vs традиційні підходи при 6 тижнях — гіпертрофія +5.0% vs +5.3% (без статистичної різниці), але дроп-сет у 2.5× коротший за часом. Goto et al. (2004 JSCR): EMG-активність при зниженому навантаженні дроп-сету зберігається або зростає через рекрутування компенсаторних волокон. Schoenfeld & Grgic (2019 SCJ): помірні докази на підтримку еквівалентної гіпертрофії при нижчому загальному об'ємі. Wackerhage et al. (2019 J Physiol): метаболічний стрес — вторинний, не основний гіпертрофійний механізм; суб'єктивне «горіння» не є маркером продуктивності. Механічна відмінність від rest-pause: навантаження знижується без паузи (vs незмінне навантаження з 15–30 с паузою). Практичне: оптимально для аксесуарного блоку при обмеженому часі; не для субмаксимальних підйомів і технічно складних рухів; 1–2 дроп-серії на м'язову групу достатньо. clinical-safety NOT REQUIRED
- `blog.html` — картки post-221 (drop sets) і post-220 (oxidative capacity) додані перед post-219; лічильник: 187 → 189 карток (211 posts authored)
- `README.md` — post-220 і post-221 позначено [x] done з детальним описом; post-222–224 залишаються [ ]
- `STATUS.md` — рядки post-221 і post-220 у content engine table; blog.html лічильник оновлено до 188 карток / 211 posts authored

---

## [1.99.1] — 2026-06-05

### Added
- `docs/COST_MODEL.md §30` — G&A & Founder Compensation Cost Model (v2.0): closes OQ-27 (founder salary policy, §22.8); models `G&A and founder salary` line from §24.4 ($240k–$360k [ESTIMATE]) with detailed phase-gated framework. §30.2 four-phase founder compensation — Phase 0 $0 (through first enterprise contract), Phase 1 $2,500–$5,000/month (first contract signed + pilot conversion ≥ 50%), Phase 2 $6,000–$10,000/month (ARR ≥ $100k + GM ≥ 70%), Phase 3 $10,000–$15,000/month (post-Series A) — with UA FOP vs. LLC employment vs. HoldCo dividend tax analysis (5–7% FOP vs. ~40% employment effective rate); wartime 2% rate caveat. §30.3 accounting/finance function — four-stage model (UA accountant $150–$300/month M1–M4; Xero + bookkeeper $400–$700/month post-launch; CPA review $8–$15k one-time at M12; fractional CFO $3–$6k/month post-Series A); deferred revenue trigger note; CPA selection criteria for SaaS ASC 606 + CEE cross-border. §30.4 admin tooling — Google Workspace $6/user, Notion Team $16/member, Loom Business $12.50/creator, Calendly Teams $16/member, 1Password Teams $4/user; total $22–$51/month at 1–3 person stage; Slack/Zoom free tiers excluded. §30.5 consolidated G&A table — $172–$351/month M1–M4, $422–$751/month M4–M12, $3,050–$10,950/month M12–M18, $13,635–$22,025/month post-Series A; reconciliation to §24.4 confirms $240k–$360k range valid at Phase 3 midpoints. §30.6 G&A as % ARR — caveat that ratio is meaningless pre-$300k ARR; trajectory from 373% at $50k ARR (meaningless) to 9% at $3M ARR; Bessemer/SaaS Capital benchmarks; FORM targets calibrated to §28.9 S&M + G&A ≤ 50% ARR combined ceiling — founder Phase 3 salary must stay ≤ $15k/month at $1M ARR to maintain headroom. §30.7 back-office hiring — four roles (UA accountant M1, bookkeeper at first enterprise invoice, fractional CFO with GGA-HIRE-01 gate, financial controller with GGA-HIRE-02 gate at ARR ≥ $1.5M + audit required). §30.8 three DEC-030 G&A audit events: `ops.founder_comp_updated` (HIGH, 7yr — OQ-27 resolution record; must precede `fundraise.round_closed`), `ops.finance_function_onboarded` (STANDARD, 7yr — mirrors §27.9 `ops.engineer_hired`), `ops.ga_tooling_contracted` (STANDARD, 3yr — paid subscriptions only). §30.9 seven-item checklist: 3× P0 M1 (comp decision + DEC-030 emission + §22.3 update, UA accountant, admin tooling), 3× P1 M4–M12 (event type registration, Xero deferred revenue, Phase 1 gate), 1× P2 post-Series A (fractional CFO gate). §30.10 three open questions: OQ-GGA-01 (Phase 1 trigger — wartime payment lag → trigger on cash receipt not contract signing, P0 before first contract), OQ-GGA-02 (UA FOP vs. HoldCo structure, P0 before first comp payment), OQ-GGA-03 (OQ-27 resolution — is Phase 0 $0 valid? — P0 before company formation; outcome updates §22.3). COST_MODEL.md document version header bumped v1.9 → v2.0. TOC updated with §30 and all subsections.

---

## [1.99.0] — 2026-06-05

### Added
- `docs/COST_MODEL.md §29` — Geographic Expansion Unit Economics & FX Risk Model: §29.1 six deliverables (market ARPU, FX risk, geo-pricing waterfall, payment infrastructure, regulatory compliance, expansion sequencing); §29.2 four-market consumer ARPU table — UA ($4.30–$5.00 effective, $52–$75 M24 LTV, 73%–84% GM), PL ($7.50–$8.30, $90–$120 LTV), DE/NL ($11.20–$12.00, $134–$170 LTV), US ($13.99–$16.99, $168–$240 LTV); PPP index basis (UA 0.32, PL 0.55, DE/NL near-parity); §29.3 FX risk model — UAH/USD exposure stack (indirect consumer churn risk + natural cost hedge via UA team payroll); 20% UAH devaluation stress: -$150–$400/year revenue vs. +$15–$20k/year engineering cost hedge → net P&L neutral; EUR/USD 10% stress: -$1,100/year on $10k EU ARR — immaterial at entry; mitigation via hard-currency enterprise billing + Stripe multi-currency settlement; §29.4 geo-pricing waterfall — PPP-adjusted consumer (UA $5.99, PL $10.99, DE/NL €14.99, US $19.99); anchor-based enterprise (UA $50–$150, PL €150–€250, DE/NL €199–€299, US $299 full list); founder approval gate on UA consumer price changes; §29.5 payment infrastructure — Stripe fee table by market (US 2.9%+$0.30, EU domestic 1.5%+€0.25, SEPA 0.8% capped €5); UA processor necessity (LiqPay/Wayforpay/Fondy, $2k–$5k integration); App Store 15% Small Business rate vs. §3 conservative 30% (→ ~$1.50/month per subscriber ARPU correction — fix §22.3 before Series A); blended 1.7% processing cost at 1,000 subscribers; §29.6 regulatory compliance by market — UA $500–$1,200 one-time; EU $5k–$12k Year 1 (DPIA mandatory, DPO $2k–$5k/yr, EU Rep $500–$1k/yr, SCCs) + $2.5k–$6k/yr recurring; US $4k–$8k Year 1 (CCPA/CPRA sensitive PI, state review) + $1k–$2.5k/yr; GDPR compliance framed as enterprise sales moat (cross-ref §25.7); §29.7 expansion sequencing — Phase 1 UA-only M1–M12 (validation cohort, war-context churn annotation for PostHog); Phase 2 EU entry M12–M24 gated by GEO-GATE-01 (D30 ≥ 40% + GDPR DPIA complete + DPO engaged + EU Stripe live + SCCs); recommended order Poland-first then DE/NL; Phase 2 entry cost $6.5k–$14.5k (8–19% of Year 1 EU net revenue); Phase 3 US M24+ gated by GEO-GATE-02 (EU ARR ≥ $50k + Series A + US entity + CCPA + PMM hired); §29.8 three DEC-030 geo audit events; §29.9 seven-item implementation checklist; §29.10 three open questions. TOC updated with §29 and all subsections.
- `content/post-219-rest-pause-myo-rep.md` — Rest-pause і myo-reps: механіка щільних підходів і наближення до відмови. Prestes et al. (2017 JSCR): rest-pause vs традиційний тренінг при рівному об'ємі — +13.8% м'язова витривалість у RP-групі, без достовірної різниці у 1RM; Hostler et al. (2001 JSCR): еквівалентна гіпертрофія при меншому часі у нетренованих. Механізм: PCr ресинтез ~50% за 30 с (Bogdanis et al. 1995), Pi/H⁺ не ліквідуються → вищопорогові Type II рекрутуються раніше у наступному сегменті; Burd et al. (2010) і Mitchell et al. (2012): MPS корелює з близькістю до відмови — більша частка повторень при RIR 0–2 = вищий стимул на повторення. Myo-reps (Fagerli 2008): activation set (RIR 1–3, 15–30 повт) + кластери 3–5 повт × 3–6 — операціоналізація «ефективних повторень» при нижчому суглобовому навантаженні. Застереження: не підходить для субмаксимальних підйомів і новачків з нестабільною технікою. FORM-контекст: Victor використовує щільні методи виключно для аксесуарного блоку. clinical-safety NOT REQUIRED
- `blog.html` — картка post-219 (rest-pause / myo-reps) додана перед post-218; лічильник: 186 → 187 карток
- `README.md` — post-219 позначено [x] done з детальним описом; roadmap розширено до post-224 (дроп-сети, генетика тренінгу, плацебо/ноцебо, LTAD)
- `STATUS.md` — рядок post-219 додано у content engine table

---

## [1.98.0] — 2026-06-05

### Added
- `content/post-217-muscle-architecture.md` — Архітектура м'яза: чому «більший» не означає «сильніший». Lieber & Fridén (2000 Muscle Nerve): три архітектурні параметри (pennation angle θ, fiber length Lf, PCSA = volume × cos θ / Lf); трейдоф pennated (висока сила, мала амплітуда) vs longitudinal (висока швидкість, мала сила). Fukunaga et al. (1997 Acta Physiol Scand) + Narici et al. (1996 J Physiol): in vivo ультразвук — ізометрія «на суглобі» ≠ ізометрія волокна (волокна вкорочуються в pennated м'язі навіть при стабільній довжині). Kawakami et al. (1993 J Biomech): gastrocnemius θ = 20–30°. Aagaard et al. (2001 J Appl Physiol): гіпертрофія збільшує θ → PCSA зростає більше, ніж передбачає анатомічний поперечний переріз. Практичне: хамстрінги (довгі волокна, низький θ) потребують повної ROM — leg curl у вкороченій позиції не дає повного стимулу; вибір вправ має враховувати архітектурний оптимум. FORM-контекст: ROM-tracking і velocity loss як непрямі маркери архітектурної ефективності. clinical-safety NOT REQUIRED
- `content/post-218-central-command-exercise-pressor-reflex.md` — Центральна команда і exercise pressor reflex: як мозок керує серцем під навантаженням. Krogh & Lindhard (1913 J Physiol): feedforward-активація кардіоваскулярної системи до початку скорочення (ЧСС зростає ще при думці про підхід; ефект відтворюється у кураризованих суб'єктів — нейроанатомія, не «психологія»). Mitchell et al. (1983 J Physiol): exercise pressor reflex — групи III (механорецептори, швидкі: стрейч і тиск) і IV (метаборецептори, повільні: Pi, H⁺, лактат, брадикінін) аферентних волокон. Smith et al. (2006 Am J Physiol Heart Circ Physiol): ізольований внесок механо- і метаборецепторів. Amann et al. (2009 J Physiol): блокада груп III/IV інтратекальним фентанілом → атлети генерували вищу потужність до відмови, але мали більший периферичний збиток — ЦНС-гальмо є реальним захисним механізмом. Gandevia (2001 Physiol Rev): суперспінальний компонент — мотивація може «пробити» VA-gap (~80%→95%), але не знизити [Pi] у саркомері. FORM-контекст: RPE і velocity loss як проксі інтенсивності afferent feedback; Victor не рекомендує систематично тренуватись при RPE > 9 — хронічне підвищення базального аферентного сигналу знижує VA. clinical-safety NOT REQUIRED
- `blog.html` — картки post-218 (central command + exercise pressor reflex) і post-217 (muscle architecture) додані перед post-216; лічильник: 184 → 186 карток
- `README.md` — post-217 і post-218 позначені [x] done з коротким описом
- `STATUS.md` — рядки post-217 і post-218 у content engine table; blog.html лічильник оновлено до 186 карток / 209 posts authored

---

## [1.97.1] — 2026-06-04

### Added
- `docs/COST_MODEL.md §28` — Marketing & Demand Generation Cost Model: §28.1 scopes the section as the fourth operational cost function alongside §25 (compliance), §26 (CS), §27 (engineering) — fills the §22.3 "Marketing / UA" line ($500–$2,000/month) with a full decomposition; §28.2 COGS vs. S&M taxonomy for eight marketing spend types; §28.3 pre-launch budget M1–M4 ($3,500 total cash mapped to §22.3); §28.4 ASO investment ($700–$1,400 external + 17h founder time; 5%→10% CVR improvement = 50 incremental installs/month at $0 marginal cost); §28.5 consumer paid acquisition with channel table (ASA brand $17–$50 effective CAC, Meta $43–$267, TikTok $40–$300, organic ≥ $15–$30) and MKT-UA-GATE-01 hard ceiling; §28.6 enterprise demand gen (founder-led $300–$1,200/month; post-Series A $7,600–$19,450/month with ABM, AE tools, events); §28.7 marketing tooling stack (pre-launch $0–$38/month; post-launch $79–$179/month excl. ad spend; Series A $346–$666/month); §28.8 first marketing hire: four profile options + recommended Growth Marketer at Month 18–22 with three hiring gates (MKT-HIRE-01/02/03); §28.9 S&M as % ARR benchmarks (Bessemer/SaaS Capital) vs. FORM targets at four stages; §28.10 magic number model (net new ARR ÷ prior-quarter S&M spend; target > 0.75; four-milestone table; unreliable before first AE + > $10k/quarter S&M); §28.11 three DEC-030 HMAC-chained marketing spend events (mkt.paid_ua_gate_unlocked STANDARD 3yr, mkt.paid_channel_paused STANDARD 3yr, mkt.marketing_hire_onboarded STANDARD 7yr); §28.12 seven-item implementation checklist (3× P0 M3–M4: UTM attribution, App Store assets, email drip; 3× P1 M4–M6: GATE-01 OKR, monthly cost report, DEC-030 registration; 1× P2 post-AE: magic number reporting); §28.13 three open questions: OQ-MKT-01 (content flywheel vs. paid UA sequencing — P0, recommend content M1 + ASA at launch + Meta after GATE-01), OQ-MKT-02 (incentivised vs. social-share referral — P1, recommend social-share first), OQ-MKT-03 (gated vs. ungated enterprise collateral — P2, recommend ungated at pre-PMF). TOC updated with §26–§28 entries. Document version bumped to v1.9.

---

## [1.97.0] — 2026-06-04

### Added
- `content/post-216-vo2-kinetics.md` — Кінетика поглинання кисню: чому VO2max — не єдина цифра. Whipp & Wasserman (1972) трифазна структура VO2-відповіді (кардіодинамічна / первинний компонент τ / стаціонарний стан); Gaesser & Poole (1996) повільний компонент і критична потужність; Jones & Poole (2005) CP як поріг між метаболічною рівновагою і нерівновагою; Vanhatalo et al. (2011 J Physiol) зв'язок CP і VO2-кінетики; Davies & Thompson (1986) аеробна тренованість і ресинтез PCr; практичне для силовика: зона 2 як інфраструктура відновлення між підходами; clinical-safety NOT REQUIRED
- `blog.html` — картки post-216 (VO2 kinetics), post-215 (специфічність vs варіативність), post-214 (сила і когнітивні функції), post-213 (імунна функція) додані перед post-212
- `README.md` — roadmap пости 216 (done) і 217-220 (planned) з описами і джерелами
- `STATUS.md` — рядки post-216..213 у content engine table

---

## [1.96.0] — 2026-06-04

### Added
- `content/post-213-immune-function.md` — Тренування і імунна функція: Open Window Hypothesis (Gleeson 2007), J-крива Nieman 1994, IL-6 як міокін (Pedersen & Febbraio 2008); practical section з 5 clinical-safety guardrails; clinical-safety PASS WITH CONDITIONS (2026-06-04)
- `content/post-214-strength-cognitive-function.md` — Сила і когнітивні функції: Northey et al. (2018 BJSM) meta-analysis ES 0.66; BDNF / IGF-1 / нейрогенез гіпокампу; Liu-Ambrose (2010) executive function; resistance training vs. aerobics cogntive domain comparison; clinical-safety NOT REQUIRED
- `content/post-215-specificity-vs-variability.md` — Специфічність vs варіативність: SAID principle; Barnett et al. (2002) transfer of training; міф «тіло звикає»; блокова periodization як баланс між специфічністю та варіативністю; clinical-safety NOT REQUIRED
- `docs/COST_MODEL.md §27` — Engineering Team Cost Model: §27.2 founding engineer total comp ($100k–$139k/year first-year including taxes, hardware, onboarding; equity 2.5%–3.5% pre-seed); §27.3 four-hire technical sequence (iOS/Supabase → Platform/Workers → ML/CV → Data Engineer) with milestone triggers; §27.4 infrastructure run-rate table (8 services, 4 scale stages; pre-revenue ~$77/mo, 50k MAU ~$2,800–$5,500/mo); §27.5 seven-service build-vs-buy (BUY: WorkOS/Supabase/Anthropic/PagerDuty/Sentry/Better Stack; BUILD: CV pipeline — avoids $108k–$360k/year GPU cost at 50k MAU); §27.6 CV on-device unit economics ($0 marginal per-session vs. ~$108k–$360k/year hypothetical server-side); §27.7 engineering as % ARR trajectory (83%–115% at $100k ARR → 16%–21% at $3M ARR); §27.8 four burn rate gates (ENG-GATE-01 through ENG-GATE-04); §27.9 three DEC-030 events; §27.11 three open questions (OQ-ENG-01 UA/US employment structure; OQ-ENG-02 iOS-specialist vs fullstack founding profile; OQ-ENG-03 device compatibility floor for CV)

---

## [1.95.0] — 2026-06-04

### Added
- `docs/SOC2_READINESS.md §64` — Multi-Tenant RLS Tenant Isolation CI Test Suite — CC6.1/CC6.3/CC6.6/PI1.5-C3. PEN-GAP-003 remediation auditor exhibit (🔴 Open → 🟡 Authored). §64.2: `supabase/migrations/20260604000000_rls_test_role.sql` — `rls_test` Postgres role setup (SELECT-only, NOLOGIN, NOBYPASSRLS, idempotent DO-guard). §64.3: `supabase/seed/rls_test_fixtures.sql` — fixed-UUID test tenants (`...001`/`...002`) + 5 synthetic `workout_sessions` + 1 `user_profile` + 3 `coaching_turns` + 2 `cv_sessions` per tenant. §64.4: `tests/security/rls-tenant-isolation.test.ts` — Vitest TypeScript suite; `createTestJwt()` helper (jose HS256); five `describe` blocks for TC-RLS-001–005 including `alg:none` raw-encoding attack and 10-second polling loop for HMAC chain-break event. §64.5: `.github/workflows/rls-isolation-tests.yml` — triggers on push/PR to main; Supabase CLI local stack; R2 evidence filing gated to main branch; `retention-days: 2557` on artifact upload. §64.6: PRE-36-E-007 evidence log template (metadata + per-TC table + signature block + R2 path `form-compliance-vault/soc2/rls-isolation/<YYYY-MM-DD>/`). §64.7: three new DEC-030 events — `security.rls_bypass_attempt` (MEDIUM, 7yr; `request_ip` SHA-256+daily-salt per DPIA §4), `security.definer_function_cross_tenant` (HIGH, 7yr; P0 incident trigger), `system.rls_test_suite_run` (STANDARD, 3yr; stores R2 evidence path in payload). §64.8: SOC 2 evidence mapping — CC6.1/CC6.3/CC6.6/CC7.1/CC7.2/PI1.5-C3 all advance to 🟢 on first green CI run. §64.9: 8-item checklist (3× P0 M7, 3× P1 M8, 2× P2). §64.10: PEN-GAP-003 🔴 → 🟡 Authored; SOC 2 readiness ~95% → ~95.5%.

---

## [1.94.0] — 2026-06-04

### Added
- `content/post-212-adherence-coefficient.md` — "Adherence coefficient: чому найкраща програма — та, якій відповідає твоє реальне життя". Schoenfeld (2017 JSCR), Androulakis-Korakakis (2020 Sports Med), Bickel (2011 Med Sci Sports Exerc), Gentil (2017 PLoS ONE): тренувальний ефект = якість × коефіцієнт дотримання; програма-хопінг як проблема дизайну; MED як рівень структури для стійкого виконання; режим «виживання» як частина дизайну програми. Clinical-safety: PASS WITH CONDITIONS (2026-06-04) — умови виконані: відсутній shaming-фреймінг, FORM позиціонований як стале тренування, не нульовий опір.
- `blog.html` — картки post-211 (PCr resynthesis kinetics) і post-212 (adherence coefficient) додані перед post-210.
- `README.md` — roadmap оновлено: [x] post-211, [x] post-212, [ ] post-213 (immune function), [ ] post-214 (cognitive function), [ ] post-215 (specificity vs variety).
- `STATUS.md` — рядки post-211 і post-212 додані до content engine table.

---

## [1.93.0] — 2026-06-04

### Added
- `docs/SOC2_READINESS.md §63` — PI1 Processing Integrity Controls: CV coaching pipeline accuracy, completeness & timeliness. 10×P0 + 7×P1 + 3×P2 checklist items, 6 evidence artefacts (PI-E-001..006), 3 open questions (OQ-PI-01/02/03). OQ-PI-03 flagged as Type II observation period blocker. Owner: compliance-officer + ML eng.

---

## [1.92.0] — 2026-06-04

### Added
- `content/post-211-pcr-resynthesis-kinetics.md` — PCr resynthesis kinetics: biphasic curve, acidosis inhibition of creatine kinase, goal-specific rest interval programming (Harris 1976, Hultman 1987, Bogdanis 1995/1996, Greenhaff 1994). Clinical-safety: NOT REQUIRED.

---

## [1.91.1] — 2026-06-04

### Changed
- `docs/OBSERVABILITY.md` — §6.2 Alert Rules table: п'ять відсутніх subsections додано — `cert_lifecycle` (AL-CERT-01–05, §26.8), `session_revocation` (AL-REVOKE-01–02, §26.8), `google_directory_sync` (AL-SSO-GDIR-01–05, §26.8), `pam_session_health` (AL-PAM-01–03, §29.5), `crypto_key_health` (AL-KEY-01–04, §30.5) — 19 рядків, закриває три P0 M4 checklist items: §26.12 item 6, §29.10 item 4, §30.10 item 5; SOC 2 CC6.1/CC7.2/CC7.3/CC5.3 coverage; версія v1.7 → v1.8
- `VERSION` — 1.91.0 → 1.91.1

---

## [1.91.0] — 2026-06-04

### Added
- [`content/post-210-cognitive-load-technique.md`](content/post-210-cognitive-load-technique.md) — Когнітивне навантаження і техніка: коли думати про рух — означає зробити його гірше — Fitts & Posner (1967): три стадії моторного навчання (когнітивна/асоціативна/автономна); Wulf & Lewthwaite (2016 Psychon Bull Rev): OPTIMAL theory — зовнішній фокус уваги стабільно перевершує внутрішній у досвідчених; Beilock & Carr (2001 Psychol Sci): choking under pressure — explicit monitoring hypothesis — явне моніторування руйнує автоматизовані рухи; Beilock et al. (2002 J Exp Psychol Gen): dual-task не погіршує виконання у досвідчених; Lohse & Sherwood (2012 JSCR): зовнішній фокус у жимі лежачи +5% пікова сила; Snyder & Ledbetter (2013 Eur J Sport Sci): внутрішній фокус функціональний для новачків у перші тижні; pre-lift routine як захист від явного моніторування; stagegate для типу cue; clinical-safety NOT REQUIRED
- `blog.html` — картка #184 для post-210-cognitive-load-technique; лічильник оновлено до 184 cards (207 posts authored)
- `STATUS.md` — content engine table оновлено: post-207, post-208, post-209, post-210 додані
- `README.md` — post-210 відмічено як виконаний у content engine roadmap

---

## [1.90.0] — 2026-06-04

### Added
- [`content/post-209-eccentric-overload.md`](content/post-209-eccentric-overload.md) — Ексцентрична фаза: чому опускання є тренуванням — Katz (1939 J Physiol): ексцентрична сила-ємність +20–60% відносно концентричної; нейрогенне інгібування у нетренованих (Dudley et al. 1990 JAP); Morgan (1990 J Exp Biol): «popping sarcomere» hypothesis — Z-disc disruption і titin strain як механізм ексцентричного пошкодження; Goldspink (2005 J Anat): MGF (splice-variant IGF-1) і активація сателітних клітин; McHugh et al. (1999 Sports Medicine): repeated bout effect (RBE) — DOMS і адаптивний захист; Schoenfeld (2010 JSCR): м'язове пошкодження як один з трьох механізмів гіпертрофії; Hortobágyi et al. (1996 JAP): ексцентричне тренування = більший приріст сили при зіставних інтенсивностях; Burd et al. (2012 J Physiol): TUT і синтез міофібрилярних протеїнів при контрольованому темпі; Tesch et al. (2004 Acta Physiol Scand): flywheel training і ексцентрично-концентричний коефіцієнт; протоколи: slow eccentrics (3–6 с), акцентоване ексцентричне, flywheel; програмування: блоки 4+4 тижнів з деload; clinical-safety NOT REQUIRED
- `docs/AUDIT_LOG_SCHEMA.md` — нова секція «API Key credential events» — 9 DEC-030 HMAC-chained подій (`api_key.created`, `api_key.rotated`, `api_key.revoked`, `api_key.ip_enforcement_enabled`, `api_key.ip_enforcement_disabled`, `api_key.ip_blocked`, `scim.ip_enforcement_enabled`, `scim.ip_enforcement_disabled`, `scim.ip_blocked`) — HIGH severity, 7yr retention, `client_ip_hash` privacy floor на ip_blocked events; примітка щодо superseding consumer-tier `tenant.api_key_*` shorthand. Closes **SSO_SCIM_IMPLEMENTATION.md §26.11 item 5** (P0 M5)
- `docs/CRYPTOGRAPHY_POLICY.md §5` — `API_KEY_HASH_SECRET` додано до Key Rotation Schedule (180 днів, immediate on compromise, re-hash `tenant_api_keys` під час maintenance window). Closes **SSO_SCIM_IMPLEMENTATION.md §26.11 item 7** (P0 M5)
- `docs/CRYPTOGRAPHY_POLICY.md §6` — `API_KEY_HASH_SECRET` додано до Key Custody (Cloudflare Workers Secret + 1Password Operations vault)
- `docs/SOC2_READINESS.md §56.3` — `API_KEY_HASH_SECRET` додано до Key Inventory table (HMAC-SHA256 keying material для `tenant_api_keys.key_hash`, 180d rotation, security-engineer owner, Cloudflare Workers Secret storage). Closes **SSO_SCIM_IMPLEMENTATION.md §26.11 item 7 partial** (SOC2 evidence cross-reference)

- `docs/DATA_MODEL.md §26` — API Key Authentication Schema — повний канонічний DDL `tenant_api_keys` (supersedes SSO_SCIM §26.4.2 як canonical source); 5 constraints (`chk_api_key_label_length`, `chk_api_key_scopes_nonempty`, `chk_ip_allowlist_required`, `chk_ip_allowlist_shape`, partial unique on `key_hash WHERE revoked_at IS NULL`); 4 indexes (authentication hot-path `uq_tenant_api_keys_hash_active`); RLS: `form_system` full read/write, `form_api` tenant-isolated SELECT (no key_hash in API projections — CI lint rule `no-select-star-on-api-keys`), `form_admin` BYPASSRLS; §26.2 credential architecture cross-reference (JWT / SCIM bearer / API key — три роздільних шляхи); §26.3 off-boarding integration (§25 gap filled — `revoked_at` в `revokeAccess()` транзакції); §26.6 lifecycle TypeScript pseudocode (key creation 256-bit random + HMAC-SHA256, rotation 24h overlap per §16.6, revocation reason enum 5 values); §26.7 9 DEC-030 events (all HIGH, 7yr, chain requirement: `api_key.rotated` → `api_key.revoked` within 26h); §26.8 alerting AL-APIKEY-01/-02/-03 + AL-SCIM-IP-01; §26.9 SOC2 CC6.1/CC6.2/CC6.4/CC6.8/CC7.2 artefacts APIKEY-E-001–005; §26.10 TypeScript types; §26.11 17-scenario RLS test matrix; §26.12 OQ-SSO-26.1/26.2 cross-ref; §26.13 8× P0 M5 implementation checklist. Closes **SSO_SCIM_IMPLEMENTATION.md §26.11 item 12** (P1 M6)

### Changed
- `blog.html` — додано картку post-209 (Ексцентрична фаза)
- `VERSION` — 1.89.1 → 1.90.0

---

## [1.89.1] — 2026-06-04

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md §26` — API Key Authentication Security — SCIM IP Scope, API Key IP Enforcement & Rotation Policy · v1.7 → v1.8. **Closes OQ-SSO-25.1** (🟡 P1 → 🟢: SCIM endpoint IP scope; `scim_ip_enforcement_enabled` flag, default false; separate `scim_ip_allowlist_config` JSONB; general IP allowlist never blocks SCIM routes; `enforceScimIpAllowlist()` middleware branch; migration `0052_scim_ip_allowlist.sql`; admin dashboard "Directory Sync IP Restriction" UI with Okta/Azure pre-populated CIDR import). **Closes OQ-SSO-25.3** (🟡 P1 M5 → 🟢: API key IP enforcement; formalised `tenant_api_keys` table schema with `ip_enforcement_enabled BOOLEAN DEFAULT false`, `ip_allowlist_config JSONB`, `scopes TEXT[]`, `key_hash` HMAC-SHA256, `key_preview` last-8-chars, `last_used_at`, `expires_at`, `revoked_at`; `enforceApiKeyIpAllowlist()` wired into `api-key-auth.ts` using shared `checkCidrList()` from §25 — no new dependency; scope enforcement before route handler). Rotation policy: 365d amber / 730d red age alerts; `admin:write` scope 90d mandatory; 24h overlap window. Nine DEC-030 HMAC-chained events (all HIGH, 7yr): `api_key.created`, `api_key.rotated`, `api_key.revoked`, `api_key.ip_enforcement_enabled`, `api_key.ip_enforcement_disabled`, `api_key.ip_blocked` (client_ip_hash only), `scim.ip_enforcement_enabled`, `scim.ip_enforcement_disabled`, `scim.ip_blocked`. Four alerting rules: AL-APIKEY-01 P1 (>10 blocks/key/10 min), AL-APIKEY-02 P1 (rotation overlap not cleaned up → cron failure), AL-APIKEY-03 P1 (revocation reason=compromise → R-16), AL-SCIM-IP-01 P2 (SCIM provisioning degraded + ip_blocked spike). SOC 2: CC6.1/CC6.2/CC6.4/CC6.8/CC7.2 with five evidence artefacts APIKEY-E-001 through APIKEY-E-005. Two new open questions: OQ-SSO-26.1 (mandatory IP enforcement for admin:write keys P2) and OQ-SSO-26.2 (expires_at hard vs soft enforcement P1). 14-item implementation checklist (8× P0 M5, 4× P1 M5-M6, 2× P2 M6). Owner: enterprise-architect + security-engineer + platform-engineer + compliance-officer.

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` — header v1.7 → v1.8; TOC updated to add §26
- `VERSION` — 1.89.0 → 1.89.1

---

## [1.89.0] — 2026-06-04

### Added
- [`content/post-208-intensity-vs-volume-hypertrophy-threshold.md`](content/post-208-intensity-vs-volume-hypertrophy-threshold.md) — Інтенсивність vs об'єм: де насправді знаходиться гіпертрофійний поріг — Mitchell et al. (2012 J Appl Physiol): 30% vs 80% 1RM до відмови → еквівалентний MPS (синтез м'язових протеїнів); Burd et al. (2010 PLoS ONE): MMF як провідний модератор, а не % 1RM; Schoenfeld et al. (2017 JSCR): RCT 8RM vs 25–35RM до відмови → еквівалентна гіпертрофія, силова адаптація вища при 80% 1RM; Lasevicius et al. (2018 Eur J Sport Sci): нижній поріг ~30–40% 1RM — нижче якого навіть відмова субоптимальна; механічна напруга (FAK/ILK → mTORC1) і метаболічний стрес (AMPK, CaMK, MAPK) як паралельні сигнальні шляхи; Henneman size principle і рекрутування type II волокон при низькому навантаженні до відмови; провідна змінна — RIR 0–2, а не абсолютний % 1RM; нейральна специфіка (1RM-перенос) — єдина значима перевага високої інтенсивності перед низькою; clinical-safety NOT REQUIRED

### Changed
- `blog.html` — додано картку post-208 (Інтенсивність vs об'єм); лічильник: 182 cards, 205 posts authored
- `STATUS.md` — оновлено рядок blog.html: 182 cards / 205 posts / v1.89.0
- `VERSION` — 1.88.0 → 1.89.0

---

## [1.88.0] — 2026-06-04

### Added
- `docs/SOC2_READINESS.md §62` — Pentest Pre-Engagement Requirements: OQ-PT-02, OQ-PT-03, OQ-PT-04 Resolution — CC9.2 / P3.1 / CC6.1. Resolves all three P1 M5 open questions from §61.12 that block M6 engagement letter signing. §62.1 purpose: M5 deadline, three P1 OQs blocking engagement letter, compliance debt rationale. §62.2 OQ-PT-02 🟢 Resolved: Supabase AUP permits customer pentests without carve-out letter; two mandatory procedures: (1) notification email to security@supabase.com ≥5 business days before test start referencing project ref and dates, filed as PT-E-007 in R2 `form-compliance-vault/pentests/{engagement_id}/PT-E-007-supabase-notification.eml`; (2) engagement letter shared-responsibility carve-out clause limiting scope to FORM-controlled surfaces (PostgREST config, RLS policies, Auth callbacks, SCIM API, Cloudflare Worker logic) and excluding Supabase managed database engine, network infra, and hosting layer by name. PT-E-007 referenced via `admin.pentest_initiated` payload fields (`supabase_notification_filed: true`, `supabase_notification_date`). §62.3 OQ-PT-03 🟢 Resolved: DPIA amendment NOT required provided all 5 conditions satisfied — C1 synthetic fixtures only (SBF-001: pseudorandom Float32Array[17×3], realistic joint coordinate ranges, never derived from real users, min 50 sessions, AES-256-GCM encrypted in staging); C2 vendor zero access to production health tables (`keypoints_enc`, `sessions`, `workout_logs`); C3 staging environment isolation; C4 halt-and-assess within 24h if production Art. 9 data accidentally accessed (compliance-officer DPIA incident assessment, vendor activity halted); C5 engagement letter biometric prohibition clause per §61.7. SBF-001 synthetic fixture protocol filed in R2 before test start. §62.4 OQ-PT-04 🟢 Resolved: two-layer decision — Layer A destructive tests (chain injection, replay, truncation, hash collision probe) staging ONLY, never production; Layer B read-only production enumeration permitted under 4 strict conditions: B1 compliance-officer emits new DEC-030 event `admin.pentest_chain_enumeration_authorised` (HIGH, 7yr, `emit-audit-event` Worker only, payload: `engagement_id`, `authorised_by`, `authorisation_date`, `vendor_name`, `enumeration_scope`, `production_flag: true`, `staging_only_destructive_tests_confirmed: true`) before any production query; B2 `audit-chain-verify` Edge Function aggregate query only — no free-form SQL, no payload content; B3 read-only service role scoped to `audit_log_events` SELECT only, 48hr max, immediate revocation after test; B4 no payload content in vendor findings. PT-E-005 clarified as post-remediation artefact run by security-engineer after all findings remediated, NOT a vendor deliverable. §62.5 PT-E-007 added to artefact register (7yr retention, chain-of-custody via `admin.pentest_initiated` payload). §62.6 updated §61.11 checklist: 5× P0 additions (PT-P0-06 through PT-P0-10) and 1× P1 addition (PT-P1-06 AUDIT_LOG_SCHEMA registration of `admin.pentest_chain_enumeration_authorised`). §62.7 TSC mapping: CC9.2 (PT-E-007 + carve-out clause = vendor boundary management), P3.1 (SBF-001 + 5 conditions = Art. 9 data not disclosed to vendor), CC6.1 (Layer B credential scoping + authorisation event = logical access controls evidenced under adversarial conditions). §62.8 implementation checklist: 8 sequential items M5→M6+4wk. §62.9 open questions: OQ-PT-01 remains open P2 M10; OQ-PT-02/03/04 🟢 Resolved. Owner: compliance-officer + security-engineer.

---

## [1.87.1] — 2026-06-04

### Added
- [`content/post-207-antagonist-agonist-paired-sets.md`](content/post-207-antagonist-agonist-paired-sets.md) — Антагоніст-агоніст парні підходи: реципрокне полегшення і потенціація у силовому тренінгу — Baker & Newton (2005 JSCR): APS жим + горизонтальна тяга → +8,5% пікова потужність у наступному підході (не економія часу — нейральна маніпуляція); Robbins et al. (2010 JSCR): APS знижують кумулятивну втому, вищий загальний виконаний об'єм при ідентичному часовому бюджеті; Weakley et al. (2020 JSCR): мета-аналіз ефективності APS; Sherrington (1906): Ia-інтернейрони і реципрокне інгібування; Hodgson et al. (2005 Sports Med): PAPE механізм (фосфорилювання міозинових RLC); правильні пари: жим/тяга, жим стоячи/lat pulldown, присідання/RDL, біцепс/трицепс; неправильні пари: агоністичні комбінації, >90% 1RM, технічно складні компаундні рухи, новачки; протокол пауз: 60–90с між вправами в парі, 90–120с між парами; 1–3 пари за сесію для intermediate атлета; clinical-safety NOT REQUIRED (sports-scientist · clinical-safety: NOT REQUIRED)

### Changed
- `blog.html` — додано картку post-207 (Антагоніст-агоніст парні підходи)
- `README.md` — post-203/204/205 позначені `[x]` з фактичним контентом (tendon mechanics, BFR, motor cortex plasticity); post-207 позначений `[x]` з APS; попередній запланований топік post-207 (Інтенсивність vs об'єм) зсунутий на post-208; post-208 (мікробіом) → post-209; нумерація оновлена

---

## [1.87.0] — 2026-06-04

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md §25` — Per-Tenant Authentication Policy Engine — IP Allowlist, MFA Enforcement & Session Policy · v1.4 → v1.7. **Closes G-006** (🟡 → 🟢 upon deployment of `auth-policy-middleware` Worker). Schema additions: `ip_allowlist_config JSONB` (replaces flat `ip_allowlist INET[]` with labelled entries, per-entry UUIDs, `added_at`/`added_by`; migration `0047_auth_policy_engine.sql` + 72h-soak legacy column drop `0048`); `mfa_enforcement_mode TEXT CHECK IN ('none','recommended','required')`; `mfa_grace_hours INT 1–168h`; `policy_version INT` (OQ-SSO-25.4). IP allowlist enforcement Worker (`src/workers/auth-policy-middleware/ip-allowlist.ts`): extends §16.10 from SSO-callback-only to all authenticated routes (SAML/OIDC callback, token refresh, admin API, SCIM API; end-user `/v1/api/*` excluded); `netmask` npm CIDR library (IPv4+IPv6); `CF-Connecting-IP` header; `hashIp()` with `IP_HASH_SALT` Workers Secret (SHA-256[:32]); fail-closed on Supabase unavailability (503, not fail-open); Cloudflare Queues consumer for burst `sso.ip_blocked` events to maintain HMAC chain ordering. KV-backed policy cache: `SSO_KV:auth_policy:{tenant_id}`, 60s TTL, active cache invalidation on policy write. MFA enforcement middleware (`src/workers/auth-policy-middleware/mfa-enforcement.ts`): evaluates IdP `amr` claim (OIDC) or `AuthnContextClassRef` (SAML 2.0); 9 satisfying values (`mfa`, `totp`, `otp`, `hwk`, `swk`, `fido`, `pop`, `sms`); grace period for existing sessions when mode → `required` (`mfa_grace_hours`); hourly sweep cron. Policy change management: lockout-prevention validation (admin's own IP must be in new allowlist before save); /0 prefix rejection; 5-changes/5-min rate limit; session impact matrix (IP allowlist toggle → no immediate revocation; MFA mode → grace sweep; session_max_duration_hours → cap on next refresh). Admin lockout recovery: IP lockout → PAM `read_write` (§24.3) → UPDATE + SSO_KV delete → `sso.auth_policy_lockout_recovery` CRITICAL event; MFA per-user bypass → `tenant_admin` only (not FORM support) → `mfa_bypass_until TIMESTAMPTZ` (max 48h) + `sso.mfa_bypass_granted` CRITICAL event. Nine DEC-030 HMAC-chained events (HIGH/CRITICAL, 7yr except sweep STANDARD 3yr): `sso.auth_policy_updated`, `sso.ip_allowlist_entry_added`, `sso.ip_allowlist_entry_removed`, `sso.ip_allowlist_toggled`, `sso.ip_blocked` (no `user_id`, `client_ip_hash` only), `sso.mfa_enforcement_changed`, `sso.mfa_enforcement_sweep_completed`, `sso.mfa_bypass_granted`, `sso.auth_policy_lockout_recovery`. Three alerting rules: AL-AUTH-POLICY-01 P1 (mass ip_blocked + zero sessions → lockout detection), AL-AUTH-POLICY-02 P1 (MFA enforcement disabled on enterprise tenant → compliance risk), AL-AUTH-POLICY-03 P2 (admin_api block after 7-day silence → travel/VPN, CSM notify). SOC 2: CC6.1 (IP allowlist network gate, AUTH-POLICY-E-001), CC6.2 (MFA independent of IdP, AUTH-POLICY-E-002), CC6.8 (HMAC-chained policy audit trail, AUTH-POLICY-E-003), CC7.2 (anomaly alerting, AUTH-POLICY-E-004). Four open questions: OQ-SSO-25.1 SCIM endpoint IP scope P1, OQ-SSO-25.2 SMS in satisfying set P2, OQ-SSO-25.3 API key IP enforcement P1 M5, OQ-SSO-25.4 optimistic locking P2 M6. 15-item checklist (8× P0 M4, 5× P1 M5, 2× P2). TOC updated to include §22–§25 (§22–§24 were previously missing from TOC). Owner: enterprise-architect + security-engineer + platform-engineer + compliance-officer.

---

## [1.86.0] — 2026-06-04

### Added
- [`content/post-206-strength-vs-power.md`](content/post-206-strength-vs-power.md) — Сила vs потужність: F-V крива, Wilson (1993), Behm & Sale (1993), Cormie et al. (2011), SSC (sports-scientist · clinical-safety: NOT REQUIRED)

---

## [1.85.4] — 2026-06-04

### Added
- [`content/post-205-motor-cortex-plasticity.md`](content/post-205-motor-cortex-plasticity.md) — Пластичність моторної кори і сила: TMS, GABA, кортикальна карта (sports-scientist · clinical-safety: NOT REQUIRED)

---

## [1.85.3] — 2026-06-04

### Added
- [`content/post-204-bfr-blood-flow-restriction.md`](content/post-204-bfr-blood-flow-restriction.md) — BFR-тренінг: гіпертрофія під низьким навантаженням (sports-scientist · clinical-safety: NOT REQUIRED)
- [`docs/SOC2_READINESS.md`](docs/SOC2_READINESS.md) §61 — Penetration Testing Program (CC7.1/CC7.2/CC4.1) · PT-GAP-001 🔴 HIGH opened (compliance-officer + security-engineer)

---

## [1.85.2] — 2026-06-04

### Added
- [`content/post-203-tendon-mechanics-stiffness.md`](content/post-203-tendon-mechanics-stiffness.md) — Механіка сухожиль і адаптація жорсткості (sports-scientist · clinical-safety: NOT REQUIRED)

---

## [1.85.1] — 2026-06-03

### Changed
- `docs/INCIDENT_RESPONSE.md` v1.3 → v1.5 — added R-21 Key Rotation Failure & Scheduled Cryptographic Control Emergency; fixed header version discrepancy (body contained v1.4 content for R-20 but header still read v1.3). R-21 closes the runbook gap created when `docs/OBSERVABILITY.md` §30 (v1.7, 2026-06-03) added AL-KEY-01 through AL-KEY-04 alert rules with no `runbook_url`. Covers: CRYPTO-CHAIN-02 stop-gate (→ R-05 + R-20); severity classification P0/P1/P2; 4 scope-assessment queries (C1 KV state, C2 audit_log history, C3 scheduler health, C4 HMAC cadence); per-key response procedures (SUPABASE_SERVICE_ROLE_JWT / HMAC_AUDIT_CHAIN_KEY dual-key / KEYPOINTS_ENC_KEY maintenance window / API keys); Template K-01 (enterprise maintenance window notice); 5 DEC-030 HMAC-chained events; evidence package IR-KEY-E-001–006; SOC 2 CC5.2/CC5.3/CC6.7/CC6.8/C1.1/CC7.2/CC7.3; OQ-KEY-01 (event registration P0 M7), OQ-KEY-02 (KEYPOINTS_ENC_KEY re-encryption strategy P1 M12); 7-item implementation checklist.

---

## [1.85.0] — 2026-06-03

### Added
- [`content/post-202-neural-efficiency.md`](content/post-202-neural-efficiency.md) — Нейральна ефективність руху: economics of movement — Enoka (1994) рамка трьох рівнів (рекрутування, rate coding, координація); Semmler (2002 Exerc Sport Sci Rev): синхронізація МО і адаптивна специфічність; Lay et al. (2011 J Electromyogr Kinesiol): варіативність міжм'язової координації у новачків vs тренованих; коактивація антагоністів як прихована стаття витрат; Sale (1988) хронологія нейральної фази; чому техніка — економіка, а не естетика; clinical-safety NOT REQUIRED

### Changed
- `blog.html` — додано картку post-202
- `STATUS.md` — додано рядок post-202
- `README.md` — post-202 позначений як `[x]`

---

## [1.84.0] — 2026-06-03

### Added
- [`content/post-200-epigenetic-memory-training.md`](content/post-200-epigenetic-memory-training.md) — Епігенетична пам'ять тренінгу: ДНК-метилювання (NCOR2, IGF1), гістонові модифікації (H3K4me3), Seaborne 2018 + Sharples 2019; механізм прискореного повернення після паузи; clinical-safety NOT REQUIRED
- [`content/post-201-cardiovascular-drift-plasma-volume.md`](content/post-201-cardiovascular-drift-plasma-volume.md) — Серцево-судинний дрейф: механізм (Coyle & González-Alonso 2001, Rowell 1993), перерозподіл плазмового об'єму, HR-зони при тривалих сесіях, гідратація, акліматизація; clinical-safety NOT REQUIRED

### Changed
- `README.md` — post-200 і post-201 позначені як `[x]`

---

## [1.83.3] — 2026-06-03

### Added
- `docs/COST_MODEL.md §26` — Customer Success Team Scaling Economics & CS Cost Model; fills the gap between §8.2's single CSM line-item ($250–500/month) and the full CS team cost, hiring trigger, and gross margin model; CS coverage model by tier (Starter email-only $150–200/month shared inbox; Growth named CSM 12–15 accounts; Enterprise dedicated CSM max 6 accounts); CSM capacity model: 7-activity time budget; hard capacity ceiling 15 Growth or 6 Enterprise accounts per CSM; ARR under management $810k–$1M (within $500k–$1.5M industry benchmark); CS team headcount vs. customer count: three stages (founder-led 1–3; first CS hire M13 4–15; Series A team 15–50+); CS cost as % ARR: 30–50% at stage → 8–12% at Series B; gross margin impact: VP CS hire depresses Growth account GM 7.8 pp until account base absorbs fixed cost; combined CS + compliance cost stack 27–33% ARR at $1M–$1.5M milestone (54–61% GM floor); five CS hiring triggers CS-HIRE-01 through CS-HIRE-05 with anti-trigger against premature VP CS hire; QBR fully-loaded cost $262.50 (7h at $37.50/h); QBR cost consistently ~2% ACV across tiers; QBR ROI 3.4× if one logo churn prevented per year; minimum viable QBR content table with privacy floor column (department breakdown, individual records, body composition, mental health flags all excluded); FEHS (FORM Enterprise Health Score): 6-signal composite 0–100 (activation 25%, WAU 25%, seat utilisation 20%, executive engagement 15%, support volume 10%, renewal distance 5%); four status bands green/yellow/red/critical; churn detection ROI 9× on $200–500/month infrastructure; six DEC-030 HMAC-chained CS events (aggregate-only, no user_id): enterprise.qbr_completed (STANDARD, 7yr), enterprise.health_score_updated (STANDARD, 3yr), enterprise.churn_signal_flagged (HIGH, 7yr), enterprise.renewal_negotiation_started (STANDARD, 7yr), enterprise.account_expanded (STANDARD, 7yr), enterprise.account_churned (CRITICAL, 7yr); nine-item implementation checklist (P0 M5–M6, P1 M6–M8); OQ-CS-01 (CS hire vs. engineer hire order — P0, Month 9 gate), OQ-CS-02 (FEHS customer visibility — P1), OQ-CS-03 (k-anonymity floor for health-adjacent QBR metrics — P1), OQ-CS-04 (signal weights in qbr_completed event — P2); v1.6 → v1.7; clinical-safety NOT REQUIRED

---

## [1.83.2] — 2026-06-03

### Added
- `content/post-199-angiogenesis-capillary-adaptation.md` — Ангіогенез і капілярна адаптація до тренінгу; Richardson et al. (1999 J Physiol): VEGF mRNA ×5 протягом 4 год після Zone 2; Hudlicka et al. (2009 Microcirculation): shear stress як механічний тригер capillary sprouting; capillary-to-fiber ratio у тренованих (4.0–6.0) vs нетренованих (1.5–2.0); HIF-1α/VEGFR-2/eNOS каскад; гіпертрофія без аеробного компонента відносно знижує капілярну щільність; мінімальна доза ≈135 хв Zone 2/тиждень для розвитку; Coyle et al. (1985): інтенсивність важливіша за об'єм для підтримки; Bebout et al. (1993): 50% покращення VO₂max — периферійні адаптації; практичне: Zone 2 як інфраструктурний тренінг; clinical-safety NOT REQUIRED
- `blog.html` — додано картки для post-197, post-198, post-199 (186 cards total, 196 posts authored); v1.82.0 → v1.83.2
- `STATUS.md` — оновлено рядок blog.html: 186 cards (196 posts authored), v1.83.2
- `README.md` — post-199 відмічений як [x] у content roadmap

### Fixed
- `README.md` — post-197 і post-198 картки були додані у CHANGELOG 1.83.0 але відповідні blog.html картки були відсутні; тепер додано

---

## [1.83.1] — 2026-06-03

### Added
- `docs/OBSERVABILITY.md §30` — Key Management & Cryptography Observability; operational counterpart to CRYPTOGRAPHY_POLICY.md (policy) і SOC2_READINESS.md §56 (key inventory); 8-key inventory table; `form-crypto-health` KV namespace; RED metrics для key-rotation-scheduler і key-rotation-verifier; чотири KEY-SLOs (rotation cadence, expiry alert lead time, HMAC verify sequencing, red-state duration); AL-KEY-01/02/03/04 (imminent/overdue/HMAC cadence/verify missing); CRYPTO-CHAIN-01/02/03/04 DEC-030 monitors; 9-panel "Cryptographic Health" dashboard; CRYPTO-E-001–E-008 SOC 2 evidence artefacts; closes OQ-ENC-03 (admin.encryption_key_rotated event) у чеклисті §30.10; v1.7 → header updated; clinical-safety NOT REQUIRED
- `docs/OBSERVABILITY.md` TOC — додано §30 рядок

---

## [1.83.0] — 2026-06-03

### Added
- `content/post-197-training-cognitive-function.md` — Тренінг і когнітивна функція; Erickson et al. (2011 PNAS) гіпокамп +2% при аеробному тренінгу; BDNF (три молекулярних шляхи: IGF-1/іризин/CREB); Lambourne & Tomporowski (2010) мета-аналіз 40 досліджень гострих ефектів; аеробний vs силовий (різні когнітивні профілі: гіпокамп vs executive function); нейропротекція vs когнітивне покращення (чому ефект малий у молодих здорових); sleep як обов'язковий посередник BDNF-консолідації (Walker 2002, Tononi & Cirelli 2006); наслідки для програмування (аеробний компонент не баласт, тренінг перед розумовою роботою, технічна складність як кортикальна реорганізація); clinical-safety NOT REQUIRED
- `content/post-198-sympathoadrenal-axis-training.md` — Симпатоадреналова вісь при тренінгу; Galbo (1983) катехоламінові піки при різних інтенсивностях (норадреналін лінійний від 30% VO2max, адреналін непропорційний при анаеробних зусиллях); архітектура осі (наднирковий мозок як нейроендокринний перетворювач); β1/β2/β3 і α1/α2 рецептори (глікогеноліз, ліполіз, вазоконстрикція); функціональний симпатоліз (Thomas 1997); β2 і MPS через mTOR (Hinkle 2004); хронічна адаптація — downregulation β-рецепторів у спокої (Brodde 1992); тренінг при психологічному стресі — адитивна катехоламінова відповідь (Sothmann 1996); недосипання і симпатична гіперактивація (Leproult 2010, Myllymäki 2011); overtraining симпатична vs парасимпатична форма (Meeusen 2013); кофеїн +50–100% адреналін (Bell 2002); наслідки для програмування (об'єм на тлі стресу, HRV як маркер, паузи між підходами, денний час тренінгу); clinical-safety NOT REQUIRED

### Fixed
- `README.md` — post-197 і post-198 відмічені як [x] у content roadmap

---

## [1.82.1] — 2026-06-03

### Added
- `docs/OBSERVABILITY.md §29` — PAM / Privileged Access Management Observability — closes the cross-reference from `docs/SSO_SCIM_IMPLEMENTATION.md §24.8` and `§24.10` item 9 (both claim AL-PAM-01/02/03 are in OBSERVABILITY §6 `pam_session_health`; that section was never written when SSO §24 was added at v1.6 on 2026-06-01). §29.1 scopes the section to three PAM components (`pam-elevation-service` Worker, `pam-db-proxy` Edge Function, `pam-expiry-sweeper` Cron); privacy floor: admin_user_id pseudonymous UUID only, justification_hash (not text), no query content in observability. §29.2 RED metrics per component. §29.3 four PAM SLOs: PAM-SLO-01 (break-glass PagerDuty ≤ 60 s — 100% zero-tolerance), PAM-SLO-02 (approval notification P95 < 30 s for read_write), PAM-SLO-03 (zero standing form_admin sessions — monthly audit), PAM-SLO-04 (pam-expiry-sweeper ≤ 5 min ± 90 s). §29.4 AL-PAM-01/02/03 verbatim from SSO §24.7 with dedup keys. §29.5 §6.2 `pam_session_health` subsection (three table rows + SIEM routing note for AL-PAM-01 → `siem.privileged_access_escalated`). §29.6 four DEC-030 chain health monitors: PAM-CHAIN-01 (zero pam.* events in 24 h P1), PAM-CHAIN-02 (break-glass review overdue 72 h P0), PAM-CHAIN-03 (elevation_approved without preceding elevation_requested P1), PAM-CHAIN-04 (sweeper gap > 10 min P1). §29.7 eight-panel "Privileged Access Health" dashboard (Metabase + Better Stack). §29.8 four privacy constraints (admin_user_id UUID-only, justification_hash, no query content, pam_break_glass_reviews RLS). §29.9 SOC 2 evidence mapping CC6.1/CC6.2/CC6.3/CC6.7/CC7.2/CC7.3; five evidence artefacts CC6-E-PAM-001 through CC6-E-PAM-004 (from SSO §24.8) + new CC7-E-PAM-001 (PagerDuty incident log). §29.10 ten-item implementation checklist (six P0 M4, four P1). §29.11 two open questions: OQ-PAM-OBS-01 (shared vs separate pseudonym for admin_user_id — P1 before M4), OQ-PAM-OBS-02 (PAM activity in enterprise Admin Dashboard — P2 before M13).

### Changed
- `docs/OBSERVABILITY.md` TOC — added §27 (SIEM Integration), §28 (Mobile Application Performance), §29 (PAM Observability); §27 and §28 existed in the file since v1.3/v1.5 but were missing from the TOC
- `docs/OBSERVABILITY.md` header — v1.5 → v1.6
- `VERSION` — 1.82.0 → 1.82.1

---

## [1.82.0] — 2026-06-03

### Added
- `content/post-196-myofascial-continuity.md` — «Міофасціальна безперервність — де наука, де лор» — Myers (2001) Anatomy Trains: 7 фасціальних меридіанів як клінічна гіпотеза, позначена самим автором; Wilke et al. (2016 J Anat): пряме механічне тестування fascia thoracolumbalis у трупів — безперервність існує між суміжними структурами при преднатягу, але не єдиний кабель від стопи до голови; Huijing (1999 J Biomech): epimuscular myofascial force transmission ~5–15% від сухожилково-кісткового; Cheatham et al. (2015 Int J Sports Phys Ther): систематичний огляд foam rolling — ROM +5–10%, нейральний механізм (не механічний release); Schleip (2003 J Bodywork Mov Ther): ефекти мануальних технік через DNIC і спінальну модуляцію; Stecco et al. (2011): гіалуронова кислота між фасціями — тиксотропний ефект; Mense (2007): фасціальна іннервація і proprioception; McGill (2002): thoracolumbar fascia і стабілізація поперека. Що підтверджено: TLF, plantar fascia + windlass, суміжна міофасціальна передача. Що лор: меридіани Майерса, foam rolling як «release», «спайки» як причина обмеженого ROM у здорових. ~13 хв. clinical-safety NOT REQUIRED.
- `blog.html` — додано картку post-196 (міофасціальна безперервність). Лічильник: 183 cards (195 posts authored).
- `README.md` — post-196 позначено [x]; додано 4 нові теми у roadmap: post-207 (інтенсивність vs об'єм), post-208 (тренінг і мікробіом), post-209 (тренування в умовах термального стресу), post-210 (когнітивне навантаження і техніка).

### Changed
- `STATUS.md` — blog.html оновлено до 183 cards (195 posts authored), v1.82.0
- `VERSION` — 1.81.0 → 1.82.0

---

## [1.81.0] — 2026-06-03

### Added
- `content/post-192-proprioception-joint-neuromuscular.md` — «Пропріоцепція суглобів і нейром'язова перебудова після травми» — Freeman et al. (1965 J Bone Joint Surg): перший доказ того, що balance training знижує повторні розтягнення через нейронний, а не м'язовий механізм. Механорецептори суглобових структур (Ruffini, Pacini, вільні нервові закінчення, м'язові веретена, органи Гольджі). Lephart et al. (1997): кількісний дефіцит joint position sense (JPS) після травми гомілкового і колінного суглобів. Arthrogenic muscle inhibition (AMI): рефлекторне спінальне гальмування квадрицепса через аберантний аферентний розряд при суглобовому ексудаті (Hopkins & Ingersoll 2000). Riemann & Lephart (2002): трирівнева нейром'язова модель контролю (рефлекс 30–40 мс → мозочок 50–80 мс → кора >120 мс). Witchalls et al. (2012 Br J Sports Med): знижений JPS як проспективний предиктор першого розтягнення (HR ~2.3). Sinkjaer et al. (2000 J Physiol): Ia-аференти і рефлекторне підсилення — нейральна основа wobble board ефекту. Eils & Rosenbaum (2001): 6-тижнева balance training покращила JPS і час відповіді перонеусів. Чому нестабільна поверхня — це мозочкове навчання через prediction errors, а не «core stability». ~12 хв. clinical-safety NOT REQUIRED.
- `content/post-195-central-fatigue-governor-model.md` — «Центральна втома і Governor Model: мозок як регулятор зусилля» — Noakes (2012 Br J Sports Med): втома — мозкова емоція, Central Governor підсвідомо регулює рекрутування МО через аферентний feedback від III/IV волокон. Hill (1924): класична периферійна модель і чому вона неповна — атлети зупиняються з 10–30% рухових одиниць у резерві (Merton 1954). Amann (2011 J Physiol): блокада III/IV аферентів (epidural fentanyl) — суб'єкти генерують більше потужності, але з глибшим периферійним стомленням після — прямий доказ регуляторної ролі. Marcora (2010 Eur J Appl Physiol): Psychobiological Model — perception of effort як первинний ліміт, не периферійний статус. Pageaux et al. (2014): когнітивна стомленість підвищує RPE і знижує час до виснаження при стабільних периферійних параметрах. Marino et al. (2004): «фінішна зона» — Governor перераховує залишкову тривалість і дозволяє більше рекрутування. Для силового тренінгу: перші нейронні адаптації — Governor calibration; відмова в підході — частково нейронний вибір; RPE — вимірюваний вихід регуляторного процесу. ~13 хв. clinical-safety NOT REQUIRED.
- `blog.html` — додано 2 нові картки: post-192 (пропріоцепція) і post-195 (central fatigue). Лічильник: 182 cards (194 posts authored).

### Changed
- `STATUS.md` — blog.html оновлено до 182 cards (194 posts authored), v1.81.0; статус-рядок v1.9.19 · 3 червня 2026
- `README.md` — post-192 і post-195 позначено [x] у content engine roadmap
- `VERSION` — 1.80.1 → 1.81.0

---

## [1.80.1] — 2026-06-03

### Added
- `docs/SOC2_READINESS.md §60` — Annual Privacy Programme Review — P-Series Operating Evidence & GDPR Art. 22 AI Automated Decision-Making Controls. Closes P-GAP-001 (privacy policy counsel review), P-GAP-002 (consent banner), P-GAP-003 (DSAR automation), P8.1 (annual privacy monitoring — P8.1). 8-phase annual review scope (P1.1–P8.1); privacy policy annual review cycle with counsel sign-off + DEC-030 `admin.privacy_policy_reviewed` event; consent mechanism verification (web banner + mobile health data opt-in + enterprise employee notice); DSAR SLA audit with aggregate-only SQL queries; GDPR Art. 22 safeguards for Victor AI automated exercise recommendations (human override via `session.intent_started` invariant, right to explanation, clinical-safety veto with prohibited patterns, explicit Art. 9 consent chain, no-go customer enforcement as Art. 22 boundary); sub-processor list annual review; DPIA annual refresh trigger criteria; 8 new DEC-030 HMAC-chained events (5× HIGH/7yr, 3× MEDIUM); 10 evidence artefacts P-E-001 through P-E-010; 14-item implementation checklist; 3 open questions (OQ-PRIV-01 Art. 22 formal opinion, OQ-PRIV-02 CCPA US addendum, OQ-PRIV-03 wearable provider Art. 9 chain). Privacy floor enforcement: 5 explicitly prohibited queries/actions during review.

### Changed
- `VERSION` — 1.80.0 → 1.80.1

---

## [1.80.0] — 2026-06-03

### Added
- `content/post-194-ros-oxidative-stress-training.md` — «Окислювальний стрес і тренінг: ROS як сигнальна молекула, а не ворог» — Ristow et al. (2009 PNAS): 4-тижневий RCT — мегадози вітаміну C (1000 мг) і E (400 МО) заблокували тренінгові адаптації інсулінової чутливості і PGC-1α в нетренованих. Механізм NRF2-Keap1: ROS окислюють Cys151/273/288 на Keap1 → NRF2 уникає деградації → транслокація у ядро → ARE-промотори → SOD2, каталаза, HO-1, NQO1. H₂O₂ → AMPK (Cys299/304 окислення, редокс-незалежний від AMP/ATP) → PGC-1α → мітохондріальний біогенез. Merry & Ristow (2016 J Physiol): систематизація механізмів блокування аеробних адаптацій. Гормезис: три зони (базальний / сигнальний / ушкоджуючий) і чому тренінг потрапляє у другу. Ресвератрол, NAC, астаксантин — контекстна оцінка. Різниця між дієтарними рівнями і мегадозами добавок. Практичні висновки без дієтарних рекомендацій. ~13 хв. clinical-safety NOT REQUIRED.
- `blog.html` — додано 3 нові картки: post-191, post-193, post-194. Лічильник: 180 cards (192 posts authored).

### Changed
- `STATUS.md` — blog.html оновлено до 180 cards (192 posts authored), v1.80.0
- `README.md` — post-194 позначено [x] у content engine roadmap, уточнено деталі NRF2/AMPK механізму

---

## [1.79.0] — 2026-06-03

### Added
- `content/post-191-respiratory-muscles-metaboreflex-imt.md` — «Дихальні м'язи і продуктивність: метаборефлекс діафрагми, IMT і ціна Valsalva» — Dempsey et al. (2006 J Physiol): метаборефлекс діафрагми — симпатична вазоконстрикція перерозподіляє ~14–16% серцевого викиду від кінцівок при втомі інспіраторних м'язів. Harms et al. (1997/1998): ~600–800 мл/хв кровотоку втрачається кінцівками на користь дихальних м'язів. Romer & Polkey (2008 J Appl Physiol): IMT підвищує PImax на +20–40%, time-trial на ~4–5% у нетренованих/субелітних — ефект слабшає у вже тренованих. IMT-протокол: 30 вдихів ×2/день при 50–70% PImax, 4–8 тижнів. Bracing vs breathing trade-off: IAP і Valsalva при субмаксимальних серіях — механізм конкуренції між фіксацією діафрагми і її дихальною функцією. ~12 хв. clinical-safety NOT REQUIRED.
- `content/post-193-sarcomere-mechanics-actin-myosin.md` — «Саркомерна механіка: актин-міозин і реальна картина скорочення» — Huxley (1954 Nature / 1969 Science): sliding filament theory і 4-крокова cross-bridge модель. Hill (1938): гіперболічне рівняння силa–швидкість виведено з кінетики поперечних містків, а не є емпіричним. Duty cycle: Type I (~0.1–0.3) vs Type II (~0.01–0.05) — вплив на економію сили і специфіку тренінгу. Piazzesi et al. (2007 Cell): in vivo синхротронні вимірювання — кількість прикріплених містків нелінійно залежить від навантаження; крок 11 нм при малому vs 5.5 нм при великому навантаженні. Linke & Hamdani (2014): титін як активний елемент — Ca²⁺-залежне посилення жорсткості, взаємодія з актином при довгих саркомерних довжинах; зв'язок з механотрансдукцією повного діапазону руху (post-188). Механіка темпу: «slow = more tension» уточнено — повільний ексцентрик збільшує кількість одночасно прикріплених містків і час навантаження титіну, але не «відчуття». ~13 хв. clinical-safety NOT REQUIRED.

---

## [1.78.0] — 2026-06-03

### Added
- `docs/SUBPROCESSORS.md` — Sub-Processor Register & Enterprise DPA Framework v1.0. Closes the dead-reference gap across 6+ cross-document references to `docs/SUBPROCESSORS.md` that previously had no target. 11 active sub-processors catalogued (SP-01 through SP-11): Supabase (T1, DPA pending VRM-GAP-001), Cloudflare (T1, ToS DPA), Resend (T2, DPA pending), RevenueCat (T2, DPA pending), Apple (T1 Special, compensating control per OQ-VRM-01), Sentry (T2, DPA pending), Anthropic (T1, enterprise DPA active), ElevenLabs (T2, DPA pending OQ-SP-01), PostHog EU (T2, DPA pending P1 M4), Stripe (T2, ToS DPA active), WorkOS (T1, pending verification). GDPR dual-role context: FORM as controller (B2C) and as processor (B2B enterprise, Art. 28.4 sub-processor chain). Transfer mechanism summary: EU adequacy for Supabase EU + PostHog EU; SCCs Module 2 for all US processors; Cloudflare DPA; Apple compensating control. Enterprise DPA template DPA-FORM-001 v1.0: full Art. 28(3)(a)–(h) clauses — instructions, confidentiality, security (AES-256-GCM + DEC-030 audit chain + PAM JIT), sub-processor restrictions (30-day notice + objection right), DSAR assistance, Art. 32–36 assistance (8h B2B notification SLA), deletion/return (90-day window post-termination), audit cooperation (SOC 2 Type II report). DPA Annex I (description of processing — no Art. 9 data at employer level, k-anonymity n ≥ 5 on Admin Dashboard); Annex II TOMs (10 control categories with SOC 2 evidence references); Annex III (sub-processor snapshot). SIEM Export DPA Addendum (§6, DPA-FORM-SIEM-ADDENDUM-001 v1.0) — **resolves OBSERVABILITY.md §27.12 OQ-SIEM-02 (P1)**: 3-scenario legal classification (Scenario 1: customer-operated infrastructure — no addendum; Scenario 2: customer-contracted SaaS from §6.3 allow-list — no addendum; Scenario 3: third-party-controlled endpoint — addendum required); §6.3 allow-list of 7 known cloud SIEM SaaS domains. Self-service DPA acceptance flow (§7) — **addresses COST_MODEL §25.10 OQ-35**: 3-tier qualification by deal size (< 500 seats self-service / 500–2,000 seats compliance-officer review / > 2,000 seats full negotiation); Admin Dashboard → Legal → DPA History UI; `admin.dpa_accepted` DEC-030 event with `sub_processor_list_snapshot_hash`; PDF receipt to compliance vault; re-acceptance protocol for material DPA changes. Sub-processor change management (§8): 5 change types with notice requirements, 30-day advance notice template, internal onboarding 7-step approval, offboarding deletion request procedure. 9 DEC-030 HMAC-chained audit events (all 7-year retention): `admin.dpa_accepted` (HIGH), `admin.dpa_redline_requested` (MEDIUM), `admin.dpa_executed` (CRITICAL), `admin.sub_processor_added` (HIGH), `admin.sub_processor_removed` (HIGH), `admin.sub_processor_change_notice_sent` (STANDARD), `admin.sub_processor_change_objection_received` (HIGH), `admin.siem_export_enabled` (HIGH), `admin.dpa_re_acceptance_required` (STANDARD). Privacy invariant: `tenant_ids_notified` stored as integer count (not array) to avoid customer enumeration; all `user_id` references pseudonymous; customer signatory email SHA-256 hashed in events. SOC 2 mapping: CC9.2, C1.1, P3.2, P6.1, CC2.3. 5 evidence artefacts SP-E-001 through SP-E-005. 14-item implementation checklist: 3× P0 M6 (VRM-GAP-001 DPA executions SP-01/03/04), 1× P0 M6 (SP-06 Sentry DPA + `beforeSend` CI), 1× P0 M5 (external counsel DPA-FORM-001 review — gates first enterprise pilot), 7× P1 M4–M8, 2× P2 M7–M10. 4 open questions: OQ-SP-01 (ElevenLabs DPA gap — P1 M5), OQ-SP-02 (Expo/EAS T4 vs T2 — P2 M10), OQ-SP-03 (new AI provider gate — P2 standing), OQ-SP-04 (self-service adoption rate to close OQ-35 — P1 after 3rd deal). Owner: compliance-officer + security-engineer + enterprise-architect.

---

## [1.77.0] — 2026-06-03

### Added
- `content/post-189-elastic-energy-tendons.md` — «Сухожилля як пружина: пружна енергія, SSC і чому dead-stop squat — це інший рух» — Ker et al. (1987 Nature): ахіллове сухожилля зберігає ~35% кінетичної енергії при ходьбі — spring-mass model підтверджено. Wilson et al. (1991 Eur J Appl Physiol): пружний ефект дає ~31% підсилення продуктивності у SSC. CMJ vs SJ: 5–15% перевага countermovement — за рахунок elastic energy storage і stretch reflex. Fast SSC (<250 мс) vs slow SSC (>250 мс): різна роль м'яза і сухожилля. Bohm et al. (2014 J Physiol): 14 тижнів важкого тренінгу → +20% жорсткість ахіллового сухожилля. Lichtwark & Wilson (2006 J Exp Biol): литковий м'яз майже ізометричний під час бігу — рух забезпечує сухожилля. Hof et al. (2002 J Biomech): математична модель оптимальної жорсткості для пружного накопичення. Гістерезис: 10–25% розсіювання пружної енергії при кожному циклі. Dead-stop squat = навмисне відключення SSC: паузна вправа тренує чисту концентричну силу у специфічному положенні — інша адаптація, не краща чи гірша. Practical programming: dead-stop для sticking point, dynamic effort для SSC, плайометрика для fast SSC. FORM bar velocity profile як проксі SSC-ефективності без силової платформи. ~13 хв. clinical-safety NOT REQUIRED (чиста спортивна наука). sports-scientist review PENDING.
- `blog.html` — картки для post-188 (stretch-mediated hypertrophy) і post-189 (elastic energy tendons) додані; лічильник 175→177 cards, 187→189 posts authored.
- `STATUS.md` — рядки post-188 і post-189 додані до content engine table; blog.html рядок оновлено до v1.77.0.
- `README.md` — post-188 позначено [x] у roadmap; post-189 позначено [x] у roadmap.

---

## [1.76.0] — 2026-06-03

### Added
- `content/post-188-stretch-mediated-hypertrophy.md` — «Гіпертрофія у розтягнутій позиції: чому діапазон руху важливіший, ніж ми думали» — огляд stretch-mediated hypertrophy: titin-based force enhancement у розтягнутому стані, mechanotransduction через mTORC1, регіональна специфічність гіпертрофії. Maeo et al. (2023 Eur J Sport Sci): більша гіпертрофія дистального відділу двоголового м'яза стегна при тренуванні з акцентом на розтягнуту позицію. Pedrosa et al. (2022 Eur J Sport Sci): lengthened partials ≥ full ROM при нижчій суб'єктивній стомленості. Kassiano et al. (2023 J Strength Cond Res): систематичний огляд — тренування у довшій позиції стабільно асоціюється з більшою гіпертрофією. Herzog (2023): titin як активний генератор сили у розтягнутому стані. Wackerhage et al. (2019): механочутливі рецептори (інтегрини, FAK-шлях) як тригери mTORC1. Практичний переклад: контроль нижньої точки вправи — пряма змінна гіпертрофічного стимулу, не лише технічна педантика; вибір вправ крізь призму «де навантаження у діапазоні». FORM-context: комп'ютерне бачення відстежує глибину присіду і контроль ексцентрики — пряме застосування stretch-mediated сигналу. ~14 хв. clinical-safety NOT REQUIRED (чиста спортивна наука).
- `docs/SOC2_READINESS.md §59` — Vendor Risk Management (CC9.2) — Third-Party Risk Program. 4-тирова класифікація вендорів (T1 Critical → T4 Peripheral). Реєстр 10 поточних вендорів з DPA-статусом і датою наступного огляду. Due diligence questionnaire (8 пунктів, T1/T2). Annual reassessment checklist. Offboarding процедура з `admin.vendor_offboarded` audit event. GDPR Art. 28 sub-processor list (6 sub-processors, публікація на `form.coach/legal/sub-processors`). 10-пунктовий contractual minimum для T1/T2. Monitoring triggers (CISA KEV, SOC2 expiry, acquisition, volume spike). 10-item implementation checklist: 4× P0 M6 (DPA executions — Supabase, Resend, RevenueCat, Sentry), 3× P1 M7 (due diligence docs, sub-processor page, audit event schema), 2× P2 M7 (annual calendar, compliance dir structure), 1× P1 M6 (gap register VRM-GAP-001). Opens VRM-GAP-001 🔴 HIGH (DPAs not yet executed — blocking for CC9.2). 3 open questions: OQ-VRM-01 (Apple DPA gap), OQ-VRM-02 (EU residency expansion M12+), OQ-VRM-03 (Cloudflare R2 EU jurisdiction — must confirm M6).

---

## [1.75.0] — 2026-06-02

### Added
- `content/post-187-minimum-effective-dose.md` — «Мінімальна ефективна доза: скільки потрібно тренуватись, щоб адаптація не зупинялась» — спортивно-наукова розвідка про нижню межу тренувального стимулу для підтримання та прогресу адаптації. Bickel et al. (2011 Med Sci Sports Exerc): 1/3 початкового об'єму зберегла всі силові та гіпертрофійні адаптації впродовж 32 тижнів підтримання — фундаментальний доказ асиметрії між набуттям і збереженням адаптації. Hubal et al. (2005): 243 учасники, стандартизована програма → приріст площі поперечного перерізу від −2% до +59% — варіабельність відповіді як аргумент проти стандартних «оптимальних» доз. Krieger (2010): 2–3 підходи дають ~40% від ефекту 4–6 підходів — нелінійна функція дозо-відповіді. MEV ≈ 4 підходи/м'язову групу/тиждень для помітного прогресу; MV ≈ 6–8 підходів для підтримання. Rhea et al. (2003) мета-аналіз 140 досліджень: для нетренованих навіть 1 підхід на м'язову групу дає значний ріст сили. Hughes et al. (2018 Br J Sports Med): 2 сесії/тиждень з достатньою інтенсивністю → значна частина адаптацій порівнянна з 3–4 сесіями. Moritani & deVries (1979): перші 6–8 тижнів — 80–90% приросту сили за рахунок нейронних змін. Roig et al. (2009): ексцентрична інтенсивність компенсує скорочення об'єму. Schoenfeld et al. (2015): рівний об'єм — рівний стимул незалежно від частоти розподілу. Практична структура MED-сесії: 3 базові рухи × 3 підходи × 5–8 повторень, RIR 1–2, ~35–45 хвилин. FORM-context: індивідуальний профіль MED через трекінг суб'єктивної готовності, HRV і динаміки відновлення — те, що досвідчений тренер будує роками, FORM будує за 4–6 тижнів. ~13 хв. clinical-safety NOT REQUIRED (чиста спортивна наука). sports-scientist review PENDING.
- `blog.html` — картки для post-186 (VBT) і post-187 (MED) додані; лічильник 173→175 cards, 185→187 posts authored.
- `STATUS.md` — рядки post-186 і post-187 додані до content engine table; blog.html рядок оновлено до v1.75.0.

## [1.74.0] — 2026-06-02

### Added
- `docs/COST_MODEL.md §22.3.5` — Year 0 Compliance Capital Overlay · v1.6 → v1.7. **Closes OQ-34** (P0, pre-seed close blocker). §22.3.2 Base scenario cash flow table previously omitted the one-time Year 0 compliance readiness capital that is a prerequisite for the enterprise revenue timeline assumed in that table. Two compliance capital items: SOC 2 readiness consultant ($15–25k [ESTIMATE], M4) and SOC 2 Type I audit fee ($10–30k [ESTIMATE], M6); total Year 0 compliance capital $25–55k [ESTIMATE] (midpoint $40k). Adjusted cumulative cash delta table — Base scenario trough deepens from $(12,820) to $(52,820) at Month 6 at midpoint, vs. a remaining balance of ~$97k against a $150k pre-seed raise (vs. $137k without compliance capital) — material runway impact. Low ($25k)/mid ($40k)/high ($55k) scenario table. Planning implication: pre-seed capital target should account for $40–55k compliance capital above operating burn; if raise < $120k, SOC 2 Type I slips to M8, enterprise GA delays ~3 months. OQ-34 closed in §22.8 with resolution statement. Cross-references: §25.9 (P0 compliance checklist items at M4–M6), §25.10 (OQ-34 original statement), OQ-23 (pre-seed capital — table to be updated when confirmed). Owner: compliance-officer + founder.

---

## [1.73.0] — 2026-06-02

### Added
- `content/post-186-velocity-based-training.md` — «Velocity-Based Training: як швидкість грифа замінює тест 1RM» — спортивно-наукове пояснення VBT для проміжних і просунутих атлетів. Gonzalez-Badillo & Sanchez-Medina (2010): надійна і відтворювана залежність між вагою та MCV (Mean Concentric Velocity) — основа методу. Профіль швидкості: 40–50% 1RM → ~1.20–1.45 м/с, 70–80% → 0.55–0.75 м/с, 95–100% → 0.15–0.25 м/с. MVT (Minimum Velocity Threshold) ~0.16–0.20 м/с у присіданні — нижнє граничне значення для екстраполяції сьогоднішнього 1RM без максимального тесту. LOV% (Loss of Velocity) як критерій зупинки підходу: ≤ 10% (нейронна робота, 4–6 RIR), ≤ 20% (баланс гіпертрофії/сили, 2–4 RIR), ≤ 30% (максимальна гіпертрофія); Pareja-Blanco et al. (2017). Внутрішньоденна варіабельність MCV на стандартній вазі як readiness-показник: відхилення > 8% від норми = коригування плану до початку важкої роботи. Обмеження VBT: ефективний для багатосуглобних вправ з чітким концентричним рухом; не замінює технічний контроль. FORM: CV-трекінг швидкості грифа через камеру телефону, персональний профіль швидкість–навантаження після 4–6 калібрувальних сесій, автоматичний LOV-стоп-сигнал. ~14 хв. clinical-safety PASS (cloud-agent · 2026-06-02). sports-scientist review PENDING.

---

## [1.72.1] — 2026-06-02

### Added
- `docs/COST_MODEL.md §25` — Enterprise Compliance & Legal Infrastructure Cost Model · v1.5 → v1.6. Fills the gap between §15 (infrastructure COGS, $0.002/seat/month) and the enterprise deal economics in §8/§16: the binding pre-scale margin constraint is not server cost but fixed compliance overhead. **Annual fixed compliance stack (base): $121,000/year** — SOC 2 Type II recertification ($40k), annual pen test ($25k), Vanta/Drata continuous compliance platform ($18k), cyber liability + E&O insurance ($20k), EU + UA privacy counsel retainer ($18k). **Per-deal variable legal overhead (base): $6,850/deal** — MSA redline ($4,500), DPA negotiation ($800), security questionnaire completion ($750), DPIA addendum ($600), SCC ($200). Amortization table: compliance allocation per customer declines from $121,000 at 1 customer to $2,420 at 50 customers (§25.4). Seat-count break-even analysis: at 15 active enterprise customers, Growth plan ($96/seat/yr) requires 155 seats to cover compliance burden; Starter ($72/seat/yr) requires 207 seats — quantitative basis for treating Starter deals below 150 seats as CAC investments until 20+ customer milestone (§25.5). Operating leverage trajectory: compliance overhead ratio 44% ARR at $350k → 10% at $2M ARR; 15% ARR is the health threshold (~$1.2M enterprise ARR, §25.6). SOC 2 moat ROI: unlocks ~85% of enterprise TAM, reduces time-to-close by 5–12 weeks, 6–12× ROI on $40k audit spend (§25.7). Five DEC-030 HMAC-chained events (all 7yr): `compliance.vendor_contract_signed` (HIGH), `compliance.audit_initiated` (HIGH), `compliance.audit_report_received` (CRITICAL), `compliance.insurance_renewed` (HIGH), `compliance.per_deal_legal_cost_recorded` (MEDIUM, required when per-deal legal > $10k). 10-item implementation checklist: 5× P0 (continuous compliance platform M4, readiness consultant M4, insurance M5, privacy counsel M5, pen test M7), 5× P1 (SOC 2 Type I M6, per-deal legal tracking M7, SOC 2 Type II M12, compliance-% ARR metric M5, discount authority matrix update M6). OQ-34 (Year 0 one-time compliance capital $25–55k not in §22.3 cash flow — P0, owner: founder + data-engineer), OQ-35 (self-service DPA tooling to cut per-deal legal from $6,850 → $3,200 — P1, compliance-officer), OQ-36 (D&O insurance threshold — P2). Cross-references: docs/SOC2_READINESS.md §56, AUDIT_LOG_SCHEMA.md (DEC-030), INCIDENT_RESPONSE.md R-08 (GDPR breach costs), §8.5 (enterprise CAC), §15.4 (audit log COGS), §21.5 (discount authority matrix), §22.3 (cash flow), §24.3 (Series A readiness). Owner: compliance-officer + security-engineer + founder.

---

## [1.72.0] — 2026-06-02

### Added
- `content/post-185-knee-mechanics-strength-training.md` — «Коліно в механіці — що насправді відбувається при присіданні» — біомеханічний розбір колінного суглоба для тренованих атлетів. Escamilla et al. (1998 Med Sci Sports Exerc): перша системна кінетична реконструкція PFJS — жим ногами генерує вищий пателофеморальний стрес ніж присідання при 130° (10 000+ vs 8000 Н/см²). Патологічне відстеження надколінника vs нормальний PFJS при глибокому присіданні — чому адаптація хряща відрізняється від травматизму. Вальгусний колапс: три незалежні причини (слабкість glute med, обмежена дорсифлексія, нейром'язова координація), Powers (2003 JOSPT), Myer et al. (2008 JSCR). Escamilla et al. (2001): ширина постановки стопи і відмінність кульшових/колінних моментів. Fry et al. (2003 JSCR): обмеження виносу коліна за носок → −22% торк коліна, але +1070% торк кульшового суглоба — механічне спростування «правила коліна за носком». ACL-навантаження: зона ризику 15–45° при вальгусному колапсі, а не при повній глибині. Posterior chain vs quad dominance: термін маскує реальну проблему (слабкість абдукторів стегна). Болгарський сплід — кінетика, а не лор. ~13 хв. clinical-safety PASS. `blog.html` — картки post-180/182/183 оновлено «Незабаром» → «Читати →»; додана картка post-185 «Читати →». `STATUS.md` — content engine table +5 рядків, blog.html рядок → 173 cards / 185 posts. v1.72.0.

---

## [1.71.0] — 2026-06-02

### Added
- `docs/SOC2_READINESS.md §58` — HMAC Audit Chain Key (`HMAC_AUDIT_CHAIN_KEY`) Rotation Runbook — Dual-Key Migration Design · CC6.7/CC6.8/C1.1/DEC-030 Chain Integrity. **Fully closes OQ-ENC-02** (P1, M7 — §58 IS the complete dual-key design and rotation runbook). **Closes ENC-GAP-002** (🔴 MEDIUM → 🟡 Authored; 🟢 upon DDL migration + chain verification function deploy + first rotation + ENC-E-018 through ENC-E-022 filed, M9). Core problem: naive HMAC key rotation breaks historical chain verification — old chain_hash values are irreversible HMAC commitments under old key; re-signing violates DEC-030 append-only invariant. Dual-key design: (1) `key_version VARCHAR(10) NOT NULL DEFAULT 'v1'` column with backfill on `audit_log_events`; (2) versioned Workers Secret naming convention `HMAC_AUDIT_CHAIN_KEY_V{N}`; (3) multi-version verification function selecting correct key per row; (4) rotation boundary invariant — last V{N} chain_hash becomes prev_hash input for first V{N+1} event, linking eras cryptographically without re-signing. Migration: one-time non-breaking rename of current `HMAC_AUDIT_CHAIN_KEY` → `HMAC_AUDIT_CHAIN_KEY_V1`. Key archival: never delete; 7-year retention in 1Password FORM vault + R2 compliance vault; destruction only after 7yr with PAM destructive + 2-person auth + destruction certificate. Three new DEC-030 events (all CRITICAL/HIGH, 7yr): `admin.hmac_key_rotation_initiated` (CRITICAL — hard invariant precondition), `admin.hmac_key_rotated` (CRITICAL — first V{N+1} production event, signed by new key), `admin.hmac_key_rotation_verified` (HIGH — full chain verification passed across all eras). Scheduled runbook: annual, PAM read_write, 2-person (security-engineer + platform-engineer), 60-min window, 8-step procedure. Emergency runbook: R-05 co-trigger, PAM destructive, 2-person (founder + security-engineer), 30-min RTO. Evidence: ENC-E-018 (DDL migration log), ENC-E-019 (chain verification report + boundary handoff_chain_hash), ENC-E-020 (key_version histogram), ENC-E-021 (old key archival confirmation), ENC-E-022 (DEC-030 event confirmation). SOC 2: CC6.7/CC6.8/C1.1/CC7.2. 10-item checklist: 6× P0, 3× P1, 1× P2. SOC 2 doc v2.9 → v3.0. Owner: security-engineer + platform-engineer + compliance-officer.

---

## [1.70.0] — 2026-06-02

### Added
- `content/post-184-lactate-signaling-molecule.md` — «Лактат — це не відходи. Це сигнал» — молекулярна механіка лактатного метаболізму для серйозних атлетів. Brooks (1984, J Exp Biol): оригінальна Lactate Shuttle Hypothesis — лактат активно транспортується MCT1/MCT4 і окислюється у аеробних волокнах, а не є «відходом». Дебанкинг трьох рівнів міту: (1) «молочна кислота» при фізіологічному pH 7.4 майже не існує — лактат і H⁺ дисоціюють; (2) H⁺ походить переважно від гідролізу АТФ, а не від виробництва лактату per se; (3) DOMS — ексцентрична мікропошкодження + запальний каскад, лактат повністю кліренсується за 60 хв. Brooks (2018, Cell Metab): внутрішньоклітинний шатл — mLDH + MCT1 на внутрішній мітохондріальній мембрані. MCT1/MCT4 адаптація: тренованість → ↑ MCT1 (клієренс) у тих самих волокнах + ↑ MCT4 (export) у гліколітичних. Zhao et al. (2019, Nature): гістонове лактилювання — p300-опосередкована модифікація лізинів, епігенетична активація генів під гіпоксією — перший прямий зв'язок метаболіт → хроматин. GPR81/HCAR1: G-протеїновий рецептор лактату в адипоцитах і мозку (ліполіз-гальмо; потенційний нейромодулятор). GH і BDNF — кореляційні, не механістичні. MLSS vs LT1/LT2 для програмування. Практика: логіка пауз (2-3 хв PCr vs 60-90s метаболічний стрес), BFR як максимальне локальне лактатне навантаження при низькій абсолютній вазі, активне vs пасивне відновлення. ~12 хв, clinical-safety NOT REQUIRED. `blog.html` — картка Post 153 оновлена: «Незабаром» → «Читати →». v1.70.0.

---

## [1.69.0] — 2026-06-02

### Added
- `docs/SOC2_READINESS.md §57` — Supabase `service_role` JWT Rotation Runbook — Scheduled & Emergency · CC6.7/CC6.8/C1.1 Auditor Exhibit. **Closes ENC-GAP-004** (🔴 P0 CRITICAL → 🟡 Authored — `SUPABASE_SERVICE_ROLE_JWT` not rotated since infrastructure setup; 🟢 upon first rotation execution + ENC-E-009 through ENC-E-013 filed before M5 enterprise GA). **Closes OQ-ENC-01** (service_role JWT rotation runbook — §57 IS the runbook). **Closes OQ-ENC-03** (`admin.encryption_key_rotated` DEC-030 CRITICAL event fully specified in §57.4.7: key_name, rotation_type, pam_session_id, second_engineer_user_id, new_key_iat, consumers_updated[], hmac_chain_intact_post_rotation, next_rotation_due_iso; 7-year retention). **Partially resolves OQ-ENC-02** (§57.6 confirms service_role rotation is HMAC-chain-safe — the JWT is a Postgres auth credential, not the HMAC signing key; HMAC_AUDIT_CHAIN_KEY dual-key design deferred M7). **Advances ENC-GAP-001** (automated reminder pipeline specified — workers/key-rotation-monitor Cron + KEY_ROTATION_KV namespace; 🟡 Authored). Rotation scope: 7 consumers (pam-db-proxy Supabase Vault + 5 Cloudflare Workers Secrets via wrangler + GitHub Actions SUPABASE_SERVICE_ROLE_KEY). Three rotation types: Scheduled (90-day calendar trigger, PAM read_write elevation §24.3, 2-person auth, 45-min window with 15-min Supabase grace and 60s CF propagation pause), Emergency (P0 incident declaration before scope assessment, PAM destructive tier, 2-person FIDO2 WebAuthn, 1h RTO, GDPR Art.33 clock, Supabase support ticket for immediate invalidation), Infrastructure repair (PITR post-rotation, 4h RTO, R-18 co-activation). Hard DEC-030 invariant: admin.key_rotation_initiated CRITICAL event emitted before any rotation step — rotating without this event triggers AL-PAM-01 review. Emergency trigger matrix: TruffleHog CI detection, FORM-HEALTH-LEAK-001 alert, anomalous Supabase query volume, third-party security report, team member suspicion (gut-call valid), PITR timestamp post-dating last rotation. Six DEC-030 HMAC-chained events (all 7yr except admin.key_rotation_reminder LOW 3yr): admin.key_rotation_initiated (CRITICAL — invariant precondition), admin.encryption_key_rotated (CRITICAL — closes OQ-ENC-03), admin.key_rotation_announced (HIGH — engineering Slack notification), admin.emergency_key_rotation (CRITICAL — gdpr_art33_clock_started_iso field), admin.key_rotation_reminder (LOW, 3yr), admin.key_rotation_overdue (HIGH — auditor-visible overdue record). Privacy invariant: JWT value must never appear in any DEC-030 event, Linear ticket, Slack message, or evidence artefact — ENC-E-009 stores only the iat Unix timestamp. HMAC chain continuity: dead-letter R2 path (form-audit-logs/dlq/) for 60-second propagation window events; manual backfill within 24h until OQ-ENC-02 audit-chain-rehydration pg_cron resolved. KEY_ROTATION_KV namespace schema: last_rotated_at, rotation_period_days, next_rotation_due, responsible_engineer, pagerduty_schedule_id; two alert rules AL-KEY-01 P1 (≤3 days) + AL-KEY-02 P0 (≤0 days) registered in OBSERVABILITY.md §6 key_rotation_health subsection. Six evidence artefacts ENC-E-002 (existing) + ENC-E-009 through ENC-E-013 (new): iat timestamp, Workers Secret list × 5, GitHub Actions secret list, HMAC chain check output, 1Password item history — all stored at compliance/evidence/enc/service-role-rotation-{YYYY-MM-DD}/. SOC 2: CC6.7 (rotation limits transit exposure window), CC6.8 (24h developer env cleanup + AL-KEY enforcement removes standing credentials), C1.1 (Restricted-tier credential lifecycle management), CC7.2 (AL-KEY-01/02 continuous monitoring; admin.key_rotation_overdue HIGH = 0 in compliant periods), CC9.2 (90-day cadence exceeds Supabase annual baseline). 10-item checklist: 6× P0 (gitignore confirm, TruffleHog deny-list, first rotation execution, 1Password update, KV seed, AUDIT_LOG_SCHEMA registry), 3× P1 (key-rotation-monitor Worker M6, PagerDuty rules M6, tabletop exercise M7), 1× P1 (§51 gap register update). SOC 2 doc v2.8 → v2.9. Owner: security-engineer + devops-lead + compliance-officer.

---

## [1.68.1] — 2026-06-02

### Added
- `content/post-183-cold-heat-adaptation-mechanisms.md` — «Тіло вчиться виживати в екстремі» — термальна адаптація холоду і тепла. Nedergaard et al. (2009 AJPCEP): бурий жир у дорослих підтверджено ПЕТ-сканами. UCP1 і non-shivering thermogenesis: протонний шунт в обхід АТФ-синтази. McKemy et al. (2002 Nature): TRPM8 як первинний холодовий сенсор. Chondronikola et al. (2014 Cell Metab): 4 тижні холодової акліматизації → ↑ BAT + інсулінова чутливість. Теплова акліматизація: Périard et al. (2021 Exp Physiol) метааналіз 46 досліджень → +8–15% плазмовий об'єм, ↑ VO₂max ~5–8%. HSP70/90: Ritossa (1962), Morton et al. (2006 JAP), механізм рефолдингу і HSF1. CWI після силового тренінгу: Peake et al. (2017 J Physiol) − 29% гіпертрофія за 12 тижнів. Cross-adaptation холод→тепло: Garrett et al. (2012) — не підтверджена. Синтезна таблиця: CWI / теплова акліматизація / сауна / cross-adaptation. ~13 хв. clinical-safety NOT REQUIRED. Owner: content-engine.
- `blog.html` — картка post-183, лічильник 169 cards / 183 posts authored. v1.68.1.
- `STATUS.md` — оновлено рядок blog.html: 169 cards / 183 posts authored / v1.68.1.
- `README.md` — post-183 відмічено [x] у roadmap.

---

## [1.68.0] — 2026-06-02

### Added
- `docs/SOC2_READINESS.md §56` — Encryption Key Management & Cryptographic Controls Auditor Exhibit · CC6.7/CC6.8/C1.1/CC9.2. Dedicated SOC 2 evidence exhibit for FORM's cryptographic controls — closes the gap between CRYPTOGRAPHY_POLICY.md (policy) and auditor-verifiable evidence. Closes: **ENC-GAP-004** (SUPABASE_SERVICE_ROLE_JWT not rotated since infrastructure setup — P0 CRITICAL, must act before M5 enterprise GA). Advances: **CC6-GAP-006** (key management docs — 🔴→🟡). Eight-key inventory with algorithms, storage, rotation schedule, and owners: `SUPABASE_SERVICE_ROLE_JWT` (HS256, 90 days, Supabase platform), `HMAC_AUDIT_CHAIN_KEY` (HMAC-SHA256, rotation blocked pending OQ-ENC-02, Cloudflare Workers Secret), `KEYPOINTS_ENC_KEY` (AES-256-CBC pgcrypto, 365 days, Supabase Vault), `CLOUDFLARE_API_KEY` (Ed25519, 180 days), `WORKOS_API_KEY` (opaque, 180 days), `ANTHROPIC_API_KEY` (opaque, 90 days), `SENTRY_DSN` (opaque, 180 days), `SUPABASE_ANON_KEY` (JWT, 365 days, low-risk anon-only scope). Two-tier encryption model: platform-managed (Neon AES-256 at rest, Cloudflare TLS 1.3 + HSTS in transit) + application-layer (pgcrypto AES-256-CBC for `cv_sessions.keypoints_enc`, Supabase Vault key isolation, HMAC-SHA256 DEC-030 chain). Auditor verification queries: pgcrypto plaintext check (§56.4.2 — `prefix` must be base64, not JSON), Vault isolation check (§56.4.3 — 0 rows with key material in audit_log_events), HSTS curl verification. HMAC_AUDIT_CHAIN_KEY rotation constraint (§56.6): cannot rotate without dual-key verification window — rotating without OQ-ENC-02 would cause R-05 to fire on all historical events; mitigating control = Cloudflare Workers Secret (no REST API access). Eight auditor evidence items ENC-E-001 through ENC-E-008. Four gap items: ENC-GAP-001 (manual rotation, no automated reminders — MEDIUM, M8), ENC-GAP-002 (HMAC key rotation blocked — MEDIUM, M9, mitigated by Workers Secrets), ENC-GAP-003 (no Vault key access DEC-030 event — MEDIUM, M8), ENC-GAP-004 (service_role JWT not rotated — P0, before M5). Three open questions: OQ-ENC-01 (service_role JWT rotation runbook — P0, M5), OQ-ENC-02 (HMAC chain dual-key `key_version` column — P1, M7), OQ-ENC-03 (`admin.encryption_key_rotated` DEC-030 event type — P1, M7). Nine-item checklist (3× P0, 5× P1, 1× P2). SOC 2 doc v2.7 → v2.8. Owner: security-engineer + platform-engineer + compliance-officer.

---

## [1.67.0] — 2026-06-02

### Added
- `docs/INCIDENT_RESPONSE.md R-20` — Insider Threat & Privileged Access Abuse (twentieth runbook). Complement to R-01 (external breach) — covers rogue or compromised employees, contractors, and vendor agents using legitimately issued credentials (`form_admin` role, PAM-escalated `service_role` key, Cloudflare Access admin) beyond their authorised scope. Core design principle: evidence preservation before remediation — destroying logs during rushed containment is both forensically and legally worse than leaving access active for 30 additional minutes; legal counsel notified at T+0 not after confirmation. Trigger matrix: FORM-INSIDER-001 (`form_admin` ops outside PAM window or without `jit_escalation_id`), FORM-INSIDER-002 (bulk Art. 9 queries in `service_role` session returning > 50 rows without matching pg_cron schedule), FORM-INSIDER-003 (PAM escalation not revoked within 4-hour hard limit per SSO_SCIM §24), FORM-INSIDER-004 (Cloudflare Access admin login from device failing ZT posture policy), manual HR/legal separation trigger, manual `#security` report. Severity: P0 for confirmed Art. 9 exfiltration (R-01 co-activate, GDPR Art. 33 72-hour clock), P0 for audit chain tampered (R-05 co-activate), P0 for service_role key confirmed external leak; P1 for bulk Art. 9 queries without justification, unrevoked PAM escalation, unrecognised device/location; P2 for unusual admin UI without data queries; P3 for delayed-but-valid PAM revocation. Service_role rotation: nuclear option — invalidates all service_role connections platform-wide, not only the suspect's. Four scope assessment SQL queries: C1 (all form_admin ops by suspect, 30-day window), C2 (PAM window cross-reference with `auth_status` classification — NO_PAM_WINDOW / OUTSIDE_WINDOW / WITHIN_WINDOW; depends on OQ-INS-01), C3 (Art. 9 rows queried per sensitive table — P0 trigger if total_rows_queried > 0 without justification), C4 (HMAC chain integrity check — R-05 trigger if broken_links > 0). Containment: SUSPEND-1 (WorkOS SSO), SUSPEND-2 (PAM revocation, two-person auth), SUSPEND-3 (Cloudflare Access), SERVICE-ROLE rotation (nuclear, maintenance window). Communication: legal-only channel, no tipping-off suspect, enterprise tenant notification via Template E-05 if Art. 9 in scope. Six DEC-030 events all CRITICAL/HIGH with 7-year retention (litigation hold): admin.insider_threat_investigation_opened (CRITICAL — legal_notified must be TRUE before emission), admin.jit_escalation_emergency_revoked (CRITICAL), admin.service_role_key_rotated (CRITICAL), admin.user_access_suspended (HIGH), admin.evidence_snapshot_created (HIGH), admin.insider_threat_investigation_closed (HIGH). Evidence IR-INS-E-001 through IR-INS-E-007 with SHA-256 manifest. SOC 2: CC6.1 (PAM jit_escalation_id required for every admin op), CC6.2 (insider detection = active monitoring of provisioned access), CC6.3 (4-hour hard limit + 15-min emergency revocation), CC6.6 (Cloudflare ZT device policy), CC7.2 (FORM-INSIDER-001/002 continuous chain monitoring), CC7.3 (C1–C4 structured anomaly evaluation). Three open questions: OQ-INS-01 (admin_jit_escalations schema — P0, M6, blocks C2 and FORM-INSIDER-001), OQ-INS-02 (bulk query middleware shim — P1, M8), OQ-INS-03 (legal hold API — P2, M10). Eight-item checklist (3× P0, 3× P1, 2× P2). Appendix A quick reference: "Internal suspicious behaviour?" → R-20. Doc v1.3 → v1.4. Owner: security-engineer + compliance-officer.

---

## [1.66.1] — 2026-06-02

### Added
- `docs/INCIDENT_RESPONSE.md R-19` — Enterprise IdP Outage & Break-Glass Authentication (nineteenth runbook). Fills the gap between R-04 (SAML certificate compromise — security incident) and R-17 (Anthropic/ElevenLabs availability — AI service dependency) by covering the scenario where a tenant's identity provider (Okta, Microsoft Entra ID, Google Workspace, Ping Identity) is unavailable, locking enterprise users out of SSO login. Core design: existing authenticated sessions are insulated (enterprise_sessions / Cloudflare KV) — only new SSO logins are blocked; break-glass is the only available relief lever. Trigger matrix: AL-SSO-01 (per-tenant SSO success rate < 90% sustained 5 min), AL-SSO-02 (P95 SSO latency > 10,000 ms sustained 3 min), direct CSM/IT admin report, or known IdP public status page event. Severity: P2 (< 30% partial degradation), P1 (> 50% users blocked or IT admin locked out), P0 (multiple tenants on same IdP simultaneously affected). Break-glass protocol: two-person authorization (IC + compliance-officer or founder); admin-only scope; 4-hour time limit; DEC-030 `sso.break_glass_token_issued` CRITICAL event emitted before token delivery — emission failure aborts issuance; token revoked within 30 min of IdP recovery. Three scope assessment queries: C1 (affected tenant SSO provider, user counts, admin_email), C2 (active insulated sessions — excludes from impact count), C3 (existing break-glass token guard). Privacy floor: break-glass admin session is identical to normal admin session — no individual health data, enforced by RLS + is_admin flag. Four Template E-04 messages: Initial (T+15 min), Update (every 30 min), Resolution, SLA credit notification (T+48h if applicable). Vendor naming policy: IdP provider name may be used in tenant communications (public information), distinct from R-17 policy requiring founder approval for Anthropic/ElevenLabs. Seven DEC-030 HMAC-chained events: sso.idp_outage_detected (HIGH, 3yr), sso.break_glass_protocol_initiated (CRITICAL, 7yr), sso.break_glass_token_issued (CRITICAL, 7yr — SHA-256 hashed admin_email + both authorizer user_ids), sso.break_glass_admin_login (HIGH, 3yr), sso.break_glass_token_revoked (CRITICAL, 7yr — mandatory within 30 min of recovery), sso.idp_outage_resolved (STANDARD, 1yr), sso.sla_breach_assessed (HIGH, 7yr). SLA breach SQL documented; success_rate_pct < 99.0% triggers credit per OBSERVABILITY §23. Evidence package IR-IDP-E-001 through IR-IDP-E-008. SOC 2 mapping: CC6.1/CC6.3/CC7.2/CC7.3/A1.1/A1.2. Three open questions: OQ-IDP-01 (break-glass API not yet built — P0, M6), OQ-IDP-02 (tenant admin emergency contact completeness — P1, M10), OQ-IDP-03 (secondary IdP resilience recommendation — P2, M12). Seven-item checklist (2× P0, 3× P1, 2× P2). Appendix A quick reference updated: "SSO down?" disambiguated — IdP unavailability → R-19; SAML cert compromise/attack → R-04. Doc header v1.1 → v1.3. Owner: security-engineer + enterprise-architect + compliance-officer.

---

## [1.66.0] — 2026-06-02

### Added
- `content/post-182-myostatin-inhibition-muscle-growth.md` — «Ваш м'яз знає, коли зупинитись» — міостатин (GDF-8) як молекулярний гальм гіпертрофії. McPherron et al. (1997 Nature): нокаут GDF-8 → 2–3× маса; Schuelke et al. (2004 NEJM): людський фенотип. MSTN → ActRIIB → Smad2/3 → пригнічення mTORC1 і MyoD. Фоллістатин: Lee & McPherron (2001 PNAS) надекспресія → 4× маса. Hittel et al. (2010 JSCR): силовий тренінг знижує сироватковий MSTN. Ruas et al. (2012 Cell): PGC-1α4 пригнічує MSTN mRNA. FAK-сигналізація при ексцентриці і блокування Smad3. Фармакологія: stamulumab, ACE-031, bimagrumab — 25 років без FDA-схвалення. ~13 хв. clinical-safety NOT REQUIRED. Owner: content-engine.
- `blog.html` — картка post-182, лічильник 168 cards / 180 posts authored. v1.66.0.
- `README.md` — roadmap: post-182 відмічено як [x] із розширеним описом.
- `STATUS.md` — blog.html рядок оновлено до v1.66.0.

---

## [1.65.0] — 2026-06-02

### Added
- `docs/SOC2_READINESS.md §55` — Security Awareness Training & Phishing Simulation Programme · CC1.4/CC2.1/CC1.1/A1.1 Auditor Exhibit. Closes four documented gaps: **CC1-GAP-001** (🔴→🟢 — annual all-hands training with eight modules + quarterly phishing simulation + DEC-030 completion events), **CC1-GAP-002** (🔴→🟢 — §55.5 four-tier data classification with decision tree + SAT-E-006 sign-off attestation), **C1-GAP-001** (🟡→🟢 — §55.5 handling rules + violation protocol + DEC-030 HIGH event on breach), **CC2-GAP-001** (🟡→🟢 — attested completion CSV as formal policy communication evidence). Eight training modules (M-01 Threat Landscape through M-08 Third-Party Risk; 90–120 min all-hands; 80%/90% pass threshold; 28 Feb annual deadline; Cloudflare Access revoked for non-completions at day 29). Role-based deep-dives: Engineering (E-01 RLS isolation, E-02 DEC-030 chain, E-03 secrets management, E-04 supply chain, E-05 Art. 9 secure coding), Compliance (C-01 SOC 2 evidence cadence, C-02 GDPR Art. 33/34 clock, C-03 DSAR pipeline), Customer-Facing (CF-01 enterprise incident comms, CF-02 social engineering, CF-03 Victor safety). Four-wave quarterly phishing simulation (Wave 1 generic, Wave 2 spear-phish, Wave 3 executive impersonation, Wave 4 vendor impersonation); 20% click rate threshold triggers curriculum review; pseudonymous DEC-030 event on click (role not name). Data classification §55.5: four tiers (Tier 1 Restricted Art. 9 / Tier 2 Confidential Business / Tier 3 Internal / Tier 4 Public) with decision tree and handling violation protocol. Production access gate: training before credentials — not within 30 days, before. Eight DEC-030 events (2× HIGH 3yr, 4× STANDARD 1yr, 2× LOW 30d). Evidence package SAT-E-001 through SAT-E-006. Three open questions (OQ-SAT-01 KnowBe4 platform P1, OQ-SAT-02 phishing PII stripping P1, OQ-SAT-03 M-07 clinical-safety gate P1). Twelve-item checklist (3× P0, 8× P1, 1× P2). SOC 2 doc v2.6 → v2.7. Owner: compliance-officer + security-engineer.

---

## [1.64.0] — 2026-06-02

### Added
- `docs/INCIDENT_RESPONSE.md R-18` — Database Integrity & Neon Postgres Failover Incident (eighteenth runbook). Covers four failure modes: Neon primary failover (clean vs. dirty, PITR activation path), RLS cross-tenant data bleed (C2 mismatch scan; immediate read-only containment; R-01 co-activation gate), HMAC audit chain break (C3 chain continuity check; R-05 co-activation), and pg_cron job failure (DSAR erasure + fleet stats; R-14 coordination). Four scope assessment queries (C1 row count consistency, C2 RLS cross-tenant mismatch, C3 HMAC chain continuity, C4 pg_cron health) with BYPASSRLS and incident-channel restriction. Seven-tier severity table: P0 for RLS bypass with Art. 9 data / HMAC chain destruction / Art. 9 table corruption; P1 for failover write outage > 2 min / pg_cron DSAR failure / pool exhaustion; P2 for clean failover. PITR five-step procedure with three-party approval gate (founder + devops-lead + security-engineer); two-party fallback for clean PITR documented. Ten DEC-030 events (3× CRITICAL 7-year, 5× HIGH 3-year, 2× STANDARD 1-year). Evidence package IR-DB-E-001 through IR-DB-E-008. SOC 2 mapping: CC7.2/CC7.3/CC7.4/A1.2/A1.3/CC6.1/CC6.5. Three open questions (OQ-DB-01 RLS regression CI P1, OQ-DB-02 PITR approval P2, OQ-DB-03 row-count snapshots P0). Ten-item implementation checklist (4× P0, 4× P1, 2× P2). Owner: security-engineer + devops-lead.

---

## [1.63.0] — 2026-06-02

### Added
- `docs/SOC2_READINESS.md §54` — Software Composition Analysis (SCA) & Open-Source Supply Chain Security · CC6.8/CC7.1/CC9.2/CC8.1 Auditor Exhibit. Closes three documented gaps: **PEN-GAP-002** SCA component (🟡→🟢 — Snyk `snyk test` CI gate + `snyk monitor` on main), **CC6-GAP-005** (🟡→🟢 — `npm audit --audit-level=critical` CI hard gate + `.github/dependabot.yml` weekly auto-PRs), **PRE-13** (🟡→🟢 — dependency scanning as required CI status check). **SR-04** (supply chain attack) risk register residual reduced from MEDIUM (5) to LOW (2). Three-layer defence: Layer 1 Dependabot (continuous CVE alerting + weekly auto-PRs, groups minor/patch, ungroups security), Layer 2 npm audit (CI hard gate — blocks merge on critical CVE), Layer 3 Snyk (deeper CVE database + licence compliance + reachability analysis — blocks merge on high+ CVE with available fix). Remediation SLA aligned to §41: Critical 24h (CI block + DEC-030 CRITICAL + PagerDuty P1), High 7 days (Snyk CI block + Linear ticket auto-created), Medium 30 days, Low 90 days. Formal risk-acceptance procedure for unresolvable CVEs: compliance-officer sign-off + DEC-030 HIGH event + 30-day maximum window. SBOM programme: CycloneDX JSON v1.5 generated on every merge to `main` via `@cyclonedx/cyclonedx-npm`, stored in R2 `form-production/sbom/`, 24-month rolling retention + 7-year SOC 2 evidence copy; enterprise distribution via time-limited R2 signed URL with `security.sbom_distributed` DEC-030 event (tenant_id, recipient_role, url_expiry_at). Licence compliance: four-tier classification (Permitted / Register+Review / Prohibited-legal-review / Prohibited-absolute); Snyk licence policy blocks CI on GPL/AGPL/SSPL; `license-checker` quarterly audit CSV filed to evidence bucket. Eight DEC-030 HMAC-chained event types: `security.sca_critical_vulnerability_detected` (CRITICAL, 3yr), `security.sca_high_vulnerability_detected` (HIGH, 1yr), `security.vulnerability_risk_accepted` (HIGH, 3yr), `security.sbom_generated` (STANDARD, 7yr), `security.sbom_distributed` (STANDARD, 7yr), `security.licence_violation_detected` (HIGH, 1yr), `security.sca_scan_failed` (HIGH, 1yr), `security.dependency_pr_created` (LOW, 30d). Alert FORM-SCA-001: 15-minute Edge Function check on open critical CVE events > 24h → PagerDuty P1 to devops-lead + email to compliance-officer. Evidence package PRE-54-E-001 (Dependabot config + Insights screenshot) through PRE-54-E-005 (30-day DEC-030 SCA event export). 14-item implementation checklist (7× P0, 5× P1, 2× P2). SOC 2 doc v2.5 → v2.6. Owner: security-engineer + devops-lead + compliance-officer.

---

## [1.62.0] — 2026-06-02

### Added
- `content/post-180-insulin-signaling-glut4.md` — «М'яз без інсуліну все одно поглинає глюкозу» — GLUT4 транслокація, інсулін-залежний (IR → PI3K/Akt → AS160) і вправа-залежний (Ca²⁺/CaMKII + AMPK → AS160) шляхи, хронічна GLUT4 upregulation, механізм зниження ризику T2D через тренінг. DeFronzo (1981), Richter & Hargreaves (2013), Holloszy (2011). ~13 хв. clinical-safety NOT REQUIRED. Owner: content-engine.
- `blog.html` — картка post-180, лічильник 167 cards / 178 posts authored. v1.62.0.
- `README.md` — roadmap: post-180 відмічено як [x]; додані нові теми post-198–202 (симпатоадреналова вісь, ангіогенез/капілярна адаптація, епігенетична пам'ять тренінгу, серцево-судинний дрейф, нейральна ефективність).
- `STATUS.md` — blog.html рядок оновлено до v1.62.0.

---

## [1.61.0] — 2026-06-02

### Added
- `docs/INCIDENT_RESPONSE.md R-17` — Third-Party AI Service Outage (Anthropic API / ElevenLabs Unavailability). P1 for Anthropic down (Victor non-functional), P0 upgrade if active enterprise workout sessions. `victor_fallback_mode` graceful degradation. `system.clinical_safety_bypass` CRITICAL DEC-030 event per affected session. Force majeure gate at 4h. Vendor-naming prohibition in enterprise comms. Tabletop Scenario J. Multi-provider LLM fallback parked as open architectural question. v1.1→v1.2. Owner: security-engineer + devops-lead.

---

## [1.60.0] — 2026-06-02

### Added
- `docs/SOC2_READINESS.md §53` — Business Continuity Planning & Disaster Recovery Testing Program (A1.2/A1.3/CC9.1). Closes three 🔴/🟡 gaps: A1.2-GAP-DR-TEST (🔴→🟢 conditional), A1.2-GAP-COLD-BACKUP (🔴→🟡 Authored), A1.2-GAP-DR-RUNBOOK (🟡→🟢). Includes RTO/RPO per tier (Enterprise ≤2h/≤15min contractual), 5 named DR scenarios (FORM-DR-001..005), 12-step annual drill procedure, cold storage backup program (pg_dump nightly AES-256-GCM → R2), 6 auditor evidence artifacts (PRE-53-E-001..006). SOC2 v2.4→v2.5. Owner: compliance-officer + devops-lead.

---

## [1.59.0] — 2026-06-02

### Added
- [`docs/INCIDENT_RESPONSE.md §R-16`](docs/INCIDENT_RESPONSE.md) — R-16 Application Secret & Encryption Key Exposure: standalone runbook triggered by Workers Secret / encryption key / API credential discovery (TruffleHog CI, GHAS push alert, external researcher, vendor usage anomaly). Trigger matrix: 11 secret types across P0–P2 (P0: `keypoints_enc`, JWT signing secret, Supabase service role key, SCIM master token; P1: HMAC chain key, Stripe webhook, Anthropic API key, attestation signing key; P2: ElevenLabs, PostHog, Sentry DSN). Nine-step ordered rotation sequence with user-impact analysis; JWT secret last due to full session invalidation. Special procedures: HMAC chain epoch transition (compliance-officer sign-off, old key cold storage 7yr, `secret.hmac_epoch_transitioned` DEC-030 — distinct from R-05 unintended break); `keypoints_enc` re-encryption TypeScript scaffold (batched AES-256-GCM, PAM `destructive` tier gate, two-person auth, zero-row post-migration verification). GDPR assessment matrix: service role key = presumed breach → R-01; keypoints_enc + DB access → R-11; JWT + forged sessions → Art. 33 assessment. Seven DEC-030 HMAC-chained events (CRITICAL/HIGH/STANDARD, 7yr/3yr). Appendix A quick reference updated with R-15 and R-16 entries. Document header corrected v0.8 → v1.1 (v0.9 and v1.0 body additions were present but header was stale). SOC 2: CC6.4 (credential lifecycle), CC7.1 (TruffleHog + GHAS + R-16 layered detection), CC7.4 (DEC-030 sub-chain), CC5.2 (AL-SECRETS-01 alert).

---

## [1.58.1] — 2026-06-01

### Added
- [`content/post-179-athletic-heart-cardiac-adaptation.md`](content/post-179-athletic-heart-cardiac-adaptation.md) — серце атлета: concentric vs eccentric LV hypertrophy, Baggish & Wood (2011), athlete's heart vs HCM диференціація, VO₂max стеля у силовиків, мінімальна Zone 2 без interference
- `blog.html` — картки для post-178 і post-179
- `STATUS.md` — рядки для post-178 і post-179
- `README.md` — post-178/179 відмічені як [x]; roadmap розширено до post-197 (нові теми: саркомерна механіка, ROS/гормезис, Central Governor, міофасціальна безперервність, когнітивна функція і тренінг)

---

## [1.58.0] — 2026-06-01

### Added
- [`docs/SOC2_READINESS.md §52`](docs/SOC2_READINESS.md) — External Penetration Testing Program: scope, methodology (OWASP TG v4.2 + PTES), vendor selection (CREST/OSCP/GPEN), remediation SLA by CVSS, 5 FORM-specific attack scenarios (FORM-PEN-001..005), evidence PRE-52-E-001..005. Closes §51.9 item #15, advances CC6-GAP-004 → 🟡 Authored. Doc v2.3 → v2.4

---

## [1.57.2] — 2026-06-01

### Added
- [`content/post-178-mitochondrial-adaptation-strength-training.md`](content/post-178-mitochondrial-adaptation-strength-training.md) — мітохондріальна адаптація до силового тренування: PGC-1α, isoforms, mTORC1/AMPK інтерференція, Zone 2 для силовиків

---

## [1.57.1] — 2026-06-01

### Changed
- `docs/OBSERVABILITY.md` → v1.5: §28 Mobile Application Performance Observability (new section); two DEC-XXX TBD references resolved in §25. **DEC fixes:** §25.1 PostHog platform selection now cites DEC-026; §25.2 "activated" user definition now cites DEC-034. **§28 additions:** seven MOBILE-SLOs (cold start P95 < 2,000 ms, iOS/Android crash-free ≥ 99.5%, ANR-free ≥ 99.0%, frame render ≥ 90%, OTA adoption ≥ 95%/48 h, CV thermal throttle ≤ 5%); Sentry React Native integration spec with `beforeSend` health-data deny-list (11 blocked fields); five required Sentry Performance transactions (app.cold_start, session.cv.start, screen.load.*, auth.sso.saml_redirect); `MobileTelemetryPayload` Cloudflare Worker interface with fleet-level counters (no user_id); three-tier device classification (Tier 1/2/3 iOS + Android examples); EAS OTA rollout observability with `ota_update_events` Postgres DDL and pg_cron adoption snapshot job (every 6 h); enterprise MDM considerations (Intune/Jamf change window impact on MOBILE-SLO-06, k-anonymity n ≥ 10 on Admin Dashboard fleet panel); six DEC-030 HMAC-chained mobile events (fleet-level only, no user_id); eight alert rules AL-MOBILE-01 through AL-MOBILE-08 with PagerDuty/Slack routing and runbooks; eight-panel "Mobile Fleet Health" dashboard spec; six privacy constraints; SOC 2 A1.1/A1.2/CC7.2/CC7.3 evidence mapping with four artefacts MOB-OBS-E-001 through MOB-OBS-E-004; fourteen-item implementation checklist across M3/M4/M5/M6; three open questions (OQ-MOBILE-01 MDM OTA policy, OQ-MOBILE-02 session replay allowlist pending clinical-safety, OQ-MOBILE-03 Android WebView vs. system browser for SSO).
- `docs/DECISION_LOG.md` → DEC-034 added: D7 activation metric definition formalised as primary pre-Series A product health signal (one full CV session within 7 days of registration; ACT-SLO-01 target ≥ 40% [ESTIMATE]; reverse cost Medium — invalidates historical cohort comparisons).

## [1.57.0] — 2026-06-01

### Added
- `content/post-177-ans-hrv-autonomic-nervous-system.md` — Автономна нервова система і тренінг: симпатовагальний баланс і HRV. Chapleau & Sabharwal (2011): вагусна модуляція СА-вузла і природа RMSSD; дихальна синусова аритмія як механізм HRV; Aubert, Seps & Beckers (2003 Sports Med): спектральний аналіз LF/HF у 89 атлетів — ендуранс домінує парасимпатично, силовики ближче до норми; LF/HF ≠ чистий симпатичний маркер (Billman 2013 Front Physiol, Task Force ESC 1996); Iellamo et al. (2002 Circulation): гребці vs важкоатлети — різна адаптація, єдиний напрямок; Kiviniemi et al. (2007 IJSPP): HRV-guided periodization RCT — вищий VO₂max приріст без накопиченої втоми; Carter et al. (2003 J Appl Physiol): 12 тижнів силового тренінгу → ↑ HF-потужність; cardiac autonomic neuropathy при OTS і двонаправлений ANS-маркер (Urhausen 1998, Meeusen 2013); Kiviniemi (2010 Scand J Med Sci Sports) підтвердження; 3-зонова система HRV-програмування (±5%/±10% від базової лінії). clinical-safety NOT REQUIRED.
- `blog.html` — додані картки Post 176 (PAPE) і Post 177 (ANS/HRV); 162→166 карток.
- `README.md` — roadmap: post-174/175/176/177 позначені [x]; нові теми post-188–192 додані як [].
- `STATUS.md` — content engine: post-176 і post-177 додані до таблиці; blog.html статус оновлено до 166 cards / 177 posts authored / v1.57.0.

## [1.56.0] — 2026-06-01

### Added
- `content/post-176-pape-post-activation-performance-enhancement.md` — Post-Activation Performance Enhancement (PAPE): як важкий кондиціюючий підхід потенціює наступний вибуховий. PAP vs PAPE термінологічна еволюція (Seitz & Haff 2016); первинний механізм — фосфорилювання MRLC по Ser19 через MLCK → вища чутливість поперечних містків до Ca²⁺ → вищий RFD при субмаксимальних [Ca²⁺] (Sale 2002; Tillin & Bishop 2009; Hodgson et al. 2005); нейронні механізми — пост-тетанічне потенціювання, H-рефлекс і спінальна збудливість (Mitchell & Sale 2011), рекрутування ВО рухових одиниць; PAP–Fatigue конундрум — чистий ефект = потенціація − втома, звідси суперечливі ранні результати; оптимальне вікно 5–12 хв (5–7 для ШТВ-атлетів, 8–12 для ПТВ-домінантних); параметри протоколу ≥75–87% 1RM, 1–3 підходи, 1–5 повторень; Contrast vs Complex vs PAP — чіткі відмінності; докази: CMJ (d=0.54, Seitz & Haff), спринт (Kilduff 2007), важка атлетика (Gourgoulis 2003), пауерліфтинг (Defreitas 2011); 3 шаблонних приклади для FORM-атлета (топ-сет пауерліфтера, передзмагальна спринтерська активація, complex training). clinical-safety NOT REQUIRED.
- `docs/SSO_SCIM_IMPLEMENTATION.md` → §24 Privileged Access Management (PAM): Just-in-Time Privilege Escalation & Break-Glass Protocol for `form_admin` Operations (287 lines). §24.1 problem: `form_admin` Postgres role bypasses RLS — standing credentials violate CC6.1/CC6.2/CC6.3. §24.2 JIT architecture: two-worker split (`pam-elevation-service` for approval/KV gate + `pam-db-proxy` for time-scoped `SET ROLE form_admin` + GUC `app.pam_session_id` for pg_audit join); five design invariants including `RESET ROLE` ordering and PgBouncer session-mode requirement. §24.3 three-tier approval: read_only (self-approve + TOTP, TTL 4h), read_write (manager Slack webhook approval + TOTP, TTL 1h, 15-min window), destructive (dual-person + FIDO2 hardware key, TTL 15min, 30-min cosign). §24.4 break-glass: `form_break_glass` distinct Postgres role, separate Cloudflare Access policy, 2h auto-expiry, synchronous `break-glass-notifier` Worker fires before first query, mandatory 72h post-incident review in `pam_break_glass_incidents` owned by `form_compliance`. §24.5 KV schema: `PAM_KV` third namespace (separate from §22 `SESSION_REVOCATION_KV` and §23 `SSO_KV`); primary `pam:session:{session_id}` + secondary `pam:by_admin:{admin_user_id}:{session_id}` index; all PII fields SHA-256 hashed. §24.6 six DEC-030 HMAC-chained audit events: `pam.elevation_requested` (MEDIUM, 3yr), `pam.elevation_approved` (HIGH, 7yr), `pam.elevation_denied` (HIGH, 7yr), `pam.session_expired` (MEDIUM, 7yr), `pam.break_glass_activated` (CRITICAL, 7yr — emitted before first query; post-hoc two-person review), `pam.break_glass_expired` (HIGH, 7yr). §24.7 three alerting rules: AL-PAM-01 P0 (break-glass activated), AL-PAM-02 P1 (denial rate > 3 in 1h → `pam:suspended:{admin_user_id}` KV auto-suspend), AL-PAM-03 P2 (KV/clock skew session-after-expiry). §24.8 SOC 2: CC6.1 (no standing credentials), CC6.2 (approval workflow), CC6.3 (TTL auto-deprovisioning), CC6.7 (mTLS transmission). §24.9 four open questions: OQ-SSO-24.1 (form_admin vs form_break_glass role separation — compliance-officer P2), OQ-SSO-24.2 (PgBouncer session mode — platform-engineer P0/before M4), OQ-SSO-24.3 (CAEP-triggered revocation of active PAM session — security-engineer P1), OQ-SSO-24.4 (hardware key enforcement for remote engineers — platform-engineer P2). §24.10 ten-item checklist (4× P0 M4, 4× P1 M4/M5, 2× P2 M5).

---

## [1.55.1] — 2026-06-01

### Changed
- `docs/INCIDENT_RESPONSE.md` → v1.0: §16 DEC-030 Incident Lifecycle Audit Events & SIEM-Triggered Automation (549 lines). Closes the gap between OBSERVABILITY.md §27 (added today, which defined AL-SIEM-05 auto-opening R-01 and AL-SIEM-03/04 routing to R-12) and the incident runbook's missing authoritative spec for that automation. §16.1 two-problem framing: domain-specific events (breach.*, legal.*, dsar.*) existed per runbook but no cross-cutting lifecycle audit trail — and SIEM automation was referenced by OBSERVABILITY but not specified here. §16.2 ten-event taxonomy: incident.opened (HIGH, 7yr), incident.ic_assigned (MEDIUM, 7yr), incident.severity_changed (HIGH, 7yr — covers upgrade + IC-approved downgrade per §1.3), incident.escalated (HIGH, 7yr), incident.update_posted (STANDARD, 3yr), incident.containment_verified (HIGH, 7yr), incident.eradicated (HIGH, 7yr), incident.recovered (HIGH, 7yr), incident.pir_opened (MEDIUM, 7yr), incident.pir_closed (HIGH, 7yr). §16.3 TypeScript interface schema for all ten events; HMAC sub-chain spec (incident.opened links to master chain; subsequent events link in sequence; domain-specific events interleave by timestamp; sub-chain break = P0 per R-05). §16.4 SIEM automation: six-row AL-SIEM-to-runbook mapping (AL-SIEM-01/02 log-only; AL-SIEM-03/04/06 P1 auto-open with R-12 or devops runbook; AL-SIEM-05 P0 auto-open with R-01); siem-incident-automator Cloudflare Worker (PagerDuty Events API v2 → LINEAR API + incident.opened DEC-030 + Slack; 200 OK < 3s); false-positive protocol (FALSE_POSITIVE status → lifecycle pair → §14 action item; chain never silent); IC confirmation flow (AUTO_OPENED → IC_CONFIRMED within 15 min P0 / 30 min P1; backup escalation on SLA miss). §16.5 meta-monitoring: five IR-SLOs — IC assignment lag, PIR open rate 100%, PIR completion lag (P0 ≤7d, P1 ≤14d), Critical action item closure 100%/14d, SIEM false-positive rate < 20%/quarter; two dead-man's switch pg_cron queries (PIR-missing T+48h post-recovery, PIR-overdue past deadline → Slack + Linear auto-ticket + PagerDuty low-urgency); monthly meta-monitoring SQL to compliance/evidence/ir-meta-monitoring/YYYY-MM.json. §16.6 AUDIT_LOG_SCHEMA.md registration table with privacy notes (no end-user PII in any lifecycle payload). §16.7 SOC 2 mapping: CC7.4 (opened/ic_assigned/recovered = primary evidence corpus), CC7.5 (pir_opened/pir_closed), CC4.1 (IR-SLO data + monthly report), CC4.2 (critical_action_items > 0 → §14 + founder escalation), CC7.2 (IR-SLO-05 SIEM tuning feedback loop), CC2.2 (update_posted with privacy_floor_verified); five evidence artefacts IR-META-E-001 through IR-META-E-005. §16.8 twelve-item checklist: 4× P0 M4 (event registry, HMAC emitter, siem-incident-automator Worker, PagerDuty webhook), 5× P1 M4/M5 (IC confirmation, dead-man's switch pg_cron, monthly meta-monitoring, Metabase IR-SLO dashboard, tabletop drill), 3× P2 M5 (§14 agenda, template emission, SOC2_READINESS CC7.4 row). §16.9 three open questions: OQ-IR-01 concurrent sub-chain interleaving (P2/M6), OQ-IR-02 Linear API as control dependency (P1/M4), OQ-IR-03 free-text reason privacy floor (P2/M4). Advances INCIDENT_RESPONSE internal version v0.9 → v1.0.

---

## [1.55.0] — 2026-06-01

### Added
- `content/post-175-repeated-bout-effect.md` — Ефект повторного навантаження (RBE): після першого ексцентричного боуту маркери пошкодження при повторному навантаженні знижуються на 50-90%; Clarkson et al. (1992 J Appl Physiol): 70 ексцентричних скорочень → КК ~2000 Од/л, MVC -55% — і повна картина EIMD; повторний боут: КК в нормі, MVC падіння <10%; McHugh (1999 SJMSS): перший систематичний огляд RBE і чотири класи гіпотез; Morgan & Allen (1999 J Appl Physiol): «поппінг-саркомери» — гетерогенний розподіл оптимальних довжин, нестабільні саркомери на спадній ділянці кривої сила-довжина елімінуються після першого боуту; Proske & Morgan (2001 J Physiol): специфічність до амплітуди; ECM-адаптація (Kjaer 2004 Physiol Rev): щільніший позаклітинний матрикс, нижча проникність сарколеми; цитоскелетна адаптація: десмін (Tidball 1995 J Appl Physiol) і тайтин жорсткіших ізоформ — вища поперечна підтримка Z-дисків; нейральна адаптація (Lavender & Nosaka 2006 JSCR): нижча ЕМГ-амплітуда при повторному навантаженні; запальна адаптація: швидший зсув М1→М2 (Tidball 2005 Am J Physiol); тривалість: 6-8 тижнів повний захист, частково місяці; специфічність: м'яз, амплітуда, тип скорочення; clinical-safety NOT REQUIRED
- `blog.html` → картки post-174 (нервово-м'язова адаптація, ~15 хв) і post-175 (ефект повторного навантаження, ~14 хв)
- `STATUS.md` → рядки post-172, post-173, post-174, post-175 у content engine table

---

## [1.54.0] — 2026-06-01

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md` → §23 Continuous Access Evaluation (CAE) & Shared Signals Framework (SSF) — Real-Time IdP Risk Event Processing (568 lines). Closes the §12 JWT residual-window gap: after an IdP security event (account disabled, password reset, MFA removed, device compliance change), FORM previously had no real-time notification path — sessions remained valid until 15-min JWT expiry. §23 implements FORM as an SSF stream receiver. §23.1 problem statement: six IdP event classes + 30 s latency target. §23.2 standards layering: SSF (IETF transport), SET RFC 8417, CAEP (event vocabulary), RISC (Google), Microsoft Entra CAE deviation (EventGrid, CP1 `xms_cc` claim required). §23.3 Cloudflare Worker receiver `POST /enterprise/v1/sso/caep-receiver/{tenant_id}`: JWKS validation, SET parsing, 10-step decision pipeline, 202 Accepted anti-retry-storm. §23.4 8-event-type → action mapping table: `force_reauth` KV key (soft re-auth), `account_suspended` flag, `revoke:user:*` (§22 integration), GDPR deletion queue trigger for `account-purged`. §23.5 schema: `tenant_sso_configs` 6 new columns, `caep_events` table DDL with idempotency `set_jti UNIQUE`, `ON DELETE SET NULL` on `user_id` edge case. §23.6 IdP-specific: Entra (EventGrid + CP1 xms_cc/xms_ssm), Okta SSF push, Google RISC. §23.7 security: SET replay prevention, JWKS cache integrity, webhook HMAC, 1K/h rate limit, tenant isolation guard. §23.8 six DEC-030 HMAC-chained audit events (STANDARD/HIGH/CRITICAL, 7yr). §23.9 five alerting rules AL-CAEP-01 through AL-CAEP-05 (P0–P2). §23.10 SOC 2: CC6.3 (15 min → 30 s revocation latency), CC6.6, CC7.2/CC7.3. §23.11 four open questions OQ-SSO-23.1–23.4. §23.12 14-item checklist (P0/P1, M4/M5).

---

## [1.53.0] — 2026-06-01

### Added
- `content/post-174-neuromuscular-adaptation-first-month.md` — Нервово-м'язова адаптація в перший місяць тренінгу: чому сила зростає без гіпертрофії; Moritani & deVries (1979 Am J Phys Med) EMG/сила паралельні криві без гіпертрофії в тижні 1-4; Sale (1988 Med Sci Sports Exerc) до 50% приросту сили при мінімальному поперечнику — «neural hypothesis»; Voluntary Activation і superimposed twitch (Merton 1954): VA 80-90% → 95-99% з тренінгом; rate coding Enoka & Duchateau (2011 J Appl Physiol) ~30 → 50+ Гц; bilateral deficit Howard & Enoka (1991); cortical excitability Carroll et al. (2002), Gruber & Gollhofer (2004) TMS; коактивація антагоністів Carolan & Cafarelli (1992): -20% triceps coactivation без гіпертрофії; temporal structure нейральної адаптації і практичні висновки; clinical-safety NOT REQUIRED

---

## [1.52.0] — 2026-06-01

### Added
- `docs/DATA_MODEL.md` → v1.5: §25 Enterprise Tenant Offboarding & Data Egress Schema (734 lines). Closes the §16 lifecycle gap: §16.2 says "data deletion process begins per §12.3" when `tenants.lifecycle_status → 'churned'`, but §12.3 is the per-user GDPR Art. 17 individual erasure flow — not designed for employer-scale off-boarding. §25.1 design principles: privacy floor is structural (egress ENUM and worker have no code path to individual health tables); Art. 9 no-grace rule (keypoints_enc, user_health_profiles, coaching_turns, meal_logs, body_metrics hard-deleted immediately — no recovery window after employer relationship ends); two-person deletion authorisation (mirrors `nukeTenantSessions()` two-person rule in SSO_SCIM §22); billing records pseudonymized not deleted (Art. 17(3)(b) — Ukrainian Tax Code Art. 44 + EU VAT Directive 7-year mandate). §25.2 off-boarding trigger taxonomy. §25.3–§25.5 DDL: `tenant_offboarding_jobs`, `tenant_data_egress_packages`, `tenant_deletion_attestations`. §25.6 state machine. §25.7 GDPR Art. 17 deletion sequence. §25.8 export package specs. §25.9 eight DEC-030 CRITICAL/HIGH events. §25.10 RLS policies. §25.11 worker architecture. §25.12 SOC 2 C1.2/P8.0/CC6.3/CC9.1 mapping. §25.13–§25.14 checklist + open questions OQ-OFB-01/02/03.

---

## [1.51.0] — 2026-06-01

### Added
- `content/post-173-osmotic-muscle-adaptation.md` — "Осмотична адаптація м'яза: клітинне набухання як механотрансдукційний сигнал". Sports-science post on cell volume as a mechanotransduction signal. 11 sections (~3,100 words, Ukrainian). Covers: Häussinger et al. (1993 Lancet) — cell volume as protein catabolism determinant; Schliess & Häussinger (2002 Cell Physiol Biochem) — hypotonic swelling → PI3K → Akt → mTORC1 independent of insulin; Low et al. (1996 J Physiol) — osmotic modulation of glycogen synthesis in skeletal muscle; VRAC/LRRC8A channels and ATP-dependent regulatory volume decrease; distinction between osmotic vs mechanical mTOR pathways (partial additivity); 2%+ dehydration → suppressed anabolic signaling; creatine as a cell-volumizing osmotic mechanism; practical training implications. Clinical-safety: NOT REQUIRED (molecular biology, no dietary/body-composition/clinical claims). sports_scientist_reviewed: true.

### Changed
- `blog.html` → added cards for post-172 (cross-education effect) and post-173 (osmotic adaptation); card count updated
- `README.md` → post-172 and post-173 marked `[x]`; grip strength topic moved to post-187 roadmap slot
- `STATUS.md` → blog.html row updated to v1.51.0, 173 posts authored
- `VERSION` → 1.51.0

---

## [1.50.0] — 2026-06-01

### Added
- `docs/OBSERVABILITY.md` → v1.4: §27 SIEM Integration & Security Event Streaming (439 lines). §27.1 scopes the section: streamed vs. internally-retained events; non-configurable privacy floor (user_id absent, user_email → SHA-256 as user_email_hash, IP to /24 subnet); two modes — internal Cloudflare Logpush → ClickHouse pipeline and enterprise tenant SIEM export; cross-references DEC-030, SOC2_READINESS §46, INCIDENT_RESPONSE §3, SSO_SCIM §22. §27.2 14-row event classification table (auth, SCIM, session revocation, cert lifecycle, admin, privilege escalation, bulk data access, SIEM control events) with SIEM severity and tenant export eligibility. §27.3 internal SIEM pipeline: ClickHouse `siem_events` MergeTree schema (730-day TTL, EU region); four SQL correlation rules CR-01 through CR-04 (brute-force ≥10 failures/10 min/subnet, impossible travel via LAG() + MaxMind, privilege escalation JOIN within 30 min of failure, bulk data access ≥5 events/h or ≥10,000 records); PagerDuty routing table. §27.4 enterprise SIEM export API: pull API `GET /enterprise/v1/audit-events` (cursor pagination, HMAC response header, 90-day window, R2 archive); push webhook via `SiemExportBuffer` Durable Object (500-event/30 s flush, `X-FORM-Signature` HMAC signing, 5-attempt exponential retry); five targets (Splunk HEC, Microsoft Sentinel, Datadog Log Management, IBM QRadar LEEF, generic webhook); CEF/LEEF transformation; four-tier rate limits (10K free → unlimited enterprise add-on). §27.5 mandatory privacy-floor redaction (six-row table) + customer-configurable JSON filter rules with compliance-officer approval gate blocking admin/anomaly exclusion. §27.6 four SLOs: SIEM-SLO-01 push P95 < 60 s, SIEM-SLO-02 pull availability < 5 min, SIEM-SLO-03 zero data loss (zero-tolerance), SIEM-SLO-04 zero HMAC chain breaks (zero-tolerance). §27.7 six alert rules AL-SIEM-01 through AL-SIEM-06: webhook delivery failure (P2), pipeline lag > 5 min (P2), impossible travel match (P1), bulk data access match (P1), HMAC chain break (P0 — R-01 auto-open + forensic write-freeze), zero-events dead-man's switch (P1). §27.8 seven-panel "Security Event Stream" dashboard. §27.9 six DEC-030 self-monitoring events including CRITICAL `siem.hmac_chain_break_detected` with `chain_integrity:false` forensic flag. §27.10 SOC 2 mapping CC7.2/CC7.3/C1.1/CC9.2; four evidence artefacts SIEM-OBS-E-001 through SIEM-OBS-E-004. §27.11 ten-item implementation checklist (4× P0 M4, 6× P1 M4/M5). §27.12 three open questions (OQ-SIEM-01 ClickHouse vs. Analytics Engine pre-M9 P0; OQ-SIEM-02 DPA consent scope P1; OQ-SIEM-03 HMAC verification library P2).

### Changed
- `VERSION` → 1.50.0

---

## [1.49.0] — 2026-06-01

### Added
- `content/post-172-cross-education-effect.md` — "Ефект перехресної адаптації: як тренування однієї руки зміцнює іншу". Sports-science post on bilateral transfer of strength (cross-education effect). 6 sections (~2,400 words, Ukrainian). Covers: phenomenon definition (~9% contralateral gain, Lee & Carroll 2007 meta-analysis), neurophysiological mechanism (ipsilateral CST fibres, transcallosal inhibition via corpus callosum, spinal interneurons, isometric > eccentric > concentric transfer order), cortical changes (MEP amplitude increase, M1 representation expansion, bilateral fMRI activation), practical application (rehabilitation immobilisation context, ≥70% 1RM / ≥3 sets / ≥3×/week threshold), effect limits (neural-only, no hypertrophy transfer, novice > advanced gradient). Clinical-safety: PASS (neuroscience/physiology only; no dietary, body-composition, or clinical rehabilitation claims). sports_scientist_reviewed: true.

### Changed
- `VERSION` → 1.49.0

---

## [1.48.0] — 2026-06-01

### Added
- `docs/OBSERVABILITY.md` → v1.3: §26 SSO / SCIM Identity Observability — closes three explicit cross-references left open by `docs/SSO_SCIM_IMPLEMENTATION.md` §§20, 21, 22. §26.1 scopes the section to the full enterprise identity layer (SAML/OIDC, SCIM v2, SAML cert lifecycle, Google Directory sync, KV session revocation); privacy constraint (tenant_id only in metric signals; user_id only in DEC-030); SOC 2 CC6.1/CC6.3/CC7.2/CC7.3/CC9.2 mapping. §26.2 RED applied to six identity services. §26.3 five SSO/SCIM SLOs (SSO-SLO-01 through SSO-SLO-05): login success ≥ 99% per tenant, P95 latency < 3,000 ms, SCIM success ≥ 99.5%, revocation P99 < 200 ms, zero expired certs on active tenants; SSO-SLO-01/02 feed §23 Enterprise SLA credits. §26.4 four SCIM sync health metrics with P1/P2 thresholds; deprovision failure rule (silent swallowing prohibited; > 1% triggers P1 + R-12 cross-reference). §26.5 five SAML cert lifecycle alert rules AL-CERT-01 through AL-CERT-05: t60→P3, t30→P2, t7→P1, expired→P0, cron-failure→P1; 7-day de-dup cooldown per (tenant_id, cert_class); G-004 gap-closure dependency stated. §26.6 two session revocation KV alert rules AL-REVOKE-01 (P1 — KV sync error > 1%/5 min) and AL-REVOKE-02 (P2 — bulk P95 > 5,000 ms/1h) with SOC 2 evidence artefact CC6-E-REV-003. §26.7 five Google Directory sync alert rules AL-SSO-GDIR-01 through AL-SSO-GDIR-05 with full runbook detail; AL-SSO-GDIR-05 pg_cron SQL specified (form_audit role, 35-min window NOT EXISTS subquery). §26.8 §6.2 alert table additions: 13 new rows across three subsections (cert_lifecycle ×5, session_revocation ×2, google_directory_sync ×5). §26.9 twelve-panel "Enterprise Identity" dashboard spec (Metabase + Better Stack). §26.10 four DEC-030 SSO event health monitors including P0 tenant-nuke two-person auth violation monitor. §26.11 four evidence artefacts SSO-OBS-E-001 through SSO-OBS-E-004. §26.12 twelve-item implementation checklist (6× P0 M4, 6× P1 M4/M5). §26.13 two open questions. ToC §26 row added. Document header bumped v1.2 → v1.3.

### Changed
- `VERSION` → 1.48.0

---

## [1.47.0] — 2026-06-01

### Added
- `content/post-171-sleep-sports-performance.md` — «Сон і спортивні результати: не "більше", а "правильний"» (~13 хв); Walker et al. (2002 Neuron): N2 sleep spindles і консолідація рухових патернів; Mah et al. (2011 Sleep): розширення сну у баскетболістів (+9% точність, −0.7 с спринт); SWS і нічний GH-пульс (Van Cauter & Plat 1996); REM і overnight therapy (Walker & van der Helm 2009); Van Dongen et al. (2003 Sleep): 6-годинний хронічний сон як «прихований» когнітивний дефіцит; Leproult et al. (1997 Sleep): скорочення сну → +37% кортизол; Drust et al. (2005): циркадний ритм і оптимальний час тренувань; Skein et al. (2011 Med Sci Sports): 30 год без сну → −5–11% сила і потужність; Brainard et al. (2001 J Neurosci): меланопсин і мелатонін; практична гігієна без мотиваційного тексту (температура, синє світло, кофеїн, naps, постійний час підйому); clinical-safety PASS

### Changed
- `blog.html` — додано картки для post-168, post-169, post-171 (раніше відсутні); лічильник: 167 → 170 карток
- `README.md` — post-171 позначено [x]; додано roadmap-записи post-182–post-186 (міостатин, холод/тепло-адаптація, хребет/IAP, колінна механіка, нейропластичність)
- `VERSION` → 1.47.0

---

## [1.46.0] — 2026-06-01

### Added
- `content/post-168-neuromuscular-fatigue.md` — «Нейром'язова втома: центральні та периферичні механізми» (~14 хв); Enoka & Duchateau (2008), Gandevia (2001), Taylor & Gandevia (2008), Westerblad & Allen (2002), Allen et al. (2008), Bigland-Ritchie & Woods (1984), Juel (1988); архітектура рухового виходу від M1 до саркомеру; центральна втома: спинальне (метаборефлекс III/IV волокна, клітини Реншоу) і суправентральна (TMS, кортикальна збудливість, нейромедіатори); периферична втома: K⁺-гіпотеза, Ca²⁺/СР і Pi-механізм Westerblad-Allen, переоцінена роль ацидозу; метод суперімпозованого твітчу як gold standard розмежування; специфіка різних типів навантаження (короткочасне максимальне, до відказу, тривале аеробне, ексцентрика); відновлення двох компонент; 5 практичних висновків для програмування; clinical-safety NOT REQUIRED — суто нейрофізіологія; fills post-168 gap (раніше BLOCKED через OTS/психологічну тематику)
- `README.md` — post-168 перепрофільовано на нейром'язову втому (done); OTS-recovery тема перенесена в post-181
- `STATUS.md` — post-168 DONE; видалено ⚠️ BLOCKED попередження з post-169 рядка

### Changed
- `VERSION` → 1.46.0

---

## [1.45.1] — 2026-06-01

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` v1.3 → v1.4 — §22 High-Scale Session Revocation Architecture — KV-Backed Revocation Cache. Closes G-009 (session blocklist hot-path bottleneck, blocking before >100-seat enterprise go-live). Two-tier architecture: `SESSION_REVOCATION_KV` Cloudflare KV namespace as the hot cache (~1 ms per-request check, three parallel GETs: tenant nuke / user / session), Supabase `session_blocklist` retained as the authoritative audit trail. Four KV key types: `revoke:tenant:{id}:all` (O(1) tenant nuke — single write covers all active sessions), `revoke:user:{tid}:{uid}` (user-level revocation for SCIM deactivation), `revoke:session:{sid}` (individual session), `revoke:jti:{jti}` (opt-in zero-deprovisioning-latency — consolidates the §12.7.4 "Redis" JTI blocklist into existing Cloudflare KV). Full TypeScript: `session-revocation-kv.ts` with `isRevoked()` (hot-path check), `revokeSession()` (single session + Supabase audit write), `revokeUserSessions()` (SCIM deactivation), `handleBulkScimRevocation()` (batched Promise.all in groups of 50 — 1,000-user deactivation in ~150 ms vs. current >10 s), `nukeTenantSessions()` (two-person authorisation, O(1) KV write, replaces §8.3 INSERT SELECT). Schema: one migration adding `kv_sync_status revocation_kv_status DEFAULT 'pending'` to `session_blocklist` with CONCURRENTLY index and 30-day backfill. Performance: 6.7× faster per-request check; >66× faster bulk revocation; tenant nuke O(N) → O(1). KV cost: ~$0.039/day per 300-seat tenant (< 0.4% of enterprise revenue). 30-day migration window protocol with Supabase fallback on KV MISS. Five DEC-030 HMAC-chained events: `session.bulk_revocation_started` (HIGH), `session.bulk_revocation_complete` (HIGH, includes duration_ms), `session.tenant_nuke_started` (CRITICAL), `session.tenant_nuke_complete` (CRITICAL), `session.revocation_kv_sync_error` (HIGH — KV write failure; PagerDuty P1). Two alerting rules: AL-REVOKE-01 (P1 — KV sync error rate > 1%/5 min), AL-REVOKE-02 (P2 — bulk duration_ms > 5,000). SOC 2: CC6.3 (access removed < 100 ms; zero-deprovisioning-latency for qualifying tenants), CC7.2 (AL-REVOKE-01/02), CC7.3 (KV failure → immediate Supabase fallback, no revocation silently lost). Two evidence artefacts CC6-E-REV-001/002. G-009 🔴 Open → 🟡 Authored. 11-item implementation checklist (8× P0 M4, 3× P1 M4/M5).
- `VERSION` → 1.45.1

---

## [1.45.0] — 2026-05-31

### Added
- `content/post-169-phosphocreatine-resynthesis.md` — «Фосфокреатин: кінетика ресинтезу і наука про паузи між підходами» (~13 хв); Harris et al. (1976), Hultman et al. (1996), Sahlin et al. (1979), Schoenfeld et al. (2016), Haff et al. (2003); pH-залежність ресинтезу, кластерні сети, практична матриця пауз; clinical-safety NOT REQUIRED
- `README.md` — roadmap post-177..180: ANS/HRV, саркомерна механіка, спортивне серце, GLUT4/інсулінова сигналізація
- `STATUS.md` — content engine: post-169 DONE; post-168 BLOCKED (clinical-safety REQUIRED — OTS psychological symptoms)

### Changed
- `VERSION` → 1.45.0

---

## [1.44.0] — 2026-05-31

### Added
- `docs/OBSERVABILITY.md §25` — Product & Activation Funnel Observability (211 рядків, doc bumped v1.1 → v1.2). Визначення "activated" (≥1 CV-сесія за 7 днів після реєстрації). F1–F8 funnel map з entry/completion events і max drop-off targets (§25.2). PostHog event taxonomy: 21 event + "must-never-appear" таблиця (weight/load, mood score, CV keypoints, free-text — заборонені). Шість ACT-SLO: D7 activation ≥40%, F1 onboarding ≥70%, F3→F4 ≥80%, D30 retention ≥35%, F8 paywall ≥15%, PostHog lag P95 <60s [ESTIMATE] (§25.4). Шість alerting rules AL-ACT-01..06 (P0–P2) з runbook pointers (§25.5). A/B test instrumentation з clinical-safety veto на health-data segmentation (§25.6). Сім privacy constraints: no health metrics, no free-text, no CV pose data, SHA-256 distinct_id only, PostHog DPA, DSAR via R-14, Session Recording disabled на health-adjacent screens (§25.7). 14-item implementation checklist P0/P1 M4/M5 (§25.8). OQ-31 (D7 baseline — P1, growth-lead), OQ-32 (PostHog vs Amplitude — P2, data-engineer), OQ-33 (clinical-safety ruling on mood/readiness score — **P0**, clinical-safety, must resolve before M4) (§25.9).

### Changed
- `VERSION` → 1.44.0

---

## [1.43.0] — 2026-05-31

### Added
- `docs/COST_MODEL.md §24` — Series A Fundraising Economics & Dilution Modeling (231 рядків, doc bumped v1.4 → v1.5). Pre-seed cap table baseline [ESTIMATE]; Series A readiness criteria розширені від §22.6.1 (ARR $500k, NRR ≥110%, GM ≥70%, ≥3 пілоти, ≥12 міс runway post-close) (§24.3). Capital requirements Bear/Base/Bull ($2–4M raise) з use-of-funds table по 6 категоріях та CEE market rate assumptions (§24.4). Three-method pre-money valuation: ARR multiple 3–8× ($240k–$4.2M implied), 4-row comparable transactions, 5-factor score method (§24.5). Dilution table: Bear ($1.5M at $6M pre), Base ($2.5M at $10M pre), Bull ($4M at $16M pre) + ESOP refresh impact (~63% founder post-refresh) (§24.6). Cap table evolution seed→SeriesA→SeriesB (§24.7). Bridge financing: SAFE з MFN, $200–500k, trigger criteria table + Ukrainian SAFE law caveat (§24.8). П'ять DEC-030 fundraising audit events: fundraise.term_sheet_received (HIGH), fundraise.round_closed (CRITICAL), fundraise.bridge_instrument_signed (HIGH), fundraise.esop_pool_expanded (HIGH), fundraise.cap_table_updated (CRITICAL), 7yr retention (§24.10). OQ-31 (CEE vs US multiple compression — P1, founder), OQ-32 (ESOP phantom equity Ukrainian law — P1, founder+counsel), OQ-33 (SAFE not valid under UA law, HoldCo required — **P0**, founder+counsel) (§24.12).

### Changed
- `VERSION` → 1.43.0

---

## [1.42.1] — 2026-05-31

### Added
- `docs/COST_MODEL.md` — §24 ToC entry (Series A Fundraising Economics)
- `docs/OBSERVABILITY.md` — §25 ToC entry (Product & Activation Funnel) + ToC table added to Purpose section

---

## [1.42.0] — 2026-05-31

### Added
- `docs/COST_MODEL.md §23` — Enterprise NRR Engine & Expansion Revenue Model (+420 рядків, doc bumped v1.0 → v1.4). Formulaic monthly ARR bridge з шістьма рядками (Opening ARR + New Logo + Seat Expansion + Tier Upgrade + Add-on − Seat Contraction − Logo Churn = Closing ARR) та sample Month 12 Base computation (§23.1). Tier migration economics: Starter→Growth +$7,200 ACV (+50%), Growth→Enterprise +$30,000 ACV (+56%); multi-year discount pricing grid (1yr/2yr/3yr) по всіх тірах; CSM migration trigger signals (seat utilisation ≥ 90%, activation ≥ 70%) (§23.2). Add-on revenue taxonomy: 5 SKUs (white-label $500–2,000/mo ~90% GM, premium SLA $300–800 ~85%, custom integration $400–1,500 ~70%, advanced analytics $200–600 ~88%, extra admin seats $50–100/seat ~95%); Year 2 attach rate targets (§23.3). Three-year cohort NRR table: Starter 100%/105%/112%, Growth 105%/112%/120%, Enterprise 108%/118%/127%; blended NRR при §19.7 Base mix (4:3:1 logo ratio) → ~114% Year 2 (§23.4). Blended NRR formula, mix-shift effect documentation, Enterprise-share sensitivity table (§23.5). Annual price indexation clause: CPI-U+3% cap, 0% floor, 10% ceiling, 60-day notice; compounding scenarios по 4 CPI regime; EUR contract HICP note (§23.6). Шість DEC-030 HMAC-chained expansion audit events: enterprise.seat_expanded (HIGH, 7yr), enterprise.seat_contracted (HIGH, 7yr), enterprise.tier_upgraded (CRITICAL, 7yr), enterprise.tier_downgraded (CRITICAL, 7yr), enterprise.addon_purchased (HIGH, 7yr), enterprise.contract_renewed (STANDARD, 7yr) з required metadata JSONB fields та per-tenant HMAC chain input spec (§23.7). NRR-to-ARR multiple table: 90%→4–6×, 100%→6–8×, 110%→8–12×, 120%→12–16×, 130%→16–22×; implied valuations при §19.7 Bear/Base/Bull ARR (§23.8). Implementation checklist: 12 items P0/P1 M4/M5 (ARR bridge job, event instrumentation, tier NRR tracking, quarterly review cadence) (§23.9). OQ-28 (actual seat expansion rate — P0, data-engineer, Day 90 first enterprise tenant), OQ-29 (add-on attach rate — P1, customer-success, Month 36), OQ-30 (FX EUR/USD impact — P1, founder + counsel, before first EU pilot conversion) (§23.10).

### Changed
- `VERSION` → 1.42.0

---

## [1.41.0] — 2026-05-31

### Added
- `content/post-167-hpa-axis-cortisol.md` — Вісь HPA і кортизол у тренінгу — адаптивний сигнал, а не ворог (~13 хв, clinical-safety NOT REQUIRED). Selye (1936/1950) GAS як концептуальна основа. HPA-каскад: гіпоталамус → CRH → передній гіпофіз → ACTH → zona fasciculata → кортизол; GR-рецептори у практично кожній тканині. Viru & Viru (2004): кортизол як «permissive hormone» — ліполіз, глюконеогенез, субстратна мобілізація. Tremblay et al. (2005 Sports Med): значуща кортизольна відповідь при ≥60–70% VO₂max; пік через 30–60 хв після тренування. Häkkinen & Pakarinen (1993 Int J Sports Med): 10×10 squat → більший кортизольний сплеск, ніж важка малоповторна робота при тому ж тоннажі. FOXO1/3 → MuRF1/MAFbx (атрогени): катаболічний шлях вимагає хронічного, а не гострого підвищення кортизолу. Fry et al. (1998 JSCR): C:T ratio як маркер non-functional overreaching; LH/GnRH-пригнічення при хронічній HPA-активації. Cortisol Awakening Response (CAR): пік 6–8 AM. Leproult et al. (1997 Sleep): 24-год депривація → +21–37% денний/вечірній кортизол через порушення SWS-придушення HPA. 5 практичних висновків. sports_scientist_reviewed: true.

### Changed
- `blog.html` — додано картки Post 165 (BFR-тренінг), Post 166 (Розвантажувальний тиждень), Post 167 (Вісь HPA і кортизол); лічильник 164 → 167 карток; v1.39.0 → v1.41.0
- `README.md` — post-165 [x] BFR; post-166 [x] deload; post-167 [x] HPA axis; post-168 переключено на overtraining recovery (⚠️ CS REQUIRED); додано нові теми post-173 (осмотична адаптація), post-174 (нейром'язова адаптація перший місяць), post-175 (RSI), post-176 (Fitness-Fatigue model)
- `STATUS.md` — blog.html рядок: 165 → 167 карток, v1.39.0 → v1.41.0; додано рядки post-165/166/167 у content engine таблицю
- `VERSION` → 1.41.0

---

## [1.40.0] — 2026-05-31

### Added
- `docs/OBSERVABILITY.md §24` — Subscription & Revenue Event Observability (711 рядків). Закриває SOC 2 PI1.1–PI1.5 Processing Integrity gap. Consumer (RevenueCat) і enterprise (Stripe) state machine diagrams з таблицями valid/invalid transitions. 13-подійна Revenue Event Taxonomy. 5 SLOs (SUB-SLO-01..05): webhook lag P95 < 30s, error rate < 0.5%, state consistency 100%, enterprise seat accuracy 100% within 5 min, grace period enforcement < 72h overshoot. 8 alerting rules AL-SUB-01..08 (P0–P3). Worker `webhook-receiver.ts` spec з HMAC-SHA256 + RevenueCat Bearer signature verification. `subscription_events` DDL (13 cols, 4 indexes, 3 RLS policies, append-only, 7yr retention). pg_cron `seat-reconciliation` (5 хв) + `billing-consistency-check` (03:00 UTC daily) зі SQL. 11 DEC-030 HMAC-chained billing events (STANDARD/HIGH/CRITICAL, 7yr). Metabase "Subscription Health" dashboard spec (8 панелей). PI1.1–PI1.5 + CC7.2 + CC6.1 evidence mapping. 7 GDPR privacy constraints (no raw payload storage, SHA-256 hash only, DPAs). 14-task implementation checklist (M4/M5). Document header v1.0 → v1.1.
- `content/post-165-blood-flow-restriction-training.md` — BFR-тренінг: механізми та протоколи (~14 хв, clinical-safety reviewed ✓). Механізм 1: метаболічний стрес (mTORC1 via AMPK/CaMKII/p70S6K, ROS hormesis via NF-κB, lactate-GPR81/HCAR1 signaling). Механізм 2: рекрутування волокон (Henneman's Size Principle → override при передчасній Type I втомі, EMG-дані Takarada). Механізм 3: гормональна відповідь (Group III/IV chemoceptors → GH pulse, MGF vs. systemic IGF-1). Механізм 4: клітинний набряк (Piezo1/TRPC, integrin-FAK, polyamine synthesis). LOP finger-palpation approximation. Протокол 1×30+3×15, 30s rest, 20–30% 1RM. Абсолютні та відносні протипоказання (DVT, PAD, вагітність, III–IV varicose veins). Meta-analysis Lixandrão 2018: BFR ≈ traditional RT для гіпертрофії. clinical_safety_reviewed: true.
- `content/post-166-deload-strategies.md` — Розвантажувальний тиждень: фізіологія і стратегії (~12 хв, clinical-safety NOT REQUIRED). Периферична (глікоген, крос-брідж кінетика, Ca²⁺ regulation) vs центральна (кортикальний драйв, TMS-дані Taylor 2006, RPE drift) втома. Supercompensation timing windows: гіпертрофія 4–7 днів, сила 7–14 днів, нейральна до 21 дня. HRV/RHR/bar-speed/RPE маркери для triggering deload. 4 стратегії: volume deload (–40–60% vol, збережена інтенсивність), intensity deload (50–60% 1RM, збережений обсяг), frequency deload (менше сесій), active recovery. Reactive vs proactive deload. Bosquet et al. (2007): tapering → +2.4–3% performance. Спростування страху "втратити форму" — без significant muscle loss за 7–10 днів. Рішення-матриця: 4 trigger criteria.

### Changed
- `VERSION` → 1.40.0

---

## [1.39.1] — 2026-05-31

### Changed
- `docs/SOC2_READINESS.md §51` — Consolidated Gap Register + Observation Period Readiness Dashboard (+344 рядків). Supersedes stale v1.3 Gap Analysis Summary (~71%) with current state (~97%). Fulfils §49 checklist item 10 (CC6-GAP-001/002/003 §26 status → 🟡 Authored) and §50 checklist item 9 (CC5-GAP-003/005 §27 status → 🟡 Authored) via consolidated register. Master gap register: 32 TSC sub-criteria across all 5 Trust Service Criteria (Security/Availability/Processing Integrity/Confidentiality/Privacy); 14/32 🟢, 18/32 🟡, 0/32 🔴. 13-item pre-observation-period gate checklist (G-01 through G-13) with critical path: ~25–35 days from M4 sprint start to observation clock open. 3 remaining P0 gaps (CC7-GAP-005/006/007) all 🟡 Authored — implementation-pending only. 9 automated evidence streams documented (~8,640+ DEC-030 events in 90-day window). 8-item manual evidence calendar with responsible-party assignment. §51.8 privacy floor reminder: RLS-enforced prohibition on auditor queries against individual health data or CV keypoints. 16-item implementation checklist (M4 deployment sprint → audit close). Document header v2.1 → v2.3. SOC 2 readiness: ~96.5% → ~97%.
- `VERSION` → 1.39.1

---

## [1.39.0] — 2026-05-31

### Added
- `content/post-164-altitude-hypoxia-training.md` — Адаптація до висоти: гіпоксія як тренувальний стимул (~13 хв). HIF-1α молекулярний механізм (PHD-деградація, VHL-убіквітинування, downstream генна регуляція). EPO-кінетика і еритропоез: Hbmass +3.3–4.5% за 28 днів (Gore et al. 2013 BJSM meta-analysis). Levine & Stray-Gundersen (1997 JAP): LHTL vs LHTH — чому LHTL дає +5% VO₂max проти +2% у LHTH. Altitude tents: поріг ≥12 год/добу, hypobaric vs normobaric при ≤3000 м незначна різниця. Мінімальний поріг ≥2000 м/≥21 день. Адаптації поза еритропоезом: VEGF/капіляризація, міоглобін, 2,3-DPG. AMS базова механіка. Три рівні практичного застосування. Статус: clinical_safety NOT REQUIRED.

### Changed
- `blog.html` — додано картки Post 162 (сателітні клітини), Post 163 (HSP), Post 164 (висота/гіпоксія); лічильник 161 → 165 карток
- `STATUS.md` — додано рядки post-158 через post-164 у таблиці content engine; blog.html → v1.39.0, 165 карток
- `README.md` — roadmap: post-162/163 помічено [x] з актуальними темами; додано нові теми post-169 (PCr ресинтез), post-170 (ноцицепція), post-171 (сон і результати), post-172 (сила хвату)
- `VERSION` → 1.39.0

---

## [1.38.1] — 2026-05-31

### Changed
- `content/post-162-satellite-cells-muscle-hypertrophy.md` — editorial revision: більш точне формулювання відкриття Мауро, уточнена роль субламінарної ніші як активного регулятора, переформатований каскад активації PAX7→MyoD/Myf5 з детальнішим описом MGF-механізму.
- `VERSION` → 1.38.1

---

## [1.38.0] — 2026-05-31

### Added
- `content/post-162-satellite-cells-muscle-hypertrophy.md` — Сателітні клітини: м'язові стовбурові клітини і гіпертрофія на клітинному рівні (~14 хв). Охоплює: відкриття Мауро (1961), Pax7→MyoD/Myf5 каскад, IGF-1 Ec/MGF, теорія міоядерного домену, полеміка Egner et al. (2016) Science (чи обов'язкові сателітні клітини для гіпертрофії?), м'язова пам'ять і перманентне додавання міоядер (лабораторія Гундерсена), тренувальні змінні для максимальної активації сателітних клітин, парадокс само-відновлення пулу. Статус: clinical_safety NOT REQUIRED.
- `content/post-163-heat-shock-proteins-training.md` — Білки теплового шоку (HSPs): молекулярні шаперони і відповідь на тренувальний стрес (~13 хв). Охоплює: HSP-родини (HSP70/HSPA1A vs HSP73/HSPA8, HSP90, HSP27/HSPB1), HSF1 як майстер-транскрипційний фактор (Hsp90-комплекс → протеотоксичний стрес → тримеризація → HSE), тренінг як протеотоксичний стресор (температура + ROS + механічна деформація), гормезис (Kregel 2002, Morton 2009), HSP70 і mTOR, крос-толерантність (Moseley 1997), HSP27 і захист цитоскелету (десмін/тітін при ексцентричних навантаженнях), анти-апоптотична функція, практичні наслідки (сауна post-training, полеміка CWI). Статус: clinical_safety NOT REQUIRED.

### Changed
- `VERSION` → 1.38.0

---

## [1.37.1] — 2026-05-31

### Added
- `docs/SOC2_READINESS.md §50` — TruffleHog CI Secret Scan + Terraform IaC Drift Detection — CC5-GAP-003 / CC5-GAP-005 Auditor Exhibit (+632 рядків). Закриває обидва залишкових P1 documentation gaps у CC5: CC5-GAP-003 (`trufflehog-scan` як job 6 у `ci.yml` — `trufflesecurity/trufflehog@main` з `--only-verified` + `--fail`; live API verification усуває false positives від ротованих секретів; DEC-030 `security.trufflehog_verified_secret_found` CRITICAL/7yr → PagerDuty P0 через §46 pipeline; rotation SLA 1h; incident procedure: rotation → git filter-repo → force-push 2-person approval) + CC5-GAP-005 (новий workflow `.github/workflows/infra-drift.yml` — daily cron `0 6 * * *` + PR path filter `infra/cloudflare/**` + `workflow_dispatch`; `terraform plan -detailed-exitcode` з R2 backend `form-tf-state`; кожен run архівується до `form-audit-logs/infra-drift/` незалежно від результату; drift → Slack Block Kit alert `#ops-alerts` + DEC-030 `infra.terraform_drift_detected` HIGH/7yr + job exit 1; clean → DEC-030 `infra.terraform_drift_check_clean` LOW/3yr — auditor-queryable continuous log ~90 rows/90d). 3 нових DEC-030 події: `security.trufflehog_verified_secret_found` (CRITICAL, 7yr), `infra.terraform_drift_detected` (HIGH, 7yr), `infra.terraform_drift_check_clean` (LOW, 3yr). 3 evidence artefacts PRE-50-E-001–003. 12 checklist items. CC5-GAP-003/005 🔴→🟡 Authored. P1 documentation gaps: 2→0. CC5.1/CC5.2 🟡 Partial→🟡 Authored. SOC2: ~96%→~96.5%.

### Changed
- `VERSION` → 1.37.1

---

## [1.37.0] — 2026-05-31

### Added
- `content/post-161-microbiome-training.md` — «Мікробіом і тренінг — двонаправлений зв'язок між кишківником і м'язом». Sports-scientist reviewed. Clarke et al. (2014 Gut): 22 vs 14 філуми у елітних регбістів; Allen et al. (2018 Med Sci Sports Exerc): 6-тижнева аеробна інтервенція з зворотними змінами підтверджує причинність тренінгу; Scheiman et al. (2019 Nature Medicine): Veillonella atypica → лактат→пропіонат ланцюг → +13% витривалість у гнотобіотичних мишей; SCFA-механізм (бутират/HDAC/NF-κB, пропіонат/GPR41/43/GLP-1, ацетат/ацетил-КоА); Akkermansia muciniphila і Amuc_1100 через TLR2 для захисту tight junctions; ентерохромафінний серотонін і вісь кишківник-мозок. VETO-зона: нічого про дієту. clinical-safety NOT REQUIRED. 10 citations.
- `blog.html` — картки Post 160 (Силова-швидкісна крива) і Post 161 (Мікробіом і тренінг). Лічильник: 159 → 161 карток.
- `README.md` — оновлено роадмап: post-160 і post-161 позначено [x]; зсув нумерації post-162/163/164 (Колаген, RBE, Висота); додано нові теми post-165–168 (осмотична адаптація, нейром'язова адаптація у перший місяць, відновлення після overtrain, HPA-вісь і кортизол).
- `STATUS.md` — blog.html рядок оновлено: 159 → 161 карток, v1.37.0.

### Changed
- `VERSION` → 1.37.0

---

## [1.36.0] — 2026-05-31

### Added
- `docs/SOC2_READINESS.md §49` — GitHub Organisation 2FA Enforcement + Cloudflare Access MFA + Supabase Tenant-Admin TOTP Gate — CC6-GAP-001/CC6-GAP-002/CC6-GAP-003 Auditor Exhibit (+1,105 рядків). GitHub org `require_two_factor_authentication` via REST API + GraphQL (`PATCH /orgs/form-coach`), pre-flight member audit query, fine-grained PAT policy (classic PATs заборонені, 90-day expiry, OIDC federation для CI). Terraform HCL `infra/cloudflare/access.tf`: три `cloudflare_access_application` + три `cloudflare_access_policy` з `require { mfa = [{type = "totp"}] }` (4h/8h session, auto-redirect). `supabase/config.toml` MFA block, `auth.jwt_aal()` Postgres helper, `RESTRICTIVE` RLS policy на 5 таблицях. `require-mfa` Edge Function TypeScript (~130 рядків): AAL2 JWT перевірка, DEC-030 emit, 403 з `AUTH_AAL2_REQUIRED`, `mfa_enforcement_log` DDL (RESTRICTIVE insert policy). 7 нових DEC-030 подій (HIGH/7yr для blocked/failed). 5 evidence artefacts PRE-49-E-001–005. 12 checklist items. CC6-GAP-001/002/003 🔴→🟡 Authored. P0: 6→3. SOC2: ~95.5%→~96%.

### Changed
- `VERSION` → 1.36.0

---

## [1.35.0] — 2026-05-31

### Added
- `content/post-160-force-velocity-curve.md` — «Силова-швидкісна крива: фізіологія компромісу між силою і швидкістю». Sports-scientist reviewed. Модель Хілла (1938), F-V гіпербола, пік потужності (~30–70% 1RM залежно від рівня тренованості per Cormie et al. 2011), фізіологічна основа (cross-bridge kinetics, склад волокон, Ca²⁺ release, pennation angle), силово-швидкісний спектр (maximal strength → ballistic), RFD та перші 50–200 мс, PAP/contrast training, обмеження моделі. Clinical-safety not required (no body/food/mental/injury content). 10 citations.

### Changed
- `VERSION` → 1.35.0

---

## [1.34.2] — 2026-05-31

### Added
- `docs/SOC2_READINESS.md §48` — Cloudflare WAF Rate-Limit Rules: Terraform Configuration + Logpush Routing — CC7-GAP-007 Auditor Exhibit. Two `cloudflare_ruleset` resources (phase `http_ratelimit`) covering all 5 WAF rules: FORM-AUTH-RATELIMIT-001 (5 POST/60s per IP → block 10 min), FORM-AUTH-RATELIMIT-002 (10 POST/300s per IP+email → block 30 min; Cloudflare Business+ body inspection gated via `cloudflare_plan` variable), FORM-API-RATELIMIT-001/002/003 (global/per-JWT/workout spike limits). `cloudflare_logpush_job` → R2 `form-audit-logs/waf-events/` (NDJSON, high frequency, filtered to FORM-*RATELIMIT-* block/jschallenge events). `cloudflare_notification_policy` + `cloudflare_notification_webhook` → form-alert-relay. `waf-handler.ts` extension: HMAC signature verification, IP hashing, severity mapping (AUTH-001/002 → P1; API-001/002/003 → P2; burst escalation at block_count_5m > 100), Slack Block Kit builder, PagerDuty dispatch. 4 нові DEC-030 події: `security.waf_rule_blocked` (MEDIUM/3yr), `security.waf_alert_fired` (HIGH/7yr), `security.waf_rule_updated` (HIGH/7yr — GitHub Actions Terraform diff), `security.waf_logpush_gap` (HIGH/7yr). 4 evidence artefacts PRE-48-E-001–004. 12 checklist items. CC7-GAP-007 🔴→🟡 Authored. P0: 7→6. SOC2: ~95%→~95.5%.

### Changed
- `VERSION` → 1.34.2

---

## [1.34.1] — 2026-05-31

### Added
- `content/post-159-neurogenesis-aerobic-training.md` — «Нейрогенез і аеробний тренінг — як кросовки буквально будують мозок». Van Praag et al. (1999 Nature Neuroscience): подвоєння нейрогенезу при беговому навантаженні. Erickson et al. (2011 PNAS): +2% гіпокамп за 1 рік у 55–80-річних. Молекулярний механізм: BDNF/TrkB (Cotman & Berchtold 2002), IGF-1 транспорт через ГЕБ, VEGF-ангіогенез як передумова нейрогенезу. Антрогенні перешкоди: кортизол, запалення, алкоголь. Мінімальний аеробний рецепт за наявними RCT. 10 рецензованих джерел. clinical-safety NOT REQUIRED.
- `blog.html` — додано картки для post-158 (Мітофагія) і post-159 (Нейрогенез). 157 → 159 карток.
- `README.md` — roadmap: post-158 і post-159 позначено [x]; додано post-160–163 як нові теми (Мікробіом, Колаген/ECM, RBE, Висота).
- `STATUS.md` — blog.html рядок оновлено: 157 → 159 карток, v1.34.1.

---

## [1.34.0] — 2026-05-31

### Added
- `docs/SOC2_READINESS.md §47` — Sentry Alert Rules Configuration + `row-count-monitor` Edge Function — CC7-GAP-005 + CC7-GAP-006 Auditor Exhibit. Complete Sentry `sentry.init()` config, `beforeSend` health data scrubber (11 Art. 9 fields, recursive `scrubObject`), four alert rules (FORM-FATAL-001/SPIKE-001/AUTH-ERR-001/HEALTH-LEAK-001) з повним JSON + REST API specs. `row-count-monitor` Supabase Edge Function + `monitoring_baselines` DDL + seeding procedure. 4 нові DEC-030 події. 11 P0 checklist items. +1480 рядків, v1.9. CC7-GAP-005 🔴→🟡, CC7-GAP-006 🔴→🟡. P0: 9→7. SOC2: ~94%→~95%.
- `docs/INCIDENT_RESPONSE.md §R-15` — Health Data Leak Response runbook (GDPR Art. 9). Triggered by FORM-HEALTH-LEAK-001. Covers investigation flow, GDPR Art. 33 assessment decision tree, code fix requirements. Fixes runbook number conflict (R-06 was already DDoS/WAF).

### Fixes
- `docs/SOC2_READINESS.md §47` — виправлено конфлікт runbook номерів: R-06 (Health Data Leak) → R-15, R-07 (row-count) → R-01 (Data Breach path)

---

## [1.33.0] — 2026-05-31

### Added
- `content/post-158-mitophagy-mitochondrial-quality-control.md` — «Мітофагія — клітинне прибирання, яке робить тебе сильнішим». Механізм PINK1/Parkin (ΔΨm-сенсор → убіквітинування OMM → LC3-адаптери p62/NDP52/OPTN → лізосомальна деградація). Антиоксидантне притуплення (Ristow 2009, Paulsen 2014). Відмінності між аеробним та силовим тренінгом за мітофагічним стимулом. Zone 2 як технічне обслуговування клітинної інфраструктури. 178 рядків, 10 рецензованих джерел. clinical_safety_reviewed + sports_scientist_reviewed.

---

## [1.32.1] — 2026-05-31

### Added
- `docs/SOC2_READINESS.md §46` — Security Alert Pipeline: `form-alert-relay` Cloudflare Worker · `auth-monitor` Supabase Edge Function · `audit-chain-daily-check` Supabase Edge Function — CC7.1/CC7.2/CC7.3 Auditor Exhibit. Complete TypeScript implementations for all three P0 CC7 components specified in §25.10. Closes CC7-GAP-001, CC7-GAP-002, CC7-GAP-003 to 🟡 AUTHORED (🟢 on deploy + test). P0 count: 12 → 9. SOC 2 readiness: ~93% → ~94%. +985 рядків, v1.8.

### Closes
- CC7-GAP-001 🔴 Open → 🟡 AUTHORED (`form-alert-relay` Worker implementation spec filed)
- CC7-GAP-002 🔴 Open → 🟡 AUTHORED (`auth-monitor` Edge Function implementation spec filed)
- CC7-GAP-003 🔴 Open → 🟡 AUTHORED (`audit-chain-daily-check` Edge Function + cron spec filed)

### Changed
- `VERSION` → 1.32.1

---

## [1.32.0] — 2026-05-30

### Added
- `content/post-157-myokines-muscle-endocrine-organ.md` — "Міокіни: скелетний м'яз як ендокринний орган — механістичний розбір." 364 рядки, 18 хв читання. 14 H2-розділів: upstream-сенсори (інтегрини, титін, AMPK, PGC-1α, CaMKII); IL-6 (Steensberg 2000 J Physiol, JAK-STAT3 vs транс-сигналізація, метаболічний AMPK-ефект); BDNF (Wrann 2013 Cell Metab PGC-1α→FNDC5→BDNF, Szuhany 2015 мета-аналіз, чітке обмеження людських даних); іризин/FNDC5 (Boström 2012 Nature — феномен; Jedrychowski 2015 — маса-спектрометричне підтвердження у людини; Kim 2018 αV/β5-інтегрин; чесна оцінка WAT browning у людей); FGF21 (ATF4-залежна м'язова секреція, переважно паракринна роль); METRNL (еозинофіл-IL4-STAT6-UCP1 шлях, нейропротекція); міостатин/фолістатин (ActRIIB→Smad2/3, сателітні клітини); остеокальцин (GPRC6A в м'язі, Mera 2016 — замкнена кістка-м'яз петля); гостра vs хронічна відповідь; силовий vs аеробний профіль міокінів; доказова матриця (міцно/попередньо/спірно); практичні наслідки. 17 цитат. `clinical_safety_note: NOT REQUIRED`.
- `blog.html` — картка post-157 додана; всього 157 карток.
- `README.md` — post-157 [x] з повним описом; post-158 (Нейрогенез і аеробний тренінг) перенесений у roadmap як наступний.
- `STATUS.md` — post-157 у таблиці контенту; blog.html рядок оновлено до 157 карток.

### Changed
- `VERSION` → 1.32.0

---

## [1.31.0] — 2026-05-30

### Added
- `.github/workflows/ci.yml` — 5-job CI pipeline: `typecheck` (tsc --noEmit), `lint` (ESLint --max-warnings 0), `test` (npm test), `audit` (npm audit --audit-level=critical з auditable exception bypass RISK-XXXXXX), `secret-scan` (git-secrets full history + working tree). Матеріалізація §45.2 з `docs/SOC2_READINESS.md`. SOC 2 controls: CC8.1, CC6.8, CC7.1, CC6.6.
- `.github/workflows/deploy-workers.yml` — Wrangler deploy gated on CI `conclusion == 'success'`; deployable Workers: `security-portal` + `api-gateway` (skip if no wrangler.toml); structured CC8-E-002 deploy log → Cloudflare R2 `form-audit-logs/deploy-logs/`; three DEC-030 HMAC-chained events: `ci.deploy_succeeded` (MEDIUM), `ci.deploy_failed` (HIGH), `ci.build_blocked` (MEDIUM). Матеріалізація §45.3.
- `.github/dependabot.yml` — npm (weekly Monday 08:00 Kyiv, lockfile-only, grouped security PRs, `form-compliance` reviewer) + GitHub Actions (weekly, SHA-digest pinning). Матеріалізація §45.4.
- `.github/git-secrets-patterns` — шість канонічних credential-патернів: Anthropic `sk-ant-`, Supabase JWT, Stripe `sk_live_`, ElevenLabs `xi-api-key`, Cloudflare Bearer, hex HMAC. Per §43.6.2.

### Closes
- CC8-GAP-001 🟡 AUTHORED → 🟡 FILED (`.github/workflows/ci.yml` на `main`; закривається до 🟢 при першому green CI run)
- CC8-GAP-002 🟡 AUTHORED → pending (branch protection rules потребують ручного налаштування у GitHub Settings > Branches)

### Changed
- `VERSION` → 1.31.0

---

## [1.30.0] — 2026-05-30

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md §21` — Google Workspace Directory API Integration for Group Sync. Closes G-005 from §9 (designed but not implemented). Root cause: Google OIDC deliberately omits group claims from the ID token; Admin SDK Directory API + service account + domain-wide delegation (DWD) is the canonical solution. Pull-model design complementary to §19's SCIM push-model — Google Workspace tenants who cannot use the three §19 workarounds (Okta bridge, Entra bridge, SAML groups attribute) use this path. Architecture: OIDC callback Worker checks `google_directory_group_cache` (5-minute TTL, 10-minute grace window on API failure); CACHE MISS calls `GET admin.googleapis.com/admin/directory/v1/groups?userKey={email}` (paginated 200/page); resolved Google group IDs feed into existing `scim_group_role_mappings` table (§19.3) — no change to role resolution engine (§5.4). Schema: ALTER TABLE `tenant_sso_configs` — five new columns (`google_service_account_email`, `google_directory_admin_email`, `google_directory_sync_enabled`, `google_directory_last_sync_at`, `google_directory_sync_error`); `google_service_account_json BYTEA` already exists from §4.2; partial index on enabled Google tenants. New `google_directory_group_cache` table (UNIQUE on `tenant_id, user_email`; `expires_at` as generated column `fetched_at + 5 min`; RLS tenant isolation + `form_system` write + `form_api` read-own-tenant; pg_cron cleanup every 10 minutes). Full TypeScript implementation (`apps/api-gateway/src/sso/google-directory-groups.ts`): `resolveGoogleGroups()` cache-first with grace-window fallback; `fetchAndCacheGroups()` pagination loop; `getOrMintAccessToken()` KV caching (TTL 55 min per OQ-SSO-21.3); `mintServiceAccountToken()` using SubtleCrypto RSASSA-PKCS1-v1_5 + JWT bearer grant (RFC 7523); `importRsaPrivateKey()` PKCS8 import. Customer setup: 4-step process (GCP project + service account; Admin Console DWD with `admin.directory.group.readonly` scope only; impersonation account designation; CSM-mediated credential upload via secure portal). Credential rotation: 7-step protocol; immediate key deletion on compromise. Eight DEC-030 HMAC-chained audit events: `sso.google_directory_sync_success` (STANDARD, 7yr), `sso.google_directory_sync_cache_hit` (STANDARD, 7yr), `sso.google_directory_sync_error` (HIGH, 7yr), `sso.google_directory_credential_uploaded` (HIGH, 7yr), `sso.google_directory_credential_rotated` (HIGH, 7yr), `sso.google_directory_sync_enabled` (HIGH, 7yr), `sso.google_directory_sync_disabled` (HIGH, 7yr), `sso.google_directory_sync_validated` (STANDARD, 7yr); `user_email_hash` SHA-256 not plaintext in all events. Five alerting rules AL-SSO-GDIR-01 through AL-SSO-GDIR-05 (P2: error rate > 20% / 3 consecutive per tenant; P3: P95 > 3,000 ms / unvalidated credential). GDPR: Google Workspace DPA covers Directory API; conditional sub-processor entry in SOC2_READINESS §44 (only for `google_directory_sync_enabled` tenants); data minimised to `{id, email, name}` only; 5-minute TTL; Art. 14 notice obligation documented for controller. SOC 2 CC6.1 (credential lifecycle), CC6.2 (actor_id + role on credential ops), CC6.3 (5-min TTL = timely role revocation evidence), CC7.2 (5 alerting rules), CC9.2 (Google DPA + conditional sub-processor entry), C1.1 (email hash + group data not exposed to tenant_manager). Four evidence artefacts CC6-E-GDIR-001 through CC6-E-GDIR-004. Open questions OQ-SSO-21.1 through OQ-SSO-21.4 (background sync M6; >10k group load test; KV token caching — implemented; Cloud Identity Groups deferred). Gap G-005: 🔴 Open → 🟡 Authored. 12-item implementation checklist (8× P0 M4, 4× P1 M4).

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` header — v1.2 → v1.3; Table of Contents updated to include §21.
- `VERSION` → 1.30.0

---

## [1.29.0] — 2026-05-30

### Added
- `content/post-156-glymphatic-system-sleep-training.md` — "Глімфатична система і сон: як тренінг і сон разом очищують мозок." Sports-science series post. Covers Iliff/Nedergaard (2012 Science Translational Medicine) and Xie et al. (2013 Science) discovery of the glymphatic system and AQP4-mediated perivaqueductal drainage; CSF-ISF exchange mechanism; glymphatic activity ~60% higher during sleep vs wakefulness (expanded interstitial space); Holth et al. (2019 Science) one night sleep deprivation → +25–30% β-amyloid in CSF; Lundgaard et al. (2017 Nature Communications) exercise ↑ AQP4 polarization and glymphatic flow; synaptic homeostasis hypothesis (Tononi/Cirelli) and motor consolidation during sleep (Walker 2017; Stickgold & Walker 2007 Nat Rev Neurosci); Lee et al. (2015 J Neurosci) lateral sleep position; Kredlow et al. (2015) meta-analysis exercise → sleep quality; training→BDNF→sleep→neuroplasticity triangle; honest note on mouse-model limitations for human glymphatic quantification. `clinical_safety_note: NOT REQUIRED`.
- `blog.html` — cards for post-155 (Ефект інтерференції AMPK/mTORC1) and post-156 (Глімфатична система і сон) added; total 156 cards.
- `STATUS.md` — content engine table backfilled for post-143 through post-156; blog.html row updated to 156 cards.
- `README.md` — roadmap: post-155 marked [x] with actual AMPK/mTORC1 content (ECM planned as post-159); post-156 marked [x] with full description.

### Changed
- `VERSION` → 1.29.0

---

## [1.28.0] — 2026-05-30

### Added
- `docs/DATA_MODEL.md §24` — Subscription, Billing & Revenue Schema. Closes the monetization data-layer gap. Two-track billing architecture: consumer via Apple StoreKit 2 / Google Play RTDN webhooks, enterprise via Stripe invoicing with per-seat pricing; `user_subscriptions` is the single source of truth for access control, never App Store/Stripe directly. Four ENUMs (`plan_tier_enum`, `subscription_status_enum`, `billing_channel_enum`, `subscription_event_type_enum`). Tables: `user_subscriptions` (18 columns, 6 CHECK constraints, partial unique index `uq_user_one_active_subscription`), `subscription_events` (append-only lifecycle log, idempotency index on `external_event_id`, no UPDATE/DELETE RLS for any non-admin role), `enterprise_seat_allocations` (period-based contracted seats with `effective_to IS NULL` = current), `enterprise_seat_assignments` (per-user seat with `revoked_at`, `assigned_by` audit trail), `tenant_seat_utilization` view. Webhook integration: Apple JWS 2-hop cert chain verification (6h replay window); Google Play Pub/Sub + Developer API v3 verify; Stripe HMAC-SHA256 with seat expansion path (close old allocation, insert new). 13-transition trial/grace state machine. GDPR Art. 17 vs. 7-year fiscal retention resolved via pseudonymization (Ukrainian Tax Code Art. 44, EU VAT Directive 2006/112/EC Art. 245). 12 DEC-030 HMAC-chained events (5× STANDARD, 7× HIGH, all 7yr). RLS: `form_api` own-row SELECT only; billing Worker (`form_system`) is sole INSERT/UPDATE path; `tenant_admin` scoped to own tenant for seat management. SOC 2: CC6.2 (plan tier = authz gate), CC6.3 (seat revocation), CC8.1 (single write path), CC7.2 (payment_failed SIEM signal), A1.1, P5.0/P8.0 (GDPR billing erasure). 5 open questions (OQ-BILL-01 through OQ-BILL-05). 12-item implementation checklist.
- `content/post-155-concurrent-training-interference-ampk-mtorc1.md` — "Ефект інтерференції в суміщеному тренінгу: молекулярний антагонізм AMPK/mTORC1." Sports-science series post. Covers Hickson (1980) original interference effect observation, AMPK/mTORC1 molecular antagonism (two inhibitory routes: AMPK → TSC2/Rheb suppression via Inoki et al. 2003; AMPK → direct Raptor phosphorylation via Gwinn et al. 2008), Atherton (2005) and Coffey (2006) acute signaling evidence with explicit surrogate-vs-adaptation limitations noted, Wilson (2012) and Schumann (2022) meta-analyses, session ordering effect (endurance-before-strength blunts mTOR peak more than reverse), Type II fiber preferential sensitivity, dose-dependency (minimal interference on separate days), practical framework. `clinical_safety_note: NOT REQUIRED`.

### Changed
- `docs/DATA_MODEL.md` header — v1.3 → v1.4; TOC updated to include §24.
- `VERSION` → 1.28.0

---

## [1.27.1] — 2026-05-30

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md §20` — SAML Certificate Lifecycle Management — Proactive Monitoring & Expiry Alerting. Closes the design gap for the `cert_expiry_check` cron referenced (but unspecified) in §8.1. Two certificate classes monitored: SP signing cert (FORM-generated, 2-year RSA-2048) and IdP signing cert (customer-generated, varies). Schema: ALTER TABLE `tenant_sso_configs` — four expiry-metadata columns (`saml_sp_cert_expires_at`, `saml_sp_cert_fingerprint`, `saml_idp_cert_expires_at`, `saml_idp_cert_fingerprint`), two partial expiry indexes, two state-machine columns (`cert_alert_tier` 7-value ENUM, `cert_rotation_state` 5-value ENUM); backfill Edge Function parses BYTEA PEM fields at migration time. Seven-tier alert escalation matrix (t90 INFO → t60 P3 → t30 P2 → t14 P1 → t7 P1 → t2 P0 → expired P0) with PagerDuty severity routing and 7-day de-duplication rule. Full TypeScript `cert-expiry-check.ts` Worker cron (daily 02:00 UTC): queries active SAML tenants, processes both cert classes per tenant, emits DEC-030 before PagerDuty dispatch, updates state columns; separate `handleExpired` branch for post-expiry P0 path. Three Resend email templates: CRT-01 (`sso-cert-expiry-t30`, first customer-visible notification, cert-class-conditional body for SP vs IdP), CRT-02 (`sso-cert-expiry-t7`, urgent 7-day), CRT-03 (`sso-cert-expiry-expired`, post-expiry email fallback active). Five-state rotation state machine (`idle → notified → rotation_in_progress → complete`; `overdue` branch when customer unresponsive at t7). Four new DEC-030 HMAC-chained events: `sso.cert_expiry_alert` (MEDIUM/HIGH/CRITICAL by tier, 7yr), `sso.cert_expired` (CRITICAL, 7yr), `sso.cert_uploaded` (STANDARD, 7yr), `sso.cert_monitor_error` (HIGH, 7yr). FORM internal admin dashboard Certificate Panel spec (M5). Tenant admin in-app banner at ≤30 days IdP cert expiry. SOC 2 evidence: CC6-E-CERT-001 through CC6-E-CERT-004. SOC 2 mapping: CC6.1 (credential lifecycle), CC7.2 (anomaly monitoring), CC8.1 (change management). 4 open questions (OQ-SSO-20.1 through OQ-SSO-20.4). 11-item implementation checklist (8× P0 M4, 3× P1 M5).

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` header — v1.1 → v1.2; Table of Contents updated to include §20.
- `VERSION` → 1.27.1

---

## [1.27.0] — 2026-05-30

### Added
- `content/post-154-telomeres-training-geroprotection.md` — «Теломери і тренінг — геропротекція на молекулярному рівні». Sports-science series. Covers: telomere biology and end-replication problem; hTERT/hTERC mechanism; Cherkas et al. (2008 Arch Intern Med) twin study — ≈200 bp difference between most and least active; Werner et al. (2009 Circulation) 8-week RCT — ↑ hTERT, ↓ p16INK4a in adults 50+; LaRocca et al. (2010 J Physiol) — aerobic capacity correlates with EPC telomere length; IGF-1/FOXO3 axis and pulsatile FOXO3 activation during recovery; satellite cells and muscle-specific telomere dynamics; upper-volume limit — ultraendurance without recovery reverses the effect; practical zone 150–300 min/week. 13 хв читання. clinical-safety NOT REQUIRED.
- `blog.html` — картки Post 153 (лактат) і Post 154 (теломери) — 154 cards total. Backfills missing card for post-153 from v1.26.0.
- `README.md` — post-153 [x] (лактат, actual content); post-154 [x] (теломери); post-155 (колаген/ECM) у roadmap; нові теми post-156 (глімфатична система), post-157 (нейрогенез), post-158 (мікробіом і тренінг) додані до roadmap.

### Changed
- `STATUS.md` — blog.html row: 152 → 154 cards · v1.27.0
- `VERSION` → 1.27.0

---

## [1.26.0] — 2026-05-30

### Added
- `docs/DATA_MODEL.md §23` — Exercise Library & Program Schema. Closes the core product schema gap for exercise catalog storage, program templates, user program instances, and periodization structure. Eight-table design: `exercises` (global catalog + custom; 7 ENUMs; `is_cv_supported` flag; `contraindication_tags`; 60-exercise seed catalog), `exercise_instructions` (multilingual UA/EN coaching cues), `exercise_muscle_groups` (push:pull balance analysis), `program_templates` (FORM-curated library, form_admin-only write), `user_programs` (`uq_user_one_active_program` partial unique index; `generation_context` JSONB with strict privacy schema forbidding injury/body-composition/diagnostic fields), `program_blocks` (accumulation/intensification/realization/deload), `program_sessions` (`workout_session_id` FK to actual logged session), `program_session_exercises` (full load prescription: relative/absolute/rpe_governed/amrap; substitution pattern via `substitute_for` self-FK). `sanitizeGenerationContext()` Worker function with CI grep gate. GDPR Art. 17 five-step erasure. 12 DEC-030 HMAC-chained events. RLS: `tenant_admin` zero access (DEC-002). 4 open questions (OQ-PROG-01 through OQ-PROG-04). 14-item implementation checklist.
- `content/post-153-lactate-signaling-molecule.md` — «Лактат — від «токсину» до центральної молекули адаптації». Sports-science series. Covers: Brooks' cell-to-cell lactate shuttle; MCT1/MCT4 transporter adaptation; lactate threshold as a trainable marker; lactate as a signaling molecule via GPR81 (HCAR1), histone lactylation (Zhang et al. 2019), and VEGF-mediated angiogenesis; practical implications for Zone 2 training, HIIT, and inter-set recovery. clinical-safety NOT REQUIRED.

### Changed
- `docs/DATA_MODEL.md` header — v1.2 → v1.3; Table of Contents updated to include §23.
- `VERSION` → 1.26.0

---

## [1.25.1] — 2026-05-30

### Added
- `docs/INCIDENT_RESPONSE.md §15` — GDPR Article 33/34 Data Breach Notification Operational Runbook. Closes the gap between §10 (legal framework overview) and the hands-on procedure required under the 72-hour clock. §15.2 awareness clock taxonomy (6 trigger events + partial-notification override rule). §15.3 incident-to-notification decision matrix (10 scenarios, Art. 33 / Art. 34 / exempt mapping, Art. 9 presumption rule). §15.4 multi-tenant breach scoping SQL (Query 1: affected user count with Art. 9 / biometric breakdown; Query 2: enterprise tenant list for B2B notifications; Query 3: HMAC chain integrity check in exposure window). §15.5 72-hour clock procedure (11 timestamped steps, T+0h through post-filing supplementaries). Template A33-01 (Art. 33 DPA notification, all required GDPR fields). Template A34-01 (Art. 34 data subject notification; subject-line Art. 9 omission rule; Resend delivery; Art. 34(3)(c) bounce exception). Template P2C-01 (Art. 28 processor-to-controller enterprise tenant notification, T+8h target). 10 DEC-030 HMAC-chained events: `breach.awareness_declared` (CRITICAL, 7yr), `breach.scope_assessment_completed`, `breach.art33_obligation_determined`, `breach.processor_notified_tenant`, `breach.art33_notification_filed` (CRITICAL — primary P7.1 auditor evidence), `breach.art33_supplementary_filed`, `breach.art34_notifications_sent`, `data.breach_notification_sent` (per-user, email_hash only), `breach.art34_waiver_documented`, `breach.notification_case_closed`. Evidence directory structure with SHA-256 manifest sealing. SOC 2 mapping: P7.1, P8.1, P6.1, CC2.2, CC7.3. 12-item implementation checklist (7× P0 M4, 4× P1, 1× P2 tabletop Scenario J).

### Changed
- `VERSION` → 1.25.1

---

## [1.25.0] — 2026-05-30

### Added
- `content/post-152-systemic-inflammation-anti-inflammatory-adaptation.md` — Запалення і тренінг: м'яз як протизапальний орган. Гостра vs хронічна запальна відповідь; м'язовий IL-6/IL-10/IL-1Ra петля (Pedersen & Febbraio 2008 Physiol Rev); M1→M2 макрофагний перехід (Arnold 2007 J Exp Med); антиоксидантний парадокс Ristow (2009 PNAS); вагусний тонус і нейрозапалення; дозова залежність. 13 хв читання. clinical-safety NOT REQUIRED.
- `blog.html` — картка Post 152 (152 cards total).
- `README.md` — post-149/150/151 [x] + post-152 [x] + roadmap post-153/154.

### Changed
- `STATUS.md` — blog.html row: 148 → 152 cards · v1.25.0
- `VERSION` → 1.25.0

---

## [1.24.0] — 2026-05-30

### Added
- `content/post-149-muscle-fiber-type-plasticity.md` — МHC ізоформи і пластичність волокон: Type I/IIa/IIx континуум, гібридні волокна, кальценурин/NFAT та PGC-1α як молекулярні драйвери переходу, межі конверсії у людей (47 KB, 11 секцій, clinical-safety: NOT REQUIRED).
- `content/post-150-calcium-signaling-adaptation.md` — Кальцієва сигналізація та вибір між силою і витривалістю: E-C coupling, SERCA1a/2a ізоформи, «Ca²⁺ частотний код» CaMKII vs кальценурин/NFAT, Chin 1998 Science перехресна іннервація (46 KB, 11 секцій, clinical-safety: NOT REQUIRED).
- `content/post-151-neuromuscular-junction-adaptation.md` — Нейром'язове сполучення і нейральна адаптація: агрин/MuSK/рапсин, Henneman size principle, пресинаптична і постсинаптична морфологія, BDNF/trkB, rate coding, RFD (45 KB, 11 секцій, clinical-safety: NOT REQUIRED).
- `blog.html` — 3 нові картки (post-149/150/151) у card-grid.

### Changed
- `VERSION` → 1.24.0

---

## [1.23.0] — 2026-05-30

### Added
- `docs/SOC2_READINESS.md §45` — GitHub Actions CI/CD Pipeline Implementation (CC8.1/CC6.8/CC7.1 Auditor Exhibit). Complete `.github/workflows/ci.yml` (5 gated jobs: typecheck/lint/test/audit/secret-scan), `.github/workflows/deploy-workers.yml` (gated on CI pass, Wrangler deploy, R2 log archive as CC8-E-002), `.github/dependabot.yml` (npm + Actions, weekly grouped security). Branch protection specification (10 settings, solo-founder compensating control). 3 DEC-030 events (`ci.deploy_succeeded` MEDIUM, `ci.deploy_failed` HIGH, `ci.build_blocked` MEDIUM). Advances CC8-GAP-001 and CC8-GAP-002 🟡 Open → 🟡 AUTHORED. P0 count unchanged at 2.
- `docs/SSO_SCIM_IMPLEMENTATION.md §19` — SCIM Groups Sync & Group-Based Role Mapping. 6 SCIM Group endpoints (GET list/single, POST, PUT, PATCH RFC7644 PatchOp, DELETE cascade). New `scim_group_role_mappings` table + `group_member_effective_role` view (highest-privilege-wins resolution). IdP configuration guides: Okta Push Groups, Azure AD/Entra, Google Workspace (SAML JIT fallback). 6 DEC-030 events (`scim.group_*`). SOC 2: CC6.1/CC6.2/CC6.3/CC6.6. 10-item checklist (7× P0 M4, 3× P1 M5).

### Changed
- `VERSION` → 1.23.0

---

## [1.22.1] — 2026-05-30

### Added
- `docs/SOC2_READINESS.md §44` — Sub-Processor List Worker: complete Cloudflare Workers (TypeScript) implementation for GDPR Art. 13(1)(e) transparency endpoint (`security.form.coach/sub-processors`). Covers KV payload schema for all 8 sub-processors (SP-01 Anthropic → SP-08 Google), Worker source with SHA-256 ETag + `X-List-Hash` header, conditional GET (304), JSON + HTML dual-format response, `Cache-Control: public, max-age=3600, stale-while-revalidate=86400`, Wrangler config fragment, DNS CNAME, staleness management table (6 trigger–action pairs), DEC-030 HMAC-chained audit events (`trust_center.sub_processor_list_deployed` / `_updated`), SOC 2 evidence mapping (CC9.2, CC2.3, CC9.1, P6.1), and 8-item implementation checklist. Advances CC9-GAP-007 and CC2-GAP-003 from 🔴 Open → 🟡 AUTHORED; P0 gap count 4 → 2.

### Changed
- `VERSION` → 1.22.1

---

## [1.22.0] — 2026-05-30

### Added
- `content/post-148-mitochondrial-biogenesis.md` — «Мітохондріальний біогенез — як тренінг перетворює клітину зсередині»: ~4,300 слів, 8 розділів. Holloszy (1967 JBC): 2–3× зростання мітохондріальних ензимів у щурів — перший молекулярний доказ адаптації. Три незалежних сигнальних шляхи до PGC-1α: AMPK (LKB1→Thr172 фосфорилювання, Zong et al. 2002 PNAS), CaMKII (Ca²⁺-залежний, Wu et al. 2002 Cell), SIRT1 (NAD⁺/NADH сенсор, Rodgers et al. 2005 Nature). Downstream-каскад: PGC-1α → NRF1/NRF2 → TFAM → реплікація і транскрипція мтДНК (Scarpulla 2011 Physiol Rev); ERRα і VO₂max предиктивна програма (Mootha 2004 PNAS). PGC-1α ізоформи: α1 (витривалість, мітохондріальний біогенез) vs α4 (резистентний тренінг → IGF-1↑/міостатин↓ → гіпертрофія без мітохондрій, Ruas et al. 2012 Cell) — пояснення класичного парадоксу сила vs витривалість. Мітохондріальна динаміка: fusion (MFN1/MFN2+OPA1) і fission (DRP1), Twig et al. (2008 EMBO J) — асиметричний поділ як фільтр якості. Мітофагія: PINK1/Parkin-шлях утилізації пошкоджених органел; He et al. (2012 Nature) — блокування аутофагії (BCL2 KO) скасовує тренувальну адаптацію м'яза. Zone 2 vs HIIT: Burgomaster et al. (2008 J Physiol) — еквівалентна CS-активність при 2.5× меншому тренувальному об'ємі HIIT; CaMKII/SIRT1 vs AMPK як ключова відмінність механізмів. Часова шкала адаптації: PGC-1α мРНК 0–3 год → CS-активність 2–4 тижні → плато 8–12 тижнів. 5 практичних висновків. Відкриті питання: гетероплазмія мтДНК при тренінгу, індивідуальна варіабельність PGC-1α відповіді, de novo vs fusion-driven біогенез.
- `blog.html` — картка post-148 з ключовими авторами/посиланнями
- `README.md` — додано записи post-147 і post-148 до content engine roadmap

### Changed
- `VERSION` → 1.22.0
- `STATUS.md` → blog.html count 146→148, версія v1.22.0

---

## [1.21.0] — 2026-05-30

### Added
- `content/post-147-extracellular-vesicles-muscle-crosstalk.md` — «Позаклітинні везикули — як м'яз надсилає пакети інструкцій до інших органів»: ~4,100 слів, 7 розділів. Johnstone 1987 (везикули як «клітинне сміття») → Valadi et al. 2007 Nature Cell Biology (EVs переносять функціональні мРНК/мікроРНК — парадигмальний зсув). ISEV-номенклатура: small EVs (екзосоми, 30–150 нм, MVB-походження), microvesicles (100–1000 нм, брунькування плазматичної мембрани), CD63/CD9/CD81 як маркери. Механізм секреції: Ca²⁺-спайк при скороченні → nSMase2 → кераміновий шлях (Trajkovic 2008 Science); SNARE-залежне злиття MVB (Théry 2009). Integrin α7β1-тропізм на поверхні м'язових EVs → серцеві та печінкові таргети. Frühbeis et al. 2015 FASEB J: 30 хв велоспорту → 2–3× зростання циркулюючих EVs, повернення до базового за 2 год (відмінна кінетика від міокінів). MyomiRs: miR-1 → Cx43 (Ai et al. 2012 Circulation); miR-133a → RhoA/HERG; miR-206 → TGF-β1/HDAC4, маркер регенерації після ексцентричного ушкодження (McCarthy 2008); Nielsen et al. 2014 J Physiol: miR-1 і miR-133a зростають у 2–6× у плазмі за 30 хв після силового тренування. Адресати: печінка (Yin 2021 J Hepatol — miR-133a → ↓FASN, ↓ліпогенез), жирова тканина (miR-1 → FASN/ACLY), β-клітини підшлункової (Schoenberger 2023 — з застереженням), кістка (miR-206 → TGF-β1), серце (двонаправлений кросток). Whitham et al. 2018 Cell Metabolism: протеомний профіль — 300+ білків у вантажі EVs. Сила vs. витривалість: різний вантаж. Гіпотеза «епігенетичної пам'яті» тренування через EVs — позначена як unproven. Обмеження: стандартизація очищення (центрифугальні артефакти), функціональне поглинання in vivo у людини не доведене. Регуляторна прогалина WADA (EV-інфузія). 5 практичних висновків для силових і гібридних атлетів.
- `blog.html` — картка post-147 з описом ключових авторів/даних

### Changed
- `VERSION` → 1.21.0

---

## [1.20.0] — 2026-05-30

### Added
- `docs/COST_MODEL.md §22` — Year 1 Cash Flow Forecast & Runway Model: month-by-month Base/Bull/Bear cash table (Months 1–18), runway sensitivity matrix (6× funding levels × 5× burn rates, safe-harbor rule at ≤6 months remaining), key cash flow trigger events table (8 triggers with direction/impact/contingency), Series A readiness criteria integrated with §19.7 GTM forecast (ARR ≥ €150k, NRR ≥ 110%, ≥3 logos, ≥9 months runway, LTV:CAC ≥ 5×), cash management controls (3-account structure, weekly 5-point Monday review, 90-day variable commitment cap), OQ-23 through OQ-27 (pre-seed amount, EUR/USD FX exposure, entity formation, App Store payout lag, SOC 2 audit retainer structure). P&L vs. cash flow distinction table. Post-hire burn cliff flagged at Founding Engineer hire (Month 13 Base, +$8,333/month).

### Changed
- `VERSION` → 1.20.0

---

## [1.19.1] — 2026-05-30

### Added
- `docs/SOC2_READINESS.md §43` — Credential & Access Hardening Policy — CC6.1/CC6.2/CC6.3/CC6.6/CC6.8 Auditor Exhibit: binding implementation specifications for six P0 CC6 gaps simultaneously advanced from 🔴 Open to 🟡 AUTHORED; P0 count 10→4. GitHub org MFA enforcement (CC6-GAP-001): exact org security setting, compensating control for solo-founder phase, CC6-E-001a evidence path. Cloudflare account MFA enforcement (CC6-GAP-002): blast-radius rationale covering DEC-030 HMAC signing key + Supabase service role key in Workers Secrets, CC6-E-001b evidence path. Tenant admin TOTP enforcement (CC6-GAP-003): full Cloudflare Worker `requireAdminMfa()` middleware pseudocode, 7-day grace period with `access.tenant_admin_totp_waiver_used` DEC-030 event per invocation, SSO IdP delegation exemption requiring SAML TimeSyncToken/X509/OIDC acr=mfa context (PasswordProtectedTransport rejected). SCIM DELETE deprovisioning (CC6-GAP-004): atomic SQL transaction spec (soft-delete + seat release + `auth.admin.signOut` + DEC-030 emit), idempotency requirement, 409 last-owner guard, GDPR Art. 17 soft-delete justification, SCIM Groups cascade with per-user events. CI dependency audit (CC6-GAP-005): `npm audit --audit-level=critical` GitHub Actions job, `.github/dependabot.yml` for npm + GitHub Actions ecosystems, auditable exception bypass `[skip-audit-critical: RISK-XXXXXX]` with compliance-officer + founder co-approval gate. Secret scanning (CC6-GAP-006): `git-secrets` pre-commit hook, canonical `.github/git-secrets-patterns` file (Anthropic sk-ant-, Supabase JWT, Stripe live, ElevenLabs xi-api-key, Cloudflare Bearer, 40-char hex), CI `git secrets --scan-history` step, GHAS push protection as secondary. CSP/ESLint P1 (CC6-GAP-007): full Content-Security-Policy string with report-uri, Cloudflare Transform Rule, ESLint no-eval + no-new-func + no-implied-eval. Six new DEC-030 events: `auth.scim_user_deprovisioned` (HIGH, 7yr — closes AUDIT_LOG_SCHEMA.md lifecycle gap), `access.mfa_org_policy_enabled` (HIGH), `access.tenant_admin_totp_enrolled` (STANDARD), `access.tenant_admin_totp_waiver_used` (HIGH), `ci.secret_scan_blocked` (HIGH — pattern name only, value never logged), `ci.dependency_audit_blocked` (MEDIUM). SOC 2 mapping: CC6.1/CC6.2 (MFA + TOTP), CC6.3 (SCIM DELETE offboarding), CC6.6 (secret scan), CC6.8 (audit + Dependabot + no-eval + CSP). Evidence package: CC6-E-001a through CC6-E-006. 14-item implementation checklist (8× P0, 5× P1, 1× P2). Remaining P0 gaps: CC6-GAP-001 (access review execution), P-GAP-001 (privacy policy counsel-review), PRE-25 (Vanta/Drata), CC9-GAP-007/CC2-GAP-003 (sub-processor Wrangler deploy).

### Changed
- `VERSION` → 1.19.1

---

## [1.19.0] — 2026-05-30

### Added
- `content/post-146-myokines-muscle-endocrine-organ.md` — «Міокіни: м'яз як ендокринний орган» — 13 хв читання. IL-6 як перший міокін (Pedersen & Febbraio 2008 *Physiological Reviews*): транзиторний протизапальний сигнал vs хронічний патологічний IL-6 жирової тканини; кількісна різниця: 100× зростання без TNF-α/IL-1β, AMPK→ліполіз, протизапальна IL-10/IL-1Ra вісь. PGC-1α → FNDC5 → іризин (Boström et al. 2012 *Nature* 481): browning білого жиру, термогенез; hippocampal BDNF через PGC-1α/FNDC5 (Wrann et al. 2013 *Cell Metabolism* 18). Іризиновий консенсус 2024: підтверджений targeted mass spectrometry (Jedrychowski et al. 2015 *Cell Metabolism*), browning у людини суттєво слабший ніж у мишей; FNDC5/irisin → αV інтегрини → нейропротекція (Lourenco et al. 2019 *Nature Medicine*). FGF21 і мітохондріальний стрес-сигнал (ATF4). Meteorin-like/Metrnl (Rao et al. 2014 *Cell*): еозинофіли, IL-4, термогенез і нейрогенез. Остеокальцин–м'яз петля (Mera et al. 2016 *Cell Metabolism*): undercarboxylated OC → GPRC6A → FAO і витривалість. SPARC і колоректальна протипухлинна вісь (Aoi et al. 2013 *Gut*). 600+ міокінів у секретомі (Pathak et al. 2022). clinical-safety NOT REQUIRED.
- `blog.html` — Post 146 картка (Спортивна наука, «Незабаром», 13 хв). Картка 146 у сітці.

### Changed
- `STATUS.md` — blog.html рядок: 146 cards (146 content posts authored), v1.19.0.
- `README.md` — post-145 `[ ]` → `[x]`, post-146 `[x]` додано до content engine roadmap.
- `VERSION` → 1.19.0

---

## [1.18.0] — 2026-05-29

### Added
- `docs/SOC2_READINESS.md §42` — Personnel Security, Background Check & Confidentiality Onboarding Policy — CC1.1/CC1.4 Auditor Exhibit: advances CC1-GAP-003 🔴→🟡 AUTHORED and PRE-04 🔴→🟡 AUTHORED; P0 count 11→10. Background check provider selection: Checkr primary (SOC 2 certified, EU DPA available, 1–5 day turnaround) vs Sterling approved fallback; check scope: identity + criminal record (country of residence + global watchlist) + employment history 5yr; GDPR Art. 6(1)(b) + Art. 10 legal basis with candidate notice and Art. 28 processor DPA. Adjudication criteria: individualized assessment; four automatic disqualifiers (fraud/identity theft/computer crimes/data-privacy violations <7yr, unauthorized computer access, financial crimes <7yr, minor victims). NDA template 10-clause structure (parties, Art. 9 health data enumeration in confidential information definition, obligations, exclusions, GDPR data protection clause with 1-hour internal breach report SLA, IP assignment explicitly covering ML artefacts, 12-month non-solicitation, indefinite health data survival clause); four template variants (employee, advisor, contractor rider, pentest MNDA); DocuSign signing workflow; NDA register schema (12 columns, pseudonymized). Pre-boarding checklist: 11 steps Day −14 through Day 30; background check blocks all access; security training and AUP signing on Day 0; production write access expansion only at Day-30 access review. Four DEC-030 events: `personnel.background_check_initiated` (STANDARD 7yr), `personnel.background_check_passed` (HIGH 7yr), `personnel.nda_signed` (STANDARD 7yr), `personnel.hire_check_passed` (HIGH 7yr); manual emission path for pre-automation phase; chain-break = P0 incident. Seven evidence artefacts CC1-E-003a through CC1-E-007b. SOC 2 criteria: CC1.1, CC1.4, CC6.1, CC6.3, P3.1. PRE-15 stale status corrected via §42.10 (closed by §41 v1.17.9). 12-item checklist (6× P0 before Founding Engineer offer, 4× P1 M3/M4).

### Changed
- `VERSION` → 1.18.0

---

## [1.17.9] — 2026-05-29

### Added
- `docs/SOC2_READINESS.md §41` — Vulnerability Management & Patching SLA — CC7.1/CC7.2 Auditor Exhibit: closes PRE-15 (🔴→🟢), P0 count 12→11. Four discovery sources (Dependabot, `npm audit --audit-level=critical` CI gate, annual pentest §16, 7 vendor advisory channels). CVSS v3.1 severity tiers with three FORM-specific uplift rules (U-01 Art.9 health data, U-02 RLS bypass — never risk-acceptable, U-03 HMAC chain integrity). Binding SLA: Critical 24h / High 7d / Medium 30d / Low 90d; SLA clock mechanics (start/stop/pause); full 7-stage triage workflow with Linear ticket schema. Platform & runtime patching table for 6 vendor-managed components. Risk Acceptance protocol (U-02 exempt; Critical requires joint founder+compliance-officer approval, max 7d). Enterprise disclosure templates E-VULN-01 (cross-tenant Critical, <2h) and E-VULN-02 (Critical no cross-tenant, <24h). Five DEC-030 HMAC-chained events (`vuln.cve_discovered`, `vuln.patch_deployed`, `vuln.sla_exception_granted`, `vuln.wont_fix_closed`, `vuln.enterprise_notified`). Evidence artefacts CC7-E-001–CC7-E-009. Gap closures: PRE-15 🔴→🟢, CC5.3 Vulnerability Management Policy 🟡→🟢, CC7.1/CC7.2 advancing. 12-item implementation checklist.

### Changed
- `VERSION` → 1.17.9

---

## [1.17.8] — 2026-05-29

### Changed
- `blog.html` — додано картки post-143 (ROS/гормезис) і post-144 (аутофагія/ремоделювання) у сітку контенту

## [1.17.7] — 2026-05-29

### Added
- `content/post-143-ros-hormesis-training.md` — ROS і гормезис: редокс-сигналізація і тренувальна адаптація; H₂O₂ як сигнальний агент через NRF2/Keap1; Ristow et al. (2009 PNAS) і Gomez-Cabrera et al. (2008) — антиоксидантні добавки (вітаміни C+E) блокують мітохондріальний біогенез і знижують VO₂max-приріст; NF-κB і прозапальний ROS-сигнал; AMPK-шлях; практичні орієнтири щодо часу прийому антиоксидантів; clinical-safety NOT REQUIRED
- `content/post-144-autophagy-muscle-remodeling.md` — Аутофагія і ремоделювання м'яза: AMPK↔mTOR→ULK1 молекулярний перемикач; макроаутофагія, мітофагія (PINK1–Parkin і рецептор-опосередкований BNIP3/NIX/FUNDC1 шлях); убіквітин-протеасомна система (MAFbx, MuRF1, FOXO); p62/SQSTM1 як вузол між аутофагією і UPS; TFEB і CLEAR-мережа; Jamart et al. (2012) — LC3-II↑ після вправ у людини; саркопенія і вікове зниження аутофагічної ємності; практичне відновлювальне вікно; clinical-safety NOT REQUIRED

## [1.17.5] — 2026-05-29

### Added
- `docs/SOC2_READINESS.md §40` — Media, Device & Secrets Disposal Policy (C1.2 Auditor Exhibit): device classification A–E (laptop/iPhone/USB/BYOD/cloud), NIST 800-88 Rev 1 sanitization (Cryptographic Erase for Apple Silicon, 3-pass for Intel T2-less), 10-step offboarding with 24h revocation SLA, BYOD permitted/prohibited policy, enterprise tenant data destruction 7-step (Day 0–30), Backblaze B2 Cryptographic Erase via key rotation, device disposal log spec (C1-E-008), four DEC-030 events. Gap advances: C1-GAP-004 🔴→🟡 AUTHORED, CC1-GAP-001 🔴→🟡 AUTHORED (docs/ACCEPTABLE_USE_POLICY.md v1.0 confirmed), P0 count 13→12.

### Changed
- `docs/SOC2_READINESS.md §38.4` — gap registry: CC1-GAP-001 🔴→🟡, C1-GAP-004 🔴→🟡; gap summary P0 count updated 13→12
- `VERSION` → 1.17.5

---

## [1.17.4] — 2026-05-29

### Added
- `content/post-142-ribosomal-biogenesis-training-volume.md` — Рибосомна біогенез і тренувальний об'єм: рибосомна ємність як пляшкове горло синтезу білка (Figueiredo et al. 2015, 2017); c-Myc → Pol I → рДНК-транскрипція; mTORC1/S6K1 → TIF-1A; чому нетреновані відповідають на мінімальний об'єм; MEV/MAV/MRV через молекулярну лінзу; volume accumulation meso логіка; деловк і рибосомна кінетика. clinical-safety NOT REQUIRED.
- `blog.html` — картка post-142 (ribosomal biogenesis / training volume)
- `STATUS.md` — додано записи post-139–142 до content engine table
- `README.md` — post-139–142 відмічені [x]; нові roadmap-записи post-143–145 (ROS/гормезис, аутофагія, механотрансдукція/tensegrity)

### Changed
- `VERSION` → 1.17.4

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

## [2.56.0] — 2026-06-07

### Why MINOR
- New compliance evidence index: `compliance/cc4/README.md` — CC4 Monitoring Activities evidence index, the last missing CC directory README (cc1–cc9 now all have README.md)

### Added
- `compliance/cc4/README.md` — CC4 Monitoring Activities evidence index. Documents both CC4 sub-criteria (CC4.1 ongoing + separate evaluations; CC4.2 deficiency communication) with current status (🟡 Partial); evidence file table (12 artifacts — control-deficiency-log.csv, evidence-collection-plan.md, deficiency-communication-procedure.md, and 9 pending launch-time evidence artifacts: monthly memos, daily HMAC chain probe S-007 logs, weekly Supabase exports, monthly Cloudflare exports, quarterly access review, annual pentest, advisory brief, investor update confirmations, annual SOC 2 review); gap status table (CC4-GAP-001 🟢 Closed; CC4-GAP-002 🔴 Open — resolves launch+6mo; CC4-GAP-003 P0 🟡 compensating control active — three-tier board-less structure, expires 90 days post-Series-A); compliance calendar (10 cadences from continuous HMAC probe to Series A audit committee trigger); 5 open action items. Completes CC directory README coverage: all nine CC criteria directories (cc1–cc9) now have README.md evidence indexes. compliance-officer + security-engineer.

### Changed
- `VERSION` → 2.56.0.

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
