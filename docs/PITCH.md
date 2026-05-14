# FORM · Investor Pitch v0.1

> 10-хвилинний пітч. 12 слайдів. Без bullshit.
> Власник: `product-strategist`. Цифри: `growth-lead`.

---

## Slide 01 · Hook (15 секунд)

> Минулого року 38 мільйонів людей у США та ЄС платили за фітнес-апку, яка лише логує тренування. Жодна з них не сказала їм, **як саме виправити техніку у п'ятому повторенні присіду**.
>
> FORM каже.

---

## Slide 02 · Проблема

**Між «беру тренера за $200/тиждень» і «треную сам з YouTube-програми» — провалля.**

| Опція | Ціна | Що отримуєш | Чого нема |
|---|---|---|---|
| Особистий тренер | $200–400/тиждень | Реальний коучинг техніки, адаптація | Дорого. Розклад. Залежність від людини. |
| Free лог (Hevy, Strong) | $0 | Лог сетів і повторень | Не коуч. Не каже що робити. Не дивиться на тебе. |
| AI-програма (Fitbod) | $13/міс | Auto-програма | Не коуч. Тон generic. Не дивиться на тебе. Не реагує на сон/HRV. |
| Recovery-tracker (Whoop) | $30/міс | Дані тіла | Не програма. Не їжа. Не голос. |

**Нікому не вдалося скласти все в один продукт. Ми складаємо.**

---

## Slide 03 · Insight

Серйозний самостійний лифтер хоче **одну річ**: щоб хтось знаючий **дивився, як він робить вправу, і коригував у моменті**.

Усе інше (план, харчування, відновлення) — допоміжне. Без real-time коучингу техніки — це просто ще один лог.

**Ось де всі попередні «AI-тренери» провалилися:** вони рекомендують програми, не коучать виконання.

---

## Slide 04 · Продукт

```
┌─────────────────────────────────────────┐
│  FORM. Особистий тренер у кишені.       │
├─────────────────────────────────────────┤
│  1. Програма під твій сон + HRV         │
│  2. Камер-аналіз техніки (8 рухів)      │
│  3. Голос Victor — без bullshit         │
│  4. Харчування з повагою (anti-ED)      │
│  5. Прогрес у даних, не у фото          │
└─────────────────────────────────────────┘
```

Серцевина: **Victor**. Один голос. 20-річний коучинговий стаж зафіксований у системі. Чотири персони на вибір. Один floor — `no shame, no surveillance, no fake numbers`.

---

## Slide 05 · Технологічна ставка

Те, що відрізняє FORM — **real-time computer vision для коучингу техніки прямо на телефоні**.

| Що зроблено | Як | Стан |
|---|---|---|
| Pose estimation (33 точки) | Apple Vision / MediaPipe | shippable |
| Rep counting | Кутова траєкторія ключових суглобів | shippable |
| Tempo detection | Time between phases | shippable |
| Кут спини / валгус коліна | Custom heuristics на keypoints | research-bet, 6-12 тижнів R&D |
| Голосовий коучинг у моменті | Streaming TTS + cached cue library | shippable |
| Privacy: video stays on device | On-device inference, no upload | shippable |

**Чесна позиція:** real-time form correction на 8 базових рухах — feasible. Все 30+ рухів — research.

---

## Slide 06 · Ринок

**TAM (Global, 2026):**
- Connected fitness apps: ~$11B
- Wearables + fitness analytics: ~$70B (apple Watch + Whoop + Oura + Garmin)
- Pерсональні тренери (digital + IRL): ~$8B

**SAM (наш wedge):**
- Серйозні самостійні лифтери з wearable, 25–45 років, Western markets + UA/EE: ~12M users
- ~3M платять > $20/міс зараз за фітнес-софт

**SOM (3 роки):**
- Цільовий захід 50–100k платних користувачів на 3-й рік
- ARR target ~$15–25M (за середнього $19/міс, 60% річні plans)

Чесно: TAM — оптимістичний фрейм. SAM — те, на що ми всерйоз граємо.

---

## Slide 07 · Бізнес-модель

**Підписка. Без free tier.**

| Plan | Western | UA / EE | Що включає |
|---|---|---|---|
| Trial | 14 днів безкоштовно | 14 днів безкоштовно | Все |
| Monthly | $19/міс | $9/міс | Все |
| Annual | $149/рік (-34%) | $69/рік (-34%) | Все |

**Чому не free tier:** MyFitnessPal модель убиває LTV. Trial + cancel у 3 кліки — наш бар довіри.

**Unit economics (цільові, не доведені):**
- CAC (органіка-first, потім paid): $20–35
- LTV (12 міс avg retention × ARPU): $130–180
- LTV/CAC: 4–7× (target)
- Gross margin: 78–84% (LLM + TTS + payment processing)

---

## Slide 08 · Traction (Чесно)

**Що є зараз (v0.2, травень 2026):**
- Marketing-сайт прототип
- Team-as-code: 14 субагентів описані як expertise
- Бренд-бук, user flows, GTM strategy
- Repo з версіюванням і CI-disциpлінoю

**Що ще НЕ зроблено (open):**
- Жодного коду застосунку
- Жодного користувача
- Жодних інтерв'ю користувачів
- Жодного CV-спайку

**Що буде до раунду:**
- 30 user interviews
- TestFlight closed beta з 200 lifters
- CV-пайплайн на 3 рухах (присід, тяга, жим)
- Перші cohort-метрики (D7, D30)

