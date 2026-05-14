# FORM · Beta Playbook v0.1

> Як run closed beta з 200 ліфтерів. Operationally.
> Власник: founder + `process-keeper`.

---

## 1 · Цілі beta

Один primary, кілька secondary.

### Primary
> **Validate or kill product-market fit on serious self-coached lifters.**
> Метрика: D30 active rate ≥ 35%. Якщо нижче — переробляємо позиціонування.

### Secondary
1. Знайти top-5 friction points у onboarding
2. Виміряти CV-quality на реальних gym-conditions
3. Calibrate Victor voice tone (yes мама-friend? холодний?)
4. Розрахувати real LTV-сигнали для unit econ
5. Зібрати 50+ unique quotes для marketing post-launch

---

## 2 · Структура beta

| Phase | Дати | Хто | Що |
|---|---|---|---|
| **Wave 0** | Aug 5-12 | 25 «hand-picked» (Reddit DM, friends-of) | Day-1 readiness; smoke-test |
| **Wave 1** | Aug 15-25 | +75 from beta.html applications | Full features; chat live |
| **Wave 2** | Sep 1-10 | +100 from applications + waitlist | Scale stress test |

Total: 200 active.

---

## 3 · Beta-учасник journey

### Day 0 (TestFlight invite sent)

- Email with TestFlight link + Slack invite
- Setup tutorial video (5 хв)
- Day-1 quick guide PDF (one-pager)

### Day 1-3 (onboarding)
- Bot in Slack pings: «Завершив onboarding?»
- If stuck > 24h — direct DM from founder
- Goal: 90% повний onboarding до D3

### Day 7 (first checkpoint)
- Auto-email B-1 (week 1 check-in)
- Goal: hear from 50%+ users by D10

### Day 14 (data-pattern emerging)
- Auto-email B-2 (baseline established)
- Slack post з aggregate stats (anonymized)

### Day 28 (founder call invite)
- B-3 email
- Goal: 40-60 calls happen between D28-D60

### Day 56 (mid-beta)
- B-4 deep check email
- Founder analyzes top friction points → ship-cycle priorities

### Day 84-90 (beta closing)
- B-5 ending email + locked-in pricing offer
- Cohort-retention analysis
- Goal: 60%+ convert to paid

---

## 4 · Daily ops (для founder + team)

### Кожен день

- [ ] Check Slack channel — respond to anything overnight
- [ ] Review TestFlight crash reports (Sentry digest 09:00)
- [ ] Read overnight email replies, respond < 24h
- [ ] Spot-check dashboard: errors, latency, signups

**Time budget:** 60-90 хв/день.

### Кожен тиждень

- [ ] Beta digest у Slack (Friday 17:00): what shipped, what's next
- [ ] Cohort retention update
- [ ] Top-5 user pain points → prioritized
- [ ] One-on-one calls scheduled (target: 4-6/тиждень mid-beta)

**Time budget:** 4-6 годин додаткові у тиждень.

### Кожен місяць

- [ ] Cohort write-up (data + qualitative)
- [ ] Update internal Decision Log (process-keeper)
- [ ] Investor monthly update (use template з INVESTOR_OUTREACH)
- [ ] Public build-in-public summary (для Twitter)

---

## 5 · Beta Slack structure

```
#announcements        - founder-only post (1-2/week)
#general              - open discussion
#feature-requests     - bug-reports і wishes
#wins                 - PR-моменти користувачів (опційно)
#dms-with-team        - DM-доступ до founder + design + eng
#feedback-victor      - конкретно про voice and chat
#feedback-cv          - про camera analysis specifically
```

**Rules:**
- No company-marketing у #announcements (тільки product, eng, ops updates)
- Founder responds у DMs within 24h
- Weekly digest pinned

---

## 6 · Feedback collection methods

### A. In-app feedback widget
- One-tap: «Поточний екран — feedback?»
- Auto-attaches: screen, version, user-id
- Goes to PostHog + Slack #feedback channel

### B. Slack #feedback channel
- Open, public у beta-Slack
- Founder reads daily

### C. Email replies
- All emails go to single beta@form.coach
- Triaged by founder daily

### D. User interviews (30-хв calls)
- Optional after week 4
- Target: 40-60 calls total
- Recorded with consent, transcribed, tagged

### E. Quarterly survey
- One after week 6, one at end of beta
- Max 12 questions
- Mix of NPS + open-ended

---

## 7 · Bug і issue handling

### Priority tiers

