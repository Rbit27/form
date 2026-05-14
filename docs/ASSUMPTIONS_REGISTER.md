# FORM · Assumptions Register v0.1

> Pre-launch hypothesis register. Документує що ми зараз вважаємо правдивим — ДО досліджень.
> Оновлюється після кожного research cycle.
> Власник: `research-lead` + `product-strategist`. Review: після 30 інтерв'ю (W5).

---

## Як читати

Кожна assumption — numbered. Confidence — наше self-assessment ДО досліджень.
Після 30 інтерв'ю оновлюємо статус:

| Status | Значення |
|---|---|
| ● Untested | До досліджень / beta |
| ✓ Validated | Підтверджено даними |
| ✗ Invalidated | Спростовано — потрібен pivot |
| ~ Partial | Частково правдиво — уточнюємо |
| ? Unclear | Дані суперечливі |

**Format per assumption:**
- **Confidence:** High / Medium / Low
- **Status:** ● → ✓/✗/~/? після research
- **Test:** де і як перевірити
- **Wrong if:** конкретний сигнал який змінює висновок
- **Pivot if:** що змінюємо у продукті або стратегії

---

## 1 · User Pain Points

### A-001 · Self-coached lifters genuinely lack form feedback

**Belief:** Intermediate lifters (1–3+ years) регулярно сумніваються в техніці і не мають надійного механізму зворотного зв'язку.
**Confidence:** High
**Status:** ● Untested
**Test:** Interview Q12 — «Чи був момент, коли ти мав сумнів у техніці?»
**Wrong if:** < 30% дають конкретні приклади сумніву в техніці
**Pivot if:** < 30% → переглядаємо CV як core USP; фокус зміщується на programming/coaching

---

### A-002 · YouTube form-checks are insufficient

**Belief:** Самокоучинг через YouTube не вирішує проблему — не персоналізований, не real-time, не доступний у гімі.
**Confidence:** Medium
**Status:** ● Untested
**Test:** Q12–Q13 — чи згадують YouTube як рішення? Як реально troubleshoot техніку?
**Wrong if:** ≥ 60% кажуть що YouTube + повільна зйомка себе — достатньо
**Pivot if:** «Wrong» → перепозиціонуємо CV як «верифікація» а не «заміна YouTube»

---

### A-003 · Existing apps feel shallow for serious lifters

**Belief:** Hevy, Strong, MFP, Whoop — вони трекають, не коучать. Серйозні lifters відчувають дефіцит.
**Confidence:** High
**Status:** ● Untested
**Test:** Q6 (які апки), Q10 (що дратує)
**Wrong if:** < 40% згадують «не coaching enough» або «роблю все сам» як frustration
**Pivot if:** «Wrong» → шукаємо інший pain point; переробляємо JTBD framing

---

### A-004 · Programming complexity is a top frustration

**Belief:** Intermediate lifters витрачають забагато часу на «що робити сьогодні» і мають статичні програми, що не адаптуються.
**Confidence:** Medium
**Status:** ● Untested
**Test:** Q5 (звідки план?), Q7 (скільки часу на підготовку?)
**Wrong if:** < 40% згадують невизначеність програми або prep-time як frustration
**Pivot if:** «Wrong» → зменшуємо акцент на adaptive planning, фокус на execution layer

---

### A-005 · Missed workouts cause meaningful distress

**Belief:** Пропущені тренування викликають guilt та «streak broken → навіщо продовжувати» спіраль.
**Confidence:** Medium
**Status:** ● Untested
**Test:** Q8 — «Розкажи останній раз, коли пропустив тренування»
**Wrong if:** Більшість описують пропуск як «не страшно», без guilt-патерну
**Pivot if:** «Wrong» → уникаємо anti-skip messaging; streak anti-feature не потрібна

---

### A-006 · Wearable users want integration, not replacement

