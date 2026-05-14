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