| Tier | Definition | Response time | Example |
|---|---|---|---|
| P0 | Blocks usage (crash, can't log in) | < 4 hours | App won't start after iOS update |
| P1 | Major feature broken | < 24 hours | CV не detect rep counts |
| P2 | Minor bug or UX issue | < 7 days | Wrong color on one button |
| P3 | Cosmetic, polish | When time allows | Slight animation jank |

### Process

1. Bug reported (Slack, email, in-app)
2. Triaged by founder (15-min daily window)
3. Logged у Linear (or similar)
4. Fixed → TestFlight new build
5. Reporter notified personally («fixed in build X»)

**Important:** every P0/P1 reporter gets a personal thank-you message. Builds trust.

---

## 8 · Communication discipline

### Cadence

- Slack #announcements: 1-2/week max. Less is better.
- Email B-sequence: as scheduled
- Direct DMs: as-needed (founder ≥ 1/тиждень з 50+ active users)

### Tone

- Personal first-person («Я думаю», not «we think»)
- Acknowledge bugs OPENLY before pivot («yes, the rep counter missed на squat. Тут чому, тут fix»)
- Never blame users («ти неправильно зайняв» — never)
- Never deflect («Apple API issue» — explain не виправдовуй)

---

## 9 · «Hard» metrics dashboard (для weekly review)

```
┌──────────────────────────────────────────────────┐
│ FORM Beta · Week 4                                │
├──────────────────────────────────────────────────┤
│                                                   │
│ Active beta users:           187 / 200            │
│ Day-7 retention (this cohort): 71%                │
│ Day-14 retention:             58%                  │
│ Day-30 retention:             41% (target 35%) ✓ │
│                                                   │
│ Sessions per user/week:        3.2                │
│ Avg session duration:         52 хв               │
│ W-ACSU (NSM):                 2.8                  │
│                                                   │
│ Crash rate:                  0.3% (target <0.5%) ✓│
│ CV success rate:              82%                  │
│ NPS:                          47 (target >40) ✓   │
│                                                   │
│ Top friction (this week):                          │
│ 1. CV fails on hex bar deadlift (4 reports)      │
│ 2. Rest timer doesn't pause on phone lock (3)    │
│ 3. Tone "national coach" мало responsive (3)     │
│                                                   │
└──────────────────────────────────────────────────┘
```

---

## 10 · «Soft» metrics (тільки якісно)

Зберігаємо у тематичній doc-base, не у dashboard:

- Top-10 quotes за тиждень (anonymized)
- Story-arcs: де користувач показав before/after «aha» moment
- Tone feedback patterns
- Vocabulary користувачі використовують → перетікає у наш copy

---

## 11 · Кризи (що робити коли)

### CV-якість виявилась слабкою на 3+ рухах

1. Stop coaching tone з CV-recommendations для тих рухів
2. Honest message: «CV-аналіз для X — наразі beta, відключено»
3. Fix у наступному ship-циклі
4. Communicate у Slack відкрито

### Користувач repository ED-trigger

1. Clinical-safety протоколи активуються (auto-soft-mode)
2. Founder personally reaches out з resources
3. Не push до cancel; sustain access у soft-mode
4. Document (anonymized) for future product improvement

### Major iOS update breaks app

1. P0 priority — все hands on deck
2. Slack #announcements: «Working on it, ETA X hours»
3. Daily updates до fix
4. Post-mortem published once resolved

### Беттер уходить (cancel during beta)

1. Особистий email від founder: «Що сталось?»
2. Не arguing; learning
3. Якщо причина actionable — фікс плануємо
4. Якщо not — note for marketing post-launch (хто не fit)

---

## 12 · Що ми **не** робимо у beta

- ❌ Hide bugs / pretend the product is more polished than it is
- ❌ Tell users they're using it «wrong»
- ❌ Send bulk announcements about features that don't work yet
- ❌ Pressure beta users to convert before they're ready
- ❌ Treat feedback as «just user opinions»
- ❌ Compare beta-users between each other publicly
- ❌ Use beta as «free QA team» — це partnership

---

## 13 · Post-beta retro

T+90 days from beta start:

- All-cohort retention writeup
- Top-30 insights consolidated
- Voice-of-customer report (quotes mapped to themes)
- Decision Log update from process-keeper
- Public blog post «What we learned in beta» (with permission)
- Investor update with metrics

If primary metric missed (D30 < 35%):
- Stop. Re-design positioning.
- Don't push to public launch with broken thesis.
- Take 6-8 weeks to address.

If primary metric hit:
- Public launch ready
- Lock-in beta pricing for participants
- Spin marketing on top of validated message

---

## 14 · Open questions

1. **Як handle Russian-speaking users у beta?**
   - UA target обмежений до Ukrainian first wave
   - Інші regions can use English UI
   - Russian language UI — окреме рішення post-launch

2. **Як уникнути beta-user fatigue?**
   - Weekly check-ins можуть стати спамом
   - Hypothesis: 1 email/тиждень окей при високій value content

3. **Compensation за referrals?**
   - Кожен beta-user привесть 1 friend → both get extra free month
   - Cap: 2 referrals per user

---

**v0.1 · травень 2026 · founder + `process-keeper`**