**Belief:** Apple Watch / Whoop / Oura users хочуть щоб дані HRV + сон інтерпретувалися і впливали на тренування — а не просто відображалися.
**Confidence:** High
**Status:** ● Untested
**Test:** Segment Артур — Q6, probe: «Як ти використовуєш дані з wearable зараз?»
**Wrong if:** < 50% wearable users кажуть що хочуть щоб тренування коригувалося під HRV/сон
**Pivot if:** «Wrong» → wearable integration залишається, але HRV-adjusted план — opt-in, не default

---

## 2 · Product Beliefs

### A-007 · «Smart direct» tone outperforms motivational

**Belief:** Архетип Дмитра надає перевагу стислому, фактичному коучингу перед мотиваційними фразами. Ненавидить filler.
**Confidence:** High
**Status:** ● Untested
**Test:** Q17 — реакція на нотифікацію з голосом Victor
**Wrong if:** ≥ 40% кажуть «хотів би більше підбадьорення» або знаходять «smart direct» холодним
**Pivot if:** «Wrong» → пересобираємо Victor default; toggle tone раніше в onboarding

---

### A-008 · Victor's AI nature doesn't kill trust

**Belief:** Tech-aware intermediate lifters приймають AI-коучинг без глибинного «це не справжній тренер» заперечення.
**Confidence:** Medium
**Status:** ● Untested
**Test:** Q23 — «Як ти ставишся до того, що це AI, а не людина?»
**Wrong if:** > 50% висловлюють фундаментальну недовіру до AI для коучингу
**Pivot if:** «Wrong» → підсилюємо «backed by sports scientists» framing; не AI-перший, наука-перша

---

### A-009 · Today screen as default landing is correct IA

**Belief:** Користувачі хочуть потрапити на «що роблю сьогодні», а не на прогрес, історію або чат.
**Confidence:** Medium
**Status:** ● Untested
**Test:** Q15 — показуємо Today screen; відзначаємо перше запитання і момент confusion
**Wrong if:** > 40% одразу питають «де моя історія?» або йдуть з Today за < 5 секунд
**Pivot if:** «Wrong» → розглядаємо Workout як primary landing або customizable start screen

---

### A-010 · Heavy gamification repels serious lifters

**Belief:** Streak counters, badges, XP — відштовхують серйозних lifters як infantilizing.
**Confidence:** High
**Status:** ● Untested
**Test:** Q22 — «Є щось у фітнес-апках, що тебе одразу від pushає?»
**Wrong if:** < 20% згадують gamification як негатив; більшість нейтральні або позитивні до streaks
**Pivot if:** «Wrong» → легкий streak-трекінг можливий, але non-prominent

---

### A-011 · ED-screening gate is non-negotiable (safety floor)

**Belief:** Інтеграція калорій/макросів без скринінгу ризикує тригернути disordered eating у вразливих користувачів.
**Confidence:** High (clinical-safety validated)
**Status:** ✓ Validated — DEC-018 (clinical-safety HARD VETO, незворотне)
**Test:** N/A — safety floor, не комерційна assumption
**Wrong if:** N/A
**Pivot if:** N/A — не переглядається

---

### A-012 · 8 CV-coached lifts cover 80% of user needs

**Belief:** Squat, bench, deadlift, OHP, Romanian DL, pull-up, row, lunge — покривають рухи, що мають найбільше значення.
**Confidence:** Medium
**Status:** ● Untested
**Test:** Q16 — які рухи важливі? На що найбільше хотів би feedback?
**Wrong if:** > 40% вказують high-priority рух поза нашими 8 (Olympic lifts, KB swings тощо)
**Pivot if:** «Wrong» → перепріоритизуємо CV lift order; можливо додаємо KB swing або Pendlay row

---

### A-013 · Day-7 retention is a valid early health proxy

