# FORM · Analytics Setup v0.1

> Instrumentation guide for the Founding Engineer.
> Companion to `PRODUCT_SPEC.md` and `METRICS.md`.
> Owner: `platform-engineer` + `growth-lead`.
> Read `METRICS.md` first — this doc is the "how to instrument" for everything defined there.

---

## 0 · Decision: PostHog (self-hosted on Cloudflare)

| Option | Why rejected / why chosen |
|---|---|
| **PostHog Cloud** | ✓ Chosen for MVP — fast setup, free tier covers beta volume, open source means EU-compliant without custom DPA |
| Mixpanel | Easier funnels UI but proprietary, more expensive at scale, no self-hosting option |
| Amplitude | Strong but GDPR posture weaker on cloud tier, expensive |
| Firebase/GA4 | Google data residency issues for UA market, event schema too rigid |
| DIY (Supabase events table) | Zero analytics UI, every insight requires SQL, kills iteration speed |

**MVP: PostHog Cloud EU region** (Frankfurt). Switch to self-hosted on Cloudflare Workers if > 1M events/month or if Supabase consolidation makes sense. Migration is one endpoint swap.

PostHog project: `form-prod` (create `form-beta` as separate project for TestFlight cohort).

---

## 1 · iOS SDK Setup

### Installation (Swift Package Manager)

```swift
// Package.swift or via Xcode File → Add Packages
// https://github.com/PostHog/posthog-ios

dependencies: [
    .package(url: "https://github.com/PostHog/posthog-ios.git", from: "3.0.0")
]
```

### Initialization (AppDelegate / App struct)

```swift
import PostHog

@main
struct FORMApp: App {
    init() {
        let config = PostHogConfig(
            apiKey: ProcessInfo.processInfo.environment["POSTHOG_API_KEY"] ?? "",
            host: "https://eu.posthog.com"
        )
        config.captureApplicationLifecycleEvents = false  // we fire manually
        config.captureScreenViews = false                  // we fire manually
        config.flushAt = 20
        config.flushInterval = 30
        PostHogSDK.shared.setup(config)
    }
}
```

**NEVER** hardcode API key. Store in:
- Local: `.env.local` (gitignored)
- CI: GitHub Actions secret `POSTHOG_API_KEY`
- Production: Cloudflare Worker env var (injected at build time via xcconfig)

### Identify on login

```swift
// After successful Apple Sign In — call once per session
PostHogSDK.shared.identify(
    distinctId: user.pseudonymousId,     // UUID, NOT email
    userProperties: [
        "persona_segment": user.personaTag,         // "dmitro" | "olena" | "artur" | "bohdan"
        "onboarding_ed_path": user.edScreeningPath, // "standard" | "soft"
        "subscription_status": "trial",             // trial | paid_monthly | paid_annual | churned
        "app_version": AppVersion.current,          // "0.9.0"
        "locale": Locale.current.identifier,        // "uk_UA" | "en_GB"
    ]
)
```

### Reset on logout

```swift
PostHogSDK.shared.reset()
// New anonymous ID assigned automatically for next user
```

---

## 2 · Pseudonymous ID Strategy

PostHog's `distinctId` must NEVER be email, name, or any PII.

```swift
// UserDefaults-persisted UUID, generated on first launch
struct AnalyticsID {
    static var current: String {
        let key = "analytics_pseudonymous_id"
        if let existing = UserDefaults.standard.string(forKey: key) {
            return existing
        }
        let new = UUID().uuidString
        UserDefaults.standard.set(new, forKey: key)
        return new
    }

    static func reset() {
        UserDefaults.standard.removeObject(forKey: "analytics_pseudonymous_id")
    }
}
```

When user enables analytics opt-out in Settings → Privacy:
- Stop firing events immediately (`PostHogSDK.shared.optOut()`)
- Do NOT delete past events — explain this to user in Privacy settings UI
- `AnalyticsID.reset()` only on full account deletion

