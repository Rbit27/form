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
- **65+ markdown documents** — strategic, operational, content
- **374 evidence-based blog posts** — sports-science content engine, clinical-safety reviewed
- **3 hiring JDs** — Founding Engineer, Design Lead, Sport Science Advisor
- **3 sample training programs** — sports-science discipline reference
- **24 specialized AI agents** — operating model embodied у `.claude/agents/`
- **Public decision log** — 31 documented decisions з reverse-cost

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
├── content/                ← 296 blog posts + social
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
│   ├── … post-36 … post-294 (спортивна наука · технічний корпус)
│   ├── post-295-strength-plateau-vs-life-plateau.md ← editorial · clinical-safety PASS
│   │
│   │   ── ROADMAP (наступні editorial теми) ──────────────────────────
│   ├── post-296: RPE-калібрування без тренера ✓ (виконано)
│   ├── post-297: «Достатньо добра» програма vs ідеальна ✓ (виконано)
│   ├── post-298: Як тренуватись при нестабільному розкладі ✓ (виконано)
│   ├── post-299: Сила без залу ✓ (виконано)
│   ├── post-300: Що ти зберігаєш під час детренування ✓ (виконано)
│   ├── post-364: Варіативність тренувальних показників ✓ (виконано · v3.12.0)
│   │
│   ├── post-365: Нейропластичність і силовий тренінг ✓ (виконано · v3.13.0)
│   ├── post-366: Сила хвату — функціональна анатомія і тренінг ✓ (виконано · v3.13.0)
│   ├── post-367: Мінімальна підтримуюча доза ✓ (виконано · v3.13.0)
│   ├── post-368: Порядок вправ у тренуванні ✓ (виконано · v3.14.0)
│   ├── post-369: Нейральна ефективність vs гіпертрофія ✓ (виконано · v3.15.0)
│   ├── post-370: Кофеїн і тренування ✓ (виконано · v3.15.0)
│   │
│   ├── post-371: Мікробіом і тренування — що каже доказова база ✓ (виконано · v3.16.0)
│   │
│   ├── post-372: Тренінг в умовах термального стресу — жара і холод ✓ (виконано · v3.17.0)
│   │
│   ├── post-373: Тренувальний стаж проти хронологічного віку після 40 ✓ (виконано · v3.17.0)
│   │
│   │   ── ROADMAP (наступні editorial теми · post-374+) ───────────────
│   ├── post-374: Чому тренувальні протоколи з соцмереж не працюють — анатомія вірусної некомпетентності ✓ (виконано · v3.18.0)
│   ├── post-375: Адаптація vs результат — коли тренування «працює» але прогрес стоїть ✓ (виконано · v3.19.0)
│   ├── post-376: Програмування для двозмінників — нестабільний графік без жертви прогресом ✓ (виконано · v3.19.0)
│   ├── post-377: Чому «прислухайся до тіла» — найгірша тренувальна порада — і що замість неї ✓ (виконано · v3.19.0)
│   │
│   ├── post-378: Як перевірити тренувальне дослідження за 5 хвилин — мінімальний фільтр якості ✓ (виконано · v3.20.0)
│   │
│   │   ── ROADMAP (наступні editorial теми · post-379+) ───────────────
│   ├── post-379: Суперпозиція втоми — коли сума двох «легких» стресів стає одним важким ✓ (виконано · v3.21.0)
│   ├── post-380: Чому новачок росте від будь-чого — а досвідчений атлет ні ✓ (виконано · v3.22.0)
│   ├── post-381: Як вибрати вагу для підходу: не «відчувай», а калібруй — практика RIR-якоря ✓ (виконано · v3.22.0)
│   ├── post-382: Мінімальна ефективна доза: скільки насправді потрібно для прогресу ✓ (виконано · v3.22.0)
│   │
│   │   ── ROADMAP (наступні editorial теми · post-383+) ───────────────
│   ├── post-383: Тренування при хронічному стресі — коригування об'єму і інтенсивності без зупинки прогресу ✓ (виконано · v3.23.0)
│   ├── post-384: Зовнішній фокус уваги і силовий тренінг: чому «штовхай підлогу» працює краще за «напружуй квадрицепс» ✓ (виконано · v3.24.0)
│   ├── post-385: Ефективність vs ефектність у тренуванні — чому той самий результат досягається з меншим об'ємом ✓ (виконано · v3.25.0)
│   ├── post-386: Тренінг в умовах часового дефіциту — науково обґрунтований 30-хвилинний протокол ✓ (виконано · v3.26.0)
│   ├── post-387: Крепатура — що відбувається в м'язах і чому вона не є маркером ефективного тренування ✓ (виконано · v3.26.0)
│   ├── post-388: Прогресія новачка: скільки часу насправді є «новачкового вікна» і як не витратити його ✓ (виконано · v3.26.0)
│   ├── post-389: Силова асиметрія — коли різниця між лівою і правою — проблема, а коли — норма ✓ (виконано · v3.26.0)
│   ├── post-390: Тренування під час хвороби — протокол «шия і вище» і його обмеження ✓ (виконано · v3.26.0)
│   │
│   │   ── ROADMAP (наступні editorial теми · post-391+) ───────────────
│   ├── post-391: Як читати дослідження про тренування: p-value, effect size і ecological validity — без PhD ✓ (виконано · v3.27.0)
│   ├── post-392: Specificity vs transferability — тренування для здоров'я vs спортивний результат ✓ (виконано · v3.28.0)
│   ├── post-393: Аеробна база у силовому спортсмені — навіщо кардіо, якщо мета — м'язи ✓ (виконано · v3.28.0)
│   ├── post-394: Психологічна втома і фізичний ліміт — чому мозок зупиняє тіло до відмови м'язів ✓ (виконано · v3.28.0)
│   ├── post-395: Plateau breaking — чому прогрес зупиняється і сім доказових виходів з плато ✓ (виконано · v3.28.0)
│   │
│   │   ── ROADMAP (наступні editorial теми · post-396+) ───────────────
│   ├── post-396: Тренування і менструальний цикл — мета-аналітична картина і питання, яке з неї виростає ✓ (виконано · v3.29.0)
│   │
│   ├── post-397: Силовий тренінг і кардіоваскулярне здоров'я — що насправді відбувається з серцем ✓ (виконано · v3.30.0)
│   ├── post-398: Прогресивне навантаження після 50 — як змінюється оптимальна програма ✓ (виконано · v3.30.0)
│   ├── post-399: Інтуїтивне тренування vs структурована програма — коли кожен підхід доречний ✓ (виконано · v3.30.0)
│   ├── post-400: Варіабельність серцевого ритму як інструмент самокоучингу — що вимірювати і що ігнорувати ✓ (виконано · v3.30.0)
│   │
│   │   ── ROADMAP (наступні editorial теми · post-401+) ───────────────
│   ├── post-401: Чому ти не відчуваєш м'язи, які тренуєш — м'язово-нервовий зв'язок і фокус уваги ✓ (виконано · v3.31.0)
│   ├── post-402: Тренінг при хронічному болю у спині — що допомагає, що не допомагає, що шкодить ✓ (виконано · v3.33.0 · clinical-safety APPROVED)
│   ├── post-403: Силовий тренінг і сон — двонаправлений зв'язок і практичний протокол ✓ (виконано · v3.33.0)
│   ├── post-404: Як скласти першу програму самостійно — мінімальний фреймворк без надмірного ускладнення ✓ (виконано · v3.33.0)
│   │
│   │   ── ROADMAP (наступні editorial теми · post-405+) ───────────────
│   ├── post-405: Прогрес без зовнішнього навантаження — сім змінних, які замінюють залізо ✓ (виконано · v3.34.0 · clinical-safety NOT REQUIRED)
│   ├── post-406: Чому ти гірше тренуєшся після поганого сну — механіка, не мотивація
│   ├── post-407: Силова витривалість проти максимальної сили — що розвивати і коли
│   ├── post-408: Мобільність у силовому тренінгу — не розтяжка, а контрольований діапазон
│   ├── post-409: Як повернутись після 3–6 місяців перерви — що зберіглось і що ні
│   ├── post-410: Тренування без тренера — як самостійно діагностувати технічні помилки
│   ├── post-411: Анатомічна варіативність і вибір вправ — чому «правильна техніка» різна для різних людей
│   ├── post-412: Надлишкова варіація — як часта зміна вправ саботує прогрес ✓ (виконано · v3.38.0 · clinical-safety NOT REQUIRED)
│   ├── post-413: Тренування після 40 — де вікові обмеження реальні, де вигадані ✓ (виконано · v3.38.0 · clinical-safety NOT REQUIRED)
│   ├── post-414: Сила та довголіття — механізми, не кореляції ✓ (виконано · v3.38.0 · clinical-safety NOT REQUIRED)
│   └── post-415: Програмування для батьків — хронічне недосипання і нестабільний розклад ✓ (виконано · v3.38.0 · clinical-safety NOT REQUIRED)
│   │
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
| Blog posts | 270 |
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
- [x] post-37: BFR-тренінг — гіпертрофія при 20% від 1RM і чому це не лайфхак (механізм оклюзії, три гіпертрофічні шляхи, протоколи, протипоказання)
- [x] post-38: Ізометричні вправи — RFD, кутова специфічність і чому 6 секунд можуть пробити плато (overcoming vs yielding, нейральна адаптація, sticking point protocols, сухожильна адаптація)
- [x] post-39: Рухова одиниця і принцип Хенемана — чому м'яз не скорочується весь одразу (size principle, рекрутинг vs rate coding, тип волокон, наслідки для програмування)
- [x] post-40: Механізми гіпертрофії — механічна напруга, метаболічний стрес і EIMD: модель Шенфельда 2010 у 2026 (stretch-mediated hypertrophy, тітін, перегляд ролі метаболічного стресу)
- [x] post-41: Grip strength як обмежувальний фактор — хват у дедлафті і тягах: три типи хвату, strop debate, протокол тренування
- [x] post-42: Velocity-Based Training — load-velocity профіль, зони MCV (0.15–1.0+ м/с), VL% як критерій зупинки сету, протокол без пристроїв
- [x] post-43: Post-Activation Potentiation / PAPE — MLC-фосфорилювання, вікно 4–8 хв, три протоколи, варіабельність відповіді
- [x] post-44: Heat і cold exposure у відновленні — CWI vs mTOR (Roberts 2015), сауна і серцево-судинна адаптація (Laukkanen), тайминг, розбір хайпу
- [x] post-45: ЦНС-втома проти периферійної — два механізми, різна кінетика відновлення, наслідки для програмування і деавантажу
- [x] post-46: Гормони і тренінг — тестостерон, кортизол, GH: що реально вимірюване і що є маркетинговим шумом
- [x] post-47: Кардіо і силова: concurrent training interference — AMPK vs mTOR молекулярний конфлікт, Hickson (1980), Wilson meta-analysis (2012), реальні модифікатори і практична 4+2 структура тижня
- [x] post-48: Тренувальний об'єм — MEV/MAV/MRV механізм, функціональне vs нефункціональне надперевантаження, хвиляста надорганізація, мінімальна ефективна доза (Schoenfeld 2017)
- [x] post-49: Rate of Force Development — RFD (N/s), часове вікно руху, нейральні драйвери (high-frequency doublets, rate coding), 4 методи тренування, FORM velocity tracking
- [x] post-50: Пропріоцепція і баланс у силовому тренінгу — м'язові веретена, GTO, joint position sense, нестабільні поверхні розбір (Anderson & Behm 2005), де пропріоцепція реально тренується у силовому контексті
- [x] post-51: Дихання і внутрішньочеревний тиск у силовому тренінгу — механіка bracing, IAP, маневр Вальсальви, 360° bracing vs hollowing (McGill & Cholewicki 2001; Grenier & McGill 2007), протоколи за інтенсивністю, пояс і його умови
- [x] post-52: Типи м'язових волокон і тренінг — Type I / IIa / IIx фізіологія, MHC-ізоформи, генетичний компонент без детермінізму, Mitchell (2012), програмування за rep-діапазонами, FORM velocity як proxy
- [x] post-53: Програмування для занятого: MEV-режим, 3-денний тиждень і нестабільний розклад — Schoenfeld (2016), Ralston (2017), Bickel (2011) maintenance, режими прогресу/обслуговування, чому пропущена сесія не переноситься
- [x] post-54: MEV, MAV, MRV — три орієнтири об'єму, що насправді керують гіпертрофією: дозо-відповідні landmarks Israetel/RP, Schoenfeld (2017) dose-response meta-analysis, мезоцикл від MEV до MRV, практичні оцінки за групами м'язів
- [x] post-55: Тренувальні сплити: доказова матриця вибору — full body vs верх/низ vs PPL за тренувальним стажем, розкладом і цілями. Schoenfeld (2016), Ralston (2017), Colquhoun (2018). Матриця вибору для трьох типових сценаріїв
- [x] post-56: Моделі periodization — лінійна, хвильова і блокова: LP, DUP, блочна з матрицею вибору за стажем і ціллю; Rhea et al. (2002), Prestes et al. (2009), residual training effects
- [x] post-57: Тренінг після 40: саркопенія, анаболічна резистентність і програмування — Wilkinson et al. (2015) mTORC1 МПС −30%, Fragala et al. (2019) NSCA, Peterson et al. (2011) приріст у 60–70
- [x] post-58: Mind-Muscle Connection — constrained action hypothesis Wulf et al. (2001), Calatayud et al. (2016) EMG internal vs external focus, Snyder & Fry (2012); практичний split за типом руху
- [x] post-59: Zone 2 cardio — LT1 physiology, Seiler 80/20 context (10–15 год/тиждень endurance), interference Wilson et al. (2012), HIIT equivalence Gibala/Burgomaster; хто реально потребує, як дозувати
- [x] post-60: Deload protocols — supercompensation model, 4 types (volume/intensity/full/frequency), reactive vs proactive triggers, practical implementation examples (Meeusen 2013, Schoenfeld, Helms)
- [x] post-61: Overreaching vs overtraining: операційна діагностика — NFO, FO, OTS, Meeusen et al. 2013, HRV-тренд, RPE-drift, decision tree, recovery by state
- [x] post-62: Velocity-Based Training — load-velocity profile (González-Badillo & Sánchez-Medina 2010), velocity loss thresholds (Pareja-Blanco 2017), 1RM estimation accuracy, FORM CV-as-proxy; clinical-safety: not-required
- [x] post-63: Жінки і силовий тренінг — фолікулярна vs лютеальна фаза, estrogen і м'язова адаптація; McNulty et al. (2020) ~1.8% effect size; hormonal contraception addressed; clinical-safety PASS WITH CONDITIONS
- [x] post-64: Плайометрика для силовиків — SSC механіка, RFD, ground contact time, foot contacts прогресія, tendon adaptation lag; clinical-safety PASS
- [x] post-65: Кофеїн і силовий тренінг — adenosine antagonism mechanism, Grgic et al. 2018 (+3–6% strength), 3–6 mg/kg dose-response, CYP1A2 polymorphism, tolerance cycling, sleep interference; clinical-safety PASS
- [x] post-66: HRV для силових атлетів — RMSSD, 7–14-денний тренд, Flatt & Esco (2016), Nuuttila (2022), протокол вимірювання і volume-adjustment thresholds; clinical-safety PASS
- [x] post-67: Foam rolling і міофасціальний реліз — механорецептори vs фасціальний реліз міф, Cheatham (2015) ROM meta-analysis, Wiewelhove (2019) DOMS meta-analysis, 5-хвилинний протокол; clinical-safety PASS
- [x] post-68: Тренінг у гарячому і холодному середовищі — heat acclimation, cold exposure, термоадаптація для силових атлетів; Racinais & Oksa (2010), Roberts et al. (2015) CWI adaptation blunting, Nielsen (1993) heat acclimation; clinical-safety PASS
- [x] post-69: Creatine — механізм PCr ресинтезу, силові vs гіпертрофія ефекти, протокол завантаження, коли не потрібне; Rawson & Volek (2003), Kreider et al. (2017) ISSN; clinical-safety PASS
- [x] post-70: Training Volume: MEV, MAV, MRV — dose-response landmarks; Schoenfeld 2017 meta-analysis; individual variation as central thesis
- [x] post-71: Allostatic load і тренувальна готовність — McEwen (1998) allostatic load framework, HPA axis, HRV як proxy, чому фіксований об'єм ігнорує total load; clinical-safety PASS
- [x] post-72: Загальний стрес і тренувальна ємність — сон-депривація (IGF-1), когнітивний стрес (HPA), ситуативні стресори (прозапальні цитокіни), практичний стрес-аудит і корекція MRV; clinical-safety PASS
- [ ] post-73: Protein timing revisited — anabolic window myth vs. daily distribution logic, Schoenfeld & Aragon (2013, 2018) ⚠️ clinical-safety required before publish (food content)
- [ ] post-74: Grip strength і longevity — Bohannon 2019 meta-analysis, grip як proxy загального здоров'я, як тренувати
- [ ] post-75: Single-leg training для двобічних атлетів — bilateral deficit, Botton et al. 2016, коли і як додавати унілатеральну роботу
- [ ] post-76: Стретчинг і м'язова гнучкість — статичний vs динамічний, PNF, loaded progressive stretching; що насправді змінює ROM (нейральна vs структурна)
- [x] post-77: Blood Flow Restriction (BFR) — механізм (венозна оклюзія, AMPK, рекрутування Type II), протокол Abe/Loenneke, коли BFR > важкого навантаження; clinical-safety PASS
- [x] post-78: Concurrent Training — interference effect (Hickson 1980), AMPK vs mTOR молекулярний конфлікт, режим/об'єм/порядок сесій, практична організація тижня; clinical-safety PASS
- [x] post-79: Принцип SAID — специфічна адаптація до накладених навантажень: три виміри (нейром'язова, метаболічна, морфологічна), перенос тренування, exercise selection logic, техніка як нейром'язова інвестиція; clinical-safety PASS
- [x] post-80: Авторегуляція: RPE і RIR — 10-бальна шкала Tuchscherer, RIR як зворотній бік RPE, три проблеми відсоткового програмування (варіація 1RM ±5–8%, контекст, застарілий max), Zourdos et al. (2016) JSCR, Helms et al. (2016) IJSPP, Colquhoun et al. (2017); clinical-safety PASS — ⚠️ попередній roadmap-запис (натщесерце) отримав VETO від clinical-safety
- [x] post-81: Нейральна vs структурна адаптація — Moritani & deVries (1979) класичний EMG-експеримент, три нейральні механізми (рекрутування, rate coding, координація), Sale (1988), Folland & Williams (2007) огляд, часова шкала нейральної/структурної фази, м'язова пам'ять (epigenetic Seaborne 2018), наслідки для програмування новачків і повернення після паузи; clinical-safety PASS — ⚠️ попередній roadmap-запис (Omega-3) отримав VETO від clinical-safety
- [x] post-82: Розвантаження (deload protocols) — Banister двофакторна модель fitness-fatigue (1975), три рівні стомлення (периферичне, центральне, сполучнотканинне), FOR/NFOR/OTS розмежування (Meeusen et al. 2013), чотири типи деload (volume/intensity/комбінований/skill), реактивний vs проактивний підхід, RPE-drift як основний trigger, тривалість 5–7 діб, суперкомпенсація post-deload; clinical-safety PASS ⚠️ витіснив planned «Training load monitoring / ACWR» — зберегти як post-84 або post-86
- [x] post-83: Типи м'язових волокон — трьохтипова класифікація (Type I / IIa / IIx; IIb у людини не виражений — Schiaffino & Reggiani 2011), Henneman Size Principle (1965): суворий порядок рекрутування I→IIa→IIx, IIx при ≥80–85% 1RM або глибокому стомленні (Morton et al. 2016), розподіл і варіабельність (Gollnick et al. 1972), конвертація IIx→IIa реальна (Staron et al. 1991), гіпертрофійний відгук Type I при наближенні до відмови (Mitchell et al. 2012), міф «правильних rep ranges» vs мета-аналіз Schoenfeld (2017); clinical-safety PASS ⚠️ витіснив planned «Calisthenics vs weighted training» — зберегти як post-85
- [x] post-84: Motor learning і техніка — Fitts & Posner (1967) три стадії (cognitive/associative/autonomous), нейронна архітектура кожної стадії (prefrontal → striatum/cerebellum → моторна кора), Shea & Morgan (1979) contextual interference effect (blocked vs random practice), Schmidt (1975) schema theory і variability of practice hypothesis, Wulf & Prinz (2001) зовнішній vs внутрішній фокус уваги (мета-аналіз — зовнішній стабільно перевершує), Walker et al. (2002) Neuron — сон +20% без практики, практика feedback-частоти (Salmoni et al. 1984 guidance effect); clinical-safety PASS ⚠️ roadmap-запис post-84 (bilateral deficit) перенесений — Botton et al. покриті у post-74 і post-31
- [x] post-85: Rate of Force Development (RFD) — RFD як dF/dt у вікні 0–200 мс (Aagaard et al. 2002); нейральна база: motor unit recruitment rate, high-frequency doublets (Van Cutsem et al. 1998); чому абсолютна сила не конвертується в потужність без velocity training; 4 методи розвитку RFD: intent-based training, ballistic/plyometric, accommodating resistance, contrast training (PAP — Tillin & Bishop 2009); practical programming; clinical-safety PASS ⚠️ витіснив planned «bilateral deficit programming» — Botton et al. покриті у post-74 і post-31
- [x] post-86: Моделі програмування: лінійна vs DUP vs блокова — GAS (Selye 1950) + SAID як фізіологічна база periodization; LP (Zatsiorsky): максимальна простота для новачків, вичерпується за 12–16 тижнів; DUP (Rhea et al. 2002, JSCR): варіація всередині тижня, Harries et al. 2015 мета-аналіз; Block (Issurin 2008): концентрація адаптацій, competing adaptations (Wilson et al. 2012); Victor-фільтр: три запитання (стаж, ціль, частота); clinical-safety PASS ⚠️ витіснив planned «stretch-mediated hypertrophy programming» — stretch mechanisms покриті у post-40 і post-73
- [ ] post-87: Тренінг натщесерце — fed vs fasted state, fat oxidation myth, Schoenfeld (2014) ⚠️ clinical-safety VETO BLOCKED · food/eating patterns — не писати без повного CS review
- [ ] post-88: Omega-3 і тренувальна адаптація — Smith et al. (2011) MPS, EPA:DHA ratio ⚠️ clinical-safety VETO BLOCKED · supplement/food content — не писати без повного CS review
- [x] post-89: ACWR і моніторинг навантаження — Gabbett (2016) BJSM: U-подібна крива ризику, sweet spot 0.8–1.3; session-RPE (Foster et al. 2001); математична критика відношення: Impellizzeri et al. (2019), Murray et al. (2017); EWMA як точніша альтернатива rolling average; external vs internal load; Victor multi-layer readiness model (load trend + RPE-drift + суб'єктивна готовність); логіка повернення після паузи; clinical-safety PASS
- [x] post-90: Calisthenics vs weighted training — Calatayud et al. (2015) EMG-еквівалентність; механізми прогресії власною вагою (leverage, односторонні варіанти, темп, діапазон); структурні обмеження: максимальна сила, гіпертрофія н/ч тіла; Ralston et al. (2018) трансфер через спільні м'язи і вектори; гібридна стратегія: базові підйоми зі штангою + допоміжна калістеніка; Victor-фільтр: прогресія, відповідність меті, послідовність; clinical-safety PASS
- [x] post-91: Time under tension — TUT як змінна: роль механічного навантаження та метаболічного стресу, Burd et al. (2012) slow vs fast tempo MPS; концентрична vs ексцентрична vs ізометрична фаза; коли TUT має значення і коли ні; clinical-safety PASS
- [x] post-92: Concurrent training interference — AMPK/mTOR молекулярний конфлікт; Wilson et al. (2012) мета-аналіз; вплив порядку, об'єму і типу кардіо; блокова логіка для аматора; clinical-safety PASS
- [x] post-93: Мобільність vs гнучкість — пасивний vs активний ROM; Behm & Chaouachi (2011); нейронна vs структурна лімітація; end-range isometrics і loaded progressive end-range training; clinical-safety PASS
- [x] post-97: Loaded carries — farmer's walk, yoke, suitcase carry: механізм gripstrength+core+gait, anti-lateral flexion під локомоцію, McGill et al. (2013), Bohannon (2019); протоколи і місце у тижні; clinical-safety PASS
- [x] post-98: Accommodating resistance — ланцюги і гуми: механізм variable resistance, Westside context, де і коли виправданий для аматора (sticking point overload, peak-force extension)
- [x] post-99: Sticking points в базових вправах — механіка точки відмови у присіданні, жимі, становій; слабкий ланцюг vs технічна помилка vs навантаження; targeted accessory selection
- [x] post-100: Повторні максимуми vs тренування в діапазонах — чому 1RM тест не потрібен більшості атлетів; E1RM формули (Epley, Brzycki, Lombardi), точність при >5 reps, velocity-based estimation як альтернатива; clinical-safety low-risk
- [x] post-101: Сон і відновлення: механізми, не поради — GH пульс у slow-wave sleep, IGF-1 сигналінг, Sleep deprivation ≥1 ніч: –11% тестостерон (Leproult & Van Cauter 2011), протокол sleep hygiene з evidence рівнями; clinical-safety PASS
- [x] post-102: Тренування у спеку і холод — performance modifiers без гіпотермії/гіпертермії тригерів; Jensen (2013) heat effect on strength; cold muscle performance reduction; практичні адаптації для тренування на вулиці
- [x] post-103: Readiness diagnostics — чотири виміри готовності (нейром'язова/системна/локальна/когнітивна), матриця рішень, Plews et al. 2013, Pageaux & Lepers 2018; clinical-safety PASS ⚠️ витіснив planned «Жіночий підхід» — перенести як post-106
- [x] post-104: Дегідрація і продуктивність — Savoie et al. (2015): −2% рідини → −2–3% сили; González-Alonso et al. (1997); ACSM колір сечі; гіпонатріємія попередження; протокол до/під час; clinical-safety PASS WITH CONDITIONS
- [x] post-105: Grip strength як маркер здоров'я — Bohannon (2019) мета-аналіз; Leong et al. (2015) PURE (n=139 691); Celis-Morales (2018) UK Biobank; нормативи NHANES; 4 методи тренування; clinical-safety not required
- [x] post-106: Жіночий підхід до силового: що відрізняється анатомічно — Q-angle myth (Collins 2009), estrogen і tendon laxity (Lowe 2017), ACL base-rate context + neuromuscular trainability (Myklebust 2003), recovery differences (Hicks 2001), cycle-phase periodization як contested area; clinical-safety PASS WITH CONDITIONS 2026-05-24
- [x] post-107: Три механізми гіпертрофії — ієрархія між mechanical tension, metabolic stress та muscle damage; Schoenfeld (2010, 2013) systematic framework; Wackerhage et al. (2019) mTORC1 signaling via FAK і integrin; Burd (2012) і Mitchell (2012) дані про повне vs часткове навантаження; repeated bout effect як контраргумент «ушкодження = зростання»; практичні наслідки: повний ROM, RIR 0–2, без гонитви за DOMS; clinical-safety PASS
- [x] post-108: Гіпертрофія при розтягнутих положеннях — Maeo et al. (2021) lengthened vs shortened partials на підколінних; Pedrosa et al. (2022) квадрицепс; Kassiano et al. (2023) мета-аналіз: lengthened position > shortened partials; titin mechanism (Ca²⁺ sensitization, sarcomerogenesis); exercise selection per muscle group (RDL, incline curl, overhead extension, full squat); Schoenfeld & Grgic (2020) ROM review; clinical-safety PASS
- [x] post-109: Крива сила-швидкість — Hill (1938) гіперболічне рівняння; пік потужності при 30–45% 1RM; explosive strength deficit (ESD, Zatsiorsky); F-V профілювання за Samozino et al. (2012, 2014) і Morin & Samozino (2016); силовий vs швидкісний дефіцит; Cormie et al. (2011) оптимальне навантаження для потужності; Haff & Nimphius (2012) принципи; Skelton et al. (1994) RFD і вік; блокова логіка; clinical-safety PASS
- [x] post-110: Архітектура м'яза — кут пір'їстості (θ) і cos(θ) передача сили; довжина пучка (FL) і V_max через послідовні саркомери; PCSA як реальний корелят ізометричної сили (Wickiewicz 1983); чому литки важко ростуть: gastrocnemius θ ≈ 25–30°, soleus FL ≈ 25–35 мм, 70–80% Type I у soleus (Johnson 1973); підколінні і довгий FL (semitendinosus FL ≈ 80–100 мм) — чому RDL і nordic curl ефективніше за leg curl; неоднорідна гіпертрофія квадрицепса (rectus femoris vs vastus lateralis); адаптація архітектури (Kawakami 1993: θ ↑ при гіпертрофії; ексцентричний тренінг → FL ↑); регіональна гіпертрофія і вибір ROM (Wakahara 2013); clinical-safety PASS
- [x] post-111: Post-Activation Potentiation (PAP) — механізм фосфорилювання регуляторних легких ланцюгів міозину (RLC) через MLCK і staircase effect (Sale 2002); вікно 4–12 хвилин між активатором і балістичним рухом (Tillin & Bishop 2009); умови спрацьовування: силовий поріг (≥ 1.5 BW squat), відповідність патерну, ненакопичена втома; complex training і contrast training; практичне програмування (3–5 RM активатор → 4–12 хв пауза → балістичний рух); clinical-safety PASS
- [x] post-112: RPE vs %1RM і авторегуляція — ефективний 1RM варіюється 5–15% залежно від сну, стресу, гідратації (Atkinson & Reilly 1996); шкала Борга 6–20 і модифікована 0–10 для силового тренінгу (Tuchscherer → Helms); RIR (Reps in Reserve) як альтернативна мова; валідність RPE в досвідчених атлетів: Zourdos et al. (2016), Helms et al. (2016, 2017); похибка у початківців 2–3 повторення; % як стартова точка, RPE як механізм щоденної корекції; обмеження: потребує ~2 роки досвіду для точної калібровки; clinical-safety PASS
- [x] post-113: Ексцентрична сила — Katz (1939) F_eccentric > F_isometric > F_concentric; ексцентричний max +20–50% vs концентричний; Roig et al. (2009) мета-аналіз: ексцентрично-акцентоване тренування → більша гіпертрофія (ES 0.79 vs 0.52); Franchi et al. (2014) морфологія: ексцентрик → дистальна гіпертрофія і довші фасцикули; Nordic Hamstring Exercise: Har-Nir et al. (2022) −51% травм підколінних; flywheel: Vicens-Bordas et al. (2018) ексцентрична сила +10.2% vs +3.8%; нейральна специфічність: Duchateau & Enoka (2008) — менший рекрутинг при вищій силі; Repeated Bout Effect і дозування першої ексцентричної сесії; clinical-safety PASS
- [x] post-94: Tendon adaptation vs muscle adaptation — різна часова шкала (м'язи: 2–4 тижні, сухожилля: 8–12 тижні); Bohm et al. (2015) tendon stiffness і RFD; чому бігуни і важкоатлети отримують сухожильні травми після стрибків навантаження, які м'язи вже переносять; практичні наслідки для програмування; clinical-safety PASS
- [x] post-95: Пікова сила vs силова витривалість — дві різні нейром'язові цілі; чому training for both одночасно може давати interference effect; блокова логіка для аматорів з обмеженим часом; clinical-safety PASS
- [x] post-96: Warming up vs activation — різниця між підвищенням температури тканин (ефективний warmup) і «активацією» (нейром'язовий праймінг); що реально робить PAP і для кого він релевантний; Fradkin et al. (2010) огляд; clinical-safety PASS

- [x] post-114: Обсяг тренувань і гіпертрофія — Krieger (2010) мета-аналіз multiple vs single sets; Schoenfeld/Ogborn/Krieger (2017) доза-відповідь (10+ сетів/м'яз/тиждень); Radaelli et al. (2015) 6-місячне RCT (5 > 3 > 1 сет); MEV/MAV/MRV операційна модель (Israetel); ознаки перевищення MRV; зворотній зв'язок інтенсивності і обсягу; clinical-safety PASS
- [x] post-115: Частота тренувань і гіпертрофія — Schoenfeld/Ogborn/Krieger (2016) 2x > 1x (ES 0.37 vs 0.14, обсяг неурівняний); Ralston et al. (2017) при рівному обсязі ефект частоти зникає; McLester et al. (2000) деградація якості пізніх сетів; Colquhoun et al. (2018) 6x = 3x; вікно MPS ~24–48h; частота як механізм розподілу обсягу; clinical-safety PASS
- [x] post-116: Варіативність гіпертрофічної відповіді — Hubal et al. (2005) 585 учасників: CSA 0–+59%, сила 0–+250% при однаковому протоколі; Bamman et al. (2007) кластери: ~26% extreme responders, ~23% non-responders; Petrella et al. (2008) базальний рівень satellite cells як передбачник; ACTN3 R577X і Type II волокна; міостатин (Schuelke 2004); Churchward-Venne et al. (2015) non-responders на низькому обсязі → responders при вищому; clinical-safety PASS
- [x] post-117: Модель Баністера і dual-factor theory — Banister et al. (1975) impulse-response model: fitness component τ₁ ≈ 42 дні і fatigue component τ₂ ≈ 13 днів; Morton et al. (1990) верифікація на плавцях, k₂ > k₁; Busso (2003) нелінійна версія і індивідуальна варіативність параметрів; Bosquet et al. (2007) meta-analysis: tapering +2–3% performance, оптимум –41–60% обсягу за 8–14 днів; практична структура accumulation/intensification/deload/peak; чому оцінювати форму під час пікового навантаження хибно; реактивний деавантаж і HRV як proxy для h(t) (Plews 2013); clinical-safety PASS
- [x] post-118: Підводка і розвантаження: доказова база деловдів — три типи деловду (зниження об'єму vs інтенсивності vs повний відпочинок); Ogasawara et al. (2013) 3-тижнева перерва не знижує гіпертрофію; зниження об'єму −40–50% при збереженні інтенсивності 85–95% — найбільш підтриманий варіант; проактивний (кожні 4–6 тижнів) vs реактивний деловд; маркери накопиченої втоми; clinical-safety: PASS
- [x] post-119: Континуум сила-гіпертрофія: що насправді визначає адаптацію — Schoenfeld et al. (2017): 25–35 vs 8–12 повторень → статистично еквівалентна гіпертрофія при роботі до відмови; Lasevicius et al. (2018); Schoenfeld & Grgic (2019) мета-аналіз; діапазон ~6–30 повторень ефективний при RIR 0–4; три механізми: механічна напруга (пріоритет), метаболічний стрес, пошкодження м'язів; clinical-safety: PASS
- [x] post-120: Міофібрилярна vs саркоплазматична гіпертрофія — Schoenfeld (2010) критика, Haun et al. (2019) PLOS ONE прямий аналіз фракцій (глікоген і саркоплазматичні білки при high-rep), що реально доведено vs гіпотеза, таблиця реальних відмінностей між діапазонами, відкриті питання науки; clinical-safety: NOT REQUIRED
- [x] post-121: Додавання саркомерів і адаптація довжини м'яза — Lynn & Morgan (1994) sarcomerogenesis; ексцентричний тренінг → збільшення оптимального кута; Nordic Hamstring і довжина фасцикул (Mjøsund 2021); чому довжина м'яза при піковій силі змінюється з тренінгом; clinical-safety: not required
- [x] post-122: Кут пір'їстості (pennation angle) і сила — адаптація θ при гіпертрофії (Kawakami 1993); компроміс між силою і швидкістю на рівні архітектури; чому газтрокнемій важко гіпертрофується; PCSA і ізометрична сила; clinical-safety: not required
- [x] post-123: Крос-едукація — тренуєш одну кінцівку, адаптується й інша — Scripture et al. (1894) перше описання; Carroll et al. (2006) meta-analysis: ~8% cross-limb strength transfer; ipsilateral corticospinal pathway і зміни motor cortex excitability (Zijdewind et al.); TMS: underused motor units активуються контралатерально; реабілітаційне застосування; обмеження ефекту; clinical-safety: NOT REQUIRED
- [x] post-124: Відпочинок між підходами і гіпертрофія — Schoenfeld et al. (2016) JSCR: 3 хв vs 1 хв → biceps +12.7% vs +5.1%, квадрицепс +3.6% vs +2.4%; кінетика PCr (Harris 1976, Greenhaff 1994): 70% відновлення за 1 хв, 92% за 3 хв; гормональна гіпотеза спростована (West et al. 2010 FASEB, Ahtiainen 2005); якість об'єму як реальний механізм; практика: компаунд ≥2–3 хв, ізоляція 60–90 с; клінічний: NOT REQUIRED
- [x] post-125: Серце атлета — Morgranroth (1975) ексцентрична vs концентрична гіпертрофія; MacDougall et al. (1985) пікові 370/360 мм рт. ст. при жимі ногами; Haykowsky (2000) мета-аналіз ехо-даних; ЕКГ-«аномалії» за ESC 2010 (Pelliccia criteria); диференційна діагностика серця атлета vs ГКМП (Maron 2009); клінічна примітка про симптоми; clinical-safety PASS
- [x] post-126: Центральна нейром'язова втома — twitch interpolation і voluntary activation (Gandevia 1996); зниження VA 95%→78% при стомленні; Central Governor Model (Noakes 2004); аферентна сигналізація Group III/IV; серотонін/дофамін-гіпотези; кумулятивна центральна втома і синдром перетренованості (Meeusen 2013); practical: HRV-тренд, деперезавантаження, RPE як маркер; clinical-safety PASS
- [x] post-127: Stretch-shortening cycle і тітін — три фази SSC (ексцентрична, аморизаційна, концентрична); сухожилля як пружний резервуар; тітін як молекулярна пружина (Nishikawa 2012, Linke 2018); Ca²⁺-залежне прикріплення тітіну до актину; CMJ vs SJ: ~8–18% SSC-внесок; slow vs fast SSC; plyometrics для силовиків; паузний vs звичайний присід; complex training; clinical-safety PASS
- [x] post-128: Синхронізація мотонейронів і rate coding — Henneman Size Principle (1965); два механізми сили: рекрутування і rate coding; МЕ-синхронізація і пікова сила (Semmler & Enoka 2000); нейронні адаптації до тренінгу: VA, rate coding, патерн координації; bilateral deficit; перший місяць = нейронна адаптація; важкий тренінг для IIb рекрутування; ізометрії для rate coding; clinical-safety PASS
- [x] post-129: Закон Вольфа і кісткова адаптація до тренінгу — механотрансдукція через остеоцити (Wnt/β-catenin, RANKL/OPG); базова ремодуляційна одиниця (BMU) і часова шкала 3–6 місяців; теорія механостату Фроста (1987) з порогами мікродеформації; вікно небезпеки між м'язовою і кістковою адаптацією; Guadalupe-Grau et al. (2009) мета-аналіз BMD; сайт-специфічність; вік і BMD; clinical-safety PASS
- [x] post-130: М'язово-сухожильна жорсткість і еластичне збереження енергії — compliance vs stiffness у performance; роль ахілового сухожилля як пружного резервуара; як тренувати жорсткість (важкі ізометрії, ударне навантаження, plyometrics); відмінності між м'язовою і сухожильною жорсткістю
- [x] post-131: Циркадна біологія і тренувальний тайминг — хронотип, добова варіація сили ~3–8% (Atkinson & Reilly 1996); циркадна генна експресія у м\'язах (Zambon 2003, Zhang 2014); Clock-генова регуляція мітохондрів (BMAL1/PGC-1α, Peek 2013); mTORC1 і добова трансляційна активність; хронотип MSFsc (Roenneberg 2003–2006); Clock-зсув від тренування (Wolff & Esser 2012 PNAS); матриця вибору часу тренування; clinical-safety PASS
- [x] post-136: Спінальні рефлекторні ланцюги і гальмування — GTO і Ib-гальмування (регулятор навантаження, не лише захист); Ia-інгібіторні інтернейрони і реципрокне гальмування (Sherrington 1907); H-рефлекс і адаптація H/M у силових vs витривалісних атлетів; нейральне розгальмування як механізм ранніх силових приростів; клітини Реншоу і рекурентне гальмування; co-activation trade-off; clinical-safety PASS
- [x] post-137: Сигнальний каскад mTORC1 — PI3K→Akt→TSC2→Rheb→mTORC1 ланцюг; phosphatidic acid як механотрансдукційний активатор (Hornberger et al. 2006 PNAS); S6K1 (TOP mRNA, рибосомальна біогенез) і 4E-BP1 (кеп-залежна трансляція) як ефектори; SESN2–GATOR2–GATOR1–Rag GTPases молекулярний сенсор амінокислот (Wolfson et al. 2016 Science); AMPK→TSC2(Thr1227/Ser1345) і AMPK→Raptor(Ser722/Ser792) при тривалому кардіо (Hardie et al. 2012); часовий профіль активації; практичні наслідки без дієтичних рекомендацій; clinical-safety PASS
- [x] post-138: Satellite cells і міоядерний домен — PAX7+/MYOD1+ активаційний каскад; HGF (Tidball & Villalta 2010), FGF-2, MGF як активатори; міоядерний домен ~1500–2000 μm³ (Cheek 1985, Kadi & Thornell 2000); responders vs non-responders (Bamman et al. 2003, Petrella et al. 2006/2008); Bruusgaard et al. (2010 PNAS): міоядра персистують після атрофії; McCarthy et al. (2011 PNAS): гіпертрофія без satellite cells до певної межі; Seaborne et al. (2018 Sci Rep): епігенетичний відбиток тренінгу; практичні наслідки ексцентрики, близькості до відмови, повернення після паузи; clinical-safety NOT REQUIRED
- [x] post-139: Постактиваційна потенціація (PAP/PAPE) — MLC-фосфорилювання як молекулярна основа PAP; PAP vs PAPE (Blazevich & Babault 2019); конкурентні ефекти потенціації і втоми, вікно 4–12 хвилин; залежність від тренувального рівня і відсотка Type II волокон; комплексне тренування; responders vs non-responders; clinical-safety NOT REQUIRED
- [x] post-140: Ексцентричний тренінг — тітін як активний пружний елемент при Ca²⁺-залежному розтягуванні (Herzog et al. 2012, Nishikawa et al. 2020); ексцентрична сила ≥ 120–150% концентричного максимуму; stretch-mediated hypertrophy і саркомерогенез у серію (Oranchuk et al. 2019); Nordic curl −51% травм підколінних сухожилків; AEL і протокол 4-секундного опускання; clinical-safety NOT REQUIRED
- [x] post-141: Авторегуляція: RPE, RIR і VBT — чому фіксований % від 1RM не враховує щоденну варіабельність; RPE 1–10 у силовому тренінгу (Zourdos et al. 2016); RIR як контролер навантаження (Helms et al. 2016); VBT: MPV–%1RM кореляція (González-Badillo 2010), MVT, load-velocity профіль; комбінована стратегія; clinical-safety NOT REQUIRED
- [x] post-142: Рибосомна біогенез і тренувальний об'єм — рибосомна ємність як пляшкове горло синтезу білка (Figueiredo et al. 2015, 2017); c-Myc → Pol I → рДНК; mTORC1/S6K1 → TIF-1A; 4–8 тижнів тренінгу нарощують рибосомний вміст; volume loading як тригер; чому нетреновані відповідають на мінімальний об'єм; MEV/MAV/MRV через молекулярну лінзу; clinical-safety NOT REQUIRED
- [x] post-143: Reactive oxygen species (ROS) і гормезис — мітохондріальний ROS як сигнальна молекула (не лише пошкодження); NRF2 activation; Ristow et al. (2009 PNAS) — антиоксиданти блокують адаптацію; Gomez-Cabrera et al. (2008); парадокс суплементації вітамінами C/E при силовому тренінгу; hormesis як принцип; clinical-safety NOT REQUIRED
- [x] post-144: Аутофагія і ремоделювання м'яза — AMPK→ULK1 і mTOR-аутофагія toggle; мітофагія як якість-контроль; Jamart et al. (2012) — тренінг активує аутофагію в м'язі; протеасомна деградація і убіквітин-шлях; практичне вікно тренінгу і відновлення; clinical-safety NOT REQUIRED
- [x] post-145: Механотрансдукція і tensegrity — integrin–costamere–cytoskeleton–LINC complex–nuclear lamina ланцюг; Piezo1/2 механочутливі канали; focal adhesion kinase (FAK) і ранній сигнальний відгук (Flück et al. 2008); tensegrity (Ingber 2003); phosphatidic acid через PLD (Hornberger); клітина як механічно-чутлива структура; clinical-safety NOT REQUIRED
- [x] post-146: Міокіни — м'яз як ендокринний орган — IL-6 як перший міокін (Pedersen & Febbraio 2008 Physiol Rev): транзиторний протизапальний vs хронічний патологічний; PGC-1α → FNDC5 → іризин (Boström et al. 2012 Nature) і hippocampal BDNF через PGC-1α/FNDC5 (Wrann et al. 2013 Cell Metabolism); іризиновий консенсус 2024 (Jedrychowski et al. 2015 Cell Metabolism, мас-спектрометрія); FGF21 і митохондріальний стрес-сигнал; Meteorin-like (Rao et al. 2014 Cell); остеокальцин–м'яз петля (Mera et al. 2016 Cell Metabolism): unOC → GPRC6A → FAO; SPARC і колоректальна протипухлинна вісь; 600+ міокінів секретому; clinical-safety NOT REQUIRED
- [x] post-147: Позаклітинні везикули — як м'яз надсилає пакети інструкцій до інших органів — Johnstone 1987 (везикули ретикулоцитів — «клітинне сміття»); Valadi et al. (2007 Nature Cell Biology): EVs переносять функціональні мРНК і мікроРНК; ISEV-номенклатура: small EVs (екзосоми 30–150 нм), microvesicles (100–1000 нм); nSMase2/кераміновий шлях секреції; MyomiRs: miR-1→Cx43, miR-133a→RhoA/FASN, miR-206→TGF-β1/HDAC4; Frühbeis et al. (2015 FASEB J): 30 хв велоспорту → 2–3× EVs; Whitham et al. (2018 Cell Metabolism): 300+ білків у вантажі; чому exercise mimetic via EVs неможливий; clinical-safety NOT REQUIRED
- [x] post-148: Мітохондріальний біогенез — як тренінг перетворює клітину зсередини — Holloszy (1967 JBC): 2–3× зростання мітохондріальних ензимів у щурів після 12 тижнів тренінгу; три сигнальних шляхи до PGC-1α: AMPK (LKB1→Thr172, Zong 2002 PNAS), CaMKII (Ca²⁺-залежний, Wu 2002 Cell), SIRT1 (NAD⁺/Rodgers 2005 Nature); PGC-1α → NRF1/NRF2 → TFAM → мтДНК реплікація (Scarpulla 2011 Physiol Rev); PGC-1α ізоформи: α1 (витривалість/мітохондрії) vs α4 (резистентний тренінг/IGF-1/гіпертрофія, Ruas et al. 2012 Cell); мітохондріальна динаміка: fusion (MFN1/2, OPA1) і fission (DRP1), Twig et al. (2008 EMBO J); мітофагія PINK1/Parkin (He et al. 2012 Nature); Zone 2 vs HIIT: Burgomaster et al. (2008 J Physiol) — еквівалентна CS-активність при 2.5× меншому обсязі; часова шкала адаптації; clinical-safety NOT REQUIRED
- [x] post-149: МHC ізоформи і пластичність волокон — Ranvier (1873) «білі vs червоні»; Schiaffino & Reggiani (2011 Physiol Rev): MHC I/IIa/IIx і ATPase кінетика; Staron et al. (1994 JAP): силовий тренінг → IIx→IIa за 8 тижнів; Andersen & Aagaard (2000): IIx overshoot після дитренінгу; Chin et al. (1998 Science): calcineurin-NFAT → повільний фенотип; Blaauw et al. (2013): Ca²⁺ частотний код Zone 2 vs HIIT; гібридні волокна і межі трансформації; clinical-safety NOT REQUIRED
- [x] post-150: Кальцієва сигналізація — CaMKII, calcineurin/NFAT і вибір між силою і витривалістю — E-C coupling і SERCA1a/2a ізоформи; Ca²⁺ частотний код як молекулярний перемикач фенотипу; Chin et al. (1998 Science) cross-innervation; Bers (2002 Nature), Wu (2002 Cell); praktychni vysnovky; clinical-safety NOT REQUIRED
- [x] post-151: Нейром'язове сполучення і синаптична пластичність — Deschenes et al. (1993 Am J Physiol): NMJ ремоделювання при тренінгу; агрин/MuSK/рапсин; BDNF/trkB двосторонній трофічний зв'язок; Henneman size principle і rate coding (Duchateau & Enoka 2011 J Appl Physiol); нейральна адаптація як механізм раннього приросту сили; clinical-safety NOT REQUIRED
- [x] post-152: Запалення і тренінг — м'яз як протизапальний орган — Petersen & Pedersen (2005 J Appl Physiol): хронічне запалення vs гостра відповідь; Pedersen & Febbraio (2008 Physiol Rev): м'язовий IL-6 → IL-10/IL-1Ra петля без TNF-α; Arnold (2007 J Exp Med): M1→M2 перехід і регенерація; Ristow (2009 PNAS): антиоксиданти блокують адаптацію; Gleeson (2011 Nat Rev Immunol): перепрограмування макрофагів і вагусний тонус; Erickson (2011 PNAS): +2% гіпокамп; clinical-safety NOT REQUIRED
- [x] post-153: Лактат — від «токсину» до центральної молекули адаптації — Brooks (1985) Cell-to-Cell Lactate Shuttle; MCT1/MCT4 транспортери і тренована адаптація; GPR81 (HCAR1) рецептор лактату; гістонове лактилювання (Zhang et al. 2019 Nature 574); лактат → VEGF → ангіогенез; Zone 2 як тренінг MCT1-кліренсу; HIIT як тренінг MCT4; активне відновлення між підходами; clinical-safety NOT REQUIRED
- [x] post-154: Теломери і тренінг як геропротекція — hTERT і ерозія теломер; Cherkas et al. (2008 Arch Intern Med): 2 401 пара близнюків, ≈200 п.о. різниця; Werner et al. (2009 Circulation): 8-тижневий RCT, ↑ hTERT ↓ p16INK4a у людей 50+; LaRocca et al. (2010 J Physiol): аеробна ємність і теломери EPC; IGF-1/FOXO3-пульс у відновленні; межа: ультраобсяги без відновлення → зворотний ефект; clinical-safety NOT REQUIRED
- [x] post-155: Ефект інтерференції в суміщеному тренінгу — молекулярний антагонізм AMPK/mTORC1 — Hickson (1980) original interference observation; AMPK→TSC2/Rheb (Inoki 2003) і AMPK→Raptor (Gwinn 2008) два маршрути пригнічення mTORC1; Atherton (2005)/Coffey (2006) гостра сигналізація; Wilson (2012) і Schumann (2022) мета-аналізи; порядок сесій і тижневий мінімум ≥6h розрив; clinical-safety NOT REQUIRED ⚠️ витіснив planned «Колаген і ECM» — зберегти як post-159
- [x] post-156: Глімфатична система і сон — як тренінг і сон разом очищують мозок — Nedergaard/Iliff et al. (2012 Sci Transl Med): відкриття глімфатичної системи; AQP4 периваскулярна поляризація; Xie et al. (2013 Science): сон ↑ глімфатичний потік ~60%, міжклітинний простір +60%; Holth et al. (2019 Science): 1 ніч депривації → +25–30% β-амілоїду в CSF; Lundgaard et al. (2017 Nat Commun): тренінг ↑ AQP4-поляризацію; синаптична консолідація (Walker 2017); Lee et al. (2015 J Neurosci): бокове положення сну; трикутник: тренінг → BDNF+якісний сон → нейропластичність; clinical-safety NOT REQUIRED
- [x] post-157: Міокіни — скелетний м'яз як ендокринний орган — Steensberg et al. (2000 J Physiol): IL-6 з м'яза до 100× під навантаженням, термін «міокін» (Pedersen 2003); IL-6 антизапальний парадокс (класична vs транс-сигналізація); Wrann et al. (2013 Cell Metab) PGC-1α→FNDC5→BDNF; іризин/FNDC5 контроверза (Boström 2012; Jedrychowski 2015 маса-спектрометрія); FGF21 ATF4-залежна м'язова секреція; METRNL еозинофіл-IL4-STAT6 шлях; міостатин/фолістатин ActRIIB→Smad2/3; остеокальцин GPRC6A кістково-м'язова петля (Mera 2016); профіль силового vs аеробного тренінгу; clinical-safety NOT REQUIRED
- [x] post-158: Мітофагія — клітинне прибирання, яке робить тебе сильнішим — PINK1/Parkin шлях: ΔΨm-сенсор, убіквітинування OMM, LC3-адаптери p62/NDP52/OPTN; He et al. (2012 Nature): блокування аутофагії скасовує мітохондріальну адаптацію; Ristow (2009 PNAS) і Paulsen (2014 J Physiol): антиоксиданти гальмують мітофагічний сигнал; Zone 2 як технічне обслуговування мітохондріального пулу; clinical-safety NOT REQUIRED
- [x] post-159: Нейрогенез і аеробний тренінг — Van Praag et al. (1999 Nature Neuroscience): бігові колеса подвоюють нейрогенез у гіпокампі мишей; Erickson et al. (2011 PNAS): аеробний тренінг +2% гіпокамп у 55–80-річних людей; BDNF як «fertilizer of the brain» (Cotman & Berchtold 2002 TINS); IGF-1 і VEGF як паралельні механізми; чому кортизол і антиоксиданти нівелюють нейрогенний ефект; clinical-safety NOT REQUIRED
- [x] post-160: Силова-швидкісна крива — фізіологія компромісу між силою і швидкістю — модель Хілла (1938), F-V гіпербола, пік потужності ~30–70% 1RM (Cormie et al. 2011), cross-bridge kinetics, RFD і перші 50–200 мс, PAP/contrast training, обмеження моделі; clinical-safety NOT REQUIRED
- [x] post-161: Мікробіом і тренінг — двонаправлений зв'язок — Clarke et al. (2014 Gut): елітні регбісти vs контроль — 22 проти 14 філумів; Allen et al. (2018 Med Sci Sports Exerc): зміни зворотні після детренінгу; Scheiman et al. (2019 Nature Medicine): Veillonella atypica конвертує лактат у пропіонат → +13% витривалість у мишей; SCFA-механізм (бутират/пропіонат/ацетат); Akkermansia і кишковий бар'єр; VETO-зона: нічого про дієту; clinical-safety NOT REQUIRED
- [x] post-162: Сателітні клітини і гіпертрофія — Mauro (1961), Pax7→MyoD/Myf5 каскад, IGF-1 Ec/MGF, теорія міоядерного домену, полеміка Egner et al. (2016 Science), м'язова пам'ять і перманентне збереження міоядер (лабораторія Гундерсена); clinical-safety NOT REQUIRED
- [x] post-163: Білки теплового шоку (HSP) — Ritossa (1962), HSP70/HSP90 і протеостатична мережа, PHD-механізм агрегації, тренувальний стрес як HSP-індуктор, горміза і адаптивна стійкість; clinical-safety NOT REQUIRED
- [x] post-164: Адаптація до висоти — HIF-1α стабілізація і downstream ефекти, EPO-кінетика і Hbmass +3.3–4.5% (Gore et al. 2013 BJSM), Levine & Stray-Gundersen (1997): Live High Train Low vs LHTH, altitude tents: поріг ≥12 год/добу, hypobaric vs normobaric при ≤3000 м, практичний поріг ≥2000 м/≥21 днів, де ефект відсутній; clinical-safety NOT REQUIRED
- [x] post-165: BFR-тренінг — чотири механізми гіпертрофії без важких ваг — Takarada et al. (2000 J Appl Physiol): рандомізоване дослідження оклюзійного тренінгу; метаболічний стрес (лактат-GPR81, ROS hormesis, mTORC1), рекрутування Type II волокон при 20% 1RM, гормональна відповідь (Group III/IV хеморецептори → GH), клітинний набряк (Piezo1/TRPC); протокол 1×30+3×15 при 20–30% 1RM; Lixandrão 2018 meta-analysis: BFR ≈ traditional RT для гіпертрофії; clinical-safety PASS
- [x] post-166: Розвантажувальний тиждень — фізіологія і стратегії диловання — периферична (глікоген, Ca²⁺ regulation) vs центральна (кортикальний драйв, TMS Taylor 2006) втома; supercompensation timing: гіпертрофія 4–7 днів, сила 7–14 днів, нейральна до 21 дня; 4 стратегії deload; Bosquet et al. (2007): tapering → +2.4–3% performance; рішення-матриця 4 trigger criteria; clinical-safety NOT REQUIRED
- [x] post-167: Вісь HPA і кортизол у тренінгу — адаптивний сигнал, а не ворог — Selye (1936/1950) GAS; HPA-каскад CRH→ACTH→кортизол; Viru & Viru (2004) кортизол як permissive hormone; Tremblay et al. (2005 Sports Med): кінетика кортизолу, поріг 60–70% VO₂max; Häkkinen & Pakarinen (1993): 10×10 squat → більший кортизол, ніж важкий малоповторний; FOXO1/3→MuRF1/MAFbx: атрогенний шлях вимагає хронічного підвищення; Fry et al. (1998): C:T ratio і non-functional overreaching; Cortisol Awakening Response; Leproult et al. (1997 Sleep): депривація → +21–37% кортизол; clinical-safety NOT REQUIRED
- [x] post-168: Нейром'язова втома: центральні та периферичні механізми — Enoka & Duchateau (2008 J Appl Physiol): визначення втоми як набутого зниження силогенерації; Gandevia (2001 Annu Rev Neurosci): суправентральна і спинальна компоненти центральної втоми; метаборефлекс (волокна III/IV типу → спинальне гальмування); Westerblad & Allen (2002): Pi-механізм втоми та роль Ca²⁺/СР при фізіологічній температурі; K⁺-гіпотеза Juel (1988); метод суперімпозованого твітчу (Bigland-Ritchie & Woods 1984); відновлення Ca²⁺/K⁺/суправентральне; практична матриця: паузи, RPE, специфіка ексцентрики; clinical-safety NOT REQUIRED
- [x] post-169: Phosphocreatine ресинтез — кінетика і тренувальні наслідки — Harris et al. (1976 Biochem J): перше вимірювання PCr у людині; Hultman et al. (1996 J Appl Physiol): кінетика повного відновлення ~3–5 хв; pH-залежність ресинтезу (Sahlin et al. 1979 Scand J Clin Lab Invest); creatine supplementation і PCr накопичення; Schoenfeld et al. (2016 JSCR): 3-хв паузи vs 1 хв — вищий гіпертрофійний приріст; Haff et al. (2003 JSCR): кластерні сети і PCr кінетика; практична матриця пауз по типу роботи; clinical-safety NOT REQUIRED
- [ ] post-170: Механіка болю і тренінг — ноцицепція vs ноципластичний біль — Woolf (2011 Nature Medicine): центральна сенсибілізація; DOMS як запальна ноцицепція (не пошкодження); PGE2/IL-6 каскад у EIMD; клінічна різниця між «нормальним» тренувальним дискомфортом і патологічним болем; ⚠️ clinical-safety REQUIRED — тема болю, route to CS before publish
- [x] post-171: Сон і спортивні результати — не «більше сну», а «правильний сон» — Walker et al. (2002 Neuron): сонні веретена N2 і консолідація рухових патернів; Mah et al. (2011 Sleep): +9% точність кидків, −0.7 с спринт при розширенні сну у баскетболістів; SWS і нічний GH-пульс (~70% добової секреції); REM і Walker & van der Helm (2009) overnight therapy; Van Dongen et al. (2003): 6-годинний хронічний сон як «прихований» дефіцит; Leproult et al. (1997): скорочення сну → +37% кортизол; циркадний ритм і час тренувань; практична гігієна сну (температура, мелатонін, кофеїн, naps); clinical-safety PASS
- [x] post-172: ⚠️ написано як «Ефект перехресної адаптації» (cross-education) — тема grip strength перенесена як post-187; clinical-safety PASS
- [x] post-173: Осмотична адаптація м'яза — клітинне набухання як механотрансдукційний сигнал — Häussinger et al. (1993 Lancet): cell volume як детермінант катаболізму; Schliess & Häussinger (2002): гіпотонічне набухання → PI3K → mTOR незалежно від інсуліну; Low et al. (1996 J Physiol): синтез глікогену і осмотичне набухання у скелетному м'язі; VRAC/LRRC8A і АТФ-залежна регуляція об'єму; механічний vs осмотичний шлях mTOR — паралельні, частково адитивні; дегідратація 2%+ → пригнічення анаболічного сигналу; крeatин і клітинний об'єм; clinical-safety NOT REQUIRED
- [x] post-174: Нервово-м'язова адаптація в перший місяць тренінгу — чому сила зростає без гіпертрофії — Moritani & deVries (1979): EMG і сила у нетренованих; Sale (1988 Med Sci Sports): 8 тижнів → до 50% приросту сили при мінімальній гіпертрофії; VA (voluntary activation) і bilateral deficit; motor cortex excitability (TMS); синхронізація мотонейронів і rate coding; clinical-safety NOT REQUIRED
- [x] post-175: Ефект повторного навантаження — як перший ексцентричний боут захищає наступний — Clarkson et al. (1992 J Appl Physiol): 70 ексцентричних скорочень → КК ~2000 Од/л; repeated bout effect: КК в нормі, MVC падіння &lt;10%; Morgan & Allen (1999): поппінг-саркомери; ECM-ремоделінг, десмін і тайтин; Lavender & Nosaka (2006) нейральна адаптація; тривалість захисту: 6-8 тижнів повний; clinical-safety NOT REQUIRED
- [x] post-176: PAPE: Post-Activation Performance Enhancement — PAP vs PAPE (Seitz & Haff 2016); MLC-фосфорилювання MLCK → Ca²⁺-чутливість → ↑ RFD; PAP–Fatigue конундрум; вікно 5–12 хв; ≥75–87% 1RM; CMJ d=0.54, спринт, важка атлетика, пауерліфтинг; Contrast vs Complex vs PAP; три шаблонні протоколи; clinical-safety NOT REQUIRED
- [x] post-177: Автономна нервова система і тренінг — симпатовагальний баланс і HRV — Chapleau & Sabharwal (2011): вагусна модуляція серця; тренінг як маніпуляція ANS balance; Aubert et al. (2003 Sports Med): HRV у атлетів — спектральний аналіз LF/HF у 89 атлетів; vagal tone у силовиків vs ендуранс-атлетів; cardiac autonomic neuropathy при OTS; Kiviniemi et al. (2007 IJSPP): HRV-guided periodization; чому RMSSD знижується вранці після важкого тренінгу; LF/HF не є чистим симпатичним маркером (Billman 2013); практичний протокол 3-зонового HRV-програмування; clinical-safety NOT REQUIRED
- [x] post-178: Мітохондріальна адаптація до силового тренування — PGC-1α ізоформи; AMPK vs mTORC1 сигнальна інтерференція; Egan & Zierath (2013 Cell Metab); Coffey & Hawley (2007) interference effect; Zone 2 для силовиків: механізм; тайминг для мінімізації конкурентного сигналінгу; clinical-safety NOT REQUIRED
- [x] post-179: Спортивне серце — кардіальна адаптація до силового тренінгу — Devereux et al. (1983): ехокардіографія і athletic heart; concentric vs eccentric LV hypertrophy; Baggish & Wood (2011 Circulation): ендуранс vs силовий фенотип адаптації; athlete's heart vs HCM: ключові диференціальні критерії; VO₂max plateau у силовиків; practical: яка кардіоваскулярна робота доповнює силовий тренінг без interference; clinical-safety NOT REQUIRED
- [x] post-180: Інсулінова сигналізація у скелетному м'язі — GLUT4, PI3K/Akt і тренінг — DeFronzo et al. (1981 J Clin Invest): клемп-техніка і периферична чутливість до інсуліну; Richter & Hargreaves (2013 Physiol Rev): GLUT4-транслокація інсулін-залежним і Ca²⁺/AMPK-незалежним шляхом; тренінг як неінсулінова GLUT4-активація; Holloszy (2011 Appl Physiol Nutr Metab): механізм поліпшення інсулінової чутливості; практичне: чому силовий тренінг знижує ризик T2D без дієтарних змін; clinical-safety NOT REQUIRED
- [ ] post-181: Підводний підйом перетренованого атлета — як кількісно виміряти відновлення і визначити момент повернення до навантаження — Meeusen et al. (2013 consensus statement): functional overreaching vs non-functional overreaching vs OTS; HRV-тренд як превентивний маркер; RPE-навантаження невідповідність; практичний протокол оцінки готовності до підвищення обсягу; ⚠️ clinical-safety REQUIRED — тема може торкатися ознак виснаження, route to CS before publish
- [x] post-182: Інгібування міостатину і м'язовий ріст — McPherron et al. (1997 Nature): myostatin knockout mouse; MSTN → ActRIIB → Smad2/3 → анти-гіпертрофійний сигнал; фоллістатин як природний антагоніст (Lee & McPherron 2001: FST надекспресія → 4× маса); Hittel et al. (2010 JSCR): силовий тренінг знижує MSTN і підвищує FST; Ruas et al. (2012 Cell): PGC-1α4 пригнічує MSTN mRNA; FAK-опосередковане блокування Smad3 при механічній напрузі; 25 років фарм-розробок (stamulumab, ACE-031, bimagrumab) без FDA-схвалення; що реально дає тренінг і що ні; clinical-safety NOT REQUIRED
- [x] post-183: Механізми адаптації до холоду і тепла — бурий жировий адипоцит (UCP1) і термогенез без тремтіння; Nedergaard et al. (2009 AJPCEP): brown fat у дорослих людей; TRPM8 і холодова сигналізація; теплова акліматизація і плазмовий об'єм (Périard et al. 2021 Exp Physiol); heat shock proteins (HSP70/90) як молекулярні шапероні і їхня роль у тренуванні; cross-adaptation: чи дає холодова підготовка термальну резистентність; clinical-safety NOT REQUIRED
- [ ] post-184: Осьові вправи і навантаження на хребет — спінальна компресія: McGill et al. (2002 Low Back Disorders): норми і ліміти для хребтових дисків; різниця між компресійним і зсувним навантаженням; як нейтральне положення хребта впливає на дискові навантаження; Internal Abdominal Pressure (IAP) як захисний механізм; dead bug vs crunch — чому вибір вправи визначається не болем а механікою; ⚠️ clinical-safety: уникнути клінічних рекомендацій при болях у спині — clinical-safety REQUIRED тільки якщо зачіпаємо реабілітацію
- [x] post-185: Механіка колінного суглоба при силовому тренінгу — пателофеморальний суглобовий стрес (Escamilla et al. 1998 Med Sci Sports): сила реакції при присіданні і жимі ногами; valgus collapse і м'язова активація; вплив ширини стопи і кута носка на ACL-навантаження; knee-over-toes debate: що говорить механіка, а не лор; posterior chain vs quad dominance; Fry et al. (2003): обмеження коліна → +1070% торк кульшового суглоба; clinical-safety PASS
- [x] post-186: Нейропластичність і силовий тренінг — Pascual-Leone et al. (1995 PNAS): моторна кора розширюється при навчанні навичці; Kleim et al. (2002 J Neurosci): синаптичне зміцнення у моторній корі після рухового навчання; Cross-education і bilateral transfer; кортикальне ремоделювання при тривалому силовому тренінгу (Pearce et al. 2013 Eur J Neurosci); практичне: чому техніка потребує роботи ЦНС, а не лише м'язів; clinical-safety NOT REQUIRED · написано як post-365
- [x] post-187: Сила хвату — функціональна анатомія і тренінг (перенесено з post-172) — три механізми хвату (crush, pinch, support); Bohannon (2019 JSCR): grip як longevity proxy; flexor digitorum profundis vs superficialis; forearm extensor neglect; farmer's carry і deadlift strap debate; протокол розвитку від нуля; clinical-safety NOT REQUIRED · написано як post-366
- [x] post-188: Гіпертрофія у розтягнутій позиції — titin, mTORC1, stretch-mediated hypertrophy; Maeo (2023), Pedrosa (2022), Kassiano (2023), Herzog (2023); clinical-safety NOT REQUIRED — Walker et al. (2002 Neuron): когнітивна і моторна консолідація під час сну; Pageaux & Lepers (2018 Sports Med): ментальна втома знижує витривалість ~5–10%; конкуруючі завдання когнітивного і фізичного навантаження; practitioner-flow: чому «важкий робочий тиждень» впливає на силову продуктивність без хронічного тренувального перевантаження; clinical-safety NOT REQUIRED
- [x] post-189: Пружна енергія сухожиль і накопичення — Ker et al. (1987 Nature): ~35% кінетичної енергії при ходьбі; SSC fast/slow, Wilson (1991): +31% SSC enhancement; Bohm (2014): +20% tendon stiffness; dead-stop vs dynamic — дві різні адаптації; Hof (2002) модель; clinical-safety NOT REQUIRED — Ker et al. (1987 Nature): ахіллове сухожилля зберігає ~35% кінетичної енергії при ходьбі; Wilson et al. (1991) elastic energy reuse у SSC; tendon stiffness і тренувальна адаптація; чому силові вправи з паузою (dead-stop) тренують концентричну силу, але не SSC-ефективність; Hof et al. (2002 J Biomech): моделювання пружного накопичення; clinical-safety NOT REQUIRED
- [ ] post-190: Голодний мозок і тренінг — glucose і когнітивна функція без харчових рекомендацій — Gailliot & Baumeister (2007 Psychol Rev): ego depletion і самоконтроль; McNay et al. (2000 J Neurosci): hipocampal glucose при когнітивному навантаженні; енергетика рухового програмування; ⚠️ clinical-safety REQUIRED — не публікувати без повного CS review (food/cognitive load)
- [x] post-191: Дихальні м'язи і продуктивність — метаборефлекс діафрагми (Dempsey et al. 2006 J Physiol): втомлені дихальні м'язи перерозподіляють кровотік від ніг; Romer & Polkey (2008 J Appl Physiol): дихальний м'яз training покращує витривалість; Inspiratory Muscle Training (IMT) і силові атлети; bracing vs breathing trade-off при субмаксимальних підходах; clinical-safety NOT REQUIRED
- [x] post-192: Пропріоцепція суглобів після травми — нейром'язова перебудова post-ACL і post-ankle sprain; Freeman et al. (1965 J Bone Joint Surg): balance training після розтягнення; Lephart et al. (1997): joint position sense і аферентна сигналізація після травми; wobble board і деформована поверхня як тренінг joint receptors, а не «core stability»; ⚠️ клінічна точка зору без реабілітаційних рекомендацій — clinical-safety NOT REQUIRED
- [x] post-193: Саркомерна механіка — актин-міозин і реальна картина скорочення — Huxley (1954 Nature / 1969 Science): sliding filament theory і cross-bridge cycle; Hill equation з механістичного боку; duty cycle і економія м'яза; Piazzesi et al. (2007 Cell): in vivo single fiber measurements при різних навантаженнях; чому «контроль швидкості руху» змінює силовий профіль і рекрутування; titin як активний компонент (Linke & Hamdani 2014); clinical-safety NOT REQUIRED
- [x] post-194: Окислювальний стрес і тренінг — ROS як сигнальна молекула — Ristow et al. (2009 PNAS): антиоксидантні добавки блокують адаптацію; гормезис і мітохондріальна редокс-сигналізація; Merry & Ristow (2016 J Physiol): exercise-induced ROS і адаптація; NRF2-ARE pathway; NRF2-Keap1 mechanism (Cys151/273/288 oxidation); AMPK Cys299/304 redox activation; чому вітамін C/E у великих дозах може послаблювати тренувальний ефект; дієтарні рівні vs мегадози; clinical-safety NOT REQUIRED
- [x] post-195: Центральна втома і Governor Model — Noakes (2012 Br J Sports Med): Central Governor як регулятор максимального зусилля; чи справді мозок зупиняє нас до м'язового відмови; Marcora (2010): perception of effort як первинний обмежувальний фактор; нейронні основи відмови від підходу; наслідки для ментальної тренованості; clinical-safety NOT REQUIRED
- [x] post-196: Міофасціальна безперервність — де наука, де лор — Myers (2001) Anatomy Trains: 7 фасціальних ліній як клінічна гіпотеза; Wilke et al. (2016 J Anat): механічне тестування fascia thoracolumbalis; Huijing (1999 J Biomech): epimuscular myofascial force transmission ~5–15%; Cheatham (2015): foam rolling нейральний ефект, не release; Schleip (2003): мануальні техніки через DNIC; що підтверджено: TLF і стабілізація поперека, plantar fascia і windlass, фасціальна іннервація і proprioception; clinical-safety NOT REQUIRED
- [x] post-197: Тренінг і когнітивна функція — Erickson et al. (2011 PNAS): аеробний тренінг збільшує гіпокамп на 2%; BDNF як молекулярний медіатор; Lambourne & Tomporowski (2010): гострий ефект вправ на когнітивну функцію; відмінності між силовим і аеробним тренінгом на когнітивні наслідки; чому physical literacy має значення поза gym; clinical-safety NOT REQUIRED
- [x] post-198: Симпатоадреналова вісь при тренінгу — катехоламіни і β-адренорецептори — Galbo (1983 Horm Metab Res): катехоламінові піки при різних інтенсивностях; адренергічна регуляція ліполізу і глікогенолізу під час вправи; α vs β адренорецептори у скелетному м'язі і жировій тканині; хронічна адаптація до тренінгу — десенситизація β-рецепторів у спокої; наслідки для тренінгу у стресових умовах (недосипання, психологічний стрес); clinical-safety NOT REQUIRED
- [x] post-199: Ангіогенез і капілярна адаптація до тренінгу — VEGF і капілярна щільність — Richardson et al. (1999 J Physiol): VEGF upregulation у м'язі при гіпоксичному навантаженні; Hudlicka et al. (2009 Microcirculation): механічний і метаболічний тригер капілярного росту; capillary-to-fiber ratio у тренованих vs нетренованих; чому Zone 2 збільшує кисневий транспорт до волокон; практичне: нижня межа аеробного об'єму для підтримки капілярізації; clinical-safety NOT REQUIRED
- [x] post-200: Епігенетична пам'ять тренінгу — ДНК метилювання і гістонові модифікації — Seaborne et al. (2018 Sci Rep): гіпометилювання NCOR2 і IGF1 після силового тренінгу; Sharples et al. (2019 PLOS Genet): збереження епігенетичних маркерів після детренування; чому атлетичний минулий досвід прискорює повернення після паузи; «м'язова пам'ять» на рівні хроматину, а не лише міонуклеусів; клінічне значення для повернення після травми; clinical-safety NOT REQUIRED
- [x] post-201: Серцево-судинний дрейф при тривалому навантаженні — cardiovascular drift і плазмовий об'єм — Coyle & González-Alonso (2001 J Appl Physiol): механізм серцево-судинного дрейфу при тривалих вправах; перерозподіл плазмового об'єму до шкіри (терморегуляція) vs м'язи; чому HR зростає при постійній потужності у третій годині; Rowell (1993): cardiovascular control в умовах теплового стресу і навантаження; практичне: чому HR-зони зміщуються при тривалих сесіях і що з цим робити; clinical-safety NOT REQUIRED
- [x] post-202: Нейральна ефективність — economics of movement — Lay et al. (2011 J Electromyogr Kinesiol): варіативність inter-muscular coordination у новачків vs тренованих; Semmler (2002 Exerc Sport Sci Rev): motor unit synchronization і strength; чому тренований атлет виконує те саме завдання «дешевше» з точки зору EMG-активності і метаболічних витрат; Enoka (1994): загальна модель нейральної ефективності; практичне: чому техніка — це не естетика, а економія; clinical-safety NOT REQUIRED
- [x] post-203: Механіка сухожиль і адаптація жорсткості — Kubo et al. (2001 J Appl Physiol): ультразвук сухожилля — сила-деформація крива; Maganaris & Paul (1999 J Physiol): in vivo жорсткість ахілова сухожилля; Reeves et al. (2003): 14 тижнів силового тренінгу → +69% жорсткість; Wilson et al. (1994): роль жорсткості у передачі SSC-зусилля; практичне: чому m'яки tendони знижують вибуховість; clinical-safety NOT REQUIRED
- [x] post-204: BFR-тренінг — гіпертрофія під низьким навантаженням — механізми, протоколи, межі застосування — Lixandrao et al. (2018 Med Sci Sports Exerc): мета-аналіз BFR vs traditional; Pearson & Hussain (2015): систематичний огляд BFR і гіпертрофії; сателітні клітини, метаболічний стрес, системний гормональний відгук; протокол: 20–30% 1RM, 75% AOP, 30–15–15–15 схема; clinical-safety NOT REQUIRED
- [x] post-205: Пластичність моторної кори і сила — як тренінг переписує карту руху в мозку — TMS-дослідження: збільшення кортикальної репрезентації після тренінгу; Carroll et al. (2002 J Appl Physiol): зниження SICI (short-interval cortical inhibition); Draganski et al. (2004 Nature): структурні зміни GM за 3 місяці; Sale (1988): нейральна фаза адаптації у перші 8 тижнів; clinical-safety NOT REQUIRED
- [x] post-206: Сила vs потужність — чому це різні адаптації і де перетинаються — визначення: сила (F = ma за кривою F-V), потужність (P = F·v); Wilson et al. (1993 JSCR): potentiation і час розвитку зусилля; Behm & Sale (1993): intent to accelerate змінює neural drive незалежно від фактичної швидкості; Cormie et al. (2011 Sports Med): огляд — тренінг потужності у спортсменів; plyometrics і stretch-shortening cycle як механізм потужності; practical: чому «важке і повільне» не будує вибухову силу; clinical-safety NOT REQUIRED
- [x] post-207: Антагоніст-агоніст парні підходи — реципрокне полегшення і потенціація у силовому тренінгу — Baker & Newton (2005 JSCR): APS жим + тяга → +8,5% пікова потужність; Robbins et al. (2010 JSCR): APS знижують кумулятивну втому, вищий виконаний об'єм; Weakley et al. (2020 JSCR): мета-аналіз ефективності APS; механізм: реципрокне інгібування Іа-інтернейронів + PAPE; протокол пауз (60–90с всередині пари, 90–120с між парами); clinical-safety NOT REQUIRED
- [x] post-208: Інтенсивність vs об'єм — де насправді знаходиться гіпертрофійний поріг — Mitchell et al. (2012 J Appl Physiol): 30% vs 80% 1RM до відмови → еквівалентний MPS; Burd et al. (2010 PLoS ONE): MMF як провідний модератор; Schoenfeld et al. (2017 JSCR): RCT 8RM vs 25–35RM до відмови → еквівалентна гіпертрофія, вища силова адаптація при 80% 1RM; Lasevicius et al. (2018 Eur J Sport Sci): нижній поріг ~30–40% 1RM; два паралельних шляхи: механічна напруга (FAK/ILK → mTORC1) і метаболічний стрес (AMPK, CaMK, MAPK); провідна змінна — RIR 0–2, а не % 1RM; clinical-safety NOT REQUIRED
- [ ] post-209: Тренінг і мікробіом — Bressa et al. (2017 PLOS ONE): різниця мікробіому у фізично активних vs малорухливих жінок; Clarke et al. (2014 Gut): мікробіом rugby-гравців проти контрольних; Mach & Fuster-Botella (2017 J Int Soc Sports Nutr): огляд зв'язків між exercise і мікробіомом; Firmicutes/Bacteroidetes ratio і VO₂max; Akkermansia muciniphila і метаболічне здоров'я; зв'язок тренінгу, кишкового мікробіому і відновлення без харчових рекомендацій; clinical-safety NOT REQUIRED
- [ ] post-209: Тренування в умовах термального стресу — фізіологічний відгук на тренінг при спеці і холоді — Ely et al. (2010 J Appl Physiol): теплова акліматизація і аеробна продуктивність; WBGT і фізіологічний threshold; O'Brien et al. (2014 Sports Med): теплова акліматизація (+7% VO₂max, +12% work capacity у тепловому стресі); холодові умови і силовий тренінг: огляд ефектів на силу і РФД; TRPM8 і нейральні адаптації до холоду; практичне: коли температура є реальним лімітуючим фактором; clinical-safety NOT REQUIRED
- [x] post-210: Когнітивний навантаження і техніка — dual-task і моторна дисфункція під тиском — Wulf & Lewthwaite (2016 Psychon Bull Rev): OPTIMAL theory — external focus і autonomy support покращують моторне навчання; Beilock & Carr (2001 Psychol Sci): choking under pressure — explicit monitoring hypothesis; Beilock et al. (2002 J Exp Psychol Gen): dual-task знижує якість автономних рухів у новачків, але не у тренованих; чому «думати про техніку» у підході іноді погіршує її; практичне: коли внутрішній фокус доречний (навчання новачків), коли зовнішній (виступ); clinical-safety NOT REQUIRED
- [x] post-211: Кінетика ресинтезу PCr — Harris et al. (1976), Hultman et al. (1987) двофазна крива (t½ швидка ≈ 21 с, повільна ≈ 170 с), Bogdanis et al. (1995/1996) продуктивність і pH при неповному ресинтезі, Greenhaff (1994) ацидозне пригнічення CK; практичні норми відпочинку (гіпертрофія 60–90 с / сила 3–5 хв / потужність 5–8 хв) через механізм, а не традицію; clinical-safety NOT REQUIRED
- [x] post-212: Adherence coefficient — Schoenfeld (2017 JSCR), Androulakis-Korakakis (2020), Bickel (2011): тренувальний ефект = якість × коефіцієнт дотримання; математика чому добра програма при 90% перемагає оптимальну при 50%; програма-хопінг як проблема дизайну, не характеру; MED як рівень структури для стійкого виконання; режим «виживання» як частина дизайну; clinical-safety PASS WITH CONDITIONS (2026-06-04)
- [x] post-213: Тренування і імунна функція — open window hypothesis після інтенсивного тренінгу (Gleeson 2007); J-curve залежність між обсягом і захворюваністю (Nieman 1994); ІЛ-6 як міокін і ранній медіатор запалення; J-крива і практичний контекст без клінічних рекомендацій; clinical-safety PASS WITH CONDITIONS (2026-06-04)
- [x] post-214: Сила і когнітивні функції — Northey et al. (2018 BJSM) мета-аналіз: resistance training → MMSE/cognitive composite ES 0.66; BDNF і гіпокамп (van Praag 2008); prefrontal cortex і IGF-1 (Trejo 2001); HIIT vs силовий тренінг: різні когнітивні домени; Liu-Ambrose (2010): executive function superior vs. aerobics; clinical-safety NOT REQUIRED
- [x] post-215: Специфічність vs варіативність — SAID principle і практичний парадокс: чому тренувати все = не тренувати нічого конкретно; transfer of training дослідження (Barnett et al. 2002); вузька специфічність при збереженні загального фітнесу; блокова логіка як компроміс; clinical-safety NOT REQUIRED
- [x] post-216: Кінетика поглинання кисню — Whipp & Wasserman (1972) трифазна структура VO2-відповіді; τ Фази II як маркер аеробної тренованості; Gaesser & Poole (1996) повільний компонент вище критичної потужності; Jones & Poole (2005) CP як межа метаболічної рівноваги; Davies & Thompson (1986) ресинтез PCr і аеробна тренованість у силовика; clinical-safety NOT REQUIRED
- [x] post-217: Архітектура м'яза і функція — Lieber & Fridén (2000) pennation angle / fiber length / PCSA; трейдоф сила vs швидкість; Fukunaga (1997) + Narici (1996) in vivo ultrasound; Aagaard (2001) гіпертрофія і зміна θ; вибір вправ і довжина навантаження; clinical-safety NOT REQUIRED ✓
- [x] post-218: Центральна команда і exercise pressor reflex — Krogh & Lindhard (1913) feedforward CV до руху; Mitchell et al. (1983) групи III/IV аферентів; Amann et al. (2009) fentanyl block experiment; Smith et al. (2006) mechano vs metaboreceptors; RPE як proxy afferent feedback; clinical-safety NOT REQUIRED ✓
- [x] post-219: Rest-pause і myo-rep механіка — Hostler et al. (2001 JSCR) та Prestes et al. (2017 JSCR): rest-pause vs традиційні підходи при рівному об'ємі (+13.8% м'язова витривалість у RP-групі); механізм: PCr ресинтез ~50% за 30 с (Bogdanis 1995), Pi/H⁺ зберігаються → рекрутування вищопорогових Type II; Myo-reps (Fagerli 2008): activation set + кластери — максимізація «ефективних повторень» (RIR 0–2); коли і коли не використовувати; clinical-safety NOT REQUIRED
- [x] post-220: Окислювальна здатність і силовий тренінг — Hood et al. (2019 Cell Metab): мітохондріальний вміст і силова адаптація; аеробна база покращує PCr-ресинтез між підходами; Wilson et al. (2012 meta-analysis) interference мінімальна при ≤ 3×30–40 хв зони 2; Coffey & Hawley (2007) AMPK-mTOR вирішується 6+ год сепаратором; стратегія для атлетів, що хочуть і силу, і витривалість; clinical-safety NOT REQUIRED
- [x] post-221: Дроп-сети — механіка і застосування — Fink et al. (2017): еквівалентна гіпертрофія (+5.0% vs +5.3%) при 2.5× меншому часі; Goto et al. (2004) EMG-активність зберігається при зниженому навантаженні; Wackerhage et al. (2019) — метаболічний стрес вторинний механізм; практичне: аксесуарний блок при обмеженому часі; відмінність від «тренування до відмови»; clinical-safety NOT REQUIRED
- [x] post-222: Генетика і тренувальна відповідь — Yang et al. (2003 Am J Hum Genet): ACTN3 R577X, XX-генотип відсутній серед олімпійських спринтерів, але ефект-розмір для рекреаційних атлетів малий (d < 0.3, Pickering et al. 2017 JSCR); компенсаторна пластичність: Scott et al. (2006 Genomics) — кальпаїн-3 і зсув до Type I у XX-індивідів. Montgomery et al. (1998 Nature): ACE I/D — II-генотип +189% vs DD +40%; мета-аналітичний ефект ~1–2% варіації (Ahmetov & Fedotovskaya 2013). Bouchard et al. (1999): HERITAGE — 481 учасник, VO₂max приріст від <5% до >50%; heritability ~50%; жоден SNP не пояснює >2% варіації. Bloomfield et al. (2004): «non-responders» → responders при зміні протоколу. Чому фенотипічний моніторинг інформативніший за комерційне генотипування. clinical-safety NOT REQUIRED
- [x] post-223: Плацебо і ноцебо у тренінгу — Beedie et al. (2006 MSSE): плацебо-«кофеїн» підвищує продуктивність пропорційно очікуваній дозі — без фармакології. Foad et al. (2008 MSSE): crossover-дизайн розкладає ефект кофеїну на фармакологічний (~3–5%) і очікувальний (~1–3%); очікування модулює реальний фармефект в обидва боки. Noakes/Marcora — Центральний Губернатор використовує RPE як регуляторний сигнал; Merton (1954): до 30% МО у резерві при суб'єктивній відмові. Benedetti et al. (2005 J Neurosci): плацебо-анальгезія → ендогенні опіоїди, блокується налоксоном. Ноцебо: Scott et al. (1983 Pain) — очікування болю підвищує сприйняту інтенсивність незалежно від об'єктивного пошкодження. Pageaux et al. (2014 EJAP): когнітивна стомленість підвищує RPE при незмінних периферійних параметрах. Feltz & Lirgg (1998 JSEP): self-efficacy → вища EMG-активація. Ariely et al. (2008 JAMA): «ціна» плацебо → сила ефекту через ті ж опіоїдні шляхи. clinical-safety NOT REQUIRED
- [x] post-224: Довгострокова атлетична адаптація — LTAD (Long-Term Athlete Development); чим відрізняється програмування на рік від програмування на 5+ років; принцип «акумулятивного потенціалу» (Zatsiorsky); як змінюється адаптаційний відгук з тренувальним стажем і що це означає для intermediate / advanced; clinical-safety NOT REQUIRED
- [x] post-225: Ефект новачка закінчився — операційне визначення intermediate статусу; нейральна vs структурна адаптація (Sale 1988, Zatsiorsky & Kraemer 2006); чому лінійна прогресія ламається після 3–6 місяців; DUP +28.8% vs linear +14.4% (Rhea et al. 2002); MEV/MAV/MRV модель (Israetel et al. 2019); три зміни в підході і чотири поведінкові сигнали; clinical-safety PASS
- [x] post-226: Залишкові тренувальні ефекти — Issurin (2008 J Sports Med Phys Fitness): RTE timelines — сила і аеробна витривалість ~30 днів, аеробна потужність ~18 днів, вибухова сила ~5 днів; логіка блокової послідовності як deployment RTEs у правильному порядку; чому не панікувати після деload; оптимальне вікно для тестування максимуму; clinical-safety PASS
- [x] post-227: RPE calibration для self-coached атлетів — Borg (1982 MSSE) оригінальна шкала 6–20; Zourdos et al. (2016 JSCR) modified RPE 0–10 (2 RIR = RPE 8 — завжди); Helms et al. (2016 IJSPP) RPE точніший у тренованих через failure exposure; три систематичні помилки calibration; три методи: failure sets / velocity tracking / rep max testing; Colquhoun et al. (2017) авторегуляція перевершує fixed % — але лише при якісному RPE-вході; clinical-safety NOT REQUIRED
- [x] post-228: Мінімальна ефективна доза при 3 год/тиждень — Ralston et al. (2017 JSCR): 1 підхід до відмови ≥ 3 підходи для новачків; MEV = maintenance у intermediate; стратегія збереження адаптацій при 2–3 тренуваннях/тиждень; що реально втрачається при переході з 5 на 2 сесії; clinical-safety NOT REQUIRED
- [x] post-229: Motor learning у силовому тренуванні — чому «вчитися рухатись краще» і «ставати сильнішим» — різні фізіологічні процеси; Fitts & Posner (1967): когнітивна → асоціативна → автономна фаза; deliberate practice vs accumulated practice; коли свідоме управління технікою заважає продуктивності; clinical-safety NOT REQUIRED
- [x] post-230: Тренування і когнітивна функція — Hötting & Röder (2013 Neurosci Biobehav Rev): аеробне тренування і BDNF; Moreau & Bherer (2015): силовий тренінг і executive function; яка доза і тип вправ дають найбільший когнітивний ефект; механізми: IGF-1, BDNF, нейрогенез у гіпокампі; Winter et al. (2007 PNAS): acute learning window; Erickson et al. (2011 PNAS): +2% hippocampal volume; clinical-safety NOT REQUIRED
- [x] post-231: Чому «functional training» — розмита категорія — SAID principle vs transfer of training; Specificity Paradox: чим вправа «функціональніша» у загальному сенсі, тим менш специфічний тренувальний стимул; Behm & Colado (2012 J Strength Cond Res): нестабільне середовище знижує силу при рівному або меншому м'язовому напруженні; де нестабільні умови обгрунтовані (спортсмени з конкретними потребами); clinical-safety NOT REQUIRED
- [x] post-232: Механіка тапера для непрофесіонального атлета — чому тижень перед тестуванням вирішує більше, ніж більшість думає; три змінні тапера (об'єм, інтенсивність, частота); Mujika & Padilla (2000 Sports Med): зниження об'єму 40–60% при збереженні інтенсивності і частоти; короткий (5–7 днів) vs розширений (2–3 тижні) тапер; що робити з GPP-роботою; clinical-safety NOT REQUIRED
- [x] post-233: Силовий тренінг і сон — двонапрямний зв'язок; Dattilo et al. (2011 Med Hypotheses): недосип → підвищений кортизол → знижений IGF-1/тестостерон; Leproult & Van Cauter (2011 JAMA): 1 тиждень 5h сну → -10–15% тестостерону у молодих чоловіків; Kredlow et al. (2015) мета-аналіз 66 RCT зворотний напрямок; програмування при хронічному недосипі (батьки, зміна); clinical-safety PASS WITH CONDITIONS (2026-06-06)
- [x] post-234: Адаптація до тренінгу у жінок: що реально відрізняється — Gentil et al. (2016): рівний гіпертрофійний відгук відносно базового рівня; Kraemer et al. (1993): нижчий абсолютний, але рівний відносний приріст сили; менструальний цикл і варіація сили (McNulty et al. 2020 Sports Med); наслідки для програмування; clinical-safety NOT REQUIRED
- [x] post-235: Вікова динаміка силової адаптації — як змінюється нейром'язова відповідь від 20 до 70 років; Lindle et al. (1997 J Appl Physiol): темп втрати сили прискорюється після 50; Aagaard et al. (2010): satellite cell response у старших атлетів; чому тренований 50-річний виграє у нетренованого 30-річного і за якими параметрами; clinical-safety NOT REQUIRED
- [x] post-236: Concurrent training без interference — логіка розподілу аеробного і силового тренінгу для атлетів, що хочуть і силу, і витривалість; Coffey & Hawley (2007 Eur J Sport Sci): AMPK-mTOR conflict window ≈ 6 год; Hickson (1980) original interference effect; Wilson et al. (2012 meta-analysis): мінімізація при ≤ 3 кардіо-сесії і розподілі в часі; практичне програмування; clinical-safety NOT REQUIRED
- [x] post-237: Accentuated eccentric loading — тренінг вище 100% концентричного максимуму; Hody et al. (2019 Front Physiol): ексцентричний режим: нижча ЕМГ, вища механічна напруга; Roig et al. (2009 Br J Sports Med): мета-аналіз — eccentric training superior for strength gains (ES 0.85 vs 0.41); Nordic hamstring curl як benchmark; супрамаксимальні пристрої і методи; clinical-safety NOT REQUIRED
- [x] post-238: Proprioception і суглобова стабільність — як силовий тренінг формує sensorimotor контроль; Freeman et al. (1965 J Bone Joint Surg): оригінальне дослідження proprioception після травми; Bruhn et al. (2004 JSCR): силовий тренінг покращує суглобову позиційну точність; відмінність від «balance training»; наслідки для програмування реабілітації і превенції; clinical-safety NOT REQUIRED
- [x] post-239: Asymmetry і сила — коли дисбаланс ліво-право є проблемою, а коли — нормою; Bishop et al. (2018 Sports Med): мета-аналіз asym і ризик травми — зв'язок слабкий при <15%; Exell et al. (2017): нормативний діапазон; де одностороннє тренування обгрунтоване; practical threshold для атлетів і рекреаційних тренуючихся; clinical-safety NOT REQUIRED
- [x] post-240: Warm-up варіативність — чому один warm-up не підходить для всіх цілей; динамічний vs статичний стретчинг перед силовою роботою (Behm & Chaouachi 2011 Appl Physiol Nutr Metab); PAP primer sets; temperature-dependent viscoelastic properties; протокол за типом тренування (max strength / power / hypertrophy); clinical-safety NOT REQUIRED
- [x] post-241: Heat stress і тренінг — cardiovascular drift, thermoregulatory competition, fluid balance; Robergs et al. (1993): serum lactate at elevated ambient temperature; Lorenzo et al. (2010 J Appl Physiol): 10-денна heat acclimatization → plasma volume +6.5%, VO₂max +5%; як регулювати навантаження у спеку без втрати адаптаційного стимулу; clinical-safety NOT REQUIRED
- [x] post-242: Оптимальна варіативність вправ — нудьга і специфічність: де баланс; Fonseca et al. (2014 J Strength Cond Res): різні вправи для гіпертрофії різних ділянок квадрицепса; SAID-принцип і вікно «корисної» варіації; практична матриця: ядро вправ + ротація аксесуарів; clinical-safety NOT REQUIRED
- [x] post-243: Тренувальний журнал як діагностичний інструмент — що записувати, чого не треба і як читати тренд; RPE-drift як early warning; load-volume tracking; common logging errors; чому тренувальний журнал ≠ мотиваційний щоденник; clinical-safety NOT REQUIRED
- [x] post-244: Тренування при обмеженому обладнанні — методологія вибору замінних вправ; функціональна класифікація рухів (push/pull/hinge/squat/carry); як зберегти training stimulus при переході з штанги на гантелі або на власну вагу; часті помилки при «домашньому» тренуванні; clinical-safety NOT REQUIRED
- [x] post-245: Темп повторень і гіпертрофія — TUT як вторинна змінна; ексцентрична фаза і пікова напруга; суперповільне тренування vs традиційний підхід (Schoenfeld & Grgic 2021, Fielding 2002, Keeler 2001); вибухова концентрика і потужність; паузи у точці пікового розтягнення (Oranchuk 2019); практична схема темпу 3-0-X-0; clinical-safety NOT REQUIRED
- [x] post-246: Адаптація сухожиль до тренінгу — колагеновий синтез і його часова динаміка; ексцентрика та ізометрія як стимул ремоделювання (Bohm 2015, Fouré 2010, Kjaer 2004); дисбаланс між м'язовою і сухожильною адаптацією; жорсткість і гістерезис; практичні орієнтири прогресії; клітинний механізм тендинопатії; clinical-safety NOT REQUIRED
- [x] post-247: Кутова специфічність силових адаптацій — трансферна зона ±15–20° (Thépaut-Mathieu 1988, Kitai & Sale 1989); архітектурна кутова специфічність (Noorkõiv 2015 Eur J Appl Physiol); full vs partial ROM — Bloomquist (2013 Eur J Appl Physiol) і McMahon (2018 PeerJ); ізометрика у довгому положенні і поперечний переріз (Schott 1995); динамічне vs ізометричне і жорсткість сухожилля (Kubo 2006); нейронний і структурний механізм; практичні орієнтири для програмування; clinical-safety NOT REQUIRED
- [x] post-248: Механічна напруга і mTORC1 — основний механізм гіпертрофії; Schoenfeld (2010) 3-factor model (механічна напруга, метаболічний стрес, м'язове ушкодження); Wackerhage et al. (2019 J Appl Physiol): FAK → mTORC1 → p70S6K1/4E-BP1 → MPS; Lasevicius et al. (2019 J Strength Cond Res): 20–80% 1RM до відмови — еквівалентна гіпертрофія; Mitchell et al. (2012 J Appl Physiol): 30% vs 80% 1RM до відмови; Burd et al. (2012 J Physiol): MPS довше при proximity to failure; ієрархія: близькість до відмови > навантаження; метаболічний стрес і м'язове ушкодження як вторинні модулятори; clinical-safety NOT REQUIRED
- [x] post-249: Частота тренувань і синтез м'язового протеїну
- [x] post-250: Нерегулярний розклад і силовий тренінг — SRA-крива і календарна незалежність; DUP vs лінійна (Rhea & Alderman 2004 мета-аналіз); Dankel et al. (2017 Eur J Appl Physiol): ефект частоти зникає при контролі обсягу; Colquhoun et al. (2018 J Strength Cond Res): 3× vs 6×/тиждень при рівному обсязі; Helms et al. (2018 PeerJ): autoregulation і психологічна вартість дотримання; Bickel et al. (2011 Med Sci Sports Exerc): 1/3 обсягу при збереженні інтенсивності підтримує адаптації 32 тижні; матриця 3/2/1/0 тренувань; тижневий якісний обсяг як якірна змінна; clinical-safety NOT REQUIRED
- [x] post-251: Планування тренувального року — чому аматори думають тижнями, а потрібно місяцями; Mujika & Padilla (2000 Sports Med): зниження обсягу після накопичувального блоку «виплачує» адаптації; Issurin (2010 Sports Med): блокова та хвильова структура макроциклу; чотири фази аматорського макроциклу; різниця між делоадом і перехідним блоком; управління психологічним ресурсом через рік; практичний річний шаблон без змагань; clinical-safety NOT REQUIRED
- [x] post-252: Активне відновлення проти пасивного відпочинку — що реально прискорює регенерацію; Peake et al. (2017 J Appl Physiol): запальна відповідь після силового навантаження — функціональний процес; Wiltshire et al. (2010 Med Sci Sports Exerc): масаж не прискорює кліренс лактату; Cheung et al. (2003 Sports Med): легка активність послаблює суб'єктивне DOMS, але не структурне відновлення; Wilson et al. (2012 J Strength Cond Res): ризик інтерференції Zone 2 при >3×/тиж або ≥45 хв; практична матриця; clinical-safety NOT REQUIRED
- [x] post-253: Доза-відповідь у силовому тренінгу — крива Krieger (2010 J Strength Cond Res): 1 vs 2–3 vs 4+ сети (+46% і +13%); Schoenfeld, Ogborn & Krieger (2017 J Strength Cond Res): тижневий об'єм і гіпертрофія, нелінійна крива; Ralston et al. (2017 J Strength Cond Res): специфічність ліфту > загальний об'єм для 1RM; Bickel et al. (2011 Med Sci Sports Exerc): 1/3 об'єму при збереженні інтенсивності підтримує адаптації 32 тижні; MEV/MRV як динамічні концепти; diminishing returns після 10–12 сетів/тиждень; clinical-safety NOT REQUIRED
- [x] post-254: Вибухова сила і гіпертрофія — чи сумісні? Frost, Cronin & Newton (2016 J Strength Cond Res): балістична vs повільна група — гіпертрофія -50% у балістичній, RFD +22% vs +7%; Newton et al. (1996 J Appl Physiol): EMG-профіль рекрутування Type II волокон; Tillin & Bishop (2009 Sports Med): RFD переважно нейральний; Alegre et al. (2006): fascicle length vs pennation angle при різних типах тренінгу; Cormie et al. (2011 Sports Med): plyometrics без interference за умов управління об'ємом; PAP contrast sets (Robbins 2005, Seitz & Mitchell 2014); блокова структура гіпертрофія → сила → потужність. clinical-safety NOT REQUIRED
- [x] post-255: Ментальна втома і фізична продуктивність — Marcora, Staiano & Manning (2009 J Appl Physiol): -15.3% витривалості при тому самому RPE/ЧСС/лактаті після AX-CPT; психобіологічна модель Marcora (2008); Pageaux & Lepers (2018 Sports Med) огляд 22 досліджень: витривалість -12–18%, peak MVC мінімальний вплив; Wiehler et al. (2022 Current Biology): glutamate accumulation у lateral PFC; матриця адаптованого тренінгу при ментальній стомленості; ранкове vs вечірнє; кофеїн (Beedie & Foad 2009). clinical-safety NOT REQUIRED
- [x] post-256: Перенесення сили між вправами — SAID principle і межі специфічності; Rutherford & Jones (1986 J Appl Physiol): +200% на тренованому тренажері, +11% ізометрично — нейральна специфічність; Bloomquist et al. (2013 Eur J Appl Physiol): часткове присідання не переносить силу в глибокий ROM; Kaneko et al. (1983 Eur J Appl Physiol): швидкісна специфічність адаптацій; Abernethy & Jurimäe (1996 Eur J Appl Physiol): присідання → +24% у тязі, тяга → +11% у присяданні; Seitz, Trajano & Haff (2014 J Strength Cond Res): з ростом стажу перенесення знижується; Carroll et al. (2006 Exerc Sport Sci Rev): cross-education 6–10%; три механізми допоміжних вправ (структурний/нейральний/компенсаторний); критерій «слабкої ланки» для відбору допоміжки. clinical-safety NOT REQUIRED
- [x] post-257: Тренінг у спеку — теплова акліматизація і адаптація; Périard et al. (2021 Sports Med) систематичний огляд протоколів теплової акліматизації; розширення плазмового об'єму як адаптація; практика тренінгу у залі без кондиціонера влітку; відмінність від post-102 (теплова продуктивність); clinical-safety NOT REQUIRED
- [x] post-258: Ефект сумісного тренінгу для аматора — що реально відбувається, коли поєднюєш силовий тренінг і кардіо в одному тижні; AMPK/mTORC1 interference і чому він менш критичний для аматорів; Murach & Bagley (2016 Sports Med) огляд: interference real but manageable; практична матриця: коли і скільки; Wilson et al. (2012) мета-аналіз; відмінність від post-47 і post-78 (глибший практичний фокус для непрофесіонала). clinical-safety NOT REQUIRED
- [x] post-259: Ізометричний тренінг — де він дійсно працює; специфічність кута суглоба як основне обмеження і основна перевага; реабілітаційне застосування (Naugle et al. 2012: isometrics as analgesic for tendinopathy); Lum & Barbosa (2019 Front Physiol) огляд ізометричних протоколів; де ізометрика перевершує динамічний тренінг і де — ні. clinical-safety NOT REQUIRED
- [x] post-266: Плато у тренуванні — операційна діагностика застою і три виходи — принцип акомодації (Zatsiorsky & Kraemer 2006); три причини плато: акомодація до стимулу, накопичена втома, дефіцит специфічного об'єму; Rhea et al. (2002 JSCR): DUP +28.8% vs linear +14.4%; Bosquet et al. (2007): tapering +2–3% без змін програми; Rutherford & Jones (1986): нейральна специфічність вправи; Banister (1975) fitness-fatigue model; Meeusen et al. (2013): збільшення навантаження не виходить із хронічного дисбалансу; практичний протокол 4 тижні; clinical-safety PASS
- [x] post-267: Тренування без плану — чим відрізняється хаотична активність від структурованого стимулу і коли перша прийнятна; SRA-крива і мінімальні вимоги до тренувального стимулу; Dankel et al. (2017 Eur J Appl Physiol): яка частота і об'єм мають значення; чому «просто рухатись» є валідною стратегією для підтримки, але не для прогресу; clinical-safety NOT REQUIRED
- [x] post-268: Психологія відмови в підході — Marcora (2010): відчуття зусилля, а не м'язова відмова, зупиняє підхід; Merton (1954): до 30% мотонейронів у резерві при суб'єктивній відмові; Central Governor і регуляція темпу; практичні наслідки для RPE-calibration і роботи до відмови; без мотиваційного контенту — суто механіка рішення зупинитись; clinical-safety NOT REQUIRED
- [x] post-269: Перший підхід і разминковий ефект — де межа між warm-up і робочим підходом; PAP activation sets vs рамкові розминкові підходи; кінетика температурної адаптації м'яза (Bennett 1985, Ranatunga 1998); чому перший робочий підхід майже завжди підпорогового RPE і що з цим робити; практичний протокол warm-up за типом тренування; clinical-safety NOT REQUIRED
- [x] post-270: Синдром «ідеальної програми» — чому пошук кращої програми є формою прокрастинації; Adherence coefficient математика: 90% виконання посередньої програми перемагає 50% виконання оптимальної; Ogasawara et al. (2013 Eur J Appl Physiol): 3-тижнева перерва не знищує гіпертрофічний приріст; Rhea et al. (2002): DUP vs linear — реальна різниця існує але вимагає виконання; Schoenfeld et al. (2017 JSCR): мінімальний поріг об'єму для гіпертрофії; тижні 4–6 як точка вразливості: нейральна фаза сповільнюється → хибний сигнал «не працює»; структурна причина program-hopping, не характер; clinical-safety NOT REQUIRED
- [x] post-271: Скільки відпочивати між підходами — наука проти залу: Schoenfeld et al. (2016 JSCR): 1 хв vs 3 хв — 3-хвилинна група +8.6% vs +3.1% CSA квадрицепса; Harris et al. (1976): PCr-ресинтез: 50% за 30 с, ~95% за 2–3 хв; Ahtiainen et al. (2005): еквівалентна гіпертрофія при 2 хв і 5 хв при рівному об'ємі; критика гіпотези метаболічного стресу (GH-сплеск без механічного напруження не транслюється); операційна матриця: сила 3–5 хв, гіпертрофія 2–3 хв; clinical-safety NOT REQUIRED
- [x] post-272: Цикл розтягування-скорочення (SSC) — як тіло зберігає і повертає пружну енергію: Komi & Bosco (1978, 1979): Series Elastic Component і внесок у вихід зусилля; міотатичний рефлекс розтягування; швидкий SSC (<250 мс) vs повільний SSC; Flanagan & Comyns (2008 SCJ): Reactive Strength Index як операційний показник; Taube et al. (2012): паралельні нейральні і механічні адаптації; Kyröläinen et al. (2005): SSC-ефективність незалежна від максимальної сили; прогресія drop jump і RSI-орієнтований підбір висоти; clinical-safety NOT REQUIRED
- [x] post-273: Соціальна фасилітація у тренінгу — чому ти піднімаєш більше (або менше), коли хтось дивиться: Triplett (1898): перший задокументований ефект; Zajonc (1965 Science): arousal + домінантна відповідь — механізм суперечності; Bond & Titus (1983 Psych Bull): мета-аналіз 241 дослідження (24 000+ учасників) — ефект реальний, залежить від складності задачі; Cottrell et al. (1968): evaluation apprehension як медіатор; audience vs coaction; Baron (1986): дистракційно-конфліктна теорія; операційна матриця: зал vs вдома, прості vs складні рухи, максимальні спроби vs технічна робота; clinical-safety NOT REQUIRED
- [x] post-274: Трифазне тренування — послідовність ексцентрик → ізометрик → концентрик блоків: Cal Dietz triphasic model; ексцентричні адаптації (саркомерогенез Lynn & Morgan 1994, Reeves 2009, Brockett 2001, сухожилля Malliaras 2013); ізометричні адаптації (кутова специфічність Tillin & Bishop 2009, жорсткість Bojsen-Møller 2005); концентрична фаза і RFD; логіка блокової послідовності Issurin (2010); 9-тижнева версія для аматора; clinical-safety NOT REQUIRED
- [x] post-275: Compensatory Acceleration Training (CAT) — навмисна швидкість при субмаксимальних вагах: Fred Hatfield (1989) F=ma логіка; size principle і HTMU (Henneman 1965); rate coding і doublet discharges (Van Cutsem 1998 — RFD +53%); намір в ізометрику дає адаптацію (Behm & Sale 1993); крива сила-швидкість Hill (1938); 4-зонна таблиця; CAT vs VBT цільові м/с; прогресія 3 місяці; clinical-safety NOT REQUIRED
- [x] post-276: Чому перший місяць у залі обманює — нейральна адаптація vs структурна і що означають перші «результати»: Moritani & deVries (1979 Am J Phys Med): 8-тижневий експеримент — перші 3–4 тижні приріст сили майже виключно нейральний, без збільшення площі поперечного перерізу; Sale (1988 Med Sci Sports Exerc): EMG-активація, рекрутування і синхронізація — три нейральні механізми; Zatsiorsky & Kraemer (2006): новачок і intermediate — різні рамки для оцінки прогресу; практичний висновок: що означає «бачити результати» на тижні 3 vs тижні 12; clinical-safety NOT REQUIRED
- [x] post-277: Детренінг і повернення — скільки реально втрачається і за який час: Mujika & Padilla (2000 Med Sci Sports Exerc): сила тримається 4 тижні, гіпертрофія — 3 тижні, аеробна витривалість падає швидше; Häkkinen et al. (1985 Eur J Appl Physiol): нейральна деградація (EMG) випереджає структурну; Seaborne et al. (2018 Sci Rep): epigenetic memory (NCOR2/IGF1 гіпометиляція) зберігається після детренінгу; Ogasawara et al. (2013): periodic vs continuous — еквівалентна гіпертрофія за 24 тижні; Bickel et al. (2011): 1/3 об'єму достатньо для підтримки; три протоколи повернення (2 тижні / 4–8 тижнів / 2+ місяці); clinical-safety NOT REQUIRED
- [x] post-278: Суперсети — коли скорочують час і не коштують сили: Weakley et al. (2017 Eur J Appl Physiol): антагоністичні суперсети зберігають силу при скороченні часу на 27%; Simpson et al. (2022): agonist суперсети знижують об'єм у другій вправі; Murach & Bagley (2016): накопичення метаболітів; матриця: антагоністи/синергісти/агоністи — різна ціна; операційні правила для busy-athlete; clinical-safety NOT REQUIRED
- [x] post-279: Rest-pause тренінг — нейром'язова механіка внутрішньосетного відпочинку; рекрутування рухових одиниць, фосфокреатинова ресинтеза, порівняння з традиційними підходами; clinical-safety NOT REQUIRED
- [x] post-280: Тренінг у спеку — теплові адаптації і термальний стрес: Ely et al. (2010 Med Sci Sports Exerc): ядерна температура вище 38.5°C — пряме гальмування м'язового виходу; Nielsen et al. (1993 J Physiol): теплова акліматизація — збільшення об'єму плазми, нижча серцева частота, рання початок потовиділення; Nybo & Secher (2004 Prog Neurobiol): гіпертермія і центральна втома; Taylor (2014 Compr Physiol): протоколи heat acclimation; практична матриця: акліматизований vs неакліматизований; post-exercise sauna як альтернатива; clinical-safety NOT REQUIRED
- [x] post-281: Чому програма на папері рідко виглядає так само у залі — тренувальна варіабельність і autoregulation: Zourdos et al. (2016): інтраіндивідуальна варіабельність сили ±5–10% від дня до дня у тренованих; Haff & Triplett (2016): external vs internal load — що вимірюється і чому вони розходяться; Helms et al. (2018 JSCR): RPE-based programming vs % RM — реальна точність; операційна система autoregulation для практичного тренінгу; clinical-safety NOT REQUIRED
- [x] post-282: Пліометрика для силових атлетів — що додає і чого коштує: Markovic & Mikulic (2010 Sports Med): мета-аналіз — пліометрика +7.5% у вертикальному стрибку; Wilson et al. (1993 J Appl Physiol): 10-тижнева plyometric + weight training; Cormie et al. (2010 JSCR): heavy resistance vs power vs combined протоколи — RFD і пікова потужність; практичні обмеження для аматора: об'єм, відновлення, технічний поріг; clinical-safety NOT REQUIRED
- [x] post-283: Варіабельність між людьми у реакції на тренінг — чому одна і та ж програма дає різні результати: Hubal et al. (2005 Med Sci Sports Exerc): 585 учасників, 12 тижнів — від -2% до +59% CSA при ідентичному протоколі; Timmons (2011 J Appl Physiol): VO2max responders vs non-responders; генетичні полімorfізми (ACTN3, ACE) і їх обмежений предиктивний внесок; що насправді пояснює варіабельність (базовий рівень, попередній досвід, відновлення); practical takeaway: чому порівняння з іншими — нерелевантне; clinical-safety NOT REQUIRED
- [x] post-284: Тренінг із обмеженнями у часі — мінімальний ефективний протокол для зайнятого: Schoenfeld & Grgic (2020 J Strength Cond Res): 3 підходи двічі на тиждень = значуща гіпертрофія; Ralston et al. (2017 JSCR): мала кількість підходів ефективна при підвищенні інтенсивності; Burd et al. (2012): muscle protein synthesis при різних режимах; операційна матриця: 2×45 хв vs 3×30 хв vs щоденні 20 хв — що і коли має сенс; clinical-safety NOT REQUIRED
- [x] post-285: Зовнішній vs внутрішній фокус уваги — де думати під час тренінгу: Wulf (2007): internal vs external focus визначення; Wulf et al. (2001 RQES): Constrained Action Hypothesis; Calatayud et al. (2016 Eur J Hum Mov): internal focus → вищий EMG pec/deltoid при 60% 1RM; Snyder & Fry (2012 JSCR): словесна інструкція змінює EMG до 22%; Schoenfeld & Contreras (2016 Strength Cond J): practical framework; practical cues для присідання/жиму/тяги/curl; clinical-safety NOT REQUIRED
- [x] post-286: Дихальна механіка і силовий тренінг — IAP, belt, Valsalva і що насправді захищає хребет: McGill et al. (1990 J Biomech): внутрішньочеревний тиск і компресія хребта — механіка та межі; Harman et al. (1989 Med Sci Sports Exerc): пояс vs без пояса — EMG і IAP порівняння; Lander et al. (1992 Med Sci Sports Exerc): belt і оцінка сили у присіданні; Cressey (2009): коли пояс допомагає і коли заміщає брак контролю; операційна схема: хто і коли використовує Valsalva без ризику; clinical-safety NOT REQUIRED
- [x] post-287: Синергетичні патерни м'язів — ЦНС управляє групами, а не окремими м'язами; координація рухових одиниць, м'язові синергії у фундаментальних рухових патернах; clinical-safety NOT REQUIRED
- [x] post-288: Торако-люмбальна фасція і передача зусилля — що відбувається між широчайшим і сідницею при важких підйомах; fascial tension system і стабілізація хребта; clinical-safety NOT REQUIRED
- [x] post-289: Специфічність швидкості у силовому тренінгу — чому темп вправи визначає тип адаптації; velocity continuum, кутова специфічність, практичні наслідки для вибору темпу; clinical-safety NOT REQUIRED
- [x] post-290: Feedforward і антиципаторні постуральні корекції — що ЦНС робить до того, як навантаження прикладене; APAs у важкому тренінгу, проактивна стабілізація vs реактивна; clinical-safety NOT REQUIRED
- [x] post-291: Адаптація, яку не бачиш — нейральна vs структурна фаза, саркоплазматична надбудова (Haun 2019), міоядерний домен (Bamman 2007), парадокс шостого тижня (Ogasawara 2013), епігенетична пам'ять (Seaborne 2018), практичні маркери ранньої адаптації; clinical-safety NOT REQUIRED
- [x] post-292: Технічна відмова vs м'язова відмова — де зупиняти підхід: Calatayud et al. (2016) і зміна м'язової активації при погіршенні техніки; практичний протокол «зупинка до технічного збою»; чому компенсаторні рухи — не прогрес; відмінність від психологічної відмови (post-268); clinical-safety NOT REQUIRED
- [x] post-293: central-vs-peripheral-fatigue — центральна vs периферична втома; clinical-safety NOT REQUIRED
- [x] post-294: lengthened-position-hypertrophy — гіпертрофія в подовженому положенні, тітін, Maeo 2021; clinical-safety NOT REQUIRED
- [x] post-295: strength-plateau-vs-life-plateau — плато сили vs плато обставин; 4 діагностичних питання, матриця RPE-drift, протоколи для обох сценаріїв; clinical-safety PASS
- [x] post-296: RPE-калібрування без зовнішнього тренера — суб'єктивне vs об'єктивне зусилля; систематичне викривлення RPE (недооцінка у початківців, переоцінка при перевтомі); практика: anchor sets, velocity reference, відеопорівняння; clinical-safety NOT REQUIRED
- [x] post-297: Compliance premium — чому «достатньо добра» програма, яку ти виконуєш, б'є ідеальну, яку ти кидаєш; дані по adherence у несупервайзованому тренінгу; що насправді визначає результат за 12 місяців; clinical-safety NOT REQUIRED
- [x] post-298: Тренінг при нестабільному розкладі — не мотиваційний BS; мінімально ефективний протокол при ≤2 сесій/тиждень; як програмувати коли дні непередбачувані; рефрейм «провального тижня»; clinical-safety NOT REQUIRED
- [x] post-299: Мінімальний ефективний протокол без обладнання (відрядження, no-gym) — дані по детренуванню за 3–7 днів, три базові рухові патерни, прогресія через tempo/паузи/unilateral; clinical-safety NOT REQUIRED
- [x] post-300: Ієрархія втрат при детренуванні — тайм-константи VO2max/гіпертрофія/сила/нейропаттерн, міонуклеарна пам'ять, нейральна ефективність, тайм-лайн повернення по тривалості паузи; clinical-safety PASS
- [x] post-301: Силовий тренінг і когнітивна продуктивність — двонаправлений зв'язок; BDNF (гострий пік 15–30 хв, механізми IGF-1/irisin/катехоламіни); гострі vs хронічні ефекти; executive function в RCT (Colcombe & Kramer 2003 d=0.68, Cassilhas 2007, Best 2014, Northey 2018); дозо-відповідь: 2–3×/тиж., 50% vs 80% 1RM comparable; чесні обмеження; clinical-safety NOT REQUIRED
- [x] post-302: Суб'єктивна vs об'єктивна готовність — коли слухати тіло, коли ігнорувати; allostatic load vs поточна втома; коли HRV бреше і коли «просто лінь» — реальний діагностичний алгоритм; clinical-safety NOT REQUIRED
- [x] post-303: Програмування для самокоуча — мінімальна документація; що треба логувати, що ні; як тренувальний журнал стає діагностичним інструментом, а не задачею для галочки; clinical-safety NOT REQUIRED
- [x] post-304: Темп і ефективний RM — як контроль швидкості руху змінює дозування навантаження; tempo notation (4010, 2111 тощо), практичні наслідки для програмування гіпертрофії; clinical-safety NOT REQUIRED
- [x] post-305: Силовий тренінг і довголіття — доза-відповідь між об'єм/частота силового тренінгу і all-cause mortality; оптимум за даними когортних досліджень; де перетинаються «тренуватись для здоров'я» і «тренуватись для сили»; clinical-safety NOT REQUIRED
- [x] post-306: Прогресія для жінок — чи відрізняється оптимальне програмування гіпертрофії і сили за статтю; гормональний контекст, відмінності у відновленні, де дані надійні і де їх немає; clinical-safety NOT REQUIRED
- [x] post-307: Тренінг і сон — двонаправлений зв'язок; як порушення сну змінює тренувальну адаптацію, і як тренінг покращує архітектуру сну; клінічно безпечна рамка без поради щодо сонних добавок; clinical-safety NOT REQUIRED
- [x] post-308: Сила у нестандартних положеннях — чому unilateral і offset-loading розвивають нейром'язову стабільність, яку білатеральний тренінг не покриває; carry patterns, asymmetrical loading, clinical-safety NOT REQUIRED
- [x] post-309: Програмування для батьків маленьких дітей — не «як знайти мотивацію», а як реорганізувати структуру тренінгу при хронічному недосипі, нестабільному розкладі і дефіциті часу; clinical-safety NOT REQUIRED
- [x] post-310: Суперсети і ефективність тренування — pre-exhaust (Brentano 2017: ЕМГ не зростає, об'єм падає), mechanical drop sets (механіка кутової специфічності), PAP через антагоністичні пари (Robbins 2010); матриця вибору техніки за метою; clinical-safety NOT REQUIRED
- [x] post-311: Внутрішньочеревний тиск і брейсинг — IAP як механічна система стабілізації (не вправа на прес); brace vs. draw-in (Grenier & McGill 2007: bracing значно вища жорсткість хребта); антиципаторна активація ТА (Cresswell 1992, Hodges & Richardson 1997); Valsalva механіка і безпека; пояс як підсилювач IAP; clinical-safety NOT REQUIRED

#### Roadmap post-312+

- [x] post-312: Стандарти сили — чи є «достатньо сильний» і що говорять Wilks/DOTS/відносна сила про реалістичні цілі для різного тренувального стажу; clinical-safety NOT REQUIRED
- [x] post-313: Дихання під час підходу — VAP, Valsalva в субмаксимальних, де затримка дихання виправдана і де надмірна; breathing pattern у багатоповторних підходах; clinical-safety NOT REQUIRED
- [x] post-314: Памп і клітинне набрякання — метаболічний стрес як механізм гіпертрофії (Schoenfeld 2010/2013 cell swelling hypothesis), BFR як природний експеримент, де докази слабнуть, памп як діагностика проти пампу як мети; clinical-safety NOT REQUIRED
- [x] post-315: Де закінчується новачок — ознаки завершення новачкового ефекту, що змінюється в програмуванні, частоті і відновленні після переходу на intermediate; clinical-safety NOT REQUIRED
- [x] post-316: Варіативність відповіді на тренінг — чому однакова програма дає різні результати у різних людей; генетика vs конситенсі; що реально під контролем; clinical-safety NOT REQUIRED
- [x] post-317: Ексцентричний акцент без спеціального обладнання — практичні методи перевантаження ексцентричної фази при звичайному тренінгу; механіка і застосування; clinical-safety NOT REQUIRED
- [x] post-318: Адаптаційна стеля і як зрозуміти, що ти до неї наближаєшся — тренувальний стаж, темп прогресії, де межа між «ще ростемо» і «вже біля максимуму»; clinical-safety NOT REQUIRED
- [x] post-319: Ізометричний тренінг — де сила не рухається, але зростає; кутова специфічність, IMTP та intra-repetition isometrics, клінічні застосування і практичне програмування; clinical-safety NOT REQUIRED
- [x] post-320: Чому відчуття «легко» не означає «мало» — суб'єктивне сприйняття зусилля і реальний тренувальний стимул; RPE-дрейф, звикання, і коли «легко» треба поважати; Borg (1970/1982), Hampson et al. (2001) efference copy, Pageaux &amp; Lepers (2018) когнітивна втома, Zourdos et al. (2016) RIR r=0.78, Helms et al. (2016), Smits et al. (2014) звикання, Roberts et al. (2018) residual fatigue; clinical-safety NOT REQUIRED
- [x] post-321: Пік сили vs силова витривалість — два окремих фенотипи, різні механізми адаптації, різне програмування; де вони перетинаються і де ні; clinical-safety NOT REQUIRED
- [x] post-322: Тренінг з нестабільним розкладом — як будувати прогресію, коли немає передбачуваних трьох днів на тиждень; мінімальні ефективні стратегії, пріоритизація; clinical-safety NOT REQUIRED
- [x] post-323: Жорсткість сухожиль і передача сили — чому слабкі сухожилля обмежують силовий вихід незалежно від м'язової маси; протоколи адаптації; clinical-safety NOT REQUIRED
- [x] post-324: Пасивні обмеження рухливості — де реально «стоїть» мобільність і чому розтяжки недостатньо; капсула vs фасція vs нейральне гальмування; Weppler &amp; Magnusson (2010) stretch tolerance vs mechanical stiffness, Aquino et al. (2010) нейральне гальмування hamstrings, Vicenzino et al. (2011) суглобова мобілізація; clinical-safety NOT REQUIRED
- [x] post-325: Силовий тренінг і кісткова щільність — механотрансдукція у кістковій тканині, осьове навантаження vs. ударне, доза-відповідь для BMD, Wolff's law у сучасному прочитанні; Duncan & Turner (1995), Watson et al. LIFTMOR (2018); clinical-safety NOT REQUIRED
- [x] post-326: Сон і тренувальна адаптація — GH/SWS cascade, Van Cauter (2000/2010), Dattilo (2011) sleep debt → катаболічний дисбаланс, Milewski (2014) 1.7× injury risk, Mah (2011) sleep extension; clinical-safety NOT REQUIRED
- [x] post-327: Білатеральний дефіцит — чому двосторонній MVC < сума унілатеральних; інтерлімбальна інгібіція, corpus callosum, кортикоспінальний драйв; Howard & Enoka (1991), Koh et al. (1993), Jakobi & Cafarelli (1998); наслідки для тестування, виявлення асиметрій і програмування унілатерального тренінгу; clinical-safety NOT REQUIRED
- [x] post-328: Аеробна база і силовий тренінг — чому VO₂ capacity важлива для силовика; PCr resynthesis залежить від окисного фосфорилювання; Bogdanis et al. (1995), MacDougall et al. (1988); мінімальна аеробна доза для силового атлета; clinical-safety NOT REQUIRED
- [x] post-329: Тестування 1RM — коли, навіщо і як не помилитися в протоколі; тест vs е1RM де формули помиляються (гіпертрофійний блок, новачки, зміна техніки); рамп-протокол розминки (~8 підходів до максимуму); правило трьох спроб і вибір ваги; відновлення 48–72h (Linnamo 1998); частота по стажу; Haff & Triplett (NSCA 2016), Aagaard et al. (2002), Zourdos et al. (2016), Bogdanis et al. (1995); clinical-safety NOT REQUIRED
- [x] post-330: RPE та RIR — авторегуляція тренування без зовнішніх приладів; шкала Tuchscherer, RIR як зворотній бік RPE, три причини чому фіксований % від 1RM системно помиляється; Zourdos et al. (2016) JSCR r=0.78, Helms et al. (2016) IJSPP; clinical-safety NOT REQUIRED
- [x] post-331: HRV для самокерованого атлета — RMSSD як проксі вагусного тонусу (Buchheit 2014 EJAP); базова лінія і коридор ±1 SD; Polar H10 vs оптичний датчик; три помилки інтерпретації; протокол ранкового вимірювання; clinical-safety NOT REQUIRED
- [x] post-332: Сила і гіпертрофія одночасно — чесний розбір; Schoenfeld et al. (2017) JSCR рівна гіпертрофія в 6–12 і 25–35 при вирівняному обсязі; де конфлікт реальний (нейральна специфічність > 85%); DUP vs powerbuilding; матриця рішень по стажу; Rhea et al. (2002) DUP +28% vs +14%; clinical-safety NOT REQUIRED

#### Roadmap post-333+

- [x] post-333: Типи м'язової відмови — м'язова vs технічна vs метаболічна відмова; матриця RIR для базових/ізольованих вправ; коли тренуватися до відмови, коли ні; clinical-safety NOT REQUIRED
- [x] post-334: Тренувальний спліт: full-body, upper/lower або PPL — порівняльна матриця і рекомендація для середньостажованого; MPS-вікно і частота стимулу; clinical-safety NOT REQUIRED
- [x] post-335: Як читати тренувальне дослідження: п'ять фільтрів для самокерованого атлета — population match, тривалість, effect size vs p-value, ecological validity, publication bias; Cohen (1992), Hecksteden et al. (2015), Ioannidis (2005); clinical-safety NOT REQUIRED
- [x] post-336: Мінімальна підтримуюча доза — скільки тренінгу достатньо щоб не втрачати; Bickel et al. (2011) 1/9 обсягу підтримує 100% сили; наслідки для зайнятих атлетів і паузи; clinical-safety NOT REQUIRED · написано як post-367
- [ ] post-337: Тренувальний стаж проти хронологічного віку після 40 — де вікові обмеження реальні, де перебільшені; Wroblewski et al. (2011) Master Athletes MRI; чому досвідчений 45-річний обганяє новачка 25-річного; clinical-safety NOT REQUIRED
- [x] post-338: Тренування без партнера — класифікація ризику по вправах, RIR як параметр управління ризиком, налаштування страхувальних упорів, roll of shame обмеження, матриця вправа/ризик/рішення; Aasa et al. (2017) BJSM, Helms et al. (2016) IJSPP; clinical-safety NOT REQUIRED
- [x] post-339: Вибір вправ: мінімальний набір для покриття гіпертрофійного стимулу — шість патернів руху (горизонтальний тяг/жим, вертикальний тяг/жим, шарнір, присід), evidence for exercise variety vs familiarity (Fonseca et al. 2014); clinical-safety NOT REQUIRED
- [x] post-340: Час тренування і циркадна ритміка — чи має значення ранок vs вечір для адаптації; Chtourou & Souissi (2012) циркадні піки сили та потужності; де гнучкість розкладу перевершує оптимізацію часу; clinical-safety NOT REQUIRED
- [x] post-341: Техніка vs обсяг для початківця — проблема когнітивного навантаження в перші 6 місяців; чому неможливо одночасно оптимізувати техніку і прогресивне навантаження; Fitts & Posner (1967) cognitive/associative/autonomous stage і bandwidth; clinical-safety NOT REQUIRED
- [x] post-342: Тренування при хронічному болі в попереку — різниця між дискогенним, фасетковим і м'язово-спастичним механізмом; механізми як освітній контекст (не самодіагностика); McGill (2007) spine stability, Steele et al. (2015) lumbar extension (дослідницький контекст); clinical-safety PASS WITH CONDITIONS — reviewed 2026-06-08
- [x] post-343: Ментальне відновлення і тренінг — cognitive fatigue як незалежний фактор зниження продуктивності (Marcora et al. 2009 EJAP); чи можна тренуватися «на вичерпаній батарейці»; RPE drift після когнітивного навантаження; clinical-safety NOT REQUIRED
- [x] post-344: Повернення до силового тренінгу після пологів — ремоделювання колагену (тип III → I), діастаз прямих м'язів без самодіагностики, IAP-координація і тазове дно, RCOG/POGP (2021) клінічний консенсус як фреймворк для розмови з провайдером; clinical-safety PASS WITH CONDITIONS 2026-06-08
- [x] post-345: Тренування під час подорожей і відряджень — Issurin (2008) резидуальні ефекти (сила 25–35 днів), Bickel et al. (2011) 1/9 обсягу = maintenance, RIR 1–2 як ключова умова, готельний протокол 40 хвилин, матриця рішень за тривалістю, чому повернення ≠ надолуження; clinical-safety NOT REQUIRED
- [ ] post-346: Тренування і менструальний цикл для практика — фолікулярна vs лютеальна фаза в контексті програмування: коли підвищувати інтенсивність, коли знижувати, що реально каже доказова база vs популярні «фазові протоколи»; ⚠️ clinical-safety required before publish
- [x] post-347: Аеробна база і силовий тренінг — чому «кардіо вбиває м'язи» — застарілий міф; Zone 2 як база VO2max, вплив аеробної ємності на відновлення між підходами і сесіями, практична інтеграція без interference; clinical-safety NOT REQUIRED
- [ ] post-348: Навантаження на сухожилля і тендинопатія — як розрізнити адаптивний стрес від патологічного; reactive vs degenerative tendinopathy (Malliaras et al.); ізометрія як перший рядок; clinical-safety PASS pending review
- [ ] post-349: Програмування для підлітків — відмінності анаболічної відповіді до і після статевого дозрівання, чому «підлітки не повинні піднімати важке» — міф; ACSM/NSCA youth strength guidelines; clinical-safety required
- [x] post-350: Самостійне ведення тренувального журналу — RPE-трекінг, 4 ключових параметри на підхід, розпізнавання стагнації vs плато; без додатків, лише папір або нотатник
- [x] post-351: Зворотна піраміда (RPT) — нейром'язова свіжість як ресурс, логіка важкого підходу першим, back-off сети для об'єму, авторегуляція через RPE; для базових рухів і проміжного стажу; clinical-safety NOT REQUIRED
- [x] post-352: Програмування після повернення з тривалої паузи — розрізнення детренованості м'язової vs нейральної, Bickel (2011) maintenance MED, Seaborne (2018) міонуклеарна пам'ять; як не «надолужувати» і не отримати DOMS-травму; clinical-safety NOT REQUIRED
- [x] post-353: Тренування при хронічному недосипанні — Knowles et al. механізм; що знижується першим (максимальна сила, мотивація, сприйняття зусилля); мінімальний протокол для підтримки адаптації; clinical-safety NOT REQUIRED
- [x] post-354: Силовий тренінг і мобільність — чому «розтяжка після тренування» не будує мобільність і що насправді змінює функціональний діапазон; тренінг в крайніх точках амплітуди як самодостатній інструмент; clinical-safety NOT REQUIRED
- [x] post-355: Паузовані повторення і мертва зупинка — виключення SSC як тренувальний інструмент; sticking point діагностика та ізоляція; Lichtwark & Wilson (2005) пружна енергія, Seynnes et al. (2009) сухожильна жорсткість, Bloomquist (2013) повна амплітуда; матриця: вправа × де зупинятись × навіщо; clinical-safety NOT REQUIRED
- [x] post-356: Частота заміни вправ — коли перемикатись і коли це зрада прогресу; SRA-крива для технічного навчання; variety vs specificity trade-off; Fonseca et al. (2014) exercise variety and hypertrophy; clinical-safety NOT REQUIRED
- [x] post-357: Темп прогресу у різних ліфтах — чому жим лежачи прогресує повільніше за присід; lever arms, м'язова маса, тренувальна економія; як керувати очікуваннями по руху; clinical-safety NOT REQUIRED
- [x] post-358: Самопрограмування vs готова програма — матриця вибору по стажу і цілях; коли template перевершує custom і навпаки; де помилки програмування стають системними; clinical-safety NOT REQUIRED
- [x] post-359: Тренування в умовах теплового стресу — cardiovascular drift, thermoregulatory threshold, WBGT-матриця, протоколи акліматизації; Nybo & Nielsen (2001) центральна теплова втома; O'Brien (2014) +7% VO₂max; Chtourou & Souissi (2012) циркадні ефекти; пасивна акліматизація сауна; clinical-safety REVIEWED PASS (MD-referral вбудований у матрицю)
- [x] post-360: Нейральна ефективність vs гіпертрофія: як розрізнити джерело сили що зростає — Moritani & deVries (1979) класичний EMG-експеримент; часова шкала нейральної та структурної фаз; що означає «я стаю сильнішим, але не більшим» і навпаки; clinical-safety NOT REQUIRED · написано як post-369
- [x] post-361: Тренування і кофеїн — що підтверджено дослідженнями і що є маркетингом; Astorino & Roberson (2010) мета-аналіз; толерантність і wash-out; дозування і вікно ефекту; clinical-safety NOT REQUIRED · написано як post-370
- [x] post-362: Як читати дослідження про тренування — ієрархія доказів для практика; RCT vs observational; effect size проти p-value; Schoenfeld «resistance training recommendations» як зразок; clinical-safety NOT REQUIRED
- [x] post-363: Індивідуальна варіативність відповіді на тренування — чому однакова програма дає різні результати; responder vs non-responder (Timmons 2010); гетерогенність гіпертрофічної відповіді; що це означає для програмування; clinical-safety NOT REQUIRED

#### Roadmap post-364+

- [x] post-411: Перший підхід завжди брехун — чому RPE першого робочого підходу систематично завищений; три механізми (м'язова температура, нейральна неготовність, efference copy); як читати RPE-траєкторію, а не точку; матриця рішень по двох підходах; Helms et al. (2016) RIR надійність, Naclerio et al. (2013) розминка −1.2 RPE, Chtourou & Souissi (2012) ранкові vs вечірні сесії; clinical-safety NOT REQUIRED
- [x] post-412: Надлишкова варіація вправ — як часта зміна рухів саботує прогрес; SRA-крива для технічного навчання vs м'язового розвитку; Fonseca et al. (2014) variety and hypertrophy; Shea & Morgan (1979) contextual interference; коли різноманіття допомагає, коли шкодить; clinical-safety NOT REQUIRED · v3.38.0
- [x] post-413: Тренувальний стаж після 40 — де вікові обмеження реальні, де перебільшені; Wroblewski et al. (2011) Master Athletes MRI; Distefano & Goodpaster (2018) саркопенія як синдром бездіяльності; відновлення, частота, вибір навантаження; чому досвідчений 45-річний обганяє новачка 25-річного; clinical-safety NOT REQUIRED · v3.38.0
- [x] post-414: Сила та довголіття — механізми, а не кореляції; м'язова маса як метаболічний резерв; Ruiz et al. (2008) рак, Leong et al. (2015) PURE study; що насправді вимірює handgrip і де він обмежений як предиктор; clinical-safety NOT REQUIRED · v3.38.0
- [x] post-415: Програмування для батьків — тренувальна адаптація при хронічному недосипанні і нестабільному розкладі; мінімальний ефективний обсяг, пріоритизація рухів, стратегії відновлення; Knowles et al. (2018) sleep restriction; Schoenfeld et al. (2016) frequency; clinical-safety NOT REQUIRED · v3.38.0
- [x] post-416: Частота тренувань по м'язових групах — Schoenfeld, Ogborn & Krieger (2016) мета-аналіз; МБС і вікно частоти; 2× vs 3+/тиждень; clinical-safety NOT REQUIRED · v3.39.0
- [x] post-417: Кластерні сети і внутрішньосетовий відпочинок — Tufano et al. (2016); Oliver et al. (2013); PCr-ресинтез; REST-PAUSE; clinical-safety NOT REQUIRED · v3.39.0
- [x] post-418: BFR-тренування — Loenneke et al. (2012) механізми; Hughes et al. (2017) безпека; реабілітаційне застосування; clinical-safety NOT REQUIRED · v3.39.0
- [x] post-419: Унілатеральне vs білатеральне — Behm et al. (2010) bilateral deficit; Speirs et al. (2016) трансфер; clinical-safety NOT REQUIRED · v3.39.0
- [x] post-420: Щільність тренування як змінна прогресії — Schoenfeld (2010) метаболічний стрес; EDT density blocks; clinical-safety NOT REQUIRED · v3.39.1
- [x] post-421: Прогресивне перевантаження — механізми, не мантри; типи прогресії; clinical-safety NOT REQUIRED · v3.39.0
- [x] post-422: Дилоад-тиждень — коли розвантаження є тренуванням; наука відновлення; протоколи; clinical-safety NOT REQUIRED · v3.39.0
- [x] post-423: Діапазони повторень — сила і гіпертрофія; Schoenfeld et al. (2017); strength of evidence; clinical-safety NOT REQUIRED · v3.39.0
- [x] post-424: Ефективні підходи — скільки з 20 запланованих сетів реально дають гіпертрофічний стимул; stimulating reps; Morton et al. (2016) proximity to failure; Henneman принцип; Amirthalingam et al. (2017) спадна віддача; junk volume; clinical-safety NOT REQUIRED · v3.39.2
- [x] post-425: Програмування A/B/C — SRA recovery windows, три архітектурні схеми (PPL/Upper-Lower/Squat-Hinge-Upper), аудит перекриття; clinical-safety NOT REQUIRED · v3.40.0
- [x] post-426: Тривалість тренувального блоку — Haff & Triplett мезоцикл 3–6 тижнів, Issurin блокова, Painter MEV/MRV як реальний тригер деловання; clinical-safety NOT REQUIRED · v3.40.0
- [x] post-427: Силове тренування і VO2max — Hagerman (1996) пауерліфтери і VO2max, концентрична LV гіпертрофія, Murlasits (2018) concurrent мета-аналіз; clinical-safety NOT REQUIRED · v3.40.0
- [x] post-428: Силові стандарти — Stone et al. алометрія, Wilks/IPF GL нормалізація, таблиця BW-кратних стандартів; clinical-safety NOT REQUIRED · v3.40.0
- [x] post-429: Прогресія у ізоляційних вправах — специфіка накопичення сили у односуглобних рухах, відмінність від базових, коли plateau нормальний; Moritani & deVries (1979) нейральна адаптація; математика відносного приросту; Lasevicius et al. (2019) темп гіпертрофії двоголового; Wolf et al. (2023) lengthened position; clinical-safety NOT REQUIRED · v3.40.1
- [x] post-430: Авторегуляція у силовому тренуванні — RPE і RIR як практичні інструменти для self-coached проміжних атлетів; Zourdos et al. (2016) RIR-based RPE валідація; Hackett et al. (2017) точність оцінки RIR; Helms et al. (2018) RPE vs %1ПМ; daily 1RM variance ±5–8%; калібрувальні помилки; коли авторегуляція НЕ підходить; clinical-safety NOT REQUIRED · v3.40.3
- [x] post-431: Ексцентрична фаза — механіка, гіпертрофія і практичне застосування для силових атлетів; Schoenfeld (2010) три механізми гіпертрофії; Franchi et al. (2017) архітектурні адаптації; Proske & Morgan (2001) «popping» саркомерів і DOMS; Douglas et al. (2017); Wolf et al. (2023) lengthened position; темп-нотація 2-0-1/3-1-2/4-0-1; clinical-safety NOT REQUIRED · v3.40.3
- [x] post-432: Надмірно ускладнений атлет — як інформаційне перевантаження руйнує тренувальний прогрес; Sweller (1988) cognitive load theory; Iyengar & Lepper (2000) парадокс вибору; Gollwitzer (1999) implementation intentions; program hopping SRA-cost; Miller (1956) working memory; Timmons et al. (2010) responder variability; Fonseca et al. (2014) варіативність вправ; мінімальний ефективний базис; n-of-1 logic; clinical-safety NOT REQUIRED · v3.41.1
- [x] post-433: Ізометричне тренування — кутова специфічність, сухожильна адаптація, overcoming vs yielding, PAP-застосування; Folland et al. (2005) joint-angle specificity ±15–20°; Bohm et al. (2015) колаген сухожиль yielding ізометрика; Maffiuletti et al. (2016) rate coding і нейральна активація; Sale (2002) PAP; Haff et al. (2015) IMTP; clinical-safety NOT REQUIRED · v3.41.3
- [x] post-434: Сон і атлетична продуктивність — нейробіологія відновлення; Van Cauter et al. (2000) GH-пульс і SWS; Walker (2005) REM і консолідація рухових навичок; Fullagar et al. (2015) депривація сили −8–11%; Mah et al. (2011) basketball sleep extension; Atkinson & Reilly (1996) циркадний ритм і час тренування; соціальний джетлаг Wittmann et al. (2006); clinical-safety NOT REQUIRED · v3.41.3
- [x] post-435: Механічна напруга vs метаболічний стрес — що насправді рухає гіпертрофію у 2020-х; Damas et al. (2016) м'язовий збиток не є механізмом; Morton et al. (2016) навантаження vs відмова; Schoenfeld et al. (2017) 8–12% vs 25–35% RM; Wolf et al. (2023) stretch-mediated hypertrophy; Hornberger et al. (2006) mTORC1 механотрансдукція; Pearson & Hussain (2015) BFR; clinical-safety NOT REQUIRED · v3.41.3
- [x] post-436: Таблиця Пріліпіна — радянська математика інтенсивності та об'єму; Stone, O'Bryant & Garhammer (1981) синтез; таблиця 4 зон % 1ПМ з оптимальним і допустимим сумарним об'ємом за сесію; нейральна втома і деградація якості повторення; Israetel et al. (2015) MEV/MAV/MRV як розширення; Morton et al. (2016) proximity to failure — обмеження Пріліпіна; поправки для присідання/жиму/тяги; clinical-safety NOT REQUIRED · v3.41.4
- [x] post-437: Криві сили і вибір вправ — чому штанга і гантель дають різний стимул у різних точках ROM; Behm & Sale (1993) кутова специфічність; Cameron & Adams (1994) resistance profile; три типи кривих (ascending/descending/bell-shaped); Kassiano et al. (2022) і Wolf et al. (2023) stretch-mediated hypertrophy; матриця вправ за профілем для грудних/біцепса/трицепса/стегна; коли кабельний тренажер перевершує вільні ваги; clinical-safety NOT REQUIRED · v3.42.1
- [x] post-438: Velocity-Based Training — load-velocity relationship; MCV як денний показник готовності; порогова втрата швидкості 20% (сила) vs 40% (гіпертрофія) — Pareja-Blanco et al. (2017); smartphone vs GymAware; VBT vs RPE/RIR; clinical-safety NOT REQUIRED · v3.42.3
- [x] post-439: Сухожилля і силовий тренінг — чому найповільніша тканина визначає прогрес; Magnusson et al. (2008) 3–6 місяців vs тижні для м'яза; стiffness як ключова адаптація; Type I vs III колаген; Bohm et al. (2015) ізометрика; HSR-протокол Kongsgaard et al. (2009); правило 10%; clinical-safety NOT REQUIRED · v3.42.3
- [x] post-440: Rate of Force Development — 0–200ms вікно; early (нейральний 0–50ms) vs late RFD (100–200ms); Aagaard et al. (2002), Tillin & Bishop (2009); intent-to-move-fast; балістичні вправи; CMJ як польовий проксі; інтеграція в блок; clinical-safety NOT REQUIRED · v3.42.3
- [x] post-441: Heat і cold exposure у відновленні — Roberts et al. (2015) CWI блокує mTORC1; Laukkanen et al. (2018) сауна і серцево-судинна адаптація; Bleakley et al. (2012) Cochrane CWI+DOMS; Søberg et al. (2021) норадреналін; вікна тайменгу; clinical-safety PASS · v3.42.3
- [ ] post-442: Подвійна прогресія — систематичний підхід до просування між вагами без перестрибування; алгоритм «репи → вага → репи»; чому відсоткові схеми погано описують реальний тренувальний процес проміжного атлета; Zourdos et al. (2016) RIR-заземлення прогресії; практичний щоденник прогресії
- [ ] post-443: Силові стандарти і самооцінка прогресу — методологія порівняння без змагальних орієнтирів; алометричне масштабування Stone et al. (1991); Wilks і IPF GL коефіцієнти; три питання самооцінки для self-coached атлета; коли стандарти корисні і коли є пасткою

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
**Last update:** 10 червня 2026 · v3.29.0
