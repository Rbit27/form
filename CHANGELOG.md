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

## [0.35.0] — 2026-05-16

### Why MINOR
- New enterprise compliance document: `docs/SOC2_READINESS.md` — full SOC 2 Type II readiness assessment across all 5 Trust Service Criteria

### Added
- [`docs/SOC2_READINESS.md`](docs/SOC2_READINESS.md) — SOC 2 Type II readiness: 5 TSC coverage (Security/Availability/Processing Integrity/Confidentiality/Privacy), 55-control gap analysis (~40% in place, 11 critical gaps, remediation roadmap), evidence collection process, audit firm engagement timeline, continuous compliance tooling. References DEC-030, AUDIT_LOG_SCHEMA.md, ENTERPRISE.md, SECURITY.md.

---

## [0.34.0] — 2026-05-16

### Why MINOR
- New blog post: `content/post-11-deload-weeks.md` — 13-min read on deload week science, supercompensation, and periodization
- Blog card added to `blog.html`

### Added
- [`content/post-11-deload-weeks.md`](content/post-11-deload-weeks.md) — deload weeks: накопичена втома, суперкомпенсація, ознаки потреби, плановий vs реактивний деload, що конкретно робити. clinical-safety PASS. sports-scientist review pending.
- `blog.html` — картка post-11 в grid (Спортивна наука, 13 хв)

---

## [0.33.0] — 2026-05-16

### Why MINOR
- New: enterprise admin dashboard mockup (`admin-dashboard.html`) — full tenant control panel for HR/People leads with privacy floor
- New: `docs/AUDIT_LOG_SCHEMA.md` — append-only HMAC-chained audit log architecture (SOC 2 / GDPR foundation)
- New: DEC-030 in DECISION_LOG codifies the audit log architecture
- iPhone app shipped multiple EAS updates: v0.20.0 → v0.21.0 (PR detection, streak grace per DEC-013, active meal slot highlight, rest timer ±30s controls)

### Added
- [`admin-dashboard.html`](admin-dashboard.html) — enterprise admin panel mockup: tenant switcher, 4 stat cards (Активовано 287/412, Тижнева 68%, W-ACSU 3.4, NPS 52), 30-day activity bar chart, department adoption breakdown, anonymous users table, privacy floor card emphasizing HR cannot see individual data
- [`docs/AUDIT_LOG_SCHEMA.md`](docs/AUDIT_LOG_SCHEMA.md) — full schema (HMAC chain, partition strategy), action taxonomy (auth/tenant/data/privacy/integration/system/support), retention policies (7y / 3y / 30d), access controls + break-glass procedure, export options (webhook, S3 sync, REST, SIEM)
- [`docs/DECISION_LOG.md`](docs/DECISION_LOG.md) — DEC-030 audit log architecture

### iPhone app (EAS Update preview branch)
- v0.20.0 — PR detection in Workout (heavier set → 🏆 alert + success haptic)
- v0.20.1 — PR within-session fix (Math.max(historical, currentSession), only alert if historical > 0); active meal slot highlight ("· зараз") based on current hour
- v0.20.2 — Streak grace per DEC-013: tolerates 1 missed day, breaks on 2 consecutive misses (Progress.tsx)
- v0.20.3 — Today.tsx streak fix to match Progress.tsx (consistency across screens)
- v0.21.0 — Rest timer ±30s adjustment buttons

### QA walks (qa-walker)
- P2 found and fixed: within-session double-PR re-trigger
- P1 found and fixed: streak contradicted DEC-013 (broke after 1 miss, not 2)

---

## [0.31.0] — 2026-05-15

### Why MINOR
- New blog post: `content/post-10-why-programs-assume-youre-always-ready.md`
- Fills a genuine content gap: posts 01/04/07 cover HRV, sleep mechanics, and progressive overload individually; no post addresses the core conceptual problem — why standardized programs can't account for variable recovery capacity — and how adaptive training solves it
- This is the differentiating argument behind FORM: Victor adapts because it reads the day's context, not only the week's schedule
- `blog.html` updated: 10th card added, hero counters updated
- Pending sports-scientist + clinical-safety review before publish (topic: training load management — no body image, no dietary claims)

