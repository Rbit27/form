# FORM · Cost Model & Unit Economics v0.4

> Owner: data-engineer + founder. Review: monthly pre-launch, quarterly post-launch. Audience: founder, investors, future CFO.

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Assumptions and Methodology](#2-assumptions-and-methodology)
3. [COGS Breakdown per Tier](#3-cogs-breakdown-per-tier)
4. [Unit Economics Summary](#4-unit-economics-summary)
5. [Break-Even Analysis](#5-break-even-analysis)
6. [Gross Margin Targets](#6-gross-margin-targets)
7. [Scaling Economics](#7-scaling-economics)
8. [Enterprise Economics](#8-enterprise-economics)
   - 8.5 Enterprise CAC Model
   - 8.6 LTV/CAC Analysis
   - 8.7 Expansion and Churn Economics
   - 8.8 Year 1 vs. Year 2+ Deal Economics
9. [App Store Tax Impact](#9-app-store-tax-impact)
10. [Sensitivity Analysis](#10-sensitivity-analysis)
    - 10.5 Enterprise Revenue Mix Sensitivity
11. [Open Questions / Gaps](#11-open-questions--gaps)
12. [Free Tier Subsidy Model & Freemium Funnel Economics](#12-free-tier-subsidy-model--freemium-funnel-economics)

---

## 1. Purpose

This document models the cost of delivering FORM to one user for one month, segmented by pricing tier. It exists to:

- Give the founder a defensible answer to "what does a user cost us?" before Series A diligence
- Surface the highest-leverage cost reduction levers (currently: Anthropic API and ElevenLabs)
- Set gross margin targets that investors in AI-native consumer SaaS will benchmark against
- Track how unit economics improve (or deteriorate) as volume scales

This is a **model, not actuals**. Every line is marked [ESTIMATE] until replaced with a reconciled invoice figure. The first real cost data will come from the closed beta cohort. Until then, treat all numbers as planning inputs, not commitments.

Cross-references: `docs/FINANCIALS.md` (revenue projections), `docs/METRICS.md` (W-ACSU, activation funnel), `docs/ENTERPRISE.md` (B2B pricing), `docs/ASSUMPTIONS_REGISTER.md` (what we are still testing).

---

## 2. Assumptions and Methodology

### 2.1 Baseline behavioral assumptions

These drive every cost line. If behavior deviates, recalculate from here.

| Assumption | Free tier | Pro tier | Enterprise seat | Source |
|---|---|---|---|---|
| Weekly active sessions (WAU) | 1.5 [ESTIMATE] | 3.5 [ESTIMATE] | 3.0 [ESTIMATE] | Target W-ACSU from METRICS.md; free users less engaged |
| Sessions per month (× 4.3 weeks) | 6.5 | 15.1 | 12.9 | Derived |
| Avg tokens input per session (Anthropic) | 2,000 [ESTIMATE] | 2,000 [ESTIMATE] | 2,000 [ESTIMATE] | Prompt = system context ~1,200 + user turn ~500 + history ~300 |
| Avg tokens output per session (Anthropic) | 500 [ESTIMATE] | 500 [ESTIMATE] | 500 [ESTIMATE] | Coaching cue + plan update; Victor is concise by design |
| Voice (ElevenLabs) adoption | 10% [ESTIMATE] | 40% [ESTIMATE] | 35% [ESTIMATE] | Free users hit quota cap faster; Pro users more likely to enable |
| Avg voice chars per voice-enabled session | 800 [ESTIMATE] | 1,200 [ESTIMATE] | 1,000 [ESTIMATE] | Coaching cues only; not full narration |
| Supabase storage per user (MB) | 15 | 80 | 60 | Workout history, meal logs, CV pose snapshots (thumbnails only) |
| Cloudflare Workers requests per session | 12 [ESTIMATE] | 18 [ESTIMATE] | 16 [ESTIMATE] | Auth, sync, LLM proxy, push, 2-3 read endpoints per session |
| Sentry/PostHog amortized user base (M12) | 5,000 MAU | 5,000 MAU | 5,000 MAU | Shared infra; amortized across all users |

### 2.2 Pricing inputs used

| Vendor | Rate | Notes |
|---|---|---|
| Anthropic Claude Sonnet-class | $3.00 / 1M input tokens, $15.00 / 1M output tokens | Current list price; no volume discount assumed pre-Series A |
| ElevenLabs | $0.30 / 1,000 characters | Creator/Business tier estimated; free tier has monthly cap |
| Supabase | $25/month Pro plan (up to 8 GB DB, 250 GB bandwidth) | Amortized across active users; switches from $0 free tier at ~500 MAU |
| Cloudflare Workers | $0.50 / 1M requests (above free 100k/day) | Workers Paid plan $5/month base included in fixed overhead |
| Sentry | ~$26/month Team plan | Amortized; covers up to ~50k events/day |
| PostHog | Free up to 1M events/month, then ~$0.00031/event | Amortized at 5k MAU × ~200 events/user/month ≈ 1M events → near-free |
| Apple/Google IAP | 30% standard; 15% Small Business Program (<$1M annual revenue) | Applies to consumer tiers only; enterprise is direct invoice |

### 2.3 What this model does not include

- **Human support labor** — ops allocation (~$0.50–0.70/user/month at scale) is noted in FINANCIALS.md but excluded here to isolate infrastructure COGS
- **R&D / CV pipeline inference** — on-device; no marginal cloud cost per user
- **CDN bandwidth for static assets** — covered by Cloudflare free tier at current scale
- **Idle compute** — Supabase and Workers have effectively zero idle cost at this architecture

---

## 3. COGS Breakdown per Tier

All figures are **per user per month**. [ESTIMATE] on every line until replaced with invoice data.

### 3.1 Anthropic API cost calculation

```
Input cost  = (tokens_input_per_session × sessions_per_month × $3.00) / 1,000,000
Output cost = (tokens_output_per_session × sessions_per_month × $15.00) / 1,000,000
```

| Tier | Sessions/month | Input tokens total | Input cost | Output tokens total | Output cost | Total API cost |
|---|---|---|---|---|---|---|
| Free | 6.5 | 13,000 | $0.039 | 3,250 | $0.049 | **$0.088** |
| Pro | 15.1 | 30,200 | $0.091 | 7,550 | $0.113 | **$0.204** |
| Enterprise | 12.9 | 25,800 | $0.077 | 6,450 | $0.097 | **$0.174** |

### 3.2 ElevenLabs voice cost calculation

```
Voice cost = voice_adoption_rate × sessions_per_month × avg_chars_per_session × ($0.30 / 1,000)
```

| Tier | Adoption | Chars/session | Sessions/month | Monthly voice cost |
|---|---|---|---|---|
| Free | 10% | 800 | 6.5 | **$0.016** |
| Pro | 40% | 1,200 | 15.1 | **$0.218** |
| Enterprise | 35% | 1,000 | 12.9 | **$0.135** |

### 3.3 Full COGS table

| Cost item | Free user/month | Pro user/month | Enterprise seat/month |
|---|---|---|---|
| **Anthropic API** [ESTIMATE] | $0.09 | $0.20 | $0.17 |
| **ElevenLabs voice** [ESTIMATE] | $0.02 | $0.22 | $0.14 |
| **Supabase** (compute + storage + bandwidth, amortized) [ESTIMATE] | $0.01 | $0.05 | $0.04 |
| **Cloudflare Workers** (requests, amortized) [ESTIMATE] | $0.01 | $0.02 | $0.02 |
| **Sentry** (amortized at 5k MAU) [ESTIMATE] | $0.005 | $0.005 | $0.005 |
| **PostHog** (amortized, near-free tier) [ESTIMATE] | $0.002 | $0.002 | $0.002 |
| **Total infrastructure COGS** | **$0.13** | **$0.50** | **$0.34** |

### 3.4 COGS notes

**Free tier:** Cost is minimal because sessions are fewer and voice adoption is low. The business risk is not cost-per-user but conversion rate — a free user who never converts to Pro is $0.13/month of pure cost with no revenue offset. Free tier must be designed to drive activation, not just engagement.

**Pro tier:** Anthropic API and ElevenLabs together account for ~84% of infrastructure COGS. This is the primary lever to manage. Prompt engineering, caching, and model tiering (Haiku for classification, Sonnet for coaching turns) can materially reduce this.

**Enterprise seat:** Slightly lower than Pro due to modeled lower session frequency (enterprise users may log workouts less consistently than self-motivated Pro subscribers) and lower voice adoption assumption. Direct billing eliminates app store fees.

---

## 4. Unit Economics Summary

| Metric | Free | Pro | Enterprise seat |
|---|---|---|---|
| ARPU (monthly) | $0 | $19.00 (Western) | $30.00 [ESTIMATE, midpoint of $25–35 range] |
| App store fee (15% SBP / 30% standard) | — | $2.85 (15%) → $5.70 (30%) | $0 (direct invoice) |
| Net revenue after store fee | $0 | $13.30–$16.15 | $30.00 |
| Infrastructure COGS/user/month | $0.13 | $0.50 | $0.34 |
| Gross profit/user/month | -$0.13 | $12.80–$15.65 | $29.66 |
| Gross margin % | n/a (no revenue) | 67.4%–82.4% | 98.9% |
| Contribution margin (after COGS + store fee) | -$0.13 | $12.80–$15.65 | $29.66 |

**Gross margin range on Pro** reflects App Store fee uncertainty: 15% Small Business Program applies while annual revenue is below $1M; transitions to 30% above that threshold. See Section 9.

**Geo-pricing note:** ARPU above is the primary Western market price ($19/month). UA/EE/PL geo-pricing ($9/month) applies where App Store detects those regions; at $9 ARPU with SBP 15%, infrastructure gross margin = ($9 × 0.85 − $0.50) / $9 = 79.4%. Enterprise deals in all geographies are billed direct at $6–12/seat/month, independent of consumer geo-pricing.

**Enterprise gross margin** is extremely high on COGS alone because the per-seat API cost is low relative to ACV. The real cost of enterprise is implementation and customer success labor, which is modeled in Section 8 and excluded from this infrastructure COGS table by design.

---

## 5. Break-Even Analysis

### 5.1 Fixed monthly infrastructure overhead

These costs exist regardless of user count (at launch scale):

| Item | Monthly cost |
|---|---|
| Supabase Pro plan | $25 |
| Cloudflare Workers Paid base | $5 |
| Sentry Team plan | $26 |
| PostHog (within free tier at launch) | $0 |
| **Total fixed overhead** | **$56/month** |

### 5.2 Break-even formula

```
Break-even Pro subscribers =
    Fixed overhead / (Net revenue per Pro user - Variable COGS per Pro user)

= $56 / ($16.15 - $0.50)    [using Western $19 ARPU, 15% SBP app store rate]
= $56 / $15.65
= ~4 Pro subscribers to cover fixed overhead
```

At 30% app store fee:

```
= $56 / ($13.30 - $0.50)
= $56 / $12.80
= ~5 Pro subscribers
```

**Interpretation:** Fixed infrastructure overhead is negligible. The real break-even question for FORM is not "how many users to cover servers" but "how many Pro subscribers to cover total burn" — which includes team salaries and is modeled in `docs/FINANCIALS.md`. Infrastructure break-even is effectively solved by a handful of subscribers.

### 5.3 Burn-rate break-even (for reference)

From FINANCIALS.md, target burn at launch is ~$80k/month. At $15.65 net contribution per Pro user (Western $19 ARPU, SBP 15%):

```
Pro subscribers needed to break even on full burn = $80,000 / $15.65 ≈ 5,112 Pro subscribers
```

This is the number that matters operationally.

---

## 6. Gross Margin Targets

### 6.1 Industry benchmarks

| Company / category | Reported gross margin | Notes |
|---|---|---|
| Calm | ~55% [ESTIMATE, public reports] | Consumer subscription, content-heavy |
| Noom | ~60% [ESTIMATE] | AI-assisted coaching, human coach layer |
| Whoop | ~45% [ESTIMATE] | Hardware + software mix; hardware drags margin |
| Pure SaaS median | 70–80% | Software only, no AI inference cost |
| AI-native SaaS (early stage) | 50–70% [ESTIMATE] | API costs before volume discounts |
| **FORM target (Pro, post-SBP)** | **75–85%** | After app store fees, before support labor; Western $19 ARPU |
| **FORM target (Enterprise)** | **>85%** | After implementation cost, on mature accounts |

### 6.2 FORM gross margin path

| Phase | Pro GM % | Driver |
|---|---|---|
| Beta (0–500 users) | ~79% | Western $19 ARPU, SBP 15% fee, ~$0.70/user support allocation, no volume discounts |
| Launch (500–5k users) | ~79–82% | SBP holds; prompt caching begins to reduce Anthropic cost |
| Growth (5k–50k users) | ~82–85% | Volume pricing discussions with Anthropic; PostHog moves to paid tier |
| Scale (50k+ users) | ~84–87% | Committed spend discounts; model fine-tuning reduces token usage; support cost amortized |

**Note on the 78% figure in FINANCIALS.md:** That figure is gross margin before App Store fees, which is a legitimate SaaS accounting presentation (treating store fees as a distribution cost rather than COGS). At Western $19 ARPU, after 15% SBP store fees, the realized margin is 63–69% (infrastructure COGS + support labor blended). Both figures will be disclosed transparently in investor materials, labeled appropriately.

---

## 7. Scaling Economics

### 7.1 How per-user COGS changes with scale

| User count (Pro MAU) | Anthropic cost/user | ElevenLabs cost/user | Supabase cost/user | Total COGS/user | Notes |
|---|---|---|---|---|---|
| 1,000 | $0.20 | $0.22 | $0.05 | $0.50 | List pricing; no discounts |
| 10,000 | $0.18 [ESTIMATE] | $0.20 [ESTIMATE] | $0.04 | $0.45 | Anthropic committed spend tier likely; Supabase scales smoothly |
| 100,000 | $0.13 [ESTIMATE] | $0.15 [ESTIMATE] | $0.03 | $0.33 | Volume discounts negotiated; caching hit rate improves; model tiering mature |

### 7.2 Key scaling levers

**Anthropic API — highest impact**

- **Prompt caching:** Anthropic supports caching of static system prompt content. Victor's base persona + safety rules + exercise library context (~800 tokens) can be cached, reducing effective input cost by ~35–40% on the static portion. [ESTIMATE: reduces total API cost by $0.03–0.06/user/month at current usage]
- **Model tiering:** Use Claude Haiku (significantly cheaper) for intent classification, set selection, and simple acknowledgments. Reserve Sonnet for actual coaching turns and program adaptation. [ESTIMATE: reduces API cost by 20–30% if 50% of calls route to Haiku]
- **Committed spend discounts:** Anthropic offers volume pricing at meaningful scale. Negotiations typically begin around $50k–100k/month in API spend, which FORM reaches at roughly 30k–50k active Pro users at current usage rates. [ESTIMATE]

**ElevenLabs — second highest impact**

- **Audio caching:** Pre-generate and cache common coaching cues ("Good depth on that squat", "Elbows up", "Brace your core"). The long tail of Victor's vocabulary is finite. [ESTIMATE: 60–70% of voice output is cacheable common cues, reducing effective voice cost by ~50%]
- **Tier upgrade economics:** ElevenLabs Business/Enterprise tiers offer better per-character rates at volume. The crossover depends on negotiated terms.

**Supabase — low impact, well-contained**

- Supabase Pro ($25/month) handles up to 8 GB database and 250 GB bandwidth — sufficient for ~10k–15k MAU at current data model. At higher scale, upgrade to Team or Enterprise plan. Cost per user drops at scale because the fixed plan cost is amortized across more users.
- Read replica usage for analytics queries (instead of hitting primary) prevents compute cost from spiking during dashboard loads.

**PostHog — crosses into paid at ~1M events/month**

- At 5k MAU × ~200 events/user/month = 1M events — right at the free tier boundary.
- At 10k MAU: ~2M events, estimated $310/month additional. Amortized cost: $0.031/user/month. Still negligible.
- At 100k MAU: ~20M events, estimated ~$5,500/month. Amortized: $0.055/user/month. Material but manageable.

---

## 8. Enterprise Economics

### 8.1 Revenue and COGS per enterprise deal

Minimum deal size: 20 seats at $25–35/seat/month = $500–700/month per deal, billed annually = $6,000–8,400 ACV.

| Metric | Min deal (20 seats, $25/seat) | Mid deal (100 seats, $30/seat) | Large deal (500 seats, $35/seat) |
|---|---|---|---|
| Monthly recurring revenue | $500 | $3,000 | $17,500 |
| Annual contract value (ACV) | $6,000 | $36,000 | $210,000 |
| Monthly infrastructure COGS (@ $0.34/seat) | $6.80 | $102 | $595 |
| Infrastructure gross margin | 98.6% | 96.6% | 96.6% |

### 8.2 Implementation and onboarding cost

Enterprise deals carry one-time and recurring non-infrastructure costs that consumer tiers do not:

| Cost item | One-time per deal | Ongoing per month | Notes |
|---|---|---|---|
| SSO/SCIM integration and testing | $800–1,500 [ESTIMATE] | $0 | Engineering time; amortizes across deal life |
| Admin dashboard onboarding and training | $200–400 [ESTIMATE] | $0 | Customer success time |
| Custom SLA / compliance review | $300–800 [ESTIMATE] | $0 | Legal + compliance-officer time; lower for repeat buyer type |
| Dedicated CSM time (ongoing) | — | $150–400 [ESTIMATE] | Scales with seat count; ~2 hrs/month per account at mid-size |
| **Total implementation (mid deal)** | **~$1,500** | **~$250** | Payback on $36k ACV deal: < 2 months |

### 8.3 Minimum viable enterprise deal

For enterprise to be worth the GTM motion at this stage:

```
Minimum ACV to justify sales + implementation overhead: ~$12,000/year [ESTIMATE]
= 100 seats at $10/seat/month (floor), or
= 40 seats at $25/seat/month (our floor pricing)
```

Deals below 20 seats are currently declined — the implementation overhead is not recoverable. This threshold is reviewed when enterprise sales tooling matures (post-Series A).

### 8.4 Enterprise vs. consumer margin comparison

| | Pro consumer | Enterprise (mid deal) |
|---|---|---|
| ARPU | $19/month (Western) | $30/seat/month |
| Store fee | 15–30% | 0% |
| Infra COGS | $0.50 | $0.34 |
| Support/CS allocation | ~$0.70 [ESTIMATE] | ~$3.00 [ESTIMATE] (dedicated CSM) |
| **Net contribution margin** | ~$12.10–14.95 | ~$26.66 |
| **Net margin %** | ~64–79% | ~89% |

Enterprise wins on margin even with dedicated CS, because the direct billing relationship eliminates the App Store tax entirely.

### 8.5 Enterprise CAC Model

Enterprise CAC is the all-in cost to acquire one signed deal: marketing attribution, sales time, and pre-sales technical work. Unlike consumer CAC (driven by paid acquisition), enterprise CAC is driven by human time.

| Cost item | Small deal (20 seats) | Mid deal (100 seats) | Large deal (500 seats) |
|---|---|---|---|
| SDR/AE outreach and qualification | $400–600 (founder-led) | $1,500–2,500 (AE at $120k OTE, 20 deals/year) | $3,500–6,000 (AE + SDR) |
| Technical evaluation / POC | $300–500 (0.5–1 eng day) | $800–1,500 (1.5–3 eng days) | $2,000–4,000 (3–6 eng days) |
| Legal and compliance review | $200–400 | $500–800 | $1,500–3,000 |
| Content and outreach (amortized) | $100–200 | $300–500 | $500–1,000 |
| **Total enterprise CAC [ESTIMATE]** | **$1,000–1,700** | **$3,100–5,300** | **$7,500–14,000** |

Notes:
- **Founder-led sales (pre-PMF):** No explicit AE cash cost. Founder time has opportunity cost but is not a cash outflow. Small-deal cash CAC at this stage: $500–800.
- **Post-PMF AE-led:** The $1,500–2,500 AE cost on mid deals assumes 20 closed deals/year per AE at $120k total compensation. Adjust as actual close rates emerge.
- All figures [ESTIMATE] until first 3 enterprise deals are time-tracked. See OQ-08.

### 8.6 Enterprise LTV/CAC Analysis

LTV = ACV × average contract life × gross margin. Enterprise average contract life: **3 years** [ESTIMATE — typical for corporate wellness; replace post-M12].

| Deal size | ACV | LTV (3yr × 89% GM) | CAC (midpoint) | LTV/CAC | Payback period |
|---|---|---|---|---|---|
| Small (20 seats) | $6,000 | $16,020 | $1,350 | **11.9×** | ~3 months |
| Mid (100 seats) | $36,000 | $96,120 | $4,200 | **22.9×** | ~1.7 months |
| Large (500 seats) | $210,000 | $560,700 | $10,750 | **52.2×** | ~0.6 months |

All three deal sizes deliver LTV/CAC ratios well above the conventional 3:1 benchmark for B2B SaaS. Even the minimum 20-seat deal returns ~12× CAC over the contract life. Economics improve dramatically at scale, justifying investment in enterprise GTM once consumer PMF is validated.

**Caveat:** These ratios assume 89% net margin from §8.4. If CS cost scales above modeled levels (e.g., dedicated CSM per account below 50 seats), small-deal margin degrades. The 40-seat minimum threshold in §8.3 exists precisely to protect this.

### 8.7 Expansion and Churn Economics

Enterprise value is not initial ACV alone — it is expansion potential and net revenue retention (NRR).

**NRR model — cohort of 10 mid deals ($360k starting ARR):**

| Scenario | Year 2 ARR | NRR | Notes |
|---|---|---|---|
| Pessimistic: 20% logo churn, 0% expansion | $288,000 | 80% | High early-churn; no land-and-expand |
| Baseline: 10% logo churn, 15% seat expansion | $378,000 | 105% | Modest organic growth offsets some churn |
| Optimistic: 5% logo churn, 30% seat expansion | $421,200 | 117% | Strong adoption → org-wide rollout |

**Expansion triggers:**
- Initial purchase = one department or team. Natural expansion = company-wide rollout.
- 20-seat pilot with proven adoption → 100-seat renewal is a 5× ACV expansion.
- New office locations, shift cohorts, or acquired subsidiaries as additional seat blocks.
- Enterprise SSO/SCIM integration (see `docs/SSO_SCIM_IMPLEMENTATION.md`) makes org-wide expansion frictionless once IdP is established — no individual onboarding per new employee.

**Churn signals to monitor:**
- Champion departure (HR lead or wellness coordinator who bought) — highest logo-churn risk
- Seat utilization below 30% by M3 — leading indicator; trigger a CS engagement
- Budget cycle reset (wellness budgets often reviewed in Q4)
- M&A activity at client company

**Investor benchmark context:** Series A enterprise SaaS targets NRR >110%. Series B targets >120%. Baseline scenario (105%) is below Series A target — validating that expansion motion is a first-class product and CS priority from day one.

### 8.8 Year 1 vs. Year 2+ Deal Economics

Implementation cost is front-loaded; from Year 2, the same deal generates higher net margin because setup is fully amortized.

| Metric | Year 1 (mid deal, 100 seats) | Year 2+ (same deal, no expansion) |
|---|---|---|
| ACV | $36,000 | $36,000 |
| Infrastructure COGS (@$0.34/seat/month × 12) | $408 | $408 |
| Implementation cost (one-time) | $1,500 | $0 |
| CSM cost ($250/month × 12) | $3,000 | $3,000 |
| **Total cost** | **$4,908** | **$3,408** |
| **Net margin** | **$31,092 (86.4%)** | **$32,592 (90.5%)** |
| Y2 margin improvement vs Y1 | — | **+4.1 pp** |

**Multi-year contract strategy:** A 5–8% discount on 2-year or 3-year prepayment improves cash flow and locks in renewal. The Year 2+ efficiency gain (+4.1 pp margin) more than offsets the discount on most deal sizes. Standard practice on deals >50 seats; offer proactively in enterprise negotiations.

---

## 9. App Store Tax Impact

### 9.1 Fee structure

| Scenario | Apple fee | Google fee | Notes |
|---|---|---|---|
| Standard (>$1M annual revenue) | 30% | 30% | Default |
| Small Business Program (<$1M annual revenue) | 15% | 15% | Applies from day one if enrolled; Apple resets to 30% when you cross $1M |
| Web billing (direct Stripe) | ~2.9% + $0.30 | ~2.9% + $0.30 | Payment processing only; no platform tax |
| Enterprise direct invoice | ~0.5–1% | ~0.5–1% | Wire/ACH; minimal processing cost |

### 9.2 Impact on Pro user economics

| Billing path | ARPU | Store/processing fee | Net revenue | Infra COGS | Gross profit | GM% |
|---|---|---|---|---|---|---|
| Apple IAP (SBP, 15%) | $19.00 | $2.85 | $16.15 | $0.50 | $15.65 | 82.4% |
| Apple IAP (standard, 30%) | $19.00 | $5.70 | $13.30 | $0.50 | $12.80 | 67.4% |
| Web billing (Stripe) | $19.00 | $0.85 | $18.15 | $0.50 | $17.65 | 92.9% |

### 9.3 Strategy

**Near-term (pre-$1M ARR):** Enroll in Small Business Program immediately on launch. 15% fee is the baseline. This is straightforward and material — the difference between 15% and 30% is $2.25/user/month, or roughly $27/year. On 5,000 Pro users that is $135k/year in additional margin.

**Web billing alternative:** iOS App Store guidelines prohibit directing users to external purchase flows from within the app (anti-steering rules), but the Epic v. Apple ruling (US) and EU Digital Markets Act create openings. Monitor legal landscape. If web billing becomes viable in primary markets, a direct Stripe billing path at $19/month could recover ~$2.00–4.85/user/month vs. IAP.

**Annual plan uplift:** Annual plans paid upfront ($120–144/year depending on discount offered) improve cash flow and reduce churn. App store fees still apply, but the timing benefit is material for a capital-light pre-Series A company.

**Enterprise billing:** All enterprise deals are invoiced directly (wire, ACH, or credit card via Stripe). No app store involvement. This is one of the structural margin advantages of B2B revenue.

---

## 10. Sensitivity Analysis

### 10.1 Anthropic API cost doubles (2× current pricing)

**Scenario:** Anthropic raises prices, or FORM's token usage increases due to longer context windows or feature expansion.

| Metric | Baseline | 2× API cost | Delta |
|---|---|---|---|
| API cost/Pro user/month | $0.20 | $0.41 | +$0.21 |
| Total COGS/Pro user | $0.50 | $0.71 | +$0.21 |
| Gross profit/Pro user (SBP) | $15.65 | $15.44 | -$0.21 |
| Gross margin % (SBP) | 82.4% | 81.3% | -1.1 pp |

**Assessment:** Surprisingly low impact at current usage. Even a 2× price increase only moves gross margin ~1.4 percentage points because API cost is a small fraction of ARPU at Pro pricing. The model is robust to Anthropic price increases at this ARPU level.

**The real risk** is not price, it is usage: if Pro users average 25+ sessions/month instead of 15 (i.e., W-ACSU of 6+ instead of 3.5), API cost doubles from usage alone, not from pricing. Impose soft usage metering on free tier; monitor Pro usage closely.

### 10.2 Voice adoption 80% vs. 20% (Pro tier)

| Scenario | Voice adoption | Voice cost/user/month | Total COGS | Gross margin % (SBP) |
|---|---|---|---|---|
| Low adoption | 20% | $0.11 | $0.39 | 83.0% |
| Baseline | 40% | $0.22 | $0.50 | 82.4% |
| High adoption | 80% | $0.43 | $0.71 | 81.3% |

**Assessment:** Voice adoption at 80% adds ~$0.21/user/month vs. 20% baseline — similar in magnitude to a 2× Anthropic price increase. The margin impact is manageable.

**However:** If voice adoption drives meaningfully higher retention (hypothesis: voice coaching is stickier), the LTV impact of high voice adoption likely outweighs its cost. Do not optimize voice adoption down purely on a cost basis without measuring the retention correlation.

**Mitigation:** Audio caching of common cues (see Section 7.2) changes the economics significantly. With 60% cache hit rate, effective cost at 80% adoption is ~$0.17/user/month — lower than the baseline uncached 40% adoption scenario.

### 10.3 Combined stress scenario: 2× Anthropic + 80% voice adoption (uncached)

| Metric | Value |
|---|---|
| API cost/Pro user | $0.41 |
| Voice cost/Pro user | $0.43 |
| Total COGS | $0.92 |
| Gross profit/Pro user (SBP) | $15.23 |
| Gross margin % | 80.2% |

Even under this combined worst case, gross margin stays above 80% at the SBP rate. The model is structurally healthy at current ARPU levels. The model breaks if ARPU drops significantly (e.g., heavy discounting or geo mix shift toward lower-price markets), not from API costs.

### 10.4 Supabase scaling inflection

Supabase Pro ($25/month) is sufficient through approximately 10,000–15,000 MAU. At higher scale:

| MAU | Supabase plan | Monthly cost | Cost/user |
|---|---|---|---|
| < 500 | Free | $0 | $0 |
| 500–15,000 | Pro | $25 | $0.002–$0.05 |
| 15,000–50,000 | Team | ~$599 [ESTIMATE] | $0.012–$0.04 |
| 50,000+ | Enterprise | Negotiated | TBD |

Supabase cost is not a meaningful gross margin risk at any pre-Series A scale.

### 10.5 Enterprise Revenue Mix Sensitivity

**Scenario:** Enterprise tier grows faster than consumer, shifting FORM's revenue mix.

| Enterprise % of MRR | Blended gross margin | Notes |
|---|---|---|
| 5% enterprise, 95% consumer Pro | ~82% | Near-pure consumer economics; Western $19 ARPU baseline |
| 20% enterprise, 80% consumer Pro | ~83.5% | Modest mix improvement |
| 50% enterprise, 50% consumer Pro | ~86% | Direct billing on half of revenue eliminates store fee |
| 80% enterprise, 20% consumer Pro | ~88% | Significant uplift; CS cost assumed at 3% of enterprise ACV |

Each percentage point of revenue shifting from consumer to enterprise improves blended gross margin by ~0.1–0.15 pp. Mix improvement is a secondary margin lever — meaningful at scale (>$5M ARR) but not the primary driver. The primary driver remains consumer Pro pricing and App Store fee management.

**The strategic case for enterprise is not margin** — it is contract predictability, expansion potential, and NRR. A 100-seat enterprise deal paying annually upfront is qualitatively different from 100 individual Pro subscribers at monthly billing with 3–5% monthly churn.

---

## 11. Open Questions / Gaps

| # | Question | Impact if wrong | How to resolve |
|---|---|---|---|
| OQ-01 | What is actual token usage per session in beta? | High — drives the largest COGS line | Instrument `tokens_used` on every Anthropic API call; log to PostHog as a property on `chat_message_sent` event |
| OQ-02 | What is actual voice adoption rate? | Medium — second-largest COGS line | PostHog `feature_viewed` + `insight_generated` with `voice_enabled: true/false` property |
| OQ-03 | Does Anthropic offer volume pricing below $50k/month spend? | Medium — affects 10k–30k user scale | Direct conversation with Anthropic sales at $10k+/month spend |
| OQ-04 | Is the Small Business Program automatic or does it require active enrollment on App Store Connect? | High at launch — $2.25/user/month difference | Confirm with Apple developer documentation before launch |
| OQ-05 | What is the actual ElevenLabs per-character rate at Creator vs. Business plan tiers? | Medium | Review ElevenLabs pricing page; confirm with account representative |
| OQ-06 | At what MAU does PostHog cost become material (>$200/month)? | Low — well-modeled | Monitor PostHog usage dashboard monthly |
| OQ-07 | Does high voice adoption (>60%) measurably improve D90 retention? | High strategic importance | Segment D90 cohort by voice_enabled flag; measure after first 3-month cohort |
| OQ-08 | What is the true implementation cost per enterprise deal (eng hours + CS hours)? | High for enterprise margin model | Time-track first 3 enterprise deals |
| OQ-09 | How does Supabase bandwidth cost behave with wearable data sync (HealthKit/Health Connect)? | Medium — wearable data is high-frequency | Benchmark bandwidth during beta with wearables enabled |
| OQ-10 | Is there a Cloudflare Workers cost cliff from CV-adjacent edge processing? | Low — CV is on-device | Confirm no server-side CV inference at launch |

---

---

## 12. Free Tier Subsidy Model & Freemium Funnel Economics

Section §3.4 flags the core risk: "a free user who never converts to Pro is $0.13/month of pure cost." This section models what that subsidy costs at scale, what conversion rate makes it economically neutral, and how to benchmark freemium as an acquisition channel against paid marketing.

### 12.1 Free tier total subsidy at scale

Infrastructure cost per free user per month: **$0.13** (from §3.3).

| Free MAU | Monthly subsidy | Annual subsidy | Notes |
|---|---|---|---|
| 500 | $65 | $780 | Closed beta phase — negligible |
| 2,000 | $260 | $3,120 | Early-access launch |
| 10,000 | $1,300 | $15,600 | Growth phase; meaningful but not alarming |
| 50,000 | $6,500 | $78,000 | Scale; now a budget line requiring active monitoring |
| 100,000 | $13,000 | $156,000 | Pre-Series A max free pool without explicit subsidy budget |

**Observation:** At any pre-Series A scale (≤100k free MAU), the raw infrastructure subsidy stays well below $200k/year and is covered by a modest number of Pro conversions. The free tier is not an infrastructure cost problem — it is a conversion discipline problem.

### 12.2 Minimum conversion rate for free tier self-financing

For the free tier to be cost-neutral from infrastructure COGS alone, converted users must generate enough gross profit to offset the ongoing subsidy of unconverted users.

**Variables:**
- `N` = free user pool size
- `C` = monthly conversion rate (free → Pro, cumulative by month M)
- Converted user generates `$15.65` gross profit/month (Pro at SBP, Western $19 ARPU, from §4)
- Remaining free user costs `$0.13`/month

**Break-even condition:**

```
N × C × $15.65  =  N × (1 − C) × $0.13

C / (1 − C)     =  0.13 / 15.65  =  0.00831

C               =  0.00831 / 1.00831  ≈  0.82%
```

**A lifetime conversion rate of ≥ 0.82% is sufficient to cover the pure infrastructure cost of the free tier** — regardless of free pool size. Typical well-designed freemium products convert 2–5% of free users to paid over the first 90 days. FORM's break-even threshold is intentionally conservative and should be achievable from the first beta cohort.

### 12.3 Conversion rate targets by phase

| Phase | Free MAU target | Required conversions/month to cover subsidy | Implied conversion rate floor |
|---|---|---|---|
| Beta (closed) | 500 | 1 Pro user | 0.2% |
| Early launch | 2,000 | 3 Pro users | 0.15% |
| Growth | 10,000 | 11 Pro users | 0.11% |
| Scale | 50,000 | 53 Pro users | 0.11% |

The required absolute conversion count is low at every phase. The strategic target is 2–5% cumulative conversion by D90 — well above the subsidy break-even floor — because the business goal is revenue, not merely cost neutrality.

### 12.4 Freemium CAC vs. paid acquisition

Freemium is an acquisition channel. The "cost" of acquiring a Pro subscriber via the free tier is the infrastructure subsidy paid during the pre-conversion period plus any organic/paid traffic cost to reach the free user.

**Infrastructure-only freemium CAC:**

```
Freemium CAC (infra) = months_to_convert × $0.13/month

If median conversion happens at month 2:  2 × $0.13 = $0.26
If median conversion happens at month 6:  6 × $0.13 = $0.78
```

Even at a 6-month pre-conversion window, the infrastructure portion of freemium CAC is under $1 per converted user — negligibly small.

**All-in freemium CAC** includes the cost of driving organic installs (App Store optimization, content marketing, referral mechanics) amortized over conversions:

| Traffic source | Cost/install [ESTIMATE] | Cost/Pro conversion (at 3% rate) | Notes |
|---|---|---|---|
| Pure organic (ASO, referral) | ~$0 | ~$0.78 (6mo subsidy only) | Best case; viral coefficient dependent |
| Content marketing (amortized) | $1–3 | $33–100 | Blog, social, educational content; high variance |
| Influencer / partnership (fitness) | $3–8 | $100–267 | Higher intent but expensive at scale |
| Paid UA (Meta/TikTok/Google) | $5–15 | $167–500 | Most expensive; often poorest LTV mix |

**Benchmark:** Consumer fitness apps typically report blended CAC of $30–80 via paid channels. If FORM can sustain meaningful organic install volume, freemium CAC stays well below this range — making the free tier a structurally superior acquisition channel compared to paid UA, provided conversion discipline is maintained.

### 12.5 Free tier cost controls

The $0.13/month figure is derived from the §2.1 usage assumptions for free users (6.5 sessions/month, 10% voice adoption). Both are controllable by design:

| Control | Current design | Cost impact if removed |
|---|---|---|
| Voice quota cap (free tier) | ElevenLabs character cap per month | Without cap: voice adoption rises; ElevenLabs cost ≈ $0.016 → potentially $0.08+/user |
| Session frequency (natural, not capped) | No hard session cap; organic 1.5×/week usage assumed | If free users use at Pro rate (3.5×/week): COGS ≈ $0.30/user — still manageable |
| Context window per session | Same as Pro (no reduction) | Context truncation on free would reduce Anthropic cost ~15%; deferred feature |
| Workout history depth | 30-day rolling window on free (hypothetical) | Reduces Supabase storage; minor COGS impact (~$0.005/user/month) |

**Design principle:** Free tier cost controls should be scoped features, not degraded experiences. The right levers are voice quota (highest cost impact) and session depth (contextual). Do not degrade coaching quality on free tier — that reduces conversion, which is the primary economic risk.

### 12.6 Free pool size governance

The free pool requires an explicit budget line once it reaches $1,300+/month (10k+ free MAU). Governance trigger:

| Monthly subsidy threshold | Action required |
|---|---|
| < $500/month (≤ 3,800 free MAU) | No action; implicit in infra budget |
| $500–2,000/month | Monitor conversion rate weekly in PostHog |
| $2,000–5,000/month | Explicit free-tier budget line in FINANCIALS.md; conversion rate OKR target |
| > $5,000/month (≥ 38,500 free MAU) | Board visibility; conversion funnel review; consider free-tier session throttle if D90 conversion < 1.5% |

**Note on free tier as a strategic asset:** Aggressive throttling of the free tier to reduce subsidy cost is a false economy. The free pool is the top of the funnel — cutting it to save $0.13/user/month sacrifices the acquisition engine. The right response to poor conversion is better activation (onboarding, first-value moment), not pool reduction.

---

**v0.4 · May 2026**

All figures marked [ESTIMATE] are pre-launch planning inputs. Replace with actuals as beta instrumentation delivers real usage data. The first reconciliation checkpoint is 30 days post-beta launch, targeting OQ-01 and OQ-02 as the highest priority gaps.

*v0.3 additions: §12 Free Tier Subsidy Model & Freemium Funnel Economics — total subsidy at scale, minimum conversion rate for cost neutrality, freemium CAC vs. paid UA comparison, free tier cost controls, free pool governance triggers.*

*v0.4 updates: §4, §5, §6, §8.4, §9, §10, §12 recalculated to reflect Pro ARPU of $19/month (Western markets, per pricing.html). Geo-pricing note added (§4). Free tier break-even threshold: 1.05% → 0.82%. Full-burn break-even: 6,530 → 5,112 Pro subscribers. Section 10 sensitivity margins updated throughout.*
