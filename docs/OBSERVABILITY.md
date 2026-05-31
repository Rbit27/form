# FORM · Observability & Monitoring Taxonomy v1.2

> Owner: devops-lead. Review: quarterly or on architecture change. SOC 2 evidence: CC7.2.

---

## Purpose and Scope

This document defines FORM's observability strategy: what we measure, how we measure it, what triggers a human response, and how long we keep each signal. It is the authoritative reference for instrumenting new services, configuring alerts, building dashboards, and demonstrating monitoring controls to auditors.

Scope covers all production systems: Cloudflare Workers (edge API), Cloudflare Pages (marketing + admin), Supabase Postgres (primary database), Supabase Auth (consumer identity), Auth0 / WorkOS (enterprise SSO), Anthropic Claude API (Victor AI coach), ElevenLabs (voice synthesis), the iOS and Android mobile applications (React Native / Expo), the on-device CV pose estimation pipeline (Apple Vision / ML Kit), and all dependent SaaS processors (PostHog, Sentry, ElevenLabs, Better Stack).

**Multi-tenant note:** FORM operates a B2C consumer tier and a multi-tenant enterprise tier. Every observability concern must propagate `tenant_id` for enterprise contexts so that per-tenant SLA reporting, breach scoping, and isolation verification are possible from signal data alone.

**SOC 2 mapping:** This document is formal evidence for CC7.2 (the entity monitors system components for anomalies that could indicate a malicious act, a natural disaster, or an operational error). The alerting rules in §7, dashboard inventory in §8, and tooling decisions in §10 constitute the operating control evidence. See `docs/INCIDENT_RESPONSE.md` for CC7.3–CC7.5 evidence (response, recovery, and post-incident review).

**GDPR note:** FORM processes workout performance data, body composition trends, and health-adjacent metrics that fall within or close to GDPR Art. 9 special category data (health data). Observability data must not re-create raw health records. Log schemas in §5 reflect this constraint: AI inference logs capture latency and token counts, not prompt content or response content.

### Table of Contents

| Section | Title |
|---|---|
| §0 | RED Methodology |
| §1 | Three Pillars Overview |
| §2 | SLIs and SLOs |
| §3 | Metrics Taxonomy |
| §4 | Log Taxonomy |
| §5 | Distributed Traces |
| §6 | Alerting Rules |
| §7 | Dashboards |
| §8 | Data Retention and Privacy |
| §9 | Tooling Decisions |
| §10 | Open Questions and Gaps |
| §11 | SLI Calculation Expressions |
| §12 | Developer Instrumentation Guide |
| §13 | Per-Tenant Observability Implementation |
| §14 | Cross-Backend Trace Correlation (Sentry ↔ Cloudflare) |
| §15 | Audit Log Export Pipeline (Enterprise) |
| §16 | Synthetic Monitoring & Uptime Verification |
| §17 | Cost Monitoring & Anomaly Alerting |
| §18 | CV Pose Estimation Model Observability |
| §19 | SLO Error Budget Management |
| §20 | Wearable Data Integration Observability |
| §21 | ClickHouse Analytics Observability (M9) |
| §22 | AI Coaching Quality Observability |
| §23 | Enterprise SLA Reporting & Credit Calculation |
| §24 | Subscription & Revenue Event Observability |
| §25 | Product & Activation Funnel Observability |

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

---

## 21. ClickHouse Analytics Observability (M9)

> Owner: data-engineer + platform-engineer. Review: on migration trigger or quarterly post-M9. SOC 2 evidence: CC7.2, A1.1. Cross-reference: `docs/COST_MODEL.md §13`, `docs/DATA_MODEL.md §17`, `docs/ENTERPRISE.md`.

This section closes the gap documented in §10 ("ClickHouse for analytics", M9). It designs the observability strategy for ClickHouse once FORM migrates product analytics and distributed traces from PostHog + Supabase materialized views to a self-hosted ClickHouse cluster. The section covers: (1) migration trigger thresholds, (2) RED metrics for ClickHouse itself, (3) the OpenTelemetry Collector → ClickHouse trace pipeline, (4) product event table schemas, (5) SLOs for analytics query latency, (6) privacy constraints, and (7) the migration runbook from PostHog.

---

### 21.1 Migration Trigger Thresholds

ClickHouse is not deployed by default. PostHog and Supabase materialized views are the analytics stack through M8. Migration to ClickHouse is triggered when **any two** of the following thresholds are breached simultaneously, or when **any one** P0 threshold fires:

| Threshold | Type | Trigger value | Rationale |
|---|---|---|---|
| **PostHog monthly cost** | P0 trigger | > $2,000/month | PostHog scale plan; at ~300k MAU PostHog exceeds $2k/month and ClickHouse self-hosting becomes cost-superior |
| **Analytics query P95 latency** | P0 trigger | > 10 seconds on core dashboards | Materalized view lag in Postgres degrades admin dashboard; ClickHouse columnar engine resolves this |
| **MAU** | Advisory | ≥ 200,000 | Leading indicator — schedule M9 sprint before PostHog costs hit P0 |
| **Supabase DB size** | Advisory | ≥ 50 GB | Analytics tables crowding transactional DB; time to separate OLAP from OLTP |
| **Enterprise tenant count** | Advisory | ≥ 30 tenants | Per-tenant analytics isolation becomes critical; ClickHouse TTL + tenant partitioning is cleaner than Postgres RLS for analytics workloads |
| **Trace volume** | Advisory | > 50M spans/month | Sentry and Cloudflare logs become expensive at this volume; ClickHouse self-hosted traces store is cost-superior |

**Monitoring the trigger thresholds:** These thresholds are themselves monitored:

```sql
-- Better Stack SQL monitor: PostHog cost proxy (tracked via cost_daily table from §17)
SELECT daily_cost_usd
FROM cost_daily
WHERE service = 'posthog'
  AND recorded_date = CURRENT_DATE - 1;
-- Alert: daily_cost_usd > 67 (= $2,000/month ÷ 30)
```

The MAU counter (`SELECT COUNT(DISTINCT user_id) FROM sessions WHERE created_at > NOW() - INTERVAL '30 days'`) is surfaced in the Metabase "Growth Metrics" dashboard and reviewed monthly. The M9 sprint is scheduled when MAU crosses 150,000 — 25% headroom before the P0 threshold.

---

### 21.2 ClickHouse Architecture Overview

```
Mobile App / Worker API
        │
        │  (1) Product events: HTTP POST → Cloudflare Worker event collector
        │  (2) Traces: W3C traceparent → OpenTelemetry Collector
        ▼
┌──────────────────────────────────────────────────────────────────┐
│  Cloudflare Worker: analytics-ingest                             │
│  • Validates tenant_id from JWT                                  │
│  • Strips PII fields (user_id → user_id_hash; no health values)  │
│  • Writes to: Cloudflare Queues → ClickHouse HTTP ingest endpoint │
└──────────────────────────────────────────────────────────────────┘
        │
        │  Cloudflare Queue (durable, at-least-once delivery)
        ▼
┌──────────────────────────────────────────────────────────────────┐
│  ClickHouse Cloud (EU-Central region for GDPR residency)         │
│  • Database: form_analytics                                       │
│  • Tables: events, traces, sessions, cost_daily_ch               │
│  • Engine: MergeTree family; partitioned by toYYYYMM(timestamp)  │
│  • TTL: configurable per table (see §21.4)                        │
└──────────────────────────────────────────────────────────────────┘
        │
        ├──► Metabase (reads via ClickHouse JDBC driver)
        └──► Grafana (reads via ClickHouse data source plugin)
```

**Deployment choice: ClickHouse Cloud vs self-hosted.** FORM uses ClickHouse Cloud (EU-Central) rather than a self-hosted cluster for the following reasons: (a) no Kubernetes operational overhead at pre-Series B headcount, (b) built-in S3-backed storage tiering, (c) SOC 2 Type II certified by ClickHouse Inc., which simplifies FORM's sub-processor register, (d) GDPR-compliant DPA available. At ≥ $10k/month ClickHouse Cloud spend, re-evaluate self-hosted on GCP/Hetzner EU. This threshold is modeled in `docs/COST_MODEL.md §13.8`.

---

### 21.3 RED Metrics for ClickHouse Itself

ClickHouse is itself a monitored service in FORM's observability stack. The standard RED framework applies:

| Signal | Metric name | Target | Alert threshold |
|---|---|---|---|
| **Rate** | `ch_queries_per_second` (from `system.query_log`) | — | Spike > 3× 7-day average → P2 Slack |
| **Errors** | `ch_failed_queries_ratio` = failed / total from `system.query_log` | < 0.5% | > 1% over 5 min → P1 PagerDuty |
| **Duration — analytics** | `ch_query_duration_p95_ms` for `form_analytics.*` queries | < 5,000 ms | > 10,000 ms → P1 (SLO breach) |
| **Duration — trace queries** | `ch_query_duration_p95_ms` for `traces.*` queries | < 2,000 ms | > 5,000 ms → P2 Slack |
| **Ingest lag** | `ch_ingest_lag_seconds` = `now() - max(timestamp)` in `events` table | < 30 s | > 120 s → P1 (data freshness SLO) |
| **Queue depth** | `cf_queue_depth{queue="analytics-ingest"}` from Cloudflare Analytics | < 10,000 messages | > 50,000 → P1 |
| **Storage** | `ch_disk_usage_bytes` | < 80% of provisioned | > 85% → P2, > 95% → P0 (data loss risk) |
| **Memory** | `ch_memory_usage_ratio` | < 70% | > 85% → P2 |

These metrics are sourced from the ClickHouse Cloud monitoring API (scraped every 60 seconds by a Cloudflare Cron Worker) and stored in the existing `cost_daily` Postgres table for cost metrics, and forwarded to Better Stack for alerting.

```typescript
// Cloudflare Cron Worker: ch-monitor (runs every 60 seconds)
const chMetrics = await fetch(`${CH_CLOUD_API}/metrics`, {
  headers: { 'X-ClickHouse-User': env.CH_MONITOR_USER, 'X-ClickHouse-Key': env.CH_MONITOR_KEY }
});
// Scrapes system.query_log aggregate, system.disks, system.asynchronous_metrics
// Posts to Better Stack heartbeat + posts anomalies to #platform-alerts
```

**ClickHouse alerting rules (Better Stack):**

| Alert ID | Condition | Severity | Routing |
|---|---|---|---|
| AL-CH-01 | `ch_failed_queries_ratio > 0.01` for 5 min | P1 | PagerDuty: platform-oncall |
| AL-CH-02 | `ch_query_duration_p95_ms > 10000` (analytics) for 5 min | P1 | PagerDuty + Slack #platform-alerts |
| AL-CH-03 | `ch_ingest_lag_seconds > 120` | P1 | PagerDuty: platform-oncall |
| AL-CH-04 | `cf_queue_depth > 50000` for 3 min | P1 | PagerDuty: platform-oncall |
| AL-CH-05 | `ch_disk_usage_bytes > 0.95 * provisioned` | P0 | PagerDuty: founder + platform-oncall |
| AL-CH-06 | `ch_disk_usage_bytes > 0.85 * provisioned` | P2 | Slack #platform-alerts |
| AL-CH-07 | `ch_memory_usage_ratio > 0.85` for 10 min | P2 | Slack #platform-alerts |
| AL-CH-08 | ClickHouse Cloud API health check fails for 2 consecutive probes | P0 | PagerDuty: founder + platform-oncall |

---

### 21.4 OpenTelemetry Collector → ClickHouse Trace Pipeline

FORM's distributed traces (§5) are exported to ClickHouse alongside Sentry. ClickHouse becomes the long-term trace store (90-day retention) while Sentry remains the primary developer debugging tool (14-day retention). This separation keeps Sentry costs bounded while preserving traces for compliance and SLO evidence.

**Collector topology:**

```
Cloudflare Worker (edge)
    │  OTLP/HTTP → OpenTelemetry Collector (deployed on Fly.io EU region)
    ▼
┌─────────────────────────────────────────────────────┐
│  OpenTelemetry Collector v0.100+                     │
│  Receivers:  otlp (HTTP + gRPC)                      │
│  Processors: batch (8192 spans, 5s timeout)          │
│              memory_limiter (80% heap → drop oldest) │
│              attributes/tenant_pii_filter            │
│              resource (add deployment.environment)   │
│  Exporters:  clickhouse (primary, 90d retention)     │
│              sentry (secondary, 14d retention)       │
│              otlphttp/better_stack (SLO breach spans)│
└─────────────────────────────────────────────────────┘
```

**Collector config (abbreviated):**

```yaml
processors:
  attributes/tenant_pii_filter:
    actions:
      # Never store user_id — only the hash
      - key: user_id
        action: delete
      # Truncate prompt content if accidentally included
      - key: ai.prompt_content
        action: delete
      - key: ai.response_content
        action: delete
      # Preserve these for multi-tenant isolation queries
      - key: tenant_id
        action: upsert
        value: "${env:OTEL_RESOURCE_TENANT_ID}"

exporters:
  clickhouse:
    endpoint: "${env:CH_CLOUD_ENDPOINT}"
    username: "${env:CH_OTEL_USER}"
    password: "${env:CH_OTEL_KEY}"
    database: form_analytics
    traces_table_name: traces
    ttl: 2160h   # 90 days
    compress: lz4
    timeout: 10s
    retry_on_failure:
      enabled: true
      initial_interval: 5s
      max_interval: 30s
      max_elapsed_time: 300s
```

**ClickHouse `traces` table (ClickHouse-native OTLP schema):**

```sql
CREATE TABLE form_analytics.traces (
    Timestamp           DateTime64(9) CODEC(Delta, ZSTD(1)),
    TraceId             String CODEC(ZSTD(1)),
    SpanId              String CODEC(ZSTD(1)),
    ParentSpanId        String CODEC(ZSTD(1)),
    TraceState          String CODEC(ZSTD(1)),
    SpanName            LowCardinality(String) CODEC(ZSTD(1)),
    SpanKind            LowCardinality(String) CODEC(ZSTD(1)),
    ServiceName         LowCardinality(String) CODEC(ZSTD(1)),
    ResourceAttributes  Map(LowCardinality(String), String) CODEC(ZSTD(1)),
    SpanAttributes      Map(LowCardinality(String), String) CODEC(ZSTD(1)),
    Duration            Int64 CODEC(Delta, ZSTD(1)),  -- nanoseconds
    StatusCode          LowCardinality(String) CODEC(ZSTD(1)),
    StatusMessage       String CODEC(ZSTD(1)),
    -- Promoted keys for fast filtering:
    tenant_id           LowCardinality(String) MATERIALIZED SpanAttributes['tenant_id'],
    route               LowCardinality(String) MATERIALIZED SpanAttributes['http.route'],
    http_status         UInt16 MATERIALIZED toUInt16OrZero(SpanAttributes['http.status_code']),
    ai_feature          LowCardinality(String) MATERIALIZED SpanAttributes['ai.feature'],
    ai_input_tokens     UInt32 MATERIALIZED toUInt32OrZero(SpanAttributes['ai.input_tokens']),
    ai_output_tokens    UInt32 MATERIALIZED toUInt32OrZero(SpanAttributes['ai.output_tokens'])
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(Timestamp)
ORDER BY (tenant_id, Timestamp, TraceId)
TTL Timestamp + INTERVAL 90 DAY DELETE
SETTINGS index_granularity = 8192;
```

**Privacy invariant:** The `user_id` field is deleted at the Collector processor stage before any span reaches ClickHouse. Only `user_id_hash` (SHA-256, keyed HMAC) is stored. Spans containing raw user data that accidentally escape the Collector processor generate an `otel_pii_leak_detected` alert (see AL-CH-09 below).

**Trace query examples (Grafana Explore):**

```sql
-- P95 latency by route for the last 24 hours
SELECT
    route,
    quantile(0.95)(Duration / 1e6) AS p95_ms,
    COUNT() AS span_count
FROM form_analytics.traces
WHERE Timestamp >= now() - INTERVAL 24 HOUR
  AND SpanName = 'http_request'
GROUP BY route
ORDER BY p95_ms DESC;

-- Error rate by tenant (for enterprise SLA report)
SELECT
    tenant_id,
    COUNTIf(http_status >= 500) AS error_count,
    COUNT() AS total_count,
    error_count / total_count AS error_rate
FROM form_analytics.traces
WHERE Timestamp >= now() - INTERVAL 7 DAY
  AND SpanName = 'http_request'
  AND tenant_id != ''
GROUP BY tenant_id
HAVING total_count > 100
ORDER BY error_rate DESC;
```

---

### 21.5 Product Event Table Schema

Product events migrate from PostHog's cloud-hosted store to ClickHouse after the M9 trigger. The schema is designed to be PostHog-export compatible (same event names, same property keys) so that historical data can be bulk-loaded.

```sql
CREATE TABLE form_analytics.events (
    event_id            UUID DEFAULT generateUUIDv4(),
    timestamp           DateTime64(3, 'UTC') CODEC(Delta, ZSTD(1)),
    event               LowCardinality(String) CODEC(ZSTD(1)),
    -- Identity: never raw user_id, always hash
    user_id_hash        FixedString(64) CODEC(ZSTD(1)),  -- HMAC-SHA256 hex
    session_id          String CODEC(ZSTD(1)),
    -- Multi-tenant context
    tenant_id           LowCardinality(String) CODEC(ZSTD(1)),
    -- Platform
    platform            LowCardinality(String) CODEC(ZSTD(1)),  -- 'ios'|'android'|'web'
    app_version         LowCardinality(String) CODEC(ZSTD(1)),
    -- Dimensions (non-PII only; health values never stored here)
    properties          Map(String, String) CODEC(ZSTD(1)),
    -- Promoted properties for fast aggregation
    screen              LowCardinality(String) MATERIALIZED properties['screen'],
    feature             LowCardinality(String) MATERIALIZED properties['feature'],
    workout_type        LowCardinality(String) MATERIALIZED properties['workout_type'],
    -- Trace linkage
    trace_id            String CODEC(ZSTD(1)),
    -- Ingest metadata
    ingest_timestamp    DateTime64(3, 'UTC') DEFAULT now64(3) CODEC(Delta, ZSTD(1))
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(timestamp)
ORDER BY (tenant_id, user_id_hash, timestamp)
TTL timestamp + INTERVAL 2 YEAR DELETE   -- 2-year product analytics retention
SETTINGS index_granularity = 8192;
```

**Event ingest pipeline:**

```
Mobile App / Web
    │  POST /api/analytics/track  (Cloudflare Worker: analytics-ingest)
    │  Body: { event, properties, session_id }
    │  Auth: JWT (extracts user_id → user_id_hash via HMAC, tenant_id)
    ▼
Worker validates + strips PII:
    - Removes: email, name, raw user_id, any health metric values
    - Enforces: properties keys must be in ALLOWED_PROPERTY_KEYS allowlist
    - Adds: timestamp, user_id_hash, tenant_id, platform, app_version
    │
    │  Cloudflare Queues (analytics-ingest) — durable buffer
    ▼
Queue consumer Worker (batch mode, max 250 events, 5s timeout):
    - Bulk inserts to ClickHouse via HTTP interface
    - On failure: dead-letter queue → alert AL-CH-03
```

**Allowed property keys allowlist (enforced in analytics-ingest Worker):**

Health-adjacent properties (HRV values, body weight, body fat, mood scores, injury flags, eating pattern signals) are **never** allowed in the events table. The Worker rejects any `properties` key that matches the `BLOCKED_PROPERTY_KEYS` deny-list. This is enforced at ingest, not at query time, to prevent inadvertent storage.