**Будуємо в публічному.** Build-in-public Twitter + дві блог-статті/тиждень — наша CAC-економія до launch.

---

## Slide 09 · Команда

> На цьому етапі — `1 founder + 14 спеціалізованих AI-агентів`.

Цей рядок звучить як шутка, але він серйозний — і ось чому це працює:

- **product-strategist** + **growth-lead** + **marketing-lead** виконують роботу, на яку інші роблять найм 3-х людей
- **clinical-safety** дає те, чого більшість fitness-стартапів не мають взагалі — раннє ED/medical screening
- **brand-voice** + **design-craft** + **brand-system** тримають consistent висоту, якої важко досягти без сеньйор-дизайнера
- **sports-scientist** + **nutrition-coach** замінюють експерт-консультантів на ранній фазі

**Що шукаємо до Seed:**
- Founding Engineer (mobile-first, iOS + Swift + Vision або React Native + ML Kit)
- Head of Product (sport-tech досвід — Whoop, Strava, Nike Training, RP)
- Optional: Sport Scientist на advisor role

---

## Slide 10 · Чому зараз

**Збіглося три речі:**

1. **Apple Vision і MediaPipe** в 2026 нарешті дають достатньо точні keypoints на consumer-телефонах для коучингу — на iPhone 12+ це 30fps real-time на on-device моделях.

2. **Wearables-проникнення.** Apple Watch + Garmin + Whoop + Oura — це 80M+ активних користувачів у Western markets з HRV-даними, доступними через API. До 2024 такий датасет був недоступний поза премум-апами.

3. **LLM-генеративний коучинг** дозрів. Claude Sonnet 4.6 робить персону тренера, яка не вибиває з характеру і дотримує безпекового флор. Це було неможливо у 2022.

Стартап у 2022: дорого + неможливо.
Стартап у 2025: дорого + можливо.
Стартап у 2026: розумно.

---

## Slide 11 · Ризики (чесно)

| Ризик | Severity | Mitigation |
|---|---|---|
| CV-якість не дотягне до коучингу | HIGH | Шипимо як «beta-помічник» + rep counter; голос+план роблять MVP без CV |
| Whoop / Apple Watch блокують API | MED | Apple HealthKit native, не залежимо від одного партнера |
| ED-related позов | MED | Clinical-safety на найвищому пріоритеті з v0.1, NEDA-стандарти зафіксовано |
| Українська команда + західний ринок | LOW-MED | Distributed-by-default; досвід є; ринок-агностична позиція |
| Paid CAC занадто високий | MED | Paid стоп до M4, Reddit/YouTube-органіка першим |
| Whoop або Future купить нас раніше за вихід | LOW | Це не ризик, це опція. |

---

## Slide 12 · Ask

**Сід-раунд: $1.5–2.5M на 18 місяців runway.**

Куди йдуть гроші:
- $700k — founding engineering team (2 mobile + 1 backend × 18 міс)
- $400k — CV R&D pipeline
- $300k — design + production
- $200k — Apple Developer + cloud + LLM costs
- $250k — first-touch marketing (creator partnerships + content)
- $400k — operations + legal + buffer

**Що ми обіцяємо до Seed-у:**
- 5k активних beta-користувачів
- D30 ≥ 22%
- CV-пайплайн на 3 рухах, NPS ≥ 40
- Перші 500 платних користувачів (без paid)

**Що шукаємо в інвесторі (крім грошей):**
- Distribution-doors на Reddit/YouTube fitness creator-сітку
- Mobile + ML hire-network
- App Store-Apple-relationship (для featuring)

---

## One-pager (для cold-надсилання)

> **FORM** is the first AI personal trainer that watches your form through your phone's camera and adapts your program to your sleep and HRV.
>
> For self-coached lifters who refuse to pay $200/week for a human coach but won't accept Fitbod-generic plans, FORM combines real-time computer-vision coaching, recovery-aware programming, and a voice (Victor) that respects your time.
>
> Pre-product. v0.2 in May 2026. Kyiv-based founder + 14 specialized AI agents on the build team. Closed beta planned Q3 2026.
>
> Raising $1.5–2.5M Seed to ship iOS v1, build the CV pipeline on 3 base lifts, and onboard 5k beta users with D30 ≥ 22%.

---

## FAQ (готові відповіді)

**«Чому ви, а не Whoop / Apple / Nike?»**
Whoop не робить програм. Apple не робить голосу. Nike не робить CV. Кожен з них пропустить це через свою організаційну гравітацію. Стартап з єдиним фокусом на коучингу — швидший.

**«AI вже всюди. Чому це не commodity?»**
Бренд + persona-фіксація + safety-floor — це не LLM-обгортка, це продуктова дисципліна. Голос Victor (з 4 персонами, ED-safe, без surveillance) не клонувати за тиждень.

**«Що, якщо CV-якість буде як у конкурентів — недостатня?»**
Тоді ми залишаємось AI-програмою з найкращим голосом і HRV-інтеграцією. Це усе ще $15-20M ARR-бізнес. CV — апсайд.

**«Чому не B2B?»**
Зробимо. Після PMF в B2C. Корпоративний wellness — добавка, не замість.

**«Ваші цифри traction — переконливо?»**
Ні. Чесно. Раунд — на віру в команду + concept + ринкові тренди, не на traction. Якщо інвестор хоче traction-driven deal, ми не той рейнджер.

---

**v0.1 · травень 2026 · правки приймає `product-strategist`**
