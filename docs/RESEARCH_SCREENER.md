# FORM · User Research Screener & Recruiting Toolkit v0.1

> Практичний інструмент для рекрутингу 30 учасників user research інтерв'ю.
> Читати разом із: RESEARCH.md (script + goals), BETA_RECRUITING.md (beta users — post-product).
> Власник: `research-lead`. Запускати: W1 із FIRST_30_DAYS.md.

---

## Ціль цього документу

`RESEARCH.md` визначає ЩО досліджувати і ЯК проводити інтерв'ю.
Цей документ — як знайти і відібрати 30 правильних людей, організувати оплату, і не втратити дані.

---

## 1 · Screener survey (copy-paste ready)

### Recruiting message — EN (Reddit / X / Telegram)

```
Hey, I'm doing user research for a fitness AI project before writing a
single line of code. Looking for 30 experienced lifters for a 45-min
video call. Compensated: $50 gift card, sent BEFORE we talk. No sales
pitch — just honest conversation about how you actually train.

Could you fill in 5 quick questions to see if you're a fit?
[screener link]
```

### Recruiting message — UA (Telegram / gym network)

```
Привіт! Роблю дослідження для fitness AI стартапу — ДО розробки, не після.
Шукаю 30 людей хто регулярно тренується. 45 хв відеодзвінок.
Оплата: $50 gift-картка ДО розмови. Без pitch.
5 коротких питань — перевіримо чи підходимо:
[посилання на screener]
```

---

## 2 · Screener questions (Google Forms / Typeform)

Зроби форму з цими питаннями в точно такому порядку.

---

**Q1: Як часто ти тренуєшся в середньому?**

| Відповідь | Рішення |
|---|---|
| Менше 1 разу на тиждень | ❌ REJECT |
| 1–2 рази на тиждень | 🟡 MAYBE (Олена-persona — обмежена квота) |
| 3–4 рази на тиждень | ✅ ACCEPT |
| 5+ разів на тиждень | ✅ ACCEPT |

---

**Q2: Скільки ти вже тренуєшся силовими / у залі?**

| Відповідь | Рішення |
|---|---|
| Менше 6 місяців | ❌ REJECT (немає усталених звичок для дослідження) |
| 6–12 місяців | 🟡 MAYBE |
| 1–3 роки | ✅ HIGH VALUE (intermediate sweet-spot) |
| 3+ роки | ✅ ACCEPT |

---

**Q3: Як ти зазвичай складаєш тренувальний план?**

| Відповідь | Рішення |
|---|---|
| Сам(а) — YouTube, Reddit, книги | ✅ HIGH VALUE (Дмитро) |
| Персональний тренер (зараз або раніше) | ✅ HIGH VALUE |
| App (Hevy, Strong, Fitbod, etc.) | ✅ ACCEPT |
| Просто йду в зал і роблю що хочеться | ❌ REJECT |

---

**Q4: Який основний телефон ти використовуєш?**

| Відповідь | Рішення |
|---|---|
| iPhone (будь-яка модель) | ✅ ACCEPT |
| Android (Samsung, Pixel, etc.) | 🟠 WAITLIST — зазнач у трекері |
| Не маю смартфону | ❌ REJECT |

---

**Q5: Який з варіантів найточніше описує тебе?**

| Відповідь | Рішення |
|---|---|
| «Тренуюсь сам, але не впевнений чи роблю правильно» | ✅ HIGH VALUE |
| «Знаю що роблю, але хочу більше даних і автоматизації» | ✅ ACCEPT (Артур) |
| «Раніше тренувався серйозно, намагаюся повернутися» | ✅ ACCEPT (Олена) |
| «Просто хочу схуднути до літа» | ❌ REJECT |

---

**Q6 (сегментація, не для відсіву): Яку wearable ти носиш?**

Checkbox (вибери всі що є):
- Apple Watch
- Whoop
- Oura Ring
- Garmin / Polar / Suunto
- Нічого

→ Використай для балансу за персонами (Артур = wearable power-user потрібен мінімум 7).

---

**Q7 (контактна): Email для зв'язку і gift card доставки**

Одне текстове поле. Зазнач: «Ця адреса використовується тільки для scheduling і платежу — ніколи для реклами».