```typescript
const BLOCKED_PROPERTY_KEYS = new Set([
  'hrv', 'hrv_rmssd', 'body_weight', 'body_fat_pct',
  'mood_score', 'stress_level', 'sleep_duration',
  'sleep_quality', 'injury_flag', 'pain_level',
  'calorie_intake', 'calorie_deficit', 'bmi',
  'readiness_score', 'recovery_score',
]);

function sanitizeProperties(raw: Record<string, unknown>): Record<string, string> {
  const out: Record<string, string> = {};
  for (const [k, v] of Object.entries(raw)) {
    if (BLOCKED_PROPERTY_KEYS.has(k)) continue;  // silently drop
    out[k] = String(v).slice(0, 512);  // cap value length
  }
  return out;
}
```

---

### 21.6 Session Table Schema

Sessions are derived from events but stored as a separate aggregated table for efficient cohort analysis. They do not contain health data.

```sql
CREATE TABLE form_analytics.sessions (
    session_id          String CODEC(ZSTD(1)),
    user_id_hash        FixedString(64) CODEC(ZSTD(1)),
    tenant_id           LowCardinality(String) CODEC(ZSTD(1)),
    platform            LowCardinality(String) CODEC(ZSTD(1)),
    app_version         LowCardinality(String) CODEC(ZSTD(1)),
    started_at          DateTime64(3, 'UTC') CODEC(Delta, ZSTD(1)),
    ended_at            DateTime64(3, 'UTC') CODEC(Delta, ZSTD(1)),
    duration_seconds    UInt32,
    event_count         UInt32,
    first_screen        LowCardinality(String) CODEC(ZSTD(1)),
    -- Derived engagement signals (non-health)
    used_ai_coach       UInt8,  -- boolean
    used_voice          UInt8,
    used_cv             UInt8,
    workout_started     UInt8,
    workout_completed   UInt8
)
ENGINE = ReplacingMergeTree(ended_at)
PARTITION BY toYYYYMM(started_at)
ORDER BY (tenant_id, user_id_hash, session_id)
TTL started_at + INTERVAL 2 YEAR DELETE;
```

Sessions are materialized by a Cloudflare Cron Worker that closes open sessions (no event for > 30 minutes) and upserts the `ReplacingMergeTree` table. The `ReplacingMergeTree` engine ensures idempotency: if the cron runs twice, the row with the latest `ended_at` wins.

---

### 21.7 SLOs for ClickHouse-Backed Analytics

| SLO ID | Description | Target | Measurement window | Error budget (30d) |
|---|---|---|---|---|
| **CH-SLO-01** | Product event ingest lag < 30 s (P95) | 99.5% | 5-min rolling | 3.6 hours |
| **CH-SLO-02** | Analytics query P95 latency < 5,000 ms | 99.0% | 1-hour rolling | 7.2 hours |
| **CH-SLO-03** | Trace query P95 latency < 2,000 ms | 99.5% | 1-hour rolling | 3.6 hours |
| **CH-SLO-04** | ClickHouse availability (AL-CH-08 not firing) | 99.9% | Monthly | 43.8 minutes |
| **CH-SLO-05** | No PII leak events in 30-day rolling window | 100% | 30-day | Zero tolerance |
| **CH-SLO-06** | Ingest queue depth < 10,000 messages (P95 of 5-min readings) | 99.0% | Daily | 14.4 minutes/day |

**Error budget burn rate thresholds (CH-SLO-01, CH-SLO-02):**

| Burn rate | Time to budget exhaustion | Action |
|---|---|---|
| 1× | 30 days (normal) | No action |
| 5× | 6 days | P2 Slack alert → investigate |
| 14.4× | 50 hours | P1 PagerDuty → ClickHouse sizing review |

---

### 21.8 Per-Tenant Analytics Isolation

Enterprise tenants require that their analytics data cannot be queried across tenant boundaries by FORM employees.

**Access control model for ClickHouse:**

| Role | Tables accessible | Row filter |
|---|---|---|
| `ch_platform_engineer` | All tables (system + form_analytics) | None — full access for ops |
| `ch_data_engineer` | `form_analytics.*` | None — aggregate queries allowed; user_id_hash never joined back to identity |
| `ch_tenant_readonly_{tenant_id}` | `form_analytics.events`, `form_analytics.sessions` | `WHERE tenant_id = '{tenant_id}'` enforced via Row Policy |
| `ch_audit_reader` | `form_analytics.traces` | `WHERE tenant_id = '{tenant_id}'` — for enterprise audit log exports |

**ClickHouse Row Policy (per-tenant isolation):**

```sql
-- Created per enterprise tenant at onboarding
CREATE ROW POLICY rp_tenant_{tenant_id}
ON form_analytics.events
FOR SELECT
USING tenant_id = '{tenant_id}'
TO ch_tenant_readonly_{tenant_id};

CREATE ROW POLICY rp_tenant_{tenant_id}
ON form_analytics.sessions
FOR SELECT
USING tenant_id = '{tenant_id}'
TO ch_tenant_readonly_{tenant_id};
```

**Breaking glass (cross-tenant query for incident response):** Any query run by `ch_platform_engineer` or `ch_data_engineer` that returns rows from more than one `tenant_id` must be pre-authorized via the break-glass process (`docs/INCIDENT_RESPONSE.md §7`). ClickHouse query logs are exported to the DEC-030 HMAC chain as `analytics.cross_tenant_query` events. Enterprise tenants are notified within 24 hours if their data is accessed cross-tenant.

---

### 21.9 PostHog Migration Runbook

The migration from PostHog (cloud-hosted) to ClickHouse is a one-time operation executed when M9 triggers fire. It preserves historical data and switches ingest atomically.

**Phase 1 — Parallel ingest (2 weeks):**

1. Deploy ClickHouse cluster and run `migrations/M9_clickhouse_schema.sql`.
2. Configure analytics-ingest Worker to dual-write: PostHog (primary) and ClickHouse (secondary).
3. Verify ClickHouse event counts match PostHog event counts within 0.1% for 7 consecutive days.
4. Validate `user_id_hash` round-trip: confirm ClickHouse hash matches PostHog distinct_id hash for the same user.

**Phase 2 — Historical data export (1 week):**

1. Export PostHog events via PostHog Batch Export to S3 (EU-Central bucket).
2. Transform PostHog JSON export to ClickHouse TSV format via Cloudflare Worker (streaming, no intermediate storage of identifiable data).
3. Bulk load to ClickHouse via `INSERT INTO form_analytics.events SELECT ... FROM s3(...)`.
4. Delete S3 export files after successful load verification.

**Phase 3 — Cutover:**

1. Disable PostHog dual-write; ClickHouse becomes the single ingest target.
2. Update Metabase data sources: replace PostHog data source with ClickHouse JDBC.
3. Rebuild enterprise analytics dashboards against ClickHouse queries.
4. Notify PostHog account team; cancel subscription at next billing cycle.
5. Retain PostHog read access for 30 days post-cutover for comparison queries.

**Rollback trigger:** If ClickHouse ingest lag CH-SLO-01 breaches > 60 s for > 15 minutes post-cutover, revert to PostHog dual-write and page platform-engineer. Root cause must be documented before re-attempting cutover.

**PostHog data deletion after migration:**

All PostHog data must be deleted within 30 days of successful migration. PostHog's "Delete all events for a person" API is used to delete all events associated with each `distinct_id`. This action is logged as a `compliance.posthog_data_deletion` DEC-030 audit event. GDPR compliance note: this deletion satisfies FORM's obligation to minimize data retention at processors (Art. 5(1)(e) storage limitation).

---

### 21.10 Privacy Constraints Summary

| Constraint | What it prohibits | Where enforced |
|---|---|---|
| No raw `user_id` in ClickHouse | User identity stored only as HMAC-keyed hash; raw user_id never reaches ClickHouse | analytics-ingest Worker; OTel Collector processor `attributes/tenant_pii_filter` |
| No health metric values in events | HRV, body weight, sleep scores, readiness, pain, calorie intake — never in `properties` | `BLOCKED_PROPERTY_KEYS` deny-list enforced in analytics-ingest Worker at ingest |
| Cross-tenant queries require break-glass | `ch_data_engineer` queries spanning multiple `tenant_id` values must be pre-authorized | ClickHouse query log exported to DEC-030 HMAC chain; automated detection via nightly audit query |
| k-anonymity floor n ≥ 15 for enterprise reports | Per-tenant analytics cohorts below 15 users suppressed in Metabase dashboards | Metabase question parameter guard; same floor as `docs/DATA_MODEL.md §17.4` |
| Event property value length cap | Values > 512 characters truncated at ingest to prevent encoding of large blobs | analytics-ingest Worker `sanitizeProperties()` |
| No ClickHouse data in AI inference context | ClickHouse query results never used as Victor context (no health data flow back to LLM) | Architecture constraint; Victor reads only from Supabase Postgres via RLS |

**AL-CH-09 — PII leak detection:**

A nightly Cloudflare Cron Worker scans a sample of ClickHouse `events` rows (1,000 random rows from the previous 24 hours) for patterns matching the `BLOCKED_PROPERTY_KEYS` list in property values. If a match is found, AL-CH-09 fires as P0:

```
PagerDuty: FORM · ClickHouse PII Leak Detected
Severity: P0
Description: Property key or value matching PII pattern found in form_analytics.events.
Immediate action: Halt analytics ingest Worker. Quarantine affected rows. Begin data breach assessment per INCIDENT_RESPONSE.md R-01.
```

---

### 21.11 SOC 2 Evidence Mapping

| SOC 2 Criterion | How §21 Satisfies It |
|---|---|
| **CC7.2 — System monitoring** | ClickHouse RED metrics (§21.3) and alerting rules AL-CH-01 through AL-CH-09 are detective controls monitoring the analytics pipeline for anomalies. The daily audit query detecting cross-tenant access is a continuous monitoring control. |
| **CC6.1 — Logical access controls** | Per-tenant ClickHouse Row Policies (§21.8) enforce tenant isolation at the query engine layer. Role taxonomy (platform_engineer, data_engineer, tenant_readonly) with least-privilege assignments. |
| **CC6.2 — Access provisioning** | ClickHouse roles provisioned at enterprise onboarding; deprovisioned via SCIM offboarding webhook. Role grants logged to DEC-030 as `analytics.ch_role_granted` / `analytics.ch_role_revoked`. |
| **C1.2 — Confidentiality protection** | `BLOCKED_PROPERTY_KEYS` deny-list (§21.5) and OTel Collector PII filter (§21.4) prevent health data entering the analytics store. Privacy constraints matrix (§21.10) documents the technical implementation. |
| **PI1.2 — Processing integrity inputs** | Ingest lag SLO CH-SLO-01 (< 30 s, 99.5%) quantifies FORM's commitment to analytics data freshness. The dual-write migration validation (§21.9 Phase 1) ensures historical data integrity during migration. |
| **A1.1 — Availability commitments** | CH-SLO-04 (99.9% ClickHouse availability) is a documented availability commitment. ClickHouse Cloud's SLA (99.9% uptime) is contractually inherited; FORM's own SLO does not exceed the vendor SLA. |
| **CC9.2 — Third-party risk management** | ClickHouse Inc. is added to FORM's sub-processor register (`docs/SOC2_READINESS.md §Sub-Processor Register`) at M9. ClickHouse Cloud's SOC 2 Type II report is obtained annually and stored in `compliance/vendor-reports/clickhouse/`. |

**Auditor evidence artefacts:**

- Nightly PII scan results stored to `compliance/evidence/clickhouse-pii-scan-YYYY-MM-DD.json` (GitHub Actions, 02:00 UTC). Every scan result, including zero-finding results, is retained.
- Monthly ClickHouse query log export filtered to `ch_platform_engineer` and `ch_data_engineer` cross-tenant queries, stored to `compliance/evidence/clickhouse-cross-tenant-YYYY-MM.csv`.
- CH-SLO-01 through CH-SLO-06 monthly compliance report stored to `compliance/evidence/clickhouse-slo-YYYY-MM.csv`.
- ClickHouse user list snapshot (first day of month) stored to `compliance/evidence/clickhouse-users-YYYY-MM-DD.csv` for access review evidence.

---

### 21.12 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Monitor M9 trigger thresholds: add PostHog daily cost to `cost_daily` table (§21.1) and configure Better Stack alert at $67/day | devops-lead | P0 | M7 (pre-M9 headroom) |
| Monitor MAU threshold: add `mau_rolling_30d` to Metabase Growth Metrics dashboard; schedule M9 sprint when MAU ≥ 150,000 | data-engineer | P0 | M7 |
| Provision ClickHouse Cloud cluster (EU-Central); configure DPA with ClickHouse Inc.; add to sub-processor register | compliance-officer | P0 | M9 sprint start |
| Write `migrations/M9_clickhouse_schema.sql`: `form_analytics` database, `traces`, `events`, `sessions` tables with TTL and partitioning per §21.4–21.6 | platform-engineer | P0 | M9 sprint start |
| Deploy OpenTelemetry Collector on Fly.io (EU region); configure receivers, PII filter processor, ClickHouse + Sentry exporters per §21.4 | platform-engineer | P0 | M9 sprint |
| Update analytics-ingest Worker to dual-write PostHog + ClickHouse; implement `BLOCKED_PROPERTY_KEYS` deny-list and `sanitizeProperties()` per §21.5 | platform-engineer | P0 | M9 sprint |
| Configure ClickHouse Row Policies per enterprise tenant (provisioning script in `scripts/ch-tenant-provision.sh`) per §21.8 | platform-engineer | P0 | M9 sprint |
| Configure Better Stack alerts AL-CH-01 through AL-CH-08 against ClickHouse Cloud monitoring API per §21.3 | devops-lead | P0 | M9 sprint |
| Run Phase 1 parallel ingest for 7 days; validate event count parity within 0.1% per §21.9 Phase 1 | data-engineer | P0 | M9 sprint +7d |
| Execute PostHog bulk export + ClickHouse load per §21.9 Phase 2; delete S3 export files after verification | data-engineer | P1 | M9 sprint +14d |
| Cutover: disable PostHog dual-write; update Metabase data sources per §21.9 Phase 3 | platform-engineer | P0 | M9 sprint +21d |
| Build AL-CH-09 PII leak scan (Cloudflare Cron, 02:00 UTC); store results to `compliance/evidence/clickhouse-pii-scan-YYYY-MM-DD.json` | devops-lead | P0 | M9 sprint +21d |
| Write cross-tenant query audit cron: nightly export to `compliance/evidence/clickhouse-cross-tenant-YYYY-MM.csv` | devops-lead | P1 | M9 sprint +21d |
| Delete PostHog event data via PostHog API (all distinct_ids); log as `compliance.posthog_data_deletion` DEC-030 event | compliance-officer | P0 | M9 sprint +30d |
| Add CH-SLO-01 through CH-SLO-06 to quarterly SLO compliance report cadence (§19) | devops-lead | P1 | M9 sprint +30d |
| Add ClickHouse Inc. to annual vendor security review cycle (`docs/SOC2_READINESS.md §17`) | compliance-officer | P1 | M9 sprint +30d |

---

## 22. AI Coaching Quality Observability

> Owner: ml-engineer + platform-engineer. Review: monthly during beta; quarterly post-launch. SOC 2 evidence: CC5.2 (control activities), CC7.2 (monitoring), CC7.4 (evaluation of deficiencies), CC1.2 (integrity and ethical values). Cross-reference: `docs/INCIDENT_RESPONSE.md` R-10 (AI Coach Safety Incident), `docs/VICTOR_PROMPT_GUIDE.md`, `docs/DATA_MODEL.md` §11 (`coaching_turns`).

**Privacy-first constraint:** Victor's coaching conversations contain GDPR Art. 9-adjacent health data (training performance, self-reported readiness, injury context). This section defines observability through **behavioral proxy metrics only** — signals that indicate coaching quality without storing or transmitting prompt or response content. No coaching turn text, no Victor response text, and no user message text appears in any observability signal defined here.

---

### 22.1 Purpose: Operational vs Quality Observability

Section §3 (Metrics Taxonomy) and §4.5 (AI Inference Log Schema) cover **operational observability** for the Anthropic API integration: latency, token counts, error rates, cache efficiency. Those signals answer "Is the system working?" They do not answer "Is Victor coaching well?"

This section defines **quality observability** — signals that answer:
- Is coaching engagement trending up or down?
- Did a prompt change regress workout plan adherence?
- Are there fleet-level signals suggesting Victor is giving unhelpful or harmful guidance?
- Is a new prompt version performing better or worse than the baseline?

Quality observability is harder than operational observability because the ground truth (coaching quality) is subjective, proxied through behavioral outcomes, and cannot be inferred from content (GDPR constraint). The signals defined here are leading indicators, not direct measurements. They must be interpreted together, never in isolation.

**Scope:** Victor coaching interactions on consumer iOS/Android app (Growth and Pro tiers). Enterprise tenant coaching is in scope for fleet aggregates; tenant-level quality drilldown is available only to tenant admins with DPA-appropriate consent.

**Out of scope:** Storing any portion of coaching conversation text for quality review. Anthropic's safety evaluations and model-level alignment monitoring are Anthropic's responsibility and are not duplicated here. Content moderation for individual messages is handled at the application layer (INCIDENT_RESPONSE.md R-10), not here.

---

### 22.2 Coaching Quality Proxy Metric Taxonomy

All metrics are computed at the **coaching session** or **weekly user cohort** level. No metric is computed at the individual message level (prevents content reconstruction).

| Metric | Name | Computation Source | Retention | What it proxies |
|---|---|---|---|---|
| **Plan adherence rate** | `victor_plan_adherence_rate` | `workout_logs` vs `training_plan_weeks` | 24 months | Did users follow the workout Victor prescribed? |
| **Coaching session depth** | `victor_session_depth_turns` | `COUNT(*) GROUP BY session_id` in `coaching_turns` | 90 days | Multi-turn engagement — shallow = unhelpful or confusing response |
| **Session abandonment rate** | `victor_session_abandoned_rate` | Sessions with 1 turn / total sessions | 90 days | User did not find first response useful |
| **Post-coaching workout start rate** | `victor_post_coaching_workout_pct` | `workout_logs.started_at` within 4h of `coaching_sessions.ended_at` | 24 months | Victor drove action, not just conversation |
| **Workout completion rate (post-coaching)** | `victor_post_coaching_completion_pct` | Workouts started post-coaching that reached `status=completed` | 24 months | Prescription quality (overloaded? underloaded? unclear?) |
| **Retry rate within session** | `victor_session_retry_rate` | Turns with `user_feedback = retry` or consecutive short-gap turns | 30 days | User found response inadequate, asked again |
| **Plan regeneration rate** | `victor_plan_regen_rate` | Regeneration requests per user per week | 30 days | User dissatisfied with plan prescription |
| **Explicit negative feedback rate** | `victor_thumbs_down_rate` | `coaching_turns.feedback = negative` / total turns with feedback | 24 months | Direct dissatisfaction signal |
| **No-engagement rate** | `victor_no_engage_7d_rate` | Users with 0 coaching sessions in trailing 7d (active subscribers) | 90 days | Victor not surfaced or not useful enough to seek |
| **Streak grace trigger rate** | `victor_streak_grace_rate` | Streak grace activations per week (DEC-013 signal) | 24 months | Real-world adherence floor (grace = miss, but within policy) |

