# FORM · Project Status v1.3.2

**Date:** 16 травня 2026
**Version:** 0.47.1
**Phase:** Pre-launch · Closed Beta planning Q3 2026
**Repo:** https://github.com/Rbit27/form

---

## TL;DR (one paragraph)

FORM is at v0.47.0 with 46+ tagged releases, 16 functional HTML pages (marketing site, product mockups, investor one-pager, careers, enterprise landing, enterprise pricing calculator, and articles index), 58+ markdown documents covering every operational domain, 16 blog posts ready for distribution, 2 newsletter issues (drafts, clinical-safety cleared), a full advisory board recruitment playbook (ADVISORY_BOARD.md — 4 advisor profiles, outreach templates, trial protocol, timeline), a platform-by-platform hiring guide (HIRING_GUIDE.md — Djinni/DOU/LinkedIn/Wellfound, screening protocol, response templates), a verbal pre-seed pitch narrative (SEED_NARRATIVE.md — 6-block investor call script, objection playbook, audience variants), an investor DD FAQ (DD_FAQ.md — 7-block due diligence prep), a user research screener toolkit (RESEARCH_SCREENER.md — screener survey with pass/reject criteria, Reddit/X/Telegram recruiting messages, interview tracker columns, gift card payment flow, anti-bias checklist — ready to use immediately without waiting for site launch), a full press kit (HTML + markdown), hiring JDs ready to post with a designed careers page, a newsletter setup guide, 12 LinkedIn posts ready to publish with a full deployment strategy, a complete beta community playbook, a beta recruiting playbook (channels, scoring, incentives, timeline to 200 users), a 30-assumption pre-research register (ASSUMPTIONS_REGISTER.md) with confidence levels and pivot criteria, sample programs and nutrition plans, full legal drafts, a public GitHub repo with 29 documented decisions, a growth loops playbook (GROWTH_LOOPS.md — 5 viral/organic loops, clinical-safety constraints on share content, FE implementation priority), a pre-launch Twitter/X playbook (TWITTER_PLAYBOOK.md — account setup, 5 content pillars with example tweets, build-in-public formula, beta recruiting via X, voice rules for X vs LinkedIn vs newsletter, quick-start checklist George can act on today without domain or FE hire), an investor CRM and relationship management framework (INVESTOR_PIPELINE.md — 7-stage pipeline model, Google Sheets schema, cadence rules, interest signal taxonomy, close mechanics, realistic raise timeline), and a LinkedIn strategy playbook (LINKEDIN_PLAYBOOK.md — profile setup with headline variants and About draft, 4 content pillars, 3-tier connection strategy, hiring InMail templates for FE + Design Lead, investor visibility playbook, build-in-public translation guide from X, 2-3x/week cadence, voice rules, SSI metrics, 10-item quick-start checklist). Built solo by founder + AI agent team + 2-hour cloud iteration cycle. No code shipped yet — that begins with Founding Engineer hire post-Seed. Product spec for FE available: `docs/PRODUCT_SPEC.md`. Analytics instrumentation guide: `docs/ANALYTICS_SETUP.md`.

---

## What's built

### Product surfaces (HTML, viewable у browser)

