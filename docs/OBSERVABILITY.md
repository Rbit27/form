# FORM · Observability & Monitoring Taxonomy v0.7

> Owner: devops-lead. Review: quarterly or on architecture change. SOC 2 evidence: CC7.2.

---

## Purpose and Scope

This document defines FORM's observability strategy: what we measure, how we measure it, what triggers a human response, and how long we keep each signal. It is the authoritative reference for instrumenting new services, configuring alerts, building dashboards, and demonstrating monitoring controls to auditors.

Scope covers all production systems: Cloudflare Workers (edge API), Cloudflare Pages (marketing + admin), Supabase Postgres (primary database), Supabase Auth (consumer identity), Auth0 / WorkOS (enterprise SSO), Anthropic Claude API (Victor AI coach), ElevenLabs (voice synthesis), the iOS and Android mobile applications (React Native / Expo), the on-device CV pose estimation pipeline (Apple Vision / ML Kit), and all dependent SaaS processors (PostHog, Sentry, ElevenLabs, Better Stack).

**Multi-tenant note:** FORM operates a B2C consumer tier and a multi-tenant enterprise tier. Every observability concern must propagate `tenant_id` for enterprise contexts so that per-tenant SLA reporting, breach scoping, and isolation verification are possible from signal data alone.

**SOC 2 mapping:** This document is formal evidence for CC7.2 (the entity monitors system components for anomalies that could indicate a malicious act, a natural disaster, or an operational error). The alerting rules in §7, dashboard inventory in §8, and tooling decisions in §10 constitute the operating control evidence. See `docs/INCIDENT_RESPONSE.md` for CC7.3–CC7.5 evidence (response, recovery, and post-incident review).

**GDPR note:** FORM processes workout performance data, body composition trends, and health-adjacent metrics that fall within or close to GDPR Art. 9 special category data (health data). Observability data must not re-create raw health records. Log schemas in §5 reflect this constraint: AI inference logs capture latency and token counts, not prompt content or response content.

---

## 0. RED Methodology

The RED method — **Rate**, **Errors**, **Duration** — provides a per-service diagnostic framework. For every external-facing service FORM operates, these three signals answer: *how much traffic?*, *how much is failing?*, *how slow is it?*

| Service | Rate (R) | Errors (E) | Duration (D) |
|---|---|---|---|
| **Cloudflare Workers API** | `cf_worker_requests_total` req/s by route | `cf_worker_errors_total` / `cf_worker_requests_total{status_class="5xx"}` | `cf_worker_request_duration_ms` P95/P99 |
| **Supabase Postgres** | Queries/s (from `pg_stat_statements.calls`) | Failed queries from `pg_stat_statements.rows = 0 AND query LIKE 'SELECT%'`; RLS policy denials | `supabase_db_query_duration_ms` P95 |
| **Supabase Auth** | `supabase_auth_signins_total` events/min | `supabase_auth_failures_total` by `failure_reason` | P95 of `auth_token_verify` span duration |
| **Anthropic Claude API** | `anthropic_api_requests_total` req/min | `anthropic_api_requests_total{outcome=~"error\|timeout\|rate_limited"}` | `anthropic_api_latency_ms` P95 (time-to-first-token + stream complete) |
| **ElevenLabs TTS** | `elevenlabs_requests_total` req/min | `elevenlabs_requests_total{outcome="error"}` | `elevenlabs_latency_ms` P95 |
| **R2 TTS Cache** | `r2_cache_hits_total + r2_cache_misses_total` req/min | Cache misses are not errors, but miss rate > 60% triggers review | Not latency-sensitive; omit |
| **Mobile App (iOS)** | Sessions/day (PostHog) | `sentry_crash_rate{platform="ios"}` | App launch P95, screen load P95 (Sentry Performance) |
| **Mobile App (Android)** | Sessions/day (PostHog) | `sentry_crash_rate{platform="android"}` | App launch P95, screen load P95 (Sentry Performance) |
| **EAS OTA Updates** | `eas_update_checks_total` checks/hour | `eas_update_checks_total{outcome="error"}` / total | Not actively measured; success rate (SLO §2.1) is the primary signal |
| **Enterprise SSO (Auth0 / WorkOS)** | SAML/OIDC assertions/min per tenant | `supabase_auth_signins_total{provider="sso", outcome="failure"}` by tenant | P95 of SSO round-trip (SP-redirect → assertion received), target < 3,000 ms |

### RED triage cascade

When an alert fires, work through RED in order:

1. **Rate** — has traffic volume changed? A spike means more absolute errors even at the same error rate. A drop could indicate clients are failing before they reach the service.
2. **Errors** — filter by label (`route`, `tenant_id`, `outcome`, `failure_reason`) to narrow scope. A single tenant's errors are very different from a platform-wide error spike.
3. **Duration** — is the latency shift at P50 (systemic) or only P99 (long-tail outlier, possibly a single bad request)? P95 is the primary SLO target.

RED signals cascade across dependency tiers: an upstream provider latency increase (Anthropic, ElevenLabs) first appears as a **Duration** spike, then escalates to **Errors** as retries exhaust, then to a **Rate** drop as clients fall back or give up. Following this cascade backwards identifies the origin service. See §7.1 for the operational dashboard layout that surfaces this cascade.

---

## 1. Three Pillars Overview

| Pillar | What it answers | Primary tooling (proposed) | Secondary tooling | Retention |
|---|---|---|---|---|
| **Metrics** | Is the system healthy right now? How is it trending? | Cloudflare Analytics + custom Prometheus-compatible counters via Workers Analytics Engine | PostHog (product metrics), Better Stack (infra metrics) | 90 days hot; 13 months aggregate |
| **Logs** | What happened and in what order? | Cloudflare Logpush (NDJSON → R2), Supabase Postgres logs, Sentry (structured error events) | Better Stack log indexer | 30–90 days queryable; cold archive per §9 |
| **Traces** | Where did this specific request spend its time? | Cloudflare Tracing (W3C TraceContext propagation), Sentry Performance | — | 7 days detailed; 30 days sampled |

All three pillars must carry `trace_id` and `tenant_id` (where applicable) as first-class fields. A request that cannot be traced end-to-end across all three pillars is a monitoring gap.

---

## 2. SLIs and SLOs

SLIs (Service Level Indicators) are the specific measurements. SLOs (Service Level Objectives) are the targets we commit to internally. Alerting thresholds are where we page before we breach an SLO.

### 2.1 SLO Table

| SLI Name | Measurement | SLO Target | Alerting Threshold | Window | Owner |
|---|---|---|---|---|---|
| **API Gateway availability** | % of non-5xx responses over total requests (excluding 429s) | 99.9% | < 99.5% in a 15-min window | Rolling 30 days | devops-lead |
| **API Gateway P95 latency** | 95th percentile response time, all endpoints | < 1,500 ms | > 1,500 ms sustained 10 min | Rolling 7 days | devops-lead |
| **API Gateway P99 latency** | 99th percentile response time, all endpoints | < 3,000 ms | > 3,000 ms sustained 5 min | Rolling 7 days | devops-lead |
| **Database availability** | Supabase Postgres reachable from Workers health check | 99.95% | Unreachable > 60 seconds | Rolling 30 days | devops-lead |
| **Database query P95 latency** | p95 query duration from `pg_stat_statements` | < 200 ms | > 500 ms sustained 5 min | Rolling 7 days | platform-engineer |
| **AI inference P95 latency** | Time from Worker receiving request to Anthropic response received | < 8,000 ms | > 12,000 ms sustained 5 min | Rolling 7 days | platform-engineer |
| **AI inference availability** | % of Anthropic API calls that receive a non-5xx response | 99.5% | < 98% in 5-min window | Rolling 7 days | devops-lead |
| **TTS audio delivery P95** | Time from ElevenLabs request to first audio byte returned | < 2,000 ms | > 3,000 ms sustained 10 min | Rolling 7 days | platform-engineer |
| **TTS cache hit rate** | % of ElevenLabs requests served from R2 cache | > 60% | < 40% in 1-hour window | Rolling 7 days | platform-engineer |
| **Mobile crash-free sessions (iOS)** | % of sessions without a native crash (Sentry) | ≥ 99.5% | < 99.0% in 1-hour window | Rolling 7 days | mobile-engineer |
| **Mobile crash-free sessions (Android)** | % of sessions without a native crash (Sentry) | ≥ 99.5% | < 99.0% in 1-hour window | Rolling 7 days | mobile-engineer |
| **JS error-free sessions** | % of React Native sessions without an unhandled JS exception | ≥ 99.0% | < 98.0% in 1-hour window | Rolling 7 days | mobile-engineer |
| **Enterprise SSO availability** | % of SAML/OIDC authentication attempts that succeed (excluding user-error failures) | 99.9% per tenant | Any tenant: < 99% in 15-min window | Rolling 30 days | security-engineer |
| **EAS Update delivery** | % of OTA update checks that successfully resolve a bundle | 99.5% | < 98% in 30-min window | Rolling 7 days | devops-lead |
| **Audit log write latency** | P95 time added to request by `auditLog()` middleware | < 50 ms | > 100 ms P95 sustained 5 min | Rolling 7 days | platform-engineer |
| **HMAC chain integrity** | Binary: chain unbroken across all tenants | 100% (any break = P0) | Any break detected | Weekly batch | security-engineer |

### 2.2 Error Budget Policy

Each SLO carries a 30-day rolling error budget. When an error budget reaches 50% consumed:
- Weekly review is elevated to devops-lead + platform-engineer sync.
- No new features deploying to the affected service until budget recovers above 75%.

When an error budget reaches 0% consumed (SLO is breached):
- Feature freeze on the affected service.
- P1 incident opened automatically.
- RCA required before feature work resumes.

---

## 3. Metrics Taxonomy

### 3.1 Infrastructure Metrics

| Metric Name | Type | Source | Labels | Retention | Notes |
|---|---|---|---|---|---|
| `cf_worker_requests_total` | Counter | Cloudflare Analytics Engine | `route`, `method`, `status_class` | 90 days | Status classes: 2xx, 3xx, 4xx, 5xx |
| `cf_worker_request_duration_ms` | Histogram | Cloudflare Analytics Engine | `route`, `method` | 90 days | Buckets: 100, 250, 500, 1000, 2000, 5000, 10000 ms |
| `cf_worker_cpu_ms` | Histogram | Cloudflare Analytics Engine | `worker_name` | 90 days | CPU time per invocation; cost proxy |
| `cf_worker_errors_total` | Counter | Cloudflare Analytics Engine | `route`, `error_type` | 90 days | `error_type`: exception, timeout, memory_exceeded |
| `cf_worker_kv_operations_total` | Counter | Cloudflare Analytics Engine | `operation`, `namespace` | 90 days | `operation`: get, put, delete |
| `cf_worker_kv_latency_ms` | Histogram | Cloudflare Analytics Engine | `operation`, `namespace` | 90 days | — |
| `supabase_db_connections_active` | Gauge | Supabase metrics endpoint (`/metrics`) | — | 90 days | Alert if > 80% of `max_connections` |
| `supabase_db_connections_idle` | Gauge | Supabase metrics endpoint | — | 90 days | — |
| `supabase_db_query_duration_ms` | Histogram | `pg_stat_statements` via Supabase | `query_hash` | 7 days | Do not log query text in metric labels |
| `supabase_db_replication_lag_bytes` | Gauge | `pg_stat_replication` | `replica` | 90 days | Alert > 16 MB |
| `supabase_db_table_size_bytes` | Gauge | `pg_total_relation_size()` | `table_name` | 90 days | Sampled hourly |
| `supabase_auth_signins_total` | Counter | Supabase Auth logs | `provider`, `outcome` | 90 days | `provider`: apple, email, sso; `outcome`: success, failure |
| `supabase_auth_failures_total` | Counter | Supabase Auth logs | `provider`, `failure_reason` | 90 days | Alert if > 50/min (credential stuffing signal) |
| `anthropic_api_requests_total` | Counter | Cloudflare Worker instrumentation | `model`, `outcome` | 90 days | `outcome`: success, rate_limited, error, timeout |
| `anthropic_api_latency_ms` | Histogram | Cloudflare Worker instrumentation | `model` | 90 days | Measures time-to-first-token + streaming complete |
| `anthropic_api_tokens_input_total` | Counter | Cloudflare Worker instrumentation | `model`, `feature` | 90 days | Cost tracking; `feature`: coach_chat, plan_generation, insights |
| `anthropic_api_tokens_output_total` | Counter | Cloudflare Worker instrumentation | `model`, `feature` | 90 days | Cost tracking |
| `anthropic_cache_read_tokens_total` | Counter | Cloudflare Worker instrumentation | `model`, `feature` | 90 days | Prompt cache efficiency |
| `elevenlabs_requests_total` | Counter | Cloudflare Worker instrumentation | `voice_id`, `outcome` | 90 days | `outcome`: success, cached, error |
| `elevenlabs_latency_ms` | Histogram | Cloudflare Worker instrumentation | `voice_id` | 90 days | — |
| `elevenlabs_characters_total` | Counter | Cloudflare Worker instrumentation | `voice_id` | 90 days | Cost tracking |
| `r2_cache_hits_total` | Counter | Cloudflare Worker instrumentation | `asset_type` | 90 days | `asset_type`: tts_audio, data_export |
| `r2_cache_misses_total` | Counter | Cloudflare Worker instrumentation | `asset_type` | 90 days | — |
| `eas_update_checks_total` | Counter | Expo EAS metrics | `channel`, `runtime_version`, `outcome` | 90 days | — |
| `sentry_crash_rate` | Gauge | Sentry API | `platform`, `release` | 90 days | Derived: crashes / sessions |
| `sentry_error_rate` | Gauge | Sentry API | `platform`, `release` | 90 days | — |

### 3.2 Product Metrics

These flow through PostHog (EU-hosted). PostHog is the source of truth for product analytics. Do not duplicate product event tracking in the infrastructure metrics pipeline.

| Metric Name | Type | Source | PostHog Event Name | Retention | Notes |
|---|---|---|---|---|---|
| `dau` | Gauge | PostHog persons + events | Session start | 24 months | Distinct users with ≥ 1 session in calendar day |
| `wau` | Gauge | PostHog persons + events | Session start | 24 months | Rolling 7-day window |
| `mau` | Gauge | PostHog persons + events | Session start | 24 months | Rolling 28-day window |
| `w_acsu` | Gauge | PostHog | `coaching_cue_delivered` or `plan_adjusted` | 24 months | North-star: Weekly Active Coached Sessions per User |
| `session_duration_seconds` | Histogram | PostHog | `session_ended` with duration property | 24 months | App session, not coaching session |
| `workout_session_duration_seconds` | Histogram | PostHog | `workout_completed` | 24 months | Active workout time only |
| `workout_completions_total` | Counter | PostHog | `workout_completed` | 24 months | Label: `completed: true/false` |
| `onboarding_completion_rate` | Gauge | PostHog funnel | `onboarding_completed` / `onboarding_started` | 24 months | — |
| `first_coached_session_rate` | Gauge | PostHog funnel | `coaching_session_started` within 7d of install | 24 months | Activation metric |
| `feature_flag_exposure_total` | Counter | PostHog | `$feature_flag_called` | 90 days | Per flag; used for canary monitoring |
| `victor_cue_delivered_total` | Counter | PostHog | `coaching_cue_delivered` | 24 months | Victor coaching interactions |
| `voice_playback_total` | Counter | PostHog | `voice_playback_started` | 24 months | ElevenLabs playback initiated |
| `voice_error_total` | Counter | PostHog | `voice_playback_failed` | 24 months | — |
| `d1_retention` | Gauge | PostHog cohort | Cohort retention query | 24 months | Day 1 return rate by install week |
| `d7_retention` | Gauge | PostHog cohort | Cohort retention query | 24 months | — |
| `d30_retention` | Gauge | PostHog cohort | Cohort retention query | 24 months | — |

### 3.3 Business Metrics

Business metrics are derived. Primary computation happens in Supabase (billing tables) and PostHog (funnel analytics). They are surfaced in the Business dashboard (§8.3).

| Metric Name | Type | Source | Retention | Notes |
|---|---|---|---|---|
| `trial_start_total` | Counter | Supabase `subscriptions` table | 36 months | Event: trial created |
| `trial_to_paid_conversions_total` | Counter | Supabase `subscriptions` table | 36 months | Conversion from trial to active subscription |
| `trial_conversion_rate` | Gauge | Derived (Supabase) | 36 months | Target: ≥ 15% |
| `mrr_usd` | Gauge | Supabase `subscriptions` + Stripe webhook | 36 months | Monthly Recurring Revenue; recalculated daily |
| `arpu_usd` | Gauge | Derived (MRR / MAU) | 36 months | Target: ≥ $12 |
| `churn_rate_monthly` | Gauge | Supabase `subscriptions` table | 36 months | Cancellations / active subscriptions at period start |
| `net_revenue_retention` | Gauge | Supabase `subscriptions` table | 36 months | Target: ≥ 95% |
| `enterprise_seats_provisioned` | Gauge | Supabase `tenant_members` | 36 months | Per tenant; SCIM-provisioned + manual |
| `enterprise_seats_active_wau` | Gauge | PostHog + tenant_id join | 36 months | Active = ≥ 1 session in 7 days |
| `enterprise_seat_utilization` | Gauge | Derived | 36 months | `seats_active_wau / seats_provisioned`; target > 60% |
| `enterprise_mrr_usd` | Gauge | Supabase `subscriptions` | 36 months | Separated from consumer MRR |
| `infra_cost_per_user_usd` | Gauge | Cost exports (Cloudflare, Supabase, Anthropic, ElevenLabs) | 36 months | Target: ≤ $1.90; reviewed monthly |
| `anthropic_cost_usd_daily` | Gauge | Anthropic usage API | 36 months | Alert if +50% day-over-day |
| `elevenlabs_cost_usd_daily` | Gauge | ElevenLabs usage API | 36 months | Alert if +50% day-over-day |

---

## 4. Log Taxonomy

All logs are structured JSON. No plaintext log lines in production. Every log entry must include `timestamp` (ISO 8601 UTC), `level`, `service`, and `trace_id`. Enterprise-context logs must also include `tenant_id`.

### 4.1 Log Table

| Log Name | Format | Source | Sensitivity | Retention (queryable) | Retention (cold archive) | GDPR Note |
|---|---|---|---|---|---|---|
| **Access log** | JSON (see §4.2) | Cloudflare Logpush | Low | 30 days | 90 days in R2 | IPs masked to /24 after 30 days |
| **API error log** | JSON (see §4.3) | Cloudflare Worker + Sentry | Medium | 30 days | 90 days in R2 | No request body content in logs |
| **Audit log** | JSON (HMAC-chained) | Supabase Postgres `audit_log` table | High | 90 days (read-indexed) | 3–10 years; see AUDIT_LOG_SCHEMA.md | Changes JSONB redacts health fields; see §4.4 |
| **AI inference log** | JSON (see §4.5) | Cloudflare Worker | High | 30 days | 90 days in R2 | No raw prompt or response content — GDPR Art. 9 |
| **Auth event log** | JSON | Supabase Auth built-in | High | 90 days | 3 years (per AUDIT_LOG_SCHEMA.md) | Auth events are audit log subset; dual-stored |
| **Enterprise activity log** | JSON (NDJSON export) | Supabase `audit_log` filtered by `tenant_id` | High | 90 days queryable | 7 years | Delivered to tenant SIEM on request; see AUDIT_LOG_SCHEMA.md §Export |
| **Mobile crash log** | Sentry envelope format | Sentry SDK (iOS + Android) | Medium | 90 days | Not archived separately | No health data in crash context; breadcrumbs must not include user health fields |
| **EAS Update log** | JSON | EAS Build service | Low | 30 days | 90 days | No user data; build metadata only |
| **Cost anomaly log** | JSON | Computed daily by cost-monitor Worker | Low | 365 days | 36 months | No user data |

### 4.2 Access Log Schema

Emitted per HTTP request by Cloudflare Logpush. Written to R2 bucket `form-logs-access` in NDJSON.

```json
{
  "timestamp": "2026-05-16T10:23:41.123Z",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "request_id": "01HY3K...",
  "method": "POST",
  "path": "/v1/coach/message",
  "status": 200,
  "duration_ms": 843,
  "client_ip_masked": "185.220.101.0",
  "user_agent": "FORM-iOS/2.1.0 (iPhone; iOS 18.2)",
  "country": "DE",
  "cf_ray": "8a3f2b...",
  "worker_name": "form-api-prod",
  "cache_status": "MISS",
  "tenant_id": "t_0193abc...",
  "user_id_hash": "sha256:e3b0c44..."
}
```

`user_id` is stored as a SHA-256 hash in access logs. The plain `user_id` is available in the audit log (which carries appropriate access controls) but must not appear in the high-volume access log stream.

### 4.3 API Error Log Schema

Emitted by Cloudflare Worker on any unhandled exception or explicit error response. Also forwarded to Sentry as a structured event.

```json
{
  "timestamp": "2026-05-16T10:23:42.001Z",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "level": "error",
  "service": "form-api",
  "error_type": "AnthropicTimeoutError",
  "error_message": "Upstream timeout after 10000ms",
  "status_returned": 503,
  "path": "/v1/coach/message",
  "worker_version": "abc123def",
  "tenant_id": "t_0193abc...",
  "sentry_event_id": "b7d3e1..."
}
```

No stack traces containing user data. Error messages must not include request parameters, user IDs in plaintext, or workout/health data.

### 4.4 Audit Log Schema Reference

The audit log is the authoritative tamper-evident record of all privileged actions. Full schema defined in `docs/AUDIT_LOG_SCHEMA.md`. Key GDPR constraints repeated here for observability engineers:

- `changes` JSONB field: fields `password`, `mfa_secret`, `api_key`, and `health_data_blob` are replaced with `"[REDACTED]"` before write.
- Individual user health or workout data values are never stored in `changes`. Only metadata about access is stored (who accessed, when, which resource ID, scope).
- `actor_ip` is masked to `/24` (IPv4) or `/48` (IPv6) before storage. Full IP retained only 30 days for security investigation purposes, then truncated by partition job.

### 4.5 AI Inference Log Schema

GDPR constraint: this log must not contain the content of prompts sent to Anthropic or the content of Anthropic responses. It captures operational telemetry only. Any monitoring of Victor's coaching quality must happen through aggregated metrics (see §8.6), not through storing individual prompt/response pairs.

```json
{
  "timestamp": "2026-05-16T10:23:41.500Z",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "level": "info",
  "service": "form-api",
  "event": "anthropic_inference_completed",
  "model": "claude-sonnet-4-6",
  "feature": "coach_chat",
  "input_tokens": 1842,
  "output_tokens": 312,
  "cache_read_tokens": 1204,
  "cache_write_tokens": 0,
  "latency_ms": 2341,
  "time_to_first_token_ms": 812,
  "outcome": "success",
  "tenant_id": "t_0193abc...",
  "session_id": "sess_xyz..."
}
```

`session_id` is a pseudonymous identifier for correlating inference calls within a coaching session. It does not map directly to a `user_id` in this log. The mapping exists only in Supabase (access-controlled).

### 4.6 Enterprise Activity Log Delivery

Enterprise tenants on the Professional and Enterprise tiers can receive their audit log stream via:
- **Webhook**: POST each event to their endpoint within P95 < 5 seconds of write.
- **S3/R2 sync**: Daily NDJSON export to a tenant-controlled bucket.
- **REST pull**: `GET /v1/audit/logs?since=...&until=...&tenant_id=...` with pagination.
- **SIEM**: Datadog, Splunk, and Sumo Logic native exporters (roadmap).

Format: NDJSON by default; CEF available for SIEM integrations. All delivery uses mutual TLS or signed URLs. Events include `hmac_self` for chain verification on the tenant side.

---

## 5. Distributed Traces

### 5.1 Trace Propagation Model

FORM uses W3C TraceContext (`traceparent` / `tracestate` headers) for trace propagation. Every inbound request to a Cloudflare Worker generates a trace root if no `traceparent` header is present, or continues the trace if one arrives from the mobile client.

```
Mobile App (React Native)
    │  traceparent: 00-{trace_id}-{span_id}-01
    ▼
Cloudflare Worker (edge)
    │  Span: cf_worker_handle_request
    │  Attaches: tenant_id, user_id_hash, route
    ├──► Cloudflare KV                   [Span: kv_get]
    ├──► Supabase Postgres (PostgREST)   [Span: supabase_query]
    │       └──► pg_stat_activity row visible in DB logs
    ├──► Anthropic Claude API            [Span: anthropic_inference]
    │       Outbound: traceparent forwarded if Anthropic accepts it
    │       Otherwise: Anthropic span is a leaf span in our trace
    └──► ElevenLabs API                  [Span: elevenlabs_tts]
             First checks R2 cache       [Span: r2_cache_lookup]
```

### 5.2 Span Definitions

| Span Name | Parent | Key Attributes | Notes |
|---|---|---|---|
| `http_request` | Root | `http.method`, `http.route`, `http.status_code`, `tenant_id`, `user_id_hash` | Root span for all inbound requests |
| `auth_token_verify` | `http_request` | `auth.provider`, `auth.outcome` | JWT verification; must never log the token value |
| `rls_query` | `http_request` | `db.table`, `db.operation`, `tenant_id` | Supabase query with RLS applied |
| `supabase_query` | `http_request` | `db.statement_hash`, `db.rows_returned`, `duration_ms` | Statement hash, not statement text, to avoid logging query parameters |
| `kv_get` | `http_request` | `kv.namespace`, `kv.key_prefix`, `kv.hit` | Key prefix only; never log the full key if it encodes user data |
| `anthropic_inference` | `http_request` | `ai.model`, `ai.feature`, `ai.input_tokens`, `ai.output_tokens`, `ai.cache_read_tokens`, `duration_ms` | No prompt or response content |
| `elevenlabs_tts` | `http_request` | `tts.voice_id`, `tts.character_count`, `tts.cache_hit`, `duration_ms` | — |
| `r2_cache_lookup` | `elevenlabs_tts` or `http_request` | `r2.bucket`, `r2.key_prefix`, `r2.hit` | — |
| `audit_log_write` | `http_request` | `audit.action`, `audit.outcome`, `duration_ms` | Must be synchronous for `auth.*`, `tenant.*`, `support.*` actions |
| `eas_update_check` | Mobile root | `eas.channel`, `eas.runtime_version`, `eas.update_available` | Mobile-only span; not propagated server-side |

### 5.3 Tenant ID Propagation Rules

`tenant_id` is the most critical field for multi-tenant isolation verification. It must be present on every span that touches tenant data.

- **Cloudflare Worker:** Extract `tenant_id` from the verified JWT claims on first auth. Attach to the root span immediately. All child spans inherit it.
- **Supabase queries:** `tenant_id` flows into RLS policies via `auth.jwt()`. The span attribute must match the RLS-enforced value — any mismatch is a monitoring finding.
- **AI inference spans:** `tenant_id` is attached but the inference content (prompt, response) is not. The log is safe to store without triggering GDPR Art. 9 concerns.
- **Missing `tenant_id`:** Any span missing `tenant_id` on a path that requires tenant context generates a `tenant_id_missing` metric counter. Alert fires if this counter exceeds 0 in a 5-minute window (indicates a code path bypassing tenant context).

### 5.4 Trace Sampling Strategy

| Traffic Type | Sampling Rate | Rationale |
|---|---|---|
| Errors (any status ≥ 500) | 100% | Never drop error traces |
| Slow requests (> 2× P95 latency) | 100% | Always capture outliers |
| Enterprise tenant requests | 100% | SLA evidence; per-tenant tracing |
| AI inference requests | 25% | High volume; full latency distribution covered |
| Standard API requests | 5% | Sufficient for P95/P99 computation |
| Health check / synthetic requests | 0% | Excluded from traces |

---

## 6. Alerting Rules

Alerts route through Better Stack (or PagerDuty once team size warrants). All P0 and P1 alerts page on-call. P2 alerts post to Slack. P3 alerts are tracked in dashboards only — no notification.

### 6.1 Severity Definitions (Alerting)

| Severity | Response SLA | Notification | Wake-up? |
|---|---|---|---|
| **P0** | Acknowledge < 15 min | PagerDuty page + SMS + call | Yes |
| **P1** | Acknowledge < 30 min | PagerDuty page | Yes |
| **P2** | Acknowledge < 2 hours (business hours) | Slack #eng-alerts | No |
| **P3** | Tracked in dashboard; reviewed weekly | No notification | No |