---

## 3 · Recruiting channels та квоти

| Канал | Цільовий сегмент | Підхід | Очікуваний CR |
|---|---|---|---|
| Reddit r/fitness, r/weightroom | Дмитро, Артур | Public post (transparent founder post) | 1–2% від views |
| Reddit r/xxfitness | Олена | Adaptation of post | 1% |
| X / Twitter DMs | Артур (wearable users) | Like+comment → DM after | 5–10% відповідей |
| Telegram (UA gym chats) | Дмитро, Богдан (UA cohort) | Особисте повідомлення з контексту | 15–20% |
| LinkedIn | Богдан (40+, professional) | Особистий пост від George | 3–5% |
| Gym personal network | Будь-який | Текст другу → реферал | 30–50% |

**Цільова квота** (розбивка з RESEARCH.md):

| Сегмент | Target N | Статус |
|---|---|---|
| Дмитро (self-coached intermediate) | 10 | Рекрутувати першим |
| Олена (lapsed gym-goer) | 7 | Паралельно |
| Артур (wearable power-user) | 7 | Через X/Twitter |
| Богдан (40+ sustainable) | 6 | LinkedIn + особиста мережа |

---

## 4 · Reddit post template (ready-to-post)

### r/fitness, r/weightroom

```
Title: Paid user research for fitness AI app — $50 for 45-min call (pre-product, not a pitch)

Hey,

I'm building a fitness AI app (based in Ukraine, targeting serious lifters)
and I'm doing user research before any code is written. I want to understand
how real gym-goers actually train — not what we assume.

What I'm looking for:
• You've lifted consistently for 1+ years
• You're mostly self-coached (coaches, coaches' clients also fine)
• You have an iPhone

What you get:
• $50 Amazon / Apple gift card — sent before we talk
• 45-minute video call, relaxed conversation, no pitch
• Your input directly shapes what we build (or don't build)

If interested, here's a short screener (5 questions):
[link]

Happy to answer any questions in comments.

— George, building FORM
```

### r/xxfitness (adapted)

```
Title: Seeking women lifters for paid UX research — $50 gift card, 45 min, no pitch

[Variation: same body but emphasize "women's training frustrations" and mention the cycle-aware training feature in screener Q2 reaction — do NOT mention in the recruiting post itself]
```

---

## 5 · X / Twitter recruiting flow

**Step 1: Find targets**

Search operators:
```
"my gym routine" OR "lifting program" OR "self coached" OR "no PT" iphone since:2025-01-01 -is:retweet
```

```
"oura" OR "whoop" OR "apple watch" "training" OR "lifting" -is:retweet
```

**Step 2: Engage first**

Like + meaningful reply to 1–2 of their tweets. Wait 24h.

**Step 3: DM (after engagement)**

```
Hey [name], I saw your post about [specific tweet reference].
I'm doing user research for a fitness AI startup — looking for 30 serious
lifters for a 45-min call. $50 gift card upfront, no pitch.
5-question screener if you're interested: [link]
```

**Rule:** Never cold-DM without prior engagement. Response rate drops to <1%.

---

## 6 · Interview tracker (Google Sheets columns)

One row = one participant. Never use real names in analysis documents.

| Column | Format | Notes |
|---|---|---|
| ID | P001–P030 | Anonymization key |
| Name | Text | Scheduling only — never in reports |
| Email | Text | Gift card delivery |
| Segment | Дмитро/Олена/Артур/Богдан | Based on Q3+Q5 |
| Channel | Reddit/X/Telegram/LinkedIn/Network | Where recruited |
| Screener date | YYYY-MM-DD | |
| Score | 🟢/🟡/🟠/🔴 | From Section 3 scoring |
| Status | Screened/Scheduled/Completed/NoShow/Cancelled | |
| Call date | YYYY-MM-DD | |
| Gift card sent | ✓ / ✗ | Must be ✓ before call |
| Recording | URL | Zoom / Otter.ai |
| Transcript | URL | Otter.ai auto-transcript |
| Pain score | 1–10 | Q18 from RESEARCH.md script |
| Price mentioned | $XX | Q19 — unprompted number |
| Top disqualifier | Text | Q20+22 — one phrase |
| Top quote | Text | 1 sentence for language bank |
| NPS proxy | 1–10 | «Кому б порадив» (Q24) |

