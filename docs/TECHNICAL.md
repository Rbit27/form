# FORM · Technical Architecture v0.1

> Stack. Архітектура. Бюджет latency. Що on-device, що в cloud.
> Власник: `platform-engineer`. Правки через PR з обґрунтуванням.

---

## 1 · Стек (рекомендований)

### Mobile (primary surface)

**iOS — first, primary:**
- Swift 6 + SwiftUI
- Apple Vision framework (VNDetectHumanBodyPoseRequest) для pose
- HealthKit (HRV, sleep, workouts, heart rate)
- AVFoundation для camera capture
- WidgetKit + LiveActivity для workout-in-progress
- StoreKit 2 для підписок
- iOS 17+ baseline (CoreML 7 потрібен)

**Android — phase 2 (M4+):**
- Kotlin + Jetpack Compose
- ML Kit Pose Detection
- Health Connect API
- CameraX
- Google Play Billing
- API 28+ baseline

**Чому не cross-platform (React Native / Flutter):**
- CV-perf вимагає native API доступ (Vision / ML Kit)
- HealthKit / Health Connect ergonomics native кращі
- Команда стартує iOS-first, додаємо Android коли є PMF

### Backend

**Мінімальний, тонкий:**
- **Cloudflare Workers** для edge endpoints (auth, sync, push)
- **Supabase** для:
  - Postgres (users, programs, workouts, food entries)
  - Auth (Apple Sign In primary)
  - Real-time для chat (low priority, можна без)
- **Anthropic API** для всього LLM (Claude Sonnet 4.6 default, Haiku для класифікації)
- **ElevenLabs** для voice (Victor TTS)
- **R2 / S3** для backup user data export (rare)

**Чого свідомо НЕ робимо у v0.1:**
- Own infrastructure (Kubernetes, тощо)
- Microservices
- Apache Kafka, queue brokers, мікросвіт
- ML training pipeline (поки не треба — використовуємо foundation models)

### LLM-роутинг

| Use case | Model | Context | Latency | Cost |
|---|---|---|---|---|
| Real-time coaching cue | **Haiku 4.5** | Cached prompts + minimal context | < 800ms | $$ |
| Chat з користувачем | **Sonnet 4.6** | History + persona | 1–3s | $$$ |
| Plan generation (weekly) | **Sonnet 4.6** | Full user state | 5–10s | $$$ |
| Food parsing voice → log | **Haiku 4.5** | Short text | < 1s | $$ |
| Image food recognition | **Sonnet 4.6 (Vision)** | Image + categories | 2–4s | $$$ |
| Weekly review essay | **Sonnet 4.6** | Week of data | 8–12s (async) | $$$ |

Prompt caching ON для всіх — 5-min TTL, до 90% cost reduction на повторних викликах.

---

## 2 · CV-пайплайн

### High-level

```
Camera (30fps)
  ↓
Apple Vision pose estimation (on-device, ~20ms/frame)
  ↓
Keypoint smoothing (Kalman filter, last 5 frames)
  ↓
Exercise classifier (rules-based per active exercise)
  ↓
Rep counter (joint trajectory peaks)
  ↓
Tempo measurer (time between phases)
  ↓
Form heuristics (custom, per-exercise)
  ↓
Cue trigger (≤ 1 per 5 sec)
  ↓
Voice playback (cached library) OR text overlay
```

### Точність очікувана (чесно)

| Метрика | Очікувано на v1.0 |
|---|---|
| Pose keypoints (35 точок) | 92–97% confidence у good light |
| Rep counting на 8 базових | 95%+ |
| Tempo measurement | 90%+ |
| Form deviation detection (grossly) | 70–85%, залежно від руху |
| Form deviation detection (subtle) | 40–60% (не shippable як «coaching») |

### 8 базових рухів (MVP scope)

1. Back squat (sagittal view)
2. Romanian deadlift (sagittal)
3. Conventional deadlift (sagittal)
4. Bench press (lateral view + camera mount)
5. Overhead press (frontal)
6. Bent-over row (sagittal)
7. Push-up (lateral)
8. Pull-up (frontal)