### 6.2 Alert Rules Table

| Alert Name | Trigger Condition | Severity | Notification Channel | Runbook |
|---|---|---|---|---|
| **API error rate critical** | HTTP 5xx rate > 10% of requests in any 5-min window | P0 | PagerDuty → on-call | INCIDENT_RESPONSE.md §R-02 |
| **API error rate elevated** | HTTP 5xx rate > 5% of requests in any 15-min window | P1 | PagerDuty → on-call | INCIDENT_RESPONSE.md §R-02 |
| **Full platform unavailable** | API health check fails for > 60 consecutive seconds | P0 | PagerDuty → on-call + IC | INCIDENT_RESPONSE.md §R-02 |
| **Database unreachable** | Supabase Postgres health check fails for > 60 seconds | P0 | PagerDuty → on-call + IC | INCIDENT_RESPONSE.md §R-02 |
| **Database connection pool near exhaustion** | `supabase_db_connections_active` > 80% of `max_connections` for > 5 min | P1 | PagerDuty → on-call | ENGINEERING_RUNBOOK.md |
| **Database slow queries** | P95 query latency > 500 ms sustained for 5 min | P2 | Slack #eng-alerts | ENGINEERING_RUNBOOK.md |
| **Replication lag high** | `supabase_db_replication_lag_bytes` > 16 MB for > 5 min | P1 | PagerDuty → on-call | ENGINEERING_RUNBOOK.md |
| **Mobile crash rate critical (iOS)** | Sentry crash-free session rate < 99.0% in any 1-hour window | P0 | PagerDuty → mobile-engineer + on-call | INCIDENT_RESPONSE.md §R-02 |
| **Mobile crash rate critical (Android)** | Sentry crash-free session rate < 99.0% in any 1-hour window | P0 | PagerDuty → mobile-engineer + on-call | INCIDENT_RESPONSE.md §R-02 |
| **Mobile crash rate elevated** | Sentry crash-free session rate < 99.5% in any 1-hour window | P1 | PagerDuty → mobile-engineer | INCIDENT_RESPONSE.md §R-02 |
| **Anthropic API down** | Anthropic API returns non-2xx > 80% of calls for > 5 min | P0 | PagerDuty → on-call | INCIDENT_RESPONSE.md §R-02 |
| **Anthropic API degraded** | Anthropic API P95 latency > 12,000 ms sustained 5 min | P1 | PagerDuty → on-call | ENGINEERING_RUNBOOK.md |
| **Anthropic rate limit approaching** | `anthropic_api_requests_total{outcome="rate_limited"}` > 10/min | P2 | Slack #eng-alerts | ENGINEERING_RUNBOOK.md |
| **ElevenLabs unavailable** | ElevenLabs returns non-2xx > 50% of calls for > 5 min | P1 | PagerDuty → on-call | ENGINEERING_RUNBOOK.md |
| **HMAC chain break detected** | Chain verification cron fails for any tenant | P0 | PagerDuty → security-engineer + IC simultaneously | INCIDENT_RESPONSE.md §R-05 |
| **Auth failure spike** | `supabase_auth_failures_total` > 50 events/min for > 2 min | P1 | PagerDuty → security-engineer | INCIDENT_RESPONSE.md §R-03 |
| **Enterprise SSO outage** | Any enterprise tenant SSO success rate < 99% for > 15 min | P1 | PagerDuty → on-call + security-engineer | INCIDENT_RESPONSE.md §R-04 |
| **API P95 latency SLO breach** | P95 latency > 1,500 ms sustained 10 min | P2 | Slack #eng-alerts | ENGINEERING_RUNBOOK.md |
| **Traffic spike anomaly** | Request rate > 5× 7-day baseline in any 10-min window | P2 | Slack #security-alerts | INCIDENT_RESPONSE.md §R-06 |
| **Cost anomaly: Anthropic** | `anthropic_cost_usd_daily` > 150% of 7-day average | P2 | Slack #eng-alerts | — |
| **Cost anomaly: ElevenLabs** | `elevenlabs_cost_usd_daily` > 150% of 7-day average | P2 | Slack #eng-alerts | — |
| **Cost anomaly: any service +20% DoD** | Any monitored cost line +20% day-over-day | P3 | Dashboard only (reviewed weekly) | — |
| **TTS cache hit rate low** | R2 TTS cache hit rate < 40% for > 1 hour | P3 | Dashboard only | — |
| **Error budget 50% consumed** | Any SLO's 30-day error budget reaches 50% consumed | P2 | Slack #eng-alerts | — |
| **Error budget 0% (SLO breach)** | Any SLO's 30-day error budget reaches 0% | P1 | PagerDuty → devops-lead | — |
| **SSL certificate expiry** | Any FORM-controlled certificate expires in < 14 days | P2 | Slack #eng-alerts | — |
| **tenant_id missing on traced path** | `tenant_id_missing` counter > 0 in any 5-min window | P1 | PagerDuty → security-engineer | INCIDENT_RESPONSE.md §R-01 |
| **Audit log write failure** | `auditLog()` middleware returns error on `auth.*`, `tenant.*`, or `support.*` action | P0 | PagerDuty → security-engineer + IC | INCIDENT_RESPONSE.md §R-05 |
| **EAS Update delivery degraded** | EAS update check success rate < 98% for > 30 min | P2 | Slack #eng-alerts | — |
| **Supabase status page incident** | Supabase status page reports incident in FORM's region | P1 (auto-opened) | PagerDuty → on-call | INCIDENT_RESPONSE.md §R-02 |
| **Cloudflare status page incident** | Cloudflare status page reports incident in Workers | P1 (auto-opened) | PagerDuty → on-call | INCIDENT_RESPONSE.md §R-02 |

### 6.3 Alert Tuning Rules

- An alert that fires as a false positive more than twice in 30 days must be tuned. Do not suppress — adjust the threshold or add a stabilization window.
- All alert changes require a commit to the alerting configuration in source control (not a dashboard-only change). Changes are reviewed in the quarterly review.
- Alert silence during deployments must be scoped (specific alert, specific window, maximum 30 minutes). No blanket silences.

---

## 7. Dashboards

Seven dashboards are maintained. Each has a designated owner who reviews it at minimum weekly and is responsible for keeping it accurate when services change.

### 7.1 Operational — System Health

**Owner:** devops-lead. **Refresh:** 1 minute.

The primary triage dashboard. Visible on the ops monitor screen at all times.

| Panel | Metric / Source | Visualization |
|---|---|---|
| API request rate | `cf_worker_requests_total` by status class | Stacked time-series |
| API error rate (%) | 5xx / total | Time-series with SLO threshold line at 0.1% |
| API P50/P95/P99 latency | `cf_worker_request_duration_ms` histogram | Multi-line time-series |
| Database connections | `supabase_db_connections_active` / max | Gauge + trend |
| Database P95 query latency | `supabase_db_query_duration_ms` | Time-series |
| Anthropic API status | `anthropic_api_requests_total` by outcome | Stat panel |
| Anthropic P95 latency | `anthropic_api_latency_ms` | Time-series |
| ElevenLabs status | `elevenlabs_requests_total` by outcome | Stat panel |
| Active incidents | Better Stack / PagerDuty open incidents | Alert list |
| Error budget burn rate | All SLOs, 30-day window | Table with RAG status |

### 7.2 Product — Engagement

**Owner:** growth-lead. **Refresh:** 5 minutes.

Used in weekly product reviews and investor updates. Sourced from PostHog.

| Panel | Metric / Source | Visualization |
|---|---|---|
| DAU / WAU / MAU trend | PostHog person queries | Multi-line time-series |
| W-ACSU (North Star) | PostHog `coaching_cue_delivered` per user per week | Time-series with target line |
| Activation funnel | Install → first coached session (7-day window) | Funnel |
| D1 / D7 / D30 retention | PostHog cohort analysis | Retention grid by install week |
| Workout completion rate | `workout_completions_total{completed=true}` / total | Time-series |
| Session duration distribution | PostHog session length histogram | Histogram |
| Victor coaching cue volume | `victor_cue_delivered_total` | Time-series |
| Feature flag exposure | PostHog feature flag calls by flag name | Table |
| Voice playback success rate | `voice_playback_total` vs `voice_error_total` | Stacked bar |

### 7.3 Business — Revenue and Cost

**Owner:** founder (until finance-lead hired). **Refresh:** Daily (batch job).

Reviewed monthly with devops-lead on the cost side.

| Panel | Metric / Source | Visualization |
|---|---|---|
| MRR trend | Supabase subscriptions + Stripe | Time-series |
| Trial-to-paid conversion rate | Supabase (trial cohorts) | Time-series with 15% target line |
| Churn rate (monthly) | Supabase subscriptions | Time-series |
| ARPU | Derived (MRR / MAU) | Stat |
| Net Revenue Retention | Supabase | Stat with 95% target |
| Enterprise MRR vs consumer MRR | Supabase segmented | Stacked bar |
| Enterprise seat utilization | `enterprise_seat_utilization` per tenant | Table (one row per tenant) |
| Infra cost per active user | `infra_cost_per_user_usd` | Time-series with $1.90 target |
| Cost breakdown by service | Cloudflare + Supabase + Anthropic + ElevenLabs cost exports | Stacked bar |
| Anthropic daily cost trend | `anthropic_cost_usd_daily` | Time-series with anomaly highlight |
| ElevenLabs daily cost + cache savings | `elevenlabs_cost_usd_daily` + R2 cache hit rate | Dual-axis time-series |

### 7.4 Enterprise — Per-Tenant SLA

**Owner:** devops-lead + customer-success. **Refresh:** 5 minutes.

Used for enterprise QBRs and SLA reporting. Filterable by `tenant_id`.

| Panel | Metric / Source | Visualization |
|---|---|---|
| Uptime (rolling 30 days) per tenant | Better Stack + `cf_worker_requests_total` filtered by `tenant_id` | Stat per tenant (target: 99.9%) |
| API P95 latency per tenant | `cf_worker_request_duration_ms` filtered by `tenant_id` | Time-series |
| SSO success rate per tenant | `supabase_auth_signins_total{provider=sso}` by tenant | Time-series |
| Seat utilization per tenant | `enterprise_seat_utilization` | Gauge per tenant |
| Active seats (WAU) per tenant | PostHog + tenant join | Time-series |
| Audit log export delivery (webhook P95) | Better Stack endpoint check | Stat per tenant |
| Open incidents affecting tenant | PagerDuty incident list filtered by tenant label | Alert list |
| SCIM provisioning events | Supabase audit log `auth.sso_provisioned` / `tenant.scim_provisioned` | Event timeline |

### 7.5 Security — Audit Integrity

**Owner:** security-engineer. **Refresh:** 15 minutes.

Used for SOC 2 evidence collection and for ongoing security operations. Access restricted to security-engineer and compliance-officer.

| Panel | Metric / Source | Visualization |
|---|---|---|
| HMAC chain status (all tenants) | HMAC cron results | Binary OK/BREAK per tenant |
| Auth failure rate | `supabase_auth_failures_total` | Time-series with 50/min threshold line |
| Auth failures by IP subnet | Supabase Auth logs | Table (top 20 subnets) |
| `support.*` audit events (24h) | Supabase audit log | Event list |
| `data.export_initiated` events | Supabase audit log | Event list |
| WAF rule trigger volume | Cloudflare WAF analytics | Time-series by rule ID |
| Credential stuffing indicators | `supabase_auth_failures_total` grouped by IP | Heatmap |
| Audit log write latency P95 | `audit_log_write` span duration | Time-series with 50 ms SLO |
| `tenant_id_missing` counter | Custom metric | Time-series (any value > 0 is a finding) |
| Geographic anomalies | Cloudflare Analytics country distribution | World map |

### 7.6 AI Quality — Victor Coach

**Owner:** platform-engineer + product-lead. **Refresh:** 15 minutes.

Monitors the operational health of Victor. Does not display or log prompt/response content. Quality is assessed via proxies and user-observable outcomes.

| Panel | Metric / Source | Visualization |
|---|---|---|
| Inference request rate | `anthropic_api_requests_total` | Time-series |
| Inference P95 / P99 latency | `anthropic_api_latency_ms` | Multi-percentile time-series |
| Time to first token P95 | `time_to_first_token_ms` (inference log) | Time-series |
| Inference error rate | `anthropic_api_requests_total{outcome!=success}` / total | Time-series |
| Input / output token ratio | `anthropic_api_tokens_output_total` / `anthropic_api_tokens_input_total` | Time-series (monitors for prompt bloat) |
| Cache read hit rate | `anthropic_cache_read_tokens_total` / `anthropic_api_tokens_input_total` | Time-series (target: maximize) |
| Cost per inference (estimated) | Derived from input + output + cache tokens × model pricing | Stat |
| Voice delivery success rate | `elevenlabs_requests_total` by outcome | Time-series |
| Voice cache hit rate | `r2_cache_hits_total{asset_type=tts_audio}` / (hits + misses) | Gauge (target: > 60%) |
| Victor cue delivery volume | `victor_cue_delivered_total` | Time-series |
| User thumbs-down events | PostHog `coaching_feedback_negative` | Counter + rate |
| Coaching session abandonment rate | PostHog funnel: session started → session ended with `completed: false` | Time-series |

### 7.7 Mobile — Release Health

**Owner:** mobile-engineer. **Refresh:** 5 minutes.

Activated during and after every EAS Update push or native build release. Compared against previous release baseline.

| Panel | Metric / Source | Visualization |
|---|---|---|
| Crash-free session rate (iOS) | Sentry by release | Time-series with 99.5% SLO line |
| Crash-free session rate (Android) | Sentry by release | Time-series with 99.5% SLO line |
| JS error rate by release | Sentry by release | Time-series |
| Crash volume by issue | Sentry top issues | Table (top 10 by impact) |
| New issues introduced in release | Sentry `first_seen` filter | Alert list |
| EAS update adoption rate | EAS metrics: downloads / eligible devices | Time-series by channel |
| Update delivery success rate | `eas_update_checks_total` by outcome | Time-series |
| ANR rate (Android) | Sentry ANR events | Time-series |
| Active release version distribution | Sentry session data | Pie chart |
| Rollout progress (canary) | PostHog feature flag exposure for canary flags | Time-series (target: 5% → 25% → 100%) |

---

## 8. Data Retention and Privacy

### 8.1 Log Retention Schedule

| Log / Data Type | Queryable Retention | Cold Archive | Deletion Method | Regulatory Basis |
|---|---|---|---|---|
| Access logs (Cloudflare Logpush) | 30 days (R2 indexed) | 90 days (R2 cold prefix) | Object expiry policy | Internal policy |
| API error logs | 30 days | 90 days in R2 | Object expiry policy | Internal policy |
| AI inference logs | 30 days | 90 days in R2 | Object expiry policy | GDPR Art. 5(1)(e) data minimisation |
| Audit log — `auth.*` | 90 days (Postgres) | 3 years (R2 archive) | Partition drop; summary hash preserved | SOC 2 CC7; common compliance |
| Audit log — `tenant.*`, `data.export/deletion`, `privacy.consent_*` | 90 days | 7 years | Partition drop; summary hash preserved | SOC 2; GDPR proof of compliance |
| Audit log — `support.*` | 90 days | 10 years | Partition drop; summary hash preserved | Legal discovery; trust requirement |
| Audit log — `integration.api_call` (sampled) | 30 days | Not archived | Partition drop | Volume management |
| Mobile crash logs (Sentry) | 90 days | Not archived | Sentry auto-expiry | Internal policy |
| Distributed trace data | 7 days (full) | 30 days (sampled 1%) | Automatic TTL | Internal policy |
| Product analytics (PostHog) | 24 months active | Aggregate only | PostHog data retention setting | Internal policy |
| Business metrics (Supabase) | Indefinite (structured DB) | N/A | Manual review process | Internal policy |
| Incident evidence (P0/P1) | Duration of investigation | 1 year minimum; 7 years if regulatory notification filed | Manual; requires security-engineer authorization | SOC 2; GDPR Art. 33 |

### 8.2 GDPR Art. 9 Constraint

FORM's workout performance data (exercise load, body weight trends, recovery scores, coaching interaction patterns) is health-adjacent and may be classified as GDPR Art. 9 special category data depending on regulatory interpretation. FORM treats it as Art. 9 data as a conservative position.

Consequences for observability:

- **AI inference logs must not store prompt content.** Victor's prompts are assembled from user health data. Storing prompt content = storing Art. 9 data in logs. The schema in §4.5 is designed to comply with this.
- **Trace span attributes must not include workout data values.** Span attributes capture structural metadata (table names, row counts, resource IDs). They do not capture field values from health-related tables.
- **Crash reports must be reviewed before sending to Sentry.** Mobile breadcrumbs and crash context are stripped of health-adjacent fields at the SDK layer before transmission. The Sentry `beforeSend` hook must filter `workout_data`, `body_metrics`, and `coaching_context` from all crash reports.
- **Log retention for AI inference is 30 days active / 90 days cold.** Shorter than other categories due to the health-adjacent inference context.

### 8.3 Audit Log Immutability

The audit log is append-only. Normal application code has no `UPDATE` or `DELETE` permission on `audit_log`. The one permitted exception is HMAC chain repair under the dual-authorization procedure described in `docs/INCIDENT_RESPONSE.md §R-05`. That repair itself generates an audit log entry.

Partition drops (for expired records per §8.1) are performed by the `compliance-officer` using the Supabase CLI. Before each partition drop, the SHA-256 hash of the final row's `hmac_self` is stored in `audit_log_retention_summary` to maintain proof of chain continuity across deletions.

---

## 9. Tooling Decisions

These are proposed tools. The observability stack is not yet fully implemented as of v0.1 (pre-launch). Tooling is selected and configurations are being written. Implementation status is noted per item.

### 9.1 Metrics

| Pillar | Tool | Role | Status |
|---|---|---|---|
| Edge metrics | **Cloudflare Analytics Engine** | Primary metrics store for Workers. Exposes a SQL-compatible API for querying custom metrics emitted via `cf.analytics.engine.writeDataPoint()`. Supports up to 20 blob fields and 20 double fields per data point. | Proposed; not yet instrumented |
| Infra metrics | **Better Stack** (Metrics) | Prometheus-compatible ingest for custom metrics exported from Workers. Used for SLO burn-rate dashboards and alerting rules that require multi-signal correlation. | Proposed |
| Product metrics | **PostHog** (EU Cloud, Frankfurt) | Event-level product analytics. Persons, funnels, cohorts, feature flags. Source of truth for all product and business metrics. | Partially implemented (SDK installed) |
| Business metrics | **Supabase Postgres** (computed views) | Subscription and billing data. Materialized views compute MRR, churn, ARPU on a daily schedule. | Proposed |
| Cost metrics | **Custom Worker** (daily batch) | Pulls usage from Anthropic API, ElevenLabs API, Cloudflare billing API, and Supabase billing. Emits cost metrics to Analytics Engine. Alerts via Better Stack if thresholds exceeded. | Proposed |

**Why not Prometheus self-hosted:** Pre-Series A. Prometheus + Grafana cluster adds operational surface area that is not justified until team size and metric volume warrants it. Cloudflare Analytics Engine + Better Stack covers the use case without running infrastructure to monitor infrastructure.

### 9.2 Logs

| Pillar | Tool | Role | Status |
|---|---|---|---|
| Edge logs | **Cloudflare Logpush** → R2 | Streams all Worker request logs (access log format) to `form-logs-access` R2 bucket. Configurable filters exclude health check traffic. NDJSON format. | Proposed |
| Database logs | **Supabase built-in logging** | Postgres logs, Auth logs, and PostgREST logs available via Supabase dashboard and `supabase logs` CLI. Retained 7 days by Supabase; exported to R2 for longer retention. | Available; export not yet configured |
| Error logs | **Sentry** | Structured error and crash reporting for Workers (Sentry Cloudflare SDK) and mobile (Sentry React Native SDK). Source of truth for crash-free session rates. | Partially implemented (mobile SDK installed) |
| Log indexing | **Better Stack** (Logs) | Ingests NDJSON from R2 via S3 source connector. Provides search, alerting on log content, and live tail. Used for incident investigation queries. | Proposed |
| Audit log | **Supabase Postgres** (`audit_log` table) | Append-only, HMAC-chained, partitioned. See `docs/AUDIT_LOG_SCHEMA.md` for full schema. | Implemented |

**Why not Datadog:** Cost. Datadog log ingest pricing scales linearly with volume and would exceed cost targets at our projected scale. Better Stack provides comparable log search at significantly lower cost for a sub-Series A company.

### 9.3 Traces

| Pillar | Tool | Role | Status |
|---|---|---|---|
| Edge tracing | **Cloudflare Tracing** (W3C TraceContext) | Cloudflare Workers natively support `traceparent` header propagation. Workers generate and forward trace context. Trace data available in Cloudflare's trace viewer for Workers paid plans. | Proposed |
| Application tracing | **Sentry Performance** | Sentry's transaction tracing for React Native (mobile) and Workers (backend). Provides waterfall views for individual requests. Integrates with Sentry error tracking — traces are linked to error events automatically. | Proposed |
| Mobile tracing | **Sentry React Native** performance transactions | App start time, screen load time, API call duration from the mobile client perspective. Propagates `traceparent` on outbound API requests. | Proposed |

**Why not OpenTelemetry collector:** Cloudflare Workers run in an edge environment without persistent processes. A collector sidecar is not available. OpenTelemetry SDK for Workers (OTLP export via fetch) is viable but adds complexity without a clear backend target at this stage. Revisit at Series A when ClickHouse is in place — OTLP → ClickHouse is a strong fit for trace analytics at scale.

### 9.4 Alerting and On-Call

| Tool | Role | Status |
|---|---|---|
| **Better Stack** (Alerts) | Alert rules for metrics and log-content triggers. Routes to PagerDuty or directly to on-call. | Proposed |
| **PagerDuty** | On-call rotation management, escalation policies, incident timeline. Required once team size > 1 engineer. | Proposed (pending first engineering hire) |
| **Slack** (webhooks) | P2 and P3 alert delivery. `#eng-alerts` for operational; `#security-alerts` for security-classified alerts. | Partially configured |
| **Statuspage** | Public-facing uptime page. Manually updated during incidents per templates in `docs/INCIDENT_RESPONSE.md §6.2`. Auto-updates from Better Stack uptime checks. | Proposed |

---

## 10. Open Questions and Gaps

The following items are known gaps as of v0.1. Each has an owner and a target resolution milestone.

| Gap | Description | Owner | Target |
|---|---|---|---|
| **Cloudflare Analytics Engine instrumentation** | No custom metrics are being emitted from Workers yet. All metric references in §3.1 are schema definitions, not live data. | platform-engineer | M3 (pre-launch) |
| **Logpush to R2 not configured** | Access logs are not being persisted. Incidents requiring log replay are currently impossible. | devops-lead | M2 |
| **Sentry Workers SDK not installed** | Backend error tracking is absent. Worker exceptions are not being captured. Mobile SDK is installed but not backend. | platform-engineer | M2 |
| **Better Stack not provisioned** | No centralized alerting or log indexing. All monitoring is currently manual dashboard checks. | devops-lead | M2 |
| **PagerDuty not provisioned** | On-call routing is manual. Acceptable solo-founder stage; required before first on-call rotation. | devops-lead | M3 or first engineering hire |
| **Per-tenant SLA reporting** | The Enterprise SLA dashboard (§8.4) depends on `tenant_id` label propagation in metrics, which is not yet implemented. Design complete — see **§13**. | platform-engineer | M4 (enterprise launch) |
| **Sentry `beforeSend` health data filter** | The hook to strip health-adjacent fields from crash reports before Sentry transmission is not yet implemented. Blocking for GDPR compliance. | mobile-engineer | M2 |
| **Cost monitor Worker** | Daily cost aggregation from all billing APIs is not yet implemented. Cost anomaly alerting is not active. Design complete — see **§17**. | devops-lead | M3 |
| **ClickHouse for analytics** | Planned from M9. Until then, PostHog and Supabase materialized views carry the analytics load. Observability strategy for ClickHouse (traces, product events at scale) is not yet designed. | platform-engineer | M9 |
| **Trace correlation between Sentry and Cloudflare** | Sentry transactions and Cloudflare trace IDs are not yet correlated. An error in Sentry cannot be mapped to a Cloudflare trace without manual log search. Requires shared `trace_id` propagation. Design complete — see **§14**. | platform-engineer | M3 |
| **HMAC chain verification cron** | The weekly chain integrity cron described in `AUDIT_LOG_SCHEMA.md` is not yet deployed. Chain breaks would not be detected automatically. | devops-lead | M3 |
| **Audit log export pipeline** | Enterprise webhook delivery and S3 sync described in §4.6 are not yet implemented. Blocking for enterprise launch. Design complete — see **§15**. | platform-engineer | M4 |

---

## 11. SLI Calculation Expressions

These expressions derive each SLI value from raw metrics. Use them as templates when configuring Better Stack SLO monitors or building SQL queries against Cloudflare Analytics Engine.

Syntax is PromQL-compatible (Better Stack / Prometheus). Where Cloudflare Analytics Engine SQL is required, a CFA equivalent is noted.

### 11.1 API Gateway availability
```promql
# 30-day rolling availability (excludes 429 rate-limit responses)
1 - (
  sum(rate(cf_worker_requests_total{status_class="5xx"}[30d]))
  /
  sum(rate(cf_worker_requests_total{status_class!="4xx_rate_limit"}[30d]))
)
# Target: ≥ 0.999
```

### 11.2 API Gateway P95 latency
```promql
# 7-day rolling P95 across all routes
histogram_quantile(0.95,
  sum by (le) (rate(cf_worker_request_duration_ms_bucket[7d]))
)
# Target: < 1500 ms
```

### 11.3 Database availability
```promql
# Computed from health-check probe (binary — up or down)
avg_over_time(supabase_db_health_check_up[30d])
# Target: ≥ 0.9995
```

### 11.4 Database query P95 latency
```promql
histogram_quantile(0.95,
  sum by (le) (rate(supabase_db_query_duration_ms_bucket[7d]))
)
# Target: < 200 ms
```

### 11.5 AI inference P95 latency
```promql
histogram_quantile(0.95,
  sum by (le) (rate(anthropic_api_latency_ms_bucket[7d]))
)
# Target: < 8000 ms
```

### 11.6 AI inference availability
```promql
1 - (
  sum(rate(anthropic_api_requests_total{outcome=~"error|timeout"}[7d]))
  /
  sum(rate(anthropic_api_requests_total[7d]))
)
# Target: ≥ 0.995
```

### 11.7 TTS cache hit rate
```promql
sum(rate(r2_cache_hits_total{asset_type="tts_audio"}[7d]))
/
(
  sum(rate(r2_cache_hits_total{asset_type="tts_audio"}[7d]))
  + sum(rate(r2_cache_misses_total{asset_type="tts_audio"}[7d]))
)
# Target: > 0.60
```

### 11.8 Mobile crash-free session rate
```promql
# Sentry exports this as a gauge; query directly
1 - sentry_crash_rate{platform="ios"}
1 - sentry_crash_rate{platform="android"}
# Target: ≥ 0.995
```

### 11.9 Enterprise SSO availability (per tenant)
```promql
# Computed per tenant_id label
1 - (
  sum by (tenant_id) (rate(supabase_auth_signins_total{provider="sso", outcome="failure"}[30d]))
  /
  sum by (tenant_id) (rate(supabase_auth_signins_total{provider="sso"}[30d]))
)
# Target: ≥ 0.999 per tenant
```

### 11.10 Error budget consumption

Error budget consumed (%) for a given SLO over its window:

```
error_budget_consumed =
  (actual_error_rate - (1 - SLO_target)) / (1 - SLO_target) × 100

# Example: API Gateway with SLO 99.9%
# Actual availability = 99.85% → error rate = 0.15%
# Allowed error rate = 0.1%
# Budget consumed = (0.15 - 0.10) / 0.10 × 100 = 50%
```

Alert fires at 50% consumed. Feature freeze at 0% (SLO breach). See §2.2 for policy.

---

## 12. Developer Instrumentation Guide