**Belief:** D7 > 50% означає що core loop (тренування → feedback → повернення) працює.
**Confidence:** Medium
**Status:** ● Pre-beta hypothesis
**Test:** Beta analytics — PostHog D7 cohort retention (вже в ANALYTICS_SETUP.md)
**Wrong if:** D7 низький, але W4 active > очікувань (users пропускають дні, повертаються пізно)
**Pivot if:** «Wrong» → зміщуємось на W-ACSU як primary metric (вже NSM — але D7 як proxy ненадійний)

---

## 3 · Pricing & Monetization

### A-014 · WTP median ≈ $9/month для UA users

**Belief:** UA self-coached lifters заплатили б ~$9/міс за продукт, який реально замінює manual program management + CV feedback.
**Confidence:** Medium
**Status:** ● Untested
**Test:** Q19 — «Скільки б ти заплатив за це на місяць?» (без leading question)
**Wrong if:** Median відповідь < $5 або > $15 (останнє = ми залишили гроші на столі)
**Pivot if:** < $5 → переглядаємо UA pricing або tier structure; > $15 → тест підняти UA до $12

---

### A-015 · WTP median ≈ $19/month для Western EU

**Belief:** Західноєвропейські users мають вищий WTP з огляду на вартість живого тренера ($100–200/сесія) та вищий disposable income.
**Confidence:** Medium
**Status:** ● Untested
**Test:** Segment Артур (EU-profile) — Q19 з уточненням ринку
**Wrong if:** Median < $12 → цінова різниця UA/Western менша ніж у моделі
**Pivot if:** «Wrong» → стискаємо gap; переглядаємо unit econ у FINANCIALS.md

---

### A-016 · No free tier is correct (DEC-015)

**Belief:** Free tier залучає wrong users (casual, без WTP, high churn), розмиває positioning і підриває «серйозний інструмент» бренд.
**Confidence:** High
**Status:** ● Partially validated — DEC-015 задокументовано
**Test:** Q20 — «Що б тебе зупинило купити це?» — чи згадують відсутність безкоштовного тріалу
**Wrong if:** > 60% кажуть «no trial = не спробую» як dealbreaker перед покупкою
**Pivot if:** «Wrong» → розглядаємо 7-денний trial (не free tier), без окремого free-flow

---

### A-017 · Annual billing captures 20–30% of subscribers

**Belief:** ~30% power users нададуть перевагу annual для економії.
**Confidence:** Low
**Status:** ● Pre-beta hypothesis
**Test:** Post-launch: StoreKit 2 analytics — annual vs. monthly split в першому місяці
**Wrong if:** < 10% обирають annual у перші 60 днів
**Pivot if:** «Wrong» → тестуємо глибший annual discount або bundle з library of programs

---

## 4 · Technical Feasibility

### A-018 · On-device CV inference is viable on iPhone 13+

**Belief:** Apple Vision + CoreML на A15+ дає squat/bench/deadlift аналіз з < 3 с latency та < 5% battery impact на 45-хв сесію.
**Confidence:** Medium
**Status:** ● Untested (pending FE hire + prototype)
**Test:** FE engineering spike Sprint 0 — benchmark на цільовому девайсі
**Wrong if:** Latency > 5 с або battery > 10% на 45-хв сесію
**Pivot if:** «Wrong» → hybrid edge-cloud (стримуємо keypoints на сервер); деталі в TECHNICAL.md

---

### A-019 · HealthKit covers 70%+ of target wearable users

**Belief:** Apple Watch + HealthKit-bridge (Whoop, Oura синкають через HealthKit) покривають більшість wearable users без прямої API-інтеграції.
**Confidence:** Medium
**Status:** ● Untested
**Test:** Q6 interview — який wearable? Маппінг на HealthKit coverage
**Wrong if:** > 40% segment Артур використовує wearable що не синкає в HealthKit
**Pivot if:** «Wrong» → пріоритизуємо Whoop API або Garmin direct у Sprint 3+

---

