# FORM

> **Тренер. У кишені.**
> AI personal trainer that watches your form, adapts to your sleep and HRV, and speaks like a 20-year coach — without the bullshit.

**Status:** v0.49.0 · pre-launch · Closed Beta planning Q3 2026 · Київ
**Public repo:** https://github.com/Rbit27/form
**Releases:** https://github.com/Rbit27/form/releases
**Founder:** George (Rbit27)

---

## What this repo contains

This is a **pre-product** project repository. There is no app code yet (mobile development begins з Founding Engineer hire). What you'll find:

- **16 HTML product surfaces** — marketing site + product mockups + investor one-pager + careers page + articles index + enterprise landing + pricing calculator
- **58+ markdown documents** — strategic, operational, content
- **21 blog post drafts** — content engine ready для launch
- **3 hiring JDs** — Founding Engineer, Design Lead, Sport Science Advisor
- **5 sample programs/nutrition plans** — sport-science discipline reference
- **24 specialized AI agents** — operating model embodied у `.claude/agents/`
- **Public decision log** — 30 documented decisions з reverse-cost

If you're здесь з first time, start з [STATUS.md](STATUS.md) для current snapshot.

---

## Quick navigation

### For investors → [investors.html](investors.html), [docs/PITCH.md](docs/PITCH.md), [docs/FINANCIALS.md](docs/FINANCIALS.md), [docs/DATA_ROOM.md](docs/DATA_ROOM.md)
### For journalists → [press.html](press.html), [docs/PRESS.md](docs/PRESS.md), [content/press-pitch-templates.md](content/press-pitch-templates.md)
### For potential hires → [jobs.html](jobs.html), [hiring/](hiring/), [docs/HIRING.md](docs/HIRING.md)
### For builders / advisors → [docs/TECHNICAL.md](docs/TECHNICAL.md), [docs/API.md](docs/API.md)
### Просто цікавий? → [about.html](about.html) і [content/](content/) blogs

---

## Repo structure