---

## 3 · Event Taxonomy

### Naming convention

`snake_case`. Pattern: `<noun>_<verb>` or `<screen>_<action>`.

Examples: `workout_started`, `cv_rep_counted`, `paywall_viewed`.

**No free-text payloads.** All properties are enums or numeric. No user-typed strings, no exercise names as freeform text.

---

### 3.1 · Onboarding events

| Event | Properties | Notes |
|---|---|---|
| `app_opened` | `session_id`, `is_first_open: Bool` | Every cold launch |
| `onboarding_started` | `source: "organic"\|"push"\|"share"` | Screen 1 reached |
| `onboarding_step_completed` | `step: 1..8`, `duration_sec: Int` | Each screen |
| `ed_screening_shown` | — | SCOFF-light gate — non-skippable |
| `ed_screening_completed` | `path: "standard"\|"soft"` | soft = Meals tab disabled |
| `health_permission_requested` | `permission: "healthkit"\|"notifications"\|"camera"` | Each prompt |
| `health_permission_granted` | `permission: String` | |
| `health_permission_denied` | `permission: String` | |
| `onboarding_completed` | `total_duration_sec: Int`, `ed_path: String` | All 8 screens done |
| `onboarding_abandoned` | `last_step: Int`, `duration_sec: Int` | App backgrounded mid-flow |

---

### 3.2 · Authentication events

| Event | Properties | Notes |
|---|---|---|
| `signup_started` | `method: "apple"` | Only Apple in v1 |
| `signup_completed` | `method: "apple"` | |
| `signup_failed` | `error_code: String` | No free-text error |
| `login_completed` | `method: "apple"` | |
| `session_started` | `session_id: UUID`, `app_version: String` | Each foreground |
| `account_deleted` | — | Cascades full data wipe |

---

### 3.3 · Today tab events

| Event | Properties | Notes |
|---|---|---|
| `today_viewed` | `has_hrv_data: Bool`, `plan_type: "standard"\|"high_readiness"\|"recovery"` | Each view |
| `hrv_check_opened` | `hrv_delta_percent: Int` | Rounded to nearest 5% |
| `plan_adjustment_shown` | `adjustment_type: "deload"\|"increase"\|"maintain"` | Victor adapts plan |
| `plan_adjustment_accepted` | `adjustment_type: String` | User taps Accept |
| `plan_adjustment_dismissed` | `adjustment_type: String` | User dismisses |

---

### 3.4 · Workout events — W-ACSU source

These events are the foundation of the North-Star Metric.

| Event | Properties | Notes |
|---|---|---|
| `workout_started` | `workout_id: UUID`, `source: "plan"\|"manual"`, `lift_count: Int` | |
| `exercise_started` | `workout_id`, `lift_type: LiftEnum`, `set_number: Int` | |
| `set_logged` | `workout_id`, `lift_type`, `reps: Int`, `weight_kg: Float`, `input_method: "manual"\|"cv"` | |
| `cv_session_started` | `workout_id`, `lift_type: LiftEnum` | Camera activated |
| `cv_rep_counted` | `workout_id`, `lift_type`, `rep_number: Int`, `confidence: Float` | Rounded to 2dp |
| `cv_form_cue_shown` | `workout_id`, `lift_type`, `cue_type: CueEnum`, `severity: "info"\|"warn"\|"stop"` | |
| `cv_form_cue_dismissed` | `cue_type`, `time_to_dismiss_sec: Int` | |
| `victor_coaching_cue_shown` | `workout_id`, `cue_source: "cv"\|"hrv"\|"plan"`, `tone: ToneEnum` | **W-ACSU trigger** |
| `victor_coaching_cue_dismissed` | `cue_source`, `response: "acted"\|"dismissed"\|"ignored"` | |
| `workout_paused` | `workout_id`, `elapsed_min: Int` | |
| `workout_resumed` | `workout_id` | |
| `workout_completed` | `workout_id`, `duration_min: Int`, `sets_logged: Int`, `cv_reps_counted: Int`, `victor_cues_shown: Int`, `is_coached_session: Bool` | |
| `workout_abandoned` | `workout_id`, `elapsed_min: Int`, `sets_logged: Int` | |

