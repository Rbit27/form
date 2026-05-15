# FORM · Growth Loops v0.1

> Механіка органічного і вірального росту FORM — від beta до публічного запуску.
> Власник: `growth-lead` + `product-strategist`. Клінічний review: `clinical-safety`. Правки через PR.

Читай разом з:
- [`docs/METRICS.md`](METRICS.md) — W-ACSU, AARRR, funnel targets
- [`docs/BETA_RECRUITING.md`](BETA_RECRUITING.md) — канали рекрутингу 200 beta-юзерів
- [`docs/RETENTION_PLAYBOOK.md`](RETENTION_PLAYBOOK.md) — D0/D7/D30/D90 retention windows
- [`docs/ANALYTICS_SETUP.md`](ANALYTICS_SETUP.md) — PostHog events для tracking loops
- [`docs/PRODUCT_SPEC.md`](PRODUCT_SPEC.md) — технічна база для sharing механіки

---

## 0 · Засади і обмеження

### Навіщо growth loops на pre-product стадії

Мета не «запустити virality». Мета — **продумати механіку до того, як Founding Engineer пише перший рядок коду**. Кожен growth loop потребує конкретних product hooks. Якщо їх не заплановано на рівні архітектури, додати post-factum = рефакторинг.

### Обмеження, які не обговорюються

**DEC-002 — No in-app social features у v1.0.**
Жодних leaderboards, друзів, публічних профілів, порівняльних екранів. Growth loops у FORM — це **sharing назовні** (Instagram, Twitter, Telegram), не всередину продукту.

**DEC-016 — No free tier.**
Loops не пропонують безкоштовний доступ. Referral дає **trial-period** (7 або 14 днів), не постійний free plan.

**clinical-safety HARD VETO на весь sharing-контент:**
- ❌ Вага, BMI, будь-яке body metric
- ❌ Before/after фото
- ❌ Калорії у share-контенті
- ❌ Порівняння з іншими юзерами
- ✅ Сила (кг/lb), об'єм тренування (sets × reps), form score
- ✅ Streak (кількість тренувань), не «скільки втратив/набрав»
- ✅ Victor-цитати (тональні, не prescriptive)

Будь-яка нова share-механіка проходить через `clinical-safety` до імплементації.

---

## 1 · Карта петель

```
[NEW USER]
    │
    ▼
Loop 5 (Beta FOMO) ──→ Waitlist ──→ [BETA INVITE]
                                           │
                                           ▼
                                    [ONBOARDING]
                                           │
                                           ▼
Loop 1 (Form Check Share) ◄── [FIRST COACHED SESSION]
    │                                      │
    ▼                                      ▼
[Organic                         Loop 2 (Streak Share)
 Acquisition]                              │
    │                                      ▼
    └───────────────────────────→ [RETENTION]
                                           │
                                           ▼
                              Loop 3 (Victor WOM)
                                           │
                                           ▼
                              Loop 4 (Referral Program)
                                           │
                                           ▼
                                    [NEW INSTALL]
```

---

## 2 · Loop 1 — Form Check Share (Основна петля)

### Механіка

Victor аналізує техніку (CV pipeline) і дає конкретний feedback на lift. Юзер отримує **form score + coaching cue**, яких не дає жоден інший app. Ця унікальність — природній тригер для шерингу.

```
CV coaching → Form Score + specific cue
    → Share card (auto-generated) → External social
    → Curiosity → Install → Onboarding
```

### Що шерується

**Share card — генерується in-app, single tap:**

```
┌────────────────────────────────────────┐
│  FORM  ·  Squat · 23 jan              │
│                                        │
│  Form score: ████████░░ 8/10          │
│                                        │
│  «Глибина окей. Коліна трохи всередину │
│   під навантаженням. Почни з 70%      │
│   і відпрацюй паузу.»                  │
│                          — Victor      │
│                                        │
│  form.coach · Beta                     │
└────────────────────────────────────────┘
```

**Що НЕ на карточці:** вага тіла, калорії, BMI, порівняння з іншими.

### Тригер

- Victor delivery: coaching cue доставлено → `share_prompt_shown` event (PostHog)
- Не автоматично, не нав'язливо. **Один soft prompt після сесії**, легко dismiss.

### Цикл

| Метрика | Beta ціль | Launch ціль |
|---|---|---|
| Сесій з share prompt | 100% coached sessions | 100% |
| Share rate (prompt → share) | Ціль: ≥ 15% | ≥ 20% |
| Share → Install (attribution) | Ціль: ≥ 3% clicks | ≥ 5% |

> Цілі, не результати — валідація в beta M3.

### Клінічний guard

