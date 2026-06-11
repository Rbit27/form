# FORM · Competitive Teardown v0.1

> Без табличок-маркетингу. Реальний розбір — що працює, що не працює, де ми б'ємо, де ні.
> Власник: `marketing-lead`. Apdate щомісяця.

---

## 1 · Карта поля

```
                 │  Coaching / Программирование
                 │
        Future ──┼── FORM (target)
                 │
   Trainerize ── │                ── Fitbod
                 │
                 │
    MyFitnessPal ┼── MacroFactor    Hevy ── Strong
                 │  (nutrition)      (workout log)
                 │
                 │── Apple Fitness+   ── Strava
                 │   Nike Training    (cardio)
                 │
                 │  Pure consumption / brand
   ──────────────┼──────────────────────────
                 │  Hardware-coupled
                 │
        Whoop ── ┤── Apple Watch ── Oura
                 │   (recovery / data)
                 │
                 │  Pure tracking
```

**Wedge FORM:** верхній правий — coaching + program + nutrition + recovery, **програмний, не hardware**.

---

## 2 · Whoop · Deep teardown

### Що це
Subscription-only wristband ($30/міс після інвестиції в strap). Track HRV (RMSSD), sleep stages, strain, recovery score. **Це фітнес-мірний пристрій, не тренер.**

### Сильні
- Найкращий HRV-tracking на ринку (правильні методи, медичний рівень)
- Бренд серед серйозних athletes (NBA, NFL adoption)
- Membership-model з день один — без bait-and-switch
- Coaching content (Whoop Podcast) високий рівень

### Слабкі
- Не каже **що робити** з даними («Recovery 47%» — окей, а зараз?)
- Програми тренувань нема
- Харчування нема
- Real-time coaching нема
- Тон pop-fitness, не expert
- Battery life слабша за конкурентів (3 дні)

### Де FORM б'є
- Програма реагує на HRV — Whoop не може цього зробити
- Inтеграція з Whoop API → їхні дані стають корисніші у нашій руці
- Голос Victor — пояснює, не лише показує

### Де FORM програє
- Дані рівня — Whoop дбає про науку, ми переймаємо
- Hardware-стабільність — Whoop працює без телефону
- Brand рівень серед athletes — нам ще будувати

### Стратегічний висновок
**Whoop = data provider. FORM = action layer.** Інтеграція через API — ми робимо їхні дані ціннішими. У 3-річному горизонті — кандидат на партнерство або викуп.

---

## 3 · Fitbod · Deep teardown

### Що це
$13/міс iOS-app. Auto-generate workout-програму. ~600k MAU. Бренд серед self-coached.

### Сильні
- Перший хороший auto-програма на масовому рівні
- Велика база вправ (~600)
- Equipment-customization (домашній, повний зал, дамбли тощо)
- iOS-якість UI висока

### Слабкі
- **Не коуч.** Каже що робити, не каже як робити.
- HRV-aware planning нема
- Тон AI generic, без персони
- CV нема
- Харчування нема
- Камера-аналіз і голосовий коучинг нема
- Plan adaptation slow (per-week, не per-day)

### Де FORM б'є
- Камер-аналіз техніки — Fitbod нема
- HRV/sleep-driven planning — Fitbod нема
- Голос (Victor) — Fitbod написaв текстів generic
- Харчування інтегровано — Fitbod не торкає

### Де FORM програє
- Equipment-customization (поки) — Fitbod glasу-стандарт
- Vast exercise library — ми орієнтуємо на 8 базових + ~50 secondary
- Бренд on App Store — Fitbod featured кілька разів

### Стратегічний висновок
**Fitbod це FORM v0.0.** Ми робимо те що Fitbod, плюс camera + voice + recovery. Прямий швидкий update може стати загрозою — але Fitbod організаційно повільний, і їх core-команда не має CV-експертизи.

---

## 4 · Future · Deep teardown

### Що це
$149/міс. Реальний живий тренер пише і коригує програму через app. Не AI — людина. ~25k платних.

### Сильні
- Real coaching value (people swear by it)
- Тренер пам'ятає тебе, твою історію, твої цілі
- Personalization найвищого рівня
- Bran серед moderate-affluent self-coached

