# FORM · Product Specification v0.1

> Технічна специфікація для Founding Engineer.
> Що будуємо, у якому порядку, і як знати що готово.
> Власник: `platform-engineer` + `product-strategist`. Правки через PR.

Читай разом з:
- [`docs/FLOWS.md`](FLOWS.md) — детальні user flows і Mermaid-діаграми
- [`docs/TECHNICAL.md`](TECHNICAL.md) — стек і latency budgets
- [`docs/API.md`](API.md) — Supabase schema і endpoint дизайн
- [`docs/BRAND.md`](BRAND.md) — токени, шрифти, кольори
- [`onboarding.html`](../onboarding.html) — онбординг прототип (8 screens)
- [`screens.html`](../screens.html) — 5 app screens прототип
- [`workout-live.html`](../workout-live.html) — workout UI прототип

---

## 1 · MVP Scope Definition

### Що є v1.0 (TestFlight Q3 2026)

| Feature | P0 (must ship) | P1 (want ship) | Owner |
|---|---|---|---|
| Apple Sign In | P0 | — | FE |
| Onboarding (8 screens з ED-screening) | P0 | — | FE |
| Today view (plan + HRV summary) | P0 | — | FE |
| Workout logging (manual + CV) | P0 | — | FE |
| CV pose coaching (3 base lifts) | P0 | — | FE |
| Meal logging (text + voice parse) | P1 | food photo | FE |
| Victor chat | P1 | — | FE |
| Progress screen (90-day) | P1 | — | FE |
| HealthKit integration (HRV + sleep) | P0 | resting HR | FE |
| Push notifications | P1 | — | FE |
| StoreKit 2 subscription | P0 | — | FE |
| Settings + 3-click cancel | P0 | — | FE |
| Crash reporting (Sentry) | P0 | — | FE |

### Що НЕ є v1.0

- Android (Q1 2027)
- Social / leaderboards / friend-system (post-PMF)
- Coach marketplace (post-PMF)
- Video workout library (P2)
- Wearable integrations окрім HealthKit (Oura, Whoop, Garmin — P2)
- B2B / corporate wellness (post-PMF)
- Web app (post-PMF)
- Russian-language UI (never)

---

## 2 · App Architecture

### File structure (suggested)

```
FORM/
├── App/
│   ├── FORMApp.swift
│   └── AppDelegate.swift
├── Core/
│   ├── Auth/
│   ├── Network/
│   ├── Storage/
│   └── Analytics/
├── Features/
│   ├── Onboarding/
│   ├── Today/
│   ├── Workout/
│   │   ├── Active/
│   │   └── History/
│   ├── Meals/
│   ├── Coach/
│   ├── Progress/
│   └── Settings/
├── CV/
│   ├── PoseDetector.swift
│   ├── RepCounter.swift
│   ├── FormAnalyzer.swift
│   └── Lifts/
│       ├── SquatAnalyzer.swift
│       ├── BenchAnalyzer.swift
│       ├── DeadliftAnalyzer.swift
│       └── [5 more lifts]
├── Victor/
│   ├── VictorClient.swift
│   ├── PromptBuilder.swift
│   ├── VoicePlayer.swift
│   └── Tone.swift
├── HealthKit/
│   ├── HKManager.swift
│   └── HKMapper.swift
└── UI/
    ├── DesignSystem/
    ├── Components/
    └── Theme/
```

### Dependency budget (Day 1)

Мінімум dependencies. Не додавай без обговорення з founder.

| Package | Purpose | Via |
|---|---|---|
| `supabase-swift` | DB + Auth | SPM |
| `sentry-cocoa` | Crash reporting | SPM |
| Anthropic (custom) | Claude API | URLSession wrapper |
| ElevenLabs (custom) | Victor TTS | URLSession wrapper |

**НЕ додаємо:** Alamofire, RxSwift, Combine-heavy frameworks, Firebase.

---

## 3 · Screen Specifications

### 3.1 Onboarding (8 screens)

Прототип: [`onboarding.html`](../onboarding.html)

**Screen 5 — ED Screening** ← clinical-safety gate, non-skippable
- 3 SCOFF-light questions (see FLOWS.md §2.2 for exact copy)
- If any flag raised → soft-path: no calorie targets, no body-comp features
- Do NOT store screening answers (privacy)
- Copy locked — changes require clinical-safety review

**Acceptance criteria:**
- [ ] Не можна пропустити Screen 5 (ED screening)
- [ ] HealthKit permission failure handled gracefully
- [ ] Весь flow ≤ 2 хвилини
- [ ] Дані збережені у Supabase до появи Screen 8

---

### 3.2 Today Tab

