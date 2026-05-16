# FORM · Mobile Product Roadmap v0.1

> **Purpose of this document:** The FORM iOS app is built in a separate repository (private, Swift/Xcode). This document is the canonical reference for mobile product scope, technical constraints, deferred decisions, and integration points so that cloud agents working in this repository do not attempt to write, scaffold, or modify mobile code here. All new mobile feature ideas, roadmap changes, and technical decisions go into this document first — not into code in this repo.
>
> Owner: `platform-engineer` + `product-strategist`. Safety gate: `clinical-safety`. Review: before each milestone boundary or on architecture change.
>
> Reads alongside: `docs/TECHNICAL.md`, `docs/FLOWS.md`, `docs/PRODUCT_SPEC.md`, `docs/DATA_MODEL.md`, `docs/OBSERVABILITY.md`, `docs/VICTOR_PROMPT_GUIDE.md`, `docs/DECISION_LOG.md`.

---

## Table of Contents

1. [Document Purpose and Boundaries](#1-document-purpose-and-boundaries)
2. [Platform Decisions Already Made](#2-platform-decisions-already-made)
3. [v1.0 Scope — TestFlight Beta (200 users)](#3-v10-scope--testflight-beta-200-users)
4. [v1.1 Scope — App Store Release](#4-v11-scope--app-store-release)
5. [v2.0 Horizon — Post-Seed, Post-Founding-Engineer](#5-v20-horizon--post-seed-post-founding-engineer)
6. [Technical Constraints](#6-technical-constraints)
7. [Deferred Items — Explicitly Out of v1](#7-deferred-items--explicitly-out-of-v1)
8. [Integration Touchpoints with This Repo](#8-integration-touchpoints-with-this-repo)
9. [Open Questions for the Founding Engineer](#9-open-questions-for-the-founding-engineer)

---

## 1. Document Purpose and Boundaries

### 1.1 What this document is

This is the mobile product roadmap for FORM. It defines what the iOS app must do, in what order, under what technical constraints, and what is explicitly deferred. It is the authoritative source for:

- Feature scope by version
- On-device vs. cloud boundaries
- Latency and battery budgets the Founding Engineer must hit
- Integration contracts between the mobile app and the backend services documented in this repo
- Decisions that cloud agents must not override unilaterally

### 1.2 What this document is not

This is not an implementation guide. It contains no Swift code, no Xcode project structure, and no build configuration. Those live in the mobile repository. When agents in this repo generate content related to mobile, they should reference this document for scope and constraints, then stop. They should not write Swift files, Xcode project files, or mobile-specific CI/CD configuration into this repository.

### 1.3 Rule for cloud agents

If a task involves mobile implementation — writing Swift, modifying a `.xcodeproj`, drafting a native UI component, or configuring an iOS SDK — the correct output is a note pointing to the relevant section of this document and a request to the Founding Engineer. Cloud agents do not ship mobile code from this repo.

---

## 2. Platform Decisions Already Made

These decisions are locked. They are recorded in `docs/DECISION_LOG.md`. Agents should not re-litigate them without a new DEC entry approved by the founder.

| Decision | Ref | Detail |
|---|---|---|
| iOS-first, Android deferred to Q1 2027 | DEC-015 | Single platform focus for launch quality |
| Swift + SwiftUI, no cross-platform framework | DEC-015 | CV-pipeline requires native Vision framework access |
| Apple Sign In as primary auth | TECHNICAL.md §7 | No password handling; OAuth on device |
| On-device CV via Apple Vision framework + CoreML | DEC-006, TECHNICAL.md §2 | Video frames never leave device |
| HealthKit for biometric data | TECHNICAL.md §1 | HRV (SDNN), sleep stages, heart rate, active energy |
| No Android Health Connect in v1 | DEC-015 | Android not in scope until post-PMF |
| No Apple Watch companion app in v1 | TECHNICAL.md §9 | Deferred post-launch |
| Anthropic Claude API for Victor | TECHNICAL.md §1 | Sonnet 4.6 for chat/plan, Haiku 4.5 for classification |
| ElevenLabs for Victor's voice (TTS) | TECHNICAL.md §1 | Streaming + R2-cached cue library |
| Supabase (Postgres + Auth + Realtime) | TECHNICAL.md §1 | Mobile-fat, server-thin architecture |
| SwiftData or Core Data for local persistence | TECHNICAL.md §4 | Offline-first workout logging |
| StoreKit 2 for subscriptions | PRODUCT_SPEC.md §1 | No RevenueCat wrapper at v1.0 (Founding Engineer decision) |
| No social features | DEC-002 | No leaderboards, no friend-system, no in-app community |
| No free tier | DEC-016 | 14-day trial, then subscription |
| 3-click cancel maximum | DEC-019 | Ethical off-ramp, no dark patterns |
| ED-screening non-skippable at onboarding | DEC-018 | Clinical safety floor, `clinical-safety` veto right |
| Editorial-brutalist design (Fraunces + Manrope) | DEC-007 | Warm-black, cream, acid lime (#DDFF2D) palette |
| iOS 17+ baseline | TECHNICAL.md §1 | CoreML 7 required for CV pipeline |
| No Russian language UI | DEC-021 | Ukrainian and English only |

---

## 3. v1.0 Scope — TestFlight Beta (200 users)

**Target:** Closed TestFlight beta, Q3 2026 (M3 in TECHNICAL.md §6 timeline). Gate: 200 real users completing at least one coached workout session.

### 3.1 Core App Shell

**P0 — must ship for TestFlight to open:**

- Apple Sign In + Supabase session handshake
- Onboarding flow: 8 screens including ED-screening (non-skippable), goal selection, experience level, training frequency, Victor tone preference. Full flow defined in `docs/FLOWS.md §2`.
- Tab bar: Today / Workout / Meals / Coach / Progress — 5 tabs, fixed structure per `docs/FLOWS.md §1`
- Settings screen: profile, Victor tone, notifications, integrations, privacy, subscription, support, about
- 3-click cancel with single retention offer maximum, per DEC-019
- Data export (full JSON, one button) and account deletion (30-day grace period, GDPR Art. 17)
- Crash reporting via Sentry iOS SDK with `beforeSend` hook stripping health-adjacent fields before transmission (required for GDPR compliance; see `docs/OBSERVABILITY.md §8.2`)

### 3.2 Today Screen (Dashboard)

**P0:**

- Date + contextual greeting (not «good morning» when it is afternoon)
- Today's planned workout card with Start CTA
- HealthKit readiness summary: HRV number in raw form during first 14 days; readiness ring visible only after 14-day baseline is established
- Victor's daily note: 1–2 sentences generated by Haiku 4.5 from HRV + sleep + today's program context. Streamed, not blocked.
- Remaining macros display (calories remaining, not consumed — anti-surveillance framing per `docs/FLOWS.md §3.1`)

**P1 — want for TestFlight, not a gate:**

- WidgetKit small widget: today's workout + readiness ring
- Live Activity (Dynamic Island / Lock Screen) during active workout session

### 3.3 Workout Screen — CV Pipeline

This is the primary technical differentiator and the highest implementation risk item. See `docs/TECHNICAL.md §2` for the full CV pipeline architecture.

**P0 — 3 base lifts at TestFlight open:**

- Back squat (sagittal view)
- Romanian deadlift (sagittal view)
- Conventional deadlift (sagittal view)

**Accuracy expectations (honest, from TECHNICAL.md §2):**

| Metric | Expected at v1.0 |
|---|---|
| Rep counting on 3 base lifts | 95%+ in good light, stable camera |
| Tempo measurement | 90%+ |
| Gross form deviation detection | 70–85%, lift-dependent |
| Subtle form deviation (e.g., valgus < 5 degrees) | Not shippable as coaching; blocked by `clinical-safety` |

**Camera setup requirements (communicated to user in-app):**

- Camera at approximately 2 meters, eye-level, tripod or propped
- Single subject in frame
- Adequate gym lighting (fall back to manual mode in poor light)
- Fitted clothing recommended (loose fabric degrades keypoint confidence)

**Real-time coaching cues — rules (from FLOWS.md §4.1):**

- Maximum 4 words per cue
- Maximum 1 cue per 5 seconds (no spam)
- User choice: voice (ElevenLabs TTS from cached library) or text overlay
- Cues sourced from pre-rendered library for common deviations; live LLM fallback for contextually novel moments only
- Victor tone applied to cue wording (smart-direct default, per user preference)

**Fallback behavior (mandatory):**

- Poor light detected: fall back to manual rep counting, disable form coaching, display explanation
- Pose lost for more than 5 consecutive seconds: pause counter, surface «can't see you — adjust phone» message
- Multiple subjects in frame: use closest and most-centered; no multi-person coaching
- Battery below 15%: offer to continue without camera, as per `docs/FLOWS.md §4.2`

**Manual mode (always available as alternative to camera):**

- Tap-to-count reps
- RPE prompt (1–10 scale, where 10 = complete failure) after each set
- Rest timer (90 seconds default, adjustable)

### 3.4 Meal Logging

**P1 — want for TestFlight:**

- Voice-to-log: local STT (Apple Speech framework) → text → Haiku 4.5 parse → food entry confirmation. Target: under 1.2 seconds end-to-end per `docs/TECHNICAL.md §5`.
- Manual search: food database search (Supabase-backed)
- Anti-shame framing enforced throughout: no red indicators over calorie limit, no compensatory messaging, no missed-meal shaming. Full rules in `docs/FLOWS.md §5.1`.

**Deferred to v1.1:**

- Photo-to-log: camera capture → Sonnet 4.6 Vision API → food recognition + portion estimate. Deferred because image latency (2–4 seconds) and accuracy in real gym/kitchen environments need field validation before the flow is polished enough to ship.

### 3.5 Coach (Victor Chat)

**P1 — want for TestFlight:**

- Chat interface with Victor, full conversation history
- Message classification via Haiku 4.5: training, nutrition, injury/pain, mental health, tech/how-to, other. Routing per `docs/FLOWS.md §6.1`.
- Clinical-safety triage on any pain or injury mention — mandatory, not optional, cannot be softened by prompt changes without `clinical-safety` sign-off
- Self-harm reference handling: resources surface immediately, soft response, not redirected to training topics
- Streaming response display (time to first token target: under 800ms on cache hit)
- Victor tone mode applied per user preference (smart-direct / mindful / cold / coach)

### 3.6 Progress Screen

**P1 — want for TestFlight:**

- Strength index composite from main lifts (squat, deadlift, bench)
- Weight trend sparkline (90-day, manual entry only, completely opt-in, hideable permanently)
- PR tracking for the 3 CV-coached lifts plus any manually entered PRs
- Consistency view with grace rule: streak breaks only after 2 consecutive misses, not 1 (DEC-013)
- No BMI, no body fat percentage display (blocked unless user explicitly opts in and enters manually)
- No before/after photo feature in v1

### 3.7 HealthKit Integration

**P0:**

- Authorization request: HRV (SDNN), sleep stages, heart rate, active energy. Request only at relevant moment, not at app launch (per `docs/FLOWS.md §2.2` permission-timing rule).
- HRV baseline computation: 14-day rolling window before readiness ring appears
- Sleep stage read: used for overnight recovery context in Victor's morning note and plan adjustments
- Heart rate read: used during workout sessions for intensity reference
- Workout write: completed sessions written back to HealthKit as `HKWorkout` records

**Denied permission handling:**

- App works fully without HealthKit. Manual workout logging, no readiness ring, no HRV context. Reminder offered once at 14 days if denied — no repeated prompting.

### 3.8 Subscriptions and Billing

**P0:**

- StoreKit 2 integration: 14-day free trial, monthly and annual subscription options
- Trial-to-paid conversion flow: no dark patterns, no false urgency
- Cancel flow: ≤ 3 taps from profile, maximum one retention offer (discount), mandatory acceptance of «no» (DEC-019)
- Subscription state synced to Supabase for backend-side entitlement checks
- PostHog events: `trial_started`, `subscription_activated`, `subscription_cancelled`, `offer_accepted`, `offer_declined`

### 3.9 Observability — Mobile Layer

These are requirements the Founding Engineer must implement. They connect to `docs/OBSERVABILITY.md §2.1` SLOs.

**P0:**

- Sentry iOS SDK installed with `beforeSend` hook that strips `workout_data`, `body_metrics`, `hrv_value`, and `coaching_context` from all crash payloads before transmission
- Crash-free session rate target: 99.5% (SLO from OBSERVABILITY.md)
- PostHog iOS SDK: onboarding funnel events, workout session events, Victor interaction events, subscription events
- W3C TraceContext `traceparent` header on all outbound API requests so traces correlate with Cloudflare Worker spans

**P1:**

- Sentry Performance: app start time, screen load time, API call duration from client perspective
- PostHog: feature flag exposure events for any canary flags used during beta

---

## 4. v1.1 Scope — App Store Release

**Target:** Public App Store submission, Q4 2026 (M6 in TECHNICAL.md §6 timeline). Gate: beta data showing session completion ≥ 75%, D7 retention ≥ 30%, crash-free rate ≥ 99.5%.

### 4.1 CV Pipeline — 5 Additional Lifts

Lifts added after TestFlight validation of the 3-lift baseline:

- Bench press (lateral view; requires camera mount or spotter-placed phone)
- Overhead press (frontal view)
- Bent-over row (sagittal view)
- Push-up (lateral view)
- Pull-up (frontal view)

Note: bench press is the highest-risk lift for CV quality because the lateral view requires an elevated, stable camera position that most gym users will not have. The in-app setup screen must communicate this honestly. If camera position is not viable, manual mode is the graceful fallback — not a broken experience.

### 4.2 Meal Logging — Photo Recognition

- Photo capture → Sonnet 4.6 Vision API → food item list + portion estimate → user confirmation step
- Target latency: 2–4 seconds for recognition response (acceptable for a deliberate logging action, not a real-time flow)
- Fallback if recognition confidence is low: show top 3 guesses with portion options, not a single auto-confirm
- Gated behind beta validation: if photo accuracy in real kitchen/gym lighting is below 70% match on user confirmation, it does not ship in v1.1

### 4.3 Push Notifications — Full System

**Hard limits (non-negotiable, per FLOWS.md §9):**

- Maximum 2 notifications per day at default settings
- No notifications between 22:00 and 07:00 (quiet hours, user-adjustable)
- No food-related notifications in real time
- No streak-shame notifications

**Notification categories:**

- Morning check-in (07:00–08:30, smart-scheduled near wake time via HealthKit sleep data): HRV summary + today's session
- Pre-workout reminder (30 minutes before planned workout time): short, specific to today's program
- Recovery cue (21:00–22:00, only when sleep quality signal exists): sleep timing recommendation tied to next training day
- Weekly review (Monday ~09:00): Victor's 60–80 word text summary of the past week

**Anti-spam enforcement:**

- Auto-reduce frequency if user ignores 3 or more consecutive notifications
- Single re-engagement notification after 7+ days of no logged workouts, then stop
- No «we miss you» copy, no guilt framing (DEC-014)

### 4.4 Weekly Victor Review

- Every Monday, Victor generates a 60–80 word written summary of the past week: lifts completed, notable PRs, recovery pattern, one forward-looking note
- Generated async by Sonnet 4.6 with full week of session data as context; delivered as push notification + in-app card on Today screen
- Generation happens server-side (Cloudflare Worker + Anthropic API), not on-device
- PostHog event: `weekly_review_delivered`, `weekly_review_opened`

### 4.5 Localization — Ukrainian and English

Per `docs/I18N.md`, FORM ships in Ukrainian and English at App Store release. No Russian language UI (DEC-021).

- All in-app strings externalized to localization files in the mobile repo
- Victor's voice responses and coaching cues generated in the language matching the user's app language setting
- App Store listing: English primary, Ukrainian secondary
- ElevenLabs voice: one English voice clone (Victor), one Ukrainian voice clone (Victor) — both required for v1.1

### 4.6 App Store Compliance and Privacy

- App Privacy Report (Data Nutrition Label): accurate declaration of HealthKit data types, analytics (PostHog, Sentry), no advertising data
- Privacy Manifest file (required by Apple from iOS 17): declare all SDK data uses and required reason APIs
- ATT (App Tracking Transparency) prompt: not applicable if PostHog is configured for no cross-app tracking (verify before submission)
- GDPR consent flow for EU users: explicit consent for health data processing (special category under GDPR Art. 9) before HealthKit authorization is requested. Separate consent step — not bundled with Terms of Service acceptance.
- Age gate: users under 18 cannot proceed past the gate screen (FLOWS.md §2.2)
- PAR-Q screening: users 65 and over are prompted with 6 cardiovascular readiness questions before any training plan is generated (FLOWS.md §2.2)

---

## 5. v2.0 Horizon — Post-Seed, Post-Founding-Engineer

**Target:** Q2–Q3 2027. Contingent on: Seed round closed, Founding Engineer hired and ramped, beta data confirming CV coaching as retention driver, and at least one additional engineering hire.

Items in this section are directional — they inform what the architecture must not block, but they are not committed scope. Each item requires its own scoping, design, and technical feasibility review before it enters a milestone.

### 5.1 Apple Watch Companion App

A native WatchOS app has been deferred since pre-launch (TECHNICAL.md §9). The case for v2.0:

- Heart rate and HRV from Apple Watch are higher quality than iPhone-estimated values
- Active workout session control (pause, skip set, finish) from wrist reduces phone interaction during lifts
- Rest timer on wrist is a high-demand user request anticipated from beta feedback

**What this requires:**

- WatchOS 10+ baseline (paired with iPhone running iOS 17+)
- WatchKit + SwiftUI for Watch (Complication + full-screen session view)
- WatchConnectivity framework for real-time session state sync between phone and watch
- HealthKit workout session managed from watch side during active workout (more accurate than iPhone)
- Battery impact assessment: watch app running concurrent with phone CV pipeline must be validated together

**What this is not:**

- The watch app is not a standalone app; it requires the paired iPhone for LLM features, sync, and plan data
- It does not replace the phone as the camera for CV coaching

### 5.2 Android (Kotlin + Jetpack Compose)

Per DEC-015, Android development begins Q1 2027 (M7 in TECHNICAL.md §6 timeline). The mobile roadmap for Android is not detailed in this document — it will be a separate addendum once the Founding Engineer is hired and the iOS baseline is stable. Items to preserve now:

- The backend API (Cloudflare Workers + Supabase) is designed platform-agnostic. No iOS-specific assumptions in the API layer.
- Health Connect (Google's replacement for Google Fit) is the Android equivalent of HealthKit. HRV access on Android is more fragmented: Pixel-native and Samsung Galaxy Watch are the most reliable sources. Oura and Whoop integrations (v2.0 scope) will matter more for Android users.
- ML Kit Pose Detection is the Android equivalent of Apple Vision framework for pose estimation. Accuracy is comparable at 30fps on modern devices (Pixel 8+, Samsung S24+). Older mid-range Android hardware is a real concern — the Android launch must define a minimum spec.
- Google Play Billing Library replaces StoreKit 2 on Android.

### 5.3 Third-Party Wearable Integrations

**Oura Ring:**

- Oura API is the cleanest third-party health API for sleep and HRV (RMSSD). Well-documented, stable, requires a user-generated personal access token.
- HRV metric difference: Oura reports RMSSD; Apple HealthKit reports SDNN. These are not directly comparable. Per `docs/TECHNICAL.md §10 question 3`, the approach is per-source baseline tracking and z-score normalization — do not mix raw values.
- Integration path: user authenticates via Oura OAuth in Settings, FORM backend polls Oura API nightly and writes normalized scores to Supabase, mobile app reads from Supabase (not directly from Oura).

**Whoop:**

- Whoop API requires a formal partnership agreement. Timeline: 3–6 months from initial contact to approved access. This should be initiated at Seed close, not at v2.0 ship date.
- Whoop provides strain score (proprietary) and HRV (RMSSD). The partnership agreement will restrict what FORM can do with the data — review terms before committing to features that depend on Whoop-specific fields.

**Garmin:**

- Garmin Health API requires an enterprise contract. Not practical at pre-Series A stage. Deferred to v3.0 or enterprise tier.

### 5.4 Voice-Only Mode (Screenless Training)

A mode where Victor coaches entirely through audio, with the screen off or turned away, for users who do not want to look at their phone during a session.

**What is required before this is viable:**

- A pre-rendered cue library large enough to cover 90%+ of common coaching moments without a live LLM call (live LLM adds 300–700ms which breaks immersion in audio-only mode)
- On-device rep detection reliable enough to trigger cues without visual confirmation from the user
- Wake-word or button activation for user input without looking at the screen
- AVAudioSession configuration that keeps audio active while display is off

**Technical risk:** The cue library must be extensive enough that the experience does not feel robotic. This is a content production challenge as much as an engineering challenge. Scoping the cue library is a prerequisite task for any v2.0 planning.

### 5.5 Adaptive Plan Engine — Real Feedback Loop

In v1.0, Victor adjusts the weekly plan based on HRV, sleep, and RPE inputs but the adjustment logic is heuristic and Sonnet-driven at plan generation time (weekly batch). For v2.0, the goal is a tighter feedback loop:

- Session RPE and completion data feed back into the next 3–5 days of programming, not just the next weekly generation
- Victor proposes a same-day modification if morning HRV is significantly below baseline (triggered on app open, before the workout starts)
- The modification is a suggestion, not an automatic change — user must confirm

**What this requires from the backend:**

- A plan-state representation in Supabase that can be mutated mid-week (not just overwritten weekly)
- A Cloudflare Worker endpoint that accepts RPE + HRV deltas and returns a proposed plan patch
- The mobile app rendering a diff view: «Victor suggests swapping Wednesday's squat session for mobility work — here's why» with Accept / Keep original options

### 5.6 Food Photo Recognition — Hardened

The v1.1 food photo feature uses Sonnet 4.6 Vision as a simple API call. For v2.0:

- Portion size calibration using the phone as a spatial reference (ARKit depth estimates or a known-size reference object in frame)
- Multi-item meal recognition (full plate with mixed foods, not just single-ingredient shots)
- User correction flow that feeds back into personalized recognition preferences (stored in Supabase)

---

## 6. Technical Constraints

This section states the hard technical limits the Founding Engineer must design within. Violating these is not a product decision — these reflect real hardware, API, and safety constraints.

### 6.1 On-Device vs. Cloud — Hard Boundary

| Data / Process | Location | Rationale |
|---|---|---|
| Camera frames | On-device only, never uploaded | Privacy floor: stated in Privacy Policy and verified by App Store review |
| Pose keypoints (raw, real-time) | On-device only during session | Derived from camera; same privacy constraint |
| Audio from voice commands | On-device STT (Apple Speech framework) | Raw audio never transmitted |
| Biometric face data | Never collected | Not applicable to pose estimation used |
| Location | Never collected | Not required for any FORM feature |
| Rep counts and tempo (per set) | Synced after set completes, aggregated | Not streamed frame-by-frame to cloud |
| Session summary | Synced to Supabase after session ends | Requires internet connection; queued if offline |
| HealthKit data (HRV, sleep, heart rate) | Synced to Supabase on request (user-authorized) | Apple HealthKit data is special category under GDPR |
| Chat messages | Transmitted to Cloudflare Worker → Anthropic API | Claude processes; no frames or audio transmitted |
| Plan generation | Cloudflare Worker → Anthropic API → Supabase | Requires internet; generates weekly async |
| TTS audio (Victor's voice) | ElevenLabs API → R2 cache → device | Common cues served from R2 cache, not live ElevenLabs for every cue |

**The «on-device only» claim in marketing copy:**
The privacy copy «camera video never leaves your device» is accurate only if the CV pipeline runs entirely on-device. If any frame, keypoint stream, or audio clip is transmitted to the backend for any reason (including debugging or quality improvement), this claim must be removed from marketing copy and the privacy policy updated. The `clinical-safety` and `compliance-officer` agents must review any proposed exception to this boundary.

### 6.2 Latency SLOs — Mobile Client

These are targets the mobile app must meet. They connect to `docs/OBSERVABILITY.md §2.1`. The Founding Engineer owns the implementation; `devops-lead` owns the server-side components.

| User action | Target latency | What the clock covers |
|---|---|---|
| App cold start → Today screen visible | Under 1.5 seconds | Bundle load, state restore, HealthKit warm-up query |
| Workout start → first camera frame analyzed | Under 800ms | Camera init, Vision framework warm-up |
| Rep detected → counter updated on screen | Under 100ms | On-device, no network |
| Form deviation detected → cue delivered | Under 600ms | Heuristic evaluation + cue library lookup + TTS play start |
| User sends chat message → Victor starts responding (first token visible) | Under 2.5 seconds | Cloudflare Worker receive + Anthropic cache hit + streaming start |
| Voice food log → entry confirmed on screen | Under 1.2 seconds | Apple STT + Haiku parse + Supabase write |
| Plan generation (weekly, async) | Under 12 seconds total | Sonnet 4.6 full context; shown as background task, not blocking UI |

### 6.3 Battery Budget for CV Pipeline

Real-time 30fps pose estimation is the most battery-intensive operation in the app. Measured drain on a reference device (iPhone 14, full battery):

- Camera + Vision framework running at 30fps: approximately 25–40% battery per hour
- This is a hard constraint on session length for CV-enabled workouts

**Required mitigations:**

- Cap CV-enabled session duration at 60 minutes. At 55 minutes, warn the user. At 60 minutes, offer to continue in manual mode or end the session.
- Evaluate whether 15fps is sufficient for coaching quality (hypothesis: saves approximately 30% battery with acceptable quality degradation — validate in beta before v1.1)
- When battery is below 15%, prompt the user to switch to manual mode before the device enters low-power mode automatically
- Do not run the CV pipeline in the background. Camera session must pause when the app is backgrounded and resume only on foreground.

### 6.4 LLM Routing — Mobile Implications

| Victor feature | Model | Requires network | Offline behavior |
|---|---|---|---|
| Real-time coaching cue (common cue library) | Pre-rendered TTS from R2 cache | No (cached on device) | Works offline |
| Real-time coaching cue (novel, live LLM) | Haiku 4.5 | Yes | Fall back to silent or generic cue |
| Chat response | Sonnet 4.6 | Yes | Show offline state, queue when reconnected |
| Morning daily note | Haiku 4.5 | Yes | Show previous note or omit |
| Weekly plan generation | Sonnet 4.6 | Yes | Keep current plan, generate when reconnected |
| Food voice parse | Haiku 4.5 | Yes | Fall back to manual search |

The mobile app must handle offline gracefully for all features. Workout logging, rep counting, and manual meal entry must work offline. LLM-dependent features must degrade with clear messaging, not errors.

### 6.5 CV Pipeline — Honest Capability Claims

This is a constraint on what FORM can truthfully market. The `clinical-safety` agent has veto authority over any claim that overstates CV capability.

**Claims that are supportable at v1.0:**

- «Counts your reps automatically»
- «Detects when your form breaks down on squats, deadlifts, and RDLs»
- «Tells you when you're moving too fast or not deep enough»
- «Works with a phone on a tripod or propped against your bag»

**Claims that are not supportable at v1.0:**

- Any specific accuracy percentage for form feedback (70–85% gross detection is not the same as «94% accuracy»)
- «Catches every rep»
- «Sees what a PT sees»
- «Works in any gym» (lighting and camera position matter significantly)
- «Works with any exercise» (only the listed base lifts have validated heuristics at v1.0)

**Claims gated behind 14-day baseline:**

- Readiness and recovery recommendations require HRV baseline. The app must not show a readiness score or make recovery-based plan adjustments during the first 14 days of HealthKit data collection. The UI must communicate this clearly.

---

## 7. Deferred Items — Explicitly Out of v1

| Item | Deferred to | Reason | DEC ref |
|---|---|---|---|
| Android app | Q1 2027 (M7) | Single platform focus. CV pipeline reliability on iOS must be proven first. | DEC-015 |
| Apple Watch companion app | v2.0 (post-Seed) | Valuable but not core to proving the coaching loop at launch. | TECHNICAL.md §9 |
| Apple TV app | Post-PMF | TV placement is not suitable for CV coaching. | Not logged |
| Social features (leaderboards, friends, community) | Post-PMF, if at all | Triggers negative comparison. Retention-harming pattern per DEC-002. | DEC-002 |
| In-app coach marketplace | Post-PMF | Not core to AI coaching value proposition. | PRODUCT_SPEC.md §1 |
| Video workout library | Post-PMF | Content production cost before PMF is premature. | PRODUCT_SPEC.md §1 |
| AR visualization of form | Post-seed | Over-engineering before CV baseline is validated. | TECHNICAL.md §9 |
| Garmin wearable integration | v3.0+ | Enterprise API contract required. Not practical pre-Series A. | TECHNICAL.md §2 |
| Whoop integration | v2.0 (pending partnership) | Partnership agreement required. Initiate at Seed close. | TECHNICAL.md §2 |
| Oura integration | v2.0 | OAuth integration feasible but HRV normalization work needed first. | TECHNICAL.md §10 |
| Web app | Post-PMF | Mobile-only is the right constraint for launch focus. | PRODUCT_SPEC.md §1 |
| Enterprise mobile features (SSO on mobile, managed device enrollment) | Post-PMF | Enterprise pilot must confirm demand before mobile scope expands. | ENTERPRISE.md |
| Voice-only screenless mode | v2.0 | Requires cue library at sufficient depth and on-device rep detection maturity. | Section 5.4 |
| Russian language UI | Never | DEC-021 | DEC-021 |
| Multi-user / family sharing | Post-PMF | Not relevant to target persona. Adds account complexity. | Not logged |
| Gamification with ranks or badges | Post-PMF | Anti-pattern to retention for serious lifters (DEC-004 target persona). | Not logged |
| Meal planning (AI-generated meal plans, grocery lists) | v2.0 | FORM logs meals and tracks macros in v1. Full meal planning is a scope expansion. | Not logged |
| Integration with gym management software | Post-Series A | B2B channel feature, not consumer. | Not logged |

---

## 8. Integration Touchpoints with This Repo

The mobile app is built in a separate repository, but it connects to backend services and documentation in this repo. These are the active integration contracts. When documents in this repo change in ways that affect these contracts, the mobile repo must be notified (via the `platform-engineer` agent or directly to the Founding Engineer).

### 8.1 API Contract (docs/API.md)

The mobile app is the primary API consumer. Key points:

- Authentication: Apple Sign In token → Supabase session. The mobile app calls `supabase.auth.signInWithIdToken()` with the Apple identity token. The session JWT is then used for all subsequent API calls.
- Sync API: debounced batch upload of aggregated session data after each workout. The mobile app does not stream data to the backend in real time during a workout.
- LLM relay: the mobile app does not call Anthropic or ElevenLabs directly. All LLM and TTS calls are proxied through Cloudflare Workers with rate limiting, key vault management, and prompt caching.
- Realtime: Supabase Realtime is used for Victor chat (streaming chat messages). The mobile app subscribes to a channel per user session.

### 8.2 Data Model (docs/DATA_MODEL.md)

The mobile app reads from and writes to the Supabase schema defined in `docs/DATA_MODEL.md`. Mobile-relevant tables:

- `users`: read/write own row (display name, avatar, role). RLS enforced.
- `user_health_profiles`: read/write own row only. Health goals, restrictions, HRV baseline. Never visible to enterprise admin roles.
- `workouts`: write after each session (`source: cv` or `manual`). `raw_cv_data` JSONB column is encrypted at the Supabase level — the mobile app writes plaintext keypoint summaries; encryption is handled server-side.
- `meal_logs`: write per meal log entry.

Any schema migration that adds, removes, or renames a column used by the mobile app requires coordination. Breaking changes require a versioned API endpoint, not a silent migration.

### 8.3 User Flows (docs/FLOWS.md)

`docs/FLOWS.md` is the authoritative source for all user flows: onboarding, daily ritual, workout session, meals, chat, progress, settings, notifications, and cancel/churn. The mobile implementation must match these flows.

Clinical-safety rules embedded in `docs/FLOWS.md` (ED-screening, age gate, PAR-Q, pain triage, self-harm response) are non-negotiable. The `clinical-safety` agent has veto authority on any change to these flows.

### 8.4 Victor Prompt System (docs/VICTOR_PROMPT_GUIDE.md)

The mobile app does not own Victor's prompt. The prompt architecture (5 layers: role definition, safety constraints, context injection, response format, user message) is defined in `docs/VICTOR_PROMPT_GUIDE.md` and implemented in the Cloudflare Worker layer.

The mobile app's responsibility is:
- Sending the correct context fields (HRV, sleep, streak, ED-flag, tone mode, current exercise, set state) with each LLM relay request
- Rendering streaming responses correctly
- Applying tone preference from user Settings when constructing the relay request payload
- Not modifying Layer 1 (role definition) or Layer 2 (safety constraints) content at the client level

### 8.5 Observability Integration (docs/OBSERVABILITY.md)

The mobile app contributes to the observability system defined in `docs/OBSERVABILITY.md`:

- Sentry iOS SDK: crash reports (with health field stripping per OBSERVABILITY.md §8.2), screen load times
- PostHog iOS SDK: all product events in the event taxonomy defined in OBSERVABILITY.md §3.2
- W3C TraceContext: `traceparent` header on all outbound requests so mobile-originated traces correlate with Cloudflare Worker spans
- App version label on all Sentry and PostHog events: required for the Mobile Release Health dashboard (OBSERVABILITY.md §7.7)

Note: OBSERVABILITY.md references React Native/Expo in its scope statement — this is an inconsistency. The mobile stack is Swift/SwiftUI. The SLOs and event names are valid regardless; the SDK references (Sentry React Native, EAS Update) in OBSERVABILITY.md should be updated by `devops-lead` to reflect Sentry iOS SDK once the Founding Engineer confirms the mobile repo setup.

### 8.6 Analytics Events — Required at v1.0

These PostHog events must be instrumented before TestFlight opens.

| Event name | Triggered when |
|---|---|
| `onboarding_started` | User opens app for the first time |
| `onboarding_step_completed` | Each onboarding screen completed (include step number property) |
| `onboarding_completed` | Final onboarding screen confirmed |
| `ed_flag_raised` | ED-screening triggers soft mode |
| `healthkit_authorized` | User grants HealthKit permission |
| `healthkit_denied` | User denies HealthKit permission |
| `workout_started` | User taps Start on workout card |
| `camera_mode_enabled` | User uses camera mode (vs manual) |
| `coaching_cue_delivered` | Victor delivers a real-time form cue (include cue type, lift name) |
| `set_completed` | User completes a set (include rpe, rep count, exercise name) |
| `workout_completed` | Workout session ends (include `completed: true/false`, duration, lift count) |
| `session_abandoned` | User exits mid-workout (include time elapsed, sets completed) |
| `meal_logged` | Any meal entry saved (include method: voice/photo/manual) |
| `victor_message_sent` | User sends a message in Coach tab |
| `victor_response_received` | Victor's response is fully rendered (include latency_ms) |
| `coaching_feedback_negative` | User taps thumbs-down on a Victor response |
| `trial_started` | Free trial begins |
| `subscription_activated` | User converts from trial to paid |
| `subscription_cancelled` | User cancels subscription |
| `retention_offer_shown` | Single retention offer presented at cancel |
| `retention_offer_accepted` | User accepts retention discount |
| `retention_offer_declined` | User declines retention offer |
| `data_export_requested` | User taps full data export |
| `account_deletion_requested` | User initiates account deletion |

No PII in event properties. User identifiers in PostHog are pseudonymous (`distinct_id` = Supabase UUID, not email). Health values (HRV scores, weight, calorie counts) must not appear in PostHog event properties.

### 8.7 Enterprise Tier — Mobile Touchpoints

The enterprise tier is defined in `docs/ENTERPRISE.md` and `docs/SSO_SCIM_IMPLEMENTATION.md`. For v1.0, enterprise features on mobile are minimal:

- Consumer mobile auth: Apple Sign In → Supabase. This is fully independent from the enterprise SSO flow (SAML/OIDC via WorkOS/Auth0), which is for admin dashboard access, not the mobile app.
- The RLS policies in `docs/DATA_MODEL.md §3.3–3.4` ensure tenant admins cannot read individual workout or health data from the mobile app. Enforced at the database layer, not the mobile layer.
- If a future enterprise plan includes managed mobile access (SSO on mobile, corporate-issued accounts), this requires a separate design — the current mobile auth architecture does not support SAML-initiated login on iOS without significant additions.

---

## 9. Open Questions for the Founding Engineer

These questions cannot be resolved by cloud agents. They require implementation experience, device testing, or product judgment from the Founding Engineer.

| # | Question | Why it matters | Target resolution |
|---|---|---|---|
| Q-M-001 | Is 15fps sufficient for coaching quality, or does the CV pipeline require 30fps? | Battery drain is approximately 30% lower at 15fps. If quality holds, 15fps should be the default for all sessions. | TestFlight beta: measure rep count accuracy and cue timing at 15fps vs. 30fps. Decision before v1.1 ship. |
| Q-M-002 | SwiftData vs. Core Data for local workout persistence? | SwiftData (iOS 17+) is simpler but less mature for complex query patterns. Core Data is battle-tested but verbose. The choice affects offline sync and data migration strategy. | Founding Engineer decision at project setup. Not reversible without significant migration work. |
| Q-M-003 | What is the minimum supported iPhone model for the CV pipeline? | Apple Vision framework performance varies significantly between A12 (iPhone XS) and A15 (iPhone 13). A12 devices may struggle at 30fps. | Benchmark pose estimation at 15fps and 30fps on iPhone XS (A12), iPhone 11 (A13), iPhone 13 (A15). Set minimum spec before the App Store listing is written. |
| Q-M-004 | How should rep counting handle interrupted sets (pause mid-set, re-rack, continue)? | Re-racking and continuing a set looks identical to finishing a set. False set completions are worse UX than missed reps. | Define heuristic during M2 implementation. May require gesture or tap-to-resume rather than automatic detection. |
| Q-M-005 | What is the UX for the GDPR EU consent flow before HealthKit authorization? | GDPR Art. 9 requires explicit consent for health data processing before HealthKit is requested. Exact screen design and copy are not yet defined. | Design before v1.1 App Store submission. Must be reviewed by `clinical-safety` and `compliance-officer`. |
| Q-M-006 | ElevenLabs streaming TTS vs. pre-rendered cue library — where is the threshold? | If the pre-rendered library is too small, every unusual situation triggers a 300–700ms live TTS call, breaking coaching immersion. | Define cue library size during M2 (CV beta sprint). Target: pre-rendered cues cover 85%+ of cue triggers in a typical 45-minute session. Validate in beta. |
| Q-M-007 | How does the app handle multiple HealthKit data sources? (Apple Watch + iPhone + third-party app) | HealthKit can have multiple HRV sources. FORM must select the most reliable source per user, not average across sources. | Founding Engineer decision during HealthKit integration (M1). Document the selection rule in this file when resolved. |
| Q-M-008 | What is the interaction model for Victor during an active set? | Victor must not interrupt a set in progress. The interaction model for the user responding to Victor during a set is not designed. Should cues interrupt music? Is there a visual-only mode? | User research during TestFlight beta. Design decision before v1.1. |
| Q-M-009 | Live Activity / Dynamic Island for workout sessions — v1.0 or v1.1? | WidgetKit Live Activity is a high-demand feature for a workout app but adds scope to v1.0. Currently listed as P1. | Founding Engineer scope decision at M1. If it adds more than 2 weeks of work, defer to v1.1. |
| Q-M-010 | What is the data schema for the on-device local model, and how does it map to the Supabase schema in DATA_MODEL.md? | The local model must support offline-first operation: create a workout session without network, complete it, sync when reconnected. Field mapping between local and remote schemas needs explicit definition to avoid sync conflicts. | Founding Engineer decision at M1, documented in the mobile repo. Divergence from `docs/DATA_MODEL.md` must be flagged to `platform-engineer` for API alignment. |
| Q-M-011 | StoreKit 2 directly vs. RevenueCat wrapper? | StoreKit 2 is leaner; RevenueCat adds analytics and multi-platform subscription management. The existing PRODUCT_SPEC says StoreKit 2 directly — but the brief mentioned RevenueCat. This is unresolved. | Founder + Founding Engineer decision before M1. Log as DEC entry when final. |

---

## Appendix A — Milestone Map Reference

From `docs/TECHNICAL.md §6` — mobile milestones only:

| Milestone | Period | Mobile gate |
|---|---|---|
| M0 | Q3 2026 start | Xcode project setup, design system tokens, Hello-World CV running on device |
| M1 | ~4 weeks | Today screen + HealthKit integration + onboarding (no CV) |
| M2 | ~4 weeks | Workout screen with CV beta (squat, RDL only) |
| M3 | ~4 weeks | TestFlight beta open: 3 lifts, meals (text+voice), Victor chat, progress screen |
| M4 | Q4 2026 | Chat + nutrition polish + plan engine V2 |
| M5 | ~4 weeks | 8-lift CV pipeline complete |
| M6 | ~4 weeks | App Store submission and public release |
| M7 | Q1 2027 | Android development begins (separate scope) |

---

## Appendix B — Persona Reference

The primary target user is Dmytro (defined in `docs/PERSONAS.md §01`): male, 25–40, 1–3+ years of strength training, self-coached, has a wearable (Apple Watch), is not a beginner, does not need motivation, needs honest feedback on technique and evidence-based programming.

Design decisions that conflict with Dmytro's profile (gamification, excessive notifications, cheerleader tone, beginner-level explanations of basic movements) are a product defect, not a preference. The FORM mobile experience is calibrated for the serious recreational lifter, not the general fitness audience.

---

**v0.1 · May 2026 · Owner: platform-engineer + product-strategist · Safety gate: clinical-safety**
**Review: before each milestone boundary (M0 through M6) or on significant architecture change.**
**Mobile code does not go in this repo. New mobile ideas and scope changes go in this document first.**