**Coached session rule** (W-ACSU component): A workout is a "coached session" if `victor_cues_shown ≥ 1` OR `cv_rep_counted ≥ 1`. Computed server-side from events, not stored as a separate event.

```
LiftEnum = "squat" | "bench_press" | "deadlift" | "ohp" | "barbell_row" |
           "rdl" | "pull_up" | "hip_thrust" | "other"
```

```
CueEnum = "depth_shallow" | "knees_caving" | "back_rounding" | "bar_path_drift" |
          "lockout_incomplete" | "hip_shift" | "tempo_too_fast" | "setup_width" |
          "breathing_reminder" | "rest_complete"
```

```
ToneEnum = "smart" | "mindful" | "cold" | "coach"
```

---

### 3.5 · Victor chat events

| Event | Properties | Notes |
|---|---|---|
| `chat_opened` | `context: "workout"\|"today"\|"standalone"` | |
| `message_sent_user` | `session_id`, `message_length_chars: Int` | No message content |
| `message_received_victor` | `session_id`, `response_latency_ms: Int`, `model: "haiku"\|"sonnet"` | |
| `voice_response_played` | `duration_sec: Int` | ElevenLabs TTS |
| `voice_response_skipped` | `seconds_played: Float` | |
| `chat_session_ended` | `message_count: Int`, `duration_sec: Int` | |

---

### 3.6 · Meals tab events (standard path only — ed_path = "standard")

| Event | Properties | Notes |
|---|---|---|
| `meals_tab_opened` | — | |
| `meal_log_started` | `input_method: "voice"\|"text"\|"photo"\|"barcode"` | |
| `meal_log_completed` | `input_method`, `parse_latency_ms: Int`, `item_count: Int` | |
| `meal_log_corrected` | `field: "calories"\|"protein"\|"carbs"\|"fats"\|"quantity"` | User edited auto-parse |
| `macro_target_viewed` | `day_of_week: 0..6` | |

---

### 3.7 · Subscription / paywall events

| Event | Properties | Notes |
|---|---|---|
| `paywall_viewed` | `trigger: "trial_end"\|"feature_gate"\|"settings"\|"organic"`, `trial_days_remaining: Int` | |
| `paywall_plan_selected` | `plan: "monthly"\|"annual"`, `price_uah: Int \| price_eur: Int` | Local currency |
| `subscription_started` | `plan: String`, `revenue_usd_cents: Int` | After StoreKit confirm |
| `subscription_renewal` | `plan: String` | |
| `subscription_cancelled` | `plan: String`, `days_active: Int`, `cancellation_reason: CancelEnum` | |
| `trial_started` | `plan: String`, `trial_days: Int` | |
| `trial_converted` | `plan: String`, `trial_duration_days: Int` | |
| `trial_expired_no_convert` | `plan: String` | Churned without pay |

```
CancelEnum = "too_expensive" | "not_using_enough" | "missing_feature" |
             "technical_issue" | "found_alternative" | "other"
```

---

### 3.8 · Settings + privacy events

| Event | Properties | Notes |
|---|---|---|
| `settings_opened` | — | |
| `notification_preference_changed` | `type: "workout_reminder"\|"rest_alert"\|"weekly_review"`, `enabled: Bool` | |
| `analytics_opt_out_toggled` | `new_state: Bool` | Fire this BEFORE disabling if opting out |
| `data_export_requested` | — | |
| `subscription_cancel_started` | — | 3-click cancel flow start |
| `subscription_cancel_completed` | — | Cancellation confirmed |
| `subscription_cancel_abandoned` | `step: 1..3` | Left cancel flow |

---