**State handling:**
- Empty → «Твій план готується»
- No HealthKit → HRV card hidden, no nag
- Rest day → «День відпочинку. Цього достатньо.»
- Offline → stale data з timestamp

**Acceptance criteria:**
- [ ] Load time < 1.5s (P95)
- [ ] Rest-day copy approved by clinical-safety

---

### 3.3 Workout Tab — Active Session

**CV trigger logic:**
- Camera активується тільки для CV-enabled lifts
- Якщо camera denied → log-only mode
- Якщо репи не визначаються > 10 сек → soft ask

**Acceptance criteria:**
- [ ] Session auto-saves кожні 30 сек
- [ ] Camera required тільки для CV lifts
- [ ] Victor audio non-blocking
- [ ] Weight/reps editable після CV запису

---

### 3.4 Meals Tab

**Note:** Весь tab відсутній для softPath users (ED-screening flag).

**Acceptance criteria:**
- [ ] Meals tab ПОВНІСТЮ відсутній для softPath users
- [ ] No calorie shame copy
- [ ] Protein-first display

---

### 3.5 Coach Tab (Victor Chat)

**Acceptance criteria:**
- [ ] System prompt кешований
- [ ] Injury keywords → hard-coded redirect до лікаря
- [ ] Victor не використовує «calories», «fat», «diet» без clinical-safety review

---

### 3.6 Progress Tab

**Copy rules:**
- Streak lost: «2 дні — не кінець. Просто наступний.»
- Long absence: «Добре що повернувся.»

**Acceptance criteria:**
- [ ] Streak grace = 2 days (не 1)
- [ ] No shame copy при streak loss

---

### 3.7 Settings

**Cancel flow:**
- Click 1: «Скасувати підписку»
- Click 2: «Ти впевнений?» + 3 factual bullets
- Click 3: redirect до Apple subscription management
- НЕ дозволяємо: retention popups, discount offers

**Acceptance criteria:**
- [ ] Cancel у ≤ 3 clicks (QA кожен реліз)
- [ ] Data export endpoint робочий до launch

---

## 4 · Victor AI — Implementation Guide

### Tone configuration

```swift
enum VictorTone: String, Codable {
    case smart      = "smart-direct"
    case mindful    = "mindful"
    case cold       = "cold-direct"
    case coach      = "old-school-coach"
}
```

### System prompt structure

```
[ROLE]
You are Victor, the AI coach in the FORM app.
Tone: {user.victorTone}
Language: {user.preferredLanguage}  // "uk" or "en"

[USER CONTEXT]
Training experience: {user.experience}
Current week: Week {program.currentWeek} of {program.totalWeeks}
Recent HRV trend: {hrv.trend}
Last 7 days: {workouts.last7}
Nutrition mode: {user.nutritionMode}  // "standard" | "soft-path"

[HARD LIMITS — never violate]
- Never give medical advice or diagnose injury
- If user mentions pain/injury: "Це варто обговорити з лікарем."
- Never comment on body weight/shape in evaluative terms
- Never use motivational poster phrases
- If soft-path mode: never mention calorie targets

[RESPONSE FORMAT]
- 1–3 sentences for chat
- Direct. No fluff.
```

### Routing

| Trigger | Model | Max tokens | Target latency |
|---|---|---|---|
| Coaching cue (CV) | Haiku 4.5 | 100 | < 800ms |
| Chat message | Sonnet 4.6 | 500 | < 3s |
| Weekly plan generation | Sonnet 4.6 | 2000 | < 10s |
| Food voice parse | Haiku 4.5 | 200 | < 1s |

---

## 5 · CV Pipeline — 8 Lifts

### Базові 3 ліфти (P0)

| Lift | Key joints | Rep trigger | Form alerts |
|---|---|---|---|
| Back Squat | Hip, knee, ankle; knee cave | Hip below parallel | Knee cave > 15°, forward lean |
| Bench Press | Elbow angle, wrist path | Elbow lock at top | Flared elbows |
| Deadlift | Hip hinge, spine angle | Full hip extension | Rounded back, bar drift |

### Наступні 5 (P1)

Overhead Press, Romanian Deadlift, Pull-up, Dumbbell Row, Push-up

### Accuracy targets

- Rep counting: ≥ 90% на base 3 lifts
- Tempo: ≥ 85%
- False-positive alerts: ≤ 15%
- **Kill criteria:** < 80% → CV вимкнути, ship without

### Implementation

```swift
// confidence threshold
let confidenceThreshold: Float = 0.6
// Frame budget: ≤ 33ms (30fps)
// Battery: disable CV after 45 min, reduce to 15fps if battery < 20%
// Privacy: відео не надсилаємо на сервер (on-device тільки)
```

---

## 6 · HealthKit Integration