### Слабкі
- $149/міс — дорого, market-cap самим self
- Не масштабується — кожен тренер веде 20–40 клієнтів
- Тренер залежність — людський bandwidth
- Asynchronous (текст-чат, не real-time)
- Не дивиться як ти робиш вправу

### Де FORM б'є
- Ціна (7–15× дешевше)
- Real-time camera coaching — Future неможливо без AI
- Доступність 24/7 — людина-тренер не цілодобово

### Де FORM програє
- «Реальний» коуч — bond з людиною непідробний
- Premium позиція (Future high-end)
- Customization до edge case (травми, цілі)

### Стратегічний висновок
**Future цільовий exit point для частини нашої аудиторії.** Той хто хоче саме людину — Future. Ми задовольняємо 70% задачі за 10–15% ціни.

---

## 5 · MacroFactor · Deep teardown

### Що це
$12/міс. Nutrition-only app. ~80k платних. Бренд серед serious cutters/bulkers.

### Сильні
- Найкращий nutrition-калькулятор на ринку (тренди по фактичних даних)
- Anti-fad approach (не keto, не carnivore, тільки calc-driven)
- Бренд серед RP/Stronger by Science community
- Дані експорт чистий

### Слабкі
- Workout нема (свідомо — focus)
- HRV-coupling нема
- Тон clinical, нема голосу
- Інтеграцій з wearables мінімум

### Де FORM б'є
- Інтеграція з workout + recovery — MacroFactor спеціально це не робить
- Голос — MacroFactor cold spreadsheet feel
- One-app experience

### Де FORM програє
- Nutrition depth — MacroFactor спеціалізовано
- Бренд серед dieting community

### Стратегічний висновок
**Часто співіснуватимуть.** Користувач Fitbod + MacroFactor — наш ідеальний прицил. Ми робимо обидва, інтегровано. Risk: користувачі прив'язалися до MacroFactor data-history і не міняють. Mitigation: import-tool на MacroFactor CSV.

---

## 6 · Hevy / Strong · Deep teardown

### Що це
Hevy: $0–6/міс. Strong: $0–5/міс. Workout-логери. Hevy: ~3M installs.

### Сильні
- Free tier і паїд-tier
- Швидкий лог
- Соц-фічі (Hevy має following / feed)
- Простий UI

### Слабкі
- Не коуч, log тільки
- Програми — або вручну, або сторонні
- Жодних AI features
- Жодних camera / voice / recovery

### Де FORM б'є
- Все, що Hevy не робить — а це 90% продукту

### Де FORM програє
- Free price point — Hevy виграє у consumer-default
- Speed of logging — Hevy purpose-built

### Стратегічний висновок
**Hevy/Strong — це наша вхідна аудиторія.** Користувач Hevy 6+ міс — найбільш вірогідно повірить у coaching upgrade. Стратегія: contribute to Hevy community on Reddit, build trust, mention FORM як «next step».

---

## 7 · Apple Fitness+ · Deep teardown

### Що це
$10/міс (або $13 з Apple One). Video workouts з instructorами. ~5M+ subscribers (Apple-leveraged).

### Сильні
- Apple bundle + Watch інтеграція — distribution distortion
- High production value
- Brand strength
- Доступність для casual

### Слабкі
- Class-based, не coached
- Не персоналізовано — групове
- Не реагує на твій рекаверу
- Не camera-aware (тільки Watch metrics)

### Де FORM б'є
- Кожна dimension persoнalization
- Не для self-coached serious lifter — Apple Fitness+ свідомо casual

### Де FORM програє
- Distribution через Apple One bundle — це структурна перевага

### Стратегічний висновок
**Не конкуренти.** Apple Fitness+ — для іншої аудиторії (casual, group, video-led). Якщо Apple колись запустить «AI Trainer» — це загроза. Але Apple-цикл будівництва ~3 років, ми будемо далі.

---

## 8 · MyFitnessPal · Deep teardown

### Що це
$10–20/міс або free. Food log + barcode scanner. ~200M реєстрацій всеглобально (legacy bran).

