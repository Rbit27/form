# FORM · Metrics & Analytics v0.1

> Що ми міряємо, що ми **не** міряємо, і як це нас рятує від оптимізації не того.
> Власник: `growth-lead`. Веб-dashboard live після M2.

---

## 1 · North-Star Metric

> **W-ACSU** · Weekly Active Coached Sessions per User

Сесія = подія, де **Victor доставив ≥ 1 coaching cue АБО ≥ 1 plan adjustment**.

### Чому саме це

| Альтернатива | Чому ні |
|---|---|
| DAU (daily active users) | Game-able через over-notifications, не reflects value |
| Minutes used | Engagement ≠ outcome, можна заохочувати залежність |
| Total workouts logged | Не різнить «coached workout» від «manual log» (Hevy суперечник) |
| Revenue/user | Lagging indicator, не actionable у short cycle |
| Retention | Важливий, але не показує **чому** користувачі повертаються |

W-ACSU фіксує **обидва** — користувач прийшов І ми справді коучили.

### Targets

| Phase | W-ACSU |
|---|---|
| Beta (M3) | ≥ 2.5 |
| Public launch (M6) | ≥ 3.0 |
| Mature (M12) | ≥ 3.5 |

---

## 2 · AARRR funnel (Pirate Metrics, адаптовано)

### Acquisition

| Metric | Target M6 | Tracking |
|---|---|---|
| Install → Open | ≥ 80% | App Store + PostHog |
| Open → Onboard | ≥ 65% | PostHog event chain |
| Onboard → Health permission | ≥ 50% (opt-in soft) | Native iOS prompt outcome |

**Cost per install (CPI):**
- Target M6: < $5 (organic-led)
- Target M12 (with paid): < $12

### Activation

| Metric | Target M6 | Тлумачення |
|---|---|---|
| Onboard → First Workout | ≥ 60% | within 72 hours |
| Onboard → First Coached Session | ≥ 35% | within 7 days |
| Onboard → 3 Coached Sessions | ≥ 25% | within 14 days = **activated** |

«Activated» — критична межа. Активовані users мають D30 retention ~2.5× vs non-activated.

### Retention

| Metric | Target M6 |
|---|---|
| D1 retention | ≥ 60% |
| D7 retention | ≥ 35% |
| D30 retention | ≥ 22% |
| D90 retention (paid only) | ≥ 70% |
| D365 retention (paid) | ≥ 35% |

**Cohort grouping:** install month + persona segment (Дмитро / Олена / Артур / Богдан).

### Revenue

| Metric | Target M6 |
|---|---|
| Trial → Paid conversion | ≥ 15% |
| Monthly ARPU (mix-adjusted) | ≥ $12 |
| Annual plan mix | ≥ 50% |
| Net Revenue Retention | ≥ 95% |

### Referral

| Metric | Target M6 |
|---|---|
| Viral coefficient (k-factor) | ≥ 0.15 (organic, без incentive) |
| Share rate on Progress screen | ≥ 8% of weekly actives |

---

## 3 · Health metrics (для team)

### Engagement health

- DAU/MAU ratio: target ≥ 0.4 (40% DAU/MAU = sticky)
- Avg session length: monitor, не optimize. Target band: 8–25 min.
- Sessions per active user/week: target ≥ 3

### Quality health

- Crash-free sessions: ≥ 99.5% (target ≥ 99.8% after M6)
- API response P95: < 1.5 sec
- CV inference P95: < 80 ms per frame

### Trust health

- Cancel rate by reason (categorized): monitor monthly
- Negative App Store reviews monitored (response within 48 hours)
- Support ticket avg resolution: < 24 hours

---

## 4 · Anti-metrics (не оптимізуємо)

Метрики, які ми **спеціально не цілимо**, бо вони ведуть до toxic patterns:

| Anti-metric | Чому ні |
|---|---|
| Total notifications sent | Більше спам ≠ більше value |
| Streak length | Гра на loss aversion = burnout |
| Calorie deficit days streak | ED-promoting |
| % users with body comp tracking on | Body fixation |
| Time spent in food logging | Frictionless ≠ healthier |
| Total push notification opens | Mass-marketing pattern |

---

## 5 · Dashboard layout (для weekly review)

