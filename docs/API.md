# FORM · API & Data Schema v0.1

> Дизайн API і структури даних. Pre-production. Готується до Founding Engineer.
> Власник: `platform-engineer`. Updates after first iOS spike.

---

## 1 · Архітектура нагадування

```
[iOS / Android App]  ←─→  [Cloudflare Worker · Edge]  ←─→  [Supabase · DB]
                                  │                              │
                                  ├─→  [Anthropic API · LLM]     │
                                  ├─→  [ElevenLabs · TTS]        │
                                  └─→  [Sentry · errors]         │
                                                                  │
                                                  [HealthKit / HC] (on device, sync only)
```

**Принципи:**
- Mobile-fat: 80% логіки на клієнті
- Server-thin: edge для auth, sync, LLM proxy
- All sensitive operations server-side validated
- Idempotent endpoints де можливо

---

## 2 · Auth

### Apple Sign In (primary)

```
POST /v1/auth/apple
Body: { "identity_token": "...", "authorization_code": "..." }
Response 200: { "session_token": "...", "user_id": "uuid", "is_new": true }
Response 401: { "error": "INVALID_TOKEN" }
```

### Token refresh

```
POST /v1/auth/refresh
Header: Authorization: Bearer <session_token>
Response 200: { "session_token": "new...", "expires_at": "..." }
```

### Sign out

```
POST /v1/auth/signout
Header: Authorization: Bearer <session_token>
Response 200: {}
```

### Delete account

```
POST /v1/auth/delete
Header: Authorization: Bearer <session_token>
Body: { "confirm": "DELETE", "reason": "optional text" }
Response 200: { "deletion_scheduled_at": "2026-06-13T00:00:00Z" }
```

**Note:** 30-day grace period для recovery. After grace — full hard delete.

---

## 3 · User profile

### Get profile

```
GET /v1/me
Header: Authorization: Bearer <session_token>
Response 200: {
  "user_id": "uuid",
  "created_at": "2026-05-14T...",
  "display_name": "Дмитро",
  "age": 28,
  "biological_sex": "male" | "female" | "prefer_not_to_say",
  "country": "UA",
  "training_age_years": 3,
  "goals": ["strength", "body_recomp"],
  "voice_persona": "smart_direct" | "mindful_friend" | "cold_professor" | "national_coach",
  "soft_mode_enabled": false,
  "subscription": {
    "status": "trial" | "active" | "cancelled" | "expired",
    "plan": "monthly" | "annual",
    "renews_at": "2026-06-14T...",
    "trial_ends_at": "2026-05-28T..."
  }
}
```

### Update profile

```
PATCH /v1/me
Body: { "display_name": "...", "voice_persona": "..." }
Response 200: {updated profile}
```

---

## 4 · Health data (read-mostly, synced from HealthKit)

### Upload health snapshot

```
POST /v1/health/snapshot
Body: {
  "timestamp": "2026-05-14T07:00:00Z",
  "source": "apple_health" | "manual" | "whoop" | "oura",
  "hrv_rmssd_ms": 68,
  "hrv_sdnn_ms": null,
  "resting_heart_rate_bpm": 54,
  "sleep": {
    "duration_minutes": 432,
    "deep_minutes": 72,
    "rem_minutes": 105,
    "efficiency_percent": 92
  },
  "body_temp_c": 36.5,
  "steps_yesterday": 4210
}
Response 200: { "snapshot_id": "uuid" }
```

### Get HRV trend

```
GET /v1/health/hrv?days=30
Response 200: {
  "baseline_rmssd_ms": 64,
  "datapoints": [
    {"date": "2026-04-14", "rmssd_ms": 62},
    {"date": "2026-04-15", "rmssd_ms": 65},
    ...
  ]
}
```

---

## 5 · Workouts

### Get today's program

```
GET /v1/program/today
Response 200: {
  "program_id": "uuid",
  "program_name": "Strength Block · Week 1",
  "session": {
    "session_id": "uuid",
    "name": "Lower A · Squat-focus",
    "scheduled_at": "2026-05-14T18:30:00Z",
    "estimated_duration_minutes": 62,
    "exercises": [
      {
        "exercise_id": "back_squat",
        "name": "Back squat",
        "sets": 4,
        "reps_target": 6,
        "rpe_target": 7.5,
        "weight_kg_suggested": 92.5,
        "tempo": "2-1-X-1",
        "rest_seconds": 180,
        "cv_coaching_enabled": true,
        "cv_metrics": ["back_angle", "depth", "knee_valgus"]
      },
      ...
    ]
  }
}
```

### Start workout session

```
POST /v1/workout/start
Body: { "session_id": "uuid" }
Response 200: { "active_workout_id": "uuid" }
```

