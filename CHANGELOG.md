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

## [0.4.6] — 2026-05-14

### Added
- `content/post-08-why-no-free-tier.md` — 8-й blog post: «Чому FORM ніколи не матиме free tier». MyFitnessPal vs Whoop case studies, чому це для users добре. ~9 хв read.
- [`docs/B2B_PLAYBOOK.md`](docs/B2B_PLAYBOOK.md) — corporate wellness playbook: target customers, pricing tiers, sales process, what we'll/won't promise companies vs employees, anti-patterns. **NOT activate** до B2C PMF proven.

### Changed
- [`index.html`](index.html) — nav оновлений: посилання на pricing.html, about.html, faq.html. CTA → beta.html. Інтегрує marketing site.

### Added
- [`docs/ENGINEERING_RUNBOOK.md`](docs/ENGINEERING_RUNBOOK.md) — on-call rotation, deployment processes (iOS / backend), monitoring dashboards, incident SEV-0 runbook, code review requirements, secrets management
- `programs/sample-cycle-aware-week.md` — cycle-aware training template (5 фаз), nutrition adjustments per phase, edge cases (birth control, menopause, irregular cycles). Position: helper, not optimizer (evidence base modest).
- [`docs/INVESTOR_UPDATE_TEMPLATE.md`](docs/INVESTOR_UPDATE_TEMPLATE.md) — monthly investor update template з cadence rules, anti-patterns, bad-month protocol, distribution list guidance

### Added
- [`faq.html`](faq.html) — FAQ page з 30+ accordion-style questions у 7 категоріях (Product, Pricing, Camera+Tech, Recovery+Data, Safety, Privacy, Team+Company)
- `content/post-07-progressive-overload-myths.md` — 7-й blog: «Progressive overload — що половина лiфтерів робить неправильно». 8 vectors прогресу, RPE-autoregulation, block periodization, 3 типові помилки intermediate-лiфтерів. ~12 хв read.
- [`docs/DECISION_LOG.md`](docs/DECISION_LOG.md) — live decision log з 24 documented decisions (DEC-001 → DEC-024). Append-only format, owner-mandatory, honest reverse-cost assessment.

### Added
- `programs/sample-deload-week.md` — deload week template (3 sessions, 50% volume, RPE 5, focus на technique polish). Signals what's working vs needs extended deload. Stop-criteria during deload.
- `nutrition/sample-cutting-plan.md` — 12-week cutting plan з 3 phases. Safety floor $1,800kcal min. Strength preservation gate. Soft-mode альтернативa для ED-history users. Stop-criteria.
- [`docs/SUPPORT.md`](docs/SUPPORT.md) — customer support playbook: 10 common ticket templates, edge cases (crisis, pricing complaints, App Store reviews), SLA tiers (P0-P3), what we never do, metrics tracking.

### Added
- [`docs/I18N.md`](docs/I18N.md) — internationalization plan: UA+EN ship v1.0, DE/PL/ES phase 1 (Q2 2027), cost breakdown ($8.5k per language), cultural calibration notes, decision framework, RTL/CJK explicitly off-roadmap
- `content/press-pitch-templates.md` — 5 ready-to-customize pitch templates: TechCrunch/Verge, Hacker News (Show HN), Lex/Huberman podcasts, YouTube creators (Jeff Nippard tier), Lenny/Stratechery newsletters. Anti-patterns + tracking method.
- `content/reddit-launch-posts.md` — 3 готові пости: r/fitness (honest review of 8 AI apps), r/weightroom (technical pose-estimation breakdown), r/xxfitness (women-focused frustrations + questions). Engagement plans, mod-relations, anti-patterns.

### Changed
- README: updated structure (поки не критично, наступний commit)

### Added
- `hiring/jd-founding-engineer-ios.md` — Standalone JD: $90-130k base + 1.5-3% equity. Process, tech stack, what success looks like 1/3/6/12 months, honest-truth section.
- `hiring/jd-design-lead.md` — Standalone JD: $80-115k + 1-2% equity. Aesthetic principles (yes/no list), take-home options, anti-profile.
- `hiring/jd-sport-science-advisor.md` — Part-time advisor: $200-350/hr або $2,500-4k/mo + 0.1-0.25% equity. Monthly time breakdown, conflict-of-interest rules.