A reference for engineers adding observability when shipping a new Worker route, Supabase RPC, or mobile screen. Follow this before opening a PR on any path that is customer-facing or tenant-scoped.

### 12.1 Adding a span to a Cloudflare Worker

FORM uses Sentry Performance (not an OTel collector — see §9.3 for rationale). The Sentry Cloudflare SDK wraps the OTel API surface.

```typescript
import * as Sentry from '@sentry/cloudflare';

export async function handleNewFeatureRoute(
  request: Request,
  env: Env,
  tenantId: string,
): Promise<Response> {
  return Sentry.startSpan(
    {
      name: 'new_feature_name',       // snake_case; add to §5.2 Span Definitions
      op: 'http.server',
      attributes: {
        'http.method': request.method,
        'http.route': '/api/v1/new-feature',
        'tenant_id': tenantId,        // REQUIRED on every tenant-scoped span
        'user_id_hash': sha256(userId), // hash before attaching — never raw user_id
        'feature': 'new_feature_name',
      },
    },
    async (span) => {
      const result = await performWork(env, tenantId);

      // Set result attributes on the span after work completes
      span.setAttribute('result_count', result.items.length);
      span.setAttribute('cache_hit', result.fromCache);

      return Response.json(result);
    },
  );
}
```

**Invariants:**
- `tenant_id` must be the verified value from JWT claims, not a request parameter.
- Never log health data, prompt content, or LLM response content as span attributes.
- Span names use `snake_case` and match the §5.2 Span Definitions table — update the table when adding a new persistent span.

### 12.2 Emitting a custom metric to Cloudflare Analytics Engine

Analytics Engine is the primary metrics store for Workers (see §9.1). Each data point supports up to 20 blob fields (strings) and 20 double fields (numbers).

```typescript
// env.ANALYTICS is the Analytics Engine binding declared in wrangler.toml
function emitMetric(
  env: Env,
  route: string,
  tenantId: string | null,
  outcome: 'success' | 'error' | 'rate_limited' | 'timeout',
  durationMs: number,
  aiTokens?: { input: number; output: number; cacheRead: number },
): void {
  env.ANALYTICS.writeDataPoint({
    blobs: [
      route,           // blob1: route label (primary filter dimension)
      tenantId ?? '',  // blob2: tenant_id (empty string for consumer tier)
      outcome,         // blob3: outcome label
    ],
    doubles: [
      durationMs,                     // double1: duration_ms
      aiTokens?.input ?? 0,           // double2: ai.input_tokens
      aiTokens?.output ?? 0,          // double3: ai.output_tokens
      aiTokens?.cacheRead ?? 0,       // double4: ai.cache_read_tokens
    ],
    indexes: [route],  // High-cardinality filter; one index per data point
  });
}
```

Query with Cloudflare Analytics Engine SQL API:
```sql
SELECT
  blob1 AS route,
  count() AS requests,
  quantileWeighted(0.95)(double1, 1) AS p95_duration_ms,
  sumIf(double1, blob3 = 'error') / count() AS error_rate
FROM ANALYTICS
WHERE timestamp > now() - interval '1' hour
GROUP BY route
ORDER BY requests DESC
```

### 12.3 OTel OTLP via fetch (Series A path)

When a ClickHouse backend is available, Workers can export OTLP traces directly without a collector sidecar — export happens via a `fetch()` call at the end of the request lifecycle.

```typescript
// Set in wrangler.toml / environment secrets
const OTLP_ENDPOINT = env.OTEL_EXPORTER_OTLP_ENDPOINT; // https://otel.example.com
const OTLP_TOKEN   = env.OTEL_AUTH_TOKEN;

async function exportSpanOtlp(spanData: SerializedSpan): Promise<void> {
  // Fire-and-forget — use ctx.waitUntil() to avoid blocking the response
  await fetch(`${OTLP_ENDPOINT}/v1/traces`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-protobuf',
      'Authorization': `Bearer ${OTLP_TOKEN}`,
    },
    body: encodeOtlpTraceRequest([spanData]),
  });
}

// In the Worker fetch handler:
export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    const span = buildSpan(request, env);
    const response = await handleRequest(request, env, span);
    ctx.waitUntil(exportSpanOtlp(span.serialize())); // non-blocking export
    return response;
  },
};
```

This pattern is the planned migration path. Activate at Series A when ClickHouse is provisioned. Until then, Sentry Performance is the active tracing backend.

### 12.4 Mobile screen instrumentation (React Native / Expo)

```typescript
import * as Sentry from '@sentry/react-native';

// Wrap screens in Sentry.wrap() for automatic transaction creation
export default Sentry.wrap(function WorkoutScreen({ route }: Props) {
  // Manual child span for a significant operation within the screen
  const transaction = Sentry.getCurrentHub().getScope()?.getTransaction();
  const span = transaction?.startChild({
    op: 'db.query',
    description: 'load_workout_history',
  });

  const history = await loadHistory();
  span?.finish();

  // ...render
});
```

Mobile spans propagate `traceparent` on outbound API requests automatically when the Sentry SDK is initialized with `tracePropagationTargets: ['api.form.coach']`.

### 12.5 Instrumentation checklist before shipping

| Checkpoint | Required | Notes |
|---|---|---|
| Root span added with `http.route` attribute | Yes | All new Worker routes |
| `tenant_id` attached on every tenant-scoped span | Yes | `tenant_id_missing` counter fires if absent (P1 alert) |
| `user_id_hash` used — never raw `user_id` in span or log | Yes | SHA-256 before attaching |
| No health data in span attributes or log fields | Yes | GDPR Art. 9 — see §4.5 AI inference log schema |
| No prompt / LLM response content in any log or span | Yes | AI inference log captures token counts only |
| Analytics Engine `writeDataPoint()` added for high-traffic paths | Yes | Rate, outcome, duration minimum |
| Existing alert rules reviewed — new critical path needs alert? | Yes | Add to §6.2 and commit |
| New span added to §5.2 Span Definitions table | Yes (if permanent) | PR updating this doc required |
| Mobile spans propagate `traceparent` on API calls | Yes | Verify in Sentry transaction waterfall |
| Sentry `beforeSend` hook excludes health-adjacent fields from crash reports | Yes | Blocks GDPR compliance until done |

---

**v0.2 · May 2026 · Owner: devops-lead**
**Review: quarterly or on architecture change. Next scheduled review: August 2026.**
**SOC 2 evidence: CC7.2 (system monitoring). See also INCIDENT_RESPONSE.md for CC7.3–CC7.5.**

---

## 13. Per-Tenant Observability Implementation

This section specifies how `tenant_id` propagates through every observability pillar — metrics, logs, and traces — to enable per-tenant SLA reporting, breach scoping, and isolation verification. It closes the gap listed in §10 ("Per-tenant SLA reporting", M4).

### 13.1 Tenant context injection (Workers)

Every inbound request to a Cloudflare Worker that carries a valid session JWT must extract `tenant_id` at the edge and attach it to all downstream signals for the lifetime of that request.

```typescript
// src/middleware/tenant-context.ts

export interface TenantContext {
  tenantId: string;
  tenantSlug: string;
  traceId: string; // W3C trace-id extracted from traceparent (see §14)
}

export async function injectTenantContext(
  request: Request,
  env: Env,
  ctx: ExecutionContext
): Promise<TenantContext | null> {
  const jwt = request.headers.get('Authorization')?.replace('Bearer ', '');
  if (!jwt) return null;

  // Validate JWT — throws on invalid signature or expiry
  const claims = await verifyJwt(jwt, env.JWT_SECRET);

  // Enterprise sessions carry tenant_id claim; consumer sessions do not
  const tenantId: string | null = claims['tenant_id'] ?? null;
  if (!tenantId) return null;

  const traceId = extractTraceId(request.headers.get('traceparent'));

  return { tenantId, tenantSlug: claims['tenant_slug'], traceId };
}
```

**Rule:** if a request is tenant-scoped and `tenantId` is absent from emitted signals, the `tenant_id_missing` counter (§6.2) fires as a P1 alert. This counter is the enforcement mechanism — missing context cannot be silently swallowed.

### 13.2 Per-tenant Analytics Engine data points

Each Analytics Engine `writeDataPoint()` call on a tenant-scoped path must include `tenant_id` as a blobs dimension. This enables SQL aggregation per tenant without scanning all rows.

```typescript
// Pattern: always emit tenant_id as blobs[0] on enterprise paths
env.ANALYTICS.writeDataPoint({
  blobs: [
    tenantCtx.tenantId,          // blobs[0]: tenant_id (filter/group-by key)
    route,                        // blobs[1]: http.route
    outcome,                      // blobs[2]: "success" | "error" | "timeout"
  ],
  doubles: [
    durationMs,                   // doubles[0]: request duration ms
    statusCode,                   // doubles[1]: HTTP status
  ],
  indexes: [tenantCtx.tenantId],  // partition key for per-tenant queries
});
```

**Per-tenant availability SLI SQL** (Cloudflare Analytics Engine):

```sql
SELECT
  blob1                                                     AS tenant_id,
  COUNT(*)                                                  AS total_requests,
  COUNTIF(blob2 = 'success')                                AS successful_requests,
  COUNTIF(blob2 = 'success') * 100.0 / COUNT(*)            AS availability_pct,
  QUANTILEWEIGHTED(0.95)(double1, 1)                        AS p95_duration_ms
FROM ANALYTICS_ENGINE_DATASET
WHERE
  timestamp >= NOW() - INTERVAL '30' DAY
  AND blob1 = $tenant_id
GROUP BY tenant_id
```

### 13.3 Per-tenant SLO targets and breach detection

Each enterprise tenant is assigned an SLO tier at contract signing. The tier determines alerting thresholds:

| SLO Tier | Availability Target | P95 Latency Target | SSO Availability | Who gets this |
|---|---|---|---|---|
| **Standard** | 99.9% | < 1,500 ms | 99.9% | Default for all enterprise contracts |
| **Premium** | 99.95% | < 1,000 ms | 99.95% | Contracts with dedicated CSM + SLA addendum |

Tier is stored in the `tenants` table (`slo_tier ENUM('standard', 'premium') NOT NULL DEFAULT 'standard'`). The per-tenant SLA dashboard (§7.4) reads this column to render the correct breach threshold line on graphs.

**Breach detection query** (runs every 15 minutes via Cloudflare Cron Trigger):

```typescript
// workers/sla-monitor.ts
async function checkTenantSla(tenantId: string, tier: SloTier, env: Env) {
  const window = '15 minutes';
  const threshold = tier === 'premium' ? 99.95 : 99.9;

  const result = await queryAnalyticsEngine(env, `
    SELECT
      COUNTIF(blob2 = 'success') * 100.0 / COUNT(*) AS availability_pct,
      QUANTILEWEIGHTED(0.95)(double1, 1)             AS p95_ms
    FROM ANALYTICS_ENGINE_DATASET
    WHERE blob1 = '${tenantId}'
      AND timestamp >= NOW() - INTERVAL '15' MINUTE
  `);

  if (result.availability_pct < threshold) {
    await emitSlaBreach(tenantId, 'availability', result.availability_pct, threshold, env);
  }
  if (result.p95_ms > (tier === 'premium' ? 1000 : 1500)) {
    await emitSlaBreach(tenantId, 'latency_p95', result.p95_ms, tier === 'premium' ? 1000 : 1500, env);
  }
}

async function emitSlaBreach(
  tenantId: string, sliName: string, actual: number, target: number, env: Env
) {
  // 1. Write breach event to audit log (HMAC-chained, see AUDIT_LOG_SCHEMA.md)
  await auditLog({ action: 'sla.breach', tenantId, metadata: { sliName, actual, target } }, env);

  // 2. Alert Better Stack
  await fetch(env.BETTER_STACK_WEBHOOK, {
    method: 'POST',
    body: JSON.stringify({
      severity: 'high',
      title: `SLA breach: ${sliName} for tenant ${tenantId}`,
      message: `${sliName} is ${actual.toFixed(3)}% (target: ${target}%)`,
      tags: { tenant_id: tenantId, sli: sliName },
    }),
  });

  // 3. Notify tenant admin via webhook if configured (see §15 export pipeline)
  await deliverTenantWebhook(tenantId, 'sla.breach', { sliName, actual, target }, env);
}
```

### 13.4 Admin dashboard SLA API

The admin dashboard (`admin-dashboard.html`) consumes a dedicated SLA reporting endpoint. This is separate from the real-time alert path.

**GET `/api/admin/tenants/:tenantId/sla`**

```typescript
// Response schema
interface TenantSlaReport {
  tenantId: string;
  tenantSlug: string;
  sloTier: 'standard' | 'premium';
  reportWindow: { from: string; to: string }; // ISO 8601
  metrics: {
    apiAvailability: { sli: number; slo: number; budgetRemaining: number };
    apiP95Latency:   { sli: number; slo: number; budgetRemaining: number };
    ssoAvailability: { sli: number; slo: number; budgetRemaining: number };
  };
  breaches: Array<{
    occurredAt: string;
    sliName: string;
    durationMinutes: number;
    resolutionStatus: 'resolved' | 'ongoing';
    incidentId: string | null; // links to INCIDENT_RESPONSE post-mortem
  }>;
  errorBudgetConsumed: number; // 0.0–1.0; >= 0.5 triggers advisory
  generatedAt: string;
}
```

**Privacy floor:** this API is scoped to the enterprise admin role (`role = 'tenant_admin'`). It returns aggregate availability figures only. No per-user activity data, no workout content, no health metrics. The HR-never-sees-individual-user-data constraint (DEC-030) extends to SLA reports — the tenant admin SLA view is infrastructure availability, not user behaviour.

### 13.5 Per-tenant isolation verification

Beyond SLA reporting, `tenant_id` propagation enables isolation audit queries — proving that data belonging to tenant A never appeared in signals for tenant B.

**Weekly isolation audit query** (run by the HMAC chain verification cron, §10):

```sql
-- Should return zero rows. Any result is a P0 cross-tenant leak.
SELECT DISTINCT blob1 AS tenant_id, blob1_expected
FROM (
  SELECT
    blob1,
    -- RLS should ensure blob1 matches the session's tenant claim
    -- If they differ, it indicates a missing RLS predicate on a query path
    session_tenant_claim AS blob1_expected
  FROM ANALYTICS_ENGINE_DATASET
  WHERE blob1 != session_tenant_claim
    AND timestamp >= NOW() - INTERVAL '7' DAY
) AS mismatches;
```

If this query returns any row, a P0 incident is raised immediately following the classification in `docs/INCIDENT_RESPONSE.md §3`.

---

## 14. Cross-Backend Trace Correlation (Sentry ↔ Cloudflare)

This section specifies the shared `trace_id` propagation pattern that allows a Sentry error or transaction to be linked to a Cloudflare Worker trace without manual log searching. It closes the gap listed in §10 ("Trace correlation between Sentry and Cloudflare", M3).

### 14.1 Problem statement

As of v0.1, Sentry and Cloudflare use separate trace ID namespaces:
- Cloudflare generates a `CF-Ray` header per request and optionally a W3C `traceparent`.
- Sentry generates its own `sentry-trace` header and transaction IDs.

An engineer investigating a Sentry error cannot navigate directly to the Cloudflare trace for the same request — they must correlate manually via timestamp and approximate path matching, which is unreliable under concurrent load.

### 14.2 Solution: shared W3C `traceparent` + Sentry custom context

The fix is a single source of truth for `trace_id`: the W3C `traceparent` header, generated by the mobile client (or the Worker itself on direct API calls), propagated unchanged through all hops, and attached to the Sentry scope as a custom tag.

```
┌──────────────────────┐
│   React Native app   │
│  Sentry SDK active   │──── outbound HTTP ────▶ Cloudflare Worker
│                      │  Headers:               │  CF-Ray: abc123
│  Transaction: T1     │  traceparent: 00-TRACE_ID-PARENT_SPAN-01
│  (auto-instrumented) │  sentry-trace: T1/S1/1  │
└──────────────────────┘                         │
                                                 ▼
                                         Supabase / Anthropic
                                         (traceparent forwarded)
```

The `TRACE_ID` (128-bit / 32 hex chars, middle segment of `traceparent`) becomes the shared key that appears in:
1. Cloudflare Analytics Engine data points (as `blob3: trace_id` or a dedicated `indexes` entry)
2. Sentry transactions (as `trace.trace_id` tag)
3. Supabase structured logs (as `trace_id` field in API error log schema, §4.3)

### 14.3 Worker-side implementation

```typescript
// src/middleware/trace-context.ts

export function extractTraceId(traceparent: string | null): string {
  if (!traceparent) return crypto.randomUUID().replace(/-/g, '');
  // traceparent = "00-{traceId:32hex}-{parentSpanId:16hex}-{flags:2hex}"
  const parts = traceparent.split('-');
  return parts.length >= 2 ? parts[1] : crypto.randomUUID().replace(/-/g, '');
}

export function buildTraceparent(traceId: string, spanId: string): string {
  return `00-${traceId}-${spanId}-01`;
}

// In the fetch handler: propagate traceparent to all upstream calls
export async function callUpstream(
  url: string,
  options: RequestInit,
  traceId: string
): Promise<Response> {
  const spanId = generateSpanId(); // 16 random hex chars
  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'traceparent': buildTraceparent(traceId, spanId),
      // Sentry propagation header — required for Sentry's distributed tracing
      'sentry-trace': `${traceId}-${spanId}-1`,
    },
  });
}
```

**Analytics Engine attachment** — `trace_id` is written as `indexes[0]` to support direct lookup by trace ID:

```typescript
env.ANALYTICS.writeDataPoint({
  blobs:   [tenantId, route, outcome],
  doubles: [durationMs, statusCode],
  indexes: [traceId],  // enables: SELECT * WHERE indexes[0] = '<trace_id>'
});
```

**Response header** — the Worker echoes `X-Trace-Id` back to the client so that React Native can attach it to the Sentry transaction:

```typescript
response.headers.set('X-Trace-Id', traceId);
response.headers.set('traceparent', buildTraceparent(traceId, rootSpanId));
```

### 14.4 React Native / Expo implementation

```typescript
// src/api/client.ts — base Axios/fetch wrapper

import * as Sentry from '@sentry/react-native';

export async function apiFetch(path: string, options: RequestInit = {}) {
  const transaction = Sentry.getCurrentHub().getScope()?.getTransaction();
  const sentryTrace = transaction
    ? `${transaction.traceId}-${transaction.spanId}-1`
    : undefined;

  const response = await fetch(`${API_BASE}${path}`, {
    ...options,
    headers: {
      ...options.headers,
      ...(sentryTrace ? { 'sentry-trace': sentryTrace } : {}),
    },
  });

  // Attach Cloudflare trace ID to the active Sentry transaction for correlation
  const cfTraceId = response.headers.get('X-Trace-Id');
  if (cfTraceId && transaction) {
    transaction.setTag('cf.trace_id', cfTraceId);
    transaction.setData('cloudflare_trace_id', cfTraceId);
  }

  return response;
}
```

With this in place, every Sentry transaction includes `cf.trace_id` as a searchable tag. An engineer can go from a Sentry error → copy `cf.trace_id` → query Cloudflare Analytics Engine directly.

### 14.5 Cloudflare Analytics Engine correlation query

Given a Sentry `cf.trace_id` value (e.g. `4bf92f3577b34da6a3ce929d0e0e4736`):

```sql
SELECT
  blob1  AS tenant_id,
  blob2  AS route,
  blob3  AS outcome,
  double1 AS duration_ms,
  double2 AS status_code,
  timestamp
FROM ANALYTICS_ENGINE_DATASET
WHERE indexes[0] = '4bf92f3577b34da6a3ce929d0e0e4736'
ORDER BY timestamp ASC
```

This returns every span emitted during the lifecycle of that trace, enabling full root-cause analysis without log searching.

### 14.6 Sentry-to-Cloudflare navigation runbook

1. Open Sentry error or transaction.
2. Locate tag `cf.trace_id` in the "Tags" panel.
3. Copy the value (32 hex chars).
4. Open Cloudflare Analytics Engine → **Explore** → paste the query from §14.5.
5. Cross-reference the `route` and `duration_ms` columns against the Sentry span waterfall.
6. If span durations match, root cause is in the identified route. If they do not appear, the request failed before reaching the Worker (DNS / Cloudflare edge error — check CF-Ray in the Sentry request headers instead).

### 14.7 Verification checklist

| Check | How to verify |
|---|---|
| `X-Trace-Id` present in API responses | `curl -I https://api.form.coach/health` — header must appear |
| `cf.trace_id` tag on Sentry transactions | Sentry → Issues → any recent transaction → Tags tab |
| Analytics Engine row retrievable by trace ID | Run §14.5 query with a known trace ID from a test request |
| `traceparent` forwarded to Anthropic calls | Check Sentry transaction waterfall for a child span with `traceparent` tag |
| `sentry-trace` forwarded to Anthropic calls | Optional; allows future Anthropic trace linkage if they support W3C propagation |

---

## 15. Audit Log Export Pipeline (Enterprise)

This section specifies the architecture and implementation for enterprise audit log delivery — real-time webhook streaming to customer SIEMs and async R2/S3 export for compliance archiving. It closes the gap listed in §10 ("Audit log export pipeline", M4). Cross-reference: `docs/AUDIT_LOG_SCHEMA.md §Export & delivery`.

### 15.1 Delivery architecture overview

```
┌─────────────────────────────────────────────────────────────┐
│                     FORM API Worker                         │
│  auditLog() middleware → Supabase audit_log table           │
│  (HMAC-chained, see AUDIT_LOG_SCHEMA.md §HMAC chaining)     │
└──────────────────────────┬──────────────────────────────────┘
                           │ INSERT trigger (pg_notify)
                           ▼
              ┌────────────────────────┐
              │  Export Dispatcher     │  ← Supabase Edge Function
              │  (edge-fn: audit-export│
              │   -dispatcher)         │
              └────────┬───────────────┘
            ┌──────────┴────────────┐
            ▼                       ▼
  ┌──────────────────┐   ┌──────────────────────────────┐
  │  Webhook delivery │   │  Batch export (R2 → S3/GCS)  │
  │  (real-time SIEM) │   │  (hourly; signed URLs)       │
  └──────────────────┘   └──────────────────────────────┘
```

Two delivery modes are independent and may be configured separately per tenant:
1. **Webhook** — near-real-time delivery (< 30 s P95) for SIEM ingestion (Splunk, Elastic, Datadog).
2. **Batch export** — hourly NDJSON files written to R2, then optionally synced to customer S3/GCS bucket via signed URL or pre-configured credentials.

### 15.2 Tenant export configuration schema

```sql
-- In the tenants table (see DATA_MODEL.md §2.1)
ALTER TABLE tenants ADD COLUMN IF NOT EXISTS audit_export_config JSONB;

-- audit_export_config shape:
-- {
--   "webhook": {
--     "enabled": true,
--     "url": "https://customer-siem.example.com/ingest/form",
--     "secret": "<HMAC secret for webhook signature>",
--     "retry_policy": { "max_attempts": 5, "backoff_seconds": [10, 30, 60, 120, 300] },
--     "event_filter": ["auth.*", "data_access.*", "admin.*"]  -- null = all events
--   },
--   "batch_export": {
--     "enabled": true,
--     "destination": "s3",   -- "s3" | "gcs" | "r2"
--     "bucket": "customer-audit-archive",
--     "prefix": "form/audit/",
--     "credentials_secret_name": "AUDIT_EXPORT_AWS_CREDS_<TENANT_ID>",
--     "include_hmac_chain": true  -- whether to include chain_hash in export
--   }
-- }
```

`secret` for webhook signing is stored as a Cloudflare Workers Secret (never in Supabase plaintext). `credentials_secret_name` is the name of the Workers Secret that holds the S3/GCS credentials for that tenant.

### 15.3 Webhook delivery implementation

```typescript
// workers/audit-export-dispatcher/webhook.ts

interface AuditWebhookPayload {
  specversion: '1.0';            // CloudEvents spec
  type: 'com.form.coach.audit';
  source: 'https://api.form.coach';
  id: string;                    // audit_log.id
  time: string;                  // ISO 8601
  datacontenttype: 'application/json';
  data: {
    tenantId: string;
    action: string;
    actorId: string;              // user_id_hash (SHA-256, never raw)
    targetType: string | null;
    targetId: string | null;
    ipAddress: string | null;
    chainHash: string;            // HMAC chain hash (DEC-030 — chain integrity)
    metadata: Record<string, unknown>;
  };
}

export async function deliverTenantWebhook(
  tenantId: string,
  auditRow: AuditLogRow,
  webhookConfig: WebhookConfig,
  env: Env
): Promise<void> {
  const payload: AuditWebhookPayload = {
    specversion: '1.0',
    type: 'com.form.coach.audit',
    source: 'https://api.form.coach',
    id: auditRow.id,
    time: auditRow.created_at,
    datacontenttype: 'application/json',
    data: {
      tenantId: auditRow.tenant_id,
      action: auditRow.action,
      actorId: sha256Hex(auditRow.actor_id),  // hash before emit
      targetType: auditRow.target_type,
      targetId: auditRow.target_id,
      ipAddress: auditRow.ip_address,
      chainHash: auditRow.chain_hash,
      metadata: auditRow.metadata ?? {},
    },
  };

  const body = JSON.stringify(payload);
  const signature = await hmacSign(body, webhookConfig.secret);

  let lastError: Error | null = null;
  for (let attempt = 0; attempt < webhookConfig.retry_policy.max_attempts; attempt++) {
    try {
      const res = await fetch(webhookConfig.url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'X-Form-Signature': `sha256=${signature}`,
          'X-Form-Delivery-Id': auditRow.id,
          'X-Form-Event': auditRow.action,
        },
        body,
      });
      if (res.ok) return;
      lastError = new Error(`HTTP ${res.status}`);
    } catch (e) {
      lastError = e as Error;
    }
    if (attempt < webhookConfig.retry_policy.max_attempts - 1) {
      await scheduler.wait(webhookConfig.retry_policy.backoff_seconds[attempt] * 1000);
    }
  }

  // All retries exhausted — write delivery failure to audit log and alert
  await auditLog({
    action: 'system.webhook_delivery_failed',
    tenantId,
    metadata: { auditRowId: auditRow.id, attempts: webhookConfig.retry_policy.max_attempts, lastError: lastError?.message },
  }, env);
}
```

**Webhook signature verification** (customer side):

```typescript
// Reference implementation for customer SIEM ingest
const expectedSig = `sha256=${await hmacSign(rawBody, webhookSecret)}`;
const receivedSig = req.headers['x-form-signature'];
if (!timingSafeEqual(expectedSig, receivedSig)) {
  return res.status(401).json({ error: 'invalid_signature' });
}
```

### 15.4 Batch export implementation (R2 → S3/GCS)

```typescript
// Cloudflare Cron Trigger: runs hourly
// workers/audit-export-dispatcher/batch.ts

export async function runBatchExport(tenantId: string, config: BatchExportConfig, env: Env) {
  const windowEnd = new Date();
  const windowStart = new Date(windowEnd.getTime() - 60 * 60 * 1000); // last hour

  // 1. Query Supabase for audit rows in window, ordered by created_at ASC
  const rows = await supabase
    .from('audit_log')
    .select('*')
    .eq('tenant_id', tenantId)
    .gte('created_at', windowStart.toISOString())
    .lt('created_at', windowEnd.toISOString())
    .order('created_at', { ascending: true });

  if (!rows.data?.length) return; // nothing to export

  // 2. Serialize as NDJSON, one row per line
  const ndjson = rows.data.map(row => JSON.stringify({
    ...row,
    actor_id: sha256Hex(row.actor_id), // hash before export
  })).join('\n');

  // 3. Write to R2 (primary archive, always)
  const r2Key = `audit/${tenantId}/${formatHour(windowStart)}.ndjson`;
  await env.R2_AUDIT_ARCHIVE.put(r2Key, ndjson, {
    httpMetadata: { contentType: 'application/x-ndjson' },
    customMetadata: {
      tenant_id: tenantId,
      row_count: String(rows.data.length),
      window_start: windowStart.toISOString(),
      window_end: windowEnd.toISOString(),
    },
  });

  // 4. Sync to customer S3/GCS if configured
  if (config.destination === 's3') {
    const creds = await getSecret(config.credentials_secret_name, env);
    await uploadToS3(ndjson, config.bucket, `${config.prefix}${formatHour(windowStart)}.ndjson`, creds);
  } else if (config.destination === 'gcs') {
    const creds = await getSecret(config.credentials_secret_name, env);
    await uploadToGcs(ndjson, config.bucket, `${config.prefix}${formatHour(windowStart)}.ndjson`, creds);
  }

  // 5. Write export manifest to audit log (chained — export events are auditable)
  await auditLog({
    action: 'system.audit_export_completed',
    tenantId,
    metadata: { rowCount: rows.data.length, r2Key, destination: config.destination },
  }, env);
}
```

