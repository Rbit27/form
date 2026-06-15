# Changelog · FORM

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

### Changed

- `README.md` — Block 851–900 table updated: actual titles for posts 856–870 replacing planned-topic placeholders; post-871 added; block status updated to in progress · 21/50 · v5.22.0; proposed topics 872–880 added to block table.
- `blog.html` — posts 871–878 entries prepended to article list (total 177 entries).

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
