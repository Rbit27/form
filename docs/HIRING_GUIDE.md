# FORM · Hiring Guide

> Покрокова інструкція для розміщення вакансій Founding Engineer (iOS) і Design Lead.
> Власник: founder (George). Агент: `process-keeper`.
> Версія: v0.18.0 · травень 2026

---

## Overview

Маємо: два JD-файли готові до публікації.
Треба: розмістити на правильних платформах в правильному порядку, отримати перших кандидатів.

| Role | Priority | Files |
|---|---|---|
| Founding Engineer (iOS) | **HIGH** — блокує product | `hiring/jd-founding-engineer-ios.md` |
| Design Lead | **MEDIUM** — блокує visual production | `hiring/jd-design-lead.md` |

Posting order: FE перший, DL тиждень потому (щоб не втопитися в CV одночасно).

---

## Platform priority

### Founding Engineer (iOS)

| # | Platform | Cost | Reach | Why |
|---|---|---|---|---|
| 1 | **Djinni** | Free | UA/remote iOS devs | Найбільша UA tech платформа. iOS розробники активні. Безкоштовно. |
| 2 | **DOU** | Free | UA senior devs | Важливий для senior Ukrainian dev community. Довірча платформа. |
| 3 | **Wellfound** | Free tier | Global startup devs | Equity-motivated кандидати — саме ті, кому ми цікаві. |
| 4 | **LinkedIn** | Free post / paid boost | Global | Зробити після Djinni/DOU. Boost ($50-150) тільки якщо мало відгуків за тиждень. |

### Design Lead

| # | Platform | Cost | Reach | Why |
|---|---|---|---|---|
| 1 | **LinkedIn** | Free / paid | Global designers | Дизайнери більше на LinkedIn ніж Djinni. |
| 2 | **Wellfound** | Free | Startup designers | Equity-мотивовані UI/UX кандидати. |
| 3 | **Djinni** | Free | UA designers | Є UA дизайнери, але менший вибір ніж LinkedIn. |
| 4 | **Behance Jobs** | Free | Portfolio designers | Якщо не вистачає відгуків після 2 тижнів. |

---

## Platform 1: Djinni (FE першочерговий)

### Аккаунт / сторінка компанії

1. Зайди на https://djinni.co
2. Sign up як **Employer** (або log in якщо вже є аккаунт)
3. Заповни Company Profile:
   - **Company name:** FORM
   - **Logo:** використай wordmark з BRAND.md (темний на кремовому фоні, або кремовий на чорному)
   - **Website:** https://form.coach (поки що GitHub repo — https://github.com/Rbit27/form)
   - **About:** 2-3 речення з `about.html` → manifesto абзац
   - **Stage:** Pre-seed / Seed
   - **Team size:** 1 + AI team

### Публікація вакансії (FE)

4. "Post a job" → New vacancy
5. **Job title:** `iOS Engineer (Founding)` або `Senior iOS Engineer — AI Fitness App`
6. **Category:** Mobile / iOS
7. **Experience:** 4+ years (Senior)
8. **Location:** Remote / Ukraine
9. **Salary:** Negotiable / Equity-heavy (вкажи "competitive" — не вигадуй цифр до розмови)
10. **Description:** Адаптуй з `hiring/jd-founding-engineer-ios.md`:
    - Перші 2 абзаци — hook (чому це унікальна роль)
    - What you'll build (bullet list)
    - What we need (must-have skills)
    - What we DON'T want (hard-noes — це відсіє неправильних кандидатів)
    - Compensation structure
    - GitHub repo link: https://github.com/Rbit27/form
    - Apply: hiring@form.coach

11. **Publish**

### Що не робити на Djinni

- Не вказуй конкретну зарплату поки не знаєш ринок → скажи "competitive"
- Не пиши "стартап" без контексту → пиши "pre-seed AI fitness company"
- Не забудь GitHub link — це конвертує скептичних senior devs

---