### 15.5 HMAC chain export and customer verification

When `include_hmac_chain: true` (default for enterprise), each export file row includes `chain_hash`. This allows the customer to independently verify audit log integrity without trusting FORM.

**Chain verification algorithm** (customer side, language-agnostic):

```
prev_hash ← "genesis"   # or the chain_hash of the last row of the previous export window
for each row in export (ordered by created_at ASC):
    expected_hash ← HMAC-SHA256(
        key  = FORM_AUDIT_HMAC_KEY,            # shared during onboarding
        data = prev_hash + "|" + row.id + "|" + row.action + "|" + row.created_at
    )
    if row.chain_hash != expected_hash:
        RAISE IntegrityError(f"Chain broken at row {row.id}")
    prev_hash ← row.chain_hash
```

The `FORM_AUDIT_HMAC_KEY` is a tenant-specific secret exchanged over a secure channel during enterprise onboarding (not stored in the export file itself). For auditors, this key is provided under NDA as part of the SOC 2 Type II evidence package. Reference: `docs/AUDIT_LOG_SCHEMA.md §HMAC chaining` and DEC-030.

### 15.6 Delivery SLAs

| Delivery Mode | P95 Latency | Availability SLO | Breach Action |
|---|---|---|---|
| Webhook (real-time) | < 30 s from audit event write | 99.9% of events delivered within 5 min | Write `system.webhook_delivery_failed` to audit log; page on-call |
| Batch export (R2) | < 5 min after hour boundary | 100% (every hour must produce a file, even if empty) | P1 incident (missing export = compliance gap) |
| Batch export (S3/GCS) | < 15 min after R2 write | 99.5% | Alert tenant admin; retry next cron cycle; max 3 cycles before P1 |

### 15.7 Privacy constraints on export

The following fields are **never** included in export files, regardless of customer request:

| Field | Reason |
|---|---|
| `prompt_content` / `response_content` | GDPR Art. 9 — AI coaching content is health-adjacent |
| Raw `user_id` | Replaced by `sha256(user_id)` before export (GDPR pseudonymisation) |
| Workout content details | Not stored in audit_log; audit_log records access events, not data content |
| Individual user health metrics | HR-never-sees-individual-user-data constraint (DEC-030) |

The export schema mirrors the privacy guarantees in `docs/AUDIT_LOG_SCHEMA.md §Privacy guarantees`. Any customer request to export raw user health data must be refused and escalated to compliance-officer.

### 15.8 Implementation checklist (platform-engineer)

| Task | Priority | Notes |
|---|---|---|
| Create `audit-export-dispatcher` Supabase Edge Function | P0 (M4) | Triggered by `pg_notify` on `audit_log` INSERT |
| Add `audit_export_config` JSONB column to `tenants` table | P0 (M4) | Migration: non-nullable default `'{}'::jsonb` |
| Store webhook secrets in Cloudflare Workers Secrets (per-tenant) | P0 (M4) | Naming convention: `AUDIT_WEBHOOK_SECRET_<TENANT_ID_UPPER>` |
| Provision `R2_AUDIT_ARCHIVE` bucket | P0 (M4) | Object lifecycle: 7-year retention (GDPR §5(1)(e) storage limitation) |
| Implement `hmacSign` for outbound webhook signatures | P0 (M4) | Use Web Crypto API `HMAC-SHA256`; no external library |
| Implement S3 upload path | P1 (M4) | Only needed for customers requesting S3 sync |
| Implement GCS upload path | P2 (M5) | Deferred until first GCS-native customer |
| Add `system.audit_export_completed` and `system.webhook_delivery_failed` to action taxonomy | P0 (M4) | Update `AUDIT_LOG_SCHEMA.md §Action taxonomy` |
| Add §15.6 SLA metrics to Better Stack monitors | P0 (M4) | Alert on missing batch export within 10 min of hour boundary |
| Add per-tenant webhook delivery alert to §6.2 alert rules | P0 (M4) | Severity: high; owner: platform-engineer |

---

*v0.2 additions: §13 Per-Tenant Observability Implementation (tenant context injection, per-tenant Analytics Engine data points, SLO breach detection, admin dashboard SLA API, isolation verification query); §14 Cross-Backend Trace Correlation — Sentry ↔ Cloudflare shared W3C `traceparent` propagation, Worker implementation, React Native client, correlation query, navigation runbook; §15 Audit Log Export Pipeline — webhook delivery (CloudEvents format, HMAC signing, retry policy), hourly R2/S3/GCS batch export, HMAC chain customer verification, privacy constraints, implementation checklist. Closes three M4/M3 gaps from §10.*

---

## 16. Synthetic Monitoring & Uptime Verification

### 16.1 Purpose

Synthetic monitoring executes scripted probes against FORM's production endpoints on a fixed schedule, independently of real user traffic. It validates that critical user flows are alive and within latency SLOs before users encounter failures — and generates availability evidence for SOC 2 CC7.2, enterprise SLA reporting, and status page truth.

Synthetic checks complement real-user monitoring (RUM) in three scenarios that RUM misses:
1. **Zero-traffic windows** — early morning UTC, when organic traffic is low but SLOs still apply.
2. **Enterprise pilot readiness** — pre-launch validation that SSO flows, SCIM endpoints, and admin APIs are reachable from outside FORM's development network.
3. **Post-deployment smoke tests** — automated gate in the Wrangler + EAS deployment pipeline, blocking promotion if a critical probe fails within 2 minutes of deploy.

### 16.2 Probe Taxonomy

Probes are categorised by **criticality** (P0 = 24/7 paging, P1 = business-hours paging, P2 = Slack-only).

| Probe ID | Name | Endpoint | Method | Assertion | Frequency | Criticality | Owner |
|---|---|---|---|---|---|---|---|
| S-001 | API Health | `GET /health` (Cloudflare Worker) | GET | HTTP 200, `{"status":"ok"}` in body, latency ≤ 200ms | 60s | P0 | devops-lead |
| S-002 | Auth Token Exchange | `POST /auth/v1/token` (Supabase) | POST | HTTP 200, `access_token` present, latency ≤ 500ms | 2m | P0 | platform-engineer |
| S-003 | Workout Logging API | `POST /api/v1/workouts/sets` (synthetic user) | POST | HTTP 201, `set_id` UUID in response, latency ≤ 800ms | 5m | P0 | platform-engineer |
| S-004 | AI Coaching Turn | `POST /api/v1/coaching/message` (synthetic session) | POST | HTTP 200, non-empty `content` in response, latency ≤ 4000ms | 5m | P1 | ml-engineer |
| S-005 | SSO SAML Metadata | `GET /sso/saml/metadata` | GET | HTTP 200, XML `EntityDescriptor` present, valid cert, latency ≤ 300ms | 5m | P0 (enterprise-only) | enterprise-architect |
| S-006 | SCIM User List | `GET /scim/v2/Users` (synthetic tenant token) | GET | HTTP 200, `{"schemas":["urn:ietf:params:scim:api:messages:2.0:ListResponse"]}`, latency ≤ 600ms | 5m | P1 (enterprise-only) | enterprise-architect |
| S-007 | Audit Log Write | Internal HMAC chain integrity check via `GET /internal/audit/health` | GET | HTTP 200, `chain_valid: true`, last entry age ≤ 10m | 10m | P0 | security-engineer |
| S-008 | R2 Backup Freshness | Check most recent object timestamp in `form-backups` R2 bucket via Worker | GET | Most recent object ≤ 25h old | 1h | P1 | devops-lead |
| S-009 | CDN Asset Reachability | `GET https://assets.form.coach/app.js` (or equivalent) | GET | HTTP 200, `Content-Type: application/javascript`, latency ≤ 500ms from 3 regions | 5m | P0 | devops-lead |
| S-010 | Status Page API | `GET https://status.form.coach/api/v1/summary` | GET | HTTP 200, `page.status` present | 5m | P2 | devops-lead |
| S-011 | Stripe Webhook Endpoint | `POST /webhooks/stripe` (Stripe test event replay) | POST | HTTP 200, event acknowledged | 30m | P1 | platform-engineer |
| S-012 | PostHog Ingest Health | `POST https://eu.posthog.com/capture/` (synthetic event) | POST | HTTP 200, `{"status":1}` | 30m | P2 | data-engineer |

### 16.3 Probe Implementation — Better Stack

All probes except S-007 and S-008 are implemented as Better Stack (Uptime) HTTP monitors. S-007 and S-008 are implemented as Cloudflare Workers scheduled tasks that report pass/fail to Better Stack via the Heartbeat API.

**Better Stack configuration per probe:**

```yaml
# better-stack-monitors.yaml
# Declarative config — apply via Better Stack API on infrastructure deploy

monitors:
  - name: "S-001 API Health"
    url: "https://api.form.coach/health"
    type: http
    frequency: 60
    method: GET
    expected_status: 200
    expected_body: '"status":"ok"'
    regions: [us_east, eu_central, ap_southeast]
    on_call_escalation: P0-api

  - name: "S-004 AI Coaching Turn"
    url: "https://api.form.coach/api/v1/coaching/message"
    type: http
    frequency: 300
    method: POST
    headers:
      Authorization: "Bearer {{ secret.SYNTHETIC_USER_JWT }}"
      Content-Type: application/json
    body: '{"session_id":"synthetic-probe","message":"test"}'
    expected_status: 200
    expected_body: '"content"'
    timeout: 8000
    on_call_escalation: P1-api
```

Secrets (`SYNTHETIC_USER_JWT`, `SYNTHETIC_TENANT_SCIM_TOKEN`) are stored in Cloudflare Workers Secrets and rotated quarterly by `security-engineer`. The synthetic probe user and tenant are flagged `is_synthetic: true` in the `users` table and are excluded from all analytics, cohort, and LTV calculations via a PostHog filter (`properties.$synthetic != true`) and a ClickHouse partition filter (`WHERE NOT is_synthetic`).

### 16.4 Probe Implementation — Scheduled Workers (S-007, S-008)

**S-007 — Audit Log Chain Integrity:**

```typescript
// workers/synthetic/audit-health.ts
// Cron: every 10 minutes via wrangler.toml [triggers.crons]

export default {
  async scheduled(_event: ScheduledEvent, env: Env) {
    const { data, error } = await env.SUPABASE.from('audit_log')
      .select('id, hmac, previous_hmac, created_at')
      .order('created_at', { ascending: false })
      .limit(2);

    if (error || !data || data.length < 2) {
      await reportHeartbeat(env.BETTERSTACK_HEARTBEAT_S007, 'fail');
      return;
    }

    const [latest, previous] = data;
    const expectedPreviousHmac = await computeHmac(previous, env.AUDIT_HMAC_KEY);
    const chainValid = latest.previous_hmac === expectedPreviousHmac;
    const ageMs = Date.now() - new Date(latest.created_at).getTime();
    const fresh = ageMs < 10 * 60 * 1000; // 10 min

    await reportHeartbeat(
      env.BETTERSTACK_HEARTBEAT_S007,
      chainValid && fresh ? 'pass' : 'fail'
    );
  }
};
```

**S-008 — Backup Freshness:**

```typescript
// workers/synthetic/backup-freshness.ts
// Cron: every hour

export default {
  async scheduled(_event: ScheduledEvent, env: Env) {
    const objects = await env.FORM_BACKUPS_R2.list({ limit: 1, prefix: 'daily/' });
    const latest = objects.objects[0];
    if (!latest) {
      await reportHeartbeat(env.BETTERSTACK_HEARTBEAT_S008, 'fail');
      return;
    }
    const ageHours = (Date.now() - latest.uploaded.getTime()) / 3_600_000;
    await reportHeartbeat(
      env.BETTERSTACK_HEARTBEAT_S008,
      ageHours <= 25 ? 'pass' : 'fail'
    );
  }
};
```

### 16.5 Multi-Region Probe Strategy

P0 probes (S-001, S-002, S-003, S-005, S-007, S-009) are executed from three regions simultaneously: `us-east-1`, `eu-central-1`, `ap-southeast-1`. A probe is declared **failed** only when ≥ 2 of 3 regions report failure within the same check window, preventing false positives from regional routing anomalies.

**Latency SLO per region (p99):**

| Region | API Health (S-001) | Auth (S-002) | Workout Log (S-003) | AI Turn (S-004) |
|---|---|---|---|---|
| eu-central-1 | ≤ 120ms | ≤ 350ms | ≤ 600ms | ≤ 3,500ms |
| us-east-1 | ≤ 150ms | ≤ 400ms | ≤ 700ms | ≤ 3,800ms |
| ap-southeast-1 | ≤ 200ms | ≤ 500ms | ≤ 900ms | ≤ 4,500ms |

Regional latency SLOs are displayed on the enterprise SLA dashboard (§7.4) and included in the per-tenant SLA report exported monthly to enterprise customers.

### 16.6 Post-Deployment Smoke Gate

Synthetic probes S-001 through S-005 are re-executed as a blocking CI step after every Cloudflare Worker deployment (`wrangler deploy`) and after every EAS production build promotion.

**CI integration (GitHub Actions step):**

```yaml
- name: Synthetic smoke gate
  run: |
    npx @betterstack/cli run-checks \
      --monitors S-001,S-002,S-003,S-004,S-005 \
      --timeout 120 \
      --fail-on-first \
      --api-key ${{ secrets.BETTERSTACK_API_KEY }}
  env:
    BETTERSTACK_API_KEY: ${{ secrets.BETTERSTACK_API_KEY }}
```

If any probe fails within 120 seconds of deploy, the deployment is flagged in Linear as a `deployment-regression` incident and the on-call engineer is paged (P1 severity). The deployment is NOT automatically rolled back — manual rollback decision is made by the on-call engineer per `docs/INCIDENT_RESPONSE.md R-02`.

### 16.7 Synthetic User Data Hygiene

The synthetic probe user (`synthetic@probe.form.coach`) and synthetic tenant (`tenant_id: 00000000-0000-0000-0000-000000000001`) must be excluded from all data pipelines:

| Pipeline | Exclusion mechanism |
|---|---|
| PostHog analytics | `properties.$synthetic != true` filter on all dashboards and exports |
| ClickHouse `fact_events` | `WHERE NOT is_synthetic` on all Metabase questions |
| Cohort retention (`fact_cohort_retention`) | Excluded at ETL dbt model layer via `dim_users.is_synthetic = false` join |
| GDPR DSAR | Exempt — synthetic user is not a natural person; document in DPIA |
| Audit log | Synthetic events are written but tagged `actor_type: synthetic_probe`; excluded from CC6.2 quarterly access reviews |
| Enterprise SLA reports | Excluded — SLA is calculated on real tenant traffic only |

The synthetic user JWT is short-lived (1-hour expiry) and rotated by a Cloudflare Worker cron daily. Historical synthetic JWTs are revoked on rotation; the audit log records each rotation event as `system.synthetic_token_rotated`.

### 16.8 Alerting and On-Call Integration

Synthetic probe failures flow into the same PagerDuty routing as infrastructure alerts (§6.2). The alert payload includes:

```json
{
  "probe_id": "S-003",
  "probe_name": "Workout Logging API",
  "region": "eu-central-1",
  "regions_failed": 2,
  "regions_total": 3,
  "latency_ms": null,
  "http_status": 503,
  "body_snippet": "Error: upstream connect error",
  "consecutive_failures": 3,
  "runbook_url": "https://github.com/Rbit27/form/blob/main/docs/INCIDENT_RESPONSE.md#r-02"
}
```

**Alert deduplication:** Consecutive failures from the same probe within the same 5-minute window are deduplicated into a single PagerDuty incident. A probe returning to passing state fires a `resolve` event that closes the incident automatically.

### 16.9 SOC 2 Evidence Contribution

| SOC 2 Criterion | How Synthetic Monitoring Satisfies It |
|---|---|
| CC7.2 — System monitoring | Probes S-001 through S-012 provide continuous automated monitoring evidence |
| CC7.3 — Anomaly evaluation | Probe failure alerts feed the incident response pipeline (§docs/INCIDENT_RESPONSE.md) |
| A1.2 — Availability processing | Multi-region probes generate the availability data for SLO compliance calculation |
| CC4.1 — Monitoring activities | Monthly probe availability report is an auditor-facing evidence artefact |

Better Stack uptime reports are exported monthly as PDF and stored in `compliance/evidence/synthetic-uptime-YYYY-MM.pdf`. These are referenced in the SOC 2 auditor evidence package under CC7.2.

### 16.10 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Create Better Stack monitor definitions for S-001 through S-012 | devops-lead | P0 | M3 |
| Provision `synthetic@probe.form.coach` user + synthetic tenant in production DB | platform-engineer | P0 | M3 |
| Implement S-007 `audit-health` scheduled Worker | platform-engineer | P0 | M3 |
| Implement S-008 `backup-freshness` scheduled Worker | devops-lead | P0 | M3 |
| Store `SYNTHETIC_USER_JWT` + `SYNTHETIC_TENANT_SCIM_TOKEN` in Workers Secrets | security-engineer | P0 | M3 |
| Add smoke gate step to Wrangler deploy GitHub Action | devops-lead | P1 | M3 |
| Add smoke gate step to EAS production promote workflow | devops-lead | P1 | M4 |
| Add `is_synthetic` column to `users` and `tenants` tables (migration) | platform-engineer | P1 | M3 |
| Configure PostHog dashboard filters to exclude `$synthetic` events | data-engineer | P1 | M3 |
| Configure ClickHouse ETL dbt model `WHERE NOT is_synthetic` | data-engineer | P1 | M4 |
| Daily JWT rotation cron Worker for synthetic user token | platform-engineer | P1 | M4 |
| Monthly Better Stack PDF export to `compliance/evidence/` | devops-lead | P2 | M4 |
| Regional latency SLO tracking in enterprise SLA dashboard | platform-engineer | P2 | M5 |

---

## 17. Cost Monitoring & Anomaly Alerting

> Owner: devops-lead + data-engineer. Review: monthly or on any cost-cliff event. SOC 2 evidence: CC7.2, CC5.2, A1.2.
> Closes the gap documented in §10 ("Cost monitor Worker"). Cross-reference: `docs/COST_MODEL.md` (unit economics and per-service thresholds), `docs/ENTERPRISE.md` (seat pricing and COGS targets).

---

### 17.1 Architecture

The cost monitor is a Cloudflare **scheduled Worker** that runs at 06:00 UTC daily. It pulls the previous day's usage from each vendor's billing API, normalises it into a `cost_daily` table in Supabase, compares against rolling baselines, and either publishes a digest to `#metrics` or triggers a PagerDuty alert if a threshold is breached.

```
06:00 UTC daily
   │
   ├── Anthropic Admin API  → input_tokens, output_tokens, cost_usd
   ├── ElevenLabs /v1/user  → character_count, cost_usd (delta from KV)
   ├── CF GraphQL Analytics → workers_requests, cost_usd
   └── Supabase Mgmt API    → db_size_gb, bandwidth_gb (no marginal billing below plan)
                │
                ▼
         cost_daily (Supabase)
                │
       ┌────────┴─────────────────────────┐
       │  anomaly detection               │  no anomaly
       │  (§17.3 thresholds)              │
       ▼                                  ▼
  PagerDuty + Slack alert         Slack daily digest
```

**Why a scheduled Worker, not a Lambda / Cloud Run job?** Cloudflare Workers are already FORM's compute plane; adding a cron trigger requires only a `wrangler.toml` entry and avoids introducing a second runtime. The Worker runs in the `eu-central-1` Cloudflare data centre to minimise latency to the Supabase EU instance.

**Excluded from initial scope:** PostHog cost (free tier until 1M events/month; trivial to add at that milestone), Redis (fixed instance cost, no per-query billing), Expo/EAS (fixed plan). Add these when marginal spend becomes material (see COST_MODEL.md §13.2).

---

### 17.2 Data Schema

```sql
-- Append-only. One row per (date, service, metric) per day. Upsert on conflict for idempotent re-runs.
CREATE TABLE cost_daily (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  date            DATE NOT NULL,
  service         TEXT NOT NULL   -- 'anthropic' | 'elevenlabs' | 'cloudflare' | 'supabase'
                    CHECK (service IN ('anthropic', 'elevenlabs', 'cloudflare', 'supabase', 'posthog', 'sentry', 'redis')),
  metric_name     TEXT NOT NULL,  -- 'input_tokens' | 'output_tokens' | 'characters' | 'requests' | 'db_size_gb'
  metric_value    NUMERIC NOT NULL,
  cost_usd        NUMERIC(10,4) NOT NULL,   -- actual billed cost in USD
  est_cost_usd    NUMERIC(10,4),            -- projection from COST_MODEL.md (seeded at launch)
  variance_pct    NUMERIC(5,2)              -- (actual − est) / est × 100; NULL until est_cost_usd populated
    GENERATED ALWAYS AS (
      CASE WHEN est_cost_usd IS NOT NULL AND est_cost_usd > 0
           THEN ROUND(((cost_usd - est_cost_usd) / est_cost_usd) * 100, 2)
           ELSE NULL END
    ) STORED,
  tenant_id       UUID REFERENCES tenants(id) ON DELETE SET NULL,  -- NULL = platform-level
  created_at      TIMESTAMPTZ DEFAULT now(),
  UNIQUE (date, service, metric_name, COALESCE(tenant_id, '00000000-0000-0000-0000-000000000000'::UUID))
);

CREATE INDEX cost_daily_date_service ON cost_daily (date, service);
CREATE INDEX cost_daily_tenant_date  ON cost_daily (tenant_id, date) WHERE tenant_id IS NOT NULL;
CREATE INDEX cost_daily_date_desc    ON cost_daily (date DESC);  -- for rolling-average lookups

-- Anomaly log: written when any threshold in §17.3 is breached.
CREATE TABLE cost_anomalies (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  detected_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
  date          DATE NOT NULL,
  service       TEXT NOT NULL,
  metric_name   TEXT NOT NULL,
  observed      NUMERIC NOT NULL,
  baseline_7d   NUMERIC NOT NULL,
  ratio         NUMERIC(5,2) NOT NULL,
  severity      TEXT NOT NULL CHECK (severity IN ('P0', 'P1')),
  resolved_at   TIMESTAMPTZ,
  resolution    TEXT
);
```

**RLS on `cost_daily`:** The `form_api` role cannot read or write this table — it is written only by the scheduled Worker (service-role key, used only in the Worker's secret binding, never in request-path code) and read by `form_admin` and Metabase service account only. Enterprise tenant admins do **not** see this table; per-tenant COGS is surfaced as an aggregated metric through the admin dashboard API (§7.4).

---

### 17.3 Anomaly Thresholds by Service

Thresholds are derived from COST_MODEL.md §13.3 "cost cliff" analysis and §2.1 behavioral assumptions. The 1.5× P1 threshold detects gradual drift; the 3× P0 threshold detects runaway usage.

| Service | Metric | P1 threshold | P0 threshold | Model baseline (daily) | Notes |
|---|---|---|---|---|---|
| **Anthropic** | `input_tokens` | > 1.5× 7d avg | > 3× 7d avg | MAU × 0.43k tokens/day [ESTIMATE] | Prompt caching hit-rate drop triggers this first |
| **Anthropic** | `output_tokens` | > 1.5× 7d avg | > 3× 7d avg | MAU × 0.108k tokens/day [ESTIMATE] | |
| **Anthropic** | `cost_usd` (daily) | > $50 (pre-5k MAU) | > $200 | ~$0.09 × MAU / 30 | Absolute floor before volume scaling kicks in |
| **ElevenLabs** | `characters` | > 1.5× 7d avg | > 3× 7d avg | Pro-MAU × adoption × 1,200 chars/session × sessions/day | Voice adoption spike is the primary driver |
| **ElevenLabs** | `cost_usd` (daily) | > $30 (pre-5k MAU) | > $100 | ~$0.22 × Pro-MAU / 30 | |
| **Cloudflare** | `workers_requests` | > 2× 7d avg | > 5× 7d avg | MAU × 15 req/session × sessions/day | Traffic spike or retry storm |
| **Supabase** | `db_size_gb` | > 6 GB | > 7.5 GB | Grows ~15 MB/user/month (§COST_MODEL §2.1) | Step-function cost at 8 GB (Team plan) |
| **Supabase** | `bandwidth_gb` (monthly) | > 200 GB | > 240 GB | 80 MB/Pro-user/month | Monitored monthly, not daily |

**Threshold adjustment policy:** Thresholds are reviewed at each MAU milestone (1k, 5k, 10k, 50k). The devops-lead updates the Worker's `COST_THRESHOLDS` environment variable (stored in Cloudflare Secrets) at each milestone. Do not hard-code thresholds in the Worker source — they must be externalised for hot-update without redeploy.

---

### 17.4 Daily Cost Aggregation Worker

```typescript
// workers/cost-monitor/index.ts
// Cron trigger: "0 6 * * *"  (06:00 UTC daily)
// See: wrangler.toml [triggers] crons = ["0 6 * * *"]

interface Env {
  ANTHROPIC_ADMIN_KEY:   string;  // Anthropic workspace admin key (not the API key)
  ELEVENLABS_API_KEY:    string;
  CF_ANALYTICS_TOKEN:    string;  // Cloudflare Analytics Read token (account-level)
  CF_ACCOUNT_ID:         string;
  SUPABASE_SERVICE_KEY:  string;
  SUPABASE_URL:          string;
  PAGERDUTY_COST_KEY:    string;  // Separate routing key from infra alerts
  SLACK_METRICS_WEBHOOK: string;
  COST_THRESHOLDS:       string;  // JSON blob: { "anthropic.input_tokens.p1_ratio": 1.5, ... }
  EL_CHAR_KV:            KVNamespace;  // Stores previous-day ElevenLabs cumulative char count
}

interface DailyCost {
  service:      string;
  metric_name:  string;
  metric_value: number;
  cost_usd:     number;
}

export default {
  async scheduled(_event: ScheduledEvent, env: Env, _ctx: ExecutionContext): Promise<void> {
    const yesterday = new Date();
    yesterday.setUTCDate(yesterday.getUTCDate() - 1);
    const date = yesterday.toISOString().split('T')[0]; // "YYYY-MM-DD"

    const [anthropic, elevenLabs, cloudflare] = await Promise.allSettled([
      fetchAnthropicUsage(date, env),
      fetchElevenLabsUsage(date, env),
      fetchCloudflareUsage(date, env),
    ]);

    const rows: DailyCost[] = [
      ...settled(anthropic),
      ...settled(elevenLabs),
      ...settled(cloudflare),
    ];

    // Upsert to Supabase cost_daily
    await upsertCostRows(date, rows, env);

    // Load 7-day rolling averages from cost_daily
    const baselines = await load7DayBaselines(date, env);

    // Detect anomalies
    const thresholds = JSON.parse(env.COST_THRESHOLDS);
    const anomalies = detectAnomalies(rows, baselines, thresholds);

    if (anomalies.length > 0) {
      await insertAnomalies(date, anomalies, env);
      await Promise.all([
        alertPagerDuty(anomalies, env),
        postSlack(date, rows, anomalies, env),
      ]);
    } else {
      await postSlack(date, rows, [], env);
    }
  },
};

// ── Anthropic ─────────────────────────────────────────────────────────────────
async function fetchAnthropicUsage(date: string, env: Env): Promise<DailyCost[]> {
  const start = `${date}T00:00:00Z`;
  const end   = `${date}T23:59:59Z`;
  const res = await fetch(
    `https://api.anthropic.com/v1/usage?start_time=${encodeURIComponent(start)}&end_time=${encodeURIComponent(end)}`,
    { headers: { 'x-api-key': env.ANTHROPIC_ADMIN_KEY, 'anthropic-version': '2023-06-01' } }
  );
  if (!res.ok) throw new Error(`Anthropic usage API ${res.status}: ${await res.text()}`);
  const body: { usage: { input_tokens: number; output_tokens: number } } = await res.json();
  const INPUT_RATE  = 3.00 / 1_000_000;
  const OUTPUT_RATE = 15.00 / 1_000_000;
  return [
    { service: 'anthropic', metric_name: 'input_tokens',  metric_value: body.usage.input_tokens,  cost_usd: body.usage.input_tokens  * INPUT_RATE  },
    { service: 'anthropic', metric_name: 'output_tokens', metric_value: body.usage.output_tokens, cost_usd: body.usage.output_tokens * OUTPUT_RATE },
  ];
}