**Derivation note:** All metrics are computed via `pg_cron` views refreshed at 04:00 UTC daily. Raw `coaching_turns` table is never exported — aggregation happens inside Supabase Postgres. `coaching_turns.content_hash` exists for deduplication but is never used in observability queries.

---

### 22.3 Prompt Version Tracking

Victor's behavior is controlled by a system prompt stored in Cloudflare Workers Secrets (`VICTOR_SYSTEM_PROMPT_HASH`) and versioned by `victor_prompt_version` (semver, e.g. `2.1.4`). This version tag is:

- Written to `coaching_turns.victor_prompt_version` at inference time.
- Included in the AI inference log (§4.5) as `prompt_version` field.
- Used to segment all quality metrics in §22.2 by version, enabling before/after comparison for prompt changes.

**Prompt version schema extension** (additive to `coaching_turns`):

```sql
-- Already exists in DATA_MODEL.md §11; confirm these columns are present:
ALTER TABLE coaching_turns
  ADD COLUMN IF NOT EXISTS victor_prompt_version TEXT NOT NULL DEFAULT '0.0.0',
  ADD COLUMN IF NOT EXISTS prompt_ab_bucket     TEXT;  -- NULL unless A/B test active

CREATE INDEX IF NOT EXISTS idx_coaching_turns_prompt_version
  ON coaching_turns (victor_prompt_version, created_at DESC);
```

**Baseline definition:** When a new prompt version ships, the prior 14 days of data under the previous version become the **quality baseline** for that dimension. Regression alerting (§22.6) compares the 7-day rolling window of the new version against this baseline.

---

### 22.4 Quality SLOs

| SLO ID | Metric | Target | Alert Threshold | Window | Owner |
|---|---|---|---|---|---|
| **QA-COACH-SLO-01** | Plan adherence rate | ≥ 65% | < 55% sustained 3 days | Rolling 7 days | ml-engineer |
| **QA-COACH-SLO-02** | Session abandonment rate (1-turn sessions) | ≤ 30% | > 45% sustained 3 days | Rolling 7 days | ml-engineer |
| **QA-COACH-SLO-03** | Post-coaching workout start rate | ≥ 40% | < 25% sustained 3 days | Rolling 7 days | platform-engineer |
| **QA-COACH-SLO-04** | Explicit negative feedback rate | ≤ 8% | > 15% sustained 2 days | Rolling 7 days | ml-engineer |
| **QA-COACH-SLO-05** | Plan regeneration rate | ≤ 15% per active user/week | > 25% sustained 3 days | Rolling 7 days | ml-engineer |
| **QA-COACH-SLO-06** | No-engagement rate (0 coaching sessions / 7d) | ≤ 40% of active subscribers | > 60% sustained 7 days | Rolling 14 days | growth-lead |

**SLO philosophy:** These targets are calibrated for a pre-launch beta with < 200 users. They should be re-baselined at M6 (post-public launch) using first 30-day cohort data. Targets tighten as sample size increases and measurement confidence improves.

**Caution on interpretation:** Quality SLO breaches are *hypotheses*, not conclusions. A drop in plan adherence rate may indicate bad coaching, a UI bug, a seasonal effect, or a cohort-mix shift. SLO breach triggers investigation, not automatic rollback.

---

### 22.5 Clinical Safety Monitoring Signals

Clinical safety observability differs from quality observability: it watches for signals that Victor may have produced potentially harmful coaching. Per INCIDENT_RESPONSE.md R-10, content itself is not accessible to monitoring systems. Instead, the following **structural signals** serve as canaries:

| Signal | Source | Trigger condition | Severity | Response |
|---|---|---|---|---|
| **Coaching session duration spike** | `coaching_turns` | P99 session duration > 3× 30d baseline for a given `victor_prompt_version` | P1 | ML-engineer + clinical-safety review of that prompt version |
| **Retry rate spike after prompt deploy** | `coaching_turns` | `victor_session_retry_rate` > 2× baseline within 24h of prompt deploy | P1 | Prompt version rollback candidate; clinical-safety review |
| **No-action post-session spike** | Behavioral | `victor_post_coaching_workout_pct` drops > 30% relative in 48h | P1 | May indicate confusing or alarming coaching; escalate to clinical-safety |
| **Negative feedback rate spike** | `coaching_turns.feedback` | `victor_thumbs_down_rate` > 25% in any 4-hour window | P0 | Immediate escalation per R-10 §2.1; FEATURE FLAG consideration |
| **Explicit harm report** | Support ticket + `incident_reports` | Any ticket tagged `victor_harmful_content` | P0 | R-10 full activation; clinical-safety VETO authority engaged |
| **Jailbreak test suite failure** | CI / scheduled | `victor_jailbreak_tests` fails any assertion | P0 | Block prompt deploy; notify security-engineer + clinical-safety |

**No-go escalation path:** If any P0 clinical safety signal fires, the response escalation leads to clinical-safety as defined in R-10. Clinical-safety has VETO power to disable the Victor feature flag without founder approval for P0 safety events (DEC-012 exception).

---

### 22.6 Alerting Rules

| Alert ID | Condition | Severity | Channel | Dedup key |
|---|---|---|---|---|
| **AL-COACH-01** | `victor_plan_adherence_rate` < 55% for 3 consecutive daily aggregations | P1 | `#alerts-quality` (Slack) | `qa_coach_adherence_{prompt_version}` |
| **AL-COACH-02** | `victor_session_abandoned_rate` > 45% for 3 consecutive daily aggregations | P1 | `#alerts-quality` | `qa_coach_abandon_{prompt_version}` |
| **AL-COACH-03** | `victor_thumbs_down_rate` > 25% in any 4-hour window (rolling) | P0 | PagerDuty (ml-engineer on-call) + `#inc-YYYYMMDD-coach` | `qa_coach_thumbsdown_p0` |
| **AL-COACH-04** | New `victor_prompt_version` deployed AND any QA-COACH-SLO breached within 48h | P1 | `#alerts-quality` + ml-engineer DM | `qa_coach_regression_{new_version}` |
| **AL-COACH-05** | `victor_jailbreak_tests` CI job fails | P0 | PagerDuty + Slack `#security` | `qa_coach_jailbreak_{commit_sha}` |
| **AL-COACH-06** | `victor_no_engage_7d_rate` > 60% for 7 consecutive daily aggregations | P2 | `#metrics` Slack digest | `qa_coach_noengage_{week}` |

**Alert suppression:** AL-COACH-01, AL-COACH-02, AL-COACH-04 are suppressed during the first 72 hours after a prompt version deploy. Small sample sizes produce false positives before the new version accumulates sufficient session volume (target: ≥ 200 coaching sessions before quality SLOs are evaluated against the new version).

---

### 22.7 Prompt A/B Testing Observability Framework

When testing two prompt variants:
1. Each session is assigned `coaching_turns.prompt_ab_bucket ∈ {control, variant_a}` at session start (via Cloudflare Workers random split, tenant-locked for consistency within a session).
2. All §22.2 quality metrics are segmented by `prompt_ab_bucket`.
3. A/B tests run for minimum 14 days or 500 sessions per bucket (whichever is later), before a statistical conclusion is drawn.
4. Statistical method: two-proportion z-test for rate metrics; Mann-Whitney U for ordinal depth metrics. α = 0.05, power = 0.80.
5. Results stored in `victor_ab_results` table (schema below); archived to `compliance/evidence/victor-ab/{test_id}/` at test conclusion.

```sql
CREATE TABLE victor_ab_results (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  test_id         TEXT NOT NULL,            -- e.g. 'ab-2026-05-24-bracing-cue'
  started_at      TIMESTAMPTZ NOT NULL,
  concluded_at    TIMESTAMPTZ,
  control_version TEXT NOT NULL,
  variant_version TEXT NOT NULL,
  winner          TEXT CHECK (winner IN ('control', 'variant', 'no_difference', 'inconclusive')),
  primary_metric  TEXT NOT NULL,            -- e.g. 'victor_post_coaching_completion_pct'
  control_value   NUMERIC(6,4),
  variant_value   NUMERIC(6,4),
  p_value         NUMERIC(6,4),
  session_count   INTEGER,
  notes           TEXT,
  concluded_by    TEXT,                     -- founder or ml-engineer
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

**DEC-030 events for A/B tests:**

| Event | Severity | When emitted |
|---|---|---|
| `victor.ab_test_started` | MEDIUM | Test bucket assignment begins |
| `victor.ab_test_concluded` | MEDIUM | Winner declared or test terminated |
| `victor.prompt_version_promoted` | HIGH | Variant promoted to production (winner = variant) |
| `victor.prompt_version_rolled_back` | HIGH | Variant retired due to regression |

---

### 22.8 Metabase Dashboard: Victor Coaching Quality

**Dashboard name:** `Victor Coaching Quality`
**Refresh:** Daily at 05:00 UTC (after pg_cron aggregation at 04:00)
**Access:** Founder + ml-engineer + clinical-safety. NOT tenant_admin (enterprise coaching quality is surfaced through separate tenant admin views with appropriate anonymisation).

| Panel | Query | Chart type |
|---|---|---|
| Plan adherence rate (7d rolling) | `victor_quality_daily` WHERE metric = 'adherence', trailing 30 rows | Line, with QA-COACH-SLO-01 threshold line |
| Session abandonment rate (7d rolling) | Same table, metric = 'abandonment' | Line, with QA-COACH-SLO-02 threshold |
| Post-coaching workout start rate | metric = 'post_coaching_start' | Line |
| Negative feedback rate | metric = 'thumbs_down' | Line, with P0 threshold at 25% in red |
| Quality by prompt version | GROUP BY victor_prompt_version | Grouped bar: adherence, completion, abandonment per version |
| Active A/B test status | `victor_ab_results` WHERE concluded_at IS NULL | Table: test_id, sessions/bucket, primary metric current value |
| No-engagement trend | metric = 'no_engage_7d' | Area chart (inverse — high is bad) |
| Streak grace rate | metric = 'streak_grace' | Line |

---

### 22.9 Supporting Database Schema

```sql
-- Daily aggregation table — computed by pg_cron, never raw coaching content
CREATE TABLE victor_quality_daily (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  date         DATE NOT NULL,
  metric       TEXT NOT NULL,               -- one of: adherence, abandonment, post_coaching_start, completion, thumbs_down, plan_regen, no_engage_7d, streak_grace, session_depth_p50
  value        NUMERIC(8,4) NOT NULL,
  prompt_version TEXT NOT NULL,
  sample_size  INTEGER NOT NULL,            -- session or user count for this metric × date × version
  ab_bucket    TEXT,                        -- NULL unless A/B test active
  tenant_id    UUID,                        -- NULL = all tenants (fleet aggregate)
  created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (date, metric, prompt_version, COALESCE(ab_bucket, ''), COALESCE(tenant_id, '00000000-0000-0000-0000-000000000000'))
);

-- RLS: founder + ml-engineer read; form_system write; tenant_admin sees only own tenant_id rows
ALTER TABLE victor_quality_daily ENABLE ROW LEVEL SECURITY;
CREATE POLICY vqd_form_admin ON victor_quality_daily FOR ALL TO form_admin USING (true);
CREATE POLICY vqd_tenant_read ON victor_quality_daily FOR SELECT TO tenant_admin
  USING (tenant_id = current_setting('app.tenant_id')::UUID);

-- pg_cron job: daily aggregation at 04:00 UTC
-- SELECT cron.schedule('victor-quality-daily', '0 4 * * *', $$
--   INSERT INTO victor_quality_daily (date, metric, value, prompt_version, sample_size)
--   SELECT
--     CURRENT_DATE - 1,
--     'adherence',
--     ROUND(COUNT(*) FILTER (WHERE followed_plan) / NULLIF(COUNT(*), 0)::NUMERIC, 4),
--     victor_prompt_version,
--     COUNT(*)
--   FROM coaching_plan_adherence_view   -- view joins training_plan_weeks × workout_logs
--   WHERE plan_date = CURRENT_DATE - 1
--   GROUP BY victor_prompt_version
--   ON CONFLICT DO NOTHING;
-- $$);

-- Retention: 24 months (GDPR Art. 5(1)(e) storage limitation — aggregate only, no personal data)
-- pg_cron hard-delete at 03:30 UTC: DELETE FROM victor_quality_daily WHERE date < now() - INTERVAL '24 months';
```

**Privacy note:** `victor_quality_daily` contains no user identifiers. `sample_size` must be ≥ 5 before the row is written — rows with `sample_size < 5` are suppressed (k-anonymity floor). This prevents micro-cohort reconstruction at the tenant + prompt_version level for small enterprise tenants.

---

### 22.10 SOC 2 Evidence Mapping

| Control | Criterion | Evidence artefact | Owner |
|---|---|---|---|
| Victor quality monitoring exists | CC7.2 — System operation monitoring | §22 policy document (this section) | ml-engineer |
| Prompt version regression detection | CC7.4 — Evaluation of processing activities | AL-COACH-04 alert rule + `victor_ab_results` table | ml-engineer |
| Ethical coaching boundary enforcement | CC1.2 — Commitment to integrity and ethical values | AL-COACH-05 (jailbreak CI gate) + R-10 escalation path | clinical-safety |
| Clinical safety escalation path documented | CC5.2 — Control activities for risk mitigation | §22.5 clinical safety signals + R-10 cross-reference | clinical-safety |
| No coaching content in observability data | C1.1 — Confidential information protected | §22.1 privacy constraint statement + §22.9 RLS policy | platform-engineer |

**Evidence artefacts:**
- `compliance/evidence/victor-quality/YYYY-MM-quality-report.md` — monthly quality metric summary (exported from Metabase §22.8), retained 24 months.
- `compliance/evidence/victor-ab/` — one directory per A/B test with `victor_ab_results` export and statistical analysis. Retained 36 months.
- `compliance/evidence/victor-jailbreak/` — monthly CI jailbreak test run logs. Retained 24 months.

---

### 22.11 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Add `victor_prompt_version` and `prompt_ab_bucket` columns to `coaching_turns` if not present; add `idx_coaching_turns_prompt_version` index | platform-engineer | **P0** | M3 |
| Create `victor_quality_daily` table with RLS policies per §22.9 | platform-engineer | **P0** | M3 |
| Create `coaching_plan_adherence_view` (joins `training_plan_weeks` × `workout_logs`) as computation base for adherence metric | platform-engineer | **P0** | M3 |
| Write pg_cron jobs for all 8 metrics in §22.2; validate k-anonymity suppression (sample_size < 5 → skip row) | platform-engineer | **P0** | M3 |
| Configure Better Stack alert rules AL-COACH-01 through AL-COACH-06 with correct dedup keys and suppression window | devops-lead | **P0** | M3 |
| Implement `victor_jailbreak_tests` CI job (prompt injection, ED-adjacent content detection, harm refusal validation); wire to AL-COACH-05 | ml-engineer + security-engineer | **P0** | M3 |
| Create `victor_ab_results` table; wire A/B bucket assignment to Cloudflare Workers random split (tenant-locked within session) | platform-engineer | **P1** | M4 |
| Build Metabase "Victor Coaching Quality" dashboard with 8 panels per §22.8; restrict access to founder + ml-engineer + clinical-safety | data-engineer | **P1** | M4 |
| Define `victor_quality_daily` k-anonymity suppression test in `__tests__/db/rls_isolation.test.ts` | platform-engineer | **P0** | M3 |
| Implement monthly quality report export to `compliance/evidence/victor-quality/` via Metabase API (pg_cron or scheduled Worker) | devops-lead | **P1** | M4 |
| Register DEC-030 events: `victor.ab_test_started`, `victor.ab_test_concluded`, `victor.prompt_version_promoted`, `victor.prompt_version_rolled_back` | platform-engineer | **P0** | M3 |
| Re-baseline QA-COACH-SLO-01 through SLO-06 after 30 days of beta data (M6) | ml-engineer | **P1** | M6 |

---

## 23. Enterprise SLA Reporting & Credit Calculation

### 23.1 Purpose

This section defines how FORM measures the 99.9% monthly uptime SLA committed in `docs/ENTERPRISE.md`, how that data is surfaced to tenant administrators, and the precise credit calculation and claim process when the SLA is missed. It closes the gap at `docs/SOC2_READINESS.md §2 A1.1` ("SLA credit calculation and process — Define formula and claim process") and provides SOC 2 Type II evidence for criterion A1.1 (Availability commitments documented and honoured).

**Owner:** devops-lead (measurement), enterprise-architect (API and admin dashboard), compliance-officer (credit issuance authority).

---

### 23.2 SLA Definition and Covered Services

**SLA target:** 99.9% monthly availability per tenant. This equals a maximum of **43.8 minutes of covered downtime per 30-day calendar month**.

**Covered services** (all must be simultaneously unavailable for downtime to count):

| Service | SLI used | SLA probe |
|---|---|---|
| API Gateway (Cloudflare Workers) | HTTP availability at `GET /health` | S-001 (§16.2) |
| Authentication (Supabase Auth / WorkOS SSO) | HTTP 200 from token exchange | S-002 + S-005 |
| Workout Logging API | HTTP 201 from `POST /api/v1/workouts/sets` | S-003 |
| AI Coaching (Victor) | HTTP 200 from coaching turn probe | S-004 |
| Admin Dashboard API | HTTP 200 from `GET /api/v1/admin/health` | new probe S-013 |

**Partial-service degradation rule:** If one service is down but others remain reachable, the incident is classified as a **partial outage**. Partial outages count at 50% weight against the downtime budget — 10 minutes of coaching unavailability = 5 credited minutes toward the 43.8-minute threshold.

**Single-tenant vs. platform-wide:** FORM measures uptime per tenant. A bug that affects only one tenant's SSO configuration is an incident for that tenant only; other tenants' SLA clocks are unaffected.

---

### 23.3 Uptime Measurement Methodology

Availability is calculated from two independent signal sources, reconciled at month-end:

**Source 1 — Better Stack synthetic probes (primary):**
- Probes S-001 through S-013 run from three regions (us-east, eu-central, ap-southeast) on the schedules defined in §16.2.
- A minute is classified as **down** when two or more probe regions report failure simultaneously. Single-region failure is classified as a **degraded** minute (50% weight).
- Better Stack generates a monthly `uptime_report` object per monitor per tenant label.

**Source 2 — Cloudflare Analytics Engine (secondary, corroborating):**

```sql
-- Cloudflare Analytics Engine SQL
-- Hourly availability SLI per tenant (run at month-end reconciliation)
SELECT
  toStartOfHour(timestamp) AS hour_bucket,
  blob1 AS tenant_id,
  countIf(double1 < 500 AND double2 = 200) AS good_requests,
  count() AS total_requests,
  good_requests / total_requests AS availability_sli
FROM cf_worker_requests
WHERE
  timestamp BETWEEN toDateTime('2026-05-01 00:00:00') AND toDateTime('2026-05-31 23:59:59')
  AND blob1 = :tenant_id          -- tenant_id label injected at Worker edge
  AND blob2 = 'api'               -- only count API surface, not status-page