### 3.9 · Error + quality events

| Event | Properties | Notes |
|---|---|---|
| `api_error` | `endpoint: EndpointEnum`, `status_code: Int`, `retry_count: Int` | |
| `cv_pipeline_error` | `error_type: "camera_denied"\|"pose_lost"\|"timeout"\|"model_load"` | |
| `llm_timeout` | `use_case: UseCaseEnum`, `elapsed_ms: Int` | |
| `crash_recovered` | `screen: ScreenEnum` | After graceful recovery |

```
EndpointEnum = "auth" | "sync" | "plan_generate" | "chat" | "food_parse" | "voice"
UseCaseEnum  = "cv_cue" | "chat" | "plan_generate" | "food_parse" | "weekly_review"
ScreenEnum   = "today" | "workout" | "chat" | "meals" | "settings" | "onboarding"
```

---

## 4 · W-ACSU Computation

The North-Star Metric is **not** a raw event — it's computed server-side from the event stream.

### Definition

```
W-ACSU = (coached_sessions in last 7 days) / (active_users in last 7 days)
```

A **coached session** = `workout_completed` WHERE `is_coached_session = true`.

A **coached session** is marked true when a workout has:
- `victor_coaching_cue_shown` count ≥ 1, OR
- `cv_rep_counted` count ≥ 1

### SQL (Supabase / PostHog SQL runner)

```sql
-- W-ACSU for a given week ending :week_end
WITH active_users AS (
    SELECT DISTINCT distinct_id
    FROM events
    WHERE event = 'workout_completed'
      AND timestamp BETWEEN :week_start AND :week_end
),
coached AS (
    SELECT DISTINCT distinct_id
    FROM events
    WHERE event = 'workout_completed'
      AND (properties->>'is_coached_session')::boolean = true
      AND timestamp BETWEEN :week_start AND :week_end
)
SELECT
    COUNT(DISTINCT c.distinct_id)::float / NULLIF(COUNT(DISTINCT a.distinct_id), 0) AS w_acsu
FROM active_users a
LEFT JOIN coached c USING (distinct_id);
```

### PostHog formula

Create a PostHog **Insight → Formula**:
- A: `Unique users who did workout_completed where is_coached_session = true` (last 7 days)
- B: `Unique users who did workout_completed` (last 7 days)
- Formula: `A / B`

---

## 5 · AARRR Funnel Setup in PostHog

Create these funnels in PostHog **Insights → Funnels**:

### Acquisition funnel

```
app_opened (is_first_open = true)
  → onboarding_started
  → onboarding_completed
  → workout_started (first)
```
Conversion window: 72 hours.

### Activation funnel

```
onboarding_completed
  → workout_started (first coached session)
  → victor_coaching_cue_shown
```
"Activated" = reached step 3. Conversion window: 7 days.

### Trial-to-paid funnel

```
trial_started
  → paywall_viewed
  → paywall_plan_selected
  → subscription_started
```
Conversion window: trial duration.

### Paywall funnel

```
paywall_viewed
  → paywall_plan_selected
  → subscription_started
```
Conversion window: 10 minutes.

---

## 6 · User Properties (Super Properties)

Set once on `identify`, update on change. These allow segment breakdowns across all insights.

| Property | Values | Updated when |
|---|---|---|
| `persona_segment` | `"dmitro"\|"olena"\|"artur"\|"bohdan"` | Onboarding completion |
| `subscription_status` | `"trial"\|"paid_monthly"\|"paid_annual"\|"churned"\|"never_subscribed"` | Subscription event |
| `ed_path` | `"standard"\|"soft"` | ED screening |
| `healthkit_connected` | `Bool` | Permission grant/deny |
| `locale` | `"uk_UA"\|"en_GB"\|"en_US"\|"de_DE"\|"pl_PL"` | App locale |
| `app_version` | semver string | App launch |
| `device_model` | `"iPhone 15"\|"iPhone 16"` etc | App launch |
| `days_since_install` | `Int` | Daily, compute server-side |
| `total_coached_sessions` | `Int` | After each workout_completed |
| `cv_ever_used` | `Bool` | After first cv_session_started |