### Added
- [`content/post-10-why-programs-assume-youre-always-ready.md`](content/post-10-why-programs-assume-youre-always-ready.md) — 13-min read, training-science; covers: the hidden assumption in every pre-written program, training load concept (acute:chronic workload ratio), autoregulation and RPE, what "adaptive" actually means vs. marketing buzzword (3 levels), practical self-coaching tips, FORM's approach via Victor, clinical-safety frontmatter note

### Changed
- [`blog.html`](blog.html) — post-10 card added (Спортивна наука, "Незабаром"); hero: 9→10 статей, 108→121 хв читання; card placed between post-04 and post-09
- [`STATUS.md`](STATUS.md) — post-10 added to content engine table; version → v0.31.0
- [`README.md`](README.md) — post-10 added to content tree; blog posts metric 9→10; version → v0.31.0
- [`VERSION`](VERSION) — 0.30.0 → 0.31.0

---

## [0.30.0] — 2026-05-15

### Why MINOR
- New HTML surface: `blog.html` — articles index for 9 existing blog posts
- Genuine gap: 9 blog posts existed as markdown drafts with no discoverable HTML entry point; journalists, investors, and beta candidates directed to the marketing site couldn't find the content engine
- Adds category filter (all / training science / industry analysis / founder notes / product), newsletter CTA with email capture, and honest status badges ("Незабаром" vs "Після Q3 interviews" for post-06 which requires real interview data)
- clinical-safety considerations: page shows titles + short neutral excerpts only; no body-image claims; post-04 framed around sleep/recovery mechanics without prescriptive language; no "never skip" phrasing — uses brand principle language

### Added
- [`blog.html`](blog.html) — articles index: 9 cards with category badges (Спортивна наука / Аналіз ринку / Нотатки засновника / Продукт / Дослідження), JS category filter, newsletter CTA with email capture, post-06 marked "Після Q3 interviews" (honest — requires real user interview data to complete), consistent editorial-brutalist design with grain overlay + acid lime + Fraunces/Manrope/JBMono

### Fixed
- [`STATUS.md`](STATUS.md) — "28 total documented decisions" → "29" (DEC-029 was added in v0.29.0 but counter not updated)
- [`README.md`](README.md) — VERSION comment in repo tree was hardcoded "0.22.0" → "(current)"; HTML page count 12 → 13; surface description updated

### Changed
- [`sitemap.xml`](sitemap.xml) — `blog.html` added (priority 0.8, changefreq weekly)
- [`VERSION`](VERSION) — 0.29.0 → 0.30.0
- [`STATUS.md`](STATUS.md) — blog.html added to HTML pages table; version → v0.30.0
- [`README.md`](README.md) — blog.html added to repo tree; metrics updated

---

## [0.29.0] — 2026-05-15

### Why MINOR
- New document: `docs/GROWTH_LOOPS.md` — viral/organic growth mechanics
- Fills a genuine gap: BETA_RECRUITING.md covers how to get 200 beta users; RETENTION_PLAYBOOK.md covers D0-D90 retention; but no document defines the specific viral mechanics (share cards, referral system, streak milestones) the Founding Engineer needs to build
- 5 loops designed with DEC-002 (no in-app social) and DEC-016 (no free tier) constraints baked in
- DEC-029 logged: growth loops policy — external sharing only, clinical-safety gate on all share content

### Added
- [`docs/GROWTH_LOOPS.md`](docs/GROWTH_LOOPS.md) — 5 growth loops: Loop 1 Form Check Share (CV result → share card → organic acquisition), Loop 2 Streak Share (milestone → share), Loop 3 Victor WOM (coaching quality drives word of mouth), Loop 4 Referral Program (formal, post-launch M4-M5), Loop 5 Beta FOMO (waitlist referral mechanics); clinical-safety allowed/blocked content tables; FE implementation priority P0-P4; PostHog events for tracking; K-factor targets; 5 open questions for future decisions
- `docs/DECISION_LOG.md` — DEC-029: growth loops sharing policy (external only, clinical-safety gate, allowed vs. blocked share content)

### Changed
- [`VERSION`](VERSION) — 0.28.0 → 0.29.0
- [`STATUS.md`](STATUS.md) — GROWTH_LOOPS.md added to docs table; doc count 55+→56+; decisions 28→29; version → v0.29.0
- [`README.md`](README.md) — GROWTH_LOOPS.md added to docs tree; doc count 55+→56+; decisions 28→29; version → v0.29.0

---

## [0.28.0] — 2026-05-15

