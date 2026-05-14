# Changelog · FORM

Усі помітні зміни в цьому проєкті документуються тут.
Формат базується на [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/).
Версіювання: **semver-light** — `MAJOR.MINOR.PATCH`.

| Bump | Коли |
|---|---|
| **PATCH** (`x.y.Z`) | Кожна cloud-ітерація. Один концепт = один bump. |
| **MINOR** (`x.Y.z`) | Нова фіча, новий розділ документації, помітна зміна. |
| **MAJOR** (`X.y.z`) | Брейкінг-чейндж, pivot бренду/скоупу. Не до релізу. |

---

## [Unreleased]

---

## [0.17.0] — 2026-05-14

### Why MINOR
- New document: `docs/ADVISORY_BOARD.md` — full advisory board recruitment playbook
- Closes the "Get sport-science advisor signed" gap in the roadmap: prior art was jd-sport-science-advisor.md (what the role is) but no plan for how to find and sign advisors
- First document that defines the full advisory strategy: 4 profiles, compensation structure, outreach templates, trial period protocol, agreement terms, timeline

### Added
- [`docs/ADVISORY_BOARD.md`](docs/ADVISORY_BOARD.md) — 8-section advisory board playbook: why we need advisors (3 risk categories: science drift, clinical liability, credibility signals); 4 advisor profiles (sport science HIGH priority, clinical/psychology HIGH priority, CV/ML MEDIUM, industry LOW/post-seed) each with profile criteria, anti-profile, where to find, what we ask, why they'd say yes; compensation table (0.10–0.25% equity per slot, 2yr monthly vesting no cliff, no cash pre-seed); warm intro + cold outreach templates with rules; 20-minute first call structure with red flags; 30-day trial protocol; advisory agreement key terms (FAST Agreement reference); timeline with hard deadline (Slot 1 + 2 signed before any food/body features ship — clinical-safety requirement)

### Changed
- [`VERSION`](VERSION) — 0.16.0 → 0.17.0
- [`README.md`](README.md) — ADVISORY_BOARD.md у docs tree; doc count 43+→44+; version refs → v0.17.0
- [`STATUS.md`](STATUS.md) — ADVISORY_BOARD.md у docs table; version → v0.17.0; TL;DR → v0.17.0

---

## [0.16.0] — 2026-05-14

### Why MINOR
- Two new content assets: `content/newsletter-01.md` + `content/newsletter-02.md`
- First full newsletter issues ready to publish on Beehiiv — closes the "перші 2 пости" gap from NEWSLETTER.md
- Closes the last "Newsletter setup плюс перші 2 пости" item from the roadmap; prior work (NEWSLETTER.md) was platform setup only
- Both passed clinical-safety review (PASS WITH NOTES: food logging feature needs ED screening gate; Victor correction loop needs ease-off exit path — product-level items logged, not newsletter blockers)

### Added
- [`content/newsletter-01.md`](content/newsletter-01.md) — Issue 01 "Why we're building FORM": personal founder letter (400–500 words), Victor off-duty tone, opens with in-person coach moment, draws category gap between tracker and trainer, lists what newsletters will and won't contain, beta CTA. clinical-safety: PASS.
- [`content/newsletter-02.md`](content/newsletter-02.md) — Issue 02 "What FORM actually does (and what it doesn't)": structured breakdown of 3 core features (CV form on 8 lifts, HRV-adjusted programming, Victor voice) + 5 explicit won't-dos (no medical diagnoses, no calorie surveillance, no before/after, no guarantees, no data monetization) + honest CV accuracy uncertainty section. clinical-safety: PASS.

### Changed
- [`VERSION`](VERSION) — 0.15.0 → 0.16.0

### Notes (product action items from clinical-safety)
- Food logging feature requires SCOFF or equivalent ED screening gate before a user can enable it
- Victor correction loop needs a documented ease-off exit path for users who re-attempt compulsively

---

## [0.15.0] — 2026-05-14

### Why MINOR
- New HTML surface: `jobs.html` — dedicated careers page for FE + Design Lead hiring
- Translates "JDs ready to post" into a shareable URL the founder can send to candidates and link from LinkedIn
- Closes the gap between having JDs in markdown and having a designed hiring entry point