GROUP BY hour_bucket, tenant_id
ORDER BY hour_bucket;
```

**Reconciliation rule:** If the two sources disagree by > 0.05% availability on a given day, the lower value (more conservative for the customer) is used. The reconciliation is logged as a DEC-030 audit event `sla.measurement_reconciled` with both source values.

**Maintenance window exclusions:** Planned maintenance is excluded from the downtime calculation if:
1. The tenant received email notification ≥ 72 hours in advance (7 days for Enterprise tier).
2. The maintenance window does not exceed 4 hours per occurrence or 8 hours per calendar month.
3. The window is registered in `maintenance_windows` table (§23.8) before it begins.

---

### 23.4 SLA Exclusions

The following do not count toward covered downtime:

| Category | Rationale |
|---|---|
| **Upstream provider outage** | Cloudflare, Supabase, Anthropic, ElevenLabs, Auth0/WorkOS outage beyond FORM's control. FORM provides best-effort workaround where possible. Documented via provider status page link in the DEC-030 `sla.incident_closed` event. |
| **Scheduled maintenance** (per §23.3) | Advance-notified windows registered in `maintenance_windows`. |
| **Customer-side network or IdP misconfiguration** | Failures traceable to tenant's own Okta/Azure AD misconfiguration, firewall rules blocking FORM IPs, or SCIM credential expiry not renewed by the tenant. Evidenced by SSO error code `idp_configuration_error` in the audit log. |
| **Force majeure** | Acts of God, internet backbone failures, national infrastructure incidents beyond FORM's ability to route around. |
| **Synthetic probe false positives** | If S-001 through S-013 report failure but zero real-user errors appear in Cloudflare analytics within the same 5-minute window, the minute is reclassified as a probe false positive and excluded. Reclassification requires security-engineer approval and is DEC-030 logged. |
| **Beta / preview features** | Features explicitly labelled `[BETA]` in the admin dashboard carry no SLA coverage. |

---

### 23.5 Credit Calculation Formula

Credits are calculated on the tenant's **Monthly Recurring Revenue (MRR)** for the affected calendar month.

**Credit tiers:**

| Monthly Availability (calendar month) | Downtime budget used | Service Credit |
|---|---|---|
| ≥ 99.9% | 0 – 43.8 min | 0% — SLA met |
| 99.0% – < 99.9% | 43.8 min – 7.2 h | **5% of monthly invoice** |
| 95.0% – < 99.0% | 7.2 h – 36 h | **15% of monthly invoice** |
| 90.0% – < 95.0% | 36 h – 72 h | **25% of monthly invoice** |
| < 90.0% | > 72 h | **50% of monthly invoice** |

**Partial-service multiplier:** Credits earned during partial-outage windows (§23.2 50% weight rule) are issued at 50% of the tier rate. A 2-hour coaching-only outage that crosses into the 99.0–99.9% tier earns 2.5% credit (5% × 50%).

**Credit cap:** Total credits in any calendar month cannot exceed 50% of the tenant's monthly invoice. Credits are applied to the following month's invoice; they are not redeemable as cash.

**Calculation example (Growth tier, 300 seats at $9/seat = $2,700 MRR):**

```
Covered downtime in May: 95 minutes (full platform outage, all probes)
  → Exceeds 43.8-minute SLA threshold
  → Falls in 99.0–99.9% tier (actual: ~99.78% availability in a 30-day month = 43,200 minutes)
  → Credit: 5% × $2,700 = $135.00 applied to June invoice
```

---

### 23.6 Customer-Facing Uptime Report

**Admin dashboard panel — "SLA Uptime Report"** (existing UI stub at `admin-dashboard.html:575`):

The panel renders the current month's availability data in real-time, refreshed every 60 seconds from the `/api/v1/admin/sla/current` endpoint.

```typescript
// GET /api/v1/admin/sla/current
// Auth: tenant_admin JWT required — RLS enforces tenant_id isolation
// Returns: current-month SLA status for the calling tenant

interface SlaCurrentResponse {
  tenant_id: string;
  report_month: string;              // "2026-05" ISO format
  availability_pct: number;          // e.g. 99.97
  covered_downtime_minutes: number;  // actual downtime against the SLA definition
  sla_threshold_minutes: number;     // 43.8 for 99.9% in a 30-day month
  sla_met: boolean;
  credit_tier: 0 | 5 | 15 | 25 | 50;  // credit % if applicable; 0 = SLA met
  projected_credit_usd: number | null;  // null if credit_tier === 0
  incidents: SlaIncident[];
  last_refreshed_at: string;         // ISO timestamp
}

interface SlaIncident {
  incident_id: string;               // matches DEC-030 sla.incident_opened event
  started_at: string;
  ended_at: string | null;           // null if still open
  duration_minutes: number;
  outage_type: 'full' | 'partial';
  affected_services: string[];
  excluded: boolean;                 // true if falls under §23.4 exclusion
  exclusion_reason: string | null;
}
```

**Monthly PDF report:** On the 3rd business day of each month, a pg_cron job generates a PDF summary for the previous month. It is stored at `r2://form-compliance/{tenant_id}/sla/{YYYY-MM}-uptime-report.pdf` and delivered to `tenant_billing_email` via Resend. The PDF contains:
- Availability percentage and downtime minutes (covered and excluded)
- Incident log with start/end times, affected services, and RCA links
- Credit calculation workings (if applicable)
- SHA-256 hash of the Better Stack monthly report (as the corroborating source reference)

---

### 23.7 SLA Credit Claim Process

**Step 1 — Eligibility check (automated):**
At month-end (00:30 UTC on the 1st of each month), a scheduled Cloudflare Worker (`workers/sla/month-close.ts`) runs the measurement reconciliation (§23.3) and writes the final `sla_monthly_reports` row (§23.8). If `credit_tier > 0`, the Worker:
1. Emits `sla.credit_calculated` DEC-030 event.
2. Creates a Linear ticket assigned to `compliance-officer` with the credit amount and invoice month.
3. Sends the tenant an automated email: "Your SLA report for [Month] is ready — you are eligible for a $X credit."

**Step 2 — Tenant claim (optional):**
Credits ≥ 5% are applied automatically without requiring a formal claim — the automated path in Step 1 covers this. Tenants may contest an SLA report (e.g., dispute an exclusion ruling) via:
- Admin dashboard → SLA Reports → "Dispute this report" button (routes to `enterprise@form.coach` with pre-filled incident IDs).
- Dispute deadline: within 30 days of the month-end report delivery.

**Step 3 — Compliance-officer review:**
On receiving the Linear ticket, `compliance-officer`:
1. Verifies the automated measurement against the Better Stack uptime export and Cloudflare Analytics reconciliation query.
2. Checks that all exclusions are documented and defensible.
3. Approves or adjusts the credit amount within 5 business days.
4. Emits `sla.credit_approved` DEC-030 event.
5. Instructs billing to apply the credit to the next invoice.

**Step 4 — Credit application:**
The credit is applied as a line-item reduction on the following month's invoice. The invoice line reads:

```
SLA Service Credit — [Month YYYY] (99.x% availability, 5% credit)    −$XXX.00
```

**Step 5 — Dispute escalation:**
If the tenant disputes the exclusion ruling and internal review does not resolve it within 10 business days, compliance-officer escalates to the founder for final determination. All communications are logged as DEC-030 `sla.dispute_*` events.

---

### 23.8 Postgres Schema

```sql
-- Stores the final monthly SLA record per tenant (written by month-close Worker)
CREATE TABLE sla_monthly_reports (
  id                        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id                 UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  report_month              DATE NOT NULL,      -- always the 1st of the month (e.g. 2026-05-01)
  calendar_minutes          INTEGER NOT NULL,   -- total minutes in the month
  covered_downtime_minutes  NUMERIC(8,2) NOT NULL DEFAULT 0,
  excluded_downtime_minutes NUMERIC(8,2) NOT NULL DEFAULT 0,
  availability_pct          NUMERIC(7,5) GENERATED ALWAYS AS (
    100.0 * (calendar_minutes - covered_downtime_minutes) / NULLIF(calendar_minutes, 0)
  ) STORED,
  sla_met                   BOOLEAN GENERATED ALWAYS AS (
    covered_downtime_minutes <= (calendar_minutes * 0.001)
  ) STORED,
  credit_tier_pct           SMALLINT NOT NULL DEFAULT 0,  -- 0 / 5 / 15 / 25 / 50
  credit_amount_usd         NUMERIC(10,2),
  mrr_snapshot_usd          NUMERIC(10,2) NOT NULL,
  betterstack_report_sha256 TEXT,        -- hash of raw Better Stack export for auditability
  cf_analytics_reconciled   BOOLEAN NOT NULL DEFAULT FALSE,
  reconciliation_delta_pct  NUMERIC(6,4),  -- |betterstack_availability - cf_availability|
  credit_approved_by        UUID REFERENCES users(id),  -- compliance-officer user_id
  credit_approved_at        TIMESTAMPTZ,
  report_pdf_r2_key         TEXT,         -- path in R2 for tenant PDF copy
  created_at                TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (tenant_id, report_month)
);

-- Incident log contributing to monthly SLA calculation
CREATE TABLE sla_incidents (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id        UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  started_at       TIMESTAMPTZ NOT NULL,
  ended_at         TIMESTAMPTZ,
  outage_type      TEXT NOT NULL CHECK (outage_type IN ('full', 'partial')),
  affected_probes  TEXT[] NOT NULL,    -- e.g. ['S-001', 'S-002']
  downtime_weight  NUMERIC(3,2) NOT NULL DEFAULT 1.0,  -- 0.5 for partial
  excluded         BOOLEAN NOT NULL DEFAULT FALSE,
  exclusion_reason TEXT,              -- must be non-null if excluded = TRUE
  incident_ref     TEXT,              -- Linear or PagerDuty incident ID
  created_at       TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Scheduled maintenance windows (must be registered before window starts)
CREATE TABLE maintenance_windows (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id           UUID REFERENCES tenants(id) ON DELETE CASCADE,  -- NULL = platform-wide
  scheduled_start     TIMESTAMPTZ NOT NULL,
  scheduled_end       TIMESTAMPTZ NOT NULL,
  notified_at         TIMESTAMPTZ NOT NULL,  -- when tenant notification was sent
  notification_hours  INTEGER GENERATED ALWAYS AS (
    EXTRACT(EPOCH FROM (scheduled_start - notified_at)) / 3600
  ) STORED,
  max_duration_hours  NUMERIC(4,1) NOT NULL,
  approved_by         UUID NOT NULL REFERENCES users(id),
  description         TEXT,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  CHECK (scheduled_end > scheduled_start),
  CHECK (scheduled_end - scheduled_start <= INTERVAL '4 hours')  -- per §23.4 cap
);

-- RLS: tenant_admin sees only their own tenant's reports and incidents
ALTER TABLE sla_monthly_reports ENABLE ROW LEVEL SECURITY;
CREATE POLICY sla_reports_tenant_isolation ON sla_monthly_reports
  USING (
    tenant_id = (current_setting('app.current_tenant_id', TRUE))::UUID
    OR EXISTS (SELECT 1 FROM users WHERE id = auth.uid() AND role = 'founder')
  );

ALTER TABLE sla_incidents ENABLE ROW LEVEL SECURITY;
CREATE POLICY sla_incidents_tenant_isolation ON sla_incidents
  USING (
    tenant_id = (current_setting('app.current_tenant_id', TRUE))::UUID
    OR EXISTS (SELECT 1 FROM users WHERE id = auth.uid() AND role = 'founder')
  );

-- Index for month-end aggregation Worker
CREATE INDEX idx_sla_incidents_tenant_month
  ON sla_incidents (tenant_id, started_at)
  WHERE ended_at IS NOT NULL;

-- Monthly aggregation query (used inside month-close Worker)
-- Returns covered_downtime_minutes for a given tenant and calendar month
SELECT
  COALESCE(
    SUM(
      LEAST(ended_at, month_end) - GREATEST(started_at, month_start)
    ) * downtime_weight / INTERVAL '1 minute', 0
  ) AS covered_downtime_minutes
FROM sla_incidents
CROSS JOIN (VALUES (
  '2026-05-01 00:00:00+00'::TIMESTAMPTZ,
  '2026-06-01 00:00:00+00'::TIMESTAMPTZ
)) AS t(month_start, month_end)
WHERE
  tenant_id = :tenant_id
  AND excluded = FALSE
  AND ended_at IS NOT NULL
  AND tstzrange(started_at, ended_at) && tstzrange(month_start, month_end);
```

---

### 23.9 DEC-030 Audit Trail

All SLA lifecycle events are HMAC-chained per `docs/AUDIT_LOG_SCHEMA.md` (DEC-030). Required event types:

| Event type | Trigger | Logged by | Key payload fields |
|---|---|---|---|
| `sla.incident_opened` | Downtime detected by month-close Worker or manual P0 declaration | devops-lead / Worker | `tenant_id`, `probe_ids`, `outage_type`, `started_at` |
| `sla.incident_closed` | Probes recover; `ended_at` written | devops-lead / Worker | `tenant_id`, `incident_id`, `ended_at`, `duration_minutes`, `exclusion_applied` |
| `sla.measurement_reconciled` | Month-end reconciliation completes | month-close Worker | `tenant_id`, `report_month`, `betterstack_pct`, `cf_analytics_pct`, `delta_pct`, `chosen_source` |
| `sla.credit_calculated` | Automated credit tier determined | month-close Worker | `tenant_id`, `report_month`, `credit_tier_pct`, `credit_amount_usd` |
| `sla.credit_approved` | compliance-officer approves | compliance-officer | `tenant_id`, `report_month`, `approved_by`, `final_credit_usd` |
| `sla.credit_adjusted` | Credit amount changed from calculated (e.g., dispute resolution) | compliance-officer | `tenant_id`, `report_month`, `original_usd`, `adjusted_usd`, `reason` |
| `sla.dispute_opened` | Tenant submits dispute via admin dashboard | platform (form_system role) | `tenant_id`, `report_month`, `dispute_reason` |
| `sla.dispute_resolved` | Dispute closed (upheld or rejected) | compliance-officer | `tenant_id`, `report_month`, `outcome`, `final_credit_usd` |
| `sla.maintenance_window_registered` | maintenance_windows row inserted | devops-lead | `tenant_id` or `null` (platform), `scheduled_start`, `scheduled_end`, `notification_hours` |
| `sla.exclusion_reclassified` | False-positive probe reclassification per §23.4 | security-engineer | `tenant_id`, `incident_id`, `original_classification`, `approved_by` |

All ten event types must carry `tenant_id` (or `null` for platform-wide events), `trace_id`, and `actor_id` in the standard DEC-030 envelope. HMAC chain must remain unbroken — a chain break on any `sla.*` event is a P0 incident per `docs/INCIDENT_RESPONSE.md §R-05`.

---

### 23.10 SOC 2 Evidence Mapping

| Control | Criterion | Evidence artefact | Owner |
|---|---|---|---|
| SLA availability commitment documented | A1.1 — Availability commitments | `docs/ENTERPRISE.md §What we promise customers` + this section | compliance-officer |
| SLA credit formula defined and published | A1.1 — Honoured commitments | §23.5 credit table (this document) | compliance-officer |
| SLA measured against committed threshold | A1.1 — Monitoring against commitments | `sla_monthly_reports` table + Better Stack monthly export | devops-lead |
| SLA credit issuance process documented | A1.1 — Remediation when commitments not met | §23.7 claim process (this document) + DEC-030 `sla.credit_*` events | compliance-officer |
| System monitoring generates availability data | CC7.2 — Anomaly detection | Better Stack probes S-001–S-013, Cloudflare Analytics Engine query (§23.3) | devops-lead |
| Maintenance windows disclosed in advance | A1.1 — Exclusions are pre-communicated | `maintenance_windows` table + `notification_hours ≥ 72` constraint | devops-lead |

**Evidence artefacts for auditors:**
- `compliance/evidence/sla/{YYYY-MM}-{tenant_id}-uptime-report.json` — machine-readable monthly report, generated by month-close Worker, retained 36 months.
- `compliance/evidence/sla/{YYYY-MM}-betterstack-export.csv` — raw Better Stack uptime export, SHA-256 hashed and recorded in `sla_monthly_reports.betterstack_report_sha256`, retained 36 months.
- `compliance/evidence/sla/{YYYY-MM}-credit-ledger.csv` — all credits issued that month, signed by compliance-officer, retained 7 years (financial record).
- DEC-030 audit log filtered on `event_type LIKE 'sla.%'` — continuous chain-verified log, retained per `docs/AUDIT_LOG_SCHEMA.md` schedule.

---

### 23.11 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Create `sla_monthly_reports`, `sla_incidents`, `maintenance_windows` tables with RLS per §23.8 | platform-engineer | **P0** | M4 |
| Add synthetic probe S-013 (Admin Dashboard API health) to `better-stack-monitors.yaml` | devops-lead | **P0** | M4 |
| Implement `workers/sla/month-close.ts` — measurement reconciliation, `sla_monthly_reports` write, credit calculation, Linear ticket creation, DEC-030 emit | platform-engineer | **P0** | M4 |
| Implement `GET /api/v1/admin/sla/current` endpoint with tenant_id RLS guard | platform-engineer | **P0** | M4 |
| Wire admin dashboard SLA panel to live endpoint (replace static stub in `admin-dashboard.html:575`) | design-craft + platform-engineer | **P1** | M4 |
| Implement `GET /api/v1/admin/sla/history` endpoint returning last 12 monthly reports | platform-engineer | **P1** | M4 |
| Implement monthly PDF report generation (pg_cron + Resend delivery) | devops-lead | **P1** | M5 |
| Register all 10 `sla.*` DEC-030 event types in `AUDIT_LOG_SCHEMA.md` event registry | security-engineer | **P0** | M4 |
| Add `sla.credit_*` event assertions to `__tests__/db/rls_isolation.test.ts` | platform-engineer | **P1** | M4 |
| Implement "Dispute this report" admin dashboard flow (form → `enterprise@form.coach` email with incident IDs pre-filled) | platform-engineer | **P2** | M5 |
| Update `SOC2_READINESS.md §2 A1.1` credit-calculation row from 🟡 Gap to ✅ Done | compliance-officer | **P0** | M4 |
| Add `sla_monthly_reports` Metabase dashboard: panel per tenant showing rolling-12-month availability % and credit history | data-engineer | **P2** | M5 |

---

*v0.8 additions: §21 ClickHouse Analytics Observability (M9) — closes the documented gap from §10 ("ClickHouse for analytics — observability strategy not yet designed"); migration trigger thresholds (PostHog cost > $2k/month P0, analytics query P95 > 10s P0, MAU ≥ 200k advisory, 30+ enterprise tenants advisory); ClickHouse Cloud architecture (EU-Central, SOC 2 certified, GDPR DPA); RED metrics table for ClickHouse itself (rate/errors/duration/ingest lag/queue/storage/memory); eight alerting rules AL-CH-01 through AL-CH-09 with severity and PagerDuty routing; OpenTelemetry Collector topology (Fly.io EU) with PII filter processor deleting user_id and ai prompt/response content before ClickHouse; `traces` MergeTree table DDL with promoted materialized columns for tenant_id, route, http_status, ai tokens; product `events` table DDL with BLOCKED_PROPERTY_KEYS deny-list enforcing no health values at ingest; `sessions` ReplacingMergeTree table; six analytics SLOs (CH-SLO-01 through CH-SLO-06) with error budget burn rate thresholds; per-tenant Row Policies in ClickHouse (least-privilege; cross-tenant queries require break-glass and are DEC-030 logged); three-phase PostHog migration runbook (parallel ingest → historical export → cutover) with rollback trigger; PostHog data deletion after migration (GDPR Art. 5(1)(e) storage limitation compliance); privacy constraints matrix (six constraints, enforcement locations documented); nightly PII scan (AL-CH-09, P0); SOC 2 CC7.2/CC6.1/CC6.2/C1.2/PI1.2/A1.1/CC9.2 evidence mapping; six auditor evidence artefact files; sixteen-item implementation checklist across M7 prep and M9 sprint.*