| Page | Purpose | Status |
|---|---|---|
| [`index.html`](index.html) | Marketing hero + 5-screen phone demo | v0.4.6 ✓ |
| [`onboarding.html`](onboarding.html) | 8-screen onboarding з ED-screening | v0.3.1 (cloud) ✓ |
| [`beta.html`](beta.html) | Closed beta application form | v0.3.6 ✓ |
| [`pricing.html`](pricing.html) | Geo-priced з competitor comparison | v0.3.7 ✓ |
| [`about.html`](about.html) | Manifesto з 4 red lines | v0.3.8 ✓ |
| [`settings.html`](settings.html) | App settings mockup (3-click cancel) | v0.3.9 ✓ |
| [`screens.html`](screens.html) | 5 app-screen mockups (cloud) | v0.3.6 ✓ |
| [`faq.html`](faq.html) | 30+ questions у 7 categories | v0.4.4 ✓ |
| [`press.html`](press.html) | Full HTML press kit | v0.6.0 ✓ |
| [`workout-live.html`](workout-live.html) | Detailed workout mockup з CV | v0.4.8 ✓ |
| [`investors.html`](investors.html) | Investor one-pager (noindex — direct link) | v0.14.0 ✓ |
| [`jobs.html`](jobs.html) | Careers page — FE + Design Lead open roles | v0.15.0 ✓ |
| [`blog.html`](blog.html) | Articles index — 97 cards (103 content posts authored), category filter, newsletter CTA | v1.9.21 ✓ |
| [`enterprise.html`](enterprise.html) | Enterprise landing — SSO/SCIM/SOC2, tiers, pilot CTA | v0.44.0 ✓ |
| [`pricing-enterprise.html`](pricing-enterprise.html) | Enterprise pricing calculator — seats/contract/discounts | v0.45.0 ✓ |

### Strategic documentation (markdown у `docs/`)