### Сильні
- Найбільша база їжі (величезна)
- Brand recognition серед casual
- Free tier — distribution

### Слабкі
- Free-tier LTV крихкий
- UX legacy, slow
- Не coach, не workout
- Acquired by Francisco Partners → slow innovation

### Де FORM б'є
- Все що про coaching і workout
- UX modernity
- HRV integration

### Де FORM програє
- Food database — це 20-річна перевага MFP
- Free-tier distribution
- Brand recognition (поки)

### Стратегічний висновок
**Food parsing — це pain point.** Або (a) ліцензуємо access до MFP food DB через USDA/Open Food Facts + LLM fuzzing, (b) робимо власну базу повільно. Wager: USDA + Open Food Facts + Anthropic Vision = 95% MyFitnessPal-quality за 3 місяці.

---

## 9 · Strava · Deep teardown

### Що це
$80/рік. Бренд серед бігунів і велосипедистів. ~25M MAU.

### Сильні
- Соц-сітка running/cycling
- Дані-rich сегменти, KOMs
- Strava-stories у crowd

### Слабкі
- Strength training secondary
- Recovery нема як в Whoop
- Coaching нема

### Де FORM б'є
- Не б'є — це інший спорт-фокус (cardio)

### Стратегічний висновок
**Можливе майбутнє co-existence.** Користувач Strava + FORM — нормально. Інтеграція через Strava Activities API — phase 2.

---

## 10 · Defensibility map FORM

| Layer | Defensibility | Чому |
|---|---|---|
| **Brand voice (Victor)** | HIGH (з часом) | Persona-fixation + ED-floor — імітувати займе 6+ міс, та не для всіх |
| **CV pipeline на 8 рухах** | MEDIUM-HIGH | Custom heuristics + data → краще з кожним користувачем |
| **HRV-aware programming** | MEDIUM | Whoop або Apple можуть зробити, але організаційно повільні |
| **Multi-source aggregation** | MEDIUM | API-integrations technical, але можна повторити |
| **Trust і community** | HIGH | Build over years, hard to copy |
| **Anti-shame floor** | HIGH | Стратегічний вибір, конкуренти бояться втратити агресивну меседжинг-частину аудиторії |

---

## 11 · Ризики від компетіторс

| Хто | Що зроблять | Loly |
|---|---|---|
| Fitbod adds AI voice + camera | Очікувано в Q4 2026 | Medium |
| Whoop launches "Coach" feature | Q1 2027 максимум | High |
| Apple Fitness+ adds personalization | Не до 2027 | Low |
| Future spins B2C AI product | Можливо, але conflict з business model | Low |
| Stripe stack AI startup (нові гравці) | 2–5 шт. вже в idea-stage | High |

**Найбільша загроза не названа: VC-backed AI fitness стартап з $20M Seed.** Якщо такий запуститься у Q3 2026 з PR-кампанією — ми ризикуємо «бути другими». Mitigation: ship швидко, brand depth, чесна позиція стане diff.

---

## 12 · Where we don't compete (свідомо)

- Pure cardio (Strava)
- Yoga / mobility-only (Down Dog, Yoga with Adriene)
- Bodybuilding contest prep (specialized coaches)
- Kids/teens fitness (separate regulatory + ethical)
- Group classes (Peloton, ClassPass)
- Hardware bundles (Tempo, Mirror, Whoop)

---

---

## 13 · Enterprise / Corporate Wellness: Карта поля B2B

> Окремий ринок від consumer-tier. HR / People бюджет, не персональна підписка.
> Тут FORM конкурує з wellness-платформами, gym-access мережами і human-coach SaaS.
> Prices нижче — [ESTIMATE] з публічних звітів (Mercer, SHRM, G2/Capterra); ніколи не публікуються вендорами.

```
               │  Exercise coaching / programming
               │
  Future Ent.──┼────────────────── ← FORM Enterprise ($6-12/seat) →
               │
               │                    Wellhub (gym access, different category)
               │
 Headspace ────┼──── Calm             Virgin Pulse ────────────────────────
  for Work     │   for Business       Personify Health
  ($10-14)     │   ($10-15)           ($40-75)
               │
               │   Mental wellness / incentive platform
──────────────────────────────────────────────────────────────────────────
               │   Wearable-only
  Whoop ───────┼── Whoop for Teams ($18-30)
               │   (device dependency — coaching нема)
```