*v0.9 additions: §22 AI Coaching Quality Observability — closes the gap between operational observability (§3/§4.5 — tokens, latency) and quality observability (is Victor coaching well?); privacy-first design established (GDPR Art. 9 constraint — no coaching content in any observability signal; all metrics are behavioral proxies computed at session or weekly cohort level); ten-metric quality proxy taxonomy: plan adherence rate, session depth (turns), session abandonment rate, post-coaching workout start rate, workout completion rate (post-coaching), retry rate within session, plan regeneration rate, explicit negative feedback rate (thumbs-down), no-engagement rate (7d), streak grace trigger rate — all computed via pg_cron `victor_quality_daily` table with k-anonymity floor (sample_size ≥ 5); prompt version tracking: `victor_prompt_version` and `prompt_ab_bucket` columns on `coaching_turns`, 14-day baseline window for regression detection; six QA-COACH-SLOs covering adherence (≥ 65%), abandonment (≤ 30%), post-coaching start (≥ 40%), negative feedback (≤ 8%), plan regen (≤ 15%/user/week), no-engagement (≤ 40%); five clinical safety monitoring signals (session duration spike, retry spike, no-action spike, negative feedback P0 at 25%, explicit harm report) with clinical-safety VETO authority path via R-10; six alerting rules AL-COACH-01 through AL-COACH-06 with 72h suppression window after prompt deploys; A/B testing observability framework: `prompt_ab_bucket` tenant-locked random split, 14-day / 500-session minimum test duration, two-proportion z-test (α = 0.05), `victor_ab_results` table DDL with concluded_by field; four DEC-030 HMAC-chained audit events for prompt lifecycle (ab_test_started, ab_test_concluded, prompt_version_promoted, prompt_version_rolled_back); Metabase "Victor Coaching Quality" dashboard spec: 8 panels (adherence trend, abandonment trend, post-coaching start, negative feedback with P0 threshold, quality-by-version grouped bar, active A/B test status, no-engagement area chart, streak grace line); `victor_quality_daily` table DDL with UNIQUE constraint, RLS (founder + ml-engineer + clinical-safety full access; tenant_admin scoped to own tenant_id; form_system write), 24-month retention pg_cron cleanup; SOC 2 evidence mapping: CC7.2 (quality monitoring), CC7.4 (prompt regression detection via victor_ab_results), CC1.2 (jailbreak CI gate AL-COACH-05), CC5.2 (clinical safety escalation path), C1.1 (no coaching content in observability — RLS policy evidence); twelve-item implementation checklist across M3/M4/M6.*

*v1.0 additions: §23 Enterprise SLA Reporting & Credit Calculation — closes the documented gap at `docs/SOC2_READINESS.md §2 A1.1` ("SLA credit calculation and process"); 99.9% monthly uptime SLA definition (43.8-minute covered-downtime budget) with partial-service degradation at 50% weight and per-tenant measurement scope; uptime measurement methodology dual-source: Better Stack synthetic probes (S-001 through S-013 per §16) as primary, Cloudflare Analytics Engine SQL as corroborating secondary (conservative reconciliation rule); probe S-013 added (Admin Dashboard API health); ten exclusion categories (upstream provider, scheduled maintenance, customer-side IdP misconfiguration, force majeure, synthetic probe false positives, beta features); five-tier credit schedule (0% SLA met → 5% at 99.0–99.9% → 15% at 95–99% → 25% at 90–95% → 50% < 90%) applied against monthly MRR with 50% credit cap; automated credit application without tenant claim requirement for credits ≥ 5%; three-table Postgres schema (`sla_monthly_reports` with generated availability_pct and sla_met columns, `sla_incidents` with downtime_weight for partial-outage support, `maintenance_windows` with 4-hour per-window cap and 72-hour notification constraint) with RLS tenant isolation; `GET /api/v1/admin/sla/current` and `/history` API specs; monthly PDF delivery via Resend on 3rd business day of each month; ten DEC-030 HMAC-chained event types covering the full incident→credit→dispute lifecycle; SOC 2 A1.1 and CC7.2 evidence mapping with four compliance artefact paths (JSON monthly report, Better Stack CSV, credit ledger, DEC-030 audit log filtered on `sla.*`); twelve-item implementation checklist across M4/M5.*

---

## 24. Subscription & Revenue Event Observability

### 24.1 Purpose & SOC 2 Gap Closed

SOC 2 Type II Processing Integrity criteria PI1.1 through PI1.5 require that system processing is **complete** (PI1.1), **accurate** (PI1.2), **timely** (PI1.3), **authorized** (PI1.4), and **valid** (PI1.5). Of all processing surfaces in FORM, subscription billing carries the highest PI risk: a mis-processed webhook can silently grant or revoke premium feature access for real users, affect revenue recognition, and — in the enterprise tier — miscount licensed seats against a contractual commitment.

The current `docs/SOC2_READINESS.md` PI1.1–PI1.5 gap row is partially addressed by the `row-count-monitor` Edge Function, which confirms that rows are being written but does not verify state correctness, transition validity, cross-system consistency, or the completeness of billing lifecycle events. This section closes that gap by defining:

- A formal subscription state machine with monitored transitions (PI1.4 — authorized, PI1.5 — valid).
- An event taxonomy covering every billing lifecycle event from both RevenueCat (consumer) and Stripe (enterprise) (PI1.1 — complete).
- SLIs and SLOs that measure delivery lag, error rate, state consistency, and seat accuracy (PI1.2 — accurate, PI1.3 — timely).
- Alerting rules that fire on anomalies in any of the above.
- A daily consistency check that verifies end-to-end state correctness across all users and tenants.
- DEC-030 HMAC-chained audit events for every billing state change, ensuring a tamper-evident record for auditors.

**Auditor reference:** Evidence artefacts produced by this section are listed in §24.9 (consistency check results table), §24.10 (DEC-030 event registry), and the implementation checklist at §24.14. The primary SOC 2 criterion map is in §24.12.

---

### 24.2 Subscription State Machine

FORM operates two parallel billing channels that converge on the same `users.subscription_tier` column in Supabase Postgres. Both channels must be monitored independently.

#### Consumer tier (RevenueCat → Supabase)

```
          ┌──────────────────────────────────────────────────────────────────┐
          │                    CONSUMER SUBSCRIPTION                         │
          │                                                                  │
          │   [new user]                                                     │
          │       │                                                          │
          │       ▼                                                          │
          │   trialing ──────────────────────────────────────────► canceled  │
          │       │          trial_expired / manual_cancel                  │
          │       │ trial_converted                                          │
          │       ▼                                                          │
          │    active ──── payment_failed ──► past_due ── recovered ──► active│
          │       │                               │                          │
          │       │ manual_cancel                 │ grace_period_expired     │
          │       │                               ▼                          │
          │       ├──────────────────────────► canceled                      │
          │       │                                                          │
          │       │ paused (voluntary)                                       │
          │       ▼                                                          │
          │    paused ──── resume ──────────────────────────────────► active  │
          │       │                                                          │
          │       │ pause_period_expired                                     │
          │       ▼                                                          │
          │   canceled                                                       │
          └──────────────────────────────────────────────────────────────────┘
```

| From state | To state | Trigger | Valid |
|---|---|---|---|
| `trialing` | `active` | `initial_purchase` / `trial_converted` RevenueCat event | Yes |
| `trialing` | `canceled` | `expiration` / `cancellation` RevenueCat event | Yes |
| `active` | `past_due` | `billing_issue` RevenueCat event (payment failure) | Yes |
| `active` | `canceled` | `cancellation` RevenueCat event | Yes |
| `active` | `paused` | `product_change` / voluntary pause RevenueCat event | Yes |
| `past_due` | `active` | `renewal` / `billing_issue_resolved` RevenueCat event | Yes |
| `past_due` | `canceled` | grace period expiry (72h) | Yes |
| `paused` | `active` | `renewal` after pause end RevenueCat event | Yes |
| `paused` | `canceled` | pause period expires without renewal | Yes |
| `canceled` | `active` | `renewal` (resubscribe) RevenueCat event | Yes |
| **any** | `trialing` | **No trigger** — backward transition | **INVALID — AL-SUB-01** |
| `canceled` | `past_due` | **No trigger** | **INVALID — AL-SUB-01** |

#### Enterprise tier (Stripe Invoicing → Supabase)

```
          ┌──────────────────────────────────────────────────────────────────┐
          │                    ENTERPRISE SUBSCRIPTION                       │
          │                                                                  │
          │   [contract signed]                                              │
          │       │                                                          │
          │       ▼                                                          │
          │    active ──── invoice_payment_failed ──► past_due               │
          │       │                                       │                  │
          │       │                           invoice_paid│ (retry)          │
          │       │                                       ▼                  │
          │       │                                    active                │
          │       │                                       │                  │
          │       │ contract_end / manual_cancel          │ overdue > 30d    │
          │       ▼                                       ▼                  │
          │   canceled ◄──────────────────────────── canceled               │
          └──────────────────────────────────────────────────────────────────┘
```

Enterprise subscriptions do not have a `trialing` state; they activate immediately upon contract execution. The `paused` state is not used for enterprise; voluntary suspension requires a contract amendment and manual admin action, which is logged as `source = 'manual_admin'` in `subscription_events`.

**Invalid transition detection:** Any state transition not in the tables above must immediately fire AL-SUB-01 (P0) and emit a `billing.subscription_state_changed` DEC-030 CRITICAL event with `invalid_transition = true`. The transition must be written to `subscription_events` with `event_type = 'invalid_transition_detected'` so the audit chain is preserved, but `users.subscription_tier` must NOT be updated.

---

### 24.3 Revenue Event Taxonomy

All events below are ingested via the Cloudflare Worker webhook receiver (`workers/billing/webhook-receiver.ts`, §24.6). Each event is persisted to `subscription_events` (§24.7) and emits the corresponding DEC-030 HMAC-chained audit event (§24.10).

| Event type | Source | Description | Severity | DEC-030 classification |
|---|---|---|---|---|
| `subscription_created` | RevenueCat / Stripe | New subscription record created (first purchase or enterprise contract activation) | INFO | STANDARD |
| `subscription_renewed` | RevenueCat / Stripe | Successful auto-renewal or manual invoice payment | INFO | STANDARD |
| `subscription_canceled` | RevenueCat / Stripe / `manual_admin` | Subscription terminated; access revocation scheduled | WARN | HIGH |
| `subscription_paused` | RevenueCat | Voluntary pause; grace period clock not started | WARN | STANDARD |
| `subscription_reactivated` | RevenueCat / Stripe | Canceled subscription restarted by customer | INFO | STANDARD |
| `subscription_grace_period_entered` | RevenueCat | Payment failed; user enters 72h grace period before access revocation | WARN | HIGH |
| `payment_failed` | RevenueCat / Stripe | Payment attempt declined; retry scheduled | WARN | HIGH |
| `payment_recovered` | RevenueCat / Stripe | Previously failed payment succeeded; state returns to `active` | INFO | STANDARD |
| `refund_issued` | RevenueCat / Stripe | Refund processed; subscription status depends on refund type | WARN | HIGH |
| `enterprise_seat_added` | Stripe | Seat quantity increased on enterprise subscription | INFO | STANDARD |
| `enterprise_seat_removed` | Stripe | Seat quantity decreased on enterprise subscription | WARN | STANDARD |
| `enterprise_invoice_paid` | Stripe | Enterprise invoice paid; covers period `valid_from`–`valid_to` | INFO | STANDARD |
| `enterprise_invoice_overdue` | Stripe | Invoice unpaid past 30-day net terms | CRITICAL | CRITICAL |
| `invalid_transition_detected` | system | State machine violation detected (see §24.2) | CRITICAL | CRITICAL |

**Source definitions:**
- `revenuecat` — webhook delivered by RevenueCat to `POST /api/v1/billing/webhook/revenuecat`.
- `stripe` — webhook delivered by Stripe to `POST /api/v1/billing/webhook/stripe`.
- `manual_admin` — action performed by a FORM admin via the admin dashboard; requires two-factor confirmation and writes actor `user_id` to `subscription_events.actor_id`.

---

### 24.4 SLIs and SLOs

The following SLOs govern the subscription processing pipeline. All SLOs are measured continuously; breach of any P0 SLO triggers an immediate PagerDuty alert routed to the on-call engineer and the billing-owner.

| ID | Name | SLI | Target | Error budget (30d) | Alert |
|---|---|---|---|---|---|
| **SUB-SLO-01** | Webhook delivery lag | P95 time from RevenueCat/Stripe event timestamp to `subscription_events.processed_at`, measured from `billing.webhook.lag_ms` histogram | 99% of webhooks processed within **30 seconds** | ≤ 1% of webhooks may exceed 30s (≈ 432 webhooks/30d at 1 webhook/min) | AL-SUB-05 fires at P95 > 60s over 5-min window |
| **SUB-SLO-02** | Webhook processing error rate | `billing.webhook.processed{outcome="error"}` / `billing.webhook.received` over 24h rolling window | **< 0.5%** error rate | 0.5% error budget over 24h rolling; 2% triggers immediate alert | AL-SUB-03 fires at > 2% over 15-min window |
| **SUB-SLO-03** | Subscription state consistency | % of users where `users.subscription_tier` matches the terminal state derived from `subscription_events` latest row for that `user_id` | **100%** at any point-in-time consistency check | Zero tolerance: any discrepancy is a P0 | AL-SUB-01 fires on any mismatch; §24.9 daily check |
| **SUB-SLO-04** | Enterprise seat count accuracy | % of enterprise tenants where `tenant_seats.seat_count` equals Stripe subscription `quantity` within 5 minutes of any seat-change event | **100%** within 5 minutes | Zero tolerance: any delta > 0 persisting > 5 min is a P1 | AL-SUB-04 fires on delta > 0 for > 5 min |
| **SUB-SLO-05** | Grace period enforcement | No user whose grace period has expired by more than 72 hours retains premium feature access | **Zero violations** | Zero tolerance: any violation is a P0 incident | AL-SUB-02 fires on detection |

**Measurement infrastructure:**
- SUB-SLO-01 and SUB-SLO-02: Cloudflare Analytics Engine, metrics emitted by `workers/billing/webhook-receiver.ts` via the `billing.*` metric namespace.
- SUB-SLO-03: Daily pg_cron `billing-consistency-check` (§24.9) + real-time AL-SUB-01 from state machine validator.
- SUB-SLO-04: pg_cron `seat-reconciliation` every 5 minutes (§24.8).
- SUB-SLO-05: pg_cron `grace-period-enforcer` runs every 30 minutes; queries users in `past_due` state with `valid_to < now() - interval '72 hours'` and revokes access, emitting `billing.access_revoked` DEC-030 CRITICAL event per user.

---

### 24.5 Alerting Rules

All alerts are routed through PagerDuty. P0 and P1 alerts page the on-call engineer immediately. P2 creates a Linear ticket. P3 writes to the observability Slack channel only.

| Alert ID | Severity | Condition | Evaluation window | Routing | Runbook |
|---|---|---|---|---|---|
| **AL-SUB-01** | **P0** | Impossible state transition detected: `subscription_events` row with `(from_state, to_state)` pair not in the valid-transitions table (§24.2), OR `users.subscription_tier` does not match latest terminal `subscription_events` row for any `user_id` | Real-time (event-driven, fired in webhook receiver) | PagerDuty P0 → on-call → billing-owner within 5 min | `docs/runbooks/RB-SUB-01.md` |
| **AL-SUB-02** | **P0** | Grace period overshoot: any `user_id` in state `past_due` / `canceled` whose `subscription_events.valid_to < now() - interval '72 hours'` AND `users.subscription_tier = 'premium'` | Every 30 min (pg_cron grace-period-enforcer) | PagerDuty P0 → on-call → billing-owner | `docs/runbooks/RB-SUB-02.md` |
| **AL-SUB-03** | **P1** | Webhook error rate: `billing.webhook.processed{outcome="error"}` / `billing.webhook.received` > 2% over any 15-minute rolling window | 15-min rolling, evaluated every 60s | PagerDuty P1 → on-call | `docs/runbooks/RB-SUB-03.md` |
| **AL-SUB-04** | **P1** | Enterprise seat drift: `tenant_seats.seat_count` ≠ Stripe `subscription.quantity` for any enterprise `tenant_id`, persisting for > 5 minutes | 5-min pg_cron `seat-reconciliation` (§24.8) | PagerDuty P1 → on-call → customer-success | `docs/runbooks/RB-SUB-04.md` |
| **AL-SUB-05** | **P1** | RevenueCat/Stripe webhook delivery lag: `billing.webhook.lag_ms` P95 > 60,000 ms over any 5-minute window | 5-min rolling, evaluated every 60s in Cloudflare Analytics Engine | PagerDuty P1 → on-call | `docs/runbooks/RB-SUB-05.md` |
| **AL-SUB-06** | **P2** | Elevated refund rate: refund events in the trailing 7 days exceed 5% of renewal events in the same window | Daily pg_cron `billing-consistency-check` (§24.9) | Linear ticket → billing-owner + fraud-review | `docs/runbooks/RB-SUB-06.md` |
| **AL-SUB-07** | **P2** | Enterprise invoice overdue: `enterprise_invoice_overdue` event present for any `tenant_id` with `created_at < now() - interval '30 days'` | Daily pg_cron `billing-consistency-check` (§24.9) | Linear ticket → customer-success escalation | `docs/runbooks/RB-SUB-07.md` |
| **AL-SUB-08** | **P3** | Subscription state orphan: `user_id` present in `subscription_events` with no matching row in `users` table | Daily pg_cron `billing-consistency-check` (§24.9) | Slack `#observability` | `docs/runbooks/RB-SUB-08.md` |

**Alert suppression:** AL-SUB-03, AL-SUB-05 are suppressed for 10 minutes immediately following a RevenueCat or Stripe announced incident (detected via their respective status-page webhooks) to avoid noise during upstream outages. Suppression itself is DEC-030 logged as `billing.alert_suppressed` (STANDARD).

---

### 24.6 Webhook Receiver Monitoring

**Worker:** `workers/billing/webhook-receiver.ts`  
**Routes:** `POST /api/v1/billing/webhook/revenuecat`, `POST /api/v1/billing/webhook/stripe`

#### Metrics emitted

| Metric name | Type | Labels | Description |
|---|---|---|---|
| `billing.webhook.received` | Counter | `source` (`revenuecat` \| `stripe`) | Incremented on every inbound webhook request, before any validation |
| `billing.webhook.processed` | Counter | `source`, `outcome` (`success` \| `idempotent` \| `error`), `event_type` | Incremented after processing completes; `idempotent` means duplicate suppressed by idempotency key |
| `billing.webhook.lag_ms` | Histogram | `source` | Milliseconds between event source timestamp and `processed_at`; buckets: 1000, 5000, 10000, 30000, 60000, 120000 |
| `billing.webhook.signature_invalid` | Counter | `source` | Incremented on HMAC/signature verification failure; feeds AL-SUB-01 |

#### Idempotency

Each webhook source provides a unique event identifier. The receiver stores this as `subscription_events.idempotency_key` (UNIQUE constraint). On duplicate delivery, the row insert is skipped and `billing.webhook.processed{outcome="idempotent"}` is incremented. The idempotency key is:

- **RevenueCat:** `transaction_id` from the webhook body (present on all purchase, renewal, cancellation, and billing-issue events).
- **Stripe:** `event.id` from the top-level event envelope (globally unique `evt_*` identifier).