| Document | Domain | Owner |
|---|---|---|
| BRAND.md | Brand book, tokens, voice | brand-system |
| FLOWS.md | User flows, IA, edge cases | ux-flow |
| MARKETING.md | GTM, channels, positioning | marketing-lead |
| PITCH.md | 12-slide pitch + one-pager | product-strategist |
| PERSONAS.md | 4 detailed personas (Дмитро primary) | research-lead |
| TECHNICAL.md | Architecture, stack, CV-pipeline | platform-engineer |
| FINANCIALS.md | Unit econ, 3-year P&L | growth-lead |
| COMPETITIVE.md | 8-product teardown | marketing-lead |
| RESEARCH.md | 30-interview plan + script | research-lead |
| LEGAL.md | Privacy, ToS, GDPR framework | product-strategist |
| METRICS.md | NSM (W-ACSU), AARRR, dashboards | growth-lead |
| FOUNDER.md | Personal narrative (INTERNAL) | founder |
| HIRING.md | First 3 roles, interview rubric | founder |
| PRESS.md | Press kit, boilerplate, founder bio | content-strategist |
| INVESTOR_OUTREACH.md | Target funds, templates, due-dilig | founder |
| API.md | API design, Postgres schema | platform-engineer |
| EMAIL_SEQUENCE.md | 18+ email templates A-D | content-strategist |
| BETA_PLAYBOOK.md | TestFlight beta operationally | founder |
| SECURITY.md | Threat model, 5-layer defense | platform-engineer |
| OKRS_2026.md | Q3 + Q4 OKRs з kill-criteria | founder |
| DATA_ROOM.md | 4-tier investor materials | founder |
| LAUNCH_CHECKLIST.md | P0/P1/P2 launch gate | founder |
| I18N.md | Localization plan UA+EN, DE/PL/ES | marketing-lead |
| APP_STORE.md | App Store listing draft (cloud) | marketing-lead |
| DECISION_LOG.md | 28 documented decisions | process-keeper |
| ENGINEERING_RUNBOOK.md | On-call, deploy, monitoring | platform-engineer |
| INVESTOR_UPDATE_TEMPLATE.md | Monthly update format | founder |
| SUPPORT.md | Customer support playbook | founder |
| B2B_PLAYBOOK.md | Corporate wellness (post-PMF) | founder |
| CONTENT_CALENDAR.md | 90-day editorial calendar | content-strategist |
| NEWSLETTER.md | Newsletter platform setup + 2 post concepts | content-strategist |
| COMMUNITY.md | Beta community setup (Slack + Discord, moderation) | process-keeper |
| PRODUCT_SPEC.md | iOS product spec для Founding Engineer (sprints, acceptance criteria) | platform-engineer |
| ANALYTICS_SETUP.md | PostHog instrumentation guide — 65 events, W-ACSU SQL, dashboards | platform-engineer |
| BETA_RECRUITING.md | Beta user recruiting playbook — channels, scoring, timeline до 200 TestFlight users | growth-lead |
| FIRST_30_DAYS.md | 4-week execution playbook — domain → interviews → investor outreach → FE hire | process-keeper |
| ASSUMPTIONS_REGISTER.md | 30 pre-research assumptions з confidence levels, pivot criteria, review schedule | research-lead + product-strategist |
| ADVISORY_BOARD.md | Advisory board recruitment strategy — 4 profiles, compensation, outreach templates, timeline | founder |
| HIRING_GUIDE.md | Platform-by-platform JD posting guide — Djinni, DOU, LinkedIn, Wellfound; screening protocol; timeline | process-keeper |
| SEED_NARRATIVE.md | Verbal pre-seed pitch script — 6 blocks, objection playbook, audience variants (angel/fund/accelerator) | product-strategist |
| DD_FAQ.md | Investor due diligence FAQ — 7 blocks, honest answers to 25 common DD questions, data room index | product-strategist |
| RESEARCH_SCREENER.md | User research recruiting toolkit — screener survey, recruiting messages, tracker template, payment flow | research-lead |
| BETA_FEEDBACK_PROTOCOL.md | Beta feedback intake, taxonomy (P0–P3), synthesis rhythm, decision filter, Victor voice tracker | process-keeper + research-lead |
| TWITTER_PLAYBOOK.md | Pre-launch X/Twitter strategy — account setup, 5 pillars, build-in-public formula, beta recruiting via X, voice rules, metrics, quick-start checklist | content-strategist + brand-voice |
| INVESTOR_PIPELINE.md | Investor CRM and relationship management — 7-stage pipeline (S0–S6), Google Sheets schema, cadence rules, interest signals, post-meeting ritual, close mechanics, raise timeline | founder |
| LINKEDIN_PLAYBOOK.md | LinkedIn strategy — profile setup, 4 content pillars, 3-tier connection strategy, hiring InMail templates, investor visibility playbook, build-in-public translation guide, cadence, metrics, 10-item quick-start checklist | content-strategist + brand-voice |
| RETENTION_PLAYBOOK.md | Retention strategy — D0/D7/D30/D90 windows, notification types + cadence, weekly streak mechanics, Victor personalization arc, re-engagement protocol (3-attempt cap), Pause>Cancel design, clinical-safety copy constraints | growth-lead + clinical-safety |
| VICTOR_PROMPT_GUIDE.md | LLM implementation guide — 5-layer system prompt architecture, TypeScript context schema (HRV, sleep, streak, ED-flag, form score), 4 tone modes with UA/EN examples, 22 scenario library, ED-flag restricted mode, Victor silence rules, 7 open questions for FE | platform-engineer + brand-voice |
| GROWTH_LOOPS.md | Viral and organic growth mechanics — 5 loops (Form Share, Streak Share, Victor WOM, Referral, Beta FOMO), clinical-safety constraints on share content, FE implementation priority, PostHog events, K-factor targets | growth-lead + clinical-safety |

### Content engine (markdown у `content/`)

