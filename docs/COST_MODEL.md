# FORM · Cost Model & Unit Economics v2.3

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
25. [Enterprise Compliance & Legal Infrastructure Cost Model](#25-enterprise-compliance--legal-infrastructure-cost-model)
    - 25.1 Purpose & Scope
    - 25.2 Annual Compliance Cost Inventory
    - 25.3 Per-Deal Legal Overhead
    - 25.4 Compliance Cost Amortization by Customer Count
    - 25.5 Seat-Count Break-Even: When Compliance Is Self-Financing per Deal
    - 25.6 Compliance Cost as % of Enterprise Revenue (operating leverage)
    - 25.7 Compliance Moat vs. TAM Expansion
    - 25.8 DEC-030 Compliance Spend Audit Events
    - 25.9 Implementation Checklist
    - 25.10 Open Questions (OQ-34 to OQ-36)
26. [Customer Success Team Scaling Economics & CS Cost Model](#26-customer-success-team-scaling-economics--cs-cost-model)
27. [Engineering Team Cost Model](#27-engineering-team-cost-model)
28. [Marketing & Demand Generation Cost Model](#28-marketing--demand-generation-cost-model)
    - 28.1 Purpose and Scope
29. [Geographic Expansion Unit Economics & FX Risk Model](#29-geographic-expansion-unit-economics--fx-risk-model)
    - 29.1 Purpose and Scope
    - 29.2 Market-by-Market ARPU Analysis (UA / PL / DE-NL / US)
    - 29.3 FX Risk Model (UAH/USD + EUR/USD stress tests)
    - 29.4 Geo-Pricing Waterfall
    - 29.5 Local Payment Infrastructure Costs
    - 29.6 Regulatory Compliance Cost by Market
    - 29.7 Expansion Sequencing (GEO-GATE-01/02)
    - 29.8 DEC-030 Geo Audit Events
    - 29.9 Implementation Checklist
    - 29.10 Open Questions (OQ-GEO-01 to OQ-GEO-03)
30. [G&A & Founder Compensation Cost Model](#30-ga--founder-compensation-cost-model)
    - 30.1 Purpose and Scope
    - 30.2 Founder Compensation Model (four phases)
    - 30.3 Accounting & Finance Function
    - 30.4 Administrative Tooling
    - 30.5 Consolidated G&A Cost Table by Stage
    - 30.6 G&A as % ARR Trajectory
    - 30.7 Back-Office Hiring Sequence
    - 30.8 DEC-030 G&A Audit Events
    - 30.9 Implementation Checklist
    - 30.10 Open Questions (OQ-GGA-01 to OQ-GGA-03)
31. [Pricing Architecture, Competitive Benchmarking & Discount Governance](#31-pricing-architecture-competitive-benchmarking--discount-governance)
    - 31.1 Purpose and Scope
    - 31.2 Consumer Pro Price Point Derivation ($19/month)
    - 31.3 Enterprise Price Point Derivation ($12 / $9 / $6–8/seat/month)
    - 31.4 Corporate Wellness Competitive Benchmarking
    - 31.5 Price Floor Analysis (COGS-Anchored Minimum Viable Price)
    - 31.6 Discount Authority Matrix
    - 31.7 Pricing Change Governance
    - 31.8 DEC-030 Pricing Audit Events
    - 31.9 Implementation Checklist
    - 31.10 Open Questions (OQ-PRICE-01 to OQ-PRICE-03)
32. [Pricing Exception Approval Procedure](#32-pricing-exception-approval-procedure)
    - 32.1 Scope
    - 32.2 Approval Authority
    - 32.3 Step-by-Step Procedure
    - 32.4 Price Floor Enforcement
    - 32.5 Record Keeping
    - 32.6 SOC 2 Relevance
    - 32.7 Checklist Closure
33. [Enterprise AI Token Budget & Billing Architecture](#33-enterprise-ai-token-budget--billing-architecture)
    - 33.1 Purpose & Scope
    - 33.2 The Two Models: Flat Per-Seat vs. Consumption Billing
    - 33.3 Token Budget Per Seat Per Tier
    - 33.4 Flat Per-Seat Model — Margin Math
    - 33.5 Consumption Billing (Not Recommended at Launch)
    - 33.6 Recommended Decision
    - 33.7 Overage Policy (Internal)
    - 33.8 Revenue Recognition (ASC 606 / IFRS 15)
    - 33.9 DEC-030 HMAC-Chained Audit Events
    - 33.10 Implementation Checklist
    - 33.11 Open Questions Resolved
34. [Enterprise Renewal Risk Register, Churn Decision Economics & Intervention Model](#34-enterprise-renewal-risk-register-churn-decision-economics--intervention-model)
    - 34.1 Purpose & Scope
    - 34.2 Churn Risk Classification Framework
    - 34.3 Financial Impact of Churn by Tier
    - 34.4 Intervention Economics: Expected Value of CSM Time
    - 34.5 Retention Discount Authorization Model
    - 34.6 Contract Vintage Churn Analysis
    - 34.7 Post-Churn Recovery Timeline
    - 34.8 DEC-030 HMAC-Chained Audit Events
    - 34.9 Implementation Checklist
    - 34.10 Open Questions
35. [Enterprise Contract Amendment, Mid-Contract Termination & Early Termination Fee Model](#35-enterprise-contract-amendment-mid-contract-termination--early-termination-fee-model)
    - 35.1 Purpose & Scope
    - 35.2 Contract Amendment Taxonomy
    - 35.3 Seat Reduction Policy (Mid-Contract)
    - 35.4 Mid-Contract Termination Economics
    - 35.5 Early Termination Fee Structure & Calculation
    - 35.6 ETF Waiver Decision Framework
    - 35.7 Multi-Year Price-Lock as Retention Tool (OQ-CHURN-02 Resolution)
    - 35.8 ASC 606 / IFRS 15 Treatment for Terminations & Amendments
    - 35.9 DEC-030 HMAC-Chained Audit Events (Closes OQ-CHURN-03)
    - 35.10 Implementation Checklist
    - 35.11 Open Questions
36. [Enterprise Implementation Cost Deep-Dive: Engineering, CS & Legal Time Model](#36-enterprise-implementation-cost-deep-dive-engineering-cs--legal-time-model)
    - 36.1 Purpose & Scope
    - 36.2 Implementation Activity Taxonomy
    - 36.3 Engineering Hour Model by Tier
    - 36.4 CS Hour Model by Tier
    - 36.5 Legal & Compliance Hour Model
    - 36.6 Bottom-Up Cost Table & §8.2 Variance Analysis
    - 36.7 Implementation Cost as % ACV — Year 1 GM Impact
    - 36.8 Multi-Deal Scale Economics
    - 36.9 Time-Tracking Instrument for OQ-08 Resolution
    - 36.10 DEC-030 HMAC-Chained Audit Events
    - 36.11 Implementation Checklist
    - 36.12 Open Questions
37. [Enterprise Pipeline Health & ARR Forecasting Model](#37-enterprise-pipeline-health--arr-forecasting-model)
    - 37.1 Purpose & Scope
    - 37.2 Pipeline Stage Definitions
    - 37.3 Stage Conversion Rate Assumptions
    - 37.4 Pipeline Health Metrics (coverage ratio, velocity, aging)
    - 37.5 ARR Build Table (Y1–Y3)
    - 37.6 NRR Decomposition & ARR Bridge
    - 37.7 Forecasting Cadence & Governance
    - 37.8 Postgres Pipeline Tracking Schema
    - 37.9 DEC-030 HMAC-Chained Audit Events
    - 37.10 Implementation Checklist
    - 37.11 Open Questions (OQ-PIPE-01 to OQ-PIPE-03)
    - 28.2 Marketing Cost Taxonomy
    - 28.3 Pre-Launch Marketing Budget (Months 1–4)
    - 28.4 App Store Optimization (ASO) Investment
    - 28.5 Consumer Paid Acquisition Economics
    - 28.6 Enterprise Demand Generation Budget
    - 28.7 Marketing Tooling Stack
    - 28.8 First Marketing Hire Economics & Hiring Gates
    - 28.9 S&M as % ARR: Industry Benchmarks vs. FORM Targets
    - 28.10 Magic Number Model
    - 28.11 DEC-030 Marketing Spend Audit Events
    - 28.12 Implementation Checklist
    - 28.13 Open Questions

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

#### 22.3.5 Year 0 Compliance Capital Overlay (resolves OQ-34)

> Cross-reference: §25.2.1 (SOC 2 Type II recertification cost), §25.9 (compliance implementation checklist), §25.10 OQ-34.

**Gap statement:** The §22.3.2 Base scenario cash flow table includes a "Legal / compliance" column covering entity formation, ongoing IP counsel, and per-deal DPA/MSA execution costs (per §22.8 OQ-26). It does not include the **one-time Year 0 compliance readiness capital**: readiness consultant engagement and SOC 2 Type I audit fee. These are non-recurring capital outlays that occur before the first enterprise pilot and are a prerequisite for the enterprise revenue timeline assumed in §22.3.2 (first enterprise contract cash at Month 8). Omitting them materially understates the cash required to reach the enterprise milestone.

**Compliance capital items and timing:**

| Item | Milestone | Low [ESTIMATE] | Mid [ESTIMATE] | High [ESTIMATE] | Source |
|---|---|---|---|---|---|
| SOC 2 readiness consultant (gap assessment + remediation guidance) | M4 (before enterprise pilot at M5) | $15,000 | $20,000 | $25,000 | §25.9, item 2 |
| SOC 2 Type I audit fee (audit firm engagement) | M6 (observation period start) | $10,000 | $20,000 | $30,000 | §25.9, item 5 |
| **Total Year 0 compliance capital** | **M4–M6** | **$25,000** | **$40,000** | **$55,000** | **§25.10 OQ-34** |

**Notes:**
- These items are separate from the continuous compliance platform (Vanta/Drata, ~$18,000/year) which is modeled in §25.2.4 and begins at M4. The platform cost is a recurring annual line; the readiness consultant and Type I audit are one-time capital items.
- The SOC 2 Type II audit ($40,000/year, per §25.2.1) is a Year 1+ recurring cost and does not appear in the Year 0 compliance capital table above.
- Timing assumption: readiness consultant engaged at M4 to align with enterprise pilot target at M5. SOC 2 Type I audit initiated at M6 (6-month observation period required for Type II, targeting report at M12 per §25.9).

**Adjusted Base scenario cumulative cash delta (compliance capital included):**

The §22.3.2 table shows cumulative delta from Month 0 without compliance capital. The overlay below shows the midpoint impact ($40,000 total: $20,000 at M4 and $20,000 at M6):

| Month | Base cumulative delta (§22.3.2) | Compliance capital outlay (mid) | Adjusted cumulative delta (mid) |
|---|---|---|---|
| 1–3 | $(6,390) | $0 | $(6,390) |
| 4 | $(9,420) | $(20,000) | $(29,420) |
| 5 | $(11,080) | $0 | $(31,080) |
| 6 | $(12,820) | $(20,000) | $(52,820) |
| 7 | $(14,185) | $0 | $(54,185) |
| 8 | $(3,295) | $0 | $(43,295) |
| 9 | $(4,335) | $0 | $(44,335) |
| 10 | $(5,520) | $0 | $(45,520) |
| 11 | $2,930 | $0 | $(37,070) |
| 12 | $2,415 | $0 | $(37,585) |
| 18 | $(18,038) | $0 | $(58,038) |

**Low scenario ($25,000 total):** Month 6 adjusted cumulative delta = $(37,820); Month 18 = $(43,038).
**High scenario ($55,000 total):** Month 6 adjusted cumulative delta = $(67,820); Month 18 = $(73,038).

**Planning implication:** The adjusted cumulative delta at Month 6 in the mid scenario is approximately $(52,820). This represents the deepest cash trough before enterprise revenue begins at Month 8. Against a pre-seed round of $150,000 [FOUNDER_INPUT placeholder], this leaves a remaining balance of approximately $97,180 at Month 6 before enterprise cash arrives — materially lower than the $137,180 implied by §22.3.2 alone. The 6-month safe-harbor fundraise trigger from §22.4 should be evaluated against the compliance-capital-adjusted burn, not the original model.

**Recommended founder action (OQ-34 resolution):** Update §22.3.2 with an explicit "Compliance capital" row added to the cash flow table — a one-time $(20,000) row at M4 and $(20,000) at M6 in the Base scenario, with footnotes anchored to §25.9. The [FOUNDER_INPUT] pre-seed capital target should account for the $40,000–$55,000 compliance capital on top of the operating burn modeled in §22.3.2. If the pre-seed raise is below $120,000, the SOC 2 Type I timeline may need to slip to M8 or later, which pushes the enterprise GA timeline by approximately 3 months.

**OQ-34 status: ✅ Closed.** Compliance capital is confirmed as a material omission from §22.3 that should be included. The §22.3.5 overlay above is the quantified resolution. §22.3.2 remains a structural reference model; founders using it for actual runway planning should apply the overlay adjustments above. The §22.3.2 table may be updated to incorporate the compliance capital rows directly when the [FOUNDER_INPUT] pre-seed amount is confirmed (OQ-23).

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

**OQ-34: Year 0 compliance capital omission from §22.3 cash flow model** ✅ **Closed**

The §22.3.2 Base scenario cash flow table did not include the one-time Year 0 compliance readiness capital (SOC 2 readiness consultant + SOC 2 Type I audit fee), totaling $25,000–$55,000 [ESTIMATE] across M4 and M6. This was a material omission: at midpoint ($40,000), the Month 6 cumulative cash trough deepens from $(12,820) to $(52,820), materially changing the runway picture before enterprise revenue begins. §22.3.5 provides the quantified compliance capital overlay, adjusted cumulative cash delta table (low/mid/high scenarios), and planning implications for the pre-seed capital target. OQ-34 is closed: the overlay is the resolution. §22.3.2 should be updated with explicit compliance capital rows when OQ-23 (pre-seed capital amount) is confirmed. Owner: compliance-officer + founder. Cross-reference: §25.9 (compliance implementation checklist), §25.10 (OQ-34 original statement). Resolved: 2026-06-02 · §22.3.5.

---

*v1.3 additions: §22 Year 1 Cash Flow Forecast & Runway Model — cash flow vs. P&L distinction and purpose (§22.1); funding assumptions structure with [FOUNDER_INPUT] placeholders (§22.2); month-by-month 18-month cash flow table across Base/Bull/Bear scenarios with per-category rows (infra, AI API, marketing, legal, hiring) aligned to §13.1 scaling tables and §19.7 enterprise revenue forecast (§22.3); runway sensitivity matrix (funding amount × monthly burn → months of runway) with 6-month safe-harbor fundraise trigger (§22.4); key cash flow triggers table covering App Store launch, first enterprise deal, Founding Engineer hire, AE hire, API cost shock, and infrastructure cliffs (§22.5); Series A fundraise timing model with readiness criteria table (ARR, NRR, logo count, W-ACSU, D30 retention) and 15–22-week process timeline (§22.6); cash management controls covering single-account policy, weekly 5-point cash review, variable-rate commitment gate (90-day max without founder approval), and expense recognition lag (§22.7); OQ-23 (pre-seed capital amount), OQ-24 (Founding Engineer hire cost), OQ-25 (App Store payout lag), OQ-26 (legal cost profile), OQ-27 (founder salary policy) (§22.8). v1.7 additions (2026-06-02): §22.3.5 Year 0 Compliance Capital Overlay — closes OQ-34 (P0, pre-seed close blocker). Two compliance capital line items not previously in §22.3.2: SOC 2 readiness consultant ($15–25k [ESTIMATE], M4) and SOC 2 Type I audit ($10–30k [ESTIMATE], M6); total Year 0 compliance capital $25–55k [ESTIMATE] (midpoint $40k). Adjusted cumulative cash delta table shows Base scenario trough deepens from $(12,820) to $(52,820) at Month 6 at midpoint — a material change to the runway picture before enterprise revenue at M8. Planning implication: pre-seed capital target should account for $40–55k compliance capital above operating burn; if raise < $120k, SOC 2 Type I timeline slips to M8 and enterprise GA delays ~3 months. OQ-34 closed; §22.8 updated with closure statement. Cross-references: §25.9 (compliance implementation checklist, P0 items at M4–M6), §25.10 (OQ-34 original statement), OQ-23 (pre-seed capital — §22.3.2 table to be updated when confirmed).*

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

---

## 25. Enterprise Compliance & Legal Infrastructure Cost Model

> Owner: compliance-officer + security-engineer + founder. Review: annually at SOC 2 renewal; triggered by each new enterprise deal signed. Audience: founder, investors, enterprise buyers (security questionnaire support), future CFO.

The COGS model in §3 and the enterprise deep-dive in §15 prove that FORM's infrastructure cost per enterprise seat is negligible ($0.34–$0.36/seat/month). What neither section models is the **fixed overhead of being enterprise-grade**: the compliance attestations, legal instruments, insurance policies, and tooling that an enterprise buyer requires before they will sign any contract. These costs are real, largely fixed, and the primary reason the minimum viable enterprise deal must exceed a floor ACV to generate acceptable gross margin. This section quantifies that overhead, models its amortization across the enterprise customer base, and derives the seat-count threshold below which a deal is compliance-cost-negative even at positive infrastructure margin.

---

### 25.1 Purpose & Scope

**What this section answers:**

1. What does it cost FORM annually to maintain enterprise-grade compliance posture, independent of how many customers have signed?
2. What is the per-deal legal cost of onboarding each enterprise customer?
3. At what number of enterprise customers does fixed compliance overhead stop being the binding margin constraint?
4. What seat count is required for a single deal to cover its pro-rata share of annual compliance costs, at current pricing?
5. How does compliance cost as a fraction of enterprise ARR evolve from $100k ARR to $2M ARR?

**Scope inclusions:**

- SOC 2 Type II audit and readiness costs (cross-reference: `docs/SOC2_READINESS.md`)
- Annual penetration testing
- Continuous compliance tooling (Vanta or equivalent)
- Cyber liability and Errors & Omissions insurance
- Privacy counsel retainer (GDPR, DPIA reviews)
- Per-deal legal costs: DPA negotiation, SCC, MSA redline, security questionnaire

**Scope exclusions:**

- Internal engineering cost of building compliant features (modelled in §8.2)
- DEC-030 HMAC audit log infrastructure (modelled in §15.4)
- SCIM/SSO implementation engineering (modelled in §15.1–15.2)
- GDPR breach notification costs (modelled in `docs/INCIDENT_RESPONSE.md` R-01 and R-08)

---

### 25.2 Annual Compliance Cost Inventory

All figures are [ESTIMATE] based on publicly available pricing, comparable-stage B2B SaaS benchmarks, and Ukrainian-founder market conditions (CEE-adjusted legal costs where applicable). Ranges represent low/high depending on provider selection, deal volume, and scope.

#### 25.2.1 SOC 2 Type II — Readiness and Audit

| Line item | One-time (Year 0) | Annual recurring | Notes |
|---|---|---|---|
| Readiness consultant (gap assessment + control implementation) | $15,000–$35,000 [ESTIMATE] | $0 (one-time) | Required before first audit; cost eliminated after Year 0 unless scope expands materially |
| SOC 2 Type II audit — reputable firm (A-LIGN, Schellman, or equivalent) | $0 (Year 0 = Type I) | $25,000–$55,000 [ESTIMATE] | Type I in Year 0 ($10–20k); Type II begins Year 1; annual recertification required |
| SOC 2 Type I (Year 0 only, for early enterprise deals) | $10,000–$20,000 [ESTIMATE] | $0 | Enables enterprise sales before Type II completion; valid for 6–12 months |
| Auditor travel and on-site time (if required) | $2,000–$5,000 [ESTIMATE] | $2,000–$5,000 [ESTIMATE] | Many firms now accept remote evidence reviews; on-site increasingly optional |
| **SOC 2 subtotal (annual, post-Year 0)** | — | **$27,000–$60,000 [ESTIMATE]** | — |

**Base assumption for this model:** $40,000/year [ESTIMATE] for annual SOC 2 Type II recertification with a mid-tier audit firm. Year 0 (Type I + readiness) total: $30,000 one-time [ESTIMATE]. These are the figures used in §25.4 amortization calculations.

#### 25.2.2 Annual Penetration Testing

| Line item | Annual cost | Notes |
|---|---|---|
| Web application + API pen test (OWASP scope, authenticated) | $10,000–$25,000 [ESTIMATE] | Required annually for SOC 2 CC7.1; must cover FORM API, admin dashboard, SSO flows |
| Cloudflare Workers + edge security review | $3,000–$8,000 [ESTIMATE] | Often bundled with web app test; separate if provider scopes differently |
| Mobile app pen test (iOS + Android, black-box) | $8,000–$18,000 [ESTIMATE] | Required if enterprise buyer security questionnaire requires it; SOC 2 does not mandate mobile specifically |
| Retesting (remediation verification) | $2,000–$5,000 [ESTIMATE] | Bundled or separate; confirm before signing with testing firm |
| **Pen test subtotal** | **$15,000–$40,000 [ESTIMATE]** | — |

**Base assumption:** $25,000/year [ESTIMATE] covering web app, API, and edge; mobile pen test deferred until M8 enterprise GA milestone or first buyer requiring it.

#### 25.2.3 Continuous Compliance Platform

| Platform | Annual cost at FORM stage | Notes |
|---|---|---|
| Vanta (Compliance Automation) | $12,000–$30,000 [ESTIMATE] | Pricing is per-employee and per-audit-framework; SOC 2 + GDPR scope increases cost |
| Drata | $12,000–$28,000 [ESTIMATE] | Similar model; pricing negotiable at early stage |
| Sprinto | $8,000–$20,000 [ESTIMATE] | CEE-friendly pricing tier; lower upfront cost, fewer enterprise integrations |
| **Base assumption (Vanta or Drata)** | **$18,000/year [ESTIMATE]** | — |

Continuous compliance tooling is the connective tissue between the controls documented in `docs/SOC2_READINESS.md` and auditor-verifiable evidence: it automates cloud provider checks (Supabase, Cloudflare), monitors policy drift, and generates evidence artefacts per control. Without it, the SOC 2 audit evidence collection process consumes an estimated 40–80 founder/engineering hours annually — a hidden cost not captured in this model but relevant to §22.3 cash flow.

#### 25.2.4 Cyber Liability and Errors & Omissions Insurance

| Policy | Annual premium range | Coverage notes |
|---|---|---|
| Cyber liability (first-party + third-party) | $8,000–$20,000 [ESTIMATE] | Coverage: breach notification costs, ransomware, business interruption, third-party liability; $1M–$5M limit typical at pre-revenue stage |
| Errors & Omissions (professional liability) | $4,000–$12,000 [ESTIMATE] | Covers SLA breach claims, failed deliverables; required by some enterprise MSA templates |
| Directors & Officers (D&O) | $5,000–$15,000 [ESTIMATE] | Required by institutional investors post-Series A; not yet required pre-Series A founder-led stage |
| **Insurance subtotal (cyber + E&O, no D&O)** | **$12,000–$32,000 [ESTIMATE]** | — |

**Base assumption:** $20,000/year [ESTIMATE] for combined cyber liability ($1M limit) and E&O ($1M limit). D&O excluded until Series A close (OQ-33 HoldCo resolution required first).

**Enterprise buyer requirement:** Many Fortune 500 and mid-market buyers require $1M–$5M cyber liability coverage as a contract condition. The $1M floor is the minimum viable insurance posture for the first three enterprise deals. Coverage must be increased to $5M+ before targeting regulated-industry buyers (HR tech for financial services, healthcare-adjacent wellness).

#### 25.2.5 Privacy Counsel Retainer

| Service | Annual cost | Notes |
|---|---|---|
| GDPR / privacy counsel retainer (EU-qualified) | $8,000–$20,000 [ESTIMATE] | Covers DPIA reviews, DPA template maintenance, breach notification support, Art. 9 compliance checks |
| Ukrainian data protection compliance counsel | $3,000–$8,000 [ESTIMATE] | Law of Ukraine on Personal Data Protection (2010); required for any processing of Ukrainian residents |
| Ad-hoc regulatory response (budget) | $3,000–$10,000 [ESTIMATE] | DSR escalations, supervisory authority correspondence, regulator inquiries |
| **Privacy counsel subtotal** | **$14,000–$38,000 [ESTIMATE]** | — |

**Base assumption:** $18,000/year [ESTIMATE] for a combined EU + Ukraine privacy counsel arrangement, with EU counsel as primary given enterprise buyer geography (EU-headquartered orgs dominate early pipeline per `docs/ENTERPRISE.md`).

#### 25.2.6 Annual Compliance Cost Summary Table

| Cost category | Low [ESTIMATE] | Base [ESTIMATE] | High [ESTIMATE] |
|---|---|---|---|
| SOC 2 Type II annual recertification | $27,000 | $40,000 | $60,000 |
| Annual penetration testing | $15,000 | $25,000 | $40,000 |
| Continuous compliance platform | $8,000 | $18,000 | $30,000 |
| Cyber liability + E&O insurance | $12,000 | $20,000 | $32,000 |
| Privacy counsel retainer | $14,000 | $18,000 | $38,000 |
| **Total annual fixed compliance overhead** | **$76,000** | **$121,000** | **$200,000** |

**Base model annual compliance overhead: $121,000 [ESTIMATE].** All subsequent calculations in this section use the base figure unless stated.

**Year 0 additional one-time costs:** SOC 2 readiness consultant ($15,000–$35,000 [ESTIMATE]) + SOC 2 Type I audit ($10,000–$20,000 [ESTIMATE]) = $25,000–$55,000 one-time [ESTIMATE]. Year 0 total: $101,000–$255,000 [ESTIMATE]. Not included in §22.3 cash flow model (OQ-34 tracks this gap).

---

### 25.3 Per-Deal Legal Overhead

Every enterprise contract requires legal instruments that go beyond the standard consumer ToS. These costs are **variable** (incurred per new deal signed) and must be included in the enterprise deal margin calculation alongside infrastructure COGS and CSM cost.

#### 25.3.1 Data Processing Agreement (DPA)

Under GDPR Art. 28, FORM acts as a data processor for the enterprise customer (controller). A compliant DPA must be executed before any personal data of the customer's employees is processed.

| DPA scenario | Legal cost | Notes |
|---|---|---|
| Customer accepts FORM standard DPA template (no redlines) | $0–$500 [ESTIMATE] | Counsel review time only; standard template maintained by privacy counsel on retainer (§25.2.5) |
| Customer submits their own DPA for review and redline | $1,500–$4,000 [ESTIMATE] | Common with larger enterprise buyers; 3–8 hours outside counsel at $200–$500/hour |
| Customer DPA requires sub-processor addendum negotiation | $500–$1,500 [ESTIMATE] | Supabase, Cloudflare, Anthropic, WorkOS must each be listed as sub-processors with adequate contractual chain |
| **DPA cost per deal (blended)** | **$500–$2,500 [ESTIMATE]** | Assumes 30% of deals require full DPA redline |

#### 25.3.2 Standard Contractual Clauses (SCC) for US–EU Transfers

Transfers of EU personal data to FORM's Cloudflare infrastructure (US-based edge) and Anthropic's API (US) require SCCs under GDPR Chapter V.

| SCC scenario | Legal cost | Notes |
|---|---|---|
| Module 2 (controller → processor) SCCs executed | $0 [ESTIMATE] | Already embedded in FORM standard DPA; no additional cost if customer accepts standard DPA |
| Customer requires bespoke SCC addendum | $500–$2,000 [ESTIMATE] | Rare below $200k ACV; more common with EU public sector buyers |
| Transfer Impact Assessment (TIA) required by customer | $2,000–$5,000 [ESTIMATE] | Required by some German/Austrian buyers post-Schrems II; not yet a standard requirement |

**Base assumption: $0–$500/deal [ESTIMATE]** (SCCs covered in standard DPA template; TIA deferred until buyer requires it).

#### 25.3.3 Master Service Agreement (MSA) Redline

Enterprise buyers almost always provide their standard MSA rather than accepting FORM's template. Redline negotiation is the single largest per-deal legal cost.

| MSA complexity | Legal cost | Typical buyer profile |
|---|---|---|
| Minor redlines (3–8 clauses, no fundamental changes to liability cap or indemnities) | $1,500–$4,000 [ESTIMATE] | Startup-friendly tech buyers; Series B+ companies with lean legal teams |
| Moderate redlines (10–20 clauses, liability cap negotiation, IP ownership, data deletion SLA) | $4,000–$8,000 [ESTIMATE] | Mid-market buyers ($500M–$2B revenue); procurement-led process |
| Complex redlines (20+ clauses, multi-jurisdiction governing law, insurance certificate requirements, right-to-audit clause) | $8,000–$20,000 [ESTIMATE] | Enterprise buyers > $2B revenue; regulated sectors (FSI, pharma) |
| **MSA cost per deal (blended across mix)** | **$3,000–$8,000 [ESTIMATE]** | Assumes 50% minor, 40% moderate, 10% complex in Year 1 pipeline |

**Base assumption: $4,500/deal [ESTIMATE]** (blended).

#### 25.3.4 Security Questionnaire Completion

Enterprise buyers require completed security questionnaires as a procurement condition. Questionnaire formats include: CAIQ (CSA), SIG Lite, HECVAT, bespoke infosec spreadsheets.

| Effort type | Internal cost | Notes |
|---|---|---|
| Standard questionnaire (50–100 questions, mostly covered by SOC 2 report) | 4–8 hours security-engineer time | At blended $75/hour internal opportunity cost = $300–$600 [ESTIMATE] |
| Complex questionnaire (150–300 questions, architecture diagrams, pen test evidence required) | 16–32 hours across security-engineer + platform-engineer | $1,200–$2,400 [ESTIMATE] internal opportunity cost |
| **Blended per deal** | **$500–$1,200 [ESTIMATE]** | — |

**Key dependency:** SOC 2 Type II report eliminates 60–80% of questionnaire questions. Pre-SOC 2, questionnaire completion time is 2–3× higher and may require external consultant support ($2,000–$4,000/deal).

#### 25.3.5 GDPR Data Protection Impact Assessment (DPIA) per Enterprise Onboarding

Under GDPR Art. 35, a DPIA is required when processing is "likely to result in a high risk" — which applies to FORM's processing of Art. 9 health and wellness data. A single DPIA covering FORM's standard processing activities (maintained by privacy counsel) covers most deals. A customer-specific DPIA addendum may be required by large regulated-sector buyers.

| DPIA scenario | Cost | Notes |
|---|---|---|
| FORM master DPIA covers deal (no customer-specific addendum) | $0/deal [ESTIMATE] | Master DPIA maintained by privacy counsel on retainer; annual update included |
| Customer requires DPIA review and sign-off on addendum | $1,000–$3,000 [ESTIMATE] | 3–6 hours outside privacy counsel; more common in healthcare-adjacent or EU public sector |
| **Blended per deal** | **$0–$600 [ESTIMATE]** | Assumes 20% of deals require addendum |

#### 25.3.6 Per-Deal Legal Overhead Summary

| Legal cost item | Low/deal [ESTIMATE] | Base/deal [ESTIMATE] | High/deal [ESTIMATE] |
|---|---|---|---|
| DPA (standard + sub-processor) | $0 | $800 | $2,500 |
| SCC addendum | $0 | $200 | $2,000 |
| MSA redline | $1,500 | $4,500 | $20,000 |
| Security questionnaire completion | $300 | $750 | $2,400 |
| DPIA addendum | $0 | $600 | $3,000 |
| **Total per-deal legal overhead** | **$1,800** | **$6,850** | **$29,900** |

**Base per-deal legal overhead: $6,850 [ESTIMATE].** High scenario ($29,900) is a regulated-industry complex deal; model as an exception requiring deal-specific approval from the discount authority matrix in §21.5.

---

### 25.4 Compliance Cost Amortization by Customer Count

The $121,000 annual fixed compliance overhead is shared across all enterprise customers active in a given year. The per-customer compliance cost decreases as the customer base grows — this is the primary source of operating leverage in the enterprise cost model beyond infrastructure COGS.

#### 25.4.1 Fixed compliance overhead per customer

| Active enterprise customers | Annual fixed compliance cost per customer [ESTIMATE] |
|---|---|
| 1 | $121,000 |
| 2 | $60,500 |
| 3 | $40,333 |
| 5 | $24,200 |
| 8 | $15,125 |
| 10 | $12,100 |
| 15 | $8,067 |
| 20 | $6,050 |
| 30 | $4,033 |
| 50 | $2,420 |

**Observation:** At 1 enterprise customer, the entire $121,000/year compliance cost must be recovered from that single deal's ACV. At $6/seat/year pricing, this requires a 20,167-seat deal — obviously impossible and not the model. The compliance overhead model assumes the cost is treated as an operating expense (OPEX) funded by total enterprise ARR, not attributed to individual deals. The question §25.5 answers is: *at what total ARR level does compliance overhead represent an acceptable % of gross revenue?*

#### 25.4.2 Variable compliance cost per deal (legal)

The $6,850/deal legal overhead from §25.3.6 is paid at deal signing, making it a deal acquisition cost alongside sales CAC (§8.5). Unlike fixed annual overhead, it scales with deal volume:

| Deals closed per year | Annual variable legal overhead | Per-deal average |
|---|---|---|
| 1 | $6,850 | $6,850 |
| 3 | $20,550 | $6,850 |
| 5 | $34,250 | $6,850 |
| 10 | $68,500 | $6,850 |
| 20 | $137,000 | $6,850 |

**Note:** The $6,850/deal figure assumes a static deal mix. As the enterprise customer profile shifts toward larger, regulated-sector buyers, the blended per-deal legal cost may increase toward $10,000–$15,000. The §21.5 discount authority matrix must be updated to flag deals where estimated legal overhead exceeds $15,000 — these require founder sign-off before proceeding.

#### 25.4.3 Total compliance cost by revenue scenario

Combining fixed annual overhead ($121,000) with variable legal costs (at $6,850/deal):

| Scenario | Deals/year | Active customers | Fixed overhead | Variable legal | Total compliance cost | Enterprise ARR | Compliance as % ARR |
|---|---|---|---|---|---|---|---|
| Pre-revenue | 0 | 0 | $121,000 | $0 | $121,000 | $0 | N/A |
| Seed pilot | 2 | 2 | $121,000 | $13,700 | $134,700 | $60,000 [ESTIMATE] | 224% |
| Early (Month 12) | 3 | 3 | $121,000 | $20,550 | $141,550 | $120,000 [ESTIMATE] | 118% |
| Growth (Month 18) | 5 | 8 | $121,000 | $34,250 | $155,250 | $350,000 [ESTIMATE] | 44% |
| Scale (Month 24) | 8 | 15 | $121,000 | $54,800 | $175,800 | $800,000 [ESTIMATE] | 22% |
| Series A (Month 30) | 12 | 25 | $121,000 | $82,200 | $203,200 | $1,500,000 [ESTIMATE] | 14% |

**Key insight:** Compliance cost is a **declining-ratio burden** that starts at >100% of ARR and decreases to ~14% by the time the enterprise business reaches Series A scale. This is characteristic of any fixed-compliance-overhead business model. The implication for enterprise pricing is that **early deals (Months 1–12) are compliance-subsidized**: FORM's compliance posture is funded primarily by the founders' capital, not by enterprise revenue. The risk is that investor expectations for enterprise contribution to gross margin may not be met until the 15+ customer milestone.

---

### 25.5 Seat-Count Break-Even: When Compliance Is Self-Financing per Deal

This analysis answers: "at what seat count does a single enterprise deal's incremental contribution to compliance overhead coverage equal the deal's pro-rata compliance cost burden?"

**Framework:** This is not a strict per-deal break-even (compliance is a shared fixed cost), but rather the question of when the ACV of a new deal is large enough to absorb its fair-share allocation of annual compliance overhead at the current customer count.

#### 25.5.1 Break-even seat count at different customer base sizes

Inputs:
- Annual pricing: $8/seat/month [ESTIMATE] = $96/seat/year (Growth plan, most common expected deal type)
- Annual fixed compliance overhead: $121,000 [ESTIMATE]
- Per-deal variable legal: $6,850 [ESTIMATE]

For each active customer count N, the per-customer compliance overhead allocation is $121,000 / N. The deal must also cover its own per-deal legal cost of $6,850.

Total compliance burden per deal = ($121,000 / N) + $6,850

Required ACV to cover compliance burden = ($121,000 / N) + $6,850

Required seats at $96/seat/year:

```
Required_seats = (($121,000 / N) + $6,850) / $96
```

| Active customers (N) | Per-deal fixed allocation | Per-deal variable legal | Total compliance burden | Required seats (at $96/seat/year) |
|---|---|---|---|---|
| 1 | $121,000 | $6,850 | $127,850 | 1,332 [ESTIMATE] — not viable as single deal |
| 3 | $40,333 | $6,850 | $47,183 | 492 [ESTIMATE] |
| 5 | $24,200 | $6,850 | $31,050 | 324 [ESTIMATE] |
| 8 | $15,125 | $6,850 | $21,975 | 229 [ESTIMATE] |
| 10 | $12,100 | $6,850 | $18,950 | 198 [ESTIMATE] |
| 15 | $8,067 | $6,850 | $14,917 | 155 [ESTIMATE] |
| 20 | $6,050 | $6,850 | $12,900 | 134 [ESTIMATE] |
| 30 | $4,033 | $6,850 | $10,883 | 113 [ESTIMATE] |
| 50 | $2,420 | $6,850 | $9,270 | 97 [ESTIMATE] |

**Critical observation:** Even at 50 enterprise customers, a deal must have ≥97 seats at $96/seat/year ($9,312 ACV) to cover its compliance burden. The current minimum deal framing ("50+ employees") aligns with this floor — but the seat floor should be monitored as compliance costs evolve.

#### 25.5.2 Impact of pricing tier on break-even

At different pricing tiers (using 15 active customers as the reference point, compliance burden = $14,917):

| Plan | Price/seat/month | Price/seat/year | Seats to cover compliance burden |
|---|---|---|---|
| Starter | $6 | $72 | 207 seats [ESTIMATE] |
| Growth | $8 | $96 | 155 seats [ESTIMATE] |
| Enterprise | $12 | $144 | 104 seats [ESTIMATE] |

**This is the quantitative justification for the minimum deal floor.** A Starter plan deal at 50 seats ($3,600 ACV) covers only 24% of its compliance burden allocation at 15 customers. The Starter plan is compliance-viable only at ≥207 seats (at 15 customers), which contradicts the entry-level positioning. Resolution options:

1. **Raise Starter minimum to 150+ seats** — difficult from a sales perspective; eliminates a market segment
2. **Absorb Starter compliance deficit as a marketing investment** — valid if Starter deals convert to Growth/Enterprise (§23.2 tier migration)
3. **Reduce per-deal legal overhead** — achievable by investing in self-service DPA tooling and standard contract acceptance tracking (OQ-35)
4. **Price Starter based on annual commitment only** — eliminates monthly flexibility, increases ACV by removing churn optionality

**Current recommendation:** Treat Starter plan deals below 150 seats as **CAC investments, not margin generators**, until the enterprise customer base reaches 20+ and compliance amortization drops to <$6,050/customer. Founders must be aware of this dynamic before offering heavy discounts on Starter to close early deals.

---

### 25.6 Compliance Cost as % of Enterprise Revenue (Operating Leverage)

Operating leverage in compliance: once the fixed overhead is covered, each incremental enterprise seat generates near-pure-margin revenue (limited only by infrastructure COGS of $0.34–$0.36/seat/month and CSM cost per §8.5).

#### 25.6.1 Compliance overhead as % of enterprise ARR at milestones

Compliance total cost = $121,000 (fixed) + ($6,850 × deals/year) (variable).

| Enterprise ARR milestone | Est. active customers | Deals/year | Total compliance cost | Compliance as % ARR |
|---|---|---|---|---|
| $50,000 | 1 | 2 | $134,700 | 269% [ESTIMATE] — funded by capital |
| $120,000 | 3 | 3 | $141,550 | 118% [ESTIMATE] — capital-subsidized |
| $350,000 | 8 | 5 | $155,250 | 44% [ESTIMATE] — approaching sustainability |
| $600,000 | 12 | 6 | $162,100 | 27% [ESTIMATE] — sustainable |
| $1,000,000 | 18 | 8 | $175,800 | 18% [ESTIMATE] — healthy SaaS ratio |
| $1,500,000 | 25 | 10 | $189,500 | 13% [ESTIMATE] — strong |
| $2,000,000 | 35 | 12 | $203,200 | 10% [ESTIMATE] — software-like leverage |
| $5,000,000 | 80 | 25 | $292,250 | 6% [ESTIMATE] — well-managed overhead |

**Target ratio:** Compliance overhead below 15% of enterprise ARR is the threshold at which the enterprise business model is compliance-cost-healthy. This milestone corresponds to approximately $1,200,000 enterprise ARR [ESTIMATE], which aligns with the pre-Series A readiness criteria in §24.3.

#### 25.6.2 Comparison to industry benchmarks

| Stage | FORM (base model) | Comparable B2B SaaS benchmark |
|---|---|---|
| Pre-revenue | N/A | N/A |
| $500k ARR | ~28% of ARR [ESTIMATE] | 20–40% for compliance-intensive SaaS |
| $1M ARR | ~17% of ARR [ESTIMATE] | 10–20% mature |
| $2M ARR | ~10% of ARR [ESTIMATE] | 8–15% mature |
| $5M ARR | ~6% of ARR [ESTIMATE] | 5–10% mature |

FORM's trajectory is in line with comparable health-data B2B SaaS (fitness, HR tech, mental health platforms) that operate under similar compliance regimes. The early-stage compliance-to-ARR ratio exceeding 100% is **not an anomaly** — it reflects the capital investment required to establish enterprise credibility before scale.

---

### 25.7 Compliance Moat vs. TAM Expansion

Compliance investment is not a pure cost — it is a revenue gate and competitive moat.

#### 25.7.1 TAM expansion unlocked by SOC 2

Without SOC 2: enterprise sales are limited to buyers willing to accept a self-attested security posture. In practice, this segment is small:

| Buyer profile | Will buy without SOC 2? | Notes |
|---|---|---|
| Series A–C startups (tech-forward) | Occasionally | At low ACV only; board/legal approval often blocked |
| Mid-market ($100M–$1B revenue) | Rarely | Security questionnaire will block without evidence |
| Enterprise ($1B+ revenue) | No | Procurement requires SOC 2 Type II or equivalent |
| Regulated sectors (FSI, pharma, healthcare-adjacent) | No | Requirement is non-negotiable |
| EU public sector | No | Requires GDPR DPA + ISAE 3000 or equivalent |

SOC 2 Type II unlocks approximately 85% of the intended enterprise TAM (`docs/ENTERPRISE.md` §Why enterprise). Without it, the addressable market is limited to ~15% of the intended buyer universe at any meaningful ACV.

#### 25.7.2 Time-to-close reduction

| Sales process stage | Pre-SOC 2 time | Post-SOC 2 time | Notes |
|---|---|---|---|
| Security review / questionnaire | 3–6 weeks | 1–2 weeks | SOC 2 report answers 60–80% of questions automatically |
| Legal / DPA negotiation | 4–8 weeks | 2–4 weeks | Standard DPA backed by SOC 2 controls reduces buyer legal pushback |
| IT/CISO approval | 2–6 weeks | 1–2 weeks | CISO sign-off faster when audited evidence available |
| **Total security-related delay** | **9–20 weeks** | **4–8 weeks** | Reduction of 5–12 weeks per deal |

At a sales velocity target of 3.0 deals/quarter (§19.1), reducing security review time by 5–12 weeks means 1–2 additional deals per quarter with no incremental headcount — **the highest-ROI enterprise investment at the current stage**.

#### 25.7.3 SOC 2 ROI calculation

| Metric | Value [ESTIMATE] |
|---|---|
| Annual SOC 2 cost (§25.2.1) | $40,000 |
| Additional deals/year enabled by SOC 2 | 3–6 [ESTIMATE] |
| Average ACV of unlocked deals | $80,000 [ESTIMATE] |
| Incremental ARR from SOC 2-enabled deals | $240,000–$480,000 [ESTIMATE] |
| SOC 2 ROI (incremental ARR / SOC 2 cost) | 6×–12× [ESTIMATE] |

**Assessment:** SOC 2 Type II is the single highest-ROI compliance investment at FORM's current stage. The $40,000/year annual audit cost generates $240,000–$480,000 in incremental ARR that would otherwise be blocked. This frames SOC 2 as a **revenue investment**, not a compliance tax — the framing that should be used in investor and board discussions.

---

### 25.8 DEC-030 Compliance Spend Audit Events

Compliance expenditures are material financial events that must be tracked in the HMAC-chained audit log per DEC-030. This serves two purposes: (1) annual SOC 2 auditor visibility into compliance investment (CC9.2 — vendor management), and (2) investor data room financial accuracy (§24.10 complements). All events are server-side, 7-year retention.

| Event type | Severity | Key metadata fields | Retention | Notes |
|---|---|---|---|---|
| `compliance.vendor_contract_signed` | HIGH | `vendor_name`, `service_category` (soc2_audit / pen_test / compliance_platform / insurance / legal_counsel), `annual_cost_usd`, `contract_start_date`, `contract_end_date`, `renewal_auto` (bool), `signed_by` (founder user_id) | 7 years | Emitted at contract execution, not at invoice payment; one event per contract, not per payment |
| `compliance.audit_initiated` | HIGH | `audit_type` (soc2_type_i / soc2_type_ii / pen_test_web / pen_test_mobile), `auditor_firm_id` (pseudonymous internal ref), `scope_description`, `start_date`, `initiated_by` (founder user_id) | 7 years | Emitted when engagement letter signed and kick-off meeting scheduled |
| `compliance.audit_report_received` | CRITICAL | `audit_type`, `report_date`, `outcome` (pass / pass_with_exceptions / qualified_opinion / adverse), `exception_count` (if pass_with_exceptions), `report_storage_ref` (S3/R2 object key, not the report contents), `received_by` (founder user_id) | 7 years | Report itself stored in compliance/evidence/ vault; only reference stored in audit event |
| `compliance.insurance_renewed` | HIGH | `policy_type` (cyber_liability / e_and_o / d_and_o), `carrier_id` (pseudonymous), `coverage_limit_usd`, `annual_premium_usd`, `policy_start_date`, `policy_end_date`, `renewed_by` (founder user_id) | 7 years | Emitted at policy binding, not renewal notice |
| `compliance.per_deal_legal_cost_recorded` | MEDIUM | `deal_id` (pseudonymous), `legal_items_completed` (array: dpa / scc / msa_redline / security_questionnaire / dpia), `total_outside_counsel_cost_usd`, `inside_hours_spent`, `recorded_by` (founder user_id), `acv_usd` (deal ACV for ratio monitoring) | 7 years | Optional event; required for deals where total per-deal legal cost exceeds $10,000 (§25.3.6 high scenario) |

**Privacy invariant:** Vendor pricing, invoice amounts, and contract terms must never appear in any per-user or per-session audit event. All compliance spend events are admin-tier events with `actor_role: 'founder'` and no association with individual user sessions or health data.

---

### 25.9 Implementation Checklist

| Item | Priority | Milestone | Owner | Definition of Done |
|---|---|---|---|---|
| Select and contract continuous compliance platform (Vanta or Drata) | P0 | M4 | compliance-officer + founder | Contract signed; Cloudflare, Supabase, GitHub integrations active; SOC 2 control list imported |
| Engage SOC 2 readiness consultant | P0 | M4 | compliance-officer | Engagement letter signed; gap assessment scheduled; §56.6 gap register from `docs/SOC2_READINESS.md` used as input |
| Obtain cyber liability + E&O insurance ($1M limits each) | P0 | M5 (before first enterprise pilot) | founder | Policy binders received; carrier confirmed; coverage certificates ready for enterprise buyer procurement |
| Engage EU-qualified privacy counsel retainer | P0 | M5 | compliance-officer | Engagement letter signed; master DPA template reviewed and approved; master DPIA completed |
| Complete SOC 2 Type I audit | P1 | M6 | compliance-officer + security-engineer | Type I report received (pass); stored in compliance/evidence/; `compliance.audit_report_received` DEC-030 event emitted |
| Record per-deal legal overhead for first 5 enterprise deals | P1 | M7 | founder + compliance-officer | Per-deal legal cost tracked in deal CRM; `compliance.per_deal_legal_cost_recorded` event emitted for any deal exceeding $10k |
| Commission annual penetration test (web app + API) | P0 | M7 | security-engineer | Scope letter signed; test executed; remediation report closed; `compliance.audit_report_received` event emitted; pen test evidence in SOC 2 control CC7.1 |
| Complete SOC 2 Type II audit | P0 | M12 | compliance-officer | Type II report received (pass or pass_with_exceptions); stored; DEC-030 event emitted; report available for enterprise buyer security questionnaires |
| Annual compliance cost tracking vs. §25.6.1 milestone table | P1 | M5 (ongoing) | data-engineer | Compliance cost as % ARR metric computed monthly from §25.2.6 total and Stripe ARR; visible in Metabase investor dashboard; alert if compliance ratio exceeds 150% for two consecutive months post-M9 |
| Update discount authority matrix (§21.5) to reflect per-deal legal overhead threshold | P1 | M6 | compliance-officer + founder | §21.5 table updated with note: deals requiring legal overhead > $15,000 require founder sign-off; field added to deal qualification checklist |

---

### 25.10 Open Questions

**OQ-34: Year 0 compliance capital requirement — should it be included in §22.3 cash flow model?**

The §22.3 Month-by-Month Cash Flow Table does not currently include the one-time Year 0 compliance readiness costs ($25,000–$55,000 [ESTIMATE]: readiness consultant + SOC 2 Type I). These are non-trivial relative to the seed funding amount and could be the deciding factor in whether the enterprise milestone timeline (M5 enterprise pilot, M12 SOC 2 Type II) is achievable within pre-seed runway. Owner: founder + data-engineer. Priority: P0. Resolution: before pre-seed close; update §22.3 with a compliance-specific budget line.

**OQ-35: Can self-service DPA tooling reduce per-deal legal overhead below $3,000?**

The $4,500/deal base MSA + DPA cost (§25.3.6) is driven primarily by outside counsel hours on MSA redlines and DPA reviews. If FORM publishes a machine-readable standard DPA (GDPR Art. 28 compliant, sub-processor addendum included) and implements a DocuSign or equivalent self-service DPA acceptance flow, buyers who accept the standard DPA pay $0 in legal overhead. Estimated impact: if 60% of buyers accept standard DPA, blended per-deal legal drops from $6,850 to approximately $3,200 [ESTIMATE]. Owner: compliance-officer. Priority: P1. Resolution: evaluate after first 3 enterprise deals — measure what fraction of buyers require redlines vs. accept standard terms.

**OQ-36: At what enterprise ARR should D&O insurance be added to the annual compliance stack?**

Directors & Officers insurance ($5,000–$15,000/year [ESTIMATE]) is excluded from §25.2.6 because it is triggered by institutional investor requirements (typically post-Series A), not by enterprise buyer contracts. However, some large enterprise buyers (particularly publicly listed companies) require their vendors to carry D&O as a condition of contract. The threshold for adding D&O to the baseline annual compliance cost is unclear. Owner: compliance-officer + founder. Priority: P2. Resolution: add D&O to §25.2.6 cost model as a conditional line item if required by any of the first three enterprise buyers, or at Series A close (whichever comes first).

---

*v1.6 additions: §25 Enterprise Compliance & Legal Infrastructure Cost Model — purpose and gap-fill vs §15 (infrastructure COGS) and §8 (enterprise economics): §15 models SSO/SCIM/audit-log infrastructure costs ($0.002/seat/month) but does not model SOC 2 audit, pen testing, insurance, or legal overhead, which represent the binding margin constraint in the pre-scale phase (§25.1); annual compliance cost inventory across five categories — SOC 2 Type II recertification ($40,000 base), pen testing ($25,000 base), Vanta/Drata continuous compliance ($18,000 base), cyber liability + E&O insurance ($20,000 base), privacy counsel retainer ($18,000 base) — total base $121,000/year (§25.2); per-deal variable legal overhead model — DPA ($800 base), SCC ($200), MSA redline ($4,500), security questionnaire ($750), DPIA addendum ($600) — total $6,850/deal base (§25.3); compliance cost amortization table showing per-customer fixed compliance allocation declining from $121,000 at 1 customer to $2,420 at 50 customers, with total compliance cost by ARR scenario from $134,700 at $50k ARR (269% ratio) to $292,250 at $5M ARR (6% ratio) (§25.4); seat-count break-even analysis showing required seats per deal to cover compliance burden at each customer-count level — at 15 customers, Growth plan ($96/seat/year) requires 155 seats; Starter plan ($72/seat/year) requires 207 seats — quantifying why Starter deals below 150 seats should be treated as CAC investments, not margin generators, until 20+ customer milestone (§25.5); operating leverage trajectory showing compliance overhead declining from 44% ARR at $350k to 10% at $2M ARR, with 15% ARR as the health threshold corresponding to ~$1.2M enterprise ARR (§25.6); compliance moat analysis quantifying SOC 2 as unlocking ~85% of intended enterprise TAM, reducing time-to-close by 5–12 weeks, and delivering 6–12× ROI on $40,000 audit cost via $240–480k incremental ARR (§25.7); five DEC-030 HMAC-chained compliance spend audit events with 7-year retention: compliance.vendor_contract_signed (HIGH), compliance.audit_initiated (HIGH), compliance.audit_report_received (CRITICAL), compliance.insurance_renewed (HIGH), compliance.per_deal_legal_cost_recorded (MEDIUM) (§25.8); implementation checklist with 10 items across P0/P1 milestones M4–M12 (§25.9); OQ-34 (Year 0 compliance capital not yet in §22.3 cash flow — P0), OQ-35 (self-service DPA tooling to reduce per-deal legal from $6,850 to $3,200 — P1), OQ-36 (D&O insurance threshold — P2) (§25.10). Cross-references: docs/SOC2_READINESS.md §56 (encryption key management evidence), docs/AUDIT_LOG_SCHEMA.md (DEC-030 event registry), docs/INCIDENT_RESPONSE.md R-08 (GDPR breach notification costs), §8.5 (enterprise CAC), §15.4 (audit log infrastructure COGS), §21.5 (discount authority matrix), §22.3 (cash flow — OQ-34 gap), §24.3 (Series A readiness). Owner: compliance-officer + security-engineer + founder.*

---

## 26. Customer Success Team Scaling Economics & CS Cost Model

### 26.1 Purpose and Scope

> Cross-references: `docs/ENTERPRISE.md` (CSM per tier; QBR cadence; privacy floor), §8.2 (one-time implementation + recurring CSM line), §8.4 (enterprise vs. consumer margin comparison), §8.7 (expansion and churn economics), §22.3 (cash flow — CS hire deferred to Month 13), §23.1 (NRR engine), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 event registry).

Section §8.2 records the marginal CSM cost per deal ($250–300/month for Growth, $400–500/month for Enterprise) as a line item in the deal P&L. It does not model:

1. **CS team headcount as a function of total customer count** — at what deal threshold does a dedicated CS hire become economically justified vs. founder-led service?
2. **CSM capacity** — how many accounts can one CSM carry before quality degrades or churn accelerates?
3. **CS cost as % ARR** — how does the CS team cost evolve relative to ARR at each scale milestone, and where does it cross the "10% CS cost as % ARR" Series A benchmark?
4. **QBR economics** — what is the fully-loaded cost of a Quarterly Business Review (QBR), and how does QBR frequency interact with CSM capacity?
5. **Churn cost avoidance** — what is the NPV of CS investment if it improves net revenue retention by 5–10 pp?

This section fills that gap. All figures are [ESTIMATE] until first 5 enterprise deals are tracked. Owner: customer-success + compliance-officer (privacy floor on any health data surfaced during QBRs) + data-engineer (health score metrics pipeline).

**Privacy floor reminder:** The CS team interacts with aggregate, tenant-level metrics only. No CSM may access individual user health data, individual workout records, or individually-identified engagement scores. Any CS tooling (CRM, health score dashboard) must enforce the same aggregate-only constraint as the admin dashboard (see `docs/ENTERPRISE.md` §Privacy floor for enterprise; `docs/DATA_MODEL.md` §17).

---

### 26.2 CS Coverage Model by Tier

`docs/ENTERPRISE.md` defines three support tiers. This section operationalises their cost structure:

| Tier | Seat range | CS coverage | SLA | QBR frequency | Estimated CS cost/account/month |
|---|---|---|---|---|---|
| **Starter** | 50–200 seats | Email-only (no named CSM); shared inbox; 24h response SLA | 24h first response | None (email check-in at 30 days and 90 days) | $150–200 [ESTIMATE] (shared inbox + tooling) |
| **Growth** | 200–1,000 seats | Named CSM, shared across 12–15 accounts | P0 < 1h, P1 < 4h | Quarterly | $250–300 [ESTIMATE] (§8.2) |
| **Enterprise** | 1,000+ seats | Named CSM, dedicated (max 6 accounts); CS Director escalation path | P0 < 30 min, P1 < 2h | Monthly | $400–500 [ESTIMATE] (§8.2) |

**Starter support note:** Starter tier does not include a named CSM. The $150–200/month shared inbox cost represents allocated time from the founder or first CS hire answering email, divided across active Starter customers. At 15 Starter customers, this totals $2,250–3,000/month in support cost that does not appear as a line item in any deal's P&L unless tracked explicitly.

---

### 26.3 CSM Capacity Model

CSM capacity is the maximum number of accounts one CS manager can service before either QBR quality degrades or proactive churn signals are missed. Capacity is measured in "ARR under management" and "account count."

#### 26.3.1 Time budget per CSM (Growth-tier accounts)

Assumes a full-time CSM at $75,000 total annual compensation ($6,250/month). Working hours: 160h/month after leave.

| Activity | Time per account per month | At 12 accounts | At 15 accounts | At 20 accounts |
|---|---|---|---|---|
| Quarterly QBR preparation (amortized) | 3h | 36h | 45h | 60h |
| Quarterly QBR delivery (amortized) | 2h | 24h | 30h | 40h |
| Monthly health score review | 1h | 12h | 15h | 20h |
| Proactive outreach (at-risk accounts) | 2h [assumes 20% at-risk] | 4.8h | 6h | 8h |
| Reactive support (P1/P2 escalations) | 1.5h | 18h | 22.5h | 30h |
| Onboarding (new accounts joining cohort) | 8h one-time, amortized across 12mo | 8h | 10h | 13.3h |
| Admin (CRM updates, DEC-030 events) | 0.5h | 6h | 7.5h | 10h |
| **Total** | — | **108.8h** | **136h** | **181.3h** |

**Capacity ceiling:** A single CSM can service 15 Growth-tier accounts within 160h/month with adequate margin (136h used, 24h buffer). At 20 accounts (181h), the CSM is over-capacity; QBR quality degrades first (prep time is cut), followed by proactive outreach. The **hard capacity ceiling is 15 Growth accounts per CSM**, with a soft warning at 13 to allow pipeline ramp-up time before a second hire is needed.

**Enterprise-tier accounts:** Enterprise accounts require ~1.5× the time per account vs. Growth (monthly cadence vs. quarterly; higher escalation frequency; white-label complexity). Hard capacity: **6 Enterprise accounts per CSM**, soft warning at 5.

**Starter accounts:** No dedicated CSM. Shared-inbox support is handled by the founder or first CS hire with a 30-minute daily triage allocation. At 20 Starter accounts, daily triage ≈ 45 minutes. At 40 accounts: 90 minutes. The inflection point where Starter support requires a dedicated support resource is **~30 accounts** (∼2h daily triage, incompatible with founder's sales and product responsibilities).

#### 26.3.2 ARR under management per CSM

ARR under management = total ACV of all accounts on one CSM's book. This is the standard Series A investor metric for CS efficiency.

| CSM type | Account capacity | Representative deal ACV | ARR under management |
|---|---|---|---|
| CSM (Growth focus) | 15 accounts | $54,000 (500-seat Growth) | **$810,000** |
| CSM (Enterprise focus) | 6 accounts | $168,000 (2,000-seat Enterprise) | **$1,008,000** |
| CSM (mixed Growth + Starter) | 12 Growth + 5 Starter | $54,000 Growth / $14,400 Starter | **$720,000** |

**Industry benchmark:** Early-stage enterprise SaaS CSMs typically manage $500k–$1.5M ARR. FORM's model ($810k–$1M ARR/CSM) sits within this range but should be recalibrated against actual QBR complexity once the first cohort of Growth accounts completes their first quarter. If QBR complexity is higher than modeled (e.g., customers require custom reporting not available in the admin dashboard), effective capacity may drop to 10–12 accounts, reducing ARR under management to $540k–$648k.

---

### 26.4 CS Team Headcount vs. Customer Count

This section models the CS headcount requirement as a function of enterprise customer count, across the three stages defined in §22 (founder-led, first CS hire at M13, Series A CS team).

#### 26.4.1 Stage 1: Founder-led CS (Months 1–12, 1–3 enterprise customers)

During Stage 1, the founder serves as CSM for all enterprise accounts. No additional CS headcount. Cost: founder opportunity cost only (not a cash COGS item). This is economically justified at ≤3 Growth accounts — the founder can deliver 12 QBRs per year (3 accounts × 4 QBRs) within a 6–8h/month time allocation.

**Stage 1 ceiling:** 3 enterprise accounts total. Above 3 accounts, the founder spends >15h/month on CS, which begins to crowd out product and sales capacity. At 4–5 accounts, a CS hire becomes a product velocity decision, not just a cost decision.

#### 26.4.2 Stage 2: First CS hire (Months 13–24, 4–15 enterprise customers)

First CS hire is a generalist Customer Success Manager at $75,000 total annual compensation. The hire carries all active enterprise accounts and inherits the existing 2–3 accounts from the founder.

| Customer count | Starter | Growth | Enterprise | Total CSM requirement | Cash CS cost/month |
|---|---|---|---|---|---|
| 4 | 1 | 3 | 0 | 0.25 FTE (founder covers) | $0 [founder time] |
| 8 | 2 | 5 | 1 | 0.65 FTE (first CS hire justified) | $4,063 [ESTIMATE] |
| 12 | 3 | 8 | 1 | 0.85 FTE | $5,313 [ESTIMATE] |
| 15 | 4 | 10 | 1 | 1.0 FTE (at capacity) | $6,250 |
| 18 | 5 | 11 | 2 | 1.3 FTE (second CS hire trigger) | $8,125 [ESTIMATE] |

**First hire timing:** The §22.3 cash flow model defers a "Founding Engineer" hire to Month 13. A CS hire competes for the same slot. Decision rule: if the customer count at Month 12 is ≥ 8 accounts (Base scenario: 2 deals in Year 1 = 2 accounts — well below threshold), the engineer hire remains the priority. If customer count exceeds 5 (unlikely in Base scenario, possible in Bull), the CS hire should be evaluated against the QBR quality risk of continued founder-led service.

**Bull scenario implication:** In the Bull scenario (3 enterprise deals in Year 1, possible pilot pipeline of 5–6 by Month 12), a dedicated CS hire may be justified as early as Month 9–10, preceding the founding engineer hire. This changes the §22.3.3 Bull scenario burn by +$4,063–$6,250/month from that point.

#### 26.4.3 Stage 3: Dedicated CS team (Months 24+, 15–50+ enterprise customers)

Post-Series A CS team structure. At 30 enterprise customers (mix of Starter, Growth, Enterprise):

| Role | Headcount | Annual compensation [ESTIMATE] | Accounts covered |
|---|---|---|---|
| VP of Customer Success | 1 | $140,000 | All (escalation path; board metrics) |
| Senior CSM (Growth + Enterprise) | 2 | $85,000 each | 12–15 Growth or 5–6 Enterprise each |
| CSM (Growth) | 1 | $70,000 | 12–15 Growth |
| Support Specialist (Starter) | 1 | $55,000 | 20–30 Starter (email + async) |
| **Total** | **5 FTE** | **$435,000/year · $36,250/month** | **30 accounts** |

At 30 accounts (§19.7 Year 2 target of 12 new + 3 retained = 15 total; 30 is the Year 3 target), the CS team cost of $435,000/year against total enterprise ARR:

| ARR at 30 accounts (representative) | CS cost | CS cost as % ARR |
|---|---|---|
| $810,000 (15 Growth × $54k ACV) | $435,000 | **53.7%** — unacceptable |
| $1,620,000 (15 Growth + 10 Starter + 5 Enterprise) | $435,000 | **26.9%** — borderline |
| $2,700,000 (30 accounts, mix weighting toward Growth) | $435,000 | **16.1%** — acceptable |
| $4,500,000 (30 accounts, average $150k ACV) | $435,000 | **9.7%** — Series A benchmark met |

**Implication:** A 5-person CS team is only justifiable when average ACV is above $90,000 or account count exceeds 40. Below that, a 2–3 person CS team is the correct structure. See §26.7 for the hiring trigger model.

---

### 26.5 CS Cost as % ARR at Scale Milestones

The standard SaaS benchmark for CS cost as % ARR is **10–15%** at Series A stage and **8–12%** at Series B. Below that threshold, CS is either under-resourced (churn risk) or highly automated (product-led CS). Above 20%, CS is over-staffed relative to revenue — a margin problem that compounds with compliance costs (§25.6).

| Scale milestone | Enterprise accounts | ARR [ESTIMATE] | CS team (FTE) | Annual CS cost | CS cost % ARR |
|---|---|---|---|---|---|
| **M12** — Year 1 close | 2–3 | $50k–$80k | 0 (founder-led) | $0 cash | 0% (founder time) |
| **M18** — first CS hire live | 5–8 | $150k–$250k | 1.0 | $75,000 | **30–50%** — expected at this stage |
| **M24** — Series A target | 12–15 | $500k–$750k | 1.5–2.0 | $112k–$150k | **15–25%** — declining toward target |
| **Series A close** | 20–25 | $1M–$1.5M | 2.5–3.0 | $175k–$220k | **12–18%** — approaching benchmark |
| **Series B target** | 40–50 | $3M–$5M | 4.0–5.0 | $315k–$435k | **8–12%** — benchmark achieved |

**Interpretation:** CS cost as % ARR will be high (30–50%) during the 1–8 account phase. This is normal and expected — the cost is largely fixed (one person) and ARR is low. The path to Series A benchmarks (10–15%) runs through deal size, not headcount cuts: each Growth or Enterprise upgrade to higher-ACV accounts is a structural improvement in the CS cost ratio without any firing.

---

### 26.6 Gross Margin Impact of CS Team Growth

CS cost is a COGS line for enterprise (not an operating expense), because it is directly required to deliver the contracted service level. This section traces how CS headcount scaling affects enterprise gross margin.

Reference deal: Growth tier, 500 seats, $9/seat/month, ACV = $54,000.

#### 26.6.1 Per-account CS cost at each stage

| Stage | CS delivery model | Monthly CS cost per Growth account | Annualised |
|---|---|---|---|
| Stage 1 (founder-led) | Founder time (opportunity cost) | $0 cash | $0 |
| Stage 2 (1 CSM, 12 accounts) | $6,250/mo CSM ÷ 12 accounts | $521 | $6,252 |
| Stage 2 (1 CSM, 15 accounts) | $6,250/mo CSM ÷ 15 accounts | $417 | $5,004 |
| Stage 3 (3 CSMs, 30 accounts) | blended cost per §26.4.3 | $375 | $4,500 |
| Scale (5 CSMs, 50 accounts) | $36,250/mo ÷ 50 accounts | $725 | $8,700 |

**Scale inversion warning:** As the VP CS hire ($140,000/year) enters the team, per-account CS cost temporarily rises from $375 to $725 until account count grows to absorb the fixed cost. This is the CS team cost cliff that must be anticipated in the hiring plan — the VP hire should coincide with a committed pipeline of 30+ accounts, not 20. Hiring the VP prematurely compresses gross margin by 3–5 pp per Growth account.

#### 26.6.2 Gross margin trajectory — Growth account, 500 seats

| Stage | ACV | Infra COGS | One-time impl (amortised Y1) | CS cost | Total COGS | Gross margin |
|---|---|---|---|---|---|---|
| Stage 1 (founder CS, Year 1) | $54,000 | $2,040 | $1,500 | $0 cash | $3,540 | **93.4%** |
| Stage 2 (1 CSM, 12 accts) | $54,000 | $2,040 | $0 (Y2+) | $6,252 | $8,292 | **84.6%** |
| Stage 2 (1 CSM, 15 accts) | $54,000 | $2,040 | $0 | $5,004 | $7,044 | **87.0%** |
| Stage 3 (blended, 30 accts) | $54,000 | $2,040 | $0 | $4,500 | $6,540 | **87.9%** |
| Scale (5 CSMs, 50 accts, pre-VP) | $54,000 | $2,040 | $0 | $4,500 | $6,540 | **87.9%** |
| Scale (VP CS added, 25 accts) | $54,000 | $2,040 | $0 | $8,700 | $10,740 | **80.1%** |

**Key insight:** The VP CS hire depresses gross margin by 7.8 pp on a Growth account — from 87.9% to 80.1% — if hired before the account base absorbs the fixed cost. At 50 accounts (blended, including larger Enterprise deals), the VP cost amortises to ~$2,800/account/year, restoring GM% to ~89–91%. This creates a strong incentive to time the VP hire to coincide with confirmed pipeline of 40+ accounts, not 20–25.

**Minimum GM% floor for enterprise viability:** The compliance cost analysis in §25.6 identifies a 15% ARR compliance overhead threshold at ~$1.2M enterprise ARR. Combined with CS costs (12–18% ARR at Series A stage), the total support cost stack (CS + compliance) is 27–33% ARR at the $1M–$1.5M ARR milestone. This leaves 54–61% gross margin after both CS and compliance — acceptable for a pre-Series B SaaS business. Gross margin below 50% at this stage is a warning signal requiring investigation of either deal size (too many small Starter accounts) or CS overstaffing.

---

### 26.7 CS Hiring Triggers & Economic Thresholds

The following decision rules operationalise the headcount model. Each trigger is binary (condition met = hire authorised).

| Trigger | Condition | Action | Justification |
|---|---|---|---|
| **CS-HIRE-01** | Active enterprise accounts ≥ 5 AND founder QBR hours > 15h/month for two consecutive months | Initiate first CSM hire at $70k–$75k total comp | Founder crowding out product capacity; churn risk rising |
| **CS-HIRE-02** | Active enterprise accounts ≥ 13 AND current CSM at-risk flag rate > 20% | Open second CSM hire | Single CSM approaching hard capacity ceiling (§26.3.1) |
| **CS-HIRE-03** | Enterprise ARR ≥ $1.5M AND deal mix ≥ 30% Enterprise tier (1,000+ seats) | Evaluate VP CS hire | Enterprise-tier complexity (monthly QBRs, white-label, custom SLAs) exceeds standard CSM scope |
| **CS-HIRE-04** | Starter account count ≥ 30 AND founder/CSM daily triage time > 90 min | Hire Support Specialist at $50k–$55k | Starter support volume requires specialised asynchronous support model |
| **CS-HIRE-05** | Net Revenue Retention (NRR) < 100% for two consecutive quarters despite CS intervention | Hire senior CSM / CSM lead with enterprise expansion focus | NRR below 100% is an expansion motion failure, not just a churn problem; requires a CS profile that can identify and close expansion opportunities |

**Anti-trigger (hiring guard):** Do not hire a VP CS before CS-HIRE-03 is met. The $140,000 VP hire is a margin-compressing fixed cost that does not generate revenue directly. A strong senior CSM ($85,000) with a VP title internally is preferable until $1.5M enterprise ARR — at which point a VP with external credibility (investor relations, QBR executive presence) generates positive ROI.

---

### 26.8 QBR Economics & Efficiency Ratios

#### 26.8.1 Fully-loaded QBR cost

A Quarterly Business Review (QBR) requires preparation, delivery, and follow-up. The fully-loaded cost includes CSM time and the customer's time (measured as a customer success investment, not a FORM cost, but relevant to whether customers participate).

| Activity | FORM hours | FORM cost (CSM at $37.50/h) | Notes |
|---|---|---|---|
| Data pull and health score assembly | 2h | $75 | Pulling aggregate metrics from admin dashboard; structured report |
| Slide/deck preparation | 3h | $112.50 | QBR template; customisation for customer's goals |
| QBR delivery (60-min call) | 1h | $37.50 | CSM + (optional) CS Director or AE for at-risk accounts |
| Action item follow-up and CRM update | 1h | $37.50 | DEC-030 event emission; CRM notes; action items to customer |
| **Total per QBR** | **7h** | **$262.50** | — |
| **Annual QBR cost per Growth account (4× per year)** | **28h** | **$1,050** | — |
| **Annual QBR cost per Enterprise account (12× per year, monthly cadence)** | **84h** | **$3,150** | — |

**QBR cost as % of ACV:**

| Tier | ACV | Annual QBR cost | QBR cost % ACV |
|---|---|---|---|
| Growth (500 seats × $9/seat) | $54,000 | $1,050 | **1.9%** |
| Enterprise (2,000 seats × $7/seat) | $168,000 | $3,150 | **1.9%** |
| Starter — no formal QBR | $14,400 | $0 | 0% |

The QBR cost is remarkably consistent at ~2% ACV across tiers. This is structurally sound — the QBR is the primary churn-prevention investment and its cost scales proportionally to the account's value to FORM.

#### 26.8.2 QBR ROI: churn avoidance

The economic case for QBRs is churn avoidance, not customer satisfaction per se. If a QBR prevents one logo churn per year (out of a 15-account book), the ROI is:

| Prevented churn | Recovered ARR | QBR cost for 15 Growth accounts | QBR ROI |
|---|---|---|---|
| 1 Growth logo ($54,000 ACV) | $54,000 | $15,750/year | **3.4× in year 1** |
| 1 Growth logo + 3-year LTV | $147,420 (§8.6) | $47,250 (3 years QBR) | **3.1× over 3 years** |

If QBRs prevent even one logo churn out of 15, they pay for themselves 3× over. This is the investment framing for the CS hiring decision at CS-HIRE-01: the first CSM does not cost $75,000/year — they protect $540,000–$810,000 in ARR that would otherwise churn at 10–20% annually without structured engagement.

#### 26.8.3 Minimum viable QBR content (privacy floor enforced)

All QBR data shown to the customer must pass the privacy floor:

| Metric | Shown in QBR? | Privacy status | Notes |
|---|---|---|---|
| % of invited employees who activated | ✅ Yes | Aggregate-only | Min denominator: 5 activated to display (k-anonymity) |
| Weekly active % (of activated) | ✅ Yes | Aggregate-only | Same k-anonymity floor |
| Average weekly workouts per activated user | ✅ Yes | Aggregate-only | No per-user breakdown |
| Opt-in NPS (if collected) | ✅ Yes | Voluntary + aggregate | Individual responses never shown |
| Engagement trend (month-over-month) | ✅ Yes | Aggregate time series | No individual trend lines |
| Department-level breakdown | ❌ No | Privacy floor violation | Manager-reports never available per ENTERPRISE.md |
| Individual user workout history | ❌ No | Privacy floor violation | HR never sees individual data — hard constraint |
| Body composition metrics (any) | ❌ No | GDPR Art. 9 — never aggregated | Body composition data never aggregated to employer |
| Mental health flag rates | ❌ No | GDPR Art. 9 — never aggregated | Mental health data never aggregated to employer |

**CSM compliance training requirement:** Every CSM must complete a privacy-floor training module before their first QBR. The training must cover: (1) what data is and is not available in the admin dashboard; (2) how to respond to a customer's request for individual user data ("Our privacy floor is a contractual and architectural constraint — this data does not exist in a reportable form for any employer"); (3) the escalation path if a customer's procurement team pressures for individual data as a contract amendment condition (escalate to compliance-officer; answer is always no). Owner: compliance-officer + customer-success.

---

### 26.9 Churn Signal Detection & CS Cost Avoidance

The CS investment is most efficiently deployed when churn is predicted 60–90 days in advance — far enough to intervene, close enough to be accurate. This section defines the health score model and its economic implications.

#### 26.9.1 FORM Enterprise Health Score (FEHS)

The FORM Enterprise Health Score is a 0–100 composite metric calculated monthly for each enterprise tenant. It combines leading indicators of churn risk. All inputs are aggregate-only (privacy floor compliant).

| Signal | Weight | Green | Yellow | Red |
|---|---|---|---|---|
| Activation rate (% invited employees who activated) | 25% | ≥ 40% | 25–40% | < 25% |
| Weekly active rate (% of activated, rolling 4-week) | 25% | ≥ 50% | 30–50% | < 30% |
| Seat utilisation trend (month-over-month change) | 20% | Increasing | Flat | Declining ≥ 10% |
| Last executive engagement (days since last QBR or champion contact) | 15% | < 45 days | 45–90 days | > 90 days |
| Support ticket volume trend (month-over-month) | 10% | Stable or decreasing | Slight increase | ≥ 2× month-over-month |
| Contract renewal distance (months until renewal) | 5% | > 6 months | 3–6 months | < 3 months (renewal risk) |

**Score thresholds:**

| FEHS | Status | CS action |
|---|---|---|
| 75–100 | Green — healthy | Routine QBR; expansion outreach when FEHS > 85 |
| 50–74 | Yellow — at-risk | Proactive CSM outreach within 5 business days; root cause identification; remediation plan |
| 25–49 | Red — high churn risk | CSM + CS Director joint outreach within 2 business days; executive sponsor engagement; recovery plan |
| 0–24 | Critical — likely churning | Founder + CSM joint call within 24h; escalation path per INCIDENT_RESPONSE §12 |

#### 26.9.2 Economic impact of early churn detection

If FEHS drops to Red status 90 days before renewal vs. 30 days before renewal, the intervention window difference is 60 days. In that 60-day window, FORM can:
- Complete one additional executive QBR
- Deliver a custom adoption workshop
- Offer a contract amendment (seat expansion, feature unlock) to re-engage the champion

Estimated churn prevention probability increase with 90-day vs. 30-day detection: 30–40 pp [ESTIMATE — replace after first 5 churn events]. On a 10-account Growth cohort ($540,000 ARR), moving from 30-day to 90-day detection and increasing save rate by 30 pp prevents:

```
Prevented churn = 0.30 (save rate uplift) × 0.20 (base churn rate) × $540,000 ARR
              = 0.06 × $540,000
              = $32,400 ARR saved per cohort per year [ESTIMATE]
```

Against an annual FEHS computation infrastructure cost of approximately $200–500/month (data-engineer time + Supabase materialized view compute — see §26.11), the ROI is:

```
FEHS ROI = $32,400 ARR saved / $3,600 FEHS infrastructure cost = 9× [ESTIMATE]
```

This justifies full investment in the health score pipeline at any stage above 5 enterprise customers.

---

### 26.10 DEC-030 Customer Success Audit Events

CS events must be emitted to the HMAC-chained audit log for two purposes: (1) SOC 2 A1.1 (availability commitments — QBR completion is evidence of SLA commitment fulfillment); (2) investor data room — CS metrics derived from audit events are the NRR foundation (§23.1) and Series A diligence artefacts (§24.3).

**Privacy constraint:** All CS audit events are tenant-scoped aggregate events. No CS event may include `user_id`, individual session identifiers, or health data fields. Events include `tenant_id`, account-level metrics only. The CS event pipeline must be reviewed by compliance-officer before any new field is added.

| Event type | Severity | Key metadata fields | Retention | Notes |
|---|---|---|---|---|
| `enterprise.qbr_completed` | STANDARD | `tenant_id`, `qbr_date`, `csm_user_id` (internal admin ID, not customer user), `qbr_type` (quarterly / monthly / kickoff / offboarding), `fehs_score_at_qbr` (0–100), `activation_rate_pct` (aggregate), `weekly_active_rate_pct` (aggregate), `action_items_count`, `customer_exec_attended` (bool) | 7 years | Emitted by CSM via admin tooling immediately after QBR completion; not self-reported — requires CSM authentication + `tenant_id` confirmation |
| `enterprise.health_score_updated` | STANDARD | `tenant_id`, `fehs_score` (0–100), `fehs_status` (green/yellow/red/critical), `score_delta` (change from previous month), `primary_risk_signal` (enum: activation_low / wau_declining / seat_utilisation_falling / champion_dark / support_spike / renewal_imminent), `computed_by` (cron job identifier) | 3 years | Emitted monthly by FEHS computation cron; not by CSM; fully automated |
| `enterprise.churn_signal_flagged` | HIGH | `tenant_id`, `signal_type` (enum: fehs_red / fehs_critical / seat_utilisation_below_threshold / champion_departure_reported / competitor_evaluation_flagged / renewal_declined / legal_hold_requested), `signal_date`, `csm_user_id` (CSM who flagged), `escalated_to_director` (bool), `recovery_plan_initiated` (bool) | 7 years | Emitted when CSM manually flags a churn signal OR when FEHS computation crosses Red threshold; both paths emit the same event type |
| `enterprise.renewal_negotiation_started` | STANDARD | `tenant_id`, `current_acv_usd`, `proposed_acv_usd`, `renewal_date`, `seat_change` (delta: positive = expansion, negative = contraction), `negotiation_owner` (csm / founder / ae), `csm_user_id` | 7 years | Emitted when renewal outreach begins (typically 90 days before contract end); not at close — `subscription_events.subscription_renewed` (DATA_MODEL §24) handles the close event |
| `enterprise.account_expanded` | STANDARD | `tenant_id`, `expansion_type` (enum: seat_add / tier_upgrade / new_module / multi_year_commit), `seats_before`, `seats_after`, `acv_before_usd`, `acv_after_usd`, `expansion_mrr_usd`, `csm_user_id`, `expansion_trigger` (enum: champion_led / csm_led / qbr_led / organic_usage) | 7 years | Emitted at contract amendment execution (not at opportunity creation); `expansion_trigger` field distinguishes CSM-driven expansion from organic |
| `enterprise.account_churned` | CRITICAL | `tenant_id`, `churn_date`, `churn_reason` (enum: budget_cut / product_fit / champion_departure / competitor / employer_offboarding / privacy_concern / pricing / unknown), `final_acv_usd`, `total_lifetime_arr_usd`, `months_as_customer`, `fehs_at_churn`, `last_qbr_date`, `was_at_risk_flagged` (bool — was churn_signal_flagged emitted before churn?) | 7 years | CRITICAL severity because churn is the primary revenue risk; `was_at_risk_flagged` enables retrospective analysis of whether FEHS detected the churn in time; feeds §26.9 detection model calibration |

**HMAC chain integrity:** All six events must be appended to the DEC-030 chain per `docs/AUDIT_LOG_SCHEMA.md`. The `enterprise.account_churned` CRITICAL event must be followed by the `enterprise.offboarding_initiated` event from `docs/DATA_MODEL.md §25` within 24 hours of `churn_date` confirmation. A gap between these two events (churn marked but offboarding not initiated) is a process failure that should appear in the FEHS audit trail.

---

### 26.11 Implementation Checklist

| Item | Priority | Milestone | Owner | Definition of Done |
|---|---|---|---|---|
| Build FEHS computation cron (§26.9.1): monthly Supabase Edge Function reading aggregate metrics from `tenant_health_snapshots` materialized view (DATA_MODEL §17); compute weighted score per §26.9.1 formula; write result to `tenant_health_scores` table; emit `enterprise.health_score_updated` DEC-030 event | P0 | M5 (before first enterprise pilot) | data-engineer + platform-engineer | Cron runs monthly; FEHS scores visible in admin dashboard for internal use; DEC-030 event emitted; no individual user data in any table or event |
| Implement CSM admin tooling for QBR completion logging: internal admin page (authenticated via Cloudflare Access) allowing CSM to record QBR completion, upload QBR deck link (not contents), note action items, and emit `enterprise.qbr_completed` DEC-030 event | P1 | M5 | platform-engineer | Admin page deployed; QBR event emitted correctly; CSM privacy-floor training completed before first QBR |
| Register all six CS DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` with full Zod schema, severity, retention, and privacy notes | P0 | M5 | platform-engineer + compliance-officer | All events registered; HMAC chain linkage between `enterprise.account_churned` and `enterprise.offboarding_initiated` documented in runbook |
| Build FEHS dashboard panel in admin dashboard (internal view — not customer-facing): show all enterprise tenants ranked by FEHS score, colour-coded by status; drill-down to signal breakdown per tenant | P1 | M6 | data-engineer + design-craft | FEHS panel deployed in admin dashboard internal view; no individual user data shown; CS director can view all tenants; CSM can view their own book |
| CSM privacy-floor training module: written guide covering (1) what data is and is not available in admin dashboard; (2) how to respond to individual data requests; (3) escalation path; reviewed by compliance-officer | P0 | M5 (before first CSM hire or first QBR) | compliance-officer + customer-success | Training document complete and reviewed by compliance-officer; completion tracked (logged in CSM onboarding checklist) |
| Add FEHS Red/Critical to PagerDuty alert routing: `enterprise.health_score_updated` event with `fehs_status: critical` triggers PagerDuty P1 to CS Director and founding team within 5 min of event emission | P1 | M6 | devops-lead | PagerDuty rule created; test critical event fired in staging; alert received within 5 minutes |
| Add CS metrics to investor dashboard (Metabase): NRR by cohort (from `enterprise.account_expanded` and `enterprise.account_churned` events); average FEHS by tier; QBR completion rate (QBRs completed / QBRs scheduled per quarter) | P1 | M8 | data-engineer | Metabase dashboard panel deployed; NRR metric computation validated against §23.1 NRR engine; visible to founder and investors with read-only Metabase access |
| Update §22.3 cash flow model to include CS hire scenario: add a conditional "CS hire at M10" column to §22.3.3 Bull scenario showing the cash impact of an early CS hire vs. the Base scenario M13 engineer hire | P2 | M6 | founder + data-engineer | §22.3 updated with CS-hire variant; OQ-CS-01 resolution path documented |
| Document CS OKRs in OKRS_2026.md: add CS-specific OKRs for QBR completion rate (target 100%), FEHS average ≥ 70, NRR ≥ 105%, churn signal detection lead time ≥ 60 days | P2 | M6 | customer-success + product-manager | OKRs added; baseline for measurement set at first cohort QBR completion |

---

### 26.12 Open Questions

**OQ-CS-01: Should the first CS hire precede or follow the founding engineer hire at Month 13?**

The §22.3 Base scenario defers all hiring to Month 13, where a founding engineer is assumed. If the customer count at Month 12 exceeds 5 (possible in the Bull scenario or with an accelerated pilot pipeline), a CS hire becomes the higher-priority hire. The CS hire ($70k–$75k) is cheaper than an engineer ($100k in §22.3.3), provides faster time-to-value for at-risk accounts, and protects the ARR base that funds the engineer hire later. Decision rule proposal: at Month 9, if active enterprise customers ≥ 4 AND founder QBR hours ≥ 12h/month, replace the M13 engineer hire with a M11 CS hire + M16 engineer hire. Owner: founder + product-manager. Priority: **P0 — decision must be made before Month 9 to allow recruiting lead time.** Resolution: establish a clear decision gate at the Month 9 board review using the trigger criteria above.

**OQ-CS-02: Should FEHS be visible to the customer in the admin dashboard, or is it internal-only?**

The FEHS score computed in §26.9.1 uses data that is already visible to customers in the admin dashboard (activation rate, weekly active rate). Making FEHS visible to customers would (a) increase transparency and trust, (b) give customers a single headline metric for their health programme, and (c) reduce the CSM time spent explaining raw metrics. Risk: if FEHS drops to Red, a customer seeing their own Red score may panic and escalate before FORM has a recovery plan ready, compressing the intervention window. Alternative: provide a customer-facing "wellness programme health" label (Good / Needs Attention / At Risk) that is coarser than the internal 0–100 score. Owner: product-strategist + customer-success. Priority: **P1 — decide before admin dashboard v2.** Resolution: ship internal-only FEHS at M5; evaluate customer-facing version at M8 after first two QBR cycles.

**OQ-CS-03: What is the right k-anonymity floor for QBR aggregate metrics?**

§26.8.3 states a k=5 floor (at least 5 activated users before any aggregate metric is displayed). This matches the admin dashboard spec in `docs/DATA_MODEL.md §17`. However, for small Starter customers (50 seats, 30% activation = 15 activated users), a k=5 floor means QBR metrics become available from the first cohort. For very small pilots (25 seats, 30% activation = 7–8 activated), k=5 may still be too low for body-metric or health-adjacent aggregates. Recommendation: maintain k=5 for engagement metrics (activation rate, weekly active), but raise to k=10 for any aggregate that touches health-adjacent data (e.g., average workout frequency, recovery scores). Owner: compliance-officer + clinical-safety. Priority: **P1 — before first QBR.** Resolution: compliance-officer to confirm floor with outside counsel for GDPR Art. 9 data categories; update §26.8.3 and admin dashboard spec accordingly.

**OQ-CS-04: Should `fehs_score_at_qbr` in the `enterprise.qbr_completed` event include the individual signal weights?**

The `enterprise.qbr_completed` event records the FEHS score at the time of the QBR but not the breakdown by signal (activation, WAU, seat trend, etc.). Including the signal breakdown in the event payload would enable retrospective analysis of which signal types most reliably predict churn. Excluding it keeps the event payload simpler and reduces the risk of a future schema interpretation error. Risk: without signal-level granularity in the QBR event, the churn prediction model (§26.9.2) can only be calibrated against total FEHS score, not individual signals. Owner: data-engineer. Priority: **P2.** Resolution: include signal weights in a `fehs_breakdown` JSONB field from Day 1 — retrofitting this after 12 months of QBR data is significantly more expensive than including it upfront.

---

*v1.7 additions: §26 Customer Success Team Scaling Economics & CS Cost Model — fills the gap between §8.2's single CSM line-item ($250–500/month) and the full CS team cost, hiring trigger, and gross margin model. §26.1 purpose and privacy floor reminder (CS tooling must enforce aggregate-only constraint identical to admin dashboard; CSM may not access individual health data). §26.2 CS coverage model by tier: Starter email-only $150–200/month shared inbox; Growth named CSM shared 12–15 accounts $250–300/month; Enterprise dedicated CSM max 6 accounts $400–500/month. §26.3 CSM capacity model: 7-activity time budget per account; hard capacity ceiling 15 Growth accounts or 6 Enterprise accounts per CSM; ARR under management $810k–$1M per CSM (within $500k–$1.5M industry benchmark). §26.4 CS team headcount vs. customer count: three stages (founder-led 1–3 accounts; first CS hire M13 4–15 accounts; Series A team 15–50+ accounts); Bull scenario implication that CS hire may precede engineer hire. §26.5 CS cost as % ARR: 30–50% at 5–8 accounts (expected at stage); declining to 12–18% at Series A; reaching 8–12% Series B benchmark at $3M–$5M ARR. §26.6 gross margin impact: per-account CS cost range $375–$725/month depending on team structure; VP CS hire depresses Growth account GM by 7.8 pp until account base absorbs fixed cost; combined CS + compliance cost stack 27–33% ARR at $1M–$1.5M milestone (54–61% GM floor). §26.7 five CS hiring triggers (CS-HIRE-01 through CS-HIRE-05) with anti-trigger against premature VP CS hire. §26.8 QBR economics: fully-loaded QBR cost $262.50 per QBR (7h at $37.50/h CSM rate); QBR cost consistently ~2% ACV across tiers; QBR ROI 3.4× in Year 1 if one logo churn prevented per year; minimum viable QBR content table with privacy floor column (department-level breakdown, individual workout history, body composition, mental health flags all excluded). §26.9 FEHS (FORM Enterprise Health Score): 6-signal composite 0–100 score (activation 25%, WAU 25%, seat utilisation 20%, executive engagement 15%, support volume 10%, renewal distance 5%); four status bands green/yellow/red/critical with CS action SLAs; churn detection ROI 9× on $200–500/month infrastructure cost. §26.10 six DEC-030 HMAC-chained CS events with privacy constraint (aggregate-only, no user_id): enterprise.qbr_completed (STANDARD, 7yr), enterprise.health_score_updated (STANDARD, 3yr), enterprise.churn_signal_flagged (HIGH, 7yr), enterprise.renewal_negotiation_started (STANDARD, 7yr), enterprise.account_expanded (STANDARD, 7yr), enterprise.account_churned (CRITICAL, 7yr); HMAC chain linkage requirement between account_churned and offboarding_initiated events. §26.11 nine-item implementation checklist (5× P0 M5–M6, 3× P1 M6–M8, 1× P2 M6). §26.12 four open questions: OQ-CS-01 (CS hire vs. engineer hire order — P0, Month 9 decision gate), OQ-CS-02 (FEHS customer visibility — P1, admin dashboard v2), OQ-CS-03 (k-anonymity floor for health-adjacent QBR metrics — P1, before first QBR), OQ-CS-04 (signal weights in qbr_completed event — P2, include from Day 1). Cross-references: docs/ENTERPRISE.md (CSM per tier; QBR cadence; privacy floor), docs/DATA_MODEL.md §17 (aggregate-only admin reporting schema), docs/AUDIT_LOG_SCHEMA.md (DEC-030 registry), docs/INCIDENT_RESPONSE.md §12 (enterprise tenant SLA breach protocol), §8.2 (CSM line-item per deal), §8.7 (expansion and churn economics), §22.3 (cash flow — CS hire timing vs. engineer hire), §23.1 (NRR engine), §24.3 (Series A readiness), §25.6 (compliance cost as % ARR — combined with CS cost for total support stack analysis). Owner: customer-success + compliance-officer + data-engineer.*

---

## 27. Engineering Team Cost Model

### 27.1 Purpose and Scope

§26 addressed CS team economics. §27 addresses the other major cost bucket at Series A: engineering. §22.3 Base scenario includes a single "founding engineer" at Month 13 with a $100k/year salary estimate. That figure is a planning placeholder, not a cost model. This section provides:

1. **Founding engineer total compensation cost** including equity dilution, hardware, and onboarding overhead
2. **Technical hiring sequence** — the order in which iOS, backend, ML, and platform roles add the most value per dollar
3. **Infrastructure run-rate costs** by growth stage — what FORM spends on Cloudflare, Supabase, AI API, and observability at 0 / 1,000 / 10,000 / 50,000 users
4. **Build vs. buy analysis** for the seven major third-party services in the FORM stack
5. **Engineering cost as % ARR** trajectory from pre-revenue through Series A
6. **CV/ML pipeline unit economics** — per-session inference cost on-device vs. server
7. **Engineering burn rate gates** — the capital efficiency checkpoints that should gate the next hire

This section is an operating document for the founder and intended as a Series A readiness artefact. All compensation figures are estimates based on 2026 market data for Ukraine-based or remote-friendly engineering talent; ranges are provided because point estimates would be false precision at this stage.

---

### 27.2 Founding Engineer: Total Compensation Cost

The §22.3 "$100k/year" line item covers base salary only. Total first-year cost is higher.

#### 27.2.1 Cash Compensation

| Component | Estimate | Notes |
|---|---|---|
| Base salary (annual) | $80,000–$110,000 | UA-based senior iOS/fullstack: $70k–$90k; remote-competitive (US-matched band): $90k–$120k. $100k midpoint in §22.3 is realistic for UA-based hire with strong iOS CV + Supabase experience. |
| Employer tax / social contributions | $13,000–$18,000 | ~16% of base; varies by employment structure (FOP vs. payroll) |
| Hardware | $3,500–$5,000 | MacBook Pro M4 Max (~$3,200) + iPhone Pro test device (~$1,200) + peripherals. One-time. |
| Onboarding overhead (founder time) | $4,000–$6,000 | ~80h of founder time at an implied opportunity cost rate of $50–75/h — access setup, codebase orientation, pair programming, architecture review. Not a cash cost; a time cost that compresses the founder's other work. |
| Total first-year cash cost | **$100,500–$139,000** | Midpoint: ~$120,000 |

**Practical implication for §22.3 cash flow:** the Base scenario's $100k line undercounts actual first-year cash outflow by $20k–$39k. At pre-seed runway, this is a material correction. The cash flow model should use $115k–$125k for the founding engineer line (base + taxes only, hardware amortised separately).

#### 27.2.2 Equity Compensation

Founding engineer equity is not a cash cost but has dilution implications for cap table planning (§24.7).

| Stage | Typical equity grant | FORM intent |
|---|---|---|
| Pre-seed / pre-product (hire before any revenue) | 1.5%–4.0% (4-year cliff+vest) | 2.5%–3.5% reasonable — justifiable by the technical risk they absorb |
| Seed-stage (hire after first revenue / funding) | 0.5%–1.5% | 1.0%–1.5% at $1.5M–$2.5M seed valuation |
| Series A (hire after round close) | 0.1%–0.5% | ~0.25% typical at $8M–$15M Series A valuation |

**FORM-specific consideration:** the founding engineer is responsible for building the CV pose-estimation pipeline — the technical moat. This is both the hardest hire and the highest-risk role. Equity at the upper end of the pre-seed range (3%–3.5%) is appropriate given that the CV pipeline is the product, not a feature. This creates a ~3.5% dilution event on the cap table — manageable within the §24.7 employee option pool reserved for early hires.

---

### 27.3 Technical Hiring Sequence

Engineering hires create value in a specific order, and getting the order wrong is expensive. The sequence below assumes a pre-seed/seed stage; Series A hiring can parallelise more freely once capital is in.

#### 27.3.1 Hire 1: Founding Engineer (iOS + Supabase) — Month 13 Base / Month 10–11 Bull

**Primary deliverable:** working iOS app with auth, workout logging, Victor AI chat, and CV pipeline integration. Without this hire, nothing ships. This is the gate hire — it unblocks every other milestone.

**Profile requirements:** strong iOS (SwiftUI, HealthKit), Supabase (RLS, Edge Functions), comfort with Python for ML model integration (not ML training — just inference calls to a served model). Bonus: React Native or any cross-platform experience reduces the iOS-only lock-in risk.

**Cost:** $80k–$110k base + $13k–$18k taxes + hardware. See §27.2.1.

#### 27.3.2 Hire 2: Platform / Backend Engineer (Cloudflare Workers + Supabase) — Month 18–22 (post-seed or post-first-ARR)

**Primary deliverable:** FORM's enterprise backend — SSO/SCIM integration (§§23–26 of SSO_SCIM_IMPLEMENTATION.md), audit log HMAC chain, multi-tenant RLS hardening, and Cloudflare Workers edge layer. Currently documented in ~60,000 lines of operational specs. Needs an engineer to implement.

**Profile requirements:** TypeScript, Cloudflare Workers, Supabase advanced (RLS, pg_cron, Edge Functions), familiarity with SAML/OIDC (WorkOS integration). Security mindset required — this engineer touches the SOC 2 surface.

**Cost:** $75k–$100k base. Likely UA-based; this role does not require mobile expertise.

**Trigger:** when the first enterprise contract is signed, or when the founder's time on enterprise backend tasks exceeds 15h/week — whichever comes first.

#### 27.3.3 Hire 3: ML / CV Engineer — Month 24–30 (post-Series A)

**Primary deliverable:** training and fine-tuning the pose-estimation model for the three base lifts (squat, deadlift, bench). The founding engineer integrates inference; this hire trains and improves the model.

**Profile requirements:** PyTorch or TensorFlow, on-device model optimization (Core ML conversion, quantization), familiarity with human pose estimation architectures (MediaPipe, MoveNet, OpenPose, ViTPose). ONNX export workflow. Series A candidate — earlier is pre-mature unless the CV pipeline is the primary bottleneck.

**Cost:** $90k–$130k base. This role is a global competitive hire; expect to pay above UA market rates or hire remotely.

**Trigger:** CV pipeline is live, used by ≥ 200 daily active users, and model quality (e.g., rep-count accuracy, joint angle accuracy) is the #1 user complaint or the primary barrier to upsell.

#### 27.3.4 Hire 4+: Series A Engineering Team (Months 28–36)

At Series A ($1.5M–$2.5M raise, §24.9), target team of 4–6 engineers: Founding Engineer (tech lead / iOS), Platform Engineer (Hire 2), ML Engineer (Hire 3), a second iOS/Android engineer, and a data engineer to own the Metabase/analytics warehouse pipeline. VP Engineering is a Series B hire.

| Role | Headcount | Annual cost (salary + taxes) |
|---|---|---|
| Founding Engineer / Tech Lead | 1 | $95k–$125k |
| Platform / Backend Engineer | 1 | $80k–$105k |
| ML / CV Engineer | 1 | $95k–$135k |
| iOS or Android Engineer | 1 | $75k–$100k |
| Data Engineer | 1 | $70k–$90k |
| **Total Series A engineering payroll** | **5** | **$415k–$555k/year** |

At $1.5M ARR (§22.3 Base scenario at Series A), engineering payroll = 28%–37% ARR — within the 25–40% benchmark for B2B SaaS pre-Series B. At $3M ARR (Bull scenario at Series A close), it compresses to 14%–18%.

---

### 27.4 Infrastructure Run-Rate Costs

FORM's infrastructure is designed on a serverless-first, usage-based pricing model. Pre-revenue costs are minimal; cost scales with users.

#### 27.4.1 Core Infrastructure Cost Table

| Service | Pre-revenue (0 users) | 1,000 MAU | 10,000 MAU | 50,000 MAU | Notes |
|---|---|---|---|---|---|
| Cloudflare Workers (API layer) | $5/mo (paid plan) | $10–20/mo | $40–80/mo | $150–300/mo | Includes D1 database requests; most edge logic runs in free tier until 10M requests/day |
| Supabase Pro | $25/mo | $25/mo | $25–100/mo (compute scale) | $599/mo (Enterprise) | Pro plan: 1 project + 8GB DB + 100GB bandwidth. Enterprise triggers at SOC 2 requirement (§26.3 SSO_SCIM_IMPLEMENTATION) |
| Cloudflare R2 (storage — video, compliance vault) | $0 (10GB free) | $2–5/mo | $15–40/mo | $75–180/mo | $0.015/GB stored + $0.36/million Class B ops |
| Anthropic API (Victor AI coaching) | $0 | $30–80/mo | $300–800/mo | $1,500–3,500/mo | Estimated at 3–5 coaching turns/session, 3 sessions/week per user; Claude Haiku at ~$0.00025/1K input tokens + $0.00125/1K output tokens |
| Sentry (error tracking) | $26/mo | $26/mo | $89/mo (Team) | $89/mo | Capped at Team plan for most of FORM's scale |
| PagerDuty (incident alerting) | $21/mo (2 users) | $21/mo | $63–105/mo (3–5 users) | $105–210/mo | Professional plan $21/user/month |
| WorkOS (SSO/SCIM) | $0 (free tier ≤100 enterprise connections) | $0–125/mo | $125–250/mo | $250–800/mo | Free up to 100 enterprise connections; $125/month Starter for SSO; Enterprise pricing negotiable at scale |
| Better Stack (uptime + log management) | $0 (free tier) | $20/mo | $40–60/mo | $150–300/mo | Scales with log ingestion volume |
| Metabase (BI / dashboards) | $0 (self-hosted) | $0 | $0–500/mo (Cloud) | $500/mo | Self-hosted free on Cloudflare Workers or EC2 micro; Cloud plan at 10k MAU if ops overhead too high |
| **Total infrastructure** | **~$77/mo** | **~$140–280/mo** | **~$680–1,300/mo** | **~$2,800–5,500/mo** | |

**Pre-revenue infrastructure cost: ~$77/month.** This is the minimum viable tech stack for a production-grade B2B SaaS with SOC 2 trajectory. It is not a "bootstrap on free tiers" stack — it includes PagerDuty, Sentry, and Cloudflare Workers paid plans from Day 1 because those are required for enterprise-grade reliability commitments.

**Anthropic API at scale:** at 50,000 MAU, AI coaching is the largest infrastructure line item ($1,500–$3,500/month). This is a product-differentiated cost (it provides Victor's coaching intelligence) and should be modelled as a COGS line, not pure infrastructure overhead. At the $79/month consumer price point, a user paying $79/month and consuming $0.10–0.15/month in Anthropic API costs represents a >500× revenue-to-API-cost ratio — extremely healthy.

#### 27.4.2 Infrastructure as % of Revenue

| ARR Stage | Monthly infra cost | Monthly ARR equivalent | Infra as % MRR |
|---|---|---|---|
| Pre-revenue | ~$77/mo | $0 | N/A (fixed cost below $1k/year) |
| $100k ARR | ~$200–400/mo | ~$8,300/mo | 2.4%–4.8% |
| $500k ARR | ~$800–1,500/mo | ~$41,700/mo | 1.9%–3.6% |
| $1M ARR | ~$1,500–2,500/mo | ~$83,300/mo | 1.8%–3.0% |
| $3M ARR | ~$4,000–7,000/mo | ~$250,000/mo | 1.6%–2.8% |

Infrastructure cost as % MRR compresses steadily — standard SaaS characteristic. Infrastructure is not the gross margin problem; engineering payroll and CS payroll are.

---

### 27.5 Build vs. Buy Analysis

Seven major third-party services in the FORM stack. Each represents a buy decision that was (or should be) evaluated against an in-house build.

| Service | Monthly cost | Build cost estimate | Verdict | Rationale |
|---|---|---|---|---|
| **WorkOS** (SSO/SCIM/Directory Sync) | $125–800/mo | 6–10 engineering weeks (~$15k–$25k once; $5k–$10k/year maintenance) | **BUY** | SAML/OIDC and SCIM spec compliance is non-trivial, especially for enterprise IdP edge cases (Okta, Azure AD, Google Workspace quirks). WorkOS is the established standard; rebuilding provides zero differentiation. |
| **Supabase** (Postgres + Auth + RLS + Edge Functions) | $25–599/mo | $40k–$80k/year (DBA + infra engineer time) | **BUY** | Self-managed Postgres at enterprise scale with SOC 2 RLS requirements is a full-time infrastructure job. Supabase provides the managed Postgres, pgvector, Row Level Security enforcement, and Auth stack FORM needs. The build-vs-buy line only becomes relevant above $5M ARR when Supabase Enterprise pricing exceeds a dedicated DBA hire cost. |
| **Anthropic API** (Victor AI coaching) | $30–3,500/mo (usage-based) | Not applicable — no alternative; fine-tuning open-source models is $50k+/year in ML infra | **BUY** | The coaching intelligence is the product. Until FORM has the data volume and ML team to fine-tune a proprietary model, API is the only realistic path. Evaluate own-model cost at Series B when MAU > 100k. |
| **PagerDuty** (incident alerting) | $21–210/mo | 2–4 engineering weeks (~$5k–$10k) | **BUY** | On-call routing, escalation chains, and on-call scheduling are not FORM's moat. PagerDuty is the category standard for SOC 2-auditable incident response (INCIDENT_RESPONSE.md R-* runbooks reference PagerDuty throughout). |
| **Sentry** (error tracking + performance) | $26–89/mo | 1–2 engineering weeks ($2.5k–$5k) | **BUY** | Stack traces, source maps, session replay, and performance profiling are well-solved problems. Building in-house adds no differentiation and costs engineering time that should be on the CV pipeline. |
| **CV pose estimation model** (on-device inference) | $0/session (on-device via Core ML) | $30k–$80k initial (model training + Core ML conversion); $10k–$20k/year (maintenance + retraining) | **BUILD** | This is FORM's technical moat. On-device inference is a privacy and latency requirement (no video leaves the device). Dependency on a third-party CV API would (a) create a cloud egress cost at scale, (b) introduce a privacy liability for GDPR/HIPAA-adjacent health data, (c) eliminate the hardware partnership narrative. The build cost is high but non-negotiable for the moat thesis. |
| **Better Stack** (uptime monitoring + log management) | $0–300/mo | 1 engineering week ($2.5k) | **BUY** | Synthetic uptime monitoring, log aggregation, and incident status pages are commodity tooling. Better Stack's free tier covers pre-revenue needs. |

**Build vs. Buy principle:** FORM builds what is the product (CV pipeline, Victor's coaching logic) and buys what is infrastructure. No differentiation comes from building your own SSO, error tracking, or alerting stack. Every in-house build of a commodity service is a hidden tax on engineering velocity.

---

### 27.6 CV / ML Pipeline Unit Economics

The CV pipeline is the most cost-opaque component of the FORM stack. On-device inference eliminates per-session server cost at the expense of model size constraints and device compatibility.

#### 27.6.1 On-Device Inference (Core ML)

| Metric | Value | Source / Notes |
|---|---|---|
| Per-session cost | ~$0 marginal | Inference runs on device; no cloud GPU or API call |
| Device requirement | iPhone 12+ (A14 Bionic Neural Engine) | Core ML quantized models run efficiently on A14+; acceptable on A13 with reduced frame rate |
| Model size on-device | 15–40 MB (INT8 quantized) | MoveNet Lightning = 9MB; ViTPose-S INT8 = ~25MB; custom 3-lift fine-tuned = 20–35MB estimate |
| Latency (P95, live camera) | 25–50 ms/frame (A15+) | 30 fps tracking requires < 33ms/frame; achievable on A15+, marginal on A13 |
| Battery impact | +8–15% per hour of tracking | Neural Engine activation vs. CPU-only; acceptable for 45–90 min session |
| Infrastructure cost scaling | Linear $0 | MAU growth does not increase server cost for inference |

**Implication:** at 50,000 MAU doing 3 tracking sessions/week, on-device inference costs FORM $0 in server compute. This is the core economics of the CV moat — it is a cost floor advantage, not just a privacy story.

#### 27.6.2 Server-Side Inference (Hypothetical — not FORM's current path)

Included for comparison only to quantify the avoided cost:

| Metric | Value |
|---|---|
| GPU instance for real-time pose inference | $0.40–$0.90/hour (e.g., AWS g4dn.xlarge at spot pricing) |
| Sessions per GPU-hour (concurrent inference) | ~30–50 concurrent streams |
| Per-session-hour cost | ~$0.009–$0.030 |
| At 50,000 MAU × 3 sessions/week × 1h/session | ~$108k–$360k/year |

On-device inference eliminates $108k–$360k/year of GPU compute cost at 50,000 MAU. This is both a gross margin advantage and a competitive moat: any competitor using cloud-side inference faces this cost structure at scale.

---

### 27.7 Engineering Cost as % ARR at Scale Milestones

Combines engineering payroll (§27.3) and infrastructure (§27.4) into a total engineering cost line.

| ARR Milestone | Engineering payroll (annual) | Infrastructure (annual) | Total engineering cost | Engineering as % ARR |
|---|---|---|---|---|
| Pre-revenue | $0 | ~$924 (~$77/mo) | ~$1k | N/A |
| $100k ARR (first enterprise customers) | $80k–$110k (1 engineer) | ~$3k–$5k | ~$83k–$115k | **83%–115%** |
| $500k ARR (seed traction) | $160k–$220k (2 engineers) | ~$10k–$18k | ~$170k–$238k | **34%–48%** |
| $1M ARR (Series A gate) | $240k–$330k (3 engineers) | ~$18k–$30k | ~$258k–$360k | **26%–36%** |
| $3M ARR (Series A scale) | $415k–$555k (5 engineers) | ~$50k–$84k | ~$465k–$639k | **16%–21%** |

**The $100k ARR engineering cost ratio (83%–115%) looks alarming** but is expected at this stage: FORM has not yet hired an engineer, so the first hire is a step-function cost before the revenue curve catches up. The correct way to read this milestone: "engineering cost at $100k ARR is front-loaded; the ratio normalises as the second enterprise customer converts."

The trajectory from 34%–48% at $500k ARR to 16%–21% at $3M ARR is the operating leverage story for Series A investors: the cost structure is fixed-heavy and compresses cleanly as revenue scales.

---

### 27.8 Engineering Burn Rate Gates

These are the checkpoints that should trigger (or block) the next engineering hire. They connect §27.3 hiring sequence to the §22.3 cash flow model.

| Gate | Trigger Condition | Hire Unlocked | Block Condition (don't hire if:) |
|---|---|---|---|
| **ENG-GATE-01** | First design prototype validated with ≥ 5 user interviews; investor pre-commitment of ≥ $300k exists | Founding Engineer (Hire 1) | No validated use case; no capital to fund the first 12 months of salary + taxes |
| **ENG-GATE-02** | First enterprise contract signed OR LOI from ≥ 2 enterprise prospects AND ARR ≥ $50k | Platform Engineer (Hire 2) | ARR < $50k AND no signed enterprise contract; founder still has capacity to handle enterprise backend |
| **ENG-GATE-03** | CV pipeline live with ≥ 200 DAU using tracking AND model accuracy is the #1 user feedback theme AND Series A round closed | ML/CV Engineer (Hire 3) | CV pipeline not yet live; model accuracy not yet a user-facing bottleneck |
| **ENG-GATE-04** | ARR ≥ $1M (Series A close) AND team has 3+ engineers AND data pipeline is the analytics bottleneck | Data Engineer (Hire 4) | Team < 3 engineers; data work can still be done by platform engineer part-time |

**Anti-pattern to avoid:** hiring an ML engineer before the product is in users' hands. The most common early-stage engineering mistake is optimising the CV model before validating that the product loop (auth → workout logging → Victor chat) retains users. Model quality matters only after product-market fit.

---

### 27.9 DEC-030 Engineering Spend Audit Events

Engineering spend audit events are required for two purposes: (1) investor data room — evidence that engineering costs are tracked against milestones; (2) internal governance — prevent uncontrolled infrastructure cost escalation as user base grows.

| Event type | Severity | Key metadata fields | Retention | Trigger |
|---|---|---|---|---|
| `ops.engineer_hired` | STANDARD | `hire_type` (founding / platform / ml / data / other), `start_date`, `annual_base_usd`, `equity_pct`, `hired_by` (founder internal ID) | 7 years | On first day of employment; emitted by founder via admin tooling |
| `ops.infra_cost_threshold_crossed` | STANDARD | `threshold_usd_monthly` (100 / 500 / 1000 / 5000), `actual_usd_monthly`, `primary_driver` (service name), `month` | 3 years | When monthly infrastructure spend crosses the next $100/$500/$1,000/$5,000 tier; emitted by monthly cost reconciliation cron |
| `ops.build_vs_buy_decision` | STANDARD | `service_name`, `decision` (build / buy / defer), `rationale_summary` (max 200 chars), `owner` (internal ID), `annual_cost_usd` (buy) or `build_cost_estimate_usd` (build) | 7 years | Whenever a significant build vs. buy decision is made (>$5k/year impact); emitted by founder or tech lead via admin tooling |

---

### 27.10 Implementation Checklist

| Item | Priority | Milestone | Owner | Definition of Done |
|---|---|---|---|---|
| Update §22.3 cash flow model to use $115k–$125k for founding engineer (base + taxes, §27.2.1), replacing the current $100k placeholder | P0 | Before next investor conversation | founder + data-engineer | §22.3 updated; investor deck cash flow slide updated to match |
| Open engineer search with JD on Djinni + Wellfound: Founding Engineer (iOS + Supabase + CV integration); equity 2.5%–3.5%, base $80k–$110k | P0 | M1 (open immediately) | founder | Job post live on at least two platforms; first screening calls scheduled |
| Define ENG-GATE-01 criteria (§27.8) as a formal decision gate in the OKR doc and Notion board; assign monthly review checkpoint | P1 | M2 | founder + product-manager | Gate criteria documented in OKRS_2026.md; first gate review scheduled |
| Add engineering infrastructure cost tracking to monthly metrics review: pull Cloudflare, Supabase, Anthropic API costs monthly; file in `ops.infra_cost_threshold_crossed` DEC-030 event format | P1 | M3 | data-engineer + devops-lead | Monthly cost report template created; first export filed |
| Register three DEC-030 engineering spend events (`ops.engineer_hired`, `ops.infra_cost_threshold_crossed`, `ops.build_vs_buy_decision`) in `docs/AUDIT_LOG_SCHEMA.md` with full Zod schema | P1 | M4 | platform-engineer + compliance-officer | All three events registered in AUDIT_LOG_SCHEMA.md |
| Document first formal build vs. buy decision log (CV pipeline BUILD decision): emit `ops.build_vs_buy_decision` event with rationale; cross-reference docs/TECHNICAL.md CV pipeline section | P2 | M3 | founder | Event emitted or logged; §27.5 CV BUILD rationale confirmed correct |

---

### 27.11 Open Questions

**OQ-ENG-01: Should the founding engineer be a UA-based employee or a US-incorporated contractor?**

UA-based employment: lower base cost ($70k–$90k), local hiring network, lower employer overhead. Risk: FX exposure on UAH salaries (engineers typically expect USD), wartime talent flight risk, complexity of termination if needed. US contractor structure: higher gross cost ($90k–$130k IRS 1099 equivalent), simpler for a US-incorporated parent company, no termination complexity. Recommendation: hire as UA FOP (self-employed contractor) invoicing monthly in USD — standard structure for UA tech startups, avoids payroll overhead, and allows conversion to employment if Series A is US-structured. Owner: founder + legal. Priority: **P0 — must be resolved before first offer letter.** Resolution: confirm with Ukrainian employment counsel and US incorporation counsel before making the offer.

**OQ-ENG-02: What is the minimum viable Founding Engineer profile — iOS-specialist or fullstack?**

iOS-specialist profile (SwiftUI-only) has higher supply in the UA market at lower cost but creates a backend dependency on the founder. Fullstack profile (iOS + Supabase + Cloudflare Workers) reduces the dependency but is a harder and more expensive hire. The risk of a pure iOS hire: the founder becomes the backend engineer by default, which is a sustainable strategy only if the founder has the Supabase/Workers depth. If not, the iOS-only hire delays backend work until Hire 2. Recommendation: minimum viable profile is iOS + Supabase Edge Functions + TypeScript; pure iOS-only is a gap that creates roadmap risk. Owner: founder. Priority: **P1 — before finalising the JD.** Resolution: founder self-assesses Supabase/backend depth; if strong, iOS-specialist acceptable; if weak, require Supabase in the JD.

**OQ-ENG-03: When does on-device CV inference become a device-compatibility liability?**

The §27.6.1 analysis assumes iPhone 12+ (A14) as the minimum. As of 2026, ~35%–40% of iOS devices are iPhone 12 or newer (global) vs. ~55%–60% in the core FORM market (Ukraine + Western EU, higher iPhone age profile). A device requirement floor of iPhone 12 excludes ~40%–45% of the global iOS user base. Options: (a) maintain A14 floor and accept the addressable market constraint; (b) ship a reduced-feature mode (no live CV tracking, manual logging only) for A13 and older; (c) invest in heavier model quantisation to bring inference to A13. Owner: ml-engineer + platform-engineer. Priority: **P2 — evaluate at TestFlight beta when device distribution data is available.** Resolution: instrument device model in analytics from Day 1; make the decision with real data at the 1,000 beta user mark.

---

---

## 28. Marketing & Demand Generation Cost Model

### 28.1 Purpose and Scope

§27 addressed engineering team economics. §28 addresses the last major cost bucket captured in §22.3's "Marketing / UA" line that has not yet been decomposed: marketing and demand generation. §22.3 allocates $500–$2,000/month to marketing but does not model what that budget buys, how it should be allocated across channels, when the first dedicated marketing hire becomes economical, or how FORM's S&M-as-%-ARR compares to B2B SaaS benchmarks.

This section provides:

1. A marketing cost taxonomy that maps each spend type to COGS vs. S&M classification for investor reporting
2. A channel-level budget model for pre-launch, post-launch consumer, and enterprise demand gen phases
3. App Store Optimization investment and ongoing cost
4. Consumer paid acquisition economics with hard-ceiling logic tied to §14.5 LTV:CAC floors
5. Enterprise demand generation spend — what founder-led sales costs in content and tooling before the first AE (§19.4)
6. A marketing tooling stack with monthly cost
7. First marketing hire economics: timing gates and compensation model
8. S&M as % ARR trajectory benchmarked against Bessemer and SaaS Capital industry data
9. Magic number calculation operationalising the §18.7 target > 0.75
10. DEC-030 audit events for marketing spend governance
11. Implementation checklist and open questions

**Scope boundaries:**
- **Not in scope:** enterprise CAC per deal (§8.5), consumer CAC payback curves (§14.4–14.5), enterprise GTM pipeline revenue model (§19). Those sections model what marketing *produces*; this section models what marketing *costs*.
- **Privacy floor:** marketing analytics must never re-identify individual users. PostHog cohort reporting uses `distinct_id` only (§25 Product & Activation Funnel Observability); attribution models operate on channel-level aggregates, not user-level join to health data.

---

### 28.2 Marketing Cost Taxonomy

**COGS vs. S&M classification** for P&L and investor reporting:

| Cost item | Classification | Rationale |
|---|---|---|
| Paid acquisition (Meta, ASA, TikTok ads) | S&M | Direct acquisition spend; excluded from gross margin calculation |
| Content production (sports science blog, video) | S&M | Demand generation; not a component of service delivery |
| ASO asset creation (screenshots, preview video) | S&M (one-time) | Capitalised equivalent; amortise over 12 months for management accounts |
| Marketing tooling (email, CRM, analytics) | S&M (fixed) | Support function for acquisition and retention |
| PR and earned media | S&M | Brand and demand gen |
| Customer referral credit / incentive | S&M | Acquisition incentive, not COGS |
| Brand design (logo, brand book) | COGS-adjacent (shared) | Primarily product delivery surface; split 80/20 S&M/COGS is acceptable |
| In-app onboarding copy creation | COGS | Part of service delivery; not demand gen |

**Consumer vs. enterprise marketing cost split:** At pre-PMF (Months 1–12), the vast majority of marketing spend is consumer-facing (content, ASO, limited paid). Enterprise demand gen at this stage is ~100% founder time (no incremental cash spend beyond tools already paid). The split inverts post-Series A when an AE and a demand gen function are funded.

---

### 28.3 Pre-Launch Marketing Budget (Months 1–4)

The §22.3 Base scenario allocates the following to Marketing / UA before App Store launch:

| Month | §22.3 Marketing/UA allocation | Primary uses |
|---|---|---|
| M1 | $500 | Waitlist landing page copywriting; basic social presence setup; domain and hosting already in §22.3 Infra |
| M2 | $500 | First long-form content piece (sports science); social scheduling tool; email marketing tool (Loops or Resend, ~$0 at pre-launch volume) |
| M3 | $1,000 | App Store screenshot design (Figma); preview video raw footage edit; TestFlight waitlist email sequence setup; second long-form content piece |
| M4 | $1,500 | App Store submission assets finalised; launch-day social content batch; first paid Apple Search Ads experiment ($200–$400 test) |

**Total pre-launch marketing cash: $3,500 [ESTIMATE] (Months 1–4).**

These are cash costs only. Founder time on content creation (estimated 10–15 hours/week) is the dominant pre-launch marketing input and is not a cash line. The content-strategist agent coordinates the editorial calendar and does not add direct cash cost at this stage.

---

### 28.4 App Store Optimization (ASO) Investment

ASO is a one-time investment (creative assets) plus an ongoing iteration budget. It is classified as S&M — it directly drives organic installs at no per-install marginal cost.

**Initial ASO asset creation [ESTIMATE]:**

| Asset | Internal (founder) | External (contractor) | Notes |
|---|---|---|---|
| Screenshots (6 iOS + 6 Android) | 8h Figma work | $300–600 | Use brand-system templates; do not outsource first version |
| App preview video (30 sec) | 4h edit | $400–800 | Screen record on device + CapCut/DaVinci; avoid agency at pre-seed |
| App icon (already in brand-system) | Included in brand-system scope | — | Not a separate marketing cost |
| Metadata copy (title, subtitle, keywords, description) | 3h founder + brand-voice | $0 | brand-voice agent; no external copywriter needed |
| Localisation (UA + EN) | 2h per locale | $0 | Native founder + brand-voice agent covers UA/EN |

**Total initial ASO investment: $700–$1,400 external cost [ESTIMATE]; 17h founder time [ESTIMATE].**

**Ongoing ASO iteration budget:** $0–$200/month [ESTIMATE] for A/B testing via App Store Connect native experiments + occasional screenshot refresh. At pre-seed scale, founder handles this directly. This is already embedded in the §22.3 Marketing/UA line.

**ASO expected return:** Health & Fitness category organic conversion rate from Product Page View to install: 3–8% [ESTIMATE, Apple App Store benchmark]. Optimised screenshots + preview video can improve this to 8–12%. At 1,000 monthly product page views (realistic at 6-month organic traction), the delta between 5% and 10% CVR is 50 incremental installs/month at $0 marginal cost — equivalent to acquiring those users at $0 CAC. This is the highest-leverage pre-launch marketing action.

---

### 28.5 Consumer Paid Acquisition Economics

FORM's consumer paid acquisition strategy is deliberately constrained by the §14.5 LTV:CAC floors. This section models the budget envelope for paid channels given those floors.

**Channel benchmarks for Health & Fitness app category [ESTIMATE — replace with actual PostHog attribution data after 3 months of paid spend]:**

| Channel | CPI (cost per install) | Install-to-Pro conversion | Effective CAC | LTV:CAC at M24 (§14.5 model) | FORM verdict |
|---|---|---|---|---|---|
| Apple Search Ads (brand + competitor) | $2–$6 | 12–18% | **$17–$50** | 3.9×–11.4× | Tier 1 — always-on once App Store live |
| Apple Search Ads (category / discovery) | $4–$10 | 5–10% | **$40–$200** | 1.0×–4.9× | Use only if CAC < $80; pause if > $80 |
| Meta (Facebook/Instagram Reels) | $3–$8 | 3–7% | **$43–$267** | 0.7×–4.5× | Only above §14.5 minimum (≥ 3× at M24); requires 90-day attribution window |
| TikTok (fitness/athlete creators) | $2–$6 | 2–5% | **$40–$300** | 0.7×–4.9× | High variance; test with $300 burst; pause if CAC > $100 |
| Organic content / SEO (sports science blog) | ~$15–$30 (content creation amortized) | n/a — drives brand search | **$15–$30 [ESTIMATE]** | ≥ 5× (higher-intent users) | Tier 1 — prioritise over all paid |
| Referral (existing Pro user invite) | $5–$15 (referral credit cost) | ~60% (warm install) | **$8–$25** | ≥ 6× | Build referral loop before scaling paid |

**Hard ceiling on paid UA:** Do not commit to > $2,000/month in paid acquisition before validating D30 retention ≥ 40% [ESTIMATE]. The §22.3 Marketing/UA line is capped at $2,000/month through Month 18 precisely because: (a) the LTV model is unproven until the first cohort reaches M12, and (b) paid acquisition at poor retention burns pre-seed capital with no return.

**Paid UA activation gate (MKT-UA-GATE-01):** Unlock paid acquisition budget above $1,000/month only when:
- App Store live (not TestFlight)
- D30 retention ≥ 35% on a cohort of ≥ 100 users [ESTIMATE]
- Install-to-Pro conversion rate measured on ≥ 50 organic installs

Until MKT-UA-GATE-01 is green, cap Marketing/UA at $1,000/month (ASO + content production only).

---

### 28.6 Enterprise Demand Generation Budget

Enterprise demand gen at the pre-PMF founder-led phase (§19.3, Deals 0–3) is almost entirely founder time — not discretionary cash spend. This is structurally different from consumer marketing.

**Founder-led enterprise demand gen cash cost (Months 1–18) [ESTIMATE]:**

| Activity | Monthly cash cost | Owner | Notes |
|---|---|---|---|
| LinkedIn thought leadership content | $0 (founder writes) | founder + content-strategist | Minimum 3 posts/week; content-strategist agent drafts; no cash cost |
| Long-form content (enterprise use cases, buyer guides) | $100–$200/month | brand-voice + content-strategist | Design/layout only; zero copywriting cash cost at this stage |
| Sales enablement collateral (one-pager, ROI calculator) | $200–$400 one-time | design-craft agent | Figma; no external designer at pre-seed |
| CRM tooling (HubSpot Free → Starter) | $0–$45/month | founder | Free tier adequate for < 10 active enterprise prospects; upgrade to Starter ($45/month) at 10+ prospects |
| Events and conferences | $0–$500/event (selective) | founder | Attend 2–3 relevant HR tech / wellness events per year; budget $500 max per event for registration; travel from pre-seed capital if UA-based |
| Cold outbound (email sequences, LinkedIn DMs) | $0–$60/month (Apollo.io or similar) | founder | Avoid at pre-seed — only if outbound is clearly faster than inbound; test with free tier first |

**Total enterprise demand gen cash: $300–$1,200/month [ESTIMATE] at pre-seed, depending on conference activity.**

This is included in the §22.3 Marketing/UA line alongside consumer marketing. The combined $1,500–$2,000/month envelope is shared across consumer and enterprise: approximately $500–$800 enterprise demand gen + $700–$1,200 consumer marketing in the Month 4–12 phase.

**Post-Series A enterprise demand gen shift:** Once an AE is hired (§19.4, Month ~24), the demand gen model shifts materially:

| Activity | Monthly cash cost (post-AE) | Notes |
|---|---|---|
| AE outbound tools (Outreach/Salesloft) | $100–$150/seat | Required for systematic outbound sequencing |
| ABM ad spend (LinkedIn targeted at HR/VP People at ICP companies) | $1,500–$5,000/month | LinkedIn CPM ~$15–$35 for targeted B2B; start with $1,500 and measure pipeline contribution |
| Content syndication (G2, Capterra listing) | $0–$300/month | Free listings first; paid placement only if inbound lead quality is verified |
| Demand gen events (HR Tech, SHRM) | $2,000–$5,000/event (booth + travel) | Budget 2 events/year at Series A; $4,000–$10,000/year |
| Content marketing agency (case studies, reports) | $2,000–$4,000/month | Only at Series A; founder-led content is the correct strategy pre-PMF |

**Post-Series A enterprise demand gen: $7,600–$19,450/month [ESTIMATE].** This is an order of magnitude above the pre-seed level and must be modeled separately in the Series A budget (not in §22.3 which covers Months 1–18 only).

---

### 28.7 Marketing Tooling Stack

The marketing tooling stack at each stage:

**Pre-seed / pre-launch (Months 1–6) [ESTIMATE]:**

| Tool | Purpose | Cost/month | Tier |
|---|---|---|---|
| Figma (already in engineering stack) | Design assets, landing pages, ASO screenshots | $0 (free) or $15 (Pro) | Free adequate |
| Canva | Social media templates | $0 (free) | Free adequate |
| Loops or Resend | Transactional + marketing email | $0 (free tier up to 1,000 contacts) | Free adequate |
| PostHog (already in product stack) | Attribution analytics, UTM tracking, funnel analysis | $0 (free up to 1M events) | Already paid |
| Buffer or Later | Social media scheduling | $0–$15/month | Free adequate pre-launch |
| Notion | Content calendar, editorial planning | $0–$8/month | Free adequate |
| HubSpot CRM | Enterprise prospect tracking | $0 (free CRM) | Free adequate to 10 prospects |
| **Total marketing tooling** | | **$0–$38/month** | |

**Post-launch / scaling (Months 7–18) [ESTIMATE]:**

| Tool | Purpose | Cost/month | Upgrade trigger |
|---|---|---|---|
| Loops (paid) or Brevo | Email sequences, waitlist, onboarding drip | $15–$35 | > 1,000 email subscribers |
| Apple Search Ads account | Paid UA (ASA) | Variable (ad spend) | App Store launch |
| Meta Ads Manager | Paid UA | Variable (ad spend) | MKT-UA-GATE-01 unlocked |
| Ahrefs or Semrush (Lite) | SEO + content keyword research | $19–$99/month | First long-form content series |
| HubSpot Starter | Enterprise CRM + email sequences | $45/month | > 10 active enterprise prospects |
| **Total tooling (excl. ad spend)** | | **$79–$179/month [ESTIMATE]** | |

**Series A marketing tooling (not in §22.3 scope) [ESTIMATE]:**

Outreach/Salesloft ($100–$150/seat), LinkedIn Sales Navigator ($80–$100/seat), Clearbit or Apollo.io ($150–$400/month), G2 Profile Management ($0 listing + optional paid), Notion AI ($16/month) — totalling $346–$666/month in tooling before ad spend.

---

### 28.8 First Marketing Hire Economics & Hiring Gates

**Hiring gates:** The first dedicated marketing hire should not precede consumer PMF. Hiring a marketing hire before the product loop (auth → workout logging → Victor interaction → habit formation) is validated wastes salary on a product that has not yet earned organic traction.

**Profile options and cost [ESTIMATE]:**

| Profile | Base salary (UA-based) | Skills needed | Wrong hire if: |
|---|---|---|---|
| Content / Growth Writer | $25k–$40k | Long-form sports science, SEO, social | You need paid acquisition expertise or enterprise demand gen; this profile cannot own B2B marketing |
| Growth Marketer | $40k–$60k | Paid acquisition, ASO, analytics, A/B testing | You do not yet have D30 retention data to optimise against; no paid UA without retention proof |
| Product Marketing Manager (PMM) | $50k–$75k | Positioning, messaging, sales enablement, competitive | You are not yet ready for enterprise sales; this profile shines at the AE-hire stage (Month 22+) |
| Head of Marketing / VP Marketing | $70k–$100k | Manages all of the above | Pre-Series A VP Marketing is usually wrong; hire a specialist, not a generalist manager |

**FORM recommended first hire: Growth Marketer at Month 18–22 [ESTIMATE], post-consumer-PMF confirmation.**

Trigger criteria (MKT-HIRE-01):
- D30 retention ≥ 40% on ≥ 500 users [ESTIMATE]
- App Store rating ≥ 4.5 on ≥ 50 reviews
- Organic install rate > 500 installs/month without paid UA
- ARR ≥ $50k (consumer + early enterprise combined)
- Founder is spending > 8h/week on marketing tasks

Block condition (do not hire if): Paid UA is the primary growth lever being tested but D30 retention < 35% — a Growth Marketer cannot fix retention; that is a product problem.

**MKT-HIRE-02 (PMM, post-AE-hire):** Triggered by first AE hired (§19.4). PMM's first deliverable is the enterprise sales deck and ROI calculator that the AE will use. This hire should be concurrent with or follow the AE by 1–2 months, not precede it.

**MKT-HIRE-03 (Head of Marketing, post-Series A):** Triggered when marketing team reaches 2–3 specialists with no coordination function. Series A budget should include this role at $70k–$90k [ESTIMATE] (UA rate; US-equivalents at $130k–$160k).

**Cash cost impact on gross margin:** Unlike CS (§26.6) and engineering (§27.7), marketing hires are 100% S&M — they do not affect gross margin directly. Their impact is on net income / operating margin. Track marketing headcount cost as % of ARR, not as a COGS line.

---

### 28.9 S&M as % ARR: Industry Benchmarks vs. FORM Targets

**Industry benchmarks [SaaS Capital / Bessemer Cloud Index / BVP State of the Cloud]:**

| ARR milestone | Median S&M as % ARR | Best-in-class S&M as % ARR | Notes |
|---|---|---|---|
| < $1M ARR | 50–80% | 30–50% | Founder-led; high initial investment relative to revenue |
| $1M–$5M ARR | 40–60% | 25–40% | First AE + growth marketer hired |
| $5M–$20M ARR | 30–45% | 20–30% | Pipeline engine established |
| $20M+ ARR | 20–35% | 15–25% | Series B+ scale efficiency |

**FORM targets [ESTIMATE — replace with actuals at each milestone]:**

| ARR milestone | FORM S&M target | Composition | Rationale |
|---|---|---|---|
| Pre-revenue → $100k ARR | 40–60% of ARR | $1,500–$2,000/month (§22.3 Marketing/UA); no marketing headcount | Consumer content + early ASO; below median because no paid AE yet |
| $100k–$500k ARR | 35–50% of ARR | Marketing/UA $2,000/month + Growth Marketer (MKT-HIRE-01) ~$4,200/month | First hire drives LTV:CAC improvement; paid UA experiments begin |
| $500k–$1M ARR | 30–40% of ARR | Marketing team + AE onboarding (§19.4) + enterprise demand gen tools | S&M spend grows but ARR growing faster; efficiency should hold |
| $1M–$3M ARR | 25–35% of ARR | Full marketing function (Growth + PMM); ABM spend; two AEs (§19.5) | Scale phase; magic number (§28.10) must stay > 0.75 to justify spend |

**Combined S&M + G&A target at Series A:** ≤ 50% of ARR. If S&M alone reaches 40% at $1M ARR, G&A must be kept below 10% of ARR — achievable at this stage with a lean back-office.

---

### 28.10 Magic Number Model

The magic number measures sales and marketing efficiency: how much net new ARR does each dollar of S&M spend generate?

**Formula:**

```
Magic Number = (Net New ARR in quarter Q) ÷ (S&M spend in quarter Q−1)
```

**Benchmark targets (§18.7):**
- > 0.75: Efficient — S&M investment is working; safe to increase spend
- 0.5–0.75: Acceptable — monitor for improvement; do not dramatically increase S&M
- < 0.5: Inefficient — diagnose before adding spend; likely a retention or conversion problem, not a volume problem

**FORM magic number model at key milestones [ESTIMATE]:**

| Quarter | Net New ARR | Prior-quarter S&M spend | Magic Number | Interpretation |
|---|---|---|---|---|
| Q4 Year 1 (launch + 2 enterprise deals) | $28,800 [§19.7 Base — Q4 enterprise cash] | $4,500 (3 months × $1,500/month marketing) | **6.4×** | Anomalously high — enterprise deals dominate; consumer revenue is negligible |
| Q2 Year 2 (post-launch growth) | $12,000 [ESTIMATE, modest consumer + 1 enterprise renewal] | $6,000 (3 months × $2,000/month) | **2.0×** | Strong — but enterprise-driven; need consumer cohort data to validate |
| Q4 Year 2 (Series A build-up) | $25,000 [ESTIMATE, consumer + 3 enterprise deals] | $18,000 (includes Growth Marketer + tooling) | **1.4×** | Good; above 0.75 threshold |
| Q2 Year 3 (AE hired, ABM active) | $60,000 [ESTIMATE — Series A ARR ramp] | $45,000 (AE + PMM + ABM tools) | **1.3×** | Target range; monitor as ABM spend stabilises |

**Magic number interpretive note:** At pre-seed stage (Months 1–18), the magic number is not a reliable signal because enterprise deals are lumpy (a single $50k ACV contract spikes the numerator) and the denominator (S&M spend) is almost entirely founder time with minimal cash. Begin tracking magic number formally only after the first AE is hired and S&M spend is > $10k/quarter — at that point the signal is meaningful.

**Data pipeline requirement:** Magic number requires ARR bridge data (§23.1), quarterly S&M expense data (classified per §28.2), and a consistent ARR recognition policy (§18.1). Assign to data-engineer to build the monthly S&M reconciliation report alongside the §27.4 infrastructure cost tracking.

---

### 28.11 DEC-030 Marketing Spend Audit Events

Marketing spend events are required for: (1) Series A diligence — investors will audit S&M efficiency and want a defensible S&M spend log; (2) internal governance — prevent uncontrolled paid UA spend when LTV:CAC deteriorates; (3) SOC 2 Type II — not directly in scope, but the HMAC chain integrity model requires all material financial events to be recorded.

| Event type | Severity | Key metadata fields | Retention | Trigger |
|---|---|---|---|---|
| `mkt.paid_ua_gate_unlocked` | STANDARD | `gate_id` (MKT-UA-GATE-01), `unlock_date`, `d30_retention_pct`, `install_count_at_unlock`, `arpu_at_unlock`, `unlocked_by` (founder internal ID) | 3 years | Emitted once when MKT-UA-GATE-01 criteria are first satisfied and paid UA budget > $1,000/month is approved; never re-emitted for the same gate |
| `mkt.paid_channel_paused` | STANDARD | `channel` (asa / meta / tiktok / other), `pause_reason` (ltv_cac_breach / budget_exhausted / experiment_failed / founder_decision), `monthly_spend_at_pause_usd`, `cac_at_pause_usd` (if known), `paused_by` (founder internal ID), `pause_date` | 3 years | Emitted whenever a paid acquisition channel spending > $200/month is paused; key control event for CAC governance |
| `mkt.marketing_hire_onboarded` | STANDARD | `hire_type` (growth_marketer / pmm / head_of_marketing / other), `start_date`, `annual_base_usd`, `hire_gate_satisfied` (MKT-HIRE-01 / MKT-HIRE-02 / MKT-HIRE-03), `hired_by` (founder internal ID) | 7 years | Emitted on first day of employment; mirrors `ops.engineer_hired` pattern from §27.9 |

**HMAC chain note:** Marketing spend events are STANDARD severity and follow the same HMAC-chain append logic as all DEC-030 events (see `docs/AUDIT_LOG_SCHEMA.md §3`). They do not require multi-party sign-off (unlike CRITICAL events) but must be emitted within 24 hours of the triggering action. The `mkt.paid_channel_paused` event is the key LTV:CAC governance control — it creates an immutable record of every paid channel pause and the CAC at pause, enabling retrospective attribution quality audits for investors.

---

### 28.12 Implementation Checklist

| Item | Priority | Milestone | Owner | Definition of Done |
|---|---|---|---|---|
| Instrument UTM parameters across all marketing channels in PostHog; confirm install-source attribution is firing for organic, ASA, and referral before any paid spend begins | P0 | Before App Store launch (M4) | platform-engineer + data-engineer | PostHog funnel report shows distinct_id attributed to channel source for ≥ 95% of new installs in staging; validated with test install from each source |
| Create App Store product page assets (screenshots × 6 iOS, × 6 Android; preview video; metadata copy) and submit for App Store review; target M4 launch per §22.3 Base timeline | P0 | M4 | brand-system + brand-voice + founder | App Store listing live; first organic impressions visible in App Store Connect analytics |
| Set up email waitlist / drip sequence in Loops (free tier): (a) waitlist confirmation; (b) 3-email nurture sequence; (c) launch announcement | P0 | M3 | brand-voice + founder | Waitlist captures ≥ 100 emails before App Store launch; drip sequence fires on schedule with < 2% bounce rate |
| Define and document MKT-UA-GATE-01 trigger criteria in OKRS_2026.md and Notion board; assign monthly gate review | P1 | M4 | founder + growth-lead | Gate criteria documented; first gate review scheduled for Month 6 |
| Build monthly marketing cost report: pull Loops/email tool cost, ad spend (ASA/Meta dashboard exports), CRM cost; reconcile against §22.3 Marketing/UA cash flow line; flag variance > 15% | P1 | M5 | data-engineer + founder | Monthly report template created; first export reconciled against §22.3 |
| Register three DEC-030 marketing spend events (`mkt.paid_ua_gate_unlocked`, `mkt.paid_channel_paused`, `mkt.marketing_hire_onboarded`) in `docs/AUDIT_LOG_SCHEMA.md` with full Zod schema; add to `emit-audit-event` Worker | P1 | M6 | platform-engineer + compliance-officer | All three events registered in AUDIT_LOG_SCHEMA.md; Zod schemas validated; Worker endpoint updated |
| Begin magic number tracking once AE is hired: build quarterly S&M reconciliation report (§28.10 formula); wire to data-engineer monthly ARR bridge (§23.1) | P2 | Post-AE hire (M22+) | data-engineer | Magic number report available in Metabase with trailing 4-quarter history; first result filed before Series A deck finalisation |

---

### 28.13 Open Questions

**OQ-MKT-01: Should FORM invest in a content flywheel (sports science blog → SEO → organic installs) before or after paid UA?**

The LTV:CAC analysis (§14.5) strongly favours organic / content-driven users (≥ 5× at M24) over paid social ($50–80 CAC, 3×–3.9× at M24). However, organic SEO has a 6–12 month compounding lag before it drives meaningful installs at scale. The risk of prioritising content-only before paid: the App Store launch cohort is small and D30 retention data takes longer to accumulate. The risk of prioritising paid-first: capital efficiency is poor before retention is proven. Recommended sequencing: content flywheel starts Month 1 (long lead time); ASA brand search starts at App Store launch (low CAC, high intent); Meta/social starts only after MKT-UA-GATE-01 is green. Owner: founder + content-strategist + growth-lead. Priority: **P0 — sequencing decision must be made before M3 content plan is locked.** Resolution: confirm content calendar priorities at M3 review; treat this OQ as closed once Month 3 content plan includes ≥ 2 long-form pieces with target keywords identified.

**OQ-MKT-02: What is the right first-party referral mechanic for consumer — incentivised (credit/discount) vs. social-share (vanity/identity)?**

Fitness app referral patterns split into two archetypes: (a) incentivised referral (Dropbox model — both parties get 1 free month of Pro) drives high volume but attracts discount-seekers with lower LTV; (b) identity/social sharing (Strava model — "I hit a PR, share to Instagram") drives lower volume but higher-LTV users who share because the product made them feel good. Given FORM's brand positioning (serious athlete, anti-gimmick), the incentivised model risks attracting users who are Pro only during the credit period. The social-sharing model is harder to engineer but more congruent with the brand. Recommended: ship social-share first (lower cash cost, higher brand alignment); test incentivised referral only after the first 500 Pro subscribers and D30 retention ≥ 40%. Owner: growth-lead + brand-system. Priority: **P1 — before App Store launch.** Resolution: decide referral mechanic at M3 product review; document in `docs/GROWTH_LOOPS.md`.

**OQ-MKT-03: Should enterprise marketing collateral (one-pager, ROI calculator) be gated (form-fill) or ungated (public PDF)?**

Gated collateral builds a B2B lead list that feeds HubSpot CRM. Ungated collateral removes friction and signals confidence in the product. In the founder-led sales phase (Deals 0–3), the lead qualification benefit of gating is minimal — the founder knows every prospect personally. Ungating removes friction for champions who want to share the ROI calculator with their CFO without a form-fill. Recommended: ungate all collateral in the founder-led phase; add gating only when a demand gen function exists to follow up on form fills (post-MKT-HIRE-02). Owner: founder + marketing-lead + customer-success. Priority: **P2 — decide before first enterprise collateral is published.** Resolution: default to ungated at pre-PMF; review at first quarterly marketing review post-App-Store-launch.

---

---

## §29 Geographic Expansion Unit Economics & FX Risk Model

> Owner: data-engineer + founder. Cross-references: §8 (enterprise economics), §14 (LTV:CAC model), §22 (cash flow), §28 (marketing & demand generation). Review: quarterly (FX), annually (market expansion gates).

---

### 29.1 Purpose and Scope

This section models the full unit economics of FORM's geographic expansion from Ukraine (founding market) through EU entry (Phase 2) to US launch (Phase 3). It exists because geographic expansion is not merely a marketing decision — it is a cost, compliance, payment infrastructure, and FX risk decision that materially affects every upstream model from the §22.3 cash flow to the §14.2–14.3 cohort survival tables. A UA-only ARPU of $5–7/month is a fundamentally different unit economics profile than a US-only ARPU of $19.99/month, and a model that treats them interchangeably will produce an incorrect runway forecast, incorrect LTV:CAC floors, and incorrect Series A narrative.

The six deliverables this section covers:

1. **Market-by-market ARPU and COGS comparison** — effective revenue per user and gross margin contribution by geography (§29.2)
2. **FX risk model for UAH/EUR/USD** — exposure analysis, natural hedge quantification, and stress tests (§29.3)
3. **Geo-pricing waterfall** — how consumer and enterprise prices are set, governed, and changed across markets (§29.4)
4. **Local payment infrastructure costs** — Stripe fees, alternative processors, and App Store tax interaction by market (§29.5)
5. **Regulatory compliance cost by market** — data protection, legal review, and DPO costs mapped to markets (§29.6)
6. **Expansion sequencing economics** — when to enter each market and the gate conditions that govern timing (§29.7)

**What this section does not cover:** product localisation costs (engineering scope, not financial), UA-market political risk modelling beyond FX (outside cost model scope), and FORM's corporate structure in each market (OQ-GEO-03 below addresses entity questions; legal counsel owns resolution).

---

### 29.2 Market-by-Market ARPU Analysis

#### 29.2.1 Framework

FORM's consumer pricing is PPP-adjusted: the list price in each market is set at a level that represents equivalent purchasing-power burden to the US list price of $19.99/month. Enterprise pricing uses an anchor-based model — the global tier list ($299/$799/$1,499/month) is the anchor, and local enterprise prices are negotiated downward from it with a floor (UA floor ~$50–150/month; EU floor ~$199–499/month) reflecting local market willingness-to-pay and the strategic value of early anchor customers in a new geography.

Effective ARPU is the net revenue per active subscriber after: (a) App Store / Google Play commission (15%–30%), (b) payment processing fees (§29.5), (c) any PPP discount vs. US list price. It is the correct numerator for LTV calculations (§14.2–14.3) — not the gross list price.

Gross margin contribution at the user level = Effective ARPU minus per-user COGS. The COGS components (Anthropic API, Supabase read, Cloudflare Workers, push notifications) are denominated in USD and do not vary by geography — a UA user consumes the same server-side COGS as a US user. This means PPP-discounted UA users have materially lower gross margin in dollar terms, which is the core unit economics risk of the UA-heavy early cohort.

#### 29.2.2 Consumer market table [ESTIMATE — replace with actual cohort data at M12]

| Market | List Price | Effective ARPU (net of store fee + FX) | Per-user COGS ($/mo, §3.2) | Gross Margin per User | LTV at M24 (§14.2 survival) | CAC | Payback Period | Notes |
|---|---|---|---|---|---|---|---|---|
| Ukraine (UA) | $5.99–$6.99/mo | $4.30–$5.00 (30% App Store fee at launch tier; Stripe web ~$5.60–$6.60) | $0.58–$1.12 | **73%–84%** | $52–$75 | $10–$25 (organic-dominant; ASO + word-of-mouth) | **6–15 months** | PPP index ~0.32 vs. US. High churn risk (economic instability, war context). UA web billing via LiqPay/Fondy reduces App Store fee to 0% but adds ~$2k–$5k integration cost (§29.5.3). Annual plan adhesion likely lower than US cohort. |
| Poland (PL) — EU-entry | $10.99–$11.99/mo | $7.50–$8.30 (App Store; Stripe web ~$10.40–$11.30) | $0.58–$1.12 | **80%–87%** | $90–$120 | $25–$60 (ASO + limited Meta; founder network proximity) | **8–16 months** | PPP index ~0.55 vs. US. Lower CAC than DE/NL due to founder network effect and eastern-European fitness community adjacency. GDPR compliance cost shared across all EU (§29.6.2). |
| Germany + Netherlands (DE/NL) — EU core | €14.99/mo (~$16.50 at 0.91 EUR/USD) | $11.20–$12.00 (App Store; Stripe EUR web ~$13.50–$14.50) | $0.58–$1.12 | **83%–89%** | $134–$170 | $60–$150 (Meta + ASA; competitive fitness market) | **12–20 months** | Highest LTV in EU tier. EUR/USD FX slippage 2–4% on USD reporting (§29.3.2). GDPR compliance already paid (shared cost). Annual plan mix expected higher than PL (higher income, lower price sensitivity). |
| United States (US) | $19.99/mo | $13.99–$16.99 (App Store 30%→15% after year 1; Stripe web ~$18.70) | $0.58–$1.12 | **85%–92%** | $168–$240 | $50–$150 (Meta/ASA; highest competition) | **10–18 months** | Full list price. Highest TAM. Highest CAC. Most competitive. Annual plan mix target 30–40% (§14.3 annual cohort table). No PPP discount. CCPA compliance ~$3k–$8k year 1 (§29.6.3). |

**Key interpretive notes on the table:**

(1) The UA gross margin per user (73%–84%) is lower than US (85%–92%) in percentage terms but directionally similar. The real unit economics difference is in LTV — $52–$75 at M24 for UA vs. $168–$240 for US. This 3–4× LTV gap is the economic argument for treating UA as a validation cohort rather than a revenue-scale market in the first 24 months. The §14.4 CAC payback analysis (which targets ≤ 12 months) is more easily satisfied in UA due to the low CAC ($10–$25 organic-dominant), even at the lower ARPU.

(2) UA enterprise pricing ($50–$150/month anchor) is 17–50% of the US Starter price ($299/month) and reflects PPP-adjusted willingness-to-pay for B2B SaaS in the Ukrainian market. The unit economics of a UA enterprise deal are not modelled in the §8 enterprise economics section (which uses US/EU pricing as baseline) — data-engineer to build a UA enterprise COGS bridge at the first UA enterprise pilot.

(3) Annual plan LTV uplift: a UA annual Pro subscriber at ₴1,999/year equivalent (~$50–55) has a shorter payback period than the monthly cohort survival model implies, because cash is received upfront. The §18.5 deferred revenue waterfall applies. However, in-app annual plans in Ukraine carry higher refund risk due to economic instability — see OQ-GEO-01 re: denominating enterprise contracts in USD.

(4) COGS is denominated in USD regardless of user geography, which creates a natural gross margin improvement as users shift from UA (low ARPU) to EU/US (higher ARPU) — the same infrastructure cost base services a higher-revenue user mix. This is the core geo-expansion gross margin lever.

---

### 29.3 FX Risk Model

#### 29.3.1 FORM's currency exposure stack

FORM's functional currency for financial reporting is USD. The exposure structure by entity:

| Cash flow type | Denominated in | FX pair at risk | Direction of risk | Magnitude |
|---|---|---|---|---|
| Consumer subscription revenue (UA App Store) | USD (Apple settles in USD even for UA region) | Indirect — UA user willingness-to-pay is UAH-denominated | UAH devaluation reduces UA users' ability to afford the $-denominated subscription | Medium |
| Consumer subscription revenue (EU App Store) | EUR | EUR/USD | EUR weakness vs. USD reduces reported USD revenue | Medium |
| Enterprise contract revenue (UA) | Recommended: USD (see OQ-GEO-01) | UAH/USD if billed in UAH | UAH devaluation erodes contract USD value if billed in UAH | High (if UAH-billed) |
| Engineering payroll (UA-based team) | UAH (salary paid in UAH) | UAH/USD | UAH devaluation reduces USD cost of UA team — natural hedge on cost side | Positive (cost hedge) |
| Stripe EUR payouts | EUR | EUR/USD | EUR weakness → lower USD settlement from EU revenue | Low–Medium |
| Infrastructure/vendor costs | USD | None | No FX exposure — vendors bill in USD | None |

#### 29.3.2 UAH/USD exposure model

The UAH/USD rate has experienced significant structural devaluation since 2022: from approximately ₴27/$ in early 2022 to ₴38–41/$ in 2024, a 41–52% move over 24 months. The National Bank of Ukraine manages the rate with periodic resets rather than free-float, creating step-function risk rather than continuous drift.

**Revenue-side UAH/USD impact:** FORM's UA consumer revenue is USD-denominated via Apple (App Store proceeds settle in USD at Apple's exchange rate, not spot UAH/USD). This means FORM does not directly receive UAH from UA App Store sales — the FX exposure is indirect: if UAH devalues further, the $5.99–$6.99/month UA Pro price represents a larger share of the average Ukrainian user's monthly income, increasing churn risk. A 20% UAH devaluation does not reduce FORM's USD revenue per active UA subscriber — it increases the probability that the subscriber churns.

**Cost-side UAH/USD impact (natural hedge):** FORM's UA-based engineering team is compensated in UAH (or USD equivalent at prevailing rate per contract terms — see OQ-ENG-01 in §27.11). If a founding engineer is paid ₴3.6M/year (~$90k at ₴40/$), a 20% UAH devaluation to ₴48/$ would reduce the USD equivalent of that payroll to $75k without any contract renegotiation — a $15k/year cost saving in USD terms. This is a real natural hedge: the same geopolitical shock that increases UA consumer churn risk also reduces UA engineering cost in USD terms.

**Net UAH/USD stress test — 20% UAH devaluation scenario:**

| Impact | Direction | Magnitude |
|---|---|---|
| UA consumer churn rate increase (PPP burden increases) | Negative revenue | +3–8 percentage points on monthly churn [ESTIMATE] |
| UA enterprise wilingness-to-pay decreases (if UAH-billed) | Negative revenue | Up to 17% effective price reduction on UAH-billed contracts |
| UA engineering payroll in USD terms decreases | Positive cost | ~$15k–$20k/year per senior engineer at current UA salary bands |
| Founder personal income (if UA-based) purchasing power | Negative (personal) | Out of scope for cost model |
| Net P&L impact (assuming 50 UA Pro subscribers at M12) | Near-neutral | Revenue delta: ~-$150–$400/year from churn; cost delta: +$15k–$20k/year engineering hedge |

**Conclusion on UAH/USD:** At current FORM scale (UA-dominant early cohort, UA-based team), UAH devaluation is net-neutral to slightly positive in P&L terms — the cost hedge outweighs the small UA revenue base. This changes when FORM scales EU/US revenue without scaling UA headcount proportionally, at which point the cost hedge naturally diminishes as a % of total cost.

#### 29.3.3 EUR/USD exposure model

EUR/USD has ranged from ~0.96 to ~1.12 over the 2022–2024 cycle — a ±15% peak-to-trough range. At EU entry (Phase 2, Month 12–24), FORM may have meaningful EUR-denominated revenue from EU App Store (settled in EUR by Apple) and EU enterprise contracts.

**Stripe EUR settlement:** Stripe offers multi-currency settlement — EUR revenue can be held in a EUR Stripe balance and converted to USD on-demand or on a schedule. At small EU revenue volumes ($5k–$30k/month), the conversion fee is 1.5% of the converted amount, plus the spot rate applied by Stripe (typically 0.5–1% worse than mid-market). Total FX slippage on EUR→USD conversion: 2–4% of EUR revenue [ESTIMATE].

**Mitigation option:** Hold EUR in Stripe EUR balance and use it to pay EUR-denominated vendors or contractors if/when FORM hires EU-based staff. At pre-Series A scale with a UA-based team, this netting opportunity is limited — EUR revenue will predominantly convert to USD. Budget 2–3% EUR/USD FX slippage in EU unit economics model.

**EUR/USD stress test — 10% EUR depreciation (EUR/USD moves from 1.08 to 0.97):**

| Line item | Before | After | Delta |
|---|---|---|---|
| EU consumer ARPU (USD-reported) | $11.20–$12.00 | $10.08–$10.80 | -$1.12–$1.20 per subscriber/month |
| EU enterprise Starter contract ($299 equivalent, billed in EUR at €277) | $299/month | $269/month USD equivalent | -$30/month per contract |
| EU contribution to ARR ($10k EUR ARR scenario) | $10,800 | $9,700 | -$1,100/year |

**Conclusion on EUR/USD:** At EU entry revenue volumes ($10k–$50k ARR), EUR/USD exposure is a second-order effect — the absolute dollar impact is small relative to total P&L. It becomes material at $200k+ EU ARR. Mitigation: price EU enterprise contracts in USD where customers accept it (larger DE/NL enterprises typically accept USD billing); price EU consumer in EUR (customer expectation, App Store convention). Review FX policy at $100k EU ARR.

#### 29.3.4 FX risk mitigation summary

| Risk | Mitigation | Residual risk |
|---|---|---|
| UAH/USD step devaluation increases UA churn | UA Pro price already PPP-adjusted; monitor churn monthly; founder approval required for UA price change (§29.4.5) | UA churn spike in severe devaluation scenario |
| UAH/USD devaluation erodes UA enterprise contract value | Bill UA enterprise in USD (OQ-GEO-01 resolution) | Customer may demand UAH billing; renegotiation risk |
| EUR/USD drift reduces USD-reported EU revenue | Stripe multi-currency settlement; consider EUR-priced enterprise contracts; quarterly FX review | 2–4% annual slippage at current volumes |
| EUR/USD moves trigger pricing review | geo.fx_threshold_exceeded event at ±15% (§29.8) triggers repricing review | Lag between rate move and price update (next App Store pricing cycle) |

---

### 29.4 Geo-Pricing Waterfall

#### 29.4.1 Pricing philosophy

FORM applies two distinct pricing philosophies depending on segment:

- **Consumer (PPP-adjusted):** The UA consumer market cannot bear US list prices. A $19.99/month subscription is ~1.5–2% of an average Ukrainian monthly salary. The US list price represents ~0.1% of US median monthly income. PPP-adjusted pricing aims to maintain equivalent purchasing-power burden across markets — approximately 0.1–0.2% of median monthly income. This is consistent with how Spotify, Netflix, and major App Store publishers price in emerging markets.

- **Enterprise (anchor-based with floor):** Enterprise pricing uses the global tier list as an anchor. Local pricing is negotiated downward based on market willingness-to-pay, competitive landscape, and strategic value of the customer as a reference. A floor exists below which FORM will not discount — the floor is set by the COGS floor plus minimum gross margin target (55% per §6.1). For UA enterprise, the floor is ~$50/month (Starter-equivalent adjusted for PPP); for EU, ~$199/month; for US, $299/month full list.

#### 29.4.2 Consumer price points by market

| Market | Monthly list price | Annual list price | vs. US list | PPP index basis | App Store tier (approximate) |
|---|---|---|---|---|---|
| Ukraine (UA) | $5.99/mo | $44.99/yr | 30% of US | UA PPP index ~0.32 (World Bank) | App Store Tier 5–7 (UA storefront) |
| Poland (PL) | $10.99/mo | $79.99/yr | 55% of US | PL PPP index ~0.55 | App Store Tier 9–11 (EU storefront, PL pricing) |
| Germany (DE) | €14.99/mo (~$16.50) | €109.99/yr (~$121) | 83% of US | DE near-parity with US | App Store Tier 13 (EU storefront) |
| Netherlands (NL) | €14.99/mo (~$16.50) | €109.99/yr (~$121) | 83% of US | NL near-parity with US | App Store Tier 13 (EU storefront) |
| United States (US) | $19.99/mo | $149.99/yr | 100% | Baseline | App Store Tier 17 (US storefront) |

**iOS App Store pricing mechanics:** Apple's App Store pricing tiers are set per storefront. FORM must configure pricing in App Store Connect for each regional storefront. App Store Connect's "International Pricing" tool auto-suggests local prices based on an anchor price — use the US Tier 17 ($19.99) as anchor and manually override the UA storefront to Tier 5–7. Annual price follows the same tier structure. Apple controls the exact converted price for each market — the figures above are targets mapped to the nearest available tier. Data-engineer to validate actual tier mapping during App Store localisation (§29.9 checklist item P1).

**Android / Google Play:** The same PPP-adjusted pricing applies to the Google Play storefront. Google Play's pricing tool offers similar regional tier controls. At pre-launch, prioritise iOS tier configuration; Android follows the same logic.

#### 29.4.3 Enterprise price waterfall

| Market | Starter | Growth | Performance | Floor (min deal size) | Notes |
|---|---|---|---|---|---|
| Ukraine (UA) | $50–150/mo | $150–400/mo | $300–800/mo | $50/mo | Anchor-based; heavily negotiated. PPP-adjusted. Recommended: USD-denominated (see OQ-GEO-01). |
| Poland (PL) | €150–250/mo | €350–600/mo | €700–1,200/mo | €150/mo | Below EU standard; founder-network pricing. First 2–3 PL deals may be strategic reference pricing. |
| Germany / Netherlands (DE/NL) | €199–299/mo | €599–799/mo | €999–1,499/mo | €199/mo | Near-full EU list price. DE/NL enterprises have lowest price sensitivity and highest compliance expectations. |
| United States (US) | $299/mo | $799/mo | $1,499/mo | $299/mo | Full list per §8.1. No geo-discount. |

**Enterprise pricing governance:** Any enterprise deal below the floor listed above requires founder sign-off. This is the same discount authority matrix as §21. The geo.price_change_approved audit event (§29.8) is emitted on approval of any below-floor deal.

#### 29.4.4 Tax handling by market

- **EU VAT:** Stripe Tax handles EU VAT collection and remittance automatically for digital services. EU consumers are charged VAT on top of the listed price (20–25% depending on member state). Stripe remits via OSS (One-Stop Shop) scheme. Cost to FORM: 0.5% of Stripe Tax transactions ($0.50 per $100 transaction). This cost is not a COGS item — it flows through net revenue as a revenue reduction (VAT is collected on behalf of tax authorities, not FORM revenue). Enterprise B2B sales in EU: VAT is reverse-charged to the customer (B2B VAT exemption under EU rules) — FORM does not collect or remit on EU enterprise invoices. Confirm with EU tax counsel by M8 (§29.9 P0 checklist).

- **UA VAT:** Ukraine imposes 20% VAT on digital services sold to Ukrainian consumers by non-resident providers as of 2022 amendments. If FORM is a Ukrainian legal entity selling to Ukrainian users, normal Ukrainian VAT registration applies. If FORM is a foreign entity selling to Ukrainian users, the "Netflix tax" (non-resident digital VAT) applies — registration with Ukrainian tax authority required. Legal review required (§29.6.1). Stripe supports UA VAT collection for non-resident entities.

- **US Sales Tax:** US does not have federal VAT. State-level sales tax on digital products varies — 27 states tax SaaS subscriptions. Stripe Tax handles US nexus analysis and state-level remittance. Cost similar to EU (0.5% of taxable transactions). Enterprise B2B in US: typically not subject to sales tax if customer provides tax-exempt certificate (enterprise procurement standard).

#### 29.4.5 Price change governance

**Consumer price change gate:** Any change to the UA consumer price (increase or decrease) requires explicit founder approval before the App Store pricing change is submitted. Rationale: UA is a war-context market where price increases carry outsized reputational risk and price decreases signal product devaluation. The geo.price_change_approved audit event (§29.8) must be emitted before the App Store change is submitted — the audit record precedes the price change, not follows it.

**EU/US consumer price changes:** At current stage (pre-launch), EU and US prices are fixed at launch-time values. Any change requires geo.price_change_approved with justification. Changes must be accompanied by an updated ARPU model reconciling the impact to §22.3 cash flow forecasts.

**FX-triggered price review:** When geo.fx_threshold_exceeded fires (EUR/USD or UAH/USD moves ≥ 15% vs. last pricing review date), the data-engineer runs a new ARPU impact model and presents to the founder within 14 days. The founder decides whether a price adjustment is warranted. For App Store price changes, Apple requires a minimum 30-day notice period for subscriber price increases — plan accordingly.

---

### 29.5 Local Payment Infrastructure Costs

#### 29.5.1 Payment infrastructure overview

FORM uses two payment rails: (1) App Store / Google Play in-app purchase (IAP) for consumer mobile subscriptions — Apple and Google handle all payment processing and remit net proceeds; (2) Stripe for web subscriptions and all enterprise billing — FORM directly bears Stripe processing fees. The interaction between these two rails has a significant cost implication: IAP fees are higher (15–30%) but include fraud management, churn recovery (Apple billing retry logic), and no incremental Stripe fee. Stripe web fees are lower (~3–4%) but require FORM to manage dunning, fraud, and tax independently.

#### 29.5.2 Stripe fee model by market [ESTIMATE — verify against current Stripe pricing page]

| Market | Card type | Stripe processing fee | Per-transaction fee | Net on $19.99 (US) or equivalent | Notes |
|---|---|---|---|---|---|
| US | Domestic card | 2.9% | + $0.30 | **$0.88** (4.4% effective) | Standard Stripe rate. For monthly Pro at $19.99: $19.11 net. |
| US | Card-present (if ever applicable) | 2.7% | + $0.05 | N/A for SaaS | Not applicable at current stage |
| EU | EU-issued card | 1.5% | + €0.25 | **€0.50** on €14.99 (3.9% effective) | Lower than US due to EU interchange cap (regulated at 0.3% for consumer cards). |
| EU | Non-EU card (US tourist, etc.) | 2.9% | + €0.30 | **€0.74** on €14.99 (5.9% effective) | Rare for EU consumer base; more relevant for EU enterprise customers with US parent company cards |
| EU | SEPA Direct Debit (enterprise) | 0.8% | capped at €5.00 | Max €5.00 per transaction | Preferred for EU enterprise recurring billing; lower fee but 2-day settlement lag. |
| UA (web/enterprise, non-App-Store) | LiqPay (domestic UA cards) | 2.75% | no fixed fee | ~$0.17 on $5.99 (2.9% effective) | LiqPay integration cost: $2k–$3k engineering (§29.5.3). Only relevant for web Pro and enterprise billing in UA. |
| UA (web/enterprise, non-App-Store) | Wayforpay | 2.5–3.0% | + ₴1–3 | ~$0.17–$0.20 on $5.99 | Alternative to LiqPay; similar fee; similar integration cost. |
| UA (web/enterprise, non-App-Store) | Fondy | 2.0–3.5% | varies | ~$0.15–$0.22 on $5.99 | Broadest card support including Visa/MC in UA; supports USD billing to UA entities. |

**App Store commission note (all markets):** Apple charges 30% commission on App Store IAP for developers with annual App Store proceeds > $1M/year. For FORM at pre-seed scale (well below $1M), the Small Business Program rate is 15%. Google Play matches at 15% for the first $1M in annual revenue. The COGS tables in §3 use 30% as a conservative estimate — at current scale, the actual rate is 15%. This is a ~$1.50/month per subscriber improvement in net ARPU vs. the §3 conservative model, and should be updated in the §22.3 cash flow model before the Series A deck is finalised.

#### 29.5.3 UA payment infrastructure: why Stripe is not sufficient

Stripe does not directly support UA-issued bank cards for charging (as of 2024 — verify before UA launch). Ukrainian consumers with Visa/MC issued by UA banks can use Apple Pay or Google Pay, which routes through Apple/Google (IAP). For web-based Pro subscriptions or enterprise billing in UA, FORM must integrate a UA-licensed payment processor.

**Options and trade-offs:**

| Processor | Pros | Cons | Engineering integration cost | Recommended? |
|---|---|---|---|---|
| LiqPay (PrivatBank) | Market leader; most UA users have PrivatBank cards; Stripe-like API; webhooks | PrivatBank state ownership (geopolitical consideration); limited English documentation | $2k–$3k (2–3 weeks engineer time) | Yes — first choice for UA web consumer |
| Wayforpay | Widely used by UA SaaS companies; supports USD billing; good API | Smaller bank base than LiqPay; customer support quality variable | $2k–$3k | Yes — backup/enterprise UA |
| Fondy | EU-regulated (Malta); supports UA+EU cards; GDPR-compliant | Higher fee; less recognisable brand in UA | $2k–$4k | Consider for EU expansion (dual-market processor) |

**Integration timing:** UA payment processor integration is a P1 (before UA enterprise pilot) item — the App Store handles all UA consumer in-app payments, so the processor is only needed for: (a) web-based Pro subscriptions (if offered), and (b) enterprise billing in UA. Defer to Month 6–8 when first UA enterprise pilot is expected. See §29.9 implementation checklist.

**Enterprise billing in UA: App Store 0% note.** Enterprise contracts are billed directly via Stripe (or UA processor) — not through the App Store. There is no App Store fee on enterprise revenue. This is consistent with the §8.1 enterprise gross margin model. The gross margin on a UA enterprise Starter deal ($50–$150/month) is lower than a US enterprise deal in absolute dollars but similar in percentage terms: COGS is $10–$25/month (infrastructure + AI API allocation per §15.7), leaving ~80–85% gross margin even at UA anchor prices — the margin is percentage-healthy but the absolute dollar contribution is low.

#### 29.5.4 Annual payment processing cost model by market [ESTIMATE at 1,000 Pro subscribers]

| Market | Consumer subscribers | Monthly list price | Net after App Store (15%) | Annual net revenue | Stripe web (blended ~5%) | Net after processing | Notes |
|---|---|---|---|---|---|---|---|
| UA | 400 (assumed 80% IAP) | $5.99 | $5.09 (IAP) / $5.60 (web) | $24,432 | ~$610 web only (20% web mix) | ~$23,822 | UA mix: 80% IAP (App Store 15%), 20% web (LiqPay 2.75%). |
| EU | 400 (assumed 70% IAP) | €14.99 (~$16.50) | $14.03 (IAP) / $16.01 (web) | $79,200 | ~$1,300 web only (30% web mix) | ~$77,900 | EU IAP at 15%; EU Stripe cards at 1.5%+€0.25 average. |
| US | 200 (assumed 75% IAP) | $19.99 | $16.99 (IAP) / $18.81 (web) | $47,976 | ~$700 web only (25% web mix) | ~$47,276 | US IAP at 15%; Stripe 2.9%+$0.30 on web. |
| **Total 1,000 subscribers** | **1,000** | — | — | **~$151,608** | **~$2,610** | **~$149,000** | ~1.7% blended processing cost on gross billings |

---

### 29.6 Regulatory Compliance Cost by Market

#### 29.6.1 Ukraine: data protection and martial law considerations

Ukraine's primary data protection law is the Law of Ukraine "On Personal Data Protection" (2010, amended). It requires: (a) data processing notification to the Ukrainian Personal Data Protection Commissioner (Uповноважений Верховної Ради України з прав людини) — a registration-style requirement; (b) lawful basis for health data processing (fitness data is sensitive); (c) data subject rights (access, deletion, correction) consistent with GDPR in spirit.

Under martial law conditions (effective from February 2022, extended), additional data rules apply, including restrictions on the export of certain categories of personal data and requirements for data storage within Ukraine or EU-equivalent jurisdictions for government and critical infrastructure sectors. FORM is not a critical infrastructure operator, but should obtain explicit legal advice on whether martial law data export restrictions affect fitness/health data processed for Ukrainian users.

**Year 1 compliance cost for UA:**

| Item | Estimated cost | Notes |
|---|---|---|
| Legal review: UA Personal Data Protection Law compliance + martial law data rules | $500–$1,000 | One-time; UA-based legal counsel; can be bundled with corporate/labour law review |
| UA Commissioner registration (if required) | $0 (administrative fee only) | Founder submits; legal counsel prepares filing |
| Privacy policy in Ukrainian (required for UA users) | $0–$200 | Translation of EN privacy policy; brand-voice agent handles draft; legal review recommended |
| **Total Year 1 UA compliance cost** | **$500–$1,200** | Primarily one-time legal review |

#### 29.6.2 European Union: GDPR health data requirements

FORM processes health data — workout logs, body composition, training load, nutrition data. This is "data concerning health" under GDPR Art. 9(1) — a special category requiring explicit consent (Art. 9(2)(a)) or another Art. 9(2) legal basis. Art. 9 compliance is not optional and not achievable through a standard privacy policy — it requires a specific consent mechanism at onboarding.

**GDPR compliance requirements for FORM at EU entry:**

| Requirement | Basis | Cost estimate | Notes |
|---|---|---|---|
| Data Protection Impact Assessment (DPIA) — mandatory per GDPR Art. 35 for large-scale health data processing | Art. 35 | $1,500–$3,000 (one-time; external DPO advisory) | DPIA must be completed before EU launch, not after. Triggers: health data + potential high risk to data subjects. |
| DPO advisory engagement (FORM does not require a formal DPO appointment unless it is a public authority or carries out large-scale systematic monitoring, but DPO advisory is recommended) | Art. 37 | $2,000–$5,000/year (retainer) | Outside counsel with GDPR specialisation. For an early-stage startup, a fractional DPO retainer is standard. |
| EU GDPR Art. 27 EU Representative | Art. 27 (required if no EU establishment) | $500–$1,000/year | If FORM operates as a UA/non-EU entity serving EU users, it must appoint an EU representative. Services like DataRep or similar charge $400–$800/year. |
| Standard Contractual Clauses (SCCs) for EU → non-EU data transfers | GDPR Chapter V | $500–$1,500 (legal review of SCC template) | Required if FORM's data processing infrastructure (Supabase, Cloudflare, Anthropic) is outside the EU. In practice, Supabase EU region + Cloudflare EU data residency mitigates but does not eliminate the SCC requirement. |
| GDPR-compliant consent flows at onboarding | Art. 7 + Art. 9(2)(a) | $0 (engineering cost already in platform) | Engineering cost only; no incremental legal fee beyond DPIA |
| EU localized privacy policy + terms | — | $500–$1,000 (legal review) | EU-specific addendum to global privacy policy; highlight Art. 9 consent, right to erasure, DPA contact |
| DPA registration (some EU member states require formal registration) | National DPA requirements | $0–$500 (country-dependent) | Germany (BfDI): no registration fee for private companies; Netherlands (AP): no registration fee; Poland (UODO): no fee. |
| **Total Year 1 EU compliance cost** | | **$5,000–$12,000** | DPO retainer is the largest recurring item; DPIA and SCCs are one-time |
| **Ongoing annual EU compliance cost (Year 2+)** | | **$2,500–$6,000/year** | DPO retainer + EU representative + annual DPIA review |

**GDPR operating leverage:** Once the DPIA is complete and DPO retainer is in place, the marginal cost of adding EU subscribers is $0 — compliance is a fixed cost that amortises over the EU subscriber base. At 400 EU subscribers (§29.2.2 table), Year 1 GDPR compliance cost is ~$12.50–$30 per EU subscriber — a significant cost per subscriber at entry, declining to ~$6–$15 at 800 EU subscribers in Year 2.

#### 29.6.3 United States: CCPA, HIPAA adjacency, and state privacy laws

FORM is not a HIPAA "covered entity" (it is not a healthcare provider, health plan, or healthcare clearinghouse) and is not a "business associate" of a covered entity. HIPAA does not directly apply. However, FORM processes health-adjacent data (workout performance, body composition, nutrition) that may be regulated as "sensitive personal information" under CCPA (California) and similar state laws.

**US data privacy compliance requirements:**

| Requirement | Basis | Cost estimate | Notes |
|---|---|---|---|
| CCPA compliance review: notice at collection, privacy policy, opt-out rights, deletion rights | CA Civil Code §1798.100+ | $1,500–$3,000 (one-time legal review) | California Privacy Rights Act (CPRA) expanded CCPA in 2023; sensitive PI provisions apply to health data. FORM must add "Do Not Sell or Share My Personal Information" if applicable and honor deletion requests. |
| California Privacy Policy update | CCPA | $500–$1,000 (legal drafting) | "Fitness or exercise data" is explicit sensitive PI under CPRA §1798.140(ae)(1)(I). |
| Virginia CDPA + Colorado CPA review | State law | $500–$1,000 (bundled review) | Virginia and Colorado have similar consumer data protection laws. Health data is sensitive PI. At sub-$25M revenue, FORM may not be a "controller" under VA CDPA threshold — confirm with counsel. |
| US entity establishment (required for US market at scale) | Corporate law | $1,500–$3,000 (Delaware C-Corp or LLC) | Not a compliance cost per se, but a prerequisite for US enterprise contracts and US bank accounts. Bundle with Series A legal preparation. |
| **Total Year 1 US compliance cost** | | **$4,000–$8,000** | One-time legal review + policy drafting; entity cost may be bundled with Series A legal |
| **Ongoing annual US compliance cost (Year 2+)** | | **$1,000–$2,500/year** | Annual policy review + state law monitoring (new state privacy laws pass regularly) |

**HIPAA note for future reference:** If FORM ever integrates with employer health plans (e.g., as a wellness benefit that intersects with EAP or HSA), the covered entity / business associate analysis must be re-run. This is relevant for the enterprise Performance tier ($1,499/month) targeting large employers. Flag for OQ resolution when first Performance-tier enterprise prospect is identified.

#### 29.6.4 Compliance cost summary and operating leverage

| Market | Year 1 compliance cost | Year 2+ annual | Per-subscriber at 400 subs | Per-subscriber at 1,000 subs |
|---|---|---|---|---|
| Ukraine | $500–$1,200 | ~$200–$500/year | $1.25–$3.00 | $0.50–$1.20 |
| European Union | $5,000–$12,000 | $2,500–$6,000/year | $12.50–$30.00 | $5.00–$12.00 |
| United States | $4,000–$8,000 | $1,000–$2,500/year | $10.00–$20.00 | $4.00–$8.00 |

**Compliance moat note:** The EU GDPR and US CCPA compliance infrastructure, once built, creates a competitive moat for FORM in the enterprise segment — enterprise prospects in regulated industries (financial services, healthcare-adjacent, insurance) will require GDPR and CCPA compliance as a baseline contract requirement. The $5k–$12k Year 1 EU investment is simultaneously a compliance cost and a sales enablement cost that unlocks enterprise deals that would otherwise be blocked. Cross-reference §25.7 (compliance moat vs. TAM expansion) for the enterprise-specific analysis.

---

### 29.7 Expansion Sequencing Economics

#### 29.7.1 Phase 1: Ukraine-only (M1–M12, pre-launch through first-year cohort)

**Rationale:** Ukraine is the founding team's home market. It offers zero incremental compliance cost (§29.6.1 review is low-cost and bundled with launch legal), the highest founder engagement quality (founder can do in-person user research), the lowest CAC (organic and founder network), and serves as the validation cohort for D30 retention, activation funnel, and product-market fit before capital is deployed into higher-CAC markets.

**Economic profile of Phase 1:**

| Metric | Phase 1 (UA-only) | Notes |
|---|---|---|
| Effective ARPU | $4.30–$5.00/month | PPP-adjusted; App Store IAP dominant |
| Blended gross margin | 73%–84% | Lower than EU/US but healthy |
| Target cohort size by M12 | 200–500 Pro subscribers | §22.3 Base scenario target |
| Target D30 retention | ≥ 40% | EU expansion gate (GEO-GATE-01); mirrors §28.5 paid UA gate |
| Total compliance cost | $500–$1,200 | One-time |
| Monthly marketing spend | $500–$1,500 | §22.3 Marketing/UA line; organic-dominant |

**Phase 1 risk:** War-context churn events (mobilisation, power outages, banking disruption) can create step-function churn that invalidates the D30 retention signal. The founder should annotate the PostHog cohort data with external events (e.g., major mobilisation waves, grid outage periods) so that the D30 signal is interpreted in context — a 35% D30 during a severe grid outage period may represent a stronger cohort than a 42% D30 in a stable period. Data-engineer to add event annotation capability to the cohort dashboard.

#### 29.7.2 Phase 2: EU expansion (M12–M24, post-consumer-PMF)

**Gate (GEO-GATE-01):** EU expansion is triggered only when all of the following are satisfied:

| Condition | Measurement | Source |
|---|---|---|
| D30 retention ≥ 40% on domestic (UA) cohort | PostHog cohort retention report | data-engineer monthly report |
| GDPR DPIA complete | DPIA document signed and filed | compliance-officer + external DPO |
| DPO advisory engaged (fractional DPO retainer signed) | Contract executed | founder |
| EU Stripe account live and tested | Stripe dashboard EUR balance active; test transaction confirmed | platform-engineer |
| EU App Store pricing configured (PL, DE, NL storefronts) | App Store Connect pricing live | platform-engineer |
| SCCs signed with EU-region infrastructure providers | Supabase EU SCC, Cloudflare EU SCC | platform-engineer + legal |

**Rationale for D30 ≥ 40% gate:** This is the same gate as MKT-UA-GATE-01 (§28.5). The logic is identical — EU expansion is capital-intensive (GDPR compliance, higher CAC, Stripe EUR setup) and should only proceed when the core product has proven it can retain users at a level that makes LTV:CAC ≥ 3× achievable. At D30 < 35%, EU expansion accelerates the burn of compliance and marketing budget without a proven retention foundation.

**EU entry market order (see OQ-GEO-02):** Recommended entry order is Poland first, then Germany/Netherlands. Poland has lower CAC (founder network, Eastern European fitness community, adjacent to UA diaspora), lower compliance marginal cost (GDPR compliance is shared — Poland adds no incremental GDPR cost once the DPIA and DPO retainer are in place), and serves as a lower-risk test market for EU App Store distribution, pricing, and localisation. Germany/Netherlands follow with full-price EU SKU once the PL distribution and retention pattern is validated.

**Phase 2 incremental economics:**

| Cost item | One-time | Annual recurring |
|---|---|---|
| GDPR compliance (DPIA + DPO + SCCs + EU Rep) | $5,000–$12,000 | $2,500–$6,000 |
| EU App Store localisation (PL, DE, NL screenshots + metadata) | $500–$1,500 (contractor) | $0 (refresh on new features) |
| EU Stripe configuration + testing | $500 (engineer time) | $0 |
| EU Meta / ASA campaign setup | $500–$1,000 (setup) | Ongoing per §28.5 CAC model |
| **Total Phase 2 entry cost** | **$6,500–$14,500** | **$2,500–$6,000/year** |

At 400 EU subscribers generating ~$77,900/year (§29.5.4 table), Year 1 EU compliance + setup cost ($6,500–$14,500) represents 8–19% of Year 1 EU net revenue — acceptable, and the ratio improves sharply in Year 2 when the one-time costs do not recur.

#### 29.7.3 Phase 3: US expansion (M24+, post-Series A)

**Gate (GEO-GATE-02):** US expansion requires:

| Condition | Measurement | Source |
|---|---|---|
| EU ARR ≥ $50,000 | ARR bridge (§23.1) | data-engineer |
| Series A closed | Cap table updated, funds wired | founder |
| US legal entity established (Delaware C-Corp recommended) | Certificate of Incorporation | corporate counsel |
| CCPA privacy policy live and reviewed by US counsel | Published URL + legal sign-off | compliance-officer |
| MKT-HIRE-02 (PMM) hired or contracted | Offer letter signed | founder |
| AE hire in process or contracted | Pipeline or offer letter | founder |

**Rationale for GEO-GATE-02:** The US market has the highest CAC ($50–$150 for consumer, §28.5) and the most competitive fitness app landscape. Entering the US before Series A means doing so with insufficient capital to run the paid acquisition experiments needed to find a sustainable CAC, which is a capital destruction trap. The EU ARR ≥ $50k gate ensures FORM has a proven international revenue engine before entering the highest-cost market. The MKT-HIRE-02 and AE gates ensure FORM has the GTM headcount to compete.

**Phase 3 economics summary:**

| Metric | US market (Phase 3) |
|---|---|
| Effective ARPU | $16.99–$18.81 (after App Store 15%/Stripe) |
| Target D30 retention | ≥ 40% (same gate applied to US cohort before scaling paid UA) |
| CAC range (consumer) | $50–$150 (Meta + ASA, §28.5 table) |
| CAC payback | 10–18 months (§29.2.2) |
| Year 1 US compliance cost | $4,000–$8,000 |
| Series A capital allocation to US | Defined in §24.4 capital requirements |
| LTV at M24 | $168–$240 (highest of all three markets) |

---

### 29.8 DEC-030 Geographic Expansion Audit Events

Geographic expansion events are required for: (1) investor diligence — Series A investors will ask when and why markets were entered, and an immutable log is the most credible answer; (2) compliance audit trail — GDPR DPAs may ask for evidence of when EU processing began and whether DPIA preceded it; (3) FX governance — the geo.fx_threshold_exceeded event creates a documented trigger for pricing reviews, preventing "we forgot to reprice" situations that erode gross margin silently.

| Event type | Severity | Key metadata fields | Retention | Trigger |
|---|---|---|---|---|
| `geo.market_activated` | STANDARD | `market_code` (ISO 3166-1: UA / PL / DE / NL / US), `activation_date`, `pricing_tier_consumer_monthly_usd`, `pricing_tier_enterprise_starter_usd`, `compliance_checklist_ref` (link to §29.9 checklist completion record), `geo_gate_satisfied` (GEO-GATE-01 / GEO-GATE-02 / N/A for UA), `activated_by` (founder internal ID) | 7 years | Emitted once per market, on the date the first paying user or enterprise contract is recorded from that market; never re-emitted for the same market |
| `geo.fx_threshold_exceeded` | STANDARD | `currency_pair` (EURUSD / UAHUSD), `rate_at_last_review`, `rate_current`, `pct_change`, `threshold_pct` (15%), `review_required_by_date` (14 days from event), `detected_by` (data-engineer pipeline or manual) | 3 years | Emitted when the monitored FX rate moves ≥ 15% vs. the rate recorded at the last pricing review date; triggers a mandatory pricing review within 14 days; does not itself change any prices |
| `geo.price_change_approved` | STANDARD | `market_code`, `segment` (consumer / enterprise), `old_price_usd`, `new_price_usd`, `change_pct`, `effective_date`, `change_reason` (fx_adjustment / ppp_rebalance / competitive / founder_decision), `approved_by` (founder internal ID), `fx_rate_at_approval` (if fx_adjustment) | 7 years | Emitted on founder sign-off of any consumer or enterprise geo-pricing change, in any market; the audit record must be created before the App Store pricing change is submitted or the enterprise rate card is updated — the record precedes the action |

**HMAC chain note:** All three geo expansion events are STANDARD severity and follow the same HMAC-chain append logic as all DEC-030 events (see `docs/AUDIT_LOG_SCHEMA.md §3`). The geo.market_activated event is particularly important — if the GDPR DPA ever asks for evidence that DPIA was completed before EU processing began, the `compliance_checklist_ref` field in this event is the audit trail. The DPA completion date must therefore precede the `geo.market_activated` activation_date. Data-engineer to enforce this ordering check in the event emission pipeline.

---

### 29.9 Implementation Checklist

| Item | Priority | Milestone | Owner | Definition of Done |
|---|---|---|---|---|
| Configure UA consumer pricing in App Store Connect (Tier 5–7, UA storefront) and Stripe (for web Pro, if launched); confirm LiqPay or Wayforpay integration is either live or explicitly deferred; obtain UA legal review of Personal Data Protection Law compliance + martial law data rules | P0 | Before App Store launch (M4) | platform-engineer + founder + legal counsel | UA App Store pricing live; UA privacy policy in Ukrainian published; legal review memo filed in compliance records |
| Complete GDPR DPIA (Data Protection Impact Assessment) for EU health data processing; engage fractional DPO advisory retainer; execute SCCs with Supabase EU, Cloudflare EU, and Anthropic (or confirm EU region data residency); appoint EU GDPR Art. 27 Representative | P0 | Before first EU user (before GEO-GATE-01 activation, targeting M12) | compliance-officer + external DPO + platform-engineer | DPIA document signed; DPO contract executed; SCCs filed; EU Representative service contracted; all items referenced in geo.market_activated compliance_checklist_ref |
| Configure EU Stripe account (EUR settlement currency) and test end-to-end EU subscription flow (PL, DE, NL storefronts on App Store + Stripe web); validate EU VAT handling via Stripe Tax | P0 | Before EU App Store launch (M12) | platform-engineer | Test EU subscription charges EUR, VAT applied correctly, Stripe EUR balance receives settlement; confirmed in staging + production |
| Complete EU App Store localisation: PL-language screenshots and metadata (App Store Connect PL storefront), DE/EN screenshots for DE/NL storefronts; configure pricing tiers per §29.4.2 | P1 | Before EU App Store launch (M12–M14) | brand-system + platform-engineer | EU storefronts live with localised assets; PL and DE metadata copy approved by brand-voice; pricing tiers confirmed in App Store Connect |
| Complete CCPA privacy policy update and California DPA compliance review; establish US legal entity (Delaware C-Corp); confirm US entity is named as data controller for US users in privacy policy | P1 | Before US App Store launch (M24+, pre-GEO-GATE-02) | compliance-officer + US legal counsel + founder | CCPA-compliant privacy policy live; Delaware entity formed; US entity confirmed as controller; geo.market_activated event emitted for US on first US subscriber |
| Implement FX rate monitoring pipeline: daily EUR/USD and UAH/USD rate fetch (ECB API + NBU API); compare vs. last pricing review rate; emit geo.fx_threshold_exceeded when ≥ 15% move detected; send Slack alert to founder and data-engineer | P2 | M6 (before EU entry, so monitoring is in place at EU launch) | data-engineer | Pipeline runs daily; test event emitted in staging; Slack alert confirmed; event registered in AUDIT_LOG_SCHEMA.md |
| Track market-by-market P&L in §22 cash flow model: add geo-split revenue and compliance cost lines for UA / EU / US; reconcile against §29.2.2 ARPU table; update ARPU actuals vs. estimates quarterly | P2 | Ongoing from M12 | data-engineer | Metabase dashboard shows revenue, ARPU, and compliance cost by market; first reconciliation completed within 30 days of EU launch |

---

### 29.10 Open Questions

**OQ-GEO-01: Should FORM offer UA enterprise pricing denominated in UAH or USD?**

If FORM bills a UA enterprise customer in UAH at ₴6,000/month (approximately $150 at ₴40/$), a 20% UAH devaluation to ₴48/$ reduces the USD-equivalent contract value to $125/month without any renegotiation — a 17% effective price cut FORM absorbs. Billing in USD protects FORM's revenue but transfers FX risk to the customer, who may not be able to budget a USD-denominated SaaS subscription in the current macro environment. Billing in USD also signals that FORM is a global product, not a local one — which is a positioning advantage for a UA-founded company trying to establish international credibility.

**Recommended resolution:** Bill UA enterprise contracts in USD with a UAH reference price in the contract addendum (for customer internal budgeting purposes only; the USD amount is the binding obligation). Include a clause allowing conversion to UAH billing upon customer request, subject to a UAH floor price that preserves the USD equivalent at the time of signing ±10%. Owner: founder. Priority: **P0 — before first UA enterprise pilot.** Resolution: confirm billing currency in enterprise contract template before first UA enterprise LOI is signed; document in `docs/ENTERPRISE.md`.

**OQ-GEO-02: What is the right EU entry order — Germany first (highest LTV, highest CAC) or Poland first (lower CAC, adjacent to founder network)?**

Germany represents the highest LTV EU market ($134–$170 at M24) but also the highest CAC ($80–$150 effective, competitive fitness market) and the most demanding compliance expectations (German users and enterprises have high data protection awareness). Poland represents lower LTV ($90–$120 at M24) but significantly lower CAC ($25–$60, founder network adjacency, Eastern European fitness community) and a linguistic/cultural bridge from the UA founding team. The standard startup argument favours Germany (larger market, higher LTV) but the unit economics argument favours Poland (faster payback, lower capital requirement at EU entry, lower risk of compliance misstep before GDPR machine is running smoothly).

**Recommended resolution:** Enter Poland first as EU validation market; enter Germany/Netherlands six months later when PL App Store distribution, retention, and GDPR compliance operations are proven. The PL cohort validates the EU GDPR workflow at lower cost and CAC before FORM invests in DE-specific creative, Meta DE campaigns, and higher-CAC acquisition. Owner: founder + growth-lead. Priority: **P1 — before EU App Store launch (M12).** Resolution: confirm EU entry order in `docs/GROWTH_LOOPS.md` and encode in §28.6 EU demand gen plan.

**OQ-GEO-03: Does FORM need a separate EU legal entity (e.g., Estonian e-Residency OÜ or Irish Ltd.) for GDPR data controller compliance, or can a Ukrainian legal entity with SCCs suffice?**

GDPR does not require a data controller to be an EU legal entity — a non-EU controller can process EU personal data with appropriate safeguards (SCCs, adequacy decision, or BCRs). Ukraine has not received an EU adequacy decision (as of 2024 — verify before EU launch). FORM would therefore need Standard Contractual Clauses between its UA entity and its EU-region infrastructure providers, and would need to appoint an EU representative (Art. 27, ~$500–$1,000/year). This is the lean path. The alternative — an Estonian e-Residency OÜ or Irish Ltd. — simplifies GDPR compliance (EU controller = no SCCs needed for EU-to-EU transfers), unlocks EU banking and payment infrastructure, and signals EU commitment to enterprise prospects. The downside: cost ($2k–$5k to establish, $1k–$3k/year maintenance), tax complexity, and operational overhead for a pre-Series A company.

**Recommended resolution:** Consult EU privacy counsel by Month 8 (4 months before target EU launch). Lean path (UA entity + SCCs + EU Representative) is viable at EU entry scale ($0–$50k ARR). Revisit EU entity question at Series A, when legal infrastructure budget allows and enterprise sales in EU may require an EU contracting entity. Owner: founder + EU privacy counsel. Priority: **P0 — before first EU user.** Resolution: legal memo from EU privacy counsel filed in compliance records; decision documented in `docs/ENTERPRISE.md` and referenced in geo.market_activated compliance_checklist_ref.

---

*v1.9 (2026-06-05): §29 Geographic Expansion Unit Economics & FX Risk Model — the geo-expansion counterpart to §28 Marketing and the financial underpinning of FORM's UA → EU → US market sequencing strategy. §29.1 scopes to six deliverables: market-by-market ARPU and COGS comparison, FX risk model (UAH/EUR/USD), geo-pricing waterfall, local payment infrastructure costs, regulatory compliance cost by market, and expansion sequencing economics. §29.2 market ARPU analysis: consumer table across UA ($4.30–$5.00 effective ARPU, $52–$75 M24 LTV, 73%–84% gross margin), Poland ($7.50–$8.30, $90–$120 LTV), Germany/Netherlands ($11.20–$12.00, $134–$170 LTV), and US ($13.99–$16.99, $168–$240 LTV); PPP index basis for pricing (UA 0.32, PL 0.55, DE/NL near-parity); core finding that UA users produce 3–4× lower LTV than US users but serve as the validation cohort at the lowest CAC; COGS is USD-denominated regardless of geography meaning PPP-discounted UA users have materially lower absolute gross margin contribution. §29.3 FX risk model: currency exposure stack (UAH/USD indirect consumer churn risk, EUR/USD Stripe settlement slippage 2–4%, UAH/USD natural hedge via UA team payroll); 20% UAH devaluation stress test (revenue impact: ~-$150–$400/year from churn at M12 scale; cost impact: +$15k–$20k/year engineering hedge — net P&L neutral to slightly positive at current scale); EUR/USD 10% depreciation stress test ($1,100/year impact on $10k EU ARR scenario — immaterial at entry scale); mitigation: hard-currency billing for enterprise, Stripe multi-currency settlement, quarterly FX review, geo.fx_threshold_exceeded monitoring. §29.4 geo-pricing waterfall: PPP-adjusted consumer pricing (UA $5.99, PL $10.99, DE/NL €14.99, US $19.99); anchor-based enterprise pricing (UA $50–$150 floor, PL €150–€250, DE/NL €199–€299, US $299 full list); iOS App Store tier mapping per market; EU VAT via Stripe Tax (B2B reverse-charge), UA VAT registration required; price change governance with founder approval gate on all UA consumer price changes and geo.price_change_approved audit event preceding any change. §29.5 payment infrastructure: Stripe fee table by market (US 2.9%+$0.30, EU domestic 1.5%+€0.25, EU SEPA 0.8% capped €5); UA payment processor necessity (Stripe does not support UA-issued cards for charging) — LiqPay, Wayforpay, and Fondy options at $2k–$5k integration cost; App Store 15% Small Business Program rate (not 30% as in §3 conservative estimate — represents ~$1.50/month per subscriber ARPU improvement; update §22.3 before Series A deck); 1,000-subscriber blended processing cost ~1.7% of gross billings. §29.6 regulatory compliance: UA $500–$1,200 one-time (Personal Data Protection Law + martial law data rules); EU $5k–$12k Year 1 (DPIA mandatory, DPO retainer $2k–$5k/year, EU Representative $500–$1k/year, SCCs) declining to $2.5k–$6k/year recurring; US $4k–$8k Year 1 (CCPA, CPRA sensitive PI, state law review) declining to $1k–$2.5k/year recurring; GDPR compliance as enterprise sales enablement moat (cross-reference §25.7). §29.7 expansion sequencing: Phase 1 UA-only M1–M12 (validation, lowest cost, founder engagement quality, war-context churn annotation requirement for PostHog cohort); Phase 2 EU entry M12–M24 gated by GEO-GATE-01 (D30 ≥ 40% domestic + GDPR DPIA complete + DPO engaged + EU Stripe live + SCCs signed); recommended EU order: Poland first (lower CAC, founder network, GDPR workflow validation), then DE/NL; Phase 2 incremental entry cost $6.5k–$14.5k (8–19% of Year 1 EU net revenue); Phase 3 US M24+ gated by GEO-GATE-02 (EU ARR ≥ $50k + Series A closed + US entity + CCPA live + PMM hired). §29.8 three DEC-030 geo audit events: geo.market_activated (STANDARD, 7yr, once per market — compliance_checklist_ref must precede activation_date for GDPR audit defence), geo.fx_threshold_exceeded (STANDARD, 3yr, at ≥ 15% FX move vs. last pricing review — triggers mandatory 14-day pricing review), geo.price_change_approved (STANDARD, 7yr, any geo pricing change — audit record precedes App Store submission). §29.9 seven-item implementation checklist: 3× P0 (UA App Store pricing + legal review, GDPR DPIA + DPO + SCCs + EU Rep, EU Stripe config), 2× P1 (EU App Store localisation, CCPA + US entity), 2× P2 ongoing (FX monitoring pipeline, market-by-market P&L tracking in Metabase). §29.10 three open questions: OQ-GEO-01 (UA enterprise billing currency UAH vs. USD — P0, before first UA enterprise LOI; recommended: USD with UAH reference clause), OQ-GEO-02 (EU entry order Germany-first vs. Poland-first — P1, before EU App Store launch; recommended: Poland first for lower CAC and GDPR workflow validation), OQ-GEO-03 (UA entity + SCCs vs. EU legal entity for GDPR controller — P0, before first EU user; recommended: consult EU privacy counsel by M8, lean path viable at EU entry scale). Cross-references: §8 (enterprise economics — UA enterprise COGS bridge to be built at first UA pilot), §14.2–14.3 (cohort survival tables — ARPU inputs must be geo-split), §14.4–14.5 (CAC payback floors — UA organic CAC satisfies ≤ 12-month payback even at PPP-adjusted ARPU), §18.1 (revenue recognition — EUR-billed contracts recognised in USD at spot rate, EUR/USD movement affects ARR bridge), §22.3 (cash flow — Marketing/UA + compliance cost lines updated by market stage), §23.1 (ARR bridge — geo-split ARR required for market P&L), §24.4 (Series A capital requirements — GEO-GATE-02 requires Series A as prerequisite), §25.7 (compliance moat — GDPR investment as enterprise sales enablement), §28.5 (MKT-UA-GATE-01 — D30 ≥ 35% paid UA gate mirrors GEO-GATE-01 D30 ≥ 40% EU entry gate), §28.6 (enterprise demand gen — EU/US demand gen budget references GEO-GATE-01/02 as unlock conditions), docs/ENTERPRISE.md (enterprise pricing + billing currency for UA deals), docs/AUDIT_LOG_SCHEMA.md (DEC-030 event registry — three geo events to be registered), docs/GROWTH_LOOPS.md (EU entry order and referral localisation). Owner: founder + data-engineer + compliance-officer.*

*v1.9 (2026-06-04): §28 Marketing & Demand Generation Cost Model — the cost and channel counterpart to §27 Engineering and the operational detail behind §22.3 "Marketing / UA" cash flow line. §28.1 scopes the section to ten deliverables: marketing cost taxonomy, pre-launch budget, ASO investment, consumer paid acquisition economics, enterprise demand gen budget, tooling stack, first marketing hire economics, S&M-as-%-ARR benchmarks, magic number model, and DEC-030 audit events. §28.2 marketing cost taxonomy: COGS vs. S&M classification for eight spend types; consumer vs. enterprise cost split noting that enterprise demand gen at pre-PMF is ~100% founder time (no incremental cash beyond tools). §28.3 pre-launch budget (M1–M4): $3,500 total cash across waitlist infrastructure, content creation, ASO screenshot design, and limited ASA test — directly mapped to §22.3 Marketing/UA line. §28.4 ASO investment: $700–$1,400 initial external cost plus 17h founder time; conversion rate improvement from 5% to 10% on 1,000 monthly product page views = 50 incremental installs/month at $0 marginal cost — highest-leverage pre-launch action. §28.5 consumer paid acquisition: channel benchmark table (ASA brand $17–$50 effective CAC, ASA discovery $40–$200, Meta $43–$267, TikTok $40–$300, organic ≥ $15–$30 with higher LTV); MKT-UA-GATE-01 hard ceiling (D30 ≥ 35%, App Store live, install-to-Pro rate measured on ≥ 50 organic); paid UA cap $1,000/month until gate is green. §28.6 enterprise demand gen: founder-led phase $300–$1,200/month (LinkedIn thought leadership $0 cash, HubSpot Free CRM, selective events $0–$500/event); post-Series A demand gen $7,600–$19,450/month (AE outbound tools, ABM LinkedIn $1,500–$5,000/month, events $4,000–$10,000/year, content agency $2,000–$4,000/month). §28.7 marketing tooling stack: pre-launch $0–$38/month (Figma free, Loops free, PostHog already paid, Buffer free, HubSpot Free CRM); post-launch $79–$179/month excluding ad spend; Series A $346–$666/month (Outreach, LinkedIn Sales Nav, Apollo.io, Clearbit). §28.8 first marketing hire economics: four profile options (Content Writer $25k–$40k, Growth Marketer $40k–$60k, PMM $50k–$75k, VP Marketing $70k–$100k UA rates); recommended first hire Growth Marketer at Month 18–22 post-consumer-PMF; three hiring gates (MKT-HIRE-01 D30 ≥ 40% + organic traction + founder time > 8h/week; MKT-HIRE-02 PMM concurrent with AE; MKT-HIRE-03 Head of Marketing post-Series A); marketing hires are 100% S&M — no gross margin impact (contrast with §26.6 CS gross margin impact). §28.9 S&M as % ARR: industry benchmarks (Bessemer/SaaS Capital) at four stages; FORM targets pre-revenue 40–60%, $100k–$500k ARR 35–50%, $500k–$1M ARR 30–40%, $1M–$3M ARR 25–35%; combined S&M + G&A target ≤ 50% ARR at Series A. §28.10 magic number model: formula (net new ARR ÷ prior quarter S&M spend); four-milestone table (Q4 Year 1 6.4× enterprise-driven anomaly, Q2 Year 2 2.0×, Q4 Year 2 1.4×, Q2 Year 3 1.3×); interpretive note that magic number is unreliable before first AE and > $10k/quarter S&M spend; data pipeline requirement (ARR bridge §23.1 + S&M expense classification + §18.1 ARR recognition). §28.11 three DEC-030 marketing spend audit events: mkt.paid_ua_gate_unlocked (STANDARD, 3yr, once per gate), mkt.paid_channel_paused (STANDARD, 3yr, on > $200/month channel pause — key CAC governance control), mkt.marketing_hire_onboarded (STANDARD, 7yr, mirrors ops.engineer_hired). §28.12 seven-item implementation checklist: 3× P0 M3–M4 (UTM attribution in PostHog, App Store assets, email drip sequence), 3× P1 M4–M6 (MKT-UA-GATE-01 OKR, monthly marketing cost report, DEC-030 event registration), 1× P2 post-AE (magic number reporting). §28.13 three open questions: OQ-MKT-01 (content flywheel vs. paid UA sequencing — P0, M3 content plan lock; recommended: content M1 + ASA at launch + Meta only after GATE-01), OQ-MKT-02 (incentivised referral vs. social-share mechanic — P1, before App Store launch; recommended social-share first as more brand-congruent), OQ-MKT-03 (gated vs. ungated enterprise collateral — P2, before first collateral published; recommended ungated at pre-PMF founder-led phase). Cross-references: §8.5 (enterprise CAC per deal — not duplicated here), §14.4–14.5 (consumer CAC payback and LTV:CAC floors — floor inputs for §28.5 paid UA ceiling), §18.7 (magic number > 0.75 target in SaaS glossary — operationalised in §28.10), §19.3–19.5 (enterprise GTM pipeline model — revenue side of the S&M efficiency ratio), §22.3 (cash flow — Marketing/UA line source data), §23.1 (ARR bridge — magic number numerator), §26.1 (CS cost taxonomy — privacy floor reminder applicable here), §27.9 (ops.engineer_hired DEC-030 pattern — mkt.marketing_hire_onboarded mirrors), docs/ENTERPRISE.md (enterprise pricing + no-go customers), docs/AUDIT_LOG_SCHEMA.md (DEC-030 event registry), docs/GROWTH_LOOPS.md (referral mechanic context). Owner: founder + marketing-lead + growth-lead + data-engineer.*

*v1.8 (2026-06-04): §27 Engineering Team Cost Model — the cost and hiring counterpart to §26 CS Cost Model and the operational detail behind §22.3 cash flow. §27.1 scopes the section to seven deliverables covering founding engineer total comp, technical hiring sequence, infrastructure run-rate by stage, build vs. buy, CV pipeline unit economics, engineering as % ARR trajectory, and burn rate gates. §27.2 founding engineer total compensation: cash cost $100k–$139k (midpoint ~$120k) including base $80k–$110k, employer taxes ~16%, hardware $3.5k–$5k one-time, onboarding overhead ~$4k–$6k founder-time cost — corrects the §22.3 $100k placeholder by $20k–$39k; equity grant 2.5%–3.5% at pre-seed, 1.0%–1.5% at seed, ~0.25% at Series A. §27.3 four-hire technical sequence: Founding Engineer (iOS + Supabase, Month 13 Base / Month 10–11 Bull, $80k–$110k); Platform Engineer (Cloudflare Workers + Supabase enterprise backend, Month 18–22, $75k–$100k, triggered by first enterprise contract or 15h/week founder backend time); ML/CV Engineer (model training + Core ML, Month 24–30 post-Series A, $90k–$130k, triggered by CV pipeline live + model quality as #1 complaint); Series A team of 5 at $415k–$555k/year total payroll. §27.4 infrastructure cost table across eight services (Cloudflare Workers, Supabase, R2, Anthropic API, Sentry, PagerDuty, WorkOS, Better Stack) at four scale stages: pre-revenue ~$77/mo, 1k MAU ~$140–280/mo, 10k MAU ~$680–1,300/mo, 50k MAU ~$2,800–5,500/mo; Anthropic API becomes the largest single line at 50k MAU ($1,500–$3,500/mo) but remains <0.3% of revenue per user at $79/mo price point. §27.5 seven-service build-vs-buy analysis: BUY verdicts for WorkOS (6–10 engineering weeks saved per IdP), Supabase (full-time DBA alternative), Anthropic API (no viable build alternative pre-Series B), PagerDuty, Sentry, Better Stack; BUILD verdict for CV pose estimation (on-device moat, privacy requirement, avoids $108k–$360k/year in GPU compute at 50k MAU). §27.6 CV pipeline unit economics: on-device inference at $0 marginal per-session cost vs. ~$108k–$360k/year hypothetical server-side at 50k MAU; device floor A14 (iPhone 12+); model size 20–35 MB INT8 quantized; P95 latency 25–50ms on A15+. §27.7 engineering as % ARR trajectory: 83%–115% at $100k ARR (step-function hire before revenue catches up); 34%–48% at $500k; 26%–36% at $1M; 16%–21% at $3M — operating leverage curve consistent with B2B SaaS benchmarks. §27.8 four burn rate gates (ENG-GATE-01 through ENG-GATE-04) with trigger conditions and block conditions; anti-pattern warning against ML hire before product-market fit. §27.9 three DEC-030 engineering spend audit events: ops.engineer_hired (STANDARD, 7yr), ops.infra_cost_threshold_crossed (STANDARD, 3yr), ops.build_vs_buy_decision (STANDARD, 7yr). §27.10 six-item implementation checklist: 2× P0 (§22.3 cash flow correction, engineer JD live), 3× P1 (ENG-GATE-01 OKR, monthly infra tracking, DEC-030 event registration), 1× P2 (build-vs-buy decision log for CV). §27.11 three open questions: OQ-ENG-01 (UA FOP vs. US contractor structure — P0, before offer letter), OQ-ENG-02 (iOS-specialist vs. fullstack founding engineer profile — P1, before JD finalisation), OQ-ENG-03 (device-compatibility floor for CV — P2, decide at 1k beta users). Cross-references: §22.3 (cash flow model — founding engineer line correction), §24.7 (cap table — equity grant dilution for founding engineer), §25.6 (compliance cost as % ARR — combined with engineering for total support+engineering stack), §26.4 (CS vs. engineer hire order), docs/TECHNICAL.md (CV pipeline architecture), docs/SSO_SCIM_IMPLEMENTATION.md §§23–26 (enterprise backend workload for Platform Engineer), docs/MOBILE_ROADMAP.md (iOS engineering context), docs/AUDIT_LOG_SCHEMA.md (engineering spend DEC-030 events). Owner: founder + data-engineer + platform-engineer.*

---

## 30. G&A & Founder Compensation Cost Model

### 30.1 Purpose and Scope

This section closes **OQ-27** (founder salary policy — §22.8) and supplies the detailed model behind the `G&A and founder salary` line in §24.4 Capital Requirements ($240,000–$360,000 [ESTIMATE] over 18 post-Series A months). Sections §25 through §29 modelled five functional cost centres (compliance, CS, engineering, marketing, geo-expansion). G&A is the sixth and final function, covering the costs of *running the company itself* — founder compensation, financial administration, and the thin layer of administrative tooling not already captured elsewhere.

**Scope of this section:**

| Category | Covered here | Covered elsewhere |
|---|---|---|
| Founder cash compensation | ✅ §30.2 | — |
| Bookkeeping / accounting | ✅ §30.3 | — |
| Fractional / full-time CFO | ✅ §30.7 | — |
| Admin tooling (G-Suite, Notion) | ✅ §30.4 | — |
| Cyber liability + E&O insurance | — | §25.2 |
| D&O insurance | — | §25 OQ-36 |
| Legal / compliance overhead | — | §25.3 |
| Engineering infrastructure | — | §27.4 |
| Marketing tooling | — | §28.7 |

**Owner:** founder + data-engineer. **Review:** quarterly, or immediately after any compensation phase change.

---

### 30.2 Founder Compensation Model

#### 30.2.1 Four-Phase Compensation Trajectory

The founding phase of a capital-efficient startup in a CEE market is distinct from the Silicon Valley pattern where seed rounds routinely fund $150k+ founder salaries from Day 1. The FORM model assumes the founder has personal runway (savings, partner income, or family support) to cover living expenses through the validation phase, and transitions to company-funded compensation in stages tied to business milestones rather than arbitrary calendar dates.

| Phase | Trigger | Monthly cash (UAH equivalent) | Rationale |
|---|---|---|---|
| **Phase 0 — Ramen** | M1 through first enterprise contract close (estimated M8–M12) | $0 [FOUNDER_INPUT] | Company pre-revenue; all cash preserved for product and compliance. §22.2 baseline assumption. |
| **Phase 1 — Sustenance** | First enterprise contract signed AND 90-day pilot conversion rate ≥ 50% | $2,500–$5,000 [ESTIMATE] | Company demonstrates revenue can cover basic founder cost. Does not require full ARR coverage. |
| **Phase 2 — Early Market** | ARR ≥ $100k AND gross margin ≥ 70% (§6.2 trajectory) | $6,000–$10,000 [ESTIMATE] | Business is demonstrably alive; founder can transition from pure capital-preservation to sustainable operations. |
| **Phase 3 — Post-Series A** | Series A closed (§24.9 triggers met) | $10,000–$15,000 [ESTIMATE] | Investor expectation: founder draws a salary after institutional capital closes; below-market comp reduces dilution pressure and signals conviction. Upper bound $15k is consistent with §24.4 G&A assumptions. |

**OQ-27 recommended resolution:** Phase 0 through first contract. Phase 1 at first enterprise contract close. Phase 2 at ARR ≥ $100k. Phase 3 at Series A. The founder's personal financial runway (savings + partner income) determines whether Phase 0 can hold to M12 — this is a [FOUNDER_INPUT] that must be documented privately before company formation, but must not be disclosed to employees or early hires. Owner: founder. Resolution: **before company formation or within 30 days of first capital receipt**.

**Annual G&A impact of founder compensation:**

| Phase | Annual founder comp cost (USD) | % of $150k initial funding | % of §24.4 Bear raise ($2M) |
|---|---|---|---|
| Phase 0 | $0 | 0% | 0% |
| Phase 1 ($3,500/mo midpoint) | $42,000 [ESTIMATE] | 28% | 2.1% |
| Phase 2 ($8,000/mo midpoint) | $96,000 [ESTIMATE] | 64% | 4.8% |
| Phase 3 ($12,500/mo midpoint) | $150,000 [ESTIMATE] | — | 7.5% |

#### 30.2.2 Ukrainian FOP Structure Note

If the company operates as a Ukrainian LLC (ТОВ) and the founder is simultaneously a Ukrainian FOP (ФОП, фізична особа-підприємець — sole trader), the salary/compensation structure is more complex than Western equivalents:

- A Ukrainian LLC can pay the founder either as an **employee** (зарплата, ~18% personal income tax + 22% social security on the employer side) or as a **service contract** with the founder's FOP entity (~5% simplified tax on FOP income for Group 3, or 2% for wartime temporary Group 3 regime).
- The FOP route is significantly cheaper in total tax load (effective ~5–7% vs. ~40% combined employment taxes) but requires the founder to maintain a separate FOP and invoice the LLC.
- **War-context complication:** Under martial law in Ukraine (as of 2026), certain FOP tax regimes have been modified. The 2% wartime rate for Group 3 FOP may be available to qualifying businesses. Confirm with Ukrainian tax counsel before structuring compensation.
- **HoldCo implication:** If a Delaware C-Corp or Cyprus HoldCo is formed (§24.8 OQ-33), founder compensation may flow through the HoldCo entity instead, which changes the tax treatment entirely. Resolve legal structure before the first compensation payment.

**Interaction with §27 OQ-ENG-01:** The engineering hire structure question (UA FOP vs. US contractor) has the same legal framework as founder compensation. Both should be resolved in a single legal consultation before the first hire. Cross-reference §27.11 OQ-ENG-01.

---

### 30.3 Accounting & Finance Function

Accounting costs scale with business complexity, not headcount. The four stages below match FORM's operational phases.

| Stage | Trigger | What's needed | Monthly cost [ESTIMATE] |
|---|---|---|---|
| **Pre-launch (M1–M4)** | Company formed, pre-revenue | Ukrainian accountant for FOP/LLC statutory filings; bank reconciliation; annual tax return | $150–$300 [ESTIMATE] |
| **Post-launch (M4–M12)** | App Store live; first consumer revenue; enterprise pilot invoices | Xero or QuickBooks subscription + virtual bookkeeper (10h/month); accrual accounting set-up; deferred revenue tracking per §18.1 | $400–$700 [ESTIMATE] |
| **Series A prep (M12–M18)** | ARR ≥ $100k; first investor diligence likely | CPA review engagement (not full audit) for 12 months of financials; audit-ready financial statements; cap table modelling per §24.7 | $8,000–$15,000 one-time [ESTIMATE] + $500–$800/month ongoing |
| **Post-Series A (M18+)** | Institutional investors on cap table; Board formed | Fractional CFO ($3,000–$6,000/month [ESTIMATE]) for Board reporting, investor updates, budget-vs-actual analysis, Series B prep; statutory audit may be required at investor insistence (~$25,000–$50,000/year [ESTIMATE]) | $3,500–$6,800/month [ESTIMATE] |

**Key inflection point — deferred revenue:** Enterprise annual contracts create deferred revenue from Day 1 of contract signing. Once the first enterprise contract closes, Xero must be configured for deferred revenue recognition (12 equal tranches per §18.1). This is not possible with a simple cash-basis spreadsheet and is the primary trigger for engaging a bookkeeper with SaaS accounting experience. Owner: founder + data-engineer.

**CPA firm selection criteria:** At Series A prep, the CPA must have experience with: (a) SaaS revenue recognition (ASC 606 / IFRS 15); (b) cross-border entities (UA LLC + Delaware HoldCo if applicable); (c) Ukrainian statutory accounting requirements (P(S)BO or IFRS-light). A Big Four firm is unnecessary at this stage — a boutique with CEE SaaS experience is the right profile at $8,000–$15,000 for a review engagement.

---

### 30.4 Administrative Tooling

This table covers G&A-category software tools that are not already modelled in §27.4 (engineering infrastructure) or §28.7 (marketing/sales tooling). Items where free tiers cover the pre-Series A phase are marked accordingly.

| Tool | Purpose | Monthly cost (1–3 person stage) [ESTIMATE] | Monthly cost (5–10 person stage) [ESTIMATE] | Notes |
|---|---|---|---|---|
| Google Workspace Business Starter | Email (@form.coach), Drive, Docs, Meet | $6/user → $6–$18 | $6/user → $30–$60 | Mandatory for professional email and document collaboration |
| Notion Team | Internal knowledge base, decision log, OKRs | $16/member → $0–$16 (free for 1 member) | $16/member → $48–$80 | Free for single member; Team plan required once second person joins internal docs |
| Loom Business | Async video for investor updates, customer onboarding | $12.50/creator → $12.50 | $12.50/creator → $25–$37.50 | One creator seat sufficient pre-Series A; expand as CS hires join |
| Calendly Teams | Scheduling for sales and enterprise demos | $16/member → $0 (basic free) | $16/member → $32–$48 | Free Calendly sufficient through first 3 enterprise pilots; Teams adds routing rules |
| 1Password Teams | Password / secrets manager for team | $4/user → $4 (already in CRYPTOGRAPHY_POLICY §3) | $4/user → $20–$40 | Cross-reference: `docs/CRYPTOGRAPHY_POLICY.md §3`; counted here for G&A completeness |
| **Admin tooling subtotal** | — | **$22–$51/month** [ESTIMATE] | **$135–$225/month** [ESTIMATE] | Excludes Slack (free up to 90-day message history), Zoom (free tier adequate for external calls pre-Series A) |

**Excluded from this table (tracked in other sections):**
- Slack Pro: $7.25/person/month — triggered at first hire; included in §27.4 at that point
- Figma: marketing/design; §28.7
- HubSpot CRM: sales tooling; §28.7
- Sentry, PagerDuty, Better Stack: engineering observability; §27.4
- PostHog: analytics; §27.4

**Admin tooling is not a meaningful cost at the pre-Series A stage.** Total spend of $22–$225/month is noise against the founder compensation and accounting line. The importance is ensuring the right tools are in place for investor-grade documentation (data room readiness per §24.3) before the Series A process begins.

---

### 30.5 Consolidated G&A Cost Table by Stage

All figures [ESTIMATE]. `—` means the cost is $0 (Phase 0 assumption) or not yet applicable.

| G&A Category | Pre-launch M1–M4 | Phase 0: Pre-revenue M4–M12 | Phase 1–2: Post-first-deal M12–M18 | Phase 3: Post-Series A M18–M30 |
|---|---|---|---|---|
| Founder compensation | $0 | $0 | $2,500–$10,000/mo | $10,000–$15,000/mo |
| Accounting / bookkeeping | $150–$300/mo | $400–$700/mo | $500–$800/mo | $3,500–$6,800/mo (incl. fractional CFO) |
| Admin tooling | $22–$51/mo | $22–$51/mo | $50–$150/mo | $135–$225/mo |
| CPA review (one-time) | — | — | $8,000–$15,000 (one-time at M12) | — |
| Statutory audit (annual) | — | — | — | $25,000–$50,000/year (if required) |
| **Monthly G&A total** | **$172–$351/mo** | **$422–$751/mo** | **$3,050–$10,950/mo** | **$13,635–$22,025/mo** |
| **Annualised G&A** | **~$2,100–$4,200** | **~$5,000–$9,000** | **~$37,000–$131,000** | **~$164,000–$265,000** |

**Reconciliation to §24.4:** The §24.4 Bear/Base/Bull G&A line ($240k–$360k over 18 months post-Series A) maps to Phase 3 annualised G&A ($164k–$265k) plus the one-time CPA review at M12 ($8k–$15k) and the first year of statutory audit if required. The §24.4 [ESTIMATE] labels are intentional: the dominant variable is the founder salary decision (OQ-GGA-03), which can shift the range by $60k–$90k depending on Phase 3 salary level chosen.

---

### 30.6 G&A as % ARR Trajectory

**Important caveat:** G&A as a percentage of ARR is misleading at sub-$300k ARR because the denominator is too small. A founder drawing $10k/month against $50k ARR produces a 240% "G&A ratio" that signals nothing about operational efficiency. The useful metric at early stage is **absolute monthly G&A burn**, not a ratio.

Once ARR exceeds $300k, the ratio becomes meaningful for investor benchmarking.

| ARR milestone | Annual founder comp ($12.5k/mo Phase 3) | Accounting + CFO (annualised) | Admin tooling | Total annualised G&A | G&A % ARR [ESTIMATE] |
|---|---|---|---|---|---|
| $50k ARR | $150,000 | $36,000 | $600 | ~$186,600 | ~373% — meaningless |
| $100k ARR | $150,000 | $42,000 | $1,200 | ~$193,200 | ~193% — still misleading |
| $300k ARR | $150,000 | $48,000 | $1,800 | ~$199,800 | ~67% |
| $500k ARR | $150,000 | $54,000 | $2,400 | ~$206,400 | ~41% |
| $1M ARR | $150,000 | $66,000 | $2,700 | ~$218,700 | ~22% |
| $3M ARR | $180,000 | $84,000 | $2,700 | ~$266,700 | ~9% |

**SaaS benchmarks (Bessemer/SaaS Capital):**

| Stage | Median G&A % Revenue |
|---|---|
| Pre-revenue / seed | Not meaningful |
| Series A ($1–5M ARR) | 15–25% |
| Series B ($5–20M ARR) | 8–15% |
| Growth stage ($20M+ ARR) | 4–8% |

**FORM targets:**

| ARR | FORM G&A target | Basis |
|---|---|---|
| < $300k | < $200k absolute/year | Founder sustenance + lean accounting |
| $300k–$1M | 20–30% ARR | Consistent with Bessemer Series A range |
| $1M–$3M | 10–18% ARR | Operating leverage from revenue growth without incremental G&A headcount |
| > $3M | < 10% ARR | Full-time CFO + established finance function; G&A growth decouples from revenue |

**Relationship to §28.9 S&M constraint:** §28.9 requires combined S&M + G&A ≤ 50% ARR at Series A ($1M ARR). If S&M is 30–40% at $1M ARR (consistent with §28.9 FORM targets), G&A must stay below 10–20%. The §30.6 targets above are consistent: at $1M ARR, G&A is modelled at 22%, but the S&M lower bound of 30% leaves only 20% for G&A to stay within the 50% combined ceiling. **Practical implication:** founder salary discipline at Phase 3 ($10k–$15k/month, not $20k+) is required to maintain G&A within bounds at $1M ARR. A $20k/month founder salary at $1M ARR would push G&A to 28–30%, which combined with 30–40% S&M exceeds the 50% ceiling.

---

### 30.7 Back-Office Hiring Sequence

G&A back-office hiring follows a fractional-before-full-time pattern consistent with the rest of the FORM hiring philosophy.

| Role | Trigger conditions | Monthly cost [ESTIMATE] | Notes |
|---|---|---|---|
| **Ukrainian accountant** (FOP/LLC filing) | Company formation | $150–$300 | First hire (not a headcount — contractor engagement). Required for statutory compliance. |
| **Virtual bookkeeper** (SaaS-experienced) | First enterprise invoice issued OR deferred revenue creates accounting complexity | $400–$700 | Remote contractor. Xero/QuickBooks fluency required. Must understand ASC 606 deferred revenue. |
| **Fractional CFO** | Series A closed AND investor reporting required AND ARR ≥ $300k | $3,000–$6,000/month | 10–15h/month engagement. Deliverables: Board pack, budget-vs-actual, cash flow forecast, Series B narrative. |
| **Financial Controller** (full-time) | ARR ≥ $1.5M AND statutory audit required AND fractional CFO time > 50h/month | $5,000–$8,000/month | First full-time finance hire. CEE market rate. Triggers statutory audit engagement. |

**GGA-HIRE-01 (Fractional CFO trigger gate):**
All four conditions must be met: (1) Series A closed; (2) investor reporting cadence established (Board meeting ≥ quarterly); (3) ARR ≥ $300k; (4) founder spending > 8h/week on financial reporting. If any condition is unmet, the founder continues to manage with the bookkeeper. A fractional CFO before these gates represents G&A overhead that does not yet deliver commensurate investor-grade value.

**GGA-HIRE-02 (Financial Controller gate):**
All three conditions must be met: (1) ARR ≥ $1.5M; (2) statutory audit required by investors or enterprise customers (some Fortune 500 procurement processes require vendor audited financials); (3) fractional CFO billing > 50h/month for two consecutive months. These conditions typically coincide with Series B prep.

---

### 30.8 DEC-030 G&A Audit Events

G&A audit events serve two purposes: (1) creating an immutable record of founder compensation decisions for investor diligence and future governance; (2) providing an auditable trail for changes to the finance function infrastructure.

| Event type | Severity | Key metadata fields | Retention | Trigger |
|---|---|---|---|---|
| `ops.founder_comp_updated` | HIGH | `phase` (`phase_0` / `phase_1` / `phase_2` / `phase_3`), `monthly_usd`, `structure` (`salary` / `fop_invoice` / `holdco_dividend`), `trigger_condition` (free-text, e.g., `first_enterprise_contract_signed`), `annual_run_rate_usd`, `updated_by` (founder user_id) | 7 years | Emitted on each compensation phase transition; also emitted at company formation to record Phase 0 ($0) decision. This is the OQ-27 resolution record. |
| `ops.finance_function_onboarded` | STANDARD | `role` (`ua_accountant` / `bookkeeper` / `fractional_cfo` / `financial_controller`), `contractor_id` (pseudonymous), `monthly_engagement_usd`, `start_date`, `engagement_type` (`contractor` / `employee`), `onboarded_by` (founder user_id) | 7 years | Emitted when each finance role engagement begins. Mirrors `ops.engineer_hired` pattern (§27.9). |
| `ops.ga_tooling_contracted` | STANDARD | `tool_name`, `plan_tier`, `monthly_cost_usd`, `seat_count`, `contract_start_date`, `contract_term_months`, `contracted_by` (founder user_id) | 3 years | Emitted when any admin tooling subscription is started or upgraded to a paid tier. Provides G&A tooling spend audit trail. Not required for free-tier activations. |

**HMAC chain note:** All three events follow the standard DEC-030 HMAC chain append logic (`docs/AUDIT_LOG_SCHEMA.md §3`). `ops.founder_comp_updated` at Phase 0 ($0) must be emitted at or before the first `fundraise.round_closed` event (§24.10) — investors should see a documented Phase 0 salary decision before the round closes, not a retroactive record.

---

### 30.9 Implementation Checklist

#### P0 — Before company formation or within 30 days of first capital receipt

| # | Task | Owner | Priority | Milestone | Definition of Done |
|---|---|---|---|---|---|
| 1 | Decide and document founder Phase 0 compensation (OQ-27 / OQ-GGA-03): confirm whether Phase 0 is $0 or > $0; if > $0, update §22.3 Hiring row accordingly; emit `ops.founder_comp_updated` event at Phase 0 | founder | **P0** | M1 | DEC-030 event emitted; §22.3 cash flow table updated if non-zero; OQ-27 and OQ-GGA-03 closed in `docs/DECISION_LOG.md` |
| 2 | Engage Ukrainian accountant for FOP/LLC statutory filing; confirm entity structure choice (FOP vs LLC vs HoldCo) aligns with §24.8 OQ-33 resolution; emit `ops.finance_function_onboarded` for `ua_accountant` role | founder | **P0** | M1 | Engagement letter signed; monthly filing schedule confirmed; DEC-030 event emitted |
| 3 | Set up Google Workspace (one Business Starter seat); set up Notion (free plan until second member); contract admin tooling as needed; emit `ops.ga_tooling_contracted` for each paid subscription | founder | **P0** | M1 | form.coach email live; Notion workspace created; DEC-030 events emitted for paid subscriptions |

#### P1 — Before first enterprise pilot goes live (M4–M5)

| # | Task | Owner | Priority | Milestone | Definition of Done |
|---|---|---|---|---|---|
| 4 | Register `ops.founder_comp_updated`, `ops.finance_function_onboarded`, and `ops.ga_tooling_contracted` as DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md`; validate Zod schema; deploy to `emit-audit-event` Worker endpoint | data-engineer + platform-engineer | **P1** | M4 | Three event types in AUDIT_LOG_SCHEMA.md; events emittable from admin console; test events in staging |
| 5 | Engage SaaS-experienced virtual bookkeeper; configure Xero/QuickBooks with deferred revenue recognition (12-month pro-ration of enterprise annual contracts per §18.1); emit `ops.finance_function_onboarded` | founder | **P1** | M4 (before first enterprise invoice) | Bookkeeper engaged; Xero deferred revenue configured; test journal entry for first pilot conversion confirms correct recognition |
| 6 | Implement Phase 1 founder compensation gate check: create a shared decision doc with the Phase 1 trigger conditions (§30.2.1); review at first enterprise contract close; emit `ops.founder_comp_updated` when Phase 1 triggers | founder | **P1** | M8–M12 | Comp updated if Phase 1 trigger met; DEC-030 event emitted; §22.3 updated if Phase 1 changes burn rate |

#### P2 — Post-Series A

| # | Task | Owner | Priority | Milestone | Definition of Done |
|---|---|---|---|---|---|
| 7 | Evaluate fractional CFO engagement against GGA-HIRE-01 gate: four conditions checked at Series A close + quarterly thereafter; engage fractional CFO if all four conditions met; emit `ops.finance_function_onboarded` for `fractional_cfo` role | founder | **P2** | Series A +1 month | Gate conditions documented; fractional CFO engaged or gate conditions recorded as unmet with rationale in DECISION_LOG.md |

---

### 30.10 Open Questions

**OQ-GGA-01: How should the founder compensation phase gate for Phase 1 account for UA wartime financial uncertainty?**

The Phase 1 trigger condition "first enterprise contract signed AND 90-day pilot conversion rate ≥ 50%" assumes that a contract signing is a reliable cash signal. However, an enterprise contract signed with a Ukrainian corporate entity during martial law may have payment timeline uncertainty (banking restrictions, FX controls, delayed settlement). If contract cash-in lags the signing date by > 60 days — which is possible under Ukrainian martial law banking conditions — Phase 1 comp should be deferred until the first invoice payment is received, not the contract signing date.

**Recommended resolution:** Phase 1 trigger is modified to: (a) first enterprise contract signed AND (b) first invoice payment received (cash confirmed in bank account). This removes the signing-to-payment lag risk and ensures the Phase 1 compensation decision is always cash-backed. Owner: founder. Priority: **P0 — before first enterprise contract is signed.** Resolution: document in `docs/DECISION_LOG.md` as part of OQ-27 closure.

**OQ-GGA-02: Should founder compensation flow through UA FOP, through the future HoldCo entity, or through a UA LLC employment contract?**

Three structures have materially different tax treatment and compliance implications:

1. **UA FOP (ФОП Group 3):** Effective tax rate ~5–7% on gross income (wartime 2% rate if eligible). Requires the company (LLC) to pay the FOP as a contractor. Advantages: low tax burden, simple structure. Risks: tax authority scrutiny if the FOP's sole client is the founder's own company (may be re-classified as employment).

2. **UA LLC employment:** Standard employment (зарплата). Tax cost to the company: ~18% PIT + 22% SSC on top of salary = ~40% on gross. Advantages: employment record, social benefits. Risks: expensive; 40% effective rate on founder salary is a meaningful cash drain pre-PMF.

3. **HoldCo dividend:** If a Delaware C-Corp or Cyprus HoldCo is formed (§24.8 OQ-33), founder withdraws as dividends (US: 15–20% qualified dividend tax; Cyprus: 0% if structured via HoldCo + personal residency). Advantages: lowest effective rate in some scenarios; clean cross-border structure. Risks: requires HoldCo, adds legal overhead, requires founder personal tax residency planning.

**Recommended resolution:** UA FOP Group 3 for Phase 0–2 (lowest cost at pre-Series A scale). Transition to HoldCo dividend structure at Series A close if OQ-33 HoldCo is formed. Owner: founder + UA tax counsel + OQ-33 counsel. Priority: **P0 — before first compensation payment.** Resolution: legal memo from UA tax counsel + OQ-33 resolution; decision documented in `docs/DECISION_LOG.md`.

**OQ-GGA-03: What is the founder's actual monthly cash requirement in Phase 0 — and is the §22.2 $0 assumption valid?**

§22.2 assumes $0 founder salary through Month 12. This assumption is valid only if the founder has sufficient personal runway (savings, partner income, family support) to cover living expenses without a company salary for 12+ months. If the founder requires > $0 from Day 1, §22.3 must be revised: the `Hiring` row increases by the founder compensation amount from Month 1, which reduces the runway calculated in §22.4 by 20–35% at the $150k initial cash level.

**This is not a judgment on the founder's financial situation — it is a planning input.** The model cannot be accurate without this number. The founder must document their personal runway and either confirm $0 for Month 1–12 or provide the monthly amount that updates §22.3.

**Recommended resolution:** Founder documents personal runway privately (not in this document — personal financial information stays outside the company record). The only output that enters this model is: (a) confirmed $0 for Phase 0, or (b) the monthly amount and start date for non-zero Phase 0 salary. Either way, emit `ops.founder_comp_updated` at company formation. Owner: founder. Priority: **P0 — before company formation.** Resolution: §22.3 cash flow table is accurate only after this decision is made.

---

*v2.0 (2026-06-05): §30 G&A & Founder Compensation Cost Model — the sixth and final functional cost centre in the §25–30 operating cost series, closing OQ-27 (founder salary policy, §22.8) and supplying the detailed model behind the `G&A and founder salary` line in §24.4 ($240k–$360k [ESTIMATE] over 18 post-Series A months). §30.1 scopes the section: founder comp, bookkeeping/accounting, admin tooling; explicitly excludes insurance (→ §25), legal/compliance overhead (→ §25), engineering infrastructure (→ §27), marketing tooling (→ §28). §30.2 founder compensation model: four phases — Phase 0 ramen $0 (M1 through first enterprise contract, §22.2 baseline); Phase 1 sustenance $2,500–$5,000/month (first enterprise contract signed + pilot conversion ≥ 50%); Phase 2 early-market $6,000–$10,000/month (ARR ≥ $100k + GM ≥ 70%); Phase 3 post-Series A $10,000–$15,000/month — provides annualised G&A impact table from $0 to $150k/year; UA FOP note — Group 3 FOP (5–7% effective rate) vs. LLC employment (~40% combined) vs. HoldCo dividend structure; wartime 2% rate caveat; interaction with §27 OQ-ENG-01 (same legal consultation covers both). §30.3 accounting/finance function: four-stage model from UA accountant ($150–$300/month M1–M4) through Xero + bookkeeper ($400–$700/month post-launch) through CPA review engagement ($8,000–$15,000 one-time at M12) through fractional CFO ($3,000–$6,000/month post-Series A); deferred revenue trigger note (first enterprise invoice = Xero deferred revenue config required); CPA firm selection criteria (SaaS ASC 606, CEE cross-border, Ukrainian statutory). §30.4 administrative tooling: five tools not covered in §27/§28 (Google Workspace $6/user, Notion Team $16/member, Loom Business $12.50/creator, Calendly Teams $16/member, 1Password Teams $4/user already in CRYPTOGRAPHY_POLICY); total $22–$51/month at 1–3 person stage, $135–$225/month at 5–10 person stage; Slack, Zoom free tiers excluded. §30.5 consolidated G&A cost table: four stages (M1–M4, M4–M12, M12–M18, M18–M30) with monthly and annualised totals ranging from $172/month (M1) to $22,025/month (Phase 3 high case); reconciliation to §24.4 Bear/Base/Bull G&A line — dominant variable is founder salary Phase 3 choice ($60k–$90k range impact). §30.6 G&A as % ARR: important caveat that ratio is meaningless pre-$300k ARR — absolute $ is the correct metric at early stage; five-row trajectory from 373% at $50k ARR (meaningless) to 9% at $3M ARR; SaaS benchmarks (Bessemer/SaaS Capital: 15–25% at Series A, 8–15% at Series B); FORM targets: < $200k absolute pre-$300k ARR, 20–30% at $300k–$1M, 10–18% at $1M–$3M, < 10% at > $3M; S&M interaction note — founder Phase 3 salary must stay ≤ $15k/month to maintain G&A < 20% at $1M ARR given 30–40% S&M ceiling from §28.9. §30.7 back-office hiring: four roles (UA accountant from M1, virtual bookkeeper at first enterprise invoice, fractional CFO post-Series A, financial controller at ARR ≥ $1.5M + audit required + fractional CFO > 50h/month); two gate conditions GGA-HIRE-01 (fractional CFO: 4 conditions) and GGA-HIRE-02 (financial controller: 3 conditions). §30.8 three DEC-030 G&A audit events (all 7-year retention except `ops.ga_tooling_contracted` at 3 years): `ops.founder_comp_updated` (HIGH — OQ-27 resolution record; emitted at company formation + each phase transition; must precede `fundraise.round_closed` for investor diligence integrity), `ops.finance_function_onboarded` (STANDARD — mirrors §27.9 `ops.engineer_hired` pattern; covers accountant, bookkeeper, fractional CFO, controller), `ops.ga_tooling_contracted` (STANDARD — paid subscription commits only; free-tier activations excluded). §30.9 seven-item implementation checklist: 3× P0 M1 (founder comp decision → DEC-030 event + §22.3 update if non-zero, UA accountant engagement, admin tooling setup), 3× P1 M4–M12 (DEC-030 event type registration + Zod schema, Xero deferred revenue config, Phase 1 gate review), 1× P2 post-Series A (fractional CFO gate evaluation). §30.10 three open questions: OQ-GGA-01 (Phase 1 trigger modification for UA wartime payment lag — P0, before first enterprise contract; recommended: trigger on cash receipt not contract signing), OQ-GGA-02 (UA FOP vs. LLC employment vs. HoldCo dividend — P0, before first compensation payment; recommended FOP Group 3 pre-Series A + HoldCo dividend at Series A), OQ-GGA-03 (OQ-27 resolution — is Phase 0 $0 assumption valid? — P0, before company formation; outcome updates §22.3). Cross-references: §22.2 (funding assumptions — Phase 0 salary assumption), §22.3 (cash flow table — Hiring row must be updated for non-zero Phase 0), §22.4 (runway sensitivity — Phase 0 salary shifts runway by 20–35% at $150k funding), §22.8 (OQ-27 — closed by this section), §24.4 (capital requirements — G&A line reconciled in §30.5), §24.8 OQ-33 (HoldCo requirement for institutional instruments — cross-referenced in OQ-GGA-02), §25.2 (insurance — cyber/E&O/D&O costs excluded from this section), §25.3 (per-deal legal overhead — excluded), §27.4 (engineering infrastructure — admin tooling exclusions defined in §30.4), §27.9 (ops.engineer_hired DEC-030 pattern — ops.finance_function_onboarded mirrors), §27.11 OQ-ENG-01 (UA FOP vs. contractor structure — same legal consultation covers both), §28.7 (marketing tooling — admin tooling exclusions defined in §30.4), §28.9 (S&M + G&A ≤ 50% ARR combined ceiling — §30.6 G&A targets calibrated to stay within this constraint), docs/DECISION_LOG.md (OQ-27, OQ-GGA-01/02/03 resolution records), docs/AUDIT_LOG_SCHEMA.md (DEC-030 event registry — three G&A events to be registered in P1 M4 checklist item 4), docs/CRYPTOGRAPHY_POLICY.md §3 (1Password Teams key inventory). Owner: founder + data-engineer.*

---

## §31 Pricing Architecture, Competitive Benchmarking & Discount Governance

> Owner: founder + product-strategist. Review: before any price change, before each enterprise sales cycle refresh (quarterly). Audience: founder, investor diligence, enterprise sales, future VP of Sales.

### 31.1 Purpose and Scope

This section documents the **derivation methodology** behind FORM's price points — both consumer ($19/month Pro) and enterprise ($12/$9/$6–8/seat/month) — and benchmarks them against the corporate wellness SaaS market. It also formalises the discount authority matrix referenced informally in §21 (pilot economics and conversion governance).

**Why this matters for the cost model:** Pricing is not independent of economics. A price that clears COGS by a thin margin cannot absorb CAC and still produce acceptable gross margin. A price that clears COGS by a wide margin but sits above WTP produces low conversion. This section proves that FORM's price points satisfy three conditions simultaneously: (1) COGS coverage with acceptable gross margin, (2) competitive positioning vs. direct corporate wellness substitutes, and (3) alignment with enterprise buyer WTP signals from the market.

**Scope:**
- §31.2 — Consumer Pro price derivation ($19/month)
- §31.3 — Enterprise price point derivation ($12/$9/$6–8/seat)
- §31.4 — Corporate wellness competitive benchmarking
- §31.5 — Price floor analysis (COGS-anchored minimum viable price)
- §31.6 — Discount authority matrix
- §31.7 — Pricing change governance (raise/reduce policy)
- §31.8 — DEC-030 pricing audit events
- §31.9 — Implementation checklist
- §31.10 — Open questions

**Out of scope:** Consumer geo-pricing tiers (→ §4 and §29), enterprise NRR and expansion pricing (→ §23), enterprise deal economics and pilot conversion rates (→ §21).

---

### 31.2 Consumer Pro Price Point Derivation ($19/month)

#### 31.2.1 WTP Anchor Analysis

FORM's $19/month Pro price was set against a matrix of consumer fitness and coaching substitutes, not against COGS alone. The logic: pricing below substitutes in perceived value destroys revenue unnecessarily; pricing above substitutes requires a demonstrably superior value proposition. [ESTIMATE on all competitor price data — confirm annually.]

| Product category | Example products | Monthly price range | Value prop overlap with FORM |
|---|---|---|---|
| **Human personal trainer (in-person)** | Local gym PT, boutique PT | $200–$600 | High (adaptive programming, feedback) — FORM undercuts 10–30× |
| **AI personal trainer (high-end)** | Future ($149/month), Caliber ($149–$199) | $149–$199 | Very high — FORM undercuts 8–10× |
| **Smart fitness app (personalised)** | Whoop (device + membership $30/mo), Oura Ring ($6/mo + $300 device) | $6–$30 | Partial (recovery data, readiness) |
| **AI coaching app (mid-tier)** | Fitbod ($15/mo), Ladder ($20/mo) | $14–$20 | Moderate (programming, no CV feedback) |
| **General fitness apps** | Nike Training Club (free), Apple Fitness+ ($10/mo) | $0–$10 | Low — no personalisation at scale |

**Key insight:** FORM competes on the "AI personal trainer" axis (vs. Future, Caliber), not the "fitness tracking" axis (vs. Whoop, Fitbod). At $19/month FORM is 85–90% cheaper than human-supervised AI coaching while delivering comparable session-by-session programming quality and adding CV pose feedback that Future does not offer. The $19 anchor is positioned just above Fitbod ($15) to signal higher value, and just below Ladder ($20) and Apple Fitness+ + Whoop bundle territory ($40+).

#### 31.2.2 COGS Coverage at $19/month

From §3–4: Pro infrastructure COGS = $0.50/user/month at modelled usage. At $19 list with 15% App Store Small Business Program fee:

```
Net revenue after store fee (SBP 15%):  $19.00 × 0.85 = $16.15
Infrastructure COGS:                    $0.50
Gross profit per user:                  $15.65
Gross margin (COGS-only):               $15.65 / $16.15 = 96.9%
```

At standard 30% App Store rate (above $1M annual revenue threshold):

```
Net revenue after store fee (30%):      $19.00 × 0.70 = $13.30
Infrastructure COGS:                    $0.50
Gross profit per user:                  $12.80
Gross margin (COGS-only):               $12.80 / $13.30 = 96.2%
```

**Conclusion:** $19/month is effectively unconstrained by COGS. Even at the worst-case store fee, COGS consumes only 3.8% of net revenue. The economic constraint on consumer pricing is conversion and retention, not margin. A price increase to $24–$29 would improve LTV by 26–53% without changing COGS. A price decrease to $14 would reduce LTV by 26% while barely changing COGS. The $19 price is set by competitive positioning, not economics.

#### 31.2.3 Price Sensitivity Range for Consumer

| Scenario | List price | SBP net | Gross margin | Notes |
|---|---|---|---|---|
| Bear (floor) | $12 | $10.20 | 95.1% | Competitive with Fitbod; lower LTV; high conversion potential |
| Current | $19 | $16.15 | 96.9% | Primary market positioning |
| Bull (ceiling) | $29 | $24.65 | 98.0% | Future/Caliber price compression territory; conversion risk |
| Stretch (validate first) | $39 | $33.15 | 98.5% | Only viable with strong brand and completion-guarantee framing |

**Price floor for consumer:** $12/month — at this price, annual Pro subscription = $144/year, App Store's alternative minimum billing threshold. Below $12, the economics are structurally sound (COGS is ~4% of revenue) but the product is positioned as a commodity app, not a coaching product. The $19 point preserves the "AI coach, not a fitness tracker" framing.

---

### 31.3 Enterprise Price Point Derivation ($12 / $9 / $6–8/seat/month)

#### 31.3.1 Derivation Methodology

Enterprise pricing was derived from three inputs, applied in order of constraint:

1. **COGS floor + target gross margin** — what price covers all-in COGS (infrastructure + CSM labour allocation) at an acceptable margin?
2. **Competitive WTP ceiling** — what do enterprise buyers currently pay for comparable corporate wellness platforms?
3. **Value anchor vs. consumer** — enterprise should cost less per seat than the consumer Pro plan (signal: bulk pricing) but more than a commodity app (signal: compliance, SSO, support).

These three inputs were then mapped to a tier structure that creates natural seat-count-driven upgrade pressure.

#### 31.3.2 Tier-by-Tier Derivation

**Starter tier ($12/seat/month · 50–200 seats)**

The Starter price equals the consumer Pro list price ($19 → $12 direct-billed, no App Store fee). Rationale: the value of removing the App Store fee (from ~$2.85–$5.70/seat/month) is passed partly to the customer as the "enterprise discount" and partly retained as margin improvement. At $12 direct-billed:

```
Revenue per seat:          $12.00
Infrastructure COGS (§3):   $0.34 (enterprise usage profile)
CSM labour allocation:       $1.67 (Starter email-only, shared; §26.2 = $150–200/month / 100 avg seats)
Total allocated COGS:        $2.01
Gross margin (fully loaded): ($12.00 − $2.01) / $12.00 = 83.3%
```

83% fully-loaded gross margin at Starter is acceptable for early-stage B2B SaaS (Bessemer Cloud Index 2024 median gross margin = 72–76%). Implementation COGS ($500–$700/deal for SSO setup at Starter, amortised over 24 months = $21–$29/month) reduces this by ~2–3pp in Year 1 only.

**Growth tier ($9/seat/month · 201–1,000 seats)**

Volume discount of 25% vs. Starter ($12 → $9). Rationale: at 300 seats, the deal is $32,400/year — large enough to justify a named CSM. The CSM cost per seat drops significantly at scale (§26.2 CSM model: one CSM serves 12–15 Growth accounts at $250–300/month fully-loaded; at 300 seats average, per-seat CSM cost = $300 / 300 = $1.00/seat/month).

```
Revenue per seat:          $9.00
Infrastructure COGS (§3):  $0.34
CSM labour allocation:     $1.00 (Growth named CSM; §26.2)
Total allocated COGS:      $1.34
Gross margin (fully loaded): ($9.00 − $1.34) / $9.00 = 85.1%
```

Higher fully-loaded gross margin than Starter despite lower ASP, because CSM labour cost per seat declines faster than price. The Growth tier is FORM's most efficient revenue band.

**Enterprise tier ($6–8/seat/month · 1,001+ seats)**

Custom pricing. The $7 midpoint (used in §8 and the pricing calculator) and the $6–8 range reflect:

- **Floor:** $6/seat — see §31.5 COGS floor analysis; below $6, fully-loaded gross margin falls below 60% at dedicated CSM coverage.
- **Ceiling:** $8/seat — above $8, large enterprise procurement will compare to Gympass/Wellhub corporate offering and find FORM expensive-relative-to-network.
- **Midpoint:** $7/seat for modelling purposes; actual negotiated rate depends on seat count, contract length, and whether white-label or dedicated support add cost.

```
Revenue per seat (midpoint): $7.00
Infrastructure COGS (§3):    $0.34
CSM labour (dedicated 6-acct max; §26.2 = $400–500/month / 1,000+ seats): $0.40–$0.50
Total allocated COGS:         $0.74–$0.84
Gross margin (fully loaded):  ($7.00 − $0.84) / $7.00 = 88.0% [ESTIMATE]
```

White-label add-on (CNAME + custom logo, §15.5): +$50–100/month incremental COGS absorbed within the negotiated rate. Does not change the price floor but compresses margin by ~0.5pp at 1,000 seats.

#### 31.3.3 ACV Floor by Tier

Minimum viable deal size — below which the deal does not generate adequate gross profit to justify sales motion and CSM overhead.

| Tier | Min seats | List price | Min ACV | Fully-loaded GM | Floor rationale |
|---|---|---|---|---|---|
| Starter | 50 | $12 | $7,200/year | 83% | Below 50 seats, email support still profitable but implementation cost (SSO setup ~$700) erodes Year 1 margin to < 70% |
| Growth | 201 | $9 | $21,708/year | 85% | Named CSM is economically justified at ≥ 201 seats; below 201 the CSM cost per seat exceeds the Starter email support model |
| Enterprise | 1,001 | $7 est. | $84,084/year | 88% | Dedicated CSM + custom SLA requires ≥ 1,001 seats to keep per-seat CSM below $0.50/seat/month |

**OQ-11 resolution note:** §11 OQ-11 asks whether the 50-seat Starter floor needs to rise to 75+ seats to maintain acceptable Year 1 GM. §31.3.3 analysis confirms: 50-seat floor at $12/seat produces $7,200/year ACV. Implementation cost ($700 one-time) amortised over 12 months = $58/month = $1.17/seat/month at 50 seats. This raises effective Year 1 COGS to $2.01 + $1.17 = $3.18/seat/month, reducing Year 1 gross margin to ($12.00 − $3.18) / $12.00 = 73.5%. Still acceptable. Floor of 50 seats is defensible; 75-seat floor would improve Year 1 margins to ~76% but reduces TAM. Recommendation: keep 50-seat floor; monitor Year 1 margins on first five Starter deals and update if < 70%.

---

### 31.4 Corporate Wellness Competitive Benchmarking

> [ESTIMATE] on all competitor pricing. Corporate wellness pricing is almost never publicly listed; figures are derived from published reports (Mercer, SHRM, KFF), industry analyst estimates, procurement conversations documented in public forums (G2, Capterra reviews), and job listings that reference budget ranges. Treat all figures as directional, not contractually verified.

#### 31.4.1 Primary Competitive Set (Direct Substitutes)

These platforms compete directly for the same HR/People budget as FORM. Enterprise buyers will compare FORM to at least one of these during procurement.

| Platform | Category | Est. price range / seat / month | Key differentiator | FORM positioning |
|---|---|---|---|---|
| **Wellhub (fmr. Gympass)** | Gym network access + wellbeing apps | $8–$45 (plan-dependent) [ESTIMATE] | Access to 50k+ gym/studio network in 11 countries; well-known brand | FORM is a coaching product, not a gym access network. Not direct substitutes for companies with distributed/remote workforces. |
| **Headspace for Work** | Guided meditation + sleep | $10–$14/seat/month [ESTIMATE] | Strong consumer brand; Calm competitor | No exercise programming, no CV feedback, no wearable integration. HR might buy both. FORM is not a mental wellness product. |
| **Calm for Business** | Meditation, sleep, stress | $10–$15/seat/month [ESTIMATE] | Sleep and stress focus; large library of content | Same as Headspace — no physical coaching component. |
| **Virgin Pulse (now Personify Health)** | Comprehensive wellness platform | $40–$75/seat/month [ESTIMATE] | Incentive management, biometric screening, health coaching | Broad platform but legacy architecture; used by large self-insured employers for incentive-based programs. FORM explicitly does not sell incentive/risk-scoring features. |
| **Noom for Work** | Behavioural weight management | $30–$50/seat/month [ESTIMATE] | Structured weight loss program; strong outcomes data | Weight-loss framing conflicts with FORM's body-neutrality policy. FORM cannot and will not compete on this axis. |
| **Peloton Corporate (Peloton for Business)** | Connected fitness hardware + subscription | $20–$40/seat/month (hardware lease included) [ESTIMATE] | Brand recognition, live classes | Hardware dependency; facilities-based. Remote/distributed workforces underserve. |
| **Whoop for Teams** | Recovery monitoring wearable | $18–$30/seat/month (device included) [ESTIMATE] | HRV/recovery science; used by elite sports teams | Device dependency; no exercise programming; strong in athletic performance context. Wearable-only, not coaching. |
| **Future (enterprise)** | Human coach via app | $99–$149/seat/month [ESTIMATE] | Real human coaches; strong outcomes | 8–12× FORM's Enterprise price. Human coaching supply constraint limits scale. |

#### 31.4.2 FORM Competitive Positioning Matrix

```
                    Low price / seat ◄────────────────────────► High price / seat
                         $5-10          $10-20           $20-50            $50+
                          │              │                │                 │
Exercise programming  ┌───┤              ├────────────────┤                 │
+ AI coaching         │   │              │   ← FORM →    │                 │
                      │   │              │   ($6-12)     │                 │
                      │   │              │               ├──── Future ─────┤
                      │   │   Headspace  │               │   ($99-149)     │
                      │   │   Calm       │               │                 │
Mental wellness       │   │  ($10-15)    │               │                 │
                      │   │              │               │                 │
Gym network access    │   ├── Wellhub ───┤               │                 │
                      │   │  ($8-45)     │               │                 │
                      │   │              │               │                 │
Incentive platform    │   │              │               ├── VirginPulse ──┤
                      │   │              │               │  ($40-75)       │
                      └───┴──────────────┴───────────────┴─────────────────┘
```

**Positioning conclusion:** FORM occupies the "$6–12 AI coaching" quadrant — the only product in this price band that offers exercise programming, CV pose feedback, and wearable integration without device dependency. The nearest competitor on price is Wellhub, but Wellhub is a gym access network, not a coaching product. The nearest competitor on features is Future, at 8–12× FORM's price.

**Key enterprise sales insight:** FORM is not replacing Wellhub (gym network) or Headspace (mental wellness). It is replacing the gym stipend ($600–$1,800/employee/year), which is untracked, non-measurable, and generates no engagement data for HR. The substitution story for buyers is: "Cancel or redirect your gym stipend budget; use FORM instead. You get measurable activation, engagement reporting, and a coaching outcome — for $72–$144/employee/year."

#### 31.4.3 Price Anchoring in Sales Conversations

When an enterprise prospect asks "how does your pricing compare to the market," the recommended framing is:

1. **Do not compare to Wellhub** — different category (gym access vs. coaching). Wellhub comparison anchors the price wrong.
2. **Compare to the gym stipend** — $600–$1,800/year vs. $72–$144/year. FORM at $12/seat is 8–12× cheaper than the stipend it replaces, with measurable outcomes.
3. **Compare to Future for the coaching angle** — $99–$149/month (human coach) vs. $12/month (AI coach with CV feedback). FORM delivers comparable programming quality at 8–12× lower cost.
4. **Acknowledge Headspace/Calm for HR budgets** — some procurement teams will have both. Frame as additive: FORM for physical activity; mindfulness products for mental wellness. The ROI model (§20) covers both value levers independently.

---

### 31.5 Price Floor Analysis (COGS-Anchored Minimum Viable Price)

The price floor is the minimum price at which FORM can serve a seat while covering all directly attributable costs and maintaining a positive contribution margin. This is distinct from the gross margin target — it is the hard floor below which a deal destroys value regardless of strategic reasons to close it.

#### 31.5.1 Price Floor Calculation by Tier

The price floor is defined as: `Infrastructure COGS + Labour COGS allocation + 10% buffer`. The 10% buffer accounts for billing overhead, payment processing fees, and unmodelled cost variance.

| Tier | Infra COGS/seat | CSM labour/seat | Compliance overhead/seat | Total loaded COGS | + 10% buffer | **Hard price floor** |
|---|---|---|---|---|---|---|
| Starter (50–200 seats) | $0.34 | $1.67 | $0.15 [ESTIMATE] | $2.16 | $2.38 | **$3.00/seat/month** |
| Growth (201–1,000 seats) | $0.34 | $1.00 | $0.15 | $1.49 | $1.64 | **$2.00/seat/month** |
| Enterprise (1,001+ seats) | $0.34 | $0.45 | $0.15 | $0.94 | $1.03 | **$1.50/seat/month** |

**These floors are theoretical — they indicate the breakeven COGS point, not the minimum contractually permitted price.** The contractual price floors (below which no approval level can go) are defined in §31.6.3 and are set significantly above the COGS floor to preserve gross margin.

**Compliance overhead per seat** ($0.15 [ESTIMATE]) allocates a share of the §25 Enterprise Compliance & Legal Infrastructure Cost Model across the enterprise seat base. At $50k ARR (~400 seats at $10.4/seat average), compliance cost = $600–$800/month; per-seat allocation ≈ $1.50–$2.00/seat. At $500k ARR (~4,000 seats), compliance cost scales more slowly (not linear with seats); per-seat allocation drops to ~$0.10–$0.20/seat. The $0.15 figure is a steady-state estimate at $300k–$1M ARR scale.

#### 31.5.2 Why the Contractual Floor Must Exceed the COGS Floor

The COGS floor ($1.50–$3.00/seat) leaves zero contribution margin for fixed cost absorption, sales cost amortisation, or product R&D. At the COGS floor, FORM is a break-even fulfilment operation, not a business. The contractual discount floors in §31.6 are set at 50% of list price per tier, which is:

- Starter floor: $12 × 50% = **$6.00/seat/month**
- Growth floor: $9 × 50% = **$4.50/seat/month**
- Enterprise floor: $8 × 50% = **$4.00/seat/month**

At these contractual floors, fully-loaded gross margin still exceeds 55% — sufficient to absorb fixed costs at $300k+ ARR. See §31.6 for floor governance.

---

### 31.6 Discount Authority Matrix

This section formalises the discount authority governance first referenced in §21 (Pilot Economics, Conversion Governance & Discount Authority Matrix). The matrix governs every discount from standard list price, including: multi-year contract discounts, upfront payment discounts, pilot conversion discounts, and ad-hoc negotiated concessions.

#### 31.6.1 Standard Discount Schedule (Pre-Approved — No Approval Required)

These discounts are built into the pricing calculator (`pricing-enterprise.html`) and can be offered by any sales representative without additional approval:

| Discount type | Amount | Conditions | Stackable? |
|---|---|---|---|
| 2-year contract | –15% | Minimum 2-year term commitment | Yes — stackable with upfront |
| 3-year contract | –25% | Minimum 3-year term commitment | Yes — stackable with upfront |
| Annual upfront payment | –10% | Full annual invoice paid upfront (not monthly) | Yes — stackable with contract discount |
| **Maximum pre-approved effective discount** | **–32.5%** | 3-year + upfront (multiplicative: 1 − (1−0.25)×(1−0.10) = 32.5%) | — |

**Discount stacking rule:** All discounts are multiplicative, not additive. A 25% contract discount + 10% upfront discount = 32.5% total reduction, not 35%. This is enforced in the pricing calculator logic.

#### 31.6.2 Non-Standard Discount Schedule (Approval Required)

Discounts beyond the pre-approved schedule require explicit approval before presentation to the customer. All approved non-standard discounts must be documented as a DEC-030 `enterprise.pricing_exception_approved` event (see §31.8) before the quote is sent.

| Discount level | Effective rate after discount | Approval required | Conditions |
|---|---|---|---|
| Up to –40% total | Starter: ≥ $7.20/seat · Growth: ≥ $5.40/seat · Enterprise: ≥ $4.80/seat | Founder only | Strategic account, volume ≥ 500 seats, 2-year minimum |
| Up to –50% total (contractual floor) | Starter: ≥ $6.00/seat · Growth: ≥ $4.50/seat · Enterprise: ≥ $4.00/seat | Founder + investor lead (pre-Series A) | Exceptional circumstances only; requires written rationale |
| Below contractual floor | Below $6.00/$4.50/$4.00/seat | NOT PERMITTED | No approval level overrides the contractual price floor |

**Price floor enforcement:** The contractual price floors ($6.00/$4.50/$4.00/seat) cannot be waived. A deal that requires pricing below the floor must be restructured (fewer seats, shorter contract, reduced features) or declined. Pre-Series A: any exception attempt requires founder + investor lead countersignature. Post-Series A: requires board approval.

#### 31.6.3 Contractual Price Floor Summary

| Tier | List price | Contractual floor | Floor as % of list | Maximum total discount |
|---|---|---|---|---|
| Starter | $12.00/seat | **$6.00/seat** | 50% | 50% |
| Growth | $9.00/seat | **$4.50/seat** | 50% | 50% — Note: maximum pre-approved (32.5%) will not breach floor; additional 17.5% available with approval |
| Enterprise | $8.00/seat (ceiling of range) | **$4.00/seat** | 50% | 50% |
| Enterprise | $6.00/seat (floor of range) | **$4.00/seat** | 67% | 33% (note: floor constrains enterprise deals heavily at $6/seat list) |

**Note on Enterprise $6/seat edge case:** At the low end of Enterprise list pricing ($6/seat, typically for deals ≥ 2,000 seats on a 3-year contract), the contractual floor of $4.00/seat allows only a 33% maximum discount. The pre-approved 3-year + upfront discount (32.5%) nearly exhausts the available headroom. Any additional negotiated concession requires founder approval and cannot result in a quote below $4.00/seat.

#### 31.6.4 Non-Pricing Concessions (Do Not Require Pricing Approval)

The following concessions can be offered without pricing approval because they affect delivery scope, not unit economics:

- Extended pilot duration (90 days → 180 days) — requires founder approval for accounts > 500 seats (see §21)
- Custom SLA upgrade from Growth to Enterprise SLA — included in Enterprise list, not a pricing concession
- Additional admin seats (above the standard RBAC seat count) — currently included in all tiers
- Priority SSO setup scheduling — included in CSM scope
- Executive sponsor introduction or architecture call — no cost

**What requires pricing approval:** Any change to the per-seat rate, total contract value, or payment terms that reduces revenue relative to standard list.

---

### 31.7 Pricing Change Governance

#### 31.7.1 Consumer Pricing Changes

FORM may adjust the consumer Pro price ($19/month) in response to market conditions, competitor pricing moves, or strategic repositioning. Governance rules:

**Minimum notice:** 30 days to existing subscribers before any price increase takes effect. App Store customers receive pricing change notice via the platform's standard upgrade flow. Direct subscribers receive email.

**Grandfathering policy:** Existing annual subscribers are locked at their contracted rate for the remainder of their annual term. Price increases apply at the next renewal. No mid-term price increases on annual contracts.

**Price decrease:** No grandfathering obligation. Price decreases take effect immediately for new subscribers; existing annual subscribers may contact support for credit adjustment (not obligated, but recommended for NPS).

**Approval required for consumer price change:** Founder-level decision. Document in `docs/DECISION_LOG.md` with WTP research and competitive data supporting the change. Emit `enterprise.consumer_price_updated` DEC-030 event.

#### 31.7.2 Enterprise Pricing Changes

Enterprise pricing changes are significantly more complex because of multi-year contracts and contractual commitments.

**List price changes:** Any change to standard list prices ($12/$9/$6–8/seat) requires:
1. Review of all active contracts to identify customers at list price (most will have negotiated rates unaffected by list change)
2. 90-day notice to existing annual customers at renewal (not mid-term)
3. Investor lead consultation pre-Series A (post-Series A: board notification)
4. Update to `pricing-enterprise.html` calculator and `docs/ENTERPRISE.md`
5. DEC-030 `enterprise.list_price_updated` event

**Annual price indexation:** Starting at the second renewal of any multi-year contract, FORM may apply an annual price escalation clause (CPI-linked, typically 3–5%) if the MSA includes this clause. The MSA template (`docs/MSA_TEMPLATE.md`) should include a standard price escalation clause of "CPI + 1%" capped at 5% per year. This clause enables §23.6 annual price indexation as an NRR expansion lever.

**Rate lock commitment:** Customers on a signed multi-year contract are rate-locked for the committed term at their contracted per-seat rate. FORM cannot increase the per-seat rate during the contracted term regardless of list price changes. At renewal, the customer is subject to current list pricing (with applicable discounts from §31.6).

#### 31.7.3 When to Consider a Price Increase

The following signals indicate that the current price may be leaving money on the table:

| Signal | Implication | Threshold for action |
|---|---|---|
| Win rate > 80% in competitive deals | Pricing is below WTP ceiling | Consider 10–15% list price increase at next review |
| Prospect never negotiates (accepts list price) | No price resistance | Possible underprice; survey 10 customers on perceived value |
| Churn rate near zero AND NPS > 60 | High satisfaction; loyalty signals | Safe to increase at renewal without significant churn |
| COGS increases materially (Anthropic volume rate changes) | Margin compression | Pass-through price increase justified; 60-day notice |

| Signal | Implication | Action |
|---|---|---|
| Win rate drops below 50% | Possible overprice OR competitive entry | Diagnose reason before cutting price; price is often not the issue |
| Competitor cuts price by > 20% | Competitive pressure | Respond with value differentiation first, price last |
| Conversion rate from pilot drops below 40% | Possible price-to-value gap | Investigate activation quality before cutting price |

---

### 31.8 DEC-030 Pricing Audit Events

Pricing decisions are high-governance, investor-relevant events. All non-standard pricing decisions must produce a tamper-evident DEC-030 audit record before the quote or contract is presented to the customer.

| Event type | Severity | Key metadata fields | Retention | Trigger |
|---|---|---|---|---|
| `enterprise.pricing_exception_approved` | HIGH | `deal_id`, `tier` (`starter`/`growth`/`enterprise`), `list_price_usd`, `approved_price_usd`, `effective_discount_pct`, `seat_count`, `contract_years`, `approver_user_id`, `approval_level` (`founder`/`founder_plus_investor`), `rationale_text` (free-text, 200 char max), `quote_ref` | 7 years | Emitted when any non-standard discount (beyond pre-approved schedule) is approved. Must be emitted **before** the quote is sent to the prospect. If the deal closes, `enterprise.contract_signed` (§23.7) links back to this event via `pricing_exception_event_id` field. |
| `enterprise.consumer_price_updated` | HIGH | `previous_price_usd`, `new_price_usd`, `effective_date`, `grandfathering_policy` (free-text), `updated_by` (founder user_id), `rationale_ref` (link to DECISION_LOG.md entry) | 7 years | Emitted when consumer Pro list price is changed. Includes explicit grandfathering policy statement. |
| `enterprise.list_price_updated` | HIGH | `tier` (`starter`/`growth`/`enterprise`), `previous_price_usd`, `new_price_usd`, `effective_date`, `updated_by` (founder user_id), `active_contracts_affected` (count of contracts at prior list rate), `rationale_ref` | 7 years | Emitted when enterprise tier list prices are updated. Captures count of active contracts that will face price change at renewal. |
| `enterprise.price_floor_override_requested` | CRITICAL | `deal_id`, `tier`, `requested_price_usd`, `floor_price_usd`, `requester_user_id`, `decision` (`approved`/`denied`/`restructured`), `approver_chain` (array of user IDs) | 7 years | Emitted if anyone attempts to quote below the contractual price floor. Even a denied request creates an audit record. Decision `denied` means the quote was not sent; `restructured` means the deal terms were changed to bring it within floor. |

**HMAC chain note:** All four events follow the standard DEC-030 append logic (`docs/AUDIT_LOG_SCHEMA.md §3`). `enterprise.price_floor_override_requested` must generate a record even when the outcome is `denied` — the absence of a denied request does not mean no floor breach was attempted; it means the system is working. Auditors can query for CRITICAL-severity pricing events to verify floor integrity over the observation window.

---

### 31.9 Implementation Checklist

#### P0 — Before first enterprise quote is sent

| # | Task | Owner | Priority | Milestone | Definition of Done |
|---|---|---|---|---|---|
| 1 | Register `enterprise.pricing_exception_approved`, `enterprise.consumer_price_updated`, `enterprise.list_price_updated`, `enterprise.price_floor_override_requested` as DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md`; validate Zod schemas; deploy to `emit-audit-event` Worker endpoint | data-engineer + platform-engineer | **P0** | M4 (before first enterprise pilot launch) | Four event types in AUDIT_LOG_SCHEMA.md; emittable from admin console or CRM workflow; test events in staging with valid HMAC chain |
| 2 | Add price floor enforcement to pricing calculator (`pricing-enterprise.html`): client-side validation that prevents quote generation below $6.00/$4.50/$4.00 contractual floors; display clear error message if floor is breached | platform-engineer | **P0** | M4 | Calculator cannot produce a quote below floor; floor breach attempt logs to console with a clear error; no DEC-030 event required for client-side prevention (only for server-side approval requests) |
| 3 | Document the pricing exception approval process in the CRM (or, pre-CRM, as a written procedure in `docs/DECISION_LOG.md`): step-by-step for how founder approves a non-standard discount, including the requirement to emit DEC-030 event before quote is sent | founder | **P0** | M4 | Written procedure exists; founder can follow it; first test non-standard quote produces a DEC-030 event |

#### P1 — Before first enterprise contract is signed

| # | Task | Owner | Priority | Milestone | Definition of Done |
|---|---|---|---|---|---|
| 4 | Add annual price escalation clause to `docs/MSA_TEMPLATE.md`: "CPI + 1%, capped at 5% per year, applied at second and subsequent renewals; requires 60-day written notice" | compliance-officer + founder | **P1** | M5 (before first MSA is sent) | MSA template updated; clause reviewed by legal counsel; rate lock for current term explicitly stated |
| 5 | Create a competitive pricing intelligence update process: once per quarter, update §31.4 with current market intelligence from G2 reviews, procurement conversations, and industry analyst reports; emit no DEC-030 event (internal research artefact, not a pricing decision) | product-strategist | **P1** | M6 (first update at 90 days post-launch) | §31.4 reviewed and updated (or explicitly confirmed as current); date of last review noted at top of §31.4 table |
| 6 | Update `docs/COMPETITIVE.md` to add a "Corporate Wellness Enterprise" section that covers the same platforms as §31.4, maintaining separate documents for consumer competitors (current COMPETITIVE.md) and enterprise substitutes (new section) | product-strategist | **P1** | M6 | New section in COMPETITIVE.md; no duplication with §31.4 (cross-reference only) |

#### P2 — Post-launch (first 6 months)

| # | Task | Owner | Priority | Milestone | Definition of Done |
|---|---|---|---|---|---|
| 7 | Conduct WTP validation study with 5–10 enterprise pilot participants: structured interview on perceived value vs. price, competitive alternatives considered, and price at which they would not proceed; update §31.2.1 and §31.3.2 with real data | product-strategist + customer-success | **P2** | M9 (after first cohort of pilots completes 90 days) | Interview notes in research repo; §31 price derivation updated from [ESTIMATE] to [VALIDATED] where confirmed |
| 8 | Run first annual pricing review: gather win/loss data, analyse whether pre-approved discount schedule matches field reality, update §31.4 competitive benchmarks, revise floors if COGS have changed materially | founder + product-strategist | **P2** | M12 (first anniversary of commercial launch) | Pricing review meeting notes; §31 updated; any price change decisions documented in DECISION_LOG.md |

---

### 31.10 Open Questions

**OQ-PRICE-01: Should FORM offer a non-profit or education discount tier?**

Non-profit organisations and educational institutions (universities with student wellness programs) represent a meaningful potential market but have constrained procurement budgets. A standard "50% off Growth list = $4.50/seat" for verified 501(c)(3)s or accredited universities would stay above the contractual floor but require approval under §31.6.2. The strategic question: does a non-profit/edu tier create pipeline for subsequent commercial deals (companies who saw FORM at university and adopt it at work), or does it create complexity without commercial return?

**Recommended resolution:** Defer non-profit/edu pricing until three commercial deals have closed. If three commercial customers mention they discovered FORM in a non-profit/edu context, the conversion channel is real and the discount tier is worth formalising. Owner: product-strategist + founder. Priority: P2. Resolution: document in DECISION_LOG.md after first five commercial closings.

**OQ-PRICE-02: At what ARR level should FORM review its consumer $19/month price?**

§31.2.3 establishes that FORM could plausibly increase consumer pricing to $24–$29/month without significant COGS impact. The question is when — too early and it signals desperation; too late and it leaves LTV on the table. The price increase is most defensible after: (a) NPS ≥ 60 across three consecutive months, (b) D90 retention ≥ 40%, and (c) consumer ARR ≥ $200k (proving the product has paying users). At those conditions, a price increase to $24/month raises consumer LTV by 26% (from §14 model) without requiring new features.

**Recommended resolution:** Trigger condition for consumer price review = D90 ≥ 40% AND NPS ≥ 60 AND consumer ARR ≥ $200k. At those conditions, survey 100 random Pro subscribers on price sensitivity before changing. Owner: growth-lead + product-strategist. Priority: P2. Resolution: checkpoint at M12 (first annual review per §31.9 item 8).

**OQ-PRICE-03: How should FORM price CV-optional enterprise deals (e.g. customers in environments where camera use is restricted by policy)?**

Some enterprise customers — particularly financial services, government contractors, and manufacturing floors — may have IT policies restricting camera use on corporate devices. For these customers, the CV pose feedback feature (FORM's primary technical moat) is unavailable. Should the price be lower for CV-disabled seats, or should the price remain the same because the coaching, programming, and wearable integration value remains intact?

**Implication:** A two-SKU model (CV-enabled vs. CV-optional) adds quoting complexity and potentially signals that CV is the only differentiating value. A single-price model (same price regardless of CV availability) is simpler but may face procurement objection: "we're paying for something we can't use."

**Recommended resolution:** Single-price model for now (pre-Series A). The coaching, wearable integration, and admin dashboard value is sufficient to justify the price without CV for most use cases. If CV-restriction becomes a recurring objection in > 3 enterprise deals, create a dedicated "Professional" SKU at $8/seat (between Growth and Enterprise) without CV. Owner: product-manager + founder. Priority: P2. Resolution: track CV-restriction objections in CRM; review at M12 pricing review.

---

---

## 32. Pricing Exception Approval Procedure

> Owner: `founder` + `compliance-officer`. Review: after every non-standard discount, minimum annually.
> References: §31.5 (price floors), §31.6 (discount authority matrix), §31.8 (DEC-030 events), `docs/AUDIT_LOG_SCHEMA.md`, `docs/ENTERPRISE.md`, DEC-038.
> Closes: `docs/COST_MODEL.md §31.9` item 3 (P0, M4).
>
> Pre-CRM note: until a CRM is deployed, all exception requests and approvals are tracked via Linear tickets or Slack threads preserved in the `pricing-exceptions` channel. Each approved exception must also have a corresponding DEC entry in `docs/DECISION_LOG.md`.

---

### 32.1 Scope

This procedure applies to every enterprise quote where the proposed effective per-seat rate falls **below the pre-approved maximum discount** from §31.6.1 (–32.5% multiplicative: 3-year contract + annual upfront).

| What this procedure covers | What it does NOT cover |
|---|---|
| Any discount that, alone or stacked, exceeds –32.5% vs list price | Standard discounts within the pre-approved schedule (no approval needed) |
| Any per-seat rate below: Starter $8.10 · Growth $6.075 · Enterprise $5.40 | Non-pricing concessions (extended pilot, priority scheduling, exec intro — see §31.6.4) |
| Any combination of discounts that approaches or reaches the contractual floor | Consumer Pro price changes (→ §31.7.1) |
| Any below-floor rate request (cannot be approved — see §32.4) | Enterprise list price changes (→ §31.7.2) |

**Quick test — does this discount need approval?**

```
effective_rate = list_price × (1 − contract_discount) × (1 − upfront_discount)

If effective_rate ≥ (list_price × 0.675):  standard schedule, no approval
If effective_rate < (list_price × 0.675):  this procedure applies
If effective_rate < contractual_floor:     cannot proceed — see §32.4
```

Contractual floors (from §31.5): Starter $6.00 · Growth $4.50 · Enterprise $4.00/seat/month.

---

### 32.2 Approval Authority

From §31.6.2:

| Effective discount after all stacking | Minimum required approver | Conditions |
|---|---|---|
| –32.5% to –40% | **Founder** (written) | Strategic account; volume ≥ 500 seats; 2-year minimum term |
| –40% to –50% / contractual floor | **Founder + investor lead** (both written, pre-Series A) | Exceptional circumstances only; written rationale required in advance |
| Below contractual floor | **Not permitted** — no approval level overrides the floor | See §32.4 |

**Post-Series A:** replace "investor lead" with "board approval" for the –40% to floor band.

---

### 32.3 Step-by-Step Procedure

#### Step 1 — Confirm that the exception applies

Before opening the approval process, calculate:

1. Seat count and tier (Starter / Growth / Enterprise)
2. Proposed contract length (1 / 2 / 3 year)
3. Upfront payment elected? (yes/no)
4. Any additional negotiated discount beyond the standard schedule
5. Effective per-seat rate after all proposed discounts stacked multiplicatively

If the effective rate after stacking is **at or above** `list_price × 0.675` — the standard schedule applies. Stop: no approval needed.

If below, continue.

#### Step 2 — Check the contractual floor

| Tier | List price | Contractual floor |
|---|---|---|
| Starter | $12.00/seat | **$6.00/seat** |
| Growth | $9.00/seat | **$4.50/seat** |
| Enterprise (reference) | $8.00/seat | **$4.00/seat** |

If the proposed rate is **below the contractual floor**: STOP. Route to §32.4 immediately. This deal must be restructured or declined. No approval unlocks a below-floor rate.

If at or above the floor, continue.

#### Step 3 — Prepare the approval request

Create a written approval request (Linear ticket, Slack thread in `#pricing-exceptions`, or email to founder). The request must include:

```
Account:           [name — can be anonymised as "Prospect A — [industry]" pre-NDA]
Tier:              [Starter / Growth / Enterprise]
Seat count:        [N seats]
Contract length:   [1 / 2 / 3 years]
Upfront payment:   [yes / no]
List price/seat:   $[list]
Standard max rate: $[list × 0.675] (–32.5%)
Proposed rate:     $[proposed]/seat/month
Effective discount: [X]% vs list
Contractual floor: $[floor]/seat — respected? [yes]
ACV at this rate:  $[seats × rate × 12] annually

Justification:
- [Strategic reason: anchor account, reference customer, competitive pressure, etc.]
- [Volume / market context]
- [Risk of losing the deal at standard pricing]

Margin impact:
- Fully-loaded gross margin at proposed rate: [X]% (vs [Y]% at standard list)
- Annual revenue impact vs standard pricing: –$[Z]/year

Precedent risk:
- [Are other accounts in the same vertical / size band that could use this as a floor?]
- [Is this discount justified by unique deal context, not repeatable as a category?]
```

The request must be written and preserved before approval is sought. Verbal approval is not sufficient.

#### Step 4 — Obtain approval

Route per §32.2:

**For –32.5% to –40% discounts (founder-only):**
1. Send the approval request to the founder directly (Slack DM or Linear assign)
2. Founder replies in writing with explicit "approved at $[rate]/seat" — not just "LGTM" or a thumbs-up
3. Screenshot or preserve the approval reply. Note the timestamp.
4. If the founder does not respond within 48 hours, the quote cannot be sent. Escalate via a second message referencing the deal timeline.

**For –40% to floor (founder + investor lead, pre-Series A):**
1. Send the approval request to both simultaneously
2. Both must reply in writing with explicit approval
3. Investor lead approval must precede or be simultaneous with founder approval — sequential approval is acceptable; founder approval alone is not sufficient at this level
4. Preserve both approvals before proceeding

Approval must be obtained **before the quote is prepared or sent**. A quote sent without prior approval is a compliance violation and triggers an immediate DEC-030 `enterprise.price_floor_override_requested` event at CRITICAL severity regardless of whether the floor was technically breached.

#### Step 5 — Emit the DEC-030 event

After written approval is in hand and **before** the quote is opened in any document tool:

```sql
-- Emit via the emit-audit-event Worker endpoint or directly via Edge Function
-- Reference: docs/AUDIT_LOG_SCHEMA.md §enterprise.pricing_exception_approved

INSERT INTO audit_log_events (
  id,
  action,
  actor_id,
  resource_type,
  resource_id,
  metadata,
  severity,
  occurred_at,
  prev_hash,
  hash
)
SELECT
  gen_random_uuid(),
  'enterprise.pricing_exception_approved',
  '<founder_user_uuid>',
  'enterprise_quote',
  '<opportunity_id_or_slug>',
  jsonb_build_object(
    'tier',                   '<starter|growth|enterprise>',
    'seats',                  <seat_count>,
    'contract_years',         <contract_years>,
    'upfront_payment',        <true|false>,
    'list_price_per_seat',    <list_price>,
    'approved_rate_per_seat', <approved_rate>,
    'effective_discount_pct', <discount_as_decimal>,   -- e.g. 0.38 for 38%
    'list_price_x_0675',      <list_price * 0.675>,    -- pre-approved ceiling
    'contractual_floor',      <floor>,
    'floor_respected',        true,
    'approver_founder',       '<founder_user_uuid>',
    'co_approver_investor',   '<investor_uuid_or_null>',
    'rationale_ref',          '<linear_ticket_url_or_slack_thread_permalink>',
    'account_anonymised',     '<Prospect A — Fintech>',
    'avc_annual',             <seats * approved_rate * 12>
  ),
  'HIGH',
  now(),
  <prev_hash_from_latest_audit_row>,
  hmac(<canonical_payload>, <HMAC_AUDIT_CHAIN_KEY>, 'sha256')
FROM (
  SELECT hash AS prev_hash
  FROM audit_log_events
  ORDER BY occurred_at DESC
  LIMIT 1
) latest;
```

**`floor_respected` must be `true`.** If it is false, stop immediately — this signals an attempted below-floor quote and triggers §32.4 regardless of whether the approval was obtained.

**`pricing_exception_event_id`** — record the UUID returned by this INSERT. It must be linked to the `enterprise.contract_signed` event when the deal closes, creating a two-event chain that proves the exception was approved before the contract was signed.

#### Step 6 — Prepare and send the quote

Only after the DEC-030 event has been written to the audit log:

- [ ] Prepare the quote at exactly the `approved_rate_per_seat` from the DEC-030 event — not higher, not lower
- [ ] Confirm the quote reflects the correct contract length and upfront terms
- [ ] Include the `pricing_exception_event_id` in internal deal metadata (CRM field or deal note, not visible to customer)
- [ ] Send the quote

A quote that differs from the approved rate by any amount is a new exception request that must restart at Step 3.

#### Step 7 — Document in DECISION_LOG.md

After the quote is sent, add an entry to `docs/DECISION_LOG.md` with:

```markdown
### DEC-XXX · Pricing exception: [Prospect identifier] — [tier], [N] seats, $[rate]/seat

- **Decision:** Non-standard discount of [X]% approved for [Prospect identifier]
  ([tier], [N] seats, [Y]-year contract). Approved rate: $[rate]/seat/month
  (effective discount [Z]% vs. list $[list]/seat; contractual floor $[floor]/seat respected).
  Approval authority: [founder / founder + investor lead]. DEC-030 event emitted:
  `enterprise.pricing_exception_approved`, event_id `[uuid]`.
- **Owner:** founder
- **Why:** [Brief justification — e.g. "anchor account in fintech vertical with reference potential"]
- **Reverse cost:** Medium — if this deal becomes a pricing floor expectation for the vertical,
  future deals in the segment will require the same level, compressing GM by ~[N]pp.
```

---

### 32.4 Price Floor Enforcement

The contractual price floors (**Starter $6.00 · Growth $4.50 · Enterprise $4.00/seat/month**) cannot be waived at any approval level. This section governs what happens if a below-floor rate is requested.

**When a below-floor request arrives:**

1. **STOP immediately.** Do not begin approval routing — there is nothing to approve.
2. **Notify the account manager** that the proposed rate is below the contractual floor and the deal must be restructured. Options: reduce the discount to bring the rate above the floor; reduce scope (fewer premium features, self-serve onboarding instead of dedicated CSM); explore whether a longer contract term can be used to reach a lower nominal rate within the approved schedule.
3. **Emit the DEC-030 event** — even for denied requests:

```sql
INSERT INTO audit_log_events (
  action,
  actor_id,
  resource_type,
  resource_id,
  metadata,
  severity,
  ...
)
VALUES (
  'enterprise.price_floor_override_requested',
  '<requesting_actor_uuid>',
  'enterprise_quote',
  '<opportunity_id>',
  jsonb_build_object(
    'tier',              '<tier>',
    'seats',             <seats>,
    'requested_rate',    <requested_rate>,
    'applicable_floor',  <floor>,
    'undershoot_amount', <floor - requested_rate>,
    'decision',          'DENIED',
    'denied_by',         'floor_enforcement',
    'reason',            'below_contractual_floor',
    'rationale_ref',     '<ticket_or_thread_url>'
  ),
  'CRITICAL',
  ...
);
```

**Severity is CRITICAL regardless of how far below the floor the request falls.** A $5.99/seat request for a Starter deal (one cent below the $6.00 floor) is as categorically impermissible as a $2.00/seat request.

4. **Add a DECISION_LOG.md entry** noting that a below-floor rate was requested and declined. Account name can be anonymised.

**If there is any pressure to override the floor:**

Escalate to compliance-officer immediately. The compliance-officer has blocking authority on any deal that would breach the floor. If the compliance-officer is pressured, they escalate to the investor lead (pre-Series A) or board (post-Series A). This is a non-negotiable contractual and compliance commitment — it cannot be unlocked by urgency, deal size, or relationship pressure.

---

### 32.5 Record Keeping

| Record | Location | Retention |
|---|---|---|
| Approval request (Linear ticket / Slack thread) | `#pricing-exceptions` Slack channel or Linear `Pricing Exceptions` project | 7 years |
| Founder approval reply | Same channel / ticket thread as the request | 7 years |
| Investor co-approval reply (where applicable) | Same thread or separate email thread preserved in deal folder | 7 years |
| DEC-030 `enterprise.pricing_exception_approved` event | `audit_log_events` table, HMAC-chained | 7 years (HIGH severity) |
| DEC-030 `enterprise.price_floor_override_requested` event (denied) | `audit_log_events` table, HMAC-chained | 7 years (CRITICAL severity) |
| `DECISION_LOG.md` entry | `docs/DECISION_LOG.md`, version-controlled | Indefinitely |
| `pricing_exception_event_id` linked to `enterprise.contract_signed` | `audit_log_events` table | 7 years |

Pre-CRM: until a CRM is deployed, the deal folder structure is:

```
pricing-exceptions/
  YYYY-MM-DD_[tier]_[prospect-slug]/
    01_request.md           -- approval request from Step 3
    02_founder_approval.txt -- copy of founder's written approval
    03_co_approver.txt      -- investor lead approval (if applicable)
    04_audit_event_id.txt   -- UUID of the DEC-030 event
    05_quote_sent.txt       -- date sent + quote document reference
```

---

### 32.6 SOC 2 Relevance

The audit trail produced by this procedure is evidence for:

| SOC 2 Criterion | How this procedure addresses it |
|---|---|
| **CC1.4** — Management communicates commitment to integrity | Documented pricing governance with floor enforcement demonstrates that pricing decisions require authorisation and are not discretionary |
| **CC5.2** — Policies and procedures for commitments to customers | The discount authority matrix (§31.6) and this procedure constitute a formal pricing policy; the DEC-030 trail proves the policy is followed |
| **CC5.3** — Monitoring effectiveness of controls | CRITICAL-severity DEC-030 events on every floor-breach attempt (including denied ones) provide a continuous signal that floor enforcement is active |
| **CC1.2** — Integrity and ethical values | Even denied below-floor requests produce a DEC-030 event — demonstrating that integrity is enforced at the system level, not at the individual level |

The four `enterprise.*` DEC-030 events emitted through this procedure appear in FORM's quarterly compliance attestation. Enterprise customers who request an audit log extract will see evidence that pricing exceptions are subject to documented approval and HMAC-chained immutable records.

---

### 32.7 Checklist Closure

This section closes `docs/COST_MODEL.md §31.9` item 3 (P0, M4).

| Definition of done item | Status |
|---|---|
| Written procedure exists (this section) | ✅ Done |
| Founder can follow it (step-by-step in §32.3) | ✅ Done |
| First test non-standard quote produces a DEC-030 event | ⬜ Closes on first use — verify Step 5 against `emit-audit-event` Worker in staging before enterprise pilot launch |

**Remaining P0 actions before first enterprise quote can be sent:**
1. Founder reads and acknowledges this procedure in writing (Slack thread or Linear ticket, preserved per §32.5)
2. Test the `emit-audit-event` Worker endpoint with a dummy `enterprise.pricing_exception_approved` event in staging; confirm HMAC chain integrity
3. Create the `pricing-exceptions/` folder structure in the deal tracking system (pre-CRM: shared folder, drive, or Linear project)
4. Add DEC-038 entry to `docs/DECISION_LOG.md` (done in this commit)

---

---

## 33. Enterprise AI Token Budget & Billing Architecture

> **Resolves OQ-COACH-02** (DATA_MODEL.md §21.15) — Decision required before first enterprise contracts are signed. Status: 🟢 Resolved — DEC-041 (2026-06-11).

---

### 33.1 Purpose & Scope

This section defines how FORM accounts for Anthropic API (Victor coaching) and ElevenLabs voice synthesis costs in enterprise contracts, and documents the internal token budget governance model.

**In scope:**
- Billing model selection: flat per-seat vs. consumption billing
- Internal token budgets per tier (cost governance, not contractual limits)
- Gross margin sensitivity analysis at multi-seat scale
- Overage policy (internal operational procedure only)
- Revenue recognition treatment under ASC 606 / IFRS 15
- DEC-030 HMAC-chained audit events for cost monitoring

**Out of scope:**
- Consumer Pro billing (covered in §4 and §31.2)
- Infrastructure COGS (covered in §3 and §15)
- `tenant_monthly_coaching_cost` materialized view schema (documented in DATA_MODEL.md §21.8)

---

### 33.2 The Two Models: Flat Per-Seat vs. Consumption Billing

| Dimension | Flat Per-Seat | Consumption Billing |
|---|---|---|
| **Predictability (enterprise)** | ✅ Fixed monthly invoice — CFO-friendly | ❌ Variable — hard to budget |
| **Predictability (FORM)** | ✅ Predictable COGS | ❌ COGS variance tied to coaching intensity |
| **Fairness** | ❌ Light users subsidise heavy users | ✅ Pay-for-what-you-use |
| **Implementation complexity** | ✅ Simple — invoice = seats × rate | ❌ Metering infrastructure, invoice reconciliation, dispute handling |
| **Sales motion** | ✅ Clean quote; no usage rider required | ❌ Requires usage cap clause, overage rate, reconciliation cadence |
| **ASC 606 / IFRS 15** | ✅ Ratable over contract term — clean | ⚠️ Variable consideration — requires constraint estimation |
| **Enterprise preference** | ✅ Strongly preferred (procurement benchmark) | ❌ Avoided unless budget is tied to outcomes |

**Conclusion**: Flat per-seat billing is the correct model for FORM's enterprise tier at launch.

---

### 33.3 Token Budget Per Seat Per Tier

Token budgets below are **internal cost-governance targets**. They are NOT contractual limits, NOT exposed to tenant admins, and NOT surfaced in any customer-facing document.

Baseline consumption derived from: median active user = ~3 coaching sessions/week, ~8 turns/session, ~400 tokens/turn average (input+output blended), 4.3 weeks/month.

| Tier | Seat range | Tokens/seat/mo (budget) | Headroom vs. baseline (32,256) | Voice chars/seat/mo | AI cost/seat (budget) | Voice cost/seat (budget) | Total AI+Voice/seat |
|---|---|---|---|---|---|---|---|
| **Starter** | 50–200 | 50,000 | 1.55× | 13,000 | $0.20 | $0.039 | **$0.24** |
| **Growth** | 201–1,000 | 50,000 | 1.55× | 13,000 | $0.20 | $0.039 | **$0.24** |
| **Enterprise** | 1,001+ | 75,000 | 2.33× | 20,000 | $0.30 | $0.060 | **$0.36** |

**Token cost calculation**: blended Anthropic rate ≈ $4.00/1M tokens (weighted 3:1 input/output at $3/$15). Voice at $0.30/1K chars.

> **Privacy invariant**: `tenant_monthly_coaching_cost` is aggregated at tenant level. FORM never exposes per-user token consumption to HR or tenant admins. Enforced at Postgres RLS layer.

---

### 33.4 Flat Per-Seat Model — Margin Math

At the per-seat rates above, AI+Voice COGS as a percentage of revenue by tier and usage intensity:

| Usage intensity | AI+Voice cost/seat | Starter $12/seat | Growth $9/seat | Enterprise $7/seat |
|---|---|---|---|---|
| **Baseline (1× median)** | $0.24–0.36 | 2.0–3.0% | 2.7–4.0% | 3.4–5.1% |
| **Heavy user (3× median)** | $0.72–1.08 | 6.0–9.0% | 8.0–12.0% | 10.3–15.4% |
| **Extreme user (5× median)** | $1.20–1.80 | 10.0–15.0% | 13.3–20.0% | 17.1–25.7% |

**Key insight**: Even at 5× median usage, AI+Voice remains a minority of COGS. The dominant cost drivers are CSM labour (§26) and infrastructure (§15). The margin risk is manageable without consumption billing.

**Fully-loaded gross margin** at baseline usage (including infra + CSM from §15 and §26):
- Starter $12: ~83% GM
- Growth $9: ~85% GM  
- Enterprise $7: ~88% GM

These figures are robust even at 3× usage intensity.

---

### 33.5 Consumption Billing (Not Recommended at Launch)

Consumption billing may be revisited if **all three** conditions are met:
1. A single tenant accounts for >15% of FORM's total Anthropic spend in any calendar month
2. The enterprise tier average seat count exceeds 2,000 (making per-tenant COGS material)
3. FORM has metering infrastructure mature enough to produce auditable per-tenant consumption reports (P2, M9+)

Until all three conditions are met, flat per-seat billing remains the standard.

If consumption billing is ever introduced, it must include:
- A contractual "included tokens" baseline (not less than the budgets in §33.3)
- A clearly disclosed overage rate
- Monthly usage reports accessible to tenant admins (aggregate only — no per-user breakdown)
- DEC-030 audit events for every overage invoice issued

---

### 33.6 Recommended Decision

| Decision | Choice | Rationale |
|---|---|---|
| **Billing model** | Flat per-seat | Predictable for enterprise CFOs; simple to invoice; ASC 606 clean; margin healthy at launch scale |
| **Token budgets** | Internal governance only | Not contractual; not disclosed to tenants; enforced via alerting, not hard cutoffs |
| **`coaching_seat_allowance_tokens`** | NULL at launch | Column exists for future optionality; not activated |
| **`tenant_monthly_coaching_cost` view** | Retained, FORM-internal only | Used for gross margin monitoring and churn-risk signalling; never exposed to tenant admins |
| **Overage policy** | Internal operational procedure (§33.7) | Not a customer-facing SLA; not referenced in MSA or enterprise contracts |

**Decision reference**: DEC-041 (2026-06-11). Owner: `enterprise-architect` + `finance` + `compliance-officer`.

---

### 33.7 Overage Policy (Internal)

> ⚠️ **This policy is INTERNAL ONLY.** It must never be referenced in customer-facing documents, MSA templates, sales decks, or pricing-enterprise.html.

When a tenant's monthly AI+Voice cost exceeds the budget in §33.3:

| Overage band | Monthly AI+Voice spend vs. budget | Action |
|---|---|---|
| **Yellow** | 100–150% of budget | Auto-alert to `enterprise-architect` + assigned CSM. Review coaching patterns. No customer contact. |
| **Orange** | 150–200% of budget | Escalate to finance lead. CSM account review. Assess whether tenant is an outlier or indicates model drift. |
| **Red** | >200% of budget | Founder review. Assess: (a) anomalous tenant behaviour, (b) model regression, (c) need to revisit flat-fee assumption for this account. |

No hard token cutoffs are applied at any band. Victor coaching continuity is never interrupted due to cost overrun — this is a financial monitoring policy, not a service limitation.

---

### 33.8 Revenue Recognition (ASC 606 / IFRS 15)

Flat per-seat enterprise contracts are treated as a single performance obligation: access to the FORM platform (including Victor AI coaching) over the contract term.

| ASC 606 step | FORM treatment |
|---|---|
| **Step 1** — Identify contract | Signed MSA + Order Form = enforceable contract |
| **Step 2** — Identify performance obligations | Single PO: platform access (coaching + analytics + admin) |
| **Step 3** — Determine transaction price | Fixed: seats × rate × months. No variable consideration. |
| **Step 4** — Allocate transaction price | Not applicable (single PO) |
| **Step 5** — Recognise revenue | Ratably over the contract term (daily or monthly proration) |

**Annual pre-pay treatment**: Cash received upfront → Deferred Revenue liability → recognised ratably over 12 months.

**No variable consideration risk**: Because token consumption is not a contractual term, there is no variable consideration adjustment required. This is a material advantage of flat per-seat billing from an accounting perspective.

---

### 33.9 DEC-030 HMAC-Chained Audit Events

Two new audit event types are registered under DEC-030. Both are STANDARD severity (3-year retention).

**Event 1: `enterprise.ai_cost_monitor_alert`**

Emitted when a tenant crosses the Yellow overage threshold (§33.7).

```sql
INSERT INTO audit_log (
  event_type,
  severity,
  tenant_id,       -- UUID only; no user_id in payload (privacy floor)
  payload,
  sequence_number,
  prev_hash,
  hash
) VALUES (
  'enterprise.ai_cost_monitor_alert',
  'STANDARD',
  $tenant_id,
  jsonb_build_object(
    'budget_period',        $yyyy_mm,          -- e.g. '2026-06'
    'budget_tokens',        $budget_tokens,    -- from §33.3 for this tier
    'actual_tokens',        $actual_tokens,    -- from tenant_monthly_coaching_cost view
    'overage_pct',          $overage_pct,      -- (actual - budget) / budget × 100
    'overage_band',         $band,             -- 'yellow' | 'orange' | 'red'
    'alert_sent_to',        $recipients        -- e.g. ['enterprise-architect', 'csm-assigned']
  ),
  nextval('audit_log_sequence'),
  $prev_hash,
  encode(hmac(
    concat($sequence_number, $prev_hash, $event_type, $tenant_id::text, $payload::text),
    current_setting('app.hmac_secret'),
    'sha256'
  ), 'hex')
);
```

**Event 2: `enterprise.ai_cost_outlier_flagged`**

Emitted when a tenant is flagged as a statistical outlier (>2σ above mean per-seat spend across all tenants in same tier).

```sql
INSERT INTO audit_log (
  event_type,
  severity,
  tenant_id,
  payload,
  sequence_number,
  prev_hash,
  hash
) VALUES (
  'enterprise.ai_cost_outlier_flagged',
  'STANDARD',
  $tenant_id,
  jsonb_build_object(
    'budget_period',        $yyyy_mm,
    'tier',                 $tier,
    'tenant_spend_usd',     $tenant_spend_usd,
    'tier_mean_spend_usd',  $tier_mean_usd,
    'tier_stddev_usd',      $tier_stddev_usd,
    'sigma_above_mean',     $sigma            -- must be ≥ 2.0 to trigger
  ),
  nextval('audit_log_sequence'),
  $prev_hash,
  encode(hmac(
    concat($sequence_number, $prev_hash, $event_type, $tenant_id::text, $payload::text),
    current_setting('app.hmac_secret'),
    'sha256'
  ), 'hex')
);
```

> **Privacy invariant (both events)**: payload contains `tenant_id` (UUID) only. No `user_id`, no per-user token breakdown, no coaching content. Enforced by schema constraint — `user_id` column is NULL for both event types.

---

### 33.10 Implementation Checklist

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Confirm `tenant_config.coaching_seat_allowance_tokens` is NULL for all tenants at launch migration | `enterprise-architect` | **P0** | M5 |
| 2 | Register `enterprise.ai_cost_monitor_alert` and `enterprise.ai_cost_outlier_flagged` event types in `AUDIT_LOG_SCHEMA.md` with Zod schemas | `enterprise-architect` | **P0** | M5 |
| 3 | Implement nightly pg_cron job: compare `tenant_monthly_coaching_cost` against §33.3 budgets; emit Yellow alert event if threshold exceeded | `data-engineer` | **P0** | M5 |
| 4 | Implement monthly outlier detection: per-tier mean + stddev computation; emit `enterprise.ai_cost_outlier_flagged` if σ ≥ 2.0 | `data-engineer` | **P0** | M5 |
| 5 | Add `enterprise.ai_cost_monitor_alert` and `enterprise.ai_cost_outlier_flagged` to DEC-030 HMAC chain validation tests | `qa-lead` | **P0** | M5 |
| 6 | Confirm `tenant_monthly_coaching_cost` view is NOT accessible via any tenant-admin API endpoint or dashboard panel | `security-engineer` | **P0** | M5 |
| 7 | Add DEC-041 entry to `docs/DECISION_LOG.md` (flat per-seat billing decision record) | `enterprise-architect` | **P1** | M6 |
| 8 | Add §33.7 overage bands to internal runbook (not customer-facing) | `enterprise-architect` | **P1** | M6 |
| 9 | Verify ASC 606 single-PO treatment with external auditor at Series A readiness review | `finance` + `compliance-officer` | **P1** | M6 |
| 10 | Update `docs/MSA_TEMPLATE.md` to confirm no token consumption clause is present | `legal` + `compliance-officer` | **P2** | M9 |
| 11 | Revisit consumption billing conditions (§33.5) at first enterprise cohort QBR | `enterprise-architect` + `finance` | **P2** | M12 |

---

### 33.11 Open Questions Resolved

| OQ | Resolution |
|---|---|
| **OQ-COACH-02** (DATA_MODEL.md §21.15) | 🟢 **Resolved — DEC-041 (2026-06-11)**. Flat per-seat billing adopted. `tenant_monthly_coaching_cost` retained for FORM-internal use only. `tenant_config.coaching_seat_allowance_tokens` NULL at launch. Token budgets in §33.3 are internal governance targets, not contractual terms. Consumption billing deferred per conditions in §33.5. |

---

## 34. Enterprise Renewal Risk Register, Churn Decision Economics & Intervention Model

### 34.1 Purpose & Scope

> Owner: customer-success + founder + finance. Audience: CS team, AE team, board. Review: monthly (risk register), quarterly (churn economics review). Cross-references: §23 (Enterprise NRR Engine — expansion mechanics and NRR formula), §26 (CS Team Scaling — CSM capacity and cost-per-account), §31 (Pricing Architecture — discount floor constraints), §32 (Pricing Exception Approval — retention discount approval procedure), `docs/OBSERVABILITY.md §33` (tenant engagement health metrics and CHS model), `docs/ENTERPRISE.md` (CS tier definitions and QBR cadence).

Section §23 models NRR as an *output* — the formula that combines expansion, contraction, and churn into a single ARR percentage. Section §26 models the *capacity* side — how many accounts one CSM can carry, and the headcount ramp required as the customer base grows. Neither section answers the financially critical operational questions:

1. **How do we rank accounts by churn probability** before they churn, not after?
2. **What is the financial cost of losing a specific account**, tier by tier, including CAC replacement cost?
3. **When is a retention discount NPV-positive** vs. when is it cheaper to let the account churn and reacquire at standard pricing?
4. **What is the expected value of CSM intervention** on an at-risk account, given intervention cost, success probability, and ACV at risk?
5. **What does churn by contract vintage** look like, and what target benchmark should FORM hold itself to in Year 1, 2, and 3 of a contract?

This section fills that gap. All figures are [ESTIMATE] until first 10 enterprise renewals are tracked. The churn risk classification in §34.2 uses the Customer Health Score (CHS) model from `docs/OBSERVABILITY.md §33.3` — this section provides the *financial translation* of CHS bands into revenue-at-risk numbers.

**Privacy floor reminder:** All analysis in this section operates at tenant level. No individual user health data, workout records, or personally identifiable coaching content is used in churn risk scoring. The engagement signals feeding the risk classifier come exclusively from `tenant_engagement_snapshots` (OBSERVABILITY §33.7), which enforces k-anonymity (n ≥ 10) and column-level RLS preventing PII access.

---

### 34.2 Churn Risk Classification Framework

FORM uses a three-band risk classification tied to the Customer Health Score (CHS) from OBSERVABILITY §33.3. CHS is a weighted composite of seat utilization (40%), coaching engagement (30%), onboarding/retention (20%), and feature adoption (10%), scored 0–100.

| Risk band | CHS score | Label | Renewal likelihood | CSM action required |
|---|---|---|---|---|
| 🟢 **Green** | 70–100 | Healthy | > 90% | Routine QBR; explore expansion |
| 🟡 **Amber** | 40–69 | At-risk | 55–75% | Proactive outreach; root-cause analysis; consider concession |
| 🔴 **Red** | 0–39 | Critical | < 40% | Executive escalation; retention play decision within 30 days |

**Risk band thresholds are reviewed quarterly.** Before first 5 renewal events, all thresholds are provisional. After 5 renewals, calibrate CHS cutoffs against actual churn outcomes (see OQ-CHURN-01).

#### 34.2.1 Trigger conditions for risk downgrade

An account is automatically downgraded from Green → Amber, or Amber → Red, when any of the following conditions are detected by OBSERVABILITY §33 alert rules:

| Trigger | Source alert | Downgrade |
|---|---|---|
| Seat utilization < 40% for 14 consecutive days | AL-ENGAGE-01 | Green → Amber |
| Seat utilization < 25% for 7 consecutive days | AL-ENGAGE-02 | Amber → Red (or direct Green → Red if sudden drop) |
| Zero coaching sessions with ≥ 5 active seats for 7 days | AL-ENGAGE-03 | → Red (immediate) |
| CHS score newly enters critical band (< 40) | AL-ENGAGE-05 | → Red |
| CHS "freefall" (< 30 for 3 consecutive weeks) | AL-ENGAGE-06 | Red — CS VP escalation required |
| Admin dashboard login: zero in last 30 days | (§33 derived) | Green → Amber |
| Renewal is < 90 days away AND account is Amber | CSM dashboard query | Amber → Red (time-sensitivity override) |

#### 34.2.2 Upgrade conditions

An account is upgraded from Red → Amber or Amber → Green when:
- CHS score exceeds the upper band threshold for 3 consecutive weekly snapshots.
- Seat utilization recovers above 50% for 2 consecutive weeks.
- Coaching sessions resume at ≥ 2 sessions per active seat per month.

Upgrades require CSM confirmation — the system flags the upgrade, but the CSM documents the recovery root cause before the CHS band is changed in the risk register.

---

### 34.3 Financial Impact of Churn by Tier

The cost of losing an enterprise account has three components: (1) immediate MRR loss, (2) sunk cost of the account acquisition (CAC, from §8.5), and (3) replacement cost — time to find and close a replacement deal.

#### 34.3.1 Immediate MRR and ACV loss by representative deal

| Tier | Representative deal | Monthly MRR lost | Annual ACV lost | MRR replacement time [ESTIMATE] |
|---|---|---|---|---|
| Starter · minimum | 50 seats × $12 | $600 | $7,200 | 3–5 months |
| Starter · typical | 100 seats × $12 | $1,200 | $14,400 | 3–4 months |
| Growth · typical | 300 seats × $9 | $2,700 | $32,400 | 4–7 months |
| Growth · large | 800 seats × $9 | $7,200 | $86,400 | 9–14 months |
| Enterprise · typical | 2,000 seats × $7 | $14,000 | $168,000 | 12–24 months |

**MRR replacement time definition:** Months from churn date until net new MRR from alternative accounts equals the lost MRR. Assumes 3:1 ACV pipeline coverage and 25% win rate (§19 GTM model). Enterprise-tier replacement time is the longest because single large deals are required; the pipeline rarely has multiple $168k-ACV opportunities in parallel.

#### 34.3.2 Total economic loss of a Year 2 churn (CAC destruction + foregone Y3 ARR)

When an account churns, the original CAC becomes a sunk loss. For Year 2 churns, the expansion ARR projected in §23 is permanently removed from the NRR numerator.

| Tier | Representative deal | Original CAC (§8.5) | Y3 ACV foregone | Total NPV loss on Y2 churn |
|---|---|---|---|---|
| Starter · typical | 100 seats | $1,500 | $14,400 | **~$15,900** |
| Growth · typical | 300 seats | $4,200 | $32,400 | **~$36,600** |
| Enterprise · typical | 2,000 seats | $10,750 | $168,000 | **~$178,750** |

**Key insight:** On a Year 2 churn the economic loss is dominated by the Year 3 ACV foregone, not the CAC. An Enterprise-typical churn at Year 2 destroys ~$179k in NPV — equivalent to finding and closing 16–17 additional Growth-tier accounts from scratch. This asymmetry is why retention investment has dramatically higher ROI than new acquisition at the 2-year mark.

---

### 34.4 Intervention Economics: Expected Value of CSM Time

Not all interventions succeed, and not all at-risk accounts are worth saving at any cost. This section models the expected value (EV) of a CSM intervention to determine whether the intervention cost is justified before committing CSM capacity.

**EV formula:**

```
EV(intervention) = [P(save | intervention) − P(save | no action)] × ACV_annual
                 − Intervention_cost

Grant intervention if EV(intervention) > 0
```

#### 34.4.1 Intervention success rate estimates by risk band

| Risk band | P(save | no action) | P(save | CSM intervention) | Intervention lift |
|---|---|---|---|---|
| Amber | 65% | 82% | +17 pp |
| Red — early (CHS 25–39) | 35% | 60% | +25 pp |
| Red — freefall (CHS < 25) | 15% | 35% | +20 pp |
| Red — admin non-responsive (zero logins 30d) | 5% | 18% | +13 pp |

All figures are **[ESTIMATE]**. Replace with actuals after first 10 at-risk interventions using the tracking procedure in §34.9 checklist item 5.

#### 34.4.2 CSM intervention cost model

One intervention unit (deep engagement, root-cause diagnosis, recovery plan, 6-week follow-up) requires:

| Activity | Hours | CSM cost at $75k/yr ($39/hr) |
|---|---|---|
| Account diagnostic review (CHS + engagement signals) | 2h | $78 |
| Executive sponsor outreach and call | 3h | $117 |
| Root-cause analysis documentation | 1h | $39 |
| Recovery plan delivery (call + written) | 3h | $117 |
| Weekly check-ins × 6 weeks | 6h | $234 |
| **Total intervention unit** | **15h** | **$585** |

Plus any retention discount concession cost (see §34.5).

#### 34.4.3 EV decision table (no concession)

| Risk band | Tier | ACV | EV (intervention, no discount) | Verdict |
|---|---|---|---|---|
| Amber | Starter · minimum | $7,200 | 0.17 × $7,200 − $585 = **+$639** | ✅ Intervene |
| Amber | Growth · typical | $32,400 | 0.17 × $32,400 − $585 = **+$4,923** | ✅ Intervene |
| Red — early | Starter · minimum | $7,200 | 0.25 × $7,200 − $585 = **+$1,215** | ✅ Intervene |
| Red — early | Growth · typical | $32,400 | 0.25 × $32,400 − $585 = **+$7,515** | ✅ Intervene |
| Red — freefall | Starter · minimum | $7,200 | 0.20 × $7,200 − $585 = **+$855** | ✅ Intervene |
| Red — freefall | Growth · typical | $32,400 | 0.20 × $32,400 − $585 = **+$5,895** | ✅ Intervene |
| Red — non-responsive | Starter · minimum | $7,200 | 0.13 × $7,200 − $585 = **+$351** | ⚠️ One unit only; second unit is NPV-negative |
| Red — non-responsive | Growth · typical | $32,400 | 0.13 × $32,400 − $585 = **+$3,627** | ✅ Intervene |

**Key finding:** At these probability estimates, every at-risk account warrants at least one CSM intervention unit. The only borderline case is a Starter-minimum account with an unresponsive admin ($351 EV) — intervene once but do not allocate a second unit unless the admin responds.

---

### 34.5 Retention Discount Authorization Model

A retention discount is a price reduction offered at renewal to retain an account that would otherwise churn. It is a subset of §32 (Pricing Exception Approval) — the same approval chain applies, and the same price floor constraint (§31.5) is a hard limit.

**NPV-positive condition for granting a retention discount:**

```
NPV(grant discount) = P_save_with_discount × (ACV × (1 − disc_pct) × remaining_years)
NPV(no discount)    = P_save_no_discount   × (ACV × remaining_years)

Grant discount if: NPV(grant) − NPV(no discount) > 0
Simplified:        [P_save_disc − P_save_base] × ACV × years  >  ACV × disc_pct × P_save_disc × years
```

#### 34.5.1 Retention discount NPV table (Year 1 renewal, 1-year renewal term)

Assumes Red-early risk band: P_save baseline 35%, P_save with discount 65% [ESTIMATE]. Discount increments are illustrative.

| Tier | ACV | Discount | Revenue conceded (Year 1) | EV of discount | Verdict |
|---|---|---|---|---|---|
| Starter · typical | $14,400 | 10% | $1,440 | 0.30 × $14,400 − $1,440 = **+$2,880** | ✅ Approve |
| Starter · typical | $14,400 | 25% | $3,600 | 0.30 × $14,400 − $3,600 = **+$720** | ✅ Approve (barely) |
| Starter · typical | $14,400 | 40% | $5,760 | 0.30 × $14,400 − $5,760 = **−$1,440** | ❌ Decline |
| Growth · typical | $32,400 | 10% | $3,240 | 0.30 × $32,400 − $3,240 = **+$6,480** | ✅ Approve |
| Growth · typical | $32,400 | 25% | $8,100 | 0.30 × $32,400 − $8,100 = **+$1,620** | ✅ Approve |
| Growth · typical | $32,400 | 40% | $12,960 | 0.30 × $32,400 − $12,960 = **−$3,240** | ❌ Decline |

**Hard constraint:** No retention discount may take the effective rate below the price floor from §31.5 (Starter: $6.00/seat/mo, Growth: $4.50/seat/mo, Enterprise: $4.00/seat/mo). This applies even when the NPV calculation suggests a deeper discount would be positive.

**Approval authority (identical to §32.2):**
- Up to –15%: CS lead approval (no §32 exception procedure required; standard multi-year rate)
- –16% to –32.5%: Founder approval required (§32 procedure applies)
- –32.5% to price floor: Founder + investor lead
- Below price floor: **Not permitted at any approval level**

**Non-pricing concessions (no approval required):** When a retention discount is NPV-negative or borderline, offer non-pricing concessions instead: extended onboarding support (4 extra weeks), priority scheduling for a feature request review, QBR cadence upgrade (Starter → Growth frequency with no pricing change), or access to beta features. These generate real customer value without triggering the §32 exception chain.

---

### 34.6 Contract Vintage Churn Analysis

Churn probability is not uniform across contract years. Industry benchmarks for B2B SaaS wellness products indicate a J-curve pattern: highest churn risk in Year 1 (low activation, program not yet embedded in culture), declining in Year 2 (survivors proved value), stabilizing in Year 3+ (program embedded in benefits infrastructure).

#### 34.6.1 Target annual gross churn benchmarks by vintage

| Contract year | Description | Target gross churn | Rationale |
|---|---|---|---|
| **Year 1** | Pilot converts, first year | < 30% [ESTIMATE] | High risk: activation phase; champion may change; program not embedded |
| **Year 2** | Proven survivors | < 15% [ESTIMATE] | Lower risk: customers who renewed once proved value; champion locked in |
| **Year 3+** | Embedded accounts | < 8% [ESTIMATE] | Lowest: program in employee benefits handbook; switching cost high |
| **Blended (steady state)** | ~50% Y1, 30% Y2, 20% Y3+ | < 18% gross churn | Aligns with §23 NRR model — >115% NRR requires gross churn < ~18% at modest expansion |

Replace with FORM actuals at first 5 renewals. If Year 1 gross churn exceeds 30%, escalate to product-strategist — this signals an activation or product-market fit problem, not a CS capacity problem.

#### 34.6.2 NRR sensitivity to Year 1 churn rate

High Year 1 churn destroys the expansion cohort permanently. The following table shows NRR impact on a Growth cohort of 10 accounts × $32,400 ACV:

| Y1 gross churn | Accounts retained | Y2 expansion ARR (15% expansion on survivors, §23.5) | Blended NRR |
|---|---|---|---|
| 15% | 8.5 accounts | $41,310 | **118%** |
| 25% | 7.5 accounts | $36,450 | **111%** |
| 35% | 6.5 accounts | $31,590 | **104%** |
| 45% | 5.5 accounts | $26,730 | **97%** (net contraction) |

**Implication:** FORM's 115% NRR target requires Y1 gross churn ≤ 25% for the Growth cohort. This makes Year 1 CSM investment — onboarding quality, Day 30 engagement review, early risk detection — the highest-leverage CS activity in the entire model, higher than expansion plays or QBR frequency.

---

### 34.7 Post-Churn Recovery Timeline

When an enterprise account churns, the net MRR deficit must be absorbed by new business pipeline. Recovery time is the number of months until net new ARR from replacement deals equals the churned ARR.

Assumptions: sales cycle 5–7 months (enterprise), 25% win rate (§19), pipeline maintains 3:1 ACV coverage.

| Churned account | ACV lost | Monthly MRR lost | Replacement deals needed | Minimum recovery time |
|---|---|---|---|---|
| Starter · minimum (50 seats) | $7,200 | $600 | 0.5 new Starter | **3–4 months** (pipeline already open) |
| Growth · typical (300 seats) | $32,400 | $2,700 | ~1 new Growth | **5–8 months** |
| Growth · large (1,000 seats) | $108,000 | $9,000 | 3–4 new Growth | **10–18 months** |
| Enterprise · typical (2,000 seats) | $168,000 | $14,000 | 5 Growth or 1 Enterprise | **15–28 months** |

**Key takeaway:** A single Enterprise-tier churn requires 15–28 months to replace through new business alone. At FORM's pre-PMF sales velocity (1–3 new enterprise deals per quarter), this could materially set back the ARR growth trajectory for 2+ years — reinforcing why every dollar spent on retention has dramatically higher ROI than equivalent spend on new acquisition for the Enterprise tier.

**Mid-contract termination:** MSA termination-for-cause clauses can trigger mid-contract exits. These are more severe than non-renewals: (a) ARR is removed immediately rather than at the scheduled renewal date; (b) deferred revenue not yet earned must be reversed under ASC 606 (§18.1). CSM trigger threshold for mid-contract termination risk: CHS score < 20 sustained for 4 consecutive weeks with > 6 months remaining on the contract. This triggers CS VP + founder escalation regardless of renewal date proximity.

---

### 34.8 DEC-030 HMAC-Chained Audit Events

Three new audit event types are registered under DEC-030. These events create an immutable record of risk classification changes and retention decisions, supporting SOC 2 CC5.2 (business risk assessment) and CC7.2 (monitoring of controls) and providing the evidence trail for pricing exception compliance (§32.6).

**Event 1: `enterprise.renewal_risk_flagged`** — STANDARD severity, 3-year retention.

Emitted when a tenant's CHS band changes to Amber or Red, or when the time-sensitivity override fires (renewal < 90 days away on an Amber account).

```sql
INSERT INTO audit_log (
  event_type,
  severity,
  tenant_id,
  payload,
  sequence_number,
  prev_hash,
  hash
) VALUES (
  'enterprise.renewal_risk_flagged',
  'STANDARD',
  $tenant_id,
  jsonb_build_object(
    'risk_band',           $risk_band,           -- 'amber' | 'red'
    'chs_score',           $chs_score,           -- current CHS composite (0–100)
    'prior_risk_band',     $prior_band,          -- 'green' | 'amber'
    'trigger_rule',        $trigger_rule,        -- 'AL-ENGAGE-02' | 'AL-ENGAGE-05' | 'renewal_proximity' | ...
    'days_to_renewal',     $days_to_renewal,     -- integer; NULL if > 180 days or no active contract
    'alerted_to',          $recipients           -- e.g. ['csm-assigned', 'cs-lead']
    -- INVARIANT: no user_id, no individual engagement data, no health metrics.
    -- CHS is a tenant-aggregate composite; k-anonymity enforced in tenant_engagement_snapshots.
  ),
  nextval('audit_log_sequence'),
  $prev_hash,
  encode(hmac(
    concat($sequence_number, $prev_hash, $event_type, $tenant_id::text, $payload::text),
    current_setting('app.hmac_secret'),
    'sha256'
  ), 'hex')
);
```

---

**Event 2: `enterprise.churn_confirmed`** — HIGH severity, 7-year retention (contract lifecycle evidence).

Emitted when a tenant formally notifies non-renewal, or when a contract is terminated early.

```sql
INSERT INTO audit_log (
  event_type,
  severity,
  tenant_id,
  payload,
  sequence_number,
  prev_hash,
  hash
) VALUES (
  'enterprise.churn_confirmed',
  'HIGH',
  $tenant_id,
  jsonb_build_object(
    'churn_type',              $churn_type,           -- 'non_renewal' | 'mid_contract_termination'
    'churn_effective_date',    $effective_date,       -- ISO 8601
    'contract_id',             $contract_id,          -- FK to tenant_contracts.id
    'acv_usd',                 $acv_usd,              -- integer USD (no cent precision)
    'final_risk_band',         $final_risk_band,      -- 'green' | 'amber' | 'red'
    'intervention_count',      $intervention_count,   -- number of CSM intervention units logged
    'churn_reason_category',   $reason_cat            -- 'low_utilization' | 'budget_cut'
                                                      --   | 'champion_left' | 'product_gap'
                                                      --   | 'competitor' | 'unknown'
    -- reason_category is CSM-classified at renewal close.
    -- Verbatim exit interview notes → CRM only; not stored in DEC-030 chain.
  ),
  nextval('audit_log_sequence'),
  $prev_hash,
  encode(hmac(
    concat($sequence_number, $prev_hash, $event_type, $tenant_id::text, $payload::text),
    current_setting('app.hmac_secret'),
    'sha256'
  ), 'hex')
);
```

---

**Event 3: `enterprise.retention_discount_granted`** — HIGH severity, 7-year retention.

Emitted when a retention discount is approved and included in the renewal quote. Links to the §32 pricing exception chain via `pricing_exception_event_id` for discounts > 15%.

```sql
INSERT INTO audit_log (
  event_type,
  severity,
  tenant_id,
  payload,
  sequence_number,
  prev_hash,
  hash
) VALUES (
  'enterprise.retention_discount_granted',
  'HIGH',
  $tenant_id,
  jsonb_build_object(
    'prior_acv_usd',              $prior_acv,             -- contract value before discount
    'new_acv_usd',                $new_acv,               -- contract value after discount
    'discount_pct',               $discount_pct,          -- e.g. 15.0
    'effective_rate_per_seat',    $effective_rate,        -- must be ≥ price floor for tier (§31.5)
    'floor_respected',            true,                   -- INVARIANT: always true; floor enforced upstream
    'risk_band_at_grant',         $risk_band,             -- 'amber' | 'red'
    'approver_role',              $approver_role,         -- 'cs_lead' | 'founder' | 'founder_investor'
    'pricing_exception_event_id', $pricing_exc_event_id  -- FK to enterprise.pricing_exception_approved
                                                         --   (§31.8); required if discount_pct > 15
  ),
  nextval('audit_log_sequence'),
  $prev_hash,
  encode(hmac(
    concat($sequence_number, $prev_hash, $event_type, $tenant_id::text, $payload::text),
    current_setting('app.hmac_secret'),
    'sha256'
  ), 'hex')
);
```

> **Invariant:** `floor_respected` is always `true` in this event. If a retention discount would violate the price floor, the discount is not granted and this event is never emitted. The attempted violation is recorded as `enterprise.price_floor_override_requested` (CRITICAL, §31.8), which fires even for denied attempts.

---

### 34.9 Implementation Checklist

#### P0 — Must complete before first enterprise renewal date (M10)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Register `enterprise.renewal_risk_flagged`, `enterprise.churn_confirmed`, and `enterprise.retention_discount_granted` in `docs/AUDIT_LOG_SCHEMA.md §6` with Zod schemas matching the payload shapes in §34.8. Deploy updated event registry to `emit-audit-event` Worker. | enterprise-architect | **P0** | M10 | [x] Closed — `docs/AUDIT_LOG_SCHEMA.md` v1.4 (2026-06-11) |
| 2 | Implement CSM renewal risk dashboard: query `tenant_engagement_snapshots` (OBSERVABILITY §33.7) and surface all tenants by CHS band, days to renewal, and assigned CSM. Filter: `renewal_date BETWEEN now() AND now() + INTERVAL '180 days'`. Privacy constraint: aggregate metrics only; no individual user data. | data-engineer + customer-success | **P0** | M10 | [ ] |
| 3 | Wire OBSERVABILITY §33 alert routing to emit `enterprise.renewal_risk_flagged` whenever AL-ENGAGE-02, AL-ENGAGE-05, or AL-ENGAGE-06 fires, and on `renewal_proximity` trigger (renewal < 90 days + Amber band). Call `emit-audit-event` with DEC-030 payload from §34.8. | devops-lead + customer-success | **P0** | M10 | [ ] |
| 4 | Add churn reason classification to the internal CRM / account record: six categories from §34.8 (`low_utilization`, `budget_cut`, `champion_left`, `product_gap`, `competitor`, `unknown`). CSM must classify before `enterprise.churn_confirmed` event is emitted. | customer-success | **P0** | M10 | [ ] |
| 5 | Create intervention tracking log (Notion or Linear template) for each at-risk account. Fields: `tenant_id`, `risk_band`, `intervention_date`, `intervention_type`, `outcome` (saved / churned / pending), `csm_hours_spent`, `concession_granted` (Y/N), `concession_type`. Use to calibrate §34.4.1 probability estimates after 10 outcomes. | customer-success | **P0** | M10 | [ ] |

#### P1 — Before first Series A board reporting package (M12)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 6 | Add churn vintage tracking to ARR bridge (§18.3): separate "Churned ARR — Year 1 cohort" from "Churned ARR — Year 2+ cohort" in the monthly MRR bridge template. Report churn by vintage at every board meeting. | finance + data-engineer | **P1** | M12 | [ ] |
| 7 | Calibrate §34.4.1 intervention probability estimates using first 10 at-risk outcomes. Update EV table in §34.4.3 with actuals. If deviation from estimates exceeds 15 pp, escalate to product-strategist for CS playbook review. | customer-success + data-engineer | **P1** | M12 | [ ] |
| 8 | Add `churn_reason_category` distribution to quarterly CS review report. If `product_gap` appears in > 30% of churns, escalate to product-manager for roadmap impact assessment. If `low_utilization` appears in > 40%, escalate to product-manager for onboarding flow review. | customer-success + compliance-officer | **P1** | M12 | [ ] |
| 9 | Add `enterprise.retention_discount_granted` cross-reference requirement to `docs/DECISION_LOG.md`: any retention discount > 15% must include a DECISION_LOG entry (DEC-XXX) linking to the `pricing_exception_event_id` from the DEC-030 chain. | compliance-officer | **P1** | M12 | [ ] |

#### P2 — Post-Series A

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 10 | Build automated intervention EV calculator for CSMs: input `tenant_id` + proposed discount → output EV from §34.5.1 using live CHS score and days-to-renewal. Ensures consistent application of the EV model across CSMs. | data-engineer + customer-success | **P2** | M16 | [ ] |
| 11 | Resolve OQ-CHURN-01 (§34.10): calibrate CHS band cutoffs against actual renewal outcomes from first 10 renewals. If Green band has > 15% churn, lower the threshold; if Red band has > 60% save rate without intervention, raise it. | data-engineer + customer-success | **P2** | M14 | [ ] |

---

### 34.10 Open Questions

| ID | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-CHURN-01** | **Are the CHS band cutoffs (70 = Amber, 40 = Red) optimal for renewal prediction?** Current thresholds are borrowed from OBSERVABILITY §33.3 (designed for operational monitoring, not renewal prediction). A cutoff that maximizes recall (catching at-risk accounts early) may differ from one maximizing precision (minimizing false alarms that consume CSM time). Recommendation: run a simple logistic regression on `{chs_score_at_90d_pre_renewal, renewed: bool}` after first 10 renewals to find the empirically optimal cutoff for FORM's specific customer base. | P2 | data-engineer + customer-success | After first 10 renewal outcomes (est. M14) |
| **OQ-CHURN-02** | ~~**Should the primary retention play be a multi-year price-lock rather than a percentage discount?**~~ **🟢 Resolved — §35.7 (2026-06-12).** NPV comparison framework adopted: price-lock advantage of $8,668 over 2yr horizon vs. 10% discount for a 300-seat Growth Amber account with CFO/procurement buyer. Five-profile decision matrix in §35.7.3 guides which tool to apply by customer profile. Empirical calibration after first 3 Amber renewals (M10–M12) per §35.10 item 10. | P1 → **Resolved** | customer-success + founder | §35.7 |
| **OQ-CHURN-03** | ~~**Should mid-contract termination risk have a separate DEC-030 event?**~~ **🟢 Resolved — §35.9 (2026-06-12).** `enterprise.mid_contract_termination_risk_flagged` (HIGH, 7yr) defined in §35.9.1: triggers when CHS < 20 sustained ≥ 4 weeks AND remaining contract term > 180 days; automated pg_cron emission; suppresses duplicates within 30 days per tenant. Register in `docs/AUDIT_LOG_SCHEMA.md §6` per §35.10 item 1 (P0/M10). | P2 → **Resolved** | enterprise-architect + customer-success | §35.9 |

---

*v1.3 (2026-06-11): §34 Enterprise Renewal Risk Register, Churn Decision Economics & Intervention Model. Three-band CHS-to-risk-band translation (Green/Amber/Red) with financial impact per tier; EV model for CSM intervention decisions (15h/$585 unit cost; positive EV for all bands except borderline Starter-minimum non-responsive); retention discount NPV model with hard price floor constraint from §31.5; contract vintage churn benchmarks (Y1 < 30%, Y2 < 15%, Y3+ < 8%) with NRR sensitivity table; post-churn recovery timeline (Starter 3–4 mo, Enterprise 15–28 mo); three new DEC-030 events (`enterprise.renewal_risk_flagged` STANDARD/3yr, `enterprise.churn_confirmed` HIGH/7yr, `enterprise.retention_discount_granted` HIGH/7yr) with privacy invariants (tenant-aggregate only, no user_id, no health data); 9-item implementation checklist (5× P0/M10, 4× P1/M12, 2× P2); 3 open questions (OQ-CHURN-01/02/03). Cross-references: §8.5 (enterprise CAC), §18.1 (ASC 606 deferred revenue reversal on mid-contract termination), §23 (NRR engine — NRR sensitivity table in §34.6.2), §26 (CS team scaling — CSM capacity and hourly cost), §31.5 (price floors — retention discount hard constraint), §31.8 (DEC-030 `enterprise.price_floor_override_requested` — fires on denied floor violations), §32 (pricing exception approval — retention discount > 15% approval chain), `docs/OBSERVABILITY.md §33` (tenant engagement health — CHS model, k-anonymity, AL-ENGAGE-* alert rules, `tenant_engagement_snapshots`), `docs/ENTERPRISE.md` (CS tier definitions, QBR cadence), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 event registry — three new event types to register), `docs/DECISION_LOG.md` (retention discount > 15% requires DEC-XXX entry per §34.9 item 9). Owner: customer-success + founder + finance + compliance-officer.*

---

*v1.2 (2026-06-11): §33 Enterprise AI Token Budget & Billing Architecture — OQ-COACH-02 Resolution (DEC-041). Flat per-seat billing adopted; internal token budgets defined per tier (Starter/Growth 50k/seat/mo, Enterprise 75k/seat/mo); `tenant_monthly_coaching_cost` view retained FORM-internal only; `tenant_config.coaching_seat_allowance_tokens` NULL at launch; overage policy §33.7 internal only; ASC 606 single-PO ratable recognition confirmed; two new DEC-030 events (`enterprise.ai_cost_monitor_alert`, `enterprise.ai_cost_outlier_flagged`) with tenant_id-only privacy invariant; 11-item implementation checklist (6× P0/M5, 3× P1/M6, 2× P2). Cross-references: DATA_MODEL.md §21.8 (`tenant_monthly_coaching_cost` view), DATA_MODEL.md §21.15 (OQ-COACH-02), docs/COST_MODEL.md §4 (unit economics), §8 (enterprise economics), §15 (enterprise infrastructure COGS), §26 (CSM cost model), §31 (pricing architecture), docs/ENTERPRISE.md (enterprise feature set), docs/AUDIT_LOG_SCHEMA.md (DEC-030 event registry), docs/DECISION_LOG.md (DEC-041). Owner: enterprise-architect + finance + compliance-officer.*

---

*v1.1 (2026-06-10): §32 Pricing Exception Approval Procedure — closes `docs/COST_MODEL.md §31.9` item 3 (P0, M4). Previously items 1 (DEC-030 event types in AUDIT_LOG_SCHEMA.md, v3.25.1) and 2 (price floor enforcement in pricing-enterprise.html calculator, v3.27.1) were closed; item 3 was the last open P0 gap. §32.1 scope: quick-test formula to determine whether approval is needed (effective_rate vs. list_price × 0.675 threshold); four-column table distinguishing covered vs. excluded scenarios. §32.2 approval authority: three-band table (–32.5% to –40% founder-only; –40% to floor founder + investor lead; below floor not permitted); post-Series A board replacement note. §32.3 step-by-step procedure: 7 steps — (1) confirm exception applies with rate calculation; (2) floor check with tier table; (3) approval request template with account/tier/seats/contract/rate/justification/margin/precedent fields; (4) approval routing with 48-hour escalation note and co-approver sequencing; (5) DEC-030 INSERT SQL with `floor_respected: true` invariant and `pricing_exception_event_id` linkage requirement; (6) quote preparation and cross-check to approved_rate; (7) DECISION_LOG.md entry template with DEC-XXX placeholder. §32.4 floor enforcement: four-step response to below-floor request; DEC-030 `enterprise.price_floor_override_requested` INSERT SQL with CRITICAL severity; escalation chain to compliance-officer and investor lead / board. §32.5 record keeping: seven-row retention table; pre-CRM folder structure `pricing-exceptions/YYYY-MM-DD_[tier]_[prospect-slug]/` with five artefact files. §32.6 SOC 2 relevance: CC1.4 / CC5.2 / CC5.3 / CC1.2 mapping. §32.7 checklist closure: three-row Definition of Done table; four remaining P0 actions before first enterprise quote. Cross-references: §31.5 (price floors), §31.6 (discount authority matrix), §31.8 (DEC-030 event specs), §31.9 (checklist being closed), docs/AUDIT_LOG_SCHEMA.md (enterprise.pricing_exception_approved + enterprise.price_floor_override_requested), docs/ENTERPRISE.md (pricing table and sales process), docs/DECISION_LOG.md (DEC-038). Owner: founder + compliance-officer.*

---

*v1.0 (2026-06-10): §31 Pricing Architecture, Competitive Benchmarking & Discount Governance — closes the gap between FORM's price list and the documented rationale behind it. §31.1 scopes the section: consumer Pro derivation, enterprise tier derivation, corporate wellness benchmarking, COGS-anchored price floors, discount authority matrix, pricing change governance, DEC-030 events, checklist, open questions. §31.2 consumer Pro price derivation ($19/month): WTP anchor table against human PT ($200–$600), AI coaching (Future/Caliber $149–$199), smart fitness (Whoop/Oura $6–$30), personalised apps (Fitbod $15/Ladder $20), general apps (free–$10); key insight — FORM competes on AI coach axis, not fitness tracker axis; 85–90% cheaper than human-supervised AI coaching; COGS-coverage analysis at $19 confirms 96.9% gross margin (SBP) and 96.2% (standard 30%); price sensitivity range bear $12 through stretch $39 with GM table; consumer price floor $12/month with reasoning. §31.3 enterprise price derivation: three-input methodology (COGS floor + target GM, competitive WTP ceiling, value anchor vs. consumer); Starter $12 derivation (removes App Store fee; fully-loaded GM 83.3% with email CSM allocation $1.67/seat); Growth $9 derivation (25% volume discount; fully-loaded GM 85.1% with named CSM $1.00/seat at 300-seat average — higher fully-loaded GM than Starter due to labour scale); Enterprise $6–8 derivation ($7 midpoint for modelling; floor $6 from §31.5; ceiling $8 from competitive benchmarking; fully-loaded GM 88.0% at $7 midpoint with dedicated CSM $0.40–0.50/seat at 1,000+ seats); ACV floor table: Starter 50 seats = $7,200/year, Growth 201 seats = $21,708, Enterprise 1,001 seats = $84,084; OQ-11 resolution: 50-seat Starter floor defensible at 73.5% Year 1 GM including implementation cost amortisation; recommend keep 50-seat floor. §31.4 corporate wellness competitive benchmarking: eight platforms benchmarked with [ESTIMATE] price ranges — Wellhub/Gympass ($8–$45 gym network), Headspace for Work ($10–$14 meditation), Calm for Business ($10–$15), Virgin Pulse/Personify Health ($40–$75 incentive platform), Noom for Work ($30–$50 weight management; FORM cannot compete here by ethics), Peloton Corporate ($20–$40 hardware-dependent), Whoop for Teams ($18–$30 wearable-only), Future enterprise ($99–$149 human coach); ASCII positioning matrix; positioning conclusion: FORM occupies unique $6–12 AI coaching quadrant with no direct competitor at this price/feature intersection; sales framing: FORM replaces gym stipend ($600–$1,800/year) not Wellhub (gym network) or Headspace (mental wellness); four-point anchor script for sales conversations. §31.5 COGS-anchored price floor analysis: COGS floor table (Starter $3.00/seat, Growth $2.00, Enterprise $1.50) including $0.15 compliance overhead amortisation; rationale that COGS floor is theoretical; contractual price floors set at 50% of list (Starter $6.00, Growth $4.50, Enterprise $4.00) to preserve > 55% GM at floor; Enterprise $6/seat edge case noted — floor allows only 33% headroom at lowest list price. §31.6 discount authority matrix: pre-approved schedule (2yr –15%, 3yr –25%, upfront –10%; max –32.5% multiplicative — enforced in calculator); non-standard discount levels (up to –40% = founder-only, up to –50%/floor = founder + investor pre-Series A; below floor NOT PERMITTED at any approval level); contractual floor summary table with floor as % of list and maximum discount per tier; non-pricing concessions list (extended pilot, priority scheduling, executive call — no pricing approval required). §31.7 pricing change governance: consumer changes (30-day notice, annual grandfathering, price decrease immediate, founder approval + DECISION_LOG); enterprise list price changes (active contract audit, 90-day notice at renewal, investor consult, MSA update, DEC-030 event); annual price indexation clause CPI+1% capped 5% (references §23.6 NRR expansion lever); rate lock commitment for contracted term; four signals indicating underprice vs. four signals indicating overprice or competitive pressure. §31.8 four DEC-030 pricing audit events: `enterprise.pricing_exception_approved` (HIGH, 7yr — before quote sent; links to `enterprise.contract_signed` via `pricing_exception_event_id`), `enterprise.consumer_price_updated` (HIGH, 7yr — includes grandfathering policy text), `enterprise.list_price_updated` (HIGH, 7yr — captures count of active contracts at prior list), `enterprise.price_floor_override_requested` (CRITICAL, 7yr — records even denied attempts). §31.9 eight-item checklist: 3× P0/M4 (DEC-030 event registration + Zod schemas, calculator floor enforcement, exception approval procedure), 2× P1/M5–M6 (MSA price escalation clause, quarterly competitive intelligence update process), 1× P1/M6 (COMPETITIVE.md enterprise section), 2× P2/M9–M12 (WTP validation study with pilot participants, annual pricing review). §31.10 three open questions: OQ-PRICE-01 (non-profit/edu discount tier — defer until three commercial deals close; track conversion channel signal), OQ-PRICE-02 (consumer $19 price review trigger — D90 ≥ 40% + NPS ≥ 60 + ARR ≥ $200k before increasing to $24/month; P2 checkpoint M12), OQ-PRICE-03 (CV-optional enterprise deals — single-price model for now; create "Professional" SKU at $8/seat if CV restriction objection recurs in > 3 deals; P2 M12 review). Cross-references: §3 (infrastructure COGS used in §31.2 and §31.3 margin calculations), §4 (unit economics table — §31.2.2 cross-checks GM figures), §8 (Enterprise Economics — §31.3 extends the enterprise pricing rationale), §11 OQ-11 (Starter 50-seat floor — resolved in §31.3.3), §15 (Enterprise Infrastructure COGS — compliance overhead allocation in §31.5), §20 (Customer ROI Model — gym stipend substitution in §31.4.2 sales framing), §21 (Pilot Economics & Discount Governance — §31.6 formalises the authority matrix referenced there), §23.6 (Annual price indexation — MSA clause in §31.7.2 enables this NRR lever), §26 (CSM Cost Model — labour allocations in §31.3.2 and §31.5), docs/ENTERPRISE.md (pricing table and sales process), docs/MSA_TEMPLATE.md (price escalation clause — §31.7.2 requires update), docs/COMPETITIVE.md (consumer competitors — enterprise benchmarks to be added per §31.9 item 6), docs/DECISION_LOG.md (pricing change decisions), docs/AUDIT_LOG_SCHEMA.md (four new DEC-030 event types). Owner: founder + product-strategist + compliance-officer.*

---

## 35. Enterprise Contract Amendment, Mid-Contract Termination & Early Termination Fee Model

### 35.1 Purpose & Scope

This section closes two open questions from §34:

- **OQ-CHURN-02 (P1):** Should the primary retention play be a multi-year price-lock rather than a percentage discount? — resolved in §35.7 with NPV comparison framework.
- **OQ-CHURN-03 (P2):** Should mid-contract termination risk have a separate DEC-030 event? — resolved in §35.9 with `enterprise.mid_contract_termination_risk_flagged` event definition.

**What §16.4, §18.5, and §31.7 already cover:**
- §16.4: Multi-year discount economics (FORM's perspective — GP sacrifice vs. churn elimination)
- §18.5: Deferred revenue waterfall for ASC 606 purposes on multi-year prepayments
- §31.7: Rate lock commitment (customers cannot be repriced during contracted term)

**What this section adds:**
- Contract amendment taxonomy and approval workflow
- Seat reduction policy (can a customer reduce contracted seats mid-term?)
- Mid-contract termination economics: immediate ARR loss, deferred revenue reversal, CAC destruction
- Early Termination Fee (ETF) structure, calculation methodology, and waiver framework
- Multi-year price-lock as a retention tool vs. cash discount (NPV comparison — OQ-CHURN-02)
- `enterprise.mid_contract_termination_risk_flagged` DEC-030 event (OQ-CHURN-03)
- `enterprise.contract_amended` and `enterprise.early_termination_fee_waived` DEC-030 events

**Audience:** CSMs handling at-risk accounts, founder in contract negotiations, compliance-officer for MSA review, finance for deferred revenue management.

---

### 35.2 Contract Amendment Taxonomy

Enterprise MSAs may be amended mid-term for four reasons. Each carries a different financial signature and approval requirement.

| Amendment Type | Description | Financial Signature | Approval Required | DEC-030 Event |
|---|---|---|---|---|
| **Seat expansion** | Customer adds seats above contracted quantity | Positive ARR delta; new MRR starts at amendment date | CSM approval (standard; > 20% ACV increase → founder review) | `enterprise.account_expanded` (§23.7) |
| **Tier upgrade** | Customer moves Starter → Growth or Growth → Enterprise mid-term | Positive ARR delta; pro-rated billing from amendment date | CSM + compliance-officer (DPA addendum if new features access health data) | `enterprise.account_expanded` with `expansion_type = 'tier_upgrade'` |
| **Seat reduction** | Customer reduces contracted seat count | Negative ARR delta; potential deferred revenue credit | Founder approval required; subject to §35.3 | `enterprise.contract_amended` with `amendment_type = 'seat_reduction'` |
| **Early termination** | Customer exits before contract end date | Immediate ARR loss + deferred revenue reversal; ETF may apply | Founder + investor (if ETF waived > $10k); see §35.6 | `enterprise.early_termination_fee_waived` or no event if ETF collected |

**General rule:** FORM does not permit seat reductions below the tier minimum (50 seats for Starter, 201 for Growth, 1,001 for Enterprise) mid-contract. A customer wanting to go below tier minimum must either terminate (ETF applies) or negotiate a downgrade at the next annual renewal window.

---

### 35.3 Seat Reduction Policy (Mid-Contract)

**Default position: seat count is fixed for the contracted term.** The customer committed to a seat count at signing; FORM committed to deliver service at that level. Seat reductions mid-term reduce FORM's ARR below the contracted amount and destroy the unit economics modeled in §16.

| Reduction Size | Policy |
|---|---|
| ≤ 10% of contracted seats | Not permitted without founder approval. If approved as goodwill concession, effective at next annual renewal date — no mid-term billing credit. |
| 10–30% of contracted seats | Founder approval required. If approved (e.g., documented company downsizing), effective at next annual renewal date. Must not fall below tier minimum. |
| > 30% of contracted seats OR below tier minimum | Treated as partial early termination. ETF applies to the terminated portion per §35.5. Customer may alternatively terminate the full contract with full ETF. |
| Force majeure (documented company closure or acquisition resulting in workforce < contracted seats) | ETF may be waived per §35.6. Requires compliance-officer review and DECISION_LOG entry. |

**Why the strict default:** Each seat represents $84–$144/year in contracted ARR. A 30% reduction on a 300-seat Growth account equals $8,100/year ARR loss. Implementation cost ($7,200 for Growth, per §8.2) is front-loaded — allowing seat reductions in Year 1 guarantees negative unit economics. Additionally, customers who are struggling most (low CHS) are most likely to request reductions; allowing easy reductions removes the renewal-pressure signal that drives CS intervention.

---

### 35.4 Mid-Contract Termination Economics

When a customer terminates a multi-year contract early, FORM faces three distinct financial hits that must all be quantified before deciding whether to enforce the ETF.

#### 35.4.1 Three Components of Early Termination Loss

| Component | Definition | Example: Growth 300-seat, 3yr contract (–25% = $24,300/yr), exits at M18 |
|---|---|---|
| **Remaining contracted ARR** | ACV for each contract year not fulfilled | 1.5yr × $24,300 = $36,450 |
| **Deferred revenue reversal** | Paid-but-unearned revenue that must be returned if customer prepaid annually | 6 months prepaid × $2,025/mo = $12,150 to be returned |
| **CAC destruction** | Unamortized CAC expensed immediately at exit | $7,200 × (18/36 remaining) = $3,600 written off |

**Total economic loss (Growth 300-seat, M18 exit from 3yr deal): ~$52,200**

Against this, the ETF (§35.5) is designed to recover a meaningful fraction of the foregone ARR without being so punitive as to trigger legal disputes or brand damage.

#### 35.4.2 Loss by Tier and Exit Point

*Note: ACV figures use contracted rates including multi-year discounts (§16.4): 2yr = –15%, 3yr = –25%.*

| Tier | Seats | Contracted ACV/yr | Contract | Exit at M6 — ARR foregone | Exit at M18 — ARR foregone | CAC written off (M6) |
|---|---|---|---|---|---|---|
| **Starter** | 50 | $6,120 (2yr) | 2yr | $9,180 | $3,060 | $1,013 |
| **Starter** | 50 | $5,400 (3yr) | 3yr | $18,900 | $12,600 | $1,013 |
| **Growth** | 300 | $27,540 (2yr) | 2yr | $41,310 | $13,770 | $5,400 |
| **Growth** | 300 | $24,300 (3yr) | 3yr | $68,850 | $45,900 | $5,400 |
| **Enterprise** | 1,000 | $71,400 (2yr) | 2yr | $107,100 | $35,700 | $16,875 |
| **Enterprise** | 1,000 | $63,000 (3yr) | 3yr | $189,000 | $126,000 | $16,875 |

---

### 35.5 Early Termination Fee Structure & Calculation

#### 35.5.1 ETF Design Principles

1. **Recovery target:** 40–60% of remaining contracted ARR. This is the B2B SaaS standard range and is defensible in contract negotiations.
2. **Not punitive:** ETF compensates FORM for unrecovered CAC and foregone contract ARR. MSA language must use *"liquidated damages representing a genuine pre-estimate of loss"* — courts in DE and US jurisdictions are more likely to uphold compensation-framed ETFs than penalty-framed ones. Outside counsel review required (§35.10 item 2).
3. **Declining balance:** ETF decreases as the customer completes more of the contracted term.
4. **Floor:** Minimum ETF = 1 month contracted ACV for any early exit, regardless of timing.

#### 35.5.2 ETF Formula

```
ETF = MAX(
  1 × monthly_contracted_ACV,                            -- minimum floor
  remaining_months × monthly_contracted_ACV × etf_rate
)
```

`etf_rate` is a function of the month of exit (§35.5.3). ETF is calculated on **contracted ACV** (after multi-year discount), not list ACV. See OQ-ETF-02 for the legal rationale.

#### 35.5.3 ETF Rate Schedule by Exit Month

| Month of Exit (from contract start) | ETF Rate (% of remaining contracted value) | Notes |
|---|---|---|
| M1–M3 | **60%** | Exit in first quarter — full pre-launch investment unrecovered |
| M4–M6 | **55%** | Pilot period; implementation cost not yet amortized |
| M7–M12 | **50%** | Standard first-year ETF; most common scenario |
| M13–M18 | **40%** | Mid-contract of 2yr deal or Year 2 of 3yr |
| M19–M24 | **30%** | Late-stage 2yr deal; lower recovery acceptable |
| M25–M30 | **25%** | Year 3 of 3yr deal |
| M31–M35 | **15%** | Near end of 3yr deal |
| M36 (last month) | **0%** (floor only) | Near-expiry; no meaningful remaining value |

#### 35.5.4 ETF Calculation Examples

**Example A: Growth 300-seat, 2yr deal (–15% = $27,540/yr contracted, $2,295/mo). Exits at M9.**
- Remaining months: 24 − 9 = 15
- ETF rate at M9: 50%
- ETF = $2,295 × 15 × 50% = **$17,213**
- Floor: $2,295 → not binding
- **ETF payable: $17,213**

**Example B: Enterprise 1,000-seat, 3yr deal (–25% = $63,000/yr contracted, $5,250/mo). Exits at M18.**
- Remaining months: 36 − 18 = 18
- ETF rate at M18: 40%
- ETF = $5,250 × 18 × 40% = **$37,800**
- **ETF payable: $37,800**

**Example C: Starter 50-seat, 2yr deal (–15% = $6,120/yr contracted, $510/mo). Exits at M23.**
- Remaining months: 24 − 23 = 1
- ETF rate at M23: 30%
- ETF = $510 × 1 × 30% = $153 → below floor
- Floor: $510
- **ETF payable: $510 (floor)**

#### 35.5.5 ETF Approval Authority Matrix

| ETF Amount | Approval Authority | Process |
|---|---|---|
| ETF ≤ $5,000 | CSM lead | Note in CRM; emit `enterprise.early_termination_fee_waived` if waived |
| $5,001 – $25,000 | Founder | 48h decision window; DECISION_LOG entry |
| $25,001 – $100,000 | Founder + compliance-officer | DECISION_LOG entry; investor notification |
| > $100,000 | Founder + investor lead | Board notification if ETF waiver > $100k |

**Waiving the ETF does not require the §32 pricing exception procedure** — ETF is a liquidated damages clause, not a pricing discount. It has its own approval chain above.

---

### 35.6 ETF Waiver Decision Framework

Waiving the ETF eliminates cash recovery but may preserve the relationship, reference value, and future re-engagement opportunity.

#### 35.6.1 Factors Favouring ETF Enforcement

| Factor | Rationale |
|---|---|
| Customer is a competitor's acquisition target | Relationship is ending regardless |
| Customer showed bad faith (false pilot metrics, disputed charges) | Enforce to set precedent for contract integrity |
| ETF > $25k and FORM is cash-constrained | Cash recovery outweighs reputational risk |
| Documented competitor undercutting on price | Reference value is likely already lost |
| No expansion potential (company downsizing permanently) | No future upsell to preserve |

#### 35.6.2 Factors Favouring ETF Waiver

| Factor | Rationale |
|---|---|
| Potential reference account in FORM's sweet spot (engineering/product org) | Reference + case study value may exceed ETF cash value |
| Documented product gap caused under-utilisation | FORM's issue; waiving is appropriate and prevents negative G2/Glassdoor reviews |
| Champion leaving and successor not yet won | Invest in relationship; waiver earns goodwill with new champion |
| ETF < $5,000 and legal enforcement cost is non-trivial | Management overhead of collecting small ETF exceeds its value |
| Force majeure (acquisition, bankruptcy, restructuring) | Ethical and legal risk of enforcement in genuine force-majeure |

#### 35.6.3 ETF Waiver Expected Value Model

Consistent with the intervention EV model in §34.4, frame the enforcement/waiver decision as an EV calculation.

**Variables:**
- `ETF_cash`: ETF amount if collected
- `P_collect`: probability of successful collection (0.7 undisputed; 0.3 if customer likely to dispute)
- `legal_cost`: estimated cost to collect if disputed ($3,000–$10,000 for B2B SaaS disputes)
- `P_re_engage_waive`: probability of future re-engagement if ETF waived (0.10–0.40 depending on churn reason)
- `future_ACV`: expected future ACV if re-engaged (tier-appropriate)
- `reference_value`: qualitative; treat as $5,000 for a named reference in an ICP industry

**EV of enforcement:**
```
EV_enforce = P_collect × ETF_cash − (1 − P_collect) × legal_cost
```

**EV of waiver:**
```
EV_waive = P_re_engage_waive × future_ACV / (1 + 0.15)^2 + reference_value
```

**Decision rule:** Waive if `EV_waive > EV_enforce`.

**Example A — Growth 300-seat, ETF $17,213, documented product gap:**
- EV enforce: 0.5 × $17,213 − 0.5 × $5,000 = $8,607 − $2,500 = **$6,107**
- EV waive: 0.25 × $32,400 / 1.32 + $5,000 = $6,136 + $5,000 = **$11,136**
- **Decision: waive**

**Example B — Enterprise 1,000-seat, ETF $37,800, competitor acquisition:**
- EV enforce: 0.8 × $37,800 − 0.2 × $5,000 = $30,240 − $1,000 = **$29,240**
- EV waive: 0.05 × $84,000 / 1.32 + $0 = **$3,182** (acquired → re-engagement probability near zero)
- **Decision: enforce**

The EV model must be documented and filed in the account record. For ETF waivers > $5,000, `ev_analysis_completed: true` is required in the DEC-030 event payload or the `emit-audit-event` Worker returns HTTP 422.

---

### 35.7 Multi-Year Price-Lock as Retention Tool (OQ-CHURN-02 Resolution)

**Resolution status: PARTIALLY RESOLVED — framework adopted 2026-06-12. Empirical calibration after first 3 Amber-band renewals per §35.10 item 10 (est. M10–M12).**

#### 35.7.1 The Retention Discount vs. Price-Lock Trade-off

§34 identifies two primary retention tools for Amber-band customers at renewal:

- **Retention discount:** reduce per-seat rate 5–15% for a 1-year renewal. Triggers §32 exception procedure if > 15%. Costs FORM ARR immediately and creates a discounted base for future renewals.
- **Multi-year price-lock:** offer renewal at *current contracted rate* for 2 years with a contractual guarantee that the CPI+1% escalation clause (§23.6) will not apply during the locked term.

The price-lock does not reduce ARR. It foregoes a future price increase that was conditional on the customer renewing anyway. The lock creates certainty for both parties.

#### 35.7.2 NPV Comparison: Discount vs. Price-Lock for Growth 300-Seat Amber Customer

*Assumptions: current contracted rate $9.00/seat/month → ACV $32,400/yr; CPI escalation foregone under price-lock ≈ 3.5%/yr = $1,134/yr; discount rate 15%.*

**Option A: 10% retention discount, 1-year renewal**

| Year | Revenue | Probability-adjusted | NPV factor | NPV contribution |
|---|---|---|---|---|
| Y1 (discounted at –10%) | $29,160 | 1.00 | 1.00 | $29,160 |
| Y2 (at-risk; no new commitment) | $32,400 | 0.80 × 0.70 = 0.56 | 0.87 | $15,713 |
| **2yr NPV** | | | | **$44,873** |

ARR sacrifice: $3,240/yr. §32 exception procedure required if discount > 15%.

**Option B: 2yr price-lock at current $9.00/seat/month**

| Year | Revenue | Probability-adjusted | NPV factor | NPV contribution |
|---|---|---|---|---|
| Y1 (locked) | $32,400 | 1.00 | 1.00 | $32,400 |
| Y2 (locked) | $32,400 | 0.75 | 0.87 | $21,141 |
| **2yr NPV** | | | | **$53,541** |

ARR sacrifice: $0 upfront + $1,134/yr CPI escalation foregone (conditional on Y2 retention anyway).

**Price-lock NPV advantage: $53,541 − $44,873 = $8,668 over 2 years for a 300-seat Growth account** where both tools achieve similar retention probabilities with a CFO/procurement-led buyer.

#### 35.7.3 Decision Matrix: Which Tool to Use by Customer Profile

| Customer Profile | Preferred Tool | Rationale |
|---|---|---|
| **CFO/procurement-led buyer** (budget certainty is the primary concern) | Price-lock | Guaranteed 2yr budget line. No discount needed — certainty itself is the value. Does not trigger §32 exception procedure. |
| **Champion is leaving** (relationship risk is the primary threat) | 1yr discount only | Price-lock requires trust over 2yrs. Win the new champion first; do not commit to a 2yr lock without an established relationship. |
| **Documented product gap** | Discount + roadmap commitment | Price-lock on a product that doesn't serve the need is not credible. Discount + named roadmap item is the right trade. |
| **Low utilisation, Amber CHS** | Price-lock + free activation sprint | Low usage means customer doesn't see value. Adding a 4-week CSM-led activation support engagement alongside the price-lock addresses the root cause. |
| **Active competitor pressure** (customer has a competing quote) | Retention discount | Competitor is anchoring on rate. Price-lock is irrelevant when the customer is negotiating absolute price. Match the competitive rate for 1yr; re-assess at Y2 renewal. |
| **Expansion expected** (headcount growing, CHS trending toward Green) | 2yr price-lock | Locks in expansion at current economics. Customer benefits from rate certainty; FORM benefits from guaranteed seat growth within the locked term. |

#### 35.7.4 Constraints on Price-Lock Offers

1. **§32 exception procedure still applies** if the locked rate is already below list price due to a prior non-standard discount. Locking a below-list rate for 2 years extends the exception period — founder sign-off required.
2. **Rate lock applies to per-seat rate only.** Seat additions within the locked term are billed at the locked rate (positive for the customer; acceptable for FORM given expansion adds positive margin contribution).
3. **CPI indexation resumes** at Year 3 renewal from the locked base rate. The lock does not permanently suppress the escalation clause.
4. **DEC-030 required:** Every accepted price-lock renewal must emit `enterprise.contract_amended` with `amendment_type = 'price_lock_renewal'` and `rate_locked_usd_per_seat` to preserve the audit trail.

---

### 35.8 ASC 606 / IFRS 15 Treatment for Terminations & Amendments

#### 35.8.1 Early Termination

| Scenario | ASC 606 / IFRS 15 Treatment |
|---|---|
| Customer prepaid annually and terminates at M6 | Remaining 6 months of prepayment is a contract liability (deferred revenue). At termination, deferred revenue is reversed and either refunded per MSA terms or retained as ETF consideration. |
| Customer pays monthly (no prepayment) | No deferred revenue to reverse. Revenue recognition ceases at termination date. ETF, if collected, is recognized as a settlement payment on receipt. |
| ETF is waived | No settlement recognized. Revenue recognition ceases at termination date. Loss = unamortized CAC (§35.4.1 component 3). |
| ETF is disputed | Constrain the variable consideration estimate: recognize ETF only to the extent it is highly probable of collection. Recognize $0 until dispute is resolved. |

#### 35.8.2 Mid-Contract Seat Reduction (If Approved per §35.3)

A seat reduction is a contract modification under ASC 606. For FORM's seat-based SaaS model:

- **Prospective modification** (remaining performance obligations repriced at reduced seat count from amendment date): most common treatment. Recognize modified price over remaining term.
- **FORM's policy:** MSA should specify that mid-contract seat reductions are effective only at the next annual renewal date. This avoids mid-period contract modification accounting complexity and is consistent with §35.3.

#### 35.8.3 Price-Lock Renewal as Modification

A price-lock renewal is a contract modification that replaces the expiring contract with a new contract at the same rate. Treatment: recognize as a new contract for the 2yr lock period at the contracted rate. The foregone CPI escalation is not an accounting entry — it is a counterfactual that does not enter the ledger.

---

### 35.9 DEC-030 HMAC-Chained Audit Events (Closes OQ-CHURN-03)

Three new DEC-030 events complete the contract amendment and termination audit chain. `enterprise.mid_contract_termination_risk_flagged` is the specific event requested in OQ-CHURN-03.

#### 35.9.1 Event: `enterprise.mid_contract_termination_risk_flagged`

| Field | Value |
|---|---|
| **Event type** | `enterprise.mid_contract_termination_risk_flagged` |
| **Severity** | HIGH |
| **Retention** | 7 years |
| **Actor** | `system:pg_cron` (automated) or `csm_user_id` (manual flag) |
| **Trigger** | CHS score < 20 sustained ≥ 4 consecutive weeks AND remaining contract term > 180 days |
| **Privacy invariant** | `tenant_id` only; no `user_id`; no health data |

**Payload (Zod v2 schema):**
```typescript
z.object({
  tenant_id:               z.string().uuid(),
  chs_score_current:       z.number().int().min(0).max(100),
  chs_weeks_below_20:      z.number().int().min(4),
  contract_end_date:       z.string().datetime(),
  days_remaining:          z.number().int().min(181),
  contract_type:           z.enum(['1yr', '2yr', '3yr']),
  contracted_acv_usd:      z.number().int().positive(),
  estimated_etf_usd:       z.number().int().nonnegative(),
  risk_signal_source:      z.enum(['pg_cron_monitor', 'csm_manual_flag']),
})
```

**Deduplication:** Suppresses duplicate events if an unresolved `enterprise.mid_contract_termination_risk_flagged` was emitted within the last 30 days for the same `tenant_id`.

**HMAC chain INSERT:**
```sql
INSERT INTO audit_log (
  event_ts, event_type, actor_id, tenant_id, payload,
  sequence_number, prev_hash, hmac_signature
)
VALUES (
  now(),
  'enterprise.mid_contract_termination_risk_flagged',
  $actor_id,           -- 'system:pg_cron' or CSM UUID
  $tenant_id,
  jsonb_build_object(
    'chs_score_current',   $chs_score,
    'chs_weeks_below_20',  $weeks_below_20,
    'contract_end_date',   $end_date,
    'days_remaining',      $days_remaining,
    'contract_type',       $contract_type,
    'contracted_acv_usd',  $contracted_acv,
    'estimated_etf_usd',   $estimated_etf,
    'risk_signal_source',  $source
  ),
  nextval('audit_log_sequence'),
  $prev_hash,
  encode(hmac(
    concat($seq::text, $prev_hash,
           'enterprise.mid_contract_termination_risk_flagged',
           $tenant_id::text, $payload::text),
    current_setting('app.hmac_secret'), 'sha256'
  ), 'hex')
);
```

---

#### 35.9.2 Event: `enterprise.contract_amended`

| Field | Value |
|---|---|
| **Event type** | `enterprise.contract_amended` |
| **Severity** | HIGH |
| **Retention** | 7 years |
| **Actor** | CSM or founder executing the amendment |
| **Trigger** | Any signed contract amendment (seat change, tier change, price-lock renewal, term extension) |

**Payload (Zod v2 schema):**
```typescript
z.object({
  tenant_id:                z.string().uuid(),
  amendment_type:           z.enum([
    'seat_reduction', 'seat_expansion', 'tier_upgrade', 'tier_downgrade',
    'price_lock_renewal', 'term_extension', 'force_majeure_modification'
  ]),
  seats_before:             z.number().int().positive(),
  seats_after:              z.number().int().positive(),
  acv_before_usd:           z.number().int().positive(),
  acv_after_usd:            z.number().int().positive(),
  rate_locked_usd_per_seat: z.number().optional(),       // price_lock_renewal only
  new_contract_end_date:    z.string().datetime().optional(),
  approval_actor_id:        z.string().uuid(),
  amendment_docusign_id:    z.string().optional(),
})
```

---

#### 35.9.3 Event: `enterprise.early_termination_fee_waived`

| Field | Value |
|---|---|
| **Event type** | `enterprise.early_termination_fee_waived` |
| **Severity** | HIGH |
| **Retention** | 7 years |
| **Actor** | Founder (or founder + investor lead for large waivers) |
| **Trigger** | ETF waiver decision per §35.5.5 approval matrix |

**Payload (Zod v2 schema):**
```typescript
z.object({
  tenant_id:               z.string().uuid(),
  etf_amount_waived_usd:   z.number().int().positive(),
  exit_month:              z.number().int().min(1),
  remaining_months:        z.number().int().min(1),
  waiver_reason:           z.enum([
    'product_gap', 'force_majeure', 'champion_departed', 'reference_value',
    'competitor_acquisition', 'legal_dispute_resolution', 'goodwill_concession',
    'small_etf_enforcement_uneconomic'
  ]),
  approval_actor_id:       z.string().uuid(),
  ev_analysis_completed:   z.boolean(),
  decision_log_ref:        z.string().optional(),
})
```

> **Enforcement invariant:** `ev_analysis_completed` must be `true` for `etf_amount_waived_usd > 5000`. If `ev_analysis_completed: false` and the amount exceeds this threshold, the `emit-audit-event` Worker returns HTTP 422 with `ETF_WAIVER_REQUIRES_EV_ANALYSIS`. CSM must complete the §35.6.3 EV model before proceeding.

---

#### 35.9.4 DEC-030 Event Summary for §35

| Event | Severity | Retention | Trigger | Privacy constraint |
|---|---|---|---|---|
| `enterprise.mid_contract_termination_risk_flagged` | HIGH | 7 yr | CHS < 20 for ≥ 4 weeks AND days remaining > 180 | `tenant_id` only; no `user_id` |
| `enterprise.contract_amended` | HIGH | 7 yr | Any signed contract amendment | `tenant_id`; pseudonymous actor IDs |
| `enterprise.early_termination_fee_waived` | HIGH | 7 yr | ETF waiver per §35.5.5 approval matrix | `tenant_id`; waiver reason enum required |

All three events must be registered in `docs/AUDIT_LOG_SCHEMA.md §6` before the first enterprise contract amendment or termination is processed.

---

### 35.10 Implementation Checklist

#### P0 — Must complete before first enterprise multi-year contract close (M10)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Register `enterprise.mid_contract_termination_risk_flagged`, `enterprise.contract_amended`, and `enterprise.early_termination_fee_waived` in `docs/AUDIT_LOG_SCHEMA.md §6` with Zod schemas from §35.9. Deploy to `emit-audit-event` Worker. | enterprise-architect | **P0** | M10 | [ ] |
| 2 | Add ETF clause to `docs/MSA_TEMPLATE.md`: liquidated damages framing (§35.5.1), declining balance rate schedule (§35.5.3), minimum floor, seat reduction policy (§35.3). Have outside counsel review for enforceability in DE/US jurisdictions. | compliance-officer + outside counsel | **P0** | M10 | [ ] |
| 3 | Add `enterprise.mid_contract_termination_risk_flagged` to `docs/OBSERVABILITY.md §12.6` pg_cron registry. Implement the weekly CHS < 20 check with 30-day deduplication. PagerDuty P1 alert (CSM + CS lead) on emission. | devops-lead + data-engineer | **P0** | M10 | [ ] |
| 4 | Implement `ev_analysis_completed` enforcement in `emit-audit-event` Worker: return HTTP 422 with `ETF_WAIVER_REQUIRES_EV_ANALYSIS` if `etf_amount_waived_usd > 5000` and `ev_analysis_completed: false`. | platform-engineer | **P0** | M10 | [ ] |

#### P1 — Before first enterprise contract renewal (M12)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 5 | Add §35.6.3 EV model to the internal CS playbook as the standard ETF waiver decision process. Run a 1-hour CSM training session before the first at-risk renewal. | customer-success | **P1** | M12 | [ ] |
| 6 | Add price-lock renewal option to standard renewal offer templates. CSMs should default to price-lock for CFO/procurement-led Amber accounts per §35.7.3. | customer-success | **P1** | M12 | [ ] |
| 7 | Add §35.8 ASC 606 treatment to the internal finance runbook: deferred revenue reversal procedure on early termination; price-lock renewal accounting. | finance | **P1** | M12 | [ ] |
| 8 | Add `enterprise.contract_amended` cross-reference requirement to `docs/DECISION_LOG.md`: any seat reduction > 10% must include a DECISION_LOG entry (DEC-XXX) linking to the `enterprise.contract_amended` event ID. | compliance-officer | **P1** | M12 | [ ] |

#### P2 — Post first 3 renewals (M14–M16)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 9 | Calibrate ETF rate schedule (§35.5.3) against actual negotiation outcomes. If customers routinely negotiate below the 50% midpoint, recalibrate the schedule to reduce friction while preserving CAC recovery. | customer-success + founder | **P2** | M16 | [ ] |
| 10 | Calibrate price-lock vs. discount NPV model (§35.7.2) using first 3 Amber renewal outcomes. Update retention probabilities (currently [ESTIMATE]) with actuals. If price-lock retention rate < 65%, rebalance toward discount. | customer-success + data-engineer | **P2** | M16 | [ ] |
| 11 | Evaluate whether the mid-contract CHS < 20 trigger should spawn an automated CSM intervention protocol (parallel to §34 renewal risk intervention at 90 days). Document outcome in `docs/DECISION_LOG.md`. | customer-success + product-manager | **P2** | M16 | [ ] |

---

### 35.11 Open Questions

| ID | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-ETF-01** | **Is the ETF rate schedule (§35.5.3) enforceable as written in DE and US jurisdictions?** DE law (§§339–340 BGB) distinguishes valid contractual penalties from void forfeitures: compensation must be a genuine pre-estimate of loss. The "liquidated damages" framing and declining balance schedule are designed to support this. US enforceability varies by state (CA disfavours penalty clauses; NY/DE are more permissive). Outside counsel review at MSA review (§35.10 item 2) resolves this. | **P0** | compliance-officer + outside counsel | M10 — MSA legal review |
| **OQ-ETF-02** | **Should the ETF calculation use contracted ACV or list ACV?** §35.5.2 uses contracted ACV (post multi-year discount). This is the correct economic basis — FORM's actual revenue commitment is the contracted rate. However, some ETF clauses reference list ACV (higher ETF amounts; stronger recovery). Using list ACV risks being characterised as a penalty clause in DE. Current position: contracted ACV basis. Confirm with counsel at MSA review. | **P1** | compliance-officer | M10 |
| **OQ-ETF-03** | **Should FORM offer a seat suspension option for accounts in temporary headcount contraction (e.g., a tech company in a hiring freeze)?** A 3–6 month seat suspension would pause billing on unused seats while preserving the relationship. Risk: sets a precedent for billing disputes; complex to implement in Stripe. Alternative: allow 10% seat reduction at next annual date at no ETF (already in §35.3 policy). Recommendation: defer until a customer explicitly requests suspension. Document the first request outcome in `docs/DECISION_LOG.md`. | **P2** | customer-success + founder | On first customer request; no later than M18 |

---

*v2.2 (2026-06-12): §36 Enterprise Implementation Cost Deep-Dive. See §36.1–§36.12 below.*

*v2.1 (2026-06-12): §35 Enterprise Contract Amendment, Mid-Contract Termination & Early Termination Fee Model. Closes OQ-CHURN-02 (P1, §34.10) with NPV comparison of 2yr price-lock vs. 10% retention discount for Amber-band Growth accounts: price-lock NPV advantage of $8,668 over 2yr horizon for CFO/procurement-led buyers (price-lock 2yr NPV $53,541 vs. discount $44,873); six-profile decision matrix (§35.7.3) guiding tool selection by customer archetype. Closes OQ-CHURN-03 (P2, §34.10) with three new DEC-030 events: `enterprise.mid_contract_termination_risk_flagged` (HIGH, 7yr — CHS < 20 sustained ≥ 4 weeks AND days remaining > 180; pg_cron automated emission; 30-day deduplication per tenant; HTTP 422 guard in emit-audit-event Worker if Worker enforcement not yet in place), `enterprise.contract_amended` (HIGH, 7yr — all signed contract amendments including price-lock renewals), `enterprise.early_termination_fee_waived` (HIGH, 7yr — `ev_analysis_completed: true` required for waivers > $5k or Worker returns 422). Contract amendment taxonomy: four types with DEC-030 event mapping. Seat reduction policy: contractually fixed for term; ≤ 10% deferred to annual renewal; > 30% or below tier minimum triggers ETF. Mid-contract termination loss: three components (remaining ARR, deferred revenue reversal, CAC destruction); loss table by tier and exit month. ETF structure: declining-balance rate schedule (60% at M1–M3 to 0%+floor at M36); calculated on contracted ACV; minimum floor of 1 month contracted ACV; five-tier approval authority matrix ($5k CSM lead / $25k founder / $100k founder+compliance-officer / >$100k founder+investor). EV waiver model (§35.6.3): two worked examples (Growth product-gap waiver: EV $11,136 waive > $6,107 enforce; Enterprise competitor-acquisition: EV $29,240 enforce > $3,182 waive). ASC 606 treatment for terminations, seat reductions, and price-lock renewals. 11-item implementation checklist (4× P0/M10, 4× P1/M12, 3× P2/M16). Three open questions (OQ-ETF-01 counsel review P0/M10; OQ-ETF-02 ACV basis P1/M10; OQ-ETF-03 seat suspension P2/M18). TOC updated. §34.10 OQ-CHURN-02 and OQ-CHURN-03 marked 🟢 Resolved. Cross-references: §16.4 (multi-year discount economics — ETF calculated on discounted contracted ACV), §18.1 (ASC 606 deferred revenue — reversal on early termination), §18.5 (multi-year deferred revenue waterfall), §23.6 (CPI indexation clause — price-lock defers application for 2yr term), §31.7 (rate lock commitment during contracted term), §32 (pricing exception approval — price-lock below list requires §32 approval chain), §34 (renewal risk register — this section extends the churn framework to mid-contract), `docs/MSA_TEMPLATE.md` (ETF clause + seat reduction policy — P0 M10 update required), `docs/AUDIT_LOG_SCHEMA.md` (three new DEC-030 event types to register — P0 before first amendment), `docs/OBSERVABILITY.md §12.6` (pg_cron job for mid-contract risk detection — P0 M10). Owner: enterprise-architect + customer-success + compliance-officer + founder.*

---

## 36. Enterprise Implementation Cost Deep-Dive: Engineering, CS & Legal Time Model

### 36.1 Purpose & Scope

**Gap this section fills:** §8.2 states one-time implementation costs of $1,150 (Starter), $1,500 (Growth), and $2,000 (Enterprise) as top-level estimates marked `[ESTIMATE]`. Open question OQ-08 (`§11`) asks for the true cost in engineering and CS hours per deal. This section provides a bottom-up derivation using hourly rates established in §26 (CSM cost model) and §27 (engineering cost model). The resulting figures revise §8.2 upward and create a trackable instrument for OQ-08 resolution after the first three enterprise deals close.

**What this section does NOT cover:**
- Ongoing CSM cost post-implementation (§26 owns that model)
- Infrastructure provisioning cost (§15 owns that; it is near-zero per §15.7)
- Sales and pre-sales cost (§8.5, §19.3–§19.5)
- Legal counsel fees for first-time contract template creation (§25.3 one-time spend; amortized here as per-deal cost after template exists)

**Privacy invariant:** All time-tracking is at `tenant_id` scope. No `user_id` is ever referenced in implementation tracking. Implementation work touches configuration, not individual health data.

**Hourly rates used throughout this section:**

| Role | Annual cost basis | Effective hourly rate | Source |
|---|---|---|---|
| Founding engineer | $115,000/yr total comp (§27.2.1) | **$62.50/hr** | ÷ 1,840 billable hours/yr |
| CSM (dedicated, Growth+) | $70,000/yr base + 16% taxes = $81,200 total | **$44.13/hr** | ÷ 1,840 hrs; §26.3 uses $37.50/hr (base-only rate); this section uses fully-loaded rate for margin accuracy |
| Compliance-officer / founder | Phase 0: $0 cash cost (founder time) | **$0 cash / $62.50 opportunity** | Opportunity cost modelled separately in §36.7 |
| External legal counsel | $250–$350/hr range (§25.3) | **$300/hr midpoint** | Per outside counsel engagement |

> **Note on CSM rate:** §26.8 uses $37.50/hr (base salary ÷ hours) for QBR economics. This section uses $44.13/hr (fully-loaded, including employer taxes and overhead) to produce the COGS-accurate implementation cost that should flow into §16.1 gross margin calculations. The two rates serve different purposes; always use the fully-loaded rate for margin analysis.

---

### 36.2 Implementation Activity Taxonomy

Implementation for each new enterprise customer consists of four work-streams executed over the Day 0–90 timeline from `docs/ENTERPRISE.md`:

| Work-stream | Primary owner | Secondary | Timing |
|---|---|---|---|
| **A. Identity integration** (SSO, SCIM, JIT provisioning) | Engineer | CSM (coordination) | Day 7–14 |
| **B. Tenant provisioning & configuration** (feature flags, RLS check, bulk import) | Engineer | CSM | Day 0–7 |
| **C. Branding & white-label** (custom domain, logo, colours; Growth+ only) | Engineer | CSM | Day 14–21 |
| **D. Training & admin onboarding** (admin dashboard, RBAC, CSM-led sessions) | CSM | Engineer (support) | Day 21–28 |
| **E. Pilot management** (weekly check-ins, adoption tracking, user comms kit) | CSM | — | Day 28–90 |
| **F. Compliance & legal** (MSA, DPA, ISQ, security questionnaire, DPIA if EU) | Founder / compliance-officer | Outside counsel | Day −30 to Day 0 |

Each work-stream has Starter / Growth / Enterprise variant hours. White-label (work-stream C) applies to Growth and Enterprise only. EU data residency verification adds ~1h to work-stream A for EU-HQ customers.

---

### 36.3 Engineering Hour Model by Tier

#### 36.3.1 Work-stream A: Identity Integration

| Activity | Starter | Growth | Enterprise | Notes |
|---|---|---|---|---|
| IdP metadata exchange + FORM app registration | 0.5 h | 0.5 h | 0.5 h | One-time per IdP; reuse playbook from `docs/SSO_SCIM_IMPLEMENTATION.md §4` |
| Test IdP-initiated SSO flow | 0.5 h | 0.5 h | 0.5 h | Standard regardless of tier |
| Test SP-initiated SSO flow | 0.5 h | 0.5 h | 0.5 h | Standard |
| JIT user provisioning verification (first login creates user) | 1.0 h | 1.0 h | 1.0 h | Verify `tenant_id` propagation and RLS row creation |
| SCIM basic user CRUD testing (create, update, deactivate) | 1.0 h | 1.0 h | 1.0 h | Standard across tiers |
| SCIM group sync testing (Growth+) | — | 2.0 h | 3.0 h | Growth: up to 10 groups; Enterprise: complex nested groups per §19 |
| SCIM group-to-role mapping verification (Growth+) | — | 1.0 h | 2.0 h | Verify `group_member_effective_role` view (higher-privilege-wins, DEC-039) |
| SSO/SCIM troubleshooting buffer | 1.0 h | 2.0 h | 3.0 h | Non-standard IdP config, customer IT delay, attribute mapping edge cases |
| EU data residency verification (EU-HQ tenants only) | 1.0 h | 1.0 h | 1.0 h | Confirm Cloudflare EU-Central routing; verify no US egress |
| **Work-stream A subtotal (non-EU)** | **4.5 h** | **8.5 h** | **11.5 h** | |
| **Work-stream A subtotal (EU customer)** | **5.5 h** | **9.5 h** | **12.5 h** | +1h data residency verification |

#### 36.3.2 Work-stream B: Tenant Provisioning & Configuration

| Activity | All tiers | Notes |
|---|---|---|
| Create tenant record + set feature flags for contracted tier | 0.5 h | `tenants` row + `tenant_feature_flags` population per DEC-040 |
| Bulk user import test (CSV, 50–1,000 users) | 1.0 h | Validate deduplication, error handling, RLS isolation |
| Cross-tenant isolation sanity check | 0.5 h | Run TC-RLS-001 through TC-RLS-003 from §25.5.1 against new tenant in staging before prod promotion |
| Admin dashboard role verification | 0.5 h | Confirm `tenant_owner`, `tenant_admin`, `tenant_manager` RBAC operates correctly |
| **Work-stream B subtotal** | **2.5 h** | Same for all tiers; scale of tenant doesn't affect config time |

#### 36.3.3 Work-stream C: Branding & White-Label (Growth+ only)

| Activity | Growth | Enterprise | Notes |
|---|---|---|---|
| Custom domain CNAME configuration + SSL verification | 0.5 h | 0.5 h | Cloudflare-managed; near-zero engineering time per §15.5 |
| Branding asset upload (logo SVG, favicon, primary colour token) | 0.5 h | 0.5 h | `tenant_config` row update |
| Accessibility contrast ratio check (WCAG AA floor) | 0.5 h | 0.5 h | Run automated check; flag if customer colour fails contrast |
| End-to-end branded URL smoke test | 0.5 h | 0.5 h | Verify CNAME resolves, logo renders, CSS override correct |
| **Work-stream C subtotal** | **2.0 h** | **2.0 h** | Not applicable to Starter |

#### 36.3.4 Work-stream A/B/C Engineering Subtotals

| Tier | Non-EU | EU customer |
|---|---|---|
| **Starter** | 4.5 + 2.5 = **7.0 h** | 5.5 + 2.5 = **8.0 h** |
| **Growth** | 8.5 + 2.5 + 2.0 = **13.0 h** | 9.5 + 2.5 + 2.0 = **14.0 h** |
| **Enterprise** | 11.5 + 2.5 + 2.0 = **16.0 h** | 12.5 + 2.5 + 2.0 = **17.0 h** |

#### 36.3.5 Engineering cost per tier

| Tier | Hours (non-EU) | Hours (EU) | Cost non-EU | Cost EU |
|---|---|---|---|---|
| Starter | 7.0 h | 8.0 h | **$437.50** | **$500.00** |
| Growth | 13.0 h | 14.0 h | **$812.50** | **$875.00** |
| Enterprise | 16.0 h | 17.0 h | **$1,000.00** | **$1,062.50** |

All figures at $62.50/hr (§27.2 founding engineer fully-loaded rate).

---

### 36.4 CS Hour Model by Tier

#### 36.4.1 Work-stream D: Training & Admin Onboarding

| Activity | Starter | Growth | Enterprise |
|---|---|---|---|
| Review MSA / order form to understand deal terms | 1.0 h | 1.0 h | 1.0 h |
| Slack Connect setup + internal account record | 0.5 h | 0.5 h | 0.5 h |
| Kickoff call preparation (agenda, deck) | 1.0 h | 1.5 h | 2.0 h |
| Lead kickoff call(s) | 1.0 h (1 call) | 1.5 h (1 call + async) | 3.0 h (2 calls) |
| Kickoff note-taking + follow-up | 0.5 h | 0.5 h | 0.5 h |
| IT coordination for SSO/SCIM (customer-side liaison) | 1.5 h | 2.5 h | 4.0 h |
| Branding input review + feedback loop (Growth+) | — | 1.0 h | 1.5 h |
| User communication kit (internal announcement email + FAQ) | 2.0 h | 3.0 h | 4.0 h |
| Admin dashboard training — session prep + delivery | 1.5 h (1 session) | 3.0 h (2 sessions) | 6.0 h (3 sessions + custom deck) |
| **Work-stream D subtotal** | **9.0 h** | **14.5 h** | **22.5 h** |

#### 36.4.2 Work-stream E: Pilot Management (Day 28–90)

| Activity | Starter | Growth | Enterprise |
|---|---|---|---|
| Weekly check-in calls | 3.0 h (2 × 30 min + prep) | 5.0 h (4 × 30 min + prep) | 8.0 h (4 × 60 min + prep) |
| Async adoption tracking + proactive outreach | 1.0 h | 2.0 h | 3.0 h |
| 30/60/90 day adoption report (prepare + present) | 2.0 h | 3.0 h | 5.0 h |
| **Work-stream E subtotal** | **6.0 h** | **10.0 h** | **16.0 h** |

#### 36.4.3 CS Hour and Cost Subtotals

| Tier | D + E total hours | Cost at $44.13/hr |
|---|---|---|
| Starter | 9.0 + 6.0 = **15.0 h** | **$661.95** |
| Growth | 14.5 + 10.0 = **24.5 h** | **$1,081.19** |
| Enterprise | 22.5 + 16.0 = **38.5 h** | **$1,699.01** |

> **Starter note:** Starter tier is email-only support (no named CSM). In the solo-founder phase, the founder performs CS duties. The $0 cash cost overstates the economics; the table above represents the true cost once a CS hire is in place, and the founder's time in the solo phase should be considered the same opportunity cost as the $44.13/hr rate for economic accuracy.

---

### 36.5 Legal & Compliance Hour Model

#### 36.5.1 Work-stream F: Pre-launch Legal & Compliance

**MSA execution cost** depends on whether a template already exists and how much the customer red-lines:

| Scenario | Internal time (founder/compliance-officer) | Counsel time | Counsel cost |
|---|---|---|---|
| **Template deal** (standard terms, minimal red-lining, post-Deal 1) | 2 h | 1 h | $300 |
| **First deal ever** (establishing template from scratch) | 4 h | 3 h | $900 |
| **Complex deal** (material MSA changes, EU data residency addendum, heavy redlines) | 8 h | 6 h | $1,800 |

**Default for modelling:** Template deal ($300 counsel, 2h internal). First-deal overhead is a one-time cost; see §36.8 for amortization.

**DPA execution:**

| Scenario | Internal time | Counsel time | Counsel cost |
|---|---|---|---|
| Standard DPA (FORM template, no modifications) | 1.0 h | 0 h | $0 |
| EU customer requiring DPIA supplement or SCC Annex review | 2.5 h | 1.0 h | $300 |

**Security / IT questionnaire response (ISQ):**

Most enterprise procurement processes include a vendor security questionnaire. FORM maintains a master response document (`docs/SECURITY_QUESTIONNAIRE.md`) that can be exported to PDF, but customer-specific customisation and follow-up are unavoidable:

| Tier | Hours to customise and submit | Notes |
|---|---|---|
| Starter | 3.0 h | Often a short checklist (20–40 questions); master doc covers ~85% |
| Growth | 5.0 h | Mid-depth ISQ (50–100 questions); requires admin dashboard and logging evidence |
| Enterprise | 9.0 h | Full enterprise ISQ (100–250 questions); network diagrams, RLS attestation, SOC 2 status, pen-test summary, GDPR DPIA, subprocessor list — each with follow-up round |

**Security review call (Growth and Enterprise):**

| Tier | Hours | Notes |
|---|---|---|
| Starter | 0 h | Typically not required |
| Growth | 1.0 h | 1-hour CISO or IT security call |
| Enterprise | 2.0 h | Formal security review call + 1h follow-up questions async |

#### 36.5.2 Work-stream F cost subtotals (template deal, non-EU)

| Tier | Internal hours (founder/CO) | Counsel cost | Total internal opp. cost | Total cash (counsel) |
|---|---|---|---|---|
| Starter | 2 + 1 + 3 + 0 = **6.0 h** | **$300** | $375 (at $62.50/h) | **$300** |
| Growth | 2 + 1 + 5 + 1 = **9.0 h** | **$300** | $563 (at $62.50/h) | **$300** |
| Enterprise | 2 + 1 + 9 + 2 = **14.0 h** | **$300** | $875 (at $62.50/h) | **$300** |

For EU customers: add 1.5h internal + $300 counsel for DPIA supplement review.

---

### 36.6 Bottom-Up Cost Table & §8.2 Variance Analysis

#### 36.6.1 Fully-loaded implementation cost (template deal, non-EU)

| Cost component | Starter | Growth | Enterprise |
|---|---|---|---|
| Engineering (§36.3) | $437.50 | $812.50 | $1,000.00 |
| CS (§36.4, fully-loaded $44.13/hr) | $661.95 | $1,081.19 | $1,699.01 |
| Internal compliance / founder time (cash = $0; opp. cost $62.50/hr) | $0 cash | $0 cash | $0 cash |
| External counsel (§36.5.2) | $300.00 | $300.00 | $300.00 |
| **Total cash implementation cost** | **$1,399.45** | **$2,193.69** | **$2,999.01** |
| + Internal opp. cost (compliance/founder 6h / 9h / 14h) | $375.00 | $562.50 | $875.00 |
| **Total fully-loaded (incl. opp. cost)** | **$1,774.45** | **$2,756.19** | **$3,874.01** |

EU customer premium (add to above):

| | Starter | Growth | Enterprise |
|---|---|---|---|
| EU data residency verification (engineering) | +$62.50 | +$62.50 | +$62.50 |
| EU DPIA supplement (counsel) | +$300 | +$300 | +$300 |
| EU DPIA supplement (internal time) | +$94 opp. | +$94 opp. | +$94 opp. |
| **EU premium total (cash)** | **+$362.50** | **+$362.50** | **+$362.50** |

#### 36.6.2 Comparison with §8.2 estimates

| Tier | §8.2 estimate (midpoint) | §36 bottom-up (cash, non-EU) | Variance | Primary driver |
|---|---|---|---|---|
| Starter | $1,150 | **$1,399** | **+$249 (+21.7%)** | ISQ response time (3h) and fully-loaded CS rate ($44.13 vs. implicit lower) |
| Growth | $1,500 | **$2,194** | **+$694 (+46.3%)** | ISQ time (5h), SCIM group testing (3h), white-label config (2h), and CS pilot check-ins |
| Enterprise | $2,000 | **$2,999** | **+$999 (+50.0%)** | ISQ time (9h), complex SCIM role mapping (5h), security review calls (2h), and extended pilot management (38.5h CS) |

**Key finding:** §8.2 underestimates implementation cost by 22–50% across tiers. The largest systematic gap is the Security / IT Questionnaire response time, which §8.2 folds into the "Custom SLA / compliance review" line at $300–$500, understating the true cost by $188–$563 depending on tier.

**Implication for gross margin:** The §16.1 break-even table uses §8.2 estimates. The corrected implementation cost reduces Year 1 gross margin on the first deal by 1.7–5.0pp depending on tier and seat count. See §36.7 for the updated GM impact table.

#### 36.6.3 §8.2 update mandate

The implementation cost figures in §8.2 should be updated after the first 3 enterprise deals are time-tracked per §36.9. Until then, the §36 bottom-up figures are the planning model. The discrepancy is not large enough to threaten viability at any tier (§16.2 minimum viable seat floor of 50 seats remains valid), but it is material enough to affect investor-facing gross margin projections.

---

### 36.7 Implementation Cost as % ACV — Year 1 GM Impact

#### 36.7.1 Implementation cost as % of 1-year ACV by deal size

| Tier | Seats | Annual ACV | §8.2 impl. | §36 impl. (cash) | §8.2 impl/ACV | §36 impl/ACV |
|---|---|---|---|---|---|---|
| Starter | 50 | $7,200 | $1,150 | $1,399 | 16.0% | 19.4% |
| Starter | 100 | $14,400 | $1,150 | $1,399 | 8.0% | 9.7% |
| Starter | 200 | $28,800 | $1,150 | $1,399 | 4.0% | 4.9% |
| Growth | 200 | $21,600 | $1,500 | $2,194 | 6.9% | 10.2% |
| Growth | 500 | $54,000 | $1,500 | $2,194 | 2.8% | 4.1% |
| Growth | 1,000 | $108,000 | $1,500 | $2,194 | 1.4% | 2.0% |
| Enterprise | 1,000 | $72,000 | $2,000 | $2,999 | 2.8% | 4.2% |
| Enterprise | 2,000 | $144,000 | $2,000 | $2,999 | 1.4% | 2.1% |
| Enterprise | 5,000 | $360,000 | $2,000 | $2,999 | 0.6% | 0.8% |

**Pattern:** Implementation cost % ACV falls steeply with seat count. Starter economics at 50 seats are the most exposed (19.4% of ACV); Enterprise at 5,000 seats is negligible (0.8%). This reinforces the 50-seat Starter minimum deal size from §16.2.

#### 36.7.2 Year 1 gross margin impact (updated vs. §16.1)

§16.1 computes Year 1 GM using §8.2 implementation cost. Substituting §36 bottom-up cost:

| Tier | Seats | §16.1 Year 1 GM (§8.2 impl.) | §36 Year 1 GM (§36 impl.) | Delta |
|---|---|---|---|---|
| Starter | 50 | 43.4% | **39.9%** | −3.5 pp |
| Starter | 100 | 65.0% | **63.3%** | −1.7 pp |
| Starter | 200 | 72.6% | **71.8%** | −0.8 pp |
| Growth | 200 | 74.9% | **71.6%** | −3.3 pp |
| Growth | 500 | 82.5% | **81.3%** | −1.2 pp |
| Enterprise | 1,000 | 85.2% | **83.8%** | −1.4 pp |

**50-seat Starter floor validity:** §31.3.3 confirmed the 50-seat Starter floor at a Year 1 GM of 73.5% using §8.2's implementation cost. With §36's updated implementation cost ($1,399 vs. $1,150), the 50-seat Year 1 GM falls to 39.9% (using the full §16.1 framework including ongoing CSM cost). This is below the 70% threshold in §31.3.3. However, that threshold referenced infrastructure COGS only; the full-cost GM including CSM was always lower. The 50-seat minimum remains defensible for the following reasons:
1. Year 2+ GM at 50 seats (no implementation cost) rises to ~80.3% — payback is fast
2. 50-seat Starter is explicitly a loss-leader to land the account; expansion to 100+ seats within 12 months is the economic thesis (§8.7)
3. The $1,399 implementation cost is fully recovered in 2.3 months of gross profit at 50 seats ($1,399 ÷ $605/month net margin)

The correct threshold for declining a deal is not Year 1 GM but deal lifetime value: a 50-seat Starter with 120% NRR over 3 years produces ~$25,920 gross profit vs. ~$1,399 implementation cost — a 17.5× implementation cost recovery.

---

### 36.8 Multi-Deal Scale Economics

#### 36.8.1 Fixed vs. variable implementation costs

Not all implementation cost scales with each new deal. Some activities are template-driven and decrease in time as FORM's playbook matures:

| Activity | Deal 1 (first ever) | Deal 2–5 | Deal 6+ (playbook mature) |
|---|---|---|---|
| MSA negotiation (engineer) | 0 (legal only) | 0 | 0 |
| MSA negotiation (internal) | 4 h (template creation) | 2 h (standard) | 1.5 h (review only) |
| MSA counsel cost | $900 (3h) | $300 (1h) | $300 (1h) |
| ISQ response | 5–9 h (build master response) | 3–7 h (customise master) | 2–5 h (minor updates) |
| SSO setup | 4.5–11.5 h (learn per-IdP quirks) | 4.0–10.0 h | 3.5–9.5 h (IdP-specific playbooks exist) |
| Admin training | Full per §36.4 | Full (but deck reused) | Full (deck reused; prep −0.5h) |
| Pilot management | Full per §36.4 | Full | Full |

**Conclusion:** Implementation fixed costs (MSA template creation, ISQ master document construction) are front-loaded in Deals 1–3. By Deal 6, implementation cost decreases by approximately 15–20% from the §36 baseline.

#### 36.8.2 Founding engineer hire timing effect

In the solo-founder phase, the founder performs all engineering work. At the §27.2 opportunity cost rate of $62.50/hr, engineering cost is the same as post-hire. However, the founding engineer hire (§27.3, Month 13 Base scenario) frees the founder from implementation work, which is critical as the enterprise pipeline scales. With 3 enterprise deals in Year 1, implementation engineering load is:
- Starter: 7h × 3 = 21h
- Mix (1 Starter, 1 Growth, 1 Enterprise): 7 + 13 + 16 = 36h

36 engineering hours = ~4.5% of a founding engineer's annual capacity (36h ÷ 800h expected first 6 months of tenure). Implementation is not the founding engineer's primary workload; it is a marginal addition to platform development. The engineering time model above is a ceiling, not a mandate.

#### 36.8.3 Non-standard IdP premium

The §36.3 model assumes Okta, Azure AD, Google Workspace, or OneLogin — the four IdPs for which FORM maintains configuration playbooks (`docs/SSO_CLIENT_CONFIG.md`). Non-standard IdPs (AD FS, PingFederate, JumpCloud, custom SAML providers) carry an additional engineering time premium:

| IdP category | Additional hours | Cost premium |
|---|---|---|
| Supported IdP (Okta / Azure AD / GWS / OneLogin) | 0 h | $0 |
| Partially-supported IdP (JumpCloud, OneLogin custom) | +2–3 h | +$125–188 |
| Non-standard SAML 2.0 (AD FS, PingFederate) | +4–6 h | +$250–375 |
| Non-standard OIDC (custom provider) | +3–5 h | +$188–313 |
| Legacy IdP requiring custom SAML assertion mapping | +8–12 h | +$500–750 |

Non-standard IdP work should be flagged in the sales discovery process. Legacy IdPs at a >$500 premium may warrant a one-time "technical integration fee" at Growth/Enterprise tier, to be negotiated by the founder before contract signature. Document any such fee in `docs/DECISION_LOG.md` as a custom commercials decision.

---

### 36.9 Time-Tracking Instrument for OQ-08 Resolution

OQ-08 (`§11`) asks for the true implementation cost per enterprise deal, to be resolved by time-tracking the first 3 enterprise deals. This subsection specifies the tracking instrument.

#### 36.9.1 Schema: `enterprise_impl_time_log`

```sql
CREATE TABLE enterprise_impl_time_log (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id       UUID NOT NULL REFERENCES tenants(id) ON DELETE RESTRICT,
  deal_sequence   INTEGER NOT NULL,          -- 1 = first ever, 2 = second, etc.
  role            TEXT NOT NULL CHECK (role IN (
                    'founder', 'engineer', 'csm', 'compliance_officer', 'counsel'
                  )),
  activity_code   TEXT NOT NULL CHECK (activity_code IN (
                    'sso_scim_setup',
                    'tenant_provisioning',
                    'white_label_config',
                    'admin_training',
                    'pilot_management',
                    'adoption_report',
                    'msa_negotiation',
                    'dpa_execution',
                    'isq_response',
                    'security_review_call',
                    'eu_data_residency_check',
                    'other'
                  )),
  hours_spent     NUMERIC(5,2) NOT NULL CHECK (hours_spent > 0 AND hours_spent <= 40),
  hourly_rate_usd NUMERIC(8,2) NOT NULL,     -- actual fully-loaded rate at time of deal
  notes           TEXT,
  logged_at       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  logged_by       UUID NOT NULL              -- actor UUID (founder or CSM user_id in admin system)
);

-- Indexes
CREATE INDEX ON enterprise_impl_time_log (tenant_id);
CREATE INDEX ON enterprise_impl_time_log (deal_sequence, activity_code);

-- RLS: only compliance_reviewer and form_admin may read; only form_system may write
ALTER TABLE enterprise_impl_time_log ENABLE ROW LEVEL SECURITY;
CREATE POLICY impl_log_read ON enterprise_impl_time_log
  FOR SELECT USING (auth.role() IN ('compliance_reviewer', 'form_admin'));
CREATE POLICY impl_log_write ON enterprise_impl_time_log
  FOR INSERT WITH CHECK (auth.role() = 'form_system');
```

#### 36.9.2 Cost extraction query

After the first 3 deals are logged, run this query to produce the OQ-08 actuals:

```sql
SELECT
  deal_sequence,
  role,
  activity_code,
  SUM(hours_spent)                                    AS total_hours,
  SUM(hours_spent * hourly_rate_usd)                  AS total_cost_usd,
  ROUND(AVG(hours_spent * hourly_rate_usd), 2)        AS avg_cost_usd
FROM enterprise_impl_time_log
GROUP BY deal_sequence, role, activity_code
ORDER BY deal_sequence, role, activity_code;
```

Compare `SUM(total_cost_usd)` by `deal_sequence` to the §36.6.1 bottom-up estimates. If actuals are within ±20%, §8.2 can be updated using §36.6 figures. If actuals deviate by > 20%, recalibrate §36 assumptions.

#### 36.9.3 OQ-08 closure protocol

OQ-08 is formally closed when:
1. `enterprise_impl_time_log` contains at least 3 rows for each `deal_sequence` value 1, 2, 3
2. The variance between actuals and §36.6.1 estimates is computed and documented
3. §8.2 is updated with the actual midpoint from the first 3 deals
4. A DEC-030 event `enterprise.implementation_cost_model_calibrated` is emitted (§36.10.3)
5. A `docs/DECISION_LOG.md` entry records the calibration outcome

---

### 36.10 DEC-030 HMAC-Chained Audit Events

Three new DEC-030 events for implementation milestones. All events use `tenant_id` scope; no `user_id` is included (privacy invariant: implementation tracking is at account level, not individual level).

#### 36.10.1 Event: `enterprise.implementation_kickoff_completed`

| Field | Value |
|---|---|
| **Event type** | `enterprise.implementation_kickoff_completed` |
| **Severity** | STANDARD |
| **Retention** | 7 years |
| **Actor** | CSM (or founder in solo phase) |
| **Trigger** | Kickoff call(s) complete; implementation project plan confirmed with customer |

**Payload (Zod v2 schema):**
```typescript
z.object({
  tenant_id:               z.string().uuid(),
  deal_sequence:           z.number().int().positive(),
  contracted_tier:         z.enum(['starter', 'growth', 'enterprise']),
  contracted_seats:        z.number().int().positive(),
  idp_type:                z.enum([
    'okta', 'azure_ad', 'google_workspace', 'onelogin',
    'adfs', 'pingfederate', 'jumpcloud', 'other_saml', 'other_oidc'
  ]),
  white_label_enabled:     z.boolean(),
  eu_data_residency:       z.boolean(),
  kickoff_date:            z.string().date(),                       // YYYY-MM-DD
  target_go_live_date:     z.string().date(),
  csm_actor_id:            z.string().uuid(),
})
```

#### 36.10.2 Event: `enterprise.sso_scim_setup_verified`

| Field | Value |
|---|---|
| **Event type** | `enterprise.sso_scim_setup_verified` |
| **Severity** | STANDARD |
| **Retention** | 7 years |
| **Actor** | Engineer (or founder) |
| **Trigger** | SSO IdP-initiated and SP-initiated flows verified; SCIM provisioning smoke-tested |

**Payload (Zod v2 schema):**
```typescript
z.object({
  tenant_id:               z.string().uuid(),
  idp_type:                z.enum([
    'okta', 'azure_ad', 'google_workspace', 'onelogin',
    'adfs', 'pingfederate', 'jumpcloud', 'other_saml', 'other_oidc'
  ]),
  sso_modes_verified:      z.object({
    idp_initiated:         z.boolean(),
    sp_initiated:          z.boolean(),
  }),
  scim_features_enabled:   z.object({
    user_crud:             z.boolean(),
    group_sync:            z.boolean(),
    role_mapping:          z.boolean(),
    jit_provisioning:      z.boolean(),
  }),
  eu_data_residency_confirmed: z.boolean(),
  engineer_actor_id:       z.string().uuid(),
  verification_date:       z.string().date(),
})
```

#### 36.10.3 Event: `enterprise.implementation_cost_model_calibrated`

| Field | Value |
|---|---|
| **Event type** | `enterprise.implementation_cost_model_calibrated` |
| **Severity** | STANDARD |
| **Retention** | 7 years |
| **Actor** | Founder (data-engineer or compliance-officer post-Series A) |
| **Trigger** | First 3 deals time-tracked; §8.2 updated; OQ-08 closed |

**Payload (Zod v2 schema):**
```typescript
z.object({
  deals_analysed:          z.number().int().min(3),
  avg_impl_cost_starter:   z.number().optional(),    // actual average if starter deals exist
  avg_impl_cost_growth:    z.number().optional(),
  avg_impl_cost_enterprise: z.number().optional(),
  variance_vs_model_pct:   z.number(),               // signed %; positive = actuals exceed model
  model_version_updated:   z.string(),               // e.g. "COST_MODEL.md §8.2 v2.3"
  decision_log_ref:        z.string(),               // e.g. "DEC-XXX"
  calibration_date:        z.string().date(),
})
```

#### 36.10.4 DEC-030 Event Summary for §36

| Event | Severity | Retention | Trigger | Privacy constraint |
|---|---|---|---|---|
| `enterprise.implementation_kickoff_completed` | STANDARD | 7 yr | Kickoff complete | `tenant_id` only; no user IDs |
| `enterprise.sso_scim_setup_verified` | STANDARD | 7 yr | SSO/SCIM smoke test passes | `tenant_id` + `idp_type`; no individual user data |
| `enterprise.implementation_cost_model_calibrated` | STANDARD | 7 yr | OQ-08 closure | Aggregate cost data only; no tenant names in payload |

All three events must be registered in `docs/AUDIT_LOG_SCHEMA.md §6` before the first enterprise deal closes.

---

### 36.11 Implementation Checklist

#### P0 — Before first enterprise deal closes (M8–M10)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Register `enterprise.implementation_kickoff_completed`, `enterprise.sso_scim_setup_verified`, and `enterprise.implementation_cost_model_calibrated` in `docs/AUDIT_LOG_SCHEMA.md §6` with Zod schemas from §36.10. Deploy to `emit-audit-event` Worker. | platform-engineer | **P0** | M8 | ✅ Documentation registered in AUDIT_LOG_SCHEMA.md v1.8 (2026-06-12). `emit-audit-event` Worker deployment remains an engineering implementation task. |
| 2 | Create `enterprise_impl_time_log` table (§36.9.1 DDL) with RLS policies. Register migration `0060_enterprise_impl_time_log.sql`. | platform-engineer | **P0** | M8 | [ ] |
| 3 | Build and test the cost extraction query (§36.9.2) in Supabase Studio. Verify it produces per-role, per-activity totals. | data-engineer | **P0** | M9 | [ ] |
| 4 | Begin time-logging all implementation activities from Deal 1, using `activity_code` values from §36.9.1. Ensure every session of engineering or CSM work is logged within 24h. | founder (solo phase) | **P0** | Deal 1 | [ ] |

#### P1 — Before third enterprise deal closes (M12–M14)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 5 | After Deal 3 closes, run extraction query. Compute variance vs. §36.6.1 model. If variance > 20%, update §8.2 implementation cost estimates and document in `docs/DECISION_LOG.md` (DEC-XXX). | data-engineer + founder | **P1** | M14 | [ ] |
| 6 | Emit `enterprise.implementation_cost_model_calibrated` DEC-030 event after §8.2 update, whether or not the model was recalibrated. | compliance-officer | **P1** | M14 | [ ] |
| 7 | Build per-IdP time tracking to identify which IdP types consistently exceed the §36.3 time model by > 2h (non-standard IdP premium, §36.8.3). Document findings in `docs/SSO_CLIENT_CONFIG.md`. | engineer | **P1** | M15 | [ ] |

#### P2 — Before enterprise GA (M13)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 8 | Evaluate whether a one-time "technical integration fee" for legacy/non-standard IdPs should be added to the standard pricing schedule. Discuss with founder; document decision in `docs/DECISION_LOG.md`. | customer-success + founder | **P2** | M13 | [ ] |
| 9 | Update §16.1 break-even table with §36.6.2 corrected implementation figures (replacing §8.2 midpoints). | data-engineer | **P2** | M14 | [ ] |

---

### 36.12 Open Questions

| ID | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-IMPL-01** | **What is the true, time-tracked implementation cost per deal across first 3 enterprise deals?** This is the direct instrument for closing OQ-08 (§11). The §36 model estimates $1,399 / $2,194 / $2,999 for Starter / Growth / Enterprise (non-EU, template deal). Actuals may be higher if ISQ response takes longer than estimated (enterprise ISQ often runs 250+ questions with follow-up rounds). | **P1** | data-engineer + founder | After Deal 3 closes; per §36.9.3 OQ-08 closure protocol |
| **OQ-IMPL-02** | **Does non-standard IdP implementation time match the §36.8.3 premium table?** AD FS and PingFederate carry the largest time premium (+8–12h, +$500–$750). Until FORM has completed at least one AD FS and one PingFederate implementation, the premium is an estimate. If the first AD FS deal takes > 16h engineering, the non-standard IdP premium for the pricing schedule (§36.8.3) should be formalised as a line-item fee in the order form. | **P2** | engineer + customer-success | On first non-standard IdP deal; no later than M18 |
| **OQ-IMPL-03** | **Does CSM implementation time decrease significantly with playbook maturity?** §36.8.1 estimates a 15–20% decrease by Deal 6. If the CSM time model does not decrease (e.g., because enterprise customers have structurally longer training sessions regardless of playbook), the §36.4 figures represent steady-state cost, not just Deal 1–3 cost. Calibrate by logging CSM time per activity across first 6 deals. | **P2** | customer-success + data-engineer | After Deal 6; no later than M20 |

---

## 37. Enterprise Pipeline Health & ARR Forecasting Model

### 37.1 Purpose & Scope

This section provides a quantitative framework for tracking enterprise pipeline health, projecting ARR from pipeline state, and governing the weekly forecast cadence. It is the forward-looking companion to §19 (Go-to-Market Financial Model), §23 (NRR Engine), and §34 (Churn Intervention Model).

**What this section adds that §19 does not:**
- Formal pipeline *stage* definitions with entry/exit criteria (not just conversion rate inputs)
- Pipeline coverage ratio thresholds (how many $ of pipeline FORM needs in each stage per $ of ARR target)
- Pipeline aging policy (how long a deal may sit in a stage before triggering a CSM or founder action)
- A weekly ARR bridge reconciling pipeline movements to recognised revenue
- Governance record for forecast overrides (DEC-030 HMAC chain — investor-grade audit trail)

**Scope:** Enterprise pipeline only. Consumer subscription revenue is forecast via the cohort model in §14 and the cash flow model in §22. This section does not cover consumer MRR.

**Not in scope:** CRM tooling selection (pre-Series A FORM runs on a spreadsheet; CRM selection is OQ-PIPE-02 below). Quota assignment to AEs (not relevant until first AE hire; see §19.4).

**Owners:** founder (pre-Series A), customer-success + data-engineer (post-Series A).

**Cadence:** Weekly pipeline review (Monday); monthly ARR bridge close (last business day of month); quarterly forecast vs. actuals retrospective.

---

### 37.2 Pipeline Stage Definitions

FORM uses a six-stage enterprise pipeline. Each stage has a single entry criterion, a single exit criterion, and a maximum age (calendar days) before escalation.

| # | Stage | Entry criterion | Exit criterion | Max age (days) | Owner |
|---|---|---|---|---|---|
| **S0 · Inbound** | Qualified lead received (email, referral, inbound form) | Qualification call booked | 14 | founder |
| **S1 · Qualified** | Qualification call complete; ≥ 50-seat count confirmed; budget signal received | Pilot agreement signed | 30 | founder |
| **S2 · Pilot** | Pilot agreement signed; DPA executed; at least 10 employees invited | Pilot success criteria assessed | 90 | founder + CSM |
| **S3 · Proposal** | Pilot success criteria met (≥ 40% D7 activation, ≥ 50 NPS opt-in); MSA and order form sent | Contract signed | 60 | founder |
| **S4 · Legal review** | Customer's legal team has the MSA for review | Contract signed | 45 | founder |
| **S5 · Closed-Won** | Contract signed; ACV recognised; first invoice issued | — (deal lifecycle continues in §34) | — | CSM |

**Stage overlap:** S3 (Proposal) and S4 (Legal review) may run in parallel if the customer's procurement and legal teams work simultaneously. In this case, the deal is logged at S4 (higher milestone) once legal review opens.

**Pilot success criteria** are agreed upfront in writing (per enterprise.html CTA language). The default criteria are:
- D7 activation ≥ 40% of invited employees (DEC-034)
- Opt-in NPS (among activated) ≥ 50
- Zero P0/P1 incidents during pilot window

If any criterion is not met, FORM does not issue an invoice. The deal re-enters S2 (Extended Pilot, per §21) or closes.

---

### 37.3 Stage Conversion Rate Assumptions

Conversion rates below are FORM's pre-revenue benchmark assumptions, grounded in comparable B2B SaaS wellness and productivity platforms (OpenView Partners SaaS Benchmarks 2024; Gartner corporate wellness vendor close rate analysis 2023). They will be calibrated against actual outcomes after the first 10 deals close (OQ-PIPE-01).

| Transition | Benchmark | FORM assumption (conservative) | FORM assumption (base) | FORM assumption (optimistic) |
|---|---|---|---|---|
| S0 → S1 (qual rate) | 20–35% | 20% | 28% | 35% |
| S1 → S2 (pilot start rate) | 45–65% | 45% | 55% | 65% |
| S2 → S3 (pilot success rate) | 55–75% | 60% | 68% | 75% |
| S3/S4 → S5 (close rate from proposal) | 65–85% | 65% | 75% | 85% |
| **End-to-end (S0 → S5)** | **10–20%** | **~7.5%** | **~12%** | **~18%** |

**Why FORM uses conservative-to-base rates at launch:**
- First deals are founder-led with no established playbook or reference customers → higher early drop-off
- Pilot success criterion (D7 ≥ 40%) is tighter than most wellness vendors use → lower S2→S3 rate initially
- Legal review (S4) is expected to take longer than average for GDPR Art. 9 DPA negotiation in EU deals

**Rate recalibration trigger:** After 10 closed-won deals, run the query in §37.8.3 to compute actual stage conversion rates. If the S2→S3 rate consistently exceeds 72%, the base assumption can be raised; if the S0→S1 rate is below 18%, qualification criteria may need tightening.

---

### 37.4 Pipeline Health Metrics

Three pipeline health metrics are tracked weekly. Each has a WARNING and CRITICAL threshold that trigger specific actions.

#### 37.4.1 Pipeline Coverage Ratio (PCR)

**Definition:** Total weighted pipeline ACV ÷ Remaining quarterly ARR target.

Weighted pipeline ACV uses stage weights:

| Stage | Weight |
|---|---|
| S0 · Inbound | 5% |
| S1 · Qualified | 15% |
| S2 · Pilot | 40% |
| S3 · Proposal | 65% |
| S4 · Legal review | 80% |
| S5 · Closed-Won | 100% (recognised) |

**Thresholds:**

| Status | PCR | Action |
|---|---|---|
| Healthy | ≥ 3.0× | No action — hold current outbound cadence |
| Warning | 2.0× – 2.9× | Increase outbound; founder reviews every deal in S0 and S1; evaluate extending pilot to recover S2 candidates |
| Critical | < 2.0× | Founder escalation; consider quarterly target revision; evaluate partner-sourced or re-engagement outbound |

**Rationale for 3.0× floor:** Enterprise sales carry binary outcomes (each deal is large; a single late-stage loss can miss 25–50% of a quarterly target). A 3.0× PCR provides a buffer for the expected 35–40% of S3/S4 deals that slip into the next quarter due to legal or procurement delays — consistent with OpenView median SaaS PCR guidance of 3×–4× for founder-led teams.

#### 37.4.2 Sales Velocity

**Formula** (from §19.1):
```
Sales Velocity = (# Qualified Deals × Average ACV × Win Rate) ÷ Sales Cycle Length (months)
```

**FORM monthly velocity at base case (Year 1):**
- 3 qualified deals/month (solo founder outbound + inbound)
- Average ACV: $32,400 (blended across tier mix from §19.7)
- Win rate (S0→S5): 12%
- Sales cycle: 6 months
- Monthly velocity = (3 × $32,400 × 0.12) ÷ 6 = **$1,944 / month ARR**

**FORM monthly velocity at Year 2 (first AE hired):**
- 8 qualified deals/month
- Average ACV: $42,000 (growing mix toward Growth/Enterprise)
- Win rate: 15% (playbook maturing)
- Sales cycle: 5.5 months
- Monthly velocity = (8 × $42,000 × 0.15) ÷ 5.5 = **$9,164 / month ARR**

**Velocity tracking query:** see §37.8.3.

#### 37.4.3 Pipeline Aging

Any deal that exceeds its stage maximum age (§37.2) triggers the following action:

| Stage exceeded | Age > | Action |
|---|---|---|
| S0 | 14 days | Founder calls the prospect; no-response → mark S0-Stale; remove from weighted pipeline |
| S1 | 30 days | Founder reviews qualification; either books pilot agreement call within 7 days or downgrades to S0-Stale |
| S2 | 90 days | Founder + CSM reviews pilot metrics; if D7 activation < 20% at day 60, trigger mid-pilot intervention; if < 20% at day 90, close or move to Extended Pilot |
| S3 | 60 days | Founder re-engages economic buyer; consider discount or extended payment terms |
| S4 | 45 days | Founder contacts customer's legal team directly; if no movement after contact, flag as DEAL_AT_RISK |

**Aging DEC-030 events** are emitted at each threshold crossing (§37.9.3).

---

### 37.5 ARR Build Table (Y1–Y3)

The table below is the **base case** from §19.7, re-stated here with the pipeline-to-ARR mechanics made explicit. Figures are cumulative ARR at end of each quarter (not incremental).

Assumptions: 12-month weighted conversion rates from §37.3 (base); churn rates from §34 (Y1 < 30%, Y2 < 15%, Y3+ < 8%); NRR from §23 (110% Year 1, 120% Year 2+).

| Quarter | New Logos | Avg ACV | New ARR | Churned ARR | Expansion ARR | Cumulative ARR |
|---|---|---|---|---|---|---|
| Q1 Y1 | 0 | — | $0 | $0 | $0 | $0 |
| Q2 Y1 | 1 | $28,800 | $28,800 | $0 | $0 | $28,800 |
| Q3 Y1 | 2 | $32,400 | $64,800 | $0 | $0 | $93,600 |
| Q4 Y1 | 2 | $32,400 | $64,800 | −$8,640 (1 × Y1 churn) | $2,880 | $152,640 |
| Q1 Y2 | 3 | $36,000 | $108,000 | −$9,180 | $6,096 | $257,556 |
| Q2 Y2 | 4 | $39,600 | $158,400 | −$9,180 | $10,302 | $417,078 |
| Q3 Y2 | 4 | $39,600 | $158,400 | −$12,753 | $16,683 | $579,408 |
| Q4 Y2 | 5 | $42,000 | $210,000 | −$12,753 | $23,176 | $799,831 |
| Q1 Y3 | 5 | $45,000 | $225,000 | −$11,997 | $31,993 | $1,044,827 |
| Q2 Y3 | 6 | $45,000 | $270,000 | −$11,997 | $41,793 | $1,344,623 |
| Q3 Y3 | 6 | $48,000 | $288,000 | −$15,596 | $53,785 | $1,670,812 |
| Q4 Y3 | 7 | $48,000 | $336,000 | −$15,596 | $66,832 | $2,058,048 |

**Notes:**
- Q1 Y1: Sales cycle is 6 months; first pilot signed in M1 cannot close before M7 → zero closed-won in Q1.
- Churn: applied at 25% annual rate Y1 (conservative end of Y1 < 30% band); 13% Y2; 8% Y3.
- Expansion: seat upsell + tier upgrades driving 10% expansion ARR on retained base Year 1, 15% Year 2+. Detailed mechanics in §23.
- Q4 Y3 cumulative ARR of ~$2.1M represents the Series A fundraising milestone (see §22.6 and §24.3).

**Downside scenario** (40% miss on new logos each quarter, S5: S3: no improvement in churn):
- Q4 Y2 cumulative ARR: ~$490k (vs. $800k base)
- Q4 Y3 cumulative ARR: ~$1.3M (vs. $2.1M base)
- Series A timing shifts to Q2 Y4 at earliest.

---

### 37.6 NRR Decomposition & ARR Bridge

The monthly ARR bridge reconciles Opening ARR → Closing ARR across five components. This bridges §23 (NRR engine) with the pipeline model above.

#### 37.6.1 ARR Bridge Components

| Component | Definition | Source |
|---|---|---|
| **New ARR** | ACV of contracts closed in the month | §37.5 new logo additions |
| **Expansion ARR** | Seat additions and tier upgrades on existing contracts | §23.2 (tier migration economics) |
| **Contraction ARR** | Seat reductions on existing contracts | §35.3 (seat reduction policy) |
| **Churned ARR** | ACV of contracts not renewed | §34.3 (financial impact of churn) |
| **Price indexation ARR** | Annual CPI-linked price adjustment on renewal cohort | §23.6 (indexation clause) |

**Monthly ARR bridge formula:**
```
Closing ARR = Opening ARR
            + New ARR
            + Expansion ARR
            − Contraction ARR
            − Churned ARR
            + Price Indexation ARR
```

#### 37.6.2 Bridge Reconciliation Cadence

The ARR bridge is reconciled on the last business day of each month by the founder (pre-Series A) or data-engineer (post-Series A):

1. Pull all `enterprise_contracts` rows where `activated_at` falls within the month → New ARR
2. Pull all `enterprise_contracts` rows where `seats_current` changed within the month → Expansion / Contraction ARR
3. Pull all `enterprise_contracts` rows where `lifecycle_status` changed to `churned` within the month → Churned ARR
4. Pull price indexation ARR from the renewal pipeline (contracts with `renewal_date` in the month that include CPI clause)
5. Sum → compute NRR for the month (Closing ARR − New ARR) ÷ Opening ARR × 12 (annualised)

**Minimum NRR thresholds** (from §23 and §14.6):
- Year 1: 100% (churn and expansion expected to roughly offset)
- Year 2: 110% (expansion begins outpacing churn as base matures)
- Year 3+: 120% (Series A readiness criterion per §24.3)

If monthly NRR drops below the annual target for two consecutive months, a DEC-030 `enterprise.nrr_slo_breach` event is emitted and the founder reviews the renewal risk register (§34) within 5 business days.

---

### 37.7 Forecasting Cadence & Governance

#### 37.7.1 Weekly Pipeline Review (Monday)

**Participants:** founder (pre-Series A); founder + VP Sales + CSM (post-Series A).

**Agenda (30 minutes):**
1. PCR calculation for the current quarter (§37.4.1)
2. New deals added to pipeline this week (stage, ACV, source)
3. Deals that changed stage this week
4. Deals that crossed an aging threshold this week (§37.4.3)
5. Action items from last week's review: closed?

**Output:** Updated pipeline spreadsheet (pre-CRM) or CRM dashboard. One `enterprise.pipeline_reviewed` DEC-030 event per review (§37.9.1).

#### 37.7.2 Monthly ARR Close (last business day)

ARR bridge reconciliation per §37.6.2. Output: signed-off ARR figure used in investor updates and board reporting. `enterprise.arr_bridge_closed` DEC-030 event emitted (§37.9.2).

#### 37.7.3 Quarterly Forecast vs. Actuals Retrospective

Held within 10 business days of quarter-end. Compares:
- Forecast at quarter-start (pipeline × weighted conversion rates)
- Actual Closed-Won ACV
- Variance analysis by stage (where did deals stall or accelerate?)
- Conversion rate actuals vs. §37.3 assumptions

If actual conversion rate deviates > 10 percentage points from base assumption in any stage, the §37.3 table is updated and a `enterprise.pipeline_conversion_model_recalibrated` DEC-030 event is emitted (§37.9.4). Recalibration requires a `docs/DECISION_LOG.md` entry.

#### 37.7.4 Forecast Override Policy

If the founder overrides the weighted-pipeline forecast (e.g., includes a deal as "highly likely" despite being in S1), the override must:
1. Be documented in the pipeline spreadsheet with a written rationale
2. Emit a `enterprise.forecast_override_applied` DEC-030 event with the unadjusted and adjusted ARR amounts
3. Be reviewed in the next quarterly retrospective

Forecast overrides that consistently under-forecast (i.e., override says "likely" but deal churns at S1) are a SOC 2 CC9.2 governance signal — FORM should not present optimistic pipeline figures to investors without a matching audit trail.

---

### 37.8 Postgres Pipeline Tracking Schema

Pre-CRM, FORM tracks enterprise pipeline in `enterprise_pipeline_stages`. The schema is designed to be CRM-replaceable at Series A — CRM becomes the source of truth and `enterprise_pipeline_stages` is retired.

#### 37.8.1 Table DDL

```sql
-- Tracks each pipeline stage entry/exit event per deal.
-- One row per stage transition, not per deal (immutable audit pattern).
CREATE TABLE enterprise_pipeline_stages (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id        UUID REFERENCES tenants(id) ON DELETE SET NULL,
                   -- NULL for pre-contract deals (no tenant provisioned yet)
  deal_id          UUID NOT NULL,
                   -- Stable identifier for the deal across stage transitions.
                   -- Generated by founder at S0 entry; persists through Closed-Won.
  stage            TEXT NOT NULL CHECK (stage IN (
                     'S0_inbound',
                     'S1_qualified',
                     'S2_pilot',
                     'S3_proposal',
                     'S4_legal_review',
                     'S5_closed_won',
                     'stale',         -- deal aged out and removed from pipeline
                     'closed_lost'    -- deal explicitly lost (with reason)
                   )),
  acv_estimate_usd NUMERIC(12,2) CHECK (acv_estimate_usd > 0),
                   -- Best estimate of ACV at stage entry; updated on seat/tier changes.
  seat_estimate    INTEGER CHECK (seat_estimate >= 50),
  tier_estimate    TEXT CHECK (tier_estimate IN ('starter','growth','enterprise')),
  entered_at       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  exited_at        TIMESTAMPTZ,       -- NULL = currently in this stage
  exit_reason      TEXT,              -- for 'stale' or 'closed_lost': required
  source           TEXT CHECK (source IN (
                     'inbound_email', 'inbound_form', 'inbound_referral',
                     'outbound_founder', 'outbound_partner', 'conference', 'other'
                   )),
  champion_name    TEXT,              -- internal champion at prospect; never PII-scoped to FORM users
  notes            TEXT,
  created_by       UUID NOT NULL      -- founder or CSM UUID (FORM team, not tenant)
);

-- RLS: form_admin read; form_system write only
ALTER TABLE enterprise_pipeline_stages ENABLE ROW LEVEL SECURITY;
CREATE POLICY pipeline_read  ON enterprise_pipeline_stages FOR SELECT USING (auth.role() IN ('form_admin', 'compliance_reviewer'));
CREATE POLICY pipeline_write ON enterprise_pipeline_stages FOR INSERT WITH CHECK (auth.role() = 'form_system');

-- Indexes
CREATE INDEX ON enterprise_pipeline_stages (deal_id);
CREATE INDEX ON enterprise_pipeline_stages (stage) WHERE exited_at IS NULL;
CREATE INDEX ON enterprise_pipeline_stages (entered_at);
```

#### 37.8.2 Pipeline Coverage Ratio Query

```sql
-- Run weekly (Monday pipeline review). Returns current quarter PCR.
WITH stage_weights AS (
  SELECT stage, weight FROM (VALUES
    ('S0_inbound',     0.05),
    ('S1_qualified',   0.15),
    ('S2_pilot',       0.40),
    ('S3_proposal',    0.65),
    ('S4_legal_review',0.80),
    ('S5_closed_won',  1.00)
  ) AS t(stage, weight)
),
current_pipeline AS (
  SELECT
    p.deal_id,
    p.stage,
    p.acv_estimate_usd,
    w.weight,
    p.acv_estimate_usd * w.weight AS weighted_acv
  FROM enterprise_pipeline_stages p
  JOIN stage_weights w ON w.stage = p.stage
  WHERE p.exited_at IS NULL
    AND p.stage NOT IN ('stale', 'closed_lost')
)
SELECT
  SUM(weighted_acv)                         AS total_weighted_pipeline_usd,
  COUNT(DISTINCT deal_id)                   AS active_deals,
  SUM(CASE WHEN stage = 'S5_closed_won' THEN acv_estimate_usd ELSE 0 END) AS closed_won_usd,
  ROUND(SUM(weighted_acv) /
    NULLIF(
      -- Replace 90000 with the quarterly ARR target from the financial model
      90000, 0
    ), 2
  )                                         AS pipeline_coverage_ratio
FROM current_pipeline;
```

#### 37.8.3 Stage Conversion Rate Actuals Query

```sql
-- Run quarterly (retrospective). Returns actual conversion rates per transition.
WITH transitions AS (
  SELECT
    deal_id,
    stage,
    LAG(stage) OVER (PARTITION BY deal_id ORDER BY entered_at) AS prev_stage,
    entered_at
  FROM enterprise_pipeline_stages
),
conversions AS (
  SELECT
    prev_stage || ' → ' || stage AS transition,
    COUNT(*) AS count
  FROM transitions
  WHERE prev_stage IS NOT NULL
    AND stage NOT IN ('stale', 'closed_lost')
  GROUP BY 1
),
losses AS (
  SELECT
    prev_stage || ' → stale/lost' AS transition,
    COUNT(*) AS count
  FROM transitions
  WHERE prev_stage IS NOT NULL
    AND stage IN ('stale', 'closed_lost')
  GROUP BY 1
)
SELECT * FROM conversions
UNION ALL
SELECT * FROM losses
ORDER BY 1;
```

#### 37.8.4 Pipeline Aging Alert Query

```sql
-- Run daily (automated pg_cron job 31 — see §37.10 item 2).
-- Returns deals in each stage that have exceeded their maximum age.
SELECT
  deal_id,
  stage,
  entered_at,
  NOW() - entered_at                  AS age,
  CASE stage
    WHEN 'S0_inbound'      THEN INTERVAL '14 days'
    WHEN 'S1_qualified'    THEN INTERVAL '30 days'
    WHEN 'S2_pilot'        THEN INTERVAL '90 days'
    WHEN 'S3_proposal'     THEN INTERVAL '60 days'
    WHEN 'S4_legal_review' THEN INTERVAL '45 days'
    ELSE NULL
  END                                 AS max_age,
  (NOW() - entered_at) >
  CASE stage
    WHEN 'S0_inbound'      THEN INTERVAL '14 days'
    WHEN 'S1_qualified'    THEN INTERVAL '30 days'
    WHEN 'S2_pilot'        THEN INTERVAL '90 days'
    WHEN 'S3_proposal'     THEN INTERVAL '60 days'
    WHEN 'S4_legal_review' THEN INTERVAL '45 days'
    ELSE FALSE
  END                                 AS is_aged_out,
  acv_estimate_usd,
  champion_name,
  notes
FROM enterprise_pipeline_stages
WHERE exited_at IS NULL
  AND stage NOT IN ('S5_closed_won', 'stale', 'closed_lost')
ORDER BY age DESC;
```

---

### 37.9 DEC-030 HMAC-Chained Audit Events

Four new DEC-030 events for pipeline governance. All use `deal_id` (not `tenant_id`) for pre-contract stage events. No `user_id` — pipeline tracking is at deal level, never at individual employee level.

#### 37.9.1 Event: `enterprise.pipeline_reviewed`

| Field | Value |
|---|---|
| **Event type** | `enterprise.pipeline_reviewed` |
| **Severity** | STANDARD |
| **Retention** | 3 years |
| **Actor** | founder |
| **Trigger** | Weekly Monday pipeline review completed |

**Payload (Zod v2 schema):**
```typescript
z.object({
  review_date:              z.string().date(),          // YYYY-MM-DD (Monday)
  active_deals:             z.number().int().min(0),
  weighted_pipeline_usd:    z.number().min(0),
  pipeline_coverage_ratio:  z.number().min(0),          // vs. quarterly target
  deals_aged_out_count:     z.number().int().min(0),    // count of aged-out deals actioned
  closed_won_this_week_usd: z.number().min(0),
  actor_id:                 z.string().uuid(),           // founder FORM-team UUID
})
```

**Privacy invariant:** No `tenant_id`, no prospect company name, no individual user data. `deal_id` references are aggregate counts only in this event — full deal_id list is in `enterprise_pipeline_stages`, not in the audit chain.

#### 37.9.2 Event: `enterprise.arr_bridge_closed`

| Field | Value |
|---|---|
| **Event type** | `enterprise.arr_bridge_closed` |
| **Severity** | STANDARD |
| **Retention** | 7 years |
| **Actor** | founder or data-engineer |
| **Trigger** | Monthly ARR bridge reconciled and signed off |

**Payload (Zod v2 schema):**
```typescript
z.object({
  period_month:           z.string().regex(/^\d{4}-\d{2}$/),  // YYYY-MM
  opening_arr_usd:        z.number().min(0),
  new_arr_usd:            z.number().min(0),
  expansion_arr_usd:      z.number().min(0),
  contraction_arr_usd:    z.number().min(0),
  churned_arr_usd:        z.number().min(0),
  indexation_arr_usd:     z.number().min(0),
  closing_arr_usd:        z.number().min(0),
  monthly_nrr:            z.number().min(0),    // (closing - new) / opening, annualised
  active_contracts:       z.number().int().min(0),
  actor_id:               z.string().uuid(),
})
```

**7-year retention rationale:** ARR bridge reconciliation is a financial record equivalent to a monthly revenue statement. Ukrainian Tax Code Art. 44 requires 7-year retention of financial records. US (SOX-adjacent) best practice: 7 years. This event is the investor-grade evidence that FORM's ARR figures were computed consistently and auditably each month.

#### 37.9.3 Event: `enterprise.deal_aged_out`

| Field | Value |
|---|---|
| **Event type** | `enterprise.deal_aged_out` |
| **Severity** | STANDARD |
| **Retention** | 3 years |
| **Actor** | `form_system` (pg_cron job 31) |
| **Trigger** | Pipeline aging query (§37.8.4) detects a deal exceeding max stage age |

**Payload (Zod v2 schema):**
```typescript
z.object({
  deal_id:             z.string().uuid(),
  stage:               z.enum([
    'S0_inbound','S1_qualified','S2_pilot','S3_proposal','S4_legal_review'
  ]),
  entered_at:          z.string().datetime(),
  age_days:            z.number().int().positive(),
  max_age_days:        z.number().int().positive(),
  acv_estimate_usd:    z.number().min(0),
  action_required:     z.enum(['call_prospect','review_qualification','mid_pilot_intervention','re_engage_buyer','contact_legal']),
})
```

**Note:** `deal_id` is a FORM-internal UUID generated at S0 entry — it is never shared with the prospect or tenant and does not map to a Postgres `tenants.id`. Privacy invariant preserved.

#### 37.9.4 Event: `enterprise.pipeline_conversion_model_recalibrated`

| Field | Value |
|---|---|
| **Event type** | `enterprise.pipeline_conversion_model_recalibrated` |
| **Severity** | STANDARD |
| **Retention** | 7 years |
| **Actor** | founder or data-engineer |
| **Trigger** | Quarterly retrospective reveals > 10pp deviation in any stage conversion rate |

**Payload (Zod v2 schema):**
```typescript
z.object({
  recalibration_date:    z.string().date(),
  deals_analysed:        z.number().int().min(1),
  stage_updates: z.array(z.object({
    stage_transition:    z.string(),       // e.g. "S1 → S2"
    old_rate:            z.number(),       // prior assumption (decimal, e.g. 0.55)
    new_rate:            z.number(),       // updated assumption
    deviation_pp:        z.number(),       // actual vs. old assumption (percentage points)
  })),
  decision_log_ref:      z.string(),       // e.g. "DEC-XXX"
  actor_id:              z.string().uuid(),
})
```

#### 37.9.5 DEC-030 Event Summary for §37

| Event | Severity | Retention | Trigger | Privacy constraint |
|---|---|---|---|---|
| `enterprise.pipeline_reviewed` | STANDARD | 3 yr | Weekly Monday pipeline review | Aggregate deal counts; no tenant IDs, no prospect names |
| `enterprise.arr_bridge_closed` | STANDARD | 7 yr | Monthly ARR bridge sign-off | Aggregate ARR components only; no per-customer breakdown |
| `enterprise.deal_aged_out` | STANDARD | 3 yr | pg_cron job 31 daily aging check | `deal_id` (FORM-internal UUID) only; no prospect PII |
| `enterprise.pipeline_conversion_model_recalibrated` | STANDARD | 7 yr | Quarterly retrospective; > 10pp deviation | Aggregate conversion rates; no per-deal data in payload |

**Registration requirement:** All four events must be registered in `docs/AUDIT_LOG_SCHEMA.md §6` before the first S0 deal is entered into `enterprise_pipeline_stages`. See §37.10 item 1.

---

### 37.10 Implementation Checklist

#### P0 — Before first S0 deal entered (pre-M8)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Register all four §37.9 DEC-030 events in `docs/AUDIT_LOG_SCHEMA.md §6`. Deploy event types to `emit-audit-event` Worker. | platform-engineer | **P0** | M7 | [ ] |
| 2 | Create `enterprise_pipeline_stages` table (§37.8.1 DDL) with RLS. Register migration `0061_enterprise_pipeline_stages.sql`. | platform-engineer | **P0** | M7 | [ ] |
| 3 | Set up weekly pipeline review cadence — Monday 09:00 UTC recurring calendar event. First review: M8. | founder | **P0** | M8 | [ ] |
| 4 | Configure pg_cron job 31: daily 07:00 UTC, runs §37.8.4 aging query; for each aged-out deal, emits `enterprise.deal_aged_out` DEC-030 event and sends Slack alert to #enterprise-pipeline channel. | platform-engineer | **P0** | M8 | [ ] |

#### P1 — Before first enterprise deal closes (M9–M10)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 5 | Set up monthly ARR bridge reconciliation process using §37.6.2 steps. First bridge close: end of month of first Closed-Won deal. Emit `enterprise.arr_bridge_closed` DEC-030 event. | founder + data-engineer | **P1** | M10 | [ ] |
| 6 | Build the §37.8.2 PCR query into a Retool or Supabase Studio dashboard accessible by founder. Target: PCR visible within 2 minutes of opening the dashboard. | data-engineer | **P1** | M9 | [ ] |
| 7 | Document quarterly ARR target for each quarter of Y1 in a `enterprise_arr_targets` config table (1 row per quarter; `target_arr_usd`; read-only for `form_admin`). Required input for the PCR query denominator. | founder | **P1** | M8 | [ ] |

#### P2 — Before Series A fundraising preparation (M18–M20)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 8 | After 10 closed-won deals, run §37.8.3 conversion rate actuals query. Compare to §37.3 base assumptions. If deviation > 10pp in any stage, update §37.3 and emit `enterprise.pipeline_conversion_model_recalibrated`. Document in `docs/DECISION_LOG.md`. | data-engineer + founder | **P2** | M18 | [ ] |
| 9 | Evaluate CRM tool selection (OQ-PIPE-02 below) and migration path from `enterprise_pipeline_stages`. If CRM adopted, `enterprise_pipeline_stages` becomes a read-only historical archive; new pipeline data flows from CRM via webhook. | founder + data-engineer | **P2** | M20 | [ ] |
| 10 | Update §37.5 ARR Build Table with actuals from the first 8 quarters. Publish revised Y3–Y5 forecast for Series A data room. | data-engineer + founder | **P2** | M22 | [ ] |

---

### 37.11 Open Questions

| ID | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-PIPE-01** | **What are FORM's actual stage conversion rates after 10 closed-won deals?** §37.3 uses conservative-to-base benchmarks derived from comparable SaaS platforms. FORM's CV-demo advantage in the pilot and the DPA-before-data privacy floor may improve S2→S3 rates; the GDPR Art. 9 DPA negotiation requirement may extend S4 beyond 45 days for EU deals. Neither effect can be measured before 10 deals. Resolution: run §37.8.3 query after Deal 10; update §37.3 and emit `enterprise.pipeline_conversion_model_recalibrated` per §37.7.3 governance. | **P1** | data-engineer + founder | After Deal 10 (est. M18) |
| **OQ-PIPE-02** | **Which CRM tool should FORM adopt, and when?** Pre-Series A, `enterprise_pipeline_stages` plus a spreadsheet is sufficient for 3–10 deals. At 10+ active pipeline deals, a CRM (HubSpot, Salesforce, Attio) reduces founder cognitive load and provides investor-grade reporting. Cost consideration: HubSpot Starter is ~$600/year; Salesforce Starter is ~$900/year; Attio is usage-based. Resolution: evaluate at M12 (when pipeline likely has 5+ active deals); adopt before M18. Compatibility requirement: CRM must support bi-directional webhook to keep `enterprise.pipeline_reviewed` DEC-030 events current. | **P2** | founder | M12 evaluation; M18 adoption |
| **OQ-PIPE-03** | **Should FORM track partner-sourced deals separately in the pipeline?** Currently all deals are founder-sourced (outbound or inbound). If FORM develops a partner channel (HR consultancies, benefits brokers, EAP platforms), partner-sourced deals may have different conversion rates and ACV profiles. A `source_partner_id` column in `enterprise_pipeline_stages` would enable attribution. Resolution: defer until first partner deal; add column at that point (backward-compatible schema change). | **P2** | founder | On first partner deal; no later than M24 |

---

*v2.3 (2026-06-13): §37 Enterprise Pipeline Health & ARR Forecasting Model — six-stage pipeline definition (S0 Inbound → S5 Closed-Won) with entry/exit criteria and maximum stage age; stage conversion rate table (conservative / base / optimistic with benchmark sources: OpenView Partners 2024, Gartner wellness 2023); three pipeline health metrics: PCR (3.0× healthy threshold; weighted by stage from 5% S0 → 100% S5), sales velocity formula (Y1 base: $1,944/month; Y2: $9,164/month), and pipeline aging triggers with required actions by stage; ARR build table Y1–Y3 base case and downside (40% logo miss) with quarterly granularity; monthly ARR bridge reconciliation (five components: new + expansion − contraction − churn + indexation) with NRR floor thresholds (100% Y1 / 110% Y2 / 120% Y3+); forecasting governance cadence (weekly pipeline review / monthly bridge close / quarterly retrospective) with forecast override policy; `enterprise_pipeline_stages` Postgres table DDL with RLS (form_admin + compliance_reviewer read; form_system write), four SQL queries (PCR, conversion actuals, aging alert, monthly bridge); four DEC-030 HMAC-chained events: `enterprise.pipeline_reviewed` (STANDARD, 3yr — aggregate deal counts; no tenant/prospect IDs), `enterprise.arr_bridge_closed` (STANDARD, 7yr — aggregate ARR components; investor-grade financial record; 7yr per Ukrainian Tax Code Art. 44), `enterprise.deal_aged_out` (STANDARD, 3yr — deal_id internal UUID only; pg_cron job 31 daily), `enterprise.pipeline_conversion_model_recalibrated` (STANDARD, 7yr — fires when >10pp deviation in any stage; requires DECISION_LOG entry); 10-item implementation checklist (4× P0/M7–M8, 3× P1/M9–M10, 3× P2/M18–M22); 3 open questions (OQ-PIPE-01 actual conversion rates after Deal 10; OQ-PIPE-02 CRM adoption decision; OQ-PIPE-03 partner channel attribution). Cross-references: §19 (GTM financial model — sales velocity inputs); §23 (NRR engine — expansion ARR source data for bridge); §34 (churn register — churned ARR source for bridge); §35 (contract amendments — contraction ARR); §36 (implementation cost — used in §37.5 Y1 ARR table comments); §22 (cash flow model — ARR bridge feeds cash projection); §24.3 (Series A readiness — $2.1M ARR Q4 Y3 target from §37.5); `docs/ENTERPRISE.md` (commercial model and team); `docs/AUDIT_LOG_SCHEMA.md` (four new DEC-030 events to register — P0 before first pipeline entry); `docs/DECISION_LOG.md` (OQ-PIPE-01 closure; forecast override policy §37.7.4). Privacy floor: no individual employee user_id in any §37 DEC-030 event; no prospect PII in audit chain; `deal_id` is FORM-internal UUID never shared externally. Owner: enterprise-architect + customer-success + data-engineer + compliance-officer.*

---

*v2.2 (2026-06-12): §36 Enterprise Implementation Cost Deep-Dive: Engineering, CS & Legal Time Model — bottom-up derivation of per-deal implementation cost using hourly rates from §26 (CSM model) and §27 (engineering model). §36.1 scopes to one-time implementation cost only (ongoing CS in §26; infrastructure COGS in §15). §36.2 six-work-stream activity taxonomy (identity integration, tenant provisioning, white-label, training, pilot management, compliance/legal). §36.3 engineering hour model by tier: SSO/SCIM setup (4.5h Starter / 8.5h Growth / 11.5h Enterprise non-EU) + tenant provisioning (2.5h all) + white-label (2.0h Growth+); per-tier engineering cost $437.50 / $812.50 / $1,000.00 at $62.50/hr (§27.2 founding engineer rate). §36.4 CS hour model by tier: training + admin onboarding (9.0h / 14.5h / 22.5h) + pilot management (6.0h / 10.0h / 16.0h) = 15.0h / 24.5h / 38.5h; per-tier CS cost $661.95 / $1,081.19 / $1,699.01 at $44.13/hr (fully-loaded CSM rate — more accurate than §26.8's base-only $37.50/hr for margin analysis). §36.5 legal/compliance model: MSA (template deal: 2h internal + $300 counsel), DPA (1h internal), ISQ response (3h / 5h / 9h by tier), security review call (0h / 1h / 2h) — total internal 6h / 9h / 14h + $300 counsel all tiers. §36.6 bottom-up cost summary: Starter $1,399 / Growth $2,194 / Enterprise $2,999 (cash, non-EU, template deal); variance vs. §8.2 midpoints: +22% / +46% / +50% — ISQ response time and fully-loaded CS rate are primary drivers of gap; EU premium +$362.50 all tiers. §36.7 implementation cost as % ACV: 19.4% at 50-seat Starter falling to 0.8% at 5,000-seat Enterprise; Year 1 GM impact vs. §16.1: Starter 50-seat −3.5pp (39.9%), Growth 200-seat −3.3pp (71.6%), Enterprise 1,000-seat −1.4pp (83.8%); 50-seat Starter minimum deal floor remains defensible (implementation cost recovered in 2.3 months of gross profit; 3-year LTV recovery 17.5×). §36.8 multi-deal scale economics: MSA template + ISQ master construction are fixed front-loaded costs in Deals 1–3 (→ 15–20% cost reduction by Deal 6); founding engineer implementation load ~36h for 3-deal Year 1 pipeline = 4.5% of 6-month capacity (not a bottleneck); non-standard IdP premium table (+$0 to +$750 by IdP type; legacy IdP may warrant a line-item fee in order form). §36.9 time-tracking instrument: `enterprise_impl_time_log` Postgres table with 12 `activity_code` values, RLS (compliance_reviewer + form_admin read; form_system write), deal_sequence column; cost extraction query; OQ-08 closure protocol (3 deals logged → §8.2 update → DEC-030 emission → DECISION_LOG entry). §36.10 three new DEC-030 events: `enterprise.implementation_kickoff_completed` (STANDARD, 7yr), `enterprise.sso_scim_setup_verified` (STANDARD, 7yr — idp_type + sso_modes_verified + scim_features_enabled + eu_data_residency_confirmed), `enterprise.implementation_cost_model_calibrated` (STANDARD, 7yr — OQ-08 closure trigger; aggregate cost data only). §36.11 nine-item implementation checklist: 4× P0/M8–M9 (DEC-030 registration, table DDL, extraction query, time-logging start at Deal 1), 3× P1/M14–M15 (Deal 3 variance analysis + §8.2 update + DEC-030 emission + non-IdP per-IdP tracking), 2× P2/M13–M14 (IdP fee decision + §16.1 table update). §36.12 three open questions (OQ-IMPL-01/02/03). TOC updated. §11 OQ-08 tracking instrument now in place — full closure per §36.9.3 after Deal 3. Cross-references: §8.2 (implementation cost summary — to be updated post OQ-08 closure); §11 OQ-08 (original open question); §15.5–15.7 (enterprise infrastructure COGS — near-zero; implementation is the true cost); §16.1 (break-even table — Year 1 GM updated in §36.7.2); §26 (CSM cost model — ongoing cost, not one-time); §26.3 / §26.8 (CSM rate basis); §27.2 (engineering rate basis); §31.3 (enterprise price derivation — fully-loaded GM cross-check); `docs/ENTERPRISE_ONBOARDING.md` (operational playbook — §36 provides the financial model behind the same timeline); `docs/SSO_SCIM_IMPLEMENTATION.md §4` (IdP configuration playbook — time basis for §36.3.1); `docs/SSO_CLIENT_CONFIG.md` (per-IdP guides — non-standard IdP premium §36.8.3 cross-ref); `docs/AUDIT_LOG_SCHEMA.md` (three new DEC-030 events to register — P0 before Deal 1). Owner: enterprise-architect + customer-success + data-engineer + compliance-officer.*

