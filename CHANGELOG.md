# Changelog · FORM

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
