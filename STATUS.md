# FORM · Project Status v0.17.0

**Date:** 14 травня 2026
**Version:** 0.17.0
**Phase:** Pre-launch · Closed Beta planning Q3 2026
**Repo:** https://github.com/Rbit27/form

---

## TL;DR (one paragraph)

FORM is at v0.17.0 with 33+ tagged releases, 12 functional HTML pages (marketing, product mockups, investor one-pager, and careers page at `jobs.html`), 44+ markdown documents covering every operational domain, 9 blog posts ready for distribution, 2 newsletter issues (drafts, clinical-safety cleared), a full advisory board recruitment playbook (ADVISORY_BOARD.md — 4 advisor profiles, outreach templates, trial protocol, timeline), a full press kit (HTML + markdown), hiring JDs ready to post with a designed careers page, a newsletter setup guide, 12 LinkedIn posts ready to publish, a complete beta community playbook, a beta recruiting playbook (channels, scoring, incentives, timeline to 200 users), a 30-assumption pre-research register (ASSUMPTIONS_REGISTER.md) with confidence levels and pivot criteria, sample programs and nutrition plans, full legal drafts, and a public GitHub repo with 27 documented decisions. Built solo by founder + AI agent team + 2-hour cloud iteration cycle. No code shipped yet — that begins with Founding Engineer hire post-Seed. Product spec for FE available: `docs/PRODUCT_SPEC.md`. Analytics instrumentation guide: `docs/ANALYTICS_SETUP.md`.

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
| DECISION_LOG.md | 27 documented decisions | process-keeper |
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
| twitter-launch-thread.md | Social | draft |
| linkedin-posts.md | Social | 12 posts ready |
| press-pitch-templates.md | Outreach | 5 templates |
| reddit-launch-posts.md | Social | 3 готові |
| newsletter-01.md | Newsletter | draft · clinical-safety PASS |
| newsletter-02.md | Newsletter | draft · clinical-safety PASS |

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

### Team agents (14 у `.claude/agents/`)

`product-strategist`, `sports-scientist`, `nutrition-coach`, `clinical-safety` (VETO), `platform-engineer`, `brand-voice`, `design-craft`, `ux-flow`, `brand-system`, `marketing-lead`, `growth-lead`, `content-strategist`, `research-lead`, `process-keeper`

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
- [x] Newsletter issues (2) drafted → `content/newsletter-01.md` + `content/newsletter-02.md` · clinical-safety PASS
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
Now (May 14, 2026)
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
├── Public GitHub repo з 31+ versions tagged
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

27 total documented decisions з reverse-cost assessment.

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
**Last update:** 14 травня 2026 · v0.15.0
**Next planned update:** після hire of Founding Engineer
