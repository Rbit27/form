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
- [ ] post-106: Жіночий підхід до силового: що відрізняється анатомічно — Q-angle myth, hip width, estrogen і сухожилля (Lowe 2017), ACL injury risk; що реально змінює програмування; clinical-safety PASS conditional
- [x] post-94: Tendon adaptation vs muscle adaptation — різна часова шкала (м'язи: 2–4 тижні, сухожилля: 8–12 тижні); Bohm et al. (2015) tendon stiffness і RFD; чому бігуни і важкоатлети отримують сухожильні травми після стрибків навантаження, які м'язи вже переносять; практичні наслідки для програмування; clinical-safety PASS
- [x] post-95: Пікова сила vs силова витривалість — дві різні нейром'язові цілі; чому training for both одночасно може давати interference effect; блокова логіка для аматорів з обмеженим часом; clinical-safety PASS
- [x] post-96: Warming up vs activation — різниця між підвищенням температури тканин (ефективний warmup) і «активацією» (нейром'язовий праймінг); що реально робить PAP і для кого він релевантний; Fradkin et al. (2010) огляд; clinical-safety PASS

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