// ── ElevenLabs ────────────────────────────────────────────────────────────────
// ElevenLabs does not expose a per-day endpoint; it returns the billing-cycle cumulative total.
// We store yesterday's cumulative in KV and subtract to derive a daily delta.
async function fetchElevenLabsUsage(date: string, env: Env): Promise<DailyCost[]> {
  const res = await fetch('https://api.elevenlabs.io/v1/user', {
    headers: { 'xi-api-key': env.ELEVENLABS_API_KEY },
  });
  if (!res.ok) throw new Error(`ElevenLabs API ${res.status}: ${await res.text()}`);
  const body: { subscription: { character_count: number } } = await res.json();
  const currentTotal = body.subscription.character_count;

  const prevKey = `el_chars_${date}`;
  const prevRaw = await env.EL_CHAR_KV.get(prevKey);
  const prevTotal = prevRaw ? parseInt(prevRaw, 10) : currentTotal; // fallback: 0 delta on first run

  // Store today's total for tomorrow's delta
  const tomorrowKey = `el_chars_${advanceDate(date, 1)}`;
  await env.EL_CHAR_KV.put(tomorrowKey, String(currentTotal), { expirationTtl: 60 * 60 * 24 * 7 }); // 7d TTL

  const dailyChars = Math.max(0, currentTotal - prevTotal);
  const CHAR_RATE = 0.30 / 1_000;
  return [{ service: 'elevenlabs', metric_name: 'characters', metric_value: dailyChars, cost_usd: dailyChars * CHAR_RATE }];
}