| Asset | Type | Status |
|---|---|---|
| post-01-hrv-is-lying.md | Blog | draft |
| post-02-what-good-form-means.md | Blog | draft |
| post-03-we-tested-8-ai-trainers.md | Blog | draft |
| post-04-sleep-and-strength.md | Blog | draft |
| post-05-building-fitness-from-ukraine.md | Blog | draft |
| post-06-30-user-interviews.md | Blog | draft template |
| post-07-progressive-overload-myths.md | Blog | draft |
| post-08-why-no-free-tier.md | Blog | draft |
| post-09-building-victor-voice.md | Blog | draft |
| post-10-why-programs-assume-youre-always-ready.md | Blog | draft · sports-scientist + clinical-safety pending |
| post-11-deload-weeks.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-12-technique-over-intensity.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-13-rir-rpe-autoregulation.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-14-sleep-deprived-training.md | Blog | draft · clinical-safety PENDING · sports-scientist DONE |
| post-15-training-frequency.md | Blog | draft · clinical-safety PASS · sports-scientist DONE |
| post-16-ai-coach-vs-pt.md | Blog | draft · clinical-safety PASS · sports-scientist N/A |
| post-17-overtraining-vs-laziness.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-18-rest-intervals.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-20-doms-muscle-soreness.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-21-cv-vs-wearables.md | Blog | draft · clinical-safety PASS · sports-scientist pending · platform-engineer pending |
| post-22-periodization.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-23-velocity-based-training.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-24-training-to-failure.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-25-breathing-mechanics.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-26-tendon-ligament-adaptation.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-27-training-age-vs-chronological-age.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-28-progressive-overload-mechanics.md | Blog | draft · clinical-safety PASS · sports-scientist DONE |
| post-29-mind-muscle-connection.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-30-eccentric-training.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-31-unilateral-training.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-32-cluster-sets.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-33-energy-systems-for-strength.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-34-supersets-antagonist-pairs.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-35-tempo-training.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-36-detraining.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-37-blood-flow-restriction.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-38-isometric-training.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-39-motor-unit-henneman.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-40-hypertrophy-mechanisms.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-41-grip-strength.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-42-velocity-based-training.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-43-post-activation-potentiation.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-44-heat-cold-recovery.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-45-cns-vs-peripheral-fatigue.md | Blog | draft · clinical-safety not-required · sports-scientist pending |
| post-46-hormones-training.md | Blog | draft · clinical-safety conditional-pass · sports-scientist pending |
| post-47-concurrent-training.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-48-training-volume-adaptation.md | Blog | draft · clinical-safety not-required · sports-scientist pending |
| post-49-rate-of-force-development.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-50-proprioception-balance.md | Blog | draft · clinical-safety not-required · sports-scientist pending |
| post-51-breathing-iap-strength.md | Blog | draft · clinical-safety conditional-pass · sports-scientist pending |
| post-52-muscle-fiber-types.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-53-programming-for-busy.md | Blog | draft · clinical-safety not-required · sports-scientist pending |
| post-54-training-volume-landmarks.md | Blog | draft · clinical-safety not-required · sports-scientist pending |
| post-55-training-splits.md | Blog | draft · clinical-safety not-required · sports-scientist pending |
| post-56-periodization-models.md | Blog | draft · clinical-safety PASS · sports-scientist DONE |
| post-57-training-after-40.md | Blog | draft · clinical-safety not-required · sports-scientist pending |
| post-58-mind-muscle-connection.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-59-zone2-cardio.md | Blog | draft · clinical-safety not-required · sports-scientist pending |
| post-60-deload-protocols.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-61-overreaching-vs-overtraining.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-62-velocity-based-training.md | Blog | draft · clinical-safety not-required · sports-scientist pending |
| post-63-women-strength-menstrual-cycle.md | Blog | draft · clinical-safety PASS WITH CONDITIONS · sports-scientist pending |
| post-64-plyometrics-strength-athletes.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-65-caffeine-strength-training.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-66-hrv-strength-athletes.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-67-foam-rolling.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-68-heat-cold-training.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-69-creatine.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-70-training-volume-mev-mav-mrv.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-71-allostatic-load.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-72-total-stress-training-capacity.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-73-stretch-mediated-hypertrophy.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-74-bilateral-deficit.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-75-cluster-sets.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-76-lactate-myth.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-77-blood-flow-restriction.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-78-concurrent-training.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-79-said-principle-specificity.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-80-rpe-rir-autoregulation.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-81-neural-vs-structural-adaptation.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-82-deload-protocols.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-83-muscle-fiber-types.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-84-motor-learning.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-85-rate-of-force-development.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-86-periodization-models.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-87-fasted-training.md | Blog | BLOCKED · clinical-safety VETO · food/eating patterns — do not write without CS review |
| post-88-omega3-training.md | Blog | BLOCKED · clinical-safety VETO · supplement/food content — do not write without CS review |
| post-89-acwr-training-load.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-90-calisthenics-vs-weighted-training.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-93-mobility-vs-flexibility.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-97-loaded-carries.md | Blog | draft · clinical-safety PASS |
| post-98-accommodating-resistance.md | Blog | draft · clinical-safety PASS |
| post-99-sticking-points.md | Blog | draft · clinical-safety PASS |
| post-100-e1rm-vs-rep-ranges.md | Blog | draft · clinical-safety PASS · milestone post-100 |
| post-101-sleep-recovery-mechanisms.md | Blog | draft · clinical-safety PASS |
| post-102-heat-cold-training-performance.md | Blog | draft · clinical-safety PASS |
| post-103-readiness-diagnostics.md | Blog | draft · clinical-safety PASS |
| post-104-dehydration-performance.md | Blog | draft · clinical-safety PASS WITH CONDITIONS |
| post-105-grip-strength-longevity.md | Blog | draft · clinical-safety not required |
| post-106-women-strength-anatomy.md | Blog | draft · clinical-safety PASS WITH CONDITIONS (2026-05-24) |
| post-107-hypertrophy-mechanisms-mechanical-tension.md | Blog | draft · clinical-safety PASS |
| post-108-lengthened-partials-stretch-hypertrophy.md | Blog | draft · clinical-safety PASS |
| post-109-force-velocity-curve.md | Blog | draft · clinical-safety PASS |
| post-110-muscle-architecture.md | Blog | draft · clinical-safety PASS |
| post-111-post-activation-potentiation.md | Blog | draft · clinical-safety PASS |
| post-112-rpe-vs-percentage-autoregulation.md | Blog | draft · clinical-safety PASS |
| post-113-eccentric-overload.md | Blog | draft · clinical-safety PASS |
| post-114-training-volume-hypertrophy.md | Blog | draft · clinical-safety PASS |
| post-115-training-frequency-hypertrophy.md | Blog | draft · clinical-safety PASS |
| post-116-hypertrophy-responder-variability.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-117-fitness-fatigue-model.md | Blog | draft · clinical-safety PASS · sports-scientist pending |
| post-120-myofibrillar-vs-sarcoplasmic-hypertrophy.md | Blog | draft · clinical-safety NOT REQUIRED |
| twitter-launch-thread.md | Social | draft |
| linkedin-posts.md | Social | 12 posts ready |
| press-pitch-templates.md | Outreach | 5 templates |
| reddit-launch-posts.md | Social | 3 готові |
| newsletter-01.md | Newsletter | draft · clinical-safety PASS |
| newsletter-02.md | Newsletter | draft · clinical-safety PASS |
| newsletter-03.md | Newsletter | draft · clinical-safety PASS |