`clinical-safety` review обов'язковий для:
- Кожного шаблону share card до випуску
- Будь-якої зміни coaching cue що йде на карточку
- Copy навколо share prompt (Victor tone, не «похвались»)

---

## 3 · Loop 2 — Streak Share (Retention → Acquisition)

### Механіка

Тижневий streak (кількість coached sessions за 7 днів) стає приводом для sharing після досягнення milestones.

```
Streak milestone hit → Celebration UI → Optional share → External
    → «Що це за app?» → Waitlist/Install
```

### Milestone тригери

| Milestone | Коли | Share content |
|---|---|---|
| Перша coached session | D0 | «Перше тренування з FORM» |
| 5 поспіль тренувань | D7-D14 | «5 coached sessions» |
| Місяць стажу з FORM | D30 | «30 днів, [X] coached sessions» |
| 100 coached sessions | Variable | «100. Victor коментує.» |

### Принципи

- **Святкування, не тиск.** Немає «не пропускай streak!» у share copy.
- **Victor-тон у підпису.** Цитата з тренування, якщо можлива (consent-based).
- **Opt-in.** Milestone → пропозиція. Dismiss = no prompt for N днів.

### Реалізація

Streak milestone event → `streak_milestone_reached` (PostHog) → celebration modal з share card + dismiss. Share target: Instagram Stories, Twitter, Telegram.

---

## 4 · Loop 3 — Victor WOM (Word of Mouth через якість)

### Механіка

Найтихіший і найпотужніший loop. Victor говорить речі, які запам'ятовуються і які розповідають іншим. Не механіка в traditional sense — це brand equity loop.

```
Victor unbelievably specific cue → User tells gym friend
    → «Звідки ти це знаєш?» → «Є такий app...»
    → Органічна рекомендація
```

### Чому це loop, а не просто «гарний продукт»

Конкретика coaching cue = conversation piece. «AI-тренер сказав мені що моєліве коліно collapsing під навантаженням у третьому підході» — цього не забудеш і розкажеш.

### Що потрібно від продукту

- Victor cues мають бути **конкретними і несподіваними**, не generic (не «гарна техніка», а «коліно 3° на 2-й rep»)
- Feedback per-rep, не post-workout summary
- Різниця між «Victor сказав щось розумне» і «Victor побачив те, чого я сам не бачив» — ось що розповідають друзям

### Метрика

- NPS задає питання «Ви б рекомендували FORM другу?» — baseline у Wave 1 (M3)
- Referral source attribution: «Як ти дізнався про FORM?» у onboarding → track «friend told me» %
- Ціль M6: ≥ 30% organic installs атрибутовані до word of mouth

---

## 5 · Loop 4 — Referral Program (Formal, post-launch)

### Статус

**Не будується у v0.2 beta.** Планування зараз, імплементація M4-M5 (перед публічним launch).

### Механіка

```
Existing subscriber → Invite link (unique) → Friend installs + subscribes
    → Referrer отримує reward → Referrer більш залучений → Referrer ділиться знову
```

### Reward-механіка (DEC-016 compliant)

Не безкоштовний tier. Варіанти:

| Reward | Для referrer | Для friend |
|---|---|---|
| Trial extension | +30 днів до наступного billing | 14-day free trial |
| Feature unlock | Early access до нової фічі (CV+ lift) | — |
| Credit | $5-10 account credit | — |

> Вибір reward-моделі = окреме рішення після beta cohort data. Тут тільки framework.

### Технічна вимога до FE

- Unique referral link per user (короткий, shareable)
- Attribution tracking: install → subscription з referral code
- Reward automation: manual у beta, automated у launch

### Anti-abuse guards

- 1 reward per referral link per new user (не chain)
- Referral link expires після 90 днів
- New user must complete onboarding + first coached session для activation reward
- Rate limit: max 10 referrals per user per month у beta (fraud prevention)

---

## 6 · Loop 5 — Beta Exclusivity FOMO (Pre-product)

### Статус

**Активний зараз (pre-launch, Q2-Q3 2026).** Це єдиний growth loop, що запускається до MVP.

### Механіка

```
Awareness (X/Twitter, LinkedIn, Reddit) → beta.html форма
    → «Invite a friend → рухайся вище у черзі»
    → Органічний поширення
```

### Реалізація через beta.html

Поточний `beta.html` збирає заявки. Для FOMO-loop потрібно додати (future sprint):

```
After submission:
  «Твоя позиція у черзі: #[N]
   Запроси друга → піднімайся вище»
   [Unique share link для конкретного аплікента]
```

### Copy (brand-voice tone, EN/UA)

```
EN:
  «You're #247 on the FORM waitlist.
   Invite one serious lifter → move up.
   They need an iPhone 13+ and to train 3x/week minimum.»

UA:
  «Ти #247 у черзі FORM.
   Запроси одного, хто тренується серйозно → піднімись.
   Потрібен iPhone 13+, ≥ 3 тренувань на тиждень.»
```

