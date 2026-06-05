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
| Blog posts (drafts) | 68 |
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
- [ ] post-186: Нейропластичність і силовий тренінг — Pascual-Leone et al. (1995 PNAS): моторна кора розширюється при навчанні навичці; Kleim et al. (2002 J Neurosci): синаптичне зміцнення у моторній корі після рухового навчання; Cross-education і bilateral transfer; кортикальне ремоделювання при тривалому силовому тренінгу (Pearce et al. 2013 Eur J Neurosci); практичне: чому техніка потребує роботи ЦНС, а не лише м'язів; clinical-safety NOT REQUIRED
- [ ] post-187: Сила хвату — функціональна анатомія і тренінг (перенесено з post-172) — три механізми хвату (crush, pinch, support); Bohannon (2019 JSCR): grip як longevity proxy; flexor digitorum profundis vs superficialis; forearm extensor neglect; farmer's carry і deadlift strap debate; протокол розвитку від нуля; clinical-safety NOT REQUIRED
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
- [ ] post-220: Окислювальна здатність і силовий тренінг — Hood et al. (2019 Cell Metab): мітохондріальний вміст і силова адаптація; чому аеробна база покращує силовий тренінг через PCr-ресинтез і відновлення між підходами; concurrent training interference — реальна при великому об'ємі кардіо, але не при помірному; стратегія для атлетів, що хочуть і силу, і витривалість; clinical-safety NOT REQUIRED
- [ ] post-221: Дроп-сети — механіка і застосування — відміна від rest-pause: повне зниження навантаження без паузи; механічна vs. метаболічна втома при drop set; Fink et al. (2017 Eur J Sport Sci): еквівалентна гіпертрофія при нижчому часі; практичне програмування і відмінність від «тренування до відмови»; clinical-safety NOT REQUIRED
- [ ] post-222: Генетика і тренувальна відповідь — ACTN3 R577X і силова адаптація (Yang et al. 2003 Am J Hum Genet); ACE I/D і витривалісний відгук (Montgomery et al. 1998 Nature); полігенна варіація і гетерогенність відповіді (Bouchard et al. 1999 Med Sci Sports Exerc HERITAGE study); чому середній ефект тренінгу маскує розподіл — high vs. low responders; clinical-safety NOT REQUIRED
- [ ] post-223: Плацебо і ноцебо у тренінгу — Beedie & Foad (2009 Sports Med): placebo effects у спорті; очікування RPE і вибір темпу; ноцебо: чому «попередження про болючість DOMS» знижує продуктивність; механізм через ендогенні опіоїди і Central Governor; практичне: framing програми впливає на adherence без маніпуляції; clinical-safety NOT REQUIRED
- [ ] post-224: Довгострокова атлетична адаптація — LTAD (Long-Term Athlete Development); чим відрізняється програмування на рік від програмування на 5+ років; принцип «акумулятивного потенціалу» (Zatsiorsky); як змінюється адаптаційний відгук з тренувальним стажем і що це означає для intermediate / advanced; clinical-safety NOT REQUIRED

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
**Last update:** 4 червня 2026 · v1.86.0