### Added
- [`jobs.html`](jobs.html) — 5-section careers page: Why FORM (4 values: mission, craft, equity, timing) / Open roles (FE iOS + Design Lead with requirements, hard-noes, compensation pills) / What We Value (6 principles from HIRING.md) / Interview Process (6 steps, honest timeline) / Compensation tables (salary + equity + benefits). Links to full JDs in `hiring/`. CTA: hiring@form.coach. Added to sitemap.xml with priority 0.8.

### Changed
- [`VERSION`](VERSION) — 0.14.0 → 0.15.0
- [`sitemap.xml`](sitemap.xml) — jobs.html added (weekly, priority 0.8)
- [`README.md`](README.md) — jobs.html in repo tree; HTML count 11→12; version refs → v0.15.0
- [`STATUS.md`](STATUS.md) — jobs.html in pages table; version → v0.15.0; TL;DR → v0.15.0; "Apply JDs publicly" gap addressed

---

## [0.14.0] — 2026-05-14

### Why MINOR
- New HTML surface: `investors.html` — standalone investor one-pager page
- Closes the "one-pager here: [link]" gap referenced throughout INVESTOR_OUTREACH.md and FIRST_30_DAYS.md
- Gives investors a single shareable URL with the full pitch summary, unit economics, honest traction, ask breakdown, and links to all data-room docs

### Added
- [`investors.html`](investors.html) — 9-section investor overview page: The ask / The gap (problem + competition table) / Product (4 layers: CV + HRV + Victor + nutrition) / Market (SAM 12M + SOM $25M ARR + why-now) / Traction (honest: what exists vs. committed milestones) / Business model + unit economics ($9/$19, LTV/CAC 4–7×, gross margin 78–84%) / Team (George + AI-agent model) / Risks (honest table) / Next steps (invest@form.coach + links to PITCH/FINANCIALS/DATA_ROOM/TECHNICAL/GitHub). Marked `noindex` — investor-direct link only.

### Changed
- [`VERSION`](VERSION) — 0.13.0 → 0.14.0
- [`README.md`](README.md) — investors.html in repo tree; HTML count 10→11; releases 29+→30+; version refs → v0.14.0
- [`STATUS.md`](STATUS.md) — investors.html in pages table; version → v0.14.0; TL;DR → v0.14.0

---

## [0.13.0] — 2026-05-14

### Why MINOR
- Новий документ: `docs/ASSUMPTIONS_REGISTER.md` — реєстр 30 product/market/tech передумов до research sprint
- Перший документ що явно фіксує що ми вважаємо правдивим ДО 30 інтерв'ю, з confidence levels і pivot criteria
- Закриває gap між RESEARCH.md (як проводити) і DECISION_LOG.md (що вирішено): тепер є місце де відстежується що ще невідомо

### Added
- [`docs/ASSUMPTIONS_REGISTER.md`](docs/ASSUMPTIONS_REGISTER.md) — 30 numbered assumptions у 6 категоріях (User Pain Points, Product Beliefs, Pricing, Technical Feasibility, Market & Competition, Go-to-Market); confidence levels High/Medium/Low; status ● Untested → ✓/✗/~/? після research; test column → конкретні interview questions (Q12, Q15, Q17, Q19, Q23); wrong-if + pivot-if для кожної; review schedule (10/30 interviews, Beta W1/W4); cross-reference таблиця до RESEARCH, PERSONAS, OKRS, ANALYTICS_SETUP, FINANCIALS

### Changed
- [`docs/DECISION_LOG.md`](docs/DECISION_LOG.md) — додано DEC-025 (FIRST_30_DAYS execution framework), DEC-026 (PostHog EU Cloud analytics), DEC-027 (Assumptions Register як research gate); decision count 24 → 27
- [`VERSION`](VERSION) — 0.12.0 → 0.13.0
- [`README.md`](README.md) — ASSUMPTIONS_REGISTER.md у repo tree; doc count 41+→42+; decision count 24→27; version refs → v0.13.0
- [`STATUS.md`](STATUS.md) — ASSUMPTIONS_REGISTER.md у docs-таблиці; decision count → 27; TL;DR → v0.13.0