## Platform 2: DOU (Джерело · Форум)

### Спосіб 1: Вакансії на DOU

1. Зайди https://jobs.dou.ua
2. Post a job → Company registration якщо немає
3. Fill: Mobile/iOS · Senior · Remote · Competitive salary
4. Description: та ж структура що Djinni, але можна трохи довша — DOU аудиторія читає уважніше
5. **Важливо:** Вкажи tech stack точно (Swift, SwiftUI, CoreML/Vision, HealthKit) — DOU розробники фільтрують по стеку

### Спосіб 2: DOU Forum (безкоштовно + organic)

1. Форум → Розділ "Робота" → тред "Шукаємо iOS розробника"
2. Напиши коротке 5-10 речень пояснення:
   - Хто ти, що будуєш
   - Чому роль унікальна (founding = equity)
   - Посилання на GitHub і hiring@form.coach
3. DOU форум індексується — good for SEO + organic reach

---

## Platform 3: Wellfound (AngelList Talent)

Це must для equity-motivated кандидатів.

### Аккаунт

1. https://wellfound.com → "Post a Job" → Create Company
2. **Company profile:**
   - Name: FORM
   - Stage: Pre-seed
   - Market: Health & Fitness / AI
   - Location: Ukraine (Remote-first)
   - Team size: 1-5
   - GitHub: https://github.com/Rbit27/form
   - Pitch: 1 paragraph (з `about.html` manifesto)

### Вакансія

3. Title: `iOS Engineer — Founding` або `Lead iOS / ML Engineer`
4. Role type: Full-time · Remote
5. Compensation: Specify equity range (наприклад 1-3% — перевір з advisor що reasonable для pre-seed founding engineer)
6. Skills: Swift, SwiftUI, CoreML, Vision Framework, HealthKit, Combine/async-await
7. Description: адаптована версія з JD-файлу
8. **Apply via:** hiring@form.coach (або Wellfound has built-in apply)

### Чому Wellfound важливий

Wellfound відображає equity % prominently. Кандидати які шукають founding-stage opportunity там активні. Це відсіює людей що хочуть тільки salary.

---

## Platform 4: LinkedIn

### Для Design Lead (PRIMARY для цієї ролі)

1. Зайди на своїй особистий профіль → Company Page якщо є (або пости від особи)
2. "Post a job" → Job type: Full-time
3. Title: `Design Lead — AI Fitness App (Founding)` або `Product Designer — iOS · Pre-seed Startup`
4. Location: Remote · Ukraine (або "Anywhere")
5. Description: з `hiring/jd-design-lead.md` — адаптуй для LinkedIn format (коротший header)
6. Apply: redirect до hiring@form.coach

### LinkedIn Budget

- **Free post:** видно 30 днів, обмежений reach
- **Boost ($50-100/week):** add тільки якщо відгуків мало після перших 5-7 днів
- Sponsored post для Design Lead більш виправданий ніж для FE (де Djinni вистачить)

### LinkedIn альтернатива (безкоштовна)

Замість платного posting — зроби **особистий пост** від свого профілю:
```
We're building FORM — an AI personal trainer that watches your form, 
adapts to your HRV, and speaks like a 20-year coach.

I'm looking for a Design Lead to join as a founding team member.

Full JD: hiring@form.coach / [link to jobs.html]
GitHub: github.com/Rbit27/form

We're pre-seed. This means equity, ownership, and building from zero. 
If that sounds like your environment — let's talk.
```
Personal posts на LinkedIn конвертують краще ніж company job postings у pre-seed stage.

---

## Response management protocol

### Перші 24 години після posting

- Очікуй: перші відгуки на Djinni за 24-48 годин
- Перша реакція має бути за **48 годин максимум** (senior кандидати не чекають)
- Надсилай acknowledgement: "Отримали CV, перегляну до [date]. Дякую."

### Скринінг CV (15-20 хвилин на кандидата)