**Wedge FORM у B2B:** coaching продукт з compliance-grade infrastructure за ціною gym stipend replacement ($72–144/seat/year vs. $600–1,800/employee/year нетрекований gym stipend). Unicorn-квадрант: SSO + SOC 2 + CV coaching + wearable, без hardware dependency, без incentive/risk-scoring feature-set.

---

## 14 · Wellhub (Gympass) · Enterprise teardown

### Що це
Gym-access network SaaS. Корпоративний план надає співробітникам доступ до 50k+ фізичних gym/studio/app у 11 країнах. ~3k корпоративних клієнтів. Брендована ребрандингом з Gympass → Wellhub у 2023.

### Сильні
- Широта мережі — якщо офіс-команда, відвідуваність висока
- HR вже купує — перша лінія захисту wellness-бюджет
- Бренд на EU ринку, особливо CEE
- Includes Headspace, Calm, Noom — bundled mental/nutritional content
- Price tier flexibility ($8 → $45) дозволяє HR знайти компроміс

### Слабкі
- **Remote/distributed workforce = zero value.** Gym access for 100% remote company доставляє нульовий ROI.
- Participation tracking superficial — «відвідав зал» ≠ тренувався продуктивно
- Жодного programming, coaching або feedback
- Жодної CV-аналізу, HRV-aware planning
- No SSO-grade access management — user self-serves on personal email
- Aggregate engagement data обмежені (відвідування, не outcomes)
- **No SOC 2 Type II** — compliance buyers у fintech/healthtech тут зупиняються

### Де FORM б'є
- Remote/hybrid workforce: FORM повністю digital, без facility dependency
- Coaching outcomes vs. raw gym visits — measurable programming progress
- Admin dashboard з aggregate metrics (privacy-first, k-anon) vs. Wellhub's coarse attendance counts
- SSO/SCIM (Okta, Azure AD, Google Workspace) — enterprise IT вимога без компромісу
- SOC 2 Type II + GDPR DPA — compliance review пасує там де Wellhub fall out

### Де FORM програє
- Фізична мережа: якщо компанія хоче «доступ до LifeFitness в центрі міста» — Wellhub, не FORM
- Бренд recognition у HR-спільноті: Wellhub відомий, FORM невідомий поки
- Bundle depth: Wellhub includes yoga apps, mental apps, meditation — FORM pure-play fitness coaching

### Стратегічний висновок
**Не конкуренти для in-office populations; прямі конкуренти для remote-first companies.** Сегментація за pitch-deck: «якщо >30% remote → Wellhub недопоставляє». У mixed-model (hybrid office), FORM + Wellhub можуть співіснувати. Не варто атакувати Wellhub headon — позиціонуватися як доповнення або replacement для remote segment.

---

## 15 · Headspace for Work · Enterprise teardown

### Що це
Корпоративна версія Headspace consumer app. Guided meditation, sleep, stress-management content. $10–$14/seat/month. Велика library of guided sessions.

### Сильні
- Consumer brand awareness серед employees (знають продукт до корпоративного плану)
- Реальний контент (meditation science-backed, clinically reviewed)
- Простий procurement: HR купує, employees self-activate
- Evidence-based stress reduction (ROI data available for HR)
- Recently merged with Calm's enterprise competitor — competitive pressure

### Слабкі
- **Не coaching, не exercise.** Meditation ≠ fitness outcome.
- Жодного wearable integration, жодного programming
- Participation rates низькі — 40–60% activation зазвичай peak у launch, decay до 10–20% за 90 днів
- No SSO at Starter tier (enterprise add-on, прихована ціна)
- Admin dashboard обмежений: activation rate, time spent — не outcome-linked
- Не замінює gym stipend у mind HR-buyer

### Де FORM б'є
- Physical coaching outcomes vs. mental wellness (different HR budgets — not competing for same line)
- Long-term engagement: HRV-aware daily adaptation vs. static content library
- SSO/SCIM included у FORM Enterprise, не за доплату