```
G:\\Клод\\form\\
│
├── index.html              ← marketing-сайт (open у browser)
├── about.html              ← manifesto + 4 red lines
├── beta.html               ← closed beta application
├── pricing.html            ← geo-priced з comparison table
├── faq.html                ← 30+ questions у 7 categories
├── settings.html           ← app settings mockup
├── screens.html            ← 5 app screen mockups (cloud)
├── onboarding.html         ← 8-screen onboarding (cloud)
├── press.html              ← HTML press kit (boilerplate, quotes, story angles)
├── workout-live.html       ← detailed workout mockup з CV
├── investors.html          ← investor one-pager (noindex — share directly)
├── jobs.html               ← careers page — FE + Design Lead open roles
├── blog.html               ← articles index (21 posts, filtered by category)
├── sitemap.xml             ← SEO sitemap
├── robots.txt              ← AI scraper blocked
│
├── VERSION                 ← single source of truth (current)
├── CHANGELOG.md            ← Keep-a-Changelog format
├── CONTRIBUTING.md         ← engineering standards
├── STATUS.md               ← project snapshot
├── README.md               ← this file
│
├── .claude/
│   └── agents/             ← 24 specialized AI agents
│
├── docs/                   ← 58+ strategic documents
│   ├── BRAND.md             — design tokens, voice, applications
│   ├── FLOWS.md             — user flows з ED-screening
│   ├── MARKETING.md         — GTM, channels, positioning
│   ├── APP_STORE.md         — App Store listing draft (cloud)
│   ├── PITCH.md             — 12-slide pitch + one-pager
│   ├── PERSONAS.md          — 4 detailed personas
│   ├── TECHNICAL.md         — architecture, CV-pipeline, latency
│   ├── FINANCIALS.md        — unit economics, 3-year P&L
│   ├── COMPETITIVE.md       — 8-product teardown
│   ├── RESEARCH.md          — 30-interview plan + script
│   ├── LEGAL.md             — privacy, ToS, GDPR framework
│   ├── METRICS.md           — NSM (W-ACSU), AARRR, dashboards
│   ├── HIRING.md            — first 3 roles, interview rubric
│   ├── PRESS.md             — press kit з founder bio
│   ├── INVESTOR_OUTREACH.md — target funds, templates
│   ├── API.md               — API design + Postgres schema
│   ├── EMAIL_SEQUENCE.md    — 18+ email templates
│   ├── BETA_PLAYBOOK.md     — TestFlight beta operations
│   ├── SECURITY.md          — threat model, 5-layer defense
│   ├── OKRS_2026.md         — Q3+Q4 OKRs з kill-criteria
│   ├── DATA_ROOM.md         — 4-tier investor materials
│   ├── LAUNCH_CHECKLIST.md  — P0/P1/P2 launch gate
│   ├── I18N.md              — localization plan
│   ├── DECISION_LOG.md      — 27 documented decisions
│   ├── ENGINEERING_RUNBOOK.md — on-call, deploy, monitoring
│   ├── INVESTOR_UPDATE_TEMPLATE.md — monthly template
│   ├── SUPPORT.md           — customer support playbook
│   ├── B2B_PLAYBOOK.md      — corporate wellness (post-PMF)
│   ├── CONTENT_CALENDAR.md  — 90-day editorial plan
│   ├── NEWSLETTER.md        — newsletter platform setup + first 2 post concepts
│   ├── DEPLOYMENT.md        — Cloudflare Pages deploy guide
│   ├── BETA_RECRUITING.md   — beta user recruiting: channels, scoring, timeline до 200 users
│   ├── COMMUNITY.md         — beta community setup (Slack + Discord)
│   ├── PRODUCT_SPEC.md      — iOS product spec для Founding Engineer (sprints, acceptance criteria)
│   ├── ANALYTICS_SETUP.md   — PostHog instrumentation guide: 65 events, W-ACSU, dashboards
│   ├── FIRST_30_DAYS.md     — 4-week execution playbook (domain → interviews → investor → FE hire)
│   ├── ASSUMPTIONS_REGISTER.md — 30 pre-research assumptions з confidence + pivot criteria
│   ├── ADVISORY_BOARD.md    — 4 advisor profiles, outreach templates, compensation, timeline
│   ├── HIRING_GUIDE.md      — platform-by-platform JD posting (Djinni/DOU/LinkedIn/Wellfound)
│   ├── SEED_NARRATIVE.md    — verbal pre-seed pitch script (6 blocks, objection playbook)
│   ├── DD_FAQ.md            — investor due diligence FAQ (7 blocks, 25 common DD questions)
│   ├── RESEARCH_SCREENER.md — user research recruiting toolkit (screener, DM templates, tracker, payment flow)
│   ├── BETA_FEEDBACK_PROTOCOL.md — beta feedback intake, taxonomy (P0–P3), synthesis rhythm, decision filter
│   ├── TWITTER_PLAYBOOK.md  — pre-launch X/Twitter growth strategy, build-in-public cadence
│   ├── INVESTOR_PIPELINE.md — investor CRM + relationship mgmt (7-stage pipeline, CRM schema, close mechanics)
│   ├── LINKEDIN_PLAYBOOK.md — LinkedIn strategy: profile, pillars, hiring, investor visibility, quick-start
│   ├── RETENTION_PLAYBOOK.md — D0/D7/D30/D90 windows, streaks, re-engagement, Pause>Cancel
│   ├── VICTOR_PROMPT_GUIDE.md — LLM implementation guide: 5-layer arch, context schema, 22 scenarios, ED-flag mode
│   ├── GROWTH_LOOPS.md      — 5 viral/organic loops, clinical-safety share constraints, FE priority, K-factor
│   └── FOUNDER.md           ← founder narrative · INTERNAL
│
├── content/                ← 35 blog posts + social
│   ├── post-01-hrv-is-lying.md
│   ├── post-02-what-good-form-means.md
│   ├── post-03-we-tested-8-ai-trainers.md
│   ├── post-04-sleep-and-strength.md
│   ├── post-05-building-fitness-from-ukraine.md
│   ├── post-06-30-user-interviews.md        ← placeholder; publishes after user interviews
│   ├── post-07-progressive-overload-myths.md
│   ├── post-08-why-no-free-tier.md
│   ├── post-09-building-victor-voice.md
│   ├── post-10-why-programs-assume-youre-always-ready.md
│   ├── post-11-deload-weeks.md
│   ├── post-12-technique-over-intensity.md
│   ├── post-13-rir-rpe-autoregulation.md
│   ├── post-14-sleep-deprived-training.md
│   ├── post-15-training-frequency.md
│   ├── post-16-ai-coach-vs-pt.md
│   ├── post-17-overtraining-vs-laziness.md
│   ├── post-18-rest-intervals.md
│   ├── post-19-warmup-protocols.md
│   ├── post-20-doms-muscle-soreness.md
│   ├── post-21-cv-vs-wearables.md           ← CV vs wearable-only arch (clinical-safety ✓)
│   ├── post-22-periodization.md
│   ├── post-23-velocity-based-training.md
│   ├── post-24-training-to-failure.md
│   ├── post-25-breathing-mechanics.md
│   ├── post-26-tendon-ligament-adaptation.md ← (clinical-safety ✓)
│   ├── post-27-training-age-vs-chronological-age.md
│   ├── post-28-progressive-overload-mechanics.md
│   ├── post-29-mind-muscle-connection.md
│   ├── post-30-eccentric-training.md
│   ├── post-31-unilateral-training.md
│   ├── post-32-cluster-sets.md
│   ├── post-33-energy-systems-for-strength.md ← (clinical-safety ✓)
│   ├── post-34-supersets-antagonist-pairs.md  ← (clinical-safety ✓)
│   └── post-35-tempo-training.md              ← (clinical-safety ✓)
│   ├── twitter-launch-thread.md
│   ├── linkedin-posts.md
│   ├── press-pitch-templates.md
│   ├── reddit-launch-posts.md
│   ├── newsletter-01.md          ← Issue 01 "Why we're building FORM" (draft, clinical-safety ✓)
│   ├── newsletter-02.md          ← Issue 02 "What FORM does/doesn't do" (draft, clinical-safety ✓)
│   └── newsletter-03.md          ← Issue 03 "Building Victor" (draft, clinical-safety ✓)
│
├── programs/               ← sport-science deliverables
│   ├── sample-strength-week.md
│   ├── sample-deload-week.md
│   └── sample-cycle-aware-week.md
│
├── nutrition/              ← nutrition-coach deliverables
│   ├── sample-3day-plan.md
│   └── sample-cutting-plan.md
│
├── legal/                  ← drafts pending counsel review
│   ├── privacy-policy-draft.md
│   └── terms-of-service-draft.md
│
└── hiring/                 ← ready-to-publish JDs
    ├── jd-founding-engineer-ios.md
    ├── jd-design-lead.md
    └── jd-sport-science-advisor.md
```