// ── Cloudflare ────────────────────────────────────────────────────────────────
async function fetchCloudflareUsage(date: string, env: Env): Promise<DailyCost[]> {
  const query = `{
    viewer { accounts(filter: { accountTag: "${env.CF_ACCOUNT_ID}" }) {
      workersInvocationsAdaptive(
        limit: 1
        filter: { date_gt: "${date}", date_lt: "${advanceDate(date, 1)}" }
        orderBy: [date_ASC]
      ) { sum { requests } }
    }}
  }`;
  const res = await fetch('https://api.cloudflare.com/client/v4/graphql', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${env.CF_ANALYTICS_TOKEN}`, 'Content-Type': 'application/json' },
    body: JSON.stringify({ query }),
  });
  if (!res.ok) throw new Error(`CF GraphQL ${res.status}`);
  const body: { data: { viewer: { accounts: Array<{ workersInvocationsAdaptive: Array<{ sum: { requests: number } }> }> } } } = await res.json();
  const requests = body.data.viewer.accounts[0]?.workersInvocationsAdaptive[0]?.sum?.requests ?? 0;
  // Workers Paid: $0.50 / 1M requests above 100k/day free threshold
  const billableRequests = Math.max(0, requests - 100_000);
  const cost = (billableRequests / 1_000_000) * 0.50;
  return [{ service: 'cloudflare', metric_name: 'workers_requests', metric_value: requests, cost_usd: cost }];
}

// ── Anomaly detection ─────────────────────────────────────────────────────────
interface Anomaly {
  service: string; metric_name: string;
  observed: number; baseline_7d: number; ratio: number; severity: 'P0' | 'P1';
}

function detectAnomalies(
  rows: DailyCost[],
  baselines: Record<string, number>,
  thresholds: Record<string, number>
): Anomaly[] {
  const out: Anomaly[] = [];
  for (const row of rows) {
    const key = `${row.service}.${row.metric_name}`;
    const baseline = baselines[key];
    if (!baseline || baseline === 0) continue;
    const ratio = row.metric_value / baseline;
    const p0 = thresholds[`${key}.p0_ratio`] ?? 3.0;
    const p1 = thresholds[`${key}.p1_ratio`] ?? 1.5;
    if (ratio >= p0) out.push({ service: row.service, metric_name: row.metric_name, observed: row.metric_value, baseline_7d: baseline, ratio, severity: 'P0' });
    else if (ratio >= p1) out.push({ service: row.service, metric_name: row.metric_name, observed: row.metric_value, baseline_7d: baseline, ratio, severity: 'P1' });
  }
  return out;
}

// ── Helpers ───────────────────────────────────────────────────────────────────
function settled<T>(result: PromiseSettledResult<T[]>): T[] {
  return result.status === 'fulfilled' ? result.value : [];
}

function advanceDate(date: string, days: number): string {
  const d = new Date(`${date}T00:00:00Z`);
  d.setUTCDate(d.getUTCDate() + days);
  return d.toISOString().split('T')[0];
}
```

**Security notes:**
- `ANTHROPIC_ADMIN_KEY` is a workspace-level admin key (not the same as the inference API key). It has read-only access to usage data. Stored in Cloudflare Workers Secrets — never in `wrangler.toml` or git.
- The Worker does not handle user data. It reads only billing API endpoints that return aggregated counts. No PII or health data flows through this Worker.
- The Supabase connection uses the service-role key in a dedicated Worker secret binding, scoped to write-only access on `cost_daily` and `cost_anomalies` via a custom Postgres role:

```sql
-- Role for the cost monitor Worker (write-only, no SELECT except rolling avg query)
CREATE ROLE form_cost_monitor NOINHERIT;
GRANT INSERT, UPDATE ON cost_daily     TO form_cost_monitor;
GRANT INSERT          ON cost_anomalies TO form_cost_monitor;
GRANT SELECT ON cost_daily TO form_cost_monitor;  -- for the 7-day baseline query only
-- No RLS bypass — cost_daily has no tenant_id RLS (it's ops-internal, not user-data)
```

---

### 17.5 Per-Tenant Cost Attribution (Enterprise)

Enterprise seats are billed at a fixed price ($6–12/seat/month, per COST_MODEL.md §8), but per-tenant COGS is tracked internally for margin accounting and early anomaly detection. A tenant whose daily COGS is 3× its 30-day average may have an integration bug, a test environment pointed at production, or unusually high activation in a short window — all worth investigating before the month-end billing reconciliation.

```sql
-- Materialized view: per-tenant daily COGS from coaching session telemetry.
-- Token counts are logged server-side by the Cloudflare Worker on every Anthropic call.
-- Voice chars are logged on every ElevenLabs call that completes successfully.
CREATE MATERIALIZED VIEW cost_daily_per_tenant AS
SELECT
  s.tenant_id,
  DATE_TRUNC('day', s.created_at AT TIME ZONE 'UTC')::DATE        AS date,
  COUNT(*)                                                          AS sessions,
  COALESCE(SUM(s.anthropic_input_tokens),  0)                      AS input_tokens,
  COALESCE(SUM(s.anthropic_output_tokens), 0)                      AS output_tokens,
  COALESCE(SUM(s.elevenlabs_chars),        0)                      AS voice_chars,
  COALESCE(SUM(s.anthropic_input_tokens),  0) * 3.00  / 1e6       AS anthropic_input_usd,
  COALESCE(SUM(s.anthropic_output_tokens), 0) * 15.00 / 1e6       AS anthropic_output_usd,
  COALESCE(SUM(s.elevenlabs_chars),        0) * 0.30  / 1e3       AS elevenlabs_usd,
  (  COALESCE(SUM(s.anthropic_input_tokens),  0) * 3.00  / 1e6
   + COALESCE(SUM(s.anthropic_output_tokens), 0) * 15.00 / 1e6
   + COALESCE(SUM(s.elevenlabs_chars),        0) * 0.30  / 1e3 )  AS total_cogs_usd
FROM coaching_sessions s
WHERE s.tenant_id IS NOT NULL
GROUP BY s.tenant_id, DATE_TRUNC('day', s.created_at AT TIME ZONE 'UTC')::DATE;

-- Refresh 60 minutes after cost_daily is populated (07:00 UTC daily)
SELECT cron.schedule(
  'refresh-cost-tenant',
  '0 7 * * *',
  $$REFRESH MATERIALIZED VIEW CONCURRENTLY cost_daily_per_tenant$$
);
```

Per-tenant anomaly thresholds (checked by a Metabase alert rule, not the daily Worker):

| Signal | Threshold | Response |
|---|---|---|
| Tenant COGS > 2× 30-day avg for 3 consecutive days | Investigate | devops-lead checks session logs; customer-success notified if no technical explanation found |
| Single-day COGS > contracted monthly revenue (any tenant) | P0 | Margin-negative situation; devops-lead + customer-success + founder notified immediately |
| Single-day COGS spike > 5× tenant 30-day avg | P1 | Integration bug suspected; review within 4 hours |
| COGS per active seat > $1.50/day (≈ $45/seat/month, 4× the $0.34 model) | P1 | Unusual per-seat usage; request explanation from tenant admin |

**Privacy constraint:** The `cost_daily_per_tenant` view is visible only to FORM operations staff (`form_admin` Postgres role and the Metabase service account). It is **not** exposed to the enterprise admin dashboard. The admin dashboard shows seat utilisation and engagement trends — not session token counts or inferred COGS. Surfacing per-session costs to tenant admins would reveal operational cost structure and coaching interaction depth, both of which are Restricted classification (§5 DATA_MODEL.md).

---

### 17.6 Cost Dashboards (Metabase — Internal)

#### 17.6.1 Infra Cost Overview

| Panel | Metric | Query source | Refresh |
|---|---|---|---|
| Daily total COGS by service | Stacked bar: `SUM(cost_usd) GROUP BY service, date` | `cost_daily` | Daily |
| 7-day rolling COGS trend | Line: 7-period moving average | `cost_daily` window function | Daily |
| COGS per Pro MAU | `total_cogs / active_pro_mau` | `cost_daily` JOIN `dim_users` | Daily |
| Anthropic input vs. output split | Pie: `cost_usd GROUP BY metric_name WHERE service='anthropic'` | `cost_daily` | Daily |
| ElevenLabs as % of total COGS | Gauge: `el_cost / total_cost` | `cost_daily` | Daily |
| Month-to-date run rate vs. model | Progress bar: `SUM(cost_usd WHERE month=current) / est_monthly_budget` | `cost_daily JOIN financials` | Daily |
| Variance from model (%) | `AVG(variance_pct) GROUP BY service` | `cost_daily` | Daily |

#### 17.6.2 Enterprise Tenant Cost Panel (Restricted — devops-lead + founder only)

| Panel | Metric | Notes |
|---|---|---|
| Top 10 tenants by COGS (current month) | Ranked bar chart | Sorted by `total_cogs_usd DESC` |
| COGS vs. contracted revenue scatter | Scatter: x=contracted_mrr, y=tenant_cogs | Points above the y=x line are margin-negative |
| COGS per active seat by tenant | Table: `total_cogs_usd / active_seats` | Outliers (> $1.50/seat/day) highlighted in orange |
| Tenant COGS 30-day spark lines | Small multiples | One spark per enterprise tenant; sorted by MRR |

#### 17.6.3 Cost Anomaly History

A dedicated Metabase page sourced from `cost_anomalies`:

| Panel | Metric |
|---|---|
| Open anomalies | `WHERE resolved_at IS NULL ORDER BY detected_at DESC` |
| Anomalies resolved in last 30 days | With `resolution` text visible to the reviewer |
| P0 anomaly frequency (trend chart) | Should read 0 at steady state |

---

### 17.7 Alerting Integration

Cost anomaly alerts share the PagerDuty and Slack infrastructure with service degradation alerts but route to a **separate PagerDuty service** (`FORM Cost`) so they do not page the on-call engineer at 02:00 UTC for a non-user-affecting cost spike.

**Urgency routing:**

| Severity | PagerDuty service | PagerDuty urgency | On-call response expectation |
|---|---|---|---|
| P0 cost anomaly | `FORM Cost` | Low (email only, not phone) | Investigate next business day unless cost spike is user-impacting |
| P1 cost anomaly | `FORM Cost` | Low | Acknowledge within 24 hours |
| P0 *service* anomaly that also happens to be a cost anomaly (e.g., runaway retry loop) | `FORM Production` | High | Immediate — service impact overrides the cost framing |

**PagerDuty event payload:**

```json
{
  "routing_key": "<PAGERDUTY_COST_ROUTING_KEY>",
  "event_action": "trigger",
  "dedup_key": "cost-anomaly-2026-05-20-anthropic-input_tokens",
  "payload": {
    "summary": "[COST P0] anthropic/input_tokens 4.2× 7d avg — $247 vs $59 expected",
    "severity": "critical",
    "source": "cost-monitor-worker",
    "timestamp": "2026-05-20T06:05:00Z",
    "custom_details": {
      "service":        "anthropic",
      "metric":         "input_tokens",
      "observed":       8400000,
      "baseline_7d":    2000000,
      "ratio":          4.2,
      "cost_usd":       247.00,
      "expected_usd":   59.00,
      "runbook":        "docs/ENGINEERING_RUNBOOK.md#cost-spike-anthropic",
      "dashboard_url":  "https://metabase.form.coach/question/cost-overview"
    }
  }
}
```

**Slack daily digest format** (06:10 UTC to `#metrics`, sent whether or not anomalies are present):

```
📊 FORM Daily Cost — 2026-05-20
───────────────────────────────────────
Anthropic     $12.40   (↑ 8% vs 7d avg)
ElevenLabs     $8.20   (↑ 3% vs 7d avg)
Cloudflare     $0.12   (→ nominal)
Supabase       $0.00   (plan cost; no marginal billing today)
───────────────────────────────────────
Total COGS    $20.72   today
Per-Pro-MAU    $0.49   (model: $0.50)
MTD total    $392.60   (on pace for $589/mo)
───────────────────────────────────────
No anomalies detected.
```

When anomalies are present, the anomaly section replaces the final line:

```
⚠️ ANOMALY P1: ElevenLabs characters 1.7× 7d avg (4.8M vs 2.8M)
   → Investigate: rising voice adoption or ElevenLabs retry storm?
   PD: https://form.pagerduty.com/incidents/QXXXXXXX
```

---

### 17.8 SOC 2 Evidence Contribution

| SOC 2 Criterion | How Cost Monitoring Satisfies It |
|---|---|
| **CC7.2 — System monitoring** | The daily cost aggregation Worker provides continuous automated monitoring evidence for the inference and voice-synthesis subsystems. An anomalous cost spike is a leading indicator of misconfiguration, unauthorized API access, or an upstream provider incident — all conditions CC7.2 requires FORM to detect. |
| **CC7.3 — Anomaly evaluation** | The threshold table in §17.3 defines explicit anomaly criteria; `cost_anomalies` provides an immutable log of every detection event; PagerDuty tickets created for each anomaly satisfy the "evaluate and respond" requirement. |
| **CC5.2 — Technology controls** | Per-tenant cost tracking (§17.5) enforces that each enterprise tenant's resource consumption is bounded and accounted for — a technology control analogous to per-tenant API rate limiting. |
| **A1.2 — Availability capacity** | Approaching Anthropic rate limits first manifests as a cost spike before it becomes a user-facing availability event. Cost monitoring provides early-warning capacity signal aligned with the A1.2 capacity planning requirement. |

**Auditor evidence artifacts:** The `cost_daily` table is exported to CSV monthly (`compliance/evidence/cost-daily-YYYY-MM.csv`) by a scheduled Metabase export and committed to the compliance store. These are filed alongside the synthetic monitoring uptime reports (§16.9) as part of the CC7.2 evidence package. The `cost_anomalies` table is exported to CSV quarterly.

---

### 17.9 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Write `migrations/YYYYMMDD_cost_daily.sql` (both tables) | platform-engineer | P0 | M3 |
| Implement `workers/cost-monitor/index.ts` (§17.4) | devops-lead | P0 | M3 |
| Add `[triggers] crons = ["0 6 * * *"]` to `wrangler.toml` for cost-monitor Worker | devops-lead | P0 | M3 |
| Store `ANTHROPIC_ADMIN_KEY`, `ELEVENLABS_API_KEY`, `CF_ANALYTICS_TOKEN` in Workers Secrets | security-engineer | P0 | M3 |
| Create `EL_CHAR_KV` KV namespace and bind it to cost-monitor Worker | devops-lead | P0 | M3 |
| Create `COST_THRESHOLDS` secret (JSON blob, §17.3 initial values) | devops-lead | P0 | M3 |
| Create Postgres role `form_cost_monitor` with scoped grants (§17.4 security note) | platform-engineer | P0 | M3 |
| Create PagerDuty service "FORM Cost" with low-urgency routing key | devops-lead | P1 | M3 |
| Configure Slack `#metrics` incoming webhook for daily digest | devops-lead | P1 | M3 |
| Add Supabase `pg_cron` job `refresh-cost-tenant` at 07:00 UTC | platform-engineer | P1 | M3 |
| Build Metabase "Infra Cost Overview" dashboard (§17.6.1, six panels) | data-engineer | P1 | M4 |
| Build Metabase "Enterprise Tenant Cost" panel (§17.6.2, restricted access) | data-engineer | P1 | M4 |
| Build Metabase "Cost Anomaly History" page (§17.6.3) | data-engineer | P1 | M4 |
| Seed `cost_daily.est_cost_usd` from COST_MODEL.md §13.1 projections (per-MAU model) | data-engineer | P2 | M4 |
| Automate monthly CSV export to `compliance/evidence/cost-daily-YYYY-MM.csv` | devops-lead | P2 | M4 |
| Improve ElevenLabs daily delta accuracy: switch from KV delta to ElevenLabs history endpoint when available | platform-engineer | P2 | M5 |

---

---

## 18. CV Pose Estimation Model Observability

> **Privacy-first constraint:** The CV pipeline processes GDPR Art. 9 biometric data (derived from video analysis). All observability signals in this section are designed so that no individual's pose data, form score, or movement pattern can be reconstructed from log or metric data. Raw frames never leave the device; only aggregate and anonymised signals flow to the observability stack. See §18.8 for the full privacy constraint matrix.

### 18.1 Purpose and Scope

FORM's computer vision (CV) pose estimation pipeline is the product's primary technical differentiator. It runs entirely on-device — raw video frames are never transmitted to FORM's backend or to any third party (`docs/DATA_MODEL.md` §15.1). This design creates observability constraints: there is no server-side frame stream to monitor. Instead, observability is distributed across three layers:

1. **On-device SDK telemetry** — non-sensitive metrics (latency, frame counts, cold-start time) collected during the session and uploaded at session completion. No pose keypoints or form scores included.
2. **Server-side session records** — the `cv_sessions` table stores aggregate quality signals (`status`, `avg_keypoint_confidence`, `frames_below_threshold`) without the encrypted biometric payload (`keypoints_enc`) being accessible to the observability stack.
3. **Aggregate fleet views** — cross-session aggregation by `model_name` × `model_version` × `tenant_id`, used for model health monitoring and rollout progress tracking.

**In scope:** Session completion rates, on-device latency telemetry, model version distribution, confidence degradation detection, `tracking_lost` spike detection, model rollout anomaly alerting, form defect prevalence at fleet level.

**Out of scope:** Per-user form score breakdowns, individual keypoint data, per-user movement quality, any signal that would allow re-identification of an individual user's health status. These constraints are enforced at the schema level.

Cross-references: `docs/DATA_MODEL.md` §15 (CV schema, RLS, `tenant_cv_summary`), `docs/INCIDENT_RESPONSE.md` R-10 (AI Coach Safety Incident — blast-radius scoping by `model_version`), `docs/SOC2_READINESS.md` §37 (PI1 Processing Integrity).

---

### 18.2 RED Signals for the CV Pipeline

Applying the RED methodology (§0) to the CV pipeline:

| Signal | Rate (R) | Errors (E) | Duration (D) |
|---|---|---|---|
| **CV session upload (Workers)** | `cv_sessions` inserts per hour, by `platform` | `status = 'tracking_lost'` rate; `status = 'partial'` rate | P95 of `cv_session_upload_duration_ms` Worker span |
| **On-device inference (iOS)** | CV sessions per day on iOS | Sessions with `avg_keypoint_confidence < 0.65` (frame noise floor) | `inference_latency_p95_ms` from `cv_sdk_telemetry` |
| **On-device inference (Android)** | CV sessions per day on Android | Same confidence threshold; separate model path (ML Kit vs Apple Vision) | `inference_latency_p95_ms` (Android) |
| **Model rollout** | % of fleet sessions per `model_name` × `model_version` | New version `pct_tracking_lost` > 2× baseline | Not latency-sensitive; omit |

**Interpretation note:** A rise in `tracking_lost` at constant session rate is an **Error** spike in RED terms. A drop in total sessions at constant `tracking_lost` could indicate a **Rate** regression (camera feature abandoned during cold start) vs a **Duration** regression (cold start too slow). Distinguish using `cold_start_ms` P95 from `cv_sdk_telemetry` before paging.

---

### 18.3 On-Device SDK Telemetry Schema

At the end of each CV session the mobile app transmits a non-sensitive telemetry payload to the Cloudflare Worker at `POST /v1/cv/session-telemetry`. Authenticated with the user's Bearer JWT. Writes to `cv_sdk_telemetry`.

**Classification: INTERNAL — non-sensitive aggregate. No GDPR Art. 9 content.**

```sql
-- Telemetry emitted by the mobile SDK at session end.
-- No pose keypoints, form scores, or movement descriptions.
-- Retention: 90 days (aligned with §8 metrics tier).
CREATE TABLE cv_sdk_telemetry (
  id                       UUID         PRIMARY KEY DEFAULT gen_random_uuid(),
  cv_session_id            UUID         NOT NULL REFERENCES cv_sessions(id) ON DELETE CASCADE,
  tenant_id                UUID         NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,

  -- Device context (non-identifying; used for model fleet health analysis)
  platform                 TEXT         NOT NULL CHECK (platform IN ('ios', 'android')),
  os_version               TEXT         NOT NULL,  -- "17.4.1" (iOS) or "14" (Android API level string)
  device_tier              TEXT         NOT NULL   -- "high" | "mid" | "low" (inferred from device benchmark)
                             CHECK (device_tier IN ('high', 'mid', 'low')),

  -- Model selection (mirrors cv_sessions columns)
  model_name               TEXT         NOT NULL,
  model_version            TEXT         NOT NULL,

  -- On-device latency (milliseconds)
  cold_start_ms            INTEGER,     -- camera init + model warm-up; SLO P95 < 800 ms (TECHNICAL.md §2)
  inference_latency_p50_ms INTEGER,     -- median per-frame inference; target ≈ 20 ms (Apple Vision)
  inference_latency_p95_ms INTEGER,     -- 95th percentile; alert threshold: > 50 ms iOS, > 80 ms Android (§18.7)
  frame_count              INTEGER,     -- total frames captured by camera
  frames_dropped           INTEGER,     -- dropped before inference (buffer overflow / CPU saturation)
  frames_skipped           INTEGER,     -- intentionally skipped (every-N-th frame strategy)

  -- Tracking stability
  kalman_resets            SMALLINT,    -- number of Kalman filter resets during session (tracking disruptions)

  -- Memory pressure (iOS only; null on Android)
  memory_warning_count     SMALLINT,    -- UIApplicationDidReceiveMemoryWarningNotification count

  created_at               TIMESTAMPTZ  NOT NULL DEFAULT now()
);

CREATE INDEX idx_cv_sdk_telemetry_session
  ON cv_sdk_telemetry (cv_session_id);
CREATE INDEX idx_cv_sdk_telemetry_model
  ON cv_sdk_telemetry (model_name, model_version, platform);
CREATE INDEX idx_cv_sdk_telemetry_created_at
  ON cv_sdk_telemetry (created_at);
```

**Privacy note:** `platform`, `os_version`, and `device_tier` are non-identifying at this granularity. FORM does not store IDFA or Android Advertising ID (`docs/DATA_MODEL.md` §13.1). `os_version` (major.minor) does not constitute personal data under GDPR.

---

### 18.4 Server-Side Aggregate Signals (`cv_sessions`)

The `cv_sessions` table (`docs/DATA_MODEL.md` §15.2) stores non-encrypted aggregate outputs per session. These columns are safe for fleet-level observability without decrypting `keypoints_enc`:

| Column | Observability use |
|---|---|
| `status` (`completed` / `partial` / `tracking_lost` / `active`) | Session completion rate; `tracking_lost` spike detection |
| `avg_keypoint_confidence` (0.000–1.000) | Confidence distribution; P50/P25 degradation alerting |
| `frames_analysed` | Session length; cross-reference with SDK `frame_count` |
| `frames_below_threshold` | Poor-confidence frame ratio (threshold: 0.65 per `REP_COUNT_CONFIDENCE_MIN`) |
| `model_name` | Model distribution across fleet |
| `model_version` | Rollout progress; per-version quality comparison |
| `cv_flags` (JSONB) | Defect prevalence at fleet level — never per-user in dashboards |
| `created_at` | Session volume time-series |

**Columns excluded from observability:** `user_id` (never grouped by user), `keypoints_enc` (encrypted biometric payload — observability stack has no decryption key), individual `form_score` per user.

#### 18.4.1 Fleet-Level Observability View

```sql
-- Internal observability view — aggregates cv_sessions at model × platform × day level.
-- No user_id, no per-user form_score, no keypoints.
-- Access: form_system, form_admin. devops-lead queries via Metabase (§18.9).
-- Refresh: nightly pg_cron at 04:00 UTC (staggered after tenant_cv_summary at 03:05 UTC).
CREATE MATERIALIZED VIEW cv_session_fleet_stats AS
SELECT
  DATE_TRUNC('day', created_at)                                         AS day,
  model_name,
  model_version,
  tenant_id,                                -- for per-tenant anomaly detection (AL-CV-08); never returned to tenant admin

  COUNT(*)                                                              AS total_sessions,
  COUNT(*) FILTER (WHERE status = 'completed')                          AS completed_sessions,
  COUNT(*) FILTER (WHERE status = 'partial')                            AS partial_sessions,
  COUNT(*) FILTER (WHERE status = 'tracking_lost')                      AS tracking_lost_sessions,

  -- Completion rate (primary SLO signal; §18.5)
  ROUND(100.0 * COUNT(*) FILTER (WHERE status = 'completed')
    / NULLIF(COUNT(*), 0), 1)                                           AS pct_completed,
  ROUND(100.0 * COUNT(*) FILTER (WHERE status = 'tracking_lost')
    / NULLIF(COUNT(*), 0), 1)                                           AS pct_tracking_lost,

  -- Confidence distribution (P50 and P25 as floor degradation proxies)
  PERCENTILE_CONT(0.50) WITHIN GROUP
    (ORDER BY avg_keypoint_confidence)                                  AS confidence_p50,
  PERCENTILE_CONT(0.25) WITHIN GROUP
    (ORDER BY avg_keypoint_confidence)                                  AS confidence_p25,

  -- Frame quality
  ROUND(AVG(
    CASE WHEN frames_analysed > 0
      THEN frames_below_threshold::NUMERIC / frames_analysed
      ELSE NULL
    END
  ), 3)                                                                 AS avg_below_threshold_ratio,

  -- Form defect fleet prevalence (directional; no per-user attribution)
  ROUND(100.0 * COUNT(*) FILTER (
    WHERE (cv_flags->>'knee_cave')::int > 0
  ) / NULLIF(COUNT(*), 0), 1)                                           AS pct_knee_cave,
  ROUND(100.0 * COUNT(*) FILTER (
    WHERE (cv_flags->>'back_arch')::int > 0
  ) / NULLIF(COUNT(*), 0), 1)                                           AS pct_back_arch,
  ROUND(100.0 * COUNT(*) FILTER (
    WHERE (cv_flags->>'depth_short')::int > 0
  ) / NULLIF(COUNT(*), 0), 1)                                           AS pct_depth_short

FROM cv_sessions
WHERE deleted_at IS NULL
GROUP BY
  DATE_TRUNC('day', created_at),
  model_name,
  model_version,
  tenant_id;

-- SELECT cron.schedule('refresh-cv-fleet-stats', '0 4 * * *',
--   $$REFRESH MATERIALIZED VIEW CONCURRENTLY cv_session_fleet_stats$$);
```

---

### 18.5 SLOs for the CV Pipeline

These SLOs supplement the platform SLO table in §2.1. Minimum session count for all SLO evaluations: **≥ 50 sessions** in the rolling window; below this threshold the alert is suppressed and flagged as insufficient data.

| SLI Name | Measurement | SLO Target | Alerting Threshold | Window | Owner |
|---|---|---|---|---|---|
| **CV session completion rate** | `pct_completed` in `cv_session_fleet_stats` (global fleet) | ≥ 85% | < 80% in 1-hour rolling window | Rolling 7 days | ml-engineer |
| **CV tracking_lost rate** | `pct_tracking_lost` in `cv_session_fleet_stats` (global fleet) | ≤ 8% | > 15% in 1-hour rolling window | Rolling 7 days | ml-engineer |
| **On-device inference P95 (iOS)** | `inference_latency_p95_ms` from `cv_sdk_telemetry` (`platform = 'ios'`) | < 30 ms | > 50 ms P95 in 6-hour window | Rolling 7 days | platform-engineer |
| **On-device inference P95 (Android)** | `inference_latency_p95_ms` from `cv_sdk_telemetry` (`platform = 'android'`) | < 50 ms | > 80 ms P95 in 6-hour window | Rolling 7 days | platform-engineer |
| **Camera cold start P95 (iOS)** | `cold_start_ms` from `cv_sdk_telemetry` (`platform = 'ios'`) | < 800 ms | > 1,200 ms P95 in 6-hour window | Rolling 7 days | platform-engineer |
| **Avg keypoint confidence P50** | `confidence_p50` in `cv_session_fleet_stats` (global fleet) | ≥ 0.80 | < 0.70 in 1-hour rolling window | Rolling 7 days | ml-engineer |
| **High-confidence session rate** | % of sessions with `avg_keypoint_confidence ≥ 0.90` | ≥ 70% | < 55% in 6-hour rolling window | Rolling 7 days | ml-engineer |
| **Frame below-threshold ratio** | `avg_below_threshold_ratio` in `cv_session_fleet_stats` | ≤ 0.12 | > 0.20 in 1-hour rolling window | Rolling 7 days | ml-engineer |

**Rationale for 85% completion target:** `docs/TECHNICAL.md` §2 reports 92–97% keypoint confidence in good-light conditions. Enterprise environments (private offices, inconsistent overhead lighting) degrade this. 85% is the floor below which user experience is materially impacted. The 92%+ rate in controlled conditions is a stretch target tracked separately in the "CV Model Health" Metabase panel (§18.9).

---

### 18.6 Model Version Registry and Rollout Monitoring

Model versions are identified in `cv_sessions` by `(model_name, model_version)`. Four model variants:

| `model_name` | Keypoints | Target device tier | iOS framework | Android framework |
|---|---|---|---|---|
| `blazepose_lite` | 33 (x, y, z, visibility) | Mid / Low | Apple Vision `VNDetectHumanBodyPoseRequest` | ML Kit Pose Detection Lite |
| `blazepose_full` | 33 | High | Apple Vision | ML Kit Pose Detection Full |
| `movenet_lightning` | 17 (x, y, z, visibility) | Mid / Low | CoreML (TFLite conversion) | TFLite |
| `movenet_thunder` | 17 | High | CoreML | TFLite |

**Rollout monitoring:** When a new `model_version` is deployed via EAS OTA, the version distribution in `cv_session_fleet_stats` tracks rollout progress. A new version that reaches > 10% of fleet with `pct_tracking_lost` > 2× the baseline version's `pct_tracking_lost` triggers AL-CV-03 (P1; §18.7).

```sql
-- Model version rollout progress query — run by ml-engineer post-deploy
SELECT
  model_name,
  model_version,
  SUM(total_sessions)                                           AS sessions_last_7d,
  ROUND(100.0 * SUM(total_sessions)
    / SUM(SUM(total_sessions)) OVER (PARTITION BY model_name), 1) AS pct_of_fleet,
  ROUND(AVG(pct_completed), 1)                                  AS avg_pct_completed,
  ROUND(AVG(pct_tracking_lost), 1)                              AS avg_pct_tracking_lost,
  ROUND(AVG(confidence_p50)::NUMERIC, 3)                        AS avg_confidence_p50
FROM cv_session_fleet_stats
WHERE day >= CURRENT_DATE - 7
GROUP BY model_name, model_version
ORDER BY model_name, sessions_last_7d DESC;
```

**Blast-radius scoping (INCIDENT_RESPONSE.md R-10):** The `idx_cv_sessions_model_version` index (`docs/DATA_MODEL.md` §15.2) allows R-10 responders to scope affected sessions by model version without a full table scan:

```sql
-- R-10 blast-radius: sessions affected by a specific model version
SELECT
  COUNT(*)          AS affected_sessions,
  MIN(started_at)   AS first_affected,
  MAX(started_at)   AS last_affected
FROM cv_sessions
WHERE model_name    = 'blazepose_full'
  AND model_version = '2.1.0'
  AND deleted_at IS NULL;
```

---

### 18.7 Alerting Rules

These rules supplement the global alerting table in §6.

| Alert ID | Condition | Severity | Action | Investigation path |
|---|---|---|---|---|
| **AL-CV-01** | `pct_tracking_lost` > 15% in any 1-hour rolling window (≥ 50 sessions, global fleet) | P1 | PagerDuty → ml-engineer + platform-engineer on-call | Check lighting-correlated geography (new rollout region? season/DST?); Sentry for camera-related errors; recent EAS deploy version distribution |
| **AL-CV-02** | `confidence_p50` < 0.70 in 1-hour rolling window (≥ 50 sessions) | P1 | PagerDuty → ml-engineer | Cross-reference with Sentry crash rate; check if platform-specific (iOS vs Android); recent model version distribution shift |
| **AL-CV-03** | New `model_version` reaches > 10% of fleet AND `pct_tracking_lost` > 2× baseline version's rate | P1 | PagerDuty → ml-engineer; consider feature-flag rollback | Run blast-radius query (§18.6); if > 25% of fleet affected, escalate to P0 + initiate rollback |
| **AL-CV-04** | `inference_latency_p95_ms` (iOS) > 50 ms sustained > 6 hours | P2 | Slack #platform-alerts → platform-engineer | Check Sentry ANR reports; check `device_tier` distribution; Kalman filter loop CPU regression |
| **AL-CV-05** | `cold_start_ms` (iOS) P95 > 1,200 ms in 6-hour window | P2 | Slack #platform-alerts → platform-engineer | AVFoundation initialization regression; check recent EAS deploy for camera permission flow changes |
| **AL-CV-06** | `frames_dropped` P50 > 5% of `frame_count` in `cv_sdk_telemetry` | P2 | Slack #platform-alerts → platform-engineer | Device CPU contention; correlate with `memory_warning_count`; check frame-rate configuration for mid/low device tier |
| **AL-CV-07** | `kalman_resets` P95 > 3 per session in `cv_sdk_telemetry` | P2 | Slack #platform-alerts → ml-engineer | Frequent tracking disruption; correlate with `pct_tracking_lost` and `pct_partial`; potential Kalman filter hyperparameter regression |
| **AL-CV-08** | `pct_tracking_lost` for a specific `tenant_id` > 25% (≥ 20 sessions in rolling 48 hours) | P2 | Slack #csm-alerts → customer-success + ml-engineer | Enterprise-specific environment issue (office lighting, camera mount, unusual exercise context); CSM initiates customer outreach within 4 hours |

**Alert deduplication:** AL-CV-01 and AL-CV-02 use PagerDuty `dedup_key = "cv-fleet-health-{window_start_epoch}"` to suppress duplicate pages within the same 1-hour window.

**Rollout suppression:** AL-CV-01 and AL-CV-02 are suppressed for the first 2 hours after a new `model_version` reaches > 5% of fleet — AL-CV-03 is the primary signal during the sensitive rollout window.

---

### 18.8 Privacy Constraints on CV Observability Data

The following constraints apply to all observability tooling, dashboards, and queries touching CV data. They are invariant — they may not be relaxed by customer request, internal product request, or a compliance exception process.

| Constraint | Rationale | Enforcement |
|---|---|---|
| Raw video frames are never transmitted to any observability system | GDPR Art. 9 special category; frames constitute biometric data | On-device inference only; no frame upload path exists at the API layer |
| `keypoints_enc` is never decrypted by the observability stack | Decryption key lives in Workers Secrets, not the observability layer | `form_cost_monitor` and Metabase roles have no `cv_enc_key` injected |
| Individual form scores are never displayed in observability dashboards | Per-user `form_score` is Personal Data; cross-user combination re-identifies | `cv_session_fleet_stats` never `GROUP BY user_id`; Metabase query review gate |
| Per-user defect flags are never surfaced in monitoring | `cv_flags` per user is health-adjacent Personal Data | All defect queries use `COUNT(*) FILTER (WHERE flag > 0)` at fleet level only |
| k-anonymity floor of n ≥ 5 applies to per-tenant monitoring queries | Sub-5 groups risk cell re-identification | `CASE WHEN COUNT(DISTINCT user_id) >= 5 THEN ... ELSE NULL END` pattern in all per-tenant views |
| `user_id` is never a `GROUP BY` key in CV monitoring queries | User-level aggregation is a re-identification vector | CI lint gate: grep for `GROUP BY user_id` in SQL files touching `cv_sessions`; fail build if found without security-engineer review annotation |

**GDPR Art. 9 legal basis:** FORM's CV feature requires explicit consent under Art. 9(2)(a) before the camera is activated. The consent gate is enforced at the Worker layer (checked before `cv_sessions` insert). Any new observability signal that could be correlated with a specific user's health status or body type must be reviewed by compliance-officer before deployment.

---

### 18.9 CV Observability Dashboards

These dashboards supplement the inventory in §7.

#### 18.9.1 "CV Model Health" Dashboard (Metabase)

**Access:** internal only — ml-engineer, devops-lead, platform-engineer, founder.
**Data source:** `cv_session_fleet_stats` (nightly refresh); `cv_sdk_telemetry` (rolling queries).

| Panel | Source | Threshold |
|---|---|---|
| **Completion rate (7-day trend)** | `pct_completed` | SLO 85% as red threshold line |
| **Tracking_lost rate (7-day trend)** | `pct_tracking_lost` | SLO 8% as threshold |
| **Confidence P50 and P25 (7-day trend)** | `confidence_p50`, `confidence_p25` | P50 target ≥ 0.80 |
| **Session volume by platform** | `total_sessions` by `platform` | Stacked bar iOS vs Android |
| **Model version distribution** | `pct_of_fleet` per `model_version` | Horizontal stacked bar; rollout progress |
| **Per-version quality comparison** | `avg_pct_completed`, `avg_confidence_p50` by version | Table with delta vs. previous version |
| **SDK inference latency P95 by platform** | `inference_latency_p95_ms` | Threshold lines: 30 ms (iOS), 50 ms (Android) |
| **Camera cold start P95** | `cold_start_ms` P95 | Threshold: 800 ms |
| **Form defect fleet prevalence** | `pct_knee_cave`, `pct_back_arch`, `pct_depth_short` | Week-over-week delta; ml-engineer prioritises fine-tuning targets |

#### 18.9.2 "CV Rollout Progress" Dashboard (Metabase)

Created per deploy; archived after rollout reaches > 95% of fleet. Shared with founder during P0 model rollback decisions.

**Access:** ml-engineer, devops-lead.

| Panel | Source |
|---|---|
| **Fleet rollout progress** | `pct_of_fleet` by `model_version` over time |
| **New version vs. baseline: completion rate** | Side-by-side `pct_completed` |
| **New version vs. baseline: tracking_lost rate** | Side-by-side `pct_tracking_lost` |
| **AL-CV-03 trip status** | PagerDuty API — open/resolved status for rollout alert |
| **Blast-radius estimate** | COUNT from R-10 blast-radius query (§18.6); auto-refreshes every 30 min |

---

### 18.10 SOC 2 Evidence Contribution

| SOC 2 Criterion | How CV Observability Satisfies It |
|---|---|
| **CC7.2 — System monitoring** | `cv_session_fleet_stats` and alerting rules AL-CV-01 through AL-CV-08 provide continuous automated monitoring of the CV pipeline as a system component. AL-CV-01 through AL-CV-03 are detection controls for anomalous conditions that could indicate on-device model failure or a supply-chain compromise of the model artefact. |
| **CC7.3 — Anomaly evaluation** | The alert taxonomy (§18.7) defines explicit criteria for when a CV anomaly requires human evaluation (PagerDuty page). Each alert event is logged in PagerDuty with timestamp and assigned responder — satisfying the "evaluate and respond" requirement. |
| **PI1.2 — Processing is complete, accurate, timely, and authorised** | The CV session completion SLO (≥ 85% `status = 'completed'`) is the primary processing integrity indicator. Sessions with `status = 'partial'` or `'tracking_lost'` are explicitly flagged as incomplete processing. The SLO dashboard panel is the auditor evidence artefact for PI1.2. |
| **PI1.4 — Output completeness** | Sessions where `form_score IS NULL` (`tracking_lost` or `partial`) are logged and counted. `pct_tracking_lost` in `cv_session_fleet_stats` is the evidence that FORM monitors and responds to incomplete outputs. |
| **C1.2 — Confidentiality of sensitive data** | The privacy constraint matrix (§18.8) ensures biometric data never appears in observability. The SQL patterns (`CASE WHEN ... ELSE NULL END` for k-anonymity; prohibition on `GROUP BY user_id`) are reviewable by auditors in the Metabase query library. |

**Auditor evidence artefacts:** Monthly CSV exports of `cv_session_fleet_stats` are stored in `compliance/evidence/cv-fleet-stats-YYYY-MM.csv`. Any AL-CV-01 or AL-CV-02 PagerDuty incidents with their resolution timelines are stored in `compliance/evidence/cv-incidents/<INC-YYYYMMDD>/`.

---

### 18.11 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Write `migrations/YYYYMMDD_cv_sdk_telemetry.sql` (`cv_sdk_telemetry` table + 3 indexes) | platform-engineer | P0 | M4 |
| Add `cv_sdk_telemetry` insert to Workers `POST /v1/cv/session-telemetry` endpoint (authenticated; JWT-validated `cv_session_id`) | platform-engineer | P0 | M4 |
| Write `migrations/YYYYMMDD_cv_session_fleet_stats.sql` (materialized view DDL + 04:00 UTC `pg_cron` job) | platform-engineer | P0 | M4 |
| Implement AL-CV-01 and AL-CV-02 in Better Stack (query `cv_session_fleet_stats`; 1-hour window, min 50 sessions; PagerDuty routing to ml-engineer + platform-engineer on-call) | devops-lead | P0 | M4 |
| Implement AL-CV-03 rollout regression alert (query: new version > 10% of fleet AND `pct_tracking_lost` > 2× baseline) | devops-lead | P0 | M4 |
| Implement AL-CV-04 through AL-CV-07 Slack alerts (#platform-alerts) | devops-lead | P1 | M4 |
| Implement AL-CV-08 per-tenant alert (Slack #csm-alerts; `tenant_id`-scoped query on `cv_session_fleet_stats`) | devops-lead | P1 | M4 |
| Configure AL-CV-01/AL-CV-02 PagerDuty dedup keys (`cv-fleet-health-{window_start_epoch}`) | devops-lead | P1 | M4 |
| Configure AL-CV-01/AL-CV-02 rollout suppression rule (2-hour suppression window post-5% fleet threshold) | devops-lead | P1 | M4 |
| Build Metabase "CV Model Health" dashboard (§18.9.1 — 9 panels; data from `cv_session_fleet_stats` + `cv_sdk_telemetry`) | data-engineer | P1 | M5 |
| Build Metabase "CV Rollout Progress" dashboard template (§18.9.2 — parameterised by `model_version`; includes PagerDuty API blast-radius panel) | data-engineer | P2 | M5 |
| Add R-10 blast-radius query (§18.6) as a saved Metabase question accessible to ml-engineer and devops-lead | data-engineer | P2 | M5 |
| Add `GROUP BY user_id` prohibition CI lint (grep pattern in migration SQL files touching `cv_sessions`; fail build without security-engineer annotation) | platform-engineer | P2 | M5 |

---

*v0.3 additions: §16 Synthetic Monitoring & Uptime Verification — twelve-probe taxonomy (S-001 through S-012) covering API health, auth, workout logging, AI coaching turn, SSO SAML metadata, SCIM, audit log chain integrity, backup freshness, CDN, status page, Stripe webhook, PostHog ingest; Better Stack declarative YAML configuration; Cloudflare scheduled Worker implementations for S-007 (HMAC chain integrity) and S-008 (backup freshness) with TypeScript pseudocode; multi-region probe strategy (us-east-1, eu-central-1, ap-southeast-1) with 2-of-3 failure threshold; regional latency SLO table per probe; post-deployment smoke gate (GitHub Actions step, 120s timeout, P1 alert on failure, no auto-rollback); synthetic user data hygiene across seven pipelines (PostHog, ClickHouse, cohort ETL, DSAR, audit log, enterprise SLA); PagerDuty alert payload schema with deduplication and auto-resolve; SOC 2 CC7.2/CC7.3/A1.2/CC4.1 evidence mapping; thirteen-item implementation checklist across M3/M4/M5.*

*v0.4 additions: §17 Cost Monitoring & Anomaly Alerting — daily cost aggregation Worker (Cloudflare Cron Trigger 06:00 UTC) fetching Anthropic Admin API, ElevenLabs character delta via KV, Cloudflare GraphQL Analytics; `cost_daily` and `cost_anomalies` Postgres schema with generated `variance_pct` column; per-service anomaly thresholds (P1 = 1.5× 7d avg, P0 = 3× 7d avg) externalised to Workers Secret for hot-update; `form_cost_monitor` Postgres role with scoped write-only grants; per-tenant COGS materialized view (`cost_daily_per_tenant`) from `coaching_sessions` token telemetry, refreshed at 07:00 UTC; Metabase dashboard specs (Infra Cost Overview, Enterprise Tenant Cost panel, Cost Anomaly History); separate PagerDuty "FORM Cost" service with low-urgency routing; Slack `#metrics` daily digest format (with and without anomalies); SOC 2 CC7.2/CC7.3/CC5.2/A1.2 evidence mapping; monthly CSV export to `compliance/evidence/`; sixteen-item implementation checklist across M3/M4/M5. Closes §10 gap: Cost monitor Worker.*

*v0.5 additions: §18 CV Pose Estimation Model Observability — on-device CV pipeline (Apple Vision / ML Kit) added to Scope; privacy-first observability design for GDPR Art. 9 biometric data; RED signal taxonomy for the CV pipeline (session upload, iOS inference, Android inference, model rollout); `cv_sdk_telemetry` table DDL (cold_start_ms, inference_latency P50/P95, frame_count, frames_dropped, kalman_resets, memory_warning_count, device_tier — no keypoints or form scores); `cv_session_fleet_stats` materialized view (daily × model_name × model_version × tenant_id aggregation: pct_completed, pct_tracking_lost, confidence_p50/P25, avg_below_threshold_ratio, three defect prevalence columns); eight SLOs (completion rate ≥ 85%, tracking_lost ≤ 8%, iOS inference P95 < 30 ms, Android P95 < 50 ms, cold start P95 < 800 ms, confidence P50 ≥ 0.80, high-confidence session rate ≥ 70%, below-threshold ratio ≤ 0.12); eight alerting rules AL-CV-01 through AL-CV-08 (fleet spike → PagerDuty, confidence drop → PagerDuty, new model version quality regression → P1 with rollback trigger, per-tenant anomaly → Slack #csm-alerts); alert deduplication and rollout suppression patterns; blast-radius query aligned with INCIDENT_RESPONSE.md R-10 and DATA_MODEL.md §15.2 `idx_cv_sessions_model_version`; privacy constraint matrix (raw frames never transmitted, keypoints_enc inaccessible to observability stack, no GROUP BY user_id, k-anonymity n ≥ 5 per tenant); Metabase "CV Model Health" (9 panels) and "CV Rollout Progress" dashboard specs; SOC 2 CC7.2/CC7.3/PI1.2/PI1.4/C1.2 evidence mapping; monthly CSV export to `compliance/evidence/cv-fleet-stats-YYYY-MM.csv`; thirteen-item implementation checklist across M4/M5.*

---

## 19. SLO Error Budget Management

### 19.1 Error Budget Concept

An error budget is the complement of the SLO target: it quantifies how much unreliability is permissible within a calendar window before the system is in breach of its reliability commitments. Operationally, the error budget converts an abstract percentage target into a concrete, finite resource that engineering and product teams consume when they ship risk.

**Formula:**

```
error_budget_minutes = window_minutes × (1 − slo_target)
```

For a 30-day window (43,200 minutes):

| SLO Target | Error Budget (minutes/month) | Error Budget (%) |
|---|---|---|
| 99.9% | 43.2 min | 0.1% |
| 99.5% | 216.0 min | 0.5% |
| 99.0% | 432.0 min | 1.0% |
| 95.0% | 2,160.0 min | 5.0% |
| 85.0% | 6,480.0 min | 15.0% |

The budget is a shared resource. Every failed request, slow response, crash, or incomplete session consumes it. Planned maintenance, canary rollouts, and failed deploys all draw from the same pool. When the budget is exhausted, the team is obligated to stop spending it (feature freezes, rollbacks) and invest in reliability work until the budget is replenished at the start of the next window.

### 19.2 Error Budget per SLO

The following tables derive the per-SLO monthly error budget from the targets defined in §2.1 (platform SLOs) and §18.5 (CV pipeline SLOs), and from the synthetic probe availability targets implicit in §16.2.

**Measurement window:** rolling 30-day calendar window (43,200 minutes). All "budget consumed" columns are evaluated by the `slo_budget_tracking` table defined in §19.5.

#### 19.2.1 Platform SLOs (from §2.1)

| SLO ID | SLI | Target | Budget (min/month) | Budget (%) | Tier | Owner |
|---|---|---|---|---|---|---|
| SLO-01 | API availability (`/health` 200 rate) | 99.9% | 43.2 | 0.10% | P0 | devops-lead |
| SLO-02 | API P95 latency ≤ 1,500 ms | 99.5% | 216.0 | 0.50% | P0 | platform-engineer |
| SLO-03 | Auth token exchange success rate | 99.9% | 43.2 | 0.10% | P0 | platform-engineer |
| SLO-04 | Workout set write success rate | 99.5% | 216.0 | 0.50% | P0 | platform-engineer |
| SLO-05 | AI coaching turn P95 latency ≤ 4,000 ms | 99.0% | 432.0 | 1.00% | P1 | ml-engineer |
| SLO-06 | Crash-free sessions (mobile) | 99.5% | 216.0 | 0.50% | P0 | platform-engineer |
| SLO-07 | Background task success rate | 99.9% | 43.2 | 0.10% | P1 | devops-lead |
| SLO-08 | Database reachability | 99.9% | 43.2 | 0.10% | P0 | devops-lead |

#### 19.2.2 CV Pipeline SLOs (from §18.5)

| SLO ID | SLI | Target | Budget (min/month) | Budget (%) | Tier | Owner |
|---|---|---|---|---|---|---|
| CV-SLO-01 | CV session completion rate ≥ 85% | 85.0% | 6,480.0 | 15.00% | P1 | ml-engineer |
| CV-SLO-02 | CV tracking_lost rate ≤ 8% | 92.0%* | 3,456.0 | 8.00% | P1 | ml-engineer |
| CV-SLO-03 | On-device inference P95 < 30 ms (iOS) | 99.0% | 432.0 | 1.00% | P1 | platform-engineer |
| CV-SLO-04 | On-device inference P95 < 50 ms (Android) | 99.0% | 432.0 | 1.00% | P1 | platform-engineer |
| CV-SLO-05 | Camera cold start P95 < 800 ms (iOS) | 99.0% | 432.0 | 1.00% | P1 | platform-engineer |
| CV-SLO-06 | Avg keypoint confidence P50 ≥ 0.80 | 99.0% | 432.0 | 1.00% | P1 | ml-engineer |
| CV-SLO-07 | High-confidence session rate ≥ 70% | 95.0% | 2,160.0 | 5.00% | P1 | ml-engineer |
| CV-SLO-08 | Frame below-threshold ratio ≤ 0.12 | 95.0% | 2,160.0 | 5.00% | P1 | ml-engineer |

*CV-SLO-02 is expressed as a success-rate equivalent: an 8% floor means 92% of sessions must not be tracking_lost.

#### 19.2.3 Synthetic Probe Availability Budgets (from §16.2)

Synthetic probes each have an implicit availability SLO derived from their criticality. P0 probes inherit the 99.9% platform availability target; P1 probes use 99.5%; P2 probes use 99.0%.

| Probe ID | Name | Criticality | Implied SLO | Budget (min/month) |
|---|---|---|---|---|
| S-001 | API Health | P0 | 99.9% | 43.2 |
| S-002 | Auth Token Exchange | P0 | 99.9% | 43.2 |
| S-003 | Workout Logging API | P0 | 99.9% | 43.2 |
| S-004 | AI Coaching Turn | P1 | 99.5% | 216.0 |
| S-005 | SSO SAML Metadata | P0 | 99.9% | 43.2 |
| S-006 | SCIM User List | P1 | 99.5% | 216.0 |
| S-007 | Audit Log Write | P0 | 99.9% | 43.2 |
| S-008 | R2 Backup Freshness | P1 | 99.5% | 216.0 |
| S-009 | CDN Asset Reachability | P0 | 99.9% | 43.2 |
| S-010 | Status Page API | P2 | 99.0% | 432.0 |
| S-011 | Stripe Webhook Endpoint | P1 | 99.5% | 216.0 |
| S-012 | PostHog Ingest Health | P2 | 99.0% | 432.0 |

---

### 19.3 Burn Rate Alerting

Burn rate quantifies how fast the error budget is being consumed relative to the rate at which it replenishes. A burn rate of 1× means the budget is being consumed at exactly the rate it refills — the system will end the 30-day window with 0% budget remaining. A burn rate of 14.4× means the 30-day budget would be exhausted in approximately 50 hours.

FORM uses the Google SRE Workbook two-window, two-severity burn rate model. Two independent windows must simultaneously exceed their respective burn rate thresholds before an alert fires; this suppresses false positives from transient spikes while ensuring slow-moving degradations are caught before the budget is materially impacted.

**Burn rate thresholds:**

| Severity | Short Window | Short Window Burn Rate | Long Window | Long Window Burn Rate | Budget Consumed at Alert | Response |
|---|---|---|---|---|---|---|
| **P0 (page immediately)** | 1 hour | ≥ 14.4× | 5 minutes | ≥ 14.4× | ~2% | PagerDuty — P0 escalation |
| **P1 (page business-hours)** | 6 hours | ≥ 6.0× | 30 minutes | ≥ 6.0× | ~5% | PagerDuty — P1 escalation |
| **Slow burn (Slack only)** | 3 days | ≥ 1.0× | 1 hour | ≥ 1.0× | On pace to exhaust | Slack `#platform-alerts` |

**Why 14.4× for P0:** at 14.4× burn rate, a 30-day budget is exhausted in 50 hours (30 days ÷ 14.4 = 2.08 days). A 1-hour confirmation window ensures the signal is real before paging.

**Why 6.0× for P1:** at 6.0×, the budget exhausts in ~5 days. A 6-hour window catches degradation that is sustained but not acute — integration failures, memory leaks, gradual model quality drift.

#### 19.3.1 Postgres Alert Rules

The burn rate is computed from the `slo_budget_tracking` table (§19.5). The following queries are scheduled as Better Stack metric checks at 1-minute resolution.

**Fast burn detection (P0 — runs every 1 minute):**

```sql
-- slo_burn_rate_fast.sql
-- Alert condition: burn_rate_1h >= 14.4 for any P0 SLO
-- Evaluated by Better Stack against Supabase read replica

SELECT
    slo_id,
    burn_rate_1h,
    budget_consumed_pct,
    window_end
FROM slo_budget_tracking
WHERE
    window_end >= NOW() - INTERVAL '2 minutes'   -- most recent completed window
    AND burn_rate_1h >= 14.4
    AND slo_id IN (
        'SLO-01', 'SLO-02', 'SLO-03', 'SLO-04',
        'SLO-06', 'SLO-07', 'SLO-08',            -- P0 platform SLOs
        'S-001',  'S-002',  'S-003',  'S-005',
        'S-007',  'S-009'                          -- P0 synthetic probes
    )
ORDER BY burn_rate_1h DESC;
-- Non-empty result → fire PagerDuty P0
```

**Slow burn detection (P1 — runs every 5 minutes):**

```sql
-- slo_burn_rate_slow.sql
-- Alert condition: burn_rate_6h >= 6.0 for any SLO

SELECT
    slo_id,
    burn_rate_6h,
    budget_consumed_pct,
    window_end
FROM slo_budget_tracking
WHERE
    window_end >= NOW() - INTERVAL '10 minutes'
    AND burn_rate_6h >= 6.0
ORDER BY burn_rate_6h DESC;
-- Non-empty result → fire PagerDuty P1
```

**Slow-burn trending (Slack — runs every 1 hour):**

```sql
-- slo_burn_rate_trend.sql
-- Alert condition: 3-day rolling burn_rate >= 1.0 (on pace to exhaust budget)

SELECT
    slo_id,
    AVG(burn_rate_6h) AS burn_rate_3d_avg,
    MAX(budget_consumed_pct) AS budget_consumed_pct_current
FROM slo_budget_tracking
WHERE window_end >= NOW() - INTERVAL '3 days'
GROUP BY slo_id
HAVING AVG(burn_rate_6h) >= 1.0
ORDER BY burn_rate_3d_avg DESC;
-- Non-empty result → post to Slack #platform-alerts
```

#### 19.3.2 PagerDuty Routing Rules

```yaml
# pagerduty-routing.yaml — SLO burn rate services

services:
  - name: "FORM SLO P0 Burn Rate"
    escalation_policy: "P0 — Platform On-Call"
    alert_grouping: "intelligent"
    dedup_key_template: "slo-burn-p0-{{ slo_id }}-{{ window_hour }}"
    auto_resolve_timeout: 1800    # 30 min — re-evaluate after incident window
    urgency: high

  - name: "FORM SLO P1 Burn Rate"
    escalation_policy: "P1 — Platform Business Hours"
    alert_grouping: "intelligent"
    dedup_key_template: "slo-burn-p1-{{ slo_id }}-{{ window_6h }}"
    auto_resolve_timeout: 3600
    urgency: low
```

**Deduplication:** Burn rate alerts for the same `slo_id` within the same 1-hour epoch (P0) or 6-hour epoch (P1) are merged into a single PagerDuty incident. This prevents alert storms when a systemic failure degrades multiple correlated SLOs simultaneously (e.g., a Supabase outage simultaneously burning SLO-03, SLO-04, SLO-07, SLO-08, and S-002).

---

### 19.4 Error Budget Policies

These policies govern what engineering and product may do at each budget level. They are enforced by the devops-lead and take precedence over feature roadmap velocity when budget thresholds are crossed. Policy state is surfaced in the Metabase "SLO Error Budget Dashboard" (§19.7) and posted to Slack `#engineering` each Monday morning.

| Budget Remaining | Policy State | Deploy Velocity | Additional Constraints |
|---|---|---|---|
| **> 50%** | Green — Normal | Unrestricted | Standard change management; canary deploys encouraged |
| **25%–50%** | Yellow — Caution | Peak-hour freeze (UTC 18:00–22:00) | All deploys require devops-lead approval; post-deploy monitoring extended to 2 hours |
| **10%–25%** | Orange — Restricted | All non-critical deploys frozen | Only P0 reliability fixes may deploy; incident review required before resuming normal velocity; devops-lead notifies founder |
| **< 10%** | Red — Freeze | All deploys frozen | P0 incident declared per INCIDENT_RESPONSE.md §12; SLA credit triggers evaluated; freeze lifted only after SRE review session and explicit devops-lead sign-off |
| **Exhausted (0%)** | Critical — P0 Incident | All deploys frozen | P0 incident declared per INCIDENT_RESPONSE.md §12; SLA credit obligations per INCIDENT_RESPONSE.md §12.4 apply; post-mortem required within 5 business days; next 30-day window starts with Orange policy until reliability improvements are verified |

**Policy enforcement mechanism:** The `slo_budget_tracking` materialized view exposes a `policy_state` computed column (see §19.6). A GitHub Actions pre-deploy check queries this column for the relevant SLOs; if any P0 SLO is in Orange or Red state, the deployment workflow pauses and requires explicit override via a GitHub environment approval gate. The override is logged in the audit trail.

**Peak-hour definition (UTC 18:00–22:00):** This window covers European evening (20:00–00:00 CET) and US East Coast prime-time (14:00–18:00 ET), which represent FORM's highest-traffic periods based on PostHog session volume data.

---

### 19.5 `slo_budget_tracking` Table Schema

This table is the source of truth for error budget state. It is written by a scheduled Cloudflare Worker (cron: every 5 minutes) that computes good/total event counts from Supabase and Better Stack APIs, then upserts a row per SLO per window.

```sql
-- migrations/YYYYMMDD_slo_budget_tracking.sql

CREATE TABLE IF NOT EXISTS slo_budget_tracking (
    id                  BIGSERIAL PRIMARY KEY,
    slo_id              TEXT        NOT NULL,           -- e.g. 'SLO-01', 'CV-SLO-01', 'S-003'
    window_start        TIMESTAMPTZ NOT NULL,           -- start of the 30-day rolling window
    window_end          TIMESTAMPTZ NOT NULL,           -- timestamp of this measurement (≈ NOW())
    target_pct          NUMERIC(6,4) NOT NULL,          -- e.g. 99.9000 for 99.9%
    good_events         BIGINT      NOT NULL,           -- count of good (successful) events in window
    total_events        BIGINT      NOT NULL,           -- count of total events in window
    budget_consumed_pct NUMERIC(8,4) GENERATED ALWAYS AS (
        CASE
            WHEN total_events = 0 THEN 0
            ELSE GREATEST(
                0,
                ((1 - target_pct / 100.0) - (1.0 - good_events::NUMERIC / total_events))
                / (1 - target_pct / 100.0) * (-100.0) + 100.0
            )
        END
    ) STORED,           -- % of monthly budget consumed; 0 = none consumed, 100 = fully exhausted
    burn_rate_1h        NUMERIC(10,4),                  -- computed by Worker: actual_error_rate_1h / (1 - target_pct/100)
    burn_rate_6h        NUMERIC(10,4),                  -- computed by Worker: actual_error_rate_6h / (1 - target_pct/100)
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT slo_budget_tracking_positive_events CHECK (total_events >= 0 AND good_events >= 0),
    CONSTRAINT slo_budget_tracking_good_lte_total  CHECK (good_events <= total_events),
    CONSTRAINT slo_budget_tracking_target_range    CHECK (target_pct BETWEEN 0 AND 100),
    CONSTRAINT slo_budget_tracking_window_order    CHECK (window_end > window_start)
);

-- Unique: one row per SLO per 5-minute window_end bucket
CREATE UNIQUE INDEX IF NOT EXISTS uix_slo_budget_tracking_slo_window
    ON slo_budget_tracking (slo_id, window_end);

-- Query pattern: latest state per SLO
CREATE INDEX IF NOT EXISTS idx_slo_budget_tracking_slo_created
    ON slo_budget_tracking (slo_id, created_at DESC);

-- Query pattern: burn rate alerts (scan recent rows across all SLOs)
CREATE INDEX IF NOT EXISTS idx_slo_budget_tracking_window_end
    ON slo_budget_tracking (window_end DESC);

-- Row-level security: slo_budget_reader role may SELECT; slo_budget_writer role may INSERT/UPDATE
ALTER TABLE slo_budget_tracking ENABLE ROW LEVEL SECURITY;

CREATE POLICY slo_budget_select ON slo_budget_tracking
    FOR SELECT TO slo_budget_reader USING (true);

CREATE POLICY slo_budget_insert ON slo_budget_tracking
    FOR INSERT TO slo_budget_writer WITH CHECK (true);

COMMENT ON TABLE slo_budget_tracking IS
    'Rolling 30-day error budget state per SLO. Written by the slo-budget-worker Cloudflare Worker every 5 minutes. Read by Better Stack alert queries, Metabase dashboards, and the GitHub Actions pre-deploy gate.';

COMMENT ON COLUMN slo_budget_tracking.burn_rate_1h IS
    'Burn rate over the last 1 hour: (actual_error_rate_1h / allowable_error_rate). '
    '1.0 = consuming budget at exactly the replenishment rate. 14.4 = P0 threshold.';

COMMENT ON COLUMN slo_budget_tracking.burn_rate_6h IS
    'Burn rate over the last 6 hours: (actual_error_rate_6h / allowable_error_rate). '
    '6.0 = P1 threshold.';
```

**Retention:** Rows older than 90 days are deleted by a `pg_cron` job running daily at 03:00 UTC:

```sql
SELECT cron.schedule(
    'slo_budget_tracking_cleanup',
    '0 3 * * *',
    $$DELETE FROM slo_budget_tracking WHERE created_at < NOW() - INTERVAL '90 days'$$
);
```

---

### 19.6 `slo_budget_weekly` Materialized View

This view rolls up the per-5-minute `slo_budget_tracking` rows into weekly summaries. It is the primary data source for the Metabase "SLO Error Budget Dashboard" (§19.7) and for the Monday Slack digest.

```sql
-- migrations/YYYYMMDD_slo_budget_weekly.sql

CREATE MATERIALIZED VIEW IF NOT EXISTS slo_budget_weekly AS
SELECT
    slo_id,
    DATE_TRUNC('week', window_end AT TIME ZONE 'UTC') AS week_start,

    -- SLO target (should be constant per SLO; take the max to surface misconfiguration)
    MAX(target_pct)                                   AS target_pct,

    -- Budget consumption: max consumed in the week = worst point reached
    MAX(budget_consumed_pct)                          AS budget_consumed_pct_peak,

    -- Budget consumption: latest value in the week = current state
    (ARRAY_AGG(budget_consumed_pct ORDER BY window_end DESC))[1]
                                                      AS budget_consumed_pct_eow,

    -- Burn rates: weekly averages and peaks
    AVG(burn_rate_1h)                                 AS burn_rate_1h_avg,
    MAX(burn_rate_1h)                                 AS burn_rate_1h_peak,
    AVG(burn_rate_6h)                                 AS burn_rate_6h_avg,
    MAX(burn_rate_6h)                                 AS burn_rate_6h_peak,

    -- Event volume
    SUM(total_events)                                 AS total_events_week,
    SUM(good_events)                                  AS good_events_week,

    -- Derived: actual error rate for the week
    CASE
        WHEN SUM(total_events) = 0 THEN NULL
        ELSE ROUND(
            (1.0 - SUM(good_events)::NUMERIC / SUM(total_events)) * 100,
            4
        )
    END                                               AS actual_error_rate_pct,

    -- Policy state at end of week
    CASE
        WHEN (ARRAY_AGG(budget_consumed_pct ORDER BY window_end DESC))[1] >= 100 THEN 'Critical'
        WHEN (ARRAY_AGG(budget_consumed_pct ORDER BY window_end DESC))[1] >= 90  THEN 'Red'
        WHEN (ARRAY_AGG(budget_consumed_pct ORDER BY window_end DESC))[1] >= 75  THEN 'Orange'
        WHEN (ARRAY_AGG(budget_consumed_pct ORDER BY window_end DESC))[1] >= 50  THEN 'Yellow'
        ELSE 'Green'
    END                                               AS policy_state_eow,

    COUNT(*)                                          AS sample_count,
    MIN(window_end)                                   AS week_first_sample,
    MAX(window_end)                                   AS week_last_sample

FROM slo_budget_tracking
GROUP BY slo_id, DATE_TRUNC('week', window_end AT TIME ZONE 'UTC')
WITH DATA;

-- Refresh every Monday at 06:00 UTC (after weekly data is complete)
SELECT cron.schedule(
    'slo_budget_weekly_refresh',
    '0 6 * * 1',
    $$REFRESH MATERIALIZED VIEW CONCURRENTLY slo_budget_weekly$$
);

CREATE UNIQUE INDEX IF NOT EXISTS uix_slo_budget_weekly_slo_week
    ON slo_budget_weekly (slo_id, week_start);

CREATE INDEX IF NOT EXISTS idx_slo_budget_weekly_week_start
    ON slo_budget_weekly (week_start DESC);

COMMENT ON MATERIALIZED VIEW slo_budget_weekly IS
    'Weekly rollup of slo_budget_tracking. Refreshed every Monday 06:00 UTC. '
    'Primary source for Metabase SLO Error Budget Dashboard and Monday Slack digest.';
```

---

### 19.7 Metabase Dashboard: "SLO Error Budget Dashboard"

**Access:** devops-lead, platform-engineer, ml-engineer, founder.
**Refresh cadence:** live query for panels 1–3 (5-minute auto-refresh); panels 4–6 query `slo_budget_weekly` (daily refresh sufficient).
**Data sources:** `slo_budget_tracking` (panels 1–3), `slo_budget_weekly` (panels 4–6).

#### Panel 1 — Error Budget Gauges by SLO Tier

**Type:** gauge array (one gauge per SLO, grouped by tier: P0 Platform / P1 Platform / CV Pipeline / Synthetic P0 / Synthetic P1).
**Source:** `slo_budget_tracking` — latest row per `slo_id` (WHERE `window_end >= NOW() - INTERVAL '10 minutes'`).
**Display:** radial gauge 0–100%; colour bands: green (< 50% consumed), yellow (50–75%), orange (75–90%), red (> 90%), critical/black (100%).
**Purpose:** single-glance view of which SLOs are under pressure. Linked to the corresponding PagerDuty service for one-click incident creation.

#### Panel 2 — Burn Rate Trend (Rolling 7 Days)

**Type:** multi-line time-series chart.
**Source:** `slo_budget_tracking` — `burn_rate_1h` and `burn_rate_6h` per `slo_id`, filtered to the last 7 days.
**Display:** one line per SLO; reference lines at 1.0× (steady state), 6.0× (P1 threshold), 14.4× (P0 threshold). Log scale on Y-axis to show slow-burn and fast-burn on the same chart. Interactive: click a line to drill to the SLO detail view.
**Purpose:** catch accelerating burn before it crosses alert thresholds; identify correlated burn events (e.g., multiple SLOs spiking simultaneously → systemic incident vs. isolated regression).

#### Panel 3 — Top-5 Budget Consumers (Current 30-Day Window)

**Type:** horizontal bar chart.
**Source:** `slo_budget_tracking` — `MAX(budget_consumed_pct)` per `slo_id` over the current 30-day window, ordered descending, limited to 5.
**Display:** bars with value labels; colour-coded by policy state. Table below bar chart with columns: `slo_id`, `target_pct`, `budget_consumed_pct`, `burn_rate_6h`, `policy_state`.
**Purpose:** prioritise reliability investment; ensure the team always knows which SLO is closest to exhaustion.

#### Panel 4 — 30-Day Rolling View (Weekly Trend)

**Type:** stacked area chart.
**Source:** `slo_budget_weekly` — `budget_consumed_pct_eow` per `slo_id` per `week_start`, rolling 13 weeks (approximately 3 months of history).
**Display:** one area per SLO tier (P0 Platform, P1 Platform, CV Pipeline); stacked to show aggregate budget pressure; colour bands correspond to policy states. X-axis: week_start. Y-axis: budget_consumed_pct_eow (0–100%).
**Purpose:** reveal seasonal patterns and trend lines; evidence artefact for quarterly SRE review and SOC 2 auditor.

#### Panel 5 — Enterprise Tenant Budget Breakdown

**Type:** table with conditional formatting.
**Source:** `slo_budget_tracking` joined to `tenants` — budget state for SLOs that have per-tenant variants (CV-SLO-01, CV-SLO-02, S-005, S-006). Grouped by `tenant_id`.
**Display:** columns: `tenant_name`, `slo_id`, `budget_consumed_pct`, `burn_rate_6h`, `policy_state_eow`. Rows sorted by `budget_consumed_pct DESC`. Conditional formatting: red background for `policy_state_eow IN ('Orange', 'Red', 'Critical')`.
**Note:** k-anonymity floor of n ≥ 5 sessions applies; tenants below threshold are suppressed (consistent with §18.8 privacy constraints).
**Purpose:** identify enterprise tenants driving disproportionate SLO budget consumption; input to CSM outreach and enterprise SLA credit evaluation.

#### Panel 6 — Audit Trail: Budget-Exhaustion Events

**Type:** table (append-only log).
**Source:** `slo_budget_tracking` filtered to `budget_consumed_pct >= 100`, joined to PagerDuty incident API (via Metabase HTTP data source) for linked incident IDs.
**Display:** columns: `window_end` (timestamp of exhaustion detection), `slo_id`, `burn_rate_1h` at exhaustion, `burn_rate_6h` at exhaustion, `pagerduty_incident_id` (linked), `resolved_at` (from PagerDuty), `postmortem_url` (manual field, populated by devops-lead after post-mortem). Sorted by `window_end DESC`.
**Purpose:** auditor evidence artefact for CC4.1 (breach detection) and CC7.2 (response logging); input to the "next-window Orange policy" decision after budget exhaustion per §19.4.

---

### 19.8 SOC 2 Evidence Mapping

| SOC 2 Criterion | How §19 Satisfies It |
|---|---|
| **CC4.1 — Monitoring of controls** | The `slo_budget_tracking` table provides continuous, automated measurement of whether each SLO control is operating within its defined tolerance. The burn rate alert rules (§19.3) are detective controls that fire before the tolerance is fully exhausted — satisfying the "timely detection" requirement of CC4.1. Panel 6 of the Metabase dashboard is the auditor-accessible evidence artefact showing when controls were breached and how quickly they were remediated. |
| **CC7.2 — System monitoring** | Error budget burn rate is a quantitative, time-stamped system monitoring signal. Every row in `slo_budget_tracking` is an immutable timestamped record of system health state. The 90-day retention policy ensures auditors can query historical monitoring data for the full SOC 2 audit period. |
| **A1.1 — Availability commitments** | The error budget per SLO (§19.2) directly encodes FORM's availability commitments as measurable, numeric targets. The `slo_budget_weekly` materialized view provides the weekly rollup that demonstrates whether commitments were met. Auditors may query `policy_state_eow` to verify that FORM's availability posture remained compliant across the audit window. |
| **A1.2 — Availability monitoring and response** | The error budget policy (§19.4) defines a documented, graduated response to availability degradation — from unrestricted velocity (> 50% budget) to mandatory P0 incident declaration (budget exhausted). The GitHub Actions pre-deploy gate enforces the policy programmatically. Policy state changes are logged in the audit trail (Panel 6). |
| **CC5.2 — Control activities to mitigate risk** | The deploy freeze policy (§19.4) is a risk mitigation control: when the error budget is under pressure, it restricts the primary mechanism by which new risk is introduced (software deployments). The PagerDuty deduplication rules (§19.3.2) are a control activity that prevents alert fatigue from masking real incidents. The `slo_budget_writer` and `slo_budget_reader` Postgres roles enforce separation of concerns between the data pipeline and the monitoring/reporting layer. |

**Auditor evidence artefacts:**
- Monthly CSV exports of `slo_budget_weekly` are stored in `compliance/evidence/slo-budget-YYYY-MM.csv`.
- Budget-exhaustion events (Panel 6) are exported to `compliance/evidence/slo-exhaustion/<SLO_ID>-<YYYYMMDD>/` including the linked PagerDuty incident timeline and post-mortem document.
- The GitHub Actions pre-deploy gate override log is stored in `compliance/evidence/deploy-overrides/YYYY-MM.csv` (columns: `timestamp`, `slo_id`, `policy_state_at_override`, `approver`, `rationale`).

---

### 19.9 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Write `migrations/YYYYMMDD_slo_budget_tracking.sql` (table DDL, indexes, RLS policies, `pg_cron` cleanup job) | platform-engineer | P0 | M3 |
| Write `migrations/YYYYMMDD_slo_budget_weekly.sql` (materialized view DDL, Monday 06:00 UTC `pg_cron` refresh) | platform-engineer | P0 | M3 |
| Create `slo_budget_writer` and `slo_budget_reader` Postgres roles with scoped grants | platform-engineer | P0 | M3 |
| Implement `slo-budget-worker` Cloudflare Worker (cron: every 5 minutes; fetches good/total counts from Supabase + Better Stack heartbeat API; upserts `slo_budget_tracking`; computes `burn_rate_1h` and `burn_rate_6h`) | devops-lead | P0 | M3 |
| Configure Better Stack metric alerts using `slo_burn_rate_fast.sql` (1-minute polling; P0 PagerDuty escalation on `burn_rate_1h >= 14.4` for P0 SLOs) | devops-lead | P0 | M3 |
| Configure Better Stack metric alerts using `slo_burn_rate_slow.sql` (5-minute polling; P1 PagerDuty escalation on `burn_rate_6h >= 6.0` for all SLOs) | devops-lead | P0 | M3 |
| Configure Better Stack hourly slow-burn trend alert using `slo_burn_rate_trend.sql` (Slack `#platform-alerts` on 3-day avg `burn_rate_6h >= 1.0`) | devops-lead | P1 | M3 |
| Configure PagerDuty "FORM SLO P0 Burn Rate" and "FORM SLO P1 Burn Rate" services with dedup key templates and escalation policies (§19.3.2) | devops-lead | P0 | M3 |
| Add GitHub Actions pre-deploy gate: query `slo_budget_tracking` for P0 SLO `policy_state`; pause workflow if Orange/Red; require environment approval gate; log overrides to `compliance/evidence/deploy-overrides/` | devops-lead | P1 | M4 |
| Build Metabase "SLO Error Budget Dashboard" — 6 panels per §19.7 spec; set auto-refresh 5 minutes for panels 1–3 | data-engineer | P1 | M4 |
| Configure Monday 07:00 UTC Slack `#engineering` digest: query `slo_budget_weekly` for prior week; post policy state per SLO tier; flag any SLO that entered Orange/Red during the week | devops-lead | P1 | M4 |
| Add monthly CSV export of `slo_budget_weekly` to `compliance/evidence/slo-budget-YYYY-MM.csv` (GitHub Actions schedule: first day of month, 08:00 UTC) | devops-lead | P2 | M4 |
| Add budget-exhaustion event export to `compliance/evidence/slo-exhaustion/` on every P0 budget incident (triggered by PagerDuty webhook → Worker → R2 export) | devops-lead | P2 | M4 |

---

*v0.6 additions: §19 SLO Error Budget Management — error budget concept formalised with per-SLO budget tables covering 8 platform SLOs, 8 CV pipeline SLOs, and 12 synthetic probes; Google SRE-style two-window burn rate alerting (fast burn 14.4× / 1 hour → P0, slow burn 6.0× / 6 hours → P1) with concrete Postgres queries and PagerDuty deduplication rules; graduated deploy-freeze policy (Green/Yellow/Orange/Red/Critical) enforced via GitHub Actions gate with override audit trail; `slo_budget_tracking` Postgres schema with generated `budget_consumed_pct` column, burn rate columns, RLS policies, and 90-day retention cleanup; `slo_budget_weekly` materialized view with Monday 06:00 UTC pg_cron refresh; Metabase "SLO Error Budget Dashboard" spec (6 panels: budget gauges, burn rate trend, top-5 consumers, 30-day rolling view, enterprise tenant breakdown, budget-exhaustion audit trail); SOC 2 CC4.1/CC7.2/A1.1/A1.2/CC5.2 evidence mapping with compliance artefact export paths; thirteen-item implementation checklist across M3/M4.*

---

## 20. Wearable Data Integration Observability

> Owner: platform-engineer + devops-lead. References: `docs/DATA_MODEL.md §14`, `docs/INCIDENT_RESPONSE.md R-10`, `docs/SOC2_READINESS.md §35.6.3`, `docs/GDPR_DPIA.md`, DEC-030.

**Scope:** HealthKit (iOS), Health Connect (Android), Whoop, Oura Ring, Garmin — all five wearable sources defined in `DATA_MODEL.md §14.1`.

**Why wearable observability is different:** A coaching API failure is immediately visible — the user gets an error screen. A wearable sync failure is a *silent failure*: Victor continues responding, but his readiness assessments silently degrade because HRV and sleep data are stale. The user has no indication anything is wrong. This makes freshness monitoring for wearable data a higher-priority concern than availability monitoring for most other services — a stale-but-responsive system is often worse than an honest error.

**Privacy boundary:** All signals in this section operate at aggregate or per-device operational metadata granularity only. The `wearable_sync_health` table (§20.6) stores sync *outcomes* — timestamps, failure counts, error codes — not health *values*. No HRV readings, recovery scores, sleep durations, or any other `wearable_readings.value_numeric` content is present in any observability signal, dashboard, or alert payload. This is a hard constraint aligned with GDPR Art. 9 and the analytics restriction matrix in `DATA_MODEL.md §14.10`.

---

### 20.1 RED Metrics Per Provider

One row per wearable source. Rate = successful syncs per time window. Errors = sync failures or stale connections. Duration = sync latency (background syncs are asynchronous; latency here means time from cron trigger to DB write completion).

| Source | Rate (R) | Errors (E) | Duration (D) |
|---|---|---|---|
| **HealthKit (iOS)** | `wearable_sync_success_total{source="healthkit"}` events/hour | `wearable_sync_failures_total{source="healthkit"}` / total syncs | P95 of background-delivery-to-db-write latency (target < 10 s) |
| **Health Connect (Android)** | `wearable_sync_success_total{source="health_connect"}` events/hour | `wearable_sync_failures_total{source="health_connect"}` / total syncs | P95 of WorkManager job completion time (target < 60 s; includes 15-min OS minimum delay) |
| **Whoop** | `wearable_sync_success_total{source="whoop"}` per cron run | `wearable_sync_failures_total{source="whoop"}` + `wearable_sync_rate_limited_total{source="whoop"}` | Cron run wall-clock time P95 (target < 120 s per run across all active Whoop users) |
| **Oura** | `wearable_sync_success_total{source="oura"}` per cron run | `wearable_sync_failures_total{source="oura"}` | Cron run wall-clock time P95 (target < 90 s) |
| **Garmin** | `wearable_sync_success_total{source="garmin"}` per cron run | `wearable_sync_failures_total{source="garmin"}` + `wearable_sync_quota_exceeded_total{source="garmin"}` | Cron run wall-clock time P95 (target < 90 s) |
| **Token refresh (Whoop/Oura/Garmin)** | `wearable_token_refresh_success_total` per 2h cron | `wearable_token_refresh_failures_total` by source | P95 token refresh round-trip latency (target < 5 s) |

All counters are emitted from the sync Cloudflare Worker via Workers Analytics Engine and stored in the `wearable_sync_health` table (§20.6). Source label values match the `wearable_readings.source` CHECK constraint in `DATA_MODEL.md §14.2`.

---

### 20.2 Wearable Data Freshness SLOs

Freshness is the primary health signal for wearable integrations. "Stale" means the coaching context builder is operating on data older than the staleness threshold, producing readiness assessments that may no longer reflect the user's actual recovery state.

**Eligibility filter:** SLOs apply only to *active, wearable-connected users* — users who: (a) have at least one non-revoked `wearable_oauth_tokens` row (or active HealthKit/Health Connect permission) for the source, and (b) have logged at least one coaching session in the prior 7 days. Users who have disconnected a source or been inactive for >7 days are excluded from freshness SLO calculation.

| SLO ID | SLI | SLO Target | Alerting Threshold | Window | Owner |
|---|---|---|---|---|---|
| **WEAR-SLO-01** | % of active HealthKit-connected users with a `wearable_readings` row for any `reading_type` with `recorded_at` within 25 hours | ≥ 95% | < 88% in 4-hour window | Rolling 7 days | platform-engineer |
| **WEAR-SLO-02** | % of active Health Connect-connected users with a reading within 26 hours | ≥ 90% | < 80% in 4-hour window | Rolling 7 days | platform-engineer |
| **WEAR-SLO-03** | % of active Whoop-connected users with a reading within 26 hours | ≥ 92% | < 82% in 4-hour window | Rolling 7 days | platform-engineer |
| **WEAR-SLO-04** | % of active Oura-connected users with a reading within 26 hours | ≥ 92% | < 82% in 4-hour window | Rolling 7 days | platform-engineer |
| **WEAR-SLO-05** | % of active Garmin-connected users with a reading within 26 hours | ≥ 88% | < 75% in 4-hour window | Rolling 7 days | platform-engineer |
| **WEAR-SLO-06** | % of active third-party OAuth tokens (Whoop/Oura/Garmin) that are valid — `expires_at IS NULL OR expires_at > NOW() + INTERVAL '1 hour'` and `revoked_at IS NULL` | ≥ 99.5% | < 98% in 1-hour window | Rolling 7 days | platform-engineer |
| **WEAR-SLO-07** | % of scheduled wearable sync cron runs that complete without terminal error within the expected window (Whoop/Oura/Garmin cron: every 2 hours ± 10 min) | ≥ 99% | < 97% in 24-hour window | Rolling 7 days | devops-lead |
| **WEAR-SLO-08** | % of multi-source users (≥ 2 active wearable sources) where the HRV normalization layer produces a valid, non-null `hrv_trend_label` from `tenant_wearable_summary` (DATA_MODEL.md §14.4) | ≥ 98% | < 94% in 12-hour window | Rolling 7 days | platform-engineer |

**Health Connect target is lower (90%) than HealthKit (95%)** because the Android OS enforces a 15-minute minimum WorkManager interval. A user who does not open the app may experience up to ~30-minute sync lag on top of data publication delay. This is an OS-imposed constraint, not a FORM failure. Victor's response templates explicitly account for this ("based on your most recent available data").

**Garmin target is lowest (88%)** because Garmin's 500 requests/day per-user limit creates an inherent staleness risk at the 2-hour cron cadence if the user has multiple wearable data types to fetch. The Garmin integration is enterprise-only, and its 26-hour freshness window is aligned to the daily summary data granularity of the Garmin Health API.

---

### 20.3 OAuth Token Health Monitoring

Only Whoop, Oura, and Garmin use OAuth tokens (see `DATA_MODEL.md §14.8`). HealthKit and Health Connect use on-device OS permissions — they have no server-side token to monitor.

**Token state machine observable via `wearable_oauth_tokens`:**

| State | Condition | Observable signal | Action |
|---|---|---|---|
| Healthy | `revoked_at IS NULL` AND (`expires_at IS NULL` OR `expires_at > NOW() + 24h`) | Normal sync | None |
| Expiry-approaching | `revoked_at IS NULL` AND `expires_at BETWEEN NOW() AND NOW() + 24h` | `wearable_token_expiry_approaching_total` counter | AL-WEAR-05 P2 alert; refresh worker must run |
| Expired | `revoked_at IS NULL` AND `expires_at < NOW()` | `wearable_token_expired_total` counter | AL-WEAR-04: user session sync will fail; requires re-authentication |
| Refresh-failed | `last_refreshed_at` not updated in >3h despite valid token | `wearable_token_refresh_failures_total{source=X}` | AL-WEAR-04 P1 alert |
| Revoked-by-user | `revoked_at IS NOT NULL` | Not an error state | Excluded from SLO calculation; source shown as disconnected in UI |

**Refresh worker guarantees:** The Cloudflare Worker cron that refreshes tokens (2-hour cadence per `DATA_MODEL.md §14.9.3`) must complete before `expires_at`. Most sources issue tokens with 6–24h expiry windows. The refresh worker targets updating tokens when `expires_at < NOW() + 4h` — providing a 2-cron-run buffer. If the refresh fails twice consecutively (`consecutive_refresh_failures ≥ 2`), the token is flagged in `wearable_sync_health.sync_status = 'refresh_failed'` and AL-WEAR-04 fires.

---

### 20.4 Sync Architecture Health Signals

Each sync mechanism requires distinct health checks:

**HealthKit background delivery:**
- Observable via: `wearable_sync_health.last_sync_attempted_at` — if iOS background delivery is functioning, this timestamp advances at least once per 25 hours for active users.
- Failure mode: HealthKit Background Delivery registration can silently expire after OS updates. Detection: `last_sync_attempted_at > NOW() - INTERVAL '25 hours'` is FALSE for > 5% of active iOS users.
- No server-side polling is available to diagnose iOS permission state — FORM relies on the absence of syncs as the primary failure signal. A freshness alert (AL-WEAR-01) is the only detection mechanism for this failure mode.

**Health Connect WorkManager:**
- Observable via: sync timestamps and `wearable_sync_failures_total{source="health_connect", error_code="permission_denied"}` counter.
- Failure mode: Android 14+ may revoke Health Connect permissions without user action in response to app backgrounding policy changes. Detection: `error_code="permission_denied"` counter increasing.
- Recovery: user must re-grant permissions in-app. Push notification template: "Your Android Health data connection needs attention — tap to restore."

**Third-party API crons (Whoop / Oura / Garmin):**
- Observable via: cron run completion timestamps in `wearable_sync_health`, cron-level error rates, and per-source `sync_status` field.
- Failure modes:
  - API upstream outage: all users of that source fail simultaneously; error rate spikes cross-source. Indicator: `wearable_sync_failures_total{source="whoop"}` rises sharply across many users in a single cron run.
  - Rate limit exhaustion: Whoop (100 req/min), Garmin (500/day per user). Indicator: `wearable_sync_rate_limited_total` and/or `wearable_sync_quota_exceeded_total`.
  - OAuth token expiry: individual user failure, not fleet-wide. Indicator: `error_code="auth_failure"` on a subset of users.
- Cron-run heartbeat: each cron invocation emits `wearable_cron_heartbeat{source="whoop"|"oura"|"garmin", status="started"|"completed"|"failed"}` to the Workers Analytics Engine. The absence of a heartbeat for > 3 hours (missed 1.5 expected runs) triggers AL-WEAR-07.

---

### 20.5 HRV Normalization Quality Monitoring

Victor's readiness module normalises HRV values from heterogeneous sources into a single per-user trend using the priority ordering in `DATA_MODEL.md §14.5` (HealthKit > Health Connect > Oura > Whoop > Garmin-categorical-only). The observability concern here is not the HRV *value* — it is whether the normalization pipeline produces a *valid trend output*.

**What to monitor:**
- `tenant_wearable_summary` materialized view (from `DATA_MODEL.md §14.4`) refreshes daily at 04:00 UTC. If the refresh job fails or produces NULL `hrv_trend_label` values for eligible users, WEAR-SLO-08 degrades.
- The `pg_cron` job that refreshes `tenant_wearable_summary` emits a DEC-030 audit event `analytics.materialized_view_refreshed{view="tenant_wearable_summary"}` with a row count. If the row count drops by > 20% vs. the prior day with no corresponding user churn, it signals a normalization pipeline failure.

**What NOT to monitor:** The numeric HRV distribution across users. Aggregate HRV statistics are GDPR Art. 9 data — they must not appear in any operational dashboard. The observable is *pipeline output validity* (was a trend label produced? yes/no), never the trend label values themselves or any derived statistics.

---

### 20.6 Database Schema: `wearable_sync_health`

This table stores operational metadata about wearable sync state. It contains **no health data values** — only timestamps, failure counts, and status codes. Classification: **INTERNAL** (not SENSITIVE — no health data present).

```sql
-- Operational sync health tracking per user per source.
-- CLASSIFICATION: INTERNAL — timestamps and error codes only; no health values.
-- Retention: rolling 90 days (pg_cron cleanup; beyond that, aggregate to wearable_sync_weekly).
-- Privacy note: no GROUP BY user_id in fleet queries — use tenant-level aggregates (§20.8).
CREATE TABLE wearable_sync_health (
  id                          UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id                     UUID        NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id                   UUID        NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,

  source                      TEXT        NOT NULL
                                CHECK (source IN (
                                  'healthkit',
                                  'health_connect',
                                  'whoop',
                                  'oura',
                                  'garmin'
                                )),

  -- When the sync worker last attempted a sync for this user+source (any outcome).
  last_sync_attempted_at      TIMESTAMPTZ,

  -- When the last sync produced at least one new row in wearable_readings.
  last_sync_succeeded_at      TIMESTAMPTZ,

  -- The recorded_at timestamp of the most recent wearable_readings row for this user+source.
  -- Derived from: MAX(recorded_at) WHERE user_id=? AND source=? AND deleted_at IS NULL.
  -- Updated on each successful sync. Used for freshness SLO calculation (§20.2).
  last_reading_recorded_at    TIMESTAMPTZ,

  -- Count of consecutive sync failures (reset to 0 on any success).
  consecutive_sync_failures   SMALLINT    NOT NULL DEFAULT 0,

  -- Total sync failures in the rolling 7-day window.
  failures_7d                 SMALLINT    NOT NULL DEFAULT 0,

  -- Sync disposition for this source.
  -- 'ok'              — last sync succeeded, freshness within SLO
  -- 'stale'           — last sync succeeded but last_reading_recorded_at outside freshness window
  -- 'sync_failed'     — last N consecutive syncs failed (N >= 2)
  -- 'refresh_failed'  — OAuth token refresh failed (Whoop/Oura/Garmin only)
  -- 'token_expired'   — OAuth token is past expires_at with no valid refresh
  -- 'permission_lost' — OS permission check returned denied (HC/HealthKit)
  -- 'quota_exceeded'  — API rate limit or daily quota exhausted (Garmin)
  -- 'disabled'        — user explicitly disconnected source (revoked_at IS NOT NULL)
  sync_status                 TEXT        NOT NULL DEFAULT 'ok'
                                CHECK (sync_status IN (
                                  'ok', 'stale', 'sync_failed', 'refresh_failed',
                                  'token_expired', 'permission_lost', 'quota_exceeded',
                                  'disabled'
                                )),

  -- Last error code from the sync worker (e.g. 'auth_failure', 'rate_limited',
  -- 'upstream_5xx', 'network_timeout', 'permission_denied', 'no_new_data').
  -- 'no_new_data' is NOT a failure — source produced no readings since last sync.
  error_code_last             TEXT,

  -- Timestamp of last OAuth token refresh (Whoop/Oura/Garmin only; NULL for HealthKit/HC).
  last_token_refreshed_at     TIMESTAMPTZ,

  -- Count of consecutive token refresh failures (reset on successful refresh).
  consecutive_refresh_failures SMALLINT   NOT NULL DEFAULT 0,

  updated_at                  TIMESTAMPTZ NOT NULL DEFAULT now(),

  -- One row per user per source. Upserted by sync worker on every run.
  UNIQUE (user_id, source)
);

ALTER TABLE wearable_sync_health ENABLE ROW LEVEL SECURITY;

-- Sync worker (form_system) reads and writes all rows.
CREATE POLICY wearable_sync_health_system ON wearable_sync_health
  AS PERMISSIVE FOR ALL
  TO form_system
  USING (TRUE)
  WITH CHECK (TRUE);

-- Users may read their own sync status (surfaced in Settings → Integrations).
CREATE POLICY wearable_sync_health_user_read ON wearable_sync_health
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND user_id = current_setting('app.current_user_id', TRUE)::UUID
  );

-- Tenant admin role: NO SELECT policy on this table.
-- Admin dashboard wearable health is exposed only via wearable_fleet_health VIEW (§20.8).
-- Absence of permissive policy = zero rows (fail-closed per §3.1).

CREATE INDEX idx_wearable_sync_health_tenant_source
  ON wearable_sync_health (tenant_id, source, sync_status);

CREATE INDEX idx_wearable_sync_health_stale
  ON wearable_sync_health (tenant_id, source, last_reading_recorded_at)
  WHERE sync_status IN ('stale', 'sync_failed', 'token_expired');
```

**Upsert pattern:** The sync Worker issues an `INSERT ... ON CONFLICT (user_id, source) DO UPDATE` after every sync attempt. The upsert sets `last_sync_attempted_at = NOW()`, conditionally updates `last_sync_succeeded_at` and `last_reading_recorded_at` on success, increments `consecutive_sync_failures` on failure, and resets it to 0 on success. The `sync_status` label is computed by the Worker from the current field values before writing, not derived by Postgres.

---

### 20.7 Alerting Rules

Alert names follow the `AL-WEAR-XX` taxonomy. All rules run against the `wearable_sync_health` table and Workers Analytics Engine counters.

| Rule ID | Signal | Threshold | Severity | Destination | Notes |
|---|---|---|---|---|---|
| **AL-WEAR-01** | % of active HealthKit-connected users with `sync_status IN ('sync_failed', 'stale')` in 4h window | > 12% | P1 | PagerDuty | Indicates iOS OS-level issue or HealthKit API change; fleet-wide |
| **AL-WEAR-02** | % of active Health Connect-connected users with `sync_status IN ('sync_failed', 'permission_lost')` in 4h window | > 18% | P1 | PagerDuty | Higher threshold due to WorkManager OS constraints; permission_lost = Android policy change |
| **AL-WEAR-03** | `wearable_sync_failures_total{source="whoop"|"oura"|"garmin"}` rate in a single cron run / total users for that source | > 20% | P1 | PagerDuty | Third-party upstream outage; check source status page before escalating |
| **AL-WEAR-04** | `consecutive_refresh_failures >= 2` across > 5% of active Whoop/Oura/Garmin-connected users | — | P1 | PagerDuty | Token refresh systemic failure; check KMS key availability and source OAuth endpoints |
| **AL-WEAR-05** | `wearable_token_expiry_approaching_total` — tokens expiring within 4 hours where `consecutive_refresh_failures >= 1` | > 0 | P2 | Slack `#platform-alerts` | Individual token expiry after one failed refresh; refresh worker should self-heal on next run |
| **AL-WEAR-06** | `wearable_sync_quota_exceeded_total{source="garmin"}` exceeds 80% of the 500/day per-user limit in the first 18 hours of the UTC day | — | P2 | Slack `#platform-alerts` | Garmin quota risk; cron may fail for high-activity users in the final 6h window |
| **AL-WEAR-07** | Wearable cron heartbeat absent: no `wearable_cron_heartbeat{source=X, status="completed"}` in > 3h (1.5× expected 2h interval) | — | P1 | PagerDuty | Cron missed two runs; check Worker cron trigger in Cloudflare dashboard; dedup key: `wearable-cron-missed-{source}` |
| **AL-WEAR-08** | Per-enterprise-tenant wearable stale rate: > 30% of active wearable-connected users within a tenant have `sync_status = 'stale'` AND `last_reading_recorded_at < NOW() - INTERVAL '48 hours'` | — | P2 | Slack `#csm-alerts` | CSM-facing alert; tenant SLA may be at risk; dedup key: `wearable-stale-tenant-{tenant_id}` |

**Alert deduplication keys** use the same `dedup_key` pattern as §6 and §18 — PagerDuty suppresses re-alerts on the same key until the incident is resolved. AL-WEAR-07 and AL-WEAR-08 are routed to their respective channels with a 15-minute dedup window.

**AL-WEAR-08 privacy gate:** The alert payload for Slack `#csm-alerts` includes only: `tenant_name`, `source`, `stale_user_pct`, `stale_48h_user_count`. It must NOT include any individual `user_id` values or any wearable reading metadata. The Slack Worker that formats this alert must enforce this stripping in its serializer.

---

### 20.8 Enterprise Per-Tenant Wearable Fleet View

Enterprise HR and People Ops admins may access aggregate wearable adoption and sync health statistics via the admin dashboard. The privacy floor defined in `DATA_MODEL.md §14.4` (tenant admin never sees individual user data) applies equally here.

**`wearable_fleet_health` view** — refreshed daily at 05:30 UTC by `pg_cron` (30 minutes after `tenant_wearable_summary` at 05:00 UTC):

```sql
-- Exposes aggregate wearable sync health to tenant_admin role.
-- Privacy: no individual user_id, no health values, k-anonymity enforced (n >= 5).
CREATE MATERIALIZED VIEW wearable_fleet_health AS
SELECT
  tenant_id,
  source,
  COUNT(*)                                                    AS connected_users,
  COUNT(*) FILTER (WHERE sync_status = 'ok')                 AS users_syncing_ok,
  COUNT(*) FILTER (WHERE sync_status = 'stale')              AS users_stale,
  COUNT(*) FILTER (WHERE sync_status IN ('sync_failed',
    'token_expired', 'refresh_failed'))                       AS users_failing,
  COUNT(*) FILTER (WHERE sync_status = 'permission_lost')    AS users_permission_lost,

  -- Freshness percentiles: hours since last_reading_recorded_at
  PERCENTILE_CONT(0.50) WITHIN GROUP (
    ORDER BY EXTRACT(EPOCH FROM (NOW() - last_reading_recorded_at)) / 3600
  ) FILTER (WHERE last_reading_recorded_at IS NOT NULL)       AS median_hours_since_reading,

  PERCENTILE_CONT(0.90) WITHIN GROUP (
    ORDER BY EXTRACT(EPOCH FROM (NOW() - last_reading_recorded_at)) / 3600
  ) FILTER (WHERE last_reading_recorded_at IS NOT NULL)       AS p90_hours_since_reading,

  NOW()                                                       AS refreshed_at
FROM wearable_sync_health
WHERE sync_status != 'disabled'
GROUP BY tenant_id, source
HAVING COUNT(*) >= 5;  -- k-anonymity: suppress rows with < 5 connected users
```

**k-anonymity enforcement:** The `HAVING COUNT(*) >= 5` clause suppresses source-level rows for tenants with fewer than 5 connected users for that source. This prevents tenant-admin users from identifying individual employees' wearable adoption status in small teams.

**RLS on the materialized view:**

```sql
ALTER MATERIALIZED VIEW wearable_fleet_health OWNER TO form_system;

CREATE POLICY wearable_fleet_health_tenant_admin ON wearable_fleet_health
  AS PERMISSIVE FOR SELECT
  TO tenant_admin, tenant_owner
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
  );
```

**Admin dashboard widget spec — "Wearable Integration Health":**

| Panel element | Data source | Display |
|---|---|---|
| Source connection counts (per source) | `wearable_fleet_health.connected_users` | Horizontal bar chart; sorted by `connected_users DESC` |
| Sync status breakdown (per source) | `users_syncing_ok` / `users_stale` / `users_failing` | Stacked bar (green/yellow/red) |
| Median freshness (hours) | `median_hours_since_reading` | Gauge; green < 25h, yellow 25–48h, red > 48h |
| P90 freshness trend (14 days) | Daily snapshots from `wearable_fleet_health_snapshots` | Line chart; threshold line at 48h |
| Users needing attention | `users_permission_lost + users_failing` (if > 0) | Orange banner with count; clicking opens a modal with source name and error type only — no user identifiers |

The "users needing attention" modal is the one interface where an admin sees an actionable signal. It shows: `source`, `sync_status`, `error_code_last` (bucketed: `permission_denied`, `auth_failure`, `quota_exceeded`, `upstream_error`), and a recommended action. It never shows `user_id` or any identifying field.

---

### 20.9 DEC-030 HMAC-Chained Audit Events

Wearable sync lifecycle events are part of the DEC-030 HMAC chain per `docs/AUDIT_LOG_SCHEMA.md`. The following events are emitted by the sync Worker and validated by the weekly chain-integrity probe (§16: S-007 synthetic monitor).

| Event | Emitted when | Severity in chain | Notes |
|---|---|---|---|
| `wearable.source_connected` | User connects a wearable source | LOW | Includes `source`, `tenant_id`; no token content |
| `wearable.source_disconnected` | User disconnects a source | LOW | Includes `source`, `revoked_at`; triggers 24h erasure timer |
| `wearable.sync_completed` | Successful sync run produces ≥ 1 new reading | LOW | Count only: `new_readings_count` integer; no reading values |
| `wearable.sync_blocked` | Sync blocked by consent gate (`user_consents.categories_consented` does not include `wearable`) | MEDIUM | Requires investigation if unexpected |
| `wearable.token_refresh_failed` | OAuth token refresh returns non-2xx after retry | MEDIUM | `source`, `error_code`, `consecutive_failures`; no token content |
| `wearable.permission_revoked` | OS permission check returns denied (HealthKit/HC) | MEDIUM | Platform, app version; triggers push notification to user |

These six events are the same as those defined in `DATA_MODEL.md §14.7`. Their presence in the HMAC chain means any tampering with sync records (e.g., attempting to backdate a sync to mask a freshness SLA breach) would be detectable by the chain integrity probe.

---

### 20.10 Privacy Constraints Summary

| Constraint | What it prohibits | Where enforced |
|---|---|---|
| No raw health values in observability | HRV values, recovery scores, sleep durations, readiness scores — never in logs, metrics, or alert payloads | `wearable_sync_health` schema (§20.6); alert Worker serializer (AL-WEAR-08); Logpush exclusion list |
| No GROUP BY user_id in fleet queries | Individual wearable sync patterns are not aggregated per-user in any dashboard | `wearable_fleet_health` VIEW DDL; Metabase question template guards |
| k-anonymity floor n ≥ 5 | Per-source fleet stats suppressed for tenants with < 5 connected users | `HAVING COUNT(*) >= 5` in `wearable_fleet_health` VIEW |
| Tenant admin blocked from raw `wearable_sync_health` rows | Admin cannot enumerate which specific users have sync failures | RLS on `wearable_sync_health` (no tenant_admin policy); admin access only via VIEW |
| No health data in error codes | `error_code_last` field is drawn from a bounded vocabulary of operational error types; no reading values leak through error messages | `wearable_sync_health.error_code_last` CHECK constraint (implicit via Worker serializer) |
| DEC-030 audit events contain count-only reading metadata | `wearable.sync_completed` payload carries `new_readings_count` integer only — not reading types, values, or timestamps | Audit log emitter in sync Worker |

---

### 20.11 SOC 2 Evidence Mapping

| SOC 2 Criterion | How §20 Satisfies It |
|---|---|
| **CC7.2 — System monitoring** | `wearable_sync_health` provides continuous, timestamped monitoring of the wearable data pipeline. AL-WEAR-01 through AL-WEAR-08 are detective controls that fire on freshness, token, and upstream-health anomalies. The `wearable_fleet_health` materialized view gives auditors a queryable snapshot of per-tenant sync health at any point in the 90-day retention window. |
| **PI1.2 — Processing integrity inputs** | Freshness SLOs WEAR-SLO-01 through WEAR-SLO-05 quantify FORM's commitment to using current data inputs in Victor's readiness module. Stale data that feeds incorrect coaching output is a Processing Integrity risk; these SLOs are the formal control. |
| **C1.2 — Confidentiality protection of third-party data** | The privacy constraint matrix (§20.10) documents how FORM prevents wearable health data from flowing into operational dashboards, logs, or analytics pipelines. The audit log event design (count-only payloads) is the technical implementation of C1.2 for this pipeline. |
| **A1.1 — Availability commitments** | WEAR-SLO-06 (OAuth token validity ≥ 99.5%) and WEAR-SLO-07 (cron run completion ≥ 99%) are availability commitments for the wearable sync pipeline. Enterprise tenants relying on wearable data for their workforce wellness programme need these commitments documented and measured. |
| **CC9.2 — Third-party risk management** | Per-provider RED metrics (§20.1) and AL-WEAR-03 / AL-WEAR-07 are the detective controls for third-party API upstream failures (Whoop, Oura, Garmin). The `error_code_last` signal allows scoping an incident to a specific provider without requiring manual log search. |

**Auditor evidence artefacts:**
- Daily `wearable_fleet_health` snapshots stored to `compliance/evidence/wearable-fleet-YYYY-MM-DD.csv` (GitHub Actions schedule: 06:00 UTC).
- Monthly freshness SLO compliance report: per-source `WEAR-SLO-01` through `WEAR-SLO-05` compliance pct stored in `compliance/evidence/wearable-slo-YYYY-MM.csv`.
- DEC-030 `wearable.sync_blocked` events: any occurrence is high-signal for auditors (unexpected consent gate blocks) and exported to `compliance/evidence/wearable-sync-blocks/YYYY-MM.csv`.

---

### 20.12 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Write `migrations/YYYYMMDD_wearable_sync_health.sql`: table DDL, indexes, RLS policies, `pg_cron` 90-day retention cleanup | platform-engineer | P0 | M4 |
| Update wearable sync Workers (HealthKit background delivery handler, Health Connect WorkManager endpoint, Whoop/Oura/Garmin cron Worker) to upsert `wearable_sync_health` after every sync attempt | platform-engineer | P0 | M4 |
| Emit Workers Analytics Engine counters: `wearable_sync_success_total`, `wearable_sync_failures_total`, `wearable_sync_rate_limited_total`, `wearable_cron_heartbeat` per source | platform-engineer | P0 | M4 |
| Write `migrations/YYYYMMDD_wearable_fleet_health.sql`: materialized view DDL, RLS, `pg_cron` 05:30 UTC refresh | platform-engineer | P0 | M4 |
| Configure Better Stack WEAR-SLO-01 through WEAR-SLO-08 monitors against `wearable_sync_health` (SQL-based monitors polling every 30 min) | devops-lead | P0 | M4 |
| Configure PagerDuty routing for AL-WEAR-01, AL-WEAR-02, AL-WEAR-03, AL-WEAR-04, AL-WEAR-07 (P1 → wearable-oncall rotation; dedup keys per §20.7) | devops-lead | P0 | M4 |
| Configure Slack `#platform-alerts` routing for AL-WEAR-05, AL-WEAR-06 (P2 Slack alerts; 15-min dedup) | devops-lead | P1 | M4 |
| Configure Slack `#csm-alerts` routing for AL-WEAR-08 (P2 per-tenant stale alert; privacy-safe payload enforced — no user_id) | devops-lead | P1 | M4 |
| Add wearable cron heartbeat monitoring to Better Stack: alert if `wearable_cron_heartbeat{status="completed"}` absent for > 3h per source (AL-WEAR-07) | devops-lead | P0 | M4 |
| Implement `wearable_fleet_health_snapshots` daily append table (materialized view snapshot → append row) for 14-day trend chart in admin dashboard | platform-engineer | P1 | M5 |
| Build Metabase "Wearable Integration Health" admin dashboard widget per §20.8 spec; enforce k-anonymity display constraints in Metabase question parameters | data-engineer | P1 | M5 |
| Add daily `wearable_fleet_health` CSV export to `compliance/evidence/wearable-fleet-YYYY-MM-DD.csv` via GitHub Actions (06:00 UTC) | devops-lead | P2 | M5 |
| Add monthly WEAR-SLO freshness compliance report to `compliance/evidence/wearable-slo-YYYY-MM.csv` (first day of month, 08:00 UTC) | devops-lead | P2 | M5 |

---

*v0.7 additions: §20 Wearable Data Integration Observability — five-source wearable pipeline (HealthKit, Health Connect, Whoop, Oura Ring, Garmin) added to observability scope; privacy-first design established (no health values in any observability signal; count-only DEC-030 audit events; k-anonymity n ≥ 5 enforced); RED metrics table per provider with Workers Analytics Engine counter names; eight freshness and operational SLOs (WEAR-SLO-01 through WEAR-SLO-08) with differentiated targets per sync mechanism (HealthKit 95%, Health Connect 90%, Whoop/Oura 92%, Garmin 88%); OAuth token state machine with five states and transition-triggered alerting; sync architecture health signals per mechanism (HealthKit background delivery, Health Connect WorkManager, third-party API cron); `wearable_sync_health` Postgres table DDL (operational metadata only — timestamps, failure counts, status codes — no health values) with RLS blocking tenant_admin direct access; eight alerting rules AL-WEAR-01 through AL-WEAR-08 (P1 PagerDuty for fleet failures and upstream outages; P2 Slack for quota and per-tenant stale; privacy-safe AL-WEAR-08 payload spec); `wearable_fleet_health` materialized view with k-anonymity HAVING clause and tenant_admin RLS; admin dashboard "Wearable Integration Health" widget spec (five panels, no user_id exposure, error code bucketing); six DEC-030 HMAC-chained audit events aligned with DATA_MODEL.md §14.7; privacy constraints summary table (five constraints, enforcement location documented); SOC 2 CC7.2/PI1.2/C1.2/A1.1/CC9.2 evidence mapping; thirteen-item implementation checklist across M4/M5. Header corrected v0.5 → v0.7 (§19 addition had bumped doc to v0.6 without updating the header).*