### Programs + Nutrition

| File | Purpose |
|---|---|
| programs/sample-strength-week.md | Intermediate strength template |
| programs/sample-deload-week.md | Deload week protocol |
| programs/sample-cycle-aware-week.md | Cycle-aware (women) |
| nutrition/sample-3day-plan.md | 3-day meal plan (~2600 kcal) |
| nutrition/sample-cutting-plan.md | 12-week cutting plan (з ED guards) |

### Legal (markdown у `legal/`)

| File | Status |
|---|---|
| privacy-policy-draft.md | DRAFT — pending counsel |
| terms-of-service-draft.md | DRAFT — pending counsel |

### Hiring (markdown у `hiring/`)

| Role | Status |
|---|---|
| jd-founding-engineer-ios.md | Ready to post |
| jd-design-lead.md | Ready to post |
| jd-sport-science-advisor.md | Ready to post |

### Team agents (24 у `.claude/agents/`)

`product-strategist`, `sports-scientist`, `nutrition-coach`, `clinical-safety` (VETO), `platform-engineer`, `brand-voice`, `design-craft`, `ux-flow`, `brand-system`, `marketing-lead`, `growth-lead`, `content-strategist`, `research-lead`, `process-keeper`, `product-manager`, `qa-walker`, `enterprise-architect`, `compliance-officer`, `devops-lead`, `data-engineer`, `qa-lead`, `security-engineer`, `ml-engineer`, `customer-success` (24 agents total)