#### HMAC signature verification

Before any event processing, the worker verifies the webhook signature:

```typescript
// workers/billing/webhook-receiver.ts (excerpt)

import { Env } from '../types';

async function verifyRevenueCatSignature(
  request: Request,
  env: Env
): Promise<boolean> {
  const authHeader = request.headers.get('Authorization');
  if (!authHeader) return false;
  // RevenueCat uses Bearer token matching REVENUECAT_WEBHOOK_AUTH_KEY
  return authHeader === `Bearer ${env.REVENUECAT_WEBHOOK_AUTH_KEY}`;
}

async function verifyStripeSignature(
  rawBody: string,
  sigHeader: string,
  env: Env
): Promise<boolean> {
  const secret = env.STRIPE_WEBHOOK_SIGNING_SECRET;
  const encoder = new TextEncoder();
  const key = await crypto.subtle.importKey(
    'raw',
    encoder.encode(secret),
    { name: 'HMAC', hash: 'SHA-256' },
    false,
    ['verify']
  );
  // Parse Stripe signature header: t=<timestamp>,v1=<signature>
  const parts = Object.fromEntries(
    sigHeader.split(',').map(p => p.split('=') as [string, string])
  );
  const signedPayload = `${parts['t']}.${rawBody}`;
  const expectedSig = await crypto.subtle.sign(
    'HMAC',
    key,
    encoder.encode(signedPayload)
  );
  const expectedHex = Array.from(new Uint8Array(expectedSig))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('');
  return expectedHex === parts['v1'];
}

export async function handleBillingWebhook(
  request: Request,
  env: Env,
  source: 'revenuecat' | 'stripe'
): Promise<Response> {
  // 1. Increment received counter
  env.ANALYTICS.writeDataPoint({
    blobs: [source],
    indexes: ['billing.webhook.received'],
  });

  const rawBody = await request.text();
  let signatureValid: boolean;

  if (source === 'revenuecat') {
    signatureValid = await verifyRevenueCatSignature(request, env);
  } else {
    const sigHeader = request.headers.get('Stripe-Signature') ?? '';
    signatureValid = await verifyStripeSignature(rawBody, sigHeader, env);
  }

  if (!signatureValid) {
    env.ANALYTICS.writeDataPoint({
      blobs: [source],
      indexes: ['billing.webhook.signature_invalid'],
    });
    // Emit DEC-030 CRITICAL event — AL-SUB-01 path
    await emitDec030Event(env, {
      event_type: 'billing.webhook_signature_invalid',
      severity: 'CRITICAL',
      source,
      trace_id: request.headers.get('cf-ray') ?? crypto.randomUUID(),
    });
    return new Response('Unauthorized', { status: 401 });
  }

  // 2. Parse, deduplicate, persist, and emit DEC-030 events
  // ... (remainder of processing pipeline)
}
```

**On signature failure:** The worker increments `billing.webhook.signature_invalid`, emits a DEC-030 `billing.webhook_signature_invalid` CRITICAL event, and returns HTTP 401. The failure is counted against SUB-SLO-02 as an `error` outcome. If the signature-failure rate exceeds 10 events in any 5-minute window, AL-SUB-01 (P0) fires with routing to the security-engineer in addition to on-call, as repeated failures indicate a possible credential compromise or active spoofing attempt.

---

### 24.7 Subscription Events Table

The `subscription_events` table implements event-sourcing for all billing state changes. It is the authoritative record for subscription history and is read by the daily consistency check (§24.9), the state machine validator (§24.2), and all DEC-030 audit queries. Raw webhook payloads are never stored; only the SHA-256 hash of the incoming payload is recorded for GDPR compliance (see §24.13).

```sql
-- migrations/024_subscription_events.sql

CREATE TYPE subscription_source AS ENUM ('revenuecat', 'stripe', 'manual_admin');

CREATE TABLE subscription_events (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id                 UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    tenant_id               UUID REFERENCES tenants(id) ON DELETE RESTRICT,  -- NULL for consumer-tier users
    event_type              TEXT NOT NULL,                                    -- see §24.3 taxonomy
    source                  subscription_source NOT NULL,
    from_state              TEXT,                                             -- state before transition; NULL for subscription_created
    to_state                TEXT NOT NULL,                                    -- state after transition
    raw_payload_hash        TEXT NOT NULL,                                    -- SHA-256 of incoming webhook body; NOT the payload itself (GDPR)
    subscription_tier       TEXT NOT NULL,                                    -- 'free' | 'premium' | 'enterprise'
    valid_from              TIMESTAMPTZ NOT NULL,                             -- start of the entitlement period granted by this event
    valid_to                TIMESTAMPTZ,                                      -- end of the entitlement period; NULL = indefinite (active)
    idempotency_key         TEXT NOT NULL,                                    -- RevenueCat transaction_id or Stripe event.id
    actor_id                UUID REFERENCES users(id),                       -- populated only for source = 'manual_admin'
    processed_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
    processing_duration_ms  INT NOT NULL,                                    -- wall-clock ms from webhook receipt to this row committed
    created_at              TIMESTAMPTZ NOT NULL DEFAULT now(),

    CONSTRAINT subscription_events_idempotency_key_unique UNIQUE (idempotency_key),
    CONSTRAINT subscription_events_valid_period_check CHECK (
        valid_to IS NULL OR valid_to > valid_from
    ),
    CONSTRAINT subscription_events_processing_duration_positive CHECK (
        processing_duration_ms >= 0
    )
);

-- Indexes for state machine queries and consistency checks
CREATE INDEX subscription_events_user_id_created_at_idx
    ON subscription_events (user_id, created_at DESC);

CREATE INDEX subscription_events_tenant_id_idx
    ON subscription_events (tenant_id)
    WHERE tenant_id IS NOT NULL;

CREATE INDEX subscription_events_event_type_idx
    ON subscription_events (event_type, created_at DESC);

CREATE INDEX subscription_events_to_state_idx
    ON subscription_events (to_state, valid_to)
    WHERE valid_to IS NOT NULL;

-- Retention: 7 years (financial records, SOC 2 PI evidence)
-- pg_cron cleanup is NOT applied to this table; rows are retained for the full 7-year period.
-- A separate pg_cron 'subscription_events_archive' job moves rows older than 7 years to
-- the compliance cold-storage bucket (R2) and deletes from primary DB after SHA-256 verification.

-- Row-Level Security
ALTER TABLE subscription_events ENABLE ROW LEVEL SECURITY;

-- form_system role: write-only (INSERT), no SELECT/UPDATE/DELETE via application path
CREATE POLICY subscription_events_form_system_insert
    ON subscription_events FOR INSERT
    TO form_system
    WITH CHECK (true);

-- tenant_admin: read rows for their own tenant (enterprise) or own user_id (consumer)
CREATE POLICY subscription_events_tenant_admin_select
    ON subscription_events FOR SELECT
    TO tenant_admin
    USING (
        tenant_id = current_setting('app.current_tenant_id', true)::UUID
        OR user_id = current_setting('app.current_user_id', true)::UUID
    );

-- compliance_officer: full read across all tenants; write is prohibited
CREATE POLICY subscription_events_compliance_officer_select
    ON subscription_events FOR SELECT
    TO compliance_officer
    USING (true);

-- No UPDATE or DELETE policies: this table is append-only.
-- Corrections are made by inserting a compensating event with event_type = 'correction'
-- and actor_id set to the admin performing the correction.

COMMENT ON TABLE subscription_events IS
    'Append-only event-sourced ledger for all subscription billing state changes. '
    'SOC 2 PI1.1–PI1.5 evidence. Raw payloads not stored (GDPR); SHA-256 hash only.';
```

---

### 24.8 Enterprise Seat Reconciliation

The pg_cron job `seat-reconciliation` runs every 5 minutes and is the enforcement mechanism for SUB-SLO-04. It compares the live `tenant_seats` seat count in Supabase Postgres against the authoritative seat quantity on the Stripe subscription object, using a Cloudflare Worker proxy to avoid direct Stripe API access from Postgres.

#### Reconciliation query

```sql
-- Executed inside the 'seat-reconciliation' pg_cron function
-- Called via: SELECT cron.schedule('seat-reconciliation', '*/5 * * * *', $$...$$);

WITH stripe_quantities AS (
    -- Fetch current Stripe seat quantities via Worker proxy
    -- The Worker at /internal/stripe/seat-quantities returns:
    --   { tenant_id: UUID, stripe_quantity: INT }[]
    -- authenticated by service-role JWT
    SELECT
        (value->>'tenant_id')::UUID  AS tenant_id,
        (value->>'stripe_quantity')::INT AS stripe_quantity
    FROM
        http_get(
            current_setting('app.internal_worker_base_url') || '/internal/stripe/seat-quantities',
            ARRAY[
                ARRAY['Authorization', 'Bearer ' || current_setting('app.service_role_key')]
            ]
        ) AS resp,
        json_array_elements(resp.content::json) AS value
),
local_counts AS (
    SELECT
        tenant_id,
        seat_count
    FROM tenant_seats
    WHERE subscription_tier = 'enterprise'
),
drift AS (
    SELECT
        lc.tenant_id,
        lc.seat_count          AS local_count,
        sq.stripe_quantity     AS stripe_count,
        ABS(lc.seat_count - sq.stripe_quantity) AS delta,
        now()                  AS checked_at
    FROM local_counts lc
    JOIN stripe_quantities sq USING (tenant_id)
    WHERE lc.seat_count <> sq.stripe_quantity
)
INSERT INTO seat_reconciliation_log
    (tenant_id, local_count, stripe_count, delta, checked_at, resolved)
SELECT
    tenant_id,
    local_count,
    stripe_count,
    delta,
    checked_at,
    false
FROM drift
ON CONFLICT (tenant_id, checked_at) DO NOTHING;
```

#### Drift escalation logic

After the INSERT above, the pg_cron function checks whether any `tenant_id` has an unresolved drift row with `checked_at < now() - interval '5 minutes'`:

```sql
SELECT COUNT(*) AS persistent_drift_count
FROM seat_reconciliation_log
WHERE resolved = false
  AND checked_at < now() - interval '5 minutes';
```

If `persistent_drift_count > 0`, the function calls the internal Worker endpoint `POST /internal/dec030/emit` with event type `billing.seat_drift_detected` (CRITICAL severity). This feeds AL-SUB-04.

#### seat_reconciliation_log schema

```sql
CREATE TABLE seat_reconciliation_log (
    id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id    UUID NOT NULL REFERENCES tenants(id),
    local_count  INT NOT NULL,
    stripe_count INT NOT NULL,
    delta        INT NOT NULL,
    checked_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    resolved     BOOLEAN NOT NULL DEFAULT false,
    resolved_at  TIMESTAMPTZ,
    resolved_by  UUID REFERENCES users(id),
    created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),

    CONSTRAINT seat_reconciliation_log_unique_check UNIQUE (tenant_id, checked_at)
);

CREATE INDEX seat_reconciliation_log_unresolved_idx
    ON seat_reconciliation_log (tenant_id, checked_at)
    WHERE resolved = false;

ALTER TABLE seat_reconciliation_log ENABLE ROW LEVEL SECURITY;

CREATE POLICY seat_recon_compliance_officer_select
    ON seat_reconciliation_log FOR SELECT TO compliance_officer USING (true);

CREATE POLICY seat_recon_tenant_admin_select
    ON seat_reconciliation_log FOR SELECT TO tenant_admin
    USING (tenant_id = current_setting('app.current_tenant_id', true)::UUID);
```

---

### 24.9 Revenue Consistency Check

The pg_cron job `billing-consistency-check` runs daily at **03:00 UTC**. It performs three independent checks across the full user and tenant population and writes results to `billing_consistency_checks`. Any check failure emits a DEC-030 CRITICAL event and triggers a PagerDuty P0 alert via `billing.consistency_check_failed`.

#### Check A — Active subscribers have a valid entitlement row

```sql
-- Check A: Every user with subscription_tier = 'premium' or 'enterprise'
-- must have a subscription_events row where valid_to IS NULL OR valid_to > now()
INSERT INTO billing_consistency_checks
    (check_name, check_date, passed, failure_count, failure_sample)
SELECT
    'active_subscriber_entitlement_coverage',
    now(),
    COUNT(*) = 0,
    COUNT(*),
    jsonb_agg(jsonb_build_object('user_id', u.id, 'tier', u.subscription_tier) ORDER BY u.id LIMIT 50)
FROM users u
WHERE u.subscription_tier IN ('premium', 'enterprise')
  AND NOT EXISTS (
      SELECT 1 FROM subscription_events se
      WHERE se.user_id = u.id
        AND se.to_state = 'active'
        AND (se.valid_to IS NULL OR se.valid_to > now())
  );
```

#### Check B — No premium access without a valid subscription record

```sql
-- Check B: No user has subscription_tier = 'premium' without a matching
-- subscription_events entry (catches manual DB edits that bypass the event pipeline)
INSERT INTO billing_consistency_checks
    (check_name, check_date, passed, failure_count, failure_sample)
SELECT
    'no_premium_without_subscription_event',
    now(),
    COUNT(*) = 0,
    COUNT(*),
    jsonb_agg(jsonb_build_object('user_id', u.id) ORDER BY u.id LIMIT 50)
FROM users u
WHERE u.subscription_tier = 'premium'
  AND NOT EXISTS (
      SELECT 1 FROM subscription_events se
      WHERE se.user_id = u.id
        AND se.subscription_tier = 'premium'
  );
```

#### Check C — Enterprise seat counts match Stripe

```sql
-- Check C: tenant_seats.seat_count = Stripe quantity for all enterprise tenants
-- (relies on the same Worker proxy used by seat-reconciliation in §24.8)
INSERT INTO billing_consistency_checks
    (check_name, check_date, passed, failure_count, failure_sample)
WITH stripe_quantities AS (
    SELECT
        (value->>'tenant_id')::UUID  AS tenant_id,
        (value->>'stripe_quantity')::INT AS stripe_quantity
    FROM
        http_get(
            current_setting('app.internal_worker_base_url') || '/internal/stripe/seat-quantities',
            ARRAY[ARRAY['Authorization', 'Bearer ' || current_setting('app.service_role_key')]]
        ) AS resp,
        json_array_elements(resp.content::json) AS value
)
SELECT
    'enterprise_seat_count_stripe_parity',
    now(),
    COUNT(*) = 0,
    COUNT(*),
    jsonb_agg(jsonb_build_object(
        'tenant_id', ts.tenant_id,
        'local_count', ts.seat_count,
        'stripe_count', sq.stripe_quantity
    ) ORDER BY ts.tenant_id LIMIT 50)
FROM tenant_seats ts
JOIN stripe_quantities sq USING (tenant_id)
WHERE ts.seat_count <> sq.stripe_quantity;
```

#### billing_consistency_checks schema

```sql
CREATE TABLE billing_consistency_checks (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    check_name      TEXT NOT NULL,
    check_date      TIMESTAMPTZ NOT NULL DEFAULT now(),
    passed          BOOLEAN NOT NULL,
    failure_count   INT NOT NULL DEFAULT 0,
    failure_sample  JSONB,   -- up to 50 anonymised failing user_id / tenant_id references
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX billing_consistency_checks_check_date_idx
    ON billing_consistency_checks (check_date DESC, check_name);

ALTER TABLE billing_consistency_checks ENABLE ROW LEVEL SECURITY;

CREATE POLICY billing_consistency_checks_compliance_officer
    ON billing_consistency_checks FOR SELECT TO compliance_officer USING (true);

CREATE POLICY billing_consistency_checks_form_system_insert
    ON billing_consistency_checks FOR INSERT TO form_system WITH CHECK (true);
```

**Failure escalation:** After all three checks are inserted, the pg_cron function executes:

```sql
SELECT COUNT(*) INTO v_failures
FROM billing_consistency_checks
WHERE check_date >= now() - interval '1 hour'
  AND passed = false;
```

If `v_failures > 0`, the function POSTs to `POST /internal/dec030/emit` with `billing.consistency_check_failed` (CRITICAL) and to `POST /internal/pagerduty/trigger` with severity P0, incident title `billing-consistency-check-failed`, and a JSON body listing the failed check names and failure counts. The on-call engineer and billing-owner are paged.

---

### 24.10 DEC-030 HMAC-Chained Audit Events

All billing audit events are written to the HMAC-chained append-only audit log defined in `docs/AUDIT_LOG_SCHEMA.md`. Each event carries the standard DEC-030 envelope (`trace_id`, `actor_id`, `tenant_id`, `timestamp`, `prev_hmac`) plus the billing-specific fields listed below. All events are retained for **7 years** (financial records).

`actor_id` for machine-generated events (all `source ≠ 'manual_admin'` events) is set to the service account identifier `form_system`. For `manual_admin` events, `actor_id` is the authenticated FORM admin user ID.

`user_id_hash` in the DEC-030 envelope is the SHA-256 hash of the plaintext `user_id` UUID — never the plaintext UUID itself. This applies to all `billing.*` events.

| DEC-030 event type | Severity | Trigger | Key fields (beyond standard envelope) |
|---|---|---|---|
| `billing.subscription_state_changed` | STANDARD | Any valid state transition persisted to `subscription_events` | `user_id_hash`, `tenant_id`, `from_state`, `to_state`, `subscription_tier`, `source`, `idempotency_key` |
| `billing.payment_failed` | HIGH | `payment_failed` event received and processed | `user_id_hash`, `tenant_id`, `source`, `retry_attempt`, `idempotency_key` |
| `billing.grace_period_entered` | HIGH | `subscription_grace_period_entered` event processed; 72h clock started | `user_id_hash`, `tenant_id`, `grace_expiry_at`, `idempotency_key` |
| `billing.access_revoked` | CRITICAL | User access revoked after grace period expiry or cancellation | `user_id_hash`, `tenant_id`, `revocation_reason` (`grace_period_expired` \| `canceled` \| `manual_admin`), `actor_id` |
| `billing.refund_issued` | HIGH | `refund_issued` event received and processed | `user_id_hash`, `tenant_id`, `source`, `refund_amount_cents`, `currency`, `idempotency_key` |
| `billing.enterprise_invoice_paid` | STANDARD | `enterprise_invoice_paid` event processed | `tenant_id`, `invoice_id_hash` (SHA-256 of Stripe invoice ID), `amount_cents`, `currency`, `period_start`, `period_end` |
| `billing.enterprise_invoice_overdue` | CRITICAL | `enterprise_invoice_overdue` event processed; invoice unpaid > 30 days | `tenant_id`, `invoice_id_hash`, `amount_cents`, `overdue_days` |
| `billing.seat_count_changed` | STANDARD | `enterprise_seat_added` or `enterprise_seat_removed` event processed | `tenant_id`, `previous_count`, `new_count`, `change_reason`, `idempotency_key` |
| `billing.seat_drift_detected` | CRITICAL | pg_cron `seat-reconciliation` detects persistent delta > 0 for > 5 minutes (§24.8) | `tenant_id`, `local_count`, `stripe_count`, `delta`, `drift_duration_minutes` |
| `billing.consistency_check_failed` | CRITICAL | Any check in the daily `billing-consistency-check` pg_cron job returns `passed = false` (§24.9) | `failed_checks` (array of check names), `total_failures` |
| `billing.webhook_signature_invalid` | CRITICAL | HMAC/Bearer signature verification failure in webhook receiver (§24.6) | `source`, `cf_ray` (Cloudflare Ray ID), `ip_hash` (SHA-256 of requester IP) |