### Log set

```
POST /v1/workout/{active_workout_id}/set
Body: {
  "exercise_id": "back_squat",
  "set_number": 3,
  "reps_completed": 6,
  "weight_kg": 92.5,
  "rpe_actual": 8,
  "tempo_actual": [2.1, 1.0, 0.8, 1.0],
  "cv_data": {
    "back_angle_min_degrees": 38,
    "depth_consistency_score": 0.91,
    "knee_valgus_detected": false,
    "bar_path_deviation_mm": 28
  }
}
Response 200: { "set_logged": true, "cue_text": "Лікті ближче." }
```

### Complete workout

```
POST /v1/workout/{active_workout_id}/complete
Body: {
  "completed_at": "2026-05-14T19:32:00Z",
  "perceived_exertion_overall": 8,
  "feeling_rating": 4
}
Response 200: {
  "workout_summary_id": "uuid",
  "highlights": ["Squat top set 92.5kg @ RPE 8 — на 2.5kg вище last week"],
  "next_session_in_days": 1
}
```

---

## 6 · Nutrition

### Log food (text)

```
POST /v1/nutrition/log
Body: {
  "logged_at": "2026-05-14T07:30:00Z",
  "meal_type": "breakfast" | "lunch" | "dinner" | "snack",
  "input_type": "voice" | "text" | "barcode" | "photo",
  "input_text": "Овес 80г, молоко 250 мл, голубики 100г, протеїн 30г",
  "items": [  // optional pre-parsed
    {
      "name": "Овес",
      "grams": 80,
      "calories": 304,
      "protein_g": 11,
      "fat_g": 5,
      "carbs_g": 52
    }
  ]
}
Response 200: {
  "log_id": "uuid",
  "items_parsed": [...],
  "totals": {"calories": 540, "protein_g": 38, "fat_g": 12, "carbs_g": 64}
}
```

### Log food (photo)

```
POST /v1/nutrition/log/photo
Body: multipart/form-data with image
Response 200: {
  "log_id": "uuid",
  "items_recognized": [...],
  "confidence": 0.74,
  "needs_confirmation": true
}
```

### Daily summary

```
GET /v1/nutrition/today
Response 200: {
  "date": "2026-05-14",
  "target_calories": 2650,
  "target_macros": {"protein_g": 180, "fat_g": 80, "carbs_g": 260},
  "logged_calories": 540,
  "logged_macros": {...},
  "meals": [...],
  "remaining": {"calories": 2110, "protein_g": 142, "fat_g": 68, "carbs_g": 196}
}
```

---

## 7 · Coach chat

### Send message

```
POST /v1/chat/message
Body: {
  "thread_id": "uuid" | null,  // null = new thread
  "content": "Болить ліве коліно",
  "context_hint": "injury" | "training" | "nutrition" | "general"
}
Response 200 (streaming):
  data: { "thread_id": "uuid", "message_id": "uuid" }
  data: { "delta": "Я " }
  data: { "delta": "не " }
  data: { "delta": "лікар. " }
  data: { "delta": null, "complete": true, "context_hint_used": "injury", "escalated": false }
```

### Get conversation history

```
GET /v1/chat/threads/{thread_id}
Response 200: {
  "thread_id": "uuid",
  "messages": [
    {"role": "user", "content": "...", "timestamp": "..."},
    {"role": "victor", "content": "...", "timestamp": "..."}
  ]
}
```

---

## 8 · Notifications

### Get pending push

```
GET /v1/notifications/pending
Response 200: {
  "notifications": [
    {
      "id": "uuid",
      "scheduled_at": "2026-05-14T06:42:00Z",
      "category": "morning_check",
      "title": "Доброго ранку",
      "body": "HRV 68, сон 7:12. Тіло окей.",
      "deep_link": "form://today"
    }
  ]
}
```

### Update preferences

```
PATCH /v1/notifications/preferences
Body: {
  "max_per_day": 2,
  "quiet_hours": {"start": "22:00", "end": "07:00"},
  "categories_enabled": ["morning_check", "pre_workout"]
}
Response 200: {updated preferences}
```

---

## 9 · Data export (GDPR)

### Export request

```
POST /v1/data/export
Response 200: { "export_id": "uuid", "status": "processing" }
```

### Export status / download

```
GET /v1/data/export/{export_id}
Response 200: {
  "status": "ready",
  "download_url": "https://r2.form.coach/exports/...",
  "expires_at": "2026-05-21T..."
}
```

JSON includes ALL user data — profile, health snapshots, workouts, sets, nutrition logs, chat history, settings.

---

## 10 · Subscription / billing

Most billing goes through Apple/Google IAP. Server validates receipt.

