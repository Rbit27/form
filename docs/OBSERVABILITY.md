# FORM · Observability & Monitoring Taxonomy v0.1

> Owner: devops-lead. Review: quarterly or on architecture change. SOC 2 evidence: CC7.2.

---

## Purpose and Scope

This document defines FORM's observability strategy: what we measure, how we measure it, what triggers a human response, and how long we keep each signal. It is the authoritative reference for instrumenting new services, configuring alerts, building dashboards, and demonstrating monitoring controls to auditors.

Scope covers all production systems: Cloudflare Workers (edge API), Cloudflare Pages (marketing + admin), Supabase Postgres (primary database), Supabase Auth (consumer identity), Auth0 / WorkOS (enterprise SSO), Anthropic Claude API (Victor AI coach), ElevenLabs (voice synthesis), the iOS and Android mobile applications (React Native / Expo), and all dependent SaaS processors (PostHog, Sentry, ElevenLabs, Better Stack).

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
| **Per-tenant SLA reporting** | The Enterprise SLA dashboard (§8.4) depends on `tenant_id` label propagation in metrics, which is not yet implemented. | platform-engineer | M4 (enterprise launch) |
| **Sentry `beforeSend` health data filter** | The hook to strip health-adjacent fields from crash reports before Sentry transmission is not yet implemented. Blocking for GDPR compliance. | mobile-engineer | M2 |
| **Cost monitor Worker** | Daily cost aggregation from all billing APIs is not yet implemented. Cost anomaly alerting is not active. | devops-lead | M3 |
| **ClickHouse for analytics** | Planned from M9. Until then, PostHog and Supabase materialized views carry the analytics load. Observability strategy for ClickHouse (traces, product events at scale) is not yet designed. | platform-engineer | M9 |
| **Trace correlation between Sentry and Cloudflare** | Sentry transactions and Cloudflare trace IDs are not yet correlated. An error in Sentry cannot be mapped to a Cloudflare trace without manual log search. Requires shared `trace_id` propagation. | platform-engineer | M3 |
| **HMAC chain verification cron** | The weekly chain integrity cron described in `AUDIT_LOG_SCHEMA.md` is not yet deployed. Chain breaks would not be detected automatically. | devops-lead | M3 |
| **Audit log export pipeline** | Enterprise webhook delivery and S3 sync described in §4.6 are not yet implemented. Blocking for enterprise launch. | platform-engineer | M4 |

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

**v0.1 · May 2026 · Owner: devops-lead**
**Review: quarterly or on architecture change. Next scheduled review: August 2026.**
**SOC 2 evidence: CC7.2 (system monitoring). See also INCIDENT_RESPONSE.md for CC7.3–CC7.5.**