---

## What's NOT built (honest gap list)

### Product (engineering work)
- [ ] iOS app codebase
- [ ] Backend services (Cloudflare Workers + Supabase setup)
- [ ] CV pipeline (Apple Vision integration)
- [ ] HealthKit integration
- [ ] Voice (ElevenLabs integration for Victor)
- [ ] Subscription / payment flow (StoreKit 2)
- [ ] Push notification scheduling
- [ ] App Store submission

### Operations
- [ ] Domain (form.coach) registered
- [ ] Email accounts (hello@, hiring@, etc.) configured
- [ ] Company incorporated (Ukraine + Delaware)
- [ ] Bank account
- [ ] Accounting setup
- [ ] Investor relationships warm
- [x] Beta community setup guide → `docs/COMMUNITY.md` (Slack structure, Discord plan, moderation)

### Validation
- [ ] 30 user interviews conducted
- [ ] Personas validated з real users
- [ ] CV-quality validated on real lifters
- [ ] First 200 TestFlight beta users
- [ ] Cohort retention metrics
- [ ] Pricing tested in market

### Content distribution
- [ ] Marketing site deployed (only local prototype)
- [ ] Blog posts published live
- [ ] Twitter / X account active
- [x] Newsletter setup guide → `docs/NEWSLETTER.md`
- [x] LinkedIn posts (12) ready → `content/linkedin-posts.md`
- [x] Newsletter issues (3) drafted → `content/newsletter-01.md` + `newsletter-02.md` + `newsletter-03.md` · clinical-safety PASS
- [ ] Newsletter platform actually configured on Beehiiv
- [ ] Reddit account з reputation built

### Legal / compliance
- [ ] Counsel-reviewed privacy policy
- [ ] Counsel-reviewed ToS
- [ ] Data Processing Agreements з processors
- [ ] App Store privacy nutrition labels
- [ ] DPIA (Data Protection Impact Assessment)

---

## Timeline forward

```
Now (May 15, 2026)
├── v0.6.0 — press kit HTML + newsletter guide
├── v0.7.0 — LinkedIn posts (12 ready) + social content complete
├── v0.8.0 — beta community setup guide (Slack + Discord + moderation)
├── v0.9.0 — iOS product spec for Founding Engineer (PRODUCT_SPEC.md)
├── v0.10.0 — analytics instrumentation guide for FE (ANALYTICS_SETUP.md)
├── v0.11.0 — beta recruiting playbook (channels, scoring, timeline до 200 users)
├── v0.12.0 — 4-week execution playbook (FIRST_30_DAYS.md)
├── v0.13.0 — 30-assumption pre-research register (ASSUMPTIONS_REGISTER.md) + DEC-025/026/027
├── v0.14.0 — investor one-pager HTML page (investors.html)
├── v0.15.0 — careers page HTML (jobs.html) — FE + Design Lead open roles
├── v0.16.0 — newsletter issues 01 + 02 (clinical-safety PASS)
├── v0.17.0 — advisory board recruitment playbook (ADVISORY_BOARD.md)
├── v0.18.0 — hiring guide platform-by-platform (HIRING_GUIDE.md) + DEC-028
├── v0.19.0 — pre-seed verbal pitch narrative (SEED_NARRATIVE.md)
├── v0.20.0 — investor DD FAQ (DD_FAQ.md) — 7-block due diligence prep
├── v0.21.0 — user research screener toolkit (RESEARCH_SCREENER.md) — recruiting-ready
├── Public GitHub repo з 37+ versions tagged
└── All strategic docs published

Next 2 weeks
├── Hire Founding Engineer (FE)
├── Hire Design Lead (DL)
├── Start iOS development
└── Conduct 30 user interviews

M1-M3 (Q3 2026)
├── iOS development z FE
├── CV pipeline build
├── TestFlight closed beta з 200 users
└── First cohort metrics

M4-M6 (Q4 2026)
├── Public App Store launch (UA + Western EU)
├── First 2,000 paid subscribers
├── Seed round close ($1.5-2.5M target)
└── Twitter / Reddit / content engine active

M7-M9 (Q1 2027)
├── Android development starts
├── Whoop / Oura partnerships
├── $1M ARR target
└── First B2B pilot consideration

M10-M12 (Q2 2027)
├── Android launch
├── Series A preparation
└── Cohort retention validation
```