---

## 7 · PostHog Dashboard: "FORM Core" (create on Day 1)

Create these panels in the **FORM Core** dashboard:

| Panel | Type | Metric | Breakdown |
|---|---|---|---|
| W-ACSU (7-day rolling) | Line chart | Formula A/B | None |
| Daily coached sessions | Bar chart | `workout_completed` where `is_coached_session=true` | — |
| Onboarding funnel | Funnel | 4-step acquisition funnel | `locale` |
| Trial conversion rate | Conversion | `trial_started → subscription_started` | `plan` |
| D7 retention | Retention | `app_opened` retained in week 1 | `persona_segment` |
| D30 retention | Retention | `app_opened` retained in month 1 | `subscription_status` |
| CV usage rate | Pie | `cv_session_started` / `workout_started` | — |
| Victor cue response | Bar | `cue_dismissed.response` breakdown | `cue_source` |
| Paywall views vs converts | Funnel | `paywall_viewed → subscription_started` | `trigger` |
| API errors (24h) | Number | `api_error` count | `endpoint` |
| LLM timeout rate | Line | `llm_timeout` / all LLM events | `use_case` |

---

## 8 · A/B Test Setup

PostHog Feature Flags drive all A/B tests.

### Creating a flag

```swift
// Feature flag check — non-blocking, cached
if PostHogSDK.shared.isFeatureEnabled("victor_tone_default") {
    // Variant B: mindful tone default
    defaultTone = .mindful
} else {
    // Control: smart-direct tone
    defaultTone = .smart
}
```

### Current planned tests (from METRICS.md §8)

| Test | Flag key | Control | Variant | Primary metric |
|---|---|---|---|---|
| Victor default tone | `victor_tone_default` | `smart` | `mindful` | D30 retention |
| Trial length | `trial_duration_days` | 7 | 14 | trial_converted rate |
| Paywall timing | `paywall_trigger` | `trial_end` | `workout_3` | subscription_started |
| Onboarding length | `onboarding_steps` | 8 | 5 | `onboarding_completed` rate |

### Minimum sample per cell: 500 activated users before reading results.

Always log which variant a user sees:

```swift
PostHogSDK.shared.capture(
    "experiment_assigned",
    properties: [
        "flag": "victor_tone_default",
        "variant": PostHogSDK.shared.getFeatureFlag("victor_tone_default") ?? "control"
    ]
)
```

---

## 9 · Privacy Implementation

Matches `METRICS.md §9` and `SECURITY.md` standards.

### What we never send

- Email, name, phone, or any PII in event properties
- Free-text user input (chat messages, food descriptions)
- Exact geolocation (country only via IP truncation)
- Photos or media

### IP truncation

PostHog EU cloud truncates IP by default. Verify in **PostHog → Project Settings → Privacy** that "Mask IPs" is enabled.

### Data retention

- PostHog: set to 13 months maximum in **Project Settings → Data Management**
- Daily aggregation job in Supabase deletes raw event rows older than 13 months

### Opt-out implementation

```swift
// Settings → Privacy → Analytics toggle
func setAnalyticsOptOut(_ optOut: Bool) {
    // Fire the change event BEFORE disabling if opting out
    if optOut {
        PostHogSDK.shared.capture("analytics_opt_out_toggled", properties: ["new_state": false])
    }

    if optOut {
        PostHogSDK.shared.optOut()
    } else {
        PostHogSDK.shared.optIn()
        PostHogSDK.shared.capture("analytics_opt_out_toggled", properties: ["new_state": true])
    }

    UserDefaults.standard.set(optOut, forKey: "analytics_opted_out")
}
```

### App Store Privacy Nutrition Label requirements