---

## [0.12.0] — 2026-05-14

### Why MINOR
- Новий документ: `docs/FIRST_30_DAYS.md` — 4-тижневий execution playbook
- Перший документ що перетворює 40+ стратегічних доків в операційний план дій
- Синтезує RESEARCH.md + INVESTOR_OUTREACH.md + DEPLOYMENT.md + CONTENT_CALENDAR.md + HIRING.md в єдину послідовність

### Added
- [`docs/FIRST_30_DAYS.md`](docs/FIRST_30_DAYS.md) — Week 0 (інфраструктура: domain, email, tools), W1 (Twitter, Show HN, перший newsletter), W2 (research sprint — recruiting DM templates, interview notes template), W3 (investor CRM setup, warm intro scripts, FE JD distribution channels + tech assessment), W4 (синтез, synthesis checklist, 8-gate checkpoint), troubleshooting таблиця

### Changed
- [`VERSION`](VERSION) — 0.11.0 → 0.12.0
- [`README.md`](README.md) — FIRST_30_DAYS.md у repo tree; doc count 40+→41+; version refs → v0.12.0
- [`STATUS.md`](STATUS.md) — FIRST_30_DAYS.md у docs-таблиці; версія → v0.12.0

---

## [0.11.0] — 2026-05-14

### Why MINOR
- Новий документ: `docs/BETA_RECRUITING.md` — повний playbook рекрутингу 200 бета-юзерів
- Закриває явний gap: `BETA_PLAYBOOK.md` покриває операції, але не пошук учасників
- 10 розділів: target profile, канали (3 tier), scoring, invite-sequence, incentives, timeline, anti-patterns

### Added
- [`docs/BETA_RECRUITING.md`](docs/BETA_RECRUITING.md) — Beta user recruiting: мета 200 qualified users, Tier 1-3 канали (особиста мережа / спільноти / контент), scoring rubric (15 балів), форма заявки (+3 нових питання), invite email sequence, incentive structure, timeline Wave 0→2, метрики тижня, anti-patterns, messaging quick-ref (UA+EN)

### Changed
- [`VERSION`](VERSION) — 0.10.0 → 0.11.0
- [`STATUS.md`](STATUS.md) — BETA_RECRUITING.md у docs-таблиці; doc count 39+→40+; версія → v0.11.0
- [`README.md`](README.md) — BETA_RECRUITING.md у repo tree; doc count оновлено; version refs → v0.11.0

---

## [0.10.0] — 2026-05-14

### Why MINOR
- Новий документ: `docs/ANALYTICS_SETUP.md` — повний instrumentation guide для Founding Engineer
- Companion до `PRODUCT_SPEC.md`: що будувати → як вимірювати
- 65 typed events, W-ACSU computation, PostHog iOS SDK setup, privacy implementation, A/B test guide

### Added
- [`docs/ANALYTICS_SETUP.md`](docs/ANALYTICS_SETUP.md) — PostHog setup (iOS SDK + Cloudflare EU), 65-event taxonomy з typed enums (no PII), W-ACSU SQL + PostHog formula, AARRR funnel setup, user super-properties, Core dashboard blueprint (11 panels), A/B test flag setup, privacy opt-out implementation, QA checklist per sprint, 3-environment separation (dev/beta/prod)

### Changed
- [`VERSION`](VERSION) — 0.9.0 → 0.10.0
- [`STATUS.md`](STATUS.md) — ANALYTICS_SETUP.md у docs-таблиці; doc count 38+→39+; версія → v0.10.0
- [`README.md`](README.md) — ANALYTICS_SETUP.md у repo tree; doc count оновлено; version refs → v0.10.0

---

## [0.9.0] — 2026-05-14