---

## The team (1 + 24)

One human founder. 24 specialized AI agents з distinct mandates.

### Product + Safety (6)
- **`product-strategist`** — scope, moat, honest claims · VETO на bloat
- **`product-manager`** — roadmap, prioritization, acceptance criteria
- **`sports-scientist`** — programming, HRV, periodization
- **`nutrition-coach`** — meal planning, anti-ED defaults
- **`clinical-safety`** — ED, mental health, injury triage · **HARD VETO**
- **`platform-engineer`** — CV, wearables, mobile reality check

### Design + Voice (4)
- **`design-craft`** — typography, motion, screen-level craft
- **`brand-system`** — tokens, identity system, applications
- **`brand-voice`** — Victor persona, UA/EN copy
- **`ux-flow`** — flows, IA, edge cases, onboarding

### Growth + Content (3)
- **`marketing-lead`** — positioning, channels, message hierarchy
- **`growth-lead`** — funnels, retention, NSM, A/B test discipline
- **`content-strategist`** — content marketing, editorial calendar

### Engineering + Infra (5)
- **`ml-engineer`** — CV pose pipeline, on-device models, MLOps
- **`devops-lead`** — CI/CD, Terraform, Kubernetes, observability
- **`security-engineer`** — threat model, secrets, OAuth, incident response
- **`enterprise-architect`** — SSO, RBAC, multi-tenant isolation
- **`data-engineer`** — analytics warehouse, ETL, cohort infrastructure