**Чому «серйозного» у copy:** Зберігає якість waitlist. Не приваблює casual users яким не потрібен coached AI trainer.

### Метрики

| Метрика | Ціль |
|---|---|
| Waitlist → Share rate | ≥ 25% |
| Share → New signup conversion | ≥ 10% |
| Viral coefficient K (через цей loop) | ≥ 0.3 у beta |

> K ≥ 1.0 = viral product. K = 0.3 означає кожні 10 заявок генерують 3 додаткових. Реалістична ціль для niche fitness app у закритій beta.

---

## 7 · Клінічна безпека — Повний перелік обмежень

> Цей розділ = обов'язковий checklist перед ship будь-якої growth механіки.

### ✅ Allowed у share content

- Lift name (Squat, Deadlift, Bench)
- Form score (числовий, 0-10)
- Victor coaching cue (per-lift, technical, non-body)
- Session count / streak count
- Volume (sets × reps), не absolute weight
- Date of session

### ❌ Blocked у share content

| Category | Приклади | Чому |
|---|---|---|
| Body metrics | Вага тіла, BMI, відсоток жиру | ED тригер |
| Caloric data | Калорії спалено, дефіцит | ED тригер |
| Comparison | «Краще ніж X% users» | Shame-adjacent |
| Before/after | Будь-яке before/after | Body image harm |
| Appearance copy | «Тіло», «схуднення», «форма тіла» | ED тригер |
| Progress-as-appearance | «Видно результат» у share context | Body image |

### Veto процедура

1. Будь-яка нова share механіка → PR з `clinical-safety` label
2. `clinical-safety` agent review (або founder + checklist вище)
3. Якщо хоч один ❌ item присутній → hard block, не «можна з застереженнями»
4. Документувати в DECISION_LOG.md якщо щось граничне

---

## 8 · Пріоритет імплементації для Founding Engineer

| Priority | Loop | Sprint | Складність |
|---|---|---|---|
| P0 | Loop 5 (Beta FOMO) | Pre-launch web sprint | Low (HTML/JS на beta.html) |
| P1 | Loop 1 (Form Share Card) | M2 Sprint 3 | Medium (card generation, share sheet) |
| P2 | Loop 2 (Streak Share) | M2 Sprint 4 | Low-Medium (milestone events + share) |
| P3 | Loop 3 (Victor WOM) | Ongoing | No code — product quality |
| P4 | Loop 4 (Referral Program) | M4-M5 | High (attribution, reward automation) |

**P0 не потребує iOS:** beta.html waitlist referral = web task, можна зробити до FE hire.

---

## 9 · Analytics events (PostHog)

> Повна інструментація у `docs/ANALYTICS_SETUP.md`. Тут — тільки growth-specific events.

| Event | Loop | Properties |
|---|---|---|
| `share_prompt_shown` | L1, L2 | loop_type, session_id, trigger |
| `share_initiated` | L1, L2 | loop_type, platform (Instagram/Twitter/Telegram/Other) |
| `share_completed` | L1, L2 | loop_type, platform |
| `share_dismissed` | L1, L2 | loop_type, dismiss_count |
| `referral_link_clicked` | L4, L5 | source, referrer_id (hashed) |
| `referral_converted` | L4 | referrer_id (hashed), days_to_convert |
| `waitlist_shared` | L5 | position_at_share, platform |
| `milestone_reached` | L2 | milestone_type, coached_sessions_count |

---

## 10 · Open questions (v0.1)

1. **Beta FOMO loop (Loop 5):** Чи робимо waitlist position visible у beta.html зараз, або чекаємо FE? → Якщо HTML/JS, можна зробити без iOS. Product-strategist + growth-lead decision.

2. **Referral reward model:** Trial extension vs. credit vs. feature unlock → потребує beta cohort data (M3+). Не вирішувати зараз.

3. **Form share card design:** Static template vs. dynamic generation → якщо dynamic, потрібен serverless endpoint. platform-engineer input needed.

4. **Attribution для Loop 3 (WOM):** Як відрізнити «friend told me» від «побачив у соцмережах» — onboarding survey питання (research-lead to design).

5. **K-factor measurement:** Коли у beta є N installs, як вимірюємо viral coefficient з confidence? → analytics-setup потребує referral attribution від day one.

---

**v0.1 · травень 2026 · live document — update після beta M3 з реальними даними**

**Власник:** `growth-lead` + `product-strategist`
**Клінічний review:** `clinical-safety` (HARD VETO)
**Пов'язані рішення:** DEC-002 (no in-app social), DEC-016 (no free tier), DEC-029
