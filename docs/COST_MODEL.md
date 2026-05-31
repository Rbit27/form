# FORM · Cost Model & Unit Economics v1.5

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
13. [Infrastructure Cost Breakdown by Service](#13-infrastructure-cost-breakdown-by-service)
    - 13.1 Service-by-service cost table (1k/10k/50k/100k MAU)
    - 13.2 Fixed vs. variable classification
    - 13.3 Plan upgrade inflection points
    - 13.4 Cost cliff analysis
    - 13.5 Optimization levers by service
14. [Cohort LTV Model & CAC Payback Period](#14-cohort-ltv-model--cac-payback-period)
    - 14.1 Model assumptions and inputs
    - 14.2–14.3 Cohort survival tables (Pro monthly / annual)
    - 14.4 CAC payback period analysis
    - 14.5 LTV:CAC ratio targets
    - 14.6 Enterprise LTV model (120% NRR)
    - 14.7 Cohort survival curve requirements
    - 14.8 Data pipeline specification
15. [Enterprise Infrastructure COGS Deep-Dive](#15-enterprise-infrastructure-cogs-deep-dive)
    - 15.1 SSO/SAML token validation overhead
    - 15.2 SCIM provisioning API costs
    - 15.3 Admin dashboard query load
    - 15.4 Audit log storage and long-term retention
    - 15.5 White-label cost model
    - 15.6 Multi-tenant RLS overhead
    - 15.7 Enterprise infrastructure COGS summary
16. [Enterprise Tier Break-Even by Plan](#16-enterprise-tier-break-even-by-plan)
    - 16.1 Plan-level break-even table
    - 16.2 Minimum viable Starter deal analysis
    - 16.3 Year 1 vs. Year 2+ margin improvement by plan
    - 16.4 Multi-year contract economics
17. [Downside Scenario Analysis & Unit Economics Stress Tests](#17-downside-scenario-analysis--unit-economics-stress-tests)
    - 17.1 Purpose & Scope of Stress Testing
    - 17.2 Scenario Definitions
    - 17.3 S1: AI API Cost Shock
    - 17.4 S2: App Store Commission Shock
    - 17.5 S3: Churn Spike
    - 17.6 S4: Price Compression
    - 17.7 S5: Enterprise Pipeline Miss (Year 1)
    - 17.8 S6: Combined Moderate Stress (S1 + S3)
    - 17.9 Open Questions Added
    - 17.10 Stress Test Summary Matrix
18. [Revenue Recognition, ARR/MRR Bridge & SaaS Metrics Taxonomy](#18-revenue-recognition-arrmrr-bridge--saas-metrics-taxonomy)
    - 18.1 Revenue Recognition Framework (ASC 606 / IFRS 15 alignment)
    - 18.2 Billings vs. Revenue vs. Cash — the three waterfalls
    - 18.3 ARR and MRR bridge methodology
    - 18.4 Free pilot timing and first-recognition date
    - 18.5 Deferred revenue waterfall for multi-year contracts
    - 18.6 Net Revenue Retention (NRR) model and target
    - 18.7 Investor-grade SaaS metrics taxonomy
    - 18.8 Open questions added (OQ-14, OQ-15)
19. [Enterprise Go-to-Market Financial Model](#19-enterprise-go-to-market-financial-model)
    - 19.1 Sales Velocity Equation
    - 19.2 Pipeline Stage Conversion Model
    - 19.3 Founder-Led Phase (Deals 0 → 3)
    - 19.4 First AE Hire Economics
    - 19.5 SDR Layer Economics
    - 19.6 CSM Cost Absorption Bridge
    - 19.7 Enterprise Revenue Forecast (Month 1–24)
20. [Customer ROI Model & Business Case Template](#20-customer-roi-model--business-case-template)
    - 20.1 Framework & Audience
    - 20.2 Status Quo Cost Baseline
    - 20.3 FORM Value Levers (utilization arbitrage, absenteeism, productivity, turnover, direct saving)
    - 20.4 Three-Year ROI Table by Tier (Starter 100-seat / Growth 300-seat / Enterprise 1,000-seat)
    - 20.5 ROI Sensitivity Analysis (activation rate, salary band, absenteeism reduction rate)
    - 20.6 Non-Financial Value Drivers (ESG, talent acquisition, compliance posture, culture)
    - 20.7 Business Case Template (copy-paste narrative for internal champion)
    - 20.8 Open Questions Added (OQ-18, OQ-19)
    - 19.8 Open Questions Added (OQ-16, OQ-17)
21. [Pilot Economics, Conversion Governance & Discount Authority Matrix](#21-pilot-economics-conversion-governance--discount-authority-matrix)
22. [Year 1 Cash Flow Forecast & Runway Model](#22-year-1-cash-flow-forecast--runway-model)
    - 22.1 Cash flow vs. P&L distinction
    - 22.2 Funding assumptions
    - 22.3 Month-by-month 18-month cash flow table
    - 22.4 Runway sensitivity matrix
    - 22.5 Key cash flow triggers
    - 22.6 Series A fundraise timing
    - 22.7 Cash management controls
    - 22.8 Open questions (OQ-23 to OQ-27)
23. [Enterprise NRR Engine & Expansion Revenue Model](#23-enterprise-nrr-engine--expansion-revenue-model)
    - 23.1 Monthly ARR bridge template
    - 23.2 Tier migration economics
    - 23.3 Add-on revenue taxonomy
    - 23.4 Three-year cohort NRR projection by tier
    - 23.5 Blended NRR weighted by §19.7 mix
    - 23.6 Annual price indexation clause
    - 23.7 Expansion audit events (DEC-030)
    - 23.8 NRR-to-valuation multiples
    - 23.9 Implementation checklist
    - 23.10 Open questions (OQ-28 to OQ-30)
24. [Series A Fundraising Economics & Dilution Modeling](#24-series-a-fundraising-economics--dilution-modeling)
    - 24.1 Purpose
    - 24.2 Seed Round Recap & Current Cap Table Baseline
    - 24.3 Series A Readiness Criteria (expanded from §22.6.1)
    - 24.4 Capital Requirements Model
    - 24.5 Pre-Money Valuation Methodology
    - 24.6 Dilution Scenarios
    - 24.7 Cap Table Evolution (Seed → Series A → Series B)
    - 24.8 Bridge Financing Analysis
    - 24.9 Series A Timing Triggers
    - 24.10 DEC-030 Fundraising Audit Events
    - 24.11 Implementation Checklist
    - 24.12 Open Questions (OQ-31 to OQ-33)

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

All enterprise pricing per `docs/ENTERPRISE.md` and `enterprise.html`: Starter $12/seat/mo (50–200 seats), Growth $9/seat/mo (200–1,000 seats), Enterprise $6–8/seat/mo (1,000+ seats, negotiated; $7 used as midpoint below). ACV figures assume annual contracts, billed directly — no App Store.

| Metric | Starter (100 seats, $12/seat) | Growth (500 seats, $9/seat) | Enterprise (2,000 seats, $7/seat) |
|---|---|---|---|
| Monthly recurring revenue | $1,200 | $4,500 | $14,000 |
| Annual contract value (ACV) | $14,400 | $54,000 | $168,000 |
| Monthly infra COGS (@ $0.34/seat) | $34 | $170 | $680 |
| Infrastructure gross margin | 97.2% | 96.2% | 95.1% |

### 8.2 Implementation and onboarding cost

Enterprise deals carry one-time and recurring non-infrastructure costs that consumer tiers do not:

| Cost item | Starter deal | Growth deal | Enterprise deal |
|---|---|---|---|
| SSO/SCIM integration and testing | $800–1,200 [ESTIMATE] | $1,000–1,500 [ESTIMATE] | $1,500–2,000 [ESTIMATE] |
| Admin dashboard onboarding and training | $200–400 [ESTIMATE] | $200–400 [ESTIMATE] | $400–600 [ESTIMATE] |
| Custom SLA / compliance review | $300–500 [ESTIMATE] | $500–800 [ESTIMATE] | $1,000–3,000 [ESTIMATE] |
| Dedicated CSM (ongoing) | — (email support only, 24h SLA) | $250–300/mo [ESTIMATE] | $400–500/mo [ESTIMATE] |
| **Total one-time implementation** | **~$1,150** | **~$1,500** | **~$2,000** |

Payback on $14,400 Starter ACV: < 1 month. Payback on $54,000 Growth ACV: < 0.5 months. Starter is email-only support; Growth and Enterprise include a named CSM.

### 8.3 Minimum viable enterprise deal

For enterprise to be worth the GTM motion at this stage:

```
Minimum ACV to justify sales + implementation overhead: ~$7,200/year [ESTIMATE]
= 50 seats at $12/seat/month (Starter floor, annual contract)
  → $600/month MRR · $7,200 ACV

Growth floor (200 seats at $9/seat/month):
  → $1,800/month MRR · $21,600 ACV
```

Deals below 50 seats are not offered — Starter starts at 50 seats by design. Section 16.2 shows that 50-seat Starter Year 1 gross margin is 47.9%; this is the economic minimum FORM accepts in exchange for landing a new enterprise logo. Deals below ~40 seats generate negative gross profit in Year 1 at Starter pricing and are declined. This threshold is reviewed when enterprise sales tooling matures (post-Series A).

### 8.4 Enterprise vs. consumer margin comparison

Growth tier (500 seats, $9/seat) used as representative enterprise deal; margin varies significantly by plan size — see §16 for full plan-level analysis.

| | Pro consumer | Enterprise (Growth, 500 seats, $9/seat) |
|---|---|---|
| ARPU | $19/month (Western) | $9/seat/month |
| Store fee | 15–30% | 0% |
| Infra COGS | $0.50 | $0.34 |
| Support/CS allocation | ~$0.70 [ESTIMATE] | ~$0.50 [ESTIMATE] (CSM @ $250/mo ÷ 500 seats) |
| **Net contribution margin** | ~$12.10–14.95 | ~$8.16 |
| **Net margin %** | ~64–79% | ~90.7% |

Despite a lower per-seat price ($9 vs. $19 consumer ARPU), Growth tier net margin exceeds consumer margin at scale because: (a) no App Store tax, (b) CSM labor amortizes favorably at 500+ seats, and (c) direct Stripe billing has no 15–30% platform fee.

**Starter tier note:** At 50 seats (minimum), per-seat support allocation is ~$4/seat/month ($200/mo email support ÷ 50 seats), compressing net margin to ~63–68% in Year 1. Year 2+ (no implementation amortization) recovers to ~82%. See §16 for full break-even by plan.

### 8.5 Enterprise CAC Model

Enterprise CAC is the all-in cost to acquire one signed deal: marketing attribution, sales time, and pre-sales technical work. Unlike consumer CAC (driven by paid acquisition), enterprise CAC is driven by human time.

| Cost item | Starter deal | Growth deal | Enterprise deal |
|---|---|---|---|
| SDR/AE outreach and qualification | $400–600 (founder-led) | $1,500–2,500 (AE at $120k OTE, 20 deals/year) | $3,500–6,000 (AE + SDR) |
| Technical evaluation / POC | $300–500 (0.5–1 eng day) | $800–1,500 (1.5–3 eng days) | $2,000–4,000 (3–6 eng days) |
| Legal and compliance review | $200–400 | $500–800 | $1,500–3,000 |
| Content and outreach (amortized) | $100–200 | $300–500 | $500–1,000 |
| **Total enterprise CAC [ESTIMATE]** | **$1,000–1,700** | **$3,100–5,300** | **$7,500–14,000** |

Notes:
- **Founder-led sales (pre-PMF):** No explicit AE cash cost. Founder time has opportunity cost but is not a cash outflow. Starter cash CAC at this stage: $500–800.
- **Post-PMF AE-led:** The $1,500–2,500 AE cost on Growth deals assumes 20 closed deals/year per AE at $120k total compensation. Adjust as actual close rates emerge.
- All figures [ESTIMATE] until first 3 enterprise deals are time-tracked. See OQ-08.

### 8.6 Enterprise LTV/CAC Analysis

LTV = ACV × average contract life × gross margin. Enterprise average contract life: **3 years** [ESTIMATE — typical for corporate wellness; replace post-M12]. GM% per tier: Starter ~80% (steady-state Y2+), Growth ~91%, Enterprise ~92% — derived from §8.4 and §16.

| Deal tier | Representative deal | ACV | LTV (3yr × tier GM%) | CAC (midpoint) | LTV/CAC | Payback |
|---|---|---|---|---|---|---|
| Starter · minimum | 50 seats × $12/seat | $7,200 | $17,280 (80% GM) | $1,350 | **12.8×** | ~2.3 mo |
| Starter · typical | 100 seats × $12/seat | $14,400 | $34,560 (80% GM) | $1,500 | **23.0×** | ~1.3 mo |
| Growth · typical | 500 seats × $9/seat | $54,000 | $147,420 (91% GM) | $4,200 | **35.1×** | ~0.9 mo |
| Enterprise · typical | 2,000 seats × $7/seat | $168,000 | $448,560 (89% GM) | $10,750 | **41.7×** | ~0.8 mo |

All deal tiers deliver LTV/CAC ratios well above the conventional 3× B2B SaaS benchmark. The minimum 50-seat Starter deal returns ~13× over 3 years. Economics improve at scale, justifying enterprise GTM investment once consumer PMF is validated.

**Caveat:** Starter GM% (80%) uses steady-state Year 2+ margins — Year 1 is lower (47.9% at 50 seats) due to implementation amortization. If CS cost scales above modeled levels on small accounts, margin degrades further. The 50-seat minimum in §8.3 exists to protect this floor.

### 8.7 Expansion and Churn Economics

Enterprise value is not initial ACV alone — it is expansion potential and net revenue retention (NRR).

**NRR model — cohort of 10 Growth deals at launch ($540k starting ARR · 500 seats × $9/seat × 12 × 10 deals):**

| Scenario | Year 2 ARR | NRR | Notes |
|---|---|---|---|
| Pessimistic: 20% logo churn, 0% expansion | $432,000 | 80% | High early-churn; no land-and-expand |
| Baseline: 10% logo churn, 15% seat expansion | $567,000 | 105% | Modest organic growth offsets some churn |
| Optimistic: 5% logo churn, 30% seat expansion | $630,900 | 117% | Strong adoption → org-wide rollout |

**Expansion triggers:**
- Initial purchase = one department or team. Natural expansion = company-wide rollout.
- 50-seat Starter pilot with proven adoption → 200-seat renewal at $12/seat is a 4× ACV expansion.
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

| Metric | Year 1 (Growth, 500 seats, $9/seat) | Year 2+ (same deal, no expansion) |
|---|---|---|
| ACV | $54,000 | $54,000 |
| Infrastructure COGS (@ $0.34/seat/month × 12) | $2,040 | $2,040 |
| Implementation cost (one-time) | $1,500 | $0 |
| CSM cost ($250/month × 12) | $3,000 | $3,000 |
| **Total cost** | **$6,540** | **$5,040** |
| **Net margin** | **$47,460 (87.9%)** | **$48,960 (90.7%)** |
| Y2 margin improvement vs Y1 | — | **+2.8 pp** |

**Multi-year contract strategy:** A 15–25% discount on 2–3 year prepayment (per `enterprise.html`) improves cash flow and locks in renewal. The Year 2+ efficiency gain (+2.8 pp) partially offsets the discount while eliminating annual logo churn risk entirely. See §16.4 for detailed multi-year contract economics.

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
| OQ-11 | What is the minimum seat threshold at which a Starter plan deal generates acceptable Year 1 gross margin? | Medium — sets enterprise minimum deal floor | Track GM per closed deal for first 5 Starter contracts; §16.2 model suggests 50-seat floor may need to rise to 75+ if dedicated CSM cost exceeds $200/month allocation |

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

## 13. Infrastructure Cost Breakdown by Service

This section replaces the high-level COGS summary in §3.3 with a service-by-service breakdown that shows absolute monthly spend, per-user cost at each scale tier, the upgrade inflection points, and the optimization levers available at each stage. Purpose: give investors and the future CFO a precise view of where spend accumulates and what can be actively managed.

**Assumptions for this section:**

- MAU mix modeled at 60% Free / 40% Pro (consumer). Enterprise is modeled separately in §8.
- Per-user behavioral rates from §2.1 apply throughout.
- Per-user COGS from §3.3 apply: Free $0.13/user/month; Pro $0.50/user/month.
- All figures [ESTIMATE] until reconciled against invoices.

### 13.1 Service-by-service cost table

| Service | Type | Current plan | 1k MAU total | 10k MAU total | 50k MAU total | 100k MAU total |
|---|---|---|---|---|---|---|
| **Anthropic Claude** | Variable | Pay-as-you-go (list price) | ~$134/mo | ~$1,344/mo | ~$6,720/mo | ~$13,440/mo |
| **ElevenLabs TTS** | Variable | Creator/Business API | ~$97/mo | ~$968/mo | ~$4,840/mo | ~$9,680/mo |
| **Supabase** | Fixed (step) | Pro ($25/mo) | $25/mo | $25/mo | $599/mo [Team] | $599/mo [Team] |
| **Cloudflare Workers** | Variable + fixed | Paid ($5/mo base) | ~$14/mo | ~$140/mo | ~$700/mo | ~$1,400/mo |
| **Sentry** | Fixed | Team (~$26/mo) | $26/mo | $26/mo | $26/mo | $26/mo [upgrade pressure] |
| **PostHog** | Variable (above 1M events) | Free tier | $0 | ~$310/mo | ~$2,790/mo | ~$5,890/mo |
| **Redis** | Fixed (step) | Managed cloud (est.) | ~$15/mo | ~$25/mo | ~$50/mo | ~$150/mo |
| **Expo / EAS** | Fixed | Production plan | ~$99/mo | ~$99/mo | ~$99/mo | ~$99/mo |
| **Total infrastructure** | — | — | **~$436/mo** | **~$3,197/mo** | **~$17,124/mo** | **~$33,884/mo** |
| **Per total MAU** | — | — | **$0.44/user** | **$0.32/user** | **$0.34/user** | **$0.34/user** |

All figures are [ESTIMATE]. The per-MAU blended cost stabilizes around $0.32–0.34 at 10k+ MAU because fixed overhead amortizes rapidly while variable cost per user holds steady until volume discounts apply.

### 13.2 Fixed vs. variable classification

**Fixed overhead (scale-insensitive at current stage):**

- Supabase plan fee — pays for compute and baseline capacity; does not move with user activity within the plan tier
- Cloudflare Workers base fee ($5/month) — pays for the Paid tier activation; request volume cost is variable but negligibly small at pre-50k scale
- Sentry Team plan — covers event volume well beyond any pre-Series A scenario
- Redis managed instance — provisioned capacity; actual memory usage scales gently
- Expo/EAS — CI/CD pipeline and build quota; not user-count driven

**Variable (scale with active users and session frequency):**

- Anthropic API — strictly proportional to sessions × tokens per session × active users
- ElevenLabs TTS — proportional to sessions × character volume × voice adoption rate
- Cloudflare Workers requests — proportional to sessions × API requests per session
- PostHog — proportional to total events per month once past the 1M free tier ceiling

**Key insight:** Fixed overhead at 1k MAU is ~$165/month (Supabase + Sentry + Redis + EAS). Variable API costs are ~$245/month. By 10k MAU, fixed overhead is still ~$175/month while variable is ~$2,700/month. The business is fundamentally variable-cost at scale, which is correct — cost scales with delivered value, not with provisioned capacity.

### 13.3 Plan upgrade inflection points (cost cliffs)

These are the scale milestones at which a service requires a plan change — either because capacity limits are reached or because a better pricing tier becomes economically justified.

| Service | Current ceiling | Upgrade trigger | New plan | Cost step-up |
|---|---|---|---|---|
| **Supabase** | ~10k–15k MAU (8GB DB, 250GB bandwidth) | DB approaching 8GB or monthly bandwidth >250GB | Team (~$599/mo) | $574/mo step-up (from $25) |
| **Supabase** | ~50k–100k MAU (Team plan limits) | Read replica contention or storage overage | Enterprise ($500–2,000/mo negotiated) [ESTIMATE] | $0–$1,400/mo additional |
| **PostHog** | 5k MAU / 1M events/month | Total events cross 1M/month | Pay-as-you-go at $0.00031/event | ~$310/mo at 10k MAU; ~$2,790/mo at 50k MAU |
| **Sentry** | Team plan: ~50k errors/day | Consistent daily error volume approaching cap, or need for performance monitoring | Business/Enterprise | $80–$150/mo [ESTIMATE] |
| **Anthropic** | No plan ceiling; list pricing | Monthly API spend reaches $10k–50k+ | Volume commitment discount tier | Rate negotiation, not a hard cliff |
| **ElevenLabs** | Creator: 100k chars/month | Exceeds plan character quota | Business tier / Enterprise API | Variable; Business ~$99/mo [ESTIMATE] |
| **Redis** | Scaled instance capacity | P99 latency increases or memory OOM errors | Larger managed instance | ~$50–200/mo additional [ESTIMATE] |
| **Cloudflare** | Workers: 50M requests/day (Paid) | Exceeds Workers Paid daily limit | Workers Unbound or Enterprise | Not expected pre-Series A |

**The only true hard cliff is Supabase** — the jump from $25/month Pro to $599/month Team is a 24× step-up in the plan line item. However, on a per-user basis at 15k+ MAU, the Team plan costs ~$0.04/user/month — less than the Anthropic API cost rounding error. This cliff is visually dramatic but economically negligible.

**ElevenLabs plan quotas** are a more immediate operational concern. At 40% Pro voice adoption with 15.1 sessions/month and 1,200 chars/session, each Pro user generates ~7,248 characters of TTS monthly. The Creator plan ($22/month) covers 100,000 characters — exhausted by approximately 14 Pro voice users. This means ElevenLabs must operate on the per-character API billing model from launch, not on plan-based character quotas. Current assumption ($0.30/1,000 chars via Creator/Business tier API billing) is correct operationally but requires confirming enterprise API pricing with ElevenLabs directly. See OQ-05.

### 13.4 Cost cliff analysis — step-function increases

The following events represent non-linear jumps in the monthly infrastructure bill:

**Cliff 1: Supabase Pro → Team at ~12k–15k MAU**
Step-up: +$574/month. Revenue context: at 15k MAU with 40% Pro mix (6,000 Pro users at $19/month Western), MRR is approximately $114,000. The Supabase step-up represents 0.5% of MRR. Not a strategic concern — but it does appear as a line-item spike in any monthly cost review. Pre-announce internally when DB usage crosses 6GB.

**Cliff 2: PostHog free → paid at ~5k MAU**
Step-up: ~$310/month at 10k MAU; $2,790/month at 50k MAU. The 10k cliff is small. The 50k cliff is real (~2.4% of MRR at that scale assuming $116k MRR from Pro users alone). PostHog costs scale predictably — no sudden jump beyond the free tier exit. Acceptable; budget as a known line item at the 10k milestone.

**Cliff 3: Anthropic volume commitment threshold at ~30k–50k Pro MAU** [ESTIMATE]
This is not a cost increase — it is a cost decrease opportunity. When monthly Anthropic API spend approaches $10k–50k/month, volume commitment pricing becomes negotiable. Missing this window (continuing on list pricing past that threshold) is the inverse cliff: a failure to capture available savings. At 30k Pro MAU with current usage, estimated Anthropic spend is ~$6,000/month. Engagement target: initiate Anthropic sales conversation at $3k/month spend.

**Cliff 4: ElevenLabs at scale — enterprise API negotiation**
At 50k total MAU with 40% Pro mix (20k Pro users), estimated ElevenLabs API cost at list pricing is ~$4,840/month (from §13.1 table). This is the point at which audio caching becomes economically decisive. A 60% cache hit rate on common coaching cues reduces this to ~$1,936/month — a $2,900/month saving. Audio caching should be implemented before reaching 10k Pro MAU, not after.

**Summary: when to act**

| Milestone | Trigger action |
|---|---|
| 5k total MAU | Monitor PostHog event volume; budget $300/month for analytics starting M+1 |
| 12k total MAU | Provision Supabase Team plan; schedule DB migration before 7.5GB |
| 3k Pro MAU or $3k/mo Anthropic spend | Initiate Anthropic sales contact for committed spend pricing |
| 8k Pro MAU (voice-enabled) | Implement audio caching for common cues; audit ElevenLabs character volume |
| 30k total MAU | Begin Sentry Business evaluation; Supabase Enterprise scoping |

### 13.5 Optimization levers by service

**Anthropic Claude — highest priority**

1. **Prompt caching.** Claude supports caching of static context segments via the `cache_control` API parameter. Victor's base persona, safety rules, and exercise reference library (~800–1,200 tokens) is static per-user-session. Caching this segment reduces input token cost on cached content by up to 90% (cache hits are billed at $0.30/1M tokens vs. $3.00/1M). [ESTIMATE: saves $0.03–0.06/Pro user/month at current usage; at 10k Pro users = $300–600/month]

2. **Model tiering.** Route low-complexity tasks (intent classification, set logging confirmation, push notification copy) to Claude Haiku (significantly cheaper than Sonnet). Reserve Sonnet for genuine coaching turns and program adaptation. Target: 40–50% of API calls routable to Haiku. [ESTIMATE: 20–30% API cost reduction on the Haiku-routed portion]

3. **Context window management.** The current assumption is 2,000 input tokens/session (§2.1). Implementing a sliding history window (most recent 5 sessions rather than full history) caps token growth as users accumulate history. Without this, long-tenure user costs grow logarithmically.

4. **Committed spend / volume pricing.** Target $10k/month API spend as the conversation threshold with Anthropic sales. Typical discount 10–25% at this level. [ESTIMATE]

**ElevenLabs — second priority**

1. **Audio caching.** Victor's coaching vocabulary is finite and highly repetitive. Pre-generate and CDN-cache the top 200–300 cue phrases ("Good depth on that squat", "Elbows up and in", "Brace your core before the lift"). [ESTIMATE: 60–70% cache hit rate on character volume, reducing effective TTS cost by ~50%]

2. **Character quota management.** Enforce per-user monthly character budgets in the application layer. Free tier: hard cap. Pro tier: soft cap with graceful degradation to text-only coaching beyond threshold.

3. **Enterprise API pricing.** Negotiate directly with ElevenLabs at the point where monthly character volume exceeds 10M characters (~1,400 Pro users at current usage rates). Direct enterprise rates typically undercut Business tier list pricing at this volume.

**Supabase — low priority, well-managed**

1. **Read replicas for analytics.** Route all dashboard and analytics queries to a read replica (available on Team plan and above). Prevents analytical workloads from impacting production DB compute, which would trigger auto-scaling charges.

2. **Storage tiering.** Workout history data older than 12 months can be moved to Supabase Storage (object storage, significantly cheaper than DB rows) or cold-archived. Reduces DB size growth rate by ~40% for mature users. [ESTIMATE]

3. **Connection pooling (PgBouncer).** Already included in Supabase architecture. Ensure Cloudflare Workers connections route through PgBouncer to prevent connection exhaustion at scale.

**PostHog — deferred**

Costs are predictable and linear above 1M events. No optimization lever is economically significant at pre-50k MAU. At 100k MAU, consider: sampling non-critical events (e.g., `feature_viewed` for low-priority screens at 50% sample rate) to reduce event volume and cost. Preserve full fidelity on revenue-critical events (subscription state changes, session completions, conversion funnel events).

**Cloudflare Workers — minimal action required**

Request costs are negligible at any pre-Series A scale. Optimization only relevant if compute-intensive Workers (e.g., video frame processing) are introduced — currently not in scope (CV is on-device per §2.3).

**Redis — passive**

Sized appropriately for session store and rate limiting. No optimization levers at current scale. Scale instance up reactively on P99 latency signal.

---

## 14. Cohort LTV Model & CAC Payback Period

This section models the expected revenue and gross profit generated by a subscriber cohort over 24 months, derives the payback period at different CAC assumptions, and establishes the LTV:CAC framework FORM will use in investor reporting. All churn figures are [ESTIMATE] — pre-launch benchmarks from comparable fitness and consumer SaaS products. These will be replaced with cohort actuals from beta as data accumulates. The first replacement checkpoint is M3 post-beta launch.

### 14.1 Model assumptions and inputs

| Parameter | Pro monthly | Pro annual | Enterprise (seat) | Source |
|---|---|---|---|---|
| Monthly subscription churn | 5.0% [ESTIMATE] | 1.5%/month [ESTIMATE] | 15%/year logo churn [ESTIMATE] | Fitness app benchmark; Noom/Calm public reports |
| Net ARPU (after App Store SBP 15%) | $16.15/month | $10.20/month equiv. | $12/seat/month (direct) | §9.2; annual plan at $144/year × 0.85 ÷ 12 |
| Infrastructure COGS | $0.50/month | $0.50/month | $0.34/seat/month | §3.3 |
| Gross margin target | 85% | 85% | 89% | §6.1 |
| CAC range modeled | $30–$120 | $30–$120 | $1,350–$10,750 | §8.5; consumer benchmark |

**Note on annual plan ARPU:** The $10.20/month net figure reflects an annual price of $144/year with 15% App Store SBP fee applied at the point of billing. Annual plan subscribers churn at approximately one-third the rate of monthly subscribers (1.5% vs. 5% monthly churn [ESTIMATE]) because the renewal decision is annual, not monthly. This lower churn more than compensates for the reduced per-month ARPU in LTV terms.

### 14.2 Pro monthly subscriber — cohort survival and cumulative revenue

Starting cohort: 100 subscribers acquired in month 0 at $16.15 net ARPU/month. Monthly churn: 5% [ESTIMATE].

**Survival formula:** `Remaining subscribers at month M = 100 × (1 − 0.05)^(M−1)`

| Month | Cohort survival rate | Monthly net revenue (per original subscriber) | Cumulative gross revenue | Cumulative GM-adjusted LTV (85%) |
|---|---|---|---|---|
| 1 | 100.0% | $16.15 | $16.15 | $13.73 |
| 3 | 90.2% | $14.58 | $46.07 | $39.16 |
| 6 | 77.4% | $12.50 | $85.57 | $72.73 |
| 12 | 56.9% | $9.19 | $148.46 | $126.19 |
| 18 | 41.8% | $6.75 | $194.70 | $165.49 |
| 24 | 30.7% | $4.96 | $228.69 | $194.38 |

**Reading the table:** By month 24, approximately 31 of the original 100 subscribers remain. The cohort has generated $228.69 in cumulative gross revenue per original subscriber, of which $194.38 is gross-margin-adjusted LTV (at 85% GM). The curve is front-loaded: the first 6 months capture 37% of 24-month LTV while retaining 77% of the cohort.

### 14.3 Pro annual subscriber — cohort survival and cumulative revenue

Annual plan: $144/year ($120/year with 20% discount, or $144 list). Net ARPU after SBP: $10.20/month equivalent. Monthly churn: 1.5% [ESTIMATE].

| Month | Cohort survival rate | Monthly net revenue equiv. | Cumulative gross revenue | Cumulative GM-adjusted LTV (85%) |
|---|---|---|---|---|
| 1 | 100.0% | $10.20 | $10.20 | $8.67 |
| 3 | 97.0% | $9.90 | $30.14 | $25.62 |
| 6 | 92.7% | $9.46 | $58.95 | $50.11 |
| 12 | 84.7% | $8.64 | $112.79 | $95.87 |
| 18 | 77.3% | $7.89 | $161.96 | $137.67 |
| 24 | 70.6% | $7.20 | $206.87 | $175.84 |

**Comparison with monthly plan:** Annual plan 24-month GM-adjusted LTV ($175.84) is lower than monthly ($194.38) in absolute terms, purely because monthly ARPU is higher ($16.15 vs. $10.20). However, the annual plan retains 70.6% of the cohort at month 24 versus 30.7% for monthly — annual subscribers are structurally more durable. For predictable cash flow and lower CAC payback pressure, annual plan mix is strategically preferable even with lower per-unit LTV.

**Annual plan deferred revenue note:** Subscription revenue is recognized ratably over the subscription period. A subscriber paying $144 upfront creates $144 of cash inflow in month 0 and $12/month of recognized revenue over 12 months. The cash timing benefit is real and material for a capital-light pre-Series A company. However, under GAAP/IFRS, annual plan payments appear as deferred revenue on the balance sheet and are recognized at $12/month — investors and auditors will see both the cash and the recognition schedule. Disclose clearly in financial reports.

### 14.4 CAC payback period analysis — Pro monthly plan

CAC payback period = the number of months until cumulative gross-margin-adjusted revenue from a subscriber cohort equals the initial acquisition cost. Using 85% GM and 5% monthly churn [ESTIMATE].

Monthly GM contribution per surviving subscriber at month M:
`GM_M = $16.15 × (1 − 0.05)^(M−1) × 0.85`

Cumulative GM contribution builds month by month until it crosses the CAC threshold:

| CAC | Payback month | M12 GM-adj LTV | M24 GM-adj LTV | LTV:CAC at M12 | LTV:CAC at M24 |
|---|---|---|---|---|---|
| $30 | Month 3 | $126.19 | $194.38 | **4.2×** | **6.5×** |
| $50 | Month 4 | $126.19 | $194.38 | **2.5×** | **3.9×** |
| $80 | Month 7 | $126.19 | $194.38 | **1.6×** | **2.4×** |
| $120 | Month 12 | $126.19 | $194.38 | **1.1×** | **1.6×** |

**Interpretation:** CAC below $50 delivers payback inside Q1 of the customer relationship — structurally excellent. CAC at $80 pays back in 7 months — acceptable for a consumer subscription with a 24-month expected life. CAC above $120 pushes payback to month 12 and yields LTV:CAC of 1.6× at M24 — this is below the conventional 3× minimum benchmark and would require either churn improvement or ARPU expansion (upgrades, annual conversion) to be defensible.

### 14.5 LTV:CAC ratio targets and rationale

**Consumer Pro targets:**

| Scenario | LTV:CAC target | Rationale |
|---|---|---|
| Organic / content-driven | ≥ 5× at M24 | CAC $30–40 expected; should deliver 4.9–6.5× |
| Paid social (Meta/TikTok) | ≥ 3× at M24 | CAC $50–80 expected; minimum defensible threshold |
| Maximum paid UA ceiling | ≥ 2× at M24 | CAC ~$100; only acceptable with path to annual conversion |

The conventional SaaS benchmark is 3:1 LTV:CAC with payback under 12 months. Consumer subscription benchmarks are slightly different — Calm, Headspace, and comparable wellness apps report blended LTV:CAC of 2–4× depending on channel mix. FORM targets the upper end of this range (3–5×) by prioritizing organic and content-led growth over paid UA.

**Why 3× is a floor, not a target:** At 3× LTV:CAC, after accounting for overhead allocation, the actual return on acquisition spend is thin. At 5×+, acquisition investment compounds meaningfully and provides headroom for higher CAC in channels that deliver better retention cohorts (e.g., users who discover FORM via sports science content convert at lower rates but churn at half the rate of paid UA users — higher CAC, much higher LTV).

**Do not optimize CAC in isolation.** A $30 CAC from influencer campaigns may deliver worse 12-month LTV than an $80 CAC from organic content (higher intent, higher activation, lower churn). Track LTV:CAC by acquisition channel, not blended. This is a PostHog + Supabase cohort join query, not a back-of-envelope calculation.

### 14.6 Enterprise LTV model

Enterprise LTV is driven by initial ACV, expansion (seat growth and new feature adoption), and logo retention. The NRR model assumes 120% NRR [ESTIMATE] — meaning each surviving customer cohort grows 20% per year in ARR terms. This requires ~41% seat/price expansion on retained logos to offset 15% annual logo churn.

**Starter minimum contract baseline:** 50 seats × $12/seat/month × 12 = **$7,200 ACV** (from §8.3; minimum deal size that passes Year 1 economics per §16.2).

**Enterprise cohort LTV at 120% NRR, 15% annual logo churn [ESTIMATE], Starter tier (80% steady-state GM):**

| Year | ACV (per average deal) | GM-adjusted annual revenue | Cumulative GM-adj LTV |
|---|---|---|---|
| Year 1 | $7,200 | $5,760 | $5,760 |
| Year 2 | $8,640 [120% NRR applied] | $6,912 | $12,672 |
| Year 3 | $10,368 | $8,294 | $20,966 |

**3-year GM-adjusted LTV on minimum Starter deal: ~$20,966.** Against a CAC of ~$1,350 (Starter deal, §8.5): **LTV:CAC = 15.5×**.

At larger deal sizes (with proportionally higher but sub-linear CAC from §8.5):

| Deal tier | Starting ACV | 3yr GM-adj LTV | CAC midpoint | LTV:CAC |
|---|---|---|---|---|
| Starter (50 seats, $12/seat) | $7,200 | $20,966 (80% GM) | $1,350 | **15.5×** |
| Growth (500 seats, $9/seat) | $54,000 | $178,870 (91% GM) | $4,200 | **42.6×** |
| Enterprise (2,000 seats, $7/seat) | $168,000 | $544,253 (89% GM) | $10,750 | **50.6×** |

Enterprise LTV:CAC ratios are structurally superior to consumer at every deal size. The minimum 50-seat Starter deal returns ~16× over 3 years. The economic argument for investing in enterprise GTM is not margin (which is already excellent at both tiers) but predictability: annual contracts, invoiced directly, with expanding seat counts, generate ARR that is fundamentally more stable than consumer monthly churn.

**Expansion revenue mechanics at 120% NRR:** A Starter account that starts at 50 seats and expands to 200 seats over 24 months goes from $7,200 ACV to $28,800 ACV — a 4× ACV expansion on a single logo. If it upgrades to Growth tier (200 seats, $9/seat), ACV is $21,600 — a 3× expansion at lower per-seat price but with the same direct-invoice economics. The CAC is fixed at ~$1,350 (acquisition cost does not repeat on expansion); the incremental seats are zero-CAC revenue. This is why NRR >100% is such a powerful metric for enterprise SaaS valuation multiples.

### 14.7 Cohort survival curve — qualitative requirements

What does FORM's retention curve need to look like to hit the targets above?

**The shape that matters:** A healthy retention curve for a fitness app is not a smooth exponential decay. It is a steep initial drop (first 30 days: users who installed out of curiosity but never activated) followed by a flattening into a durable core cohort. The users who complete 3+ coached sessions in the first 14 days (the activation milestone from METRICS.md) are the ones who define the flat tail. The initial drop is expected and acceptable; the floor of the curve is what determines LTV.

**Target curve shape:**
- D1 retention: >60% [ESTIMATE] — users open the app the day after install
- D7 retention: >40% [ESTIMATE] — first week engagement signal
- D30 retention: >25% [ESTIMATE] — the "will this person stick?" threshold
- M3 retention (monthly active): >20% [ESTIMATE] — 5% monthly churn means 86% survive month-to-month, so M3 = ~86^2 = ~74% for already-retained users; but accounting for trial-to-paid churn cliff, effective M3 paying retention should be ~65–70%
- M12 retention (monthly active): >10% [ESTIMATE] — aligns with 56.9% cohort survival at month 12 in the LTV table above

**The churn cliff to avoid:** Consumer subscriptions show a consistent pattern of elevated churn at the first billing renewal (day 7–14 for free trials, day 30 for first paid month). FORM's onboarding and early activation design must specifically address this cliff — users who have not experienced a "coaching moment that felt personalized" by day 10 are extremely likely to cancel at first renewal. The activation funnel target (3 coached sessions in 14 days) is designed to get users past this cliff.

**What 5% monthly churn means in practice:** A cohort of 1,000 subscribers acquired in January will have approximately 540 remaining at December (month 12) — a 46% loss over the year. This is the benchmark being modeled. If actual beta churn comes in below 5%, every LTV figure above improves materially. If churn comes in at 8–10% (worse than benchmark), the CAC payback math at $80+ CAC breaks down and acquisition spend must be curtailed until churn is diagnosed and addressed.

**Leading indicators to watch (from PostHog):**

- W-ACSU (Weekly Active Coached Sessions per User) — the NSM from METRICS.md. Users with W-ACSU ≥ 2.5 in their first month churn at roughly half the rate of users with W-ACSU < 1.0. [ESTIMATE — to be validated in beta]
- Chat message sent frequency — users who use the AI coaching chat ≥ 3 times per week show substantially lower churn in comparable AI wellness products
- Session streak length — unbroken session streaks are a strong leading indicator; a 7-day streak in week 2 is the strongest early predictor of 90-day retention in fitness apps

### 14.8 Data pipeline for cohort LTV tracking

LTV is not a formula you run once — it is a live cohort measurement that requires a functioning data pipeline. FORM's approach:

**Source tables:** `fact_events` (session completions, billing events), `dim_users` (acquisition channel, install date, tier), subscription state from Supabase (server-side, trusted).

**Cohort definition:** Cohort = week of first subscription payment (not install). A user who installs in week 1 and subscribes in week 3 enters the cohort in week 3. This is the correct definition for LTV purposes — LTV clock starts at first revenue, not first install.

**Retention metric computed daily** via ETL (§pipeline in ANALYTICS_SETUP.md): for each cohort-month, count users who made at least one session in that calendar month divided by original cohort size. Output: retention matrix stored in ClickHouse `fact_cohort_retention` table.

**CAC by channel** sourced from paid attribution (UTM parameters → PostHog → Supabase), organic attribution (ASO, referral codes), and estimated content marketing cost amortized over organic installs per month.

**LTV:CAC report** computed as a saved Metabase question: join `fact_cohort_retention` × `dim_users.acquisition_channel` × marketing spend table. Updated weekly. Owner: data-engineer.

**Backfill note:** Until 90 days of post-launch data exist, LTV figures in investor reports will be labeled "projected based on [ESTIMATE] benchmark churn." The first cohort with real 90-day data is the point at which [ESTIMATE] tags are removed from §14.2 and §14.3. Target: M6 post-launch.

---

## 15. Enterprise Infrastructure COGS Deep-Dive

Unlike consumer tiers, enterprise deployments carry additional infrastructure features — SSO, SCIM, admin dashboard, audit log delivery, and optional white-label. This section quantifies the marginal infrastructure cost of each enterprise-specific capability and explains why none of them materially change the baseline $0.34/seat/month COGS from §3.3.

**Core finding: enterprise features are operationally lightweight.** SSO/SCIM, audit logs, admin dashboards, and SIEM webhooks are all low-compute, low-bandwidth operations that together add less than $0.002/seat/month to infrastructure COGS. The enterprise tier's high gross margin (~97% on infrastructure, ~89% blended with CS) is not a modeling artifact — it reflects the reality that identity, auditing, and analytics workloads are inexpensive at the compute level.

### 15.1 SSO/SAML token validation overhead

Each enterprise login generates additional Cloudflare Workers calls compared to a standard JWT auth flow:

| Step | Workers requests | Description |
|---|---|---|
| SAML assertion receipt | 1 | IdP POST to FORM ACS endpoint (`/auth/saml/acs`) |
| Assertion validation + RLS context set | 1 | HMAC verification, tenant scoping |
| Session JWT issuance | 1 | Signed token issued and cached in Redis |
| **Total per SSO login** | **3** | vs. 2 requests for standard email/password |

Marginal SSO overhead per seat per month:

```
Sessions/month (enterprise): 12.9
Additional Workers requests vs. standard JWT: 1 per login
Cost: 12.9 × $0.50 / 1,000,000 = $0.0000065/seat/month
At 500 seats: $0.003/month total
```

**OIDC vs. SAML:** OIDC token validation (OAuth 2.0 code exchange) follows the same 3-step pattern. No cost differential between SAML and OIDC implementation at any realistic scale.

**Assessment: negligible.** The entire Cloudflare Workers overhead for enterprise SSO across 500 seats for a full year is approximately $0.04. Not a material cost driver at any deal size.

### 15.2 SCIM provisioning API costs

SCIM v2 events (create, update, deactivate) occur when the IdP syncs directory changes to FORM. Per `docs/SSO_SCIM_IMPLEMENTATION.md §4`, FORM exposes `/scim/v2/Users` and `/scim/v2/Groups`. Assumed frequency: 5% monthly workforce turnover = approximately 0.2 SCIM events/seat/month on average.

| SCIM event type | Workers requests | DB writes | Typical frequency |
|---|---|---|---|
| User create (new hire) | 2 | 1 (insert + RLS policy attach) | Hire rate |
| User update (role/group change) | 2 | 1 (update) | Occasional |
| User deprovision (termination) | 2 | 1 (soft-delete + seat release) | ~5% annually |
| Group sync (SCIM Groups → FORM roles) | 2 | 1 (role mapping update) | On group change |
| **Per seat average** | **~0.4 req/month** | **~0.2 writes/month** | At 5% monthly turnover |

Monthly cost per seat:

```
Workers: 0.4 × $0.50/1M = $0.0000002/seat/month
DB writes: within Supabase plan capacity — no marginal charge
```

**SCIM bearer token** is per-tenant, stored hashed, rotated on customer request or annually (DEC-030 mandated rotation cadence). Token validation adds zero marginal compute.

**Assessment: negligible.** SCIM is an availability and latency engineering concern, not a cost concern.

### 15.3 Admin dashboard query load

Admin dashboard users (HR leads, People ops, IT admins) run analytics queries against the enterprise admin API. Assumed: 2 admin users per 100 seats, checking dashboard 5×/week. All queries routed to Supabase read replica (available on Team plan and above).

| Dashboard view | Read-replica DB queries | Workers requests | Cadence |
|---|---|---|---|
| Engagement overview (weekly trend chart) | 4 | 1 | Daily |
| Seat utilisation table | 2 | 1 | 2×/week |
| Audit log viewer (last 50 events) | 3 | 1 | Weekly |
| User provisioning list | 2 | 1 | As-needed |

Monthly admin query load per 100 seats:

```
Admin users: 2
Sessions/week per admin: 5
Weeks/month: 4.3
Avg DB queries per session: 4

Total DB queries: 2 × 5 × 4.3 × 4 = 172 queries/month per 100 seats
= 1.72 queries/seat/month

Workers requests: 2 × 5 × 4.3 × 1 = 43 requests/100 seats/month
= 43 × $0.50/1M = $0.0000215/month per 100 seats
= $0.000000215/seat/month
```

**Supabase cost:** Read-replica queries are included in Team plan ($599/month base). No per-query billing. Marginal cost of admin dashboard load: $0 on the existing Supabase plan.

**Assessment: negligible.** Admin dashboard is among the highest-value enterprise features and costs effectively nothing to serve. The investment is in the product surface (see `admin-dashboard.html`), not in its runtime cost.

### 15.4 Audit log storage and long-term retention

Audit log events are generated for every material action per `docs/AUDIT_LOG_SCHEMA.md` and the HMAC-chain specification in DEC-030. Events are append-only and immutable once written.

| Audit event type | Bytes per record | Events/seat/month |
|---|---|---|
| Session completion | ~350 | 12.9 |
| SCIM provisioning event | ~480 | 0.2 |
| Admin dashboard access | ~280 | 1.72 |
| Billing / seat change | ~420 | 0.08 (annual contract) |
| Break-glass access | ~620 | < 0.01 (rare) |
| **Weighted average** | **~370 bytes** | **~14.9 events/seat/month** |

Monthly storage growth per seat:

```
14.9 events × 370 bytes = 5,513 bytes ≈ 5.4 KB/seat/month
```

**Hot tier (0–90 days in PostgreSQL `audit_log` table):**

```
90-day hot window: 5.4 KB × 3 months = 16.2 KB/seat
At 500 seats: 7.9 MB total — negligible within Supabase Team plan (1 TB)
```

**Cold tier (90 days–7 years in Supabase Storage):**

HMAC-chained `.ndjson` files, one per tenant per calendar day, synced to Supabase Storage (and optionally customer S3):

```
7-year retention: 5.4 KB × 84 months = 453.6 KB/seat total
At Supabase Storage rate $0.021/GB:
0.44 MB / 1,024 MB × $0.021 = $0.0000090/seat for full 7-year archive
Per month amortized: $0.0000090 / 84 months = $0.00000011/seat/month
```

**SIEM webhook delivery (Growth+ tier only):**

```
Events streamed to SIEM: ~14.9/seat/month
1 Workers outbound request per event
Cost: 14.9 × $0.50/1M = $0.0000075/seat/month
At 500 seats with SIEM enabled: $0.0037/month
```

**Assessment: negligible.** Seven-year tamper-evident audit log with SIEM streaming adds under $0.001/seat/month total. The value delivered (SOC 2 CC6.1 evidence, customer SIEM integration, regulatory retention compliance) is not a function of its infrastructure cost.

### 15.5 White-label cost model (Growth+, ≥ $50k ARR)

White-label delivers a custom domain (e.g., `fit.acme.com`) routing to FORM's product. Infrastructure components:

| Component | Provider | Per-tenant monthly cost | Notes |
|---|---|---|---|
| Custom domain routing | Cloudflare Workers Routes | $0 | Custom domains included in Workers Paid plan; no per-domain charge |
| SSL/TLS certificate | Cloudflare | $0 | Cloudflare-managed; auto-renewed; no per-certificate cost |
| DNS CNAME validation | Cloudflare | $0 | Tenant adds one CNAME record; Cloudflare handles resolution |
| Tenant logo + favicon storage | Supabase Storage | ~$0.001 | ~50 KB assets; $0.021/GB = $0.001/month |
| Custom color palette | CSS variable override in `tenant_config` | $0 | Theme tokens in DB row; no compute overhead |
| Accessibility floor enforcement | One-time engineering check | $0 recurring | WCAG AA contrast ratio verified at setup; not a recurring cost |

**Monthly white-label infrastructure cost: ~$0.001/tenant/month.**

The one-time setup cost is modeled in §8.2 ($800–1,500 per deal, including custom domain DNS, branding asset upload, and accessibility validation). The recurring infrastructure cost of white-label is negligible.

**Why white-label is gated at ≥ $50k ARR, not priced as a feature add-on:** The gate exists to ensure the one-time engineering setup cost is amortized against sufficient contract value, and to prevent brand dilution from small or low-commitment accounts. It is a strategic threshold, not a cost-driven pricing mechanism. The cost of delivering white-label is near-zero.

### 15.6 Multi-tenant RLS overhead

Row-Level Security enforces tenant isolation at the PostgreSQL layer (see `docs/DATA_MODEL.md §3`). Every query against a RLS-protected table has the relevant policy evaluated server-side before rows are returned.

**Performance overhead:** RLS adds 2–8% additional query latency on typical Supabase-hosted workloads — well within FORM's P99 < 200ms API response target. CPU overhead is negligible.

**Billing impact:** RLS is a PostgreSQL native feature. Supabase does not charge per-policy or per-evaluation. The Supabase plan cost is unchanged by RLS enablement.

**Scale behavior:** RLS overhead is constant regardless of tenant count. The policy `tenant_id = current_setting('app.tenant_id')::uuid` is evaluated in O(1) per row scan — it does not grow with the number of tenants on the platform.

**Cross-tenant query prevention:** RLS policies reject cross-tenant access at the DB layer before rows are returned to the application. This eliminates the need for an application-layer tenant lookup on every query, which would be both slower and less secure.

**Assessment: zero cost impact.** RLS is a security architecture decision. Its value is the isolation guarantee, not a cost tradeoff.

### 15.7 Enterprise infrastructure COGS summary

| Enterprise feature | Marginal cost/seat/month | Assessment |
|---|---|---|
| SSO/SAML token validation overhead | $0.0000065 | Negligible |
| SCIM provisioning (at 5% monthly turnover) | $0.0000002 | Negligible |
| Admin dashboard queries (2 admins/100 seats) | $0.0000002 | Negligible |
| Audit log hot-tier DB storage | $0.0000001 | Negligible |
| Audit log cold-tier (7yr Supabase Storage) | $0.0000001 | Negligible |
| SIEM webhook delivery (Growth+ only) | $0.0000075 | Negligible |
| White-label infrastructure (per tenant, amortized/seat) | $0.000002 | Negligible |
| Multi-tenant RLS | $0 | No billing impact |
| **Total enterprise feature overhead** | **< $0.00001/seat/month** | **Effectively zero** |
| **Baseline enterprise COGS (§3.3)** | **$0.34/seat/month** | From coaching sessions + voice |
| **True enterprise all-in COGS** | **~$0.34/seat/month** | No material change |

**The enterprise premium features — SSO, SCIM, admin dashboard, audit log, white-label, RLS — add less than 0.003% to per-seat infrastructure COGS.** This is the structural explanation for why enterprise infrastructure gross margin reaches 97%+. The real cost of enterprise is not compute — it is engineering time (one-time, modeled in §8.2) and customer success labor (ongoing, modeled in §8.4). Enterprise infrastructure itself is delivered at near-zero marginal cost.

---

## 16. Enterprise Tier Break-Even by Plan

This section extends §8 by computing the break-even deal economics for each enterprise plan tier (Starter, Growth, Enterprise), incorporating all cost components: infrastructure COGS, one-time implementation cost amortized over Year 1, and dedicated CS labor allocation. It identifies the minimum seat threshold below which Year 1 economics are marginal, and quantifies the economic incentive for multi-year contracts.

**Rate card used:** Starter $12/seat/month (50–200 seats), Growth $9/seat/month (200–1,000 seats), Enterprise $6–8/seat/month (1,000+ seats), per `enterprise.html` and `docs/ENTERPRISE.md`. All figures [ESTIMATE] until first 5 signed deals per tier are time-tracked.

**Assumptions:**
- Infrastructure COGS: $0.34/seat/month (§3.3 + §15.7 — enterprise features add nothing material)
- Implementation cost amortized over 12 months (Year 1 only; zero in Year 2+)
- Implementation cost midpoints from §8.2 ($1,150 for small deals, $1,500 for mid, $2,000 for large)
- CS allocation: $200/month for Starter accounts; $300/month for Growth; $400–500/month for Enterprise
- All figures are annual

### 16.1 Plan-level break-even table

| Plan | Seats | Monthly ACV | Annual ACV | Infra COGS (yr) | Impl. cost | CS cost (yr) | Total cost (yr) | Gross profit (yr) | GM% Y1 |
|---|---|---|---|---|---|---|---|---|---|
| Starter · minimum | 50 | $600 | $7,200 | $204 | $1,150 | $2,400 | $3,754 | $3,446 | 47.9% |
| Starter · maximum | 200 | $2,400 | $28,800 | $816 | $1,150 | $2,400 | $4,366 | $24,434 | 84.8% |
| Growth · minimum | 200 | $1,800 | $21,600 | $816 | $1,500 | $3,600 | $5,916 | $15,684 | 72.6% |
| Growth · maximum | 1,000 | $9,000 | $108,000 | $4,080 | $1,500 | $4,800 | $10,380 | $97,620 | 90.4% |
| Enterprise · minimum | 1,000 | $6,000 | $72,000 | $4,080 | $1,500 | $4,800 | $10,380 | $61,620 | 85.6% |
| Enterprise · mid | 2,500 | $18,750 | $225,000 | $10,200 | $2,000 | $6,000 | $18,200 | $206,800 | 91.9% |

All figures [ESTIMATE]. Implementation cost is a one-time charge amortized against Year 1 ACV. CS cost is the dedicated CSM labor allocation (§8.2).

### 16.2 Minimum viable Starter deal analysis

The 50-seat Starter deal at $12/seat/month generates $7,200 ACV but only 47.9% gross margin in Year 1. The drag is the fixed implementation ($1,150) and CS labor ($2,400/year) amortized over a small seat base.

**Year 2 dynamics:** When implementation cost drops to zero, the same 50-seat deal yields:

```
Y2 GM = ($7,200 − $204 infra − $0 impl − $2,400 CS) / $7,200
       = $4,596 / $7,200 = 63.8%
```

63.8% Year 2 margin is below the enterprise blended target (89%, §8.4) but above the consumer Pro margin floor at the lower end of the App Store tax range (~67%). The deal is economically viable across the full contract term; Year 1 is the investment round.

**The minimum floor problem:** §8.3 cites a 20-seat minimum as historically the floor, but this section reveals that 20 seats at $12/seat/month ($2,880 ACV) would generate negative gross profit in Year 1:

```
20-seat Y1 cost: $82 infra + $1,150 impl + $2,400 CS = $3,632
20-seat Y1 ACV: $2,880
Y1 gross profit: −$752 (−26.1%)
```

**20-seat deals destroy margin in Year 1 at Starter pricing.** The minimum economic floor, given current CS cost allocation, is approximately **40–50 seats at $12/seat/month** for Year 1 neutrality. This is OQ-11.

**Recommendation:** The 40-seat minimum stated in §8.3 is at the edge of viability. If dedicated CSM cost per small account exceeds the $200/month model allocation (e.g., if onboarding requires more than 1 hr/week sustained for 6 months), the minimum should rise to 75 seats. Track actual CS hours on first 5 Starter deals before committing to sub-50-seat pilots.

### 16.3 Year 1 vs. Year 2+ margin improvement by plan

Implementation cost is a one-time event. From Year 2, the same deal generates materially higher net margin:

| Plan | Seats | Y1 GM% | Y2 GM% | Improvement | Notes |
|---|---|---|---|---|---|
| Starter · 50 seats | 50 | 47.9% | 63.8% | +15.9 pp | Large Y1 drag from fixed impl/CS on small base |
| Starter · 200 seats | 200 | 84.8% | 88.0% | +3.2 pp | Fixed costs amortized; near-steady-state |
| Growth · 200 seats ($9/seat) | 200 | 72.6% | 79.6% | +7.0 pp | Y1 impl drag on lower ACV-per-seat |
| Growth · 1,000 seats | 1,000 | 90.4% | 91.8% | +1.4 pp | Fixed costs negligible vs. ACV |
| Enterprise · 1,000 seats ($6/seat) | 1,000 | 85.6% | 87.7% | +2.1 pp | CS cost dominant at lower per-seat rate |

**The Year 1 investment period** is longest and most visible at small seat counts. For Starter 50-seat accounts, Year 1 margin (47.9%) is substantially below steady-state (63.8%). This means the first cohort of small enterprise accounts will show compressed margins in investor financials until Year 2 data is available. Pre-empt this in investor communication: label Year 1 enterprise margin as "including implementation amortization" and show the Year 2 steady-state figure alongside it.

**Expansion within the contract term** changes these numbers materially. A 50-seat Starter that expands to 100 seats at M6 (mid-contract) sees effective Year 1 economics shift toward the 200-seat row — expansion revenue carries zero additional implementation cost.

### 16.4 Multi-year contract economics

Multi-year contracts (2-year at 15% discount, 3-year at 25% discount per `enterprise.html`) are offered on Growth and Enterprise tiers. Despite the lower per-year ACV, multi-year deals are economically superior for three compounding reasons:

**1. Implementation cost is a smaller percentage of lifetime ACV:**

```
200-seat Growth, $9/seat, annual:
  Y1 total cost: $816 + $1,500 + $3,600 = $5,916 on $21,600 ACV
  Implementation as % of Y1 ACV: 6.9%

Same deal, 3-year at 25% discount ($9 × 0.75 = $6.75/seat/month):
  3-year ACV: $6.75 × 200 × 36 = $48,600
  Implementation as % of 3-year ACV: 3.1%
```

The implementation cost halves as a percentage of lifetime revenue. The discount is given back to the customer; the economics improve for FORM through amortization.

**2. Logo churn risk is eliminated for the signed term:**

Enterprise logo churn is modeled at 15% annually (§8.7). In a single-year renewal model, a 10-deal cohort at $21,600 ACV loses ~1.5 logos per year. A 3-year signed contract locks all 10 logos for the full term regardless of churn signal — worth approximately $32,400 in protected ARR per renewal cycle on a 10-deal cohort.

**3. Cash flow timing:**

Annual prepay on a 3-year contract delivers 3× the upfront cash versus annual renewals. For a pre-Series A company, this materially reduces the funding needed to reach ARR milestones.

**Multi-year net margin summary (200-seat Growth example):**

| Contract structure | 3-year gross ACV | 3-year total cost | 3-year gross profit | Blended 3yr GM |
|---|---|---|---|---|
| 3× annual renewals (undiscounted) | $64,800 | $16,732 | $48,068 | 74.2% |
| 3-year prepay at 25% discount | $48,600 | $14,332 | $34,268 | 70.5% |
| Δ (discount cost to FORM) | −$16,200 ACV | −$2,400 cost | −$13,800 GP | −3.7 pp |

**The customer receives a $16,200 discount over 3 years; FORM gives up $13,800 in gross profit but eliminates ~$32,400 of churn-exposure ARR.** On a cohort basis, multi-year contracts are the correct offer for accounts where Year 2 expansion is expected (i.e., the pilot showed >50% seat utilisation at M3). Offer 3-year proactively on Growth and Enterprise tiers to accounts meeting the expansion signal threshold.

---

## 17. Downside Scenario Analysis & Unit Economics Stress Tests

### 17.1 Purpose & Scope of Stress Testing

This section stress-tests the v0.6 unit economics baseline against six adverse scenarios before the seed raise. The purpose is to identify which events are survivable (margin compresses but the business remains viable), which require active mitigation (product or pricing response needed), and which are structurally threatening (would force a strategic pivot or emergency fundraise).

All scenarios use v0.6 baseline inputs. The key baseline inputs are restated here for reference:

| Metric | Baseline (v0.6) |
|---|---|
| Pro ARPU (Western) | $19.00/month |
| App Store commission | 15% (Small Business Program) |
| Net Pro revenue after IAP | $16.15/month |
| Pro infrastructure COGS | $0.50/user/month |
| — of which Anthropic API | $0.20/user/month |
| — of which ElevenLabs voice | $0.22/user/month |
| — of which Supabase | $0.05/user/month |
| — of which Cloudflare | $0.02/user/month |
| — of which Sentry + PostHog | $0.007/user/month |
| Pro contribution margin per sub | $15.65/month |
| Pro monthly churn | 5.0% [ESTIMATE] |
| Pro subscriber average life (1/churn) | 20 months |
| Simple LTV (net revenue × avg life) | ~$323/subscriber |
| Full-burn monthly overhead | $80,000/month |
| Full-burn break-even | 5,112 Pro subscribers |

All results in this section are [ESTIMATE]. Replace with actuals at beta instrumentation.

---

### 17.2 Scenario Definitions

| ID | Scenario | Trigger |
|---|---|---|
| S1 | Anthropic API cost doubles | Rate increase, model change, or token usage spike above baseline assumption |
| S2 | App Store commission reverts to 30% | FORM crosses $1M annual App Store revenue, or Apple policy change removes SBP eligibility |
| S3 | Pro monthly churn doubles | Onboarding quality problem, seasonal drop, competitive entrant stealing active users |
| S4 | Price compression — 20% forced cut | Competitive pressure or App Store editorial pressure to match lower-cost competitors |
| S5 | Enterprise pipeline miss (Year 1) | Zero enterprise deals closed in Year 1; 100% consumer-only revenue |
| S6 | Combined moderate stress: S1 + S3 | Simultaneous API cost shock and churn spike — the "manageable worst-case" |

Scenarios are evaluated independently (S1–S5) and in combination (S6). They are not modeled as tail-risk black swans — each is a plausible operating event within the first 18 months post-launch. The goal is to know the break-even and LTV impact in advance, not to discover them under board pressure.

---

### 17.3 S1: AI API Cost Shock

**Trigger:** Anthropic doubles API rates from current list price. Pro Anthropic cost moves from $0.20 → $0.40/user/month.

**Updated COGS breakdown:**

| Cost item | Baseline | S1 | Delta |
|---|---|---|---|
| Anthropic API | $0.20 | $0.40 | +$0.20 |
| ElevenLabs voice | $0.22 | $0.22 | — |
| Supabase | $0.05 | $0.05 | — |
| Cloudflare | $0.02 | $0.02 | — |
| Sentry + PostHog | $0.007 | $0.007 | — |
| **Total COGS** | **$0.50** | **$0.70** | **+$0.20** |

**Impact on unit economics:**

```
Net Pro revenue (15% SBP):           $16.15    (unchanged)
Pro COGS under S1:                    $0.70
Contribution margin under S1:        $15.45    (vs. $15.65 baseline)

Full-burn break-even under S1:       $80,000 / $15.45 = ~5,178 Pro subscribers
vs. baseline:                                                 5,112 Pro subscribers
Delta:                                                           +66 subscribers (+1.3%)
```

Infrastructure gross margin: ($16.15 − $0.70) / $16.15 = **95.7%** [vs. 96.9% baseline; infra-only, excluding support labor]

**Assessment: S1 is survivable.** The contribution margin compresses by $0.20/subscriber/month — material at volume but not structurally threatening. Break-even rises by 66 subscribers, a 1.3% increase. The primary exposure is if S1 coincides with a period of rapid user growth, where the total monthly API bill scales proportionally.

**Mitigations (in priority order):**

1. Volume pricing negotiation with Anthropic. At $0.40 Anthropic cost per user, 5,178 subscribers = ~$2,070/month in Anthropic spend. Volume pricing thresholds are typically $10k+/month — not reachable until ~25,000 Pro subscribers at doubled rates. Flag to founder for Anthropic BD conversation as the pipeline grows (see OQ-12).
2. Token budget optimisation: Victor system prompt compression, prompt caching for static context (§7.2 estimates 35–40% input cost reduction on cached portion), and Haiku routing for non-coaching turns.
3. Model tiering: Haiku for intent classification and set acknowledgment reduces effective Anthropic cost by 20–30% if 50% of calls route to the cheaper model (§7.2).

---

### 17.4 S2: App Store Commission Shock

**Trigger:** FORM crosses $1M in annual App Store revenue, automatically exiting the Small Business Program. At $19/month ARPU, this threshold is reached at approximately:

```
$1,000,000 / ($19 × 12) ≈ 4,386 annual Pro subscribers
```

This is a success-triggered event, not an adverse external shock. However, the economics shift materially and the timing is predictable — it should be modeled before it happens.

**Updated revenue per subscriber:**

| Scenario | Gross ARPU | Store fee | Net ARPU | COGS | Contribution margin | Break-even |
|---|---|---|---|---|---|---|
| Baseline (15% SBP) | $19.00 | $2.85 | $16.15 | $0.50 | $15.65 | 5,112 |
| S2 (30% standard) | $19.00 | $5.70 | $13.30 | $0.50 | $12.80 | 6,250 |

```
Full-burn break-even under S2:  $80,000 / $12.80 = 6,250 Pro subscribers
vs. baseline:                                         5,112 Pro subscribers
Delta:                                                +1,138 subscribers (+22.3%)
```

The net revenue hit is $2.85/subscriber/month — a 17.6% reduction in contribution margin per subscriber. At 5,000 subscribers, this is a $14,250/month reduction in gross profit.

Infrastructure gross margin under S2: ($13.30 − $0.50) / $13.30 = **96.2%** (infra-only). The commission shock is not a COGS event — it is a net revenue event.

**Assessment: Triggered by success — not a pre-milestone threat.** The S2 trigger requires crossing 4,386 Pro subscribers, which is 86% of the full-burn break-even. At the point this activates, FORM is already near break-even. The break-even target simply shifts from 5,112 to 6,250 — a larger number, but one that is reachable without structural changes.

**Mitigations:**

1. **Web billing path (highest impact).** Direct web billing (Epic v. Apple precedent, EU DMA external link entitlement) bypasses App Store commission entirely. Subscribers acquired or migrated via web pay 0% commission rather than 30%. Monitoring per existing §9 note. Even a 30% web-billing mix reduces the blended commission from 30% toward ~21%, materially closing the gap to SBP economics.
2. **Annual plan mix.** Annual plan subscribers ($144/year billed once) still pay 15% commission on the single transaction regardless of total revenue tier in most App Store interpretations. Driving annual plan conversion before crossing the SBP threshold locks more subscribers into the lower effective commission rate.
3. **Explicit threshold monitoring.** Track cumulative App Store revenue in real-time via server-side billing events. Flag to founder at $750k annual run-rate (90% of SBP threshold) to trigger web billing migration sprint.

---

### 17.5 S3: Churn Spike

**Trigger:** Pro monthly churn doubles from 5% to 10%. Plausible causes: weak first-week habit formation, onboarding flow failure, competitive entrant, or seasonal drop-off (e.g., post-January resolution cohort churning in bulk).

**Cohort survival comparison:**

| Month | Baseline (5% churn) | S3 (10% churn) | Delta |
|---|---|---|---|
| M1 | 95.0% | 90.0% | −5.0 pp |
| M3 | 85.7% | 72.9% | −12.9 pp |
| M6 | 73.5% | 53.1% | −20.4 pp |
| M12 | 54.0% | 28.2% | −25.8 pp |
| M24 | 29.2% | 8.0% | −21.2 pp |

Survival computed as (1 − monthly_churn)^months.

**LTV impact:**

```
Average subscriber life = 1 / monthly_churn

Baseline:  1 / 0.05 = 20 months
S3:        1 / 0.10 = 10 months

Simple LTV (net revenue × avg life):
  Baseline: $16.15 × 20 = $323.00/subscriber
  S3:       $16.15 × 10 = $161.50/subscriber
  Delta:                  −$161.50/subscriber (−50%)
```

**CAC payback and LTV:CAC at $80 CAC:**

```
Monthly gross margin per subscriber (at 85% GM):
  $16.15 × 0.85 = $13.73/month  (unchanged by churn — churn affects LTV, not payback period)

Payback period = $80 / $13.73 = ~5.8 months  (consistent with §14.4 figures)

LTV:CAC at M24 (85% GM-adjusted LTV):
  Baseline: ~$274 / $80 = 3.4×
  S3:       ~$137 / $80 = 1.7×   — below the 3× minimum benchmark (§14.5)
```

At 10% monthly churn, GM-adjusted M24 LTV falls to approximately $137 per subscriber. Against an $80 CAC, LTV:CAC is 1.7× — below the 3× floor cited in §14.5. Paid acquisition is economically unjustifiable at these retention levels. Organic and referral channels only until churn is diagnosed and corrected.

**Assessment: S3 requires product intervention.** The break-even subscriber count does not change (churn rate does not appear in the contribution margin formula used for break-even). However, to maintain a steady-state subscriber base of 5,112 at 10% monthly churn, FORM must acquire **511 gross new subscribers per month** (vs. 256/month at 5% churn) — doubling the gross acquisition burden to maintain the same net subscriber count. At $80 CAC, that is $40,880/month in acquisition spend to hold steady, up from $20,480/month.

**Primary signal to watch:** D7 → D30 retention funnel in PostHog. If D30 paying retention drops below 55% in beta, treat as S3 onset. See §14.7 for cohort survival curve requirements.

**Mitigations:**

1. Victor onboarding quality: first-session coaching outcome must establish a habit anchor. Instrument session completion rate at the D1, D3, D7 checkpoints.
2. Deload reminder system: proactive re-engagement for users who miss two consecutive sessions. Server-side trigger on Supabase `last_session_at` field.
3. First-week program lock: users who complete 3 coached sessions in 14 days churn at materially lower rates (§14.7 activation insight). Make this the primary activation funnel metric.

---

### 17.6 S4: Price Compression

**Trigger:** Competitive pressure forces a 20% price cut. Pro tier moves from $19/month → $15.20/month.

**Updated unit economics:**

```
Gross ARPU under S4:             $15.20/month
App Store fee (15% SBP):          $2.28
Net revenue under S4:            $12.92/month   (vs. $16.15 baseline)
Pro COGS (unchanged):             $0.50
Contribution margin under S4:    $12.42/month   (vs. $15.65 baseline)

Full-burn break-even under S4:   $80,000 / $12.42 = ~6,441 Pro subscribers
vs. baseline:                                          5,112 Pro subscribers
Delta:                                               +1,329 subscribers (+26.0%)
```

Infrastructure gross margin: ($12.92 − $0.50) / $12.92 = **96.1%** (infra-only).

**Annual plan pricing constraint:** The current annual plan implies a monthly equivalent of ~$15.20/month at $19 monthly pricing with standard annual discount structure. At a $15.20 monthly price, the equivalent annual plan would need repricing to approximately $130–$139/year to maintain a meaningful discount over monthly billing — annual plan repricing is not automatic and requires deliberate product and pricing action alongside any monthly price move.

**LTV under S4:**

```
Simple LTV: $12.92 × 20 = $258.40/subscriber   (vs. $323 baseline; −20%)
GM-adjusted LTV (85%): ~$220

LTV:CAC at $80 CAC: $220 / $80 = 2.75×  — below 3× floor; organic channels only are safe at $80 CAC
```

**Assessment: Survivable but structurally weakening.** $15.20 is near the psychological floor for a premium AI fitness app. Generic fitness apps and AI-adjacent competitors (Future: $149/month; Whoop: $30/month hardware bundle) create a wide pricing corridor. FORM's defensibility at $19 rests on the CV moat and Victor coaching quality — if that moat is validated in beta, price should be held. If the market responds to premium pricing with low conversion, the correct response is product differentiation investment, not price cutting.

**Mitigation: hold the price.** Compete on quality, not price. The data to make this call is trial-to-paid conversion rate in beta, segmented by acquisition channel.

---

### 17.7 S5: Enterprise Pipeline Miss (Year 1)

**Trigger:** Zero enterprise deals close in Year 1. Consumer-only revenue for the full first operating year.

**Baseline Year 1 enterprise revenue assumption:**

From §8 and §16, a conservative Year 1 enterprise pipeline assumption is 3–5 deals closing in H2 Year 1 (post-consumer PMF). Representative deal sizes: Starter 50-seat at $12/seat/month or Growth 200-seat at $9/seat/month.

| Pipeline scenario | Deals | Representative ACV | Year 1 ARR contribution |
|---|---|---|---|
| Conservative (3 Starter deals) | 3 × 50-seat Starter | $12 × 50 × 12 = $7,200/deal | ~$21,600 |
| Moderate (2 Starter + 2 Growth) | Mixed | ~$7,200 + ~$21,600 avg | ~$57,600 |
| Optimistic (5 deals mixed) | 3 Starter + 2 Growth | ~$21,600 + ~$43,200 | ~$64,800 |

Under S5, all of this ARR is $0. Lost Year 1 ARR range: **$21,600 – $64,800** [ESTIMATE].

**Impact on break-even timeline:**

Consumer break-even is unchanged at 5,112 Pro subscribers — enterprise miss does not affect the consumer equation. The impact is on time-to-break-even and seed runway consumption:

```
Lost enterprise gross profit at ~89% GM (§8.4):
  Conservative miss: $21,600 × 0.89 = ~$19,224/year
  Optimistic miss:   $64,800 × 0.89 = ~$57,672/year

At $80k/month full burn ($960k/year):
  Conservative miss: runway impact = $19,224 / $80,000 = ~0.24 months
  Optimistic miss:   runway impact = $57,672 / $80,000 = ~0.72 months
```

Against a $2M seed at $80k/month burn (~25-month runway), S5 at its worst extends runway consumption by less than 1 month. The seed runway impact is **not material** — enterprise is Year 2+ ARR by design.

**Assessment: Not existential.** The Year 1 plan is consumer-led. Enterprise revenue in Year 1 is a bonus, not a dependency. The real risk of S5 is not financial — it is the signal risk: if zero enterprise pilots can be closed after consumer PMF, the enterprise ICP hypothesis (§8) needs revisiting. Track enterprise pipeline separately from consumer revenue metrics; do not allow enterprise delays to create false pessimism about consumer performance.

**Mitigation:** Prioritise consumer traction and PMF validation first. Run enterprise pilots only when the product is stable enough to not require major changes mid-pilot (estimated M6 post-launch). Do not staff a dedicated enterprise sales role until consumer ARR reaches $300k+ annually — enterprise should be founder-led until that threshold.

---

### 17.8 S6: Combined Moderate Stress (S1 + S3)

**Trigger:** Simultaneous API cost shock and churn spike. This is the "manageable worst-case" — two independent adverse events occurring in the same period. Not a black-swan scenario; both S1 and S3 are operationally plausible within 18 months of launch.

**Updated unit economics under S6:**

```
Net Pro revenue (15% SBP, $19 ARPU):              $16.15    (unchanged)
Pro COGS under S6 (Anthropic doubled from S1):      $0.70
Contribution margin under S6:                      $15.45    (vs. $15.65 baseline)

Full-burn break-even (contribution margin):        $80,000 / $15.45 = ~5,178 Pro subscribers
```

The contribution margin impact of S1 alone is modest (+66 subs to break-even). The churn shock (S3) does not change the static break-even count but doubles the gross acquisition burden needed to maintain any given subscriber base.

**Combined impact on steady-state acquisition requirement:**

```
To maintain 5,178 subscribers at steady state:
  Baseline churn (5%):   5,178 × 0.05 = ~259 gross new subs/month needed
  S3 churn (10%):        5,178 × 0.10 = ~518 gross new subs/month needed

At $80 CAC:
  Baseline acquisition spend to hold steady: ~259 × $80 = ~$20,720/month
  S6 acquisition spend to hold steady:       ~518 × $80 = ~$41,440/month
  Delta:                                                   +$20,720/month (+100%)
```

**LTV:CAC under S6:**

```
Average subscriber life at 10% churn:  10 months
Simple LTV:                            $16.15 × 10 = $161.50
GM-adjusted LTV (85%):                 ~$137

LTV:CAC at $80 CAC: $137 / $80 = 1.7×   — below 3× floor; same result as S3 alone
```

The combined scenario is operationally more stressful than either S1 or S3 alone because it couples higher COGS with compressed LTV. However, the infrastructure cost delta ($0.20/user/month) is small relative to the churn-driven LTV compression. The dominant risk in S6 is the churn component.

**Assessment:** S6 is survivable at a seed-stage company with $2M in the bank and a lean team, but the growth plan does not hold. The correct response to S6 is not to increase acquisition spending — it is to freeze paid acquisition, extend runway by reducing discretionary burn, and diagnose the churn root cause before resuming scale.

**Decision rule for S6:** If PostHog shows D30 cohort retention below 55% AND the Anthropic invoice increases by more than 50% month-over-month, trigger the "steady-state mode" protocol: pause all paid acquisition, extend runway by 30%, and run a 6-week Victor onboarding sprint targeting the D7 → D30 activation step.

---

### 17.9 Open Questions Added

**OQ-12: Anthropic volume pricing threshold**

At what monthly Anthropic spend does FORM qualify for volume pricing? Volume pricing negotiations with Anthropic typically begin at approximately $10,000–$50,000/month in spend.

```
At baseline Anthropic cost ($0.20/user/month):
  5,000 Pro users:   $1,000/month   — well below volume threshold
  25,000 Pro users:  $5,000/month   — approaching threshold
  50,000 Pro users:  $10,000/month  — at likely threshold entry

At S1 doubled rates ($0.40/user/month):
  25,000 Pro users:  $10,000/month  — at threshold
```

Volume pricing is not reachable at seed-stage user counts under baseline rates. Under S1 (doubled rates), volume pricing becomes relevant at ~25,000 Pro subscribers. Action: flag to founder for proactive Anthropic BD conversation when monthly API spend crosses $5,000/month (estimated at ~25,000 Pro subscribers under baseline; ~12,500 under S1 rates). Reference OQ-03.

**OQ-13: Actual token budget per session (Victor)**

The Pro Anthropic cost of $0.20/user/month is derived from assumptions in §2.1: 2,000 input tokens + 500 output tokens per session, 15.1 sessions/month. Actual token usage depends on Victor system prompt length, context window carry-forward, and tool call overhead.

Victor's system prompt is estimated at ~1,200 tokens (§2.1). If context history carry-forward is larger than modeled, or if tool calls (e.g., program adaptation writes) add tokens not in the base assumption, the actual cost could be 1.5–2× the modeled figure. Instrument actual token counts per session in beta (per OQ-01). Until replaced with actuals, the $0.20/user/month figure should be treated as a planning floor estimate, not a ceiling.

---

### 17.10 Stress Test Summary Matrix

| Scenario | Description | Contribution margin/sub | Full-burn break-even (subs) | LTV impact | Assessment |
|---|---|---|---|---|---|
| **Baseline** | v0.6 inputs | $15.65/month | 5,112 | $323 simple LTV | — |
| **S1** | Anthropic API doubles ($0.20→$0.40) | $15.45/month | 5,178 (+1.3%) | Unchanged | Survivable — modest COGS compression; mitigate with prompt caching |
| **S2** | App Store 30% commission | $12.80/month | 6,250 (+22.3%) | −20% on net ARPU | Triggered by success; manageable with web billing path and annual plan mix |
| **S3** | Churn doubles (5%→10%) | $15.65/month | 5,112 (unchanged) | −50% ($323→$162) | Requires product intervention; doubles gross acquisition spend to hold steady |
| **S4** | 20% price cut ($19→$15.20) | $12.42/month | 6,441 (+26.0%) | −20% | Survivable; LTV:CAC falls below 3× floor at $80 CAC — hold price, not cut |
| **S5** | Enterprise pipeline miss Year 1 | $15.65/month | 5,112 (unchanged) | None on consumer LTV | Not existential; enterprise is Year 2+ revenue by design |
| **S6** | S1 + S3 combined | $15.45/month | 5,178 (+1.3%) | −50% (churn-driven) | Survivable; freeze paid acquisition, run churn diagnosis sprint |

**Reading this table:** The scenarios that raise break-even the most are S4 (+26%) and S2 (+22%), both net-revenue events rather than COGS events. S3 is the highest-severity scenario despite not changing break-even count — it doubles the gross acquisition spend required to hold any given subscriber base and collapses LTV:CAC below the 3× investment threshold. S1 is the most benign event in this set.

**Monitoring priority:**

1. S3 (churn) — highest LTV impact; primary beta signal; watch D7 → D30 retention daily in PostHog
2. S2 (App Store commission) — success-triggered; monitor cumulative App Store revenue monthly against $750k threshold
3. S4 (price) — competitive intelligence; review quarterly vs. competitor pricing
4. S1 (API cost) — Anthropic invoice; review monthly against $0.20/user/month baseline; flag at 1.5×

---

---

## 18. Revenue Recognition, ARR/MRR Bridge & SaaS Metrics Taxonomy

> Owner: founder + future CFO. Audience: board, investors, auditors. Review: monthly (MRR bridge) / quarterly (ARR bridge, deferred rev waterfall).

This section models the gap between *economic value generated* (ARR/MRR) and *cash received* (billings) and *GAAP-recognized revenue* — the three numbers that diverge on day one of any annual or multi-year contract. Getting them wrong in investor reporting is fatal for Series A due diligence.

---

### 18.1 Revenue Recognition Framework (ASC 606 / IFRS 15 Alignment)

FORM's SaaS subscription is a single performance obligation: access to the platform over the contract period. Revenue is recognized *ratably over the service period*, not at invoice date.

**Core rule:**
```
Monthly Recognized Revenue = Total Contract Value ÷ Contract Duration (months)
```

| Contract type | Billing event | Revenue recognition |
|---|---|---|
| Annual (1-year) | Day 0, annual upfront | 1/12 per month for 12 months |
| Annual (monthly payment) | Month-by-month | 1/12 per month (concurrent with billing) |
| 2-year, upfront | Day 0 | 1/24 per month for 24 months |
| 3-year, upfront | Day 0 | 1/36 per month for 36 months |
| Free 90-day pilot | No billing | No revenue recognition during pilot |

**Variable consideration (seat expansion mid-contract):** When a customer adds seats mid-term, the incremental TCVis allocated over the remaining contract months. The existing seats continue on their original schedule. Seat reductions (downgrades) follow the same logic — excess recognized revenue becomes a refund obligation if seats drop below contracted minimum; this is a credit note risk, not an unbundling of the performance obligation.

**Practical expedient (ASC 606-10-50-14):** FORM may apply the right-to-invoice practical expedient — if billing equals the value delivered in the period (monthly billing = monthly service) — which allows matching of billing and revenue recognition for the monthly-payment variant. Annual upfront deals do **not** qualify and require deferred revenue treatment.

---

### 18.2 Billings vs. Revenue vs. Cash — The Three Waterfalls

For an annual upfront contract signed on 1 Jan (Starter, 100 seats × $12 × 12 months = $14,400 TCV):

```
Day 0   Invoice issued:    $14,400  → Accounts Receivable
Day 0   Cash collected:    $14,400  → Cash + Deferred Revenue $14,400

Month 1   Revenue recognized: $1,200  → Deferred Revenue ↓ to $13,200
Month 2   Revenue recognized: $1,200  → Deferred Revenue ↓ to $12,000
...
Month 12  Revenue recognized: $1,200  → Deferred Revenue = $0
```

**Three numbers at end of Month 3:**

| Metric | Amount | Note |
|---|---|---|
| Billings (invoiced YTD) | $14,400 | Cash received on Day 0 |
| Revenue (recognized YTD) | $3,600 | 3 × $1,200 |
| Deferred Revenue (balance sheet) | $10,800 | Future obligation |
| ARR (annualized run-rate) | $14,400 | Unchanged — no seats added/dropped |

**Why this matters for investor reporting:** Billings outpace revenue at the start of a high-growth year, then converge. A company growing net-new ARR fast will show rising deferred revenue on the balance sheet — this is a *positive* signal (customer cash advance), not a liability risk. Presenting the wrong line to investors conflates bookings momentum with revenue momentum.

---

### 18.3 ARR and MRR Bridge Methodology

**Definitions:**

| Term | Formula | Unit |
|---|---|---|
| ARR | Sum of all active annual contract values | $/year |
| MRR | ARR ÷ 12 | $/month |
| New ARR | ARR from customers who did not exist in prior period | $/year |
| Expansion ARR | Additional ARR from existing customers (seat adds, tier upgrades) | $/year |
| Contraction ARR | Reduced ARR from existing customers (seat removals, downgrades) | $/year |
| Churned ARR | ARR from customers who did not renew | $/year |
| Net New ARR | New + Expansion − Contraction − Churned | $/year |

**Monthly MRR bridge template** (investor-grade):

```
Opening MRR (Month N-1)          $X,XXX
  + New MRR                        +$AAA   (first invoice of new contracts)
  + Expansion MRR                  +$BBB   (upsell / seat adds in existing contracts)
  − Contraction MRR                −$CCC   (seat drops, downgrades)
  − Churned MRR                    −$DDD   (non-renewals / cancellations)
  = Closing MRR (Month N)          $Y,YYY
```

**Key rule:** MRR bridge movements are recorded on the *service start date of the change*, not the billing date. A contract signed 15 Dec for service from 1 Jan is recorded as New MRR in January.

**Enterprise tier application:**

Starter 100-seat deal signed 1 Feb ($12/seat × 100 = $1,200/month MRR):
- Feb bridge: +$1,200 New MRR
- If customer adds 25 seats in May: +$300 Expansion MRR in May
- If customer drops to 75 seats at renewal: −$300 Contraction MRR at renewal month
- If customer does not renew at 12 months: −$1,200 Churned MRR

**Consumer tier MRR bridge:** Monthly subscribers are the cleanest case — billing month = service month = MRR recognition month. Annual Pro subscribers contribute $19/month to MRR but bill once annually; the MRR represents the amortized view.

---

### 18.4 Free Pilot Timing and First Recognition Date

FORM's 90-day free pilot (no charge, no obligation) creates a specific revenue recognition question: when does ARR and MRR recognition begin?

**Rule:** Zero ARR/MRR during the pilot period. Revenue recognition begins on the first day of the paid service term.

```
Pilot phase (Day 0 → Day 90):   ARR contribution = $0
                                  Deferred Revenue = $0
                                  Billings = $0

Conversion event (Day 90):       Contract signed, first invoice issued
                                  ARR recognized from Day 91
                                  Deferred Revenue created on Day 91 (if annual upfront)
```

**Pipeline vs. ARR:** During the pilot, the deal may be tracked in the sales pipeline as "in-pilot" with a probability-weighted ARR contribution (e.g., 60% × $14,400 = $8,640 weighted ARR). This is **pipeline ARR**, never included in reported/committed ARR. Only convert to committed ARR at contract signature.

**90-day pilot cadence and ARR concentration risk:** If FORM runs 12 enterprise pilots simultaneously and 8 convert in the same calendar month, ARR will spike in that month followed by a quiet quarter — a pattern that creates ARR lumpiness. Mitigate by staggering pilot start dates across the quarter (Sales ops KPI: no more than 40% of active pilots expiring in the same calendar month).

---

### 18.5 Deferred Revenue Waterfall for Multi-Year Contracts

Multi-year deals (2-year: 15% discount; 3-year: 25% discount per pricing in §16.4) are paid upfront or annually. Each creates a deferred revenue balance that must be modeled on the balance sheet.

**Example: Growth deal, 500 seats × $9 × 12 months × 3 years (25% discount applied)**

```
Undiscounted 3-year TCV:   500 × $9 × 36 = $162,000
25% discount:              −$40,500
Discounted 3-year TCV:     $121,500

If paid upfront on Day 0:
  Day 0 Deferred Revenue:  $121,500
  Monthly recognition:     $121,500 ÷ 36 = $3,375/month
  ARR (annualized):        $3,375 × 12 = $40,500/year (discounted rate)

If paid annually (3 × annual invoices):
  Year 1 invoice:          $121,500 ÷ 3 = $40,500 (upfront, creates $40,500 Def Rev)
  Year 2 invoice:          $40,500 (on anniversary, creates $40,500 Def Rev)
  Year 3 invoice:          $40,500 (on anniversary, creates $40,500 Def Rev)
```

**Balance sheet deferred revenue schedule (annual payment variant):**

| Month | Deferred Rev Balance | Monthly Rev Recognized |
|---|---|---|
| 0 (invoice) | $40,500 | $0 |
| 1 | $37,125 | $3,375 |
| 6 | $20,250 | $3,375 |
| 12 (Year 1 close) | $0 | $3,375 |
| 12 (Year 2 invoice) | $40,500 | $0 (invoice hits same day as last recognition) |
| ... | ... | ... |
| 36 | $0 | $3,375 |

**Short-term vs. long-term deferred revenue:** For annual upfront contracts, the full deferred balance is current (recognized within 12 months). For 3-year upfront contracts, only the next 12 months of revenue is current; the remainder is long-term deferred — material for balance sheet presentation at Series A and beyond.

---

### 18.6 Net Revenue Retention (NRR) Model and Target

NRR (also called Net Dollar Retention / NDR) is the single most-watched enterprise SaaS metric for Series A investors. It measures the revenue growth from the existing customer base, independent of new customer acquisition.

**Formula:**

```
NRR = (Opening ARR + Expansion ARR − Contraction ARR − Churned ARR) ÷ Opening ARR × 100%
```

**FORM targets and benchmarks:**

| Company stage | NRR target | Benchmark reference |
|---|---|---|
| Pre-scale (<$1M ARR) | ≥ 100% | Survival floor |
| Growth ($1M–$10M ARR) | ≥ 110% | Good SaaS B2B |
| Scale ($10M+ ARR) | ≥ 120% | Top-quartile; used in §14.6 LTV model |
| Best-in-class | 130–140% | Snowflake, Datadog at peak |

**FORM NRR model (Year 2 target at 120% NRR):**

Starting 100-seat Starter customer ($14,400 ARR):

```
Opening ARR (Month 1):          $14,400
  + Expansion (seats 100→125):  +$3,600   (25 seat add at renewal)
  − Contraction (no downgrade):  $0
  − Churned (no churn):          $0
Closing ARR (Month 13):         $18,000

NRR = $18,000 ÷ $14,400 = 125% ✓ (exceeds 120% target)
```

**NRR sensitivity to churn rate (100-seat Starter cohort, Year 2):**

| Annual churn rate | Churned ARR | Expansion needed for 110% NRR | Expansion needed for 120% NRR |
|---|---|---|---|
| 5% | $720 | +$2,120 (15 seats) | +$3,320 (23 seats) |
| 10% | $1,440 | +$2,840 (20 seats) | +$4,040 (28 seats) |
| 20% | $2,880 | +$4,280 (30 seats) | +$5,480 (38 seats) |

**NRR data pipeline requirement:** NRR is a trailing 12-month (T12M) metric. Computing it requires:
1. A snapshot of all active contracts at T-12 months (opening cohort)
2. Tracking of every expansion, contraction, and churn event in the intervening 12 months for that exact cohort
3. Strict exclusion of any new logos that joined after T-12 (they belong in New ARR, not NRR)

This requires a contract-events table in the data warehouse — logged at the moment of each change, not reconstructed from billing. Instrument from Day 1 of first enterprise contract (pre-condition for Series A data room).

**Gross Revenue Retention (GRR):** NRR minus the expansion component. GRR measures retention only (churn + contraction as a floor):

```
GRR = (Opening ARR − Contraction ARR − Churned ARR) ÷ Opening ARR × 100%
```

GRR cannot exceed 100%. Target ≥ 90% (top-quartile B2B SaaS). A GRR of 85% with NRR of 120% means the expansion engine is working hard to mask a churn problem — flag this pattern early.

---

### 18.7 Investor-Grade SaaS Metrics Taxonomy

Single-source definitions to prevent inconsistent numbers across pitch decks, data rooms, and board reports.

| Metric | Definition | FORM notes |
|---|---|---|
| **ARR** | Annualized value of all active, contracted recurring revenue | Includes enterprise (committed) + Pro annual (committed). Excludes monthly Pro (volatile), free tier, pilot revenue. |
| **MRR** | ARR ÷ 12 | Use ARR for enterprise reporting; MRR for consumer/PLG reporting |
| **ACV** | Annual Contract Value = ARR ÷ number of customers | Starter ACV floor: $7,200 (50 seats × $12 × 12). Growth ACV mid: $21,600 (200 seats × $9 × 12). |
| **TCV** | Total Contract Value = ACV × contract years | Includes multi-year deals; TCV > ARR for 2/3-year deals |
| **Bookings** | TCV of contracts signed in the period | Booking a 3-year Growth deal = booking $121,500 TCV |
| **Billings** | Cash invoiced in the period | For annual upfront: billings spike in Month 1; for monthly: billings ≈ revenue |
| **Revenue** | GAAP-recognized: ratably over service period | Always lags billings for annual/multi-year upfront |
| **NRR** | Net Revenue Retention (see §18.6) | 120% target at growth stage |
| **GRR** | Gross Revenue Retention (see §18.6) | ≥ 90% target |
| **CAC** | Customer Acquisition Cost = total sales & marketing spend ÷ new customers | Enterprise CAC: model in §8.5. Consumer CAC: model in §14.4 |
| **LTV** | Lifetime Value = ARPU × gross margin % ÷ monthly churn rate | Enterprise LTV: model in §14.6 |
| **LTV:CAC** | Ratio target ≥ 3× (floor), ≥ 5× (healthy) | |
| **Payback period** | CAC ÷ (ARPU × gross margin %) | Enterprise target < 18 months |
| **Magic number** | Net New ARR ÷ prior quarter S&M spend | Target > 0.75 (efficient) |
| **Rule of 40** | ARR growth rate % + FCF margin % ≥ 40 | Not applicable pre-$1M ARR; compute at Series A |

**Reporting cadence:**

| Metric | Frequency | Owner | Audience |
|---|---|---|---|
| MRR bridge | Monthly | data-engineer | Founder, board |
| ARR bridge | Quarterly | Founder | Board, investors |
| NRR / GRR | Quarterly (T12M) | data-engineer | Board, investors |
| Deferred revenue waterfall | Monthly | Founder (finance) | Auditors, board |
| Bookings vs. revenue bridge | Monthly | Founder | Board |
| Magic number | Quarterly | data-engineer | Board |

---

### 18.8 Open Questions Added

**OQ-14: Revenue recognition policy for seat-count adjustments mid-contract**

When a customer adds seats on Day 45 of a 12-month annual contract (67% remaining), what is the incremental recognized revenue per period? The draft treatment is *allocation of incremental TCV over remaining months* (modification type: addition of distinct goods — new seats are functionally similar to contracted seats but start on a different date). This requires confirmation from FORM's future auditors and should be included in the accounting policy memo prior to Series A close.

**OQ-15: 90-day pilot convertibility and ASC 606 variable consideration**

If the 90-day pilot is structured as a conditional discount (i.e., the full-year contract exists from Day 0 with a "waive first 3 months" provision), the TCV is recognized over 15 months rather than 12. If the pilot is a separate, stand-alone free trial with no commitment, revenue recognition begins strictly at Day 91. The legal structure of the pilot agreement must be aligned with this accounting distinction before the first enterprise contract is signed. Flag to compliance-officer and future CFO at Series A prep.

---

---

## 19. Enterprise Go-to-Market Financial Model

> Owner: founder. Audience: founder, future VP Sales, board. Review: monthly during enterprise ramp; quarterly thereafter.

This section models enterprise revenue as a capacity-and-velocity system: sales headcount → qualified pipeline → closed ARR → recognized revenue. It answers the pre-Series A questions: *how many people does it take to close 12 enterprise deals in Year 2, what does it cost, and what does the ARR look like month by month?*

---

### 19.1 Sales Velocity Equation

The sales velocity equation converts pipeline activity into a predictable closed-ARR rate:

```
Sales Velocity ($/month) = (Opportunities × Win Rate × ACV) ÷ Sales Cycle (months)
```

FORM enterprise planning inputs [ESTIMATE]:

| Input | Year 1 (founder-led) | Year 2 (AE-led) | Source |
|---|---|---|---|
| Win rate | 25% | 25% | Mid-market B2B SaaS benchmark; calibrate from first 5 outcomes |
| Blended ACV | $18,000 | $23,000 | Starter-heavy Y1; Starter + Growth mix Y2 (see §19.4) |
| Sales cycle | 6 months | 6 months | Average of 5–7 month range (ENTERPRISE.md) |
| Target closed deals | 2 | 12 | ENTERPRISE.md targets |
| Opportunities needed | 8 | 48 | Target ÷ Win Rate |

```
Year 1 Sales Velocity: (8 × 0.25 × $18,000) ÷ 6 = $6,000/month closed ARR [ESTIMATE]
Year 2 Sales Velocity: (48 × 0.25 × $23,000) ÷ 6 = $46,000/month closed ARR [ESTIMATE]
```

**The 4× pipeline coverage rule:** To hit Year 2 target of 12 closed deals, maintain ≥ 48 opportunities in active pipeline at any time. Pipeline health is the leading indicator; closed ARR is the lag. When pipeline coverage drops below 4× the close target for two consecutive months, treat it as a critical signal — not a planning note.

---

### 19.2 Pipeline Stage Conversion Model

Based on the ENTERPRISE.md sales process stages, assuming a 5–7 month average cycle:

| Stage | Typical timing | Stage-to-next conversion | Disqualification signal |
|---|---|---|---|
| Outbound / inbound lead qualified | Week 1–2 | 30% → Discovery | No named sponsor, < 50 seat budget |
| Discovery call (champion identified) | Week 2–4 | 50% → Technical eval | Champion has no budget authority |
| Technical evaluation + security review | Week 4–8 | 60% → Pilot agreement | SSO/SCIM incompatible IdP; no DPA authority |
| 90-day pilot agreement signed | Week 8–12 | 70% → Contract | Pilot activation < 40% at Day 60 |
| Legal / DPA / MSA execution | Week 12–24 | 85% → Close | Insurance risk-scoring or backdoor request (auto-decline) |
| **Lead → signed contract** | **5–7 months** | **≈ 25% overall** | — |

**Stage-exit gates (hard):**
- Discovery → Technical: champion has confirmed internal sponsor with budget authority
- Technical → Pilot: IT/security sign-off on SSO/SCIM; DPA in review; no-go check passed (ENTERPRISE.md §5)
- Pilot → Contract: ≥ 60% pilot activation at Day 60; legal cleared to proceed; MSA review scheduled
- Contract → Close: MSA + DPA fully executed; SOC 2 bridge letter or report shared

**BANT disqualification triggers:**
- Budget: < 50 contracted seats → deal floor not met (§16.2); decline or re-scope
- Authority: no named sponsor with budget sign-off → stall risk; add as disqualification criterion
- Need: no wellness program owner internally → no adoption driver; deprioritize
- Timeline: > 9 months to decision → move to nurture; re-qualify at 90-day cadence

---

### 19.3 Founder-Led Phase (Deals 0 → 3)

The first three enterprise deals are closed by the founder. No explicit AE cash cost — the cost is founder time (opportunity cost: not on product, not on fundraising). Budget it as a time allocation, not a line item.

**Founder time per enterprise deal [ESTIMATE]:**

| Activity | Hours |
|---|---|
| Outreach and qualification | 4 |
| Discovery call(s) | 3 |
| Technical evaluation support | 6 |
| Pilot kickoff and 90-day oversight | 10 |
| Legal and contract negotiation | 8 |
| Onboarding (D0 → D60, handoff to CS) | 6 |
| **Total per deal** | **37 hours** |

At a 5-hour/week enterprise allocation (~15% of a 33-hour founder week), one deal in active stages 3–5 is manageable. Two concurrent deals at different pipeline stages is the practical maximum before quality degrades or founder misses product milestones.

**Founder-led close timeline (Base Case):**
- Month 2–3: first pilot signed (outreach begins Month 0–1)
- Month 7–8: Deal 1 closes post-pilot conversion
- Month 10–11: Deal 2 closes (pipeline built in parallel with Deal 1)
- Month 13–14: Deal 3 closes; AE search has started at Month 10–11

**AE hire trigger:** when the founder is simultaneously holding ≥ 3 deals in stages 3–5, or loses ≥ 2 qualified deals due to bandwidth. Search lead time: ~3 months (1 month recruiting + 2 months ramp to first close). Initiate AE search no later than when Deal 2 is in legal/contract stage.

---

### 19.4 First AE Hire Economics

| Parameter | Value | Notes |
|---|---|---|
| Total compensation (OTE) | $120,000/yr | $60k base + $60k variable [ESTIMATE]; reference §8.5 |
| Target quota | 20 deals/year | Calibrated to 4× OTE quota multiple target |
| Deal mix (Year 2) | 60% Starter / 35% Growth / 5% Enterprise | Starter-heavy while brand is building |
| Blended ACV | $23,000 [ESTIMATE] | 0.60 × $14,400 + 0.35 × $27,000 + 0.05 × $100,800 |
| Annual ARR at quota | $460,000 | 20 deals × $23,000 |
| Quota multiple | 3.8× OTE | Below 4× target; see adjustment options below |

**Quota multiple adjustment levers:**
```
Current: 20 deals × $23,000 ACV = $460,000 ARR → 3.8× OTE

To reach 4× ($480,000 ARR):
  Option A: Shift 2 Starter → Growth deals in AE's target account list
  Option B: Require minimum 300-seat Starter floor (raises blended ACV ~$1k)
  Option C: Add one Enterprise-tier target per half-year to AE's named accounts

Do not adjust by cutting AE OTE — compresses the available talent pool.
```

**AE gross profit contribution (Year 2, steady-state):**
```
ARR production at quota:          $460,000
Gross margin @ 65%:               $299,000 GP generated
AE total comp:                   −$120,000
Net GP contribution:              $179,000/year [ESTIMATE]
Monthly GP contribution (avg):     $9,917/month including 2-month ramp
```

**AE ramp model:**

| Month post-hire | Productivity | Deals closed | Quarterly ARR closed |
|---|---|---|---|
| 1–2 | 0% | 0 | $0 (pipeline building only) |
| 3–4 | 50% | 2 | $46,000 |
| 5–6 | 75% | 3 | $69,000 |
| 7–12 | 100% | 12 | $276,000 |
| **Full Year 1 (AE)** | — | **17** | **$391,000** |

Ramp cost to GP break-even: Month 1–2 AE salary = $20,000 outlay before first closed deal. Month 3 first close: $23,000 ACV × 65% GM = $14,950 annual GP ($1,246/month recognized). Full ramp-cost recovery at Month 5 post-hire (cumulative GP covers ramp salary spend).

---

### 19.5 SDR Layer Economics

The SDR hire is justified when AE pipeline coverage drops below 4× for two consecutive months, meaning the AE is self-sourcing all pipeline, which degrades close rate. Premature SDR hire creates pipeline generation without close capacity.

**SDR hire trigger:** pipeline coverage < 4× for 2 consecutive months AND AE is sourcing > 70% of own pipeline.

| Parameter | Value | Notes |
|---|---|---|
| SDR OTE | $70,000/yr | $42k base + $28k variable [ESTIMATE] |
| SQL quota | 40 SQLs/year | ~3–4 qualified meetings/month delivered to AE |
| SQL → closed deal conversion | 25% | Consistent with pipeline model (§19.2) |
| Deals attributable to SDR | 10/year | 40 SQLs × 25% |
| ARR from SDR-sourced deals | $230,000 | 10 × $23k blended ACV |
| GP from SDR-sourced deals | $149,500 | $230k × 65% |
| SDR net GP contribution | $79,500/year | After $70k SDR cost |
| SDR ROI | 2.1× | GP generated ÷ SDR cost |

**SDR outreach model for FORM's ICP:**

```
ICP: Head of People, CHRO, or Benefits Manager at companies with 200–5,000 employees
     in wellness-forward industries (tech, professional services, CPG)

Outreach channels (ranked by FORM fit):
  1. LinkedIn Sales Navigator — people-leader titles, company size filter
  2. Intent signal tools (Bombora, G2) — "employee wellness" or "fitness benefits"
  3. Conference presence — SHRM Annual, Wellbeing at Work, Health 2.0
  4. Closed-customer referrals — warmest lead source, zero SDR cost

Capacity model:
  Active sequences: 50 accounts/week (personalized, not mass spray)
  Response rate:    3–5% → 2 conversations/week
  SQL rate:         50% of conversations → ~1 SQL/week = 45–50 SQLs/year
```

**SQL qualification criteria (passed to AE before handoff):**
1. Company ≥ 50 confirmed employees in target geo
2. Named internal sponsor with budget authority confirmed in conversation
3. Wellness program budget confirmed or in active planning cycle
4. Next step is a scheduled technical evaluation or demo with IT/HR attendees

---

### 19.6 CSM Cost Absorption Bridge

CSM cost is the single largest post-platform-COGS cost for enterprise accounts below 600 seats. The absorption bridge below shows at what seat count CSM cost falls below 5% of account revenue — the threshold at which it no longer materially compresses gross margin.

| Plan | Seats | Monthly revenue | CSM cost/month [ESTIMATE] | CSM % of revenue | Absorbed? |
|---|---|---|---|---|---|
| Starter | 50 | $600 | $0 (email only, shared CS pool) | 0% | Yes |
| Starter | 150 | $1,800 | $0 | 0% | Yes |
| Growth | 200 | $1,800 | $275 | 15.3% | No |
| Growth | 300 | $2,700 | $275 | 10.2% | No |
| Growth | 500 | $4,500 | $275 | 6.1% | Approaching |
| Growth | 700 | $6,300 | $275 | 4.4% | **Yes** |
| Enterprise | 1,000 | $7,000 (@ $7/seat) | $450 | 6.4% | Approaching |
| Enterprise | 1,500 | $10,500 | $450 | 4.3% | **Yes** |
| Enterprise | 5,000 | $33,333 | $500 | 1.5% | De minimis |

**Implication:** The CSM cost problem is a Growth-tier, sub-700-seat problem. For Growth accounts below 300 seats, CSM consumes > 10% of revenue — the largest single margin lever at that deal size. Three mitigation paths:

1. **Pooled CSM model:** one CSM handles 3–4 sub-300-seat Growth accounts concurrently; effective per-account CSM cost reduces to ~$92/month/account
2. **Scaled CS for sub-300 seats:** async support + shared playbooks, not dedicated CSM; reserve named CSM for ≥ 400 seats
3. **Raise Growth minimum to 300 seats (OQ-16):** eliminates the below-threshold band; weigh vs. pipeline lost in the 200–299-seat range

**COGS vs. S&M classification:**

| CSM activity | Classification |
|---|---|
| Onboarding, SSO/SCIM setup, admin training | COGS (delivery of contracted service) |
| D0–D60 adoption enablement | COGS |
| QBR facilitation, success metrics review | S&M (retention / renewal) |
| Expansion selling, upsell discovery | S&M |

At current scale (< 10 enterprise accounts), classify 100% of CSM cost as S&M (operationally simpler; consistent with pre-revenue GAAP guidance). Migrate to blended COGS/S&M treatment pre-Series A audit — flag to compliance-officer.

---

### 19.7 Enterprise Revenue Forecast (Month 1–24)

Three scenarios using inputs from §19.1–19.6. All enterprise ARR is recognized ratably per §18.1. No revenue is recognized during the 90-day free pilot. Pilot success rate: 70% (30% of signed pilots do not convert to a paid contract).

**Common assumptions:**
- Month 0: product launch; enterprise outreach begins
- First pilot signed: Month 2–4
- First contract signed: Month 7–10 (90-day pilot + legal cycle)
- AE hire: Month 12 (Base), Month 10 (Bull), Month 15 (Bear)
- AE ramp: 2 months to first close post-hire
- Year 1 blended ACV: $18,000; Year 2: $23,000

**Base Case — 2 deals Year 1, 12 deals Year 2:**

| Month | Event | Cumulative deals | Monthly ARR recognized | Enterprise ARR |
|---|---|---|---|---|
| 0–6 | Outreach, pilots running | 0 | $0 | $0 |
| 7 | Deal 1: 100-seat Starter | 1 | $1,200 | $14,400 |
| 10 | Deal 2: 75-seat Starter | 2 | $2,100 | $25,200 |
| 12 | Year 1 close; AE hired | 2 | $2,100 | $25,200 |
| 14 | Deal 3 (founder-led) | 3 | $3,050 | $36,600 |
| 18 | Deals 4–7 (AE at full productivity) | 7 | $7,375 | $88,500 |
| 21 | Deals 8–10 | 10 | $10,708 | $128,500 |
| 24 | Deals 11–14 (first Growth deal) | 14 | $15,667 | $188,000 |

**Bull Case — 3 deals Year 1, 18 deals Year 2:**

| Month | Cumulative deals | Monthly ARR recognized | Enterprise ARR |
|---|---|---|---|
| 7 | 1 | $1,200 | $14,400 |
| 10 | 2 | $2,100 | $25,200 |
| 12 | 3 | $3,350 | $40,200 |
| 18 | 10 | $10,708 | $128,500 |
| 24 | 21 | $26,000 | $312,000 |

**Bear Case — 1 deal Year 1, 6 deals Year 2:**

| Month | Cumulative deals | Monthly ARR recognized | Enterprise ARR |
|---|---|---|---|
| 9 | 1 | $1,200 | $14,400 |
| 12 | 1 | $1,200 | $14,400 |
| 18 | 4 | $4,200 | $50,400 |
| 24 | 7 | $7,175 | $86,100 |

**Scenario interpretation:** The bull/bear spread at Month 24 is $86k – $312k ARR — a 3.6× range. This spread is driven primarily by pilot velocity in Months 2–4 and pilot-to-contract conversion rate — both founder-controllable before any sales hire. The two highest-leverage actions in Year 1 are: (a) signing the first pilot before Month 3, and (b) maintaining pilot activation ≥ 60% by Day 60.

**Series A context for enterprise ARR:** Enterprise ARR at Month 24 in all scenarios is sub-$350k — pre-material relative to a $2M–5M raise. Enterprise matters at Series A for *ARR quality* (NRR ≥ 110%, LTV:CAC ≥ 5×, expansion proof) rather than raw enterprise ARR quantity. Primary enterprise signal for Series A: ≥ 3 active accounts with ≥ 90% seat activation and measurable positive NRR.

---

### 19.8 Open Questions Added

**OQ-16: Growth-tier minimum seat floor for CSM economics**

§19.6 shows dedicated CSM cost (≥$275/month) consumes > 10% of revenue for Growth accounts below 300 seats. The current Growth minimum is 200 seats (ACV: $21,600). Raising the floor to 300 seats (ACV: $32,400) improves Year 1 gross margin by ~4 points on these accounts, at the cost of losing the addressable 200–299-seat market segment. Before adjusting: estimate how many prospects fall in the 200–299-seat band from CRM and outreach data. If fewer than 2–3 anticipated per year, raising the floor is economically rational. Flag to product-manager and founder at first Growth deal close.

**OQ-17: Win rate calibration — is 25% defensible?**

The 25% win rate is an industry median for mid-market B2B SaaS, not a FORM-specific measurement. The sensitivity is material:

```
Win rate 15% → 80 opportunities needed for 12 Year 2 closed deals
               (vs. 48 at 25%) — requires SDR hire at Month 10 vs. 14
Win rate 25% → 48 opportunities (base model)
Win rate 35% → 34 opportunities — AE self-sources; SDR deferred to Month 18+
```

Instrument win rate from the first five closed/lost outcomes. Create a closed-lost reason taxonomy before any AE is hired so the data is structured from Day 1. Until five data points exist, target pipeline coverage at 5× the close target (conservative buffer for the 15% scenario) rather than the 4× base model.

---

---

## 20. Customer ROI Model & Business Case Template

> Audience: customer-success, enterprise-architect, founder (sales conversations). This section models the **buyer's** return on investment — not FORM's economics. Use it to equip the sales champion with defensible numbers for their internal approval process.

> Privacy note: the ROI model relies exclusively on aggregate and anonymized signals. No individual health data enters this calculation. HR sees activation rates, not individual records. This is enforced at the data layer per the privacy floor.

---

### 20.1 Framework & Audience

Enterprise wellness buyers face two approval gates: (1) the CFO who wants a dollar figure, and (2) the CHRO/People lead who needs the programmatic case. This section gives the sales champion inputs for both.

**ROI model structure:**

```
ROI = (Value delivered − FORM annual cost) / FORM annual cost × 100%

Value delivered = direct cost savings
               + absenteeism reduction value
               + productivity uplift value
               + turnover avoidance value
```

**Model inputs:** All values are [ESTIMATE] drawn from published research and industry benchmarks. Replace with actuals as pilot data accumulates. Sources cited in each subsection. Conservative assumptions used throughout — the base model is designed to be defensible under CFO scrutiny, not to maximise the headline number.

---

### 20.2 Status Quo Cost Baseline

What the average prospect spends before FORM:

| Wellness program component | Typical spend/seat/month | Utilisation rate | Effective cost per active participant |
|---|---|---|---|
| Corporate gym stipend / membership | $25–50 | 15–25% | $100–333 |
| Third-party wellness app bundles (Calm, Peloton Corporate, etc.) | $8–15 | 20–35% | $23–75 |
| Annual health risk assessments (HRA) | $17–33 (amortised) | 60–80% | $21–55 |
| Fitness class reimbursement programs | $10–25 | 10–20% | $50–250 |
| **Combined fitness/wellness line (fitness-specific)** | **$43–90** | **~20% blended** | **$215–450** |

Source: Willis Towers Watson 2024 Global Benefits Attitudes Survey; Mercer 2023 National Survey of Employer-Sponsored Health Plans.

**Key insight:** the average employer spends **$43–90/seat/month** on fitness and wellness benefits, but only **15–25%** of employees actively use them. The effective cost per *active* participant is **$215–450/month** — 24–50× higher than FORM's per-seat cost at target engagement levels.

---

### 20.3 FORM Value Levers

Five quantifiable levers. Conservative estimates throughout; sensitivity shown in §20.5.

#### Lever 1: Utilisation arbitrage

FORM target activation rate (65%) vs. typical gym stipend utilisation (20%):

```
Current wellness program active participants:    20% of N
FORM active participants (pilot target):         65% of N
Utilisation uplift factor:                       3.25×
```

At the same per-seat cost, FORM delivers **3.25× more engaged employees**. CFO framing: same budget, triple the coverage.

The 90-day free pilot validates this assumption before the annual contract commits any budget. Success criterion: ≥ 55% activation by Day 30 (conservative threshold; below base model to give the pilot room to prove itself).

#### Lever 2: Absenteeism reduction

Published evidence on structured exercise programs and absenteeism:

| Source | Finding |
|---|---|
| RAND Corporation (2013) — wellness meta-analysis, 22 studies | Wellness programs reduce sick-day frequency by **28%** among participants |
| Baicker et al. (2010) — J. Occupational and Environmental Medicine | Medical cost ROI: **$3.27** returned per $1 spent on wellness |
| Goetzel et al. (2018) — J. Occupational and Environmental Medicine | High-activity employees have **31% lower** voluntary absenteeism |
| BLS (2023) — National Compensation Survey | US private-sector baseline absenteeism: **9.0 sick days/year** |

Base model uses **50% of the RAND estimate** (14% absenteeism reduction) to reflect uncertainty in FORM-specific vs. generic wellness program attribution:

```
Baseline absenteeism:              9.0 days/year (BLS 2023)
FORM participant reduction:        14% (50% of RAND 28%, conservative)
Days saved per active employee:    9.0 × 0.14 = 1.26 days/year

For 300-seat Growth deal (65% activation = 195 active employees):
  Days saved: 195 × 1.26 = 245.7 days/year
  Average daily wage ($75k salary): $75,000 / 250 working days = $300/day
  Absenteeism value: 245.7 × $300 = $73,710/year
```

#### Lever 3: Productivity uplift

Published evidence on exercise and workplace cognitive performance:

| Source | Finding |
|---|---|
| Lambourne & Tomporowski (2010) — meta-analysis | Post-exercise executive function uplift: **+13–17%** |
| Pronk et al. (2004) — J. Occupational and Environmental Medicine | High-fitness employees: **11%** higher productivity scores |
| WHO Global Action Plan on Physical Activity (2020) | Active adults: 15–30% lower risk of depression/anxiety (productivity co-benefit) |

Conservative model uses **0.5% productivity uplift** (vs. Pronk's measured 11%) to remain defensible under CFO scrutiny:

```
Active employees: 195 (300-seat Growth at 65% activation)
Average salary: $75,000 → hourly rate $36/hour
Productive hours/day: 6 (excluding meetings, admin)
Productivity uplift: 0.5%
Annual value: 195 × 6h × $36 × 250 days × 0.005 = $52,650/year
```

Disclose the conservative basis when presenting: "we used 1/22nd of the published effect size." This builds credibility with financially literate buyers.

#### Lever 4: Turnover avoidance

Integrated Benefits Institute (2023): employees enrolled in comprehensive wellness programs show **10–15% lower voluntary turnover** vs. non-enrolled peers. Model uses 1 departure prevented per year for the base scenario (conservative; does not require a population-level attribution claim):

```
Turnover replacement cost: $9,000 (SHRM 2022 direct-cost estimate for knowledge-worker role)
Events prevented: 1 per year (conservative; no attribution claim required)
Annual value: $9,000

Note: at the 10% lower-turnover figure for 195 active employees (20% baseline annual turnover):
  195 × 0.20 × 0.10 = 3.9 departures prevented → $35,100/year
  The base model uses 1 event ($9,000) as the highly conservative floor.
```

#### Lever 5: Direct cost comparison (if FORM displaces gym stipend)

```
Status quo gym stipend:   $30/seat/month (conservative estimate)
FORM Growth tier:          $9/seat/month
Net saving per seat:       $21/seat/month = $252/seat/year

For 300 seats: $75,600/year direct budget reduction
```

In the supplement model (FORM is an *addition* rather than a *replacement*), this lever is zero and the ROI tables below use only Levers 2–4. Present both scenarios to the buyer; CFOs prefer the supplement model because it avoids internal benefit-cutting politics.

---

### 20.4 Three-Year ROI Table by Tier

**Model parameters:** supplement model (no gym stipend displacement), conservative inputs. Pilot year (Day 0–90) is free and not included in the 3-year count. Seat counts are illustrative mid-points within each tier band.

Common assumptions:
- Activation rate: 65% (base); all levers applied only to active employees
- Average salary: $75,000 (US knowledge-worker median)
- Absenteeism: 9.0 days/year baseline, 14% reduction, $300/day
- Productivity uplift: 0.5% applied to 6 productive hours/day, 250 working days
- Turnover avoidance: 1 event per year at $9,000 replacement cost

#### 20.4.1 Starter Tier — 100 seats at $12/seat/month ($14,400/year)

| Item | Year 1 | Year 2 | Year 3 | 3-year total |
|---|---|---|---|---|
| FORM annual cost | $14,400 | $14,400 | $14,400 | $43,200 |
| Active employees (65%) | 65 | 65 | 65 | — |
| Absenteeism reduction value | $24,570 | $24,570 | $24,570 | $73,710 |
| Productivity uplift (0.5%) | $17,550 | $17,550 | $17,550 | $52,650 |
| Turnover avoidance | $9,000 | $9,000 | $9,000 | $27,000 |
| **Total value delivered** | **$51,120** | **$51,120** | **$51,120** | **$153,360** |
| **Net benefit** | **$36,720** | **$36,720** | **$36,720** | **$110,160** |
| **ROI** | **255%** | **255%** | **255%** | **255%** |
| **Payback period** | **3.4 months** | — | — | — |

#### 20.4.2 Growth Tier — 300 seats at $9/seat/month ($32,400/year)

| Item | Year 1 | Year 2 | Year 3 | 3-year total |
|---|---|---|---|---|
| FORM annual cost | $32,400 | $32,400 | $32,400 | $97,200 |
| Active employees (65%) | 195 | 195 | 195 | — |
| Absenteeism reduction value | $73,710 | $73,710 | $73,710 | $221,130 |
| Productivity uplift (0.5%) | $52,650 | $52,650 | $52,650 | $157,950 |
| Turnover avoidance | $9,000 | $9,000 | $9,000 | $27,000 |
| **Total value delivered** | **$135,360** | **$135,360** | **$135,360** | **$406,080** |
| **Net benefit** | **$102,960** | **$102,960** | **$102,960** | **$308,880** |
| **ROI** | **318%** | **318%** | **318%** | **318%** |
| **Payback period** | **2.9 months** | — | — | — |

#### 20.4.3 Enterprise Tier — 1,000 seats at $7/seat/month ($84,000/year)

| Item | Year 1 | Year 2 | Year 3 | 3-year total |
|---|---|---|---|---|
| FORM annual cost | $84,000 | $84,000 | $84,000 | $252,000 |
| Active employees (65%) | 650 | 650 | 650 | — |
| Absenteeism reduction value | $245,700 | $245,700 | $245,700 | $737,100 |
| Productivity uplift (0.5%) | $175,500 | $175,500 | $175,500 | $526,500 |
| Turnover avoidance (3 events) | $27,000 | $27,000 | $27,000 | $81,000 |
| **Total value delivered** | **$448,200** | **$448,200** | **$448,200** | **$1,344,600** |
| **Net benefit** | **$364,200** | **$364,200** | **$364,200** | **$1,092,600** |
| **ROI** | **434%** | **434%** | **434%** | **434%** |
| **Payback period** | **2.3 months** | — | — | — |

#### Summary across tiers:

| Tier | Seats | Annual FORM cost | Annual value | 1-year ROI | 3-year net benefit | Payback |
|---|---|---|---|---|---|---|
| Starter | 100 | $14,400 | $51,120 | 255% | $110,160 | 3.4 months |
| Growth | 300 | $32,400 | $135,360 | 318% | $308,880 | 2.9 months |
| Enterprise | 1,000 | $84,000 | $448,200 | 434% | $1,092,600 | 2.3 months |

Larger deals deliver higher ROI because fixed value components (turnover avoidance) scale sub-linearly with seats while absenteeism and productivity scale linearly, improving the ratio.

---

### 20.5 ROI Sensitivity Analysis

Three key assumptions drive the model. All sensitivity tables use the **Growth 300-seat scenario** ($32,400/year) as the reference deal.

#### 20.5.1 Activation rate sensitivity

| Activation rate | Active employees | 1-year value | 1-year ROI | Payback |
|---|---|---|---|---|
| 40% (pessimistic) | 120 | $84,480 | 161% | 4.5 months |
| 55% (conservative) | 165 | $115,160 | 255% | 3.4 months |
| **65% (base)** | **195** | **$135,360** | **318%** | **2.9 months** |
| 75% (optimistic) | 225 | $155,250 | 379% | 2.5 months |
| 85% (stretch) | 255 | $175,140 | 441% | 2.2 months |

**Floor check:** at 40% activation (pessimistic), ROI is still **161%** with a **4.5-month payback**. Use 55% as the pilot success criterion to give the programme room to prove itself while maintaining a positive ROI case even in a downside activation scenario.

#### 20.5.2 Average salary sensitivity (absenteeism value driver)

| Avg salary | Day rate | Absenteeism value (195 active) | 1-year total value | 1-year ROI |
|---|---|---|---|---|
| $50,000 | $200 | $49,140 | $111,990 | 246% |
| $75,000 | $300 | $73,710 | $136,560 | 321% |
| **$100,000** | **$400** | **$98,280** | **$161,130** | **397%** |
| $125,000 | $500 | $122,850 | $185,700 | 473% |
| $150,000 | $600 | $147,420 | $210,270 | 549% |

FORM's ICP — engineering/product/design organisations — skews toward $100k–$140k median salaries, placing realistic ROI in the **400–500%** band. Lead with $75k in initial presentations (conservative); update to ICP-specific salary band after discovery questionnaire captures workforce composition data.

#### 20.5.3 Absenteeism reduction rate sensitivity

| Reduction rate | Attribution basis | Absenteeism value | 1-year total | 1-year ROI |
|---|---|---|---|---|
| 5% | Extreme conservative (1/6 RAND) | $26,325 | $89,175 | 175% |
| 10% | Conservative (1/3 RAND) | $52,650 | $115,500 | 257% |
| **14%** | **Base (50% RAND)** | **$73,710** | **$135,360** | **318%** |
| 20% | Moderate (70% RAND) | $105,300 | $167,250 | 416% |
| 28% | RAND upper bound | $147,420 | $209,370 | 546% |

Conclusion: even at the **extreme conservative floor (5% reduction)**, the model delivers **175% ROI** and a **4.1-month payback**. The business case is robust across the full plausible range of absenteeism attribution.

---

### 20.6 Non-Financial Value Drivers

Real but hard to quantify precisely. Include as supporting narrative — do not roll into the ROI calculation.

**ESG and sustainability reporting**

Employee wellness programmes are cited in ESG disclosures under GRI 401-2 (benefits provided to full-time employees) and, for healthcare-adjacent employers, SASB HC-DY-330a.2. The EU Corporate Sustainability Reporting Directive (CSRD) — mandatory for companies ≥ 250 employees reporting from FY 2025 — requires disclosure of employee health and safety policies and outcomes. FORM's admin dashboard (aggregate engagement, activation rate, opt-in NPS) provides audit-ready evidence for ESG reporting at no additional cost beyond the core platform.

**Talent acquisition**

LinkedIn Talent Insights (2024): 73% of candidates rank employee wellbeing programmes as a top-5 factor in job selection. Glassdoor (2023 employer brand study): reviews citing inadequate wellness benefits correlate with −0.3 points on overall company rating. For technology and engineering employers competing for talent at $150k+ packages, a differentiated wellness programme is a meaningful signal — especially one that respects privacy (a concern among younger knowledge workers who are aware of employer surveillance risks).

**Compliance risk avoidance**

EEOC v. Honeywell Inc. (2014) and AARP v. EEOC (2017) established that incentive-based wellness programmes face material EEOC, HIPAA, and ADA challenge risk when they tie financial incentives to health outcomes or make participation feel coercive. FORM's privacy floor (HR never sees individual data; enforced at the data layer, not by policy) and voluntary participation model eliminate this legal exposure entirely. Frame to buyer's legal team: FORM's architecture makes coercive use structurally impossible — the risk that exists with programme-specific biometric screenings or outcomes-linked incentives does not apply. **Do not assign a dollar value to legal risk avoidance — flag to buyer's legal team and let them calculate.**

**Culture and belonging signal**

A wellness programme that treats employees as autonomous adults — rather than surveilled participants — sends a culture signal that matters to employee trust. Correlates tracked in QBRs: eNPS trend post-launch, Glassdoor review mentions of wellness, opt-in programme NPS trajectory. These are soft metrics; report them in aggregate QBR slides rather than financial summaries.

---

### 20.7 Business Case Template

A copy-pasteable narrative structure for the internal champion to adapt and present to their CFO + CHRO. Fill in [brackets] from the relevant row in §20.4.

---

**[DRAFT — INTERNAL USE ONLY — TO BE ADAPTED BY CHAMPION]**

**Subject: Business case for FORM enterprise wellness pilot — [Company name]**

**Executive summary**

We are evaluating FORM, an AI-powered fitness coaching platform, for a 90-day free pilot with [pilot N] employees, followed by an annual contract for [N] seats at $[tier price]/seat/month = **$[annual cost]/year**.

Our conservative financial model projects a **[ROI]% first-year return** and a **[X]-month payback period**. The 90-day pilot at no cost validates this before any budget is committed.

**The gap in our current programme**

We spend approximately $[current wellness budget] on fitness and wellness benefits annually. Utilisation is roughly [current utilisation %] — meaning we pay $[effective cost] per *active* participant. We have no systematic way to measure programme impact.

**What FORM changes**

| Dimension | Today | With FORM |
|---|---|---|
| Active participants | ~[20%] of enrolled | ≥55% targeted (pilot success criterion) |
| Measurability | Anecdotal | Aggregate engagement dashboard (no individual data) |
| Privacy risk | Admin sees participation | HR never sees individual data — enforced architecturally |
| Compliance posture | Standard | EEOC/HIPAA/ADA exposure eliminated by design |

**Financial case (conservative inputs — published research discounted 50%+)**

| Value driver | Annual value | Source / basis |
|---|---|---|
| Absenteeism reduction | $[value] | RAND meta-analysis (2013), 50% discount applied |
| Productivity uplift | $[value] | Pronk et al. (2004), 0.5% applied (vs. published 11%) |
| Turnover avoidance | $[value] | SHRM 2022 replacement cost, 1 event/year |
| **Total annual value** | **$[total]** | |
| **FORM annual cost** | **($[cost])** | |
| **Net annual benefit** | **$[net]** | |
| **1-year ROI** | **[ROI]%** | |
| **Payback period** | **[X] months** | |

**Pilot success criteria (agreed upfront)**

- ≥ 55% of invited employees complete onboarding by Day 30
- ≥ 45% weekly active rate by Day 60
- Opt-in NPS ≥ 40 by Day 90
- Zero compliance incidents (no individual data surfaced to HR dashboard)

If these criteria are met → proceed to annual contract. If not → no cost, no obligation.

**Risk profile**

| Risk | Mitigation |
|---|---|
| Financial risk | 90-day pilot is free — zero spend until criteria are met |
| Data security risk | SOC 2 Type II audit in progress; GDPR DPA signed before any data flows |
| Privacy/EEOC risk | Individual health data never reaches HR — enforced at the data layer, not by policy |
| Employee trust risk | Employees can revoke company association at any time |

**Recommendation:** Approve the 90-day free pilot. Allocate $[annual cost] in the FY[year] benefits budget, contingent on pilot success criteria.

---

**Notes for the FORM sales champion (do not share with buyer):**

- Fill all [brackets] from the matching row in §20.4 for the prospect's tier and headcount
- Use the **55% conservative activation rate** for pilot success criteria — lower bar than the 65% base model, gives the pilot room to succeed
- Lead with **absenteeism data** (Lever 2) as the primary ROI argument — it is auditable via sick-day records and credible to CFOs
- Present productivity uplift (Lever 3) as a **supporting figure only**, disclosing the conservative basis: "we used 1/22nd of the published effect size"
- Do not combine all levers into a single headline number in the first discovery call — introduce them sequentially as the buyer pushes for more
- If the prospect asks for case study data: we do not fabricate case studies — offer to run the pilot and make this organisation *the* reference case
- Offer a live ROI calculator session in the discovery call (pre-populated spreadsheet from customer-success) — buyers who build numbers themselves trust them more

---

### 20.8 Open Questions Added

**OQ-18: First pilot activation data to calibrate §20.4 estimates**

The 65% activation rate is drawn from SHRM 2023 Wellness Programme Survey benchmarks for software-based programmes with structured onboarding. FORM's activation rate may exceed this (superior product, AI personalisation, CV feedback novelty) or fall below (behaviour change is difficult; corporate rollout competing with daily calendar load). Instrument the first pilot rigorously:

- Day 7 login rate (early signal)
- Day 30 first-session completion rate (onboarding activation)
- Day 60 weekly active rate (sustained engagement signal)
- Day 90 opt-in NPS (satisfaction signal)

Replace the 65% base model estimate after three pilot cohorts complete. Until then, use the 55% conservative threshold for all prospect-facing ROI presentations. Assigned to: customer-success (instrumentation request to data-engineer) before first pilot kickoff.

**OQ-19: ICP salary band for absenteeism value calibration**

The base model uses $75,000 average salary (US private-sector knowledge-worker median, BLS 2023). FORM's ICP — engineering/product/design organisations — skews toward $110–140k median total compensation. If CRM data confirms ICP-average salary at $110k, the Growth 300-seat 1-year ROI increases from **318% → ~430%** (Growth 300, $440/day absenteeism rate). Sales champion to add "average salary band of target employee population" as a mandatory field in the discovery questionnaire from first outreach call. customer-success to update §20.4 with ICP-calibrated tables once 5 deal records exist with salary band data.

---

---

## 21. Pilot Economics, Conversion Governance & Discount Authority Matrix

> Closes OQ-15 (ASC 606 variable consideration for pilot discounts) and OQ-18 (pilot activation data calibration). Defines the commercial framework for converting free pilots to paid contracts.

---

### 21.1 Purpose

This section establishes the complete commercial framework governing how FORM runs pilots, converts them to paid contracts, and manages discount authority. It closes two open questions from prior sections:

- **OQ-15** (from §18.8): ASC 606 variable consideration treatment for conditional pilot discounts — resolved in §21.3.
- **OQ-18** (from §20.8): First pilot activation data to calibrate the §20.4 ROI model — resolved by the KPI instrumentation plan in §21.7 and the OQ-20 calibration trigger.

It also provides the discount authority matrix required for sales governance as the team scales from founder-led to AE-led selling (§19.3–19.4).

---

### 21.2 Pilot Structures

Three pilot types are offered. Selection is determined by account size and procurement constraints.

#### Free Pilot (Standard)

- **Duration:** 90 days
- **Seat limit:** ≤ 250 seats
- **Payment required:** None. No credit card on file during pilot period.
- **Features:** All features unlocked — same product experience as paid Growth tier.
- **Revenue recognition:** Zero. No performance obligation exists during pilot. First-recognition date is the execution date of the paid annual contract (§18.4 methodology applies).
- **Eligibility:** Default pilot type for all inbound and outbound prospects. No founder approval required.

#### Extended Free Pilot

- **Duration:** 180 days
- **Seat limit:** No hard cap, but requires written founder approval.
- **Approval criteria:** Strategic account size — minimum ≥ 500 seats at conversion. Extended timeline must be justified by procurement cycle length (e.g., large-enterprise budget calendar constraints, legal review backlog) or documented strategic relationship value.
- **Revenue recognition:** Same as Free Pilot — zero revenue until paid contract executes.
- **Risk note:** Extended pilots carry higher time-cost per deal. Founder approval gate exists to prevent overuse as a discount substitute.

#### Paid Pilot

- **Duration:** 90 days
- **Pricing:** $1/seat/month — flat rate regardless of tier.
- **Onboarding fee:** Waived.
- **Credit on conversion:** Full paid-pilot amount ($1/seat/month × seats × 3 months) applied as a credit against Year 1 contract value if the account converts within 30 days of pilot end.
- **Revenue recognition:** Recognized ratably over the 90-day pilot period (§21.3 below).
- **Eligibility:** Designed for accounts ≥ 500 seats where procurement policy requires a vendor payment before multi-year commitment. Also suitable where the buyer's procurement team requires an established billing relationship prior to budget approval.

---

### 21.3 ASC 606 Variable Consideration Treatment (Closes OQ-15)

Each pilot type creates a distinct revenue recognition scenario under ASC 606 / IFRS 15.

#### Free Pilot (Standard and Extended)

No contract exists during the pilot period. FORM has no performance obligation — the pilot is a pre-contract evaluation. Revenue: $0. Contract inception date for ASC 606 purposes is the execution date of the signed paid annual agreement. This aligns with the first-recognition policy documented in §18.4.

#### Paid Pilot — Credit on Conversion

The $1/seat/month Paid Pilot creates a variable consideration question: if the account converts, the pilot fees become a credit against Year 1 ACV. Under **ASC 606-10-32-11**, variable consideration must be constrained to the extent it is probable that a significant revenue reversal will not occur.

Treatment:

1. During the 90-day pilot: recognize $1/seat/month ratably. Record a corresponding **contingent credit liability** (deferred revenue offset) equal to the total pilot fees collected, classified as short-term deferred revenue on the balance sheet.
2. At conversion (paid contract executed within 30 days of pilot end): release the deferred revenue offset against the Year 1 contract ACV. The net recognized amount equals full Year 1 ACV minus the credit applied.
3. At pilot expiry without conversion: the contingent credit liability lapses. Release to revenue at the period of expiry — the pilot fees earned ratably become fully recognized revenue with no further obligation.
4. Most likely amount estimate during pilot: treat expected revenue as **$0 net** (recognize pilot fees, offset by full credit liability) until the convert/not-convert decision point is reached. This is the conservative position consistent with ASC 606-10-32-11 constraint principles.

#### Extended Pilot with Price Concession at Conversion

If a price concession greater than **20% off list price** is offered at conversion following an extended pilot, the full contract is recognized at the discounted ACV. No step-up accounting applies unless the contract contains an explicit multi-element structure with separately stated price increases in future periods (which FORM does not currently offer). The concession is treated as a reduction to transaction price, not a separate performance obligation.

#### Audit Trail Requirement

Add the following event types to **AUDIT_LOG_SCHEMA.md** (DEC-030):

| Event type | Trigger | Required fields |
|---|---|---|
| `pilot.started` | Pilot agreement signed or pilot environment provisioned | `tenant_id`, `pilot_type`, `start_date`, `seat_count`, `CSM_owner` |
| `pilot.converted` | Paid contract executed within 30 days of pilot end | `tenant_id`, `contract_value_ACV`, `seats`, `discount_pct`, `approver` |
| `pilot.expired` | 30-day post-pilot conversion window closes without contract | `tenant_id`, `pilot_type`, `reason_code` |

---

### 21.4 Pilot Conversion Rate Targets

All figures below are [ESTIMATE] — pre-launch planning inputs. No live pilot data exists yet.

| Pilot type | Conversion target | Benchmark basis | Conversion defined as |
|---|---|---|---|
| Free Pilot (90d) | 35% | Slack-comparable bottom-up SaaS free-to-paid: 30–40% | Signed annual contract ≥ 12 months executed within 30 days of pilot end |
| Extended Free Pilot (180d) | 55% | Filtered for strategic accounts with ≥ 500-seat commitment signal; higher close probability by selection | Same |
| Paid Pilot | 70% | Payment indicates procurement intent; low-friction signal of budget availability | Same |

**OQ-20 (new):** Track actual pilot conversion rate from the first live pilot. Update the targets in this table after 10 pilot data points have been collected. Assigned to: customer-success (data-engineer to instrument `pilot.*` events; customer-success to report conversion rate in monthly metrics digest). Review checkpoint: first 10 pilots completed or 12 months post-launch, whichever comes first.

---

### 21.5 Discount Authority Matrix

All discounts are measured against published list prices: Starter $12/seat/month, Growth $16/seat/month, Enterprise custom (reference $14/seat/month for discount floor purposes).

#### Approval authority by role

| Approver | Discount limit | Scope | Required justification |
|---|---|---|---|
| **Founder** | Up to 40% from list price | Any tier, any contract term | Strategic, competitive, or relationship rationale; written note logged in CRM deal record before discount is communicated to prospect |
| **Future AE (Year 2+)** | Up to 20% from list price | Starter and Growth tiers only | Competitive displacement: must document competitor name, displacement reason, and confirm buyer has received competing proposal |
| **SDR / BDR** | 0% — not authorized to discount | — | All discounts require AE or founder approval before communication to prospect. SDRs must escalate any pricing discussion immediately |
| **CSM (expansion / renewal)** | Up to 10% on renewal | Same tier as current contract | Retention risk documented in CRM with churn risk score and intervention history |
| **Discount > 40%** | Requires board approval | Any | Pre-Series A: founder + lead investor sign-off in writing. Post-Series A: board approval per standard authority matrix |

#### Floor prices (hard minimums — no discount authority overrides these)

| Tier | List price | Floor price | Floor as % of list |
|---|---|---|---|
| Starter | $12/seat/month | $6/seat/month | 50% |
| Growth | $16/seat/month | $8/seat/month | 50% |
| Enterprise | Custom pricing | $7/seat/month | Not applicable to custom structures; floor applies to reference price |

Floor prices cannot be waived by any single approver. Pricing below floor requires a separate commercial exception process (founder + investor lead sign-off pre-Series A).

---

### 21.6 Contract Minimums and Commercial Floors

| Parameter | Floor | Rationale |
|---|---|---|
| Minimum contract value | $600/year | Starter 100 seats × 12 months × $6 floor price. Sub-$600 deals are unprofitable after CSM, billing, and support overhead |
| Minimum contract term | 12 months for annual pricing | Month-to-month is not offered in the Enterprise tier. Consumer Pro tier retains month-to-month optionality (separate from enterprise) |
| Minimum seat increment at expansion | 25 seats | Operational overhead of provisioning, billing, and CSM notification makes sub-25-seat expansions uneconomic |
| Minimum seat count at renewal to retain same unit price | Maintain ≥ original contracted seat count | Seat count reductions at renewal trigger a unit price renegotiation. No ratchet-down without price adjustment — protecting ACV floor |

---

### 21.7 Pilot KPIs for Conversion Prediction

The following KPIs are instrumented via the beta analytics pipeline and the admin dashboard. They serve as leading indicators of pilot conversion likelihood.

| KPI | Target | Priority | Measurement | Interpretation |
|---|---|---|---|---|
| Week-2 activation rate (seats with ≥ 1 session logged ÷ total pilot seats) | ≥ 50% | P0 — primary leading indicator | Computed daily from `session_completed` events joined to pilot seat roster | < 30% at Day 14 triggers CSM save protocol (see below) |
| Week-4 30-day retention (WAU ÷ total pilot seats) | ≥ 30% | P1 | Computed weekly from `app_opened` event cohort | Below target suggests low stickiness; review onboarding funnel |
| Admin dashboard logins in 90-day pilot | ≥ 3 logins by account admin | P1 — champion health signal | Tracked via `feature_viewed` events with `screen_name = admin_dashboard` | < 2 logins by Day 45 indicates low champion engagement; escalate to founder |
| Victor interaction positive-reaction rate (thumbs-up ÷ total rated interactions) | ≥ 60% | P1 | Tracked via explicit feedback events in chat flow | Below 60% triggers product review; may indicate coaching quality mismatch for ICP |

**CSM save protocol trigger:** If Week-2 activation rate is < 30% at Day 14, initiate the save protocol. Pilot is statistically less likely to convert. (See OQ-21 for full protocol definition.)

**OQ-21 (new):** Define the CSM save protocol — intervention steps, trigger thresholds, escalation path, and success criteria. Owner: customer-success. Target: protocol documented and approved before first live pilot kickoff. Review by Q3 2026.

---

### 21.8 Multi-Year Contract Economics

Integrates §16.4 (multi-year contract economics) and §19 (enterprise revenue forecast) data.

#### 2-year contract (10% annual discount)

A 10% annual discount on a Growth 300-seat deal ($32,400/year list) reduces ACV to $29,160/year. Over 2 years vs. two sequential 1-year renewals:

- ARR sacrifice: $3,240/year × 2 = $6,480 over the contract period
- Churn exposure eliminated: at 5% annual churn (§17.5), a 300-seat account has ~15% probability of churn over 2 years. Expected GP at risk: $32,400 × 2 × 60% GM × 15% = **$5,832**
- Net economics at 2 years: ARR sacrifice ($6,480) vs. churn GP preserved (~$5,832) — roughly break-even at the 300-seat Growth level. Minimum account size where 2-year discount is clearly net-positive: **≥ 480 seats** (where churn-exposure GP saved exceeds ARR sacrifice).

#### 3-year contract (20% annual discount)

A 20% annual discount reduces ACV to $25,920/year on the same Growth 300-seat deal. ARR sacrifice: $6,480/year × 3 = $19,440 over contract. Justifiable for Enterprise 1,000+ seat accounts where (a) churn risk is existential to the deal, (b) multi-year lock-in provides ARR visibility critical for Series A narrative, and (c) the ARR sacrifice is offset by account size.

**Implementation checklist item:** Add the multi-year discount schedule (10% for 2 years, 20% for 3 years) to the `pricing-enterprise.html` FAQ section. Owner: design-craft. Milestone: M3.

---

### 21.9 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Add `pilot.started`, `pilot.converted`, `pilot.expired` event types to AUDIT_LOG_SCHEMA.md | security-engineer | P0 | M3 |
| Update `pricing-enterprise.html` FAQ with pilot terms, discount floor note, and multi-year discount schedule | design-craft | P1 | M3 |
| Build `pilots` table in DATA_MODEL.md — track `pilot_id`, `tenant_id`, `pilot_type`, `start_date`, `end_date`, `converted_at`, `seats`, `CSM_owner` | platform-engineer | P1 | M4 |
| Instrument Week-2 activation alert in admin dashboard (trigger when activation rate < 30% at Day 14) | platform-engineer | P1 | M4 |
| Create CRM deal template with discount approval field and competitive displacement reason field | customer-success | P1 | M4 |
| Add pilot economics sensitivity to §10 sensitivity analysis table | data-engineer | P2 | M5 |

---

### 21.10 Open Questions

**OQ-20:** Track actual pilot conversion rate from the first live pilots. Update §21.4 conversion rate targets after 10 pilot data points have been collected. Assigned to: customer-success + data-engineer. Checkpoint: first 10 completed pilots or 12 months post-launch.

**OQ-21:** Define CSM save protocol — full intervention steps, trigger thresholds (Day-14 activation < 30%), escalation path to founder, and success criteria. Owner: customer-success. Target: Q3 2026, before first pilot kickoff.

**OQ-22:** Determine whether the Paid Pilot ($1/seat/month) is compliant with procurement thresholds in target EU enterprise buyers. Some EU public-sector and regulated-industry procurement frameworks require full tender processes above a minimum contract value threshold (e.g., € 1,000 in some jurisdictions). If Paid Pilot fees cross this threshold for large-seat pilots, the structure may trigger procurement requirements that the Free Pilot avoids. Owner: legal / founder. Checkpoint: before first Paid Pilot is offered to an EU-domiciled buyer.

---

**v1.0 · May 2026**

All figures marked [ESTIMATE] are pre-launch planning inputs. Replace with actuals as beta instrumentation delivers real usage data. The first reconciliation checkpoint is 30 days post-beta launch, targeting OQ-01 and OQ-02 as the highest priority gaps.

*v0.3 additions: §12 Free Tier Subsidy Model & Freemium Funnel Economics — total subsidy at scale, minimum conversion rate for cost neutrality, freemium CAC vs. paid UA comparison, free tier cost controls, free pool governance triggers.*

*v0.4 updates: §4, §5, §6, §8.4, §9, §10, §12 recalculated to reflect Pro ARPU of $19/month (Western markets, per pricing.html). Geo-pricing note added (§4). Free tier break-even threshold: 1.05% → 0.82%. Full-burn break-even: 6,530 → 5,112 Pro subscribers. Section 10 sensitivity margins updated throughout.*

*v0.8 updates: §8 (Enterprise Economics) and §14.6 (Enterprise LTV model) recalculated to reflect the current three-tier pricing from docs/ENTERPRISE.md and enterprise.html — Starter $12/seat (50–200 seats), Growth $9/seat (200–1,000 seats), Enterprise $6–8/seat (1,000+). Previous §8 used pre-launch placeholder pricing ($25–35/seat, 20/100/500-seat deal sizes) that was never aligned to the official rate card. §16 was already correct and is unchanged.*

*v0.5 additions: §13 Infrastructure Cost Breakdown by Service — service-by-service table at 1k/10k/50k/100k MAU, fixed vs. variable classification, upgrade inflection points, cost cliff analysis, per-service optimization levers. §14 Cohort LTV Model & CAC Payback Period — 24-month cohort survival tables for Pro monthly and annual plans, CAC payback at $30/$50/$80/$120, LTV:CAC targets by channel, enterprise LTV at 120% NRR, cohort survival curve requirements, data pipeline specification.*

*v0.6 additions: §15 Enterprise Infrastructure COGS Deep-Dive — per-feature marginal cost quantification for SSO/SAML overhead, SCIM provisioning, admin dashboard queries, 7-year audit log storage (hot + cold tier), SIEM webhook delivery, white-label DNS/SSL, and multi-tenant RLS; consolidated summary showing enterprise features add < $0.00001/seat/month (< 0.003% of baseline COGS). §16 Enterprise Tier Break-Even by Plan — plan-level break-even table (Starter/Growth/Enterprise × seat count), minimum viable seat floor analysis (50-seat Starter Y1 GM = 47.9% vs 20-seat = −26.1%), Y1 vs Y2+ margin improvement by plan, multi-year contract economics (25% 3-year discount gives up $13,800 GP vs eliminates ~$32,400 churn-exposure ARR). OQ-11 added: minimum seat threshold for acceptable Y1 Starter margin.*

*v0.7 additions: §17 Downside Scenario Analysis & Unit Economics Stress Tests — six adverse scenarios stress-tested against v0.6 baseline (S1: Anthropic API doubles, break-even +1.3%; S2: App Store 30% commission, break-even +22.3%; S3: churn doubles 5%→10%, LTV −50%; S4: 20% price cut, break-even +26.0%; S5: enterprise Year 1 miss, not existential; S6: S1+S3 combined, freeze paid acquisition protocol). Survivability assessments, mitigation priority lists, and monitoring thresholds for each scenario. OQ-12 added: Anthropic volume pricing threshold (~25k Pro subs at baseline rates). OQ-13 added: actual Victor token budget per session — instrument in beta per OQ-01.*

*v0.9 additions: §18 Revenue Recognition, ARR/MRR Bridge & SaaS Metrics Taxonomy — ASC 606 / IFRS 15 ratable recognition rule for annual and multi-year contracts; billings vs. revenue vs. cash waterfall; monthly MRR bridge template (New/Expansion/Contraction/Churned); free 90-day pilot timing and first-recognition date; deferred revenue schedule for 2-year and 3-year deals (short-term vs. long-term split); NRR / GRR model with target table (≥110% growth, ≥120% scale) and churn-rate sensitivity; investor-grade SaaS metrics taxonomy (ARR, MRR, ACV, TCV, Bookings, Billings, Revenue, NRR, GRR, CAC, LTV, Magic Number, Rule of 40) with FORM-specific notes; reporting cadence table. OQ-14 added: revenue recognition policy for mid-contract seat additions. OQ-15 added: ASC 606 variable consideration treatment for conditional pilot discounts.*

*v1.1 additions: §20 Customer ROI Model & Business Case Template — buyer-facing financial model covering status quo cost baseline (Willis Towers Watson / Mercer benchmarks), five quantifiable value levers (utilisation arbitrage, absenteeism reduction at 50% of RAND meta-analysis, productivity uplift at 0.5% of Pronk published effect, turnover avoidance, direct cost comparison), three-year ROI tables by tier (Starter 100-seat: 255% / Growth 300-seat: 318% / Enterprise 1,000-seat: 434% with 2.3–3.4-month payback), sensitivity analysis on activation rate / salary band / absenteeism reduction rate (floor case 40% activation + 5% reduction still yields 161–175% ROI), non-financial drivers (ESG/CSRD, talent acquisition, EEOC/HIPAA/ADA risk avoidance, culture), copy-paste business case template for internal champion, and sales champion usage notes. OQ-18 added: first pilot activation data to calibrate model. OQ-19 added: ICP salary band discovery to update absenteeism value estimate.*

*v1.0 additions: §19 Enterprise Go-to-Market Financial Model — sales velocity equation (opportunities × win rate × ACV ÷ sales cycle) for Year 1 founder-led ($6k/month) and Year 2 AE-led ($46k/month) phases; pipeline stage conversion model with hard exit gates and BANT disqualification triggers; founder time-budget per deal (37 hours total) and AE hire trigger criteria; first AE hire economics ($120k OTE, 3.8× quota multiple, $179k net GP/year at quota, Month-5 ramp-cost recovery); SDR layer economics ($70k OTE, 2.1× ROI, SQL qualification criteria, 40 SQL/year quota); CSM cost absorption bridge (dedicated CSM < 5% of revenue at Growth 700+ seats and Enterprise 1,500+ seats; pooled CSM model for sub-300-seat accounts); three-scenario Month 1–24 enterprise ARR forecast (Base: $188k ARR Month 24; Bull: $312k; Bear: $86k) with month-by-month recognized ARR; Series A ARR quality framing (NRR ≥ 110%, LTV:CAC ≥ 5×, ≥ 3 active accounts). OQ-16 added: Growth-tier minimum seat floor for CSM absorption. OQ-17 added: win rate calibration sensitivity (15–35%) on SDR hire timing and pipeline coverage targets.*

*v1.2 additions: §21 Pilot Economics, Conversion Governance & Discount Authority Matrix — three pilot structures (Free 90d, Extended 180d, Paid $1/seat/month); ASC 606 variable consideration treatment for each (closes OQ-15); pilot conversion rate targets (35%/55%/70% by type); discount authority matrix (founder/AE/CSM authority levels with floor prices); contract minimums ($600/year floor, 25-seat increment); pilot KPIs for conversion prediction (Day-14 activation rate as P0 leading indicator); multi-year contract economics integrated from §16.4; six-item implementation checklist; OQ-20 (actual conversion rate calibration), OQ-21 (CSM save protocol), OQ-22 (EU procurement threshold compatibility for Paid Pilot).*

---

## 22. Year 1 Cash Flow Forecast & Runway Model

> Owner: data-engineer + founder. Audience: founder, lead investor, future CFO. Review cadence: weekly cash position check; full model refresh monthly pre-Series A. All figures marked [ESTIMATE] are pre-launch planning inputs — not commitments.

---

### 22.1 Purpose

A P&L shows whether the business is profitable. A cash flow forecast shows whether the business is alive. Pre-revenue, the two diverge materially: accrual accounting records expenses when incurred regardless of payment timing, while cash accounting records when money enters or leaves the bank account. For a pre-seed company on personal savings with no credit facility, the cash balance on a given day is the only number that determines operational continuity.

This section models cash inflows and outflows month-by-month for Months 1–18, providing three things a P&L cannot:

1. **Runway visibility** — how many months of operations remain at current burn, under each scenario
2. **Trigger clarity** — which events materially accelerate or decelerate the cash trajectory, so they can be managed proactively
3. **Fundraise timing** — when to initiate the Series A process based on runway position, not on panic

Key distinctions from other financial views in this document:

| View | Question answered | Timing basis | Primary audience |
|---|---|---|---|
| P&L (§3–§10) | Are we profitable per unit? | Accrual | Investors (post-revenue) |
| ARR forecast (§19.7) | How much recurring revenue are we building? | Recognized revenue | Investors, board |
| Billings vs. revenue (§18.2) | When does cash from contracts arrive? | Cash receipt | Finance, investor diligence |
| **Cash flow forecast (this section)** | **Will we run out of money?** | **Actual bank cash** | **Founder, lead investor** |

The cash flow model is a living document. It must be updated every time a material assumption changes — a delayed hire, an earlier-than-expected first contract, or a change in the Anthropic API rate. It is not a forecast to present; it is a tool to navigate.

---

### 22.2 Funding Assumptions

#### 22.2.1 Pre-seed funding input

| Parameter | Value | Notes |
|---|---|---|
| Pre-seed capital raised | [FOUNDER_INPUT] | Total cash available at Month 0, including any personal savings committed to the company |
| Cash at Month 0 | [FOUNDER_INPUT] | Assume equal to pre-seed capital raised unless a portion is held in reserve by investor instrument terms |
| Funding instrument | [FOUNDER_INPUT] | SAFE / convertible note / priced round — affects dilution model but not cash-flow mechanics |
| Wire receipt date | [FOUNDER_INPUT] | The date the capital is in the operating bank account; cash flow starts from this date |

Until [FOUNDER_INPUT] values are confirmed, the runway model in §22.4 is presented as a sensitivity matrix across a range of pre-seed amounts rather than a single-path forecast.

#### 22.2.2 Burn rate categories

Cash out is organized into five categories. Each maps to a cash-flow table row in §22.3.

| Category | What it covers | Fixed or variable? | Primary driver |
|---|---|---|---|
| **Infra** | Anthropic API, ElevenLabs, Supabase, Cloudflare Workers, Sentry, PostHog, Redis, Expo/EAS | Hybrid — fixed base + variable per MAU | MAU × per-user cost (per §13.1 scaling tables) |
| **AI API costs** | Anthropic Claude API specifically, isolated from above for sensitivity tracking | Strictly variable | Pro MAU × sessions × tokens (per §3.1) |
| **Marketing / UA** | Paid acquisition (App Store ads, social), content production, SEO tooling | Discretionary variable | Founder decision per growth phase |
| **Legal / compliance** | Entity formation, IP filings, privacy counsel (GDPR/HIPAA), SOC 2 readiness, contract review | Lumpy fixed | Milestone-driven: launch, first enterprise deal, audit |
| **Hiring** | Founding Engineer cash compensation (salary only; equity excluded from cash model) | Fixed once hired | Hire date + negotiated salary [FOUNDER_INPUT] |

**Note on founder compensation:** The model assumes $0 founder salary during Months 1–12 [ESTIMATE]. If the founder draws a salary, add it to the Hiring row from the month payment begins. This is [FOUNDER_INPUT].

**Note on non-cash items excluded from this cash model:** equity-based compensation (stock options, SAFE conversions), depreciation, and deferred revenue adjustments. The cash flow model uses receipts and disbursements only.

---

### 22.3 Month-by-Month Cash Flow Table (Months 1–18)

#### 22.3.1 Model construction notes

**Revenue cash timing:** Consumer Pro subscriptions are charged monthly at billing date (MRR = cash-in in the same month, net of App Store payout lag of ~45 days [ESTIMATE]). Annual enterprise contracts: cash received upfront at contract signing; recognized as revenue ratably per §18.1, but cash-in occurs in the month the invoice is paid. For cash flow purposes, annual contract cash lands in the month of invoice settlement — typically Month 1 of the contract term, with 30-day net payment terms adding one month lag [ESTIMATE].

**Infra costs per §13.1:** At the MAU levels modeled in Months 1–18, total infrastructure cost stays within the sub-1k MAU band ($436/month) through approximately Month 6, then scales toward the 1k–10k MAU band ($436–$3,197/month range) as the beta cohort grows. The AI API (Anthropic) line is separated and grows proportionally to Pro MAU per §3.1 ($0.20/Pro user/month).

**Base scenario revenue ramp:** Consumer Pro MRR grows from $0 at Month 0 to an estimated $2,000–$4,000/month by Month 12 [ESTIMATE], driven by closed beta invite → App Store launch → organic growth. Enterprise revenue per §19.7 Base scenario: first contract cash receipt Month 8 ($14,400 upfront annual payment [ESTIMATE]), second contract cash receipt Month 11 ($10,800 [ESTIMATE]).

**Three scenarios defined:**

| Scenario | Consumer Pro ramp | Enterprise deals (Year 1) | Key assumption |
|---|---|---|---|
| **Base** | Moderate: 50 Pro subs Month 6 → 200 Month 12 [ESTIMATE] | 2 deals (per §19.7 Base) | Aligned to §19 Base; App Store launch Month 4 |
| **Bull** | Faster: 100 Pro subs Month 6 → 400 Month 12 [ESTIMATE] | 3 deals (per §19.7 Bull) | Viral coefficient > 1 in beta; first enterprise deal Month 7 |
| **Bear** | Slow: 20 Pro subs Month 6 → 80 Month 12 [ESTIMATE] | 1 deal (per §19.7 Bear) | App Store review delay; low beta-to-paid conversion |

#### 22.3.2 Base Scenario Cash Flow (Months 1–18)

All figures [ESTIMATE]. Positive = cash in. Negative = cash out. "Cumulative Cash" assumes [FOUNDER_INPUT] initial balance; shown here as delta from Month 0 for scenario comparability.

| Month | Consumer MRR cash | Enterprise contract cash | Total Cash In | Infra (excl. AI API) | AI API cost | Marketing / UA | Legal / compliance | Hiring | Total Cash Out | Net Cash | Cumulative Delta |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | $0 | $0 | **$0** | $(440) | $(10) | $(500) | $(2,000) | $0 | **$(2,950)** | $(2,950) | $(2,950) |
| 2 | $0 | $0 | **$0** | $(440) | $(20) | $(500) | $(500) | $0 | **$(1,460)** | $(1,460) | $(4,410) |
| 3 | $0 | $0 | **$0** | $(440) | $(40) | $(1,000) | $(500) | $0 | **$(1,980)** | $(1,980) | $(6,390) |
| 4 | $500 | $0 | **$500** | $(450) | $(80) | $(1,500) | $(1,500) | $0 | **$(3,530)** | $(3,030) | $(9,420) |
| 5 | $950 | $0 | **$950** | $(460) | $(150) | $(1,500) | $(500) | $0 | **$(2,610)** | $(1,660) | $(11,080) |
| 6 | $950 | $0 | **$950** | $(470) | $(220) | $(1,500) | $(500) | $0 | **$(2,690)** | $(1,740) | $(12,820) |
| 7 | $1,425 | $0 | **$1,425** | $(490) | $(300) | $(1,500) | $(500) | $0 | **$(2,790)** | $(1,365) | $(14,185) |
| 8 | $1,900 | $14,400 | **$16,300** | $(530) | $(380) | $(1,500) | $(3,000) | $0 | **$(5,410)** | $10,890 | $(3,295) |
| 9 | $1,900 | $0 | **$1,900** | $(560) | $(380) | $(1,500) | $(500) | $0 | **$(2,940)** | $(1,040) | $(4,335) |
| 10 | $2,375 | $0 | **$2,375** | $(600) | $(460) | $(2,000) | $(500) | $0 | **$(3,560)** | $(1,185) | $(5,520) |
| 11 | $2,850 | $10,800 | **$13,650** | $(650) | $(550) | $(2,000) | $(2,000) | $0 | **$(5,200)** | $8,450 | $2,930 |
| 12 | $3,325 | $0 | **$3,325** | $(700) | $(640) | $(2,000) | $(500) | $0 | **$(3,840)** | $(515) | $2,415 |
| 13 | $3,800 | $0 | **$3,800** | $(760) | $(730) | $(2,000) | $(500) | $(8,333) | **$(12,323)** | $(8,523) | $(6,108) |
| 14 | $3,800 | $12,960 | **$16,760** | $(820) | $(730) | $(2,000) | $(2,500) | $(8,333) | **$(14,383)** | $2,377 | $(3,731) |
| 15 | $4,275 | $0 | **$4,275** | $(880) | $(820) | $(2,000) | $(500) | $(8,333) | **$(12,533)** | $(8,258) | $(11,989) |
| 16 | $4,275 | $0 | **$4,275** | $(950) | $(820) | $(2,000) | $(500) | $(8,333) | **$(12,603)** | $(8,328) | $(20,317) |
| 17 | $4,750 | $0 | **$4,750** | $(1,020) | $(910) | $(2,000) | $(500) | $(8,333) | **$(12,763)** | $(8,013) | $(28,330) |
| 18 | $5,225 | $18,000 | **$23,225** | $(1,100) | $(1,000) | $(2,000) | $(500) | $(8,333) | **$(12,933)** | $10,292 | $(18,038) |

**Key observations — Base scenario:**
- Month 8 and Month 11 enterprise cash receipts provide material cushion against consumer-only burn rate. Without enterprise cash, cumulative delta at Month 11 would be approximately $(24,000) deeper [ESTIMATE].
- Founding Engineer hire at Month 13 ($100,000 annual salary → $8,333/month) is the single largest permanent step-change in monthly burn. This timing should be treated as a cash-sensitive milestone, not a fixed calendar date (see §22.5).
- By Month 18, cumulative cash consumed from initial balance is approximately $(18,000) [ESTIMATE], meaning the business has consumed pre-seed capital net of enterprise cash receipts. Actual ending balance = [FOUNDER_INPUT] − $18,038. This must remain well above zero; see §22.4 for the safe-harbor trigger.

#### 22.3.3 Bull Scenario Cash Flow — Summary (Months 1–18)

Full month-by-month detail omitted for brevity; key variance lines vs. Base [ESTIMATE]:

| Parameter | Base | Bull | Delta |
|---|---|---|---|
| Month 12 consumer MRR | $3,325 | $6,650 | +$3,325/month |
| Enterprise cash receipts (Year 1) | $25,200 | $40,200 | +$15,000 |
| Month 12 AI API cost | $(640) | $(1,280) | $(640)/month |
| Month 12 marketing UA | $(2,000) | $(3,000) | $(1,000)/month |
| Cumulative cash delta at Month 18 | $(18,038) | $(4,500) [ESTIMATE] | +$13,538 |
| Months of additional runway vs. Base | — | +2–3 months [ESTIMATE] | — |

Bull scenario implication: Series A process can begin from a position of comfort rather than urgency. Founding Engineer hire can be accelerated to Month 10–11 without materially compressing runway, enabling faster product velocity.

#### 22.3.4 Bear Scenario Cash Flow — Summary (Months 1–18)

| Parameter | Base | Bear | Delta |
|---|---|---|---|
| Month 12 consumer MRR | $3,325 | $1,330 | $(1,995)/month |
| Enterprise cash receipts (Year 1) | $25,200 | $14,400 | $(10,800) |
| Founding Engineer hire | Month 13 | Deferred to Month 16+ | Saves $25,000 in cumulative cash |
| Cumulative cash delta at Month 18 | $(18,038) | $(38,000) [ESTIMATE] | $(19,962) |
| Months of additional burn vs. Base | — | +3–4 months of higher need [ESTIMATE] | — |

Bear scenario implication: founding engineer hire must be deferred. Marketing/UA spend must be cut to $500/month by Month 9. Series A process starts earlier on weaker metrics — the worst-case fundraising position. The bear scenario does not imply business failure, but it does require a deliberate decision by Month 6 to cut discretionary spend if the early signals (beta activation rate, first pilot progress) are tracking below plan.

**Bear scenario decision gate:** If consumer Pro subscribers at Month 6 are below 30 (i.e., below the Bear scenario's own 20-sub estimate by ≥33%), treat it as a signal to invoke the S6 combined stress protocol from §17.8 — freeze paid acquisition, defer any hiring, and initiate a pre-seed extension conversation.

---

### 22.4 Runway Sensitivity Matrix

The table below shows months of runway as a function of initial cash balance and average monthly net burn. "Net burn" is Total Cash Out minus Total Cash In per month, averaged across the period.

Average monthly net burn ranges are drawn from the Base scenario:
- **Early burn (Months 1–6):** approximately $2,000–$3,500/month net [ESTIMATE] (no revenue, legal costs front-loaded)
- **Mid burn (Months 7–12):** approximately $1,000–$2,000/month net [ESTIMATE] (consumer revenue partially offsets; enterprise cash receipts are lumpy)
- **Post-hire burn (Months 13–18):** approximately $7,500–$9,000/month net [ESTIMATE] (Founding Engineer salary dominates)

#### Runway table — months of cash remaining

| Initial Cash | $4,000/mo avg burn | $6,000/mo avg burn | $8,000/mo avg burn | $10,000/mo avg burn | $12,000/mo avg burn |
|---|---|---|---|---|---|
| $100,000 | 25 months | 17 months | 13 months | 10 months | 8 months |
| $150,000 | 38 months | 25 months | 19 months | 15 months | 13 months |
| $200,000 | 50 months | 33 months | 25 months | 20 months | 17 months |
| $250,000 | 63 months | 42 months | 31 months | 25 months | 21 months |
| $300,000 | 75 months | 50 months | 38 months | 30 months | 25 months |
| $400,000 | 100 months | 67 months | 50 months | 40 months | 33 months |

All figures [ESTIMATE]. Burn rate is the average across the modeled period — not the peak monthly burn. Post-hire burn (Months 13+) at Base scenario is $8,000–$9,000/month net; early-phase burn is lower. Use weighted average burn for the period in question when reading this table.

**Safe-harbor rule:** When projected cash remaining falls below **6 months of current burn rate**, the Series A fundraising process must be initiated immediately. Six months is the minimum required lead time: 2–3 months for process (deck, data room, initial meetings) + 1–2 months for term sheet negotiation + 1 month for close and wire. Starting a raise with fewer than 6 months of runway places the founder in a structurally weak negotiating position and increases the probability of a distressed raise on unfavorable terms.

**Fundraising process start trigger:** (Current cash balance) ÷ (current monthly net burn) ≤ 6 months → initiate Series A process immediately. Do not wait for the next monthly review — treat this as a hard real-time threshold.

**Emergency reserve floor:** Maintain a minimum $15,000 [ESTIMATE] in the operating account as a non-deployable reserve. This covers one month of post-hire burn in a scenario where the Series A wire is delayed after a signed term sheet. Do not let operating cash fall below this floor under any scenario.

---

### 22.5 Key Cash Flow Triggers

These are the events that produce a step-change in the cash trajectory. Each has a directional impact (positive = more cash, negative = more burn) and a dependency that makes the timing uncertain.

| Trigger | Direction | Estimated cash impact | Timing dependency | Action if delayed |
|---|---|---|---|---|
| **App Store launch (consumer)** | Positive | Begins consumer MRR ramp; Base: +$500/month from Month 4 [ESTIMATE] | App Store review approval (typically 1–3 days, but rejections add 2–4 weeks) | Submit 4 weeks before planned launch; have a web-payment fallback for beta users |
| **First paid enterprise pilot conversion (Deal 1)** | Positive | $14,400 upfront cash receipt [ESTIMATE] per §19.7 Base; single-month spike | Pilot activation rate ≥ 60% at Day 60 per §21.7; legal turnaround time | If Day 60 activation is below threshold, invoke CSM save protocol (§21.7) immediately; do not wait for Day 90 |
| **Second enterprise deal close** | Positive | $10,800 upfront cash receipt [ESTIMATE] per §19.7 Base (75-seat Starter × $12 × 12 months) | Dependent on second pilot signed by Month 5; conversion per §21.4 timeline | Accelerate second pilot outreach to Month 2–3; pipeline coverage is the lead metric |
| **Founding Engineer hire** | Negative | +$8,333/month permanent burn increase (at $100,000 salary [ESTIMATE]) | Offer acceptance date; conditional on runway ≥ 12 months post-hire [ESTIMATE] | If runway < 12 months post-hire at time of offer, defer 60–90 days or restructure offer (lower base + higher equity) |
| **AE hire (Month 12, Base)** | Negative | +$5,000–$10,000/month incremental burn [ESTIMATE] during 5-month ramp-to-quota (§19.4) | Triggered by Deal 2 closing + AE trigger criteria per §19.3; Base scenario Month 12 | Defer if fewer than 2 enterprise deals closed — founder-led sales remains viable up to 3 deals per §19.3 |
| **Anthropic API cost shock (S1)** | Negative | +$0.20/Pro user/month incremental COGS per §17.3 | External event (rate change) | Activate model-tiering fallback (Haiku for classification, Sonnet only for active coaching turns) within 48 hours of rate notice |
| **Supabase Pro → Team upgrade (§13.4 Cliff 1)** | Negative | +$574/month step-up in infra spend | Triggered at ~12k–15k MAU; not a Year 1 concern in Base scenario | Pre-provision Team plan in Month 5 budget if beta cohort growth is outpacing forecast |
| **Series A wire received** | Positive | [FOUNDER_INPUT] raise amount — resets runway clock | Dependent on fundraise process start timing (§22.6) | See §22.4 safe-harbor trigger |

---

### 22.6 Fundraise Timing Model

Series A fundraising should be initiated when the business has sufficient evidence to command a strong valuation and sufficient runway to negotiate without duress. These two requirements point in opposite directions: evidence takes time to accumulate, but runway constrains how long you can wait. The model below identifies the window where both conditions are met simultaneously.

#### 22.6.1 Series A readiness criteria (integrating §19.7 GTM model)

These criteria are thresholds, not checkboxes — they inform the fundraise story, not a binary gate.

| Metric | Minimum threshold for credible Series A | Strong Series A | Source |
|---|---|---|---|
| Enterprise ARR | ≥ $50,000 [ESTIMATE] | ≥ $120,000 [ESTIMATE] | §19.7 Base scenario: $25,200 at Month 12; $88,500 at Month 18 |
| Enterprise logo count | ≥ 2 active, paid accounts | ≥ 4 active accounts | §19.7 Base: 2 deals by Month 12; 7 deals by Month 18 |
| NRR | ≥ 100% (no contraction) | ≥ 110% (expansion evident) | §18.6 target; first renewal data available Month 13+ |
| Consumer Pro subscribers | ≥ 500 [ESTIMATE] | ≥ 1,500 [ESTIMATE] | §22.3.2 Base: 200 at Month 12; 300 at Month 18 [ESTIMATE] |
| W-ACSU (NSM) | ≥ 3.0 sessions/week/active user | ≥ 3.5 sessions/week | Defined in METRICS.md; computed from `session_completed` events |
| D30 retention | ≥ 40% [ESTIMATE] | ≥ 55% [ESTIMATE] | Cohort retention curves per data-engineer pipeline |
| Pilot conversion rate | ≥ 30% (per §21.4 free pilot target) | ≥ 45% | First measurable after 10 completed pilots (OQ-20) |

**Observation:** Under the Base scenario, the minimum threshold for a credible Series A is not met until approximately Month 16–18, when enterprise ARR crosses $50k and logo count reaches 4+. This means the fundraise process should be initiated at Month 12–13 (four to six months before the anticipated close) — even though the story is not yet at full strength. Starting at Month 12 allows close at Month 17–18 on minimum-threshold metrics, which is better than starting at Month 16 and closing on no runway.

Under the Bull scenario, minimum thresholds are met by Month 12. Starting the fundraise at Month 10–11 enables a close at Month 14–15 from a position of relative strength.

Under the Bear scenario, minimum thresholds may not be met within 18 months. If base enterprise ARR is below $25k at Month 15, evaluate a pre-seed extension rather than a formal Series A process. A bridge round of $50,000–$100,000 [ESTIMATE] from existing investors or strategic angels buys 6–9 months of additional time at Bear burn rates.

#### 22.6.2 Fundraise process timeline

| Phase | Duration | Milestone |
|---|---|---|
| Preparation: deck, data room, metrics dashboard | 3–4 weeks | Data room complete; Metabase dashboards investor-ready |
| Warm introductions + first meetings | 3–5 weeks | ≥ 10 first meetings completed |
| Partner meetings + diligence | 4–6 weeks | 2–3 firms in active diligence |
| Term sheet negotiation | 2–3 weeks | Term sheet signed |
| Legal close + wire | 3–4 weeks | Cash in bank |
| **Total elapsed time** | **15–22 weeks (4–6 months)** | — |

**Earliest recommended fundraise start date (Base scenario):** Month 12–13, targeting close at Month 17–18 with 3–4 months of runway remaining at close.

**Safe-harbor override:** If the §22.4 safe-harbor trigger fires before Month 12 (cash balance ÷ burn ≤ 6 months), begin fundraise preparation immediately, regardless of milestone achievement. Revenue evidence is preferred but runway pressure is existential.

---

### 22.7 Cash Management Controls

These are operational policies, not strategic decisions. They exist to prevent avoidable errors — missed payments, unmonitored commitments, or a cash position surprise during a board meeting.

#### 22.7.1 Banking

| Policy | Requirement |
|---|---|
| Operating account | Single business checking account for all inflows and outflows [FOUNDER_INPUT: institution] |
| Reserve account | Separate account for the $15,000 [ESTIMATE] emergency reserve floor; no debit card attached |
| Payroll | Separate payroll account once the Founding Engineer is hired; funded by transfer from operating account on salary date |
| Payment processor float | App Store payouts (Apple/Google) have 45-day lag [ESTIMATE]; model this as a 1.5-month MRR lag in cash receipts |
| Enterprise invoice collections | Net-30 payment terms; chase at Day 32 if unpaid; escalate to founder at Day 45; no new provisioning for accounts > 60 days overdue |

#### 22.7.2 Weekly cash position review

Every Monday (or the next business day), the founder reviews:

1. Current operating account balance
2. Outstanding invoices (enterprise accounts receivable)
3. Upcoming commitments in the next 30 days (payroll, annual subscriptions, legal retainer)
4. Current burn rate vs. prior week
5. Runway = (current balance − $15,000 reserve) ÷ (trailing 4-week average monthly burn)

This review takes 15 minutes with a maintained cash tracker. It is not optional — a founder who discovers a cash crisis in a board meeting rather than 6 weeks before it is a preventable failure of process.

**Tooling:** Maintain a shared Google Sheet (or Notion database) with the above five fields updated weekly. data-engineer to automate Supabase/Stripe revenue figures into the tracker once instrumentation is live; until then, manual entry.

#### 22.7.3 Commitment controls

| Control | Policy |
|---|---|
| Variable-rate commitments | No open-ended variable spend commitment > 3 months duration without founder approval (e.g., agency retainers, contractor agreements, SaaS annual plans with usage-based upside) |
| Annual SaaS renewals | All annual plan renewals > $500/year require explicit founder approval 30 days before renewal date. Maintain a renewal calendar in the cash tracker |
| Contractor and freelancer engagements | Maximum 90-day engagements with explicit renewal decision at each term end. No auto-renewal clauses in contractor agreements |
| Board approval threshold (post-seed) | Any single expenditure or commitment > [FOUNDER_INPUT] requires lead investor notification. Any expenditure > [FOUNDER_INPUT] requires board approval. Exact thresholds to be set in the SHA/investor rights agreement |
| Credit cards | No corporate credit card balances carried month-to-month. All card spend cleared at statement close. No personal guarantee on corporate debt instruments without founder explicit sign-off |

#### 22.7.4 Expense recognition lag

Vendor invoices for the prior month (Anthropic, ElevenLabs, Supabase) typically arrive in the first week of the following month. The cash tracker must account for this lag: record the expense in the month it was incurred, not the month the invoice arrives. This prevents a false "good month" in month N followed by a surprise double-expense in month N+1.

---

### 22.8 Open Questions

**OQ-23: Pre-seed capital amount and wire date**

The entire cash flow model in §22.3 is conditioned on [FOUNDER_INPUT] initial balance. Until this figure is confirmed, the month-by-month table should be read as a structure and scenario tool, not a literal forecast. Owner: founder. Resolution: before Month 1 of company operations. Priority: P0 — blocks all runway calculations.

**OQ-24: Founding Engineer hire cost and timing**

§22.3 uses $100,000 annual salary as the Founding Engineer cash compensation [ESTIMATE], triggering at Month 13 in the Base scenario. The actual offer and acceptance date may shift this by several months in either direction, and the salary range depends on the candidate and market. A $20,000 difference in annual salary moves monthly burn by $1,667 and modifies runway by 1–2 months at the $150k initial cash level. Owner: founder. Resolution: when the engineering hire decision is made; update §22.3 immediately on offer acceptance.

**OQ-25: App Store payout lag and payment processor terms**

The Base scenario models a 45-day [ESTIMATE] lag between consumer subscription revenue and cash receipt (Apple/Google payout terms). If the payout lag is shorter (Apple Connect typically pays monthly with 30-day settlement), the MRR-to-cash conversion is faster than modeled, improving early cash position by up to $1,500–$2,000 in cumulative improvement by Month 12 [ESTIMATE]. Owner: data-engineer (instrument actual payout dates via Stripe/RevenueCat reconciliation). Resolution: first full month of live App Store revenue.

**OQ-26: Legal and compliance cost profile**

§22.3 allocates $(2,000) in Month 1 for entity formation and initial IP counsel, plus lumpy $(2,000–$3,000) amounts at enterprise deal milestones for contract review and DPA execution. Actual legal costs depend on counsel hourly rate, deal complexity, and whether a pre-negotiated MSA template reduces per-deal review time (§21.3 approach). Highly variable: a contested enterprise DPA with a Fortune 500 legal team could cost $5,000–$15,000 in counsel fees alone. Owner: founder + legal counsel. Resolution: get a fixed-fee engagement letter from legal counsel before Month 1, covering at minimum entity formation and standard enterprise contract templates.

**OQ-27: Founder salary policy and personal runway**

§22.2 assumes $0 founder salary through Month 12. This assumption must be validated against the founder's personal financial position and the pre-seed funding amount. If the founder requires a market-rate or sustenance salary from Day 1, total monthly burn in §22.3 increases by $5,000–$10,000/month [ESTIMATE], reducing runway by 20–35% at the $150k initial cash level. This is not a judgment — it is a planning input that must be explicit. Owner: founder. Resolution: before company formation or within 30 days of first capital receipt.

---

*v1.3 additions: §22 Year 1 Cash Flow Forecast & Runway Model — cash flow vs. P&L distinction and purpose (§22.1); funding assumptions structure with [FOUNDER_INPUT] placeholders (§22.2); month-by-month 18-month cash flow table across Base/Bull/Bear scenarios with per-category rows (infra, AI API, marketing, legal, hiring) aligned to §13.1 scaling tables and §19.7 enterprise revenue forecast (§22.3); runway sensitivity matrix (funding amount × monthly burn → months of runway) with 6-month safe-harbor fundraise trigger (§22.4); key cash flow triggers table covering App Store launch, first enterprise deal, Founding Engineer hire, AE hire, API cost shock, and infrastructure cliffs (§22.5); Series A fundraise timing model with readiness criteria table (ARR, NRR, logo count, W-ACSU, D30 retention) and 15–22-week process timeline (§22.6); cash management controls covering single-account policy, weekly 5-point cash review, variable-rate commitment gate (90-day max without founder approval), and expense recognition lag (§22.7); OQ-23 (pre-seed capital amount), OQ-24 (Founding Engineer hire cost), OQ-25 (App Store payout lag), OQ-26 (legal cost profile), OQ-27 (founder salary policy) (§22.8).*

---

## 23. Enterprise NRR Engine & Expansion Revenue Model

> This section adds the financial machinery that sits above the static ARR forecasts in §19.7 and the NRR targets in §18.3 and §18.6. It does not restate those sections — it provides the formulaic bridge mechanics, tier-migration economics, add-on taxonomy, cohort NRR projections, price escalation policy, audit instrumentation, and the valuation translation layer that converts NRR into enterprise value multiples.

---

### 23.1 Monthly ARR Bridge Template

The ARR bridge is the single most important recurring financial report for an enterprise SaaS business. It answers the question: why did ARR change this month? Every movement in ARR falls into exactly one of six buckets. The template below is the canonical structure to be reproduced in the admin dashboard and investor reporting.

#### 23.1.1 Bridge formula

```
Closing ARR = Opening ARR
            + New Logo ARR
            + Seat Expansion ARR
            + Tier Upgrade ARR
            + Add-on ARR
            − Seat Contraction ARR
            − Logo Churn ARR
```

Gross expansion = New Logo ARR + Seat Expansion ARR + Tier Upgrade ARR + Add-on ARR
Gross contraction = Seat Contraction ARR + Logo Churn ARR
Net new ARR = Gross expansion − Gross contraction
NRR (trailing 12 months) = (Opening cohort ARR + expansion − contraction − churn) ÷ Opening cohort ARR

#### 23.1.2 Bridge row definitions

| Row | Definition | Source event | Unit |
|---|---|---|---|
| **Opening ARR** | Sum of all active annual contract values at start of period | Snapshot from `tenant_contracts` table | $ |
| **New Logo ARR** | ARR from tenants whose first paid invoice falls in the current period | `enterprise.contract_renewed` on first contract | $ |
| **Seat Expansion ARR** | Incremental ARR from seat count increases at existing tenants, within the same tier | `enterprise.seat_expanded` events | $ |
| **Tier Upgrade ARR** | Incremental ARR from a tenant moving to a higher price tier (Starter → Growth, Growth → Enterprise) | `enterprise.tier_upgraded` event | $ |
| **Add-on ARR** | ARR from add-on SKUs (white-label, premium SLA, custom integration) purchased by existing tenants | `enterprise.addon_purchased` event | $ |
| **Seat Contraction ARR** | ARR reduction from seat count decreases at existing tenants, within the same tier | `enterprise.seat_contracted` events | $ |
| **Logo Churn ARR** | Full ARR lost from tenants who do not renew or are terminated | Absence of `enterprise.contract_renewed` at contract end date | $ |
| **Closing ARR** | Computed output; must reconcile to sum of all active contract values at end of period | Reconcile against `tenant_contracts` daily batch | $ |

#### 23.1.3 Reporting cadence and owners

The bridge is computed monthly by data-engineer from the event stream and reconciled against Stripe invoice records and `tenant_contracts`. The founder reviews it in the first week of each month alongside the §22.7.2 cash position review. Discrepancies > $500 between the event-derived bridge and Stripe-derived ARR require same-day investigation — they typically indicate a provisioning event that was not matched to a contract amendment.

#### 23.1.4 Sample bridge (Month 12, Base scenario) [ESTIMATE]

Numbers derived from §19.7 Base forecast. Presented as a structural example, not a prediction.

| Row | Amount |
|---|---|
| Opening ARR (Month 11 closing) | $25,200 [ESTIMATE] |
| + New Logo ARR | $14,400 [ESTIMATE] |
| + Seat Expansion ARR | $2,160 [ESTIMATE] |
| + Tier Upgrade ARR | $0 [ESTIMATE] |
| + Add-on ARR | $0 [ESTIMATE] |
| − Seat Contraction ARR | $(0) [ESTIMATE] |
| − Logo Churn ARR | $(0) [ESTIMATE] |
| **Closing ARR (Month 12)** | **$41,760 [ESTIMATE]** |

Seat expansion of $2,160 assumes the first deal (100 seats × $12 = $14,400 ACV) adds 15 seats in Month 12 at the Starter rate: 15 × $12 × 12 = $2,160. No tier upgrades or churn are modeled at Month 12 — these metrics become observable only after the first renewal cycle (Month 13+).

---

### 23.2 Tier Migration Economics

Tier migration is the highest-leverage expansion motion in the FORM enterprise model. When a tenant upgrades from Starter to Growth, the ACV increase is larger than adding a second Starter tenant. This section calculates the exact economics of each migration path.

#### 23.2.1 Starter → Growth migration

The canonical migration event is a Starter tenant at 100 seats (the median Starter seat count [ESTIMATE]) growing to 200 seats, which crosses the 200-seat threshold that defines the Growth tier.

| Parameter | Starter (pre-migration) | Growth (post-migration) | Delta |
|---|---|---|---|
| Seat count | 100 | 200 | +100 seats |
| Price per seat per month | $12.00 | $9.00 | −$3.00 |
| Monthly contract value | $1,200 | $1,800 | +$600 |
| **Annual contract value (ACV)** | **$14,400** | **$21,600** | **+$7,200 (+50%)** |

The per-seat price decreases by 25% but the seat volume more than doubles, yielding a net +50% ACV increase. The ACV gain of $7,200 represents a single-event contribution to the Tier Upgrade ARR row of the bridge.

**Gross margin at migration:** Starter per-seat COGS is approximately $1.80/seat/month at 100 seats (§15.7). Growth per-seat COGS is approximately $1.50/seat/month at 200 seats due to RLS query amortization. Gross margin improves from 85% to 83.3% in absolute terms — the lower price per seat is more than offset by the COGS reduction from scale. See §15.6 for RLS overhead detail.

#### 23.2.2 Growth → Enterprise migration

The canonical migration event is a Growth tenant at 500 seats growing to 1,000 seats, crossing the Enterprise threshold. Enterprise pricing is negotiated; this model uses $7.00/seat as the midpoint of the $6–8 range specified in ENTERPRISE.md.

| Parameter | Growth (pre-migration) | Enterprise (post-migration) | Delta |
|---|---|---|---|
| Seat count | 500 | 1,000 | +500 seats |
| Price per seat per month | $9.00 | $7.00 | −$2.00 |
| Monthly contract value | $4,500 | $7,000 | +$2,500 |
| **Annual contract value (ACV)** | **$54,000** | **$84,000** | **+$30,000 (+56%)** |

The +$30,000 ACV jump from a single Growth → Enterprise migration equals approximately two new Starter logo deals. In the §19.7 Base scenario, the first Growth → Enterprise migration is not expected until Year 2, making it a key Bull scenario differentiator.

#### 23.2.3 Multi-year contract pricing at migration

When a tenant upgrades tier, the multi-year discount from ENTERPRISE.md applies to the new tier rate:

| Contract term | Starter/seat/mo | Growth/seat/mo | Enterprise/seat/mo (midpoint) |
|---|---|---|---|
| Annual (1yr) | $12.00 | $9.00 | $7.00 |
| 2-year (15% off) | $10.20 | $7.65 | $5.95 |
| 3-year (25% off) | $9.00 | $6.75 | $5.25 |

**Important:** the 2-year Starter rate ($10.20) is equal to the 1-year Growth rate rounded ($9.00 + margin), which creates a decision boundary. A 100-seat tenant signing a 2-year Starter contract pays $10.20 × 100 × 12 = $12,240 ACV — lower than the $14,400 they would pay on a 1-year Starter contract. The Tier Upgrade ARR calculation must use the contracted per-seat rate, not the list price, to avoid overstating expansion. The contracting system must record the effective per-seat rate at the time of migration, not the list rate.

#### 23.2.4 Migration trigger signals

The CSM layer (§19.6) should monitor the following leading indicators that precede a migration event, typically 60–90 days in advance [ESTIMATE]:

- Seat utilization ≥ 90% of contracted seats for two consecutive months
- Admin dashboard exports showing org-wide activation rate ≥ 70%
- Inbound support requests for features gated at higher tier (e.g., custom integrations)
- QBR conversation noting headcount growth plans or new department rollouts

These signals should feed a CSM alert in the admin dashboard, triggering an expansion conversation before the tenant self-identifies the need.

---

### 23.3 Add-on Revenue Taxonomy

Add-on revenue sits in its own ARR bridge row because it is sold to existing tenants rather than new logos, it has different gross margins than seat-based ARR, and it is an early proxy for product-market depth. This section defines the canonical add-on SKUs, their pricing, and their margin profiles.

#### 23.3.1 Add-on SKU table

All prices are per tenant per month, billed annually. These are pre-launch estimates pending first enterprise deals.

| Add-on SKU | Description | Price range (per tenant/month) | Minimum tier | Gross margin [ESTIMATE] |
|---|---|---|---|---|
| **White-label** | Custom domain, tenant logo, tenant primary color, "Powered by FORM" removal | $500–$2,000 [ESTIMATE] | Growth (≥ $50k ARR per ENTERPRISE.md) | ~90% |
| **Premium SLA** | P0 response < 30 min (vs standard 1hr); dedicated on-call escalation path; monthly uptime report | $300–$800 [ESTIMATE] | Starter | ~85% |
| **Custom Integration** | Webhook delivery to tenant HRIS (Workday, BambooHR, HiBob); custom field mapping; integration support | $400–$1,500 [ESTIMATE] | Growth | ~70% |
| **Advanced Analytics** | Extended data retention (24mo vs 12mo default); custom aggregate report builder; CSV/JSON export on demand | $200–$600 [ESTIMATE] | Starter | ~88% |
| **Additional Admin Seats** | Extra `tenant_admin` role seats above the default two included per plan | $50–$100/seat [ESTIMATE] | Starter | ~95% |

**White-label margin note:** The high margin on white-label reflects minimal incremental COGS (SSL certificate provisioning, DNS CNAME setup, branding asset storage). The primary cost is the one-time implementation effort at onboarding (~2–4 engineer-hours), which is amortized over the contract life. See §15.5 for white-label infrastructure cost model.

**Custom integration margin note:** The lower margin reflects ongoing webhook delivery infrastructure (retry queues, dead-letter handling, signature verification) and integration support engineer time. At scale, dedicated integration infrastructure costs approximately $50–$150/month per active integration endpoint [ESTIMATE].

#### 23.3.2 Add-on attach rate targets [ESTIMATE]

These targets are unvalidated and should be treated as planning assumptions until first-renewal data is available (OQ-29).

| Add-on SKU | Target attach rate at Year 2 renewal | Expected ARR per attached tenant |
|---|---|---|
| White-label | 15% of Growth + Enterprise tenants [ESTIMATE] | $9,600/yr (midpoint $800/mo) [ESTIMATE] |
| Premium SLA | 25% of all tenants [ESTIMATE] | $6,600/yr (midpoint $550/mo) [ESTIMATE] |
| Custom Integration | 20% of Growth + Enterprise tenants [ESTIMATE] | $11,400/yr (midpoint $950/mo) [ESTIMATE] |
| Advanced Analytics | 30% of all tenants [ESTIMATE] | $4,800/yr (midpoint $400/mo) [ESTIMATE] |
| Additional Admin Seats | 40% of all tenants [ESTIMATE] | $900/yr (3 extra seats × $75/mo) [ESTIMATE] |

#### 23.3.3 Add-on revenue in the §19.7 Base forecast

The §19.7 Base forecast does not model add-on revenue explicitly. Add-on ARR is expected to be immaterial (< 5% of total ARR) in Year 1 and becomes a meaningful NRR driver in Year 2 as tenants renew and expand. Any add-on ARR realized in Year 1 should be treated as upside to the §19.7 Base forecast, not as a budget line.

---

### 23.4 Three-Year Cohort NRR Projection by Tier

The §14.6 enterprise LTV model assumes a single 120% NRR figure across all enterprise tenants. This section disaggregates that assumption by tier, reflecting the empirical pattern in comparable B2B wellness SaaS businesses where larger tenants expand more predictably [ESTIMATE] but also have higher churn risk if the champion departs.

#### 23.4.1 NRR assumptions by tier

All figures are [ESTIMATE] — pre-launch projections pending first renewal cycle data.

| Tier | Year 1 NRR | Year 2 NRR | Year 3 NRR | Primary NRR driver |
|---|---|---|---|---|
| **Starter** (50–200 seats) | 100% | 105% | 112% | Gradual seat fill-up to contracted maximum; low upgrade rate |
| **Growth** (200–1,000 seats) | 105% | 112% | 120% | Department-by-department rollout; first tier upgrade cycle in Year 2 |
| **Enterprise** (1,000+ seats) | 108% | 118% | 127% | Seat expansion across subsidiaries; add-on attach; multi-year renewal uplift |

**Year 1 NRR note:** Year 1 NRR is measured at the first renewal (Month 12–13). It is inherently lower than Year 2+ because expansion motions (tier upgrades, add-on purchases, subsidiary rollouts) require the CSM relationship to mature. A Starter tenant that renews at 100% in Year 1 is performing to plan — contraction at Year 1 renewal is the warning signal.

**Year 3 compounding note:** The Year 3 NRR figures compound on the Year 2 cohort ARR, not the original ACV. A Growth tenant that enters at $54,000 ACV and achieves 112% NRR in Year 2 has a Year 2 closing ARR of ~$60,500. Applying 120% NRR in Year 3 yields a Year 3 closing ARR of ~$72,600 — a 34% increase from the original ACV over three years before any tier upgrade.

#### 23.4.2 Blended NRR projection

The §19.7 Base scenario mix at Month 24 is approximately 4 Starter : 3 Growth : 1 Enterprise by logo count [ESTIMATE]. Applying a revenue-weighted blend (Starter deals average $14,400 ACV, Growth $54,000 ACV, Enterprise $84,000 ACV):

**Year 2 blended NRR calculation:**

| Tier | Logo count (Month 24, Base) | Avg ACV | Total ARR weight | Tier Year 2 NRR | Weighted NRR contribution |
|---|---|---|---|---|---|
| Starter | 4 | $14,400 | $57,600 | 105% | 2.87% |
| Growth | 3 | $54,000 | $162,000 | 112% | 8.15% |
| Enterprise | 1 | $84,000 | $84,000 | 118% | 2.81% |
| **Total / Blended** | **8** | — | **$303,600** | — | **~113.8%** |

Blended Year 2 NRR of ~114% [ESTIMATE] is consistent with the §18.3 growth-stage target of ≥ 110% NRR and the §14.6 assumption of 120% NRR (which is achievable in the Bull scenario where Enterprise logos are a larger share of the mix).

**Year 3 blended NRR** (assuming mix shifts toward Growth/Enterprise as Starter tenants migrate): ~119% [ESTIMATE], approaching the §14.6 steady-state assumption.

---

### 23.5 Blended NRR Weighted by §19.7 Mix

This section documents the ongoing methodology for computing blended NRR as the enterprise mix evolves, so the calculation in §23.4.2 can be reproduced in future periods without re-deriving the formula.

#### 23.5.1 Blended NRR formula

```
Blended NRR = Σ (Tier_i ARR × Tier_i NRR) / Σ (Tier_i ARR)
```

Where Tier_i ARR is the opening ARR of the cohort being measured (not the new logo ARR added during the period), and Tier_i NRR is the trailing 12-month NRR for tenants in that tier.

#### 23.5.2 Mix-shift effect

As the enterprise book grows and Starter tenants either migrate to Growth or churn, the ARR-weighted mix shifts toward higher-NRR tiers. This mix-shift effect means blended NRR can improve even if each tier's individual NRR remains flat — it is a portfolio composition benefit, not an operational improvement. Investor reporting should separate tier-level NRR from blended NRR to prevent the mix-shift effect from masking deterioration within a specific tier.

#### 23.5.3 Sensitivity to Enterprise logo share

| Enterprise share of total ARR | Blended Year 2 NRR (all else equal) |
|---|---|
| 10% (Base scenario, Month 24) | ~114% [ESTIMATE] |
| 20% | ~116% [ESTIMATE] |
| 30% | ~117% [ESTIMATE] |
| 50% | ~119% [ESTIMATE] |

The sensitivity is moderate — Enterprise NRR is higher than Starter, but not dramatically so at Year 2. The primary NRR lever in Year 2 is Growth tier expansion, which drives volume at a meaningful ACV.

---

### 23.6 Annual Price Indexation Clause

Multi-year enterprise contracts create a revenue predictability benefit (§16.4) but expose FORM to cost inflation risk over the contract term. The price indexation clause is the contractual mechanism that protects against this risk.

#### 23.6.1 Standard clause terms

The following terms are the default position in FORM's MSA template. Deviations require founder approval and must be logged in the contract management system.

| Parameter | Standard term |
|---|---|
| Indexation metric | Consumer Price Index (CPI), US Urban All Items (CPI-U), published by the US Bureau of Labor Statistics |
| Annual increase cap | CPI + 3 percentage points, per contract year |
| Floor | 0% (no price decrease, even if CPI is negative) |
| Ceiling | 10% absolute maximum increase per year [ESTIMATE] |
| Application | Applied to the per-seat rate at each annual renewal within a multi-year term |
| Notice period | Written notice ≥ 60 days before the anniversary date |
| Opt-out | Tenant may opt out of renewal (not of the indexation clause itself) |

**Example:** A Growth tenant signs a 3-year contract at $9.00/seat/month in Year 1. If CPI runs at 3.5% in Year 2, the Year 2 rate is $9.00 × (1 + min(3.5% + 3%, 10%)) = $9.00 × 1.065 = $9.585/seat/month. If CPI runs at 5% in Year 3, the Year 3 rate is $9.585 × 1.065 = $10.21/seat/month.

#### 23.6.2 Three-year compounding scenarios

Using the $9.00 Growth base rate and varying CPI assumptions:

| CPI assumption | Year 1 rate | Year 2 rate | Year 3 rate | 3yr cumulative increase |
|---|---|---|---|---|
| CPI = 2% (low inflation) | $9.00 | $9.45 (+5%) | $9.92 (+5%) | +10.3% |
| CPI = 3.5% (moderate) | $9.00 | $9.59 (+6.5%) | $10.21 (+6.5%) | +13.4% |
| CPI = 7% (elevated) | $9.00 | $9.90 (+10% cap) | $10.89 (+10% cap) | +21.0% |
| CPI = 0% (deflation) | $9.00 | $9.27 (+3% floor above CPI) | $9.55 (+3%) | +6.1% |

The CPI+3% formula is designed so that even in a zero-inflation environment, FORM captures approximately 3% annual price growth to cover headcount cost inflation (§22.3 burn model). At the 10% annual cap, cumulative 3-year price growth of 21% still falls within the range that comparable SaaS companies apply without material customer pushback [ESTIMATE].

#### 23.6.3 EUR contract FX considerations

For EU-based tenants paying in EUR, the indexation clause should reference HICP (Harmonised Index of Consumer Prices) rather than US CPI-U. The FX impact on USD-denominated ARR is a separate risk addressed in OQ-30. The contract currency (USD vs EUR) must be specified in the MSA and should default to USD unless the tenant's treasury function explicitly requires EUR invoicing.

---

### 23.7 Expansion Audit Events (DEC-030)

Per the DEC-030 decision (HMAC-chained audit log, 7-year retention), all expansion and contraction events that affect ARR must be recorded in the audit log. These events are the authoritative source for the ARR bridge computation (§23.1) and are immutable once written.

The audit log schema is defined in AUDIT_LOG_SCHEMA.md. The events below follow the `event_type` naming convention established in that document.

#### 23.7.1 Expansion event registry

| Event type | Trigger | Severity | Retention | ARR bridge row |
|---|---|---|---|---|
| `enterprise.seat_expanded` | Seat count increased on an active tenant contract (admin action or SCIM-driven) | HIGH | 7 years | Seat Expansion ARR |
| `enterprise.seat_contracted` | Seat count decreased on an active tenant contract (admin action or SCIM-driven) | HIGH | 7 years | Seat Contraction ARR |
| `enterprise.tier_upgraded` | Tenant plan migrated to a higher tier (Starter → Growth, Growth → Enterprise) | CRITICAL | 7 years | Tier Upgrade ARR |
| `enterprise.tier_downgraded` | Tenant plan migrated to a lower tier (any direction) | CRITICAL | 7 years | Seat Contraction ARR (net delta) |
| `enterprise.addon_purchased` | Add-on SKU activated for a tenant | HIGH | 7 years | Add-on ARR |
| `enterprise.contract_renewed` | Tenant contract renewed (with or without price change); also the first-contract activation event | STANDARD | 7 years | New Logo ARR (first occurrence); Logo Churn prevention signal (subsequent) |

#### 23.7.2 Required audit log fields per event

In addition to the standard AUDIT_LOG_SCHEMA.md fields (`event_id`, `tenant_id`, `actor_id`, `created_at`, `hmac_chain`), expansion events must include the following in the `metadata` JSONB column:

| Field | Type | Description |
|---|---|---|
| `previous_seat_count` | integer | Seat count before the event |
| `new_seat_count` | integer | Seat count after the event |
| `previous_tier` | enum | `starter` / `growth` / `enterprise` |
| `new_tier` | enum | `starter` / `growth` / `enterprise` |
| `previous_mrr` | numeric(10,2) | Monthly contract value before the event |
| `new_mrr` | numeric(10,2) | Monthly contract value after the event |
| `contract_amendment_ref` | text | Internal contract amendment ID; no customer PII |
| `effective_date` | date | Contract amendment effective date (may differ from event timestamp) |

**Privacy constraint:** The `metadata` field must not contain any individual user health data, PII, or data that could identify a specific employee's behavior. The audit log records commercial terms and administrator actions only. This is consistent with the §22.7.4 expense recognition lag principle — financial events, not health events.

#### 23.7.3 HMAC chaining for expansion events

Expansion events participate in the same HMAC chain as all other audit log entries. The chain is computed per-tenant (not globally), so a high-volume enterprise tenant with frequent seat changes does not create chain contention with other tenants. The HMAC input for expansion events is:

```
HMAC_input = event_id || tenant_id || event_type || created_at || metadata_hash || previous_hmac
```

Where `metadata_hash` is SHA-256 of the serialized `metadata` JSONB (canonical JSON, sorted keys). See AUDIT_LOG_SCHEMA.md for the full chain specification.

---

### 23.8 NRR-to-Valuation Multiples

Series A investors in SaaS businesses apply ARR revenue multiples that are conditioned on NRR. A business with 130% NRR commands a materially higher multiple than one at 90% NRR, because NRR is a forward-looking revenue predictor that embeds both the quality of the customer base and the growth trajectory. This section documents the translation table that converts FORM's NRR into an ARR valuation range.

#### 23.8.1 Market multiple reference table

Multiples reflect early-stage B2B SaaS comparable transactions and public SaaS comp data as of the document date [ESTIMATE]. These are directional — actual Series A valuation depends on growth rate, TAM narrative, team quality, and market timing.

| NRR | ARR multiple (early-stage Series A) | Interpretation | FORM target scenario |
|---|---|---|---|
| 90% | 4–6× ARR [ESTIMATE] | Contraction-stage; investors expect turnaround story | Bear scenario §19.7 |
| 100% | 6–8× ARR [ESTIMATE] | Flat retention; growth comes from new logos only | Below target |
| 110% | 8–12× ARR [ESTIMATE] | Growth-stage minimum; expansion covers some churn | §18.3 growth-stage target |
| 120% | 12–16× ARR [ESTIMATE] | Strong expansion; §14.6 steady-state assumption | Base + Bull scenario Year 2 |
| 130% | 16–22× ARR [ESTIMATE] | Best-in-class; requires consistent Enterprise tier expansion | Bull scenario Year 3+ |

#### 23.8.2 Valuation implications at §19.7 ARR levels

Applying the multiple table to the §19.7 Month 24 ARR forecasts:

| Scenario | Month 24 ARR | Projected NRR | ARR multiple range | Implied valuation |
|---|---|---|---|---|
| Bear | $86,000 [ESTIMATE] | ~100% [ESTIMATE] | 6–8× | $516k–$688k [ESTIMATE] |
| Base | $188,000 [ESTIMATE] | ~114% [ESTIMATE] | 10–13× | $1.88M–$2.44M [ESTIMATE] |
| Bull | $312,000 [ESTIMATE] | ~120% [ESTIMATE] | 12–16× | $3.74M–$4.99M [ESTIMATE] |

**Important caveat:** Series A valuations at sub-$500k ARR are predominantly driven by the growth rate and team narrative rather than pure ARR multiples. The multiple table above becomes the dominant valuation framework at ARR > $500k. Before that threshold, the valuation is more accurately modeled as a story premium on top of a floor set by the multiple table.

#### 23.8.3 NRR as a fundraise gating metric

The §22.6.1 Series A readiness criteria table requires NRR ≥ 100% (minimum) and ≥ 110% (strong). The NRR-to-multiple table explains why: a business with NRR < 100% at Series A faces a 30–40% multiple compression versus the 110% NRR case, reducing the pre-money valuation by a similar magnitude. Given that the founder's equity stake is determined by the post-money dilution at Series A, maintaining NRR ≥ 110% is not just a SaaS health metric — it is directly accretive to founder economics.

---

### 23.9 Implementation Checklist

This checklist tracks the work required to make the financial machinery in §23 operational. Items are prioritized P0 (blocks investor reporting) or P1 (improves expansion motion) and mapped to the Milestone schedule used in §19.3 and §21.

#### 23.9.1 Dashboard and reporting infrastructure

| Item | Priority | Milestone | Owner | Definition of done |
|---|---|---|---|---|
| ARR bridge computation job (monthly, event-driven from audit log) | P0 | M4 | data-engineer | Job runs on 1st of each month; outputs bridge rows to `arr_bridge_monthly` table; reconciles against Stripe ARR within $1 |
| ARR bridge display in admin dashboard | P0 | M4 | enterprise-architect | Dashboard shows current-month bridge for `tenant_owner` and `tenant_admin` roles; read-only |
| Tier NRR tracking by cohort (trailing 12-month rolling) | P0 | M5 | data-engineer | Cohort NRR computed per §23.5.1 formula; available per tier and blended; investor export CSV available |
| Add-on ARR tracking (separate row from seat-based ARR) | P1 | M5 | data-engineer | Add-on ARR isolated in bridge; attach rate metric available per §23.3.2 |
| Multi-year contract price escalation calculator | P1 | M5 | enterprise-architect | Contracting tool applies CPI+3% cap per §23.6.1; outputs year-by-year rate schedule for MSA attachment |

#### 23.9.2 Event instrumentation

| Item | Priority | Milestone | Owner | Definition of done |
|---|---|---|---|---|
| `enterprise.seat_expanded` event emission | P0 | M4 | enterprise-architect | Emitted on every seat count increase in `tenant_contracts`; HMAC-chained per §23.7.3; metadata fields per §23.7.2 |
| `enterprise.seat_contracted` event emission | P0 | M4 | enterprise-architect | Emitted on every seat count decrease; same fields as expansion |
| `enterprise.tier_upgraded` event emission | P0 | M4 | enterprise-architect | Emitted on tier migration; `previous_tier` and `new_tier` populated correctly; CRITICAL severity |
| `enterprise.tier_downgraded` event emission | P0 | M4 | enterprise-architect | Emitted on downgrade; CRITICAL severity; triggers CSM alert |
| `enterprise.addon_purchased` event emission | P0 | M5 | enterprise-architect | Emitted when add-on SKU is activated; SKU identifier in metadata |
| `enterprise.contract_renewed` event emission | P0 | M4 | enterprise-architect | Emitted at contract renewal and at first contract activation; STANDARD severity |
| Integration test: ARR bridge reconciliation | P0 | M4 | qa-lead | CI test confirms that a simulated seat expansion event produces a matching delta in the ARR bridge computation; must pass before M4 deploy |

#### 23.9.3 Review cadence

| Review | Frequency | Participants | Inputs | Outputs |
|---|---|---|---|---|
| ARR bridge review | Monthly (1st week) | Founder + data-engineer | Bridge output + Stripe reconciliation | Approved ARR figure for investor reporting |
| Tier NRR review | Quarterly | Founder + customer-success | Cohort NRR table per §23.4.1 | CSM action items for at-risk tenants |
| Add-on attach rate review | Quarterly | Founder + customer-success | §23.3.2 attach rate actuals vs targets | Pricing and packaging decisions |
| Valuation multiples refresh | Semi-annually | Founder | §23.8.1 market comps vs updated NRR | Updated Series A narrative |

---

### 23.10 Open Questions

**OQ-28: Actual seat expansion rate**

§23.1 and §23.4 use [ESTIMATE] figures for seat expansion rates within a tier. The key unknown is: what fraction of Starter tenants add seats between contract signing and first renewal, and at what pace? Comparable B2B SaaS businesses report seat utilization of 60–80% at contract signing, with fill-up to 90%+ by Month 9 [ESTIMATE]. If FORM tenants fill seats faster (high engagement product), the Seat Expansion ARR in the bridge will exceed the §23.1.4 sample. Owner: data-engineer (instrument `enterprise.seat_expanded` events from Day 1 of first enterprise deal). Resolution: first 90 days of first enterprise tenant live. Priority: P0 — materially affects NRR measurement.

**OQ-29: Add-on attach rate**

§23.3.2 targets are pre-launch estimates with no comparable internal data. The white-label attach rate in particular is uncertain: ENTERPRISE.md gates white-label at $50k ARR, which limits the eligible population in Year 1 to zero or one tenants. The premium SLA attach rate is more testable early — if the first Starter tenant asks about SLA guarantees during onboarding, that is a signal that a paid SLA tier has immediate demand. Owner: customer-success (track all add-on inquiries from pilot Day 1; do not wait for formal pricing). Resolution: after first 3 renewal cycles (approximately Month 36 at current pipeline pace). Priority: P1 — affects LTV model in §14.6 but does not block Year 1 operations.

**OQ-30: FX impact on EUR contracts**

§23.6.3 flags the EUR contract currency question. The specific risk is: if FORM signs Growth or Enterprise tenants in the EU with EUR-denominated invoices, and the EUR/USD rate moves materially (±10% is not uncommon over a 3-year contract term), the USD-equivalent ARR in the §19.7 forecast shifts without any change in operating performance. At $188k Base ARR (Month 24), a 10% EUR depreciation on a 50% EUR-denominated book would reduce USD ARR by approximately $9,400 — a 5% hit to the Base scenario headline. Owner: founder + legal counsel (specify contract currency in MSA template before first EU enterprise deal). Resolution: before first EU enterprise pilot conversion. Priority: P1 — material at scale; immaterial in Year 1 at current pipeline.

---

## 24. Series A Fundraising Economics & Dilution Modeling

### 24.1 Purpose

§22 models cash flow and runway — it tells the founder how long the company can survive on a given funding amount and when to initiate a fundraise. §23 models NRR and expansion revenue — it tells the founder how healthy the enterprise book is and whether it supports a premium valuation multiple. What neither section does is model the fundraise itself: how much to raise, at what valuation, on what instrument, and what the resulting dilution looks like across scenarios. This section fills that gap. It provides a structured framework for pre-seed through Series A capital planning, dilution arithmetic, bridge instrument analysis, and the event-level audit trail that institutional investors will expect during diligence. All figures are [ESTIMATE] unless sourced from another section of this document.

---

### 24.2 Seed Round Recap & Current Cap Table Baseline

All figures below are placeholders until the pre-seed round closes. The table represents the intended post-pre-seed state; actuals will replace [ESTIMATE] markers after close.

| Round | Amount | Pre-Money Valuation | Post-Money Valuation | Founder % | Investor % | ESOP Pool % |
|---|---|---|---|---|---|---|
| Formation (day zero) | — | — | — | 100% [ESTIMATE] | 0% | 0% |
| Pre-seed (target) | $150,000–$300,000 [ESTIMATE] | $500,000–$1,000,000 [ESTIMATE] | $650,000–$1,300,000 [ESTIMATE] | 77–85% [ESTIMATE] | 5–10% [ESTIMATE] | 10% [ESTIMATE] |
| Post-pre-seed (working baseline) | $200,000 [ESTIMATE] | $800,000 [ESTIMATE] | $1,000,000 [ESTIMATE] | 80% [ESTIMATE] | 10% [ESTIMATE] | 10% [ESTIMATE] |

**Note:** All figures are placeholders until pre-seed closes. The ESOP pool is created pre-money (i.e., it dilutes the founder, not the incoming investor, unless negotiated otherwise). Ukrainian company law does not natively support stock options; a Cyprus or Delaware HoldCo structure is likely required before any option grants are issued — see OQ-33 below.

---

### 24.3 Series A Readiness Criteria

§22.6.1 defines readiness criteria focused on operational metrics (ARR, logo count, NRR, W-ACSU, D30 retention). This section expands those criteria to include financial thresholds drawn from §19 Base scenario ARR projections, §23 NRR targets, and §6 gross margin targets. Both tables must be read together before initiating a Series A process.

| Criterion | Target | Current Status | Source Section |
|---|---|---|---|
| Total ARR | ≥ $500,000 [ESTIMATE] | Pre-launch; $0 actual | §19.7 Base: $188k at Month 24; $500k requires Bull scenario or acceleration |
| Enterprise ARR | ≥ $120,000 [ESTIMATE] | Pre-launch; $0 actual | §22.6.1 "strong" threshold |
| Enterprise logo count | ≥ 3 signed pilots (at least 1 converted to paid) | 0 | §22.6.1; §21 pilot governance |
| Net Revenue Retention (NRR) | ≥ 110% [ESTIMATE] | Not measurable pre-first-renewal | §23.5 blended NRR target; §23.8 valuation driver |
| Gross margin | ≥ 70% [ESTIMATE] | Not measurable pre-launch | §6 gross margin targets |
| Consumer Pro subscribers | ≥ 1,500 [ESTIMATE] | 0 | §22.6.1 "strong" threshold |
| W-ACSU (NSM) | ≥ 3.5 sessions/week/active user | Not measurable pre-launch | METRICS.md; §22.6.1 |
| D30 retention | ≥ 55% [ESTIMATE] | Not measurable pre-launch | §22.6.1 "strong" threshold |
| Runway at raise (post-close) | ≥ 12 months [ESTIMATE] | Depends on raise amount; see §24.4 | §22.4 safe-harbor logic |
| Data room readiness | Complete: all §§ cross-referenced, dashboards investor-grade | Not started | §24.11 checklist P0 item |

**Observation:** The $500k total ARR threshold is more aggressive than the §22.6.1 enterprise-only threshold. Reaching $500k total ARR from the §19.7 Base scenario requires either a meaningfully larger consumer subscriber base than the Base case projects, or a Bull enterprise pipeline (7+ paid accounts by Month 18). Founders should treat $500k ARR as the aspirational Series A anchor, and $188k (Base Month 24) as the floor — a fundable story with appropriate narrative framing around trajectory.

---

### 24.4 Capital Requirements Model

The raise amount at Series A should fund 18 months of operations at the post-hire burn rate with a 20% buffer for surprises. The table below models use of funds for a $2–4M raise [ESTIMATE], which is appropriate for a Ukrainian/CEE-market seed or seed-extension round. Note: for most Western VCs, a $2–4M raise at a $6–16M pre-money valuation is classified as a Seed or Seed-extension, not a Series A. The label used with any given investor should match their own stage taxonomy.

| Use of Funds Category | Bear ($2M total) | Base ($2.5M total) | Bull ($4M total) | Assumptions |
|---|---|---|---|---|
| Engineering headcount (3 engineers × 18 months) | $540,000 [ESTIMATE] | $540,000 [ESTIMATE] | $900,000 [ESTIMATE] | $10,000/month/engineer [ESTIMATE] CEE market rate; Bull adds 2 senior engineers |
| ML/CV infrastructure scale-up | $120,000 [ESTIMATE] | $180,000 [ESTIMATE] | $300,000 [ESTIMATE] | GPU inference, ClickHouse Cloud, PostHog scale; §13 infrastructure model |
| Sales & marketing (enterprise GTM) | $200,000 [ESTIMATE] | $350,000 [ESTIMATE] | $600,000 [ESTIMATE] | Founder-led through Month 6; first AE hire at Month 9 per §19.4 |
| Legal & compliance (HoldCo, SAFE, contracts) | $80,000 [ESTIMATE] | $100,000 [ESTIMATE] | $150,000 [ESTIMATE] | Delaware/Cyprus HoldCo if not pre-funded; GDPR legal review; MSA templates |
| Working capital buffer (20% of total) | $333,000 [ESTIMATE] | $416,000 [ESTIMATE] | $666,000 [ESTIMATE] | API cost shock, hiring delays, enterprise sales cycle elongation per §17 |
| G&A and founder salary | $240,000 [ESTIMATE] | $300,000 [ESTIMATE] | $360,000 [ESTIMATE] | $10,000–$15,000/month founder salary [ESTIMATE] — see OQ-27 |
| **Subtotal before buffer** | ~$1,180,000 | ~$1,470,000 | ~$2,310,000 | — |
| **Rounded raise target** | **$2,000,000 [ESTIMATE]** | **$2,500,000 [ESTIMATE]** | **$4,000,000 [ESTIMATE]** | Buffer absorbed; headline raise numbers for term sheet |

**Target raise range: $2–4M [ESTIMATE].** The raise should be sized to guarantee ≥ 12 months of post-close runway under Base burn assumptions (§22.4), with sufficient upside to pursue the Bull engineering and GTM plan without a premature return to market.

---

### 24.5 Pre-Money Valuation Methodology

Three independent methods are used to triangulate the pre-money valuation at Series A. All inputs are [ESTIMATE] until first term sheet received.

#### 24.5.1 ARR Multiple Method

| Scenario | ARR at Raise | Multiple Range | Implied Pre-Money |
|---|---|---|---|
| Bear (Month 18 Bear ARR) | $80,000 [ESTIMATE] | 3–5× [ESTIMATE] | $240,000–$400,000 [ESTIMATE] |
| Base (Month 24 Base ARR) | $188,000 [ESTIMATE] | 5–8× [ESTIMATE] | $940,000–$1,504,000 [ESTIMATE] |
| Bull (Month 18 Bull ARR) | $350,000 [ESTIMATE] | 8–12× [ESTIMATE] | $2,800,000–$4,200,000 [ESTIMATE] |

Note: ARR multiples of 3–8× are appropriate for early-stage SaaS at this revenue scale in CEE/Ukrainian markets [ESTIMATE]. US/EU multiples at similar ARR but with AI-native differentiation can reach 10–20×; however, applying US multiples to a Ukrainian-founded company without a Delaware/Cyprus entity established is not supportable until the HoldCo structure is in place (OQ-33).

#### 24.5.2 Comparable Transactions (Fitness / AI SaaS)

| Company | Stage | Round Size | Pre-Money | ARR at Raise | ARR Multiple | Notes |
|---|---|---|---|---|---|---|
| AI coaching SaaS (undisclosed, CEE) | Seed | $1.5M [ESTIMATE] | $6M [ESTIMATE] | ~$400k [ESTIMATE] | ~15× [ESTIMATE] | Comparable geography; AI-native; [ESTIMATE] — unverified market intel |
| B2B wellness SaaS (EU, 2023) | Series A | $3M [ESTIMATE] | $12M [ESTIMATE] | ~$800k [ESTIMATE] | ~15× [ESTIMATE] | Closer to FORM's target; [ESTIMATE] |
| Consumer fitness app (US, 2022) | Seed | $2M [ESTIMATE] | $8M [ESTIMATE] | ~$300k [ESTIMATE] | ~27× [ESTIMATE] | US premium; not directly comparable [ESTIMATE] |
| Enterprise HR-tech (CEE, 2024) | Seed | $2.5M [ESTIMATE] | $10M [ESTIMATE] | ~$600k [ESTIMATE] | ~17× [ESTIMATE] | B2B SaaS; CEE comparable [ESTIMATE] |

**All comparable transaction figures are [ESTIMATE] and require verification against public data sources before use in investor presentations.**

#### 24.5.3 Score-Based Method (Berkus-Adjacent)

| Factor | Weight | Score (1–3) | Weighted Score | Notes |
|---|---|---|---|---|
| Team (founder + early hires) | 25% | [FOUNDER_INPUT: 1–3] | — | Solo founder pre-launch = 1; founding team of 2–3 with relevant background = 2–3 |
| Traction (ARR, users, pilots) | 30% | [FOUNDER_INPUT: 1–3] | — | 0 ARR = 1; $50k ARR = 2; $200k+ ARR = 3 |
| Market size (TAM/SAM) | 20% | 2 [ESTIMATE] | 0.40 | Global corporate wellness + consumer fitness; large but competitive |
| Product differentiation (AI, NRR, retention) | 15% | 2 [ESTIMATE] | 0.30 | AI-native architecture; NRR ≥ 110% if achieved = strong differentiator |
| NRR / revenue quality | 10% | [FOUNDER_INPUT: 1–3] | — | Pre-first-renewal = 1; NRR ≥ 100% = 2; NRR ≥ 115% = 3 |
| **Total** | **100%** | — | **[computed post-launch]** | Score maps to: 1.0–1.5 → $500k–$2M pre; 1.5–2.0 → $2M–$6M pre; 2.0–2.5 → $6M–$12M pre; 2.5–3.0 → $12M+ pre [ESTIMATE] |

---

### 24.6 Dilution Scenarios

The table below models founder dilution across three raise scenarios, including the effect of a 10% ESOP pool refresh required by most institutional investors as a pre-close condition. The ESOP pool is created pre-money (dilutes the founder before the investor's ownership is calculated), which increases the effective dilution beyond the headline percentage.

#### 24.6.1 Primary dilution table

| Scenario | Raise Amount | Pre-Money | Post-Money | Investor % | Founder % Pre-ESOP Refresh | ESOP Refresh (+10% pre-close) | Founder % Post-Refresh |
|---|---|---|---|---|---|---|---|
| Bear | $1,500,000 [ESTIMATE] | $6,000,000 [ESTIMATE] | $7,500,000 [ESTIMATE] | 20.0% [ESTIMATE] | 72.0% [ESTIMATE] | −9.0% [ESTIMATE] | 63.0% [ESTIMATE] |
| Base | $2,500,000 [ESTIMATE] | $10,000,000 [ESTIMATE] | $12,500,000 [ESTIMATE] | 20.0% [ESTIMATE] | 72.0% [ESTIMATE] | −9.0% [ESTIMATE] | 63.0% [ESTIMATE] |
| Bull | $4,000,000 [ESTIMATE] | $16,000,000 [ESTIMATE] | $20,000,000 [ESTIMATE] | 20.0% [ESTIMATE] | 72.0% [ESTIMATE] | −9.0% [ESTIMATE] | 63.0% [ESTIMATE] |

#### 24.6.2 Dilution mechanics note

The 10% ESOP pool refresh is applied to the pre-money capitalisation: if the pre-money cap table is 80% founder / 10% pre-seed investors / 10% existing ESOP, and a 10% refresh is required, the new pool is carved from all existing holders pro-rata (or from founder only if negotiated that way). The table above assumes the refresh is borne entirely by the founder, which is conservative. In practice, it is shared pro-rata across all pre-Series A holders. The headline investor ownership percentage is constant across scenarios because all three scenarios assume a 20% Series A stake — the variable is the pre-money valuation, which determines the price-per-share but not the ownership fraction.

---

### 24.7 Cap Table Evolution (Seed → Series A → Series B)

All figures are [ESTIMATE]. The Series B row is illustrative only — no fundraise thesis has been developed for Series B at this stage.

| Holder | Post-Formation | Post-Pre-Seed | Post-Series A (Base) | Post-Series B (illustrative) |
|---|---|---|---|---|
| Founder | 100% [ESTIMATE] | 80% [ESTIMATE] | 63% [ESTIMATE] | 47–52% [ESTIMATE] |
| Pre-seed investors | 0% | 10% [ESTIMATE] | 8% [ESTIMATE] | 6% [ESTIMATE] |
| ESOP pool | 0% | 10% [ESTIMATE] | 17% [ESTIMATE] | 17–20% [ESTIMATE] |
| Series A investors | 0% | 0% | 20% [ESTIMATE] | 15% [ESTIMATE] |
| Series B investors | 0% | 0% | 0% | 20% [ESTIMATE] |
| **Total** | **100%** | **100%** | **108% [ESTIMATE]** | **105–115% [ESTIMATE]** |

**Note:** Post-Series A total exceeds 100% in this table because the ESOP pool refresh is applied pre-money and the investor percentage is computed post-money, creating a double-counting artefact in a simplified illustration. A fully diluted cap table computed on actual share counts will reconcile to exactly 100%. The figures above are directional; use a dedicated cap table tool (e.g., Carta, Pulley, or equivalent) for legally accurate dilution modelling before any term sheet negotiation.

---

### 24.8 Bridge Financing Analysis

A bridge round is preferable to a premature Series A when traction is insufficient to command a strong valuation but runway is deteriorating. The analysis below defines the conditions under which a bridge is the correct decision and models the dilution cost.

#### 24.8.1 Bridge trigger criteria

| Criterion | Bridge-Favourable Signal | Series A-Favourable Signal |
|---|---|---|
| Runway | < 6 months remaining [ESTIMATE] | ≥ 9 months remaining |
| Total ARR | < $300,000 [ESTIMATE] | ≥ $500,000 [ESTIMATE] |
| Qualified pipeline (enterprise) | ≥ 3 qualified leads, not yet signed | ≥ 3 signed pilots (at least 1 paid) |
| NRR | First renewal cycle not yet completed | NRR ≥ 110% [ESTIMATE] on ≥ 2 renewals |
| Investor interest level | Warm interest but no term sheets; process premature | ≥ 2 firms in active diligence |

#### 24.8.2 Bridge instrument

Preferred instrument: **SAFE with MFN (Most Favoured Nation) clause and valuation cap**. The MFN clause ensures that if a subsequent SAFE investor receives better terms, existing SAFE holders receive the same terms retroactively. The valuation cap protects bridge investors from an outsized up-round without limiting founder optionality.

**Important:** A SAFE as typically structured under US law (Y Combinator standard form) is not recognised as a security under Ukrainian company law. A Cyprus or Delaware HoldCo is likely required before issuing any SAFE instrument to institutional investors. See OQ-33.

#### 24.8.3 Bridge dilution model

| Scenario | Bridge Amount | Valuation Cap | Discount Rate | Implied Conversion Price Ceiling | Estimated Dilution at Series A Conversion |
|---|---|---|---|---|---|
| Conservative bridge | $200,000 [ESTIMATE] | $4,000,000 [ESTIMATE] | 20% [ESTIMATE] | $3,200,000 effective [ESTIMATE] | ~5–6% [ESTIMATE] |
| Standard bridge | $350,000 [ESTIMATE] | $6,000,000 [ESTIMATE] | 20% [ESTIMATE] | $4,800,000 effective [ESTIMATE] | ~6–7% [ESTIMATE] |
| Aggressive bridge | $500,000 [ESTIMATE] | $8,000,000 [ESTIMATE] | 20% [ESTIMATE] | $6,400,000 effective [ESTIMATE] | ~6–8% [ESTIMATE] |

**Observation:** A $350k bridge at a $6M cap adds approximately 6–7% dilution on top of the Series A round, reducing founder ownership from the §24.6 Base scenario of ~63% to approximately 56–57% [ESTIMATE]. This is meaningful but not fatal if the bridge buys sufficient time to reach the ARR and NRR thresholds that justify a $10M+ pre-money Series A.

---

### 24.9 Series A Timing Triggers

The following trigger table operationalises the readiness criteria in §24.3 and §22.6.1 as discrete, monitorable thresholds. When all P0 triggers are green, the founder should initiate the fundraise preparation phase immediately (deck, data room, warm introductions). When any P0 trigger is red, assess whether a bridge is appropriate (§24.8).

| Trigger | Threshold | Current Status | Priority | Action on Breach |
|---|---|---|---|---|
| ARR milestone | ≥ $500,000 total ARR [ESTIMATE] | Pre-launch; $0 | P0 | Track monthly per §19.7; initiate raise prep when ≥ $300k and trajectory to $500k within 4 months |
| NRR milestone | ≥ 110% [ESTIMATE] on ≥ 2 enterprise renewal cycles | Not measurable | P0 | Track per §23.5; first measurable at Month 13+; use trailing NRR from most recent cohort |
| Enterprise pilot count | ≥ 3 signed pilots (at least 1 converted to paid) | 0 | P0 | Track per §21 conversion governance; initiate raise only after 3rd pilot signed |
| Gross margin floor | ≥ 70% blended [ESTIMATE] | Not measurable | P1 | Track per §6; monitor monthly once first revenue recognised; red if < 65% two months running |
| Runway floor | ≥ 9 months post-close [ESTIMATE] | Depends on pre-seed close | P0 | If < 6 months before trigger set is green, go to §24.8 bridge analysis immediately |
| Founder readiness | Deck complete, data room populated, legal structure resolved (OQ-33) | Not started | P0 | Start data room at M5; legal structure resolved before first institutional meeting |

---

### 24.10 DEC-030 Fundraising Audit Events

Fundraising events generate legal and financial obligations that must be recorded with the same integrity as revenue-recognition events. The following DEC-030 events are required for a complete audit trail. All events are server-side only (never client-emitted), HMAC-chained in sequence, and retained for 7 years to satisfy financial record-keeping requirements under applicable company law.

| Event Type | Severity | Key Metadata Fields | Retention | Notes |
|---|---|---|---|---|
| `fundraise.term_sheet_received` | HIGH | `round_name`, `lead_investor_id` (pseudonymous), `pre_money_valuation_usd`, `raise_amount_usd`, `term_sheet_date`, `expiry_date`, `received_by` (founder user_id) | 7 years | Emitted when founder receives and acknowledges receipt of term sheet in internal system; not on external signature |
| `fundraise.round_closed` | CRITICAL | `round_name`, `total_raise_usd`, `pre_money_valuation_usd`, `post_money_valuation_usd`, `close_date`, `investor_count`, `wire_confirmed` (bool), `holdco_jurisdiction` | 7 years | Emitted only after wire received and confirmed in bank account; single emission per round close |
| `fundraise.bridge_instrument_signed` | HIGH | `instrument_type` (SAFE/convertible_note/other), `amount_usd`, `valuation_cap_usd`, `discount_rate_pct`, `mfn_clause` (bool), `signing_date`, `investor_id` (pseudonymous) | 7 years | Emitted per instrument, not per investor if multiple instruments in same closing |
| `fundraise.esop_pool_expanded` | HIGH | `previous_pool_pct`, `new_pool_pct`, `expansion_date`, `board_resolution_ref`, `total_authorised_shares_post`, `holdco_jurisdiction` | 7 years | Emitted on every pool expansion; triggers cap table recomputation job |
| `fundraise.cap_table_updated` | CRITICAL | `update_reason` (round_close/esop_expansion/transfer/conversion), `founder_pct_post`, `investor_pct_post`, `esop_pct_post`, `fully_diluted_shares`, `update_date`, `verified_by` (founder user_id), `cap_table_tool_ref` | 7 years | Emitted after every material cap table change; must reconcile with cap table tool (Carta/Pulley/equivalent) within 48 hours of emission |

**7-year retention note:** Financial records relating to share issuance, investment instruments, and cap table changes are subject to legal retention requirements that vary by jurisdiction (Ukrainian law: 5 years minimum; Cyprus/Delaware: 7+ years standard). All fundraising audit events are retained for 7 years as the conservative common denominator across likely HoldCo jurisdictions.

---

### 24.11 Implementation Checklist

| Item | Priority | Milestone | Owner | Definition of Done |
|---|---|---|---|---|
| Pre-seed close and cap table documentation in authoritative tool (Carta or equivalent) | P0 | M4 | founder | Cap table tool populated with post-pre-seed state; founder % and ESOP pool confirmed; [FOUNDER_INPUT: tool selection] |
| Legal structure review: HoldCo jurisdiction decision (Cyprus / Delaware / Ukraine-only) | P0 | M4 | founder + legal counsel | Written legal memo from qualified counsel; HoldCo resolution made before any institutional fundraise contact |
| Data room skeleton: all relevant §§ cross-referenced and dashboards investor-grade | P0 | M5 | founder + data-engineer | Data room folder structure complete; §§ 6, 13, 14, 17, 18, 19, 22, 23, 24 linked or exported; Metabase dashboards accessible to founder |
| ARR tracking vs. §24.9 Series A trigger ($500k threshold) | P0 | M5 | data-engineer | `arr_monthly` metric computed per §18.3 bridge methodology; trigger threshold monitored in Metabase; Slack alert on 80% threshold cross |
| NRR tracking per §23.5 feeding into §24.3 readiness table | P0 | M5 | data-engineer | Cohort NRR query runs monthly; output piped to investor-facing dashboard; alert on NRR < 100% for any trailing quarter |
| Gross margin tracking vs. §24.3 ≥ 70% floor | P1 | M5 | data-engineer | §6 gross margin computed monthly from Stripe + infrastructure invoices; alert on < 65% for two consecutive months |
| Legal counsel engaged for SAFE / term sheet review | P0 | Pre-raise | founder | Retainer or engagement letter with counsel qualified in relevant HoldCo jurisdiction; completed before first institutional meeting |
| DEC-030 event emission: all five fundraising audit events in §24.10 | P1 | M5 | security-engineer | Event schema validated in Zod; server-side emission confirmed in staging; HMAC chaining tested; 7-year retention policy applied in Supabase |
| Cap table scenario model (§24.6 Bear/Base/Bull) populated with actual pre-seed figures | P1 | M4 | founder | §24.6.1 table updated with actual pre-seed close figures; [ESTIMATE] markers replaced where actuals known |
| Bridge instrument analysis reviewed by legal counsel | P0 | Pre-bridge (if triggered) | founder + legal counsel | SAFE template reviewed for HoldCo jurisdiction compatibility; MFN clause and valuation cap confirmed; OQ-33 resolved before signing |

---

### 24.12 Open Questions

**OQ-31: Pre-money valuation at first institutional round — CEE vs. US multiple compression**

The §24.5.1 ARR multiple method uses a 3–8× range [ESTIMATE] for CEE/Ukrainian market conditions, versus 10–20× for comparable US-market AI SaaS at Series A. The key unknown is the actual compression factor that Ukrainian-founded companies face when raising from EU or US institutional investors without a Delaware/Cyprus HoldCo already in place. If the HoldCo structure is resolved early (OQ-33), the applicable multiple may be closer to the EU norm (8–15× at this ARR scale [ESTIMATE]), materially improving the pre-money valuation in the §24.6 Bear scenario. Owner: founder. Priority: P1. Resolution: after first term sheet received — prior to that, any specific multiple is speculation.

**OQ-32: ESOP pool target — Ukrainian labour law implications for option issuance vs. phantom equity**

§24.7 assumes a standard equity option pool. However, Ukrainian labour law does not recognise stock options as a legal construct in the same way US/UK law does. Depending on the HoldCo structure, employee equity participation may need to be structured as phantom equity (cash-settled), virtual equity in a Cyprus company, or options over Delaware shares. Each has materially different tax treatment for the employee and administrative cost for the company. The 10% ESOP pool figure in §24.2 and §24.6 may need to be sized differently (larger, to account for phantom equity cash-settlement cost being a COGS item rather than an equity item) depending on the resolution. Owner: founder + legal counsel. Priority: P1. Resolution: before first engineer hire with an equity component.

**OQ-33: Bridge instrument choice — SAFE not recognised under Ukrainian company law; HoldCo structure required before institutional raise**

A Y Combinator-form SAFE is a US legal instrument that presupposes a Delaware C-corp cap table. It is not natively recognised under Ukrainian company law and has limited enforceability in Cyprus without specific drafting. This means FORM cannot issue a SAFE to an institutional investor without first establishing a HoldCo in an appropriate jurisdiction (Delaware or Cyprus are the most common choices for Ukrainian founders targeting EU/US investors). The risk is that a bridge or pre-seed investor requests a SAFE before the HoldCo is ready, creating either a legal void or a costly retroactive restructuring. Resolution requires: (1) decision on HoldCo jurisdiction, (2) HoldCo incorporated, (3) qualified legal counsel sign-off on instrument form. Owner: founder + legal counsel. Priority: P0. Resolution: before any bridge instrument is signed or any institutional investor meeting where a term sheet is a realistic near-term outcome.

---

*v1.5 additions: §24 Series A Fundraising Economics & Dilution Modeling — purpose and gap-fill vs §22 (runway) and §23 (NRR) (§24.1); pre-seed cap table baseline with [ESTIMATE] placeholders for round amount ($200k), pre-money ($800k), and founder/investor/ESOP splits pending pre-seed close (§24.2); expanded Series A readiness criteria table adding $500k total ARR, ≥70% gross margin, ≥12 months post-close runway, and data room readiness to the §22.6.1 operational metrics (§24.3); capital requirements model for $2–4M raise across Bear/Base/Bull scenarios with per-category use-of-funds table (engineering headcount, ML/CV infra, enterprise GTM, legal, working capital, G&A) and CEE market rate assumptions (§24.4); three-method pre-money valuation framework: ARR multiple (3–8× range, $240k–$4.2M implied pre-money across scenarios), four-row comparable transactions table (CEE/EU/US fitness and B2B SaaS), and five-factor score-based method with 1–3 scale and [FOUNDER_INPUT] placeholders (§24.5); dilution scenarios table across Bear ($1.5M at $6M pre), Base ($2.5M at $10M pre), Bull ($4M at $16M pre) including 10% ESOP pool refresh impact reducing founder ownership to ~63% post-refresh across all scenarios (§24.6); three-round cap table evolution table (post-formation through illustrative Series B) with fully-diluted reconciliation caveat (§24.7); bridge financing analysis with trigger criteria table (runway < 6 months, ARR < $300k, pipeline > 3 qualified leads), SAFE instrument specification with MFN and valuation cap, three bridge size scenarios ($200k–$500k) with conversion dilution estimates, and Ukrainian SAFE law caveat (§24.8); Series A timing triggers table with six P0/P1 triggers (ARR, NRR, pilot count, gross margin, runway, founder readiness) and breach actions (§24.9); five DEC-030 fundraising audit events (fundraise.term_sheet_received HIGH, fundraise.round_closed CRITICAL, fundraise.bridge_instrument_signed HIGH, fundraise.esop_pool_expanded HIGH, fundraise.cap_table_updated CRITICAL) with key metadata fields and 7-year retention policy (§24.10); implementation checklist with P0/P1 priorities across M4/M5/pre-raise milestones covering cap table tooling, HoldCo legal structure, data room, ARR/NRR/gross margin tracking, legal counsel engagement, DEC-030 event emission, and bridge instrument legal review (§24.11); OQ-31 (pre-money valuation CEE vs US multiple compression — P1, owner: founder, resolution: after first term sheet), OQ-32 (ESOP pool — Ukrainian labour law phantom equity vs options — P1, owner: founder + legal counsel, resolution: before first equity hire), OQ-33 (SAFE not recognised under Ukrainian company law; HoldCo required before any institutional instrument — P0, owner: founder + legal counsel, resolution: before any bridge or institutional meeting) (§24.12).*