### A-020 · Rep counting ≥ 90% accuracy in real gym conditions

**Belief:** CV rep counting на 3 base lifts (squat, bench, deadlift) досягає > 90% accuracy при реальних умовах гіму (змінне освітлення, кути, одяг).
**Confidence:** Medium
**Status:** ● Pre-FE hypothesis; maps to KR 2.1 (OKRS_2026.md)
**Test:** Beta — порівняння CV rep count vs. manual log у PostHog
**Wrong if:** Accuracy < 80% на beta cohort у реальних умовах
**Pivot if:** «Wrong» → CV стає «optional overlay»; manual entry залишається primary

---

### A-021 · Push notifications ≤ 2/day won't cause fatigue

**Belief:** ≤ 2 контекстних нотифікації на день не викликають notification fatigue у нашому сегменті.
**Confidence:** Medium
**Status:** ● Pre-beta hypothesis
**Test:** Beta: notification opt-out rate + кореляція з churn (PostHog funnel)
**Wrong if:** > 20% opt out з нотифікацій І вища кореляція з churn
**Pivot if:** «Wrong» → знижуємо до ≤ 1/день або переходимо на silent in-app badges

---

### A-022 · SwiftUI is sufficient for MVP performance

**Belief:** SwiftUI (не UIKit) достатній для workout-live CV overlay, charts та animations при цільових 60 fps.
**Confidence:** Medium
**Status:** ● Pre-FE hypothesis
**Test:** FE Sprint 1 — performance test CV overlay у SwiftUI на iPhone 13
**Wrong if:** FE повідомляє SwiftUI limitations блокують < 3 с feedback або < 60 fps на CV screens
**Pivot if:** «Wrong» → hybrid SwiftUI / UIKit для CV-heavy screens

---

## 5 · Market & Competition

### A-023 · No competitor combines CV + HRV + AI coach

**Belief:** Станом на початок 2026, жодна UA/EU апп не доставляє CV form analysis + HRV-adjusted programming + AI coaching voice в одному продукті.
**Confidence:** High
**Status:** ~ Partial — COMPETITIVE.md (8-product teardown) підтвердив; моніторинг ongoing
**Test:** Продовжуємо моніторинг; Q6 interview: «що ти пробував?»
**Wrong if:** User згадує апп що робить всі три добре і платить за неї зараз
**Pivot if:** «Wrong» → загострюємо диференціацію на Victor's coaching personality + Kyiv/UA positioning

---

### A-024 · UA market is underserved AND has sufficient WTP

**Belief:** Ukrainian serious lifters — underserved segment з достатнім WTP для premium (по UA-стандартах) продукту.
**Confidence:** Medium
**Status:** ● Untested
**Test:** Q19 з UA cohort + friction comments
**Wrong if:** UA users cite price + «є безкоштовні апки» + WTP < $5
**Pivot if:** «Wrong» → пріоритизуємо Western EU як primary launch market; UA як secondary/delayed

---

### A-025 · Majority of serious gym-goers are self-coached

**Belief:** Більшість intermediate+ gym users не мають активних стосунків з особистим тренером.
**Confidence:** Medium
**Status:** ● Untested (implied by PERSONAS.md)
**Test:** Q14 — personal trainer history; вибірка: coach-free є default?
**Wrong if:** > 40% мають активний особистий тренер
**Pivot if:** «Wrong» → «trainer supplement» positioning замість «trainer replacement»

---

## 6 · Go-to-Market

### A-026 · Reddit + Twitter are primary beta recruiting channels

**Belief:** r/fitness, r/weightroom, X fitness communities — правильні канали для рекрутингу першіх 200 beta users.
**Confidence:** Medium
**Status:** ● Untested
**Test:** BETA_RECRUITING.md tracking — convert rate по каналам Tier 1/2/3
**Wrong if:** < 20% beta invites з Reddit/Twitter конвертуються в active users
**Pivot if:** «Wrong» → зміщуємось на YouTube comments outreach, gym partnerships

