# FORM · Unit Economics & Financials v0.1

> Гіпотези, не доведено. Цифри — це **target model**, не traction.
> Власник: `growth-lead`. Sanity-check: `product-strategist`.

---

## 1 · Unit Economics (target, M6)

| Метрика | Western | UA / EE | Comment |
|---|---|---|---|
| ARPU (monthly equivalent) | $14.50 | $7.50 | Зважено на mix (60% annual / 40% monthly) |
| Monthly churn (post-trial) | 4.5% | 6.0% | UA churn вищий — поки причина гіпотеза |
| Avg customer lifetime (months) | 22 | 17 | 1 / monthly churn |
| **LTV** | **$320** | **$128** | ARPU × lifetime |
| Gross margin | 78% | 78% | Same cost structure |
| **LTV (gross-adjusted)** | **$250** | **$100** | LTV × 78% |
| CAC (blended) | $50 | $20 | Реалістично після M4 |
| **LTV/CAC** | **5.0×** | **5.0×** | Target ≥ 3× для здоров'я, ≥ 5× для росту |
| Payback period | 4.2 міс | 3.3 міс | < 12 міс — здорово |

### Що б нас вбило

| Сценарій | Якщо реально |
|---|---|
| Monthly churn > 8% | Платний marketing нерентабельний |
| ARPU < $10 (Western) | Annual plan не приймається ринком |
| CAC > $80 | Канали не працюють — органіка не масштабується |
| Gross margin < 65% | LLM-кост вище очікуваного, треба урізати features |

---

## 2 · Cost структура (на 1 active subscriber, monthly)

| Item | Cost/місяць/користувач | Як рахуємо |
|---|---|---|
| **LLM (Anthropic API)** | $1.20–2.00 | Sonnet 4.6 chat + Haiku classification + caching |
| **TTS (ElevenLabs)** | $0.40–0.80 | Cached library + 30 dynamic cues |
| **Supabase** | $0.05 | Free tier до 50k MAU, потім $25/місяць flat |
| **Cloudflare Workers** | $0.02 | Workers Paid $5/міс + per-request |
| **Apple/Google payment** | 15–30% від доходу | Apple 15% після року, Google аналогічно |
| **Sentry + PostHog** | $0.10 | Sub-$200/міс на 5k users |
| **Total infra cost** | **~$1.80–3.00** | без payment processing |

### Margin розрахунок (Western $19/міс)

```
ARPU:                        $14.50  (mix-adjusted)
Apple/Google fee (avg 18%):  -$2.60
Infra:                       -$2.40  (середнє)
Support / ops allocation:    -$0.70  (50k user база)
─────────────────────────────────────
Gross profit per user:       $8.80
Gross margin:                60.7%
```

**Wait — 60.7%, а ми кажемо 78%?**

Чесно: 78% — це **до Apple/Google fees**. Після них — ~60–65%. Це нормальний SaaS-margin (Spotify ~25%, Netflix ~40%, Calm ~55%). Для AI-продукту з LLM-витратами це адекватно.

---

## 3 · Revenue projection (3-year, optimistic-but-defensible)

| Period | Paid users | MRR | ARR | Burn | Net |
|---|---|---|---|---|---|
| **Q3 2026** (launch) | 200 (beta) | $0 | $0 | $35k | -$35k |
| **Q4 2026** | 1,500 | $20k | $240k | $80k | -$60k |
| **Q1 2027** | 5,000 | $65k | $780k | $120k | -$55k |
| **Q2 2027** | 12,000 | $155k | $1.86M | $160k | -$5k |
| **Q3 2027** | 22,000 | $285k | $3.42M | $200k | +$85k |
| **Q4 2027** | 38,000 | $495k | $5.94M | $280k | +$215k |
| **2028 Q4** | 90,000 | $1.18M | $14.1M | $700k | +$480k |

### Що це означає

- **Beak-even**: orientовно Q3 2027 (15 міс після launch)
- **$10M ARR**: M21 (mid 2028) при cautious-aggressive scenario
- **$25M ARR**: M30 (Q1 2029) якщо все працює

### Що НЕ ввімкнено в модель

- B2B/enterprise pilot revenue (потенційно $200–500k ARR від M12)
- International beyond UA + EU (German + Spanish + Polish phase 2)
- Premium tier ($39/міс з human-trainer call) — апсайд +20% ARPU