### Research + Process (3)
- **`research-lead`** — interviews, JTBD, persona validation
- **`process-keeper`** — decision log, dependency tracking
- **`customer-success`** — enterprise onboarding, retention, QBRs

### Quality + Compliance (3)
- **`qa-lead`** — test strategy, automation, release qualification
- **`qa-walker`** — manual user-flow walk after every ship (P0–P3)
- **`compliance-officer`** — SOC 2, GDPR Art. 9, HIPAA-adjacent · VETO на features з compliance risk

Full agent definitions у [.claude/agents/](.claude/agents/).

---

## Operating model

### Default pipeline для нову фічу

```
research-lead       → чи є попит?
product-strategist  → scope, moat
sports-scientist / nutrition-coach / platform-engineer
                    → content + technical
clinical-safety     → VETO або fix-list  ← critical gate
brand-voice         → копірайт (UA + EN)
clinical-safety     → ще раз на copy
design-craft        → UI mock
ux-flow             → flows + edge cases
brand-system        → token application
ship
```

### Default pipeline для маркетингу

```
marketing-lead      → positioning, audience
content-strategist  → content plan
brand-voice         → copy
clinical-safety     → anti-shame review
growth-lead         → funnel + metrics
design-craft        → visual production
```

---

## Як працювати

### Solo session

Pишеш «продовжуй» → я роблю наступну ітерацію.

### Cloud routine (active)

Sonnet 4.6 cloud-agent iterates every 2 години автоматично:
- https://claude.ai/code/routines/trig_01FYEjmyX5cwtRZtEWXWuLtf

Each iteration:
1. Pulls latest main
2. Reads VERSION + CHANGELOG + STATUS
3. Picks 1-3 roadmap items
4. Bumps version, updates CHANGELOG
5. Tags + pushes

### Versioning

`MAJOR.MINOR.PATCH` (semver-light):
- **PATCH** — кожна ітерація
- **MINOR** — нова фіча или розділ доків
- **MAJOR** — брейкінг чи pivot

Кожен bump = commit + git tag + CHANGELOG entry.