**HMAC chain integrity:** A break in the HMAC chain on any `billing.*` event is a P0 incident per `docs/INCIDENT_RESPONSE.md §R-05`. The chain is verified nightly by the `dec030-chain-verify` pg_cron job. Any gap in the `billing.*` event sequence (missing event for a known `idempotency_key`) is also flagged as a P0.

---

### 24.11 Metabase Revenue Observability Dashboard

**Dashboard name:** "Subscription Health"  
**Access:** billing-owner, compliance-officer, founder, on-call engineer (read-only for on-call)  
**Refresh interval:** 5 minutes  
**Data source:** Supabase Postgres (primary) + Cloudflare Analytics Engine (webhook metrics)

| Panel # | Panel title | Visualisation | Primary query / metric | Alert threshold shown |
|---|---|---|---|---|
| 1 | Webhook Processing Rate & Error Rate (last 24h) | Dual-axis line chart: left = requests/min, right = error % | `billing.webhook.received` and `billing.webhook.processed{outcome="error"}` from Cloudflare Analytics Engine, 5-min buckets | Red band at error rate > 2% (AL-SUB-03 threshold) |
| 2 | Subscription State Distribution | Stacked bar chart, one bar per hour for last 7 days | `SELECT to_state, COUNT(*) FROM subscription_events WHERE created_at > now()-'7 days' GROUP BY to_state, date_trunc('hour', created_at)` | N/A (trend visibility) |
| 3 | Daily Churn Rate (consumer) | Line chart, daily granularity, rolling 30d | `canceled` events / `active` user-days for consumer tier, from `subscription_events` | Reference line at 2% daily churn (advisory) |
| 4 | Grace-Period Users (live count) | Single stat + trend sparkline | `SELECT COUNT(*) FROM users u JOIN subscription_events se ON se.user_id = u.id WHERE se.to_state = 'past_due' AND (se.valid_to IS NULL OR se.valid_to > now())` | Red threshold at any count > 0 where `valid_to < now() - interval '72 hours'` |
| 5 | Enterprise Seat Drift Count | Single stat (target: 0) + last-7d bar chart | `SELECT COUNT(*) FROM seat_reconciliation_log WHERE resolved = false AND checked_at > now() - interval '24 hours'` | Red threshold at count > 0 (AL-SUB-04) |
| 6 | Daily Revenue Events Count | Grouped bar chart by event_type, last 14d | `SELECT event_type, COUNT(*), date_trunc('day', created_at) FROM subscription_events GROUP BY 1,3 ORDER BY 3` | N/A (volume trend) |
| 7 | Refund Rate (rolling 7d) | Gauge + line trend | `refund_issued` count / `subscription_renewed` count over trailing 7d from `subscription_events` | Yellow at > 3%, red at > 5% (AL-SUB-06 threshold) |
| 8 | Billing Consistency Check Status | Status table: one row per check name, latest result | `SELECT check_name, check_date, passed, failure_count FROM billing_consistency_checks WHERE check_date >= now() - interval '25 hours' ORDER BY check_date DESC` | Red row background on `passed = false`; green on `passed = true` |

**Dashboard annotations:** Any AL-SUB-01 or AL-SUB-02 P0 alert that fires is automatically annotated on all time-series panels via the Metabase annotation API, with the alert ID and timestamp. This allows post-incident correlation without requiring access to PagerDuty.

---

### 24.12 SOC 2 PI1 Mapping

| PI criterion | Requirement | Evidence artefact in this section | Owner |
|---|---|---|---|
| **PI1.1** — Processing is complete | All subscription events are captured without omission | `subscription_events` append-only table; idempotency-key uniqueness constraint prevents gaps; `billing-consistency-check` Check A verifies every active subscriber has a valid event row | platform-engineer |
| **PI1.2** — Processing is accurate | State transitions are validated against the state machine; seat counts match Stripe authoritative source | State machine validator in webhook receiver (§24.2); `seat-reconciliation` pg_cron + `seat_reconciliation_log` (§24.8); `billing-consistency-check` Check C (§24.9) | platform-engineer |
| **PI1.3** — Processing is timely | Webhooks are processed within committed latency SLO; seat reconciliation runs every 5 minutes; daily consistency check at 03:00 UTC | SUB-SLO-01 (P95 < 30s), SUB-SLO-04 (5-min reconciliation); `billing.webhook.lag_ms` histogram; AL-SUB-05 fires on lag regression | devops-lead |
| **PI1.4** — Processing is authorized | Webhook signatures are verified before any state change; `manual_admin` events require authenticated session and two-factor; RLS on `subscription_events` prevents unauthorised writes | HMAC/Bearer verification in §24.6; `subscription_events` RLS policies (§24.7); `billing.webhook_signature_invalid` DEC-030 CRITICAL event on auth failure | security-engineer |
| **PI1.5** — Processing is valid | Invalid state transitions are rejected and never written to `users.subscription_tier`; grace period overshoot is detected and alerted; consistency checks flag any data integrity breach | State machine invalid-transition block (§24.2); AL-SUB-01 (impossible transition P0); AL-SUB-02 (grace period overshoot P0); `billing-consistency-check` Check B (§24.9) | platform-engineer |

**Additional SOC 2 criterion coverage:**

| Criterion | Coverage |
|---|---|
| **CC7.2** — Anomaly detection | AL-SUB-01 through AL-SUB-08 alerting rules; `billing.webhook_signature_invalid` CRITICAL event; DEC-030 chain-break detection |
| **CC6.1** — Logical access controls | RLS on `subscription_events`, `seat_reconciliation_log`, `billing_consistency_checks`; `compliance_officer` read-only; `form_system` write-only; `tenant_admin` scoped to own tenant |

**Evidence artefacts for auditors:**
- `compliance/evidence/billing/{YYYY-MM}-consistency-check.json` — machine-readable daily consistency check results, retained 7 years.
- `compliance/evidence/billing/{YYYY-MM}-subscription-events-count.json` — monthly row count and event-type distribution from `subscription_events`, retained 7 years.
- DEC-030 audit log filtered on `event_type LIKE 'billing.%'` — continuous chain-verified log, retained 7 years.
- `seat_reconciliation_log` export: monthly CSV of all drift events per tenant, retained 7 years.

---

### 24.13 Privacy Notes

The following privacy controls are enforced by design. Each is a hard constraint; any change requires a privacy impact assessment reviewed by the compliance-officer and documented in `docs/PRIVACY_IMPACT.md`.

1. **Raw webhook payloads are not stored.** `subscription_events.raw_payload_hash` contains only the SHA-256 hash of the incoming webhook body. This is sufficient for audit integrity verification without retaining payment card data, billing addresses, or other personal data present in raw Stripe/RevenueCat payloads. The raw body is discarded after hash computation and processing, before any await boundary.

2. **No payment card data or billing addresses are stored anywhere in FORM's Supabase database.** All payment instrument data resides exclusively with Stripe and RevenueCat as sub-processors. FORM stores only subscription state, entitlement periods, and identifiers (Stripe customer ID, RevenueCat app user ID) that cannot be used to reconstruct payment details.

3. **`subscription_events` contains no personal data beyond `user_id`.** The table holds subscription state, timestamps, and processing metadata. No name, email address, IP address, device fingerprint, or health-adjacent data is stored in this table. `user_id` is a UUID primary key with no semantic meaning to a third party without access to the `users` table, which is subject to separate RLS policies.

