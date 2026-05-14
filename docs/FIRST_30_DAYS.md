# FORM · First 30 Days Execution Playbook v0.1

> Перетворює 40+ стратегічних доків у конкретний план дій.
> Власник: founder. Синтез: `process-keeper` + `growth-lead`.
> Читати разом із: DEPLOYMENT.md, RESEARCH.md, INVESTOR_OUTREACH.md, CONTENT_CALENDAR.md.

---

## TL;DR

Чотири тижні. Чотири фокуси.

| Тиждень | Фокус | Критерій успіху |
|---|---|---|
| **W0** (зараз) | Інфраструктура | Domain + accounts + tools налаштовано |
| **W1** | Перша публічна присутність | Сайт живий, Twitter активний, newsletter форма готова |
| **W2** | Research sprint | 10 перших інтерв'ю заброньовано та проведено |
| **W3** | Investor outreach + наймання | 10 warm intro email надіслано, FE JD опублікована |
| **W4** | Синтез + наступна фаза | Персони оновлені, 10 нових інтерв'ю, beta waitlist росте |

Цей документ — checklist, не стратегія. Стратегія вже є в інших доках.

---

## Передумови (зроби до W0)

- [ ] Creditcard для платних інструментів ($50–100/міс: Beehiiv, Zoom, Otter.ai)
- [ ] Особистий GitHub Copilot або Claude Pro account (для швидкої роботи)
- [ ] Ноутбук з нормальним інтернетом + запасний hotspot
- [ ] 3 year .coach domain budget (~$50 via Namecheap чи Cloudflare Registrar)
- [ ] 20 годин вільних на тиждень (мінімум для цього плану)

---

## WEEK 0 · Інфраструктура (1–3 дні)

### День 1 · Domain + DNS + Email

**1. Зареєструй form.coach**

```bash
# Cloudflare Registrar (рекомендовано — без markup + автоматичний DNS)
# Йди на: https://dash.cloudflare.com → Domain Registration → search "form.coach"
# ~$50/рік для .coach TLD
```

**2. Налаштуй DNS для Cloudflare Pages**

```
# Після реєстрації — автоматично. Або якщо domain у Namecheap:
# Зайди в Namecheap → Manage Domain → Custom DNS → вкажи Cloudflare nameservers
# Cloudflare: ivan.ns.cloudflare.com + mia.ns.cloudflare.com (показує у dashboard)
```

**3. Forwarding email** (до реального inbox)

Використай Cloudflare Email Routing (безкоштовно):
- `hello@form.coach` → твій Gmail/Fastmail
- `hiring@form.coach` → твій Gmail/Fastmail
- `press@form.coach` → твій Gmail/Fastmail

Повний гайд деплою: [`docs/DEPLOYMENT.md`](DEPLOYMENT.md).

---

### День 2 · Deploy marketing site

```bash
# Детальна інструкція у docs/DEPLOYMENT.md
# Короткий шлях:

git clone https://github.com/Rbit27/form
cd form

# Cloudflare Pages dashboard → Create a project → Connect to GitHub
# Repository: Rbit27/form
# Build command: (порожньо — статичний сайт)
# Build output: / (root)
# Production branch: main

# Після першого deploy:
# Settings → Custom Domains → Add form.coach
```

**Чекліст після deploy:**
- [ ] form.coach відкривається в браузері
- [ ] Всі 10 HTML сторінок доступні (index, beta, pricing, about, faq, screens, onboarding, settings, press, workout-live)
- [ ] sitemap.xml доступний: form.coach/sitemap.xml
- [ ] robots.txt доступний: form.coach/robots.txt
- [ ] HTTPS working (автоматично через Cloudflare)

---

### День 3 · Accounts + Tools

**Соціальні аккаунти (заводь у такому порядку):**

| Platform | Handle | Мета |
|---|---|---|
| X / Twitter | `@form_coach` | Первинний build-in-public канал |
| Reddit | u/form_coach або founder handle | Для recruiting + community |
| LinkedIn | George + FORM Company page | Для FE hiring + investor |
| HN | founder account (якщо немає) | Для Show HN пост |

**Інструменти:**

