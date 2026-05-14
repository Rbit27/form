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

## [0.9.0] — 2026-05-14

### Why MINOR
- Новий документ: `docs/PRODUCT_SPEC.md` — повна технічна специфікація iOS-продукту для Founding Engineer
- Перший документ що синтезує FLOWS + TECHNICAL + API в єдиний "що будуємо" артефакт
- Включає 12-тижневий sprint plan, acceptance criteria, clinical-safety checklist, CV pipeline spec

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