| Data type | Read | Write | When |
|---|---|---|---|
| HRV (SDNN) | ✓ | — | Onboarding |
| Sleep analysis | ✓ | — | Onboarding |
| Resting heart rate | ✓ | — | Onboarding |
| Workout sessions | ✓ | ✓ | First workout |
| Body mass | — | — | Never (intentional) |

### HRV-to-plan logic

```
HRV_delta = today_hrv - rolling_7day_avg

if > +15%:  plan_modifier = "high-readiness"
elif < -15%: plan_modifier = "recovery"
else:        plan_modifier = "standard"

Victor comments only on high/recovery. Never nags on standard.
```

---

## 7 · StoreKit 2 — Subscription

| Product ID | Price | Market |
|---|---|---|
| com.form.coach.monthly.ua | ₴350 (~$9) | Ukraine |
| com.form.coach.monthly.eu | €19 | Western EU |
| com.form.coach.monthly.us | $19 | US, UK |
| com.form.coach.annual.ua | ₴3150 (~$79) | Ukraine |
| com.form.coach.annual.eu | €169 | Western EU |
| com.form.coach.annual.us | $169 | US, UK |

- 7-day free trial, feature-full
- 24h grace before hard entitlement lock (connectivity)
- No retention modal, no discount offer on cancel (DEC-019)

---

## 8 · Authentication

- Apple Sign In primary
- user_identifier → Keychain (not UserDefaults)
- Supabase Auth → JWT < 1h TTL
- Logout: clear Keychain + CoreData + Supabase session

---

## 9 · Data Model (Summary)

Повна схема у `docs/API.md`.

```
users          — id, apple_id, experience, goal, victor_tone, nutrition_mode
programs       — id, user_id, template_id, current_week
workouts       — id, user_id, date, duration, exercises_json, cv_session_id
cv_sessions    — id, workout_id, lift_type, reps_counted, form_events_json
food_entries   — id, user_id, date, items_json, source (voice/text/photo)
hk_snapshots   — id, user_id, date, hrv_sdnn, sleep_hours, resting_hr
```

Privacy: no raw voice audio, no video server upload, body mass never stored.

---

## 10 · Sprint Plan (12 тижнів → TestFlight)

> Founding Engineer hired M-1. Each sprint = 2 тижні.

| Sprint | Weeks | Goal | Done when |
|---|---|---|---|
| 0 | pre | Setup: Xcode Cloud, Supabase, Sentry, design tokens | CI pushes to TestFlight |
| 1 | 1–2 | Auth + Shell | Signed in, tab bar visible |
| 2 | 3–4 | Onboarding + Today | Plan visible after onboarding |
| 3 | 5–6 | Workout logging (manual) | Full workout saved to DB |
| 4 | 7–8 | CV pipeline (3 lifts) | ≥ 80% rep accuracy internal |
| 5 | 9–10 | Victor Chat + Voice | < 3s response, hard limits tested |
| 6 | 11–12 | Subscription + TestFlight prep | App in TestFlight, purchase works |

---

## 11 · Non-Functional Requirements

| Metric | Target |
|---|---|
| Today view load | < 1.5s P95 |
| Victor first token | < 3s |
| CV frame budget | ≤ 33ms |
| App binary size | < 50MB |
| Crash-free sessions | ≥ 99.5% |
| Color contrast | ≥ 4.5:1 WCAG AA |
| Touch target | ≥ 44×44pt |

---

## 12 · Engineering Discipline

### Pre-merge checklist
- [ ] SwiftUI Previews working
- [ ] No `print()` in production paths
- [ ] No force-unwraps without comment
- [ ] Privacy check: what does new API call store?
- [ ] Clinical-safety check: does copy touch food/body/pain?

### Branch strategy
```
main → production (TestFlight)
dev  → integration
feat/* → feature (from dev)
hotfix/* → from main
```

---

## 13 · Clinical Safety Checklist (per Sprint)

Required before any TestFlight push:

- [ ] ED screening non-skippable (UI test)
- [ ] softPath users see no Meals tab (UI test)
- [ ] Victor forbidden vocabulary passes (unit test on PromptBuilder)
- [ ] Injury redirect fires on keywords (unit test)
- [ ] Streak grace = 2 days (unit test)
- [ ] Cancel flow ≤ 3 clicks (UI test)

---

## Open questions

- [ ] ElevenLabs voice ID for Victor — decide before Sprint 5
- [ ] Anonymous trial (no Apple Sign In) — P1 or P2?
- [ ] TestFlight invitation process — waitlist from beta.html → manual?
- [ ] Supabase Vault cost at ~200 users

---

**v0.1 · травень 2026 · для Founding Engineer (pre-hire document)**

*Читай разом з FLOWS.md, TECHNICAL.md, API.md — не замість них.*
