# FORM · Investor Pipeline & Relationship Management v0.1

> Як перетворити cold-контакти на warm-стосунки, а warm-стосунки — на term sheets.
> Власник: founder. Суміжно: INVESTOR_OUTREACH.md (цілі + templates), SEED_NARRATIVE.md (pitch script), DD_FAQ.md (due diligence), DATA_ROOM.md (materials).

---

## 1 · Що цей документ, а що ні

**Це є:**
- CRM-схема для Google Sheets / Notion
- Визначення стадій воронки
- Cadence правила (хто коли з'являється)
- Ознаки справжнього інтересу vs. ввічливого "no"
- Пост-мітинг ритуал
- Механіка закриття раунду

**Це не є:**
- Список цільових інвесторів → INVESTOR_OUTREACH.md §2
- Cold-email шаблони → INVESTOR_OUTREACH.md §4
- Pitch-скрипт для call → SEED_NARRATIVE.md
- Due-diligence Q&A → DD_FAQ.md
- Матеріали дата-руму → DATA_ROOM.md

---

## 2 · Pipeline: 7 стадій

| Stage | Label | Визначення |
|---|---|---|
| **S0** | Listed | Ідентифікований, контакту ще не було |
| **S1** | Contacted | Відправлено cold email або запит на intro |
| **S2** | Responded | Відповіли. Немає зустрічі, але діалог є |
| **S3** | Meeting | Зустріч відбулась (Zoom / in-person) |
| **S4** | Evaluating | Активно розглядають. DD-питання пішли |
| **S5** | Term Sheet | Отримали term sheet або verbal commitment |
| **S6** | Closed / Declined | Угода закрита або офіційний pass |

**Правило 1.** Жоден контакт не живе в S0 без терміну наступної дії.
**Правило 2.** S2+ без наступної дії у наступні 7 днів = staleness flag.
**Правило 3.** Ніколи не переходь у S4 без того, щоб вони першими задали DD-питання. Push = відлякує.

---

## 3 · CRM-схема (Google Sheets)

### Обов'язкові колонки

| Колонка | Тип | Приклад |
|---|---|---|
| `fund_name` | text | Earlybird Venture Capital |
| `contact_name` | text | Jan Müller |
| `role` | text | General Partner |
| `stage` | S0–S6 | S2 |
| `thesis_fit` | High / Med / Low | High |
| `intro_type` | warm / cold / inbound | warm |
| `intro_source` | text | Vlad @ OTB |
| `first_contact_date` | date | 2026-06-01 |
| `last_contact_date` | date | 2026-06-08 |
| `next_action` | text | Send deck after call |
| `next_action_date` | date | 2026-06-10 |
| `check_size_range` | text | $100–500K |
| `portfolio_conflict` | yes / no / unknown | no |
| `conviction_signal` | text | Asked for cap table |
| `notes` | text | Free-form field |
| `status_color` | 🟢🟡🔴 | 🟢 |

### Кольорова логіка

- 🟢 На треці. Наступна дія є, дата не минула.
- 🟡 Увага. next_action_date сьогодні або завтра.
- 🔴 Прострочено або заморожено >14 днів без дії.

### Додаткові вкладки

- **Warm intro tracker** — хто може інтро кому, статус запиту.
- **Meeting log** — дата, хто був, короткий recap, наступний крок.
- **Term sheet tracker** — умови, дедлайн підпису, коментарі counsel.

---

## 4 · Cadence правила

### S0 → S1 (первинний контакт)

- Warm intro > cold email завжди. Конверсія тепла x3–x5.
- Запит на intro: дай мережі 5 днів. Якщо тиша — один soft follow-up.
- Cold email: → шаблони у INVESTOR_OUTREACH.md §4.
- Максимум 5 нових S1 на тиждень. Якість > кількість.

### S1 → S2 (якщо не відповіли)

- Follow-up після 7 робочих днів. Один раз. Коротко.
- Якщо тиша після follow-up → переводь у S0 зі статусом "quiet".
- Не переслідуй. Репутація важливіша за цей check.

### S2 → S3 (scheduling)

- Запропонуй конкретні таймслоти протягом 48 годин після відповіді.
- Використовуй Calendly або 2-3 варіанти у тілі листа.
- Підтвердь за 24 год до зустрічі.

### S3 → S4 (post-meeting)

- Recap email протягом 2 годин після зустрічі (→ шаблон у §7).
- Відправ deck + data room посилання якщо вони попросили.
- Не проси рішення. Запитай: «Що потрібно для наступного кроку?»
- Якщо вони мовчать 7 днів → один short follow-up.

### S4 → S5 (evaluating)

- Відповідай на DD-питання протягом 24 годин.
- Один weekly check-in максимум — і тільки якщо є update.
- Дай їм вести темп. Тиск у S4 вбиває угоди.
- Запропонуй reference call з ким, кому ти довіряєш.

### S5 → S6 (closing)

- Term sheet → підключи counsel відразу (LEGAL.md).
- Переговори: SAFE з valuation cap — стандарт для pre-seed.
- Deadline підпису: 2 тижні є норма. Більше = втрата моменту.
- Після підпису → notify всіх S3/S4 що раунд закривається.

---

## 5 · Warm intro: як просити

### Кого просити

Кращі джерела інтро для FORM:
- Founders у їхньому портфелі (ex. попросити у них intro до GP)
- Advisors та mentors з fitness/health фону
- Колишні колеги-засновники
- Angel investors, що вже вклали

### Шаблон запиту на intro (до мережі)

```
Subject: Quick ask — intro до [FUND]?

[Ім'я], —

Я знаю ти знаєш команду в [Fund]. Ми підіймаємо pre-seed для FORM —
AI-тренер з комп'ютерним зором (форма штанги + адаптація до HRV).

Якщо ти думаєш що є fit — буду вдячний за intro. Якщо ні — теж окей,
скажи чесно.

Текст для форварду + один-пейджер — нижче.

[Підпис]
```

### Forwardable blurb (англ.)

```
[Ім'я], — forwarding this at George's request. He's building FORM,
an AI fitness coach using computer vision for real-time form feedback,
adapting workouts to sleep and HRV. Pre-product, pre-seed. Ukraine-based
founder, serious discipline. Might be worth 20 minutes.

—[Ім'я того хто інтро]
```

---

## 6 · Prep-ритуал перед зустріччю (30 хвилин)

**За 24 год до зустрічі:**
- [ ] Прочитай 3 останні публічні пости / tweets контакту
- [ ] Перевір 2–3 портфельних компанії фонду (особливо fitness/health/AI)
- [ ] Визнач їх thesis одним реченням — запиши
- [ ] Перевір чи є конфлікти у портфелі (competitor)

**Безпосередньо перед:**
- [ ] Відкрий SEED_NARRATIVE.md §2 (opening statement)
- [ ] Відкрий DD_FAQ.md §3 (Financials) на випадок питань
- [ ] Підготуй 3 "honest unknowns" — краще сказати першим ніж щоб знайшли самі
- [ ] Склади тихе місце, camera включена, headphones

**Mindset:**
- Питай більше ніж говориш (особливо перші 5 хвилин)
- Мета не «продати» — мета зрозуміти чи є fit
- Якщо не fit — дізнайся чому. Це research.

---

## 7 · Post-meeting ритуал

### Recap email (надіслати протягом 2 годин)

```
Subject: FORM — recap від [дата]

[Ім'я],

Дякую за час сьогодні. Коротко те що ми обговорили:

• [Тезис 1 — те що їх зацікавило]
• [Тезис 2 — відкрите питання яке лишилось]
• [Наступний крок — конкретно, з датою]

[Якщо вони попросили матеріали] — deck і data room: [посилання]

Напиши якщо є питання.

George
```

### Що логувати у CRM після зустрічі

- Дата і тривалість
- Хто був з їхньої сторони
- Їх ключові питання (дослівно — важливо)
- Сигнали інтересу (→ §8)
- Що обіцяли надіслати / зробити
- Наступна дія + дата

---

## 8 · Ознаки справжнього інтересу

### Зелені прапори (genuine interest)

- Запитали cap table або ownership structure
- Назвали конкретну суму / розмір чека
- Запропонували reference call з портфельною компанією
- Самі ініціювали наступну зустріч
- Ввели юриста або associate у ланцюжок листів
- Попросили ексклюзивний доступ до дата-руму
- Запитали про timeline раунду ("коли ви закриваєтесь?")

### Жовті прапори (wait and see)

- "Дуже цікаво, залишимось на зв'язку"
- "Надішліть нам deck коли будете готові"
- Тільки junior associate на дзвінку — GP не з'явився
- Не задали жодного DD-питання після другої зустрічі

### Червоні прапори (polite no)

- Затягування відповіді >14 днів без конкретної причини
- "Зараз не наш timing" (без конкретного trigger event)
- Переключились на generic feedback про ринок
- Попросили бачити метрики але ще нема продукту (= відклали без reject)

**Важливо:** ввічлива відмова краща за підвішений стан. Якщо бачиш червоні прапори — запитай прямо:
```
"Буду вдячний за прямий фідбек — чи є причина через яку FORM
не підходить для вашого фонду? Це допоможе нам краще фокусуватись."
```

---

## 9 · Коли рухатися далі

### Парк (не відмовляй, але не вкладай час)

Умова парку: 2 follow-up без відповіді або >1 місяць без руху.

Дія: Перевести у S0 з позначкою "parked", додати в monthly investor update (→ INVESTOR_UPDATE_TEMPLATE.md). Нехай бачать прогрес passively. Повернутись при наступному milestone (TestFlight launch, перший retention cohort, term sheet від іншого).

### Hard decline

Якщо офіційний pass — лаконічна відповідь, жодних аргументів:
```
"Ціную прямість. Якщо в майбутньому щось зміниться — буду радий оновити."
```

Запиши причину відмови у notes — це product research.

---

## 10 · Monthly investor updates (до raise)

Ключовий інструмент: будувати стосунки до того як потрібні гроші.

**Кого включати:** S2+ у pipeline + angel advisors + warm intro sources.

**Частота:** щомісяця, ~300 слів, без spin.

**Структура** (→ повний шаблон у INVESTOR_UPDATE_TEMPLATE.md):
- Прогрес цього місяця (факти)
- Що вчинили (вирішення, зміни)
- Де потрібна допомога (конкретно)
- Наступний milestone + дата

**Правила:**
- Ніколи не приховуй погані новини. Investors cite "surprised by bad news" as #1 red flag post-investment.
- Один чіткий ask per update ("знаєш когось у Earlybird?")
- Без вигаданих метрик (Принцип #1)

---

## 11 · Закриття раунду: механіка

### Коли оголошувати що раунд закривається

Тільки якщо є real commitment (verbal або signed). Штучний тиск ("closing in 2 weeks!") без anchor investor = credibility damage.

### Якщо є lead investor

1. Погодь економіку (valuation cap, discount, pro-rata rights)
2. Підклич counsel (LEGAL.md §X) — не підписуй сам
3. Announce lead → create FOMO для S3/S4 pipeline
4. Дай решті 7–14 днів на рішення
5. Close з wire + підтверджуй електронно

### SAFE vs. Priced Round

Pre-seed = SAFE (YC standard). Причини:
- Швидше (тижні vs. місяці)
- Дешевше юридично (~$3–5K vs. $30–50K)
- No dilution event до series A
- Valuation cap важливіша ніж розмір чека при цьому розмірі

**Наші параметри (референс, не зобов'язання):**
- Target raise: $1.5–2.5M (README + FINANCIALS.md)
- Instrument: SAFE
- Valuation cap: TBD після research (не вигадуй цифру — поговори з counsel)
- Discount: стандарт 20%

### Після закриття

- Data room залишається відкритим для signed investors
- Board observer rights: якщо >$500K від одного фонду — обговори умови
- Перший investor update — через 30 днів після close

---

## 12 · Anti-patterns (що не робити)

| ❌ Anti-pattern | ✓ Альтернатива |
|---|---|
| Питати "що ти думаєш?" без конкретики | "Що потрібно для наступного кроку з вашої сторони?" |
| Вигадувати метрики ("ми растемо 20% WoW") | "Ми pre-revenue, але ось наш plan" |
| Pitch всіх підряд | 15 якісних > 150 random |
| Ховати конкурентів | Назви їх першим і поясни диференціацію |
| Тиснути після term sheet | Дай розумний дедлайн, потім тиша |
| Приймати bad terms щоб закрити швидше | Поганий investor = pain на 5+ років |
| Розповідати умови одного investor іншому | Завжди NDA / discretion |

---

## 13 · Перший раунд: реалістичний timeline

```
M0 (зараз)
├── Build warm network + research targets (INVESTOR_OUTREACH.md)
├── Start monthly investor updates → S2+ pipeline
└── Підготуй матеріали: DATA_ROOM.md tier 1-2 ready

M1-M2
├── 30+ user interviews completed (→ RESEARCH.md)
├── First 5-10 warm intro meetings
└── Founder narrative sharp (→ SEED_NARRATIVE.md)

M3 (post-FE hire)
├── TestFlight alpha з 20-50 users
├── First retention data
├── Investor pipeline: S3/S4 мають retention signal
└── Warm intro to anchor investor

M4-M5
├── Soft close з anchor (2+ firms)
├── Round fills with S3/S4 relationships
└── Announce close + next milestone

Note: Timeline залежить від FORM-specific milestones, не від stock calendar.
Raise when you have something to show, not когда running low on runway.
```

---

## 14 · Інтеграція з іншими документами

| Коли | Документ |
|---|---|
| Потрібен pitch script для call | → SEED_NARRATIVE.md |
| Отримали DD-питання | → DD_FAQ.md |
| Запросили матеріали | → DATA_ROOM.md |
| Потрібен target fund список | → INVESTOR_OUTREACH.md |
| Час писати monthly update | → INVESTOR_UPDATE_TEMPLATE.md |
| Term sheet отримано | → LEGAL.md |
| Питання про фінанси | → FINANCIALS.md |

---

## 15 · Open questions

- [ ] Який валuation cap є розумним для pre-seed на цьому ринку? → Питання для counsel + comparable rounds research
- [ ] Чи варто pitchати акселератори (YC, Antler) паралельно? → Обговорити з product-strategist
- [ ] Як будувати warm network у Earlybird / OTB без existing connections? → Research intro paths через LinkedIn 2nd-degree
- [ ] SAFE vs. convertible note для Ukrainian entity? → LEGAL.md followup

---

*Оновлювати після кожного раунду: pre-seed → seed → series A.*
*Власник: founder. Суміжні агенти: product-strategist (моат, claims), process-keeper (CRM hygiene).*

**v0.1 · травень 2026**