### Why MINOR
- New document: `docs/VICTOR_PROMPT_GUIDE.md` — LLM implementation guide for Victor
- Fills a genuine gap: BRAND.md has tone rules, post-09 explains the challenge, PRODUCT_SPEC.md lists Victor chat as P1 — but no document tells the FE HOW to actually build Victor's LLM layer
- Critical for FE hire: demonstrates technical depth to candidates and lets them start building Victor correctly from day 1
- 9 sections, 22 scenario examples with concrete UA/EN responses, full clinical-safety checklist embedded
- clinical-safety: reviewed — hard blocks defined in Layer 2, ED-flag mode documented in §7, distress response placeholder explicitly marked TBD (not fabricated), jurisdiction-appropriate mental health resources flagged as pending before ship

### Added
- [`docs/VICTOR_PROMPT_GUIDE.md`](docs/VICTOR_PROMPT_GUIDE.md) — LLM implementation guide: 5-layer system prompt architecture (role + safety constraints + context injection + format + user message), TypeScript context schema with all runtime fields (HRV, sleep, streak, ED-flag, form score, etc.), 4 tone modes with UA/EN phrase examples (smart_direct/mindful/cold/coach), response format rules (length per context, forbidden constructions), 22 scenario library with exact Victor phrasing (Day 1 / PR / mediocre session / 1-missed / 3-missed / week-missed / injury / plateau / food no-flag / food ED-flag / low HRV / high HRV / unmotivated / 30-streak / comparison / faster results / deload start / post-deload / supplements / 2-weeks-away / pre-event / out-of-scope), ED-flag restricted mode (what changes, invisible flag rationale, distress response), Victor silence rules (when NOT to speak, 1/day notification cap), 7 open questions for FE, clinical-safety embedded checklist

### Changed
- [`VERSION`](VERSION) — 0.27.0 → 0.28.0
- [`STATUS.md`](STATUS.md) — VICTOR_PROMPT_GUIDE.md added to docs table; doc count 54+→55+; version → v0.28.0
- [`README.md`](README.md) — VICTOR_PROMPT_GUIDE.md added to docs tree; doc count 54+→55+; version → v0.28.0

---

## [0.27.0] — 2026-05-15

### Why MINOR
- New document: `docs/RETENTION_PLAYBOOK.md` — Day 0/7/30/90 retention mechanics for FORM beta and beyond
- Fills genuine gap: ANALYTICS_SETUP.md tracks what to measure, BETA_PLAYBOOK.md handles TestFlight ops, COMMUNITY.md handles Slack/Discord, but nothing defined what happens inside the product when users show churn signals
- Key decisions: weekly streaks (not daily — respects programmed rest days), 3-attempt re-engagement cap then silence, Pause always before Cancel (3-click max), Victor memory arc (Week 1 → Month 3), clinical-safety checklist for all retention copy
- 11 sections, 4 open questions for FE + design hire, explicit clinical-safety constraints embedded throughout
- clinical-safety: reviewed — retention copy constraints documented as hard blocks/approved/gray-zone checklist

### Added
- [`docs/RETENTION_PLAYBOOK.md`](docs/RETENTION_PLAYBOOK.md) — 560-line retention strategy: critical windows (D0/D1-3/D7/D14/D30/D90) with Victor example phrases, notification types + cadence (1/day cap, user-controlled windows), weekly streak mechanics (session-based, grace week, "не пропускай два поспіль" embedded), Victor personalization arc, 3-stage re-engagement protocol (passive/at-risk/churned, max 3 attempts), Pause>Cancel design (3-click, 2Q exit survey), clinical-safety constraints as hard-blocks/approved/gray-zone checklist, metrics framework (no fabricated targets), integration map, 4 open questions

### Changed
- [`VERSION`](VERSION) — 0.26.0 → 0.27.0
- [`STATUS.md`](STATUS.md) — RETENTION_PLAYBOOK.md added to docs table; doc count 53+→54+; version → v0.27.0
- [`README.md`](README.md) — RETENTION_PLAYBOOK.md added to docs tree; doc count 53+→54+; version → v0.27.0

---

## [0.26.0] — 2026-05-15