**Setup expectation:**
- Camera at 2m, eye-level, single subject
- Decent gym light
- User wears fitted clothing (loose hoodie = poor detection)

**Edge cases:**
- Bad light → fall back to manual rep counting, no form coaching
- Subject occluded → pause counter, show "не бачу тебе"
- Multiple people → use closest+centered subject

---

## 3 · Дані-флоу

### What stays on-device (privacy floor)

- Video frames (always)
- Pose keypoints (raw, real-time)
- Rep counts during set (synced after set complete)
- Voice notes (transcribed locally if possible)

### What goes to cloud

- Aggregated session data (sets, reps, weights, tempo summaries)
- HRV / sleep / heart rate (HealthKit → Supabase)
- Food entries (text-based)
- Chat messages (LLM context)
- Photos (if user takes one, optionally — opt-in)

### What's NEVER uploaded

- Video frames (camera feed)
- Raw audio (voice commands)
- Biometric face data
- Location

### Privacy claim чесний

> "Camera video and audio never leave your device. We use on-device computer vision for pose estimation, and we never store or upload raw frames."

(Це треба підтвердити кодом і третьою стороною аудитом до production.)

---

## 4 · Архітектура додатку

```
┌─────────────────────────────────────────────────────────────┐
│  iOS App (SwiftUI)                                          │
│                                                              │
│  ┌──────────────┐  ┌─────────────┐  ┌──────────────────┐    │
│  │  Today       │  │  Workout    │  │  Meals / Chat    │    │
│  │  (Dashboard) │  │  (CV Live)  │  │  (Logging)       │    │
│  └──────────────┘  └─────────────┘  └──────────────────┘    │
│         │                 │                  │              │
│  ┌──────┴─────────────────┴──────────────────┴──────────┐   │
│  │           Core (Swift, in-process)                    │   │
│  │  - StateStore (Combine + Observable)                  │   │
│  │  - HealthKit bridge                                   │   │
│  │  - Vision pipeline (camera → pose → cues)            │   │
│  │  - LLM client (cached prompts, streaming)            │   │
│  │  - Local DB (SwiftData / Core Data)                  │   │
│  └────────────────────────┬──────────────────────────────┘   │
│                            │                                 │
└───────────────────────────┼────────────────────────────────┘
                            │ HTTPS (auth, sync only)
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  Edge (Cloudflare Workers)                                  │
│  - Auth proxy (Apple Sign In token → session)               │
│  - Sync API (debounced batch upload of aggregated data)     │
│  - LLM proxy (with key vault + rate limiting + caching)     │
│  - Webhook receivers (Whoop / Oura — phase 2)               │
└────────────┬──────────────────────────┬─────────────────────┘
             │                          │
             ▼                          ▼
┌──────────────────────┐    ┌───────────────────────────────┐
│  Supabase            │    │  Anthropic API                │
│  - Postgres          │    │  - Sonnet 4.6 (chat / plan)   │
│  - Auth (Apple SSO)  │    │  - Haiku 4.5 (parse / cue)    │
│  - Realtime (chat)   │    │  - Prompt caching on          │
└──────────────────────┘    └───────────────────────────────┘
```

### Чому така архітектура

- **Mobile-fat, server-thin** — швидко на старті, низькі ops costs, privacy за замовчуванням
- **Edge для всього server-side** — низька latency, нема cold start, копійки
- **Supabase замість власного backend** — auth + DB + realtime з коробки, ~1 інженер замість 3 на старті
- **LLM проксі окрема відповідальність edge worker** — key vault + caching + rate limit + observability в одному місці

---

## 5 · Бюджет latency

| Дія | Цільовий budget | Що включає |
|---|---|---|
| App cold start → Today | < 1.5s | Bundle + state restore + HealthKit warm-up |
| Workout start → first frame analyzed | < 800ms | Camera init + Vision warm-up |
| Rep detected → counter updated | < 100ms | On-device, instant |
| Form deviation → coaching cue | < 600ms | Heuristic + cue library lookup |
| User question → Victor answer (chat) | < 2.5s (streaming start) | Anthropic API + prompt cache hit |
| Voice food log → entry confirmed | < 1.2s | Local STT + Haiku parse |