When submitting, declare:
- **Usage Data**: App functionality (crash logs, performance)
- **Identifiers**: Device ID (not linked to user)
- **Diagnostics**: Crash data (not linked to user)
- Do NOT declare: contacts, location, browsing history, health (HealthKit data stays on device)

---

## 10 · Debug Mode + QA

### Local development

```swift
#if DEBUG
let config = PostHogConfig(apiKey: "phc_test_key", host: "https://eu.posthog.com")
config.debug = true  // logs all events to console
PostHogSDK.shared.setup(config)
#endif
```

Use PostHog's **Activity** tab to verify events are arriving with correct properties during development.

### QA checklist per sprint

Before each TestFlight build:

- [ ] `app_opened` fires on cold launch
- [ ] `onboarding_step_completed` fires for all 8 steps
- [ ] `ed_screening_completed` fires with correct `path` value
- [ ] `workout_completed` has `is_coached_session: true` when Victor cues shown
- [ ] `paywall_viewed` fires on trial expiry
- [ ] Analytics opt-out disables all subsequent events
- [ ] No PII visible in PostHog Activity stream
- [ ] `victor_coaching_cue_shown` fires for CV cues and plan adjustments
- [ ] `subscription_started` fires after StoreKit purchase confirm

### Verifying W-ACSU calculation

After Sprint 4 (CV live):
1. Do a coached workout in simulator
2. Check `workout_completed.is_coached_session = true` in PostHog Activity
3. Run the W-ACSU SQL query against the test project
4. Confirm result ≥ 1.0 for single user / single session

---

## 11 · Environment Separation

| Environment | PostHog Project | API Key source |
|---|---|---|
| Local dev | `form-dev` | `.env.local` |
| TestFlight beta | `form-beta` | GitHub Actions secret |
| Production | `form-prod` | Cloudflare Worker env |

Never mix beta and production event streams. Beta cohort analysis should be done in `form-beta` project only.

---

## 12 · Cloudflare Worker: Analytics Proxy (optional, post-beta)

If we need to prevent ad blockers from blocking PostHog (impacts install attribution):

```javascript
// workers/analytics-proxy.js
export default {
    async fetch(request, env) {
        const url = new URL(request.url)
        url.hostname = 'eu.posthog.com'
        return fetch(new Request(url, request))
    }
}
```

Route: `analytics.form.coach/*` → PostHog EU. Configured in `wrangler.toml`. Not needed for beta — add if attribution gap > 15% post-launch.

---

## Appendix · Full Event List (alphabetical)

```
account_deleted
analytics_opt_out_toggled
api_error
app_opened
chat_opened
chat_session_ended
crash_recovered
cv_form_cue_dismissed
cv_form_cue_shown
cv_pipeline_error
cv_rep_counted
cv_session_started
ed_screening_completed
ed_screening_shown
exercise_started
experiment_assigned
health_permission_denied
health_permission_granted
health_permission_requested
hrv_check_opened
llm_timeout
login_completed
macro_target_viewed
meal_log_completed
meal_log_corrected
meal_log_started
meals_tab_opened
message_received_victor
message_sent_user
notification_preference_changed
onboarding_abandoned
onboarding_completed
onboarding_started
onboarding_step_completed
paywall_plan_selected
paywall_viewed
plan_adjustment_accepted
plan_adjustment_dismissed
plan_adjustment_shown
session_started
set_logged
settings_opened
signup_completed
signup_failed
signup_started
subscription_cancel_abandoned
subscription_cancel_completed
subscription_cancel_started
subscription_cancelled
subscription_renewal
subscription_started
today_viewed
trial_converted
trial_expired_no_convert
trial_started
victor_coaching_cue_dismissed
victor_coaching_cue_shown
voice_response_played
voice_response_skipped
workout_abandoned
workout_completed
workout_paused
workout_resumed
workout_started
```

Total: **65 events** · No event fires PII · All properties are typed enums or numeric.