### Why MINOR
- New content: `content/newsletter-03.md` — Issue 03 "Building Victor: What it takes to design an AI coaching voice"
- Extends newsletter content engine to 3 issues; natural third chapter after "Why we're building FORM" and "What FORM does/doesn't do"
- Victor voice demonstrated via side-by-side example (competitor vs Victor on skipped workout) — clinical-safety PASS
- brand-voice + clinical-safety reviewed: no food/body-image/injury/mental health content

### Added
- [`content/newsletter-03.md`](content/newsletter-03.md) — Issue 03 "Building Victor" (~720 words): problem statement (app praising on rest days), two bad coaching defaults (drill sergeant vs cheerleader), "не пропускай два поспіль" philosophy explained, side-by-side skipped-workout example (competitor vs Victor), clinical-safety as design constraint framed honestly, beta CTA · clinical-safety PASS

### Changed
- [`VERSION`](VERSION) — 0.25.0 → 0.26.0
- [`STATUS.md`](STATUS.md) — newsletter-03.md added to content table; version → v0.26.0
- [`README.md`](README.md) — newsletter-03.md added to content tree; version → v0.26.0

---

## [0.25.0] — 2026-05-15

### Why MINOR
- New document: `docs/LINKEDIN_PLAYBOOK.md` — LinkedIn strategy for FORM's pre-launch phase
- Fills genuine gap: `content/linkedin-posts.md` has 12 posts ready but no strategic framework for deploying them; `docs/TWITTER_PLAYBOOK.md` exists for X but LinkedIn is a distinct platform (different audience, algorithm, format, value proposition for FORM)
- LinkedIn is FORM's primary hiring and investor-visibility channel — engineers/investors are on LinkedIn, not necessarily on Fitness Twitter
- 15 sections: LinkedIn vs X comparison table, profile setup (headline/about/featured/Creator Mode), company page recommendation (defer pre-beta), 4 content pillars, 3-tier connection strategy, LinkedIn for hiring (InMail templates, FE+DL cadence), LinkedIn for investor relationships (what VCs check, warm intros), build-in-public translation guide (thread vs single post, repurposing blogs), 2-3x/week cadence with day/time table, voice rules comparison table, metrics (SSI, engagement rate, hiring funnel UTMs), honest limitations (not a consumer growth channel), 10-item quick-start checklist, integration map to 9 docs, 4 open questions
- clinical-safety: not required (no food, body image, mental health, or injury content)

### Added
- [`docs/LINKEDIN_PLAYBOOK.md`](docs/LINKEDIN_PLAYBOOK.md) — 15-section LinkedIn strategy: X vs LinkedIn comparison table, profile optimization (headline variants, About section draft, Creator Mode setup), company-page recommendation (defer until post-beta), 4 content pillars with post assignments from linkedin-posts.md, 3-tier connection strategy (Tier 1: 50 connections week 1, Tier 2: pre-outreach warm-up, Tier 3: follow only), hiring InMail templates + cadence for FE + Design Lead, investor visibility playbook (what VCs check before first meeting), build-in-public translation guide (what works/doesn't from X, repurposing blog content), 2-3x/week cadence with day/time table, voice rules table (LinkedIn vs X vs newsletter), metrics framework (SSI, engagement rate, profile views, hiring funnel UTMs), honest limitations (LinkedIn won't drive App Store installs), 10-item quick-start checklist (doable today without domain or live product), integration map to 9 companion docs, 4 open questions (network size, language split UA/EN, LinkedIn newsletter vs Beehiiv, existing network segmentation)

### Changed
- [`VERSION`](VERSION) — 0.24.0 → 0.25.0
- [`STATUS.md`](STATUS.md) — LINKEDIN_PLAYBOOK.md added to docs table; TL;DR → v0.25.0; doc count 51+→52+; releases 40+→41+; version header + footer → v0.25.0
- [`README.md`](README.md) — LINKEDIN_PLAYBOOK.md added to docs tree; doc count 51+→52+; version refs → v0.25.0

---

## [0.24.0] — 2026-05-15