### Де FORM програє
- Mental wellness coverage: FORM не замінює stress-management/meditation
- Clinical evidence base: Headspace має peer-reviewed research за продуктом; FORM не має (поки)
- Brand у wellness коло: Headspace = trusted у HR-спільноті

### Стратегічний висновок
**Типово co-exist, не compete.** HR може купити обидва з різних budget lines (EAP/mental wellness vs. physical fitness). FORM не конкурує за Headspace budget — конкурує за gym stipend budget. Якщо HR каже «у нас вже є Headspace» — правильна відповідь: «добре, це різні бюджети, різна цінність, різна ROI».

---

## 16 · Calm for Business · Enterprise teardown

### Що це
Корпоративна версія Calm. Meditation, sleep, Focus Music. $10–$15/seat/month. Конкурент Headspace for Work.

### Сильні
- Consumer brand (Calm є найбільш завантаженим meditation app)
- Sleep content depth — кращий ніж Headspace у sleep segment
- Corporate wellbeing reporting (activation, session counts)

### Слабкі
- Аналогічно Headspace: не coaching, не exercise
- Activation decay аналогічний (high launch, low D90)
- Procurement-heavy: Calm B2B sales cycle довгий, ціни negotiated

### Де FORM б'є та програє
Ідентично §15 Headspace for Work. Обидва — mental wellness, не physical coaching.

### Стратегічний висновок
**Same analysis as Headspace.** Different provider, same buyer-framing answer. HR teams often use Headspace OR Calm, not both — FORM does not enter this competition.

---

## 17 · Virgin Pulse / Personify Health · Enterprise teardown

### Що це
Comprehensive corporate wellness platform. Incentive programs, biometric screening, health coaching, challenges, rewards. $40–$75/seat/month. Фокус: large self-insured employers (Fortune 500), health-insurance-adjacent wellness.

### Сильні
- Повна platform: challenges, biometrics, coaching, rewards, EAP integration
- Incentive management — HR може прив'язати wellness participation до premium discount (insurance integration)
- Governance + compliance legacy: SOC 2, HIPAA-adjacent documentation
- Long enterprise relationships (multi-year, high switching cost)
- Strong in US large employer segment

### Слабкі
- **Legacy architecture:** UX є FORM's competitive nightmare — і великою можливістю. Mobile app отримує 2.1–2.7 зірок у App Store reviews
- **Incentive/punishment framing** — прив'язка до insurance premium знижки є точно тим що FORM **не робить** (no-go per ethics policy)
- $40–$75/seat = 4–8× дорожче ніж FORM; часто під питанням при budget cuts
- No real-time exercise coaching або CV feedback
- No SSO-native (often SCIM-only at high tier)
- Heavy implementation: 3–6 month setup, onboarding consultants required
- Engagement зазвичай incentive-driven, не intrinsic — churn high якщо incentives removed

### Де FORM б'є
- UX: FORM coach-first mobile experience vs. Virgin Pulse's compliance-form UX
- Price: 4–8× cheaper
- Privacy floor: FORM never links wellness participation to insurance or HR decisions; VP does — this is a deal-breaker for EU buyers (GDPR Art. 9)
- Implementation speed: FORM SSO/SCIM in days, not months
- Real-time exercise coaching vs. step challenges and quizzes

### Де FORM програє
- Biometric screening integration: VP has lab-result ingestion; FORM does not and will not (ethics)
- Incentive program management: companies that want reward points and premium discounts must use VP or similar
- US large-employer relationships: VP has incumbent contracts in Fortune 1000; FORM has zero reference customers to start
- HIPAA depth: VP markets HIPAA BAA for health data; FORM GDPR-first, HIPAA-adjacent

### Стратегічний висновок
**Virgin Pulse is the category we are NOT building.** Incentive-based wellness platforms with insurance-premium linkage are ethically out of scope for FORM (see `docs/ENTERPRISE.md §"When we say no"`). The correct framing for procurement: «VP is a compliance and incentive tool. FORM is a coaching product. They serve different problems.» In competitive situations, surface the UX gap (2-star App Store vs. coaching-first mobile) and the EU privacy concern (incentive-linked wellness may not pass GDPR Art. 9 review without careful DPA architecture — FORM has a clean floor by design).