---

## Decision history (highlights from DECISION_LOG.md)

- DEC-016: No free tier (sub-only)
- DEC-017: 8 base lifts CV-coaching scope (chosen quality over breadth)
- DEC-018: ED-screening non-skippable (clinical-safety HARD VETO)
- DEC-021: No Russian language UI у v1.0
- DEC-023: Geo-pricing $9 UA / $19 Western
- DEC-029: Growth loops — sharing зовні, no in-app social, clinical-safety gate on all share content
- DEC-030: Audit log append-only HMAC-chained, 7-year retention, support actions auto-notify tenant
- DEC-031: Agent team expanded 14 → 24 agents
- DEC-032: Law enforcement no-backdoor policy formalized (INCIDENT_RESPONSE R-13)
- DEC-033: Post-100 milestone: quality over velocity (clinical-safety-blocked posts remain deferred)

33 total documented decisions з reverse-cost assessment.

---

## What's working well

1. **Documentation discipline.** Every decision logged. Every doc versioned. Every tag pushed.
2. **Brand consistency.** Editorial-brutalist aesthetic holds across 12 HTML pages.
3. **Safety-first design.** clinical-safety VETO has shaped many decisions positively.
4. **Operational thinking.** Не just product — full operational playbook (support, ops, B2B, hiring, legal, finance).
5. **Build-in-public.** Public repo з transparent decision-making.

---

## What's worrying

1. **Zero code shipped.** All strategy, no engineering yet. Founding Engineer hire critical.
2. **Zero validated users.** 30 interviews planned but not conducted.
3. **CV-quality unknown.** Technical bet not validated.
4. **No revenue.** Pre-seed, runway = personal savings.
5. **Solo founder bottleneck.** Many critical decisions concentrated.

---

## What's next (immediate)

- [x] Careers page live → `jobs.html` (shareable URL for FE + DL hiring)
- [ ] Apply Founding Engineer + Design Lead JDs on platforms (Djinni, DOU, LinkedIn, Wellfound)
- [ ] Domain form.coach registered
- [ ] Twitter / X account з first posts
- [ ] Schedule 5 first user interviews
- [ ] Configure Beehiiv newsletter (гайд у docs/NEWSLETTER.md)

---

## Public links

- **Repo:** https://github.com/Rbit27/form
- **Releases:** https://github.com/Rbit27/form/releases
- **CHANGELOG:** [CHANGELOG.md](CHANGELOG.md)
- **All decisions:** [DECISION_LOG.md](docs/DECISION_LOG.md)
- **Hiring:** [hiring/](hiring/)
- **Press:** [press.html](press.html)

---

## What this status doc is для

- Quick onboarding for new hires
- Investor update foundation
- Recruiter context
- Friends who ask «що ти будуєш?»
- My own clarity check

---

**Updated weekly.** Major changes → new version bump.

**v1.9.18 · 23 травня 2026**
**Last update:** 23 травня 2026 · v1.9.18
**Next planned update:** після hire of Founding Engineer