### Why MINOR
- New document: `docs/INVESTOR_PIPELINE.md` — investor CRM and relationship management framework
- Fills genuine gap: INVESTOR_OUTREACH.md (target lists + cold templates), SEED_NARRATIVE.md (verbal pitch), DD_FAQ.md (due diligence Q&A), and DATA_ROOM.md (materials) all exist — but no doc defines HOW to manage the pipeline itself: stage definitions, CRM schema, follow-up cadence, signals of interest, post-meeting ritual, close mechanics
- 15 sections: 7-stage pipeline model (S0–S6), Google Sheets CRM schema (15 columns + 3 tabs), cadence rules per stage transition, warm intro templates (request + forwardable blurb), 30-min pre-meeting prep ritual, post-meeting recap template, green/yellow/red signal taxonomy, anti-pattern table, realistic raise timeline, integration map to 7 other docs
- clinical-safety not required (no food/body/injury/mental health content)

### Added
- [`docs/INVESTOR_PIPELINE.md`](docs/INVESTOR_PIPELINE.md) — investor relationship management: 7-stage pipeline (S0 Listed → S6 Closed/Declined), CRM spreadsheet schema (15 columns, color logic, 3 tabs), stage-by-stage cadence rules, warm intro request + forwardable blurb templates, 30-min pre-meeting prep ritual, post-meeting recap email template + log checklist, green/yellow/red interest signals, park vs. hard-decline logic, monthly investor updates strategy (links INVESTOR_UPDATE_TEMPLATE.md), SAFE mechanics + round close checklist, 12-item anti-pattern table, realistic M0→M5 timeline, 14-doc integration map, 4 open questions for counsel + product-strategist

### Changed
- [`VERSION`](VERSION) — 0.23.0 → 0.24.0
- [`STATUS.md`](STATUS.md) — INVESTOR_PIPELINE.md added to docs table; TL;DR → v0.24.0; doc count 50+→51+; releases 39+→40+; version header + footer → v0.24.0
- [`README.md`](README.md) — INVESTOR_PIPELINE.md added to docs tree; doc count 50+→51+; version refs → v0.24.0

---

## [0.23.0] — 2026-05-15

### Why MINOR
- New document: `docs/TWITTER_PLAYBOOK.md` — pre-launch X/Twitter growth strategy
- Fills genuine gap: `content/twitter-launch-thread.md` covers launch day only; nothing covered the 3-4 month pre-launch build-in-public window
- George can start today: account setup, handle rationale, 3 bio variants, pinned tweet concept, 5 content pillars with real example drafts, weekly cadence, engagement strategy, beta recruiting via X (connects to RESEARCH_SCREENER.md), voice rules for X, metrics framework, honest limitations, quick-start checklist
- clinical-safety not required (no food/body/injury/mental health content)