---

## 18 · Whoop for Teams · Enterprise teardown

### Що це
Корпоративний план Whoop. HRV/recovery monitoring для команд. $18–$30/seat/month з device included. Фокус: performance-oriented organizations (sports teams, military, elite training environments).

### Сильні
- Найкращий recovery/HRV monitoring на ринку (підтверджено науково)
- Device included — no BYOD complication
- Strong brand у athletic performance context (NFL, NBA, military)
- Real data quality: RMSSD measurement methodology медично обґрунтований

### Слабкі
- **Device dependency = logistic overhead** для corporate deployment (inventory, breakage, hygiene)
- Не coaching: «Recovery 47%» — але що з цим робити на тренуванні?
- No exercise programming, no CV feedback, no nutrition
- Remote team: must physically send/track devices — nightmare at 500+ seats
- No SSO/SCIM; no admin dashboard beyond team management basics
- No SOC 2 documentation publicly available for enterprise procurement
- Meaningful only для employees who exercise consistently — general office population under-adopts wearables

### Де FORM б'є
- No hardware dependency: FORM works on every iPhone, no shipment logistics
- Coaching layer: FORM uses HRV data (from Whoop API або інших wearables) і каже що робити
- SSO/SCIM/SOC 2 для compliance procurement
- Ширша addressable population: не тільки athletes, але self-coached employees з будь-яким рівнем

### Де FORM програє
- Recovery data quality: Whoop outperforms Apple Watch і Garmin у RMSSD accuracy; FORM depends on what employees already own
- Вже-купили ситуація: якщо HR вже роздав Whoop devices — integration, не displacement; Whoop і FORM комплементарні

### Стратегічний висновок
**Partnership opportunity > competitive threat.** FORM інтегрується з Whoop API (§Identity у `docs/ENTERPRISE.md`). Для компаній, які вже мають Whoop devices: FORM стає coaching layer поверх Whoop data. Pitch: «Whoop gives you the signal. FORM tells you what to do with it.» Direct competition лише якщо HR хоче тільки recovery monitoring без coaching — тоді Whoop wins.

---

## 19 · Future (Enterprise) · Enterprise teardown

### Що це
Human-coach-via-app корпоративний план. Тренер пише і коригує програму через мобільний додаток. $99–$149/seat/month. Дивись також §4 (consumer teardown) — enterprise tier аналогічний модель.

### Сильні
- Real human coach: персональний bond, пам'ять контексту
- Strongest outcomes data in the segment
- Premium positioning — reference customer quality

### Слабкі
- **Ціна:** $99–$149/seat × 100 seats = $120–$180k/year для 100 осіб. FORM = $12k/year для тих самих 100 на Starter tier — 10–15× різниця.
- Human bandwidth: кожен тренер веде 20–40 clients; не масштабується до 500+ seat deals
- No real-time CV feedback — coach communicates async text/video
- No SSO/SCIM: designed for consumer, enterprise adoption limited
- Procurement: $100k+ deal requires CFO approval, not HR budget authority

### Де FORM б'є
- Ціна (10–15×) — для HR-buyer з mid-market budget
- SSO/SCIM/SOC 2 compliance infrastructure
- Real-time CV coaching (Future неможливо без AI) 
- Scale: 1,000-seat FORM deal = операційно trivial; Future = неможливо (потрібно ~25–50 нових тренерів)

### Де FORM програє
- «Реальна людина» bond: для employees, які хочуть саме людину-тренера, FORM не відповідь
- Outcomes evidence: Future має peer-cited outcomes data; FORM on Day 1 не має

### Стратегічний висновок
**Future Enterprise є aspirational ceiling, не direct threat.** Типовий customer overlap малий (Future є premium benefit для топ-executives; FORM є fleet-wide coaching для всіх). Якщо HR-buyer порівнює — це сигнал: вони серйозні про coaching outcomes, не тільки checkbox wellness. Вграти ціновий аргумент і SSO.

---

## 20 · Enterprise defensibility map FORM