| Tool | Plan | Для чого |
|---|---|---|
| [Beehiiv](https://beehiiv.com) | Launch (безкоштовно до 2.5k) | Newsletter |
| [Otter.ai](https://otter.ai) | Basic free | Транскрипти інтерв'ю |
| [Zoom](https://zoom.us) | Free (40 min limit) | Remote interviews |
| [Notion](https://notion.so) | Free | Interview notes tracker |
| Google Sheets | Free | Investor CRM (поки Notion) |

**Beehiiv setup (30 хв):**
1. Signup → Publication name: «FORM · Notes from the gym floor»
2. Import your custom domain (form.coach/newsletter або newsletter.form.coach)
3. Налаштуй базовий template: заголовок, підписка-форма, unsubscribe
4. Встав форму підписки на index.html (якщо ще не є — додай Beehiiv embed)
5. Деталі: [`docs/NEWSLETTER.md`](NEWSLETTER.md)

---

## WEEK 1 · Перша публічна присутність

### Понеділок · Перші твіти

**Запуск аккаунту @form_coach:**

Перший твіт (pinned):

```
Building @form_coach — an AI personal trainer that watches your squat form 
through your phone camera and adapts your program to your HRV.

No app yet. No team yet. Building it publicly from Kyiv.

Follow along →
```

Другий твіт (наступного дня):

```
Why does every AI trainer give you a plan but none of them watch how you actually move?

That's the gap FORM fills.
```

Третій:

```
Pre-product. 0 users. 0 code shipped.

What we do have:
- A pitch (docs.form.coach coming soon)
- A brand
- 9 blog drafts
- 14 AI agents on the build team

Raising seed to hire the first engineer. DMs open.
```

**Кого фоловь першим (fitness + indie + AI):**
- r/weightroom mods on Twitter
- Fitness tech journalists: @[знайди у fitness tech beat]
- Founders з fitness-tech: Whoop founder, Strava founders
- AI-fitness ентузіасти: шукай «AI fitness» у Twitter

---

### Середа · Перший Show HN (опційно)

Якщо є 10+ хв — постав на HN:

```
Show HN: FORM – AI personal trainer with real-time CV coaching (pre-product)

We're building an iOS app that uses computer vision to coach your squat/deadlift/bench
form in real-time, adapts your program to HRV, and gives you a consistent training
voice (Victor) that isn't generic.

No code shipped yet. We have: a full pitch, marketing mockups, 9 blog posts,
and 14 specialized AI agents on the build team. Raising $1.5-2.5M seed.

Docs: github.com/Rbit27/form
Landing: form.coach (if deployed)

Happy to discuss the CV technical bet or the clinical-safety design decisions.
```

---

### П'ятниця · Перший newsletter

Надішли перший email через Beehiiv. Твоя ціль W1 — 50 subscribers (перші з твого особистого нетворку).

**Email 1 template:**

```
Subject: Я будую AI-тренера. Ось чому.

[Перше ім'я],

Я будую FORM — AI personal trainer, який дивиться на твою техніку через камеру 
телефону і адаптує програму під твій сон і HRV.

Ще немає коду. Ще немає команди. Є: [посилання на GitHub] і чесна roadmap.

Чому я роблю це публічно? Тому що fitness-startup без довіри — просто ще одна апка.

Слідкуй — буде багато конкретного.

Джордж (Founder, FORM)
```

**Recruiting перших 50 subscribers:**
- Особистий DM у telegram/WhatsApp всім хто тренується (20–30 людей)
- Пост у особистому Facebook/Instagram
- LinkedIn post з посиланням на форму підписки

---

## WEEK 2 · Research Sprint — перші 10 інтерв'ю

### Понеділок–Середа · Recruiting

**Reddit recruiting DM (копіюй і надсилай):**

```
Hey [username],

Saw your post about [specific thing they wrote]. I'm building an AI personal trainer
called FORM that uses computer vision to coach your squat/deadlift form in real-time.

Would you be up for a 45-min Zoom call to share your current training setup and
what frustrates you about existing tools?

Paying $50 (Amazon or Wise transfer, your pick) — upfront before the call, not conditional.

No pitch. Genuinely want to understand how serious self-coached lifters think.

— George (Founder, @form_coach)
```

**Де шукати перших 10:**

| Subreddit | Фільтр | Мета n |
|---|---|---|
| r/weightroom | «self coached» + «technique check» posts | 4 |
| r/Fitness | «program» + «I don't have a trainer» | 3 |
| r/xxfitness | Жінки, «self-coached» | 2 |
| r/flexibility / r/bodyweightfitness | Edge cases | 1 |

**Ціль W2:** 10 конкретних людей з підтвердженим timeslot у Google Cal.

Скрипт інтерв'ю: [`docs/RESEARCH.md`](RESEARCH.md) — секція 3.
Компенсація: $50 Wise/Amazon, надіслати ДО дзвінку.

---

### Четвер–П'ятниця · Перші 5 інтерв'ю

**Шаблон нотаток для кожного:**

```markdown
## Interview #[N] — [Segment] — [Date]

**Participant:** [Letter+number — напр. D1 для Дмитро сегменту]
**Duration:** X хв
**Platform:** Zoom / in-person

### Ключові цитати
> «[Дослівно те що сказали]»

### JTBD (що вони насправді хочуть)
Коли я [контекст], я хочу [мотивація], щоб [результат].

### Реакція на FORM
- Screen 1 (Today): [реакція]
- Screen 2 (Workout+CV): [реакція]
- Victor tone: [реакція]
- Ціна: [що назвали]

### Red flags / disqualifiers
[Що сказали що зупинить їх від покупки]

### Follow-up needed
[ ] Або ні
```

Зберігай нотатки в Notion. Не в документах проєкту — інтерв'ю-дані не в публічний repo.

---

## WEEK 3 · Investor Outreach + FE Hiring

### Дні 1–2 · Investor CRM setup

Відкрий Google Sheets → создай таблицю:

| Investor | Fund | Tier | Warm intro? | Who? | Status | Last contact | Notes |
|---|---|---|---|---|---|---|---|
| [Name] | PHC | 1 | Yes | [mutual] | Research | — | Health-focused |

Заповни перші 20 рядків з [`docs/INVESTOR_OUTREACH.md`](INVESTOR_OUTREACH.md) Tier 1 + Tier 2 списку.

**Критерій: warm intro першим.** Сканери LinkedIn кожного фонду → хто з твоїх 1st connections знає їх?

---

### Дні 3–4 · Перші 10 warm intro запитів

**Email до mutual connection (теплий):**

```
Subject: Quick ask — intro to [Investor Name]

Hey [Mutual],

Working on something at the intersection of AI + fitness — FORM, an app that uses
CV to coach technique in real-time.

Would you be comfortable introducing me to [Investor] at [Fund]? Seems like a good fit
based on their portfolio.

Happy to send you a one-pager first so you know what you're intro-ing to.

Thanks,
George
```

**Не надсилай більше 3 warm-intro запитів одному mutual за один тиждень.**

Коли mutual дасть добро — надішли йому one-pager із [`docs/PITCH.md`](PITCH.md) (секція «One-pager»).

---

### Дні 5–7 · FE Job Posting live

**Де публікувати `hiring/jd-founding-engineer-ios.md`:**

| Channel | Як | Ціна | Очікувана якість |
|---|---|---|---|
| YC Hacker News (Who's Hiring) | Перший понеділок місяця — пост у треді | безкоштовно | висока |
| YC Co-founder Matching | Профіль проєкту + вакансія | безкоштовно | висока |
| LinkedIn | Company page post + personal post | безкоштовно | середня |
| Twitter/X | Thread з JD деталями + @-mentions | безкоштовно | variable |
| Wellfound (AngelList Talent) | Company profile + job post | безкоштовно | висока (стартап-орієнтований) |
| DOU.ua | Ukrainian iOS developer ринок | безкоштовно / $150 featured | висока для UA ринку |

**Не публікуй скрізь одночасно.** Порядок: HN Who's Hiring → YC Co-founder → Wellfound → LinkedIn → DOU.

**Що оцінювати у FE кандидатах:**

Мінімальний бар (hard requirements):
- Swift + SwiftUI: 3+ роки production досвіду
- Один shipped iOS app у App Store (посилання)
- Готовий до equity-heavy comp у pre-seed
- Зрозуміє та поважає clinical-safety constraints

Сильні сигнали (nice-to-have):
- Досвід з AVFoundation або Vision framework
- Власний fitness-tracking проєкт (GitHub, blog, або app)
- Публічний GitHub з iOS code

Red flags (instant disqualify):
- «Зроблю MVP за 2 тижні» без розуміння CV-complexity
- Не розуміє різниці між MediaPipe і Apple Vision
- Не читав PRODUCT_SPEC.md перед першим дзвінком (якщо прийшов підготованим)

Технічне завдання (home assignment, 3–4 год):
> Побудуй iOS prototype що використовує AVCaptureSession + Vision framework для real-time detection
> людської фігури (bodyDetection) і виводить у екран лічильник: скільки разів людина присіла.
> Немає правильної відповіді — нас цікавить твій підхід до невизначеності.

Деталі ролі: [`hiring/jd-founding-engineer-ios.md`](../hiring/jd-founding-engineer-ios.md).

---

## WEEK 4 · Синтез + Наступна фаза

### Дні 1–2 · Synthesis сесія (після 10 інтерв'ю)

Відкрий [`docs/RESEARCH.md`](RESEARCH.md) секцію 6 (Synthesis framework). Заповни:

1. **Affinity map** — згрупуй цитати за темою (Notion board або стікери)
2. **Persona validation** — чи Дмитро primary persona підтверджується? Скорегуй.
3. **JTBD refinement** — чи наша формулювання відповідає реальним словам?
4. **Pricing signals** — медіана ціни по сегментах
5. **Top-3 killers** — що треба фіксити до launch

**Action thresholds** (з RESEARCH.md секції 7):
- > 50% кажуть «$10 max» → переглянь цінову стратегію
- > 50% кажуть «Camera у гімі — незручно» → готуй no-CV mode
- > 50% кажуть «Voice дратує» → default text-only, voice opt-in

---

### Дні 3–5 · Наступні 10 інтерв'ю (Олена + Артур сегменти)

Recruiting по тому самому шаблону. Ціль: 20 total до кінця W4.

Паралельно — пости у Twitter кожен день:
- Понеділок: один insight з інтерв'ю (без PII)
- Середа: один технічний пост про CV
- П'ятниця: один founder update (що дізнався за тиждень)

---

### Кінець W4 · Checkpoint

**Чи ми готові до наступної фази?**

| Gate | Target | Статус |
|---|---|---|
| form.coach живий | ✓ | |
| Twitter активний (>100 followers) | ✓ | |
| Newsletter (>100 subscribers) | ✓ | |
| 20 інтерв'ю проведено | ✓ | |
| Persona оновлені | ✓ | |
| FE JD опублікована | ✓ | |
| 10+ warm intro надіслано | ✓ | |
| 1–2 investor conversations started | ✓ | |

**Якщо всі 8 checkpoints green → переходиш до M1: iOS development sprint.**

Якщо ні → ще один 2-тижневий цикл з тим що не виконано.

---

## Інструменти і посилання

| Потреба | Де шукати |
|---|---|
| Investor list | [`docs/INVESTOR_OUTREACH.md`](INVESTOR_OUTREACH.md) |
| Interview script | [`docs/RESEARCH.md`](RESEARCH.md) → секція 3 |
| Deploy guide | [`docs/DEPLOYMENT.md`](DEPLOYMENT.md) |
| Newsletter setup | [`docs/NEWSLETTER.md`](NEWSLETTER.md) |
| FE Job description | [`hiring/jd-founding-engineer-ios.md`](../hiring/jd-founding-engineer-ios.md) |
| Beta recruiting | [`docs/BETA_RECRUITING.md`](BETA_RECRUITING.md) |
| Content calendar | [`docs/CONTENT_CALENDAR.md`](CONTENT_CALENDAR.md) |
| Twitter launch thread | [`content/twitter-launch-thread.md`](../content/twitter-launch-thread.md) |
| Pitch one-pager | [`docs/PITCH.md`](PITCH.md) → секція «One-pager» |

---

## Що може піти не так (і що робити)

| Проблема | Симптом | Action |
|---|---|---|
| form.coach не доступний | DNS propagation > 24 год | Перевір Cloudflare DNS records, TTL |
| Reddit recruiting не конвертить | < 30% reply rate на DM | Зміни підхід: коментуй спочатку, потім DM |
| Інтерв'ю показують що FORM-концепт слабий | > 60% оцінюють < 6/10 | Не панікуй — pivot pitch, не продукт |
| Жодного investor response за 2 тижні | 0 replies | Shift до warm intros якщо були cold emails |
| FE кандидати не приходять | < 5 applications за 2 тижні | Personalize DMs до конкретних iOS devs |
| Twitter росте повільно | < 50 followers за W1 | OK — нормально для нового аккаунту. Продовжуй |

---

## Принципи (нагадування)

1. **Жодних вигаданих цифр.** Цілі — окей. Результати — тільки коли є дані.
2. **Все про їжу/тіло/травму → clinical-safety review перед публікацією.**
3. **Якщо фразу не можна сказати другу в гімі — не кажи у Twitter.**
4. **Не пропускай два поспіль** — ні тижні без дій, ні дні без одного конкретного кроку.

---

**v0.1 · травень 2026 · `process-keeper` + `growth-lead`**
