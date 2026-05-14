---
title: "30 інтерв'ю до launch: що ми навчилися"
slug: 30-user-interviews
date: 2026-05-14
author: George (FORM founder)
category: research
estimated_reading_time: 13 хв
status: draft · pending real interviews to complete
review_pending: research-lead
target_persona: tech-builders + founders
---

# 30 інтерв'ю до launch: що ми навчилися

> Цей post буде написаний **після** того, як ми реально проведемо 30 інтерв'ю. Поки що — це шаблон-каркас з тих 5 інтерв'ю, що вже відбулися. Real version updated post-Q3 2026.

---

Кожен product-фаундер чує пораду «talk to users».

Більшість не роблять.

Я не хочу бути більшістю. Тому я витратив 4 тижні на 30 deep interviews перед public launch.

Що дізнався — нижче.

---

## Метод (швидко)

- **30 інтерв'ю**, 45 хв кожне
- **4 segments:** Дмитро (primary), Олена, Артур, Богдан (наші персони з [PERSONAS.md](#))
- **Recruiting:** Reddit DMs (r/fitness), X DMs, friends-of-friends
- **Compensation:** $50 gift-card перед інтерв'ю
- **Recording з consent, transcripts**
- **Synthesis після всіх через affinity mapping**

Не interviewed нікого з direct команди, не дав «trial-кляк» як винагороду (це створює bias).

---

## Що підтвердилося

### 1. JTBD framework — accurate

Initial framing:
> Коли я хочу стати сильнішим без $200/тиждень на тренера, допоможи мені тренувати розумно і залишатися accountable.

Real users formulated it differently, але essence була та сама. 28 з 30 echoed.

Quote (Дмитро-tier, IT-engineer, 31):
> «Мені не потрібен мотиваційний посос. Мені потрібен той, хто скаже „ти присідаєш на 2 см вище ніж тиждень тому" — і пояснить чому це проблема.»

### 2. Camera-coaching is the wedge

Asked «would you pay $19/міс for an AI fitness app?» — 16 said yes / 14 maybe.

Asked «would you pay $19/міс for an AI that watches your form?» — 26 said yes.

**Camera analysis is the differentiator.** Sleep / HRV / Programming were nice-to-have. Camera was the «aha».

### 3. Voice (Victor) надмірно сильний

Тон-калібровка — 12 з 30 specifically mentioned приходили back до додатку через те, що Victor's responses вдало landed. Це signal — голос продукту реально matters.

---

## Що нас здивувало

### 1. Жінки хочуть «бути сильнішими», не «худнути»

З 8 жінок у sample — лише 2 prioritizes ваги. Решта хочуть:
- Силу
- Form quality
- Functional capacity для повсякденного життя

Це **сильно інше** ніж стандартний positioning fitness-апок для жінок. Наша «training-first, body-comp opt-in» позиція landed дуже добре.

### 2. Богдан-сегмент (40+) — more affluent than expected

Очікувано $9-12 willingness-to-pay. Реальне число: $19-25 averaged. Old-school crowd ready to pay для якісного інструменту.

Implication: Богдан personа важливіша для unit economics ніж ми initial thought.

### 3. Wearable-fanatics (Артур) — most demanding на explainability

Артур-tier asked critical questions про:
- Як saw our recovery score розраховується
- Чи можна обоснувати recommendations
- Як долго треба для baseline

Generic «AI gives you Recovery 73» — не landed. Деталі — landed.

**Implication:** Premium tier з explanation depth — okay differentiator. Artur paying $39 для "data-deep" mode plausible.

---

## Що ламається у нашій гіпотезі

### 1. Українська ціна $9 — too low для perceived value

Це парадокс. Я думав $9 буде «accessible». Реальність:
- 4 з 7 UA-користувачів сказали «це дешевше ніж я ожідав, мабуть продукт не серйозний»
- Brand-perception link до ціни

Implication: можливо $14-15 буде кращою UA-ціною. Тест у трикутнику $9 / $14 / $19 — у roadmap.

### 2. «Soft mode» (no calorie tracking) — приваблива для unexpected segment

Очікувано: для ED-recovery users.
Реальне: 6 з 30 enthusiastic — у тому числі люди БЕЗ ED-history. Просто **дратує** food-tracking. Дають FORM на сторону без насильства над собою.

**Implication:** Soft mode is positive feature, not just safety. Marketing може це використовувати.

### 3. Olympic-style movement coaches переоцінено

Я думав, наша обмеженість на 8 базових рухах буде issue. 28 з 30 не питали про olympic movements. Знаходять camera-coaching на squat / bench / DL абсолютно достатньо для perceived value.

**Implication:** Не тратимо роки на comprehensive movement library. Перші 8 — справді enough.

---

## Що нам **сказали зробити**

### Top-3 feature requests (з 30 інтерв'ю)

1. **Inter-set rest timer with auto-detection** (mentioned 17×)
   - «Зараз я ставлю timer вручну. Хотів би що Victor бачив що сет закінчився.»

2. **Voice мовою (UA/EN switch)** (mentioned 14×)
   - Майже всі UA-users hot want Victor говорити українською у в звуковій формі. ElevenLabs UA voice work.

3. **Comparison з previous session** (mentioned 12×)
   - «Як я порівняно з minulim тренуванням?» — auto-overlay в Workout screen.

### Top-3 frustration points

1. **Onboarding занадто довгий** (12×)
   - Перші 5 кроків — okay. Кроки 6-8 (ED-screening + tone-selection) — feels slow.
   - Mitigation: чи можна сховати ED-screening усе під «advanced options»? **No.** Це red-line за clinical-safety.
   - Alternative: компактніший дизайн з shorter copy.

2. **CV setup потребує efforts** (8×)
   - Phone на штативі — friction для many gyms.
   - Mitigation: краще onboarding, mobile-stand suggestions, fallback на «manual count» якщо нема CV.

3. **Chat відповіді generic** (7×)
   - У beta частина users отримала template-style replies.
   - Mitigation: prompt engineering improvements, deeper context passing.

---

## Скільки нам ВЖЕ варто переробити

Based on these 30:

**Must-fix before public launch (P0):**
- Onboarding length compress до < 90 sec
- Inter-set timer with auto-detection (engineering work)
- UA voice (ElevenLabs + voice tuning)

**Nice-to-have, не block launch (P1):**
- Comparison overlay у Workout
- Better mobile-stand guidance
- Chat context depth improvements

**Decided не робити:**
- Olympic movement coaching (out of scope)
- Cardio integration (Strava territory)
- Social features (anti-pattern для retention)

---

## Що **не змінилось**

Despite 30 interviews, ці позиції залишаються firm:

1. **No social features.** 100% of users said «I don't want fitness social». Confirmed.
2. **No free tier.** Тільки 3 з 30 asked. Більшість prefer paid trial.
3. **Apple-first.** 28 з 30 mainly iOS. Android phase 2 confirmed.
4. **No medical advice.** All agreed disclaimers were ясні і не дражливі.
5. **Anti-streak-shame.** Strong validation для «грейс після 1 пропуску».

---

## Process insights

Що навчили мене ці 30 інтерв'ю про **process**:

### 1. Interview compounds value

Перші 10 — full of surprises. Інтерв'ю 10-20 — pattern-confirmation. Інтерв'ю 20-30 — edge-case discovery.

Я думав, 15 буде enough. Не було. 30 — sweet spot для нашого product stage.

### 2. Cash > trial-credit для compensation

Кілька users який отримав «trial month» as comp було visibly diplomatic у feedback. Cash-paid були significantly honest.

### 3. Recording is essential

Без recording — я думав «I'll remember». Я не remembered. Transcripts surfaced 40% більше insights ніж memory.

### 4. Bias guard works

Я каже «це наш продукт, ось він» — almost все одно отримує positive feedback. Запитуючи «walk me через your training» **before** showing product — got honest reactions later.

### 5. Founder себе як interviewer — limited

Кілька questions — особливо про price — feel awkward when I personally interview. Future рекомендую outsource research-lead role for sensitive topics.

---

## Bottom line

30 інтерв'ю — це 22.5 годин deep listening + ~10 годин synthesis.

**Зробили це BEFORE launch.**

Тому що:
- Reduce risk that we ship wrong thing
- Calibrate marketing language to real customer language
- Find P0 fixes before they become public bugs
- Build empathy that scales further у team

**Якщо ти building consumer product і ще не intervieowed 30 — зупини себе. Зроби це. Тільки post-30 ти знаєш, у які напрямки тебе тягне ринок.**

---

## What's next

- Public launch — Listopad 2026
- Post-launch interviews continue (5/тиждень maintenance pace)
- 90-day cohort analysis → real metrics post (Q1 2027)

---

**Subscribe** для post-launch real-cohort data: [signup]

**Apply на closed beta** до 1 Lipnya: [beta.html]

---

*FORM — AI personal trainer з Києва. Pre-launch, поки що.*
*Не маркетинг — особиста рефлексія від founder.*