| Layer | Defensibility | Чому |
|---|---|---|
| **SSO/SCIM/SOC 2 compliance stack** | HIGH | Engineering investment + audit timeline = 9–12 months to replicate; incumbent advantage після першого SOC 2 report |
| **Privacy floor by design** | HIGH | «HR never sees individual data» is a product architecture decision — competitors who already sold analytics features cannot reverse; FORM builds clean from Day 1 |
| **CV coaching у corporate context** | MEDIUM-HIGH | On-device model + FORM-specific form cues; no corporate wellness competitor ships CV feedback |
| **Admin dashboard aggregate UX** | MEDIUM | Virgin Pulse UX bar є низьким; well-designed aggregate analytics dashboard є rare and differentiating |
| **Audit log integrity (DEC-030 HMAC chain)** | HIGH | Tamper-evident chain is a technical architectural choice that satisfies CC6.1/CC7.2 без bespoke vendor attestation; hard to bolt on post-facto |
| **Pricing efficiency** | MEDIUM | $6–12/seat is structurally defensible at 83–88% fully-loaded GM; attacker must accept < 70% GM to undercut meaningfully |
| **No insurance/incentive features** | MEDIUM | Ethical constraint = market segment restriction, але також differentiation for EU buyers and employee-trust-sensitive companies |

---

## 21 · Enterprise sales objection map

| Objection | Source | Answer |
|---|---|---|
| «У нас вже є Wellhub» | HR / Procurement | Different category. Wellhub = gym access for in-office. FORM = daily coaching for everyone, including 100% remote. |
| «У нас вже є Headspace/Calm» | HR | Different budget line. Mental wellness ≠ physical coaching. Many companies run both. |
| «Ми розглядаємо Virgin Pulse» | Benefits manager | VP = incentive/compliance platform. FORM = coaching product. VP UX score = 2.1 stars. FORM does not link wellness to insurance or HR decisions — GDPR-clean by design. |
| «Покажіть ROI» | CFO / Finance | §20 `docs/COST_MODEL.md`: gym stipend $600–1,800/employee/year vs. FORM $72–144/year. Measurable activation + engagement data from admin dashboard. ROI model available in `docs/ENTERPRISE.md`. |
| «SOC 2?» | InfoSec / IT | SOC 2 Type II підготовка in progress; Type I target M6; observation period start M6; report M12. Penetration test annual. Full evidence package in data room. |
| «Де наші дані зберігаються?» | InfoSec / Legal | Supabase EU region (Frankfurt). Cloudflare EU routing. DPA підписано. SCC Module 2 для US processors. Subprocessor list: `form.coach/subprocessors`. |
| «Ви невеликий стартап — ризик» | Procurement / Legal | Business Continuity Plan (`docs/BUSINESS_CONTINUITY.md`). Data export on request (GDPR Art. 20). MSA з termination-for-convenience clause. Escrow option доступний від M9. |
| «Нам потрібен HIPAA BAA» | US Health benefits lead | FORM is GDPR-native, HIPAA-adjacent. Health data не передається третім особам. BAA доступний на запит, але FORM не є covered entity і не приймає PHI. |
| «Можемо побачити demo адмін-панелі?» | HR / People ops | `admin-dashboard.html` — live preview available. Sandbox environment для pilot accounts. |

---

*v0.2 (2026-06-11): §§13–21 Enterprise / Corporate Wellness competitive analysis — closes `docs/COST_MODEL.md §31.9` checklist item 6 (P1/M6: «COMPETITIVE.md enterprise section»). Covers: B2B field map; teardowns of Wellhub, Headspace for Work, Calm for Business, Virgin Pulse/Personify Health, Whoop for Teams, Future Enterprise; enterprise defensibility map; sales objection map. Pricing data [ESTIMATE] from public reports (Mercer, SHRM, G2/Capterra) — verify annually. No fabricated customer logos or case studies. Features referenced grounded in `docs/ENTERPRISE.md`. Owner: marketing-lead + enterprise-architect + compliance-officer.*

**v0.1 · травень 2026 · v0.2 · 2026-06-11 · review monthly · `marketing-lead`**