---

## 4 · Funding plan

### Seed · $1.5–2.5M (target close: Aug 2026)

| Use of funds | Amount | Що даємо |
|---|---|---|
| Engineering (3 hires × 18 міс) | $700k | iOS + Android + backend |
| CV R&D budget | $400k | 8-rух pipeline + iteration |
| Design + production | $300k | Brand assets, video, motion |
| Infra + APIs (LLM, TTS, cloud) | $200k | 18-month runway |
| First-touch marketing | $250k | Creator partnerships, content production |
| Ops + legal + buffer | $400k | Legal counsel, accounting, contingency |
| **Total** | **$2.25M** | 18 months at planned burn |

### Series A · $8–12M (target close: Q4 2027)

Trigger metrics:
- ARR ≥ $3M
- D90 paid retention ≥ 65%
- NPS ≥ 50
- D30 free-to-paid ≥ 15%

Use:
- US/UK market entry
- 12–15 additional hires (team scales to ~25)
- Paid acquisition test budget ($1–2M)
- Series B prep

---

## 5 · Sensitivity Analysis

### Якщо CAC виявиться вищим (paid scaling складніше)

| CAC scenario | LTV/CAC | Payback | Decision |
|---|---|---|---|
| $50 (target) | 5.0× | 4.2 міс | Healthy |
| $80 (challenge) | 3.1× | 6.6 міс | Слабше, але живе |
| $120 (real bad) | 2.1× | 9.9 міс | Тільки органіка масштабує |
| $200 (catastrophe) | 1.25× | 16.6 міс | Канали не працюють |

### Якщо ARPU нижчий (annual mix або UA-only)

| ARPU scenario | LTV | LTV/CAC@$50 | Comment |
|---|---|---|---|
| $14.50 (target) | $320 | 6.4× | Mix-adjusted base |
| $12 (more monthly) | $264 | 5.3× | Annual mix нижчий |
| $9 (UA dominant) | $198 | 4.0× | Western expansion slow |
| $7 (UA-only) | $154 | 3.1× | Geo-pricing цінит вгору |

---

## 6 · Що ми НЕ робимо

- **Lifetime deals** ($199 once) — убиває LTV, ризикує перевагу
- **Free tier** — MyFitnessPal проблема: free-rider economics
- **Family plan** до Series A — disсount-bait без чітких use cases
- **Advertising revenue** — incompatible з brand-floor (no shame, no surveillance)
- **Affiliate / referral fees > 1 free month** — fitness-mafia анти-патерн

---

## 7 · Key Metrics & Dashboard (для board reporting)

**Monthly:**
- Paid users + new + churned
- MRR / ARR + MoM growth %
- Net Revenue Retention
- D30 / D90 retention
- NPS / CSAT
- CAC за каналом
- LTV (rolling cohort)

**Quarterly:**
- Cohort retention curves (every cohort, 12-month rolling)
- Margin trend (gross, contribution, EBITDA)
- Channel mix (organic vs paid vs referral)
- Operating efficiency (revenue per employee, burn multiple)

**Anti-metric we don't celebrate:**
- DAU (gameable through over-notifications)
- Total notifications sent
- Total minutes in app
- Total food entries

---

## 8 · Open Financial Questions

1. **Чи Apple виставить fee 15% з першого року чи 30%?**
   - Small Business Program: 15% з дня одного якщо AppStore Connect-revenue < $1M/рік
   - Probable у v0.1, переходить на 30% при scale (∼$1.5M ARR)

2. **Geo-pricing risks?**
   - EU vs UA — фактор 2.1× різниці. VPN-arbitrage можливий.
   - Mitigation: account-bind to first-login country, no easy switch

3. **Currency hedging?**
   - Revenues USD/EUR, costs USD (Anthropic, ElevenLabs, AWS) — natural hedge
   - Team в UAH — потрібно hedge при $5M+ ARR

4. **Tax structure?**
   - Ukrainian Дiya City statut (special tax regime for IT) для team
   - Delaware C-Corp для investor relationships
   - Open: чи створювати Estonian OÜ як holdco для EU compliance

---

**v0.1 · травень 2026 · правки приймає `growth-lead`**
**Чесний disclaimer:** усе вище — модель, не результат. Перевірені цифри з'являться після TestFlight beta та перших 1000 платних.