---

### A-027 · Build-in-public generates qualified leads

**Belief:** Показ build process (decision log, product thinking) залучить якісних early adopters і potential team members.
**Confidence:** Medium
**Status:** ● Untested
**Test:** GitHub repo stars/watches growth; inbound DMs що посилаються на repo
**Wrong if:** 0 inbound qualified leads з GitHub/Twitter за перші 60 днів
**Pivot if:** «Wrong» → build-in-public = культура, не acquisition; зміщуємось на direct outreach

---

### A-028 · 9 blog posts drive qualified top-of-funnel

**Belief:** Пости про HRV, форму, AI-тренери — ранжуються в пошуку і залучають users що вже шукають рішення наших pain points.
**Confidence:** Low
**Status:** ● Untested (drafts готові, не опубліковані)
**Test:** Публікуємо 2 пости, вимірюємо 90-day organic traffic + beta waitlist conversion з блогу
**Wrong if:** < 1% відвідувачів блогу конвертуються в beta signup
**Pivot if:** «Wrong» → pivot content на distribution (Reddit threads, Twitter) замість SEO

---

### A-029 · Wave 0 comes from founder's personal network

**Belief:** Перші 30 beta users (Wave 0) можна знайти через особисту мережу засновника без paid acquisition.
**Confidence:** High
**Status:** ● Pre-execution hypothesis; maps to BETA_RECRUITING.md Wave 0
**Test:** Трекаємо по каналах — скільки Wave 0 users прийшли з personal vs. cold outreach
**Wrong if:** Засновник не може знайти 30 qualified users з особистої мережі
**Pivot if:** «Wrong» → запускаємо Tier 2 (community) recruiting паралельно з Wave 0

---

### A-030 · First 2,000 paid users in 90 days post-launch is achievable

**Belief:** При умовах: 200 beta users, хороший D7 retention, правильний App Store listing і content distribution — 2,000 платних users за 90 днів після launch є досяжним цілем.
**Confidence:** Low
**Status:** ● Aspirational goal — не pre-launch assumption, а post-launch checkpoint
**Test:** Post-launch: StoreKit 2 subscription cohorts
**Wrong if:** < 500 paid users через 90 днів
**Pivot if:** «Wrong» → reassess pricing, onboarding, paid acquisition; review FINANCIALS.md unit econ

---

## Review schedule

| Тригер | Дія |
|---|---|
| Після 10 інтерв'ю | Quick scan — позначаємо high-confidence assumptions що вже суперечать даним |
| Після 30 інтерв'ю | Повний review — оновлюємо всі статуси |
| Beta W1 | Technical assumptions A-018–A-022 — оновлюємо з engineering observations |
| Beta W4 | Product + GTM assumptions — оновлюємо з retention + feedback data |
| Post-launch D+30 | Monetization assumptions A-014–A-017 — перший реальний сигнал |

---

## Зв'язки з іншими документами

| Document | Що використовує з ASSUMPTIONS_REGISTER |
|---|---|
| `docs/RESEARCH.md` | Test columns → interview questions (Q12, Q15, Q17, Q19, Q23) |
| `docs/PERSONAS.md` | A-001–A-006 (user pain) + A-025 (market size) |
| `docs/OKRS_2026.md` | A-020 → KR 2.1; A-013 → KR 1.5; A-029 → KR 1.2 |
| `docs/ANALYTICS_SETUP.md` | A-013, A-020, A-021 → event taxonomy і dashboard |
| `docs/FINANCIALS.md` | A-014, A-015, A-017, A-030 → unit econ inputs |
| `docs/COMPETITIVE.md` | A-023 (moat validation) |
| `docs/BETA_RECRUITING.md` | A-026, A-029 (channel strategy) |

---

**v0.1 · травень 2026 · `research-lead` + `product-strategist`**