**FE iOS — Red flags:**
- Немає досвіду з SwiftUI (UIKit alone = no)
- Тільки React Native / Flutter досвід (hybrid = no для CV-heavy role)
- Жодного публічного коду (GitHub порожній) без пояснення
- "5 роки досвіду" але тільки B2B enterprise apps (не consumer-focused experience)

**FE iOS — Green flags:**
- Side projects що shipped (навіть малі)
- Досвід з CoreML, Vision, HealthKit — будь-який
- Consumer app experience (games, health, lifestyle)
- Open source contributions

**Design Lead — Red flags:**
- Тільки web/desktop portfolio — жодного mobile
- Generic UI kit remixing (no original thinking)
- "Я можу зробити будь-який стиль" — відсутність точки зору

**Design Lead — Green flags:**
- Strong iOS Figma work
- Opinion про продукт (не просто виконавець)
- Experience з health/fitness або emotional product design

### Відповідь відмова (template)

```
Привіт [ім'я],

Дякую за відгук на [роль] в FORM.

Ми уважно переглянули CV. На цьому етапі ми рухаємося далі з іншими кандидатами,
чий досвід найтісніше відповідає нашим потребам прямо зараз.

FORM будується публічно — слідкуй за розвитком на github.com/Rbit27/form.
Якщо наші шляхи перетнуться у майбутньому — будемо раді.

Успіху, 
George
FORM
```

---

## First interview logistics

- **Format:** 45 хвилин відео (Google Meet або Zoom)
- **Структура:**
  - 5 хв: context (George представляє FORM)
  - 20 хв: кандидат розповідає про проект, який найбільше ним пишається
  - 15 хв: technical questions (по HIRING.md rubric)
  - 5 хв: питання від кандидата → RED FLAG якщо немає питань
- **Recording:** тільки з consent
- **Notes:** одразу після дзвінка, до 1 години

---

## Timeline

| Week | Action |
|---|---|
| W1 | Post FE on Djinni + DOU + Wellfound |
| W1 | Post Design Lead on LinkedIn (personal post) + Wellfound |
| W2 | First screening calls |
| W3 | Technical interviews (top 2-3 per role) |
| W4 | Reference checks + offer |
| W5 | Onboarding |

Якщо після 2 тижнів менше 5 якісних CV — boost LinkedIn ($50) і запости в DOU форум вручну.

---

## Budget breakdown

| Item | Cost | Notes |
|---|---|---|
| Djinni posting | Free | Пріоритет для FE |
| DOU jobs | Free | 30-day free post |
| Wellfound | Free | Equity-focused |
| LinkedIn post (personal) | Free | Better conversion than paid |
| LinkedIn paid boost (if needed) | $50-150/week | Only if leads dry out |
| **Total planned** | **$0-150** | Start free, boost if needed |

---

## Checklist before first posting

- [ ] Company email hiring@form.coach active (або redirect до особистого на час)
- [ ] jobs.html відкривається коректно у браузері (відправляємо як landing)
- [ ] GitHub repo public і README.md актуальний
- [ ] HIRING.md відкритий — interview rubric під рукою
- [ ] Calendar заблоковані слоти для screening calls (ранок чи ввечері — кандидати часто employed)
- [ ] Notion або Google Sheet для відстеження кандидатів (мінімум: ім'я, роль, платформа, статус, дата останнього контакту)

---

## Tracking sheet columns (мінімум)

| Column | Values |
|---|---|
| Name | — |
| Role | FE / DL |
| Platform | Djinni / DOU / LinkedIn / Wellfound / Referral |
| Source date | YYYY-MM-DD |
| CV score | 1-5 |
| Status | Applied → Screening → Interview → Offer → Rejected |
| Last contact | YYYY-MM-DD |
| Notes | red/green flags |

---

**Owner:** founder (George)
**Agent support:** `process-keeper` (tracking), `product-strategist` (scope questions)
**Last updated:** v0.18.0 · травень 2026
