# FORM · Newsletter Setup & Strategy

> Власник: `content-strategist`
> Мета: pre-launch list building + content engine distribution

---

## TL;DR

Beehiiv. Безкоштовно до 2,500 підписників. Налаштувати до deploy marketing site. Мета першого місяця — 200 підписників до Beta Q3 2026.

---

## 1 · Платформа: Beehiiv

### Чому Beehiiv, а не ConvertKit / Mailchimp

| Критерій | Beehiiv | ConvertKit | Mailchimp |
|---|---|---|---|
| Free tier | 2,500 підп. безкоштовно | 1,000 підп. | 500 підп. |
| Web3 / blog | Вбудований | Ні | Ні |
| Referral growth | Вбудовано | Ні | Ні |
| Tone | Builder-first | Creator-first | Business |
| Export data | Повний | Повний | Обмежено |
| **Рішення** | **✓ Beehiiv** | | |

ConvertKit — гарна альтернатива якщо Beehiiv не підходить по UX. Обидва мають clean API для майбутньої автоматизації.

---

## 2 · Налаштування (~45 хвилин)

### Крок 1 — Акаунт

1. Зареєструватися на beehiiv.com
2. Publication name: **FORM**
3. Handle: `form-coach` (або `formcoach`)
4. Subdomain: `newsletter.form.coach` → кастомний домен після deploy

### Крок 2 — Брендинг

- Logo: FORM wordmark (чорний на cream, або white на ink)
- Primary color: `#DDFF2D` (acid)
- Background: `#0B0A07` (ink)
- Font: Manrope (Google Fonts, є у Beehiiv)
- Accent: `#FF5733` (ember)

> Beehiiv дозволяє кастомний CSS у платному плані. На безкоштовному — використовуй їхній темний template як базу.

### Крок 3 — Welcome email

**Subject:** You're on the FORM list. Here's what that means.

```
Hi,

You signed up for FORM — an AI personal trainer that watches your form
and adjusts your program to your sleep and HRV.

Here's what you'll get from this newsletter:

→ Product updates (honest ones — no hype, no fake metrics)
→ Training science translated to practice
→ Thinking behind our design decisions

What you won't get:
✕ Before/after content
✕ Discount emails
✕ "Your body is capable of anything" copy

If you're a self-coached lifter who takes training seriously —
you're in the right place.

We're in closed beta planning. First 200 beta slots are for people
on this list.

George
Founder, FORM · Kyiv
```

### Крок 4 — Subscribe form

Embed на:
- `index.html` → після hero або в footer
- `beta.html` → alternate CTA якщо beta форма заповнена
- `about.html` → після founder section

Текст CTA: **"Join the list"** — не "Subscribe to newsletter" (boring), не "Get free content" (cheap).

---

## 3 · Перші два пости

### Post A · Welcome to the list

**Subject:** Why we're building FORM

**Format:** Personal letter, 400–600 слів

**Outline:**
- Відкрити з конкретним моментом: перший рік тренувань, перша сесія з тренером, конкретний підйом
- Що живий тренер дав, чого не дала жодна апка: людина, яка бачить тебе
- Чому це продуктова задача, а не ринкова — нікого не будує coaching-first
- Що таке FORM у двох реченнях
- Що чекати від листів: product updates, training science, design thinking
- Що НЕ буде: hype, fake metrics, motivational posters

**Tone:** Victor off-duty — директивно, без пафосу, особисто

**Victor voice check:**
- "Перший рік я тренувався з тренером. Він бачив, коли я не закінчував рух. Апки — ні."
- НЕ: "Моя пристрасть до фітнесу привела мене до..."

---

### Post B · What FORM will and won't do

**Subject:** What FORM actually does (and what it doesn't)

**Format:** Structured breakdown, 500–700 слів

**Outline:**
1. **Що буде:** камера на 8 базових рухах, real-time feedback, VM голос
2. **Що буде:** програма, яка змінюється на основі HRV + сну щодня
3. **Що буде:** чесний тренер без surveillance мови
4. **Чого не буде:** медичні діагнози, харчовий panopticon, before/after, lifetime deals
5. **Що поки невідомо:** якість CV без тестування на реальних атлетах — чесно кажемо
6. Invite: beta реєстрація, 200 слотів

**Purpose:** Встановити довіру через чесність.

**Key line:**
> "We're not promising perfect form analysis. We're promising rep counting, tempo, and gross-deviation detection on eight base lifts. That's what we can ship honestly. Everything else is research."

---

## 4 · Cadence (pre-launch)

| Фаза | Частота | Тип контенту |
|---|---|---|
| Now → Beta launch | 1×/2 тижні | Product thinking + training science |
| Beta (Aug–Oct 2026) | 1×/тиждень | Beta updates + user stories (з consent) |
| Post-launch | 1×/тиждень | Feature deep-dives + community |

**Правила cadence:**
- Краще пропустити, ніж видати порожній лист
- Якщо немає що сказати — не кажи нічого
- Мета не в кількості листів, а в retention: якщо > 50% відкривають — ти на правильному шляху

---

## 5 · Growth mechanics (pre-launch)

### Referral loop

Beehiiv має вбудований referral program:
- "Refer 3 friends → jump the beta waitlist queue"
- Текст: "Знаєш когось, хто серйозно тренується? Поділись. Вони подякують. І ти стрибнеш вище в черзі."

### Distribution checklist

Після кожного посту:
- [ ] Twitter/X thread (ключова ідея у 3–5 твіти)
- [ ] Reddit пост у r/fitness або r/weightroom (якщо відповідає темі — без spam)
- [ ] LinkedIn storypost (для investor/hiring visibility)
- [ ] Share у особистому колі

### SEO

Beehiiv автоматично публікує posts як web pages.
- Не налаштовувати окремий blog поки marketing site не deployed
- Після deploy: redirect beehiiv posts → blog на form.coach

---

## 6 · Метрики

| Метрика | Ціль M1 | Ціль Beta launch |
|---|---|---|
| Підписники | 200 | 500 |
| Open rate | > 40% | > 45% |
| Click rate | > 10% | > 12% |
| Referrals | — | 50+ |
| Unsubscribe rate | < 2% | < 1.5% |

> Ці цілі — орієнтири, не contractual KPIs. Якщо open rate нижче 30% після 5 листів — переглянути subject lines і content mix.

---

## 7 · Anti-patterns (не робити)

- **Не купувати підписників.** Ніколи.
- **Не import cold lists** — тільки opted-in.
- **Не надсилати email серед ночі** — quiet hours 22:00–07:00 (як і в аппці).
- **Не робити рекламні листи** маскованими під editorial.
- **Не обіцяти "ексклюзивний контент"** якщо той самий контент буде на сайті.
- **Не форсувати frequency** заради metric appearance.

---

## 8 · Перед першим надсиланням (checklist)

- [ ] Beehiiv акаунт створено
- [ ] Брендинг налаштовано (logo, colors, font)
- [ ] Welcome email написано і протестовано (надіслати самому собі)
- [ ] Subscribe embed на index.html і beta.html
- [ ] Custom domain підключено (після form.coach deploy)
- [ ] Перший пост написаний і scheduled
- [ ] clinical-safety review на welcome email та Post A і Post B ← ОБОВ'ЯЗКОВО

---

**Версія:** 1.0 · травень 2026
**Власник:** content-strategist
**Наступний перегляд:** після Beta launch