### Changed
- README: новий `hiring/` розділ додано

### Why MINOR
- Закінчено повну launch-операційну документацію (LAUNCH_CHECKLIST + OKRs + DATA_ROOM)
- 6 blog posts published — content strategy complete для launch
- Twitter launch thread prepared — go-to-market ready
- Marketing site complete: 7 HTML pages, всі продуктові поверхні showcased
- Документація переходить з «strategic thinking» у «execution-ready»

### Added
- [`docs/LAUNCH_CHECKLIST.md`](docs/LAUNCH_CHECKLIST.md) — pre-launch gate. P0 safety/quality/business/marketing checks. T-24h до T+72h sequence. Roll-back plan.
- `content/post-06-30-user-interviews.md` — 6-й blog: «30 інтерв'ю до launch». Шаблон для post-research synthesis. Bias guards, what surprised us framework. ~13 хв read.
- `content/twitter-launch-thread.md` — 14-tweet launch thread, ready-to-post format. Anti-patterns для launch day. Follow-up schedule T+24h/T+72h/T+7d.

### Changed
- README: чек roadmap items closed

---

## [0.3.9] — 2026-05-14

### Added
- [`settings.html`](settings.html) — Settings screen mockup у phone-frame: профіль, нотифікації з toggles, інтеграції (Apple Health, Whoop, Oura), privacy controls (soft mode opt-out, analytics opt-out), підписка з cancel у 3 кліки, danger zone з GDPR delete
- [`docs/OKRS_2026.md`](docs/OKRS_2026.md) — Q3 + Q4 2026 OKRs з kill-criteria, scoring formula, weekly/monthly review cadence, personal founder OKRs
- [`docs/DATA_ROOM.md`](docs/DATA_ROOM.md) — 4-tier investor data room map (Public / Light NDA / Strong NDA / Closing-stage), process flow, sharing tools, status checklist готового vs to-prepare

### Changed
- README: settings.html, OKRs, DATA_ROOM додано до структури

### Added
- [`about.html`](about.html) — manifesto page: who we are, 14-agent team structure, 4 red lines, what we don't do (10-list), Ukraine context
- [`docs/SECURITY.md`](docs/SECURITY.md) — threat model, 5-layer defense (device/network/app/storage/processors), secrets management, CV-specific privacy, LLM prompt-injection mitigation, GDPR breach response, incident severity tiers
- `content/post-05-building-fitness-from-ukraine.md` — 5-й blog post: building fitness startup з України 2026. Особиста founder-перспектива, чому UA — advantage не despite, як говорити з investors, що hire community. ~10 хв read.

### Changed
- README: about.html, SECURITY.md додано до структури

### Added
- [`pricing.html`](pricing.html) — pricing page з toggle Monthly/Annual, geo-pricing display, comparison table 5 конкурентів, 8-question FAQ
- [`docs/EMAIL_SEQUENCE.md`](docs/EMAIL_SEQUENCE.md) — 18+ готових email templates: beta waitlist (A1-A5), active beta (B1-B5), newsletter (C1-C4), transactional (D1-D5), anti-patterns, frequency caps, success metrics
- [`docs/BETA_PLAYBOOK.md`](docs/BETA_PLAYBOOK.md) — як run TestFlight beta: 3 waves, daily/weekly/monthly ops, Slack structure, bug triage tiers, crisis protocols, post-beta retro

### Changed
- README: pricing.html додано до структури

### Added (from cloud · Sonnet 4.6)
- [`screens.html`](screens.html) — 5 статичних app-екранів у phone-мокапах для дизайн-рев'ю
  - **01 Today**: readiness ring (SVG анімований), Victor note, workout card, суб'єктивна готовність
  - **02 Workout**: CV-скелет acid lime (SVG суглоби + лінії), великий rep-counter, cue chip "Лопатки вниз", сет-доти, таймер відпочинку
  - **03 Coach**: Victor chat bubbles (контекстна розмова про плато жиму), input bar
  - **04 Progress**: 90-денний sparkline (SVG з градієнтом), PR-рядки, рядок консистентності (placeholder-позначені)
  - **05 Honest (§ Чесно)**: 6 пунктів "що FORM не робить" з ember-✕ маркерами, вбудований callout