### Added
- [`docs/TWITTER_PLAYBOOK.md`](docs/TWITTER_PLAYBOOK.md) — 555-line pre-launch X strategy: account setup (handle, 3 bio variants, pinned tweet), Phase 1/2/3 content phases, 5 content pillars with example tweets, weekly cadence (5-7 posts/week), build-in-public formula (what to share vs not), engagement map (5 account categories), beta recruiting via X → RESEARCH_SCREENER.md, voice rules for X vs LinkedIn vs newsletter, launch-week coordination → twitter-launch-thread.md, metrics (engagement rate, UTM attribution, beta recruits), honest limitations (X won't scale installs, build-in-public shelf life), quick-start checklist (8 tasks for today)

### Changed
- [`VERSION`](VERSION) — 0.22.0 → 0.23.0
- [`STATUS.md`](STATUS.md) — TWITTER_PLAYBOOK.md added to docs table; TL;DR → v0.23.0; doc count 49+→50+; releases 38+→39+; version header + footer → v0.23.0
- [`README.md`](README.md) — TWITTER_PLAYBOOK.md added to docs tree; doc count 49+→50+; version refs → v0.23.0

---

## [0.22.0] — 2026-05-15

### Why MINOR
- New document: `docs/BETA_FEEDBACK_PROTOCOL.md` — operational protocol for processing beta user feedback
- Closes genuine gap: BETA_PLAYBOOK.md handles TestFlight operations, BETA_RECRUITING.md handles acquisition, COMMUNITY.md handles Slack/Discord — but no doc defines what happens AFTER feedback arrives
- 12 sections: channels, taxonomy (7 categories + P0–P3 severity), daily intake process, safety escalation, weekly synthesis, monthly report template, feedback→decision filter, Victor voice tracker, privacy rules, integration map, open questions
- clinical-safety not required (no food/body/injury/mental health content — safety escalation paths reference clinical-safety but the doc itself is operational procedure)

### Added
- [`docs/BETA_FEEDBACK_PROTOCOL.md`](docs/BETA_FEEDBACK_PROTOCOL.md) — intake, taxonomy (7 categories, P0–P3 severity), daily triage process, safety escalation path, weekly synthesis rhythm, monthly report template, feedback→decision filter with 5-step logic gate, Victor voice tracker, privacy rules, integration with COMMUNITY/RESEARCH/ANALYTICS_SETUP/DECISION_LOG/PRODUCT_SPEC/SUPPORT, 3 open questions for Founding Engineer + LEGAL.md followup

### Changed
- [`VERSION`](VERSION) — 0.21.0 → 0.22.0
- [`STATUS.md`](STATUS.md) — BETA_FEEDBACK_PROTOCOL.md added to docs table; TL;DR → v0.22.0; doc count 48+→49+; releases 37+→38+; version header + footer → v0.22.0
- [`README.md`](README.md) — BETA_FEEDBACK_PROTOCOL.md added to docs tree; doc count 48+→49+; version refs → v0.22.0

---

## [0.21.0] — 2026-05-15

### Why MINOR
- New document: `docs/RESEARCH_SCREENER.md` — practical recruiting toolkit for 30 pre-product user interviews
- Fills genuine operational gap: RESEARCH.md has the 45-min interview script and synthesis framework, but no screener survey, no recruiting messages, no tracker template, no payment flow
- George can use this TODAY to start recruiting — does not require site to be live
- 12 sections: screener questions with pass/fail criteria, channel-specific messages (Reddit, X, Telegram, LinkedIn), email templates, Google Sheets tracker columns, gift card payment flow (Amazon/Apple/Monobank/PayPal), anti-bias checklist, weekly velocity targets
- clinical-safety not required (no food/body/injury/mental health content; ethics reference deferred to RESEARCH.md §9 which already has the safeguards)

### Added
- [`docs/RESEARCH_SCREENER.md`](docs/RESEARCH_SCREENER.md) — user research recruiting toolkit: 7-question screener (Q1–Q7 with pass/reject/maybe criteria), recruiting messages EN+UA for Reddit/X/Telegram/LinkedIn, Reddit post templates for r/fitness + r/xxfitness, X DM flow with search operators, interview tracker (17-column Google Sheets spec), scheduling email sequence (confirmation + 24h + 1h reminders), gift card payment flow (Amazon/Apple/Monobank/PayPal), rejection template, anti-bias pre-call checklist, weekly velocity targets (8→20→30 calls), 3 open questions for LEGAL.md followup

### Changed
- [`VERSION`](VERSION) — 0.20.0 → 0.21.0
- [`STATUS.md`](STATUS.md) — RESEARCH_SCREENER.md added to docs table; TL;DR → v0.21.0; doc count 47+→48+; releases 36+→37+; version header + footer → v0.21.0; timeline extended to v0.21.0
- [`README.md`](README.md) — RESEARCH_SCREENER.md added to docs tree; doc count 47+→48+; version refs → v0.21.0; date → 15 травня 2026

---

## [0.20.0] — 2026-05-15

### Why MINOR
- New document: `docs/DD_FAQ.md` — investor due diligence FAQ prep
- Completes the investor preparation toolkit: PITCH.md (deck) + SEED_NARRATIVE.md (call script) + INVESTOR_OUTREACH.md (outreach) + FINANCIALS.md (model) + DATA_ROOM.md (materials) were all present, but no DD Q&A prep
- Covers 7 blocks: Team, Market, Product/Tech, Financials, Execution risks, Round terms, Data Room
- All answers follow Principle #1: no fabricated numbers; honest about unknowns; confident on known facts
- Fix: STATUS.md table had stale decision count (27 → 28) — DEC-028 was logged in v0.18.0 but table row wasn't updated
- Fix: STATUS.md and README.md dates updated to 15 травня 2026

### Added
- [`docs/DD_FAQ.md`](docs/DD_FAQ.md) — 7-block investor DD FAQ: Team (co-founder risk, Ukraine risk), Market (TAM, competition, Apple/Google risk), Product/Tech (CV accuracy, privacy, LLM costs, iOS-only rationale), Financials (subscription model, pricing, break-even, fund use), Execution risks (3 biggest risks, kill criteria, Ukraine), Round (amount, terms, what beyond money), Data Room index. Companion to SEED_NARRATIVE.md, PITCH.md, FINANCIALS.md, DATA_ROOM.md.

### Changed
- [`VERSION`](VERSION) — 0.19.0 → 0.20.0
- [`STATUS.md`](STATUS.md) — DD_FAQ.md added to docs table; DECISION_LOG.md row fixed 27→28 decisions; date → 15 травня 2026; TL;DR → v0.20.0; doc count 46+→47+; releases 35+→36+; version header + footer → v0.20.0; timeline extended to v0.20.0
- [`README.md`](README.md) — DD_FAQ.md added to docs tree; doc count 46+→47+; version refs → v0.20.0; date → 15 травня 2026

---

## [0.19.0] — 2026-05-14

### Why MINOR
- New document: `docs/SEED_NARRATIVE.md` — verbal pre-seed pitch script
- Complements PITCH.md (slide structure) and INVESTOR_OUTREACH.md (outreach templates)
- Fills genuine gap: George needs a spoken narrative for angel calls, not just deck slides
- Covers the specific challenge of raising pre-product pre-revenue: 6 narrative blocks, objection playbook, audience variants (angel/seed fund/accelerator/strategic)
- Fix: STATUS.md and README.md had stale decision count (27 → 28, DEC-028 was added in v0.18.0)
- Fix: STATUS.md timeline was missing v0.16.0–v0.19.0 milestones (stopped at v0.15.0)

### Added
- [`docs/SEED_NARRATIVE.md`](docs/SEED_NARRATIVE.md) — 6-block verbal pitch script for angel/pre-seed investor calls: Block 0 (opening line), Block 1 (personal story), Block 2 (product — 3 things), Block 3 (timing / tech window), Block 4 (market + model), Block 5 (honest state + what's next), Block 6 (ask). Objection playbook: 6 common objections with verbatim responses. Tonal rules table (do/don't). Audience variants table (angel-tech / angel-fitness / seed fund / strategic / accelerator). Companion references to PITCH.md, INVESTOR_OUTREACH.md, FINANCIALS.md.

### Changed
- [`VERSION`](VERSION) — 0.18.0 → 0.19.0
- [`STATUS.md`](STATUS.md) — SEED_NARRATIVE.md added to docs table; TL;DR → v0.19.0; doc count 45+→46+; releases 34+→35+; version header + footer → v0.19.0; decision count 27→28; timeline extended to v0.19.0
- [`README.md`](README.md) — SEED_NARRATIVE.md added to docs tree; doc count 45+→46+; version refs → v0.19.0; decision count 27→28

---

## [0.18.0] — 2026-05-14

### Why MINOR
- New document: `docs/HIRING_GUIDE.md` — platform-by-platform JD posting guide
- Closes the "Apply Founding Engineer + Design Lead JDs on platforms" gap from STATUS.md "What's next (immediate)"
- Prior art: JD markdown files in `hiring/` + designed careers page (`jobs.html`) — but no concrete posting instructions for George
- New: DEC-028 (platform priority for hiring) in DECISION_LOG.md
- Fix: STATUS.md had wrong version reference (v0.15.0) — corrected to v0.18.0

### Added
- [`docs/HIRING_GUIDE.md`](docs/HIRING_GUIDE.md) — 8-section platform-by-platform hiring guide: role/platform priority table (FE: Djinni > DOU > Wellfound > LinkedIn; Design Lead: LinkedIn > Wellfound > Djinni > Behance); per-platform step-by-step setup (Djinni account, DOU jobs + forum, Wellfound company profile, LinkedIn personal post vs. paid job post); CV screening red/green flags by role; 48-hour response protocol; rejection email template; interview logistics (45-min structure); 5-week timeline; $0-150 budget breakdown; pre-posting checklist; tracking sheet columns

### Changed
- [`VERSION`](VERSION) — 0.17.0 → 0.18.0
- [`STATUS.md`](STATUS.md) — HIRING_GUIDE.md у docs table; TL;DR → v0.18.0; doc count 44+→45+; releases 33+→34+; version header + footer → v0.18.0
- [`README.md`](README.md) — HIRING_GUIDE.md у docs tree; doc count 44+→45+; version refs → v0.18.0
- [`docs/DECISION_LOG.md`](docs/DECISION_LOG.md) — DEC-028 added (platform priority: Djinni-first для FE, LinkedIn-first для Design Lead; $0 start, boost as fallback)

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