---

## 7 · Scheduling protocol

**24h response SLA** — відповідай на screener протягом доби.

### Confirmation email

```
Subject: Research call confirmed — [Day, Date, Time Timezone]

Hi [first name],

Confirmed for [Day], [Date] at [Time] [Timezone].

Zoom: [link]

Your $50 [Amazon/Apple] gift card will arrive at this email address at least
1 hour before our call. If you don't see it, check spam or reply here.

A few things:
• 45 minutes, relaxed — no right or wrong answers
• I'll share a couple of app screenshots partway through (not a pitch)
• Recording with your consent — anonymous in any reports we share

See you then.

George / FORM
```

### 24h reminder

```
Subject: Quick reminder — our call tomorrow at [time]

Hi [name], just a reminder for tomorrow at [time].
Zoom: [link]
Gift card was sent [date] — let me know if there was any issue.
```

### 1h reminder (Calendly can automate this)

```
Subject: Starting in 1 hour — [time]

See you soon. Zoom: [link]
```

---

## 8 · Gift card payment flow

Send **before** the call. Non-negotiable — it's a bias guard, not a courtesy.

**Option A: Amazon gift card (international, EN participants)**
1. amazon.com → Gift Cards → Email delivery → $50
2. Recipient's email → Deliver immediately → Send

**Option B: Apple gift card (US / EU iPhone users)**
1. App Store → Buy App Store & iTunes Gift Cards → $50 → email delivery
2. Works in most markets where participant has Apple ID

**Option C: Monobank send (UA participants)**
1. Додаток Monobank → Переказати → по номеру телефону
2. Сума: ₴2100 (еквівалент $50 за поточним курсом НБУ — уточнюй щоразу)

**Option D: PayPal Friends & Family (backup)**
1. PayPal → Send → Friends & Family → $50
2. Не застосовуй Business/Goods & Services (комісія + tracking)

**Timing rule:** Надіслати ≥ 1 год ДО початку дзвінка. Підтвердити що отримали.

**NoShow rule:** Гроші залишаються у них. Ми не повертаємо — це честь угоди. Пропонуй reschedule.

---

## 9 · Polite rejection template

```
Hi [name],

Thanks for filling out our screener — really appreciate it!

Your profile doesn't quite match our current research focus (we're specifically
looking at intermediate+ lifters who are mostly self-coached). But we'll keep
your contact on file — if our research expands into other segments, we'll reach
out.

Thanks for your interest in FORM.

George
```

**Тон:** тепло, не формально. Майбутній користувач.

---

## 10 · Anti-bias pre-call checklist

Перед кожним інтерв'ю:

- [ ] Gift card надіслано і отримано підтвердження
- [ ] Запис увімкнено і consent отримано (**«Окей якщо запишу?»**)
- [ ] Не показуєш / не пояснюєш продукт ДО Block 3 (15-й хвилині)
- [ ] Pitch = нуль навіть у Block 3 — тільки показ екрана, питання реакції
- [ ] Пауза 5+ секунд після питання — не заповнюй тишу
- [ ] Помилкових відповідей не існує — кивай і нотуй
- [ ] Якщо людина raises ED / self-harm → стоп, дай ресурси (RESEARCH.md §9)

---

## 11 · Weekly velocity та milestones

| Тиждень | Ціль | Running total |
|---|---|---|
| W1 (recruiting) | 15+ screener submissions | — |
| W2 | 8 calls completed | 8 |
| W3 | 12 calls completed | 20 |
| W4 | 10 calls completed | 30 |
| W5 | Synthesis complete | → RESEARCH.md §6 |

Recruiting стартує W1, паралельно з сайтом. Не чекай поки сайт буде живим.

---

## 12 · Open questions

1. Чи EU-учасники потребують окрему consent форму під GDPR? (→ LEGAL.md)
2. Чи Otter.ai transcript вистачає для синтезу, чи варто додати Notion template?
3. Яку локацію вказувати в Reddit пості — «Ukraine-based» або «global team»?
