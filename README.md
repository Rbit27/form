# FORM

> Тренер. У кишені.
> AI-тренер з real-time аналізом техніки, адаптивним плануванням і голосом, який поважає тебе.

**Status:** v0.2 · Closed Beta planning · Київ 2026

---

## Структура

```
G:\Клод\form\
├─ index.html              ← marketing-сайт прототип (відкрити в браузері)
├─ onboarding.html         ← onboarding flow прототип, 8 екранів з ED-screening
├─ beta.html               ← beta signup landing з 11-полями форми
├─ README.md               ← цей файл
├─ .claude\
│  └─ agents\              ← команда субагентів
│     ├─ README.md         ← склад команди + workflow
│     ├─ product-strategist.md
│     ├─ sports-scientist.md
│     ├─ nutrition-coach.md
│     ├─ clinical-safety.md       ← HARD VETO
│     ├─ platform-engineer.md
│     ├─ brand-voice.md
│     ├─ design-craft.md
│     ├─ ux-flow.md
│     ├─ brand-system.md
│     ├─ marketing-lead.md
│     ├─ growth-lead.md
│     ├─ content-strategist.md
│     ├─ research-lead.md
│     └─ process-keeper.md
└─ docs\
   ├─ BRAND.md             ← бренд-бук v0.1
   ├─ FLOWS.md             ← user flows v0.1
   ├─ MARKETING.md         ← GTM strategy v0.1
   ├─ APP_STORE.md         ← App Store listing draft v0.1 (5 screenshot-описи, keywords, A/B)
   ├─ PITCH.md             ← investor pitch + one-pager v0.1
   ├─ PERSONAS.md          ← 4 детальні user personas v0.1
   ├─ TECHNICAL.md         ← архітектура + стек + CV-pipeline v0.1
   ├─ FINANCIALS.md        ← unit economics + 3-year model v0.1
   ├─ COMPETITIVE.md       ← deep teardown 8 конкурентів v0.1
   ├─ RESEARCH.md          ← 30-інтерв'ю план + script v0.1
   ├─ LEGAL.md             ← privacy + ToS + GDPR + App Store compliance
   ├─ METRICS.md           ← NSM, funnel, dashboard, A/B test discipline
   ├─ HIRING.md            ← 3 ролі до Seed, interview rubric, compensation
   ├─ PRESS.md             ← press kit, boilerplate, founder bio, FAQ
   ├─ INVESTOR_OUTREACH.md ← target funds, templates, due-dilig pack
   ├─ API.md               ← API design + Postgres schema + rate limits
   └─ FOUNDER.md           ← founder narrative · INTERNAL · not for agents
content\
   ├─ post-01-hrv-is-lying.md            ← blog 1: «HRV тобі бреше» (draft)
   ├─ post-02-what-good-form-means.md    ← blog 2: «Що означає гарна форма» (draft)
   ├─ post-03-we-tested-8-ai-trainers.md ← blog 3: «Ми протестували 8» (draft)
   └─ post-04-sleep-and-strength.md      ← blog 4: «Сон забирає силу» (draft)
programs\
   └─ sample-strength-week.md ← приклад тижня для intermediate
nutrition\
   └─ sample-3day-plan.md     ← приклад 3-денного meal plan (anti-ED)
legal\
   ├─ privacy-policy-draft.md    ← Privacy Policy draft (GDPR-ready)
   └─ terms-of-service-draft.md  ← ToS draft з jurisdiction map
```

---

## Команда (14 субагентів)

### Продукт і безпека (5)
- `product-strategist` — scope, moat, чесність цифр
- `sports-scientist` — програми, HRV, періодизація
- `nutrition-coach` — харчування, ED-обережність
- `clinical-safety` — **HARD VETO** на все шкідливе
- `platform-engineer` — CV, носимі, реальність

### Дизайн і ремесло (3)
- `design-craft` — UI, типографіка, motion
- `brand-system` — бренд-бук, токени, application
- `ux-flow` — флоу, IA, edge cases

### Голос (1)
- `brand-voice` — копірайт, тон Victor, UA/EN

### Зростання (3)
- `marketing-lead` — позиціонування, GTM
- `growth-lead` — funnel, retention, NSM
- `content-strategist` — контент-маркетинг

### Дослідження і координація (2)
- `research-lead` — інтерв'ю, JTBD, валідація
- `process-keeper` — decision log, dependencies

---

## Як працювати

### Запустити цикл `/loop`

Якщо хочеш, щоб я працював над FORM без перерви — викликай:

```
/loop 25m продовжуй роботу над FORM по нашому roadmap
```

Кожні 25 хв я прокидаюся з контекстом, дивлюся що зроблено, продовжую.

Без `/loop` — пишеш «далі», я виконую наступну ітерацію.

### Default pipeline на нову фічу