### Why MINOR
- Новий документ: `docs/PRODUCT_SPEC.md` — повна технічна специфікація iOS-продукту для Founding Engineer
- Перший документ що синтезує FLOWS + TECHNICAL + API в єдиний "що будуємо" артефакт
- Інключає 12-тижневий sprint plan, acceptance criteria, clinical-safety checklist, CV pipeline spec

### Added
- [`docs/PRODUCT_SPEC.md`](docs/PRODUCT_SPEC.md) — iOS product specification: MVP scope (P0/P1/P2), 7 screen specs з acceptance criteria, Victor AI implementation guide, CV pipeline для 8 ліфтів з accuracy targets, HealthKit integration, StoreKit 2 config, 12-тижневий sprint plan (Sprint 0–6 → TestFlight), NFR (performance/battery/privacy/a11y), pre-merge checklist, clinical-safety checklist per sprint

### Changed
- [`VERSION`](VERSION) — 0.8.0 → 0.9.0
- [`STATUS.md`](STATUS.md) — PRODUCT_SPEC.md додано до docs-таблиці; doc count 37+→38+; версія → v0.9.0
- [`README.md`](README.md) — PRODUCT_SPEC.md у repo tree; doc count оновлено; version refs → v0.9.0

---

## [0.8.0] — 2026-05-14

### Why MINOR
- Новий операційний документ: `docs/COMMUNITY.md` — beta community setup
- Закриває явний gap у STATUS.md ("Beta community (Discord/Telegram)" — not built)
- Підтримує BETA_PLAYBOOK.md (Slack-invite згадувався без деталей реалізації)
- Модераційні правила (секція 8) відповідають clinical-safety defaults

### Added
- [`docs/COMMUNITY.md`](docs/COMMUNITY.md) — Slack (closed beta ≤200) + Discord (post-launch); 11 каналів; ролі; welcome flow; тижневий ритм; feedback збір; moderation guidelines з ED/mental-health defaults; метрики здоров'я спільноти; pre-launch checklist; план переходу Slack→Discord з OG Beta privileges

### Changed
- [`STATUS.md`](STATUS.md) — beta community gap позначений addressed; COMMUNITY.md у docs-таблиці; timeline оновлено; версія → v0.8.0
- [`README.md`](README.md) — COMMUNITY.md у repo tree; doc count 36+→37+; releases 23+→24+; version refs → v0.8.0

---

## [0.7.0] — 2026-05-14

### Why MINOR
- New content asset: `content/linkedin-posts.md` — 12 LinkedIn posts across 4 categories
- Closes social media gap (Twitter + Reddit existed; LinkedIn was missing)
- LinkedIn is the primary channel for hiring, investor outreach, and user interviews

### Added
- [`content/linkedin-posts.md`](content/linkedin-posts.md) — 12 ready-to-post LinkedIn posts: 3 founder-story, 4 product/insight, 3 hiring, 2 build-in-public templates. EN/UA mix. 12-week posting calendar + hashtag bank.

### Changed
- [`STATUS.md`](STATUS.md) — post-09 added to content table (was missing); linkedin-posts.md added to content table; version refs bumped to v0.7.0; TL;DR updated (23+ releases)

---

## [0.6.0] — 2026-05-14

### Why MINOR
- New HTML surface: `press.html` — full press kit page
- New doc: `docs/NEWSLETTER.md` — Beehiiv setup guide, first 2 post concepts

### Added
- [`press.html`](press.html) — 9-section HTML press kit.
- [`docs/NEWSLETTER.md`](docs/NEWSLETTER.md) — newsletter setup guide.

### Changed
- [`README.md`](README.md) — press.html added to quick-nav; HTML count 9→10; doc count 35+→36+
- [`sitemap.xml`](sitemap.xml) — press.html added
- [`STATUS.md`](STATUS.md) — updated to v0.6.0

---

## [0.5.0] — 2026-05-14

### Why MINOR
- README перероблений як справжній project entry point
- Marketing site deployment-ready

### Changed
- [`README.md`](README.md) — full overhaul. Quick-navigation, full repo tree, team, operating model, metrics.

---

## [0.4.9] — 2026-05-14

### Added
- [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) — Cloudflare Pages deployment runbook
- [`sitemap.xml`](sitemap.xml) — SEO sitemap
- [`robots.txt`](robots.txt) — blocks AI training scrapers