```
┌─────────────────────────────────────────────────────────────┐
│ FORM · Week 24 · Dashboard                                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  NSM: W-ACSU                                                 │
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░  3.1  ↑ from 2.9               │
│                                                              │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐         │
│  │  Trial        │ │  D7 Retention │ │  MRR          │        │
│  │  18.2% conv  │ │  ↑ 37.4%      │ │  $48.6k       │        │
│  │  (↑ +1.2 pp) │ │  (target 35%) │ │  (↑ 8.4%)     │        │
│  └──────────────┘ └──────────────┘ └──────────────┘         │
│                                                              │
│  Activation funnel:                                          │
│  Install ───→ Open ───→ Onboard ───→ 1st Wo ───→ 3 Coached │
│   100%        82%        67%          61%          26%       │
│                                                              │
│  Cohort retention curves:                                    │
│  Apr 2027 ▁▂▃▃▃▃▃▃ (24 weeks)                                │
│  Mar 2027 ▁▂▃▃▃▃▃                                            │
│  Feb 2027 ▁▂▃▄▄                                              │
│                                                              │
│  Top 3 reasons for cancel (этого тижня):                     │
│  1. "Too expensive" (8)                                      │
│  2. "Doesn't fit my schedule" (4)                            │
│  3. "Form analysis missed too much" (3) ← REVIEW             │
│                                                              │
│  Quality alerts:                                             │
│  ✓ Crash-free 99.7%                                          │
│  ⚠ CV inference P95 92ms (target < 80)                       │
│  ✓ API P95 1.1s                                              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 6 · Alerting rules

**Automatic alerts → ops channel:**
- Crash rate > 0.3% in any 1-hour window
- API error rate > 5% in any 15-min window
- Anthropic / ElevenLabs API failures > 2% in 15 min
- Daily new sign-ups drop > 40% (DoD)
- D1 retention drops > 5 pp WoW

**Weekly digest → team:**
- All KPIs with trends
- Cohort additions
- Cancel reasons
- Top user feedback items

---

## 7 · Cohort analysis framework

Кожна когорта = users who **installed in the same week**.

Tracked metrics per cohort:
- Retention curve (D1, D7, D30, D60, D90, D180, D365)
- Activation rate (% reached 3 coached sessions in 14 days)
- Trial-to-paid conversion
- 12-month LTV (rolling)
- Avg sessions/week (engagement evolution)

Cohort comparisons:
- By acquisition channel (Reddit vs YouTube vs ASO vs direct)
- By onboarding tone choice (smart-direct vs mindful vs professor vs coach)
- By has-wearable vs no-wearable
- By geo (UA vs DE vs UK vs US)

Cohorts are **never combined** for averages — that hides shifts.

---

## 8 · Experiments (A/B test discipline)

### Rules

- Hypothesis written BEFORE experiment
- Pre-registered metric, sample size, duration
- Min sample size: 1000 users per cell
- Min duration: 14 days (capture weekday + weekend behavior)
- One experiment per surface at a time (no overlap)
- Documented in `/experiments/` folder з write-up

### Active experiment example

```
ID: EXP-024
Title: Default voice tone — smart-direct vs mindful-friend
Hypothesis: smart-direct converts trial-to-paid +2 pp,
            mindful-friend has higher D30 retention +3 pp.
Cells:
  A (control): smart-direct default (current)
  B (variant): mindful-friend default
Metric (primary): D30 retention by cell
Metric (secondary): Trial-to-paid by cell
Sample target: 2500 per cell
Duration: Mar 1 - Apr 14
Kill criteria: If either cell shows D7 < 28% by week 1
Status: RUNNING
```

---

## 9 · Privacy у analytics

Стандарт **pseudonymous, aggregated**:

- User ID — UUID, не email
- Events — без free-text payload (тільки enums)
- IP — truncated на edge level
- Geolocation — country only
- Device — model + OS major (iPhone 15, iOS 17)

Daily anonymization job:
- Delete event data older than 13 months
- Aggregate to monthly for retention purposes

User opt-out:
- One toggle у Settings → Privacy
- Respected immediately (event firing disabled)
- Past events not retroactively deleted (use Delete Account для цього)

---

## 10 · Що ми перевіряємо у кожній фічі

Перед shipping будь-якої фічі:

- [ ] Чи її успіх можна виміряти?
- [ ] Чи метрика **корелює** з NSM, не суперечить?
- [ ] Чи може ця фіча погіршити anti-metric (notifications, time-in-app)?
- [ ] Чи є kill-criteria — за яких чисел ми її відкочуємо?
- [ ] Чи інструментована перед launch?

Якщо ≥ 2 «ні» — фіча не готова.

---

## 11 · Open metric questions

1. **Чи W-ACSU справді corelates з retention?**
   - Hypothesis: yes, але треба перевірити після перших 1000 paid users
   - Якщо ні — переробити NSM

2. **Чи trial-to-paid 15% target реалістичний при «no credit card required»?**
   - Whoop досягає ~12%, але має sunk cost (band $30)
   - Можливо потрібен soft commitment у trial (наприклад, постановка ціль)

3. **Чи активність 3+/тиждень — це здорова межа чи burnout-driver?**
   - Hypothesis: оптимум 3-4. < 2 = lapsed, > 5 = burnout risk
   - Перевіримо за реальною кореляцією Cancel-causes

---

**v0.1 · травень 2026 · правки приймає `growth-lead`**
