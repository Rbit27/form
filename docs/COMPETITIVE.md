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

**v0.1 · травень 2026 · review monthly · `marketing-lead`**