- Hover-ефект на phone-мокапах (translateY + scale)
- Footer з нотатками по placeholder-числах, CV, clinical-safety посиланням, навігацією до index.html/onboarding.html

### Notes
- Числа на Progress-екрані — placeholder, потребують реальних beta-когортних медіан
- CV-скелет — ілюстративний; фінальний рендер залежить від MediaPipe / Vision API
- Всі копірайти на Honest-екрані пройшли clinical-safety review (v0.3.1)

---

## [0.3.5] — 2026-05-14

### Added (from local · Opus 4.7)
- [`nutrition/sample-3day-plan.md`](nutrition/sample-3day-plan.md) — 3-day sample meal plan (intermediate lifter, ~2600kcal). Кожен прийом з рецептом, макросами, swap-варіантами. Anti-ED дисципліна, no compensatory механіки. Sample-only.
- [`docs/API.md`](docs/API.md) — API design v0.1: auth (Apple SSO), workouts, nutrition, chat (streaming SSE), health snapshots, GDPR export, billing. Postgres schema. Rate limiting. Versioning.
- `content/post-03-we-tested-8-ai-trainers.md` — третій blog post: «Ми протестували 8 AI-тренерів». Деталі по Fitbod, Future, Whoop, Apple Fitness+, MacroFactor, Centr, Trainerize, Hevy. Чому жоден не «coach». ~16 хв read.

### Changed
- README: розділи `nutrition/`, оновлено `content/`

---

## [0.3.4] — 2026-05-14

### Added (from local · Opus 4.7)
- `content/post-02-what-good-form-means.md` — другий blog post: «Що насправді означає гарна форма», ~14 хв read, deep на трьох рівнях форми + чесний breakdown що камера може/не може
- [`docs/INVESTOR_OUTREACH.md`](docs/INVESTOR_OUTREACH.md) — playbook: target investor list (Tier 1-3 + anti-list), warm/cold templates, monthly update format, due-diligence pack, term sheet review, 10 prepared answers, 12-тижневий outreach calendar
- `programs/sample-strength-week.md` — приклад тижневої програми для intermediate, 5 сесій, з RPE-настройкою і пунктами де CV-camera coaches які параметри. Sample-only, для demo.

### Changed
- README: новий розділ `programs/` додано до структури

---

## [0.3.3] — 2026-05-14

### Added (from local · Opus 4.7)
- [`docs/HIRING.md`](docs/HIRING.md) — перші 3 ролі до Seed (Founding Engineer, Design Lead, Sport Science Advisor), interview rubric, compensation philosophy, onboarding 30/60/90, no-go профілі
- [`docs/PRESS.md`](docs/PRESS.md) — boilerplate (3 lengths), founder bio, company facts, ready-to-use quotes, FAQ для journalists, story angles
- `content/post-01-hrv-is-lying.md` — перший long-form blog post: «Чому твоє HRV тобі бреше». ~12 хв read, target Артур + Дмитро персони. Draft, pending sport-science advisor review.

### Changed
- README index extended (HIRING, PRESS) + новий розділ `content/`

---

## [0.3.2] — 2026-05-14

### Added (from local · Opus 4.7)
- [`docs/LEGAL.md`](docs/LEGAL.md) — privacy policy outline, ToS framework, GDPR special-category data handling, App Store compliance map, ED legal positioning, data breach response plan
- [`docs/METRICS.md`](docs/METRICS.md) — NSM (W-ACSU) detailed, AARRR funnel targets, anti-metrics, dashboard layout, cohort analysis framework, A/B test discipline
- [`docs/FOUNDER.md`](docs/FOUNDER.md) — founder narrative, red lines, anti-motivators, failure thresholds, personal sustainability rules. **Internal — not for agent modification.**

### Added (from cloud · Sonnet 4.6 · commit 9b44ab5, prior to 0.3.2 bump)
- `onboarding.html` — 8-screen onboarding prototype з ED-screening (SCOFF-light), tone selection, HealthKit permission flow
- [`docs/APP_STORE.md`](docs/APP_STORE.md) — App Store listing draft: 5 screenshot descriptions, keywords UA/EN, A/B copy variants