---

## 6 · Build timeline (агресивний, але виконуваний)

```
Q3 2026 (M0–M3)
├── M0  · Setup + design system in Xcode + Hello-World CV
├── M1  · Today screen + HealthKit + onboarding (without CV)
├── M2  · Workout screen with CV beta (presquit only)
└── M3  · TestFlight closed beta (200 users), 3 lifts
                                       
Q4 2026 (M4–M6)
├── M4  · Chat + nutrition + plan engine
├── M5  · 8 lifts CV pipeline complete
└── M6  · Public release on App Store

Q1 2027 (M7–M9)
├── M7  · Android development starts
├── M8  · Whoop / Oura integrations
└── M9  · Android closed beta

Q2 2027 (M10–M12)
├── M10 · Android public release
├── M11 · B2B pilot (1–2 companies)
└── M12 · Series A readiness
```

**Critical-path риски:**
- CV-якість на 8 рухах до M5 — найбільший unknown
- Apple Vision API limitations on older devices (iPhone 12 SE, etc.) — треба графовий fallback
- HealthKit edge cases (multiple Apple Watch sources) — давній bug-fest

---

## 7 · Operational discipline

### Code review

- Кожен PR — 1 reviewer мінімум
- Жодних direct-to-main push (виключення: cloud-агент on this repo, з обмеженнями)
- Pre-commit hooks: lint, type check, test
- Coverage target: 60% (не 100% — pragmatic)

### Observability

- **Sentry** для crashes
- **PostHog** для product analytics (privacy-respect: no PII leaks)
- **Cloudflare Analytics** для edge

### Безпека

- Apple Sign In primary (no password handling)
- Secrets у Cloudflare KV
- Anthropic key never on client
- HealthKit data — Apple-encrypted at rest
- GDPR export endpoint: full JSON download in < 30 sec

---

## 8 · Стартап-готовий чек-лист (Seed)

| Категорія | Готово? |
|---|---|
| Apple Developer Program enrolled | TBD |
| GitHub org з branch protection | TBD (зараз personal account, на Seed — переносимо) |
| Supabase project (free tier) | TBD |
| Anthropic API key + monitoring | TBD |
| ElevenLabs voice clone (Victor) | TBD |
| Sentry + PostHog | TBD |
| Cloudflare Workers + KV | TBD |
| Domain — form.coach (.coach is free as of v1) | TBD |
| Privacy Policy + ToS drafts | TBD (legal review pre-launch) |

---

## 9 · Чого свідомо немає в v0.1

- **AR-визуалізація форми** (over-engineering поки CV-baseline немає)
- **Watch app** (важливо, але phase 2)
- **Voice-only mode** (для тренування без екрану — phase 2)
- **Multi-user / family** (не PMF-relevant)
- **Гейміфікація з рангами** (anti-pattern до retention proof)
- **Marketplace тренувань** (creator economy — phase 3+)

---

## 10 · Open Technical Questions

1. **CV-фрейми обробляти кожен N-й чи keep-30fps?**
   Hypothesis: 15 fps достатньо для коучингу, рятує батарею на 30%. Need to validate.

2. **Voice latency через streaming TTS чи pre-rendered cue library?**
   Hypothesis: Hybrid — cached 200 базових cues + LLM fallback для контексту. ElevenLabs streaming для chat-голосу.

3. **HRV-нормалізація між пристроями (Whoop RMSSD vs Apple SDNN)?**
   Hypothesis: Per-source baseline track, normalize до z-score, не mix raw values. Потрібен 14-day baseline до перших insights.

4. **Offline mode — повний чи частковий?**
   Hypothesis: Workout працює офлайн (CV on-device), sync coли інтернет. Chat і plan-generation потребують онлайну.

---

**v0.1 · травень 2026 · правки приймає `platform-engineer`**