Стандарти у [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Принципи

1. **Жодних вигаданих цифр.** Цілі — окей. Результати — тільки коли є дані.
2. **`clinical-safety` має VETO на все шкідливе.** Не консультант — блок.
3. **Якщо фразу не можна сказати другу в гімі — її не каже Victor.**
4. **Не пропускай два поспіль** — і для продукту, і для команди.

Повні стандарти інженерної дисципліни — [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

## Метрики проєкту (як of v0.49.0)

| Metric | Value |
|---|---|
| Tagged releases | 60+ |
| HTML pages | 16 |
| Markdown documents | 65+ |
| Blog posts (drafts) | 21 |
| Social content assets | 7 (Twitter, LinkedIn, Reddit, press pitches, 3 newsletter issues) |
| Total lines committed | 38,000+ |
| AI agents defined | 24 |
| Documented decisions | 31 |

| Days від v0.1.0 до v0.10.0 | 2 |

---

## What's next

### Близько (M0-M1)
- [ ] Deploy marketing site на form.coach (Cloudflare Pages, ~85 min setup)
- [ ] Hire Founding Engineer (iOS)
- [ ] Hire Design Lead
- [ ] Conduct 30 user interviews
- [ ] Get sport-science advisor signed

### Mid (M2-M4)
- [ ] iOS app development (auth, today, workout, meals, chat)
- [ ] CV pipeline на 3 base lifts
- [ ] TestFlight closed beta з 200 users
- [ ] Counsel-reviewed legal docs
- [ ] First investor conversations

### Public launch (Q4 2026)
- [ ] App Store release (UA + Western EU)
- [ ] 8 blog posts published
- [ ] First 2,000 paid subscribers
- [ ] Seed round close ($1.5-2.5M target)

### Content engine roadmap (post-16+)
- [x] post-16: AI-тренер vs PT — чесний trade-off
- [x] post-17: Overtraining vs. laziness — як відрізнити перетренованість від лінощів
- [x] post-18: Відпочинок між підходами — PCr кінетика і Schoenfeld 2016
- [x] post-19: Warm-up протоколи — статичний vs динамічний стретчинг, PAP
- [x] post-20: Крепатура (DOMS) — EIMD, repeated bout effect, «без болю немає росту» міф
- [x] post-21: CV vs wearable-only — pose estimation, on-device inference, bar path tracking
- [x] post-22: Periodization — лінійна, DUP, блочна: матриця вибору за рівнем
- [x] post-23: Velocity Based Training — load-velocity relationship, VLT, без Gymaware
- [x] post-24: Training to failure — proximity-to-failure, три типи відмови, RIR 0 vs RIR 1–2
- [x] post-25: Breathing mechanics під навантаженням — Valsalva, bracing, IAP
- [x] post-26: Адаптація сухожиль і зв'язок — чому вона повільніша і чому це має значення
- [x] post-27: Training age vs. chronological age — адаптаційна ємність і сенс "починати пізно"
- [x] post-28: Progressive overload mechanics — SRA-крива, форми прогресії, чому лінійна ламається у intermediate
- [x] post-29: Mind-muscle connection — internal vs. external focus, EMG studies, коли IFA має сенс і коли заважає
- [x] post-30: Eccentric training — чому фаза опускання важливіша за підйом (mechanical tension, muscle damage, time under tension)
- [x] post-31: Unilateral training — коли і чому (strength asymmetry, core stability, transfer to sport)
- [x] post-32: Cluster sets — накопичення об'єму без кумулятивної втоми (inter-repetition rest, power maintenance)
- [x] post-33: Energy systems для силовика — ATP-PCr, glycolytic, oxidative: що реально важливо для вашого тренування
- [x] post-34: Supersets і antagonist pairs — time efficiency без компромісу з продуктивністю
- [x] post-35: Tempo-prescribed тренування — наука і практика контрольованої швидкості повторення
- [x] post-36: Детренованість — що реально втрачається і з якою швидкістю (нейром'язові vs структурні адаптації, міонуклеарна пам'ять, maintenance MED, матриця повернення)

Full roadmap у [STATUS.md](STATUS.md) і [docs/OKRS_2026.md](docs/OKRS_2026.md).

---

## How to read this repo

If you read **one file**: [STATUS.md](STATUS.md)
If you read **two**: + [docs/PITCH.md](docs/PITCH.md)
If you read **three**: + [docs/FINANCIALS.md](docs/FINANCIALS.md)

Якщо ти **журналіст**: [press.html](press.html) → [docs/PRESS.md](docs/PRESS.md)
Якщо ти **інвестор**: [investors.html](investors.html) → [docs/PITCH.md](docs/PITCH.md) → [docs/DATA_ROOM.md](docs/DATA_ROOM.md)
Якщо ти **потенційний hire**: [hiring/](hiring/) → [docs/FOUNDER.md](docs/FOUNDER.md)
Якщо ти **advisor**: [docs/TECHNICAL.md](docs/TECHNICAL.md) і [docs/METRICS.md](docs/METRICS.md)

---

## Contact

- **Email:** hello@form.coach (TBD on domain setup)
- **Press:** press@form.coach
- **Hiring:** hiring@form.coach
- **Investors:** invest@form.coach
- **Privacy:** privacy@form.coach
- **GitHub Issues:** https://github.com/Rbit27/form/issues

---

## Build-in-public commitment

Цей repo — public source-of-truth. Decision log є transparent. Доки розвиваються відкрито.

Те що ми **не** робимо public:
- User data (privacy)
- Confidential metrics post-launch (без consent)
- Negotiation details з investors
- Внутрішні founder notes ([docs/FOUNDER.md](docs/FOUNDER.md) есть у repo, but marked internal)

---

**Lead:** Claude (Opus 4.7, 1M context) · координує
**Cloud iterator:** Claude (Sonnet 4.6) · 2-hour cycle
**Owner:** Rbit27 (George)
**Last update:** 16 травня 2026 · v0.49.0