```
research-lead       → чи є попит? (інтерв'ю)
product-strategist  → scope, moat
sports-scientist / nutrition-coach / platform-engineer
                    → зміст + техніка
clinical-safety     → VETO або fix-list
brand-voice         → копірайт (UA + EN)
clinical-safety     → ще раз, копірайт
design-craft        → UI mock
ux-flow             → флоу + edge cases
brand-system        → застосування токенів
ship
```

### Default pipeline на маркетинг

```
marketing-lead      → позиціонування, аудиторія
content-strategist  → план контенту
brand-voice         → копірайт
clinical-safety     → анти-shame review
growth-lead         → funnel + метрики
design-craft        → візуальна продакшн
```

---

## Що зроблено · v0.2 (травень 2026)

**Маркетинг-сайт прототип** ([index.html](index.html))
- Editorial-естетика: Fraunces + Manrope + JBMono
- 5 інтерактивних екранів продукту в phone-мокапі
- Голос Victor через секцію Voice
- Bento-система фіч
- Секція «Чесно» — що FORM НЕ робить
- Реалістична CTA з UA / Західна ЄС pricing

**Бренд-бук** ([docs/BRAND.md](docs/BRAND.md))
- Назва, лого, токени кольору, типографіка, мотіон
- Voice rules + персони
- Заборони чітким списком

**User flows** ([docs/FLOWS.md](docs/FLOWS.md))
- Onboarding (ED-screening non-skippable)
- Daily ritual, Workout (live CV), Meals (anti-shame), Chat, Progress
- Settings, Cancel-flow без friction
- Push cadence ≤ 2/день, quiet hours 22:00–07:00

**Marketing & GTM** ([docs/MARKETING.md](docs/MARKETING.md))
- Позиціонування і JTBD
- Аудиторія: primary / secondary / tertiary
- Канали ранжовано: Reddit, YouTube, ASO, X. **Paid стоп до M4.**
- 4-фазний GTM roadmap
- Ціна: $9 UA / $19 Western, без free tier

---

## Що далі (відкритий roadmap)

### Близько (наступні 2 тижні)
- [ ] 30 user interviews · `research-lead`
- [x] App Store listing draft (5 screenshots, опис) · `marketing-lead` — [`docs/APP_STORE.md`](docs/APP_STORE.md) · v0.3.1
- [x] Onboarding HTML-прототип з ED-screening · `ux-flow` + `clinical-safety` — [`onboarding.html`](onboarding.html) · v0.3.1
- [ ] Прес-кіт сторінка · `content-strategist`
- [ ] Mobile app static screens (Figma або HTML) · `design-craft`

### Середньо (M2)
- [ ] Стек-вибір і репозиторій · `platform-engineer`
- [ ] CV pose pipeline спайк (MediaPipe vs Vision на iOS) · `platform-engineer`
- [ ] Newsletter setup + перші 2 пости · `content-strategist`
- [ ] Beta sign-up форма + auto-respond email · `growth-lead`

### Далеко (M3+)
- [ ] TestFlight closed beta · 200 users
- [ ] Cohort analysis · перші реальні цифри
- [ ] Paid тест якщо unit econ працює

---

## Версіювання

Поточна версія: див. [`VERSION`](VERSION) (single source of truth).

Формат: **`MAJOR.MINOR.PATCH`** (semver-light).
- **PATCH** — кожна cloud-ітерація. ~12/добу на 2-годинному циклі.
- **MINOR** — нова фіча, новий розділ доків.
- **MAJOR** — брейкінг, pivot бренду. Не до релізу.

Кожен bump = коміт + git tag + запис у [`CHANGELOG.md`](CHANGELOG.md).
Стандарти комітів — у [`CONTRIBUTING.md`](CONTRIBUTING.md).

**Дивитися історію змін:**
- [CHANGELOG.md](CHANGELOG.md) — людиночитабельно
- https://github.com/Rbit27/form/releases — теги і релізи
- https://github.com/Rbit27/form/commits/main — коміт-стрім

---

## Принципи (read this before contributing)

1. **Жодних вигаданих цифр.** Цілі — окей. Результати — тільки коли є дані.
2. **`clinical-safety` має право зупинити будь-що.** Не консультант — блок.
3. **Якщо фразу не можна сказати другу в гімі — її не каже Victor.**
4. **Не пропускай два поспіль** — це принцип і для команди.

Повні стандарти інженерної дисципліни — [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

**Lead:** Claude (Opus 4.7) · координує команду
**Cloud iterator:** Claude (Sonnet 4.6) · кожні 2 години через [routine](https://claude.ai/code/routines/trig_01FYEjmyX5cwtRZtEWXWuLtf)
**Owner:** Rbit27 (George)
**Last update:** 14 травня 2026 · v0.3.2