---

## [0.4.8] — 2026-05-14

### Added
- [`workout-live.html`](workout-live.html) — high-fidelity workout mockup з CV
- `content/post-09-building-victor-voice.md`

---

## [0.4.7] — 2026-05-14

### Added
- [`docs/CONTENT_CALENDAR.md`](docs/CONTENT_CALENDAR.md)
- [`STATUS.md`](STATUS.md) — comprehensive project status snapshot
- `content/post-08-why-no-free-tier.md`
- [`docs/B2B_PLAYBOOK.md`](docs/B2B_PLAYBOOK.md)
- [`docs/ENGINEERING_RUNBOOK.md`](docs/ENGINEERING_RUNBOOK.md)
- `programs/sample-cycle-aware-week.md`
- [`docs/INVESTOR_UPDATE_TEMPLATE.md`](docs/INVESTOR_UPDATE_TEMPLATE.md)
- [`faq.html`](faq.html) — 30+ questions у 7 категоріях
- `content/post-07-progressive-overload-myths.md`
- [`docs/DECISION_LOG.md`](docs/DECISION_LOG.md) — 24 documented decisions
- `programs/sample-deload-week.md`
- `nutrition/sample-cutting-plan.md`
- [`docs/SUPPORT.md`](docs/SUPPORT.md)
- [`docs/I18N.md`](docs/I18N.md)
- `content/press-pitch-templates.md`
- `content/reddit-launch-posts.md`
- `hiring/jd-founding-engineer-ios.md`
- `hiring/jd-design-lead.md`
- `hiring/jd-sport-science-advisor.md`
- [`docs/LAUNCH_CHECKLIST.md`](docs/LAUNCH_CHECKLIST.md)
- `content/post-06-30-user-interviews.md`
- `content/twitter-launch-thread.md`

### Changed
- [`index.html`](index.html) — nav оновлений

---

## [0.3.9] — 2026-05-14

### Added
- [`settings.html`](settings.html) — Settings screen mockup
- [`docs/OKRS_2026.md`](docs/OKRS_2026.md)
- [`docs/DATA_ROOM.md`](docs/DATA_ROOM.md)
- [`about.html`](about.html)
- [`docs/SECURITY.md`](docs/SECURITY.md)
- `content/post-05-building-fitness-from-ukraine.md`
- [`pricing.html`](pricing.html)
- [`docs/EMAIL_SEQUENCE.md`](docs/EMAIL_SEQUENCE.md)
- [`docs/BETA_PLAYBOOK.md`](docs/BETA_PLAYBOOK.md)
- [`screens.html`](screens.html)

---

## [0.3.5] — 2026-05-14

### Added
- [`nutrition/sample-3day-plan.md`](nutrition/sample-3day-plan.md)
- [`docs/API.md`](docs/API.md)
- `content/post-03-we-tested-8-ai-trainers.md`

---

## [0.3.0] — 2026-05-14

### Added
- [`docs/PITCH.md`](docs/PITCH.md)
- [`docs/PERSONAS.md`](docs/PERSONAS.md)
- [`docs/TECHNICAL.md`](docs/TECHNICAL.md)

---

## [0.2.1] — 2026-05-14

### Added
- `VERSION` — single source of truth
- `CHANGELOG.md`
- `CONTRIBUTING.md`

---

## [0.2.0] — 2026-05-14

### Added
- [`index.html`](index.html) — marketing site
- 14 субагентів
- [`docs/BRAND.md`](docs/BRAND.md)
- [`docs/FLOWS.md`](docs/FLOWS.md)
- [`docs/MARKETING.md`](docs/MARKETING.md)

### Changed
- Victor: «Не пропускай. Ніколи.» → «Не пропускай два поспіль.»
- Гасло: «Тренер. У кишені.»

---

## [0.1.0] — 2026-05-13

### Added
- Перша версія marketing-сайту
- 7 початкових субагентів
- Базова естетика: warm-black + cream + acid lime + Fraunces/Manrope/JBMono