4. **DEC-030 `billing.*` events carry `user_id_hash` (SHA-256 of the plaintext `user_id`), not the plaintext UUID.** This preserves auditability (the same user's events are linkable by hash) while preventing the audit log from becoming a direct lookup table for user identity.

5. **RevenueCat and Stripe are designated sub-processors** under FORM's Data Processing Agreement framework. Both hold current signed DPAs with FORM. RevenueCat's DPA covers EU/UK GDPR and CCPA; Stripe's DPA covers EU/UK GDPR, CCPA, and PCI-DSS. Sub-processor status is recorded in `docs/SUBPROCESSORS.md`.

6. **Tenant-level data minimisation:** The `tenant_id` column in `subscription_events` is NULL for consumer-tier users. Enterprise tenant administrators can only read events for their own `tenant_id` via RLS. A `tenant_admin` cannot enumerate consumer-tier subscription events.

7. **Data subject access requests (DSARs):** A DSAR for subscription history is fulfilled by querying `subscription_events WHERE user_id = :user_id` and returning event type, date, and subscription tier only. `raw_payload_hash` is excluded from DSAR responses as it is a system integrity field, not personal data in meaningful form.

---

### 24.14 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Create `subscription_events` table with full DDL, indexes, and RLS policies per §24.7 | platform-engineer | **P0** | M4 |
| Register `subscription_source` ENUM and all `subscription_events` constraints in migration `024_subscription_events.sql` | platform-engineer | **P0** | M4 |
| Implement `workers/billing/webhook-receiver.ts` with HMAC/Bearer signature verification, idempotency deduplication, state machine validator, and all metric emissions per §24.6 | platform-engineer | **P0** | M4 |
| Create `seat_reconciliation_log` table with DDL and RLS per §24.8 | platform-engineer | **P0** | M4 |
| Implement pg_cron `seat-reconciliation` job (every 5 minutes) per §24.8; include Worker proxy endpoint `GET /internal/stripe/seat-quantities` | platform-engineer | **P0** | M4 |
| Implement pg_cron `billing-consistency-check` job (daily 03:00 UTC) with all three checks and PagerDuty escalation per §24.9 | platform-engineer | **P0** | M4 |
| Create `billing_consistency_checks` table with DDL and RLS per §24.9 | platform-engineer | **P0** | M4 |
| Register all 11 `billing.*` DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` event registry with severity and 7-year retention | security-engineer | **P0** | M4 |
| Implement pg_cron `grace-period-enforcer` job (every 30 minutes) that queries past-due users beyond 72h and revokes access; emits `billing.access_revoked` CRITICAL | platform-engineer | **P0** | M4 |
| Configure AL-SUB-01 through AL-SUB-08 alerting rules in PagerDuty and Cloudflare Analytics Engine; write runbooks `RB-SUB-01` through `RB-SUB-08` | devops-lead | **P1** | M4 |
| Build "Subscription Health" Metabase dashboard with all 8 panels per §24.11; configure 5-minute refresh and AL-SUB-01/AL-SUB-02 annotations | data-engineer | **P1** | M5 |
| Update `docs/SOC2_READINESS.md` PI1.1–PI1.5 gap rows from 🟡 Gap to ✅ Done, citing this section as evidence | compliance-officer | **P0** | M4 |
| Add `billing.webhook_signature_invalid` rate assertion to `__tests__/db/rls_isolation.test.ts` and add webhook-receiver integration tests covering idempotency, signature rejection, and invalid-transition blocking | platform-engineer | **P1** | M5 |
| Add RevenueCat and Stripe to `docs/SUBPROCESSORS.md` with DPA reference dates and data categories per §24.13 | compliance-officer | **P2** | M5 |

---

*v1.1 additions: §24 Subscription & Revenue Event Observability — closes the PI1.1–PI1.5 Processing Integrity gap documented in `docs/SOC2_READINESS.md`, which was previously only partially addressed by the `row-count-monitor` Edge Function; defines the full subscription state machine for both the consumer tier (RevenueCat → Supabase, states: trialing → active → past_due → canceled / paused) and the enterprise tier (Stripe Invoicing → Supabase, states: active → past_due → canceled), with a complete valid-transitions table and an invalid-transition blocking path that prevents `users.subscription_tier` updates on impossible transitions while preserving the audit record; thirteen-event revenue taxonomy covering both sources (`revenuecat`, `stripe`, `manual_admin`) with DEC-030 classification for each event type; five SUB-SLOs governing webhook delivery lag P95 < 30s (SUB-SLO-01), processing error rate < 0.5% over 24h (SUB-SLO-02), 100% subscription state consistency (SUB-SLO-03), 100% enterprise seat count accuracy within 5 minutes of seat changes (SUB-SLO-04), and zero grace-period overshoot beyond 72h (SUB-SLO-05); eight alerting rules AL-SUB-01 through AL-SUB-08 spanning P0 impossible state transitions and grace-period overshoot, P1 webhook error rate / seat drift / delivery lag, P2 elevated refund rate and overdue enterprise invoices, and P3 subscription state orphans; Cloudflare Worker `workers/billing/webhook-receiver.ts` specification with four metric types (`billing.webhook.received`, `billing.webhook.processed`, `billing.webhook.lag_ms`, `billing.webhook.signature_invalid`), HMAC-chained idempotency using RevenueCat `transaction_id` and Stripe `event.id`, full TypeScript signature-verification code for both RevenueCat Bearer-token and Stripe HMAC-SHA-256 webhook authentication; `subscription_events` append-only Postgres DDL with thirteen columns, two CHECK constraints, four indexes, and three RLS policies (form_system write-only, tenant_admin scoped read, compliance_officer full read); enterprise seat-reconciliation pg_cron job running every 5 minutes with full SQL using `http_get` Worker proxy to Stripe, `seat_reconciliation_log` DDL with unresolved-drift index, and DEC-030 CRITICAL emission path on drift persisting beyond 5 minutes; daily `billing-consistency-check` pg_cron at 03:00 UTC with three SQL checks (active-subscriber entitlement coverage, no-premium-without-subscription-event, enterprise-seat-count-stripe-parity), `billing_consistency_checks` DDL, and PagerDuty P0 escalation path; eleven DEC-030 HMAC-chained event types with severity levels and key fields, all retained 7 years; "Subscription Health" Metabase dashboard specification with eight panels (webhook rate/error rate, state distribution, daily churn, grace-period live count, enterprise seat drift, daily revenue events, rolling refund rate, consistency check status); PI1.1–PI1.5 evidence mapping table plus CC7.2 and CC6.1 supplementary coverage; seven privacy constraints (no raw payload storage — SHA-256 hash only; no payment card data in Supabase; no PII beyond user_id in subscription_events; user_id_hash in DEC-030 events; RevenueCat and Stripe DPAs documented; tenant-level data minimisation; DSAR fulfilment scope); fourteen-item implementation checklist across M4/M5 with owners and priorities.*

---

## 25. Product & Activation Funnel Observability

> Owner: growth-lead + devops-lead. Review: monthly or on any funnel architecture change.
> Cross-references: §1 Three Pillars Overview (PostHog pillar), §12 Developer Instrumentation Guide, §24 Subscription & Revenue Event Observability, `docs/FLOWS.md` (F1–F8 user flows).

---

### 25.1 Purpose and Scope

Sections §1–§24 cover infrastructure observability: RED metrics, SLOs, crash rates, error budgets, trace correlation, and billing event integrity. Those signals answer *is the system up and operating correctly?*

This section closes the complementary gap: *are users activating, and is the product delivering its core value loop?* Infrastructure metrics can show a 99.9%-available API with sub-200 ms P95 latency at the same moment that zero users are completing their first coaching session — a product failure that no infrastructure alert would detect.

§25 defines FORM's product observability layer: the PostHog event taxonomy for F1–F8 user flows (see `docs/FLOWS.md`), the activation and engagement SLOs that sit alongside §2's infrastructure SLOs, the alerting rules that fire on product metric regressions, A/B test instrumentation standards, and the privacy constraints that govern what may and may not enter PostHog.

PostHog (EU Cloud, Frankfurt) is the sole destination for all events in this section. It was selected in DEC-XXX (TBD) for open-source pedigree, EU data residency, combined feature flag and analytics surface, and competitive cost versus Amplitude at early-stage scale. See OQ-32 for the open Amplitude comparison question. PostHog is already referenced in the §1 Three Pillars table as the product metrics tool and in §12.5 as an instrumentation checkpoint.

**Privacy-first constraint:** FORM's workout data, body composition trends, recovery scores, and coaching content are GDPR Art. 9 health-adjacent special category data. Nothing in §25 permits health data to enter PostHog. All event schemas below are designed with this constraint as a hard gate. The clinical-safety team holds a veto on any proposed event property that is body-adjacent, injury-adjacent, or could reconstruct an individual's health status from the PostHog event stream.

---

### 25.2 Activation Funnel Definition

FORM defines **"activated"** as: a user who completes at least one full coaching session with CV pose feedback enabled within 7 days of registration [DEC-XXX TBD]. This is the D7 Activation metric (ACT-SLO-01). It is the primary product health signal pre-Series A.

The activation journey maps to flows F1 through F4 in `docs/FLOWS.md`. Flows F5–F8 are engagement and monetisation steps that follow activation.

| Flow ID | Flow Name | Entry Event | Completion Event | Activation Gate | Max Acceptable Drop-off |
|---|---|---|---|---|---|
| F1 | Onboarding | `app.opened` | `onboarding.completed` | YES | < 30% |
| F2 | Daily Check-in | `checkin.started` | `checkin.submitted` | NO | < 15% |
| F3 | Session Start | `session.intent_started` | `session.confirmed` | YES | < 20% |
| F4 | Live Session (CV) | `session.confirmed` | `session.completed_with_cv` | YES | < 25% |
| F5 | Post-Session | `session.completed_with_cv` | `postsession.dismissed` | NO | < 5% |
| F6 | Progress Review | `progress.opened` | `progress.milestone_viewed` | NO | < 40% |
| F7 | Program Change | `program.change_started` | `program.change_confirmed` | NO | < 30% |
| F8 | Paywall / Upgrade | `paywall.shown` | `subscription.purchase_started` | NO | < 85% |

**Activation gate** means the flow is a required step on the path to the D7 Activation definition. Drop-off floors are targets, not hard SLOs — they trigger investigation, not a page. Paging thresholds are defined in §25.5. All drop-off floors are [ESTIMATE] values pending real baseline data from the first 500 installs.

---

### 25.3 PostHog Event Taxonomy

All events below are captured by the mobile clients (iOS and Android) via the PostHog React Native SDK and sent to `https://eu.posthog.com`. No events transit through FORM's Cloudflare Workers — they are sent client-side directly. This architectural choice is intentional: it decouples product analytics uptime from API uptime (see AL-ACT-06 in §25.5 which detects a zero-event window as a signal of a client-side or CDN failure).

**`distinct_id` field:** Every event uses `user_id_hash` (SHA-256 of the plaintext Supabase `user_id`) as the PostHog `distinct_id`. The plaintext `user_id` is never transmitted to PostHog. See §25.7 constraint 4.

**PII risk column:** LOW = no personal data; MEDIUM = pseudonymous or aggregatable; HIGH = must not reach PostHog.

| Event Name | Trigger | Key Properties | PII Risk | Retained in PostHog | Notes |
|---|---|---|---|---|---|
| `app.opened` | App foregrounded or cold-launched | `platform` (ios\|android), `app_version`, `is_fresh_install` (boolean) | LOW | YES | Entry point for all funnel analysis |
| `onboarding.step_viewed` | Each onboarding screen renders | `step_name`, `step_index` | LOW | YES | Used to identify drop-off step within F1 |
| `onboarding.completed` | User taps through final onboarding screen | `duration_seconds`, `profile_complete` (boolean) | LOW | YES | F1 completion event |
| `checkin.started` | Daily check-in flow entered | `source` (push_notification\|manual\|widget) | LOW | YES | F2 entry event |
| `checkin.submitted` | Check-in form submitted | `checkin_completed` (boolean: true) | LOW | YES | F2 completion. Mood score (1–5) and readiness score (1–5) are NOT sent — see §25.7 constraint 1 and OQ-33 |
| `session.intent_started` | User taps "Start Session" | `program_id` (opaque UUID — no health semantics) | LOW | YES | F3 entry. No exercise names, no load data |
| `session.confirmed` | Session parameters confirmed, session begins | `exercise_count`, `estimated_duration_min` | LOW | YES | F3 completion / F4 entry |
| `session.completed_with_cv` | Session ends with CV enabled and at least one rep counted | `duration_actual_min` (rounded to nearest 5 min), `cv_enabled` (boolean), `rep_count_total` | LOW | YES | F4 completion. Weight, load, RPE, and form scores are NOT sent — see §25.7 constraints 1 and 3 |
| `session.abandoned` | User exits mid-session | `step_at_abandon` (warm_up\|working_sets\|cool_down), `cv_enabled` (boolean) | LOW | YES | Used to identify abandonment point within F3/F4 |
| `postsession.dismissed` | Post-session summary screen dismissed | — | LOW | YES | F5 completion event |
| `progress.opened` | Progress screen tapped | `weeks_active` | LOW | YES | F6 entry event |
| `progress.milestone_viewed` | User scrolls to or taps a milestone card | — | LOW | YES | F6 completion event |
| `program.change_started` | User initiates program change flow | — | LOW | YES | F7 entry event |
| `program.change_confirmed` | Program change confirmed | — | LOW | YES | F7 completion event |
| `paywall.shown` | Paywall screen rendered | `source` (session_gate\|feature_gate\|manual), `tier_offered` | LOW | YES | F8 entry event |
| `paywall.dismissed` | Paywall screen dismissed without purchase | `time_shown_sec` | LOW | YES | Complement to F8 conversion |
| `subscription.purchase_started` | User taps purchase CTA on paywall | `tier`, `billing_period` | LOW | YES | F8 completion event. Actual transaction outcome is in §24, not PostHog |
| `victor.message_sent` | User submits a message to Victor | `message_length_bucket` (short\|medium\|long) | LOW | YES | Message content is NOT sent — see §25.7 constraint 2 |
| `victor.response_received` | Victor response rendered in UI | `latency_ms`, `response_length_bucket` (short\|medium\|long) | LOW | YES | Response content is NOT sent |
| `feature.cv_permissions_granted` | User grants camera permission for CV | `platform` (ios\|android) | LOW | YES | Key friction point in F4 |
| `experiment.enrolled` | User assigned to an A/B experiment variant | `experiment_id`, `variant` (control\|treatment_A\|...), `user_id_hash` | LOW | YES | See §25.6. Plaintext `user_id` is NOT the value — `user_id_hash` is a property here too |
| `experiment.converted` | User triggers the primary conversion event of an experiment | `experiment_id`, `variant`, `conversion_event_name` | LOW | YES | See §25.6 |

**Properties that must never appear in any PostHog event:**

| Prohibited Property | Category | Reason |
|---|---|---|
| Weight / load values (kg, lb, 1RM %) | Health data | GDPR Art. 9; clinical-safety veto |
| Mood score (1–5), readiness score (1–5) | Health-adjacent | Excluded pending OQ-33 ruling |
| Body measurements (height, weight, composition) | Health data | GDPR Art. 9 |
| CV keypoints or pose coordinates | Biometric data | GDPR Art. 9 |
| Victor message or response content | Personal data | Free-text health context |
| GPS or location data | Personal data | Not collected by FORM |
| `user_id` (plaintext Supabase UUID) | Personal data | Use `user_id_hash` only |

---

### 25.4 Activation & Engagement SLOs

These SLOs sit alongside the infrastructure SLOs in §2. They are measured in PostHog using funnel and cohort queries, not in Prometheus or Cloudflare Analytics Engine. The measurement cadence is daily or weekly depending on the window — they are not real-time signals.

All targets marked [ESTIMATE] are benchmarks derived from comparable fitness app cohort studies. They must be recalibrated after the first 500 real installs (see OQ-31).

| SLO ID | Definition | Target | Measurement Window | Alerting Threshold | Owner |
|---|---|---|---|---|---|
| ACT-SLO-01 | D7 Activation Rate: % of installs that complete at least one session with CV in the 7 days following registration | ≥ 40% [ESTIMATE] | 7-day rolling cohort | < 35% → AL-ACT-01 | growth-lead |
| ACT-SLO-02 | F1 Onboarding Completion Rate: `onboarding.completed` / `app.opened` for new installs | ≥ 70% [ESTIMATE] | 7-day rolling | < 60% → AL-ACT-02 | ux-flow |
| ACT-SLO-03 | F3 → F4 Session Start Completion: `session.completed_with_cv` / `session.confirmed` | ≥ 80% [ESTIMATE] | 24h rolling | < 70% → AL-ACT-03 | platform-engineer |
| ACT-SLO-04 | D30 Retention: % of D7-activated users with ≥ 1 session in days 8–30 | ≥ 35% [ESTIMATE] | 30-day cohort | < 25% → growth-lead review | growth-lead |
| ACT-SLO-05 | F8 Paywall Conversion: `subscription.purchase_started` / `paywall.shown` | ≥ 15% [ESTIMATE] | 7-day rolling | < 10% → AL-ACT-04 | growth-lead |
| ACT-SLO-06 | PostHog event delivery lag: P95 time from app event to PostHog ingestion | < 60 s (P95) | Rolling 30-min window | P95 > 120 s → AL-ACT-05 | devops-lead |

**Error budget policy for product SLOs:** Product SLOs do not carry a formal error budget in the same sense as infrastructure SLOs (§2.2). Instead, breaching an alerting threshold triggers a structured investigation (see §25.5 runbook pointers) rather than a feature freeze. Feature freeze logic is reserved for infrastructure SLOs where system correctness is at risk. Product SLO responses are growth team actions: funnel analysis, user interviews, A/B tests.

---

### 25.5 Alerting Rules (AL-ACT-XX)

Product alerts route through PostHog alert conditions and PagerDuty. P1 alerts page growth-lead during business hours (not 24/7 — product metrics do not constitute user-facing outages unless combined with AL-ACT-06). P2 alerts post to Slack `#product-alerts`. AL-ACT-06 is the exception: it is a P0 alert because zero events in a 6-hour window during peak hours indicates a client-side crash or CDN failure, not a product metric regression.

| Alert ID | Condition | Severity | Response SLA | Runbook / Response |
|---|---|---|---|---|
| AL-ACT-01 | ACT-SLO-01 D7 Activation Rate drops below 35% over a 48h rolling window (minimum 50 users in window) | P1 | 4h (business hours) | growth-lead opens PostHog F1→F4 funnel, identifies step with highest drop-off delta vs prior 7-day baseline; escalates to platform-engineer if F4 (`session.completed_with_cv`) drop-off > 50% (CV pipeline regression — cross-reference §18 AL-CV alerts) |
| AL-ACT-02 | F1 Onboarding completion rate drops below 60% in any 24h window with ≥ 20 new installs in that window | P1 | 2h (business hours) | ux-flow reviews `onboarding.step_viewed` funnel in PostHog to identify exit step; checks for app version regression in Sentry (§7.7 Mobile Release Health dashboard); if crash-correlated, escalates to P0 |
| AL-ACT-03 | `session.abandoned` rate exceeds 30% of `session.confirmed` events in a 24h rolling window | P2 | 8h | platform-engineer checks CV pipeline inference latency (§18.5 SLOs) and session UI for regressions; cross-reference §18 AL-CV-01 through AL-CV-03 for on-device latency spikes; if CV cold-start P95 > 800 ms, likely root cause |
| AL-ACT-04 | F8 Paywall conversion (`subscription.purchase_started` / `paywall.shown`) drops below 10% in a 72h rolling window with ≥ 50 paywall impressions | P2 | Next business day | growth-lead + product-manager review paywall variant performance in PostHog; check for A/B test contamination (§25.6); check RevenueCat purchase completion rate (§24) to confirm drop is at presentation not at payment processing |
| AL-ACT-05 | PostHog event delivery lag P95 exceeds 120 s in any 30-minute window (measured via S-012 synthetic probe in §16.2) | P2 | 4h | devops-lead checks PostHog EU Cloud status page; checks PostHog JS SDK network path from app; verifies Cloudflare is not blocking `eu.posthog.com` egress; checks S-012 probe history in Better Stack |
| AL-ACT-06 | Zero `app.opened` events received by PostHog in any 6-hour window during 08:00–22:00 UTC | P0 | 30 min | devops-lead: this alert indicates app is not launching or PostHog ingest is completely broken — NOT a product metric regression; cross-reference §6 AL-002 (Full platform unavailable) and S-001 (API Health probe); if API and mobile crash rate are normal but PostHog events are zero, the PostHog SDK initialization or the `eu.posthog.com` route is broken; immediately check Sentry for React Native init errors |

---

### 25.6 A/B Test Instrumentation

FORM uses PostHog Feature Flags for all A/B experiments. The following rules are mandatory for any experiment that affects a user-facing flow.

**Experiment definition requirements (before launch):**

Every experiment must document, in the feature flag description in PostHog and in the relevant Linear ticket:
1. **Hypothesis** — the causal mechanism being tested.
2. **Primary metric** — must map directly to an ACT-SLO (e.g., "improve ACT-SLO-02 F1 completion rate").
3. **Sample size calculation** — minimum detectable effect (MDE) ≥ 5%, significance threshold p < 0.05, power ≥ 80%.
4. **Minimum duration** — at least 14 calendar days to control for day-of-week effects. Experiments shorter than 14 days are not considered statistically valid.
5. **Rollback condition** — explicit metric regression threshold that triggers immediate flag kill (e.g., "if treatment arm session abandonment rate exceeds 50%, kill flag immediately regardless of significance").

**Required events:**

| Event Name | When | Properties | PII Constraint |
|---|---|---|---|
| `experiment.enrolled` | User is assigned to an experiment variant (first flag evaluation) | `experiment_id`, `variant` (control\|treatment_A\|...), `user_id_hash` | `user_id_hash` is SHA-256; plaintext `user_id` is NOT a property here |
| `experiment.converted` | User triggers the experiment's primary conversion event | `experiment_id`, `variant`, `conversion_event_name` | Same constraint |

**Privacy constraints on A/B tests:**
- No health data may be used as an experiment segmentation variable. This is a clinical-safety hard veto. Segmenting by "users who have logged weight data" or "users with high readiness scores" is prohibited.
- Opaque identifiers (e.g., `program_id`, `user_id_hash`) are permitted as segmentation dimensions.
- Experiment results are reported as aggregate conversion rates only. Individual user variant assignments are not exported or shared outside PostHog.

**Statistical rules:**
- Significance threshold: p < 0.05.
- Minimum detectable effect: ≥ 5% relative change.
- Do not call a winner before both minimum sample size and minimum duration are met. Peeking before both conditions are satisfied inflates false positive rates.
- Conflicting experiments (two experiments modifying the same UI component simultaneously) require explicit approval from growth-lead before launch.

---

### 25.7 Privacy Constraints

The following constraints are enforced by design. They are hard requirements, not guidelines. Any proposed change requires a privacy impact assessment reviewed by compliance-officer and documented in `docs/PRIVACY_IMPACT.md`. The clinical-safety team holds a hard veto on any constraint relaxation for items 1, 2, and 3.

1. **No raw health metrics in PostHog.** Weight, load, body composition, RPE, and body measurement values are never sent as event properties. The boolean `checkin_completed` is permitted as a check-in completion signal; the numerical `readiness_score` (1–5) and `mood_score` (1–5) are excluded pending the OQ-33 ruling by clinical-safety. Until that ruling, treat both as prohibited.

2. **No free-text content of any kind.** Victor conversation turns (both user messages and AI responses), custom workout notes, and any other free-text user input must not appear as PostHog event properties. Length-bucket proxies (`message_length_bucket`: short|medium|long) are permitted in place of character counts or content.

3. **No CV keypoints or pose data.** Individual frame keypoint coordinates, form scores (numerical), or per-rep quality assessments are never sent to PostHog. The boolean `cv_enabled` property on `session.completed_with_cv` is permitted.

4. **User identification via `user_id_hash` only.** The PostHog `distinct_id` for every event is `sha256(plaintext_user_id)`. The plaintext Supabase `user_id` UUID is never transmitted to PostHog as `distinct_id`, as a property, or in any other field. The hash is computed in the mobile app before the PostHog SDK call, not by PostHog itself.

5. **PostHog is a designated sub-processor under FORM's GDPR framework.** The Data Processing Agreement with PostHog (EU Cloud, Frankfurt) must be executed before any production event is sent. Sub-processor status is recorded in `docs/SUBPROCESSORS.md`. See §25.8 checklist item for the compliance-officer action.

6. **DSAR fulfilment.** PostHog supports person deletion via its Admin API (`DELETE /api/person/:id/`) and the `$delete` capture event. The DSAR erasure procedure for PostHog is: identify the PostHog person record by `distinct_id` (which equals `sha256(user_id)`) and call the deletion endpoint. This procedure must be documented in `docs/INCIDENT_RESPONSE.md` as runbook R-14 (DSAR runbook). Note: SHA-256 is a one-way function; FORM cannot look up `distinct_id` from `user_id` without recomputing the hash at the time of the DSAR request.

7. **PostHog Session Recording is disabled for all health-adjacent screens.** The PostHog Session Recording feature must be explicitly disabled on the following screens via `posthog.optOutSessionRecording()` or equivalent before the screen mounts: daily check-in screen, active session screen (live CV), post-session summary with rep data, and progress charts. These screens may render health-adjacent metrics in their UI state even if those metrics are not sent as PostHog events. Recording any of these screens would constitute Art. 9 data capture. Session Recording is permitted on marketing, onboarding, and paywall screens only.

---

### 25.8 Implementation Checklist

| Task | Priority | Milestone | Owner |
|---|---|---|---|
| Implement all §25.3 events in iOS app (PostHog React Native SDK calls with correct properties) | P0 | M4 | platform-engineer |
| Implement all §25.3 events in Android app | P0 | M4 | platform-engineer |
| Execute PostHog Data Processing Agreement before any production event is sent | P0 | Before M4 deploy | compliance-officer |
| Configure `user_id_hash` (SHA-256 of plaintext `user_id`) as PostHog `distinct_id` — verify plaintext UUID is never transmitted | P0 | M4 | security-engineer |
| Disable PostHog Session Recording on check-in, live session, post-session summary, and progress chart screens | P0 | M4 | platform-engineer |
| Add PostHog to `docs/SUBPROCESSORS.md` with DPA reference date, data categories, and EU hosting confirmation | P1 | M4 | compliance-officer |
| Verify zero health metric leakage in QA: automated test that inspects the PostHog event queue in a test session and asserts no prohibited properties are present | P0 | Before M4 deploy | qa-lead |
| Set up PostHog "Activation Funnel" visualisation matching the F1 → F2 → F3 → F4 gate sequence | P1 | M5 | growth-lead |
| Configure AL-ACT-01 through AL-ACT-06 alert conditions in PostHog Alerts + PagerDuty routing rules | P1 | M5 | devops-lead |
| Baseline ACT-SLO-01 through ACT-SLO-06 after first 500 real installs and update [ESTIMATE] targets with measured values | P1 | M5 | growth-lead |
| Document PostHog DSAR erasure procedure in `docs/INCIDENT_RESPONSE.md` as runbook R-14 | P1 | M5 | compliance-officer |
| Add `experiment.enrolled` and `experiment.converted` instrumentation to feature flag evaluation path | P1 | M5 | platform-engineer |
| Configure S-012 PostHog Ingest Health synthetic probe latency alerting in Better Stack to feed AL-ACT-05 | P1 | M5 | devops-lead |
| Add ACT-SLO-01 through ACT-SLO-06 to the §7.2 Product Engagement dashboard in PostHog | P2 | M5 | growth-lead |

---

### 25.9 Open Questions

| OQ | Question | Owner | Resolution Target | Priority |
|---|---|---|---|---|
| OQ-31 | D7 Activation Rate baseline — 40% is an [ESTIMATE] derived from comparable fitness app benchmarks (Strava, Fitbod cohort data, published mobile fitness retention studies). FORM's CV-gated activation definition is more demanding than most comparators; the true baseline may be lower. | growth-lead | First 500 installs; recalibrate ACT-SLO-01 target and AL-ACT-01 threshold based on observed distribution | P1 |
| OQ-32 | PostHog vs Amplitude — PostHog was selected for open-source option, EU data residency, and combined feature flag + analytics surface in a single contract. If enterprise customers require data portability guarantees for product analytics data (e.g., exporting their own users' funnel data to their own BI tool), Amplitude has a stronger export and data governance story. This is not a current blocker but may become one at enterprise GA. | data-engineer | Before enterprise GA; revisit if a prospective enterprise customer explicitly requires analytics data portability as a contract term | P2 |
| OQ-33 | Readiness score and mood score in PostHog — the numerical `readiness_score` (1–5) and `mood_score` (1–5) captured in the daily check-in flow are excluded from PostHog pending clinical-safety review. If ruled as non-health data (i.e., subjective self-report that does not constitute a clinical measure), they could be included as bucketed properties (`readiness_bucket`: low\|medium\|high) to improve check-in drop-off analysis. Clinical-safety has not yet issued a ruling. Until ruled, treat as prohibited per §25.7 constraint 1. | clinical-safety | Before M4 deploy; this must be resolved before the check-in event schema is finalised | P0 |

---

*v1.2 additions: §25 Product & Activation Funnel Observability — closes the gap between infrastructure observability (§1–§24) and product health by defining FORM's PostHog-backed product observability layer; §25.1 scopes the section and cross-references PostHog from §1 Three Pillars and §12 Developer Instrumentation Guide; §25.2 defines the "activated" user criterion (one full CV session within 7 days of registration) and maps F1–F8 flows (`docs/FLOWS.md`) to entry events, completion events, activation gate status, and max acceptable drop-off targets; §25.3 specifies the full PostHog event taxonomy (21 events), including explicit prohibitions on mood/readiness scores pending OQ-33, weight/load data, CV keypoints, free-text content, and plaintext user_id — each event row documents PII risk and PostHog retention decision; §25.4 defines six product SLOs (ACT-SLO-01 through ACT-SLO-06) covering D7 activation rate ≥ 40% [ESTIMATE], F1 onboarding completion ≥ 70% [ESTIMATE], F3→F4 session start completion ≥ 80% [ESTIMATE], D30 retention ≥ 35% [ESTIMATE], F8 paywall conversion ≥ 15% [ESTIMATE], and PostHog event delivery lag P95 < 60 s, with measurement windows, alerting thresholds, and owners; §25.5 specifies six alerting rules (AL-ACT-01 through AL-ACT-06) with severities P0–P2, response SLAs, and structured runbook descriptions — AL-ACT-06 (zero `app.opened` events in 6h peak window) is P0 as it signals client crash or CDN failure rather than a product metric regression; §25.6 specifies A/B test instrumentation rules (experiment definition requirements, `experiment.enrolled` / `experiment.converted` event schemas, clinical-safety veto on health-data segmentation, p < 0.05 significance threshold, 14-day minimum duration, MDE ≥ 5%); §25.7 lists seven privacy constraints (no health metrics; no free-text; no CV pose data; SHA-256 distinct_id; PostHog DPA as sub-processor; DSAR via PostHog deletion API documented in R-14; Session Recording disabled on health-adjacent screens); §25.8 provides a fourteen-item implementation checklist across M4/M5; §25.9 documents three open questions (OQ-31 D7 activation baseline, OQ-32 PostHog vs Amplitude enterprise portability, OQ-33 clinical-safety ruling on readiness/mood scores in PostHog — P0 resolution required before M4 deploy).*