### Changed
- README index extended to include all 9 strategic docs + onboarding prototype

---

## [0.3.1] — 2026-05-14

### Added
- [`docs/FINANCIALS.md`](docs/FINANCIALS.md) — unit economics target model, 3-year P&L projection, sensitivity tables, Seed/Series A funding plan
- [`docs/COMPETITIVE.md`](docs/COMPETITIVE.md) — глибокий teardown Whoop, Fitbod, Future, MacroFactor, Hevy, Apple Fitness+, MyFitnessPal, Strava + defensibility map
- [`docs/RESEARCH.md`](docs/RESEARCH.md) — 30-інтерв'ю план, скрипт (25 питань × 5 блоків), bias guards, synthesis framework, ethics

### Changed
- README index доповнено новими доками

---

## [0.3.0] — 2026-05-14

### Added
- [`docs/PITCH.md`](docs/PITCH.md) — 12-slide investor pitch outline + one-pager + FAQ
- [`docs/PERSONAS.md`](docs/PERSONAS.md) — 4 detailed user personas (Дмитро, Олена, Артур, Богдан) з real-quote patterns
- [`docs/TECHNICAL.md`](docs/TECHNICAL.md) — архітектура, стек, CV-pipeline, latency-budget, build timeline

### Why MINOR
- Нова доковa система (стратегічний рівень: pitch + personas + technical)
- Перший раз з'являється архітектура застосунку у документації
- Personas стають референс-точкою для всіх feature-рішень

### Changed
- README: index з посиланнями на нові доки

---

## [0.2.1] — 2026-05-14

### Added
- `VERSION` файл — single source of truth для версії
- `CHANGELOG.md` — цей файл, формат Keep-a-Changelog
- `CONTRIBUTING.md` — стандарти комітів, версіювання, тегів
- Секція **Версіювання** у `README.md`
- Інструкція тегування `v0.2.X` у git

### Changed
- README: посилання на CHANGELOG і CONTRIBUTING додано
- Cloud routine prompt: тепер вимагає bump VERSION + CHANGELOG + git tag на кожну ітерацію

---

## [0.2.0] — 2026-05-14

### Added
- Marketing-сайт прототип ([`index.html`](index.html)) — editorial-естетика, 5 інтерактивних phone-екранів
- 14 субагентів у [`.claude/agents/`](.claude/agents/) (продукт, безпека, дизайн, маркетинг, growth, research, coordination)
- Бренд-бук v0.1 ([`docs/BRAND.md`](docs/BRAND.md)) — токени, типографіка, voice rules
- User flows v0.1 ([`docs/FLOWS.md`](docs/FLOWS.md)) — onboarding з ED-screening, daily ritual, cancel
- GTM v0.1 ([`docs/MARKETING.md`](docs/MARKETING.md)) — позиціонування, канали, ціна geo-priced
- Секція **§ 04 · Чесно** у маркетинг-сайті — 6 пунктів того, що FORM НЕ робить
- Honest cohort-goal badges замість fake-метрик

### Changed
- Голос Victor очищений за результатом brand-voice + clinical-safety брейнсторма
- «Не пропускай. Ніколи.» → «Не пропускай два поспіль.» (streak with grace)
- Гасло: «Тренер, який не пропускає» → «Тренер. У кишені.»
- Ціна: $19 → $9 UA / $19 Western (geo-priced)

### Removed
- Фабриковані метрики «94% retention», «+2.7 kg muscle» з лендингу
- Surveillance-мова про їжу («Я бачу пасту»)
- Compensatory механіки («-200 ккал завтра»)
- Hollywood-фрази: «Тіло пам'ятає все», «Дисципліна — це свобода»
- Possessive-мова: «Мені потрібен твій сон»

---

## [0.1.0] — 2026-05-13

### Added
- Перша версія marketing-сайту (концепт-прототип)
- 7 початкових субагентів (product-strategist, sports-scientist, nutrition-coach, clinical-safety, platform-engineer, brand-voice, design-craft)
- Базова естетика: warm-black + cream + acid lime + Fraunces/Manrope/JBMono

### Notes
- Перший проход без user research
- Кілька surveillance-моментів у копірайті (видалено у v0.2.0)
- Фабриковані метрики (видалено у v0.2.0)