### Validate receipt

```
POST /v1/billing/validate
Body: { "platform": "apple" | "google", "receipt": "..." }
Response 200: {
  "subscription_status": "active",
  "plan": "annual",
  "expires_at": "..."
}
```

### Cancel (in-app, redirects to OS subscription manager)

```
GET /v1/billing/manage-url
Response 200: { "url": "https://apps.apple.com/account/subscriptions" }
```

---

## 11 · Error handling

Стандартний error envelope:

```
{
  "error": {
    "code": "INVALID_TOKEN" | "RATE_LIMITED" | "VALIDATION" | ...,
    "message": "Human-readable explanation",
    "details": {...}  // optional
  }
}
```

HTTP status codes:
- 200 — Success
- 400 — Validation
- 401 — Auth
- 403 — Forbidden (subscription required, etc.)
- 404 — Not found
- 409 — Conflict (e.g., duplicate operation)
- 429 — Rate limited
- 500 — Server error
- 503 — Maintenance

---

## 12 · Rate limiting

| Endpoint | Limit |
|---|---|
| Auth | 10/hour per user |
| Chat message | 30/hour per user |
| Workout set log | 100/hour per user (overkill, для multi-set safety) |
| Food log (photo) | 20/day per user |
| Data export | 1/day per user |
| All others | 1000/hour per user |

429 response includes `Retry-After` header.

---

## 13 · Data schema (Postgres / Supabase)

### Core tables

```sql
users (
  id UUID PRIMARY KEY,
  apple_user_id TEXT UNIQUE,
  email_hash TEXT,
  display_name TEXT,
  age INT,
  biological_sex TEXT,
  country TEXT,
  training_age_years INT,
  goals JSONB,
  voice_persona TEXT DEFAULT 'smart_direct',
  soft_mode_enabled BOOLEAN DEFAULT false,
  ed_screening_completed BOOLEAN DEFAULT false,
  ed_flags JSONB,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
)

health_snapshots (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  timestamp TIMESTAMPTZ,
  source TEXT,
  hrv_rmssd_ms FLOAT,
  resting_heart_rate_bpm INT,
  sleep_duration_minutes INT,
  sleep_deep_minutes INT,
  sleep_rem_minutes INT,
  body_temp_c FLOAT,
  steps INT,
  raw_data JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
)
CREATE INDEX idx_health_user_time ON health_snapshots (user_id, timestamp DESC);

programs (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  name TEXT,
  goal TEXT,
  weeks_total INT,
  weeks_completed INT,
  started_at TIMESTAMPTZ,
  metadata JSONB
)

workout_sessions (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  program_id UUID REFERENCES programs(id),
  name TEXT,
  scheduled_at TIMESTAMPTZ,
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  perceived_exertion INT,
  feeling_rating INT,
  metadata JSONB
)

workout_sets (
  id UUID PRIMARY KEY,
  workout_session_id UUID REFERENCES workout_sessions(id),
  exercise_id TEXT,
  set_number INT,
  reps_completed INT,
  weight_kg FLOAT,
  rpe_actual FLOAT,
  tempo_actual_seconds JSONB,
  cv_data JSONB,
  logged_at TIMESTAMPTZ DEFAULT now()
)

food_entries (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  logged_at TIMESTAMPTZ,
  meal_type TEXT,
  input_type TEXT,
  input_text TEXT,
  items JSONB,
  totals JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
)
CREATE INDEX idx_food_user_time ON food_entries (user_id, logged_at DESC);

chat_threads (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  context_hint TEXT,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
)

chat_messages (
  id UUID PRIMARY KEY,
  thread_id UUID REFERENCES chat_threads(id),
  role TEXT,
  content TEXT,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
)
```

### Privacy considerations

- `email_hash` — sha256 of email, NOT actual email
- `apple_user_id` — sub from Apple JWT, opaque
- `ed_flags` — boolean flags, NOT actual answers stored
- Chat content — encrypted at rest (Supabase default), can be deleted on user request
- Health snapshots — most sensitive; access logged

---

## 14 · API versioning

All endpoints under `/v1/`. Breaking changes → `/v2/`. Old version supported для 12 місяців minimum.

Deprecation header `Sunset: ...` 90 days before removal.

---

## 15 · Open API questions

1. **WebSocket для chat вже у v1 чи стрімінг через SSE?**
   - SSE простіше і дешевше, скоріше за все starting point
   - WebSocket тільки якщо real-time коучинг through chat (поки не plan)

2. **Sync conflict resolution для offline events?**
   - Last-write-wins на server, з conflict flag для review (rare cases)

3. **Public API для third-party integrations?**
   - Phase 2 (M9+). Дозволити Notion / Health-export / Whoop sync
