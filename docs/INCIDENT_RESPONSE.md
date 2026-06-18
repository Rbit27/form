# FORM · Incident Response Runbook v2.7

> Owner: security-engineer + compliance-officer. Review: after every P0/P1 incident, minimum annual. SOC 2 evidence: CC7.2–CC7.5, CC9.2, P4.0, P5.0, P8.0.

---

## Purpose and Scope

This runbook is the authoritative operational guide for detecting, responding to, containing, and recovering from security and availability incidents at FORM. It covers the backend platform (Cloudflare Workers, Supabase Postgres), the mobile applications (iOS/Android), and all third-party processors in scope (Anthropic, ElevenLabs, PostHog, Sentry).

This document is formal evidence for SOC 2 Type II criteria CC7.2 (monitoring for and evaluating system events), CC7.3 (response to identified anomalies), CC7.4 (response to identified security incidents), CC7.5 (post-incident review), and CC9.2 (risk mitigation through vendor management and business continuity).

**GDPR note:** FORM processes health data classified as GDPR Art. 9 special category data. Any incident involving potential exposure of this data triggers a 72-hour supervisory authority notification window under Art. 33. Responders must track elapsed time from first detection, not from confirmation of breach scope.

---

## 1. Incident Classification

### 1.1 Severity Matrix

| Severity | Name | User Impact | Data Exposure | Availability | Regulatory Trigger | Target Response |
|---|---|---|---|---|---|---|
| **P0** | Critical / SEV1 | Any user, confirmed harm or high-probability harm | Any PII or health data confirmed or suspected exposed | Full platform or critical path down | Yes — GDPR Art. 33 / Art. 34 likely | Acknowledge < 15 min; IC on call immediately |
| **P1** | High / SEV2 | Significant user cohort impacted; single-tenant admin blocked | No confirmed exposure, but plausible access path exists | Partial outage; enterprise feature unavailable | Possible — monitor actively | Acknowledge < 30 min; IC notified |
| **P2** | Medium / SEV3 | Degraded experience; non-critical path | No data exposure path identified | Degraded performance; SLA at risk | No | Acknowledge < 2 hours; handled in business hours |
| **P3** | Low / SEV4 | Cosmetic, edge-case, or single user | None | None | No | Tracked in Linear; no paging |

### 1.2 Classification Criteria Detail

**P0 — Declare when ANY of the following are true:**
- Confirmed or strongly suspected unauthorized access to health data, PII, or tenant data
- Active exfiltration or evidence of data leaving FORM-controlled systems
- HMAC audit log chain break detected (see R-05)
- Authentication bypass enabling cross-tenant access
- Compromised signing keys, Supabase service role key, or Cloudflare API tokens
- Full platform unavailability exceeding 10 minutes
- Active attack vector that has not yet been closed (zero-day, open session, live attacker)
- Ransomware or destructive payload executed on any FORM infrastructure

**P1 — Declare when ANY of the following are true:**
- Single-tenant SSO/SAML outage affecting enterprise customer login
- Elevated authentication failure rate suggesting credential stuffing or account takeover attempt
- Supabase RLS misconfiguration discovered (no confirmed cross-tenant access yet)
- API key confirmed compromised but no confirmed access to data
- Payment validation endpoint returning errors for >5 minutes
- Partial availability loss affecting >20% of users
- Anomalous PostHog event pattern indicating possible data scraping

**P2 — Declare when ANY of the following are true:**
- Sustained elevated error rates (>5% on any endpoint for >15 minutes)
- Slow query degradation causing P95 latency >3s
- Single-user account lockout without clear cause
- Dependency vulnerability (CVSS ≥ 7.0) detected in production code
- Non-critical third-party integration failure (analytics, non-auth integrations)

**P3 — All other operational issues.** Handled in normal sprint cycles.

### 1.3 Severity Upgrade Rules

- **Initial severity can be upgraded at any time** without approval when new information warrants it.
- **Severity can only be downgraded with explicit Incident Commander (IC) approval**, documented in the incident channel with reasoning. Downgrade does not retroactively change evidence collection requirements — all evidence gathered at the higher severity level is retained.
- If you are uncertain between two severity levels, **always classify higher** and downgrade once the situation is clearer.
- A P1 that reveals evidence of data exposure **auto-promotes to P0 immediately**. Do not wait to confirm scope before upgrading.

### 1.4 Classification Examples

| Scenario | Severity | Rationale |
|---|---|---|
| Confirmed health data rows returned to wrong tenant | P0 | Cross-tenant data breach; GDPR Art. 9 special category exposed |
| HMAC chain break detected in audit log | P0 | Presumed tampering until proven otherwise |
| Supabase service role key found in git history | P0 | Credentials compromised; all data at risk |
| Camera frames found uploading to server (bug) | P0 | Biometric data exposure; GDPR Art. 9 |
| Single enterprise tenant SSO login failing | P1 | Enterprise SLA impact; no data exposure confirmed |
| Credential stuffing attack detected by WAF | P1 | Active account takeover attempt; no confirmed breach |
| RLS policy gap found in code review before deploy | P1 | Potential cross-tenant risk; not yet deployed |
| API P95 latency at 4s for 20 minutes | P2 | Degraded experience; SLA risk |
| PostHog analytics down | P2 | No user-facing impact |
| Button label incorrect on settings screen | P3 | Cosmetic; no impact |

---

## 2. Roles and Responsibilities

### 2.1 Role Definitions

**Incident Commander (IC)**
- Single decision-maker for the duration of the incident. All go/no-go decisions flow through the IC.
- Owns the incident channel, declares severity, authorizes communications, and approves containment actions.
- Does not do hands-on technical work during the incident — coordinates others.
- Responsible for regulatory clock management (GDPR 72-hour window tracking).
- Produces or delegates the Post-Incident Review.

**Security Lead**
- First responder for all security-classified incidents.
- Responsible for evidence preservation (read-only copies, no modification of production logs).
- Leads threat containment: session revocation, key rotation, RLS verification, endpoint disabling.
- Maintains chain of custody for forensic artifacts.
- Interfaces with external counsel if PII breach is confirmed.

**Technical Lead**
- Root cause diagnosis. Runs queries, reviews logs, identifies the blast radius.
- Implements containment patches and fixes.
- Coordinates rollbacks and deploys.
- Owns the technical post-mortem section.

**Communications Lead**
- Drafts and publishes all external communications: status page, customer emails, GDPR notifications.
- Manages the internal Slack incident channel narrative.
- Does not speculate publicly — all external statements reviewed by IC before posting.
- Maintains the public-facing communication log.

**Customer Lead** (activates for P0/P1 with confirmed enterprise tenant impact)
- Directly contacts affected enterprise tenant admins via phone or email — not just status page.
- Coordinates with customer-success on account relationship management post-incident.

### 2.2 On-Call Rotation

**Phase 0 — Solo Founder (current state):**
Founder acts as IC, Technical Lead, and Communications Lead. All alerts route to founder's mobile device. No formal rotation.

**Phase 1 — Team of 2–3:**
Founder is IC for all P0. Founding engineer as primary on-call Technical Lead. Week-by-week rotation. Compensated $200/week on-call stipend. PagerDuty implemented for routing.

**Phase 2 — Team of 4+ (target state this runbook documents):**
- Formal PagerDuty rotation: primary on-call + secondary (backup)
- Primary on-call = Technical Lead for any incident during their shift
- IC role rotates weekly among senior engineers + founder; founder is always IC backup
- Security Lead = security-engineer (dedicated role) — always paged for P0 regardless of rotation
- Communications Lead = content-strategist or devops-lead (designated per rotation)
- No engineer on primary on-call for more than 1 week in 3

**Rotation schedule documented in:** PagerDuty schedule (source of truth). This runbook documents the structure; PagerDuty holds the live roster.

### 2.2.1 Phase 0 Permanent On-Call: Compensating Control Statement

**SOC 2 criterion:** CC2.2 — External communication of commitments and responsibilities.
**Gap reference:** CC2-GAP-005 (`docs/SOC2_READINESS.md §30.3`).
**Owner:** security-engineer. **Effective:** 2026-06-09. **Next review:** within 30 days of second engineering hire joining.

#### Acknowledgement

During Phase 0 (solo founder), FORM does not operate an on-call rotation. The founder is permanently on-call 24/7. FORM acknowledges this to auditors as **CC2-GAP-005**, accepted at P1 risk, with the compensating controls below in lieu of a rotation.

No enterprise customer contract will be signed until Phase 1 is reached, or until the customer has accepted the Phase 0 on-call arrangement in writing as a documented, recorded risk. This gate is enforced in `docs/ENTERPRISE.md §Pre-conditions for enterprise GA`.

#### Compensating Controls

| Control | Mechanism | Evidence artefact |
|---|---|---|
| Permanent coverage | Founder carries PagerDuty mobile 24/7 with critical alerts enabled, overriding silent/DND mode (iOS Critical Alerts entitlement active) | PagerDuty notification policy export — CC2-E-004 |
| Multi-escalation on silence | PagerDuty escalation policy: T+3 min phone call, T+8 min SMS + second call, T+15 min emergency contact page (registered out-of-band contact in PagerDuty off-hours escalation policy) | PagerDuty escalation policy screenshot — CC2-E-004 |
| Independent uptime failsafe | Better Uptime synthetic probes run every 30 s from three regions. Three consecutive failures (90 s) trigger an emergency SMS + email to a secondary address, independent of PagerDuty | Better Uptime alert config export — CC2-E-005 |
| Audit chain dead-man's switch | HMAC chain integrity pg_cron runs every 10 min. A chain break fires a PagerDuty P0 using the escalation policy above. No acknowledgement is required to advance to the next escalation tier; the timer starts at alert creation, not at acknowledgement. | `audit_log` pg_cron job schedule — CC2-E-004 |

#### PagerDuty Escalation Path (Phase 0)

```
T+0  min  Alert fires
          ↳ PagerDuty high-urgency push notification + SMS → founder mobile

T+3  min  No acknowledgement
          ↳ PagerDuty phone call → founder mobile (voice read-aloud of alert title + service)

T+8  min  No acknowledgement
          ↳ PagerDuty SMS retry + second phone call → founder mobile

T+15 min  No acknowledgement — escalation tier 2
          ↳ PagerDuty off-hours policy → phone call to registered emergency contact
          ↳ Slack #security-alerts auto-post (persists regardless of PagerDuty acknowledgement state)
          ↳ Better Uptime failsafe email → secondary address (independent delivery path)
```

No alert is dropped silently. At T+15, three independent notification channels have fired (PagerDuty, Slack, Better Uptime email) and an out-of-band emergency contact has been paged. This is the maximum automation available for a single-person organisation.

#### Acceptance Statement

This arrangement is accepted at **P1 risk**. The gap cannot be closed until a second engineering hire joins and a real rotation is in place. Expiry trigger: second engineering hire joins → Phase 1 rotation schedule published in PagerDuty within 30 days → CC2-GAP-005 status updated to 🟢 Closed in `docs/SOC2_READINESS.md §30.3`.

#### Phase 1 Trigger Checklist

When the second engineering hire joins, complete the following within 30 days:

```
[ ] Publish Phase 1 PagerDuty rotation schedule (founding engineer + founder alternating weekly)
[ ] Update § 2.2 Phase 0 note with a link to the live PagerDuty schedule
[ ] Archive this section as §2.2.2 Historical: Phase 0 Compensating Control (closed YYYY-MM-DD)
[ ] Notify compliance-officer to update CC2-GAP-005 in SOC2_READINESS.md §30.3: 🟡 Partial → 🟢 Closed
[ ] File PagerDuty Phase 1 schedule export as CC2-E-006
```

#### Phase 1 Rotation Schedule Placeholder

```
Primary on-call:  [Founding engineer — to be named]    Weeks starting: [YYYY-MM-DD]
Secondary backup: [Founder]                            All weeks
On-call stipend:  $200/week for primary on-call slot
PagerDuty:        https://form-coach.pagerduty.com/schedules/[SCHEDULE_ID]

Handoff cadence:  Monday 09:00 Kyiv time
P0 rule:          Founder is always secondary for P0 regardless of rotation week
Security page:    security-engineer paged simultaneously with primary on-call for all P0
```

---

### 2.3 Extended Incident Handoff Protocol (>4 hours)

When an incident extends beyond 4 hours, formal handoff is required to prevent responder fatigue and knowledge gaps.

**Handoff checklist** (outgoing IC completes before stepping off):

```
[ ] Incident channel pinned message updated with current state
[ ] Current hypothesis of root cause documented in channel
[ ] Outstanding action items listed with owners
[ ] Regulatory clock status noted (e.g., "GDPR T+2h45m of 72h elapsed")
[ ] All active sessions, queries, or scripts documented
[ ] External parties currently engaged listed (legal, customers, DPA)
[ ] Incoming IC has read and acknowledged the pinned summary
[ ] Verbal (or video) briefing completed — text alone is insufficient for P0
```

Handoff requires **both outgoing and incoming IC to explicitly acknowledge** in the incident channel with a timestamp. No silent handoffs.

---

## 3. Detection and Alerting

### 3.1 Alert Sources

| Source | What It Detects | Alert Channel |
|---|---|---|
| **Cloudflare WAF** | SQLi, XSS, rate limit violations, geo-anomalies, WAF rule triggers | PagerDuty → on-call |
| **Cloudflare Analytics** | Error rate spikes, traffic volume anomalies, latency P99 thresholds | PagerDuty → on-call |
| **Supabase dashboard / logs** | DB connection pool exhaustion, replication lag, slow queries, auth failures | PagerDuty → on-call |
| **Supabase Auth audit** | Elevated login failures, impossible travel, token anomalies | PagerDuty → security-engineer |
| **HMAC chain integrity cron** | Chain break in `audit_log.hmac_self` / `hmac_prev` sequence | PagerDuty → security-engineer (P0 auto-classify) |
| **PostHog anomaly detection** | Unusual event sequences, export spikes, bulk data reads | Slack #security-alerts |
| **Uptime monitor (Better Uptime)** | Endpoint availability, SSL certificate expiry, DNS resolution | PagerDuty → on-call |
| **Sentry** | Crash rate spikes, new error types, performance regressions | Slack #eng-alerts + PagerDuty if P0 threshold |
| **GitHub secret scanning** | Committed secrets, exposed credentials in PRs | Slack #security-alerts + email to security-engineer |
| **Dependabot / npm audit** | Known CVE in dependencies (CVSS ≥ 7.0 → immediate alert) | Slack #security-alerts |
| **User reports** | Via support@form.coach or in-app feedback | Support queue → on-call if P0/P1 keywords |

### 3.2 Escalation Matrix

```
Alert fires
    │
    ▼ T+0
PagerDuty notifies Primary On-Call (push notification + SMS)
    │
    ▼ T+5 min (no acknowledgement)
PagerDuty escalates to Secondary On-Call
    │
    ▼ T+15 min (no acknowledgement)
PagerDuty escalates to Incident Commander (IC)
    │
    ▼ T+30 min (no acknowledgement OR P0 confirmed by any responder)
IC notifies Founder directly (phone call, not Slack)
    │
    ▼ P0 confirmed with regulatory impact
Founder notifies Board / Investors (within 24h of P0 confirmation)
```

**Hard rules:**
- P0 alerts **always** page security-engineer simultaneously with on-call, regardless of who is primary.
- A HMAC chain break alert pages IC and security-engineer simultaneously — it does not go through the standard escalation ladder.
- If on-call does not acknowledge within 5 minutes, the alert has already escalated. Do not wait 5 minutes to manually escalate — trust the automated system.

### 3.3 PagerDuty Alert Routing Configuration

| Alert Type | Routing | Urgency |
|---|---|---|
| HMAC chain break | security-engineer + IC simultaneously | Critical (immediate phone call) |
| Auth failure spike (>50 failures/min for 2 min) | security-engineer | High |
| Full platform outage | on-call + IC | Critical |
| Error rate >10% for 5 min | on-call | High |
| Error rate >5% for 15 min | on-call | High |
| Dependency CVE CVSS ≥ 9.0 | security-engineer | High |
| Dependency CVE CVSS 7.0–8.9 | security-engineer | Low (business hours) |
| SSL cert expiry <14 days | devops-lead | Low |

### 3.4 False Positive Handling

False positives must be documented — not silently dismissed — to prevent log gaps that auditors interpret as missing evidence.

**When standing down a false positive:**
1. Acknowledge the alert in PagerDuty (do not let it auto-resolve without human acknowledgement).
2. Post in `#security-alerts` with format: `[STAND-DOWN] [alert-name] [timestamp] — Confirmed false positive. Reason: [explanation]. No incident declared. Verified by: [name].`
3. If the false positive is recurring (same alert type >2 times in 30 days), open a Linear ticket to tune the alert threshold. Do not suppress the alert — tune it.
4. Do not mark a security alert as false positive without at least one of: log verification, code review, or a second engineer agreeing.

---

## 4. Incident Response Lifecycle

### 4.1 Phase 1 — Detect and Triage (T+0 to T+15 min)

**Goal:** Classify the incident, declare severity, open the incident channel, assign roles.

```
T+0    Alert fires or report received
T+0    On-call acknowledges in PagerDuty
T+2    On-call makes initial severity assessment
T+5    If P0/P1: open incident Slack channel → #inc-YYYYMMDD-[slug]
       (Example: #inc-20260516-rls-bypass)
T+5    Post initial message to incident channel using template (§6.1)
T+5    If P0: page IC and security-engineer immediately — do not wait
T+10   IC assumes command, confirms or adjusts severity
T+10   IC assigns Technical Lead, Security Lead, Comms Lead roles in channel
T+15   First external status page update posted (even if "Investigating")
T+15   GDPR clock noted in channel if data exposure is plausible
```

**Timeline documentation requirement:** Every action must be recorded in the incident channel with a timestamp and the name of the person taking the action. Format: `[HH:MM UTC] [Name] — [Action taken / finding]`. This log is SOC 2 evidence (CC7.3) and forms the foundation of the PIR.

### 4.2 Phase 2 — Contain (T+15 to T+60 min)

**Goal:** Stop the bleeding. Prevent further damage. Preserve evidence before containment actions destroy it.

**Evidence first, containment second** — in that order, unless live data exfiltration is actively occurring. If exfiltration is live, containment takes priority; document what you preserved and what you could not.

Core containment actions by incident type:
- **Data access / breach:** Disable affected RLS policy, revoke API keys, kill active DB sessions for affected user/tenant, block IP at Cloudflare WAF.
- **Account takeover:** Force-revoke all sessions for the compromised account, disable the account if needed, block source IPs.
- **Outage:** Identify failing component, initiate rollback or failover, scale if capacity issue.
- **Key compromise:** Rotate immediately — see R-03 for sequence.
- **HMAC chain break:** Do not write any new audit log entries until chain is repaired. Freeze the chain in place for forensic analysis.

**Containment must be logged** in the incident channel as each action is taken, with timestamp and operator name.

### 4.3 Phase 3 — Eradicate (T+1h to T+4h)

**Goal:** Remove the root cause. Confirm the attack vector is closed. Verify no persistence mechanisms remain.

- Identify root cause (not hypothesis — confirmed via log evidence or code review)
- Patch, revert, or reconfigure as needed
- Verify fix in staging before production if time allows; document if deployed directly to production
- Run RLS test suite if database permissions were involved: `supabase test db --filter=rls`
- Confirm no lateral movement occurred (see R-03 for lateral movement check procedure)
- Rotate any credentials that were or may have been exposed, even if not confirmed compromised

### 4.4 Phase 4 — Recover (T+4h to T+24h)

**Goal:** Restore full service. Validate integrity. Communicate resolution.

- Restore any disabled endpoints or features after confirming fix is in place
- Validate service health across all monitoring sources: Cloudflare Analytics, Supabase dashboard, uptime monitor, Sentry
- Run smoke test suite against production: `npm run test:smoke:prod`
- If HMAC chain was broken: complete chain repair procedure (R-05) and verify chain integrity before declaring recovery
- Post resolution update to status page
- Send customer-facing resolution communication (see §6.5)
- Confirm GDPR notification obligations with compliance-officer before closing the incident

**Do not close a P0 incident without IC sign-off.**

### 4.5 Phase 5 — Post-Incident Review (T+24h to T+72h)

**Goal:** Learn. Prevent recurrence. Produce SOC 2 evidence that the review occurred.

- Draft PIR within 24 hours of incident close (IC or delegate owns this)
- PIR published internally within 72 hours
- For P0: board notification within 24 hours of incident resolution
- See §8 for full PIR structure and requirements

---

## 5. Runbooks

Each runbook follows the structure: **Trigger → Immediate Actions → Containment → Evidence Preservation → Recovery → Communications.**

---

### R-01: Data Breach / Unauthorized Data Access

**Trigger:** Confirmed or strongly suspected unauthorized access to user data, health data, PII, or cross-tenant data access. Includes: Supabase RLS bypass, API returning wrong tenant's data, admin dashboard exposing cross-tenant records, third-party processor reporting unauthorized access.

**Severity:** P0 by default. Do not downgrade without IC + compliance-officer agreement.

#### Immediate Actions (T+0 to T+15 min)

```
1. Open incident channel: #inc-YYYYMMDD-data-breach
2. Page: IC + security-engineer + compliance-officer simultaneously
3. Note exact time of first detection in channel — GDPR 72-hour clock starts now
4. Do NOT alert affected tenants yet — premature notification without scope assessment
   creates legal risk and panic. IC authorizes all external comms.
5. Identify the vector: RLS bypass? API bug? Compromised credential? Insider?
6. Screenshot or export any live dashboard showing the anomaly BEFORE taking any
   containment action that would change system state
```

#### Containment

**If cause is a running query / active session:**
```sql
-- List active connections
SELECT pid, usename, application_name, client_addr, state, query
FROM pg_stat_activity
WHERE state != 'idle';

-- Terminate specific session
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = <target_pid>;
```

**If cause is an RLS policy gap:**
```sql
-- Check RLS enabled on the affected table
SELECT schemaname, tablename, rowsecurity
FROM pg_tables
WHERE tablename = '<affected_table>';

-- List current RLS policies
SELECT policyname, cmd, qual, with_check
FROM pg_policies
WHERE tablename = '<affected_table>';
```

Disable the affected API endpoint at Cloudflare Worker level immediately — add a temporary return-403 block — rather than attempting a live DB policy patch under pressure.

**If cause is a compromised credential:**
See R-03 for full credential rotation sequence.

**If cause is a code bug (API returning wrong data):**
Roll back the Worker deployment to last known good:
```bash
wrangler rollback --env production
```
Then verify rollback took effect before re-enabling any disabled endpoints.

#### Evidence Preservation

Preserve before any containment action modifies system state. If containment is urgent (live exfiltration), document what could not be preserved.

```
1. Export affected Supabase table query logs:
   supabase db logs --since 2h > /secure-evidence/db-logs-$(date -u +%Y%m%dT%H%M%SZ).txt

2. Export Cloudflare Worker logs for the affected endpoint (via dashboard or API):
   Time window: 2 hours before detection through containment

3. Screenshot Cloudflare Analytics dashboard showing traffic anomaly

4. Export audit_log entries for the affected tenant and time window:
   SELECT * FROM audit_log
   WHERE tenant_id = '<affected_tenant_id>'
   AND created_at >= now() - interval '4 hours'
   ORDER BY created_at ASC;
   -- Save output as read-only copy; do NOT modify audit_log

5. Export audit_log HMAC chain entries for integrity verification:
   SELECT id, hmac_prev, hmac_self, created_at FROM audit_log
   WHERE tenant_id = '<affected_tenant_id>'
   ORDER BY created_at ASC;

6. Hash all exported files with SHA-256 immediately after export:
   sha256sum /secure-evidence/* > /secure-evidence/MANIFEST.sha256

7. Upload to encrypted incident evidence store (see §7) with chain of custody entry
```

**HMAC chain integrity during a data breach incident:**
- The audit log is the primary forensic record. It must not be modified.
- All audit log reads must be read-only. Do not run any operation that could write to `audit_log` during evidence collection.
- If containment actions will generate new audit log entries (e.g., key rotation, session revocation), this is expected and correct — those entries extend the chain normally. Document their expected presence.
- Do not attempt to repair a chain break discovered during a data breach investigation until the Security Lead and IC both authorize it — the break itself is evidence.

#### Scope Assessment

IC leads scope assessment. Answer these questions before any external notification:

```
- Which tenant(s) are affected?
- Which users within those tenants?
- Which data categories: health data (Art. 9)? PII? Workout logs? Chat history?
- How many rows / records?
- Was data accessed (read) or exfiltrated (copied to external system)?
- What is the access window? (First possible unauthorized access → containment)
- Is the vector closed? Confirmed?
```

#### Who to Notify and When

| Recipient | Trigger | Timing | Channel |
|---|---|---|---|
| IC | Always | T+0 | PagerDuty |
| security-engineer | Always | T+0 | PagerDuty |
| compliance-officer | Always | T+5 min | Slack DM + phone |
| External legal counsel | PII or health data confirmed exposed | T+2h | Phone |
| Affected enterprise tenant admins | IC authorizes, scope known | T+24h or earlier if required | Direct email from IC |
| Supervisory Authority (DPA) | Any personal data exposed | T+72h from detection (hard deadline) | See template §6.3 |
| Affected data subjects | High risk to individuals, or DPA directs | Per DPA guidance | See template §6.4 |
| Board / investors | P0 confirmed | T+24h from resolution | Founder notifies directly |

**GDPR Art. 33 hard deadline: 72 hours from first detection, not from scope confirmation.** If scope is not fully known at 72 hours, file a partial notification and supplement it. Filing late is an aggravating factor in regulatory enforcement. Filing incomplete-but-timely is expected and accepted.

#### Supabase RLS Verification Steps

After any breach involving data access, run full RLS verification before restoring access:

```bash
# Run the RLS test suite
supabase test db --filter=rls

# Verify RLS is enabled on all sensitive tables
psql $DATABASE_URL -c "
SELECT schemaname, tablename, rowsecurity
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY tablename;"

# For each table where rowsecurity = false, this is a finding — escalate immediately
# All tables containing user data MUST have rowsecurity = true

# Verify tenant isolation for the affected table manually:
# Log in as a test user from tenant A and attempt to read tenant B's data
# This must return zero rows — not an error, zero rows
```

---

### R-02: Service Outage (Full or Partial)

**Trigger:** Uptime monitor alert, Cloudflare error rate threshold exceeded, Supabase reporting connectivity issues, user reports of complete inability to use the service.

**Severity:** Full platform outage ≥ 10 minutes = P0. Partial outage or degraded service = P1 or P2 depending on scope.

#### Triage Checklist

Work through these in order — each layer can mask issues in the layers below it.

**Step 1 — Is Cloudflare itself the issue?**
```
1. Check https://www.cloudflarestatus.com
2. Check Cloudflare Workers dashboard for error rate on your routes
3. If Cloudflare is having an incident: this is a dependency outage, not a FORM failure.
   Update status page immediately, set expectation based on Cloudflare's ETA.
   You cannot fix a Cloudflare outage — focus on communication.
```

**Step 2 — Is Supabase the issue?**
```
1. Check https://status.supabase.com
2. Check Supabase dashboard: DB connections, replication lag, API error rate
3. Run: supabase db logs --since 30m
4. Attempt direct DB connection from a test client to verify connectivity
5. If Supabase is having an incident: same as above — communicate, wait, monitor.
```

**Step 3 — Is DNS the issue?**
```
1. Resolve form.coach from multiple external locations (use https://dnschecker.org)
2. Check Cloudflare DNS dashboard for propagation anomalies
3. Verify certificate validity: openssl s_client -connect api.form.coach:443 -servername api.form.coach
```

**Step 4 — Is it a FORM deployment issue?**
```
1. Check recent deploys: git log --oneline -10 (compare with Cloudflare Workers deploy log)
2. Check Cloudflare Workers error logs for the specific route that is failing
3. Check Sentry for new error type that appeared after the last deploy timestamp
4. If a bad deploy: initiate rollback immediately (see below)
```

**Step 5 — Is it a dependency (Anthropic, ElevenLabs)?**
```
1. Check https://status.anthropic.com and https://status.elevenlabs.io
2. Test a direct API call to the dependency from a non-production context
3. If dependency is down: disable the affected feature via feature flag in PostHog,
   serve a degraded-mode response, update status page
```

#### Rollback Procedure

**Cloudflare Workers rollback:**
```bash
# List recent deployments
wrangler deployments list --env production

# Roll back to a specific version
wrangler rollback <deployment-id> --env production

# Verify rollback is live (Workers propagate in ~30 seconds globally)
curl -I https://api.form.coach/health
```

**Supabase migration rollback:**
```bash
# Migrations are NOT automatically reversible — this is manual
# If migration caused the outage, assess whether a revert migration is safe
# For additive migrations (added column): can usually revert by dropping the column
# For destructive migrations: assess data loss before reverting

# Create a revert migration:
supabase migration new revert_<original_migration_name>
# Write the inverse SQL in the new migration file
supabase db push --env production
```

**Mobile app rollback:** React Native / Expo apps cannot be rolled back on-device. Options:
- Push an OTA update via Expo Updates if the bug is in JS bundle (not a native build)
- Submit an expedited App Store hotfix (12–24 hours with expedite request)
- Use a feature flag in PostHog to disable the broken feature client-side
- Communicate via in-app message or push notification if users are stuck

#### Customer Communication SLA by Severity

| Severity | First External Update | Update Frequency | Resolution Comms |
|---|---|---|---|
| P0 (full outage) | T+15 min from detection | Every 30 min | Within 30 min of resolution |
| P1 (partial / enterprise) | T+30 min from detection | Every 60 min | Within 1 hour of resolution |
| P2 (degraded) | T+2h from detection (or next business hour) | Every 2–4h | Within 2h of resolution |

Enterprise customers with active incidents must receive **direct outreach** (email or phone from customer-success), not just status page updates.

---

### R-03: Compromised Credentials / Account Takeover

**Trigger:** Alert on impossible travel or auth anomaly, user reports unauthorized access, credential found in git history or third-party leak database, WAF detecting credential stuffing pattern, internal discovery of credentials in plaintext.

**Severity:** P0 if system credentials (Supabase service role, Cloudflare API token, signing keys). P1 if single-user account takeover with no evidence of data access. Upgrade to P0 if any evidence of lateral movement or data access.

#### Immediate Actions

```
1. Open incident channel: #inc-YYYYMMDD-credential-compromise
2. Page: IC + security-engineer
3. Preserve audit log entries for the affected account/credential BEFORE revoking
   (revocation changes system state; you want the pre-revocation view preserved)
4. Export the affected account's audit_log entries for the past 30 days:
   SELECT * FROM audit_log
   WHERE actor_id = '<affected_user_or_system_id>'
   ORDER BY created_at ASC;
```

#### Force-Revoke All Sessions for Affected User or Tenant

**Single user account takeover:**
```sql
-- Revoke all active sessions for the user in Supabase Auth
-- Via Supabase dashboard: Auth > Users > [user] > Sign out all sessions
-- Or via Admin API:
-- POST https://<project>.supabase.co/auth/v1/admin/users/<user_id>/logout
-- Authorization: Bearer <service_role_key>

-- Then disable the account pending investigation:
UPDATE auth.users SET banned_until = now() + interval '24 hours'
WHERE id = '<affected_user_id>';
```

**Full tenant session revocation (enterprise SSO compromise):**
```sql
-- Revoke all sessions for all users in the affected tenant
-- This forces re-authentication for every user — IC must authorize this
-- Impact: all tenant users are logged out immediately
SELECT auth.uid() FROM auth.users
WHERE raw_user_meta_data->>'tenant_id' = '<affected_tenant_id>';
-- Then revoke each session via Admin API or Supabase dashboard bulk action
```

#### Credential Rotation Sequence

Rotate in this order. Do not rotate all simultaneously — stagger to avoid a lock-out cascade.

```
1. Supabase service role key
   a. Generate new key in Supabase dashboard: Settings > API > Service Role Key (Reveal > Roll)
   b. Update in Cloudflare KV:
      wrangler kv:key put --binding=SECRETS SUPABASE_SERVICE_ROLE_KEY "<new_key>"
   c. Verify Workers can still reach Supabase: curl https://api.form.coach/health
   d. Invalidate old key in Supabase dashboard

2. Cloudflare API tokens (if Cloudflare access is compromised)
   a. Cloudflare dashboard: My Profile > API Tokens > Roll the affected token
   b. Update any CI/CD pipelines using the old token (GitHub Actions secrets)
   c. Verify Workers still deploy correctly with a test deploy to staging

3. Anthropic API key
   a. Rotate in Anthropic console
   b. Update in Cloudflare KV:
      wrangler kv:key put --binding=SECRETS ANTHROPIC_API_KEY "<new_key>"
   c. Test a single API call to verify

4. ElevenLabs API key — same pattern as Anthropic

5. Apple App Store shared secret — rotate in App Store Connect, update in KV

6. HMAC signing key for audit log
   - This requires a special procedure — see R-05 for key rotation handling
   - Coordinate with compliance-officer before rotating; rotation requires chain annotation
```

#### Lateral Movement Check

After credential compromise, check for evidence that the attacker accessed other systems:

```sql
-- Audit log entries for the compromised actor_id across all tenants
SELECT tenant_id, action, resource_type, resource_id, created_at, actor_ip
FROM audit_log
WHERE actor_id = '<compromised_actor_id>'
AND created_at >= '<compromise_window_start>'
ORDER BY created_at ASC;

-- Check for any support.impersonation_started events in the window
SELECT * FROM audit_log
WHERE action = 'support.impersonation_started'
AND created_at >= '<compromise_window_start>';

-- Check for any data export events in the window
SELECT * FROM audit_log
WHERE action IN ('data.export_initiated', 'data.bulk_deletion')
AND created_at >= '<compromise_window_start>';
```

Also check Cloudflare WAF logs for requests made with the compromised credential from unexpected IP ranges or user agents.

---

### R-04: SSO/SAML Compromise or Certificate Compromise

**Trigger:** Enterprise tenant reports users cannot authenticate via SSO, SAML assertion validation failures in logs, IdP security team contacts FORM about a security event, SAML certificate found expired or tampered.

**Severity:** P1 (single tenant SSO down, no data breach) or P0 (evidence of SAML assertion forgery enabling unauthorized access).

#### Emergency: Disable SSO for Affected Tenant, Enable Email Fallback

```
1. Open incident channel: #inc-YYYYMMDD-sso-[tenant-slug]
2. Page: IC + security-engineer + customer-success for affected tenant
3. In the tenant admin settings (Supabase-backed):
   UPDATE tenant_sso_config
   SET enabled = false, fallback_email_auth = true
   WHERE tenant_id = '<affected_tenant_id>';
4. Notify tenant admin directly by phone — they need to know their SSO is down
   and that email-based login is temporarily enabled as a fallback
5. Audit log the manual override:
   INSERT INTO audit_log (action, actor_type, tenant_id, reason, ...)
   VALUES ('tenant.sso_configured', 'system', '<tenant_id>',
           'Emergency SSO disable - incident <inc-id>', ...);
```

**IMPORTANT:** Enabling email fallback for an enterprise tenant that may have email-based SSO disabled for security policy reasons must be authorized by the **tenant's own security contact**, not just FORM's IC. Get written confirmation (email or Slack screenshot) from the tenant's security lead before enabling email login if that conflicts with their security policy.

#### SAML Certificate Rotation

Reference `docs/SSO_SCIM_IMPLEMENTATION.md §8` for the full certificate lifecycle management procedure.

Quick reference:

```
1. Generate new X.509 certificate pair (2048-bit RSA minimum; 4096-bit recommended)
   openssl req -x509 -nodes -days 365 -newkey rsa:4096 \
     -keyout form-saml-new.key -out form-saml-new.crt

2. Upload new certificate to FORM's SAML SP configuration:
   -- Store new cert in encrypted DB column
   UPDATE tenant_sso_config
   SET saml_sp_cert = '<new_cert_pem>', saml_sp_key = '<encrypted_key>'
   WHERE tenant_id = '<affected_tenant_id>';

3. Send new SP certificate metadata to the affected tenant's IdP admin
   (they must update the SP certificate in Okta / Azure AD / Google Workspace)

4. Test SAML flow in staging with the tenant's sandbox IdP before activating in production

5. Once tenant confirms IdP is updated, activate new cert in production

6. Keep old cert valid for 24h overlap period during transition
   (some IdPs cache the SP metadata)

7. Revoke old cert after overlap period confirmed clear
```

#### Notify Affected IdP Security Team

If the compromise is on the IdP side (e.g., Okta reporting a security incident), notify them:
- Okta: security@okta.com or through your Okta admin console incident report
- Azure AD / Microsoft Entra: via Microsoft Security Response Center (MSRC)
- Google Workspace: via Google Workspace Admin console > Support

Provide them: affected tenant identifier, suspected compromise window, nature of the SAML anomaly you observed on your side.

---

### R-05: HMAC Audit Log Chain Break

**A chain break is a P0 incident.** It indicates either a bug in the HMAC computation pipeline or active tampering with the audit log. Treat it as tampering until forensic analysis proves otherwise.

**Trigger:** Weekly HMAC chain integrity cron fails and pages security-engineer. Or: anomaly detected in chain validation during a separate investigation. Or: OFB-REGION-01 chain invariant violation — `enterprise.data_export_completed` event emitted with `is_eu_region: true` but `r2_bucket != 'form-offboarding-exports-eu'` (HTTP 422 + P1 PagerDuty `form-platform`; see `docs/DATA_MODEL.md §36.6`; cross-tenant data residency breach — treat as potential data breach until forensic analysis confirms routing bug).

#### Immediate Actions

```
1. Open incident channel: #inc-YYYYMMDD-chain-break
2. Page: IC + security-engineer + compliance-officer simultaneously
3. NOTE: Do NOT write any new audit log entries until the break is characterized.
   This means pausing any batch operations or maintenance scripts that write to audit_log.
   Normal application writes (user actions) continue — you cannot stop those,
   and their continued logging is itself forensic evidence.
4. Identify the exact row where the chain breaks:
   -- Run the chain verification query (read-only, never modifies data)
   SELECT
     id,
     hmac_prev,
     hmac_self,
     created_at,
     LAG(hmac_self) OVER (ORDER BY created_at ASC) AS expected_prev
   FROM audit_log
   WHERE tenant_id = '<affected_tenant_id>'  -- or all tenants if global
   ORDER BY created_at ASC;
   -- Find the first row where hmac_prev != expected_prev (prior row's hmac_self)
```

#### Evidence Preservation

```
1. Export the full audit_log table for the affected tenant as a read-only copy:
   COPY (SELECT * FROM audit_log WHERE tenant_id = '<tenant_id>'
         ORDER BY created_at ASC)
   TO '/secure-evidence/audit_log_export_$(date -u +%Y%m%dT%H%M%SZ).csv'
   WITH CSV HEADER;

2. Export the audit_log_retention_summary table for chain continuity reference

3. Hash the export immediately:
   sha256sum /secure-evidence/audit_log_export_*.csv > /secure-evidence/MANIFEST.sha256

4. Document the chain break location:
   - Row ID where break occurs
   - Row ID of the last valid entry (immediately before break)
   - Timestamp of break
   - Whether the break is a single entry or a range (multiple entries affected)

5. Preserve the Cloudflare Worker logs for the HMAC computation service
   for the time window around the break — these will show whether
   the break is a computation bug or post-write modification
```

#### Forensic Procedure

**Determining: bug vs tampering**

A **bug** typically produces:
- A single chain break at a predictable location (e.g., after a deployment, after a key rotation)
- The affected row's `hmac_self` is internally consistent (correct computation) but uses wrong `hmac_prev`
- No other audit log anomalies (no deleted rows, no resequenced timestamps)

**Tampering** typically produces:
- Deleted or modified rows (row count mismatch vs expected sequence)
- `created_at` timestamps that are out of order or have suspicious gaps
- Multiple breaks rather than a single break point
- The break coincides with a sensitive action (data export, bulk delete, admin escalation)

```sql
-- Check for timestamp ordering violations
SELECT id, created_at,
  LAG(created_at) OVER (ORDER BY created_at ASC) AS prev_created_at
FROM audit_log
WHERE tenant_id = '<tenant_id>'
ORDER BY created_at ASC;
-- Any row where created_at < prev_created_at is a timestamp ordering violation

-- Check row count vs expected sequence (if your IDs are sequential)
-- Compare total row count with what your application metrics show logged
SELECT COUNT(*) FROM audit_log WHERE tenant_id = '<tenant_id>';

-- Look for gaps in the created_at sequence that are suspiciously large
SELECT
  id,
  created_at,
  created_at - LAG(created_at) OVER (ORDER BY created_at ASC) AS gap
FROM audit_log
WHERE tenant_id = '<tenant_id>'
ORDER BY created_at ASC;
-- Flag gaps > 10 minutes for events that should be continuous
```

**If tampering is confirmed or cannot be ruled out:** This is now simultaneously an R-01 incident. Escalate to P0 if not already classified. Engage external counsel. Notify supervisory authority within 72 hours.

**If bug is confirmed (e.g., key rotation caused the break):**
```
1. Document root cause in incident channel with evidence
2. IC + compliance-officer must both approve before repairing the chain
3. Repair procedure:
   a. Identify the break point row
   b. Recompute hmac_self for the break point row using correct hmac_prev
      (current row's payload + correct previous row's hmac_self)
   c. Update the row — this is the ONE time audit_log rows are modified,
      and it must be done by security-engineer with IC authorization,
      logged as a system action with justification
   d. Run full chain verification after repair to confirm chain is intact
   e. Document the repair in audit_log itself:
      INSERT INTO audit_log (action, actor_type, reason, ...)
      VALUES ('system.config_changed', 'system',
              'HMAC chain repair - incident <inc-id> - approved by IC <name>', ...);
4. Update HMAC computation code to prevent recurrence
5. Run PIR (mandatory for P0)
```

---

### R-06: DDoS / WAF Bypass

**Trigger:** Cloudflare analytics showing anomalous traffic spike, WAF alert on rule trigger volume, uptime monitor detecting elevated latency or timeouts that correlate with traffic spike, Supabase connection pool exhaustion coinciding with traffic anomaly.

**Severity:** P1 if service is degraded but available. P0 if service is unavailable or if the DDoS is masking a concurrent intrusion attempt.

#### Cloudflare WAF Escalation Steps

**Step 1 — Verify it is a DDoS and not a traffic spike from a legitimate event:**
```
1. Check PostHog for in-app user activity — does real user activity correlate?
2. Check Cloudflare Analytics: what is the geographic distribution of traffic?
   Legitimate growth is roughly proportional to your user distribution.
   DDoS typically shows an anomalous concentration from unexpected regions or ASNs.
3. Check the User-Agent distribution — bot traffic often has uniform or obviously
   fake User-Agent strings.
```

**Step 2 — Cloudflare WAF response options (escalating aggressiveness):**
```
Option A — Challenge suspicious traffic (least disruptive):
  Cloudflare dashboard > Security > WAF > Custom Rules
  Add rule: [ASN or country or IP range] → Challenge (Managed Challenge)
  This adds a CAPTCHA-like step for suspect traffic without blocking real users.

Option B — Rate limit specific endpoints under attack:
  Cloudflare dashboard > Security > WAF > Rate Limiting Rules
  Set threshold: e.g., max 100 requests per minute per IP to /api/auth/*
  Action: Block for 10 minutes

Option C — Enable I'm Under Attack Mode (maximum disruption — use only for severe DDoS):
  Cloudflare dashboard > Quick Actions > I'm Under Attack Mode (toggle)
  WARNING: This adds a 5-second challenge to ALL requests including real users.
  This is a last resort. Notify IC before enabling.

Option D — Block specific IP ranges at WAF (precise, if source identified):
  Security > WAF > IP Access Rules
  Action: Block; applies globally to your zone
```

**Step 3 — Escalate to Cloudflare support for sustained attacks:**
```
1. File a Cloudflare support ticket: dash.cloudflare.com > Support > Create Ticket
2. Select: Security — DDoS Attack
3. Provide: traffic charts, time window, affected endpoints
4. Cloudflare Pro/Business/Enterprise plans include DDoS mitigation support SLAs
   If on Free plan: this is an engineering risk to escalate with devops-lead
```

#### Rate Limit Tightening

During an active DDoS or WAF bypass attempt, tighten rate limits at the application layer as well:

```javascript
// In the Cloudflare Worker, check and tighten rate limits:
// Current: 100 req/min per user token; 50 req/min per IP unauthenticated
// Tighten to: 30 req/min per user token; 10 req/min per IP unauthenticated

// This is done via Cloudflare KV or Durable Objects config update,
// not a code deploy — update the rate limit thresholds in KV:
wrangler kv:key put --binding=CONFIG RATE_LIMIT_AUTH_PER_MIN "30"
wrangler kv:key put --binding=CONFIG RATE_LIMIT_UNAUTH_PER_MIN "10"
```

Restore normal limits after the attack subsides and traffic returns to baseline for 30 consecutive minutes.

#### Origin Protection Verification

Verify that direct-to-origin requests (bypassing Cloudflare WAF) are blocked:

```bash
# Find your Cloudflare-assigned origin IP range
# Your Cloudflare Worker URL should not expose the Supabase direct URL
# Verify that Supabase direct DB connection is NOT reachable from the public internet:
nmap -p 5432 <supabase_db_host>  # Should be filtered/closed

# Verify that only Cloudflare IPs can reach your Worker origin
# Cloudflare publishes their IP ranges: https://www.cloudflare.com/ips/
# If you have a backend server (not just Workers): restrict ingress to Cloudflare IPs only
```

If the Supabase connection string or Cloudflare Worker origin is exposed in client code or public repositories, treat as P0 credential compromise — see R-03.

---

### R-07: Sub-processor Breach Notification Received

**Trigger:** A FORM sub-processor — Anthropic, Supabase, ElevenLabs, Cloudflare, PostHog, Sentry, or any other processor with a GDPR Data Processing Agreement — notifies FORM of a security incident that may have involved access to FORM user data. Notification may arrive via: processor status page incident report, email to security@form.coach or legal contact, account manager outreach, or regulatory authority advisory.

**Critical legal distinction from R-01 to R-06:** This incident originates at a sub-processor, not at FORM's own systems. However, FORM remains the **controller** under GDPR Art. 4(7) for all user data processed on its behalf. FORM's Art. 33 notification obligation to the supervisory authority is triggered by FORM's **awareness** of a probable breach involving FORM users' data — regardless of where the breach occurred. **The 72-hour clock starts when FORM receives credible notification from the processor, not when the processor itself discovered the incident.**

**Severity — determined by the data categories the processor holds, not by the processor's own incident classification:**

| Processor | Data categories held for FORM | FORM severity floor |
|---|---|---|
| **Supabase** | Full database: health profiles, workouts, meal logs, coaching turns, auth tokens | P0 — Art. 9 health data confirmed |
| **Anthropic** | Coaching conversation turns (system prompt + user turns contain health context) | P0 — Art. 9 probable |
| **ElevenLabs** | Text coaching cue strings (health-adjacent language possible) | P1 — Art. 9 possible |
| **Cloudflare** | Request logs, IP addresses, auth tokens in-flight, Workers KV (API keys) | P1 minimum; P0 if session token or API key exposure confirmed |
| **PostHog** | Pseudonymous analytics events (no raw PII in standard instrumentation) | P2 minimum; audit instrumentation before downgrading |
| **Sentry** | Error reports — may contain health-adjacent data in stack traces if scrubbing misconfigured | P1 minimum; audit scrubbing config before classification |
| **Stripe** | Payment processing only; FORM holds no raw card data | P1 minimum for billing exposure; no health data |

A Supabase or Anthropic breach notification is **always P0** until scope assessment proves otherwise.

#### Immediate Actions (T+0 to T+30 min)

```
1. Open incident channel: #inc-YYYYMMDD-processor-[vendor-slug]
   Example: #inc-20260520-processor-supabase

2. Page: IC + security-engineer + compliance-officer simultaneously.
   Do not wait for scope assessment — page all three at T+0.

3. Note exact receipt time in channel. This is the GDPR Art. 33 clock start.
   Format: "[HH:MM UTC] GDPR Art. 33 clock started — processor notification
            received from [vendor]. T+0h of 72h."

4. Preserve the notification artifact immediately:
   - Screenshot the processor's status page at time of notification
   - Forward or save any email notification to the incident evidence store (§7)
   - SHA-256 hash the notification artifact and record in channel

5. Do NOT publicly comment on the processor's incident.
   FORM's external communications are independent of the processor's own.
   IC authorizes all external statements.

6. Identify data categories the processor holds for FORM.
   Reference the processor registry in §11.2.
```

#### Scope Assessment

IC leads scope assessment. Answer all of the following before external notification, but do not let assessment delay a partial Art. 33 filing if the 72-hour window is closing.

**Data exposure questions:**
```
- Which FORM data categories does this processor hold? (§11.2)
- What is the processor's stated scope? What systems? What time window?
- Does the processor's incident intersect with FORM user data specifically?
  (Shared infrastructure breach vs. FORM-tenant-specific exposure?)
- Are Art. 9 health data fields in scope?
  Anthropic: were coaching turns with health context transmitted in the window?
  Supabase: were user_health_profiles, coaching_turns, workouts, meal_logs in scope?
- How many FORM data subjects are potentially affected?
- Which enterprise tenants (if any) are affected?
  Enterprise DPAs may impose stricter notification timelines.
```

**FORM-side immediate mitigations:**
```
- Rotate the affected processor's API key immediately (R-03 rotation sequence)
  even if the breach is not confirmed — precautionary rotation is correct.
- If Supabase auth infrastructure is involved:
  consider forcing re-authentication for all active sessions (R-03).
- If Cloudflare is involved:
  run HMAC chain integrity check immediately (R-05).
  Cloudflare handles all API traffic; a breach there could mask a concurrent intrusion.
```

#### GDPR 72-Hour Clock — Sub-processor Context

```
Processor notifies FORM → T+0 — Art. 33 clock starts NOW

T+0–T+12   Scope assessment + immediate mitigations
T+12–T+24  Draft Art. 33 notification (partial if scope not fully known)
T+48       FILE partial Art. 33 notification — leave 24h buffer
           Use template §6.3, adding:
           "This notification is made on the basis of information received
            from FORM's sub-processor [name] on [date/time UTC].
            FORM's investigation is ongoing.
            A supplementary notification will follow as scope is confirmed."
T+48–T+72  Continue scope assessment; supplement notification if new information
T+72       Hard deadline — no extension available

Do NOT wait for the processor to complete their own investigation before filing.
```

**Common error:** Assuming FORM's clock starts from when the processor completes their investigation. It does not. FORM's clock starts from the moment FORM receives credible notification. If the processor says "we think your data may have been affected," that is sufficient to start the clock.

#### Enterprise Tenant Notification

For any enterprise tenant whose data may be in scope:

- Customer Lead (or customer-success) contacts affected tenant admins **by phone** within 4 hours of IC authorization — not via status page alone.
- Check each tenant's DPA for notification timelines that may be stricter than Art. 33's 72-hour default.
- Communicate: processor name, incident nature, which of their employees' data categories may be affected, what FORM has done to contain, what FORM's Art. 33 status is.
- Do not speculate about the processor's own root cause or timeline — FORM communicates what it knows about its users' data, not the processor's internal investigation.

#### Evidence Package (R-07 additions to standard §7 requirements)

```
[ ] Original notification artifact + SHA-256 hash + collection timestamp
[ ] Processor's public incident report (if published) + hash
[ ] Scope assessment document (completed assessment questions above)
[ ] Art. 33 notification filing receipt + any supplementary filings
[ ] Enterprise tenant notification records: timestamps + recipient confirmations
[ ] Credential rotation log if API keys rotated (from R-03 sequence)
[ ] Processor DPA review notes (was Art. 28(3)(f) clause satisfied?)
```

#### Post-Incident: Vendor Management Review

After every processor breach incident, regardless of severity:

```
1. Review processor's DPA notification performance:
   - Was Art. 28(3)(f) notification obligation met?
   - Was "without undue delay" satisfied? (< 48h from processor's discovery)
   - Was sufficient scope detail provided to assess FORM's Art. 33 obligation?

2. If notification was inadequate:
   - Document the gap in the PIR
   - compliance-officer initiates formal written inquiry to the processor
   - Review continued suitability of the processor for Art. 9 data processing
   - For critical processors (Supabase, Anthropic, Cloudflare):
     document the risk acceptance and mitigation plan in DECISION_LOG.md

3. Update the processor registry (§11.2) if scope, data categories,
   or notification contacts have changed.

4. Every processor breach — regardless of severity — is a mandatory
   inclusion in the annual vendor review (SOC 2 CC9.2 control evidence).
   See §11.5.
```

---

### R-08: Supply Chain Attack / Compromised Dependency

**Trigger:** Intelligence — from GitHub Advisory, Dependabot alert, npm security advisory, external CERT, or security researcher disclosure — indicates that a package present in FORM's production dependency tree has been actively backdoored, typosquatted, or supply-chain-injected (distinct from a mere CVE: the package has been modified to execute malicious code, exfiltrate data, or install a reverse shell).

**Examples from industry:** `event-stream` (2018, crypto-miner injected via maintainer handover); `ua-parser-js` (2021, cryptominer + password stealer); `xz-utils` (2024, SSH backdoor). Enterprise security questionnaires routinely ask whether you have a supply chain incident runbook.

**Severity — determined by what the package has access to:**

| Package location | Access surface | FORM severity floor |
|---|---|---|
| Cloudflare Workers runtime dependency | Network egress, env vars (API keys), KV store | P0 — all production secrets potentially exposed |
| Supabase edge functions / DB trigger | Database credentials, RLS bypass surface | P0 |
| React Native / Expo app (production build) | User device, JWT in AsyncStorage, camera/health permissions | P0 if malicious code reaches a published build |
| Development toolchain only (not shipped) | CI environment, developer machines | P1 — secrets may be in dev env; no production user data |
| Transitive dependency (not directly imported) | Depends on call chain — assess before downgrading | P1 minimum until call chain confirmed clean |

**Do not downgrade from P0 without confirming the package is not on the production request path.**

#### Immediate Actions (T+0 to T+30 min)

```
1. Open incident channel: #inc-YYYYMMDD-supply-chain-[package-name]
   Example: #inc-20260520-supply-chain-lodash

2. Page: IC + security-engineer + devops-lead simultaneously.

3. Freeze all deployments immediately.
   - Cloudflare: pause Wrangler deploy workflows in GitHub Actions
   - EAS: revoke the active production release channel update if the build
     includes the affected package
   - Do NOT merge any PRs until the full dependency audit is complete.

4. Identify the blast radius:
   - Run: npm ls <package-name> (or equivalent for the package manager)
   - Confirm whether the package is in the production bundle or dev-only
   - Check if the malicious version range overlaps with what is in
     package-lock.json (or yarn.lock / bun.lockb)

5. Preserve evidence immediately:
   - Snapshot package-lock.json (or lockfile) at the current commit hash
   - Run npm audit --json > /tmp/npm-audit-YYYYMMDD.json and SHA-256 hash it
   - Check Cloudflare Workers outbound connection logs for anomalous destinations
     (look for unexpected IPs, cryptocurrency mining pools, C2 callback patterns)
   - Pull Supabase auth logs for any anomalous access in the past 24–72 hours
     (a compromised package may have already exfiltrated data before detection)

6. Assess: Has malicious code already executed in production?
   - Check Sentry for any new error types consistent with the advisory description
   - Check PostHog for anomalous event sequences (bulk data reads, export spikes)
   - Check Cloudflare Analytics for unexpected outbound request destinations
```

#### Scope Assessment

Answer in this order before external communication, but do not delay Art. 33 filing past T+48h if health data exposure is plausible:

```
[ ] What is the exact malicious version range?
    - Advisory CVE / GitHub Security Advisory ID:
    - Malicious version(s):
    - FORM's installed version (from lockfile):
    - Versions overlap? YES / NO

[ ] Is this package on the production request path?
    - Workers production bundle? YES / NO
    - Shipped mobile app (current production build in App Store)? YES / NO
    - Development/CI environment only? YES / NO

[ ] What is the package's runtime access surface?
    - Access to network egress? (potential exfiltration)
    - Access to environment variables / secrets?
    - Access to database credentials or Supabase service key?
    - Access to user device storage (AsyncStorage, Keychain)?
    - Access to camera, microphone, health data?

[ ] Time window of exposure:
    - First deploy of malicious version: [YYYY-MM-DD]
    - Most recent deploy: [YYYY-MM-DD]
    - Package version pinned? YES (check if pin covers malicious range) / NO

[ ] Evidence of active exploitation:
    - Cloudflare outbound logs: clean / anomalous [describe]
    - Supabase auth anomalies: none / detected [describe]
    - Sentry new error patterns: none / detected [describe]
```

#### Containment

```
1. Pin the affected package to the last confirmed-clean version in package-lock.json.
   If no clean version exists (entire package is compromised), remove the dependency
   and replace with an alternative or inline the minimal required functionality.

2. Remove the compromised package from the dependency tree:
   npm install <package>@<clean-version> --save-exact
   Verify lockfile updated before committing.

3. Rotate any secrets that were accessible to the package:
   - Supabase service role key and anon key → rotate immediately via Supabase dashboard
   - Anthropic API key → rotate in Anthropic console
   - ElevenLabs API key → rotate
   - All Cloudflare API tokens → rotate via CF dashboard
   - Any secrets stored in Cloudflare Workers environment variables
   After rotation, redeploy all Workers immediately with the new secrets.

4. Revoke all active user sessions if the package had access to JWT signing
   or session storage. Users will be required to re-authenticate.
   - Supabase: rotate the JWT secret in project settings (forces all token re-issue)
   - Notify enterprise tenant admins via §12 protocol before forcing re-auth.

5. If malicious code shipped in a mobile build:
   - EAS: revoke OTA update access for the affected build
   - Coordinate with devops-lead on forced update push to App Store if feasible
   - Do NOT submit an emergency App Store update without IC approval —
     App Store review SLA may be faster than the active exploit window, but
     confirm timeline with Apple Developer Support before waiting.
```

#### Eradication

```
1. Full dependency tree audit (all direct and transitive dependencies):
   npm audit --audit-level=high
   Review each CVSS ≥ 7.0 finding individually — do not auto-apply npm audit fix
   without reviewing what it changes.

2. Review all packages that underwent maintainer changes in the past 90 days
   using Socket.dev or similar supply chain analysis tool.

3. Confirm clean build:
   - Build from clean checkout on a fresh CI runner (no cached node_modules)
   - Verify bundle hash matches pre-incident baseline (if available)

4. Implement preventive controls (post-incident, to §9 testing program):
   - Add Dependabot security update auto-merge for patch-level CVSS < 5.0
   - Add npm audit step to CI pipeline (fail build on CVSS ≥ 7.0)
   - Add Socket.dev or Snyk to PR check pipeline for supply chain analysis
   - Pin lockfile versions for all production dependencies; automate lockfile
     integrity check in CI
```

#### Evidence Package (SOC 2 CC6.1, CC6.8, CC7.2, CC7.4)

```
[ ] Snapshot of package-lock.json at the malicious version (read-only copy)
[ ] npm audit output (JSON, SHA-256 hashed)
[ ] Cloudflare outbound connection log export for the exposure window
[ ] Supabase auth anomaly query results
[ ] Sentry error report export for the exposure window
[ ] Key rotation confirmation records (new key fingerprints + rotation timestamps)
[ ] Session revocation confirmation (Supabase JWT secret rotation timestamp)
[ ] Clean dependency audit output post-eradication
[ ] audit_log entries: all secret rotation events (HMAC-verified chain)
```

**Regulatory note:** If the package had runtime access to health data or PII and there is any evidence it executed network egress, this is a probable Art. 9 breach. Trigger Art. 33 assessment and start the 72-hour clock. Consult compliance-officer immediately.

---

### R-09: Ransomware / Destructive Payload

**Trigger:** Evidence that a destructive payload has executed within FORM-controlled systems: database records deleted or overwritten at scale; Cloudflare Workers source replaced with malicious code; git repository force-pushed with destructive history rewrite; infrastructure systematically destroyed or made unavailable; or files/objects replaced with encrypted versions and a ransom note detected.

**This is always P0. No exceptions.**

**Critical protocol difference from other runbooks:** In all other P0 incidents, the priority order is Evidence → Containment. In ransomware, the priority is **Isolation → Evidence → Recovery**. A live ransomware process will continue to encrypt or destroy data while you investigate. Stop the spread first.

**Do not pay.** FORM's recovery path is Supabase PITR and git history — not decryption. Ransom payment does not guarantee decryption, funds criminal infrastructure, and may trigger OFAC sanctions exposure. The decision to pay (if ever considered) requires founder + legal + compliance-officer unanimous approval and is categorically excluded from IC authority alone.

#### Immediate Actions (T+0 to T+15 min)

```
1. Open incident channel: #inc-YYYYMMDD-ransomware immediately.
   Page: IC + security-engineer + devops-lead. No delay.

2. ISOLATE FIRST — before any investigation:
   a. Supabase: revoke the service role key immediately from the dashboard.
      This stops any automated process using it from continuing to write/delete.
   b. Cloudflare: disable the production Workers via CF dashboard
      (set workers to maintenance mode or toggle off)
   c. Revoke all Cloudflare API tokens and GitHub personal access tokens
      from any account that shows anomalous activity.
   d. If the attack vector is a developer machine: have that developer
      immediately disconnect from all VPNs and network connections and
      power down the machine. Do not reboot — preserve volatile memory.

3. Take the platform offline deliberately:
   - Redirect all DNS to a maintenance page (Cloudflare Pages static maintenance page)
   - This is intentional — an isolated platform stops the spread.
   - Enterprise customer communication starts NOW via §12 protocol.
   - Post on status page: "We are performing emergency maintenance. No ETA yet."
     (Do not mention ransomware in the first public post — IC must approve any
     disclosure language.)

4. Preserve state before any recovery action:
   - DO NOT delete the ransom note artifacts
   - DO NOT attempt to decrypt or restore anything yet
   - Screenshot all visible ransom note text, attacker contact info, and
     demanded amounts. SHA-256 hash and store.
   - Note exact UTC timestamp of first detected symptom.
```

#### Scope Assessment

```
[ ] What systems are confirmed affected?
    - Supabase database (tables, rows affected): [describe]
    - Cloudflare Workers (source code modified): YES / NO
    - GitHub repository (history modified / force push): YES / NO
    - Developer machines (local files encrypted): YES / NO / UNKNOWN
    - R2 object storage (objects replaced / deleted): YES / NO

[ ] What is the attack vector hypothesis?
    - Compromised service role key (→ R-03 protocol also applies)
    - Compromised Cloudflare API token
    - Compromised GitHub account / Actions workflow
    - Compromised developer machine → lateral movement
    - Supply chain package (→ R-08 protocol also applies)
    - Social engineering / phishing

[ ] Timeline of destruction:
    - Earliest audit_log entry showing bulk DELETE or UPDATE: [timestamp]
    - Supabase PITR earliest clean restore point: [timestamp]
      (Supabase PITR window: 7 days at Pro tier; verify from dashboard)
    - Gap between last known-clean state and confirmed destruction: [duration]

[ ] Is the attack vector still active?
    - Are destructive operations still occurring? (check Supabase realtime metrics)
    - Has the isolated service role key been used after revocation? (check CF logs)
    - Are any GitHub Actions workflows still running? Cancel all immediately.
```

#### Recovery (executed only after full isolation is confirmed)

```
1. Verify Supabase PITR availability:
   - Log into Supabase dashboard → Database → Backups
   - Confirm the continuous backup stream shows a clean timestamp before the
     attack's earliest confirmed impact.
   - PITR target timestamp: [last clean timestamp from scope assessment]

2. Restore via Supabase PITR to the target timestamp.
   Follow the full procedure in docs/DATA_MODEL.md §10 (PITR Tenant-Isolated
   Restore Runbook). Key steps:
   a. Create a new Supabase project in the same region (do not restore into
      the compromised project — it may retain attacker access artifacts)
   b. Initiate PITR restore to the clean timestamp
   c. Verify row counts and spot-check health profiles against expected totals
   d. Verify HMAC audit log chain integrity on the restored database:
      run the chain verification script (§docs/AUDIT_LOG_SCHEMA.md)
      before trusting any audit log entries — the attacker may have
      attempted to inject false audit entries to obscure the timeline.

3. Restore Cloudflare Workers from the last known-good git commit:
   - Identify last clean commit hash in git history
   - If GitHub repo is compromised: restore from a local developer checkout
     (this is why every engineer maintains a local clone)
   - Deploy from clean commit using new CF API tokens (old tokens are revoked)

4. Provision new secrets — full rotation:
   - New Supabase project → new service role key, new anon key, new JWT secret
   - New Cloudflare API tokens (principle of least privilege — scope narrowly)
   - New Anthropic API key, ElevenLabs key, all third-party secrets
   - Update Workers environment variables before re-enabling traffic
   - Document all new key fingerprints in audit_log with actor_type: 'system'

5. Enterprise tenant validation before re-enabling traffic:
   - Verify each enterprise tenant's data is intact in the restored database
   - Run the RLS verification test suite (§docs/DATA_MODEL.md §3.5)
   - Confirm cross-tenant isolation is intact post-restore
   - Brief enterprise tenant admins directly before public re-opening (§12)

6. Re-enable production traffic:
   - IC authorizes re-enable only when: all vectors isolated, clean restore
     verified, new secrets in place, enterprise tenants validated, legal
     and compliance-officer have no objection.
   - Update status page with resolution message.
   - Internal post-mortem timeline starts: T+24h PIR.
```

#### Attack Vector Eradication

Do not re-enable production until the entry point is closed. Work through the hypothesized attack vector (scope assessment) with security-engineer:

```
Compromised service role key → key rotated (already done in recovery)
                                 + audit how key was exposed (git history,
                                   env file, CI logs, developer machine)

Compromised CF API token    → token rotated + audit CF access log for
                                 all API calls made with the token

Compromised GitHub account  → revoke all OAuth apps + re-enable MFA
                              + audit GitHub Actions workflow runs for
                                unauthorized secret access

Developer machine           → forensic image of machine before wiping
                              (preserve for potential law enforcement)
                              + rotate all secrets that machine could access
                              + audit git commits from that machine in past 30 days

Supply chain package        → follow R-08 eradication protocol
```

#### Evidence Package (SOC 2 CC7.4, CC9.1)

```
[ ] Ransom note artifacts (screenshots + SHA-256 hashes)
[ ] Supabase database metrics showing destruction timeline
[ ] audit_log entries for bulk DELETE / UPDATE operations (HMAC-verified)
[ ] Cloudflare Workers API access log covering attack window
[ ] GitHub Actions workflow run logs (if CI compromise suspected)
[ ] All key rotation confirmation records (audit_log entries)
[ ] PITR restore confirmation (new project ID, restore timestamp, row count verification)
[ ] HMAC chain re-verification report post-restore
[ ] RLS test suite pass result post-restore
[ ] Forensic image hash (if developer machine compromised)
```

**Law enforcement note:** Ransomware is a criminal act. Preserve all forensic artifacts. Do not engage law enforcement without founder + legal approval. FBI IC3 and CISA accept voluntary reports (US); NCSC accepts voluntary reports (UK/EU equivalent for major incidents). Reporting does not trigger mandatory disclosure independently of GDPR Art. 33.

**GDPR note:** If any user health data was accessed, exfiltrated, or destroyed during the attack, Art. 33 is triggered. Destruction of health data is a reportable breach even if no exfiltration occurred — "accidental or unlawful destruction" is explicitly in scope. Start the 72-hour clock at first awareness and consult compliance-officer immediately.

---

### R-10: AI Coach Safety Incident (Victor Harmful Guidance)

**Trigger:** Evidence that FORM's AI coach Victor has generated — or is generating at scale — coaching guidance that is clinically unsafe, triggers eating-disorder-adjacent behaviors, produces body-shaming content, or bypasses the safety guardrails defined in `docs/VICTOR_PROMPT_GUIDE.md` and the clinical-safety protocol. Triggers include:

- Direct user report of harmful coaching cue (injury, ED-adjacent language, dangerous load progression)
- clinical-safety review flags a `coaching_turns` record as violating a hard constraint
- Sentry alert: spike in `coaching_response_flagged` events above the 1% daily baseline
- PostHog alert: unusual `feedback_negative` event cluster (>5 in 15 min from distinct users)
- Anthropic safety filter bypass detected: response contains content that should have been blocked
- Enterprise tenant admin reports that Victor gave harmful guidance to an employee
- FORM employee discovers a prompt injection vector in user input that altered Victor's behavior

**This runbook does not cover: individual user complaints about response quality (handled by support), feature requests for Victor behavior changes, or disputes about workout programming philosophy. Those are support tickets, not incidents.**

**Severity:**

| Condition | Severity Floor |
|---|---|
| Single user, isolated session, no clinical harm reported | P2 — log and investigate in business hours |
| Multiple users (≥3) affected by the same harmful pattern in a 24-hour window | P1 — immediate investigation; IC notified |
| Confirmed ED-adjacent content (weight loss advice crossing clinical thresholds, body-weight comparisons, restriction advocacy) | P1 minimum; upgrade to P0 if >10 users affected |
| Body composition data surfaced to employer or admin dashboard (privacy floor violation) | P0 — triggers §12 enterprise tenant protocol immediately |
| AI-generated advice linked to a user injury report | P0 — legal, clinical-safety, and compliance-officer on call |
| Prompt injection vector confirmed: user-supplied text altered Victor's core persona or safety rules | P0 — all affected sessions are a security incident (CC7.4); assess GDPR Art. 34 |
| Harmful guidance generated for ≥50 users or persisted across >1 model version deployment | P0 — model-level or prompt-level defect; Anthropic notification required |

**Classification nuance:** AI safety incidents sit at the intersection of product safety and security. An Anthropic API safety filter failure is a third-party processor incident (→ R-07 cross-reference for sub-processor protocol). A prompt injection vector is a security vulnerability (→ CC7.4, treat as security incident). Harmful guidance from a well-functioning model is a product safety incident. All three paths start here; the scope assessment determines which branch applies.

#### Immediate Actions (T+0 to T+30 min)

```
1. Open incident channel: #inc-YYYYMMDD-ai-safety-[brief-slug]
   Example: #inc-20260520-ai-safety-ed-content-p1

2. Page: IC + clinical-safety + security-engineer.
   - clinical-safety has veto power on all containment decisions.
   - Do NOT disable the coaching feature without clinical-safety approval of
     the scope — a false positive that disables coaching for all users creates
     a different harm (users mid-session with no feedback).

3. Retrieve the harmful content immediately:
   a. Query coaching_turns table (read-only replica):
      SELECT session_id, user_id, content_hash, created_at, coach_response
      FROM coaching_turns
      WHERE created_at > NOW() - INTERVAL '24 hours'
        AND (tenant_id = '<affected_tenant>' OR user_id = '<affected_user>')
      ORDER BY created_at DESC LIMIT 50;

   b. Do NOT log the raw text of harmful coaching responses to the incident
      channel — they may contain ED-adjacent language that could harm responders.
      Use content_hash and session_id references only.

   c. Retrieve the system prompt version in use at the time:
      Check victor_prompt_version label in the coaching_turns record.
      Compare against the current production prompt in
      workers/victor/system-prompt.ts (or equivalent).

4. Assess prompt injection:
   - Retrieve the user input from the same coaching_turns record
     (user_message field or equivalent).
   - Check for: role override instructions, "ignore previous instructions",
     jailbreak patterns, excessive special characters or Unicode confusables,
     or content that redirects Victor's persona.
   - If prompt injection is confirmed → classify as P0 security incident immediately.
     Notify security-engineer; do not proceed with product-safety branch alone.

5. Check blast radius:
   - Is this a single session or a systematic prompt/model issue?
   - Query for the same victor_prompt_version or model_version across all
     coaching_turns in the past 72 hours:
     SELECT COUNT(*), victor_prompt_version
     FROM coaching_turns
     WHERE created_at > NOW() - INTERVAL '72 hours'
     GROUP BY victor_prompt_version;
   - Check PostHog: feedback_negative events by timestamp — look for a cluster
     that correlates with a specific deployment or model version.
```

#### Scope Assessment

Answer before any external communication. P0 upgrade if any "yes" answer:

```
[ ] Was the harmful content ED-adjacent (caloric restriction, weight comparisons,
    body-weight language that a clinical-safety reviewer would flag)?
    YES / NO

[ ] Was body composition data surfaced to an employer admin dashboard?
    YES — this is a privacy floor violation; trigger §12 and DEC-030
    data.read_individual event audit immediately.
    NO

[ ] Is the harmful pattern reproducible? Test in staging using the same
    user_input → system_prompt combination (never test on production).
    REPRODUCIBLE / NOT REPRODUCIBLE

[ ] Is this a model-level issue (same input produces same output on
    multiple model calls) or a prompt-level issue (system prompt change
    introduced the regression)?
    MODEL-LEVEL (→ Anthropic notification; R-07 branch)
    PROMPT-LEVEL (→ system prompt rollback; this runbook)
    PROMPT-INJECTION (→ P0 security incident; R-01/CC7.4 branch also active)

[ ] Scope of affected users:
    - Users who received the harmful content: ___
    - Enterprise tenants affected: ___
    - Time window of exposure: ___
    - Is the system prompt still deployed? YES / NO

[ ] Has any affected user reported a physical injury or clinical harm?
    YES — notify clinical-safety immediately; begin Art. 34 assessment with
    compliance-officer. Do not make any public statement without legal review.
    NO — continue assessment.

[ ] Was this a wellness-as-punishment scenario (enterprise tenant using
    FORM to punish employees for non-participation, or employer pressure
    linked to Victor guidance)?
    YES — this is a no-go customer contract violation; escalate to founder.
    NO
```

#### Containment

```
1. Disable the affected coaching pathway via feature flag:
   - Use the tenant_feature_flags table (or equivalent Workers KV entry)
     to disable the specific feature — not the entire coaching product.
   - If the issue is the system prompt, not a specific feature:
     a. Do NOT disable all coaching — this harms all users.
     b. Patch the system prompt in victor_prompt_configs (or Workers secret)
        with the previous clean version. Requires security-engineer approval
        to write a production secret.
     c. Deploy the patched prompt to production Workers via Wrangler deploy.
        Standard change management (§9.4 change management) is waived for
        P0/P1 safety incidents; IC approval suffices.

2. Prevent new affected sessions:
   - If the harmful pattern is deterministic and reproducible: deploy the fix
     before re-enabling the pathway.
   - If the harmful pattern is stochastic (occurs occasionally for certain
     input patterns): add a content filter check at the edge Worker level —
     scan Victor's response for the blocked pattern before delivering to client.
     DEC-030 event: coaching.response_blocked (new event type if not already
     defined; add to AUDIT_LOG_SCHEMA.md under session events).

3. Do not delete or modify coaching_turns records.
   The coaching_turns table is append-only. Evidence preservation (§7) applies.
   Harmful responses are retained for post-incident review and pattern analysis.
   The content_hash makes each response identifiable without exposing full text.

4. Notify clinical-safety owner of the exact nature of the harmful content.
   clinical-safety has veto power on the "safe to re-enable" decision.
   No pathway is re-enabled until clinical-safety signs off in the incident channel.

5. If the affected system prompt version was deployed via CI:
   - Revert the relevant commit (git revert — do not git reset --hard)
   - Open a hotfix PR with the prompt correction
   - clinical-safety must approve the PR before merge
   - This is the only change management exception: clinical-safety approval
     takes precedence over the normal code review requirement.
```

#### Eradication

```
1. Root cause determination:
   a. PROMPT REGRESSION: Which prompt change introduced the harmful pattern?
      - git log workers/victor/system-prompt.ts (or equivalent path)
      - Compare the deployed prompt version with the previous version
      - Identify the specific clause or instruction that allowed harmful output
      - clinical-safety reviews the delta before any new prompt version is drafted

   b. MODEL BEHAVIOR DRIFT: Same prompt, same input, but model output changed.
      - Record the exact Anthropic model version used (from API response header
        or from the model_version field in coaching_turns)
      - File an issue with Anthropic safety team via their developer safety
        reporting channel: https://www.anthropic.com/safety
      - Document the input/output pair (without user PII) for Anthropic report
      - Do not assume a model regression is benign — it may affect all FORM users
        until a model version pin or Anthropic fix is in place

   c. PROMPT INJECTION: User-supplied input manipulated Victor's core behavior.
      - Document the injection vector exactly
      - Implement input sanitization at the edge Worker: reject or transform
        inputs containing role-override patterns before they reach the Anthropic API
      - Sanitization rules must be tested against the Victor Jailbreak Test Suite
        (add to §9 testing program if not yet present)
      - Review whether other user-controlled input fields (nutrition log,
        workout description free text) have the same injection surface

2. Clinical-safety review of the full remediated prompt before re-deployment.
   This is not optional. Duration: typically 24-48h for clinical-safety turnaround.
   For P0 incidents, request an emergency clinical-safety review (<4h SLA).

3. Implement preventive controls (add to §9 testing program):
   - Add the harmful input pattern to the Victor regression test suite
   - Add a clinical-safety review step to the prompt change PR template
   - Add PostHog alert: feedback_negative rate >1% on any 15-minute window
   - Add Sentry alert: coaching_response_flagged event spike
   - Add a content filter Worker middleware that screens Victor responses for
     blocked patterns (ED-adjacent language list maintained by clinical-safety)
   - Review VICTOR_PROMPT_GUIDE.md — if the safety rules were unclear,
     rewrite the relevant section before the patched prompt deploys
```

#### Evidence Package (SOC 2 CC7.4, CC5.2, CC1.2)

```
[ ] coaching_turns records (SHA-256 hashed content_hash identifiers only —
    do NOT include raw harmful text in evidence package; store separately
    in a restricted-access compliance folder with clinical-safety access control)
[ ] System prompt diff: affected version vs. previous clean version
[ ] PostHog feedback_negative event export for the affected time window
[ ] Sentry coaching_response_flagged events (if applicable)
[ ] Blast radius query result: count of affected users and sessions
[ ] DEC-030 audit log entries for the incident window (HMAC chain verified)
[ ] clinical-safety sign-off message from incident channel (timestamp + approver)
[ ] Anthropic API model version in use at time of incident
[ ] Prompt injection input sample (PII stripped) if applicable
[ ] Patched prompt version (after clinical-safety review)
[ ] Re-enable confirmation: timestamp + clinical-safety approver name
[ ] Post-incident clinical-safety review report (filed to compliance/cc7/ai-safety/)
```

**Clinical-safety note:** Any coaching_turns content that a clinical reviewer judges to meet the criteria for ED-adjacent advice, dangerous injury-risk guidance, or wellness-as-punishment must be retained in the restricted compliance folder and treated as clinical evidence. This material is not for general incident channel distribution. Access is limited to: clinical-safety, compliance-officer, security-engineer, and founder.

**Regulatory note (GDPR Art. 34):** If harmful AI guidance caused physical injury to a user and that injury is attributable to FORM's coaching product, Art. 34 (notification to the individual) may be required. This determination requires outside counsel. The 72-hour Art. 33 window for supervisory authority notification applies if the incident also involves unauthorized access to or alteration of the user's health data. Consult compliance-officer within 4 hours of confirming any physical injury report.

**Anthropic sub-processor note:** If the root cause is a model-level regression or safety filter failure at Anthropic, activate R-07 (Sub-processor Breach Notification Received) in parallel. The R-07 protocol governs sub-processor incident coordination; R-10 governs FORM's internal product-safety response. Both run simultaneously.

**No-go customer note:** If the investigation reveals that an enterprise customer is using FORM in a wellness-as-punishment configuration (e.g., reporting non-engagement to HR for disciplinary action, using engagement metrics in performance reviews, insurance risk-scoring integrations), this is a contract violation and a no-go customer classification. Escalate immediately to founder. Do not disclose the harmful-guidance incident to other customers until the no-go customer situation is resolved — the two incidents may have different disclosure obligations.

---

### R-11: CV / Pose Estimation Biometric Data Privacy Incident

> **Always P0.** GDPR Art. 9 biometric data breach carries the highest individual risk classification. Declare P0 immediately on any credible exposure path — do not wait for confirmation of actual access.

#### Why R-11 Is Different from R-01

R-01 covers general PII/health-data breaches. R-11 applies specifically when the CV pipeline's output data — pose keypoints, form-score signals, or the `keypoints_enc` column in `cv_sessions` — is potentially exposed. Biometric data processed for the purpose of uniquely identifying a natural person is Art. 9(1) special category data. Supervisory authority notification under Art. 33 is presumed required; individual notification under Art. 34 is likely required (biometric breach = high risk to data subjects by default).

**FORM's privacy-first CV architecture reduces but does not eliminate risk:**

| Data layer | Where it lives | Breach surface |
|---|---|---|
| Raw video frames | On-device only — never transmitted | No server-side exposure path |
| Real-time keypoints | On-device only — processed in-memory by ML Kit / BlazePose | No server-side exposure path |
| `keypoints_enc` | `cv_sessions` column in Supabase; AES-256-GCM encrypted at rest | Exposed **only** if encryption key is compromised or RLS policy fails |
| Per-session aggregates | `cv_sessions.avg_confidence`, `defect_flags`, etc. | Standard Art. 9 health data — covered by R-01 if exposed |
| Fleet stats | `cv_session_fleet_stats` materialized view — k-anonymity enforced, no `user_id` | Low individual risk; de-identified by design |

Most R-11 incidents will be **Track B** (server-side, limited to `cv_sessions`). Track A (on-device) applies when a mobile device is physically compromised.

#### Severity Assignment

| Condition | Severity |
|---|---|
| `keypoints_enc` column accessible due to RLS failure or key compromise | **P0** |
| CV session aggregate data (`avg_confidence`, `defect_flags`) cross-tenant RLS failure | **P0** (R-01 also active) |
| Physical device loss / theft where CV session data may be cached locally | **P0** (coordinate with affected user directly) |
| `cv_session_fleet_stats` view exposed (k-anonymity, no `user_id`) | **P1** — lower individual risk; upgrade to P0 if k-anonymity n < 5 |
| CV feature flag misconfiguration exposing CV to users who opted out | **P1** |

#### Immediate Actions (T+0 to T+15 min)

```
[ ] Declare P0 in #inc-YYYYMMDD-cv-biometric. Page security-engineer + compliance-officer.
[ ] Start GDPR Art. 33 72-hour clock. Record exact time of first awareness.
[ ] Page clinical-safety. CV pose data is health-category data; clinical-safety
    advises on user-harm risk assessment throughout the incident.
[ ] Disable the cv_enabled feature flag globally (all tenants) via admin dashboard.
    Workers pick up flag changes within ~30 seconds.
    Do NOT selectively disable — if keypoints_enc is at risk, disable fleet-wide.
[ ] Confirm which track applies:
    - Track A (on-device): device lost/stolen, malware on device → coordinate
      with affected user; instruct them to remotely wipe via iOS/Android MDM
    - Track B (server-side): Supabase `cv_sessions` or `keypoints_enc` access
      path identified → proceed to scope assessment immediately
[ ] Do NOT run SELECT on keypoints_enc values — this would retrieve decrypted
    biometric data into memory. Run COUNT(*) and metadata queries only.
    If decrypted content is needed for scope assessment, use compliance-officer
    + security-engineer approval with dedicated audit log entry.
```

#### Track B: Scope Assessment (Server-Side)

```sql
-- Step 1: How many users and sessions are in the exposure window?
SELECT
  tenant_id,
  COUNT(DISTINCT user_id)         AS affected_users,
  COUNT(*)                         AS affected_sessions,
  MIN(started_at)                  AS earliest_session,
  MAX(started_at)                  AS latest_session
FROM cv_sessions
WHERE started_at >= $breach_window_start
  AND started_at <= $breach_window_end
GROUP BY tenant_id
ORDER BY affected_sessions DESC;

-- Step 2: Are keypoints_enc values actually populated (or all NULL)?
-- NULL = session recorded but keypoints were never stored server-side.
SELECT
  COUNT(*)                              AS total_sessions,
  COUNT(*) FILTER (WHERE keypoints_enc IS NOT NULL)  AS sessions_with_keypoints,
  COUNT(*) FILTER (WHERE keypoints_enc IS NULL)      AS sessions_without_keypoints
FROM cv_sessions
WHERE started_at >= $breach_window_start;

-- Step 3: Verify RLS is enforced on cv_sessions.
-- Run as the Supabase anon role — should return zero rows.
SET ROLE anon;
SELECT COUNT(*) FROM cv_sessions;
-- Expected: 0. Any other result = RLS failure → P0 confirmed, escalate to R-01 also.

-- Step 4: Check keypoints_enc encryption key access log.
-- This is outside Postgres — check 1Password audit trail or KMS access logs
-- for cv_keypoints_encryption_key. Any access not by platform-engineer during
-- normal operations is a finding.
```

**If keypoints_enc IS NULL for all sessions in the window:** biometric data was never persisted server-side for this cohort. Individual risk is significantly reduced. The session aggregates (`avg_confidence`, `defect_flags`) are still Art. 9 health data but not biometric in the strict sense. Downgrade to P1 after confirming RLS integrity; Art. 33 notification may still be required — consult compliance-officer.

**If keypoints_enc IS NOT NULL for any session in the window:** biometric data was persisted. Treat as confirmed biometric exposure even if the encryption key has not been confirmed compromised — the precautionary principle applies under GDPR.

#### Containment

```
[ ] CV feature flag is disabled (confirmed in step 1 of immediate actions).
[ ] Rotate cv_keypoints_encryption_key in 1Password / Cloudflare Workers Secrets.
    New key is active for any future sessions. Historical keypoints_enc values
    remain encrypted with the old key — assess whether re-encryption is required
    (only if the old key is confirmed compromised, not merely suspected).
[ ] If RLS failure confirmed (Step 3 above returned rows):
    - Activate R-01 in parallel
    - Block all Supabase direct-connection access except platform-engineer
    - Run §5.R-01 RLS verification steps for cv_sessions table specifically
[ ] Preserve a read-only DB snapshot with PITR marker for evidence (see §7).
[ ] Block any export / analytics job that might touch cv_sessions during investigation.
    - Pause pg_cron job for cv_session_fleet_stats materialized view refresh.
    - Suspend any Metabase dashboard that queries cv_sessions directly.
```

#### GDPR Art. 9 Notification Assessment

Biometric data exposure triggers a heightened notification analysis. Complete this within T+4 hours.

| Question | Answer required before Art. 33 notification |
|---|---|
| Was `keypoints_enc` accessible (key compromised or RLS failure)? | Yes → Art. 33 notification required; likely Art. 34 also |
| Were keypoints_enc values all NULL (never persisted)? | Art. 33 may be required for session aggregates; Art. 34 lower threshold |
| How many data subjects are affected? | Required for notification form; report range if exact count unavailable |
| Which EU supervisory authorities have jurisdiction? | Determined by user location; FORM's lead authority: determined by main establishment |
| Is there a DPA in place with relevant enterprise tenants? | Enterprise tenant must be notified per their DPA SLA (§12) |

**Art. 34 trigger for biometric data:** GDPR Recital 75 and 85 identify biometric data as inherently high-risk. Supervisory authorities in most EU jurisdictions treat biometric breaches as meeting the Art. 34 "high risk to rights and freedoms" threshold. Do not apply the R-01 assessment that sometimes concludes Art. 34 is not required — for biometric data, assume Art. 34 applies unless outside counsel advises otherwise.

**Notification timeline:**
- T+0: GDPR Art. 33 clock starts
- T+4h: compliance-officer completes initial Art. 9 risk assessment
- T+24h: draft Art. 33 notification to supervisory authority (partial if scope not confirmed)
- T+48h: file Art. 33 notification (partial notification acceptable; update as scope confirmed)
- T+72h: hard deadline for Art. 33 notification
- T+72h + 0: Art. 34 individual notifications begin (if required), coordinated with enterprise tenant notifications (§12)

#### Enterprise Tenant Notification (CV-Specific)

If affected users belong to enterprise tenants:
1. Follow §12 Template E-01/E-02/E-03 timeline (30 min P0 initial notification).
2. Disclose specifically that **pose estimation data** was involved, not just generic "health data." Enterprise DPAs may have specific clauses about biometric data sub-processing.
3. Provide the tenant's DPO (if they have one) with the preliminary scope assessment: session count, user count, whether `keypoints_enc` was populated.
4. Enterprise tenant may have their own Art. 33 obligations as joint controllers — coordinate with their DPO before filing separate notifications to avoid conflicting statements to supervisory authorities.

**Privacy floor enforcement:** Under no circumstances disclose the list of affected users to an HR administrator or corporate wellness manager. Affected users receive individual Art. 34 notifications directly from FORM. The enterprise tenant's administrative contact receives aggregate scope information only.

#### Clinical-Safety Coordination

CV pose estimation records movement patterns that may reveal health conditions (injury compensation patterns, mobility limitations, chronic pain adaptations). clinical-safety must:
- Review the affected cohort's session metadata for any red-flag indicators (sessions flagged `below_confidence_threshold` on sensitive exercises may indicate users self-treating injuries)
- Assess whether any affected users have a pending clinical referral in their coaching session history
- Advise on Art. 34 notification language — the user-facing notification must not inadvertently stigmatize users or reveal health inferences drawn from their pose data

clinical-safety sign-off on Art. 34 notification text is mandatory before sending.

#### Evidence Package (SOC 2 CC7.2, CC7.4, C1.1, PI1.2, A1.1)

```
[ ] cv_sessions scope query output (session count, user count, tenant breakdown)
    — do NOT include keypoints_enc column values in evidence package
[ ] keypoints_enc null/populated ratio for the exposure window
[ ] Encryption key access audit trail: 1Password activity log or KMS audit
    for cv_keypoints_encryption_key covering 30 days before incident
[ ] RLS verification output: pg_tables rowsecurity check for cv_sessions
    (expected: rowsecurity = true; any other result = finding)
[ ] cv_session_fleet_stats: materialized view DDL verification that
    GROUP BY user_id is absent and k-anonymity n≥5 guard is present
[ ] Feature flag disable confirmation: timestamp + operator name
[ ] DEC-030 audit log entries for cv_sessions access events in incident window
    (HMAC chain verified; run §5.R-05 chain verification if any gaps)
[ ] Cloudflare Workers request logs: any anomalous traffic to CV-related endpoints
[ ] OBSERVABILITY.md §18 dashboard screenshots: CV Model Health panel at time of incident
[ ] clinical-safety sign-off on Art. 34 notification draft (timestamp + approver)
[ ] Art. 33 supervisory authority notification draft + filing confirmation
[ ] Enterprise tenant DPA reference and notification sent confirmation
[ ] Post-incident key rotation confirmation: new cv_keypoints_encryption_key
    deployed + old key decommissioned from Workers Secrets
```

#### Post-Incident: Mandatory Controls Review

After resolution, before re-enabling CV feature flag:

1. **Encryption key governance review:** Confirm cv_keypoints_encryption_key is rotated on a schedule (recommendation: 90-day rotation; align with CRYPTOGRAPHY_POLICY.md). Confirm key access is logged and alerted.
2. **`keypoints_enc` storage necessity review:** Assess whether server-side keypoints storage remains necessary. If it can be eliminated or replaced with on-device-only storage, this eliminates the biometric exposure surface entirely. Requires enterprise-architect + ml-engineer sign-off.
3. **k-anonymity enforcement in `cv_session_fleet_stats`:** Confirm the `HAVING COUNT(DISTINCT user_id) >= 5` guard is in the materialized view DDL (per OBSERVABILITY.md §18).
4. **OBSERVABILITY.md §18 alerting:** Confirm AL-CV-01 through AL-CV-08 alerts are active and routed to PagerDuty (not just Slack) for P0-class signals.
5. **Re-enable gate:** ml-engineer + security-engineer + clinical-safety must all sign off before CV feature flag is re-enabled for any tenant.

---

## 6. Communication Templates

### 6.1 Internal Incident Channel Opening Message

Channel naming convention: `#inc-YYYYMMDD-[slug]` (e.g., `#inc-20260516-rls-bypass`)

Post this as the first message and **pin it** immediately. Update the pinned message as situation evolves.

```
INCIDENT DECLARED
==================
Severity: [P0 / P1 / P2]
Type: [Data Breach / Outage / Credential Compromise / SSO / Chain Break / DDoS]
Detected: [HH:MM UTC]
IC: [Name]
Technical Lead: [Name]
Security Lead: [Name]
Comms Lead: [Name]

CURRENT STATUS: [Investigating / Containing / Eradicating / Recovering]

WHAT WE KNOW:
[1-3 sentences of confirmed facts only — no speculation]

WHAT WE DON'T KNOW YET:
[What is still being determined]

GDPR CLOCK: [T+Xh Ymin of 72h / Not triggered / Monitoring]

AFFECTED:
- Tenants: [All / Specific tenants listed / Unknown]
- Users: [Estimated count or Unknown]
- Data: [Types involved or Unknown]

NEXT UPDATE: [HH:MM UTC]
```

### 6.2 Status Page Update Templates (by Severity)

**P0 — Initial investigation post:**
```
Title: Investigating service disruption
We are investigating a report of [brief description]. Our team is actively working to
identify the cause. We will provide an update within 30 minutes.

Impact: [Service unavailable / Degraded performance / Potential data issue]
Started: [HH:MM UTC]
```

**P1 — Partial impact:**
```
Title: Investigating partial service issue
We are investigating a report affecting [feature/region/tenant type].
[Unaffected functionality] continues to operate normally.
We will provide an update within 60 minutes.
```

**P2 — Performance:**
```
Title: Degraded performance on [feature]
We are investigating elevated response times on [affected area].
No data is at risk. We will update as we learn more.
```

**Resolution post (all severities):**
```
Title: [Issue] resolved
The issue affecting [description] has been resolved as of [HH:MM UTC].

Summary:
- Issue started: [HH:MM UTC]
- Issue resolved: [HH:MM UTC]
- Impact: [Brief honest description of user impact]
- Root cause: [1-2 sentences — honest and specific]

We are conducting a post-incident review and will share our findings.
We apologize for the disruption.
```

### 6.3 GDPR Art. 33 — Supervisory Authority Notification Template

File with the lead supervisory authority (determined by FORM's EU establishment or, if none, the member state where most affected data subjects are located).

File at: [Applicable DPA's online breach notification portal — e.g., ICO (UK), CNIL (FR), BfDI (DE), DPC (IE)]

```
PERSONAL DATA BREACH NOTIFICATION
Article 33 GDPR — Controller Notification to Supervisory Authority

Controller name: FORM Technologies Ltd [confirm legal entity name]
Controller address: [registered address]
DPO / Contact: [security-engineer contact details]
Date/time of notification: [ISO 8601]
Date/time of discovery: [ISO 8601] — NOTE: [X hours Y minutes elapsed since discovery]

1. Nature of the breach
[Describe: unauthorized access / accidental disclosure / loss of data / other]
Categories of data involved: [health data / name / email / workout logs / other]
Approximate number of data subjects: [number or "under investigation — estimated range X–Y"]
Approximate number of records: [number or "under investigation"]

2. Categories and approximate number of data subjects concerned
[Describe: FORM enterprise employees at [company name] / consumer users / both]
[Note if special categories (Art. 9) are involved — health data is Art. 9]

3. Likely consequences of the breach
[Describe potential harm: identity theft risk / reputational harm / discrimination risk /
unauthorized access to health information / other]

4. Measures taken or proposed to address the breach
- Containment: [describe what has been done]
- Eradication: [describe root cause fix]
- Recovery: [describe restoration actions]
- Prevention: [describe planned changes]

5. Contact for further information
[Name, role, email, phone]

NOTE: This notification is made while the investigation is ongoing.
We will supplement this notification with additional information as it becomes available.
Supplementary notification expected: [date, within 72h of additional findings if applicable]
```

### 6.4 GDPR Art. 34 — Data Subject Notification Template

Required when the breach is likely to result in a **high risk** to the rights and freedoms of individuals (the DPA may also direct this notification even if you assess risk as not high).

Send via: the primary contact method the user registered with (email). Send from a recognizable FORM address. Do not use a no-reply address — users may have questions.

```
Subject: Important security notice about your FORM account

Dear [User name or "FORM member" if name not safely available],

We are writing to inform you of a security incident that may have affected your FORM account.

What happened:
[Specific, honest description of the incident. No jargon. No minimization.]
[Example: "On [date], we discovered that an error in our access control software meant that
a small number of users' workout data could have been accessed by other users for a period
of [time window]."]

What information was involved:
[Specific list: workout logs / meal logs / health metrics / account email / other]
[Be specific. "Your data" is not specific enough.]

What we have done:
- [Specific action taken, e.g., "Fixed the error as of [time] on [date]"]
- [Specific action, e.g., "Reviewed all potentially affected accounts"]
- [Specific action, e.g., "Notified the relevant data protection authority"]

What you should do:
[Only include actionable steps if genuinely needed. Do not create unnecessary alarm.]
[Example: "No action is required from you at this time. If you notice any unusual activity
in your account, please contact us at security@form.coach"]
[If password/credential exposure: "We recommend you change your password on any service
where you use the same password."]

Contact us:
If you have any questions or concerns, please contact us at security@form.coach.
We are available to answer your questions.

We sincerely apologize for this incident. We take the security of your health data
seriously and are committed to preventing this from happening again.

[Signature — IC name and title]
FORM
```

### 6.5 Post-Incident Customer Communication Template

Sent after full resolution, before PIR publication. For enterprise customers, delivered via direct email from the IC or account manager in addition to status page.

```
Subject: Post-incident summary — [brief description] — [date]

[For enterprise customers: address the tenant admin by name]

We're writing with a full summary of the [incident type] that occurred on [date].

What happened:
[Plain-language description of root cause. More detail than the initial comms.]

Timeline:
- [HH:MM UTC] — [Event]
- [HH:MM UTC] — [Event]
[Continue as appropriate]

Impact:
[Specific, honest description of what was affected and for how long.]
[For data incidents: explicitly state what data was and was not involved.]

What we fixed:
[Specific technical changes made.]

What we're changing to prevent recurrence:
[Specific commitments with timelines. Do not over-promise.]

Our post-incident review:
We are conducting a full post-incident review. We will share our findings [publicly /
with affected customers on request] within [7 / 30] days.

If you have questions:
[Direct contact — name, email, phone for enterprise customers]

[Signature]
```

---

## 7. Evidence Preservation

### 7.1 What to Collect

Collect in this order of priority (most time-sensitive first):

| Priority | Evidence Type | Retention Period | Notes |
|---|---|---|---|
| 1 | In-memory state / live sessions | Capture immediately | Will be lost on instance restart |
| 2 | Real-time log streams (Cloudflare, Supabase) | Export immediately | Logs may roll over |
| 3 | Audit log export for affected tenant/time window | Within 30 min | Read-only export only |
| 4 | HMAC chain export | Within 30 min | Before any chain repair |
| 5 | PostHog event export for anomalous window | Within 1 hour | — |
| 6 | Screenshots of dashboards showing anomaly | Within 1 hour | Dashboards change |
| 7 | Network captures (Cloudflare WAF logs) | Within 2 hours | — |
| 8 | Git history for recent deployments | As needed | Stable |
| 9 | Dependency manifest snapshots | As needed | Stable |

**Specific commands:**

```bash
# Supabase DB logs (requires Supabase CLI authenticated)
supabase db logs --since 2h --project-ref <project_ref> \
  > /secure-evidence/supabase-db-$(date -u +%Y%m%dT%H%M%SZ).log

# Supabase Auth logs
supabase auth logs --since 2h --project-ref <project_ref> \
  > /secure-evidence/supabase-auth-$(date -u +%Y%m%dT%H%M%SZ).log

# Cloudflare Worker logs (via Wrangler tail — snapshot current, pipe to file)
wrangler tail --env production --format json \
  > /secure-evidence/cf-worker-$(date -u +%Y%m%dT%H%M%SZ).ndjson

# Audit log export from Postgres
psql $DATABASE_URL -c "\COPY (
  SELECT * FROM audit_log
  WHERE created_at >= now() - interval '4 hours'
  ORDER BY created_at ASC
) TO '/secure-evidence/audit-log-$(date -u +%Y%m%dT%H%M%SZ).csv' WITH CSV HEADER;"

# Hash all collected evidence
sha256sum /secure-evidence/* > /secure-evidence/MANIFEST.sha256
```

### 7.2 Chain of Custody

Every piece of evidence must have a chain of custody record. Create a custody log in the incident Slack channel and in the encrypted evidence store.

Format for each artifact:

```
Evidence item: [filename or description]
Collected by: [name]
Collection time: [ISO 8601 UTC]
Collection method: [command run / screenshot / export / other]
SHA-256 hash: [hash of file at time of collection]
Stored at: [path in encrypted evidence store]
Copied to (if transferred): [destination + by whom + when]
Notes: [any context — was the system state modified before collection? if yes, what changed?]
```

### 7.3 Evidence Storage

**Primary evidence store:** Encrypted S3-compatible object storage (Cloudflare R2 or AWS S3).
- Bucket name: `form-incident-evidence` (access-logged)
- Path structure: `/<YYYYMMDD>-<incident-slug>/<artifact-name>`
- Encryption: SSE-S3 or SSE-KMS
- Access: security-engineer + IC only during incident; read access for compliance-officer post-incident
- Every access to the bucket is logged

**Access control:**
- Evidence bucket has no public access
- Bucket policy allows only the incident response IAM role
- All access generates CloudTrail / R2 audit log entries

**Minimum retention: 1 year** for all P0/P1 incident evidence. 7 years for incidents involving confirmed data breach or regulatory notification.

### 7.4 Immutability Rules

- **Never modify production logs.** All log collection is read-only. Use `SELECT`, `COPY TO`, or log export APIs only.
- **Never delete evidence during an active incident.** If you need to free disk space, compress — do not delete.
- **Audit log chain repair** is the single exception to the no-modification rule, and it requires dual authorization (IC + security-engineer) with the repair itself logged as a `system.config_changed` event.
- **Write-once principle:** once an evidence artifact is written to the incident evidence store, it is not overwritten. If you collect an updated version, store it as a new file with a new timestamp.

---

## 8. Post-Incident Review (PIR)

### 8.1 When PIR is Required

| Severity | PIR Required | PIR Format | Distribution |
|---|---|---|---|
| P0 | Mandatory | Full PIR (§8.3 template) | IC, all leads, compliance-officer, board notification |
| P1 | Mandatory | Full PIR | IC, all leads, compliance-officer |
| P2 | Required if pattern detected (≥2 P2 incidents of same type in 30 days) | Abbreviated PIR | IC, Technical Lead |
| P3 | Not required | — | — |

### 8.2 PIR Timeline

```
T+24h   PIR draft owner assigned (IC assigns or self-assigns)
T+24h   Incident channel converted to read-only; channel preserved as evidence
T+48h   PIR draft circulated to all incident responders for factual corrections
T+72h   Final PIR published to internal incident record
T+72h   For P0: board notification sent with PIR summary
T+30d   For P0: public post-mortem published at form.coach/security (IC decision)
```

### 8.3 PIR Template

```markdown
# Post-Incident Review — [Incident ID: inc-YYYYMMDD-slug]

**Severity:** [P0 / P1 / P2]
**Date of incident:** [YYYY-MM-DD]
**Date of PIR:** [YYYY-MM-DD]
**PIR author:** [name]
**Reviewed by:** [names]
**Status:** [Draft / Final]

---

## Blameless Culture Statement

This PIR is conducted in the belief that engineers and responders act in good faith
with the information and tools available to them at the time. The purpose of this
review is to improve systems and processes, not to assign blame to individuals.
Findings are about systems, not people.

---

## Incident Summary

[2–4 sentences: what happened, what was affected, how it was resolved.]

## Timeline

| Time (UTC) | Event | Who |
|---|---|---|
| [HH:MM] | [Event] | [Name] |
[Continue for full incident timeline — every significant action]

## Root Cause Analysis

### What happened (the technical root cause)
[Specific technical description of the failure mode]

### Why it happened (5 Whys)

1. **Why** did [symptom] occur?
   Because [cause 1].

2. **Why** did [cause 1] occur?
   Because [cause 2].

3. **Why** did [cause 2] occur?
   Because [cause 3].

4. **Why** did [cause 3] occur?
   Because [cause 4].

5. **Why** did [cause 4] occur?
   Because [systemic root cause].

### Contributing factors
- [Factor 1 — e.g., monitoring gap that delayed detection]
- [Factor 2 — e.g., RLS policy not covered by test suite]

## Impact

- **Users affected:** [number or range]
- **Tenants affected:** [list or count]
- **Data involved:** [categories, if any]
- **Downtime duration:** [HH:MM]
- **Regulatory impact:** [GDPR notification filed / not triggered / monitoring]

## What went well

- [Thing that worked — detection was fast / runbook was accurate / team communicated well]

## What could be improved

- [Process or system gap — do not name individuals]

## Action Items

| Item | Owner | Due Date | Status |
|---|---|---|---|
| [Specific change to code / process / monitoring] | [name] | [YYYY-MM-DD] | Open |
[Continue]

## Evidence Links

- Incident channel: #inc-YYYYMMDD-slug (archived, read-only)
- Evidence store: form-incident-evidence/YYYYMMDD-slug/
- Regulatory filings: [link or "None required"]
```

### 8.4 SOC 2 Evidence Requirements for PIR

SOC 2 CC7.5 requires evidence that FORM evaluates and responds to detected security events and that a review process exists. The following constitute the evidence package for each P0/P1 PIR:

- The incident channel log (exported, read-only)
- The evidence manifest with SHA-256 hashes
- The final PIR document (signed off by IC)
- Regulatory filing receipt(s) if applicable
- Action items tracked to completion in Linear (with close dates)

**The PIR is not complete for SOC 2 purposes until all Critical and High action items have a confirmed due date and owner assigned.** They do not need to be closed before the PIR is finalized, but they must be tracked.

---

## 9. Testing and Drills

### 9.1 Testing Program

| Test Type | Frequency | Scope | Owner |
|---|---|---|---|
| Alert verification | Monthly | Confirm all PagerDuty alerts fire on test triggers | devops-lead |
| Tabletop exercise (P0 scenario) | Quarterly | Walk through a P0 scenario without touching production | security-engineer |
| Fire drill (simulated outage) | Semi-annual | Simulate a real outage; test rollback and comms | IC + devops-lead |
| Runbook accuracy review | After every P0/P1 | Update runbook based on what actually happened | security-engineer |
| DR test (backup restoration) | Annual | Restore from Supabase backup to staging; verify data integrity | devops-lead |
| Pen test | Annual (pre-launch + annual thereafter) | External firm; full scope | security-engineer |

### 9.2 Alert Verification Procedure (Monthly)

```
1. Schedule: first Monday of each month, 10:00–11:00 UTC (low-traffic window)
2. For each alert source, trigger a test condition:
   a. PagerDuty: use PagerDuty's built-in "Send Test Notification"
   b. Cloudflare WAF: trigger a WAF rule with a test request from a known IP
      curl -H "X-Test-WAF: true" https://api.form.coach/health (configure WAF rule for this)
   c. Uptime monitor: temporarily point monitor at a non-existent URL; verify alert fires;
      restore within 5 minutes
   d. HMAC cron: run the chain verification script manually; verify it completes and
      produces a success log entry in audit_log
3. Document results in Linear ticket labeled "alert-test-YYYY-MM"
4. Any alert that fails to fire is a P1 — fix before next business day
```

### 9.3 Tabletop Exercise Scenario Catalog

Run one of these per quarter, rotating through scenarios. Document findings in a tabletop PIR.

**Scenario A — Cross-tenant RLS bypass (Q1):**
"A customer reports they can see another company's employee workout data in their admin dashboard. It's 02:00 UTC on a Saturday."

**Scenario B — Supabase service role key leaked in git (Q2):**
"GitHub secret scanning fires an alert: a Supabase service role key was committed 3 hours ago in a PR that has since been merged and deployed."

**Scenario C — Full Supabase outage (Q3):**
"Supabase has declared a P1 incident affecting your region. ETA unknown. Your enterprise customers are calling."

**Scenario D — HMAC chain break with concurrent unusual data access (Q4):**
"The weekly chain integrity cron fires at 03:00 UTC. Simultaneously, PostHog shows an anomalous spike in data export events over the past 6 hours."

**Scenario E — Supply chain: popular npm package backdoored (Year 2, Q1):**
"GitHub Advisory publishes a critical advisory: `@upstash/redis` v1.34.2 (released 18 hours ago) contains an obfuscated payload that calls home to an external IP with the contents of `process.env`. Your Dependabot alert fires at 07:15 UTC. npm ls confirms FORM's Cloudflare Workers bundle includes this version. The Workers have been serving production traffic since the package was deployed yesterday morning."

Key discussion points: Has the malicious code executed? What was accessible in the Workers environment? How quickly can you confirm scope? What needs to rotate? What do enterprise tenants need to know, and when?

**Scenario F — Ransomware discovered in staging, lateral movement suspected (Year 2, Q2):**
"A developer messages the team at 14:30 UTC: their local checkout of the monorepo shows files being replaced with encrypted versions and a ransom note. Staging database on Supabase shows 40,000 rows deleted in the past 2 hours. The developer's machine is on the company VPN. They have access to the production service role key, which is stored in their local `.env` file."

Key discussion points: Is production compromised? What is the isolation sequence? What is the PITR target? What do enterprise tenants need to know before production goes offline deliberately? Who makes the call to take the platform down?

### 9.4 Year 1 Testing Schedule

| Month | Activity |
|---|---|
| Month 1 | Alert verification setup; verify all PagerDuty routes fire |
| Month 2 | First alert verification run (monthly cadence begins) |
| Month 3 | Tabletop exercise — Scenario A (RLS bypass) |
| Month 4 | Alert verification |
| Month 5 | Alert verification |
| Month 6 | Tabletop exercise — Scenario B (leaked credentials) + Fire drill #1 |
| Month 7 | Alert verification |
| Month 8 | Alert verification |
| Month 9 | Tabletop exercise — Scenario C (Supabase outage) |
| Month 10 | Alert verification |
| Month 11 | Alert verification |
| Month 12 | Tabletop exercise — Scenario D (chain break) + Fire drill #2 + Annual DR test |
| Month 12 | External pen test (pre-SOC 2 observation period close) |

### 9.5 Year 2 Testing Schedule

Expand from 4 scenarios to 6; add supply chain and ransomware to the rotation. Cross-functional drills begin — enterprise customer-success participates in Scenarios E and F.

| Month | Activity |
|---|---|
| Month 13 | Alert verification + review Year 1 PIR findings; update runbooks |
| Month 14 | Alert verification |
| Month 15 | Tabletop exercise — Scenario E (supply chain npm backdoor); include customer-success in drill |
| Month 16 | Alert verification |
| Month 17 | Alert verification |
| Month 18 | Tabletop exercise — Scenario F (ransomware + lateral movement) + Fire drill #3 |
| Month 19 | Alert verification |
| Month 20 | Alert verification |
| Month 21 | Tabletop — Scenario A variation (RLS bypass in enterprise white-label context) |
| Month 22 | Alert verification |
| Month 23 | Alert verification |
| Month 24 | Annual DR test (PITR full restore to new project) + External pen test year 2 + Tabletop — choose based on Year 1 PIR gaps |

---

## 10. Regulatory Obligations Summary

### 10.1 GDPR Art. 33 — Notification to Supervisory Authority

**What triggers it:** Any breach of security leading to the accidental or unlawful destruction, loss, alteration, unauthorized disclosure of, or access to, personal data transmitted, stored, or otherwise processed — **unless the breach is unlikely to result in a risk to the rights and freedoms of natural persons.**

In practice for FORM: any incident involving unauthorized access to or disclosure of user health data, workout data, meal data, email addresses, or any other stored user PII triggers Art. 33 notification. The exemption (no notification if no risk) applies only to incidents where, for example, encrypted data was lost and the encryption key was not compromised. When in doubt, notify.

**72-hour clock starts:** At the moment the controller (FORM) becomes aware of the breach — not when the breach is confirmed or fully scoped. Awareness of a probable breach is sufficient to start the clock.

**What to include in the Art. 33 notification:**
- Nature of the breach (categories and approximate number of data subjects and records)
- Name and contact details of the DPO or security contact
- Likely consequences of the breach
- Measures taken or proposed to address the breach and mitigate its effects

**If full information is not available within 72 hours:** File a partial notification and state that further information will follow. Supplementary notifications are explicitly permitted by Art. 33(4). Filing incomplete-but-timely is strongly preferred over filing complete-but-late.

**Lead supervisory authority:** Determined by FORM's main establishment in the EU. Until a main establishment is determined, notify the authority in each member state where affected data subjects are located.

### 10.2 GDPR Art. 34 — Communication to Data Subjects

**When mandatory:** When the breach is likely to result in a **high risk** to the rights and freedoms of the affected individuals. High risk indicators include: health data exposed, financial data exposed, data that could enable identity theft, data that could enable discrimination.

**When discretionary (but strongly recommended):** When the risk is medium — e.g., email addresses only, no sensitive categories.

**When not required:** When the data was encrypted and the key was not compromised (technical measure renders data unintelligible), or when individual notification would involve disproportionate effort (mass breach) and a public communication is made instead.

**Timing:** Without undue delay. In practice, as soon as the communication can be prepared accurately — typically within 72 hours of the Art. 33 notification, sometimes concurrently.

**Form:** Clear and plain language. Specific about what happened. Specific about what data. Specific about what to do. Contact details for follow-up.

### 10.3 SOC 2 Control Mapping

This document is formal evidence for the following SOC 2 Trust Service Criteria:

| Criteria | Description | Where This Doc Provides Evidence |
|---|---|---|
| **CC7.2** | The entity monitors system components for anomalies that could indicate a malicious act, a natural disaster, or an operational error | §3 (Detection and Alerting) — alert sources, escalation matrix, PagerDuty routing |
| **CC7.3** | The entity evaluates security events to determine whether they could or have resulted in a failure of a commitment or requirement | §1 (Incident Classification) — severity matrix and criteria; §4.1 (Triage phase) |
| **CC7.4** | The entity responds to identified security incidents by executing a defined incident response program | §4 (Incident Response Lifecycle); §5 (Runbooks R-01 through R-06) |
| **CC7.5** | The entity identifies, develops, and implements activities to recover from identified security incidents | §4.4 (Recover phase); §8 (Post-Incident Review) — including action item tracking |
| **CC9.2** | The entity assesses and manages risks associated with vendors and business partners | §3.1 (alert sources include third-party dependencies); §10 (regulatory obligations for processor incidents); R-01 includes third-party processor notification |

**Auditor evidence package for CC7 series:**
- This document (version-controlled in git)
- PagerDuty escalation policy configuration export (quarterly)
- Incident channel archives for all P0/P1 events in observation period
- PIR documents for all P0/P1 events (see §8)
- Action item completion evidence from Linear (closed tickets)
- Alert test results (monthly verification records)
- Tabletop exercise records (quarterly)

---

## 11. Sub-processor Incident Management

### 11.1 Framework Purpose

FORM is the **data controller** under GDPR Art. 4(7) for all personal data it collects and processes, including data processed by sub-processors on FORM's behalf. Sub-processor incidents are operationally distinct from self-originated incidents (R-01 to R-06): FORM does not control the pace of the processor's investigation, cannot contain the breach at source, and must derive its own Art. 33/34 assessment from second-hand information.

This section provides the standing framework that governs R-07 execution: the sub-processor registry, Art. 28 contractual requirements, and the vendor management integration required for SOC 2 CC9.2 evidence.

**Privacy floor applies to processors too.** A processor breach does not grant license to expand data analysis. FORM's scope assessment is limited to: *which of our users may have been affected, and what data categories?* FORM does not attempt to correlate or reconstruct individual user data from a processor's breach report. HR-restricted data (individual health profiles, coaching turns) remain HR-restricted throughout incident response.

### 11.2 Sub-processor Registry

**Last reviewed: May 2026 · Next review: November 2026 · Owner: compliance-officer + security-engineer**

| Processor | Purpose | Data categories held for FORM | Art. 9 data? | DPA in place? | Security notification contact | FORM severity floor |
|---|---|---|---|---|---|---|
| **Supabase** | Primary database + auth | All user data: health profiles, workouts, meal logs, coaching turns, auth tokens, PII | ✅ Yes | ✅ via Supabase DPA | security@supabase.io; Supabase dashboard ticket | **P0** |
| **Anthropic** | Victor AI coaching API | Coaching conversation content (system prompt + user turns contain health-adjacent context) | ✅ Probable | ✅ via Anthropic enterprise API DPA; notification clause under review — see gap below | security@anthropic.com; account team | **P0** |
| **ElevenLabs** | Voice synthesis for coaching cues | Text coaching cue strings (health-adjacent language possible) | 🟡 Possible | 🟡 DPA required before production use — P1 action item | security@elevenlabs.io | **P1** |
| **Cloudflare** | API gateway, CDN, Workers runtime, KV | Request logs (IP, UA, URL), auth tokens in-flight, API keys in Workers KV | 🟡 Auth tokens only; no health data at rest | ✅ via Cloudflare DPA | Cloudflare dashboard security ticket | **P1** (token exposure → **P0**) |
| **PostHog** | Product analytics | Pseudonymous events (user_id hashed; no raw PII in standard instrumentation per OBSERVABILITY.md §5) | 🔴 No — if instrumented correctly | ✅ via PostHog DPA (EU-hosted) | privacy@posthog.com | **P2** |
| **Sentry** | Error reporting | Error reports with stack traces — may contain health-adjacent data if scrubbing misconfigured | 🟡 Possible — depends on scrubber config | ✅ via Sentry DPA | security@sentry.io | **P1** (audit scrubbing config before downgrading) |
| **Stripe** | Payment processing | Payment method metadata; FORM never touches raw card data | 🔴 No health data | ✅ via Stripe DPA; PCI-DSS certified | security@stripe.com | **P1** for billing exposure |
| **Apple** (App Store / APNs) | App distribution + push notifications | App Store receipts, APNs device tokens | 🔴 No | Platform terms | App Store Connect security workflow | **P2** |
| **Google** (Play Store / FCM) | App distribution + push notifications | Play receipts, FCM device tokens | 🔴 No | Platform terms | Play Console security workflow | **P2** |

**Known gaps as of May 2026:**
- **Anthropic DPA notification clause:** Art. 28(3)(f) language is present but the sub-24h notification SLA is not explicitly enumerated. compliance-officer to obtain explicit SLA commitment at next contract renewal. Risk: interim Art. 33 assessment relies on Anthropic's voluntary cooperation speed. Mitigant: Anthropic holds Art. 9-adjacent data; treat any Anthropic breach as P0 regardless of their initial classification.
- **ElevenLabs DPA:** Not finalized before production. P1 action item — must be signed before the first ElevenLabs API call in production. Until signed, ElevenLabs must be treated as an unauthorized processor, and legal must be notified.
- **Sentry scrubbing:** Unverified in production. A Sentry error report during a health data API call could capture health-adjacent data in `request.body` or response objects. Verification task open in Linear.

**Annual registry review checklist:**
```
[ ] Confirm DPA is signed and current for every processor that touches GDPR Art. 9-adjacent data
[ ] Verify security notification contacts are active (test by sending a test inquiry)
[ ] Review whether any processor's data scope has expanded (new API endpoints, new data categories)
[ ] Confirm EU data residency for processors holding health data
    (Supabase: EU region; PostHog: EU Cloud; Anthropic: US — verify DPA adequacy decision)
[ ] Review ElevenLabs and Anthropic DPA notification SLA gaps
[ ] Update Sentry scrubbing status
[ ] Log registry review in audit_log:
    { action: 'system.access_review_completed', resource_type: 'vendor_registry',
      metadata: { review_date, reviewer, gaps_identified: N } }
```

### 11.3 Art. 28 Contractual Requirements for FORM Sub-processors

Under GDPR Art. 28(3)(f), processors must *"assist the controller in ensuring compliance with the obligations pursuant to Articles 32 to 36."* In practice, this requires each FORM sub-processor DPA to commit to:

**1. Breach notification** — notify FORM "without undue delay" upon discovering a breach involving FORM data. Regulators interpret "without undue delay" as ≤ 24 hours for confirmed high-severity incidents; ≤ 48 hours maximum. Any DPA that does not specify a notification SLA is a gap risk.

**2. Cooperation** — provide FORM with sufficient information to assess whether Art. 33/34 obligations are triggered: nature of breach, data categories involved, approximate number of data subjects, recommended measures.

**3. Security measures** — implement appropriate technical and organizational measures per Art. 32, proportional to the risk of the data they hold.

**4. Sub-processor restrictions** — not engage a sub-sub-processor without FORM's prior or general written approval, with notification mechanism for additions.

**DPA signing checklist:**
```
[ ] Art. 28(3)(f) language is explicit — not implied by general compliance clauses
[ ] Notification timeline is enumerated (< 48h from processor's discovery — not "without undue delay" only)
[ ] Security notification contact is specified in the DPA or its annex
[ ] Data processing scope annex accurately lists all data categories the processor will access
[ ] Sub-processor list disclosed; mechanism for FORM approval of additions
[ ] Governing law and DPA dispute resolution clause
[ ] Data deletion / return obligations on contract termination
[ ] For Art. 9 processors: explicit acknowledgment of special category data in annex
```

### 11.4 Processor Notification SLA Tracker

Track sub-processor notifications against contractual and Art. 28(3)(f) obligations. SLA misses feed into the annual vendor review and SOC 2 CC9.2 evidence.

| Processor | DPA notification SLA | Last incident | Notified within SLA? | Notes |
|---|---|---|---|---|
| Supabase | ≤ 48h from discovery | — | N/A | No incidents to date |
| Anthropic | Not explicitly enumerated (gap — see §11.2) | — | N/A | Risk accepted with interim mitigant; renewal target |
| ElevenLabs | N/A (DPA not finalized) | — | N/A | **P1 action item — DPA required before production** |
| Cloudflare | ≤ 72h from discovery | — | N/A | No incidents to date |
| PostHog | ≤ 72h from discovery | — | N/A | No incidents to date |
| Sentry | ≤ 72h from discovery | — | N/A | Scrubbing verification open |
| Stripe | ≤ 72h from discovery | — | N/A | PCI-DSS independent audit provides additional assurance |

**SLA miss consequence:** Any processor failing to notify within the DPA-committed SLA triggers:
1. compliance-officer formal written notice to the processor within 5 business days of discovering the miss
2. Mandatory inclusion in the next SOC 2 vendor risk review
3. Assessment of whether to continue using the processor for Art. 9 data processing

### 11.5 SOC 2 CC9.2 Integration

SOC 2 Trust Service Criterion CC9.2 requires: *"the entity assesses and manages risks associated with vendors and business partners."*

**Evidence this framework provides:**

| Evidence Type | Source | Storage Location | Frequency |
|---|---|---|---|
| Processor registry with risk classification | §11.2 (this doc) | SOC 2 evidence folder: `CC9.2/Vendor-Registry/<year>` | Annual review |
| DPA signing confirmation | Processor contract repository | `CC9.2/DPAs/<vendor>` | On signing + renewal |
| Notification SLA tracker | §11.4 (this doc) | Updated in-place; SOC 2 snapshot at audit | Ongoing |
| Annual registry review record | §11.2 checklist completion + audit_log entry | `CC9.2/Vendor-Registry/<year>/review-complete.md` | Annual |
| R-07 PIR documents | §8 PIR template | `CC9.2/Processor-Incidents/<YYYYMMDD>-<vendor>` | Per incident |

**Auditor package for CC9.2:**
- This document (version-controlled in git)
- Annual registry review completion record with audit_log entry
- DPA copies for all processors handling Art. 9-adjacent data
- R-07 PIR documents for any processor incidents in the observation period
- Notification SLA tracker with any miss disposition documentation

---

## 12. Enterprise Tenant SLA Breach & Incident Communication Protocol

### 12.1 Purpose

Enterprise contracts carry explicit SLA commitments (typically 99.9% monthly uptime on the API gateway). When an incident breaches or imminently threatens those commitments, enterprise tenants have contractual rights to: (a) direct notification from a named contact, (b) SLA credit calculation, and (c) a formal post-incident summary. This section defines the operational protocol so that the customer-success and communications functions have a playbook that does not require IC judgment calls in the middle of an active incident.

This section supplements §6 (general communication templates) — it covers the enterprise-specific layer on top.

### 12.2 SLA Breach Definition

**Standard enterprise SLA tier (unless contract specifies otherwise):**

| Metric | SLA target | Breach threshold |
|---|---|---|
| API gateway monthly availability | 99.9% | Any calendar month ending below 99.9% (allows ~44 min/month downtime) |
| API gateway P95 latency | < 500 ms | Sustained P95 > 500 ms for > 30 consecutive minutes |
| SSO/SAML login availability (per tenant) | 99.5% | Any calendar month ending below 99.5% |
| Admin dashboard availability | 99.5% | Any calendar month ending below 99.5% |

**Downtime definition for SLA purposes:** Availability is measured from the Better Stack uptime probe (endpoint `/health`). Scheduled maintenance with ≥ 72 hours advance notice is excluded. Incidents caused by third-party infrastructure (Supabase, Cloudflare platform) count unless FORM has a compensating control (failover, degraded mode) that it did not activate.

**SLA credit schedule (standard):**

| Monthly uptime achieved | Credit as % of that month's invoice |
|---|---|
| 99.0% – 99.9% | 10% |
| 95.0% – 99.0% | 25% |
| < 95.0% | 50% |

Credits are applied to the next invoice. Credits do not compound. Maximum credit in any calendar month is 50% of that month's invoice. Credits are the sole and exclusive remedy for SLA breaches unless the contract specifies otherwise — consult legal before making any additional commitments during an incident.

### 12.3 Who Contacts Enterprise Tenants — and When

**The Customer Lead** (as defined in §2.1) is the primary point of contact for all enterprise tenants during P0 and P1 incidents. The Customer Lead does not improvise — they use the templates in §12.5 and send only after IC approval.

| Incident phase | Who sends | Channel | Timing |
|---|---|---|---|
| Incident declared (P0/P1) | Customer Lead via dedicated CSM | Direct email to tenant admin contact + Slack if shared workspace | Within 30 min of P0 declaration; within 60 min of P1 declaration |
| Status update | Customer Lead | Same channels | Every 60 min for P0; every 2 hours for P1 |
| Resolution | Customer Lead | Email + status page | Within 2 hours of resolution declaration |
| Formal PIR summary | Customer Lead + IC | Email (written, not verbal) | Within 5 business days of resolution |
| SLA credit notification | Customer-success + legal | Email + next invoice annotation | Within 10 business days of month-end if breach confirmed |

**Do not:** route enterprise tenant communications through the public status page alone. Enterprise customers require direct personal contact. The status page is supplemental, not a substitute.

**Tenant admin contact registry:** Maintained in the CRM (customer-success owns). Before any incident is declared, the on-call Customer Lead must be able to retrieve the primary and secondary contact for each enterprise tenant within 5 minutes. If the registry is not populated for a tenant, that is a P1 action item for customer-success, independent of any incident.

### 12.4 Privacy Floor During Enterprise Incident Communication

Under the FORM privacy floor (see `docs/ENTERPRISE.md`): HR and company admins **never** see individual user health data, even during an incident. This holds during incident communication.

**What tenant admins are told:**
- Whether their tenant's data was in scope for a breach
- How many user records were affected (aggregate count only, no names)
- What data categories were involved (e.g., "workout session logs" — not specific user health details)
- What remediation steps were taken
- Whether they need to take any action (e.g., re-authenticate users, update SSO configuration)

**What tenant admins are NOT told:**
- Individual user names, health profiles, or coaching session contents
- Details about other tenants affected by the same incident
- Internal FORM system architecture details beyond what is necessary for the tenant to take action

### 12.5 Enterprise Incident Notification Templates

#### Template E-01: Initial P0/P1 Notification to Tenant Admin

```
Subject: [FORM — Service Incident] Impact to [Tenant Name] — [Incident ID] — [Date]

[Tenant Admin Name],

We are writing to inform you that FORM is currently experiencing a service
incident that affects your organization's access to the platform.

Incident Reference: [INC-YYYYMMDD-slug]
Severity: [P0 / P1]
Detected: [HH:MM UTC, Date]
Status at time of this email: [Investigating / Containing / Recovering]

CURRENT IMPACT TO YOUR ORGANIZATION:
[One to three sentences describing confirmed impact to this tenant specifically.
Example: "Users at [Tenant Name] are unable to log into the FORM platform via
your SSO configuration. Workout logging and AI coaching features are unavailable
for authenticated users."]

WHAT WE ARE DOING:
[1–2 sentences on active steps being taken.]

WHAT YOU MAY NEED TO DO:
[Specific action for the tenant admin — or "No action required from your side
at this time."]

NEXT UPDATE:
We will contact you again by [HH:MM UTC] or sooner if the situation changes.

Your dedicated contact for this incident: [Customer Lead name, direct contact]

We understand the impact this has on your team and are treating this as our
highest priority.

[Customer Lead Name]
Customer Success, FORM
```

#### Template E-02: Status Update (60-min cadence for P0)

```
Subject: [UPDATE #N] FORM Service Incident [INC-YYYYMMDD-slug] — [HH:MM UTC]

[Tenant Admin Name],

Update #[N] on the ongoing incident affecting [Tenant Name].

Status: [Investigating / Containing / Eradicating / Recovering]
Time elapsed: [X hours Y minutes since incident declared]

WHAT WE KNOW NOW:
[2–4 bullet points of confirmed new information since last update.
Do not speculate. Do not include information about other tenants.]

CURRENT IMPACT:
[Confirmed impact to this tenant at this moment.]

ESTIMATED RESTORATION:
[Best estimate, explicitly labeled as an estimate, or "No ETA — we will
update you as soon as we have one." Do not over-promise.]

NEXT UPDATE: [HH:MM UTC]

[Customer Lead Name]
```

#### Template E-03: Resolution Notification

```
Subject: [RESOLVED] FORM Service Incident [INC-YYYYMMDD-slug]

[Tenant Admin Name],

The service incident affecting [Tenant Name] has been resolved.

Resolved at: [HH:MM UTC, Date]
Total duration: [X hours Y minutes]

SUMMARY OF IMPACT TO YOUR ORGANIZATION:
- Features affected: [list]
- Duration of user-visible impact: [X hours Y minutes]
- Data impact: [e.g., "No data was lost or exposed for your organization"
  OR "X workout sessions from [date range] may need to be re-logged —
  a full account of affected records will be in the formal PIR."]

ROOT CAUSE (preliminary):
[One paragraph — what went wrong, in plain language. Not a technical dissertation.
Do not assign blame to team members or third parties by name.]

WHAT WE ARE DOING TO PREVENT RECURRENCE:
[2–3 bullet points. Specific, actionable, and honest about timeline.]

FORMAL POST-INCIDENT REVIEW:
You will receive a written post-incident summary within 5 business days.
If this incident triggered an SLA credit under your contract, you will
receive confirmation with your next invoice.

Thank you for your patience. If you have any questions, please contact
[Customer Lead Name] directly.

[Customer Lead Name]
Customer Success, FORM
```

#### Template E-04: SLA Credit Notification (sent with next invoice)

```
Subject: SLA Credit Applied — [Month Year] Invoice — [Tenant Name]

[Tenant Admin Name],

This message confirms that an SLA credit has been applied to your [Month Year]
invoice in connection with the service incident on [incident date].

Incident Reference: [INC-YYYYMMDD-slug]
Downtime duration (for SLA calculation): [X hours Y minutes]
Calculated monthly uptime: [XX.X%]
Credit applied: [X%] of the [Month] invoice = $[amount]
Reflected on invoice: [invoice number / date]

Your SLA credit has been applied automatically. No action is required.

If you have questions about this calculation or believe the downtime duration
is incorrect, please reply to this email or contact your CSM within 30 days.

[Customer Lead Name / Finance contact]
FORM
```

### 12.6 SOC 2 Evidence from §12

Enterprise incident communication generates the following SOC 2 evidence:

| SOC 2 Criterion | Evidence from §12 |
|---|---|
| CC9.2 — Vendor risk and customer commitments | Tenant admin contact registry; SLA credit log |
| A1.1 — Availability commitments | SLA definition (§12.2); credit schedule |
| A1.2 — Availability monitoring | Better Stack availability reports used for SLA calculation |
| CC7.3 — Communication to affected parties | Templates E-01 through E-04; sent-message archive |
| CC2.2 — Internal communications | Incident channel pinned updates + Customer Lead email thread |

**Evidence storage:** All Template E-01 through E-04 communications are archived in `compliance/evidence/incident-comms/<INC-YYYYMMDD-slug>/` and referenced in the PIR document (§8). The email archive + PIR constitute the auditor package for any observation-period incident involving enterprise tenants.

---

## 13. Board & Investor Communication Protocol

### 13.1 Purpose and Trigger Threshold

This section governs communication to FORM's board members and lead investors during security and availability incidents. Board-level communication is distinct from enterprise tenant communication (§12) and regulatory notification (§10). It is forward-looking and strategic; regulatory notifications are legal obligations with fixed formats.

**Activate §13 when ANY of the following are true:**

| Trigger | Rationale |
|---|---|
| P0 incident confirmed and not resolved within 2 hours | Material service disruption; board should not be surprised by press or investor questions |
| Confirmed Art. 9 biometric or health data breach (R-01, R-11) | Material privacy event; investor rights under typical SAFE/equity agreements |
| Enterprise tenant loss directly attributable to an incident | Revenue impact; relevant to board's fiduciary view |
| Incident generates media coverage or social media attention | Reputational; board must have context before receiving external inquiries |
| Regulatory action initiated (supervisory authority investigation) | Legal and compliance materiality |
| Estimated financial impact > $10,000 (credits, legal fees, remediation) | Financial materiality threshold at seed stage |

**Do NOT activate §13 for:**
- P1 or P2 incidents resolved without customer impact
- Routine maintenance windows or planned degradation
- False positive alerts

### 13.2 Authority and Ownership

**The founder is the sole person authorized to send board/investor communications.** No one else — including the IC (Incident Commander), security-engineer, or customer-success — may contact board members or investors about an active incident without explicit founder approval.

During an incident, the founder plays dual roles: executive decision-maker and board communicator. If the founder is the IC, they must hand off IC duties to the most senior available responder before drafting board communications. IC responsibilities and board communication responsibilities must not be handled simultaneously.

### 13.3 Communication Timeline

| Milestone | Action | Deadline |
|---|---|---|
| §13 trigger activated | Founder acknowledges board communication responsibility | T+30 min from trigger |
| Initial board notification | Send Template B-01 | T+4h from P0 declaration |
| First status update | Send Template B-02 | T+12h if incident unresolved |
| Subsequent updates | Send Template B-02 (repeat) | Every 12h while P0 is active |
| Resolution notification | Send Template B-03 | Within 4h of incident closure |
| Post-incident board summary | Send Template B-04 | Within 5 business days of PIR completion |

### 13.4 Communication Templates

#### Template B-01: Initial Board Notification

Subject: `[FORM Incident] [Incident ID] — Initial Notification`

```
Board / [Investor name],

I'm writing to flag an active incident at FORM.

Incident ID: [INC-YYYYMMDD-slug]
Declared: [timestamp UTC]
Severity: P0
Status: Active — investigating

What we know:
[2–3 sentences. State facts only: what system is affected, what behavior was
observed, when we became aware. No speculation, no root cause claims.]

What we're doing:
[1–2 sentences. State the current action: investigating, contained, rolling back.]

What we don't know yet:
[1 sentence. State the key open question: scope of exposure, cause, ETA to resolve.]

User impact:
[State the observable impact: "Service unavailable for all users" /
"Degraded performance on enterprise features" / "No current user-facing impact —
investigating potential data exposure." Do not minimize, do not catastrophize.]

Regulatory:
[State whether GDPR Art. 33 clock is running, if known. If uncertain: "Assessing."]

Next update: [timestamp UTC or condition: "within 12 hours, or sooner if material change"]

I'll continue to keep you informed. Reach me at [phone] for urgent questions.

[Founder name]
```

**What NOT to write in B-01:**
- Root cause (not confirmed yet — premature statements create legal exposure)
- Customer names or tenant identifiers
- Specific dollar amounts (revenue impact, credit exposure)
- Attribution (external attacker, vendor failure, internal error)
- Legal conclusions ("we do/don't believe this triggers GDPR notification")
- Speculation about press coverage or competitive impact

#### Template B-02: Status Update

Subject: `[FORM Incident] [Incident ID] — Update [N]`

```
Board / [Investor name],

Update [N] on incident [INC-YYYYMMDD-slug] — [elapsed time] since declaration.

Status: [Active / Contained / Recovering]

What's changed since last update:
[Facts only. What was confirmed, what was ruled out, what action was taken.]

Current status of affected systems:
[One sentence per affected system.]

Regulatory:
[Update on Art. 33 status if applicable: "Art. 33 notification filed at [timestamp]" /
"Assessing — will update within [timeframe]."]

Revised ETA to resolution: [estimate or "unknown — will update in [timeframe]"]

Next update: [timestamp or condition]

[Founder name]
```

#### Template B-03: Resolution Notification

Subject: `[FORM Incident] [Incident ID] — Resolved`

```
Board / [Investor name],

Incident [INC-YYYYMMDD-slug] is resolved.

Resolved at: [timestamp UTC]
Total duration: [X hours Y minutes]

Summary:
[3–5 sentences. What happened (high level), what the impact was,
what we did to resolve it, what we're doing to prevent recurrence.
Facts only — no blame attribution, no speculative root cause.]

Customer impact:
[Was any customer-facing SLA breached? Enterprise tenants notified?
Credits to be issued? State factually.]

Regulatory:
[Art. 33 notification filed: [Y/N, date] / Not required / Under assessment]

Post-incident review:
A full post-incident review will be completed within 72 hours.
I will share the summary findings with the board after the review.

[Founder name]
```

#### Template B-04: Post-Incident Board Summary

Subject: `[FORM Incident] [Incident ID] — Post-Incident Summary`

Attach the post-incident review document (§8.3) with commercial-sensitive details redacted (do not attach if it contains specific vulnerability details that could assist future attackers).

```
Board / [Investor name],

Following up on incident [INC-YYYYMMDD-slug] with the post-incident summary.

What happened (technical summary — 1 paragraph):
[Plain-language summary suitable for a non-technical reader. Avoid jargon.]

Impact summary:
- Users affected: [count or range]
- Downtime: [duration, if applicable]
- Enterprise tenants affected: [count; no names in written comms]
- SLA credits issued: [$ amount or "none"]
- Regulatory: [Art. 33 filed / not required; Art. 34 notifications sent if applicable]

Root cause: [Confirmed root cause in one sentence]

What we're changing:
[Bulleted list of 3–5 concrete actions from the PIR action items.
Link to the action items in Linear if board has access.]

What this means for [relevant OKR / investor concern]:
[One sentence connecting the incident to a known board-level concern, if relevant.
If not relevant, omit this section entirely.]

[Founder name]
```

### 13.5 Communication Channel

Board/investor notifications are sent by email to the individual's preferred contact. Do not use Slack, WhatsApp, or other messaging platforms for initial incident notifications — email provides a timestamped audit trail.

After the initial email, a phone call is appropriate if:
- The incident involves confirmed Art. 9 biometric or health data breach
- The incident is generating or likely to generate press coverage
- The incident involves confirmed loss of an enterprise customer

### 13.6 Evidence and Archiving

All board/investor communications related to an incident are archived in:

```
compliance/evidence/incident-comms/<INC-YYYYMMDD-slug>/board/
  B-01-initial-notification-[timestamp].eml
  B-02-update-1-[timestamp].eml
  B-02-update-N-[timestamp].eml
  B-03-resolution-[timestamp].eml
  B-04-post-incident-summary-[timestamp].eml
```

These records are retained for 7 years (consistent with the HMAC audit log retention policy per DEC-030) and are included in the SOC 2 evidence package under CC2.2 (internal and external communication of information relevant to risk and control activities).

### 13.7 SOC 2 Evidence Mapping

| SOC 2 Criterion | Evidence from §13 |
|---|---|
| CC2.2 — Communication of information | Board notification email archive; B-04 post-incident summary |
| CC9.1 — Risk mitigation commitments | B-03 resolution notification; PIR action items communicated in B-04 |
| CC7.3 — Response to identified anomalies | B-01 through B-03 timeline shows timely escalation to governance |
| A1.1 — Availability commitments | B-03 duration and impact statement cross-referenced to SLA |

---

### R-12: Insider Threat / Privileged Access Abuse

**Definition:** A security incident in which a current or former FORM team member — or a contractor with legitimate access credentials — intentionally misuses their access to exfiltrate data, sabotage systems, or abuse permissions for personal gain, competitive intelligence, or coercion.

This runbook is architecturally distinct from external credential compromise (R-03): the actor started with legitimate access. Standard containment (immediate revocation) may alert the actor, accelerate data destruction, or contaminate the investigation. Evidence preservation MUST precede containment except when live exfiltration is actively in progress.

---

#### R-12.1 Trigger Matrix

| Signal | Source | Initial Severity |
|---|---|---|
| Bulk data export via Supabase Studio during off-hours | Supabase audit log / `audit_log_events` HMAC chain | P1 pending scope |
| `service_role` key used from IP outside known developer ranges | Cloudflare Workers access log | P1 pending scope |
| Admin dashboard accessed by team member who is in HR disciplinary process | HR tip to security-engineer or founder | P1 |
| Sudden access to tables/RLS policies outside normal role | `audit_log_events` (action: `pg.read`, unusual `resource_type`) | P1 |
| Termination notice issued → credential revocation NOT yet completed | HR → security-engineer standard offboarding check | P2 (escalates if access confirmed post-termination) |
| Whistleblower report alleging data misuse by team member | Direct report to founder or compliance-officer | P1 |
| HMAC audit chain break co-incident with staff admin activity | §5 R-05 + this runbook cross-activate | **P0** |
| Former contractor requests re-use of old API key | Customer-success or platform-engineer reports | P2 |
| Access log shows download of `health_profiles`, `coaching_turns`, or `cv_sessions` at volume | Supabase log analysis | P0 if Art. 9 data confirmed |

**Do not trigger this runbook for:**
- External actor using stolen staff credentials → R-03 (possible concurrent activation)
- Misconfigured automation or CI/CD pipeline accessing data unexpectedly → standard debugging
- Developer accessing their own user data in production for debugging → standard access review

---

#### R-12.2 Severity Matrix

| Severity | Criteria |
|---|---|
| **P0** | Confirmed exfiltration of Art. 9 data (health profiles, coaching turns, CV keypoints) at any volume; cross-tenant data accessed by staff credential; HMAC chain tampered by insider; live exfiltration in progress |
| **P0** | Art. 9 data accessed by staff credential from outside normal working hours AND volume > 100 records, until ruled out |
| **P1** | Bulk access (> 500 records, any category) without confirmed exfiltration; anomalous privilege use (e.g., RLS policy alteration); access after HR process begins |
| **P1** | Former team member retains active credentials (any environment) |
| **P2** | Single anomalous access event, no sensitive data, historical (> 7 days ago), no active session |

**Upgrade triggers:** Any access to `health_profiles`, `coaching_turns`, `cv_sessions`, or `tenant_admins` — regardless of volume — is an automatic P1 floor. Discovery of data uploaded to external storage is an automatic P0.

---

#### R-12.3 Unique Response Constraints

These constraints override the standard incident response defaults for this runbook only:

**1. Evidence before containment (reinforced)**
Unlike R-01 (active exfiltration exception) and R-09 (isolation-first for ransomware), insider threat investigations require forensic snapshots BEFORE the actor is aware of the investigation. Premature revocation may cause the actor to:
- Accelerate data exfiltration before access expires
- Delete or alter local copies of exfiltrated data
- Alert co-conspirators (if any)
- Trigger legal claims of wrongful termination if the investigative basis is not documented

Do not revoke access until legal counsel has been consulted AND evidence has been preserved, unless live exfiltration is active or destruction of evidence is imminent.

**2. Restricted incident channel**
This runbook uses a **private investigation channel** (`#inc-YYYYMMDD-insider`), accessible only to:
- Founder
- security-engineer
- compliance-officer
- Legal counsel (if retained)

Do NOT add the HR lead, other team members, or the subject to this channel at any stage of the investigation. Customer-success is notified only if enterprise tenant data is confirmed affected, and then only via the standard E-01 template (§12) — the cause is disclosed as "internal access control incident" without naming the individual.

**3. Employment law jurisdiction**
FORM is subject to employment law that varies by jurisdiction. Key differences that affect evidence collection and monitoring rights:
- **EU/GDPR jurisdiction:** Staff monitoring rights under GDPR Art. 6(1)(f) (legitimate interest for security) exist, but the monitoring must be proportionate and pre-disclosed in the employment contract or acceptable use policy. Bulk retrospective log access for a specific individual is proportionate for a security investigation.
- **US jurisdiction:** At-will employment; monitoring of company systems is generally permitted if disclosed.
- **Ukraine jurisdiction:** Monitoring permitted for company systems under labor law Art. 31 equivalent; legal counsel required before using investigation findings in termination.

Rule: **Legal counsel before HR briefing, HR briefing before any contact with the subject.**

**4. HMAC audit log integrity**
The HMAC chain (DEC-030) is the forensic backbone. If the insider has `form_admin` role in the database, they may have attempted to alter audit log records. Always retrieve the immutable R2-archived copies (§7 in this document, §8.3 of OBSERVABILITY.md) rather than querying live `audit_log_events` for forensic purposes. The live table is for operational reference only during an insider investigation.

---

#### R-12.4 Immediate Actions (T+0 to T+30 min)

**Prerequisite:** Before taking ANY action, open `#inc-YYYYMMDD-insider` (private channel), add founder + security-engineer + compliance-officer ONLY. Post time-stamped opening message.

```
Step 1 — EVIDENCE SNAPSHOT (do NOT wait for scope assessment)

1a. Export R2-archived HMAC audit log for the past 90 days:
    aws s3 cp s3://form-audit-logs/audit_log_events/ \
      /tmp/insider-investigation/audit-export-$(date +%Y%m%d-%H%M%S)/ \
      --recursive --profile form-r2
    # SHA-256 the export directory immediately:
    find /tmp/insider-investigation/ -type f | sort | xargs sha256sum \
      > /tmp/insider-investigation/MANIFEST.sha256

1b. Export Supabase Studio access log for the subject's email from the dashboard.
    (Supabase Dashboard → Settings → Audit Logs → filter by user email)
    Save as PDF + raw JSON. SHA-256 both.

1c. Export Cloudflare Workers access logs for the suspected timeframe.
    Use Cloudflare dashboard → Workers → Logs.
    Filter by any relevant Worker routes (service_role key calls leave no Worker log,
    but REST API access via anon/service key does appear in gateway logs).
    Save + SHA-256.

1d. If the subject has access to Workers Secrets (Cloudflare dashboard):
    Note when each secret was last rotated. Do not rotate yet.

1e. Capture a point-in-time snapshot of the subject's Supabase role grants:
    -- Run as form_admin (read-only query — do NOT use in live channel)
    SELECT grantee, table_catalog, table_schema, table_name, privilege_type
    FROM information_schema.role_table_grants
    WHERE grantee = '<subject-db-role>';

    SELECT rolname, rolsuper, rolcreaterole, rolcreatedb
    FROM pg_roles WHERE rolname = '<subject-db-role>';

Step 2 — SCOPE ASSESSMENT (parallel with Step 1)

2a. Query audit log for subject's activity in the past 30 days:
    -- Use R2-archived copy, not live table
    SELECT event_type, resource_type, resource_id,
           tenant_id, created_at, ip_address, metadata
    FROM audit_log_events
    WHERE actor_id = '<subject-user-id>'
      AND created_at >= NOW() - INTERVAL '30 days'
    ORDER BY created_at DESC;

2b. Identify any access to sensitive tables:
    SELECT resource_type, COUNT(*) as access_count,
           MIN(created_at) as first_access, MAX(created_at) as last_access
    FROM audit_log_events
    WHERE actor_id = '<subject-user-id>'
      AND resource_type IN (
        'health_profiles', 'coaching_turns', 'cv_sessions',
        'users', 'tenant_admins', 'sso_configurations',
        'audit_log_events'  -- self-reference = red flag
      )
      AND created_at >= NOW() - INTERVAL '90 days'
    GROUP BY resource_type
    ORDER BY access_count DESC;

2c. Flag any access to audit_log_events as an actor — this indicates potential
    log tampering attempt. Cross-reference with HMAC chain integrity.

2d. Identify whether multi-tenant access occurred:
    SELECT DISTINCT tenant_id FROM audit_log_events
    WHERE actor_id = '<subject-user-id>'
      AND created_at >= NOW() - INTERVAL '90 days';
    -- More than one tenant_id = P0 cross-tenant access

Step 3 — CLASSIFY AND ESCALATE

3a. If Art. 9 data accessed → P0. Start GDPR Art. 33 clock.
3b. If multi-tenant access → P0. Enterprise tenant notification required (§12).
3c. If live access is currently active → proceed to §R-12.6 Containment.
3d. If no live access and investigation is early → hold containment pending legal counsel.

Step 4 — LEGAL COUNSEL NOTIFICATION

Within 30 min of classification:
    "We have a [P0/P1] internal access incident. Team member [role only, not name
    in written record until legally advised] may have accessed [data category].
    Requesting guidance on: evidence collection rights, monitoring scope, and
    process for HR involvement and potential termination."

Do NOT take employment action without explicit legal guidance.
```

---

#### R-12.5 Blast Radius Assessment

```sql
-- 1. Total records accessed by subject in exposure window
SELECT resource_type,
       COUNT(*) AS access_events,
       COUNT(DISTINCT resource_id) AS unique_records,
       COUNT(DISTINCT tenant_id) AS tenants_affected
FROM audit_log_events
WHERE actor_id = '<subject-user-id>'
  AND event_type IN ('read', 'list', 'export', 'download')
  AND created_at BETWEEN '<window_start>' AND '<window_end>'
GROUP BY resource_type;

-- 2. Check for any bulk export events (PostHog or audit)
SELECT event_type, metadata, created_at
FROM audit_log_events
WHERE actor_id = '<subject-user-id>'
  AND (event_type LIKE 'export%' OR metadata::text LIKE '%bulk%' OR metadata::text LIKE '%download%')
  AND created_at >= NOW() - INTERVAL '90 days'
ORDER BY created_at DESC;

-- 3. Identify enterprise tenants potentially affected
SELECT t.id, t.name, t.plan, COUNT(ale.id) AS access_events
FROM tenants t
JOIN audit_log_events ale ON ale.tenant_id = t.id
WHERE ale.actor_id = '<subject-user-id>'
  AND t.plan IN ('starter', 'growth', 'enterprise')
  AND ale.created_at >= NOW() - INTERVAL '90 days'
GROUP BY t.id, t.name, t.plan
ORDER BY access_events DESC;

-- 4. Cross-reference with health_profiles record count for impacted users
-- Use only if Art. 9 data access is confirmed
SELECT COUNT(DISTINCT hp.user_id) AS health_profiles_potentially_exposed,
       COUNT(DISTINCT hp.tenant_id) AS tenants_with_health_data_exposure
FROM health_profiles hp
WHERE hp.user_id IN (
  SELECT DISTINCT resource_id::uuid
  FROM audit_log_events
  WHERE actor_id = '<subject-user-id>'
    AND resource_type = 'health_profiles'
    AND created_at BETWEEN '<window_start>' AND '<window_end>'
);
```

**Multi-tenant exposure → mandatory enterprise notification.** Use §12 Template E-01 with cause: "internal access control incident involving a privileged credential." Do not name the individual. Legal counsel approves communication before dispatch.

---

#### R-12.6 Containment Options

Containment is graduated based on live access status and investigative stage. Apply the least-revealing option sufficient to stop harm.

| Option | Action | Reveals Investigation? | When to Use |
|---|---|---|---|
| **C-1 Passive monitoring** | No containment; observe audit log in real time | No | First 30 min while evidence is being gathered |
| **C-2 Session expiry** | Set session expiry to "now" in Supabase Auth → revokes session token on next API call | Partial (subject notices next API call fails) | Live access active; investigation evidence secured |
| **C-3 Role restriction** | Remove sensitive grants without disabling account | Yes — but access still works for permitted tables | Exfiltration of specific table type; legal advises no full revocation yet |
| **C-4 Full revocation** | Supabase Auth → disable user account + revoke all Supabase role grants + rotate Workers Secrets if applicable | Yes | Legal counsel authorises; termination decision made |
| **C-5 Workers Secrets rotation** | Rotate `SUPABASE_SERVICE_ROLE_KEY` and all other relevant secrets | Operational impact — all Workers redeploy | Only if subject had direct access to Workers Secrets |

**C-5 note:** Rotating `SUPABASE_SERVICE_ROLE_KEY` triggers a full Workers redeployment. Coordinate with devops-lead. Plan a 5-minute maintenance window if at all possible, but do not delay if live exfiltration via direct API key is confirmed.

**After full revocation (C-4):**
```
□ Supabase Auth user disabled (Dashboard → Users → Disable)
□ Database role grants revoked:
    REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM <subject_role>;
    REVOKE USAGE ON SCHEMA public FROM <subject_role>;
□ Workers Secrets verified as NOT accessible to subject's accounts
□ GitHub repository access removed (Settings → Collaborators)
□ Cloudflare dashboard access removed
□ Internal tooling (Linear, Slack, PostHog) deprovisioned
□ EAS / Expo build credentials access revoked if applicable
□ SSO IdP account suspended (if FORM is the SP in its own SSO)
□ Confirm no shared credentials the subject may have had knowledge of remain active
```

---

#### R-12.7 Legal and HR Interface

```
Timeline:
T+0         → Open private channel. Security-engineer + founder.
T+30 min    → Legal counsel notified. Do not proceed to HR without legal guidance.
T+Legal go  → HR Lead briefed by founder (verbal first; written summary follows).
             → Legal prepares termination documentation if warranted.
             → HR prepares separation agreement if warranted.
T+Containment → Subject notified of access revocation. This is the moment the
               investigation is revealed to the subject.
T+Post      → Formal written documentation to subject per employment law requirements.
             → If regulatory reporting required: legal counsel coordinates Art. 33.
```

**Legal hold notice (issued to IT/devops-lead as soon as investigation opens):**
> "Legal hold is in effect for all data related to [role descriptor, not name at this stage]. Do not delete, overwrite, or allow expiry of any logs, email records, access logs, or storage related to this matter. Retain all items regardless of normal retention schedules. This hold remains in effect until explicitly lifted in writing."

**Privacy floor applies during investigation:** The investigation accesses only system access logs and metadata. The content of coaching turns, health profiles, or workout data belonging to the subject qua user is accessed only if the subject's own data is within the blast radius of an exfiltration event, and then only by security-engineer with compliance-officer oversight and legal authorisation.

---

#### R-12.8 GDPR Implications

| Data Exfiltrated | GDPR Implication | Notification Required |
|---|---|---|
| None confirmed | No breach | No |
| Email addresses or names only | Art. 33 if > low risk; assess | Probable Art. 33; Art. 34 unlikely |
| Workout logs, meal logs | Art. 33 (health-adjacent) | Art. 33 required; Art. 34 assess |
| `health_profiles` (Art. 9 explicit) | Art. 33 mandatory; Art. 34 likely | Both required |
| `coaching_turns` (health context) | Art. 33 mandatory; Art. 34 assess | Art. 33 required; Art. 34 based on content review |
| `cv_sessions.keypoints_enc` (biometric) | Art. 9; R-11 cross-activate | Art. 33 mandatory; Art. 34 presumed (align with R-11) |
| Multi-tenant: multiple enterprise customers | Art. 33 per affected jurisdiction | Notify each relevant SA; enterprise tenants via §12 |

**72-hour clock starts** when the founder or security-engineer first becomes aware of a probable breach — the trigger is awareness, not confirmation.

**The subject's own GDPR rights:** The subject employee retains data subject rights under GDPR during and after the investigation. However, the right to access (Art. 15) may be limited where disclosure would prejudice an ongoing investigation (GDPR Art. 23(1)(f)). Legal counsel advises on the specific derogation available in the relevant jurisdiction.

**HR data handling:** Employment records created during this investigation (investigation notes, evidence summary, legal correspondence) are processed under GDPR Art. 6(1)(c) (legal obligation) and Art. 9(2)(b) (employment law) if health data about the employee is involved. These records are stored in a restricted compliance folder (`compliance/hr-investigations/<case-id>/`) accessible only to founder + legal + compliance-officer. Retention: 7 years post-employment end or as required by applicable employment law, whichever is longer.

---

#### R-12.9 Evidence Package

```
compliance/evidence/insider-<YYYYMMDD>-<case-id>/
  ├── 00-manifest.sha256              ← SHA-256 of all files at collection time
  ├── 01-audit-log-export/            ← R2-archived HMAC-chained audit log (read-only)
  │     audit_log_events-<window>.jsonl
  ├── 02-supabase-studio-logs/        ← Dashboard audit log PDF + JSON
  ├── 03-cloudflare-access-logs/      ← Workers + Gateway logs for timeframe
  ├── 04-scope-assessment/            ← SQL query outputs (blast radius)
  │     blast-radius-by-table.csv
  │     multi-tenant-exposure.csv
  ├── 05-role-snapshot/               ← pg_roles + information_schema at T+0
  ├── 06-github-access-log/           ← GitHub audit log for subject (if applicable)
  ├── 07-legal-hold-notice.txt        ← Timestamped legal hold
  ├── 08-incident-channel-export/     ← Exported private channel log (founder only)
  ├── 09-containment-actions.txt      ← Timestamped log of every access change
  ├── 10-gdpr-assessment.md           ← Art. 33/34 necessity determination
  └── 11-pir-draft.md                 ← Post-incident review (redacted for external use)
```

**Chain of custody:** Each file in the evidence package is SHA-256 hashed at collection time and recorded in `00-manifest.sha256`. The package is stored in R2 (`form-compliance-evidence/insider-<case-id>/`) with object versioning enabled. Only founder + compliance-officer have read access. No modifications are permitted after initial upload.

**Retention:** 7 years from case closure date, consistent with DEC-030 audit log retention policy.

---

#### R-12.10 Post-Incident Preventive Controls

The following controls must be verified or implemented after every insider threat incident (P0/P1), regardless of whether exfiltration was confirmed:

| Control | Verification Action | Owner |
|---|---|---|
| Principle of least privilege audit | Review all `form_admin` and `form_system` role grants; remove any that are broader than operationally necessary | security-engineer |
| Offboarding checklist compliance | Confirm the current offboarding procedure covers all access systems in §R-12.6 Containment (C-4 list) | compliance-officer |
| Audit log alert on bulk read | Add an alert rule in OBSERVABILITY.md §6.2 for: actor reads > 500 records in `health_profiles`, `coaching_turns`, or `cv_sessions` within 60 minutes | devops-lead |
| Workers Secrets access review | Confirm only active team members with operational need have Cloudflare dashboard access | security-engineer |
| Employee access review cadence | Confirm quarterly access review is scheduled (SOC 2 CC6.3 evidence requirement) | compliance-officer |
| HMAC chain integrity alert | Confirm the chain integrity cron (§5 R-05) fires on any actor-is-staff + resource-type-is-audit_log_events read event | devops-lead |
| Acceptable use policy acknowledgment | Confirm all current team members have signed an AUP that explicitly covers system access monitoring rights | compliance-officer |

---

#### R-12.11 SOC 2 Evidence Mapping

| SOC 2 Criterion | Evidence from R-12 |
|---|---|
| **CC6.2** — Prior to issuing system credentials, the entity registers and authorizes new internal and external users | Offboarding checklist (§R-12.6 C-4) and access review cadence (§R-12.10) demonstrate that credential lifecycle is managed |
| **CC6.3** — The entity authorizes, modifies, or removes access to data, software, functions, and other protected information assets based on approved and documented requests | Quarterly access review process; role grant snapshots in evidence package |
| **CC7.2** — The entity monitors system components for anomalies that could indicate a malicious act | Bulk-read alerting rule (§R-12.10); HMAC chain monitoring cross-reference |
| **CC7.3** — The entity evaluates security events | Severity matrix (§R-12.2); blast radius assessment (§R-12.5) |
| **CC7.4** — The entity responds to identified security incidents by executing a defined incident response program | This runbook as executed evidence; private channel export; containment log |
| **CC9.1** — The entity identifies, selects, and develops risk mitigation activities for risks arising from potential business disruptions | Insider threat as a documented risk in the FORM risk register (SOC2_READINESS.md §14); preventive controls (§R-12.10) |
| **CC1.2** — COSO Principle 2: The board demonstrates a commitment to integrity and ethical values | Acceptable use policy acknowledgment requirement (§R-12.10); no-pay and no-bypass policies |

**Auditor evidence artefacts:**
- Executed copy of this runbook with timestamped actions
- Evidence package manifest (`compliance/evidence/insider-<case-id>/00-manifest.sha256`)
- Quarterly access review records (`compliance/evidence/access-reviews/YYYY-QN.csv`)
- AUP acknowledgment log (`compliance/hr/aup-acknowledgments.csv`)

---

#### R-12.12 Tabletop Scenario G (for §9 drill catalog)

**Scenario G — Insider data exfiltration before resignation (Year 2, Q3):**

*"It is 22:30 UTC on a Thursday. The HMAC chain integrity cron fires normally. But a PostHog anomaly alert, configured for bulk `health_profiles` read events, fires at 22:47 UTC: an actor with `form_admin` privileges has queried 2,200 records from `health_profiles` across 14 enterprise tenants in the past 90 minutes. A database role snapshot shows this actor is a current team member who gave verbal notice of resignation two weeks ago. Their official last day is in 8 days. No formal offboarding has been initiated."*

Key discussion points:
- Can you determine within 30 minutes whether data left FORM's systems?
- The R2 archive is the forensic source — is the HMAC chain still intact?
- Who is in the investigation channel? Who is NOT? Why does it matter?
- When does the legal counsel call happen? Before or after the Supabase access is revoked?
- This affects 14 enterprise tenants — what is the timeline for E-01 notifications?
- The 72-hour Art. 33 clock: when exactly did it start?
- Is this a P0 or P1? At what point does it upgrade?
- What does the employment contract say about monitoring rights in this jurisdiction?
- The subject still has 8 working days of access. How do you balance ongoing access needs with containment?

**Who participates:** founder, security-engineer, compliance-officer, customer-success.
**Legal observer invited:** external counsel or someone simulating their role.
**Document findings in a tabletop PIR. Add to Year 2 testing schedule.**

---

## 14. Continuous Improvement Program

### 14.1 Purpose

The Post-Incident Review (§8) generates action items. This section defines how those action items are tracked to completion, how control effectiveness is reviewed on a recurring cadence, and how PIR findings feed back into runbook updates. This is the SOC 2 CC4 evidence layer: the organization not only responds to incidents but continuously evaluates and improves its controls.

**Without this section, the IR program has a structural gap:** incidents generate PIR documents with action items, but there is no formal mechanism to verify closure, identify recurring failure patterns, or demonstrate to auditors that the feedback loop closes. SOC 2 CC4.1 (performance evaluations of controls) and CC4.2 (evaluation of deficiencies) require documented evidence of this loop.

---

### 14.2 PIR Action Item Registry

Every action item generated in a PIR (§8.3) is entered into the PIR Action Item Registry within 48 hours of PIR finalization. The registry is a Linear project labeled `[IR] Action Items` with the following field schema:

| Field | Values | Required |
|---|---|---|
| `incident_id` | `INC-YYYYMMDD-slug` | Yes |
| `severity_of_source_incident` | P0 / P1 / P2 | Yes |
| `action_item_priority` | Critical / High / Medium / Low | Yes |
| `title` | Short description of the action | Yes |
| `owner_role` | security-engineer / devops-lead / compliance-officer / founder / platform-engineer | Yes |
| `due_date` | ISO 8601 date | Yes for Critical/High |
| `soc2_criterion` | CC7.5 / CC4.1 / CC6.3 / etc. | Yes if SOC 2 relevant |
| `status` | Open / In Progress / Blocked / Closed | Yes |
| `close_date` | ISO 8601 date | Required on Close |
| `verification_note` | How the action was verified as complete | Required on Close |

**Mandatory SLAs for action item closure:**

| Priority | Source Severity | Closure SLA | Escalation if Missed |
|---|---|---|---|
| Critical | P0 | 14 calendar days | Founder + compliance-officer paged |
| Critical | P1 | 30 calendar days | Founder notified |
| High | P0 | 30 calendar days | Founder notified |
| High | P1 | 60 calendar days | security-engineer reviews |
| Medium | Any | 90 calendar days | Quarterly review flags |
| Low | Any | 180 calendar days | Semi-annual review flags |

**SOC 2 evidence:** The Linear export of all `[IR] Action Items` tickets (closed and open) is produced monthly and stored at `compliance/evidence/ir-action-items/YYYY-MM.csv`. This is the primary CC7.5 evidence artefact (entity identifies and develops activities to recover from security incidents).

---

### 14.3 Escalation Thresholds

The following conditions trigger an automatic escalation to founder + compliance-officer, regardless of individual ticket status:

| Threshold | Condition | Escalation Action |
|---|---|---|
| Overdue Critical action items | Any Critical action item past due date | Immediate: founder + compliance-officer notified by security-engineer; root cause documented in same ticket |
| Recurring failure pattern | Same control fails in 2 P0/P1 incidents within 12 months | Quarterly review convened within 2 weeks; root cause treated as systemic, not one-off |
| > 3 High items open simultaneously | More than 3 High-priority items open at any point | Weekly stand-up added: security-engineer + founder review open items until count falls below 3 |
| SOC 2 observation period breach | Any Critical or High item remains open at the start of the SOC 2 observation period | Automatic SOC 2 readiness flag (SOC2_READINESS.md §38 gate) — auditor must be notified of open items at period start |

---

### 14.4 Quarterly Control Effectiveness Review

Held on the first Monday of each calendar quarter. Owner: security-engineer + compliance-officer. Attendees: founder (required for Q4).

**Agenda (60 minutes maximum):**

1. **Action item registry review** (15 min) — All items opened since last quarter. Overdue items: root cause. Closed items: verification adequate? Patterns across items.

2. **Incident trend analysis** (15 min) — Incidents in the past quarter by severity and type. Any type appearing more than once: systemic? Alert quality: false positive rate (target < 20% of P2 pages). Mean time to detect (MTTD) and mean time to respond (MTTR) by severity.

3. **Runbook gap check** (15 min) — Did any incident activate a runbook that was inadequate or absent? Are all runbooks current with the most recent PIR findings? Cross-check: do all runbooks reference current infrastructure (e.g., Supabase plan, Workers configuration)?

4. **SOC 2 evidence completeness** (10 min) — Are all evidence artefacts per the master index (SOC2_READINESS.md §38.6) current? Any gaps ahead of the next observation period?

5. **Decisions and action items** (5 min) — Record new action items in the registry immediately. Set due dates. Confirm owners.

**Output:** A quarterly review record stored at `compliance/evidence/quarterly-ir-review/YYYY-QN.md`. Template:

```markdown
# IR Quarterly Review — YYYY Q[N]

**Date:** YYYY-MM-DD
**Attendees:** [roles, not names]
**Owner:** security-engineer

## Action Item Registry Summary
- Total open items: [N]
- Overdue items: [N] — [list with root cause]
- Closed this quarter: [N]
- New items added: [N]

## Incident Trend Analysis
- Incidents this quarter: P0=[N], P1=[N], P2=[N]
- Recurring patterns: [none / describe]
- MTTD (P0): [Xh average]
- MTTR (P0): [Xh average]
- Alert false positive rate: [X%]

## Runbook Gaps Identified
[List or "none"]

## SOC 2 Evidence Completeness
[Green / Yellow / Red — explain if not Green]

## Decisions Made
[List]

## New Action Items
[Entered in Linear — link to filter]
```

---

### 14.5 Runbook Update Protocol

Runbooks become stale when infrastructure changes, when incidents reveal gaps, or when new threat types emerge. This protocol governs how and when runbooks are updated.

**Mandatory update triggers:**

| Trigger | Runbook(s) Affected | Update SLA | Owner |
|---|---|---|---|
| P0/P1 incident reveals runbook gap or error | Relevant runbook(s) | 7 days after PIR finalized | security-engineer |
| Infrastructure change affects response steps (e.g., Supabase migration, new Worker) | All runbooks referencing that component | Before change is deployed to production | devops-lead |
| New third-party processor added | R-07, §11 sub-processor registry | Before processor handles production data | compliance-officer |
| Regulatory change affecting notification obligations | R-01, R-07, R-11, §10 | 30 days from regulatory effective date | compliance-officer |
| Annual review with no incidents | All runbooks | Once per year (calendar Q4) | security-engineer |
| Tabletop exercise reveals gap | Relevant runbook(s) | 14 days after tabletop | security-engineer |

**Update procedure:**
```
1. Create a Linear ticket: [IR Runbook Update] <runbook-name> — <brief reason>
   Link to PIR or tabletop record that triggered the update.

2. Draft the change in a git branch: incident-response-patch/<YYYYMMDD>-<slug>

3. Review gate: compliance-officer reviews all changes to:
   - Regulatory notification timelines (§10)
   - GDPR obligations language (any runbook)
   - Sub-processor registry (§11.2)
   - Privacy floor enforcement clauses
   security-engineer reviews all technical steps.

4. Merge to main after dual approval.
   Commit message: incident-response: <change-summary> (runbook update per <PIR-ID or tabletop>)

5. Version bump (patch): update footer version, CHANGELOG.
   Update ToC entry if section numbers changed.

6. Archive: store a tagged snapshot of this document in
   compliance/evidence/runbook-versions/INCIDENT_RESPONSE-v<X.Y>.md
   at each minor version bump.
```

---

### 14.6 Annual Control Self-Assessment

Once per year (calendar Q4), the security-engineer conducts a formal control self-assessment against the complete SOC 2 CC7 and CC4 criteria. This is distinct from the quarterly review: it is structured against audit criteria, not operational metrics.

**Self-assessment template:**

```markdown
# Annual Control Self-Assessment — IR Program
**Year:** YYYY
**Owner:** security-engineer
**Reviewer:** compliance-officer
**Stored at:** compliance/evidence/annual-csa/YYYY-ir-csa.md

## CC4.1 — Performance Evaluations of Controls

| Control | Evidence | Self-Rated Status | Gap |
|---|---|---|---|
| Alert verification monthly | Linear tickets labeled alert-test-* | ✅ / 🟡 / 🔴 | |
| Tabletop quarterly | Tabletop PIR records | | |
| DR test annual | DR test report | | |
| Pen test annual | Pen test report | | |
| Runbook review after P0/P1 | Git commit history on INCIDENT_RESPONSE.md | | |
| Action item closure SLAs met | Linear action item export | | |

## CC4.2 — Evaluation of Deficiencies

List all deficiencies identified this year:
| Deficiency | Source | Severity | Remediated? | Evidence |
|---|---|---|---|---|
| [describe] | PIR-YYYYMMDD / Tabletop / Alert failure | Critical/High/Medium | Yes/No | [link] |

## CC7.2 — Monitoring for Anomalies
[Confirm all alert sources in §3.1 remain configured and verified. List any gaps.]

## CC7.4 — Incident Response Program
[Confirm all runbooks were reviewed in the past 12 months. List last review date per runbook.]

## CC7.5 — Recovery Activities
[Confirm PIR was completed for every P0/P1 in the past 12 months. List incidents and PIR dates.]

## Summary Assessment
Self-rated SOC 2 readiness for CC4+CC7 cluster: [Ready / Conditionally Ready / Gaps Require Remediation]
Identified gaps to address before next observation period: [list or "none"]
```

**Output** stored at `compliance/evidence/annual-csa/YYYY-ir-csa.md`. Shared with the audit firm at observation period open.

---

### 14.7 SOC 2 Evidence Contribution

| SOC 2 Criterion | Evidence Generated by §14 |
|---|---|
| **CC4.1** — The entity evaluates and communicates internal control deficiencies | Quarterly review records; annual CSA; action item registry — these demonstrate a functioning evaluation cycle |
| **CC4.2** — The entity evaluates and communicates deficiencies and their remediation | Overdue escalation records; recurring pattern analysis in quarterly review; runbook update protocol with PIR linkage |
| **CC7.5** — The entity identifies, develops, and implements activities to recover from identified security incidents | Action item registry with closure evidence; verification notes on each closed item; SLA compliance data |
| **CC2.1** — The entity obtains or generates and uses relevant, quality information to support the functioning of internal control | Quarterly review trend analysis; MTTD/MTTR metrics; false positive rate tracking |

**Auditor evidence package for §14:**
- `compliance/evidence/ir-action-items/YYYY-MM.csv` (monthly Linear export) — 12 months
- `compliance/evidence/quarterly-ir-review/YYYY-QN.md` — 4 records per year
- `compliance/evidence/annual-csa/YYYY-ir-csa.md` — 1 per year
- `compliance/evidence/runbook-versions/` — tagged snapshots at each version bump
- Git commit history on `docs/INCIDENT_RESPONSE.md` showing review cadence

---

### R-13: Law Enforcement / Government Data Request

**Definition:** A request from any government body, law enforcement agency, national security authority, court, regulatory body, or supervisory authority — domestic or foreign — seeking disclosure of FORM user data, metadata, or system records.

This runbook is architecturally distinct from all prior runbooks: there is no attacker to contain and no breach to remediate. The triggering event is a legal instrument demanding disclosure. FORM will comply only with lawful, compulsory process and only to the minimum extent required. This runbook operationalizes `PRIVACY_POLICY.md §6.4`, DEC-030, GDPR Art. 48, and ENTERPRISE.md no-go customer policy (government backdoors).

**No voluntary disclosure.** FORM does not produce user data in response to informal requests, tips, phone calls, or non-compulsory letters — regardless of agency or claimed urgency. Only compulsory legal process (subpoena, court order, search warrant, administrative order, GDPR supervisory authority binding decision) triggers this runbook. Informal requests receive a written response directing the agency to obtain compulsory process.

---

#### R-13.1 Request Type Taxonomy

| Type | Jurisdiction | Compulsory? | Examples |
|---|---|---|---|
| **Criminal subpoena** | Any | Yes — court-issued | Grand jury subpoena, criminal court order, police production order |
| **Civil subpoena** | Any | Yes — court-issued | Litigation discovery, arbitration subpoena |
| **Search warrant** | Any | Yes — search compelled | Criminal warrant served by law enforcement |
| **National Security Letter (NSL)** | US | Yes — FBI-issued; gag order standard | Terrorism/espionage financial + communication records |
| **FISA court order / Section 215** | US | Yes — secret court; gag order standard | Foreign intelligence surveillance order |
| **Foreign intelligence order** | Non-US states | Yes — depends on jurisdiction | EU member-state intelligence agency orders |
| **EU DPA supervisory authority demand** | EU | Yes — binding under GDPR Art. 58 | Regulatory investigation, cross-border enforcement |
| **EU Art. 48 cross-border court order** | Non-EU → EU data | Conditionally — requires treaty basis | US DOJ order for EU-resident user data |
| **Informal / voluntary request** | Any | No | Phone call, email tip, unserved "request", courtesy notice |
| **Emergency informal request (life safety)** | Any | No — special handling | Claimed imminent harm; see R-13.3 constraint #5 |

---

#### R-13.2 Severity Matrix

| Severity | Trigger | Rationale |
|---|---|---|
| **P0** | NSL or FISA order received | Gag order likely; specialized counsel required before any acknowledgment; maximum sensitivity |
| **P0** | Foreign intelligence order from state security service | High risk of coerced disclosure; Art. 48 GDPR likely implicated |
| **P0** | Any request covering Art. 9 health / biometric data for ≥ 10 users | Special-category data; Art. 34 data-subject notification likely required independently |
| **P0** | Any request covering an enterprise tenant's user base | Enterprise DPA triggered; IT admin notification required; may constitute enterprise SLA event |
| **P1** | Criminal court order or search warrant | Compulsory; legal review within 4 hours; likely compliance after narrow-scope review |
| **P1** | GDPR supervisory authority investigation demand (GDPR Art. 58) | Regulatory consequence if ignored; 10-business-day response window typical |
| **P2** | Civil subpoena (litigation) | Court-issued but lower urgency than criminal; 14-day response window typical |
| **P2** | Voluntary request accompanied by expressed intent to obtain process | Log and prepare; no disclosure until process received |
| **P3** | Informal voluntary request with no legal instrument | Decline in writing; no disclosure |

---

#### R-13.3 Unique Response Constraints

**Constraint 1 — Founder-only authority.**
No team member may initiate a disclosure, acknowledge a request to a third party, or waive any legal right without founder sign-off. This applies regardless of request type, jurisdiction, or claimed urgency. If the founder is unreachable for more than 4 hours on a P0 request, the designated backup is compliance-officer with mandatory founder briefing within 1 hour of regaining contact.

**Constraint 2 — Legal counsel before any action.**
FORM must engage outside legal counsel (EU-qualified for GDPR matters; US-qualified for NSL/FISA) before responding to any compulsory process. For P0 NSL/FISA: retain counsel before acknowledging receipt to the agency. For P1/P2: retain counsel within 24 hours of receipt. Counsel reviews (a) validity of the legal instrument, (b) scope minimization opportunities, (c) notification obligations, (d) grounds to challenge.

**Constraint 3 — Narrow scope enforcement.**
FORM will not produce data beyond what the legal instrument explicitly compels. If a request is overbroad (see R-13.6), counsel files a motion to quash or narrow before any partial disclosure. Standard scope questions: Does the order name specific users? A specific time range? Specific data categories? FORM produces a minimum-necessary export scoped to named users, named date range, and named data types only.

**Constraint 4 — No Art. 9 voluntary production.**
Workout performance data, coaching session content, form scores, HR data, biometric keypoints, health profiles — are never produced voluntarily. For compulsory process: these require explicit Art. 9 data category acknowledgment in the instrument (not just "all records pertaining to user X") and a higher legal standard in most jurisdictions. Counsel must confirm Art. 9 production is specifically required by the instrument.

**Constraint 5 — Emergency informal requests.**
If a law enforcement officer claims imminent risk of death or serious physical harm and requests voluntary disclosure without a court order ("emergency exception"), the response is: (a) take the contact details and claimed basis in writing, (b) call the founder within 10 minutes, (c) counsel assesses within 30 minutes whether voluntary disclosure is legally defensible and warranted, (d) if produced: absolute minimum (e.g., last known GPS only), logged in full with DEC-030 event `legal.emergency_voluntary_disclosure`. This exception is rare and requires documented founder decision.

**Constraint 6 — GDPR Art. 48 for non-EU orders on EU data.**
FORM stores EU user data in Supabase EU region (`eu-west-1`). A non-EU court order (e.g., US DOJ subpoena) requiring production of EU-resident user data must satisfy GDPR Art. 48: the order must be based on an international agreement (MLAT, adequacy decision) in force between the EU and the requesting country. Without a treaty basis, FORM can only comply if the order is narrowly construed as necessary to avoid a disproportionate sanction under national law — and only after DPA notification in the EU member state where the data subject resides. Counsel must assess Art. 48 compliance before any cross-border production. See R-13.10 for detailed GDPR Art. 48 analysis.

**Constraint 7 — No-go customer no-backdoor reaffirmation.**
ENTERPRISE.md and DEC-030 explicitly prohibit providing government backdoor access, insurance risk-scoring, or proactive cooperation beyond legal compulsion. This constraint survives any change in team or external pressure. Any request to install surveillance software, provide ongoing access credentials, or create a programmatic monitoring hook is refused unconditionally and reported to legal counsel immediately.

**Constraint 8 — Restricted incident channel.**
Open channel `#inc-YYYYMMDD-legal` — access: founder + compliance-officer + legal counsel only. Not security-engineer unless specific technical data extraction is required. Reason: legal privilege considerations; minimizing internal knowledge of NSL/FISA gag orders; limiting who can inadvertently disclose the existence of the order.

---

#### R-13.4 Immediate Actions (T+0 to T+4h)

**T+0: Receive and log.**
1. The person who receives the request (email, physical mail, phone call, in-person service) notifies the founder and compliance-officer immediately via Signal direct message — not Slack.
2. Photograph or scan the instrument (including envelope, delivery metadata). Store in `compliance/evidence/legal-requests/<YYYY-MM-DD-slug>/received/`.
3. Record exact timestamp of receipt, delivery method, serving agency name, agency contact information, stated deadline.
4. Do NOT acknowledge receipt to the agency until legal counsel is retained (exception: a formal signed acceptance of service if required by the instrument).
5. Do NOT discuss the request with anyone outside the restricted channel.
6. Emit DEC-030 `legal.request_received` event immediately (see R-13.11).

**T+0 to T+1h: Classify and retain counsel.**
7. Founder classifies severity using R-13.2 matrix.
8. For NSL/FISA (P0): engage NSL-experienced US counsel immediately. Do not log the existence of the order in any system that is accessible to staff without explicit legal advice — gag order may prohibit even internal acknowledgment.
9. For all other requests: compliance-officer opens the restricted channel `#inc-YYYYMMDD-legal`. Retain outside counsel within 2 hours (P0/P1) or 24 hours (P2).
10. Trigger enterprise tenant notification assessment (R-13.9) within 30 minutes of receipt if the request names enterprise tenant users.

**T+1h to T+4h: Initial legal assessment.**
11. Provide counsel with: the instrument, FORM's data architecture summary (DATA_MODEL.md §4.1 tenant isolation, §5 data classification), applicable enterprise DPAs, current GDPR Data Processing Agreement.
12. Counsel provides initial assessment:
    - Is the instrument legally valid (properly issued, within jurisdiction, served correctly)?
    - Is it overbroad? Grounds to challenge or narrow?
    - Is user notification required or prohibited?
    - Does GDPR Art. 48 or Art. 9 special-category analysis apply?
    - Response deadline and next steps.
13. Founder makes initial compliance/challenge decision based on counsel's advice.

---

#### R-13.5 Legal Review Process

| Step | Actor | Timing | Output |
|---|---|---|---|
| 1. Instrument validity check | Counsel | T+4h | Written memo: valid / defective / service error |
| 2. Scope analysis | Counsel + compliance-officer | T+8h | Named users, date range, data categories required by instrument |
| 3. Over-breadth assessment | Counsel | T+12h | Challenge recommendation or narrowing letter drafted |
| 4. GDPR Art. 48 / Art. 9 analysis | Counsel (EU-qualified) | T+12h | Art. 48 treaty basis assessment; Art. 9 production checklist |
| 5. Notification obligations | Counsel | T+12h | Can we notify users? Must we notify DPA? Timeline? |
| 6. Enterprise DPA review | Compliance-officer | T+8h | Does any enterprise DPA require tenant notification before compliance? |
| 7. Disclosure decision | Founder (+ counsel sign-off) | T+24h (P1/P2); T+4h (P0 warrant) | Comply / challenge / partial comply with reservation |
| 8. Data extraction | Platform-engineer (read-only query; scoped to named users/range) | After founder + counsel sign-off | Minimal data extract per scope analysis |
| 9. Legal hold activation | Compliance-officer | T+4h | `legal_hold_active = true` flag in `tenants` table; no deletion for affected records |
| 10. Disclosure execution | Compliance-officer + counsel | Per court deadline | Data package produced; production letter signed by counsel |
| 11. Post-disclosure | Compliance-officer | Within 5 business days | Evidence package finalized; transparency tally updated |

---

#### R-13.6 Challenging Overbroad Requests

FORM challenges requests that are:
- **Temporally unlimited** (no date range specified) — file for temporal narrowing
- **User-class based** ("all users who did X") — challenge: US 4th Amendment particularity; EU GDPR proportionality; require individual identification
- **Art. 9 health data without explicit authorization** — challenge on grounds that production requires a higher legal standard (explicit court finding) not present in a general subpoena
- **Cross-border (Art. 48) without treaty basis** — file GDPR Art. 48 objection; notify relevant EU DPA before complying; seek stay pending DPA guidance
- **Enterprise-wide** (covering all users of a named tenant) — challenge proportionality; produce only named individuals

**Challenge process:**
1. Counsel files motion to quash or motion to modify (civil court) or challenges via counsel letter (administrative orders).
2. Do not delay compliance with a valid order while a challenge is pending unless a court has granted a stay — continue evidence preservation and legal hold activation.
3. If challenge is denied: comply with narrowed scope if any scope narrowing was achieved; comply in full otherwise.
4. Log challenge outcome in the incident record.

---

#### R-13.7 Scope Minimization: What to Produce

**Permitted for production (with valid court order + counsel sign-off):**

| Data Category | Condition for Production |
|---|---|
| User ID (UUID) | Named user, specific request |
| Account creation timestamp | Named user, specific request |
| Last login timestamp | Named user, specific request |
| Email address | Named user, specific request; counsel confirms email in scope |
| Subscription status and plan | Named user, specific request |
| IP address logs (if retained) | Named user, named date range; GDPR Art. 6(1)(c) basis |
| SSO login audit events | Named user, named date range |
| Push notification delivery logs | Named user, named date range |

**Requires higher legal standard (explicit court order + counsel Art. 9 confirmation):**

| Data Category | Additional Requirements |
|---|---|
| Workout logs and form scores | Art. 9 explicit reference in instrument |
| Coaching session content (`coaching_turns`) | Art. 9; plus production creates clinical harm risk — clinical-safety review required |
| Health profile data | Art. 9; explicit finding of necessity |
| Meal logs | Art. 9 |
| Wearable biometrics (HRV, sleep, HR) | Art. 9 |

**Never produced under any legal process:**

| Data Category | Rationale |
|---|---|
| CV session `keypoints_enc` raw data | On-device only; FORM cannot decrypt per architecture (on-device key); production is architecturally impossible |
| `cv_session_fleet_stats` (k-anonymity aggregates) | Aggregate only; cannot be de-aggregated to named individual |
| Other users' data not named in the instrument | No legal basis; FORM does not volunteer additional users |
| HR aggregate dashboards | FORM holds privacy floor: HR aggregates cannot be attributed to individuals; production of aggregate only possible if no individual inference is possible |

---

#### R-13.8 User Notification Protocol

**Default:** FORM notifies the named user(s) before complying with legal process unless legally prohibited from doing so.

**Notification timing:** As soon as disclosure decision is made and before data is produced (except where a court order expressly prohibits prior notice).

**Prohibited notification (gag order or court order):**
- NSL: Federal gag order standard; counsel advises on scope of prohibition.
- FISA: Gag standard; counsel advises.
- Some criminal court orders include explicit prior-notice prohibitions.
- Where notification is prohibited, FORM updates its public **Transparency Report** (see R-13.12) with aggregate statistics without identifying the specific order.

**User notification template (where permitted):**

> Subject: Legal process notice — FORM account
>
> We are writing to inform you that FORM has received [a court order / a subpoena / a legal demand] requiring us to produce certain information about your account. We believe in transparency and have a policy of notifying affected users when legally permitted to do so.
>
> What was requested: [description of data categories scoped to the instrument — not more].
> What we are producing: [description of minimal data set after scope minimization].
> When: [date of planned disclosure].
>
> You have the right to seek legal advice and, if you believe the order is unlawful, to seek a court order preventing or limiting production. We cannot delay production past [court deadline] without a court-issued stay.
>
> Questions: legal@form.coach

**GDPR Art. 34 co-trigger:** If the request covers Art. 9 health data and the legal production would constitute a "high risk" disclosure to the data subject, compliance-officer assesses whether an independent Art. 34 notification obligation is triggered under GDPR (separate from the user notification above). Counsel advises.

---

#### R-13.9 Enterprise Tenant Notification

**When triggered:** Any legal request naming users who belong to an enterprise tenant, or any request naming the tenant organization directly.

**Who notifies:** compliance-officer (never platform-engineer or CSM without founder approval).

**Timing:** Within 30 minutes of P0 classification; within 4 hours of P1 classification; within 24 hours of P2 classification.

**What to disclose:** Only what the enterprise DPA requires FORM to disclose. Standard DPA provisions require notification if:
- User data within the tenant's contracted scope is subject to a legal demand.
- Production may affect the tenant's own GDPR obligations (e.g., the tenant is the data controller for its users).

**What NOT to disclose:** The specific text of NSL or FISA orders (gag order applies). The identity of the agency or investigators beyond what is publicly available in an unsealed court order. Any information that counsel advises must be kept confidential.

**Template T-01: Initial Enterprise Tenant Legal Request Notification**

> Subject: Legal process affecting your FORM account — confidential
>
> [Tenant admin name],
>
> FORM has received a legal demand from [a court / a regulatory authority / [agency name if unsealed]] that may require us to produce certain account records related to [user(s) within your organization / your organization's account].
>
> We are in the process of legal review and will only produce data that we are legally required to produce. We are committed to minimizing scope and will challenge any overbroad aspects of the request.
>
> At this stage: No data has been disclosed. We are retaining legal counsel.
>
> We are informing you because your DPA with FORM requires us to provide notice when legally permitted. Depending on your role as data controller, you may have independent obligations to assess and act on this notification.
>
> We will provide an update within [48 hours / prior to any disclosure].
>
> Please direct any questions to your FORM Customer Lead or legal@form.coach. Please treat this communication as confidential.

**Note:** If counsel advises that enterprise notification is legally prohibited (e.g., gag order covers tenant identity), do NOT send this template. Log the prohibition decision in the incident record with counsel's written guidance.

---

#### R-13.10 GDPR Art. 48 Analysis (Non-EU Court Orders on EU Data)

FORM's user data for EU-resident users is stored in Supabase `eu-west-1` (Frankfurt region). GDPR Art. 48 governs whether FORM can comply with a non-EU court order or governmental request requiring transfer of EU-resident data to a non-EU jurisdiction.

**Art. 48 rule:** Judgment or court order of a third country is only enforceable and recognised if based on an international agreement (e.g., Mutual Legal Assistance Treaty — MLAT) in force between the third country and the EU or a Member State.

**MLAT status (as of drafting):**

| Requesting Country | MLAT with EU in force? | Notes |
|---|---|---|
| United States | Partial — EU-US MLAT (2003) | MLAT exists but non-self-executing; US orders must be channeled through EU Member State competent authority for binding production |
| United Kingdom | UK-EU MLA Convention pending | Post-Brexit: UK requests should be routed through individual EU Member State agreements; uncertain status |
| Other | Case-by-case | Counsel must assess per country |

**Art. 48 compliance procedure:**
1. Counsel confirms whether the requesting country has a valid MLAT or international agreement with the EU in force.
2. If yes (e.g., US MLAT): production may proceed via the treaty channel after scope minimization and notification obligations are met.
3. If no or unclear: FORM cannot comply with the non-EU court order for EU-resident user data unless one of the limited Art. 49 derogations applies (vital interests of the data subject is the only plausible one — narrow, requires counsel confirmation).
4. Compliance-officer notifies the competent EU DPA in the data subject's Member State of the request and FORM's intended response, before production (unless gag order prohibits).
5. Document Art. 48 analysis in the incident evidence package.

**DPA notification contact (EU):**

| Likely jurisdiction | DPA | Contact |
|---|---|---|
| Germany (Supabase server location) | BfDI (Bundesbeauftragter für Datenschutz) | deregister request via bfdi.bund.de |
| User's residence | Member State DPA | Per GDPR Art. 56 one-stop-shop: lead DPA is Ireland (DPC) for cross-border cases |
| Ukraine-resident users | UAPDP | uapdp.gov.ua |

---

#### R-13.11 Evidence Package

The following artifacts are filed at `compliance/evidence/legal-requests/<YYYY-MM-DD-slug>/`:

```
legal-requests/
└── YYYY-MM-DD-slug/
    ├── received/
    │   ├── instrument-scan.pdf          # Original legal instrument (scanned)
    │   ├── delivery-metadata.txt        # Receipt timestamp, method, agent details
    │   └── sha256-manifest.txt          # SHA-256 hash of each file
    ├── counsel/
    │   ├── retention-engagement.pdf     # Signed counsel retention letter
    │   ├── validity-memo.pdf            # Instrument validity assessment
    │   ├── scope-analysis.pdf           # Data categories analysis
    │   ├── gdpr-art48-memo.pdf          # Art. 48 / Art. 9 assessment (EU cases)
    │   └── challenge-filing.pdf         # Motion to quash / narrowing letter (if filed)
    ├── disclosure/
    │   ├── production-package.zip       # Data produced (encrypted; password via 1Password)
    │   ├── production-letter.pdf        # Signed production cover letter (counsel)
    │   ├── disclosure-log.txt           # Exact data categories, user IDs, date range produced
    │   └── sha256-manifest.txt
    ├── notifications/
    │   ├── user-notification.txt        # Notification sent to user (or gag-order prohibition memo)
    │   ├── enterprise-tenant-notification.txt  # T-01 sent (or prohibition memo)
    │   └── dpa-notification.pdf         # EU DPA notification (Art. 48 cases)
    └── incident-log.md                  # Chronological log of all actions, decisions, timestamps
```

**Retention:** 7 years (DEC-030 HMAC chain; consistent with Art. 9 data record-of-processing retention under GDPR Art. 30; US litigation hold best practice).

**DEC-030 HMAC-chained audit events for R-13:**

| Event | Severity | When emitted |
|---|---|---|
| `legal.request_received` | HIGH | T+0: instrument received and logged |
| `legal.counsel_retained` | MEDIUM | When outside counsel engagement confirmed |
| `legal.legal_hold_activated` | HIGH | When `legal_hold_active` flag set on affected records |
| `legal.challenge_filed` | MEDIUM | If motion to quash or narrowing letter filed |
| `legal.user_notified` | MEDIUM | When user notification sent |
| `legal.tenant_notified` | MEDIUM | When enterprise tenant notification sent |
| `legal.dpa_notified` | HIGH | When EU supervisory authority notified (Art. 48 cases) |
| `legal.disclosure_executed` | CRITICAL | When data production is delivered to requesting party |
| `legal.emergency_voluntary_disclosure` | CRITICAL | Emergency Art. 9 voluntary disclosure (R-13.3 constraint #5 exception) |
| `legal.legal_hold_released` | MEDIUM | When litigation hold lifted post-resolution |

---

#### R-13.12 Transparency Reporting

FORM publishes an annual **Government Transparency Report** covering:
- Number of requests received (by type: criminal, civil, NSL, FISA, EU DPA) — or "0 to N" range to protect NSL gag order status
- Number of requests where FORM disclosed data vs. challenged vs. declined
- Number of users affected (aggregate; not individual identities)
- Number of requests where user notification was prohibited
- GDPR Art. 48 requests and outcome

**Report owner:** compliance-officer + founder.
**Publication cadence:** Annual; published in the `press/` section or as a standalone public document.
**First report:** To be published 12 months after first enterprise launch or upon receipt of first legal request — whichever comes first.
**Format:** Plain-text table. No attorney-client privileged content. No specific incident dates or agency names unless the underlying order is publicly available.
**Gag order constraint:** NSL and FISA gag orders prohibit specific disclosure. FORM uses the "0 to 499" range disclosure method (permitted under Freedoms Act) when applicable.

---

#### R-13.13 SOC 2 Evidence Mapping

| SOC 2 Criterion | How R-13 Satisfies It |
|---|---|
| **CC6.3** — The entity authorizes, modifies, or removes access to data and systems based on approved and documented access policies | R-13 defines the policy under which FORM authorizes government access to user data: compulsory legal process only, minimum necessary scope, founder authorization required. The `legal.disclosure_executed` DEC-030 event is the auditor-accessible record of each authorization. |
| **CC6.1** — The entity implements logical access security software, infrastructure, and architectures over protected information assets to protect them from security events | FORM's refusal to comply with informal requests, the legal-hold mechanism, and the never-produce list (§R-13.7) demonstrate that information assets are protected from unauthorized government access as rigorously as from technical attackers. |
| **C1.1** — The entity identifies and maintains confidential information to meet its objectives related to confidentiality | GDPR Art. 9 production constraints (§R-13.7 higher-standard table) and the never-produce list operationalize FORM's confidentiality commitments for health data specifically. |
| **CC9.1** — The entity identifies, selects, and manages business relationships to achieve its objectives | R-13.9 enterprise tenant notification ensures that FORM's contractual obligation to notify enterprise customers of legal demands is documented and consistently executed. |
| **P6 (Privacy) — Disclosure and Notification** | R-13.8 user notification protocol and R-13.12 transparency report together fulfill the Privacy TSC requirement that the entity discloses information as defined in commitments and agreements. |

**Auditor evidence artefacts:**
- `compliance/evidence/legal-requests/` — case files per request (7-year retention)
- `compliance/evidence/transparency-report/YYYY.md` — annual published report
- DEC-030 `legal.disclosure_executed` events in `audit_log_events` — auditable disclosure record
- `legal.legal_hold_activated` DEC-030 events — demonstrating information preservation on receipt of legal demand

---

#### R-13.14 Tabletop Scenario H (for §9 drill catalog)

**Scenario title:** US DOJ Criminal Subpoena Covering EU-Resident Enterprise Tenant Users

**Setup:**
A US DOJ criminal division issues a grand jury subpoena to FORM demanding "all records, communications, and data pertaining to [TenantName] and its users" for a named 90-day date range. The named tenant is a Growth-tier enterprise customer with 350 users, 80% of whom are EU residents (Germany and France). The subpoena was served by email to legal@form.coach at 22:47 UTC on a Friday. The stated compliance deadline is 14 days from service.

**Participants:** founder, compliance-officer, security-engineer, enterprise-architect, and the tenant's Customer Lead (CS).

**Discussion points:**

1. Who is the first person to act, and what are the first three actions within the first hour?
2. Is this P0, P1, or P2 per R-13.2? What drives the classification?
3. What Art. 48 GDPR analysis applies? Which DPA must be notified, and when?
4. The subpoena says "all records" — what is the scope minimization strategy? What do you challenge and on what grounds?
5. The 80% EU-resident user population: does production differ for EU vs. non-EU users within the same subpoena response?
6. Does the restricted incident channel include the Customer Lead for this tenant? Why or why not?
7. When is the enterprise tenant IT admin notified, using which template, and who sends it?
8. The tenant's IT admin calls demanding to know "why FORM is talking to the DOJ about our employees." How does the Customer Lead respond?
9. Does coaching session content (`coaching_turns`) get produced? What legal standard justifies or prohibits it?
10. It is now T+72h. Counsel has filed a narrowing motion but the court has not yet ruled. Can FORM begin preparing a partial production? Who decides?
11. After disclosure, what DEC-030 events must be in the audit log, and who verifies the HMAC chain integrity?
12. The tenant's legal team sends a cease-and-desist claiming production would breach GDPR. Counsel advises the US order is technically enforceable but Art. 48 is unresolved. Who decides the final outcome?
13. One week after disclosure, the news media reports on the criminal investigation. A journalist contacts FORM for comment. Who responds and what is the policy?
14. In the post-incident review: what one preventive control, if it had existed, would most have reduced FORM's legal exposure? How does that control feed into §14 PIR Action Item Registry?

**Target duration:** 90 minutes. **Facilitator:** compliance-officer. **Required:** outside legal counsel observing (not advising during drill). **Output:** updated R-13 if gaps identified; potential §6 template additions; one §14 action item minimum.

---

### R-14: DSAR / Data Subject Rights Incident

**Trigger:** Any of the following:
- 30-day deadline (Art. 15/20 access/portability) missed or at imminent risk of being missed
- 72-hour Art. 17 erasure failure — Art. 9 health data confirmed still queryable after erasure job ran
- DSAR export confirmed to contain another user's or tenant's data (cross-tenant contamination)
- Bulk or coordinated DSAR campaign (≥ 10 requests in 24 hours from suspected coordinated source)
- DPA notification received of a data subject complaint filed against FORM regarding a DSAR

**Severity:**

| Scenario | Default Severity | Notes |
|---|---|---|
| DSAR export contains cross-tenant data | **P0** | R-01 also active immediately — this is a data breach |
| Art. 9 health data confirmed present after erasure deadline | **P0** | R-01 scope assessment; assess R-11 for `cv_sessions.keypoints_enc` |
| 30-day DSAR deadline missed, DPA not yet contacted | **P1** | Compliance-officer decides on DPA self-report |
| Export worker complete failure (zero exports delivered) | **P1** | Data-engineer + compliance-officer |
| Single DSAR delayed, within cure window (days 28–30) | **P2** | IC informed; manual delivery if possible |
| Bulk DSAR suspected abuse, no deadline risk yet | **P2** | Monitor; → P1 if deadlines materialise |
| DPA complaint received from a data subject | **P1** | Compliance-officer classifies; legal counsel notified |

#### Immediate Actions (T+0 to T+30 min)

**For P0 (cross-tenant contamination or Art. 9 erasure failure):**
```
1. Open incident channel: #inc-YYYYMMDD-dsar
2. Page: IC + compliance-officer + security-engineer simultaneously
3. If cross-tenant contamination: activate R-01 in parallel — treat as a data breach
4. If Art. 9 erasure failure: lock the affected user_id from new data ingestion while
   scope is assessed; do NOT delete further until audit is complete (legal hold applies)
5. Note exact detection timestamp — Art. 33 clock may start now if health data was exposed
6. Capture current state BEFORE remediation:
   - Export contamination: download the offending archive; compute SHA-256
   - Erasure failure: run R-14.1 scope assessment queries; screenshot results
```

**For P1 (missed deadline or export worker failure):**
```
1. Open incident channel or Linear ticket (P1 may be business-hours handled)
2. Notify compliance-officer within 1 hour
3. Missed deadline: draft Template D-01 (DPA self-report — see R-14.7)
4. Export worker failure: page data-engineer; capture job failure logs
5. Check DSAR register for all in-flight requests — identify any others at deadline risk
```

#### R-14.1 Severity Assessment Queries

```sql
-- In-flight DSARs approaching or past deadline
SELECT id, user_id, request_type, submitted_at,
       submitted_at + interval '30 days' AS deadline,
       now() > submitted_at + interval '30 days' AS overdue,
       status
FROM dsar_requests
WHERE status NOT IN ('fulfilled', 'rejected_with_reason')
ORDER BY submitted_at ASC;

-- Check if export delivery was generated and sent
SELECT id, user_id, export_url_generated_at, export_delivered_at,
       export_download_confirmed_at
FROM dsar_requests
WHERE status = 'export_generated' AND export_delivered_at IS NULL;

-- Erasure completeness check for a specific user (run after erasure job claims completion)
SELECT 'user_profiles' AS tbl, count(*) FROM user_profiles WHERE user_id = '<uid>'
UNION ALL
SELECT 'health_profiles', count(*) FROM health_profiles WHERE user_id = '<uid>'
UNION ALL
SELECT 'workout_sessions', count(*) FROM workout_sessions WHERE user_id = '<uid>'
UNION ALL
SELECT 'coaching_turns', count(*) FROM coaching_turns WHERE user_id = '<uid>'
UNION ALL
SELECT 'wearable_readings', count(*) FROM wearable_readings WHERE user_id = '<uid>'
UNION ALL
SELECT 'cv_sessions', count(*) FROM cv_sessions WHERE user_id = '<uid>';
-- All values should be 0 after hard-delete; 1 tombstone row acceptable after soft-delete
```

#### R-14.2 Cross-Tenant Data in Export (P0)

If a DSAR export is confirmed to contain data belonging to a user or tenant other than the requester:

```
1. This is simultaneously a DSAR incident AND a data breach — activate R-01 in parallel
2. Revoke the export download link immediately if not yet downloaded:
   - Invalidate the pre-signed R2/S3 URL or expire the signed delivery token
   - Log: dsar.export_link_revoked (DEC-030, CRITICAL)
3. If already downloaded: you cannot un-download
   - If health data (Art. 9) was in the contaminated archive: Art. 33 clock starts now
   - Focus on scope assessment and notification, not just technical containment
4. Do NOT re-run the export job until root cause is confirmed and a cross-tenant
   isolation regression test passes
5. Scope assessment:
   - Whose data was in the export? (user_id / tenant_id of contaminated data)
   - What data categories? (Art. 9 health data → P0 Art. 34 presumed; PII only → P0, lower Art. 34 risk)
   - Did the requester download the archive?
   - Is this a systematic query bug or an isolated event?
6. If the contaminated data belongs to an enterprise tenant employee:
   → Notify tenant admin via Template E-01 (§12) within 30 min of P0 confirmation
   → Customer Lead owns tenant comms; IC approves all statements
7. After fix and cross-tenant RLS/export isolation test passes:
   → Deliver a corrected export to the original requester
   → Log: dsar.export_redelivered with correction_reason field
```

#### R-14.3 Failed Erasure (Art. 17 Right to Erasure)

FORM implements erasure as a multi-step job (see DATA_MODEL.md §12.1). An erasure incident occurs when the erasure worker completed without error but data remains queryable, a worker timed out leaving partial erasure, or a cascading failure leaves a tombstone row accessible.

```
1. Run R-14.1 scope assessment queries first — document results before any action
2. Check the last erasure worker run in audit log:
   SELECT * FROM audit_log_events
   WHERE event_type = 'data.user_deleted'
     AND metadata->>'user_id' = '<uid>'
   ORDER BY created_at DESC LIMIT 5;

3. Check the erasure worker job logs:
   - Cloudflare Workers tail for the erasure Worker
   - Supabase pg_cron logs for scheduled deletion sweeps

4. Determine scope of partial erasure:
   - Which tables still contain rows?
   - Document in incident channel

5. Containment: prevent the user from logging in while erasure is being completed:
   DELETE FROM auth.sessions WHERE user_id = '<uid>';

6. Manual erasure for remaining rows (IC + compliance-officer authorization required):
   -- Execute per table; commit after each; emit DEC-030 event per table
   DELETE FROM health_profiles WHERE user_id = '<uid>';
   -- repeat for each remaining table
   -- Log: dsar.erasure_manual_supplement (CRITICAL) with table_name, row_count, operator_id,
   --      ic_authorization_ref

7. Art. 9 data confirmed accessible after erasure deadline:
   → Assess Art. 33: was the data accessed (by someone other than the subject) after deadline?
   → Yes → R-01 also active; Art. 33 clock starts from date of that access
   → No (data present but not accessed) → no Art. 33 obligation; document assessment

8. cv_sessions.keypoints_enc specific:
   If keypoints_enc rows remain populated after erasure → activate R-11 alongside R-14
   Biometric Art. 9 protocol in R-11 takes precedence on notification timeline
```

#### R-14.4 Missed GDPR Deadline (Art. 15/20)

Statutory deadline: **30 calendar days** from the date of the verified DSAR request. One-time extension to **90 days** is permitted for complex requests, but the data subject must be notified within the original 30-day window that the extension is being applied and why.

```
Days 28–30 with no delivery (P2 cure window):
1. Notify compliance-officer immediately
2. Check export worker status — has the job run? Output valid?
3. If job ran but delivery failed: manually re-trigger; log to DEC-030
4. If job not yet run: trigger manually; monitor to completion
5. Document delay cause in Linear DSAR register
6. No DPA self-report required if delivered before the hard deadline

Day 30 passed with no delivery (P1 — missed):
1. Open #inc-YYYYMMDD-dsar; notify compliance-officer within 1 hour
2. Deliver the export as soon as technically possible
3. Draft Template D-01 (DPA voluntary self-report)
4. Notify data subject of the delay (Template D-02)
5. Compliance-officer decides on DPA self-report:
   → GDPR Art. 12(3) imposes no explicit self-report obligation for missed DSARs
   → Self-reporting demonstrates cooperation (Art. 83(2)(f)) and reduces fine exposure
   → Default: self-report for any Art. 9 DSAR with a missed deadline
   → Compliance-officer has authority to self-report without founder sign-off
     when no data breach is involved
```

#### R-14.5 Bulk DSAR / Coordinated Abuse (P2 → P1)

GDPR Art. 12(5) permits FORM to refuse or charge a reasonable fee for requests that are "manifestly unfounded or excessive, in particular because of their repetitive character."

```
Trigger threshold: ≥ 10 DSAR submissions from distinct accounts in 24 hours
sharing the same IP range, employer, or coordinated submission pattern

1. Flag requests in DSAR register as suspected coordinated
2. Do NOT automatically reject — each request requires individual assessment
3. Notify compliance-officer; they decide on Art. 12(5) refusal strategy
4. If refusing on Art. 12(5) grounds:
   - Each refusal must state the reason in writing
   - User retains the right to complain to the DPA (Art. 77)
   - Use Template D-03 (Art. 12(5) refusal notice)
5. Document volume, source analysis, and refusal rationale in Legal folder
6. If coinciding with a social media campaign or activist group:
   - Notify founder and customer-success (reputational risk)
   - Do NOT characterise as abuse in any external communication
7. If ≥ 5 requests approach day 28 simultaneously → escalate to P1:
   data-engineer prioritises export worker capacity
```

#### R-14.6 Enterprise Employee DSAR

Under FORM's enterprise licence, the employer is the data controller for employee fitness data. FORM is the data processor. This creates a joint-controller complexity when an enterprise employee submits a DSAR directly to FORM.

```
FORM's position:
- Art. 15/20 DSARs run against the controller (the employer), not the processor
- FORM should redirect the employee to their employer but cannot refuse to acknowledge

Protocol:
1. Acknowledge receipt in writing within 5 business days
2. Notify the enterprise tenant IT admin or DPO via Template E-DSAR-01
3. Compliance-officer assesses:
   a. Is FORM also an independent controller for any data? (e.g., support chat, billing data)
   b. If yes: FORM must fulfill the DSAR for the data it independently controls
   c. If no: redirect employee to their employer with Template D-04

4. Do NOT produce the employee's workout, coaching, or health data without
   explicit written instruction from the employer (controller) or a valid legal order

5. Privacy floor — if the employer instructs FORM to withhold data FROM the employee:
   - FORM cannot be party to suppressing an individual's Art. 15 right of access
   - If employer objects and FORM cannot fulfill without instruction:
     escalate to compliance-officer + external counsel
   - Do not comply with a blocking instruction that extinguishes the data subject's rights
```

#### R-14.7 Communication Templates

**Template D-01 — DPA Voluntary Notification (Missed DSAR Deadline)**
```
To: [competent DPA — see §10 Regulatory Obligations for DPA contact table]
Subject: Voluntary Notification — DSAR Response Delay

We are writing voluntarily to inform [DPA name] of a delayed response to a data
subject access request received by FORM (form.coach).

The request was submitted on [date] by a data subject resident in [member state] under
Article 15 GDPR. The 30-day statutory response period elapsed on [date] without delivery.

Root cause: [brief factual description — e.g., "A configuration error in the export
processing worker caused the job to fail silently without alerting operators."]

Remediation: The data subject received their complete export on [delivery date]. We have
implemented [brief preventive control, e.g., "day-25 and day-29 Slack alerts"]. The DSAR
register has been audited; no other requests are currently overdue.

Our data protection contact: privacy@form.coach.
[compliance-officer name, title]
```

**Template D-02 — User Notification (Delayed DSAR Response)**
```
Subject: Your data request — brief delay

Hi [Name],

We received your data request on [date] and owe you an apology — we've missed
the 30-day deadline we're required to meet.

What happened: [plain-language explanation]

Your complete data export is [attached / available at the secure link below].
It includes [list categories: workout history, coaching conversations, account data, etc.].

If you have questions, contact privacy@form.coach. You also have the right to
lodge a complaint with your supervisory authority at any time:
[link to relevant DPA — BfDI/Germany, DPC/Ireland, ICO/UK, etc.]

[compliance-officer / privacy team name]
```

**Template D-03 — Art. 12(5) Refusal Notice**
```
Subject: Your data request — our response

We received your data request on [date].

After assessment, we have determined that this request is one of multiple requests
submitted in a short period that, taken together, are excessive in character within
the meaning of Article 12(5) GDPR.

[If charging a fee:] We are exercising our right to charge a reasonable fee of
[amount] to process this request. Please confirm payment to privacy@form.coach to proceed.

[If refusing:] We are declining to process this request on the above grounds.
You have the right to lodge a complaint with your supervisory authority and
to seek a judicial remedy under Article 79 GDPR.

[compliance-officer name, title]
```

**Template D-04 — Employee DSAR Redirect to Employer**
```
Subject: Your data request — next step

Hi [Name],

Thank you for your request. Under your employer's FORM Enterprise licence, [TenantName]
is the data controller responsible for the employee fitness data collected during your use
of FORM as a workplace benefit.

For your request under Article 15 GDPR, please contact [TenantName]'s designated data
protection contact at [employer DPO contact if known, otherwise HR].

FORM holds some data as an independent controller (for example, billing and support
interactions not related to your workplace licence). If you wish to request that data,
please reply to this email and we will handle it directly within 30 days.

[privacy@form.coach]
```

**Template E-DSAR-01 — Enterprise Tenant Notification (Employee DSAR Received)**
```
To: [Tenant IT admin / DPO]
Subject: Data request received from an employee

We have received a data subject access request from an individual who identifies as an
employee covered under [TenantName]'s FORM Enterprise licence. The request was received
on [date] and the 30-day response window runs until [date].

As data controller for employee data under your licence, this request is primarily
directed at you. We are notifying you as required under our Data Processing Agreement.

If FORM holds data as an independent controller that also falls within scope,
we will handle that portion directly. Please instruct us within 5 business days
if you have any direction on the employer-controlled data.

Contact: enterprise@form.coach
[Customer Lead name, FORM Enterprise]
```

#### R-14.8 DEC-030 Audit Events

| Event Type | Severity | Trigger | Required Metadata |
|---|---|---|---|
| `dsar.request_received` | MEDIUM | New DSAR submitted | `request_type`, `user_id`, `jurisdiction`, `dsar_id` |
| `dsar.identity_verified` | LOW | ID verification completed | `dsar_id`, `verification_method` |
| `dsar.export_initiated` | MEDIUM | Export job started | `dsar_id`, `export_job_id` |
| `dsar.export_delivered` | MEDIUM | Secure link sent to requester | `dsar_id`, `delivery_method`, `link_expiry` |
| `dsar.export_link_revoked` | CRITICAL | Link revoked — contamination discovered | `dsar_id`, `reason`, `was_downloaded` |
| `dsar.export_redelivered` | HIGH | Corrected export delivered after revocation | `dsar_id`, `correction_reason` |
| `dsar.erasure_initiated` | HIGH | Erasure job started | `dsar_id`, `user_id`, `tables_in_scope` |
| `dsar.erasure_completed` | HIGH | All tables confirmed empty | `dsar_id`, `tables_erased`, `rows_deleted_total` |
| `dsar.erasure_manual_supplement` | CRITICAL | Manual erasure by ops after job failure | `dsar_id`, `table_name`, `row_count`, `operator_id`, `ic_authorization_ref` |
| `dsar.deadline_missed` | HIGH | Day 30 passed without delivery | `dsar_id`, `deadline`, `current_status` |
| `dsar.extension_applied` | MEDIUM | 90-day extension applied (Art. 12(3)) | `dsar_id`, `reason`, `new_deadline` |
| `dsar.rejected_art_12_5` | HIGH | Request refused as manifestly excessive | `dsar_id`, `reason`, `legal_basis` |
| `dsar.dpa_notified` | HIGH | Voluntary DPA self-report sent | `dsar_id`, `dpa_name`, `notification_method` |
| `dsar.enterprise_tenant_notified` | MEDIUM | Enterprise admin notified of employee DSAR | `dsar_id`, `tenant_id`, `notification_timestamp` |
| `dsar.data_provided` | HIGH | Enterprise processor-side: export package made available to Customer (Art. 15; ENTERPRISE_SLA §19.1 — SLO: ≤ 24 h) | `request_id`, `requester_tenant_id`, `export_sha256`, `sla_hours` (float — hours from forwarded request to provision), `slo_met` (bool: `sla_hours ≤ 24.0`) |
| `dsar.deletion_soft` | HIGH | Soft delete executed; employee account access revoked (Art. 17 step 1; ENTERPRISE_SLA §19.2) | `request_id`, `tenant_id`, `soft_deleted_at` (ISO 8601); **privacy floor:** no `user_id` in payload — cross-ref via `form_system`-only RLS |
| `dsar.deletion_confirmed` | HIGH | Hard-delete cascade complete; deletion certificate issued to tenant (Art. 17 step 2; ENTERPRISE_SLA §19.2 — SLO: certificate within 30 days of soft delete) | `request_id`, `tenant_id`, `rows_deleted_count` (aggregate), `certificate_ref` (UUID — maps to written certificate sent to `tenant_ops_email`) |
| `dsar.portability_export_completed` | STANDARD | Employee self-serve Art. 20 portability export fulfilled (ENTERPRISE_SLA §19.3) | `user_id_hash` (SHA-256 — **never plaintext user_id**), `export_size_bytes`, `tables_included` (string array), `delivery_method` (`in_app_download` \| `email_link`) |
| `dsar.offboarding_export_available` | STANDARD | Tenant contract-end bulk export package ready for admin download (ENTERPRISE_SLA §19.4 — window opens within 24 h of contract termination) | `tenant_id`, `export_package_id` (UUID), `expires_at` (ISO 8601 — 30-day window), `seat_count` (integer) |

> **Retention (ENTERPRISE_SLA §19.5):** `dsar.data_provided`, `dsar.deletion_soft`, `dsar.deletion_confirmed` — 7 years. `dsar.portability_export_completed`, `dsar.offboarding_export_available` — 7 years. All five events HMAC-chained per DEC-030. Cross-ref: `docs/ENTERPRISE_SLA.md §19.5` (canonical event definitions); `docs/AUDIT_LOG_SCHEMA.md` (Zod schema registration required — see R-14.10 checklist item 9).

#### R-14.9 SOC 2 Evidence Mapping

| SOC 2 Criterion | Evidence Type | Source |
|---|---|---|
| **P4.0** (Data subject inquiries and complaints) | DSAR register with submission, status, and resolution timestamps | Linear DSAR register + `dsar_requests` table |
| **P5.0** (Data subject requests) | Fulfilled export packages with delivery timestamps | DEC-030 `dsar.export_delivered` events |
| **P5.1** (Consent and choice) | Erasure completion confirmations | DEC-030 `dsar.erasure_completed` + scope query results |
| **P8.0** (Disposal) | Manual erasure supplementation log | DEC-030 `dsar.erasure_manual_supplement` events |
| **CC2.2** (External communications) | DPA notifications, user delay notifications | Template D-01, D-02 sent copies in evidence folder |
| **CC6.5** (Access controls — disposal) | Post-erasure verification query showing zero rows | SQL output saved to `compliance/evidence/dsar/` |

Evidence package location: `compliance/evidence/dsar/YYYY-MM/<dsar_id>/`

```
request/          — original DSAR submission (redacted copy)
verification/     — identity verification record
export/           — manifest of delivered data (not the export archive itself)
erasure/          — post-erasure verification query output + SHA-256 manifest
comms/            — all emails sent to data subject and DPA
dec030/           — DEC-030 event export for this dsar_id
```

Retention: 7 years per DEC-030 policy. Evidence folder is read-only after case closure.

#### R-14.10 Post-Incident Preventive Controls

| Control | Owner | Verification Method |
|---|---|---|
| Day-25 and day-29 DSAR deadline Slack alerts | data-engineer | Check pg_cron + Slack alert config |
| Monthly end-to-end export worker test (synthetic DSAR) | qa-lead | CI canary job in Linear |
| Post-erasure verification query (automated, runs 1h after erasure job) | platform-engineer | Check job exists; alerts on non-zero count |
| Cross-tenant export isolation regression test | qa-lead | Test asserts export zip contains only requester's `user_id` |
| Weekly DSAR register review | compliance-officer | Linear [Privacy] project weekly cadence |
| `cv_sessions.keypoints_enc` NULL check after erasure | security-engineer | Separate automated check; alerts to #security |

#### R-14.11 Tabletop Scenario I

**Scenario title:** DSAR Export Cross-Tenant Contamination — Enterprise Scale

**Setup:**
A data engineer pushed an update to the DSAR export worker with a query bug: instead of filtering by `user_id = $REQUESTER`, the query filters by `tenant_id = $TENANT_ID` with no user-level isolation. A Growth-tier enterprise customer (420 users, UK and Ireland) had three employees submit DSARs in the same week. Each received an export containing data for all 420 users in the tenant. All three archives were downloaded; one employee has already opened the file and emails FORM: "I can see everyone's data including health information."

**Participants:** IC, security-engineer, compliance-officer, data-engineer, enterprise-architect, Customer Lead (CS).

**Discussion points:**

1. Severity classification — which runbooks are simultaneously active?
2. First action and who takes it — can the export links be revoked? For which of the three?
3. All three archives have been downloaded. Scope of the data breach — is this Art. 33-reportable?
4. Health data (`health_profiles`, `coaching_turns`) is in the contaminated exports — does this trigger Art. 34 individual notification for all 420 affected employees?
5. The enterprise tenant has 420 employees whose data was in the wrong export — how do you notify them, and who owns that communication?
6. The tenant admin demands to know "whose data went to whom" — can you share the three requester names with the admin?
7. The ICO must be notified within 72 hours — who files, what does the initial filing contain, and is a partial filing acceptable at T+48h?
8. The employee's email ("I can see everyone's data") is a witness statement — how does this affect evidence preservation protocol?
9. Can FORM re-run the three original DSARs with the fixed worker? What is the corrected-export re-delivery protocol?
10. One year later during the SOC 2 Type II audit — what evidence in `compliance/evidence/dsar/` covers this incident?

**Target duration:** 75 minutes. **Facilitator:** compliance-officer. **Required:** data-engineer present for root-cause analysis portion. **Output:** updated cross-tenant export regression test; updated R-14 post-incident controls; one §14 PIR action item minimum.

---

### R-15: Health Data Field Detected in Error Payload (GDPR Art. 9)

**Trigger:** `FORM-HEALTH-LEAK-001` Sentry alert fires — tag `health_field_detected: true` detected on a Sentry event. The `beforeSend` scrubber (§47.5 of `docs/SOC2_READINESS.md`) fired and emitted a `system.sentry_health_field_detected` DEC-030 CRITICAL event.

**Severity:** P0. Any detection of Art. 9 health data (HRV, heart rate, weight, body fat, sleep score, readiness, SpO₂, VO₂max) in an error payload is a GDPR Art. 9 potential breach until investigation proves otherwise.

**Why this matters:** Sentry receives error events from production. If a health field name appears in `event.request.data`, `event.extra`, or `event.contexts` before the `beforeSend` hook scrubs it, the scrubber catches and redacts it before transmission — but the presence of the field indicates a code path that may be inadvertently logging Art. 9 data to error payloads. The scrubber prevents the leak to Sentry; this runbook investigates the upstream cause.

#### Immediate Actions (first 15 minutes)

```
1. Open incident channel: #inc-YYYYMMDD-health-leak
2. Page: IC + security-engineer + compliance-officer simultaneously
3. Query DEC-030 audit log for the triggering event:
   SELECT id, created_at, actor_id, metadata
   FROM audit_logs
   WHERE action = 'system.sentry_health_field_detected'
   ORDER BY created_at DESC
   LIMIT 10;
4. Note: metadata.field_names contains which health fields were detected
5. Note: metadata.sentry_event_id links to the specific Sentry issue
6. DO NOT dismiss the Sentry alert — preserve it as evidence
```

#### Investigation (T+0h–T+4h)

```
1. Open the Sentry issue linked in the DEC-030 event metadata
2. Review the scrubbed event: what endpoint / Worker route emitted it?
3. Determine whether the field reached Sentry before beforeSend ran:
   - beforeSend fires before transmission; scrubbed event = field was in the event object
   - The field was NOT transmitted to Sentry in plaintext
   - But: was it logged anywhere else? (Cloudflare Logpush, Supabase logs, console.error)
4. Search Cloudflare Logpush for the same timestamp range:
   SELECT * FROM cloudflare_logs WHERE timestamp BETWEEN T-5m AND T+5m
   AND (message LIKE '%hrv%' OR message LIKE '%heart_rate%' OR message LIKE '%weight_kg%');
5. Check Supabase Edge Function logs for the same window
6. Identify the code path: what caused the health field to appear in the error context?
   Common causes:
   a. Developer passed a full user object to Sentry.setExtra() — contains health fields
   b. A request body containing health data triggered an unhandled exception
   c. A structured log statement included health data in error metadata
```

#### GDPR Assessment (T+4h)

```
If investigation concludes the field reached Sentry before scrubbing:
  → GDPR Art. 33 72-hour clock starts at awareness time (DEC-030 event timestamp)
  → Engage DPO and outside counsel immediately
  → File partial Art. 33 notification if root-cause not resolved within 48h
  → Follow R-01 (Data Breach) procedure from GDPR assessment step onward

If investigation confirms field was scrubbed and not transmitted:
  → No Art. 33 obligation (no personal data reached processor)
  → Document conclusion in incident channel with supporting evidence
  → File incident as "contained — no breach" in compliance/evidence/
  → Required: code fix to prevent recurrence + post-incident review
```

#### Code Fix (required in all cases)

```
1. Identify the code path that introduced the health field into error context
2. Fix: never pass raw user objects, workout objects, or health metric objects to:
   - Sentry.setExtra() / Sentry.setContext()
   - console.error() / console.log() with structured data
   - Any logging path that may be captured by error monitoring
3. Add a unit test to the Vitest scaffold (§47.5) covering this specific field
4. Deploy fix + confirm FORM-HEALTH-LEAK-001 does not fire in staging
```

#### Evidence Preservation

```
1. Export DEC-030 events covering the incident window
2. Export Sentry issue (scrubbed event JSON) as evidence
3. If no breach: compliance/evidence/cc7/R15-YYYY-MM-DD-contained.md
4. If breach: follow R-01 evidence preservation protocol
```

#### Post-Incident Review (mandatory)

1. How did a health field reach the error payload? Code review of the fix.
2. Are there other code paths that could produce the same result? Full audit.
3. Is the `beforeSend` scrubber test suite covering this field name? Add test if not.
4. Update `system.sentry_health_field_detected` DEC-030 event `metadata.field_names` with any new fields discovered.

**Runbook owner:** security-engineer. **Escalation:** compliance-officer (Art. 33 assessment). **GDPR clock owner:** compliance-officer.

*Added v1.9 (2026-05-31) — referenced by SOC2_READINESS.md §47.4 FORM-HEALTH-LEAK-001 alert rule.*

---

### R-16: Application Secret & Encryption Key Exposure

> **Scope:** Triggered when a Workers Secret, application encryption key, or third-party API credential is discovered to have been exposed — via TruffleHog CI, GitHub Advanced Security (GHAS), bug bounty report, or internal discovery. Distinct from R-03 (user credential compromise), R-04 (SAML certificate), R-08 (supply chain attack — rotation is remediation there), R-09 (ransomware), R-11 (CV biometric breach — activated in parallel when `keypoints_enc` is affected). References: `docs/CRYPTOGRAPHY_POLICY.md`, SOC2_READINESS.md §50 (TruffleHog), SSO_SCIM_IMPLEMENTATION.md §24 (PAM break-glass for destructive DB operations).

#### Trigger Matrix

| Secret | Storage | Blast Radius | Severity |
|---|---|---|---|
| `keypoints_enc` (AES-256-GCM key for CV biometric data) | Cloudflare Workers Secret | All `cv_sessions.keypoints_enc` decryptable — GDPR Art. 9 biometric data | **P0** — activate R-11 in parallel |
| `SUPABASE_JWT_SECRET` (JWT signing key) | Supabase project setting | All user sessions forgeable; enterprise admin sessions at risk | **P0** |
| Supabase service role key (`SUPABASE_SERVICE_ROLE_KEY`) | Cloudflare Workers Secret | BYPASSRLS — full DB read/write access across all tenants | **P0** — activate R-01 immediately |
| SCIM bearer token master secret (`SCIM_TOKEN_SIGNING_SECRET`) | Cloudflare Workers Secret / KV | All SCIM integrations impersonatable; employee directory accessible | **P0** |
| HMAC audit chain signing key (`AUDIT_HMAC_KEY`) | Cloudflare Workers Secret | Audit log integrity unverifiable; SOC 2 chain evidence invalidated | **P1** |
| Stripe webhook signing secret (`STRIPE_WEBHOOK_SECRET`) | Cloudflare Workers Secret | Fraudulent Stripe events injectable; subscription manipulation | **P1** |
| Anthropic API key (`ANTHROPIC_API_KEY`) | Cloudflare Workers Secret | AI API billing fraud; no user data exposure without DB access | **P1** |
| Compliance attestation signing key (`ATTESTATION_SIGNING_KEY`) | Cloudflare Workers Secret | `tenant_deletion_attestations` HMAC signatures forgeable | **P1** |
| ElevenLabs API key (`ELEVENLABS_API_KEY`) | Cloudflare Workers Secret | TTS billing fraud; no user data exposure | **P2** |
| PostHog project key (`POSTHOG_API_KEY`) | Cloudflare Workers Secret | Analytics data poisoning; no GDPR Art. 9 data in PostHog | **P2** |
| Sentry DSN (`SENTRY_DSN`) | Workers Secret / client bundle | Attacker can inject fake error events; no outbound PII risk if scrubber active | **P2** |

**Severity upgrade rule:** Any P1 or P2 secret is upgraded to P0 if (a) the exposure window assessment (below) confirms unauthorized access, or (b) the secret was in a public repository for > 24 hours.

#### Trigger Events

1. **GitHub Advanced Security (GHAS) push alert** — credential detected before or after commit reaches GitHub
2. **TruffleHog CI failure** — `trufflehog-scan` CI job detected a secret pattern (SOC2_READINESS.md §50 / CC5-GAP-003)
3. **Cloudflare Audit Log anomaly** — Workers Secret read from unexpected IP or service account (AL-SECRETS-01)
4. **External security researcher / bug bounty report** — credential found in public repository, pastebins, or leaked logs
5. **Team member discovery** — production credentials found in a screenshot, shared document, or Slack message
6. **Vendor usage anomaly** — unexpected spike in Anthropic, ElevenLabs, or Stripe API usage in the Anthropic/vendor console

#### Immediate Actions (T+0 to T+15 min)

```
CRITICAL: DO NOT rotate immediately — rotation order matters and some rotations
cause immediate user impact (JWT secret rotation = ALL sessions invalidated).

1. Open restricted incident channel: #inc-YYYYMMDD-secrets
   Members: IC + security-engineer + compliance-officer + founder only
   Do NOT add HR or engineering team until legal review (R-12 insider exception
   applies if internal team member caused the exposure)

2. Identify WHICH secret was exposed (from trigger event details)

3. Determine EXPOSURE WINDOW: when was the secret first accessible?
   a. Git-committed credential:
      git log --all --full-history -S "ANTHROPIC_API_KEY" --format="%H %ci" | head -5
   b. GHAS alert: use GHAS first_detected_at timestamp
   c. Public repo: assume exposure from commit timestamp (GitHub indexes within minutes)

4. Open Linear ticket [R16-YYYYMMDD]; emit `incident.opened` DEC-030 event
5. P0: page founder + security-engineer simultaneously
6. P1: page security-engineer; notify founder within 30 min
7. P2: security-engineer handles in business hours unless exposure window > 24h
```

#### Exposure Window Assessment (T+15 to T+60 min)

**Goal: determine whether the secret was USED (accessed), not just exposed.**

**Supabase service role key** — check for unexpected access:
```sql
-- Run via Supabase Dashboard → Logs → API Logs
-- Any user_agent not matching expected FORM service accounts = R-01 immediately
SELECT timestamp, path, method, status_code, user_agent
FROM   supabase_api_logs
WHERE  timestamp BETWEEN $exposure_start AND $exposure_end
  AND  user_agent NOT LIKE 'FORM-Worker/%'
ORDER  BY timestamp;
```

**Anthropic API key** — check Anthropic Console → Usage for the exposure window:
```
→ Look for: unexpected IPs, model versions FORM does not use, requests outside
  expected volume patterns
→ Estimate financial impact: unauthorized tokens × current Anthropic pricing
→ No user data exposure from Anthropic key alone (Victor sessions require DB access too)
```

**JWT signing secret** — check for forged sessions:
```sql
SELECT tenant_id, user_id, created_at, ip_address, user_agent
FROM   enterprise_sessions
WHERE  created_at BETWEEN $exposure_start AND $exposure_end
  AND  authenticated_via = 'direct'   -- non-SSO sessions most at risk of forgery
ORDER  BY created_at;
-- Any session with an unexpected IP not matching user's prior pattern?
-- If yes: activate R-03 for those users + enterprise tenants
```

**keypoints_enc encryption key** — biometric exposure is conditional on DB access:
```
→ The key alone does not decrypt data — attacker also needs to access cv_sessions.
→ Was Supabase accessible during the exposure window (via service role key or RLS bypass)?
→ If YES: treat as R-11 + GDPR Art. 33 clock starts at awareness time.
→ If NO confirmed DB access: rotation is required but Art. 33 may not be triggered.
   Document this assessment with evidence in the incident channel.
```

#### Rotation Sequence

**Order is critical. Follow this sequence when multiple secrets are exposed simultaneously:**

| Step | Secret | User Impact | Notes |
|---|---|---|---|
| 1 | Supabase service role key | None | Assess R-01 before or in parallel |
| 2 | SCIM bearer token master secret | SCIM re-provision cycle per tenant | Notify CSM team |
| 3 | Anthropic API key | None | Parallel with step 2 |
| 4 | ElevenLabs / PostHog / Sentry keys | None | Parallel with step 3 |
| 5 | Stripe webhook signing secret | None | Update in Stripe Dashboard + Workers Secret |
| 6 | Compliance attestation signing key | None | Old key to cold storage; document chain-of-custody |
| 7 | HMAC audit chain signing key | SOC 2 chain epoch break | See §R-16 special procedure; compliance-officer approval required |
| 8 | `keypoints_enc` encryption key | CV feature temporarily disabled | See §R-16 biometric re-encryption; R-11 parallel |
| 9 | JWT signing secret | **ALL user sessions invalidated** | Last; 30 min pre-notification to enterprise tenants |

**Never reorder without unanimous IC + security-engineer + compliance-officer agreement.**

#### HMAC Audit Chain Key Rotation (Special Procedure)

Rotating `AUDIT_HMAC_KEY` creates a **chain epoch boundary**. This is NOT the same as an R-05 (unintended chain break) — it is a planned, documented transition.

```
Before rotating AUDIT_HMAC_KEY:
1. Export and verify the current chain:
   SELECT verify_hmac_chain(
     start_seq := (SELECT MIN(seq) FROM audit_log_events),
     end_seq   := (SELECT MAX(seq) FROM audit_log_events)
   ) AS chain_verified;
   → Save output: compliance/evidence/audit-chain/pre-rotation-YYYYMMDD.json

2. Compliance-officer signs off on the epoch transition:
   File: compliance/evidence/audit-chain/epoch-transition-YYYYMMDD.md
   Content: reason, IC identity, compliance-officer identity, last pre-rotation seq, timestamp

3. Generate new key: openssl rand -hex 32
   Deploy as AUDIT_HMAC_KEY in Cloudflare Workers Secrets

4. Store old key in cold storage (encrypted, access-controlled) for 7 years
   Required to verify pre-rotation chain events during future SOC 2 audits

5. Emit `secret.hmac_epoch_transitioned` DEC-030 event with:
   old_epoch_last_seq, new_epoch_first_seq, transition_reason, authorised_by

6. Update docs/AUDIT_LOG_SCHEMA.md §5 to document the epoch boundary
   (Auditors will see a verification gap without this cross-reference)
```

#### `keypoints_enc` Re-Encryption (biometric data)

Always activate R-11 in parallel. Requires PAM `destructive` tier elevation (SSO_SCIM_IMPLEMENTATION.md §24) and two-person authorisation before execution.

```typescript
// workers/cv-key-migration/index.ts — triggered by form_admin via PAM elevation only
async function reEncryptKeypoints(
  oldKey: string,
  newKey: string,
  db: SupabaseClient,
  incidentId: string,
  rotationStartedAt: string,
) {
  const BATCH_SIZE = 500;
  let cursor = 0;
  let totalReEncrypted = 0;

  while (true) {
    const { data: rows } = await db
      .from('cv_sessions')
      .select('id, keypoints_enc')
      .not('keypoints_enc', 'is', null)
      .range(cursor, cursor + BATCH_SIZE - 1);

    if (!rows || rows.length === 0) break;

    for (const row of rows) {
      const plaintext     = await aesGcmDecrypt(row.keypoints_enc, oldKey);
      const newCiphertext = await aesGcmEncrypt(plaintext, newKey);
      await db.from('cv_sessions').update({ keypoints_enc: newCiphertext }).eq('id', row.id);
    }
    totalReEncrypted += rows.length;
    cursor += BATCH_SIZE;
    if (totalReEncrypted % 5_000 === 0) {
      await emitAuditEvent(db, {
        event_type: 'secret.biometric_re_encryption_progress',
        severity:   'STANDARD',
        metadata:   { rows_completed: totalReEncrypted, incident_id: incidentId },
      });
    }
  }

  // Verify: no row remains encrypted with old key (check updated_at)
  const { count } = await db
    .from('cv_sessions')
    .select('id', { count: 'exact' })
    .not('keypoints_enc', 'is', null)
    .lt('updated_at', rotationStartedAt);

  if (count !== 0) throw new Error(`Re-encryption incomplete: ${count} rows not migrated`);

  await emitAuditEvent(db, {
    event_type: 'secret.biometric_re_encryption_completed',
    severity:   'CRITICAL',
    metadata:   {
      incident_id:                        incidentId,
      rows_re_encrypted:                  totalReEncrypted,
      zero_remaining_old_key_rows_verified: true,
    },
  });
}
```

After re-encryption: remove `KEYPOINTS_ENC_OLD` Workers Secret. Confirm all `cv_sessions.keypoints_enc` rows have `updated_at ≥ rotation_started_at`.

#### JWT Signing Secret Rotation

```
Pre-rotation (30 min advance notice when timeline allows — immediate if P0):
→ Send Template E-ROTATE-01 to all enterprise tenant admins (§6 Communication Templates):
  Subject: "FORM planned authentication maintenance [DATE] [TIME UTC]"
  Body:    "A brief maintenance window requires all users to re-authenticate.
            SSO users are re-authenticated automatically via your identity provider.
            Non-SSO users will need to log in again. Duration: ~5 minutes."

Rotation:
1. Supabase Dashboard → Project Settings → API → JWT Secret → Rotate
2. Update SUPABASE_JWT_SECRET in all Cloudflare Workers: wrangler deploy --env production
3. Confirm all Workers serving new JWT verification within 2 minutes

Post-rotation monitoring (T+0 to T+30 min after rotation):
→ Monitor OBSERVABILITY.md §13 SSO error rate dashboard
→ Expected: authentication surge (normal — sessions re-establish)
→ Alert threshold: SSO error rate > 5% sustained for > 5 min post-rotation
  → Page security-engineer; Workers Secret propagation issue likely
```

#### GDPR Assessment

| Secret Exposed | Art. 9 Data at Risk? | Art. 33 Required? | Action |
|---|---|---|---|
| `keypoints_enc` key only, no confirmed DB access | Possible | Case-by-case — consult compliance-officer | Art. 33 assessment within 12h; partial filing at T+48h if uncertain |
| `keypoints_enc` + confirmed DB access | **YES** — biometric data | **YES** | R-01 + R-11 immediately; GDPR 72h clock owner: compliance-officer |
| Supabase service role key (any exposure) | **YES** — BYPASSRLS across all tables | **YES — presumed breach** | R-01 immediately; Art. 34 assessment |
| JWT secret exposed, unauthorized sessions found | Indirect (session forgery enables access) | Assessment required | Investigate sessions; if forged: Art. 33 clock starts |
| Anthropic / ElevenLabs / PostHog / Sentry | NO | NO | No Art. 33 obligation; document in evidence |
| HMAC chain key | NO (chain key ≠ data key) | NO | Epoch documentation; notify audit firm if SOC 2 observation period active |
| Stripe webhook secret | NO | NO | Financial fraud assessment only |

#### DEC-030 Audit Events

| Event Type | Severity | Retention | Key Metadata |
|---|---|---|---|
| `secret.exposure_discovered` | CRITICAL | 7 years | `secret_type`, `exposure_vector`, `exposure_window_start_utc`, `access_confirmed` |
| `secret.rotation_initiated` | HIGH | 7 years | `secret_type`, `initiated_by`, `estimated_user_impact` |
| `secret.rotation_completed` | HIGH | 7 years | `secret_type`, `rotation_completed_at`, `re_encryption_required` |
| `secret.biometric_re_encryption_progress` | STANDARD | 3 years | `rows_completed`, `incident_id` |
| `secret.biometric_re_encryption_completed` | CRITICAL | 7 years | `rows_re_encrypted`, `zero_remaining_old_key_rows_verified` |
| `secret.hmac_epoch_transitioned` | CRITICAL | 7 years | `old_epoch_last_seq`, `new_epoch_first_seq`, `authorised_by` |
| `secret.exposure_window_assessed` | HIGH | 7 years | `access_confirmed`, `gdpr_art33_triggered`, `assessment_rationale` |

**Sub-chain rule:** All `secret.*` events form a sub-chain within the master HMAC chain, keyed by `incident_id`. A break in this sub-chain is treated as P0 per R-05.

#### Evidence Package

```
compliance/evidence/incident-comms/<incident-slug>/
├── discovery/
│   ├── trigger-artifact.json       # GHAS alert export / TruffleHog output / researcher report
│   ├── exposure-window.md          # git log / vendor usage log analysis
│   └── access-confirmed.md         # yes/no + supporting evidence per secret type
├── rotation/
│   ├── rotation-log.md             # timestamp + actor + secret_type per rotation step
│   ├── tenant-notifications.md     # E-ROTATE-01 delivery confirmation (JWT rotation)
│   ├── biometric-migration.md      # if keypoints_enc rotated: row counts + verification query
│   └── hmac-epoch-transition.md    # if AUDIT_HMAC_KEY rotated: epoch boundary + chain export
├── gdpr/
│   ├── art33-assessment.md         # mandatory for P0 secrets
│   └── breach-notification/        # if Art. 33 triggered — follow §15
└── dec030/
    └── secret-events-chain.json    # all secret.* DEC-030 events for this incident_id
```

#### Post-Incident Preventive Controls

1. **TruffleHog CI** (SOC2_READINESS.md §50): confirm `trufflehog-scan` covers Cloudflare Worker config files and `wrangler.toml`, not only `.env` files
2. **GHAS push protection**: verify enabled on the repository — secrets blocked before reaching GitHub's servers
3. **Pre-commit hook** (`detect-secrets`): install on all engineer machines; blocks committed secrets before `git push`
4. **Cloudflare Audit Log alert `AL-SECRETS-01`** (P1): Workers Secret read from IP outside expected Cloudflare CI/CD range → PagerDuty
5. **CRYPTOGRAPHY_POLICY.md** rotation schedule review: update proactive rotation cadence for the affected secret type based on this incident
6. **Developer debrief**: engineer involved in the exposure participates in PIR and refreshes secret handling training

#### SOC 2 Evidence Mapping

| Criterion | Control | Evidence Artefact |
|---|---|---|
| **CC6.4** — Credential lifecycle and revocation | Rotation procedure per key type; rotation log with timestamps and actor IDs | `secret.rotation_completed` DEC-030 events; `compliance/evidence/.../rotation-log.md` |
| **CC7.1** — Vulnerability identification | TruffleHog CI + GHAS push protection (preventive) + R-16 runbook (detective/corrective) | TruffleHog CI green build artefacts; GHAS alert configuration screenshot |
| **CC7.4** — Incident response procedures | `secret.exposure_discovered` → `secret.rotation_completed` HMAC chain = auditable timeline | DEC-030 sub-chain for `incident_id` |
| **CC5.2** — Technology controls include monitoring | AL-SECRETS-01 (Cloudflare Workers Secret access monitoring) = automated credential monitoring | PagerDuty AL-SECRETS-01 rule configuration; Cloudflare audit log screenshot |

#### Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Register all 7 `secret.*` DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` event registry with severity and retention | security-engineer | **P0** | M4 |
| Add Template E-ROTATE-01 (JWT rotation advance notice) to §6 Communication Templates in this document | compliance-officer | **P0** | M4 |
| Configure Cloudflare Audit Log → PagerDuty alert `AL-SECRETS-01` (P1): Workers Secret reads from IP outside expected Cloudflare CI/CD IP range | devops-lead | **P1** | M4 |
| Scaffold `workers/cv-key-migration/index.ts` per §R-16.6 pseudocode; implement PAM `destructive` tier gate per SSO_SCIM_IMPLEMENTATION.md §24; add two-person authorisation check before execution | platform-engineer | **P1** | M5 |
| Add Scenario K to §9 tabletop drill catalog: *Anthropic API key committed to public GitHub fork; 3-hour exposure window; no confirmed unauthorized usage; discovered via GHAS alert on a Friday at 18:42 UTC; on-call engineer is junior; compliance-officer is on PTO* | security-engineer + compliance-officer | **P1** | M5 |
| Confirm `detect-secrets` pre-commit hook documented in `docs/ENGINEERING_RUNBOOK.md` onboarding checklist | devops-lead | **P2** | M5 |
| Cross-reference R-16 from R-11 §R-11.3 (keypoints_enc re-encryption section) and R-08 §R-08.5 (full secrets rotation in supply chain scenario) | security-engineer | **P2** | M5 |

---

### R-17: Third-Party AI Service Outage (Anthropic API / ElevenLabs Unavailability)

> **Scope:** Victor (FORM's AI coach) depends on the Anthropic Claude API for all coaching intelligence. ElevenLabs provides text-to-speech for voice coaching. This runbook covers service *availability* failures at either vendor — it is NOT a security breach runbook. For a security breach at a sub-processor (e.g., Anthropic reporting unauthorized data access), activate R-07 instead. For the Anthropic API key being exposed, activate R-16. References: §11.2 sub-processor registry (Anthropic, ElevenLabs), §12 Enterprise Tenant SLA Breach & Incident Communication Protocol, OBSERVABILITY.md §18 (AI service health monitoring).

#### Trigger

Alert `FORM-AI-OUTAGE-001` fires from the Cloudflare Workers health-check Worker when the Anthropic API (`api.anthropic.com`) returns HTTP 529, 503, or 500 for **five or more consecutive health-check cycles** (5 minutes at 1-minute cadence). Alert `FORM-VOICE-OUTAGE-001` fires when the ElevenLabs TTS endpoint returns 5xx for **ten or more consecutive cycles** (10 minutes). Both alerts route to PagerDuty → `#ops-ai-status` Slack channel → on-call platform-engineer.

#### Severity Classification

| Condition | Severity | Reason |
|---|---|---|
| Anthropic API down (any confirmed duration) | **P1** | Victor non-functional for all users; core product value destroyed |
| Anthropic API down AND enterprise tenant has an active workout session in progress | **P0** | Live safety-critical coaching interrupted; enterprise SLA breach likely; clinical-safety gate cannot run |
| Anthropic API down + ElevenLabs down simultaneously | **P1** | Compound failure accelerates enterprise SLA breach assessment; same P0 upgrade rule applies |
| Anthropic partial degradation (P95 latency on `coaching_turns` > 30s) | **P2** | Degraded but functional; monitor closely for upgrade |
| ElevenLabs TTS down, Anthropic API healthy | **P2** | Voice output degraded; text coaching intact; no safety-critical path affected |

**P0 upgrade trigger:** If at any point during a P1 outage a platform-engineer confirms via scope SQL (below) that enterprise tenant users have active workout sessions, the IC must upgrade to P0 immediately, page founder + customer-success, and follow the enterprise communication SLA in §12.

#### Why This Matters

Victor is the product. Anthropic's API is not an enhancement — it is the mechanism by which FORM delivers its core value proposition. If the Anthropic API is unavailable, FORM cannot create `coaching_turns`, cannot generate workout programs, and cannot run RPE/fatigue check-ins. For enterprise customers with active wellness programs or team training sessions, an extended outage triggers contractual SLA obligations per §12.

Unlike infrastructure incidents (R-02, R-09), FORM cannot remediate Anthropic's unavailability by deploying a fix. This runbook covers detection, graceful degradation, enterprise communication, and the multi-provider fallback architectural question that sits in DECISION_LOG as an open item.

#### Immediate Actions (T+0 to T+15 min)

```
1. Open incident channel: #inc-YYYYMMDD-ai-outage
   IC: platform-engineer (on-call)
   Notify: devops-lead + customer-success immediately

2. Confirm the outage source — is this FORM-specific or a global Anthropic outage?
   → Check status.anthropic.com (Anthropic status page)
   → Check status.elevenlabs.io if FORM-VOICE-OUTAGE-001 also fired
   → A global outage = FORM cannot remediate; skip to Containment
   → A FORM-specific error = investigate Workers config, IP allowlist, API key
     (if API key suspected invalid, check R-16 trigger — do NOT rotate in this runbook)

3. Confirm FORM's error rate in PostHog:
   → Dashboard: "AI Coach Health" → coaching_turns failure rate (last 5 minutes)
   → Threshold for incident confirmation: > 50% failure rate sustained for 5 min

4. Check Sentry for Anthropic 5xx error events:
   → Project: workers/victor-edge/*
   → Filter: last 30 min, tag anthropic_http_status IN (500, 503, 529)
   → Note: is the error hitting all users, or a specific tenant / geographic region?

5. Scope assessment — active workout sessions (run as form_admin, BYPASSRLS):
```

```sql
-- Identifies active workout sessions that may be experiencing Victor silence right now.
-- Output goes to restricted incident channel only — not general engineering Slack.
SELECT
  cs.id            AS session_id,
  cs.user_id,
  u.tenant_id,
  t.display_name   AS tenant_name,
  t.billing_email  AS tenant_billing_email,
  cs.started_at,
  EXTRACT(EPOCH FROM (NOW() - cs.started_at)) / 60  AS session_age_minutes
FROM cv_sessions cs
JOIN users u  ON u.id  = cs.user_id
JOIN tenants t ON t.id = u.tenant_id
WHERE cs.ended_at IS NULL
  AND cs.started_at > NOW() - INTERVAL '30 minutes'
ORDER BY cs.started_at ASC;
```

```
6. IF active enterprise workout sessions found (query returns rows):
   → Upgrade to P0 immediately
   → Page founder + customer-success within 5 min of P0 declaration
   → Each affected session requires a system.clinical_safety_bypass DEC-030 event (see below)
```

#### Containment

**If Anthropic API down > 10 minutes:**

Set `victor_fallback_mode = true` in Cloudflare Workers environment. This degrades Victor to a pre-written response set rather than making live Anthropic API calls. Fallback mode is a Workers environment variable toggled by a platform-engineer with production access — no code deploy required.

```
wrangler secret put VICTOR_FALLBACK_MODE --env production
# value: "true"
# Confirm propagation: wrangler tail --env production | grep fallback_mode
```

**Victor fallback mode behavior (what still works):**
1. The most recent successful AI response for a user's session is cached and replayed if the user asks the same coaching question within the session window
2. Structured workout logging (sets, reps, weight entry, timer) continues to function with no AI dependency
3. Victor displays a pre-written message: "I'm having a moment — here's your planned session" and surfaces the user's programmatic workout plan from the `workout_programs` table
4. `coaching_turns` rows are NOT created — no Anthropic API calls are made, no billing impact, no cascading 5xx errors in logs

**Victor fallback mode — what does NOT work (non-negotiable):**
- Initial session personalization (first onboarding session requires a real-time Anthropic response)
- RPE / fatigue check-ins — these feed the clinical-safety gate; if Victor cannot process RPE, the safety check cannot run
- Injury-flag detection responses — Victor's response to a user reporting pain or injury requires a live Anthropic call; fallback mode cannot safely substitute
- Progress analysis and workout trend responses
- Workout program generation (requires real-time generation; pre-written responses are insufficient)

**CRITICAL — clinical safety obligation during outage:** For every workout session started while Victor is in fallback mode, the RPE and injury-flag checks cannot run. Each such session must receive a `system.clinical_safety_bypass` DEC-030 CRITICAL event (7-year retention). This event is the audit record that FORM's clinical-safety gate was unavailable for a specific user session. The platform-engineer enabling fallback mode must trigger the clinical-safety-bypass event emitter, which runs the active-session scope SQL above and emits one event per affected session before confirming fallback is active.

**If ElevenLabs TTS down, Anthropic API healthy:**

Set `voice_coaching_enabled = false` in Workers environment. Victor continues to function with full AI intelligence; only voice audio output is suppressed. The UI falls back to text-only coaching display. No user data is affected; no clinical-safety gate is bypassed.

```
wrangler secret put VOICE_COACHING_ENABLED --env production
# value: "false"
```

#### Blast Radius Assessment

| Feature | Anthropic API Down | ElevenLabs TTS Down | Fallback Available |
|---|---|---|---|
| Real-time coaching guidance | Unavailable | Fully available | Cached last response only |
| Workout program generation | Unavailable | Fully available | No |
| RPE / fatigue check-in | Unavailable | Fully available | No (clinical-safety gate) |
| Injury-flag detection | Unavailable | Fully available | No (clinical-safety gate) |
| Voice cue delivery | Fully available | Unavailable | No (text-only mode) |
| Set / rep / weight logging | Fully available | Fully available | N/A — no AI dependency |
| Progress analysis | Unavailable | Fully available | No |
| Wearable data sync | Fully available | Fully available | N/A — no AI dependency |
| Workout plan display (pre-written) | Available via fallback | Fully available | Yes — fallback mode surfaces plan |

#### Recovery Protocol (when Anthropic or ElevenLabs confirms resolution)

```
1. Confirm service restored:
   → Anthropic status page shows green AND/OR ElevenLabs status page shows green
   → Do not rely solely on status pages — run the canary test below before re-enabling

2. Canary test — send 5 coaching_turns via synthetic user:
   → Tenant: non-production canary tenant (tenant_slug: 'form-canary')
   → User: synthetic canary user (user_id pre-provisioned in canary env)
   → Assert: HTTP 200, response latency < 3s P95 (OBSERVABILITY §13 SLA baseline)
   → Assert: coaching_turns row created in DB with non-null ai_response field

3. If canary passes:
   → Disable victor_fallback_mode:
     wrangler secret put VICTOR_FALLBACK_MODE --env production
     # value: "false"
   → Emit system.victor_fallback_mode_disabled DEC-030 event

4. If ElevenLabs is also recovering:
   → Re-enable voice coaching:
     wrangler secret put VOICE_COACHING_ENABLED --env production
     # value: "true"

5. Monitor for 30 minutes post-recovery:
   → PostHog: coaching_turns failure rate — confirm return to baseline (< 1%)
   → Sentry: confirm no new Anthropic 5xx events in workers/victor-edge/*
   → OBSERVABILITY §13 AI coach P95 latency dashboard — confirm < 3s

6. If error rate returns to baseline and canary is green for 30 min:
   → IC closes #inc-YYYYMMDD-ai-outage channel
   → Emit system.ai_service_recovery_confirmed DEC-030 event
   → Update Linear ticket status to Resolved
```

#### Enterprise Tenant Communication

**If outage duration > 15 minutes AND enterprise tenants had active or recently active users:**

Send Template E-01 (§12) with AI service outage context substituted. The Customer Lead owns all external communications; IC approves all statements before sending.

Approved template language for Victor outage:

```
Subject: FORM Service Update — [DATE] [TIME UTC]

Victor coaching is currently limited due to an infrastructure dependency issue.
Set and rep logging, wearable sync, and workout history are fully available.
We are working to restore full coaching functionality and will update within
[30 / 60] minutes. We apologise for the disruption to your team's sessions.

FORM Team · enterprise@form.coach
```

Do NOT name Anthropic or ElevenLabs in tenant communications without explicit founder approval. Naming a sub-processor in an outage communication is a vendor disclosure decision with commercial and contractual implications. Use "infrastructure dependency issue" as the approved term.

**If outage duration > 60 minutes:** SLA assessment per §12.3 — check whether the AI feature availability SLA has been breached. Enterprise contracts guarantee Victor API availability at >= 99.5% monthly uptime. An Anthropic outage counts toward FORM's SLA obligation regardless of root cause.

**Force majeure clause review:** If the Anthropic outage exceeds 4 hours, the founder and external counsel must assess whether the force majeure clause in enterprise agreements applies to third-party AI service unavailability. This analysis must be documented before any credit commitment or SLA waiver is communicated to enterprise tenants. Do not promise SLA credit or invoke force majeure without counsel review.

| Outage Duration | Communication Action | Owner |
|---|---|---|
| > 15 min with active enterprise users | Template E-01 — initial notification | Customer Lead, IC approval |
| > 30 min | Template E-02 — 30-min status update | Customer Lead |
| > 60 min | SLA breach assessment per §12.3; update enterprise contacts | Customer Lead + compliance-officer |
| > 4 h | Force majeure clause review; founder + counsel required before any credit commitment | Founder + counsel |
| Resolution | Template E-03 — resolution notification | Customer Lead |

#### Post-Incident Review

PIR is required for all P0 and P1 incidents within 5 business days of resolution (§8). For R-17 incidents, the PIR must address:

1. Was the `FORM-AI-OUTAGE-001` alert integrated with the Anthropic status page at the time of the incident? If FORM's own health-check alert fired before `status.anthropic.com` showed the issue, that indicates status page webhook integration (OBSERVABILITY §18) is not yet live — add it to the P1 implementation checklist below.
2. Was `victor_fallback_mode` activated within the 10-minute threshold? If not, what caused the delay?
3. Did any active enterprise workout sessions experience Victor silence without fallback mode being active? If yes, document the gap between outage onset and fallback activation — this gap may represent unlogged clinical-safety-bypass sessions.
4. How many `system.clinical_safety_bypass` CRITICAL events were emitted? Is the count consistent with the active-session scope SQL run at T+0?
5. Does Anthropic's historical uptime record support a force majeure defense against enterprise SLA credit obligations, or does this outage represent a pattern requiring contract renegotiation?

**Preventive controls to review at PIR:**

1. **Anthropic status page webhook** — wire status change notifications from `status.anthropic.com` → `form-alert-relay` Cloudflare Worker → `#ops-ai-status` Slack alert. This provides early warning before FORM's own health check detects the outage (OBSERVABILITY §18). Owner: devops-lead.
2. **ElevenLabs status page webhook** — same pipeline as above. Owner: devops-lead.
3. **Multi-provider LLM fallback** (open architectural question — Milestone M6) — route to a secondary LLM provider if Anthropic returns 5xx for > 60 seconds. This is a significant architectural decision requiring product-strategist review for model parity, clinical-safety review for output quality consistency, and compliance-officer review for sub-processor DPA implications. Added to DECISION_LOG as open question `DQ-LLM-FALLBACK`. Do not implement without those reviews completed.
4. **Pre-cached Victor response library** — build a library of 20 high-frequency coaching moments in Victor voice for use during fallback mode (replaces the current single-message fallback with more contextual responses). Owner: platform-engineer + ml-engineer + sports-scientist.
5. **Quarterly Anthropic dependency review** — current model version in use, API version, FORM's usage tier (tier affects priority of outage communication from Anthropic account team), model deprecation timeline. Owner: platform-engineer.

#### DEC-030 HMAC-Chained Audit Events

All events are HMAC-chained per `docs/AUDIT_LOG_SCHEMA.md`. A break in any event in this set for a given `incident_id` is P0 per R-05.

| Event Type | DEC-030 Severity | Retention | Trigger Condition | Key Metadata |
|---|---|---|---|---|
| `system.ai_service_outage_detected` | HIGH | 3 years | `FORM-AI-OUTAGE-001` or `FORM-VOICE-OUTAGE-001` alert fires and is IC-confirmed | `service` (anthropic / elevenlabs), `first_error_at`, `error_rate_pct`, `incident_id` |
| `system.victor_fallback_mode_enabled` | HIGH | 3 years | platform-engineer sets `VICTOR_FALLBACK_MODE = true` in Workers env | `fallback_mode`, `trigger_incident_id`, `enabled_by`, `enabled_at` |
| `system.clinical_safety_bypass` | CRITICAL | 7 years | Active workout session exists when Victor enters fallback mode — one event per affected session | `session_id`, `user_id`, `tenant_id`, `outage_started_at`, `bypass_reason: "anthropic_api_unavailable"` |
| `system.ai_service_recovery_confirmed` | MEDIUM | 3 years | Canary test passes post-recovery and error rate returns to baseline | `service`, `recovery_confirmed_at`, `outage_duration_minutes`, `canary_latency_p95_ms` |
| `system.victor_fallback_mode_disabled` | HIGH | 3 years | platform-engineer sets `VICTOR_FALLBACK_MODE = false` — normal operation restored | `fallback_disabled_at`, `disabled_by`, `incident_id` |
| `system.ai_sla_breach_assessed` | HIGH | 7 years | Outage duration crosses enterprise SLA threshold (99.5% monthly uptime) | `tenant_ids_affected[]`, `duration_minutes`, `credit_assessed` (bool), `force_majeure_invoked` (bool) |

**Retention note:** `system.clinical_safety_bypass` carries 7-year retention (vs. the 3-year default for system events) because it documents a moment when FORM's clinical-safety gate was unavailable for a specific user session. This is the closest R-17 comes to a health-data regulatory event and must survive the same retention window as Art. 9 breach records.

#### SOC 2 TSC Mapping

| TSC Criterion | How R-17 Satisfies It |
|---|---|
| **A1.1** (Availability commitments and system components) | Anthropic and ElevenLabs commitments are documented in §11.2 sub-processor registry; this runbook defines the outage response protocol and is the evidence that FORM has documented procedures for vendor-driven availability events |
| **A1.2** (Recovery time and recovery point objectives) | Victor fallback mode and text-only degradation paths are documented availability protections; references DR-004 scenario (AI service dependency failure) in the disaster recovery plan |
| **CC9.2** (Sub-processor monitoring and risk mitigation) | Status page webhook integration (preventive control 1 above) constitutes sub-processor monitoring; this runbook constitutes the vendor failure response procedure required by CC9.2 |
| **CC7.2** (Monitoring for and evaluating system events) | `FORM-AI-OUTAGE-001` and `FORM-VOICE-OUTAGE-001` alerts cover anomaly detection for AI service degradation; the canary test protocol constitutes continuous monitoring post-recovery |

#### Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Configure `FORM-AI-OUTAGE-001` Cloudflare Worker health-check alert: Anthropic endpoint health-check at 1-min cadence; fire after 5 consecutive 5xx responses; route to PagerDuty + `#ops-ai-status` | platform-engineer | **P0** | M4 |
| Configure `FORM-VOICE-OUTAGE-001` ElevenLabs TTS endpoint health-check: fire after 10 consecutive 5xx responses; same PagerDuty + Slack routing | platform-engineer | **P1** | M4 |
| Implement `VICTOR_FALLBACK_MODE` Workers env flag + fallback response logic in `apps/api/src/workers/victor-edge/`: cached-response replay, workout plan surface from `workout_programs` table, `coaching_turns` creation skip, `system.clinical_safety_bypass` DEC-030 emitter for all active sessions at fallback activation | platform-engineer | **P0** | M5 |
| Build pre-cached Victor response library: 20 high-frequency coaching moments in Victor voice; reviewed by sports-scientist for accuracy and by clinical-safety for harm pattern compliance before deployment | ml-engineer + sports-scientist | **P1** | M5 |
| Add Anthropic and ElevenLabs status page webhooks to `form-alert-relay` Worker → `#ops-ai-status` Slack channel (OBSERVABILITY §18) | devops-lead | **P1** | M5 |
| Tabletop Scenario J: *Anthropic API enters a 90-minute outage at 07:30 UTC on a Tuesday; three enterprise tenants have team workout sessions in progress; FORM-AI-OUTAGE-001 fires at T+5 min; on-call engineer is platform-engineer; compliance-officer is not yet online*. Run and validate: fallback activation SLA, clinical-safety-bypass DEC-030 emission count vs. active-session scope SQL, enterprise E-01 communication timing, SLA breach threshold assessment. | security-engineer | **P2** | M5 |
| Add multi-provider LLM fallback (route to secondary provider if Anthropic 5xx for > 60s) to DECISION_LOG as open architectural question `DQ-LLM-FALLBACK`; assign to product-strategist for review with clinical-safety and compliance-officer as co-reviewers | product-strategist | **P2** | M5 |

---

*v1.2 additions (2026-06-02): R-17 Third-Party AI Service Outage — new runbook covering Anthropic API and ElevenLabs TTS unavailability. Distinct from R-07 (sub-processor security breach — R-07 handles cases where a vendor reports unauthorized data access; R-17 handles service availability only) and from R-16 (API key exposure — R-16 handles the case where the Anthropic API key is compromised; R-17 handles the case where Anthropic's service is simply unavailable). Trigger matrix: FORM-AI-OUTAGE-001 (Anthropic endpoint 5xx for > 5 consecutive minutes) and FORM-VOICE-OUTAGE-001 (ElevenLabs TTS 5xx for > 10 consecutive minutes). Severity: P1 baseline for Anthropic down (Victor non-functional for all users); P0 upgrade if enterprise tenant has active workout sessions in progress (live safety-critical coaching interrupted; clinical-safety gate cannot run). Victor fallback mode: VICTOR_FALLBACK_MODE Workers env flag activates cached-response replay, workout plan surface from workout_programs table, coaching_turns creation skip (no billing impact, no cascading 5xx errors); does NOT apply to initial session personalization, RPE/fatigue check-ins, injury-flag detection, or program generation — all gated by clinical-safety. Clinical-safety bypass event: system.clinical_safety_bypass CRITICAL DEC-030 event (7-year retention) emitted once per active session during Anthropic outage — the audit record that FORM's clinical-safety gate was unavailable for a specific user session. Enterprise SLA protocol: enterprise contracts guarantee Victor API availability >= 99.5% monthly; Anthropic outage counts toward FORM's SLA obligation regardless of root cause; force majeure clause review required for outages > 4h (founder + counsel); do not name Anthropic or ElevenLabs in tenant communications without founder approval — use "infrastructure dependency issue." Six DEC-030 events: system.ai_service_outage_detected (HIGH, 3yr), system.victor_fallback_mode_enabled (HIGH, 3yr), system.clinical_safety_bypass (CRITICAL, 7yr — highest-retention event in this runbook), system.ai_service_recovery_confirmed (MEDIUM, 3yr), system.victor_fallback_mode_disabled (HIGH, 3yr), system.ai_sla_breach_assessed (HIGH, 7yr). SOC 2 mapping: A1.1 (vendor commitments and outage protocol documented), A1.2 (fallback mode as documented availability protection — DR-004 reference), CC9.2 (status page webhook as sub-processor monitoring; vendor failure response procedure), CC7.2 (FORM-AI-OUTAGE-001/FORM-VOICE-OUTAGE-001 anomaly detection; canary test post-recovery monitoring). Seven-item implementation checklist: two P0 (M4 FORM-AI-OUTAGE-001 alert, M5 VICTOR_FALLBACK_MODE logic + clinical-safety-bypass emitter), two P1 M4/M5 (FORM-VOICE-OUTAGE-001 alert, pre-cached response library, status page webhooks), two P2 M5 (Tabletop Scenario J, DECISION_LOG open question DQ-LLM-FALLBACK). Document header updated v1.1 → v1.2.*

---

*v1.1 additions (2026-06-02): R-16 Application Secret & Encryption Key Exposure — standalone runbook for the scenario where a Workers Secret, encryption key, or third-party API credential is discovered to be exposed. Distinct from R-03 (user credentials), R-04 (SSO certificate), R-08 (supply chain attack — secret rotation is remediation step there), R-09 (ransomware), R-11 (biometric breach — activated in parallel when keypoints_enc affected). Trigger matrix: 11 secret types mapped to severity (P0: keypoints_enc / JWT secret / service role key / SCIM master secret; P1: HMAC chain key / Stripe webhook / Anthropic API key / attestation signing key; P2: ElevenLabs / PostHog / Sentry DSN). Severity upgrade: any P1/P2 upgraded to P0 if exposure window > 24h or unauthorized access confirmed. Six trigger vectors: GHAS push, TruffleHog CI, Cloudflare audit log anomaly, external researcher, team member discovery, vendor usage spike. Exposure window assessment procedures per key type: service role key (Supabase API log SQL), Anthropic key (vendor console usage check), JWT secret (enterprise_sessions forged-session SQL), keypoints_enc (conditional — DB access required to decrypt; GDPR assessment deferred until access confirmed). Nine-step ordered rotation sequence: service role key first (no user impact); SCIM / API keys middle; HMAC chain key requires compliance-officer sign-off and epoch documentation; keypoints_enc requires biometric re-encryption Worker; JWT signing secret last (all sessions invalidated; 30 min E-ROTATE-01 notice to enterprise tenants). HMAC epoch transition procedure: pre-rotation chain verification export, compliance-officer sign-off file, old key cold storage for 7 years, `secret.hmac_epoch_transitioned` DEC-030 event distinguishes from R-05 unintended break, AUDIT_LOG_SCHEMA.md epoch boundary documentation. keypoints_enc re-encryption TypeScript scaffold: batched AES-256-GCM decrypt+re-encrypt; 5,000-row progress DEC-030 events; zero-row post-migration verification query; PAM destructive tier gate + two-person auth. JWT rotation procedure: E-ROTATE-01 template (30 min advance notice; immediate if P0), off-peak scheduling for P1, post-rotation auth surge monitoring via OBSERVABILITY §13. GDPR assessment matrix: keypoints_enc without DB access (case-by-case); keypoints_enc with DB access or service role key (presumed breach → R-01 + Art. 33); JWT (investigate forged sessions first); non-data keys (no Art. 33); HMAC key (epoch documentation + notify audit firm). Seven DEC-030 events: secret.exposure_discovered (CRITICAL, 7yr), secret.rotation_initiated (HIGH, 7yr), secret.rotation_completed (HIGH, 7yr), secret.biometric_re_encryption_progress (STANDARD, 3yr), secret.biometric_re_encryption_completed (CRITICAL, 7yr), secret.hmac_epoch_transitioned (CRITICAL, 7yr), secret.exposure_window_assessed (HIGH, 7yr); sub-chain keyed by incident_id; chain break = P0 per R-05. Evidence package: four directories (discovery, rotation, gdpr, dec030). Six post-incident preventive controls: TruffleHog CI path coverage, GHAS push protection, detect-secrets pre-commit, AL-SECRETS-01 Cloudflare alert, CRYPTOGRAPHY_POLICY.md rotation schedule, developer debrief. SOC 2 mapping: CC6.4 (credential lifecycle), CC7.1 (layered vulnerability detection via TruffleHog + GHAS + R-16 runbook), CC7.4 (DEC-030 sub-chain as incident timeline), CC5.2 (AL-SECRETS-01 monitoring). Seven-item implementation checklist: two P0 M4 (AUDIT_LOG_SCHEMA.md event registry, E-ROTATE-01 template), one P1 M4 (AL-SECRETS-01 Cloudflare alert), two P1 M5 (cv-key-migration Worker, Scenario K tabletop), two P2 M5 (detect-secrets onboarding, cross-references from R-11 and R-08). Document header updated v0.8 → v1.1 (v0.9, v1.0 additions preserved in body version notes).*

---

## Appendix A — Quick Reference Card

For use at 3am. IC: read §1 to classify, open the incident channel, then jump to the relevant runbook in §5.

```
P0?  → Open #inc-YYYYMMDD-slug immediately. Page security-engineer + IC. Start GDPR clock.
P1?  → Open #inc-YYYYMMDD-slug. Page IC. Monitor for upgrade.
P2?  → Track in Linear. Handle in business hours unless escalating.

Data breach?                → R-01
Outage?                     → R-02
Creds compromised?          → R-03
SSO login failures?
  Customer IdP unavailable (Okta/Azure AD outage)? → R-19 (availability; NOT a security breach)
    Multiple tenants on same IdP?  → R-19 P0; page compliance-officer + founder
    Need IT admin access NOW?      → R-19: break-glass protocol (two-person auth required)
    Outage > 30 min?               → R-19: SLA breach assessment per §23
  SAML cert expired or compromised? → R-04 (security incident)
  SAML assertion tampered / replay attack? → R-04 + R-01
Chain break?                → R-05 (always P0)
DDoS?                       → R-06
Processor notified us?      → R-07 (72h clock starts NOW — page compliance-officer)
Compromised npm package?    → R-08 (freeze deploys, check Workers env surface)
Ransomware / destruction?   → R-09 (ISOLATE FIRST — do not pay)
Victor harmful guidance?    → R-10 (clinical-safety on call; no raw harmful text in channel)
  Prompt injection?         → R-10 + P0 security reclassify → R-01 branch also active
  Model behavior drift?     → R-10 + R-07 (Anthropic sub-processor notification)
  Wellness-as-punishment?   → R-10 + escalate to founder (no-go customer contract violation)
CV / pose keypoints exposed? → R-11 (always P0; disable cv_enabled flag immediately)
  keypoints_enc at risk?    → R-11 + rotate cv_keypoints_encryption_key NOW
  Art. 9 biometric breach?  → R-11 + Art. 33 clock + likely Art. 34 + clinical-safety
  RLS failure on cv_sessions → R-11 + R-01 both active simultaneously
Insider / privileged access abuse? → R-12 (evidence BEFORE containment — do NOT revoke first)
  HMAC chain intact?        → R2 archive is forensic truth; do not use live audit_log_events
  Legal counsel BEFORE HR   → No employment action without legal go-ahead
  Multi-tenant access?      → P0; enterprise E-01 within 30 min of P0 declaration
  Art. 9 data read?         → P0 + Art. 33 clock + R-12.8 GDPR table
Law enforcement / government request? → R-13 (NO voluntary disclosure — compulsory process only)
  NSL or FISA?              → P0; gag order likely; do NOT log in shared systems before counsel
  Non-EU order on EU data?  → Art. 48 analysis required BEFORE any production; notify EU DPA
  Enterprise tenant in scope? → T-01 notification within 30 min (P0) / 4h (P1)
  User notification?        → Default YES (unless gag order); send before production
  Art. 9 health data?       → Higher legal standard required; counsel must confirm explicitly
  Founder authority only    → No disclosure, no acknowledgment without founder sign-off
  Legal hold:               → Activate legal_hold_active flag; no deletion for affected records
DSAR / data subject rights?  → R-14 (compliance-officer leads; note Art. 33 clock risk)
  Cross-tenant export?       → R-14 + R-01 simultaneously (P0 data breach — revoke link NOW)
  Erasure failure (Art. 9)?  → R-14 + assess R-11 for cv_sessions.keypoints_enc
  Missed 30-day deadline?    → R-14, P1; deliver ASAP; draft Template D-01 for DPA
  Enterprise employee DSAR?  → R-14.6; notify tenant admin; do not produce without instruction
  Bulk / coordinated abuse?  → R-14.5; Art. 12(5) assessment; notify compliance-officer
Health data in Sentry error payload? → R-15 (P0 — FORM-HEALTH-LEAK-001 alert; scrubber fires)
  Confirmed transmission to Sentry?  → R-15 → GDPR Art. 33 assessment → R-01 if breach confirmed
  Scrubbed before transmission?       → R-15 — no Art. 33 obligation; fix code path + PIR
Secret / key exposed?        → R-16 (assess exposure window BEFORE rotating — order matters)
  keypoints_enc key?         → R-16 + R-11 parallel (biometric re-encryption required)
  Service role key (SUPABASE_SERVICE_ROLE_KEY)? → R-16 + R-01 immediately (BYPASSRLS = presumed breach)
  JWT signing secret?        → R-16 (all sessions invalidated; notify tenants 30 min before)
  HMAC audit chain key?      → R-16 + epoch transition procedure; compliance-officer sign-off required
  Anthropic / ElevenLabs?    → R-16 (check vendor usage logs; no user data exposure unless DB accessed)
  SCIM master token?         → R-16 (P0; re-sign all tenant SCIM tokens; notify CSM team)
Anthropic API / ElevenLabs down? → R-17 (NOT R-07 — this is availability, not a security breach)
  Active enterprise workout sessions? → R-17 P0 upgrade; page founder + customer-success NOW
  Anthropic down > 10 min?   → R-17: set VICTOR_FALLBACK_MODE=true; emit clinical_safety_bypass DEC-030
  ElevenLabs down only?      → R-17 P2: set VOICE_COACHING_ENABLED=false; text-only mode; no safety impact
  Outage > 60 min?           → R-17: SLA breach assessment per §12.3; enterprise tenant comms via E-01
  Outage > 4h?               → R-17: force majeure clause review; founder + counsel before any credit commitment
  DO NOT name Anthropic/ElevenLabs in tenant comms without founder approval — use "infrastructure dependency issue"
Database integrity / Neon Postgres?  → R-18 (assess P0 upgrade if C2 cross_tenant_rows > 0 → also activate R-01)
  HMAC chain affected?              → R-18 + R-05 simultaneously (chain break is always P0)
  RLS bypass confirmed?             → R-18 P0 upgrade + R-01 immediately
  Failover clean (< 2 min, no data loss)? → R-18 P2; confirm integrity; no R-01 activation needed
  Art. 9 tables in scope?           → R-18 + Art. 33 clock; assess R-11 if cv_sessions.keypoints_enc affected
Insider / privileged access (PAM-aware)?  → R-20 (evidence BEFORE containment — do NOT revoke first)
  Form_admin ops outside PAM window? → R-20 via FORM-INSIDER-001; IC + compliance-officer + legal at T+0
  FORM-INSIDER-001 fires?           → R-20 + legal counsel immediately before HR notification
  Audit chain intact?               → R2 archive is forensic truth; do not use live audit_log_events
  service_role key externally leaked? → R-20 P0 + R-01; rotate immediately after evidence snapshot
  Post-separation access?           → R-20; confirm PAM window status; check jit_escalation_id on audit events
  (Also see R-12 for historical insider incidents pre-PAM implementation)
Key rotation overdue or failing?  → R-21 (AL-KEY-02 or AL-KEY-04 fires)
  CRYPTO-CHAIN-02 stop-gate: unauthorized rotation? → STOP R-21; activate R-05 + R-20 instead
  AL-KEY-02 fires (rotation overdue, P0)?  → R-21; 2-hour rotation SLA clock starts
  AL-KEY-04 fires (verification missing)?  → R-21; check C1 KV state and C3 scheduler health first
  KEYPOINTS_ENC_KEY rotation?       → R-21; maintenance window required (OQ-KEY-02); enterprise tenant 72h notice
Admin dashboard showing individual employee data? → R-22 (privacy floor breach)
  Any individual user health data visible to HR/tenant admin? → R-22 P0; disable admin dashboard immediately
  ED screening / body composition / mental health in employer report? → R-22 P0; clinical-safety VETO activates
  k-anonymity cohort < 5 in employer-visible report? → R-22 P1; suppress offending cohort
  Tenant demanding individual data or punishing non-participants? → R-22.9 no-go escalation; founder authority only
Victor AI coach producing unsafe content?    → R-23 (clinical-safety VETO applies; do NOT share raw harmful text in channel)
  Dangerous training advice (extreme loads, injury training)?  → R-23 P1; R-23.3 immediate actions; R-23.5 Tier 2 containment
  Medical diagnosis / medication recommendation?  → R-23 P0 CRITICAL; clinical-safety VETO; T+3min global disable
  Body-shaming / eating-disorder-adjacent content?  → R-23 P0 CRITICAL; clinical-safety VETO; T+3min global disable
  Dangerously caloric-deficit advice?  → R-23 P0 CRITICAL; clinical-safety VETO; T+3min global disable
  Sports-science consensus violated (no rest days, ignore pain)?  → R-23 P2; R-23.5 Tier 1 guardrail injection
  Hallucinated exercises / contradictory periodisation?  → R-23 P2; R-23.5 Tier 1 guardrail injection
  Wrong language in coaching response?  → R-23 P3; R-23.5 Tier 1 guardrail injection
  Prompt injection suspected?  → R-23.7 Hypothesis 1; also activate R-01 if PII exfiltration possible
  Enterprise tenant users affected?  → R-23.6 Template V-03; also R-22 if privacy floor breach suspected
SCIM mass deprovisioning event?       → R-24 (data integrity + availability; NOT R-04 or R-19)
  ≥ 50% of tenant seats deprovisioned in < 30 min?  → R-24 P0; suspend SCIM sync (§R-24.3); page CSM + platform-engineer
  ≥ 10% or ≥ 100 users unexpectedly deprovisioned?  → R-24 P1; run R-24-C1 scope SQL; tenant admin notification MD-01 at T+10
  All tenant_admin roles deprovisioned (admin lockout)?  → R-24 P0 override; magic link recovery §R-24.4
  AL-SCIM-MASS-01 fires?             → R-24; run C1 scope SQL BEFORE contacting tenant; confirm P0 vs P1 threshold
  IdP admin confirms deprovision was intentional?  → Do NOT downgrade on verbal; confirm in writing before P0 → P1 downgrade
  FORM-side SCIM bug suspected (H3)?  → R-24 + consult OQ-R24-03; legal consultation on Art. 33 obligation
  All external comms: aggregate count only — NO individual user names to HR contacts

GDPR Art. 33 clock: 72h from first awareness, not from confirmation.
  Sub-processor breach: clock starts when FORM receives notification, not when
  processor discovered the incident. File partial notification at T+48h maximum.
  Biometric data (R-11): Art. 34 individual notification presumed required — do not skip.
R-09 exception: Isolation BEFORE evidence — stop spread first.
Evidence first, containment second — unless live exfiltration (R-01) or live destruction (R-09).
Never modify production logs. Read-only copies only.
Log every action in incident channel with timestamp and your name.
Sub-processor registry: §11.2 — check which data categories the vendor holds.
Enterprise tenant comms: §12 — Customer Lead owns; IC approves all external statements.
  Template E-01 within 30 min (P0) / 60 min (P1) of declaration.
Board / investor comms: §13 — founder only; Template B-01 within 4h of P0 declaration.
  Triggers: P0 > 2h / Art. 9 breach confirmed / enterprise tenant loss / press coverage.
Continuous improvement: §14 — action item registry in Linear [IR] project.
  Quarterly review: first Monday of each quarter (security-engineer + compliance-officer).
  Runbook update SLA: 7 days post-PIR for gaps; before deploy for infra changes.
  Annual CSA: Q4; output stored compliance/evidence/annual-csa/YYYY-ir-csa.md.
```

---

## 15. GDPR Article 33/34 Data Breach Notification Operational Runbook

> Owner: compliance-officer. Deputy: security-engineer. Audience: compliance-officer, founder, external counsel. Review: annually or after any Art. 33 filing.
> SOC 2 controls: P7.1 (breach notification commitments), P8.1 (disposal — notification as closure step), CC2.2 (external communication obligations), P6.1 (disclosure to third parties).

**Relationship to other sections:** §10 defines the legal framework. R-01 through R-14 are the *technical* response runbooks. This section is the *notification infrastructure* runbook — it operates in parallel with any technical runbook whenever a confirmed or probable breach involves personal data. The 72-hour clock does not wait for containment.

**Multi-tenant note:** FORM processes personal data in two capacities simultaneously:
- **B2C consumer tier:** FORM is the data controller. Art. 33 notifications go to the lead supervisory authority.
- **B2B enterprise tier:** FORM is a data processor. The enterprise tenant (employer) is the data controller. FORM's Art. 28 DPA obligations require FORM to notify the enterprise tenant *without undue delay* (§15.8) so the tenant can discharge their own Art. 33 obligation.

---

### 15.1 Purpose and Scope

This section provides the step-by-step operational procedure for managing GDPR breach notification under Articles 33 and 34. It covers:

1. Determining when the 72-hour clock starts (§15.2)
2. Deciding which incidents require Art. 33, Art. 34, or both (§15.3)
3. Scoping which data subjects and enterprise tenants are affected (§15.4)
4. Executing the 72-hour procedure (§15.5)
5. Filing notifications using pre-approved templates (§15.6, §15.7, §15.8)
6. Maintaining an auditable DEC-030 chain throughout (§15.9)
7. Preserving evidence for regulatory submissions (§15.10)
8. Managing follow-up and supplementary notifications (§15.11)

This runbook does **not** replace R-01 through R-14. Run the appropriate technical runbook concurrently. The breach commander for the technical runbook and the compliance-officer for this runbook operate in parallel; they must sync at T+1h, T+24h, and T+48h.

---

### 15.2 Awareness Clock — When Does the 72-Hour Window Open?

The 72-hour notification window begins at the moment FORM (as controller) becomes **aware** of a probable breach — not when the breach is confirmed or fully scoped.

**Awareness events that start the clock:**

| Event | Clock Starts | Who Determines |
|---|---|---|
| Alert fires in PagerDuty for `data.bulk_deletion`, `data.read_individual`, or `data.export_initiated` by unexpected actor | Immediately on acknowledge | devops-lead pages compliance-officer |
| Sub-processor notifies FORM of a breach (R-07 activation) | When FORM receives written notification from sub-processor | compliance-officer |
| Internal detection via HMAC chain break alert (S-007) | When chain break alert is confirmed as tamper (not probe error) | security-engineer pages compliance-officer |
| Security researcher discloses probable data exposure | When disclosure is received and triaged as credible | security-engineer + compliance-officer |
| R-11 (CV biometric) or R-12 (insider) runbook confirms Art. 9 data access | When scope assessment SQL confirms populated `keypoints_enc` or Art. 9 table rows in blast radius | compliance-officer |
| DSAR export (R-14) reveals cross-tenant contamination | When contamination confirmed in export manifest | compliance-officer |

**Clock override rule:** If there is genuine uncertainty about whether a breach occurred, FORM has an obligation to investigate promptly and file a partial Art. 33 notification before T+72h if the investigation has not ruled out a reportable breach. Filing a partial notification (Art. 33(4)) and supplementing it later is compliant. Filing nothing and later discovering a reportable breach was "missed" is not.

**Clock tracking:** Create a `#inc-YYYYMMDD-gdpr-clock` Slack message immediately on awareness. Pin the message. Update it at T+4h, T+24h, T+48h, and T+72h. Archive the thread to `compliance/evidence/breach-notifications/<slug>/clock-log.txt`.

---

### 15.3 Incident-Class to Notification Decision Matrix

Not every security incident requires GDPR notification. Use this matrix to determine the obligation.

| Incident Scenario | Data Categories Involved | Art. 33 Required? | Art. 34 Required? | Notes |
|---|---|---|---|---|
| Unauthorized read of `health_profiles` or `coaching_turns` (Art. 9 data) | Special category health data | **Yes — P0** | **Yes — P0** | Art. 9 data triggers Art. 34 presumption; override requires documented mitigation |
| Unauthorized read of `cv_sessions.keypoints_enc` (biometric) | Art. 9 biometric | **Yes — P0** | **Yes — P0** | See R-11; Art. 34 individual notification required unless key not compromised |
| Unauthorized read of `users.email` only | Contact data | Yes — P1 | Discretionary | High risk only if enables further harm (phishing, identity theft); assess with security-engineer |
| Accidental export cross-tenant contamination | Any | Yes — P0 | Depends on data in export | If Art. 9 data in export: Art. 34 required. Email-only: discretionary |
| Ransomware with confirmed data destruction (no exfiltration) | All FORM data | **Yes — P0** | Conditional | Destruction is a breach. Art. 34 if data cannot be restored and subjects are harmed |
| Insider threat — Art. 9 data read confirmed (R-12) | Art. 9 + contact | **Yes — P0** | **Yes — P0** | Art. 9 read by unauthorized actor is high risk by default |
| Sub-processor breach affecting FORM user data (R-07) | Depends on sub-processor surface | Yes — assess Art. 9 tier | Depends | FORM may need to file Art. 33 even if sub-processor also files; Art. 28 DPA determines |
| Compromised Supabase auth token — no confirmed data access | Auth data only | Maybe — investigate | No | Unauthorized access potential; file if investigation cannot rule out data read within 72h |
| Government data request (R-13) | Any | No | No | Art. 33 applies to *security* breaches, not lawful government access; Art. 34 may apply separately if FORM notifies user |
| DSAR export failure — no data delivered, no third-party exposure | N/A | No | No | Not a security breach; R-14 DSAR process governs |

**Assessment authority:** compliance-officer makes the Art. 33 / Art. 34 determination. For Art. 9 data, external counsel must be consulted within 24 hours. The founder must be briefed before any DPA notification is filed.

---

### 15.4 Multi-Tenant Breach Scoping SQL

Run these queries to determine which users and enterprise tenants are affected before drafting notifications. Results are used to populate the Art. 33 notification fields ("approximate number of data subjects affected").

Execute with the `form_admin` role (BYPASSRLS). Results must be treated as restricted — route to `#inc-YYYYMMDD-gdpr-clock` only; do not paste into general incident channel.

**Query 1: Identify affected user count and data categories**

```sql
-- Scope: users active in the exposure window whose Art. 9 data was accessible
-- Replace :exposure_start and :exposure_end with confirmed breach timestamps
-- Replace :suspect_actor_id with the actor confirmed in blast radius assessment

WITH affected_users AS (
  SELECT DISTINCT
    u.id               AS user_id,
    u.tenant_id,
    u.created_at,
    CASE
      WHEN hp.id IS NOT NULL THEN 'Art.9:health'
      ELSE 'Art.6:contact'
    END                AS data_category,
    hp.id IS NOT NULL  AS has_art9_data,
    cs.keypoints_enc IS NOT NULL AS has_biometric
  FROM users u
  LEFT JOIN health_profiles hp ON hp.user_id = u.id
  LEFT JOIN (
    SELECT DISTINCT user_id, keypoints_enc
    FROM cv_sessions
    WHERE keypoints_enc IS NOT NULL
      AND created_at BETWEEN :exposure_start AND :exposure_end
  ) cs ON cs.user_id = u.id
  WHERE u.id IN (
    SELECT resource_id::uuid
    FROM audit_log_events
    WHERE action IN ('data.read_individual', 'data.export_initiated')
      AND created_at BETWEEN :exposure_start AND :exposure_end
      AND actor_id = :suspect_actor_id
  )
)
SELECT
  COUNT(*)                                  AS total_affected_users,
  COUNT(*) FILTER (WHERE has_art9_data)     AS users_with_art9_data,
  COUNT(*) FILTER (WHERE has_biometric)     AS users_with_biometric,
  COUNT(DISTINCT tenant_id)                 AS enterprise_tenants_affected,
  MIN(created_at)                           AS earliest_user_created,
  MAX(created_at)                           AS latest_user_created
FROM affected_users;
```

**Query 2: Identify affected enterprise tenants for B2B notification**

```sql
-- Produces the list of enterprise tenants requiring Art. 28 processor-to-controller notification
SELECT
  t.id            AS tenant_id,
  t.slug,
  t.display_name,
  t.billing_email,
  t.it_admin_email,
  COUNT(u.id)     AS affected_seat_count,
  BOOL_OR(hp.id IS NOT NULL) AS has_art9_exposure
FROM tenants t
JOIN users u ON u.tenant_id = t.id
LEFT JOIN health_profiles hp ON hp.user_id = u.id
WHERE u.id IN (
  SELECT resource_id::uuid
  FROM audit_log_events
  WHERE action IN ('data.read_individual', 'data.export_initiated')
    AND created_at BETWEEN :exposure_start AND :exposure_end
    AND actor_id = :suspect_actor_id
)
GROUP BY t.id, t.slug, t.display_name, t.billing_email, t.it_admin_email
ORDER BY has_art9_exposure DESC, affected_seat_count DESC;
```

**Query 3: Verify HMAC chain integrity in exposure window**

```sql
-- Confirms the audit log was not tampered during or after the breach
-- Any row where hmac_prev ≠ LAG(hmac_self) is a chain break — P0 per docs/AUDIT_LOG_SCHEMA.md
SELECT
  id,
  created_at,
  action,
  actor_id,
  hmac_self,
  LAG(hmac_self) OVER (ORDER BY created_at) AS expected_hmac_prev,
  hmac_prev
FROM audit_log_events
WHERE created_at BETWEEN :exposure_start - INTERVAL '1 hour'
                     AND :exposure_end   + INTERVAL '1 hour'
ORDER BY created_at;
```

Output of all three queries must be saved to `compliance/evidence/breach-notifications/<slug>/scope-assessment/` before the Art. 33 notification is filed.

---

### 15.5 72-Hour Clock Procedure

| T+ | Action | Owner | Output |
|---|---|---|---|
| T+0h | Clock starts. Pin message in `#inc-YYYYMMDD-gdpr-clock`. Open Linear ticket `[GDPR] <slug> Art.33 clock`. Activate relevant technical runbook in parallel. | compliance-officer | Slack message ID, Linear ticket URL |
| T+0h | Brief founder. Provide: breach type, estimated scope, 72h deadline timestamp. One paragraph, no root-cause speculation. | compliance-officer | Founder acknowledgment (Slack DM logged in thread) |
| T+2h | Run §15.4 scope assessment SQL. Log query results in Linear ticket. | security-engineer + compliance-officer | Scope assessment files committed to `compliance/evidence/breach-notifications/<slug>/scope-assessment/` |
| T+4h | Determine Art. 33 / Art. 34 obligations using §15.3 matrix. Log decision and rationale in Linear ticket. Retain external counsel if Art. 9 data confirmed. | compliance-officer | Decision logged; counsel engaged if required |
| T+8h | For enterprise tenants in scope: send B2B processor-to-controller notification (§15.8 Template P2C-01). Emit `breach.processor_notified_tenant` DEC-030 event per enterprise tenant notified. | compliance-officer | Per-tenant email sent; DEC-030 event chain confirmed |
| T+24h | Draft Art. 33 notification (§15.6 Template A33-01). Share with founder and counsel for review. Draft Art. 34 notification if required (§15.7 Template A34-01). | compliance-officer + counsel | Draft documents in `compliance/evidence/breach-notifications/<slug>/drafts/` |
| T+48h | If Art. 33 required and scope is not yet fully established: prepare partial notification per Art. 33(4). Do not wait for complete scope if partial filing preserves the 72h window. Founder gives final sign-off. | compliance-officer + founder | Go / No-go decision; if No-go, documented rationale as to why breach is not reportable |
| T+71h | File Art. 33 notification via the lead DPA's online portal. Screenshot the confirmation receipt. Save to `compliance/evidence/breach-notifications/<slug>/art33/`. Emit `breach.art33_notification_filed` DEC-030 event. | compliance-officer | Filing receipt, DEC-030 event ID |
| T+72h | 72-hour deadline. If not filed: this is a compliance failure. Notify founder and counsel immediately. File anyway — late is better than never. Document the delay and cause in the Art. 33 form. | compliance-officer | — |
| Post-filing | If Art. 34 required: send data subject notifications within 24 hours of Art. 33 filing. Use Template A34-01 (§15.7). Emit `breach.art34_notifications_sent` DEC-030 event. | compliance-officer | Bulk send confirmation; DEC-030 event |
| Post-filing | File supplementary Art. 33(4) notifications as scope investigation completes. Emit `breach.art33_supplementary_filed` for each supplementary submission. | compliance-officer | Supplementary filing receipts |

---

### 15.6 Article 33 Supervisory Authority Notification Template

**Template A33-01 — Initial or Partial Art. 33 Notification**

File via the lead DPA's online portal. If the online portal is unavailable, send by encrypted email to the DPA's security incident contact. Retain the sent copy and delivery confirmation.

```
SUBJECT: GDPR Article 33 Data Breach Notification — FORM · [INC-YYYYMMDD-slug]

CONTROLLER DETAILS
  Name:           FORM (form.coach)
  Address:        [FOUNDER_INPUT: registered address]
  Contact email:  enterprise@form.coach
  DPO / Contact:  [FOUNDER_INPUT: compliance-officer name and direct contact]

BREACH DESCRIPTION
  Date breach occurred (or estimated range): [INSERT]
  Date controller became aware:              [INSERT — this is T+0h for the 72-hour clock]
  Type of breach: [Confidentiality / Integrity / Availability — check all that apply]
  Description:
    [Plain-language description. Example: "Unauthorized access to the health_profiles
    table by an insider actor between [DATE] and [DATE]. Affected data includes workout
    performance metrics, body composition logs, and AI coaching session metadata for
    approximately [N] users."]

CATEGORIES AND APPROXIMATE NUMBER OF DATA SUBJECTS AFFECTED
  Data subject categories:          [e.g., consumer users / enterprise employees]
  Approximate number of subjects:   [Insert from §15.4 Query 1 — err high if uncertain]
  Member states involved:           [List EU member states where affected subjects are located]

CATEGORIES AND APPROXIMATE VOLUME OF RECORDS AFFECTED
  Record categories:   [health_profiles rows / coaching_turns rows / cv_sessions rows / users rows]
  Approximate count:   [Insert — err high if uncertain]
  Art. 9 special category data involved: [Yes / No — if Yes, specify categories]

LIKELY CONSEQUENCES
  [Describe risks to data subjects. Example: "Affected data subjects may experience distress
  from the exposure of health and fitness data. In the worst case, the data could be used to
  infer medical conditions or create discriminatory profiles. Financial harm is not anticipated
  as payment data is not stored by FORM (processed exclusively by Stripe)."]

MEASURES TAKEN OR PROPOSED
  Containment: [Brief summary of containment steps completed at time of filing]
  Eradication: [Steps planned or in progress]
  Recovery:    [Estimated restoration timeline]
  Preventive:  [Improvements committed to in post-incident review]

SUPPLEMENTARY INFORMATION
  Is this a partial notification under Art. 33(4)? [Yes / No]
  If yes, further information will be provided by: [Date — typically within 7 days]
  FORM internal reference: [INC-YYYYMMDD-slug]
  Has Art. 34 notification been issued to data subjects? [Yes / No / In progress]

Signed: [compliance-officer name]
Date:   [ISO 8601 timestamp UTC]
```

---

### 15.7 Article 34 Data Subject Notification Template

**Template A34-01 — Data Subject Notification (High Risk)**

Send via email to affected users using `users.email`. For Art. 9 breaches, do not include the specific Art. 9 data category in the email subject line — this prevents re-exposing sensitive data in email headers, which may be logged by email providers.

```
SUBJECT: Important security notice about your FORM account

Hi [first_name, or "FORM user" if first_name unavailable],

We are writing to let you know about a security incident that may have affected your FORM account.

WHAT HAPPENED
[Plain language description. Example: "Between [DATE] and [DATE], an unauthorized person
may have had access to your FORM account data. We detected and stopped the incident on [DATE]."]

WHAT DATA WAS INVOLVED
[Specific to this user's data tier. Example: "Your account may have included: your workout
history, training plans, and fitness metrics you logged in the FORM app."]

[If the user's account did NOT include Art. 9 data, state that clearly:]
"Your account did not include detailed health or fitness data beyond basic profile information."

WHAT WE HAVE DONE
- We have contained the incident and stopped the unauthorized access.
- We have notified [DPA name, e.g., the UK Information Commissioner's Office] as required by GDPR.
- We are conducting a full investigation and will implement additional security controls.

WHAT YOU CAN DO
- Review your account for any unusual activity.
- Change your FORM password at [link].
- Request a full copy of your data (Subject Access Request): [DSAR portal link or enterprise@form.coach].
- Questions? Contact us at enterprise@form.coach. We will respond within 3 business days.

CONTACT FOR FOLLOW-UP
  Incident queries:        enterprise@form.coach
  Data rights (DSAR):      [DSAR link]
  Supervisory authority:   [DPA name + URL, e.g., ICO: ico.org.uk/make-a-complaint]

We take this seriously and apologise for any concern this causes.

FORM Team
```

**Delivery:** Use Resend via the `breach_notification` email template slot. Do not use PostHog (product analytics sub-processor). Emit `data.breach_notification_sent` DEC-030 per user delivered (§15.9). For users with undeliverable email, log failed delivery in `art34/send-report.json` and apply Art. 34(3)(c) disproportionate-effort exception with a 30-day status page notice as the substitute public communication.

---

### 15.8 Enterprise Tenant B2B Processor-to-Controller Notification

Under FORM's Art. 28 Data Processing Agreement with each enterprise tenant, FORM (processor) must notify the enterprise tenant (controller) **without undue delay**. FORM targets T+8h from awareness; the contractual maximum in FORM's standard DPA is 48 hours.

**Template P2C-01 — Processor-to-Controller Notification**

```
TO:      [it_admin_email from tenants table] + [legal_contact if on file]
CC:      enterprise@form.coach
SUBJECT: Data Processing Incident Notification — [tenant.display_name] · FORM · [INC-YYYYMMDD-slug]

Dear [tenant.display_name] IT / Legal Team,

Pursuant to our Data Processing Agreement (Clause [X] — processor breach notification),
we are writing to inform you of a security incident affecting data we process on your behalf.

INCIDENT REFERENCE: [INC-YYYYMMDD-slug]
NOTIFICATION TIMESTAMP (UTC): [ISO 8601]

NATURE OF THE INCIDENT
[Brief description — what is known at T+8h. Avoid speculation on root cause.]

YOUR ORGANIZATION'S DATA
  Approximate employees affected:            [from §15.4 Query 2 — affected_seat_count]
  Data categories involved:                  [Art. 9 / contact / usage data]
  Art. 9 special category data involved:     [Yes / No]

CURRENT STATUS
  Containment:           [status]
  Investigation:         [ongoing / complete]
  Estimated resolution:  [date or "investigation ongoing — update by T+48h"]

YOUR GDPR OBLIGATIONS AS CONTROLLER
As the data controller for your employees' data, you may have independent obligations
under GDPR Art. 33/34 to notify your national supervisory authority and/or your employees.
We recommend you consult your Data Protection Officer or legal counsel.

  FORM's Art. 33 filing status: [Filed / Pending — target T+71h from FORM awareness]
  DPA reference (if filed):     [reference number from DPA portal, or "pending"]

NEXT STEPS FROM FORM
We will provide a full scope update by [T+48h date].
Your CSM [name] is available for calls: [contact].
Urgent queries: enterprise@form.coach

This notification is confidential and sent pursuant to our DPA obligations.

[compliance-officer name], Compliance
FORM (form.coach)
```

**Tracking:** For each enterprise tenant notified, create `compliance/evidence/breach-notifications/<slug>/tenant-notifications/<tenant-slug>.md` containing: sent timestamp, recipient emails, DEC-030 `breach.processor_notified_tenant` event ID, and CSM acknowledgment status.

---

### 15.9 DEC-030 Audit Events for Breach Notification Lifecycle

All breach notification events must be HMAC-chained per `docs/AUDIT_LOG_SCHEMA.md`. The chain provides tamper-evident evidence that notifications were filed on time and in sequence — this is the primary SOC 2 P7.1 auditor evidence. A break in the `breach.*` event chain is itself a P0 incident (R-05).

| Event Type | Severity | Retention | Trigger | Key Payload Fields |
|---|---|---|---|---|
| `breach.awareness_declared` | CRITICAL | 7 years | T+0h — compliance-officer declares clock start | `incident_id`, `awareness_timestamp`, `data_categories[]`, `art9_involved` (bool), `declared_by` |
| `breach.scope_assessment_completed` | HIGH | 7 years | §15.4 SQL queries executed and results committed | `incident_id`, `user_count`, `tenant_count`, `art9_user_count`, `biometric_user_count`, `query_executed_by` |
| `breach.art33_obligation_determined` | HIGH | 7 years | §15.3 matrix applied; decision reached | `incident_id`, `art33_required` (bool), `art34_required` (bool), `rationale`, `determined_by`, `counsel_consulted` (bool) |
| `breach.processor_notified_tenant` | HIGH | 7 years | Template P2C-01 sent to enterprise tenant | `incident_id`, `tenant_id`, `notification_sent_at`, `recipient_emails[]`, `hours_since_awareness` |
| `breach.art33_notification_filed` | CRITICAL | 7 years | Art. 33 notification submitted to DPA portal | `incident_id`, `dpa_name`, `dpa_reference`, `filed_at`, `hours_since_awareness`, `is_partial` (bool), `filed_by` |
| `breach.art33_supplementary_filed` | HIGH | 7 years | Art. 33(4) supplementary notification submitted | `incident_id`, `dpa_reference`, `supplement_number`, `new_information_summary`, `filed_at` |
| `breach.art34_notifications_sent` | HIGH | 7 years | Bulk send to affected data subjects complete | `incident_id`, `total_sent`, `art9_subjects_count`, `delivery_provider`, `send_completed_at`, `template_version` |
| `data.breach_notification_sent` | STANDARD | 7 years | Per-user Art. 34 notification delivered | `incident_id`, `user_id`, `email_hash` (SHA-256 of email — never raw), `delivered_at` |
| `breach.art34_waiver_documented` | HIGH | 7 years | Compliance-officer documents rationale for NOT sending Art. 34 | `incident_id`, `waiver_basis` (e.g., "encryption key not compromised"), `documented_by`, `counsel_reviewed` (bool) |
| `breach.notification_case_closed` | HIGH | 7 years | All obligations met; Linear ticket closed | `incident_id`, `art33_filed` (bool), `art34_filed` (bool), `tenant_count_notified`, `dpa_case_reference`, `closed_by` |

**Manual emission path:** All events may be emitted via the manual SQL path in `docs/AUDIT_LOG_SCHEMA.md` when the automated path is unavailable during an active incident. Document the manual emission in the Linear ticket with executor role, timestamp, and reason for manual path.

---

### 15.10 Evidence Preservation for Regulatory Submissions

Create `compliance/evidence/breach-notifications/<INC-YYYYMMDD-slug>/` immediately at T+0h. Populate using the following structure:

```
compliance/evidence/breach-notifications/<INC-YYYYMMDD-slug>/
  README.md                            — incident overview, clock log, final decision log
  clock-log.txt                        — archived Slack #inc-YYYYMMDD-gdpr-clock thread
  scope-assessment/
    query1-user-count.csv              — §15.4 Query 1 output (no raw email — user_id only)
    query2-tenant-list.csv             — §15.4 Query 2 output
    query3-hmac-check.csv             — §15.4 Query 3 output (chain integrity)
    query-executed-at.txt             — ISO 8601 timestamp + executor role
  drafts/
    art33-draft-v1.md                  — iterative drafts of Art. 33 notification
    art34-draft-v1.md                  — iterative drafts of Art. 34 (if applicable)
  art33/
    filed-notification.pdf             — copy of submitted Art. 33 notification
    dpa-confirmation.png               — screenshot of DPA portal confirmation receipt
    filed-at.txt                       — ISO 8601 timestamp of filing
    supplement-01.pdf                  — first supplementary Art. 33(4) filing (if applicable)
    dpa-correspondence-YYYY-MM-DD.pdf  — all subsequent DPA communications
  art34/
    send-report.json                   — Resend bulk delivery report (user_id + email_hash + status)
    waiver-rationale.md                — if Art. 34 waived: documented basis + counsel sign-off
  tenant-notifications/
    <tenant-slug>.md                   — per-tenant §15.8 notification record
  board/
    b01-initial.md                     — Board notification Template B-01 (see §13)
  legal/
    counsel-engagement.md              — date retained, scope, billing reference
  dec030/
    breach-event-chain.json           — breach.* events exported for auditor, HMAC chain included
  SHA256MANIFEST.txt                   — generated at case close (see below)
```

**Tamper-evident sealing at case close:**

```bash
find compliance/evidence/breach-notifications/<slug>/ -type f | sort \
  | xargs sha256sum > compliance/evidence/breach-notifications/<slug>/SHA256MANIFEST.txt
git add compliance/evidence/breach-notifications/<slug>/SHA256MANIFEST.txt
git commit -m "evidence: <slug> SHA-256 manifest sealed [compliance-officer]"
```

The git commit hash constitutes immutable sealing. Retain the full directory for 7 years.

---

### 15.11 Post-Notification Follow-Up and Supplementary Notifications

**Supplementary Art. 33(4) filings** are required when material new information becomes available: a more accurate data subject count, confirmation of previously uncertain Art. 9 exposure, or discovery of additional affected enterprise tenants. File within 7 days of learning new material information; do not wait for the full investigation to close.

**DPA follow-up:** EU DPAs typically acknowledge receipt and may request additional information within 2–4 weeks. All DPA correspondence routes exclusively through compliance-officer. Log each communication in the Linear ticket and in `art33/dpa-correspondence-YYYY-MM-DD.pdf`. DPA investigations for non-systemic breaches typically close in 6–18 months. Do not respond to DPA information requests without counsel review.

**Data subject follow-up:** Failed Art. 34 deliveries (bounce) must be logged in `art34/send-report.json`. If no alternative delivery method exists (no in-app account), apply Art. 34(3)(c) disproportionate-effort exception and publish a 30-day public notice on the FORM status page as the substitute. Document the exception and notice in `art34/waiver-rationale.md`.

**PIR integration:** The breach notification process itself must be reviewed in the §8 Post-Incident Review. Required PIR questions: (1) Did the clock start on the correct awareness event? (2) Was the T+8h enterprise tenant notification SLA met? (3) Was Art. 33 filed within 72h? (4) Were all affected enterprise tenants identified by Query 2 and notified? (5) Were Art. 34 decisions (filed or waiver) documented before filing? Add gaps to the §14 PIR Action Item Registry.

---

### 15.12 SOC 2 Privacy Criterion Mapping

| Privacy Criterion | Description | Evidence Provided by §15 |
|---|---|---|
| **P7.1** | The entity provides notification of breaches and incidents to affected data subjects, business partners, and regulators as required by commitments, agreements, and applicable laws | §15.5 procedure ensures timely filing; `breach.art33_notification_filed` DEC-030 CRITICAL event with `hours_since_awareness` field is the primary timing evidence for auditors |
| **P8.1** | The entity provides data subjects with an accounting of personal data held and corrects or deletes inaccurate data in a timely manner | §15.7 Art. 34 notifications include a DSAR reference; §15.11 PIR integration links breach closure to erasure verification |
| **P6.1** | The entity discloses personal data to third parties with the knowledge and consent of data subjects or as required by law or regulation | §15.8 documents the Art. 28 processor-to-controller notification; `breach.processor_notified_tenant` DEC-030 chain evidences timely disclosure |
| **CC2.2** | The entity communicates information to improve security knowledge and address security deficiencies | Board notifications (§13 Template B-01) and enterprise tenant notifications (§15.8 Template P2C-01) demonstrate external communication obligations are met |
| **CC7.3** | The entity evaluates security events to determine whether they could or have resulted in a failure of a commitment or requirement | §15.3 notification decision matrix constitutes the documented evaluation; `breach.art33_obligation_determined` DEC-030 event is the auditable outcome |

**Auditor evidence artefacts:**
- `compliance/evidence/breach-notifications/` directory (all incidents, SHA-256 sealed, 7-year retention)
- DEC-030 audit log filtered on `event_type LIKE 'breach.%'` with HMAC chain verification
- DPA filing receipts and all subsequent correspondence (`art33/` subdirectory per incident)
- Resend bulk delivery reports for Art. 34 (`art34/send-report.json` per incident)
- Enterprise tenant notification records (`tenant-notifications/` per incident)

---

### 15.13 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Register all 10 `breach.*` and `data.breach_notification_sent` DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` event registry | security-engineer | **P0** | M4 |
| Implement `breach.*` HMAC chain writer calls in the standard DEC-030 emitter function; validate chain in staging with a simulated breach sequence | platform-engineer + security-engineer | **P0** | M4 |
| Create `compliance/evidence/breach-notifications/` directory in repo with `TEMPLATE-README.md`, `TEMPLATE-SHA256MANIFEST.sh`, and `TEMPLATE-scope-queries.sql` | compliance-officer | **P0** | M4 |
| Determine lead supervisory authority: confirm FORM's main EU establishment in `docs/GDPR_DPIA.md §2`; document lead DPA name, portal URL, and encrypted-email fallback contact | compliance-officer + counsel | **P0** | M4 |
| Create `breach_notification` email template in Resend (Template A34-01, §15.7); confirm EU-region delivery node; send test to internal mailbox | platform-engineer | **P0** | M4 |
| Implement breach notification sender: `apps/api/src/breach/notify-subjects.ts` — reads `user_id[]`, renders A34-01, sends via Resend, emits `data.breach_notification_sent` DEC-030 per user | platform-engineer | **P0** | M4 |
| Add `scripts/breach-scope-assessment.sql` containing §15.4 Query 1, Query 2, and Query 3 with commented parameter documentation; confirm `form_admin` role BYPASSRLS access works for each | platform-engineer | **P0** | M4 |
| Create Linear project template `[GDPR] Art.33 Clock` with tasks pre-populated from §15.5 table; test by creating a template instance and archiving immediately | compliance-officer | **P1** | M4 |
| Write `compliance/policy-templates/P2C-01-processor-controller-notification.md` with Template P2C-01 (§15.8) and FORM standard DPA clause references pre-filled | compliance-officer | **P1** | M4 |
| Update `docs/SOC2_READINESS.md §35 P7.1` row from 🟡 to 🟢 after checklist items 1–7 are complete and verified in staging | compliance-officer | **P1** | M4 |
| Tabletop drill (Scenario J): simulate P0 breach involving `health_profiles` for a 150-seat enterprise tenant; run §15.5 checklist end-to-end in staging (mock DPA portal, test Resend send, mock tenant email); record elapsed time vs. 72h target; file result as `compliance/evidence/annual-csa/YYYY-gdpr-breach-drill.md` | compliance-officer + security-engineer | **P1** | M5 |
| Add Scenario J to §9 Testing and Drills rotation (§9.5 Year 2 schedule or first available annual slot) | compliance-officer | **P2** | M5 |

---

## 16. DEC-030 Incident Lifecycle Audit Events & SIEM-Triggered Automation

> Owner: security-engineer + devops-lead. Review: quarterly, or after any automation failure. Cross-references: OBSERVABILITY.md §27 (SIEM correlation rules CR-01–CR-04 and alert rules AL-SIEM-01–AL-SIEM-06), AUDIT_LOG_SCHEMA.md (DEC-030 event registry), §14 Continuous Improvement Program.

---

### 16.1 Purpose

Every runbook in this document (R-01 through R-14) emits **domain-specific** DEC-030 events — `breach.*`, `legal.*`, `dsar.*`, `coaching.*`, `siem.*` — that capture what happened *within* a specific type of incident. What was missing until this section: a **cross-cutting audit trail of the incident management process itself**.

Two problems that absence creates:

1. **SOC 2 CC7.4** ("The entity has implemented policies and procedures to respond to security incidents") requires evidence that the incident response program is *actively used and evaluated*, not just documented. A log of `breach.awareness_declared` events proves a breach was discovered; a log of `incident.pir_closed` events proves the post-mortem process runs to completion. Both are required.

2. **OBSERVABILITY.md §27** defines SIEM correlation rules (CR-01 through CR-04) and alert rules (AL-SIEM-01 through AL-SIEM-06) that *automatically trigger runbooks* — AL-SIEM-05 (HMAC chain break) auto-opens R-01; AL-SIEM-03 (impossible travel) routes to R-12. Until this section, that automation had no authoritative specification in the incident response runbook. Responders had no documented protocol for verifying that the auto-opened ticket was correct, escalating if automation failed, or confirming that SIEM-sourced incidents are covered by the same lifecycle obligations as manually declared ones.

This section defines:
- The ten `incident.*` DEC-030 events that form the universal lifecycle record for every FORM incident, regardless of type
- The SIEM-triggered automation architecture that creates Linear tickets and emits `incident.opened` on behalf of correlation rules
- Incident management meta-monitoring: SLOs on IC assignment time, PIR completion rate, and action item close rate
- SOC 2 CC7.4 / CC7.5 / CC4.1 evidence mapping

**Scope:** All incidents declared under this runbook — P0 through P2 — must emit the full `incident.*` lifecycle event set. P3 issues tracked in Linear are exempt from DEC-030 emission.

---

### 16.2 Incident Lifecycle Event Taxonomy

Ten events form the universal lifecycle spine. Every incident emits them in sequence; domain-specific events (breach.*, legal.*, etc.) are emitted *in addition*, woven into the same HMAC chain so the chain reflects both the incident process and the domain-specific actions.

| Event Type | Severity Level | Emitting Role | Trigger | Retention | SOC 2 Criterion |
|---|---|---|---|---|---|
| `incident.opened` | HIGH | Incident Commander (IC) or SIEM automation | IC declares incident OR SIEM AL-* rule auto-triggers (§16.4) | 7 years | CC7.3, CC7.4 |
| `incident.ic_assigned` | MEDIUM | IC (self-reporting) or automation | First confirmed IC takes command | 7 years | CC7.4 |
| `incident.severity_changed` | HIGH | IC | Any upgrade or IC-approved downgrade (§1.3) | 7 years | CC7.4 |
| `incident.escalated` | HIGH | IC | Founder, board, or external counsel engaged | 7 years | CC7.4, CC2.2 |
| `incident.update_posted` | STANDARD | IC or Communications Lead | Public status page update or enterprise tenant notification sent | 3 years | CC2.2 |
| `incident.containment_verified` | HIGH | IC + security-engineer | Active threat vector confirmed closed; no further exfiltration possible | 7 years | CC7.4 |
| `incident.eradicated` | HIGH | IC | Root cause removed; vulnerable code/config/credential no longer in production | 7 years | CC7.4 |
| `incident.recovered` | HIGH | IC | All services restored to normal; SLAs re-met; monitoring confirmed clean | 7 years | CC7.4, CC7.5 |
| `incident.pir_opened` | MEDIUM | Compliance officer | Post-incident review meeting scheduled; Linear PIR project created | 7 years | CC7.5 |
| `incident.pir_closed` | HIGH | Compliance officer | PIR complete; all action items logged in §14 registry; document signed off | 7 years | CC7.5, CC4.1 |

**Important:** Severity levels in this table refer to the DEC-030 audit event severity (how important the event is to preserve in the HMAC chain), not the incident severity classification (P0–P3). A P0 incident and a P2 incident both emit `incident.opened HIGH` — the event severity is the same; only the payload field `incident_severity` differs.

---

### 16.3 Incident Lifecycle Event Schema

All ten events share a common base schema. Domain-specific events embed their own schemas; the lifecycle events below are always present in every incident's DEC-030 chain.

```typescript
// Common fields — present in all ten incident.* events
interface IncidentLifecycleEvent {
  event_type: `incident.${LifecyclePhase}`;
  event_id: string;          // UUID v4, unique per event
  incident_id: string;       // INC-YYYYMMDD-NNN (matches Linear ticket ID)
  incident_severity: 'P0' | 'P1' | 'P2';
  incident_runbook: RunbookId | RunbookId[] | 'TBD';  // e.g. "R-01", ["R-01","R-11"]
  incident_source: 'manual' | 'siem_automation' | 'monitoring_alert';
  actor_role: ActorRole;     // 'security-engineer' | 'devops-lead' | 'compliance-officer' | 'founder' | 'siem_worker'
  actor_user_id: string | 'system'; // 'system' for SIEM-automation-triggered events
  timestamp_utc: string;     // ISO 8601
  linear_ticket_url: string; // always required; automation populates before emitting
  hmac_prev: string;         // SHA-256 HMAC of previous event in chain (DEC-030)
  hmac_current: string;      // SHA-256 HMAC of this event body
}

// Event-specific payloads (extend the common schema)

// incident.opened
interface IncidentOpenedPayload extends IncidentLifecycleEvent {
  event_type: 'incident.opened';
  title: string;              // Short description (max 100 chars)
  trigger_alert_id?: string;  // AL-SIEM-01 etc., if SIEM-triggered
  trigger_correlation_rule?: 'CR-01' | 'CR-02' | 'CR-03' | 'CR-04'; // if applicable
  siem_correlation_score?: number; // confidence score from correlation rule (0–1)
}

// incident.ic_assigned
interface IncidentIcAssignedPayload extends IncidentLifecycleEvent {
  event_type: 'incident.ic_assigned';
  ic_user_id: string;         // user_id of the IC
  ic_role: ActorRole;
  minutes_since_open: number; // lag between incident.opened and IC assignment
}

// incident.severity_changed
interface IncidentSeverityChangedPayload extends IncidentLifecycleEvent {
  event_type: 'incident.severity_changed';
  previous_severity: 'P0' | 'P1' | 'P2';
  new_severity: 'P0' | 'P1' | 'P2';
  direction: 'upgrade' | 'downgrade';
  reason: string;             // IC-authored free text (required for downgrade per §1.3)
  approver_user_id?: string;  // required for downgrade (IC approval per §1.3)
}

// incident.escalated
interface IncidentEscalatedPayload extends IncidentLifecycleEvent {
  event_type: 'incident.escalated';
  escalation_target: 'founder' | 'board' | 'legal_counsel' | 'dpa' | 'enterprise_tenant';
  reason: string;
  template_used?: string;     // e.g. "B-01", "E-01", "P2C-01"
}

// incident.update_posted
interface IncidentUpdatePostedPayload extends IncidentLifecycleEvent {
  event_type: 'incident.update_posted';
  channel: 'status_page' | 'enterprise_tenant_email' | 'board_email' | 'internal_slack';
  audience_scope: 'public' | 'enterprise_tenants_affected' | 'all_enterprise' | 'internal';
  privacy_floor_verified: boolean; // must be true; individual health data never in updates
}

// incident.containment_verified
interface IncidentContainmentVerifiedPayload extends IncidentLifecycleEvent {
  event_type: 'incident.containment_verified';
  containment_actions: string[];  // brief list of actions taken
  data_exfiltration_confirmed: boolean;
  hmac_chain_integrity: boolean;  // result of R-05 HMAC chain check post-containment
}

// incident.eradicated
interface IncidentEradicatedPayload extends IncidentLifecycleEvent {
  event_type: 'incident.eradicated';
  root_cause_category: 'config_error' | 'code_defect' | 'credential_compromise' |
    'third_party_breach' | 'insider_threat' | 'hardware_failure' | 'unknown';
  deploy_sha?: string;         // git SHA of the fix deploy, if applicable
}

// incident.recovered
interface IncidentRecoveredPayload extends IncidentLifecycleEvent {
  event_type: 'incident.recovered';
  duration_minutes: number;    // incident.opened → incident.recovered delta
  services_verified: string[]; // list of service endpoints verified healthy
  sla_breached: boolean;       // true if any enterprise SLA threshold was crossed
}

// incident.pir_opened
interface IncidentPirOpenedPayload extends IncidentLifecycleEvent {
  event_type: 'incident.pir_opened';
  pir_meeting_date: string;    // ISO 8601 date
  pir_doc_url: string;         // Link to the §8 PIR document draft
  deadline_utc: string;        // pir_closed must appear by this date (P0: +7d; P1: +14d)
}

// incident.pir_closed
interface IncidentPirClosedPayload extends IncidentLifecycleEvent {
  event_type: 'incident.pir_closed';
  action_items_count: number;
  critical_action_items: number; // priority 'Critical' per §14 registry
  pir_doc_url: string;
  days_since_recovery: number;
}
```

**HMAC chain integrity rule:** All ten `incident.*` events for a given `incident_id` form a sub-chain within the master DEC-030 HMAC chain. The `incident.opened` event's `hmac_prev` links to the last event in the master chain before the incident; each subsequent `incident.*` event for that incident links to the previous one. Domain-specific events (breach.*, legal.*, etc.) emitted during the same incident also link into the same master chain, interleaved by timestamp. The result: a tamper-evident timeline of both what happened and how it was managed.

---

### 16.4 SIEM-Triggered Incident Automation

OBSERVABILITY.md §27 defines six alert rules (AL-SIEM-01 through AL-SIEM-06) that fire when correlation rules detect anomalies. For two of those rules — AL-SIEM-03 (impossible travel, P1) and AL-SIEM-05 (HMAC chain break, P0) — the alert also *auto-opens* a FORM incident. This section specifies the automation architecture that bridges the OBSERVABILITY alerting layer to this runbook's lifecycle.

#### 16.4.1 Alert-to-Runbook Mapping

| Alert Rule | Severity | Auto-Open Incident | Auto-Assign Runbook | IC Paged | Human Confirmation Required |
|---|---|---|---|---|---|
| **AL-SIEM-01** | P2 (webhook delivery failure) | No — logged only | — | No | Engineer reviews within 2h |
| **AL-SIEM-02** | P2 (pipeline lag > 5 min) | No — logged only | — | No | devops-lead reviews within 1h |
| **AL-SIEM-03** | P1 (impossible travel match) | **Yes** | R-12 (Insider Threat) | Yes (PagerDuty) | IC must confirm P1 within 30 min of page |
| **AL-SIEM-04** | P1 (bulk data access match) | **Yes** | R-12 (Insider Threat) or R-01 (Data Breach) — IC determines | Yes (PagerDuty) | IC must confirm P1 and choose runbook within 30 min |
| **AL-SIEM-05** | P0 (HMAC chain break) | **Yes — P0 immediately** | R-01 (Data Breach), R-05 branch | Yes (PagerDuty P0 escalation) | IC must confirm P0 within 15 min |
| **AL-SIEM-06** | P1 (zero-events dead-man's switch) | **Yes** | Devops runbook (platform health) | Yes (PagerDuty) | IC must confirm P1 and diagnose pipeline within 30 min |

**Confirmation requirement:** Auto-opened incidents begin in status `AUTO_OPENED`. An IC must confirm the incident within the response window above, changing status to `IC_CONFIRMED`. If no confirmation arrives within the SLA window, a second PagerDuty escalation fires to the backup on-call.

**False-positive downgrade:** If the IC confirms the SIEM auto-open was a false positive, they must:
1. Set Linear ticket status to `FALSE_POSITIVE`
2. Emit `incident.opened` followed immediately by `incident.pir_closed` with `action_items_count: 0` and a `false_positive: true` flag in the payload (non-standard field, noted in payload)
3. Add a note to the `#inc-YYYYMMDD-xxx` channel explaining the false-positive basis
4. Add one action item to the §14 registry to tune the correlation rule to avoid recurrence

False positives are still emitted to DEC-030. The HMAC chain is never silent; a false-positive incident is evidence that the detection system is active and reviewed.

#### 16.4.2 Auto-Open Worker Architecture

The automation runs in a Cloudflare Worker `apps/api/src/workers/siem-incident-automator.ts`. It is triggered by PagerDuty Events API v2 webhooks (incident.trigger events from the FORM Security PagerDuty service defined in OBSERVABILITY §27).

```
PagerDuty Events API v2
        │  incident.trigger webhook (HTTPS, HMAC-signed)
        ▼
[siem-incident-automator Worker]
   1. Verify PagerDuty webhook HMAC (Workers Secret: PAGERDUTY_WEBHOOK_SECRET)
   2. Parse alert rule ID from alert.body.custom_details.alert_rule_id
   3. Look up runbook assignment from ALERT_RUNBOOK_MAP (§16.4.1)
   4. If auto-open = true:
      a. Create Linear issue via Linear API:
         - Title: "[AUTO] <alert_rule_id>: <alert_summary>"
         - Team: Security & Compliance
         - Priority: Urgent (P0) or High (P1)
         - Label: "SIEM_AUTO_OPEN", severity label
         - Description: SIEM correlation details + runbook link
      b. Generate incident_id: INC-<YYYYMMDD>-<seq>
      c. Emit incident.opened DEC-030 event:
         - incident_source: 'siem_automation'
         - actor_user_id: 'system'
         - trigger_alert_id: alert_rule_id
         - trigger_correlation_rule: rule if applicable
      d. Post to PagerDuty: acknowledge alert + add note with Linear URL
      e. Post to #incidents Slack channel: "@on-call P<N> auto-opened INC-<id>"
   5. Respond 200 OK to PagerDuty (within 3s to avoid retry)
```

The worker writes no user data and emits no telemetry. Its only I/O: PagerDuty inbound webhook → Linear API + DEC-030 emitter + Slack notification.

**Failure mode:** If the worker fails to create the Linear ticket (Linear API down, rate limit), it still emits the `incident.opened` DEC-030 event with `linear_ticket_url: 'PENDING'` and pages the on-call engineer directly via PagerDuty. The engineer creates the Linear ticket manually and patches the DEC-030 event `linear_ticket_url` field via the audit log amendment procedure (AUDIT_LOG_SCHEMA.md §4.2). The HMAC chain is not broken by this amendment; the amendment is itself an HMAC-chained event.

#### 16.4.3 IC Confirmation Flow

On receiving a PagerDuty page for a SIEM-auto-opened incident, the IC must:

1. Open the Linear ticket linked in the PagerDuty alert
2. Review the SIEM correlation details (rule, matched events, time window)
3. Within the response SLA (§16.4.1):
   - If confirmed real: Change Linear status to `IC_CONFIRMED`; emit `incident.ic_assigned` DEC-030; proceed with the assigned runbook
   - If false positive: Change Linear status to `FALSE_POSITIVE`; emit lifecycle pair per §16.4.1 false-positive procedure; tune the correlation rule
4. Post first update to `#inc-YYYYMMDD-xxx` Slack channel within the response SLA

If the IC cannot confirm within SLA (e.g., page was missed), the backup on-call receives a second escalation. The incident remains `AUTO_OPENED` until a human confirms. No automated action beyond ticket creation occurs — containment, evidence collection, and communication are always human-gated.

---

### 16.5 Incident Management Meta-Monitoring

The incident response program is itself a system that can degrade. This section defines how FORM monitors the incident management process — watching the watchdog.

#### 16.5.1 Incident Response SLOs

These SLOs measure the *quality of the incident management process*, not the underlying platform. They feed into the §14 Quarterly Control Effectiveness Review as CC7.4 evidence.

| SLO ID | Metric | Target | Measurement | Alert Threshold | Alert Route |
|---|---|---|---|---|---|
| **IR-SLO-01** | IC assignment lag: `incident.ic_assigned.minutes_since_open` | P0 ≤ 15 min; P1 ≤ 30 min | DEC-030 query on `incident.ic_assigned` events per month | > target for any P0/P1 in period | security-engineer + devops-lead Slack DM |
| **IR-SLO-02** | PIR open rate: `incident.pir_opened` emitted for every P0/P1 `incident.recovered` | 100% | DEC-030 join: `incident.recovered` left join `incident.pir_opened` on `incident_id` | Any P0/P1 missing PIR open event after 48h post-recovery | compliance-officer Linear comment + Slack DM |
| **IR-SLO-03** | PIR completion lag: `incident.pir_closed` within deadline field of `incident.pir_opened` | P0: ≤ 7 days; P1: ≤ 14 days | DEC-030 query on `incident.pir_closed.days_since_recovery` | Approaching deadline (T-24h) and still open | compliance-officer Slack DM |
| **IR-SLO-04** | Critical action item closure: §14 registry `priority='Critical'` items closed by SLA | 100% within 14 days | Linear API query on `[IR] Action Items` project, `priority='Critical'`, `status != Done` | Open Critical item approaching 14-day SLA | devops-lead + founder Slack DM |
| **IR-SLO-05** | SIEM false-positive rate: `incident.opened` events with `incident_source='siem_automation'` that result in `FALSE_POSITIVE` status | < 20% per quarter | DEC-030 query: `incident.opened` WHERE `incident_source='siem_automation'` GROUP BY false_positive flag, quarter | > 20% false-positive rate in trailing quarter | devops-lead + security-engineer: tune correlation rules |

#### 16.5.2 Dead-Man's Switch: Overdue PIR Detection

A pg_cron job runs daily at 06:00 UTC:

```sql
-- Detect P0/P1 incidents missing pir_opened after 48h post-recovery
SELECT
  recovered.incident_id,
  recovered.incident_severity,
  recovered.timestamp_utc AS recovered_at,
  EXTRACT(EPOCH FROM (NOW() - recovered.timestamp_utc::timestamptz)) / 3600 AS hours_since_recovery
FROM audit_log_events recovered
WHERE recovered.event_type = 'incident.recovered'
  AND recovered.incident_severity IN ('P0', 'P1')
  AND recovered.timestamp_utc > NOW() - INTERVAL '30 days'
  AND NOT EXISTS (
    SELECT 1 FROM audit_log_events pir
    WHERE pir.event_type = 'incident.pir_opened'
      AND pir.incident_id = recovered.incident_id
  )
HAVING hours_since_recovery > 48;
```

If rows are returned: Slack alert to `#compliance` tagging `@compliance-officer`. If the same `incident_id` appears for 72h without `incident.pir_opened`, a Linear ticket is created automatically (P1 priority, `[IR] PIR Overdue` label) and the compliance-officer receives a PagerDuty low-urgency page.

Similarly, a dead-man's switch detects PIR open without close past the deadline:

```sql
-- Detect pir_opened past its deadline without pir_closed
SELECT
  pir_open.incident_id,
  pir_open.incident_severity,
  pir_open.payload->>'deadline_utc' AS deadline_utc,
  NOW() AS now_utc
FROM audit_log_events pir_open
WHERE pir_open.event_type = 'incident.pir_opened'
  AND (pir_open.payload->>'deadline_utc')::timestamptz < NOW()
  AND NOT EXISTS (
    SELECT 1 FROM audit_log_events pir_close
    WHERE pir_close.event_type = 'incident.pir_closed'
      AND pir_close.incident_id = pir_open.incident_id
  );
```

Rows returned → Slack alert to `#compliance` tagging `@compliance-officer` and `@security-engineer`. The §14 PIR escalation thresholds (Overdue Critical: founder paged) apply from the moment a deadline passes.

#### 16.5.3 Monthly Meta-Monitoring Report

The `audit-chain-daily-check` pg_cron job (OBSERVABILITY.md §14.3 / SOC2_READINESS.md §46) is extended to produce a monthly incident meta-monitoring summary:

```sql
-- Monthly incident lifecycle completeness report
SELECT
  DATE_TRUNC('month', opened.timestamp_utc::timestamptz) AS month,
  opened.incident_severity,
  COUNT(*) AS incidents_opened,
  COUNT(ic_assigned.incident_id) AS with_ic_assigned,
  COUNT(recovered.incident_id) AS with_recovered,
  COUNT(pir_opened.incident_id) AS with_pir_opened,
  COUNT(pir_closed.incident_id) AS with_pir_closed,
  AVG(ic_assigned_ev.payload->>'minutes_since_open'::numeric) AS avg_ic_lag_min,
  AVG(pir_closed_ev.payload->>'days_since_recovery'::numeric) AS avg_pir_days
FROM audit_log_events opened
LEFT JOIN audit_log_events ic_assigned_ev
  ON ic_assigned_ev.event_type = 'incident.ic_assigned'
  AND ic_assigned_ev.incident_id = opened.incident_id
-- ... (additional joins for each lifecycle event)
WHERE opened.event_type = 'incident.opened'
  AND opened.incident_severity IN ('P0', 'P1', 'P2')
GROUP BY 1, 2
ORDER BY 1 DESC, 2;
```

Output is saved to `compliance/evidence/ir-meta-monitoring/YYYY-MM.json` and surfaced in the OBSERVABILITY §7 SOC 2 Evidence Dashboard. This is the primary CC7.4 evidence artefact that demonstrates the incident response program runs to completion on every incident, not just that it is documented.

---

### 16.6 DEC-030 Audit Event Registration

The ten `incident.*` events must be registered in `docs/AUDIT_LOG_SCHEMA.md` event registry. Registration summary:

| Event Type | Category | DEC-030 Severity | Retention | Privacy Notes |
|---|---|---|---|---|
| `incident.opened` | Incident Management | HIGH | 7 years | No user PII in payload; `incident_id` is pseudonymous INC-YYYYMMDD-NNN |
| `incident.ic_assigned` | Incident Management | MEDIUM | 7 years | `ic_user_id` is internal role identifier, not end-user PII |
| `incident.severity_changed` | Incident Management | HIGH | 7 years | `reason` field: IC must not include user PII; free-text field is audited |
| `incident.escalated` | Incident Management | HIGH | 7 years | `escalation_target: 'enterprise_tenant'` must not include tenant employee names |
| `incident.update_posted` | Incident Management | STANDARD | 3 years | `audience_scope` and `privacy_floor_verified` fields are mandatory |
| `incident.containment_verified` | Incident Management | HIGH | 7 years | `containment_actions` list: must not include user data; reference R-number only |
| `incident.eradicated` | Incident Management | HIGH | 7 years | `deploy_sha` is a git commit hash; no user data |
| `incident.recovered` | Incident Management | HIGH | 7 years | `services_verified` list: endpoint names only, no user data |
| `incident.pir_opened` | Incident Management | MEDIUM | 7 years | `pir_doc_url` links to internal doc; ensure link is not publicly accessible |
| `incident.pir_closed` | Incident Management | HIGH | 7 years | `action_items_count` and `critical_action_items` are counts, no user data |

**Chain break rule:** A break in the HMAC chain within the `incident.*` sub-chain for a given `incident_id` is treated identically to a break in the master chain: P0 per R-05, forensic write-freeze activated, external forensic investigator engaged. The sub-chain break implies either tampering with the incident record or a bug in the HMAC computation pipeline.

---

### 16.7 SOC 2 Evidence Mapping

| Criterion | Evidence from §16 |
|---|---|
| **CC7.4** — Entity implements policies and procedures to respond to security incidents | `incident.opened` events provide the authoritative incident count and timing; `incident.ic_assigned` proves IC assignment SLO compliance; `incident.recovered` proves resolution; the complete lifecycle chain for each incident_id is the primary CC7.4 evidence corpus for the SOC 2 observation period |
| **CC7.5** — Entity performs post-incident reviews to identify control deficiencies | `incident.pir_opened` and `incident.pir_closed` events directly evidence that PIR occurs for every P0/P1; `action_items_count` and `critical_action_items` in `incident.pir_closed` prove control deficiencies are identified; the §14 PIR Action Item Registry links from `incident.pir_closed` to the remediation record |
| **CC4.1** — Entity uses information from ongoing monitoring to evaluate control effectiveness | §16.5 incident response SLOs (IR-SLO-01 through IR-SLO-05) are the ongoing monitoring mechanism; the monthly meta-monitoring report to `compliance/evidence/ir-meta-monitoring/` is the evaluation artefact; the quarterly control effectiveness review (§14.4) consumes this data for the CC4.1 self-assessment |
| **CC4.2** — Entity evaluates and communicates internal control deficiencies | `incident.pir_closed.critical_action_items > 0` triggers a §14 registry entry; overdue Critical action items trigger founder escalation per §16.5.1 IR-SLO-04; the §14 quarterly review communicates aggregate deficiency trends to the compliance-officer and founder |
| **CC7.2** — Entity monitors system components | IR-SLO-05 (false-positive rate monitoring) creates a feedback loop ensuring SIEM correlation rules are continuously tuned; the dead-man's switch queries (§16.5.2) themselves constitute monitoring of the incident response system |
| **CC2.2** — Entity communicates information externally to users | `incident.update_posted` with `privacy_floor_verified: true` is the auditable record that external communications — status page updates, enterprise tenant notifications — were reviewed for privacy floor compliance before being sent |

**Primary auditor evidence artefacts:**

| Artefact ID | Description | Collection Method |
|---|---|---|
| **IR-META-E-001** | Complete `incident.*` lifecycle chain for all P0/P1 incidents in the observation period, with HMAC chain verified | `SELECT * FROM audit_log_events WHERE event_type LIKE 'incident.%' AND incident_severity IN ('P0','P1') AND timestamp_utc BETWEEN $obs_start AND $obs_end ORDER BY timestamp_utc` + `audit-chain-daily-check` verification report |
| **IR-META-E-002** | Monthly IR meta-monitoring reports for each month in the observation period | `compliance/evidence/ir-meta-monitoring/YYYY-MM.json` — all months in observation window |
| **IR-META-E-003** | PIR completion rate: join of `incident.recovered` and `incident.pir_closed` events proving 100% PIR rate for P0/P1 | Derived from IR-META-E-001 query; any missing `incident.pir_closed` for a P0/P1 is a gap finding |
| **IR-META-E-004** | §14 PIR Action Item Registry export for the observation period | Linear export of `[IR] Action Items` project filtered to observation period; `compliance/evidence/ir-action-items/YYYY-MM.csv` series |
| **IR-META-E-005** | SIEM-to-incident automation log: all `incident.opened` events with `incident_source='siem_automation'`, their AL-SIEM trigger rule, and final status (confirmed / false positive) | `SELECT * FROM audit_log_events WHERE event_type = 'incident.opened' AND incident_source = 'siem_automation' AND timestamp_utc BETWEEN $obs_start AND $obs_end` |

---

### 16.8 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Register all ten `incident.*` events in `docs/AUDIT_LOG_SCHEMA.md` event registry with schema, severity, and retention fields per §16.6 | security-engineer | **P0** | M4 |
| Implement `incident.*` HMAC chain writer in the standard DEC-030 emitter (`apps/api/src/audit/emitter.ts`); validate sub-chain linking (each `incident.*` event's `hmac_prev` links to previous event in the master chain) in staging with a scripted test incident | platform-engineer | **P0** | M4 |
| Build `siem-incident-automator` Cloudflare Worker (§16.4.2): PagerDuty webhook verification, Linear issue creation, `incident.opened` emission with `incident_source='siem_automation'`; deploy to production with Workers Secret `PAGERDUTY_WEBHOOK_SECRET` | platform-engineer | **P0** | M4 |
| Configure PagerDuty → `siem-incident-automator` webhook for the FORM Security service: enable for `incident.trigger` events only; add HMAC signing; test with synthetic AL-SIEM-05 (HMAC chain break) trigger in staging | devops-lead | **P0** | M4 |
| Wire IC confirmation flow: PagerDuty acknowledgement → Linear `IC_CONFIRMED` status update → `incident.ic_assigned` DEC-030 emission; validate confirmation lag is captured in `minutes_since_open` field | platform-engineer | **P1** | M4 |
| Implement dead-man's switch pg_cron jobs (§16.5.2): PIR-missing detection (runs daily 06:00 UTC) and PIR-overdue detection; wire Slack notifications and Linear auto-ticket creation on breach | devops-lead | **P1** | M4 |
| Extend `audit-chain-daily-check` pg_cron job to output monthly IR meta-monitoring summary SQL (§16.5.3); save output to `compliance/evidence/ir-meta-monitoring/YYYY-MM.json`; confirm first month's output is correctly formatted | devops-lead + data-engineer | **P1** | M5 |
| Add IR-SLO-01 through IR-SLO-05 metric queries (§16.5.1) to the OBSERVABILITY §7 SOC 2 Evidence Dashboard in Metabase; set Slack alert thresholds per §16.5.1 table | devops-lead | **P1** | M5 |
| Tabletop exercise validating the SIEM auto-open flow end-to-end: synthetic AL-SIEM-05 trigger → siem-incident-automator → Linear ticket → PagerDuty page → IC confirmation → `incident.ic_assigned` DEC-030 → R-01 runbook start; record elapsed time vs. 15-min P0 SLA; document in `compliance/evidence/annual-csa/YYYY-siem-automation-drill.md` | security-engineer + devops-lead | **P1** | M5 |
| Update §14.3 Quarterly Control Effectiveness Review agenda to include IR-SLO-01–05 results and IR-META-E-001–E-005 review as standing agenda items | compliance-officer | **P2** | M5 |
| Add `incident.update_posted` emission call to the §12 enterprise tenant notification templates (E-01 through E-04) and §13 board communication templates (B-01 through B-04); add `privacy_floor_verified: true` assertion as mandatory pre-send gate | platform-engineer | **P2** | M5 |
| Update `docs/SOC2_READINESS.md` CC7.4 row from 🟡 to 🟢 after all P0 checklist items are verified in staging and first month of IR-META-E-001 artefact is available | compliance-officer | **P2** | M5 |

---

### 16.9 Open Questions

**OQ-IR-01: DEC-030 sub-chain vs. master chain ordering for concurrent incidents**

~~When two incidents are simultaneously active (e.g., a P0 breach discovered while a P1 SIEM false-positive is being adjudicated), both incident ID sequences write to the same master DEC-030 HMAC chain. The interleaving order is timestamp-based, which means the sub-chains for INC-A and INC-B are non-contiguous segments of the master chain. Auditors verifying INC-A's chain must skip INC-B events, which complicates verification tooling. Options: (a) Keep current design and document the skip-and-verify algorithm for auditors — acceptable complexity given concurrent incidents are rare. (b) Implement per-incident parallel HMAC sub-chains that rejoin the master chain at `incident.recovered` — cleaner for auditors but introduces a chain merge step not currently in the DEC-030 spec. Owner: security-engineer. Priority: P2 (resolve before SOC 2 Type II observation period opens). Resolution target: M6.~~ **🟢 Resolved (2026-06-14, v2.6) — see §18 (DEC-053).** Option (a) adopted: timestamp-interleaved master chain retained; documented skip-and-verify SQL (§18.3.1) extracts per-incident event sequences for auditors; CONC-CHAIN-01 cross-contamination invariant established (§18.3.3).

**OQ-IR-02: Linear API as a control dependency**

~~The `siem-incident-automator` Worker depends on the Linear API for ticket creation. If Linear is unavailable when a SIEM auto-open triggers, the worker falls back to PagerDuty-only notification (§16.4.2). However, if the engineer on call is paged via PagerDuty without a Linear ticket, they have no authoritative incident ID to attach to DEC-030 events until the ticket is manually created and the `linear_ticket_url` field is patched via the amendment procedure. This creates a short window where lifecycle events have `linear_ticket_url: 'PENDING'`, which auditors may flag. Options: (a) Generate the `incident_id` in the Worker before Linear API call so DEC-030 can emit immediately with the correct ID, and create Linear ticket asynchronously with the same ID as the title prefix. (b) Pre-generate a pool of incident IDs and consume the next available one on failure. Owner: platform-engineer. Priority: P1. Resolution target: M4 implementation sprint.~~ **🟢 Resolved (2026-06-14, v2.4) — see §17.2.** Option A adopted: `siem-incident-automator` generates `incident_id` as its first operation before any external API call; `incident.opened` is emitted at T+0 with `linear_ticket_url: "PENDING"`; Linear ticket is created asynchronously (5× exponential-backoff retry); `incident.linear_ticket_linked` (new LOW event, 7yr) closes the linkage loop. AL-IR-LINEAR-01 (P2 Slack) fires on persistent Linear failure; IC uses amendment endpoint `POST /internal/v1/incident/link-ticket`.

**OQ-IR-03: Privacy floor for `incident.severity_changed.reason` free-text field**

~~The `reason` field in `incident.severity_changed` is IC-authored free text. Under time pressure, an IC may inadvertently include a user's name, email, or health data category when describing why severity was upgraded. The DEC-030 record is retained for 7 years and cannot be deleted. Mitigation options: (a) Add a real-time regex filter in the emitter that rejects the event if the reason field matches patterns for email addresses, Ukrainian national ID numbers, or known health category terms — and prompts the IC to rephrase. (b) Document the prohibition clearly in the IC training checklist and accept residual risk. (c) Require reason field to reference only incident_id, runbook IDs, and DEC-030 event types, and enforce this constraint in the emitter schema validator using an allowlist. Owner: security-engineer + compliance-officer. Priority: P2. Resolution target: M4 emitter implementation review.~~ **🟢 Resolved (2026-06-14, v2.4) — see §17.3.** Option A + SHA-256 hash masking adopted (consistent with DEC-044 `bypass_reason_hash` pattern, §40): `reason_plaintext` is never stored in the DEC-030 chain; `reason_hash = SHA-256(incident_id + '|' + reason_plaintext + '|' + INCIDENT_REASON_HASH_SALT)` is the chain record; plaintext lives exclusively in the Linear ticket. Runtime blocklist (email, national IDs, Art. 9 terms) emits advisory `incident.pii_risk_detected` (MEDIUM, 7yr) to Slack `#security-ops` without blocking incident workflow. `INCIDENT_REASON_HASH_SALT` catalogued in CRYPTOGRAPHY_POLICY.md §5 (annual rotation).

---

*v1.0 additions (2026-06-01): §16 DEC-030 Incident Lifecycle Audit Events & SIEM-Triggered Automation. Closes the gap between OBSERVABILITY.md §27 (added 2026-06-01), which defined AL-SIEM-05 auto-opening R-01 and AL-SIEM-03/04 routing to R-12, and the incident response runbook's lack of a documented, authoritative specification for that automation. §16.1 purpose statement: distinguishes domain-specific DEC-030 events (breach.*, legal.*, dsar.*) from the missing cross-cutting lifecycle audit trail; two-problem framing — SOC 2 CC7.4 evidence gap and undocumented SIEM automation. §16.2 ten-event lifecycle taxonomy table: incident.opened (HIGH, 7yr, CC7.3/CC7.4), incident.ic_assigned (MEDIUM, 7yr), incident.severity_changed (HIGH, 7yr — covers both upgrade and IC-approved downgrade per §1.3), incident.escalated (HIGH, 7yr, CC7.4/CC2.2), incident.update_posted (STANDARD, 3yr, CC2.2), incident.containment_verified (HIGH, 7yr), incident.eradicated (HIGH, 7yr), incident.recovered (HIGH, 7yr, CC7.4/CC7.5), incident.pir_opened (MEDIUM, 7yr, CC7.5), incident.pir_closed (HIGH, 7yr, CC7.5/CC4.1). §16.3 TypeScript schema for common base and all ten event-specific payload interfaces; HMAC sub-chain specification (incident.opened links to master chain; subsequent incident.* events link in sequence; domain-specific events interleave by timestamp; sub-chain break = P0 per R-05). §16.4 SIEM-triggered automation: six-row alert-to-runbook mapping table (AL-SIEM-01/02 log-only; AL-SIEM-03/04/06 auto-open P1 with R-12 or devops runbook; AL-SIEM-05 auto-open P0 with R-01 immediately); siem-incident-automator Cloudflare Worker architecture (PagerDuty Events API v2 webhook → Linear issue creation → incident.opened DEC-030 → Slack notification; 200 OK within 3s); false-positive protocol (FALSE_POSITIVE status → incident lifecycle pair emission → §14 action item for correlation rule tuning; HMAC chain never silent); IC confirmation flow (AUTO_OPENED → IC_CONFIRMED within P0 15 min / P1 30 min SLA; backup escalation on SLA breach). §16.5 incident management meta-monitoring: five IR-SLOs — IR-SLO-01 IC assignment lag (P0 ≤15 min, P1 ≤30 min), IR-SLO-02 PIR open rate (100% for P0/P1), IR-SLO-03 PIR completion lag (P0 ≤7 days, P1 ≤14 days), IR-SLO-04 Critical action item closure (100% within 14 days), IR-SLO-05 SIEM false-positive rate (< 20%/quarter); two dead-man's switch pg_cron queries (PIR-missing detection at T+48h post-recovery → Slack + Linear auto-ticket; PIR-overdue past deadline field → Slack + PagerDuty low-urgency); monthly meta-monitoring SQL report to compliance/evidence/ir-meta-monitoring/YYYY-MM.json. §16.6 AUDIT_LOG_SCHEMA.md registration table for all ten events with privacy notes (no end-user PII in any lifecycle event payload; free-text fields audited). §16.7 SOC 2 evidence mapping: CC7.4 (incident.opened/ic_assigned/recovered = primary evidence corpus), CC7.5 (incident.pir_opened/pir_closed), CC4.1 (IR-SLO data + monthly meta-monitoring report), CC4.2 (critical_action_items > 0 → §14 registry entry → founder escalation), CC7.2 (IR-SLO-05 creates SIEM tuning feedback loop), CC2.2 (incident.update_posted with privacy_floor_verified); five evidence artefacts IR-META-E-001 through IR-META-E-005 with collection SQL and file paths. §16.8 twelve-item implementation checklist: four P0 M4 items (event registry, HMAC emitter, siem-incident-automator Worker, PagerDuty webhook configuration); five P1 items (IC confirmation flow, dead-man's switch pg_cron jobs, monthly meta-monitoring extension, Metabase IR-SLO dashboard, SIEM automation tabletop drill M5); three P2 items (§14 agenda update, template emission integration, SOC2_READINESS CC7.4 row update). §16.9 three open questions: OQ-IR-01 (concurrent incident sub-chain interleaving — P2, security-engineer, M6); OQ-IR-02 (Linear API as control dependency — P1, platform-engineer, M4); OQ-IR-03 (privacy floor for free-text reason field in severity_changed — P2, security-engineer + compliance-officer, M4 emitter review). Advances INCIDENT_RESPONSE internal version v0.9 → v1.0. SOC 2 evidence: primary CC7.4 artefact corpus established.*

---

**v1.2 · 2026-06-02 · Owner: security-engineer + compliance-officer**
**Review: after every P0/P1 incident, minimum annual.**
**Next scheduled review: June 2027 or after first P0/P1 — whichever comes first.**

---

*v0.9 additions (2026-05-30): §15 GDPR Article 33/34 Data Breach Notification Operational Runbook. Closes the gap between §10 (legal framework overview) and the step-by-step operational procedure required when a breach occurs — the two sections are complementary, not duplicative: §10 defines the law, §15 executes the procedure. Awareness clock taxonomy (§15.2): six trigger events mapped to the person responsible for determining clock start (devops-lead, compliance-officer, security-engineer), plus clock override rule for uncertain-but-probable breaches mandating partial Art. 33(4) filing within 72h. Incident-to-notification decision matrix (§15.3): ten breach scenarios mapped to Art. 33 required / Art. 34 required / optional with rationale and assessment authority assignments; Art. 9 data triggers Art. 34 presumption overridable only with documented mitigation. Multi-tenant breach scoping SQL (§15.4): three queries with form_admin BYPASSRLS — Query 1 (affected user count with Art. 9 / biometric / contact data breakdown), Query 2 (enterprise tenant list with affected_seat_count and has_art9_exposure flag for B2B notifications), Query 3 (HMAC chain integrity verification in exposure window; chain break = P0 per R-05); results routed exclusively to restricted incident channel. 72-hour clock procedure (§15.5): eleven timestamped steps from T+0h declaration through post-filing supplementary submissions with named owners and output artefacts. Template A33-01 (§15.6): complete Article 33 supervisory authority notification template with all required GDPR fields (controller details, breach description, data categories and approximate subject count, likely consequences, measures taken, partial-notification declaration, internal reference). Template A34-01 (§15.7): plain-language Article 34 data subject notification template with explicit instruction to omit Art. 9 category label from email subject line (prevents header-based re-exposure); delivery via Resend breach_notification slot; failed-delivery Art. 34(3)(c) exception path with 30-day status page substitute. Template P2C-01 (§15.8): Art. 28 processor-to-controller enterprise tenant notification with T+8h target, 48h contractual maximum, per-tenant tracking record. Ten DEC-030 HMAC-chained events (§15.9): breach.awareness_declared (CRITICAL, 7yr), breach.scope_assessment_completed (HIGH, 7yr), breach.art33_obligation_determined (HIGH, 7yr), breach.processor_notified_tenant (HIGH, 7yr), breach.art33_notification_filed (CRITICAL, 7yr — primary P7.1 auditor evidence with hours_since_awareness field), breach.art33_supplementary_filed (HIGH, 7yr), breach.art34_notifications_sent (HIGH, 7yr), data.breach_notification_sent (STANDARD, 7yr — per-user with email_hash not raw email), breach.art34_waiver_documented (HIGH, 7yr), breach.notification_case_closed (HIGH, 7yr); chain break on breach.* is P0 per R-05. Evidence directory structure (§15.10): eight subdirectories (scope-assessment, drafts, art33, art34, tenant-notifications, board, legal, dec030) with SHA-256 manifest generation command and git commit tamper-evident sealing. Post-notification follow-up (§15.11): supplementary Art. 33(4) cadence (within 7 days of material new information), DPA correspondence tracking protocol, bounce / Art. 34(3)(c) exception path, five PIR review questions specific to breach notification process quality. SOC 2 privacy criterion mapping (§15.12): P7.1 (timely notification — breach.art33_notification_filed CRITICAL event with hours_since_awareness is primary auditor evidence), P8.1 (disposal chain via PIR integration), P6.1 (processor-to-controller disclosure evidenced by breach.processor_notified_tenant), CC2.2 (board + tenant external communication), CC7.3 (§15.3 decision matrix as documented evaluation with auditable DEC-030 outcome). Implementation checklist (§15.13): twelve tasks across M4/M5 (7× P0 M4, 4× P1 M4, 1× P1 M5 tabletop drill Scenario J, 1× P2 M5 annual schedule integration). Advances SOC2_READINESS.md §35 P7.1 from 🟡 to target 🟢 on checklist completion.*

---

**v0.9 · 2026-05-30 · Owner: security-engineer + compliance-officer**
**Review: after every P0/P1 incident, minimum annual.**
**Next scheduled review: May 2027 or after first P0/P1 — whichever comes first.**

---

*v0.2 additions: R-07 Sub-processor Breach Notification Received (new runbook — vendor-originated incident protocol, GDPR Art. 33 clock management when breach is at processor level, enterprise tenant notification, evidence package, post-incident vendor management review). §11 Sub-processor Incident Management (processor registry for Supabase/Anthropic/ElevenLabs/Cloudflare/PostHog/Sentry/Stripe with Art. 9 classification and severity floors; known DPA gaps including Anthropic notification SLA and ElevenLabs unsigned DPA; Art. 28(3)(f) contractual requirements and DPA signing checklist; notification SLA tracker; SOC 2 CC9.2 evidence integration). Appendix A updated to include R-07 quick reference and sub-processor context for GDPR clock.*

*v0.3 additions: R-08 Supply Chain Attack / Compromised Dependency — trigger matrix by package location and runtime access surface; immediate isolation protocol (freeze deploys, EAS revocation); blast radius assessment checklist; dependency tree audit with clean-build verification; preventive CI controls (Dependabot, npm audit, Socket.dev); SOC 2 CC6.1/CC6.8/CC7.2/CC7.4 evidence package. R-09 Ransomware / Destructive Payload — always-P0 protocol with explicit isolation-first ordering (overrides the standard evidence-first rule); no-pay policy with decision authority constraints; Supabase PITR recovery to new project; full secrets rotation sequence; enterprise tenant validation before re-enabling traffic; law enforcement preservation guidance; GDPR note on destruction as a reportable breach. §9.3 Scenarios E and F — supply chain (backdoored npm package in Workers env) and ransomware (staging-to-production lateral movement) tabletop scenarios with discussion point frameworks; both include enterprise customer-success participation requirement. §9.5 Year 2 Testing Schedule — 6-scenario rotation covering all runbook types; cross-functional drills (customer-success joins E and F). §12 Enterprise Tenant SLA Breach & Incident Communication Protocol — SLA tier definition and breach thresholds for API availability, P95 latency, and SSO; credit schedule (10%/25%/50% by uptime band); contact responsibility matrix with timing SLAs; privacy floor enforcement for tenant communications; Templates E-01 (initial notification), E-02 (60-min status update), E-03 (resolution), E-04 (SLA credit); SOC 2 evidence mapping (CC9.2, A1.1, A1.2, CC7.3, CC2.2). Appendix A updated to include R-08, R-09, and §12 enterprise comms reference.*

*v0.4 additions: R-10 AI Coach Safety Incident (Victor Harmful Guidance) — first AI-specific incident runbook in the taxonomy. Eight-level severity matrix distinguishing isolated quality issues (P2), systematic harmful patterns (P1), ED-adjacent content or injury reports (P0), and prompt injection vectors (P0 security incident). Three incident sub-types with different response branches: prompt regression (system prompt rollback + clinical-safety review gate), model behavior drift (Anthropic safety team notification + R-07 cross-reference), prompt injection (P0 security incident + input sanitization). Immediate actions include coaching_turns read-only query protocol with content_hash-only channel references (ED-adjacent text not distributed to responders). Blast radius assessment queries by victor_prompt_version. Containment: feature-flag scoping (pathway not full product), edge Worker content filter middleware, clinical-safety veto on re-enable decisions. Eradication: prompt diff methodology, Anthropic safety reporting path, Victor Jailbreak Test Suite addition to §9 testing program, clinical-safety PR review gate for all prompt changes. Evidence package: SOC 2 CC7.4/CC5.2/CC1.2 artifacts. Clinical-safety note: restricted compliance folder for harmful content (clinical-safety + compliance-officer + security-engineer + founder access only). Regulatory notes: GDPR Art. 34 physical injury path, Art. 33 prompt-injection health-data branch, Anthropic sub-processor R-07 parallel activation. No-go customer escalation: wellness-as-punishment detection triggers founder escalation. Appendix A updated with R-10 quick reference.*

*v0.5 additions: R-11 CV / Pose Estimation Biometric Data Privacy Incident — second AI-adjacent runbook; addresses GDPR Art. 9 biometric data specifically (distinct from R-01 general health-data breach). Architecture-aware two-track design: Track A (on-device device loss — raw frames never server-side) vs. Track B (server-side `cv_sessions.keypoints_enc` exposure). Severity table: always P0 for `keypoints_enc` RLS failure or key compromise; P1 for k-anonymity fleet stats. Scope assessment SQL: tenant/user/session count in exposure window; keypoints_enc null/populated ratio; RLS anon-role verification. Containment: cv_enabled feature flag global disable; cv_keypoints_encryption_key rotation in Workers Secrets; pg_cron pause for `cv_session_fleet_stats` refresh. GDPR Art. 9 notification assessment table with Art. 34 presumption for biometric data (overrides standard R-01 assessment that sometimes waives Art. 34); 72h notification timeline ladder (T+4h assessment, T+24h draft, T+48h file partial, T+72h hard deadline). Enterprise tenant CV-specific notification requirements including DPO coordination for joint-controller Art. 33 filings. Clinical-safety mandatory coordination: movement-pattern health inference review; Art. 34 notification text sign-off gate. Evidence package: 12-item checklist including encryption key access audit trail and OBSERVABILITY.md §18 dashboard screenshots. Post-incident mandatory controls: encryption key rotation schedule, `keypoints_enc` storage necessity review, k-anonymity enforcement verification, OBSERVABILITY.md §18 alerting confirmation, five-way re-enable gate (ml-engineer + security-engineer + clinical-safety). §13 Board & Investor Communication Protocol — six-trigger threshold table (P0 > 2h, Art. 9 breach, enterprise tenant loss, media coverage, regulatory action, financial impact > $10k). Founder-exclusive authority (no other role contacts board during incident). Five-template set: B-01 Initial Notification (T+4h), B-02 Status Update (every 12h), B-03 Resolution (T+4h post-closure), B-04 Post-Incident Summary (within 5 business days of PIR); each template includes explicit "what NOT to write" constraints (no root cause, no attribution, no legal conclusions in B-01/B-02). Email-first channel requirement; phone call for biometric breach / press / customer loss. Evidence archiving in `compliance/evidence/incident-comms/<slug>/board/` retained 7 years per DEC-030. SOC 2 mapping: CC2.2 (board communication), CC9.1 (risk mitigation commitment), CC7.3 (anomaly escalation), A1.1 (availability commitment). Appendix A updated with R-11 and §13 quick references.*

*v0.7 additions: R-13 Law Enforcement / Government Data Request — thirteenth runbook; operationalizes `PRIVACY_POLICY.md §6.4` and ENTERPRISE.md no-backdoor policy into a full incident-level response procedure. Request type taxonomy: 9 types from criminal subpoena through FISA order to EU DPA supervisory authority demand and informal voluntary requests. Severity matrix: P0 for NSL/FISA, foreign intelligence orders, Art. 9 requests for ≥ 10 users, and enterprise-tenant-scope requests; P1 for criminal court order or EU DPA investigation; P2 for civil subpoena; P3 for informal requests. Eight unique response constraints: founder-only authority (no disclosure without founder sign-off); legal counsel required before any action (P0: before acknowledging receipt); narrow scope enforcement (only what the instrument compels); Art. 9 no-voluntary-production rule; emergency exception protocol for claimed life-safety informal requests (counsel within 30 minutes, minimum-necessary, CRITICAL DEC-030 event); GDPR Art. 48 analysis required for non-EU court orders on EU-resident data; no-backdoor reaffirmation (unconditional refusal of programmatic access or surveillance hooks); restricted channel `#inc-YYYYMMDD-legal` (founder + compliance-officer + counsel only — legal privilege protection). 11-step legal review process with timing SLAs per severity tier. Challenge protocol for overbroad requests: temporal scope, user-class requests, Art. 9 production, Art. 48 non-treaty orders. Scope minimization table: permitted-for-production list (user ID, timestamps, email, SSO events, subscription status, IP logs); higher-legal-standard list (all Art. 9 health/coaching/wearable data); never-produce list (CV keypoints_enc — architecturally impossible; fleet stats — aggregate only; uninstrumented user data). User notification protocol: default notify-before-produce unless gag order; template for permitted notifications; GDPR Art. 34 co-trigger assessment for Art. 9 production. Enterprise tenant notification: Template T-01 (within 30 min P0, 4h P1, 24h P2); prohibition memo if gag order applies. GDPR Art. 48 analysis section: MLAT status table (US DOJ — partial EU-US MLAT 2003 non-self-executing; UK — post-Brexit uncertainty; others case-by-case); Art. 48 compliance procedure (treaty basis check, DPA notification before production, limited Art. 49 derogation path); DPA contact table (BfDI/Germany, DPC/Ireland one-stop-shop, UAPDP/Ukraine). Evidence package: structured directory (`received/`, `counsel/`, `disclosure/`, `notifications/`) with SHA-256 manifest; 7-year retention per DEC-030. 10 DEC-030 HMAC-chained audit events: `legal.request_received` (HIGH), `legal.counsel_retained` (MEDIUM), `legal.legal_hold_activated` (HIGH), `legal.challenge_filed` (MEDIUM), `legal.user_notified` (MEDIUM), `legal.tenant_notified` (MEDIUM), `legal.dpa_notified` (HIGH), `legal.disclosure_executed` (CRITICAL), `legal.emergency_voluntary_disclosure` (CRITICAL), `legal.legal_hold_released` (MEDIUM). Transparency reporting: annual Government Transparency Report (request counts by type, disclosure vs. challenge vs. decline, users affected, gag-order prohibition count, Art. 48 outcomes); NSL "0 to 499" range disclosure method. SOC 2 mapping: CC6.3 (government access authorization documented and auditable), CC6.1 (no-backdoor and no-informal-disclosure rules protect information assets), C1.1 (Art. 9 never-produce list enforces health data confidentiality), CC9.1 (enterprise tenant notification obligation), P6 Privacy (user notification + transparency report). Tabletop Scenario H: US DOJ criminal grand jury subpoena covering EU-resident Growth-tier tenant (350 users, 80% EU) served at 22:47 UTC Friday with 14-day deadline; 14 discussion points covering Art. 48 analysis, EU DPA notification, scope minimization, tenant IT admin communication, coaching_turns production gate, partial production during pending challenge, DEC-030 chain verification, media inquiry response, and PIR action item derivation. Appendix A updated with R-13 quick reference.*

*v0.6 additions: R-12 Insider Threat / Privileged Access Abuse — twelfth runbook; covers current and former team members or contractors with legitimate credentials misusing access for exfiltration, sabotage, or personal gain. Evidence-before-containment constraint reinforced (overrides standard flow; exception only for active live exfiltration); private restricted investigation channel (`#inc-YYYYMMDD-insider`: founder + security-engineer + compliance-officer only — not HR, not subject). Trigger matrix: 9 signal types from bulk Supabase Studio access to HMAC chain break co-incident with staff activity; severity table: P0 for Art. 9 data read or multi-tenant cross-access, P1 for bulk access without confirmed exfiltration or post-HR-process access, P2 for historical single anomalous event. Unique response constraints: graduated containment options C-1 (passive monitoring) through C-5 (Workers Secrets rotation) ordered by investigative stealth; legal counsel notification before HR briefing rule with jurisdiction notes (EU/GDPR proportionality, US at-will, Ukraine labour law); HMAC chain R2 archive as forensic truth (live `audit_log_events` table not used forensically). Scope assessment SQL: blast radius by table with Art. 9 classification, multi-tenant exposure query, enterprise tenant identification, `health_profiles` record count for impacted users. Legal and HR interface timeline with legal hold notice template. GDPR implications table: 8 data category rows mapping to Art. 33/34 requirements; 72h clock start note; data subject rights derogation for ongoing investigation (Art. 23(1)(f)); HR investigation records retention under Art. 9(2)(b). Evidence package: 11-item directory structure with SHA-256 manifest, R2 object versioning, 7-year retention per DEC-030. Post-incident preventive controls: 7 controls covering least-privilege audit, offboarding checklist, bulk-read alert rule, Workers Secrets access review, quarterly access review cadence, HMAC chain integrity alert on staff actor + audit log resource, AUP acknowledgment. SOC 2 mapping: CC6.2/CC6.3 (access lifecycle), CC7.2/CC7.3/CC7.4 (monitoring and response), CC9.1 (risk mitigation), CC1.2 (integrity and ethical values). Tabletop Scenario G added to §9 drill catalog: insider data exfiltration before resignation affecting 14 enterprise tenants; 14 discussion points covering forensic chain, legal timing, tenant notification, Art. 33 clock, graduated response. §14 Continuous Improvement Program — closes the PIR-to-action-item loop for SOC 2 CC4.1/CC4.2 evidence. PIR Action Item Registry: Linear project `[IR] Action Items` with 10-field schema (incident_id, severity, priority, title, owner, due_date, SOC2 criterion, status, close_date, verification_note); closure SLA table (Critical P0: 14 days → founder paged; Critical P1: 30 days; High: 30–60 days; Medium: 90 days; Low: 180 days); monthly Linear export to `compliance/evidence/ir-action-items/YYYY-MM.csv`. Escalation thresholds: 4 triggers (overdue Critical, recurring failure pattern, > 3 High items open, SOC 2 observation period breach). Quarterly control effectiveness review: 60-min agenda (action item registry / incident trend analysis / runbook gap check / SOC 2 evidence completeness / decisions); MTTD/MTTR/false-positive-rate metrics; output template stored at `compliance/evidence/quarterly-ir-review/YYYY-QN.md`. Runbook update protocol: 6 mandatory triggers with SLAs; 5-step update procedure with dual-approval gate (compliance-officer + security-engineer); tagged archive at each minor version bump. Annual control self-assessment: Q4 cadence; structured against CC4.1, CC4.2, CC7.2, CC7.4, CC7.5 criteria; self-rated readiness with gap list; stored `compliance/evidence/annual-csa/YYYY-ir-csa.md`; shared with audit firm at observation period open. SOC 2 evidence mapping for §14: CC4.1, CC4.2, CC7.5, CC2.1; complete evidence package list. Appendix A updated with R-12 and §14 quick references.*

*v0.8 additions: R-14 DSAR / Data Subject Rights Incident — fourteenth runbook; operationalises GDPR Arts. 15, 17, 18, 19, 20 rights enforcement into an incident-level response procedure, closing the gap flagged in SOC2_READINESS.md (🔴 DSAR SLA, 🟡 DSAR process) and referenced in GDPR_DPIA.md §10.1.5. Seven-scenario severity matrix: P0 for cross-tenant contamination in export (simultaneous R-01 activation) and for Art. 9 health data confirmed present after erasure deadline; P1 for missed 30-day deadline or export worker complete failure; P2 for single DSAR in cure window or bulk DSAR with no immediate deadline risk. R-14.1 scope assessment SQL: in-flight DSAR register query with overdue flag, export delivery check, erasure completeness query per-table (6 health data tables). R-14.2 cross-tenant contamination protocol: immediate export link revocation (DEC-030 CRITICAL), parallel R-01 activation, Art. 33 clock start on Art. 9 data, enterprise tenant notification within 30 min via Template E-01 (§12), corrected re-delivery with dsar.export_redelivered event. R-14.3 failed erasure protocol: last-good erasure job lookup via audit_log_events, pg_cron log diagnosis, targeted per-table manual erasure with IC + compliance-officer authorization gate, Art. 9 access assessment for Art. 33 trigger, cv_sessions.keypoints_enc → R-11 cross-reference. R-14.4 missed deadline: cure-window (day 28–30) and post-deadline protocols; Art. 12(3) 90-day extension handling; DPA self-report decision framework with compliance-officer authority (no founder sign-off required for non-breach DSAR delays). R-14.5 bulk DSAR/coordinated abuse: Art. 12(5) refusal strategy, deadline escalation threshold, social-media campaign awareness. R-14.6 enterprise employee DSAR: joint-controller complexity where employer is controller for workplace fitness data; privacy floor enforcement — FORM cannot comply with an employer instruction that extinguishes an employee's Art. 15 right. Five communication templates: D-01 DPA voluntary notification (missed deadline), D-02 user delay notification, D-03 Art. 12(5) refusal, D-04 employee redirect to employer, E-DSAR-01 enterprise tenant notification. 14 DEC-030 HMAC-chained audit events covering full DSAR lifecycle from `dsar.request_received` (MEDIUM) through `dsar.export_link_revoked` (CRITICAL) and `dsar.erasure_manual_supplement` (CRITICAL). SOC 2 evidence mapping: P4.0 (data subject inquiries), P5.0 (data subject requests), P5.1 (consent/choice), P8.0 (disposal), CC2.2 (external communications), CC6.5 (logical access — disposal). Evidence package structure: `compliance/evidence/dsar/YYYY-MM/<dsar_id>/` with 6 subdirectories (request, verification, export manifest, erasure, comms, dec030); 7-year retention per DEC-030; read-only after case closure. Six post-incident preventive controls (day-25/29 Slack deadline alerts, monthly export canary, automated post-erasure verification query, cross-tenant isolation regression test, weekly DSAR register review, keypoints_enc NULL check). Tabletop Scenario I: DSAR export cross-tenant contamination affecting 420-user Growth-tier enterprise tenant (UK/Ireland); bug filters by tenant_id instead of user_id; 3 employees received full-tenant exports; all downloaded; Art. 33/34 assessment; ICO notification; evidence chain; re-delivery protocol; 10 discussion points. Document header corrected from v0.5 → v0.8 (intermediate versions v0.6 and v0.7 were committed out of order; header had not been updated past v0.5). Owner line corrected: devops-lead → compliance-officer (matching footer since v0.5). Appendix A updated with R-14 quick reference. SOC 2 scope line in document header updated to include P4.0, P5.0, P8.0.*

---

### R-18: Database Integrity & Neon Postgres Failover Incident

> **Scope:** Covers all degradation and failure modes originating in FORM's primary data store — Neon Postgres (multi-tenant, RLS-enforced). Scenarios include: Neon-side failover/replica promotion, detected data corruption in any table, RLS policy failure or bypass (cross-tenant data bleed), pg_cron job failures affecting HMAC chain integrity or data lifecycle, and point-in-time recovery (PITR) activation. A database incident simultaneously threatens FORM's most sensitive data surfaces: `health_profiles`, `coaching_turns`, `cv_sessions`, `dsar_requests`, and the DEC-030 HMAC audit chain. This runbook operates in parallel with R-01 (Data Breach) if corruption confirms unauthorised data access; with R-05 (HMAC Chain Break) if `audit_log_events` integrity is affected; and with R-11 (CV Privacy) if `cv_sessions.keypoints_enc` is in scope.

#### Trigger

| Source | Alert ID | Condition |
|---|---|---|
| Neon Control Plane | `FORM-DB-FAILOVER-001` | Neon Postgres primary → replica failover event; email to devops-lead + `#db-alerts` Slack via Neon webhook |
| `audit-chain-daily-check` Edge Function | `FORM-DB-CRON-001` | `hmac_valid = FALSE` returned for any row in `audit_log_events` |
| Sentry | `FORM-DB-ERROR-001` | `PostgresError` code `42501` (RLS policy violation) for `anon` or `authenticated` role at rate > 0.1% of requests in any 5-min window |
| Sentry | `FORM-DB-ERROR-002` | `PostgresError` code `P0001` or `23505` on `audit_log_events.chain_seq` — chain sequence collision indicating lost or duplicated event |
| Cloudflare Dead-Man's Switch | `FORM-DB-CHAIN-001` | `audit_log_events` row count for `created_at > NOW() - INTERVAL '1 hour'` returns 0 during a known-active platform period; fires if HMAC chain silently paused |
| Manual Detection | — | devops-lead observes Neon Console: write latency P95 > 500 ms, PgBouncer `cl_waiting` > 0 for > 60 s, or enterprise customer-success receives report of wrong-user data visible in UI |

**All database integrity triggers are treated as potential P0 until §R-18.2 scope assessment confirms otherwise.**

#### Severity Classification

| Condition | Severity | Reason |
|---|---|---|
| RLS bypass confirmed: `authenticated` role query returning rows from another `tenant_id` or `user_id` | **P0** | Multi-tenant isolation failure; cross-tenant Art. 9 health/coaching data exposure; immediate R-01 co-activation required |
| `audit_log_events` corruption or HMAC chain gap detected | **P0** | Chain break destroys SOC 2 audit evidence integrity; DEC-030 is the forensic foundation for all other runbooks |
| `health_profiles`, `coaching_turns`, or `cv_sessions.keypoints_enc` corrupted or partially deleted | **P0** | Art. 9 health data destroyed or exposed; GDPR data integrity obligation breach; enterprise SLA breach likely |
| Neon primary failover + write operations failing > 2 minutes | **P1** | Full write-path outage; coordinate with R-02; PITR may be needed if failover completes with inconsistent state |
| pg_cron job failure halting DSAR erasure or `cv_session_fleet_stats` refresh | **P1** | DSAR erasure failure → Art. 17 obligation at risk; coordinate with R-14 for any in-window DSAR requests |
| Connection pool exhaustion — no new connections accepted > 3 minutes | **P1** | Product non-functional; coordinate with R-02; root cause may be runaway query or ORM N+1 bug |
| Neon failover completed cleanly; writes resumed < 2 min; no data loss on check C1 | **P2** | Normal Neon HA operation; confirm integrity; monitor 30 min; no R-01 activation needed |
| Non-critical table corruption (`user_preferences`, `notification_settings`) | **P2** | No Art. 9 data; restore from PITR or pg_dump; notify affected users if data loss > 24 h |
| Isolated pg_cron schedule drift < 5 min with no missed job window | **P3** | Monitor; root-cause analysis next business day |

**P0 upgrade trigger:** If scope assessment (§R-18.2 Check C2) returns `cross_tenant_rows > 0`, or `cv_sessions.keypoints_enc` is readable by an incorrect `user_id`, immediately upgrade to P0 and activate R-01 (and R-11 if keypoints_enc confirmed in scope) in parallel.

#### Why This Matters

FORM's database is the product. Every workout, coaching turn, biometric estimate, and audit event exists only in Neon Postgres. Unlike application-tier failures mitigated with edge caching, a database integrity incident can simultaneously trigger: (1) user data loss, (2) cross-tenant health data exposure activating GDPR Art. 9 notification obligations, (3) destruction of the DEC-030 HMAC chain that every other runbook depends on for forensic evidence, and (4) SOC 2 audit evidence gaps that could affect the Type II opinion.

The multi-tenant RLS architecture means a single missing `tenant_id = auth.jwt()->>'tenant_id'` filter on a new table creates silent cross-tenant data bleed detectable only by scope assessment queries. FORM currently has no real-time RLS regression test in CI for every table (tracked as OQ-DB-01 below).

Neon's architectural guarantee is a 500 ms failover SLA for primary promotion. However, the window between primary failure and replica promotion can result in in-flight `INSERT`/`UPDATE` transactions being lost, and the first 30–60 seconds post-failover may return write errors while PgBouncer reconnects. This runbook covers both the clean-failover and dirty-failover (with confirmed data loss) paths.

#### Immediate Actions (T+0 to T+15 min)

```
1. Open incident channel: #inc-YYYYMMDD-db-integrity
   IC: devops-lead (or on-call platform-engineer if devops-lead unreachable)
   Notify: platform-engineer + security-engineer immediately

2. Classify trigger type — pick exactly one:
   A) Neon failover event (FORM-DB-FAILOVER-001)
      → Follow Immediate Path A below — confirm write path, run Check C1
   B) HMAC chain break (FORM-DB-CRON-001 or FORM-DB-CHAIN-001)
      → Activate R-05 IMMEDIATELY in parallel; this runbook handles DB-side forensics
   C) RLS violation or cross-tenant bleed (FORM-DB-ERROR-001 or UI report)
      → Activate R-01 in parallel; treat as P0 until C2 scan disproves
   D) pg_cron failure (non-chain job)
      → Identify which job failed; if DSAR erasure: coordinate with R-14; assess deadline risk

3. Confirm connectivity and failover state (run as form_readonly):
```

```sql
SELECT
  pg_postmaster_start_time() AS pg_started_at,
  pg_is_in_recovery()        AS is_replica,   -- TRUE = replica (failover still in progress)
  current_database()         AS db_name,
  NOW()                      AS query_time;
```

```sql
-- Confirm write path health post-failover
SELECT
  relname AS table_name,
  n_tup_ins AS inserts,
  n_tup_upd AS updates,
  last_autoanalyze
FROM pg_stat_user_tables
WHERE relname IN (
  'audit_log_events','coaching_turns','health_profiles',
  'cv_sessions','workout_sessions','dsar_requests'
)
ORDER BY relname;
```

```
4. Post in #inc-YYYYMMDD-db-integrity within T+10 min:
   - pg_is_in_recovery() result
   - Write errors in Sentry (last 10 min)
   - Which trigger fired and at what timestamp
   - IC identity confirmed
```

**Immediate Path A (Neon Failover):**
```
A1. Check Neon Console → Project → Operations for failover event details:
    - Start time, end time, duration
    - Planned (Neon maintenance) vs. unplanned (primary death)
    - Whether in-flight transactions were rolled back

A2. Check for PgBouncer reconnect errors in Sentry:
    - `connection refused` or `server closed the connection unexpectedly`
    - Any pg_is_in_recovery() = FALSE but writes still failing?

A3. Run Check C1 (row count consistency) within T+5 min of trigger

A4. Classify clean vs. dirty failover:
    - Clean: writes accepted, C1 consistent → P2, monitor 30 min, no R-01
    - Dirty: write errors > 30 s post-failover OR C1 inconsistency → escalate to P1
    - Very dirty: confirmed data loss in Art. 9 tables → P0, activate R-01
```

#### Scope Assessment SQL (§R-18.2)

> Run all queries as `form_admin` with `row_security = off`. Output restricted to `#inc-YYYYMMDD-db-integrity` only — never share in general engineering channels.

**Check C1: Row count consistency** (run post-failover to detect in-flight transaction loss)
```sql
-- Compares current row counts against the last pg_cron hourly snapshot
-- table_row_count_snapshots is populated by OBSERVABILITY §7 dead-man's switch job
-- If that table does not yet exist, compare manually against STATUS.md last-known counts
SELECT
  s.table_name,
  s.row_count                                          AS snapshot_count,
  s.snapshot_time,
  ROUND(EXTRACT(EPOCH FROM (NOW() - s.snapshot_time)) / 60) AS minutes_since_snapshot,
  c.live_count                                         AS current_count,
  (c.live_count - s.row_count)                         AS delta,
  CASE
    WHEN ABS(c.live_count - s.row_count) > 100
      AND s.table_name IN ('health_profiles','coaching_turns','cv_sessions','audit_log_events')
    THEN '⚠️ INVESTIGATE'
    ELSE '✅ OK'
  END AS status
FROM (
  SELECT table_name, row_count, snapshot_time
  FROM table_row_count_snapshots
  WHERE snapshot_time = (SELECT MAX(snapshot_time) FROM table_row_count_snapshots)
) s
JOIN (
  SELECT 'health_profiles' AS table_name, COUNT(*) AS live_count FROM health_profiles
  UNION ALL SELECT 'coaching_turns',  COUNT(*) FROM coaching_turns
  UNION ALL SELECT 'cv_sessions',     COUNT(*) FROM cv_sessions
  UNION ALL SELECT 'audit_log_events',COUNT(*) FROM audit_log_events
  UNION ALL SELECT 'workout_sessions',COUNT(*) FROM workout_sessions
  UNION ALL SELECT 'dsar_requests',   COUNT(*) FROM dsar_requests
) c ON s.table_name = c.table_name
ORDER BY s.table_name;
-- Expected: delta within ±100 for all tables. Larger negative delta = possible data loss.
```

**Check C2: RLS cross-tenant bleed detection** (run immediately on FORM-DB-ERROR-001 or any suspicion)
```sql
SET LOCAL role = 'form_admin';
SET LOCAL row_security = off;

WITH user_tenants AS (
  SELECT id AS user_id, tenant_id AS assigned_tenant_id FROM users
)
SELECT 'coaching_turns' AS table_name, ct.id AS row_id,
       ct.user_id, ct.tenant_id AS row_tenant, ut.assigned_tenant_id, 'MISMATCH' AS status
FROM coaching_turns ct
JOIN user_tenants ut ON ct.user_id = ut.user_id
WHERE ct.tenant_id <> ut.assigned_tenant_id

UNION ALL

SELECT 'health_profiles', hp.id, hp.user_id,
       hp.tenant_id, ut.assigned_tenant_id, 'MISMATCH'
FROM health_profiles hp
JOIN user_tenants ut ON hp.user_id = ut.user_id
WHERE hp.tenant_id <> ut.assigned_tenant_id

UNION ALL

SELECT 'cv_sessions', cs.id, cs.user_id,
       cs.tenant_id, ut.assigned_tenant_id, 'MISMATCH'
FROM cv_sessions cs
JOIN user_tenants ut ON cs.user_id = ut.user_id
WHERE cs.tenant_id <> ut.assigned_tenant_id;

-- Expected: 0 rows.
-- ANY rows returned = P0. Activate R-01 immediately. Start Art. 33 clock.
```

**Check C3: HMAC chain continuity** (run if FORM-DB-CHAIN-001 fired or as post-failover validation)
```sql
SELECT
  chain_seq,
  LAG(chain_seq) OVER (ORDER BY chain_seq)                           AS prev_seq,
  chain_seq - LAG(chain_seq) OVER (ORDER BY chain_seq)               AS gap,
  created_at,
  event_type
FROM audit_log_events
WHERE created_at >= NOW() - INTERVAL '2 hours'
HAVING chain_seq - LAG(chain_seq) OVER (ORDER BY chain_seq) > 1
ORDER BY chain_seq;
-- Expected: 0 rows. Any gap = activate R-05 in parallel; document gap range and timestamp.
```

**Check C4: pg_cron job health** (run if FORM-DB-CRON-001 fired)
```sql
SELECT
  j.jobname,
  j.schedule,
  d.status,
  d.start_time  AS last_run,
  d.return_message
FROM cron.job j
JOIN LATERAL (
  SELECT status, start_time, return_message
  FROM cron.job_run_details d2
  WHERE d2.jobid = j.jobid
  ORDER BY start_time DESC
  LIMIT 1
) d ON TRUE
ORDER BY j.jobname;
-- Failed jobs show status = 'failed'. Identify which job(s) and assess DSAR/chain impact.
```

#### Containment

**Failover path:**
```
FA-1. If pg_is_in_recovery() = TRUE, do NOT attempt manual failover intervention.
      Neon manages promotion automatically. Monitor Neon Console Operations tab.

FA-2. If writes are failing > 5 min post-promotion:
      → Check Workers connection strings (Neon HTTP endpoint vs. pool endpoint)
      → Trigger Cloudflare Workers redeploy if connection strings are stale: wrangler deploy
      → Do NOT restart the Neon project unless Neon support advises

FA-3. Once writes are confirmed healthy, pause any active pg_cron DSAR erasure jobs:
      → Prevents partial erasure commits during an unstable write period
      → Resume only after C1 passes
```

**RLS bypass / cross-tenant bleed (P0):**
```
RLS-1. Set affected tenant(s) to read-only via feature flag:
       UPDATE tenants SET read_only_mode = true WHERE id IN (<affected_tenant_ids>);
       → customer-success notified immediately per §12 Template E-01

RLS-2. Do NOT delete or modify mismatched rows — preserve as forensic evidence.
       Snapshot the C2 result:
         COPY (SELECT * FROM ...<C2 query>...) TO STDOUT CSV HEADER
       Upload to R2: s3://form-soc2-evidence/incidents/<slug>/rls-mismatch-<timestamp>.csv

RLS-3. Identify the code path that produced the mismatch:
       → Git blame the migration that created/modified the affected table's RLS policy
       → Check for a recent migration that removed or weakened USING/WITH CHECK clauses
       → Check for an ORM model that inadvertently calls BYPASSRLS

RLS-4. R-01 is now active and owns the Art. 33 clock. Feed it:
       → Art. 9 data category: health_profiles (YES), coaching_turns (YES), cv_sessions (YES if keypoints_enc populated)
       → Blast radius: unique user_ids and tenant_ids from C2 result
       → Earliest affected timestamp: MIN(created_at) from mismatched rows
```

**HMAC chain break:**
```
Chain-1. R-05 is now the primary runbook. This runbook handles DB-side recovery.
Chain-2. Restrict audit_log_events to service-role writes only:
         REVOKE INSERT ON audit_log_events FROM authenticated, anon;
         GRANT INSERT ON audit_log_events TO form_audit_writer;
         → Prevents further chain extension that could obscure the break point
Chain-3. Dump chain state to R2 immediately:
         pg_dump --table=audit_log_events --data-only --format=custom \
           $DATABASE_URL > /tmp/audit_log_snapshot_$(date +%Y%m%dT%H%M%S).dump
         aws s3 cp /tmp/audit_log_snapshot_*.dump \
           s3://form-soc2-evidence/incidents/<slug>/audit_log_snapshot.dump
```

#### Eradication: PITR Recovery Procedure

> Activate only if P0 data corruption is confirmed. Requires three-party approval.

```
PITR-1. Identify the target recovery point:
        → Last known-good timestamp BEFORE the corruption event
        → Use C3 chain_seq gap to pinpoint the event window
        → Cross-reference Neon Console Operations log for failover timestamp

PITR-2. Create a PITR branch in Neon Console:
        → Neon Console → Branches → "Create branch" → set timestamp to recovery point
        → Branch name: inc-<slug>-recovery-<timestamp>
        → DO NOT set as primary yet

PITR-3. Validate the recovery branch:
        → Connect via separate connection string
        → Run C1, C2, and C3 against recovery branch
        → Confirm corruption is absent; confirm chain is continuous to recovery point

PITR-4. Quantify data loss:
        → Compare recovery branch row counts vs. production branch for each table
        → For each table with negative delta: classify whether rows are user-generated
          (unrecoverable from server) or system-generated (reconstructible)

PITR-5. THREE-PARTY APPROVAL GATE — founder + devops-lead + security-engineer:
        → Document approval in #inc-YYYYMMDD-db-integrity with timestamps
        → Emit DEC-030 CRITICAL event: database.pitr_activation_authorized
        → If any enterprise tenants in scope: activate §12 Template E-01 BEFORE switching
        → Two-party fallback: if third approver unreachable after 15 min, two-party approval
          permitted for clean PITR (no confirmed data loss) only; dirty PITR always requires three.

PITR-6. Switch primary to recovery branch (Neon Console):
        → "Set as primary" on recovery branch
        → Update Workers DATABASE_URL environment variable → wrangler deploy
        → Monitor write-path health for 30 min; watch Sentry for PostgresError spike

PITR-7. Post-PITR reconciliation:
        → coaching_turns lost: users must re-trigger Victor for affected session dates
        → workout_sessions lost: import from wearable sync if available
        → health_profiles lost: user re-onboarding may be required; notify CSM for enterprise
        → Emit DEC-030 event: database.pitr_recovery_completed
```

#### Recovery

```
R1. Validate all six Art. 9 tables intact and RLS policies functioning:
    → Rerun C2 (RLS mismatch): expect 0 rows
    → Rerun C1 (row count): document final counts vs. pre-incident snapshot

R2. Resume pg_cron jobs in order:
    1. audit-chain-daily-check (validate chain first)
    2. cv_session_fleet_stats refresh
    3. dsar_erasure_job (check for any DSAR requests that missed their deadline during incident)

R3. Restore write access:
    → GRANT INSERT ON audit_log_events TO authenticated, anon; (if revoked)
    → UPDATE tenants SET read_only_mode = false WHERE id IN (<affected>);
    → customer-success sends §12 Template E-03 (Resolution) to affected tenants

R4. All-clear post to #ops-general and #inc-YYYYMMDD-db-integrity:
    → Total incident duration, data loss assessment (none / quantified), tenants affected
    → IC confirms PIR scheduled within 5 business days
    → Emit DEC-030: incident.recovered
```

#### Regulatory Assessment

| Scenario | Regulation | Obligation | Clock Start |
|---|---|---|---|
| C2 mismatch confirmed in `health_profiles`, `coaching_turns`, or `cv_sessions` (Art. 9 data) | GDPR Art. 33 + Art. 9 | Supervisory authority notification within 72 h | Time C2 confirmation query ran |
| Data corruption destroying rows in `health_profiles` or `coaching_turns` | GDPR Art. 5(1)(f) integrity | Art. 33 if health data of > 0 users destroyed; availability breach assessment | IC P0 declaration time |
| PITR activated with confirmed data loss | GDPR Art. 32 | PIR must document why PITR was the appropriate technical measure; DPA notification if large-scale health data lost | PITR activation time |
| HMAC chain break in `audit_log_events` during SOC 2 observation period | SOC 2 CC7.4 | Audit evidence gap; auditor notification required | Chain break confirmed timestamp |

**Art. 33 co-trigger:** If C2 returns rows in Art. 9 tables, R-01 owns the Art. 33 clock. This runbook feeds the Art. 9 data category classification and blast radius numbers into R-01's breach notification template.

#### DEC-030 Audit Events

| Event Type | Severity | Trigger | Retention |
|---|---|---|---|
| `database.failover_detected` | HIGH | Neon primary failover webhook received; `pg_is_in_recovery()` confirmed | 3 years |
| `database.write_path_restored` | STANDARD | Post-failover writes confirmed healthy; latency P95 < 100 ms | 1 year |
| `database.rls_mismatch_detected` | CRITICAL | C2 returns > 0 rows; cross-tenant data confirmed | 7 years |
| `database.pitr_activation_authorized` | CRITICAL | Three-party approval documented; PITR branch created | 7 years |
| `database.pitr_recovery_completed` | HIGH | Primary switch to recovery branch confirmed; C1 + C2 clean | 3 years |
| `database.hmac_chain_gap_detected` | CRITICAL | C3 returns > 0 rows; chain_seq discontinuity confirmed | 7 years |
| `database.pg_cron_job_failure` | HIGH | Any pg_cron job exits with error in `cron.job_run_details` | 3 years |
| `database.rls_policy_corrected` | HIGH | Migration fix deployed; C2 reruns clean post-fix | 3 years |
| `database.read_only_mode_activated` | HIGH | Tenant(s) set to `read_only_mode = true` pending investigation | 3 years |
| `database.read_only_mode_lifted` | STANDARD | Tenant(s) restored to read-write post-recovery | 1 year |

```typescript
// DEC-030 emission for RLS mismatch (emitted by scope-assessment Worker endpoint)
const event = {
  event_type: 'database.rls_mismatch_detected',
  severity: 'CRITICAL',
  incident_id: incidentId,
  payload: {
    mismatch_row_count: result.rowCount,
    affected_tables: result.tables,              // ['health_profiles', 'coaching_turns']
    query_executed_at: new Date().toISOString(),
    tenants_affected_count: result.tenantIds.length, // count only — no identifiers
    art9_tables_in_scope: result.art9TablesInScope,  // true = R-01 auto-activation
  },
};
await emitDec030Event(event, supabaseAdminClient);
```

#### Evidence Package

| Evidence ID | Artefact | Collection Method | Location |
|---|---|---|---|
| **IR-DB-E-001** | Neon Console failover event log | Screenshot: Neon Console → Operations tab showing failover timestamp, duration, promotion type | `compliance/evidence/incidents/<slug>/neon-failover-log.png` |
| **IR-DB-E-002** | C1 row count report (pre- and post-incident) | SQL query output from §R-18.2 Check C1 exported as JSON at T+0 and post-recovery | `compliance/evidence/incidents/<slug>/row-count-pre.json` + `row-count-post.json` |
| **IR-DB-E-003** | C2 RLS mismatch scan output | Full SQL result (0 rows = clean; > 0 rows = breach evidence) as JSON | `compliance/evidence/incidents/<slug>/rls-mismatch-scan.json` |
| **IR-DB-E-004** | C3 HMAC chain continuity check | SQL result showing chain_seq gaps (0 rows expected); R-05 chain verification report if activated | `compliance/evidence/incidents/<slug>/hmac-chain-check.json` |
| **IR-DB-E-005** | DEC-030 `database.*` event extract | All `database.*` events from `audit_log_events` for incident window, HMAC-verified | `compliance/evidence/incidents/<slug>/dec030-database-events.json` |
| **IR-DB-E-006** | pg_cron job health report (C4) | SQL output of `cron.job_run_details` for 24-hour window around incident | `compliance/evidence/incidents/<slug>/pgcron-health.json` |
| **IR-DB-E-007** | PITR activation approval record (if activated) | Three-party approval from incident channel + `database.pitr_activation_authorized` event | `compliance/evidence/incidents/<slug>/pitr-approval.md` |
| **IR-DB-E-008** | Post-recovery C1 + C2 clean run | Confirmation reruns of Checks C1 and C2; both expected clean | `compliance/evidence/incidents/<slug>/post-recovery-checks.json` |

All evidence under `compliance/evidence/incidents/<slug>/` with SHA-256 manifest; 7-year retention per DEC-030 CRITICAL events; R2 object versioning enabled.

#### SOC 2 TSC Mapping

| TSC Criterion | Control | Evidence from This Runbook |
|---|---|---|
| CC7.2 — Anomaly monitoring | FORM-DB-ERROR-001/002 RLS alert, FORM-DB-CHAIN-001 dead-man's switch, pg_cron failure alert | Alert configurations + C2 automated daily scan as continuous monitoring evidence |
| CC7.3 — Evaluate and communicate security events | Scope assessment C1–C4, severity classification matrix, IC role definition | IR-DB-E-001 through IR-DB-E-008 evidence package; documented query framework |
| CC7.4 — Respond to identified security events | Containment procedures, PITR recovery, RLS fix, DEC-030 emission | PITR procedure + three-party approval gate as documented response authority |
| A1.2 — Environmental threats | Neon failover handling, PITR activation, pg_cron resilience | IR-DB-E-001 (failover log) + IR-DB-E-007 (PITR approval): demonstrates recovery from HA events |
| A1.3 — Recovery objectives | PITR procedure with target recovery point identification; < 2 min RTO for clean failover | PITR steps 1–7 as documented recovery procedure |
| CC6.1 — Logical access controls | BYPASSRLS restricted to `form_admin`; RLS policy framework across all tables | C2 mismatch scan as ongoing control effectiveness check; RLS correction procedure |
| CC6.5 — Logical access — disposal | PITR branch cleanup post-recovery; C2 mismatch row disposition without modification | IR-DB-E-003 + IR-DB-E-008: before/after evidence of RLS correction |

#### Open Questions

**OQ-DB-01: RLS regression testing in CI**
No automated test currently validates that every user-facing table has a correct `tenant_id = auth.jwt()->>'tenant_id'` USING clause. A migration that adds a table without an RLS policy passes CI today. Options: (a) `pgTAP` test suite in the CI migration step asserting `RLS IS ENABLED` and `POLICY EXISTS` for every `public` schema table; (b) custom `check-rls.ts` script querying `pg_policies` via Supabase admin client in a migration smoke test. Option (a) is preferred (idiomatic pg testing). Owner: platform-engineer. Priority: **P1**. Resolution target: M5 (before first enterprise onboarding).

**OQ-DB-02: PITR three-party approval out-of-band mechanism**
Current approval relies on a Slack channel. If Slack is unavailable during the incident, there is no out-of-band path. Recommended resolution: PagerDuty incident command acknowledgement from three named responders constitutes formal approval; document as the out-of-band method. Two-party fallback for clean PITR (no confirmed data loss); three-party remains required for dirty PITR. Owner: security-engineer + devops-lead. Priority: **P2**. Resolution target: M6.

**OQ-DB-03: `table_row_count_snapshots` table existence**
Check C1 assumes a `table_row_count_snapshots` table from the OBSERVABILITY §7 dead-man's switch pg_cron job. Until it exists, Check C1 must be run manually against STATUS.md last-known counts. Owner: devops-lead. Priority: **P0** (blocking C1). Resolution target: M4.

#### Implementation Checklist

| # | Action | Priority | Milestone | Owner | Status |
|---|---|---|---|---|---|
| 1 | Create `table_row_count_snapshots` table + pg_cron job (every 30 min; row counts for 6 Art. 9 tables); confirm first snapshot within 30 min of deploy; closes OQ-DB-03 | P0 | M4 | devops-lead | [ ] |
| 2 | Configure Neon Console webhook → `#db-alerts` Slack; confirm test delivery; register as `FORM-DB-FAILOVER-001` in OBSERVABILITY.md §3 alert registry | P0 | M4 | devops-lead | [ ] |
| 3 | Implement `FORM-DB-ERROR-001` Sentry alert (42501 > 0.1% in 5-min window) and `FORM-DB-ERROR-002` (23505 on `audit_log_events.chain_seq`); wire to PagerDuty + `#db-alerts` | P0 | M4 | devops-lead + platform-engineer | [ ] |
| 4 | Implement `FORM-DB-CHAIN-001` dead-man's switch: Cloudflare Edge Function (cron, every 15 min) checks `audit_log_events` row count for last 1 h; fires PagerDuty P1 to devops-lead if 0 during active platform hours | P0 | M4 | devops-lead | [ ] |
| 5 | Add pgTAP test suite to CI migration step: assert RLS enabled + USING policy exists for every `public` table; fail CI if any table missing; closes OQ-DB-01 | P1 | M5 | platform-engineer | [ ] |
| 6 | Document OQ-DB-02 PITR approval protocol in §R-18 PITR step 5: add two-party fallback rule for clean PITR; register PagerDuty IC acknowledgement as out-of-band approval path; closes OQ-DB-02 | P2 | M6 | security-engineer | [ ] |
| 7 | Register all ten `database.*` DEC-030 events in `docs/AUDIT_LOG_SCHEMA.md` with schema, severity, and retention fields | P1 | M4 | security-engineer | [ ] |
| 8 | Add automated daily C2 (RLS mismatch) scan: pg_cron job at 02:00 UTC; if result > 0, emit `database.rls_mismatch_detected` + PagerDuty P0; results to `compliance/evidence/rls-scans/YYYY-MM-DD.json` | P1 | M5 | platform-engineer | [ ] |
| 9 | Confirm Neon project PITR retention window ≥ 7 days; document setting in OBSERVABILITY.md §3 database SLO entry | P1 | M4 | devops-lead | [ ] |
| 10 | Add R-18 tabletop scenario to §9.5 Year 2 Testing Schedule: Neon dirty failover simulation + cross-tenant RLS mismatch discovery; enterprise CSM participates; record elapsed time vs. 15-min P0 SLA | P2 | M6 | security-engineer | [ ] |

---

### R-19: Enterprise IdP Outage & Break-Glass Authentication

> **Scope:** This runbook covers the scenario where an enterprise tenant's identity provider (IdP) — Okta, Microsoft Azure AD / Entra ID, Google Workspace, Ping Identity, or any SAML/OIDC IdP configured in WorkOS — becomes unavailable or misconfigured, preventing enterprise users from authenticating into FORM via SSO. **This is an availability incident, not a security breach.** For SAML certificate compromise or a SAML authentication attack, activate R-04 instead. For FORM's own SSO infrastructure being down, activate R-02. References: `docs/SSO_SCIM_IMPLEMENTATION.md` §§1–2 (SAML/OIDC flows), `docs/OBSERVABILITY.md` §26 (SSO identity observability and AL-SSO-* alert rules), §12 (Enterprise Tenant SLA Breach & Communication Protocol), §23 (SLA credit calculation).

#### Trigger

Any of the following conditions activates this runbook:

1. **Alert `AL-SSO-01`** fires from OBSERVABILITY §26: per-tenant SSO login success rate drops below 90% in a rolling 5-minute window, sustained for ≥ 3 consecutive evaluation cycles.
2. **Alert `AL-SSO-02`** fires from OBSERVABILITY §26: P95 SSO round-trip latency exceeds 10,000 ms for a specific tenant for ≥ 3 consecutive minutes (indicative of IdP responding slowly rather than completely down).
3. **CSM or enterprise tenant IT admin** reports SSO login failures to `#enterprise-support` Slack channel or via the enterprise support email address. Direct report from tenant supersedes alert timing — open the incident immediately.
4. **WorkOS status page** or a known IdP public status page (Okta Trust, Azure Service Health, Google Workspace Status) shows a degradation or outage affecting the tenant's IdP service.

**What this is NOT:**
- FORM's own WorkOS/Auth0 integration being misconfigured → diagnose and fix as a platform bug (R-02 if user-facing)
- SAML certificate expiry on FORM's side → R-04 (§20 of SSO_SCIM handles proactive certificate lifecycle)
- SAML certificate expiry on the IdP's side → this runbook R-19 (customer-side certificate expiry is an IdP availability event)
- A user's individual SSO account being locked or deprovisioned → SCIM deprovisioning flow (SSO_SCIM §§14–15); not an incident

#### Severity Classification

| Condition | Severity | Rationale |
|---|---|---|
| Single tenant IdP partial degradation (< 30% of tenant users affected; SSO still working for most) | **P2** | Monitor; no break-glass needed yet; CSM notifies tenant IT admin |
| Single tenant IdP down > 15 min; > 50% of tenant users unable to log in | **P1** | Enterprise SLA clock starts; IC activates break-glass assessment; CSM E-04 notification |
| Single tenant IdP down > 15 min AND tenant IT admin is also locked out of FORM admin dashboard | **P1 → P0 upgrade** | Break-glass mandatory; two-person auth required immediately |
| Multiple tenants sharing the same IdP provider (e.g., Okta global outage) all affected | **P0** | Fleet-level event; page compliance-officer + founder; all affected tenant CSMs on bridge |
| Tenant's on-premises IdP (ADFS) unreachable and no cloud fallback exists | **P1** | Different remediation path — on-prem ADFS requires tenant's network team; FORM break-glass is the only relief lever |

**P0 upgrade trigger:** If five or more enterprise tenant users are actively locked out (cannot log in, have no cached session), and the outage duration exceeds 30 minutes with no IdP recovery timeline, upgrade to P0 immediately, page compliance-officer and founder, and notify all affected tenant CSMs.

**Note on cached sessions:** Existing authenticated sessions in FORM's enterprise session store (`enterprise_sessions` table, backed by Cloudflare KV) remain valid during an IdP outage — affected users who are already logged in are not disrupted. Only users attempting a new SSO login are blocked. Scope the impact accordingly.

#### Immediate Actions (T+0 to T+15 min)

```
1. Open incident channel: #inc-YYYYMMDD-idp-{tenant_slug} (one channel per affected tenant;
   if multiple tenants → single channel #inc-YYYYMMDD-idp-fleet)
   IC: platform-engineer (on-call)
   Notify: customer-success + devops-lead immediately

2. Confirm the outage source:
   → Check the IdP's public status page for the affected tenant's provider:
     Okta:             trust.okta.com
     Microsoft Entra:  status.azure.com (filter: "Microsoft Entra ID")
     Google Workspace: workspace.google.com/status
     Ping Identity:    trust.pingidentity.com
   → If the IdP status page is green: the issue may be tenant-specific misconfiguration
     (SAML certificate expiry on tenant side, SCIM token rotation, IdP policy change).
     Run scope queries C1 and C2 below immediately.
   → If the IdP status page shows degradation or outage: this is out of FORM's control.
     Skip C2 (not actionable); proceed to Containment.

3. Confirm FORM's SSO infrastructure is healthy (rule out R-02):
   → Check OBSERVABILITY §26 SSO fleet dashboard: is the outage tenant-specific, or is
     FORM's WorkOS/Auth0 integration itself unhealthy?
   → If WorkOS/Auth0 is reporting errors across all tenants → R-02 (service outage), not R-19

4. Run scope assessment queries (as form_admin, BYPASSRLS; output to incident channel only):
```

#### Scope Assessment Queries

```sql
-- C1: Identify affected tenant(s), SSO provider, and recently active user count
-- Run as form_admin (BYPASSRLS). Output restricted to incident channel.
SELECT
  t.slug                AS tenant_slug,
  t.display_name        AS tenant_name,
  t.sso_provider,       -- 'okta' | 'azure_ad' | 'google_workspace' | 'ping_identity' | 'saml_custom'
  t.sso_domain,
  t.admin_email,        -- Primary IT admin contact for break-glass notification
  COUNT(u.id)           AS total_sso_users,
  COUNT(u.id) FILTER (
    WHERE u.last_seen_at > NOW() - INTERVAL '24 hours'
  )                     AS active_users_last_24h,
  COUNT(u.id) FILTER (
    WHERE u.last_seen_at > NOW() - INTERVAL '1 hour'
  )                     AS active_users_last_1h,
  t.sla_tier            -- 'standard' | 'premium' — determines SLA credit threshold
FROM tenants t
JOIN users u ON u.tenant_id = t.id
WHERE t.sso_enabled = true
  AND t.sso_provider = $1  -- pass the suspected provider string, or remove WHERE for all SSO tenants
GROUP BY t.id, t.slug, t.display_name, t.sso_provider, t.sso_domain, t.admin_email, t.sla_tier
ORDER BY active_users_last_1h DESC;
```

```sql
-- C2: Check for active sessions (users already logged in — NOT affected by IdP outage)
-- These users' sessions are insulated; do not count them in the impact blast radius.
SELECT
  t.slug    AS tenant_slug,
  COUNT(es.id) FILTER (WHERE es.expires_at > NOW()) AS active_valid_sessions,
  COUNT(es.id) FILTER (WHERE es.expires_at < NOW()) AS expired_sessions
FROM enterprise_sessions es
JOIN users u  ON u.id  = es.user_id
JOIN tenants t ON t.id = u.tenant_id
WHERE t.sso_enabled = true
  AND t.sso_provider = $1
GROUP BY t.slug
ORDER BY active_valid_sessions DESC;
```

```sql
-- C3: Identify whether break-glass has already been activated for this tenant
-- (guards against duplicate break-glass tokens for the same tenant)
SELECT
  event_type,
  payload->>'tenant_slug'       AS tenant_slug,
  payload->>'token_id'          AS token_id,
  payload->>'break_glass_expires_at' AS expires_at,
  payload->>'authorized_by_ic'  AS authorized_by_ic,
  payload->>'authorized_by_cop' AS authorized_by_cop,
  event_ts
FROM audit_log_events
WHERE event_type IN ('sso.break_glass_token_issued', 'sso.break_glass_token_revoked')
  AND payload->>'tenant_slug' = $1
  AND event_ts > NOW() - INTERVAL '24 hours'
ORDER BY event_ts DESC;
```

```
5. Determine user impact from C1 + C2:
   Impact = (active_users_last_1h) - (active_valid_sessions for same tenant)
   This is the number of users who will be blocked if they attempt to log in now.
   Severity classification uses this number. Report it in the incident channel.

6. Contact tenant IT admin via out-of-band channel:
   → CSM reaches tenant IT admin via the pre-registered emergency contact in the
     enterprise account record (admin_email from C1; supplemented by the Slack Connect
     channel established during onboarding).
   → DO NOT attempt to contact the tenant via SSO-authenticated channels — they may
     not be able to read them.
   → Initial message: Template E-04 initial notification (§19.1 below).
   → Ask tenant IT admin: "Are you able to log in to your IdP admin console?
     Do you have a recovery timeline from your IdP provider?"
```

#### Containment

**Standard containment (IdP global outage — FORM cannot fix):**

When the IdP's own status page confirms an outage, FORM cannot remediate the root cause. The only available relief lever is break-glass authentication for the tenant's IT admin.

**Break-glass authentication — requirements before activation:**

| Requirement | Detail |
|---|---|
| Two-person authorization | IC (on-call platform-engineer) AND compliance-officer must both approve. If compliance-officer is unavailable, founder substitutes. |
| Tenant IT admin request | Break-glass is only issued in response to a direct request from the tenant's registered IT admin (verified via admin_email or Slack Connect). FORM does not pre-emptively issue break-glass tokens. |
| Time-limited scope | Token expires in 4 hours by default. IT admin may request extension up to 8 hours with compliance-officer sign-off. |
| Admin-only scope | Break-glass token grants access to the tenant's admin dashboard only (`/admin/*` routes, `is_admin = true` session flag). It does NOT grant access to individual employee fitness data. Privacy floor: this control is designed for IT operational continuity, not data access. |
| DEC-030 mandatory | `sso.break_glass_token_issued` must be emitted with both authorizers' user_ids before the token is delivered. The token is not valid until this event is confirmed in the audit chain. |
| Revocation commitment | Both authorizers commit to revoking the token within 30 minutes of IdP recovery confirmation. |

**Break-glass issuance procedure:**

```
1. IC confirms two-person auth:
   → IC: "I am authorizing break-glass for tenant {slug}, incident {inc_id}. I am {name}."
   → Compliance-officer: "Confirmed. compliance-officer {name} co-authorizing."
   → Both statements logged in #inc-YYYYMMDD-idp-{slug} before any token is generated.

2. Platform-engineer calls the internal break-glass API endpoint
   (Cloudflare Workers admin endpoint, not externally accessible):

   POST /admin/v1/tenants/{tenant_slug}/break-glass
   Authorization: Bearer {SERVICE_ROLE_KEY}
   Content-Type: application/json

   {
     "incident_id": "inc-YYYYMMDD-idp-{slug}",
     "authorized_by_ic_user_id": "{ic_user_id}",
     "authorized_by_cop_user_id": "{cop_user_id}",
     "expires_in_hours": 4,
     "scope": "admin_only",
     "reason": "IdP outage — {sso_provider} service degradation confirmed on status page"
   }

   Response includes:
   {
     "token_id": "bg-{uuid}",
     "one_time_code": "{8-digit-numeric}",
     "admin_login_url": "https://app.form.coach/enterprise/break-glass?token={signed_jwt}",
     "expires_at": "{ISO8601}"
   }

3. Platform-engineer confirms that sso.break_glass_token_issued DEC-030 event is
   present in audit_log_events before delivering the token.
   → Query: SELECT id, event_ts FROM audit_log_events
             WHERE event_type = 'sso.break_glass_token_issued'
               AND payload->>'token_id' = '{token_id}'
             LIMIT 1;
   → If 0 rows: DO NOT deliver the token. Investigate DEC-030 emission failure.

4. Deliver the break-glass token to the tenant IT admin:
   → ONLY via the pre-registered emergency contact channel (Slack Connect or admin_email).
   → Include: the login URL, the 8-digit one-time code, the 4-hour expiry time.
   → NEVER send via a channel that requires the tenant's SSO to access.

5. Confirm the IT admin has successfully logged in.
   → Verify: check enterprise_sessions for a new session for admin_email within 5 min.
   → If login fails: escalate immediately — investigate token delivery failure.
```

**Break-glass authentication — what the admin can do:**

- View the admin dashboard: seat count, user list (aggregated, no individual health data), usage overview
- Export the user list CSV to identify which employees are affected (name + work email only)
- Send an in-app announcement to all tenant users via the admin broadcast feature
- Contact FORM support directly from the admin dashboard without SSO re-authentication

**Break-glass authentication — privacy floor:**

The admin dashboard served via break-glass is identical to the normal admin session. The privacy floor applies equally:
- HR/admin never sees individual workout logs, coaching turns, health profiles, or biometric data
- Aggregate metrics visible: session counts, active user %, engagement trend — no individual breakdown
- This is enforced at the API layer by `is_admin = true` session flag + RLS policy; break-glass does not expand data access

**Containment for tenant-side SAML certificate expiry (IdP status page: green):**

If C2 (scope assessment) and IdP status pages both indicate that FORM's SSO is healthy but a specific tenant's SSO is failing, the likely cause is an expired SAML certificate on the tenant's side, or an IdP policy change (conditional access policy added, IP restriction modified, MFA requirement changed).

```
1. Confirm with tenant IT admin: "Has your IdP admin team made any changes in the
   last 24 hours to your SAML app configuration or certificate?"

2. If certificate expiry suspected:
   → Request tenant IT admin to check certificate expiry in their IdP admin console.
   → SSO_SCIM_IMPLEMENTATION.md §20 documents the FORM SAML certificate lifecycle;
     the tenant's IdP certificate is the tenant's responsibility.
   → If tenant rotates their IdP certificate, they must re-upload the new
     certificate fingerprint via FORM's admin dashboard or via WorkOS admin API.
   → Platform-engineer assists with re-upload if tenant IT admin is struggling:
     WorkOS Admin API: PUT /saml_connections/{connection_id}
     with updated idp_sso_url and certificate field.

3. After any IdP configuration change, run a test SSO login using the
   FORM-canary synthetic user for that tenant's IdP domain.

4. Monitor AL-SSO-01 for the affected tenant for 30 minutes post-fix.
```

#### Recovery Protocol (when IdP confirms restoration)

```
1. Confirm IdP service restored:
   → IdP status page back to green AND tenant IT admin confirms SSO is working.
   → Do not rely solely on status page: ask the IT admin to attempt a test login.

2. Confirm FORM SSO metrics recovering:
   → OBSERVABILITY §26 dashboard: per-tenant SSO success rate for affected tenant
     returning to ≥ 99% in a rolling 5-minute window.
   → AL-SSO-01 must self-resolve (auto-resolved when success rate returns above threshold).

3. Revoke break-glass token immediately (do not wait for natural expiry):
   POST /admin/v1/tenants/{tenant_slug}/break-glass/{token_id}/revoke
   Authorization: Bearer {SERVICE_ROLE_KEY}
   {
     "revoked_by_user_id": "{ic_user_id}",
     "revocation_reason": "IdP service restored; break-glass no longer required"
   }
   → Confirm sso.break_glass_token_revoked DEC-030 event is present before notifying tenant.
   → Notify tenant IT admin: "Normal SSO access is restored. Your break-glass emergency
     access has been revoked."

4. Invalidate any enterprise sessions created via break-glass if the session owner
   is not the registered admin_email (edge case — safety check):
   SELECT es.id, es.user_id, u.email
   FROM enterprise_sessions es
   JOIN users u ON u.id = es.user_id
   WHERE es.session_metadata->>'auth_method' = 'break_glass'
     AND u.tenant_id = $1
     AND es.expires_at > NOW();
   -- Any unexpected non-admin sessions → revoke immediately + emit sso.break_glass_token_revoked

5. Monitor for 30 minutes post-recovery:
   → OBSERVABILITY §26 SSO dashboard: success rate, P95 latency stable.
   → Sentry: no new SSO-related error events for affected tenant.
   → Check enterprise_sessions: confirm new sessions are being created via SSO
     (session_metadata->>'auth_method' = 'saml' or 'oidc', not 'break_glass').

6. Close incident channel:
   → IC declares incident resolved.
   → Emit sso.idp_outage_resolved DEC-030 event.
   → CSM sends Template E-04 resolution notification to tenant IT admin.
   → If SLA breach threshold was crossed: CSM initiates credit assessment per §23.
```

#### Enterprise Tenant Communication Templates

**Template E-04-Initial (IdP Outage — opening notification):**

```
Subject: FORM Service Update — SSO Login Disruption · [DATE] [TIME UTC]

Hi [Tenant IT Admin Name],

We're aware that some [Company Name] employees may be experiencing difficulty
logging into FORM via [Okta / Microsoft Entra ID / Google Workspace] SSO.

Our platform is operating normally. The issue appears to be with your
identity provider's service. We are monitoring the situation closely.

What still works for employees who are already logged in:
• Workout logging, progress tracking, and coaching sessions continue uninterrupted.
• Employees who are already authenticated are not affected.

For employees who need to log in now:
• We are standing by to provide emergency administrative access to your IT team.
• If you need immediate admin access to FORM, please respond to this message or
  reach your FORM customer success manager directly.

We will send updates every 30 minutes until the issue is resolved.

[FORM Customer Success Manager Name]
enterprise@form.coach
```

**Template E-04-Update (30-minute status update):**

```
Subject: FORM Status Update — [TIME UTC] · SSO Login Disruption

Hi [Tenant IT Admin Name],

Update as of [TIME UTC]:

• Status: [SSO login remains unavailable / SSO login is partially restored]
• Impact: [N] employees affected (those without an active session)
• Root cause: [IdP provider name] is reporting a service disruption on their
  status page: [URL]
• Estimated resolution: [Time from IdP provider / Unknown — monitoring]
• FORM systems: all healthy; your team's data and progress are secure

We will send the next update at [TIME + 30 min UTC] or sooner on resolution.

[FORM Customer Success Manager Name]
```

**Template E-04-Resolution:**

```
Subject: FORM Resolved — SSO Login Restored · [DATE] [TIME UTC]

Hi [Tenant IT Admin Name],

SSO login via [Okta / Microsoft Entra ID / Google Workspace] has been restored
for all [Company Name] employees as of [TIME UTC].

Employees who were unable to log in during the disruption can now log in normally.
No data was lost or affected.

Outage summary:
• Duration: [X] minutes
• Affected: new SSO logins only; active sessions were uninterrupted
• Root cause: [IdP provider name] service disruption

[If break-glass was issued:]
Emergency admin access issued during the incident has been fully revoked.
Normal SSO is the only active authentication method for your organisation.

[If SLA breach threshold crossed:]
We will be in touch separately regarding SLA credit assessment per your agreement.

Thank you for your patience.

[FORM Customer Success Manager Name]
```

**Communication ownership and timing:**

| Duration | Action | Owner |
|---|---|---|
| T+15 min (P1 declared) | Template E-04-Initial to tenant IT admin via Slack Connect + email | CSM; IC approval |
| T+30 min | Template E-04-Update | CSM |
| T+60 min | SLA breach threshold assessment (check against §23 credit table) | CSM + compliance-officer |
| Every 30 min thereafter | Template E-04-Update | CSM |
| Resolution | Template E-04-Resolution + break-glass revocation confirmation | CSM |
| T+48h post-resolution | SLA credit calculation (if applicable) sent to tenant admin | CSM + compliance-officer |

**Vendor naming policy:** In all tenant communications, name the IdP provider — unlike R-17 (Anthropic/ElevenLabs, where naming requires founder approval), an IdP outage is public information the tenant's own IT team is observing. Referring to the named provider (Okta, Microsoft, Google) is accurate and reduces confusion. Do not imply FORM caused the outage.

#### SLA Breach Assessment

Enterprise contracts guarantee SSO login availability at ≥ 99.0% of login attempts per calendar month per tenant. If an IdP outage causes the per-tenant SSO success rate to fall below 99.0% for the month, a credit is triggered per §23 of OBSERVABILITY.md.

```sql
-- SLA calculation: SSO availability for a tenant in the current calendar month
-- Run at incident close; re-run at month-end for final credit calculation.
SELECT
  DATE_TRUNC('month', event_ts)                          AS billing_month,
  COUNT(*) FILTER (WHERE payload->>'outcome' = 'success') AS successful_logins,
  COUNT(*)                                               AS total_login_attempts,
  ROUND(
    COUNT(*) FILTER (WHERE payload->>'outcome' = 'success')::numeric
    / NULLIF(COUNT(*), 0) * 100, 4
  )                                                      AS success_rate_pct,
  99.0                                                   AS sla_threshold_pct,
  CASE
    WHEN ROUND(
      COUNT(*) FILTER (WHERE payload->>'outcome' = 'success')::numeric
      / NULLIF(COUNT(*), 0) * 100, 4
    ) < 99.0 THEN 'BREACHED'
    ELSE 'WITHIN_SLA'
  END                                                    AS sla_status
FROM audit_log_events
WHERE event_type = 'sso.login_attempt'
  AND payload->>'tenant_slug' = $1
  AND event_ts >= DATE_TRUNC('month', NOW())
GROUP BY 1;
```

If `sla_status = 'BREACHED'`, initiate credit per §23. Record the credit assessment in a `sso.sla_breach_assessed` DEC-030 event (HIGH severity, 7-year retention).

#### Post-Incident Review

PIR is required for all P0 and P1 incidents within 5 business days of resolution (§8). For R-19 incidents, the PIR must address:

1. **Was break-glass used?** If yes: were both authorizations captured in the incident channel before token issuance? Is the `sso.break_glass_token_issued` DEC-030 event present with both user_ids? Was the token revoked within 30 minutes of recovery?

2. **What was the scope of user impact?** C1 + C2 delta from the scope assessment — how many users were effectively blocked vs. insulated by active sessions? Was the initial severity classification correct given the actual impact?

3. **Was Template E-04-Initial sent within 15 minutes of P1 declaration?** If not, what caused the delay? The CSM communication SLA is 15 minutes for P1 IdP outage (faster than the 60-minute default in §12 because this is a high-visibility, customer-observed event).

4. **Did the tenant IT admin have an active emergency contact method that worked?** If FORM could not reach the IT admin via Slack Connect or admin_email within 10 minutes, update the emergency contact record for that tenant.

5. **Was the SLA threshold breached?** If yes: was the credit calculation initiated within 48 hours? Is the tenant's CSM satisfied with the communication?

6. **Preventive controls to review:**
   - Did OBSERVABILITY §26 `AL-SSO-01` fire at the right time? Was it the first signal, or did the CSM receive a report before the alert fired?
   - Does the tenant have a secondary authentication method configured (e.g., an emergency local admin account in WorkOS) that FORM could recommend as a long-term mitigation?
   - Should FORM recommend that enterprise tenants configure a backup IdP connection in WorkOS as a resilience mechanism?

#### DEC-030 HMAC-Chained Audit Events

All events are HMAC-chained per `docs/AUDIT_LOG_SCHEMA.md`. The `sso.break_glass_*` chain is a sub-chain keyed by `incident_id`; a break in this sub-chain is P0 per R-05.

| Event Type | DEC-030 Severity | Retention | Trigger Condition | Key Metadata |
|---|---|---|---|---|
| `sso.idp_outage_detected` | HIGH | 3 years | AL-SSO-01 fires and IC opens incident channel | `tenant_slug`, `sso_provider`, `alert_id` (AL-SSO-01), `first_failure_at`, `incident_id` |
| `sso.break_glass_protocol_initiated` | CRITICAL | 7 years | IC formally requests break-glass in incident channel; two-person auth verbally confirmed | `incident_id`, `tenant_slug`, `initiated_by_ic_user_id`, `cop_user_id`, `initiation_reason` |
| `sso.break_glass_token_issued` | CRITICAL | 7 years | Break-glass API endpoint generates and returns a valid token | `incident_id`, `tenant_slug`, `token_id`, `admin_email` (SHA-256 hashed), `expires_at`, `authorized_by_ic_user_id`, `authorized_by_cop_user_id`, `scope: "admin_only"` |
| `sso.break_glass_admin_login` | HIGH | 3 years | Tenant IT admin successfully authenticates using break-glass token | `incident_id`, `tenant_slug`, `token_id`, `session_id`, `login_at`, `ip_subnet` (last two octets masked) |
| `sso.break_glass_token_revoked` | CRITICAL | 7 years | Platform-engineer calls revocation endpoint on recovery | `incident_id`, `tenant_slug`, `token_id`, `revoked_by_user_id`, `revocation_reason`, `revoked_at` |
| `sso.idp_outage_resolved` | STANDARD | 1 year | SSO success rate returns to ≥ 99% AND IT admin confirms SSO working | `incident_id`, `tenant_slug`, `outage_duration_minutes`, `recovery_confirmed_at` |
| `sso.sla_breach_assessed` | HIGH | 7 years | SLA calculation query shows success_rate_pct < 99.0% for billing month | `incident_id`, `tenant_slug`, `billing_month`, `success_rate_pct`, `credit_triggered` (bool), `credit_amount_usd` (if known) |

**7-year retention for break-glass events:** The `sso.break_glass_token_issued` and `sso.break_glass_token_revoked` events carry 7-year retention because they document a deliberate bypass of the SSO security control — the only legitimate bypass mechanism in FORM's architecture. Any unexplained gap between `token_issued` and `token_revoked` events, or any `break_glass_admin_login` event without a preceding `token_issued` event, is a P0 security anomaly (R-01 activation).

```typescript
// DEC-030 emission for break-glass token issuance
// Emitted by the break-glass API endpoint BEFORE returning the token to the caller.
const event = {
  event_type: 'sso.break_glass_token_issued',
  severity: 'CRITICAL',
  incident_id: incidentId,
  payload: {
    tenant_slug: tenantSlug,
    token_id: tokenId,
    admin_email_sha256: sha256(adminEmail),     // never plaintext admin_email
    expires_at: expiresAt.toISOString(),
    authorized_by_ic_user_id: icUserId,
    authorized_by_cop_user_id: copUserId,
    scope: 'admin_only',
    sso_provider: tenant.sso_provider,
  },
};
// Block token delivery until this event is confirmed in the chain.
const emitted = await emitDec030Event(event, supabaseAdminClient);
if (!emitted.success) {
  throw new Error('DEC-030 emission failed — token issuance aborted');
}
return { token_id: tokenId, one_time_code: otp, admin_login_url: signedUrl, expires_at: expiresAt };
```

#### Evidence Package

| Evidence ID | Artefact | Collection Method | Location |
|---|---|---|---|
| **IR-IDP-E-001** | C1 scope assessment output — affected tenant(s), SSO provider, user counts | SQL query output from §R-19 Check C1 exported as JSON at incident open; re-run at close | `compliance/evidence/incidents/<slug>/idp-scope-assessment.json` |
| **IR-IDP-E-002** | IdP status page screenshot at time of detection | Screenshot of trust.okta.com / status.azure.com / workspace.google.com/status showing degradation | `compliance/evidence/incidents/<slug>/idp-status-page-T0.png` |
| **IR-IDP-E-003** | DEC-030 `sso.*` event extract for incident window | `SELECT * FROM audit_log_events WHERE event_type LIKE 'sso.%' AND payload->>'incident_id' = $1 ORDER BY event_ts` | `compliance/evidence/incidents/<slug>/dec030-sso-events.json` |
| **IR-IDP-E-004** | Break-glass authorization record (if break-glass used) | Incident channel log showing both IC and compliance-officer verbal authorization statements; plus `sso.break_glass_protocol_initiated` DEC-030 event JSON | `compliance/evidence/incidents/<slug>/break-glass-authorization.md` |
| **IR-IDP-E-005** | Break-glass issuance and revocation confirmation (if used) | `sso.break_glass_token_issued` + `sso.break_glass_token_revoked` events with matching `token_id`; confirm no open tokens at incident close | `compliance/evidence/incidents/<slug>/break-glass-token-lifecycle.json` |
| **IR-IDP-E-006** | SLA calculation query output (if SLA breach assessed) | SQL output from §R-19 SLA calculation query; billing month; credit assessment | `compliance/evidence/incidents/<slug>/sla-calculation.json` |
| **IR-IDP-E-007** | AL-SSO-01 PagerDuty incident record | PagerDuty alert details: trigger time, acknowledge time, resolve time; correlated with incident open/close timestamps | `compliance/evidence/incidents/<slug>/pagerduty-al-sso-01.json` |
| **IR-IDP-E-008** | Enterprise tenant communication log | All Template E-04 messages sent: timestamp, recipient (CSM channel), IC approval record | `compliance/evidence/incidents/<slug>/tenant-comms-log.md` |

All evidence under `compliance/evidence/incidents/<slug>/` with SHA-256 manifest; 7-year retention for CRITICAL break-glass events; standard 3-year retention for availability events.

#### SOC 2 TSC Mapping

| TSC Criterion | Control | Evidence from This Runbook |
|---|---|---|
| **CC6.1** (Logical access controls) | Break-glass protocol enforces two-person authorization and admin-only scope; DEC-030 event chain provides tamper-evident record of every break-glass activation | IR-IDP-E-003 through IR-IDP-E-005: break-glass issuance, login, and revocation events form the access control audit trail; unexplained gaps are detectable anomalies |
| **CC6.3** (Access removal timeliness) | Break-glass tokens are revoked within 30 minutes of IdP recovery; `sso.break_glass_token_revoked` event confirms revocation with timestamp | IR-IDP-E-005: token lifecycle evidence; revocation within SLA window documented |
| **CC7.2** (Monitoring for and evaluating system events) | AL-SSO-01 provides per-tenant SSO success rate monitoring; C1 scope assessment systematically quantifies blast radius | IR-IDP-E-001 (scope assessment) + IR-IDP-E-007 (PagerDuty record): alert triggered before tenant report in well-implemented instances |
| **CC7.3** (Evaluate and communicate anomalies) | Severity classification matrix, scope assessment, C1/C2/C3 queries, and break-glass issuance procedure constitute the documented response protocol | IR-IDP-E-001 through IR-IDP-E-008 as a complete evidence package for one incident |
| **A1.1** (Availability commitments) | SSO availability SLA (≥ 99.0% per tenant per month) is committed in enterprise contracts; credit mechanism per §23 documents the consequence of SLA breach | IR-IDP-E-006: SLA calculation evidence demonstrates FORM measures and acts on the commitment |
| **A1.2** (Resiliency and recovery) | Break-glass mechanism provides operational continuity for enterprise tenant administrators during IdP outage; OBSERVABILITY §26 monitors recovery | IR-IDP-E-004 + IR-IDP-E-005: break-glass activation and recovery evidence demonstrates a documented resiliency control for third-party IdP dependency |

#### Open Questions

**OQ-IDP-01: Break-glass API endpoint implementation status**

The break-glass token issuance API (`POST /admin/v1/tenants/{slug}/break-glass`) does not yet exist. Until it is implemented (checklist item 1 below), the break-glass procedure requires a manual workaround: platform-engineer creates a temporary WorkOS "magic link" for the tenant admin's email via the WorkOS Admin Dashboard. This is less auditable than the DEC-030-gated API approach. The manual path must not be used without compliance-officer awareness. Owner: platform-engineer. Priority: **P0** (blocking clean break-glass activation). Resolution target: M6.

**OQ-IDP-02: Tenant IT admin emergency contact completeness**

The break-glass procedure depends on FORM being able to reach the tenant IT admin via an out-of-band channel (Slack Connect or admin_email). Not all enterprise tenant records may have a Slack Connect channel established or a verified admin_email. A tenant without an out-of-band contact cannot receive break-glass delivery. Audit current enterprise tenant records for completeness before the first enterprise SSO customer goes live. Owner: customer-success. Priority: **P1**. Resolution target: M10 (before first SSO customer onboarding).

**OQ-IDP-03: Secondary IdP connection as a resilience recommendation**

WorkOS supports configuring a secondary (backup) SAML connection for a tenant's organisation. If the primary Okta connection fails, WorkOS can failover to the secondary connection (e.g., Google Workspace as a backup IdP). Recommending this to enterprise customers during onboarding would reduce FORM's break-glass exposure. However, it adds complexity to the tenant's IdP configuration and requires the tenant to maintain two IdP apps. Decision needed: should FORM include secondary IdP configuration as a recommended onboarding step in the enterprise pilot runbook (SSO_SCIM §17)? Owner: enterprise-architect + customer-success. Priority: **P2**. Resolution target: M12 (before enterprise GA).

#### Implementation Checklist

| # | Action | Priority | Milestone | Owner | Status |
|---|---|---|---|---|---|
| 1 | Build break-glass API endpoint: `POST /admin/v1/tenants/{slug}/break-glass` and `POST /admin/v1/tenants/{slug}/break-glass/{token_id}/revoke`; enforce two-person auth via `authorized_by_ic_user_id` + `authorized_by_cop_user_id` validation against `form_admin` role; emit DEC-030 events before returning token; 4-hour default expiry with admin-only session scope; block if active unexpired break-glass token already exists for tenant | P0 | M6 | platform-engineer | [ ] |
| 2 | Register `sso.idp_outage_detected`, `sso.break_glass_protocol_initiated`, `sso.break_glass_token_issued`, `sso.break_glass_admin_login`, `sso.break_glass_token_revoked`, `sso.idp_outage_resolved`, `sso.sla_breach_assessed` in `docs/AUDIT_LOG_SCHEMA.md` with full schema, severity, and retention fields; validate Zod schema for each in staging | P1 | M6 | security-engineer | [ ] |
| 3 | Audit all enterprise tenant records: confirm `admin_email` is populated, verified, and has out-of-band contact (Slack Connect channel established or confirmed email deliverability); flag any tenant without a complete emergency contact record to CSM for resolution before M10 | P1 | M10 | customer-success | [ ] |
| 4 | Add R-19 tabletop scenario to §9.5 Year 2 Testing Schedule: Okta global outage simulation; CSM attempts E-04-Initial within 15 min SLA; break-glass API called with two-person auth; token issued and delivered to simulated tenant IT admin; recovery procedure and revocation flow completed; PIR template filled for the exercise | P2 | M7 | security-engineer | [ ] |
| 5 | Document the WorkOS magic-link manual break-glass fallback procedure (OQ-IDP-01 interim path) in a restricted internal ops runbook; require compliance-officer awareness log entry for every use; deprecate once break-glass API is live | P0 | M6 | platform-engineer + compliance-officer | [ ] |
| 6 | Add AL-SSO-01 per-tenant SSO login monitoring to PagerDuty "FORM Enterprise SSO" service per OBSERVABILITY §26.5; confirm per-tenant routing (one PagerDuty incident per tenant, not fleet-wide merge); test with a synthetic SSO failure against a canary tenant | P1 | M10 | devops-lead | [ ] |
| 7 | Evaluate secondary IdP connection recommendation for enterprise onboarding (OQ-IDP-03); if approved, add to SSO_SCIM §17 pilot runbook as a Day 1 configuration item with a how-to guide | P2 | M12 | enterprise-architect + customer-success | [ ] |

---

*v1.3 additions (2026-06-02): R-19 Enterprise IdP Outage & Break-Glass Authentication — nineteenth runbook in the taxonomy. Fills the gap between R-04 (SAML certificate compromise / authentication attack — security incident) and R-17 (Anthropic API availability — AI service dependency) by covering the enterprise-specific scenario where a tenant's identity provider (Okta, Microsoft Entra ID, Google Workspace, Ping Identity) is unavailable due to a service outage or tenant-side misconfiguration, preventing enterprise users from authenticating via SSO. Core design: existing authenticated sessions are insulated (enterprise_sessions / Cloudflare KV); only new SSO logins are blocked; break-glass is the relief lever for IT admin operational continuity. Trigger matrix: AL-SSO-01 (per-tenant SSO success rate < 90% sustained 5 min), AL-SSO-02 (P95 SSO latency > 10,000 ms sustained 3 min), direct CSM/IT admin report, or known IdP public status page event. Severity: P2 for < 30% impact partial degradation; P1 for > 50% of tenant users blocked or IT admin locked out; P0 for multiple tenants on same IdP affected simultaneously. Break-glass protocol: two-person authorization (IC + compliance-officer or founder); admin-only scope; 4-hour time limit; DEC-030 event emitted before token delivery (emission failure aborts issuance); token revoked within 30 min of IdP recovery. Three scope assessment queries: C1 (affected tenant SSO provider, user counts, admin_email), C2 (active insulated sessions per tenant — excludes from impact count), C3 (existing break-glass tokens guard against duplicate issuance). Privacy floor: break-glass admin session is identical to normal admin session — no access to individual health data, coaching turns, or biometric fields; enforced at API layer by RLS + is_admin flag. Four Enterprise Template E-04 messages: E-04-Initial (T+15 min P1), E-04-Update (every 30 min), E-04-Resolution (on recovery), SLA credit notification (T+48h if applicable). Vendor naming policy: IdP provider name may be used in tenant communications (public information, reduces confusion) — distinct from R-17 where Anthropic/ElevenLabs naming requires founder approval. Seven DEC-030 HMAC-chained events: sso.idp_outage_detected (HIGH, 3yr), sso.break_glass_protocol_initiated (CRITICAL, 7yr — two-person auth record), sso.break_glass_token_issued (CRITICAL, 7yr — contains SHA-256 hashed admin_email + both authorizer user_ids), sso.break_glass_admin_login (HIGH, 3yr), sso.break_glass_token_revoked (CRITICAL, 7yr — revocation within 30 min of recovery is mandatory), sso.idp_outage_resolved (STANDARD, 1yr), sso.sla_breach_assessed (HIGH, 7yr). SLA breach assessment SQL documented; success_rate_pct < 99.0% triggers credit per §23. Evidence package IR-IDP-E-001 through IR-IDP-E-008 with SHA-256 manifest. SOC 2 mapping: CC6.1 (break-glass two-person auth as documented access control bypass with tamper-evident audit trail), CC6.3 (30-min revocation SLA post-recovery), CC7.2 (AL-SSO-01 per-tenant monitoring), CC7.3 (severity matrix + scope assessment procedure), A1.1 (SSO SLA commitment + credit mechanism), A1.2 (break-glass as documented resiliency control for IdP dependency). Three open questions: OQ-IDP-01 (break-glass API not yet implemented — P0 blocking; interim: WorkOS magic-link manual path; resolve M6), OQ-IDP-02 (tenant admin emergency contact completeness audit — P1; resolve M10 before first SSO customer), OQ-IDP-03 (secondary IdP connection as resilience recommendation — P2; resolve M12). Seven-item implementation checklist (2× P0, 3× P1, 2× P2). Appendix A quick reference card updated: "SSO down?" disambiguated into IdP unavailability (R-19) and SAML cert compromise / SAML attack (R-04). Document header updated v1.1 → v1.3 (v1.2 = R-17 additions per body version note).*

---

*v1.0 additions (2026-06-02): R-18 Database Integrity & Neon Postgres Failover Incident. Eighteenth runbook in the taxonomy; fills the gap left by existing runbooks that assume the database as a given substrate — R-01 through R-17 all presuppose database availability and integrity. R-18 covers four distinct failure modes: (1) Neon primary failover (clean vs. dirty; PITR activation path for dirty failover), (2) RLS cross-tenant data bleed (C2 mismatch scan; immediate read-only containment; R-01 co-activation gate), (3) HMAC audit chain break (C3 chain continuity check; chain state R2 snapshot; R-05 co-activation), (4) pg_cron job failure (DSAR erasure and fleet stats jobs; R-14 coordination for in-window DSAR requests). Four scope assessment queries (C1 row count consistency, C2 RLS cross-tenant mismatch, C3 HMAC chain continuity, C4 pg_cron job health) — all with BYPASSRLS and restricted to incident channel. Severity table with seven tiers: P0 for RLS bypass with Art. 9 data, P0 for HMAC chain destruction, P0 for Art. 9 table corruption, P1 for failover write outage > 2 min, P1 for pg_cron DSAR failure, P1 for connection pool exhaustion, P2 for clean failover. PITR five-step procedure with three-party approval gate (founder + devops-lead + security-engineer); two-party fallback for clean PITR documented. Ten DEC-030 HMAC-chained events: database.failover_detected (HIGH), database.write_path_restored (STANDARD), database.rls_mismatch_detected (CRITICAL, 7-year retention), database.pitr_activation_authorized (CRITICAL, 7-year), database.pitr_recovery_completed (HIGH), database.hmac_chain_gap_detected (CRITICAL, 7-year), database.pg_cron_job_failure (HIGH), database.rls_policy_corrected (HIGH), database.read_only_mode_activated (HIGH), database.read_only_mode_lifted (STANDARD). Evidence package IR-DB-E-001 through IR-DB-E-008 with SHA-256 manifest and 7-year retention for CRITICAL events. SOC 2 mapping: CC7.2 (RLS + chain alerts), CC7.3 (scope assessment + IC framework), CC7.4 (containment + PITR), A1.2 (Neon HA + pg_cron resilience), A1.3 (PITR recovery objectives), CC6.1 (BYPASSRLS restriction + RLS policy framework), CC6.5 (mismatch row disposition evidence). Three open questions: OQ-DB-01 (RLS regression testing in CI — P1 pgTAP), OQ-DB-02 (PITR out-of-band approval — P2), OQ-DB-03 (table_row_count_snapshots existence — P0 blocking C1). Ten-item implementation checklist (4× P0, 4× P1, 2× P2). Owner: security-engineer + devops-lead.*
### R-20: Insider Threat & Privileged Access Abuse

> **Scope:** Covers all scenarios in which a rogue or compromised employee, contractor, or vendor agent who holds `form_admin` role, a PAM-escalated `service_role` key, or Cloudflare Access admin privileges accesses, queries, or exfiltrates data beyond their authorised scope. This runbook is the **insider threat** complement to R-01 (external breach) — the threat actor is already inside the perimeter and has legitimate credentials to some system. **Co-activate R-01 immediately if exfiltration of Art. 9 data is confirmed.** **Co-activate R-05 immediately if the DEC-030 HMAC audit chain shows evidence of tampering or deletion of audit events.** Distinction from R-01: R-01 covers an external attacker who obtained credentials; R-20 covers an internal actor using credentials they were legitimately issued, or a compromised internal account where the actor's intent (malicious vs. inadvertent) is not yet established. Until intent is confirmed, all triggers are treated as potential P0. References: `docs/SSO_SCIM_IMPLEMENTATION.md` §24 (PAM JIT architecture), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 schema), R-01 (data breach co-activation), R-05 (HMAC chain tampering co-activation).

#### Trigger

| Source | Alert ID | Condition |
|---|---|---|
| DEC-030 audit chain | `FORM-INSIDER-001` | `audit_log_events` shows `form_admin` operations on Art. 9 tables (`health_profiles`, `coaching_turns`, `cv_sessions`, `dsar_requests`) outside an approved PAM window, or without a matching `jit_escalation_id` in the event metadata |
| DEC-030 audit chain | `FORM-INSIDER-002` | `bulk_export` or `SELECT *` query on `health_profiles`, `coaching_turns`, or `cv_sessions` returning > 50 rows in a single `service_role` session that does not match any scheduled pg_cron job or documented batch operation |
| PAM system | `FORM-INSIDER-003` | `admin.jit_escalation_expired_without_revocation` event: PAM escalation not revoked within the 4-hour hard limit defined in SSO_SCIM §24 |
| Cloudflare Access | `FORM-INSIDER-004` | Admin login to the Cloudflare Access admin panel from an unexpected IP or device that fails the Zero Trust device posture policy (ZT device policy mismatch) |
| Manual | — | HR or legal counsel initiates a formal separation process or internal investigation that requires evidence preservation before the suspect is notified |
| Manual | — | Any team member files a `#security` Slack report of suspicious internal behaviour (data downloads, unusual admin UI access, unexpected query patterns) |

**All insider threat triggers are treated as a potential P0 with legal hold until §R-20.2 scope assessment confirms severity.** The key principle is that evidence preservation must precede every other action — destroying evidence while rushing to contain is operationally and legally worse than leaving the suspect's access active for an additional 30 minutes.

#### Severity Classification

| Condition | Severity | Reason |
|---|---|---|
| Confirmed exfiltration of Art. 9 data (`health_profiles`, `coaching_turns`, `cv_sessions.keypoints_enc`) by an internal actor | **P0** | GDPR Art. 9 + Art. 33 72-hour supervisory authority notification clock starts at confirmation; activate R-01 immediately |
| Audit chain tampered or `form_admin` operations that deleted or modified DEC-030 `audit_log_events` rows | **P0** | Evidence chain destroyed; activate R-05 immediately; SOC 2 evidence integrity at risk; potential obstruction of justice |
| `service_role` key or Supabase JWT signing secret confirmed leaked externally (e.g., found in public git, external API calls using FORM credentials) | **P0** | `service_role` bypasses RLS completely; all tenant data on the platform is compromised until key is rotated; activate R-01 |
| Bulk queries on Art. 9 tables in a PAM session without a matching business justification; no confirmed exfiltration yet | **P1** | Plausible access path to Art. 9 data exists; scope assessment is mandatory; upgrade to P0 if C3 `total_rows_queried > 0` for sensitive tables |
| PAM escalation not revoked within the 4-hour hard limit (`FORM-INSIDER-003`) | **P1** | Control failure; may indicate intentional persistence; if combined with Art. 9 queries, immediately upgrade to P1 pending C1–C4 |
| `form_admin` account accessed from an unrecognised device or location outside an active PAM window | **P1** | Potential account compromise; treat as insider-or-external until established; parallel assessment with R-01 if login origin is clearly external |
| Unusual admin UI access patterns (browsing admin screens, loading dashboards); no data queries; no Art. 9 tables touched | **P2** | Preliminary review only; document in incident channel; reassess within 4 hours; do not suspend access without C1–C4 confirmation |
| Expired PAM escalation found in audit log with a valid revocation record but revocation was delayed > 30 min beyond the 4-hour window | **P3** | Process gap, not a security incident; document in PAM system; schedule process improvement; no user suspension required |

**P0 upgrade trigger:** If scope assessment Check C3 shows `total_rows_queried > 0` for Art. 9 tables (`health_profiles`, `coaching_turns`, `cv_sessions`) without a documented business justification for the PAM session, immediately upgrade to P0 and activate R-01. The burden of proof is inverted: unexplained Art. 9 data access is presumed a breach until demonstrated otherwise.

#### Why This Matters

FORM processes GDPR Article 9 special-category health data across every production table. An insider with `form_admin` or `service_role` access faces no Row-Level Security barrier — `service_role` bypasses RLS completely, and `form_admin` with `row_security = off` achieves the same effect. A single malicious or careless query in a PAM session can return the entire `health_profiles` table across all tenants. This is simultaneously a multi-tenant data isolation failure, an Art. 9 GDPR breach, and a SOC 2 multi-criterion failure.

The DEC-030 HMAC audit chain is both the primary detection mechanism for insider threats (FORM-INSIDER-001 and FORM-INSIDER-002 fire on anomalous admin events in the chain) and the evidentiary chain that supports any subsequent legal or regulatory action. If an insider tampers with the chain to cover their tracks, they simultaneously destroy the detection signal and the evidence — which is why a chain break during an insider investigation immediately activates R-05 and is treated as a P0 regardless of whether data access has been confirmed.

PAM JIT escalation (SSO_SCIM §24) is the primary access control. Every `form_admin` operation must carry a `jit_escalation_id` that cross-references a time-bounded, two-person-authorised escalation record in `admin_jit_escalations`. Any `form_admin` operation without a valid `jit_escalation_id`, or with a `jit_escalation_id` that falls outside the approved window, is definitionally unauthorised — the control design makes no allowance for exceptional cases. This is intentional: the PAM system exists precisely to force every privileged operation to be pre-authorised and time-bounded.

The `service_role` key bypass makes a leaked or misused service_role key the highest-severity credential compromise possible in the FORM architecture. It is a complete multi-tenant data access compromise — every tenant's Art. 9 data is reachable with a single key. Service_role key rotation is the nuclear option in this runbook: it invalidates all existing service_role connections platform-wide, not only the suspect's session. Rotation must happen in a maintenance window and requires advance coordination, but it cannot be deferred once a service_role compromise is confirmed.

Legal hold is a first-class requirement in this runbook. Unlike external breach scenarios where the priority order is contain → preserve → notify, insider threat inverts the first two steps: **preserve evidence first, then contain**. Destroying access logs, Cloudflare logs, or git history during a rushed containment action is both operationally harmful (destroys the forensic basis for prosecution) and potentially legally harmful (evidence spoliation). Legal counsel must be notified at T+0 — not after scope is confirmed.

#### Immediate Actions (T+0 to T+15 min)

```
1. Open incident channel: #inc-YYYYMMDD-insider
   IC: security-engineer
   Notify immediately: founder + compliance-officer + legal counsel
   (This is a legal matter from T+0 — do NOT delay legal notification pending scope confirmation)

2. EVIDENCE PRESERVATION FIRST (before any remediation):
   Snapshot audit_log_events for the suspect user/session to the evidence path.
   Do NOT delete, update, or archive any log records.
   Do NOT notify the suspect or their direct manager until legal authorises.

   Path: compliance/evidence/incidents/<slug>/audit-snapshot.jsonl
   Method: export via BYPASSRLS query in the incident channel only.
   Record SHA-256 of the snapshot file immediately after creation.

3. Run scope assessment queries C1–C4 (§R-20.2 below).

4. Based on C1–C4, classify severity and decide co-activation:
   - C3 shows Art. 9 rows queried without justification → P0; activate R-01
   - C4 shows hmac_valid = FALSE for any event in the window → P0; activate R-05
   - C2 shows ops outside all PAM windows → P1 minimum; reassess after legal review

5. Suspend the suspect user's SSO access (WorkOS) and revoke any active PAM sessions.
   REQUIREMENT: Two-person authorisation required (IC + compliance-officer or founder).
   Record both authorising user IDs in the incident channel BEFORE executing the suspension.
   WorkOS suspension: WorkOS Admin Dashboard → Users → Suspend.
   PAM revocation: DELETE /admin/v1/jit-escalations/<id> via `pam-elevation-service` Worker
   (`admin_jit_escalations` schema: DATA_MODEL §29, migration 0058_pam_schema.sql; API endpoint implementation pending DATA_MODEL §29.10 item 2).

6. If service_role key is in scope:
   Rotate Supabase service_role key immediately (Supabase Dashboard → Project Settings → API).
   WARNING: This invalidates ALL active service_role connections across the platform.
   Coordinate with devops-lead before rotation. Schedule during lowest-traffic window if time permits.
   Update Cloudflare Workers Secrets: wrangler secret put SUPABASE_SERVICE_ROLE_KEY
   After rotation, confirm Workers are healthy via Sentry error rate dashboard.
```

#### Scope Assessment SQL (§R-20.2)

> Run all queries as `BYPASSRLS` (incident-channel access only). Output is restricted to the `#inc-YYYYMMDD-insider` channel and the compliance evidence package. Never share in general engineering channels. These queries constitute the legal evidence record — export and SHA-256 hash each output immediately.

**Check C1: All `form_admin` operations by suspect identity in last 30 days**

```sql
-- C1: Identify all form_admin operations by suspect identity in last 30 days
-- Run as BYPASSRLS (incident channel only)
SELECT
  ale.id,
  ale.event_type,
  ale.actor_user_id,
  ale.tenant_id,
  ale.created_at,
  ale.metadata->>'jit_escalation_id' AS jit_escalation_id,
  ale.metadata->>'table_name'        AS table_name,
  ale.metadata->>'row_count'         AS row_count,
  ale.hmac_valid
FROM audit_log_events ale
WHERE ale.actor_user_id = '<suspect_user_id>'
  AND ale.created_at >= NOW() - INTERVAL '30 days'
ORDER BY ale.created_at DESC;
-- Examine: jit_escalation_id presence, table_name for Art. 9 tables,
-- row_count values, and hmac_valid. Any hmac_valid = FALSE triggers R-05.
```

**Check C2: Cross-reference PAM escalation windows — identify operations outside approved windows**

```sql
-- C2: Cross-reference PAM escalation windows — find ops outside approved windows
SELECT
  ale.id,
  ale.event_type,
  ale.created_at,
  ale.metadata->>'jit_escalation_id' AS jit_id,
  jit.escalation_start,
  jit.escalation_expiry,
  jit.revoked_at,
  CASE
    WHEN jit.id IS NULL
      THEN 'NO_PAM_WINDOW'
    WHEN ale.created_at NOT BETWEEN jit.escalation_start
      AND COALESCE(jit.revoked_at, jit.escalation_expiry)
      THEN 'OUTSIDE_WINDOW'
    ELSE 'WITHIN_WINDOW'
  END AS auth_status
FROM audit_log_events ale
LEFT JOIN admin_jit_escalations jit
  ON ale.metadata->>'jit_escalation_id' = jit.id::text
WHERE ale.actor_user_id = '<suspect_user_id>'
  AND ale.event_type ILIKE 'admin.%'
  AND ale.created_at >= NOW() - INTERVAL '30 days'
ORDER BY ale.created_at DESC;
-- Expected for authorised ops: auth_status = 'WITHIN_WINDOW' for all rows.
-- NO_PAM_WINDOW or OUTSIDE_WINDOW = unauthorised operation; escalate immediately.
-- admin_jit_escalations table: DATA_MODEL §29 (migration 0058_pam_schema.sql). jit_escalation_id injected by fn_inject_pam_session_id() trigger.
```

**Check C3: Quantify Art. 9 data exposure — rows queried per sensitive table**

```sql
-- C3: Quantify Art. 9 data exposure — rows queried per sensitive table
SELECT
  (ale.metadata->>'table_name')             AS table_name,
  SUM((ale.metadata->>'row_count')::int)    AS total_rows_queried,
  COUNT(*)                                  AS query_count,
  MIN(ale.created_at)                       AS first_at,
  MAX(ale.created_at)                       AS last_at
FROM audit_log_events ale
WHERE ale.actor_user_id = '<suspect_user_id>'
  AND ale.metadata->>'table_name' IN (
    'health_profiles', 'coaching_turns', 'cv_sessions',
    'dsar_requests', 'user_devices', 'workout_sessions'
  )
  AND ale.created_at >= NOW() - INTERVAL '30 days'
GROUP BY ale.metadata->>'table_name'
ORDER BY total_rows_queried DESC;
-- If total_rows_queried > 0 for health_profiles, coaching_turns, or cv_sessions
-- AND no matching business justification exists for the PAM session:
-- immediately upgrade to P0 and activate R-01. Art. 33 clock starts NOW.
```

**Check C4: HMAC chain integrity check for audit events in the suspect window**

```sql
-- C4: HMAC chain integrity check for audit events in suspect window
SELECT
  COUNT(*) FILTER (WHERE hmac_valid = FALSE) AS broken_links,
  COUNT(*) FILTER (WHERE hmac_valid = TRUE)  AS valid_links,
  MIN(created_at) FILTER (WHERE hmac_valid = FALSE) AS first_break_at
FROM audit_log_events
WHERE created_at BETWEEN '<window_start>' AND '<window_end>';
-- Expected: broken_links = 0.
-- Any broken_links > 0: activate R-05 immediately. This is P0 regardless of
-- whether data access is confirmed. Evidence chain is compromised.
```

> After running all four checks, document the results in `compliance/evidence/incidents/<slug>/scope-assessment.md` with SHA-256 hashes of all query outputs. This document is the primary scope assessment record for legal and regulatory purposes.

#### Containment

```
After evidence preservation and scope assessment:

SUSPEND-1. WorkOS SSO suspension (two-person auth confirmed in incident channel):
           WorkOS Admin → Users → [suspect user] → Suspend
           Emit DEC-030 event: admin.user_access_suspended

SUSPEND-2. PAM escalation revocation (all active JIT sessions for suspect user):
           DELETE /admin/v1/jit-escalations/<id> for each active session via `pam-elevation-service` Worker
           (admin_jit_escalations schema: DATA_MODEL §29 migration 0058_pam_schema.sql;
            API endpoint implementation pending DATA_MODEL §29.10 item 2 — manual fallback:
            UPDATE admin_jit_escalations SET status='revoked', revoked_at=NOW()
            WHERE actor_user_id='<suspect>' AND status='approved' [BYPASSRLS; two-person auth])
           Emit DEC-030 event: admin.jit_escalation_emergency_revoked

SUSPEND-3. Cloudflare Access revocation (if Cloudflare admin access is in scope):
           Cloudflare Zero Trust → Users → [suspect user] → Revoke access
           Revoke all active Cloudflare Access service tokens associated with suspect

SERVICE-ROLE (only if service_role compromise confirmed or strongly suspected):
           Supabase Dashboard → Project Settings → API → Regenerate service_role key
           Update Cloudflare Workers Secrets: wrangler secret put SUPABASE_SERVICE_ROLE_KEY
           Redeploy Workers: wrangler deploy
           Confirm Sentry error rate returns to baseline within 5 minutes of redeploy
           Emit DEC-030 event: admin.service_role_key_rotated
```

#### Communication Protocol

All communications during an insider threat investigation are legally sensitive. Deviations from this protocol without explicit legal counsel authorisation are prohibited.

- **Internal-only until founder + legal counsel authorise any external disclosure.** Do not discuss the investigation in Slack channels other than `#inc-YYYYMMDD-insider` (private, IC-controlled membership).
- **Do NOT communicate to the suspect or their direct team** until legal counsel explicitly authorises. Even a routine check-in message can constitute tipping off and may constitute evidence spoliation in a subsequent employment or criminal proceeding.
- **Enterprise tenant notification:** If enterprise tenant Art. 9 data is confirmed within scope, prepare Template E-05 (Insider Incident Tenant Notification) for compliance-officer approval before sending. The same 72-hour GDPR Art. 33 clock applies as in R-01. Tenant notification must be reviewed by legal counsel before transmission.
- **HR/legal liaison:** All communications with HR, the suspect's manager, and external counsel route through counsel. No Slack direct messages to the suspect. No calendar invites from engineering. No informal conversations.
- **Status page / public communication:** Do not post to the public status page for insider threat incidents. If service disruption occurs as a side-effect (e.g., service_role key rotation causing a brief connectivity gap), post a generic "connectivity issue" notice that does not reference the investigation.
- **Board notification:** Founder notifies board at T+24h if P0 is confirmed. Compliance-officer drafts the board communication; legal counsel reviews before sending.

#### DEC-030 Audit Events

All events listed below are HMAC-chained per `docs/AUDIT_LOG_SCHEMA.md`. CRITICAL 7-year retention applies to all events in this runbook due to potential litigation hold — employment disputes, regulatory investigations, and criminal referrals all require long-lived evidence chains. Do not alter retention downward without legal counsel sign-off.

| Event Type | Severity | Retention | Key Metadata Fields |
|---|---|---|---|
| `admin.insider_threat_investigation_opened` | CRITICAL | 7 years | `suspect_user_id`, `trigger_alert_id` (FORM-INSIDER-001 through -004, or `manual`), `ic_user_id`, `legal_notified` (bool), `incident_slug` |
| `admin.jit_escalation_emergency_revoked` | CRITICAL | 7 years | `escalation_id`, `revoked_by_user_id`, `authorised_by_cop_user_id`, `reason: insider_threat`, `incident_slug` |
| `admin.service_role_key_rotated` | CRITICAL | 7 years | `rotated_by_user_id`, `authorised_by` (two user_ids), `reason` (e.g., `insider_threat_confirmed` or `precautionary`), `incident_slug` |
| `admin.user_access_suspended` | HIGH | 3 years | `suspended_user_id`, `suspended_by_user_id`, `authorised_by_second_user_id`, `incident_slug`, `suspension_reason` |
| `admin.evidence_snapshot_created` | HIGH | 7 years | `snapshot_path`, `sha256`, `row_count`, `table_coverage` (array of table names covered), `incident_slug` |
| `admin.insider_threat_investigation_closed` | HIGH | 7 years | `outcome` (`confirmed` / `unsubstantiated` / `ongoing`), `r01_activated` (bool), `r05_activated` (bool), `gdpr_notification_required` (bool), `incident_slug` |

**7-year retention rationale:** Employment litigation timelines in relevant jurisdictions (US, EU) extend to 3–6 years post-event. GDPR regulatory investigation timelines extend 3–5 years. Criminal referrals have no fixed window. All events from an insider threat investigation must be retained for 7 years regardless of outcome.

```typescript
// DEC-030 emission for investigation open — emitted before any other action
// in the runbook, including evidence preservation. If emission fails, abort
// and retry: the investigation must be on-chain from the first moment.
const event = {
  event_type: 'admin.insider_threat_investigation_opened',
  severity: 'CRITICAL',
  incident_id: incidentId,
  payload: {
    suspect_user_id: suspectUserId,          // internal UUID, never external email
    trigger_alert_id: triggerAlertId,        // FORM-INSIDER-001 | 002 | 003 | 004 | manual
    ic_user_id: icUserId,
    legal_notified: true,                    // must be TRUE before emitting; block if false
    incident_slug: incidentSlug,
  },
};
// Block all subsequent actions until this event is confirmed in the chain.
const emitted = await emitDec030Event(event, supabaseAdminClient);
if (!emitted.success) {
  throw new Error('DEC-030 emission failed — investigation open aborted; retry required');
}
```

#### Evidence Package

| Evidence ID | Artefact | Collection Method | Location |
|---|---|---|---|
| **IR-INS-E-001** | C1 query output — all form_admin operations by suspect (30-day window) | SQL output from §R-20.2 Check C1 exported as JSONL at T+15 min; SHA-256 recorded | `compliance/evidence/incidents/<slug>/c1-admin-ops.jsonl` |
| **IR-INS-E-002** | C2 query output — PAM window cross-reference | SQL output from §R-20.2 Check C2 showing `auth_status` per operation; identifies all `NO_PAM_WINDOW` and `OUTSIDE_WINDOW` events | `compliance/evidence/incidents/<slug>/c2-pam-cross-ref.jsonl` |
| **IR-INS-E-003** | C3 query output — Art. 9 data exposure quantification | SQL output from §R-20.2 Check C3 showing `total_rows_queried` per sensitive table; primary blast-radius record for GDPR Art. 33 notification if R-01 is activated | `compliance/evidence/incidents/<slug>/c3-art9-exposure.jsonl` |
| **IR-INS-E-004** | C4 HMAC chain integrity check output | SQL output from §R-20.2 Check C4; if `broken_links > 0`, R-05 evidence package supplements this record | `compliance/evidence/incidents/<slug>/c4-hmac-integrity.jsonl` |
| **IR-INS-E-005** | Access suspension log | WorkOS Admin audit log extract showing suspension event + timestamp; DEC-030 `admin.user_access_suspended` event with both authorising user IDs | `compliance/evidence/incidents/<slug>/access-suspension.json` |
| **IR-INS-E-006** | PAM revocation record | DEC-030 `admin.jit_escalation_emergency_revoked` event + `admin_jit_escalations` row export confirming `status = 'revoked'` and `revoked_at` timestamp (schema: DATA_MODEL §29.3) | `compliance/evidence/incidents/<slug>/pam-revocation.json` |
| **IR-INS-E-007** | Credential rotation evidence | If service_role rotated: Supabase key rotation confirmation screenshot + `admin.service_role_key_rotated` DEC-030 event + Sentry error rate post-rotation baseline screenshot confirming no connectivity degradation | `compliance/evidence/incidents/<slug>/credential-rotation.json` |

All evidence under `compliance/evidence/incidents/<slug>/` with SHA-256 manifest. 7-year retention for all IR-INS-E-* artefacts due to litigation hold. R2 object versioning enabled; no lifecycle deletion rule applies to evidence paths.

#### SOC 2 TSC Mapping

| TSC Criterion | Control | Evidence from This Runbook |
|---|---|---|
| **CC6.1** (Logical access controls) | PAM JIT escalation (SSO_SCIM §24) requires `jit_escalation_id` for every `form_admin` operation; no `form_admin` operation is authorised without a pre-approved, time-bounded PAM window | IR-INS-E-002: C2 cross-reference output demonstrates that all admin operations are matched to approved PAM windows; `NO_PAM_WINDOW` events are the detection signal |
| **CC6.2** (Access provisioning controls) | Insider detection programme demonstrates active monitoring of provisioned privileged access; FORM-INSIDER-001/002 alerts fire on anomalous exercise of provisioned access | IR-INS-E-001: C1 query output as the access-monitoring record for the provisioned `form_admin` role |
| **CC6.3** (Access removal timeliness) | PAM 4-hour hard limit enforces time-bounded access; `FORM-INSIDER-003` detects violations; emergency revocation procedure (SUSPEND-1 through SUSPEND-3) removes access within 15 minutes of P0 declaration | IR-INS-E-005 + IR-INS-E-006: suspension and PAM revocation evidence demonstrating timely access removal |
| **CC6.6** (Logical access restriction from outside the system) | Cloudflare Access Zero Trust device posture policy is the external access gate; `FORM-INSIDER-004` fires on device policy mismatch, which may indicate credential theft enabling access from an unmanaged device | Cloudflare Access audit log + `FORM-INSIDER-004` alert record as evidence of the external access gate control |
| **CC7.2** (Monitoring for and evaluating system events) | `FORM-INSIDER-001` (admin ops outside PAM window) and `FORM-INSIDER-002` (bulk Art. 9 queries in service_role session) are continuous automated alerts against the DEC-030 chain | DEC-030 `audit_log_events` + alert configurations for FORM-INSIDER-001/002 as continuous monitoring evidence |
| **CC7.3** (Anomaly evaluation and communication) | Scope assessment queries C1–C4 constitute the structured anomaly evaluation procedure; communication protocol (legal at T+0, two-person auth for suspension, legal hold) is the communication procedure | IR-INS-E-001 through IR-INS-E-007 as a complete evidence package demonstrating systematic anomaly evaluation for one insider investigation |

#### Open Questions

**OQ-INS-01: 🟢 Resolved** — `admin_jit_escalations` schema defined in `docs/DATA_MODEL.md §29` (migration `0058_pam_schema.sql`); `pam_break_glass_reviews` schema included; `fn_inject_pam_session_id()` trigger unblocks C2 forensic query.

~~The C2 query (§R-20.2), FORM-INSIDER-001 alert, and PAM revocation API all depend on `admin_jit_escalations` and `jit_escalation_id` in `audit_log_events.metadata` — neither existed in production schema.~~ **Resolved by `docs/DATA_MODEL.md §29` (2026-06-10, migration `0058_pam_schema.sql`):** both tables authored with full RLS, indexes, and audit trigger. C2 query is now executable once migration `0058` is deployed to production. FORM-INSIDER-001 automated detection is unblocked — implement per DATA_MODEL §29.10 item 9 (P1 M7, security-engineer + platform-engineer). PAM revocation API (`DELETE /admin/v1/jit-escalations/<id>`) pending DATA_MODEL §29.10 item 2; manual fallback documented in SUSPEND-2 above.

**OQ-INS-02: Bulk query detection for `service_role` sessions (FORM-INSIDER-002) — requires row count audit logging**

FORM-INSIDER-002 requires that `audit_log_events` captures the number of rows returned by each `SELECT` query executed in a `service_role` session. Postgres does not natively log per-query row counts at the application level. Options: (a) `pg_stat_statements` extension — captures query text and row count, but requires a Worker-side polling job to read and emit DEC-030 events; (b) a middleware shim in the Cloudflare Workers database client that intercepts `service_role` queries and logs response cardinality before returning results to the caller. Option (b) is preferred as it emits real-time DEC-030 events rather than polling. Until implemented, FORM-INSIDER-002 cannot fire automatically. Owner: platform-engineer. Priority: **P1**. Target: M8.

**OQ-INS-03: Legal hold automation — `POST /admin/v1/incidents/<slug>/legal-hold` API**

Today, evidence preservation requires a human to manually run the C1 snapshot query and upload the output to R2. A `POST /admin/v1/incidents/<slug>/legal-hold` API endpoint that: (a) atomically snapshots all `audit_log_events` rows matching a suspect user_id and time window into an immutable R2 object; (b) sets a `legal_hold = true` flag on those rows preventing deletion by any lifecycle job; (c) emits the `admin.evidence_snapshot_created` DEC-030 event with SHA-256; would improve evidence preservation speed and reliability at T+0. Until implemented, the manual snapshot procedure in step 2 of the Immediate Actions is the required path. Owner: platform-engineer + security-engineer. Priority: **P2**. Target: M10.

#### Implementation Checklist

| # | Action | Priority | Milestone | Owner | Status |
|---|---|---|---|---|---|
| 1 | Create `admin_jit_escalations` + `pam_break_glass_reviews` schemas; add `fn_inject_pam_session_id()` audit trigger on all six sensitive tables; close OQ-INS-01 schema blocker | P0 | M6 | platform-engineer | [x] — **Done**: `docs/DATA_MODEL.md §29` (migration `0058_pam_schema.sql`, 2026-06-10); see DATA_MODEL §29.10 for remaining deployment checklist |
| 2 | Implement PAM revocation API endpoint `DELETE /admin/v1/jit-escalations/<id>`; enforce two-person auth (IC + compliance-officer); emit `admin.jit_escalation_emergency_revoked` DEC-030 event before returning 200; block revocation of already-revoked escalations | P0 | M6 | platform-engineer | [ ] — schema prerequisite (item 1) now unblocked via DATA_MODEL §29 |
| 3 | Implement FORM-INSIDER-001 alert: Cloudflare Edge Function (cron, every 15 min) queries `audit_log_events` for `form_admin` ops on Art. 9 tables with NULL or mismatched `jit_escalation_id`; fires PagerDuty P0 to security-engineer + IC on any result; confirm in staging with synthetic mismatched event | P0 | M6 | devops-lead + security-engineer | [ ] |
| 4 | Implement FORM-INSIDER-002 alert: Worker middleware shim to log `service_role` query response cardinality; emit `admin.bulk_query_detected` DEC-030 event when row count > 50 on Art. 9 table in a single session not matching pg_cron schedule; close OQ-INS-02 | P1 | M8 | platform-engineer | [ ] |
| 5 | Register all six `admin.insider_threat_*` and `admin.jit_escalation_*` DEC-030 events in `docs/AUDIT_LOG_SCHEMA.md` with full schema, severity, and 7-year retention fields; validate Zod schema for each event type in staging | P1 | M6 | security-engineer | [ ] |
| 6 | Implement legal hold API `POST /admin/v1/incidents/<slug>/legal-hold`: snapshot matching `audit_log_events` rows to immutable R2 object; set `legal_hold = true` on source rows; emit `admin.evidence_snapshot_created` with SHA-256; close OQ-INS-03 | P2 | M10 | platform-engineer + security-engineer | [ ] |
| 7 | Add R-20 tabletop scenario to §9.5 Year 2 Testing Schedule: simulate FORM-INSIDER-001 firing on a synthetic `form_admin` query outside a PAM window; IC opens incident channel; C1–C4 queries run; suspension two-person auth documented; evidence snapshot created; PIR template completed; measure elapsed time vs. 15-min T+0 SLA | P2 | M7 | security-engineer | [ ] |
| 8 | Update Appendix A quick reference: add "Internal suspicious behaviour?" → R-20; disambiguate from "External breach" → R-01; add note "If audit chain broken during R-20" → also activate R-05 | P1 | M6 | security-engineer | [ ] |

---

*v1.5 (2026-06-10): R-20 cross-reference patch — OQ-INS-01 resolved by `docs/DATA_MODEL.md §29` (migration `0058_pam_schema.sql`, 2026-06-10). Five locations updated: (1) SUSPEND-2 containment step — interim manual path note replaced with `admin_jit_escalations` schema reference + manual SQL fallback with explicit column names (`status='revoked'`, `revoked_at=NOW()`); (2) C2 forensic query — "Depends on admin_jit_escalations table — see OQ-INS-01" removed; replaced with canonical schema reference + `fn_inject_pam_session_id()` trigger note; (3) PAM revocation step 5 — PAM API note updated to reference DATA_MODEL §29.10 item 2 (implementation pending); (4) IR-INS-E-006 evidence description — "manual record … incident channel screenshot" replaced with `admin_jit_escalations` row export description confirming `status = 'revoked'` and `revoked_at`; (5) OQ-INS-01 open question — heading updated to 🟢 Resolved; body replaced with resolution narrative citing DATA_MODEL §29; checklist item 1 status updated to `[x] Done` with migration reference; checklist item 2 (PAM revocation API) remains `[ ]` open but notes schema prerequisite now unblocked. Remaining open items: OQ-INS-02 (bulk query middleware — P1 M8), OQ-INS-03 (legal hold API — P2 M10), FORM-INSIDER-001 automation (P1 M7 — DATA_MODEL §29.10 item 9). Cross-references: `docs/DATA_MODEL.md §29` (admin_jit_escalations + pam_break_glass_reviews schemas; fn_inject_pam_session_id trigger), `docs/SSO_SCIM_IMPLEMENTATION.md §24` (PAM KV architecture; DEC-030 events; AL-PAM-01/02/03), `docs/OBSERVABILITY.md §29` (PAM observability; AL-PAM-BG-01). Owner: security-engineer + compliance-officer.*

*v1.4 additions (2026-06-02): R-20 Insider Threat & Privileged Access Abuse — the twentieth runbook and complement to R-01 (external breach). Key design decisions: (1) evidence preservation before remediation — destroying logs while rushing to contain is both forensically harmful and potentially legally harmful; suspending access 30 minutes later than the fastest possible time is always preferable to evidence spoliation; (2) legal notification at T+0, not after confirmation — in insider cases, legal must direct the investigation from the start; (3) two-person authorisation required for all suspension and revocation actions; (4) service_role key rotation is the nuclear option — invalidates all service_role connections platform-wide; (5) DEC-030 chain is simultaneously the primary detection signal and the evidentiary chain — chain tampering treated as P0 regardless of whether data access is confirmed. Trigger matrix: FORM-INSIDER-001 (admin ops outside PAM window), FORM-INSIDER-002 (bulk Art. 9 queries in service_role session), FORM-INSIDER-003 (PAM escalation not revoked within 4-hour hard limit), FORM-INSIDER-004 (Cloudflare Access device policy mismatch), manual HR/legal separation, manual #security report. Severity: P0 for confirmed exfiltration, audit chain tamper, or service_role key external leak; P1 for bulk Art. 9 queries without justification, unrevoked PAM escalation, unrecognised device/location; P2 for unusual admin UI without data queries; P3 for delayed-but-valid PAM revocation. Four SQL queries C1–C4. Six DEC-030 events all CRITICAL/HIGH 7-year retention. Evidence IR-INS-E-001 through IR-INS-E-007. SOC 2: CC6.1/CC6.2/CC6.3/CC6.6/CC7.2/CC7.3. Original open questions: OQ-INS-01 (admin_jit_escalations schema — resolved v1.5), OQ-INS-02 (bulk query middleware — P1, M8), OQ-INS-03 (legal hold API — P2, M10). Eight-item checklist (3× P0, 3× P1, 2× P2). Appendix A updated: "Internal suspicious behaviour?" → R-20. Owner: security-engineer + compliance-officer.*

---

### R-21: Key Rotation Failure & Scheduled Cryptographic Control Emergency

**Owner:** security-engineer + devops-lead · **Reviewer:** compliance-officer
**Triggers:** AL-KEY-01 (imminent ≤14d), AL-KEY-02 (overdue P0), AL-KEY-03 (HMAC cadence >395d), AL-KEY-04 (verification missing >24h), key-rotation-scheduler Worker silence
**SOC 2 evidence:** CC5.2, CC5.3, CC6.7, CC6.8, C1.1, CC7.2, CC7.3

> **Scope:** This runbook covers operational key rotation failures — keys that are overdue, rotation procedures that failed mid-execution, or verification events that did not follow rotation events. It does NOT cover unauthorized key rotation. If the trigger is `CRYPTO-CHAIN-02` (rotation event without a preceding initiation event within 1 hour), **stop and activate R-05 + R-20 immediately**. Do not continue with R-21.

#### Severity Classification

| Condition | Severity | Response SLA |
|---|---|---|
| Any key overdue (past scheduled rotation date) | **P0** | IC within 15 min; rotation within 2 hours |
| `HMAC_AUDIT_CHAIN_KEY` cadence >400 days (AL-KEY-03 escalated) | **P0** | IC within 15 min; HMAC dual-key rotation within 4 hours |
| `hmac_key_rotation_verified` missing >24h after `hmac_key_rotated` (AL-KEY-04) | **P0** | IC within 15 min; verification run within 1 hour |
| key-rotation-scheduler Worker silent (no KV write in >26h) | **P1** | IC within 30 min; scheduler restart within 1 hour |
| Any key within ≤14-day window (AL-KEY-01) | **P1** | IC within 4 hours; rotation before deadline |
| `HMAC_AUDIT_CHAIN_KEY` cadence 366–399 days (AL-KEY-03) | **P1** | IC within 4 hours; schedule dual-key rotation |
| TLS certificate expiry ≤30 days (Better Stack SSL) | **P2** | Inform devops-lead; schedule renewal within 7 days |

**Upgrade rule:** Any P1 that reaches the key deadline without rotation automatically becomes P0.

#### Trigger Matrix

| Alert ID | Source | Severity | Action |
|---|---|---|---|
| AL-KEY-01 (≤14d remaining) | key-rotation-scheduler Worker → form-crypto-health KV → Better Stack | P1 | Open incident channel; identify affected key; schedule rotation |
| AL-KEY-01 (≤3d remaining, auto-escalate) | Same, re-alert per OBSERVABILITY.md §30.4 | P0 | Immediate IC; emergency rotation if planned rotation is not achievable before deadline |
| AL-KEY-02 (overdue) | key-rotation-scheduler → PagerDuty | P0 | Immediate IC; two-person auth; emergency rotation per key-type procedure below |
| AL-KEY-03 (HMAC cadence >395d) | key-rotation-scheduler → PagerDuty | P1 | Open incident; activate HMAC dual-key rotation procedure (SOC2_READINESS §58) |
| AL-KEY-04 (verify missing >24h) | pg_cron audit check → PagerDuty | P0 | Open incident; re-run verification immediately; if verification fails twice, treat as potential compromise → R-05 |
| key-rotation-scheduler silent (>26h) | Better Stack synthetic via `crypto:scheduler:last_run` KV key | P1 | Restart Worker; confirm KV writes resume; check Sentry for errors |
| CRYPTO-CHAIN-02 (rotation without initiation) | DEC-030 HMAC batch checker | **STOP — NOT R-21** | Activate R-05 + R-20 immediately |

#### Immediate Actions (T+0 to T+15 min)

```
R21-OPEN. Open incident channel: #inc-YYYYMMDD-keyrot in Slack.
           Assign IC (security-engineer or devops-lead on-call).
           Emit DEC-030 admin.key_rotation_incident_opened BEFORE any rotation action.
           If emission fails, abort and retry — the incident must be on-chain from the first moment.

R21-ID.   Identify affected key from PagerDuty alert payload:
           key_id (e.g., SUPABASE_SERVICE_ROLE_JWT, HMAC_AUDIT_CHAIN_KEY, KEYPOINTS_ENC_KEY)
           days_overdue / days_until_rotation from form-crypto-health KV.
           Confirm trigger is NOT CRYPTO-CHAIN-02 before continuing.

R21-AUTH. Two-person authorisation for any P0 rotation action.
           IC nominates second approver (compliance-officer or founder if security-engineer is IC).
           Both user_ids captured in admin.encryption_key_rotation_initiated DEC-030 event.
```

#### Scope Assessment Queries

All queries require PAM-authorised `service_role` access (SSO_SCIM §24). Run only after `admin.key_rotation_incident_opened` is confirmed on-chain.

**Check C1: form-crypto-health KV state**

```bash
# Query Cloudflare KV for all key health records
curl -s \
  "https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/storage/kv/namespaces/<CRYPTO_KV_NS>/values/crypto:status" \
  -H "Authorization: Bearer $CF_API_TOKEN" | jq .
# Expected: JSON with key_id, days_until_rotation, key_version, last_checked_at for each key
# Red flag: days_until_rotation < 0 (overdue), scheduler_healthy: false
```

**Check C2: Recent key rotation events in audit_log**

```sql
-- C2: Key rotation event history — last 60 days for the affected key
SELECT
  event_type,
  created_at,
  metadata->>'key_id'           AS key_id,
  metadata->>'key_version'      AS key_version,
  metadata->>'rotated_by'       AS rotated_by,
  hmac_valid
FROM audit_log_events
WHERE event_type IN (
  'admin.encryption_key_rotation_initiated',
  'admin.encryption_key_rotated',
  'admin.encryption_key_rotation_verified'
)
  AND metadata->>'key_id' = '<AFFECTED_KEY_ID>'
  AND created_at >= NOW() - INTERVAL '60 days'
ORDER BY created_at DESC;
-- Expected: initiated → rotated → verified triples, all hmac_valid = TRUE.
-- Red flag: rotated without preceding initiated → STOP, activate R-05 + R-20.
-- Red flag: rotated without subsequent verified within 24h → AL-KEY-04 scenario.
-- Red flag: no entries for the affected key in > rotation_period days → rotation skipped entirely.
```

**Check C3: key-rotation-scheduler Worker health**

```bash
# Check scheduler last-run timestamp from KV
curl -s \
  "https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/storage/kv/namespaces/<CRYPTO_KV_NS>/values/crypto:scheduler:last_run" \
  -H "Authorization: Bearer $CF_API_TOKEN" | jq .
# Expected: last_run_at within the last 25 hours
# Red flag: last_run_at > 26 hours ago → scheduler crashed or misconfigured
# Also check Sentry for key-rotation-scheduler Worker errors in the same window
```

**Check C4: HMAC_AUDIT_CHAIN_KEY cadence (AL-KEY-03 scenario)**

```sql
-- C4: HMAC_AUDIT_CHAIN_KEY last rotation date
SELECT
  created_at                                                        AS last_hmac_rotation,
  NOW() - created_at                                               AS age,
  EXTRACT(EPOCH FROM (NOW() - created_at)) / 86400                 AS age_days
FROM audit_log_events
WHERE event_type = 'admin.encryption_key_rotated'
  AND metadata->>'key_id' = 'HMAC_AUDIT_CHAIN_KEY'
ORDER BY created_at DESC
LIMIT 1;
-- Expected: age_days < 366 (annual schedule with 10% buffer = AL-KEY-03 threshold at 395d)
-- Red flag: age_days > 395 → dual-key rotation required per SOC2_READINESS §58
-- Red flag: no result → HMAC_AUDIT_CHAIN_KEY has never been formally rotated → P0, R-05 assessment
```

> Document all C1–C4 outputs in `compliance/evidence/incidents/<slug>/scope-assessment.md` with SHA-256 hashes before proceeding to rotation.

#### Response by Key Type

**SUPABASE_SERVICE_ROLE_JWT (90-day schedule):** Follow `docs/SOC2_READINESS.md §57`. Generate new key in Supabase Dashboard → deploy to Cloudflare Workers Secrets (`wrangler secret put SUPABASE_SERVICE_ROLE_KEY`) → redeploy all Workers → confirm Sentry error rate returns to baseline → emit `admin.encryption_key_rotated` → verify within 24h.

**HMAC_AUDIT_CHAIN_KEY (annual dual-key rotation):** Follow `docs/SOC2_READINESS.md §58`. Higher ceremony: emit `admin.encryption_key_rotation_initiated` BEFORE generating new key; activate dual-key mode (24-hour overlap window — old and new keys both valid); after overlap: deactivate old key, emit `admin.encryption_key_rotated`; verify HMAC chain integrity for all events written during the overlap window; emit `admin.encryption_key_rotation_verified`. Do NOT complete this rotation in less than 24 hours — a rush rotation without overlap breaks chain verification for events in the transition window.

**KEYPOINTS_ENC_KEY (365-day schedule; Art. 9 biometric data):** Two-person auth required. Requires a maintenance window — communicate to enterprise tenants ≥72h in advance via Template K-01. Emit `admin.encryption_key_rotation_initiated` with `requires_reencryption: true`. Run re-encryption job in a transaction; confirm row count before and after matches. Emit `admin.encryption_key_rotated` and `admin.encryption_key_rotation_verified` within the same maintenance window. See OQ-KEY-02 for live re-encryption vs. maintenance window decision.

**CLOUDFLARE_API_KEY, WORKOS_API_KEY, ANTHROPIC_API_KEY, SENTRY_DSN (API keys, 90–180d):** Lower ceremony. Generate new credential in provider dashboard → deploy to Cloudflare Workers Secrets → redeploy affected Workers → confirm service functionality via Sentry → revoke old credential in provider dashboard after confirming new key is live → emit `admin.encryption_key_rotated`.

#### Communication Protocol

**Planned-rotation-overdue (no compromise suspected):** Internal-only. No enterprise tenant notification unless KEYPOINTS_ENC_KEY requires a maintenance window (Template K-01). Compliance-officer notified at T+30 min for P0. Founder at T+1h if key remains unrotated.

**Template K-01 — KEYPOINTS_ENC_KEY Maintenance Window (enterprise tenants)**

```
Subject: FORM · Scheduled Maintenance — [DATE] [TIME UTC]

Hi [Customer Name],

We are scheduling a maintenance window on [DATE] at [START TIME UTC]
(estimated duration: [DURATION]) for routine security maintenance including
a scheduled cryptographic key rotation.

The FORM service will be unavailable for approximately [DURATION].
No action is required from your team. Your data remains secure.

We will send a follow-up confirmation when complete.

The FORM Team
```

**Compromise-suspected escalation:** If C2 shows `admin.encryption_key_rotated` without a preceding `admin.encryption_key_rotation_initiated` within 1 hour — STOP. This is CRYPTO-CHAIN-02. Activate R-05 and R-20 immediately. If AL-KEY-04 fires and verification continues to fail after two retries, suspect compromise — activate R-05 and notify compliance-officer immediately.

#### DEC-030 Audit Events

All events HMAC-chained per `docs/AUDIT_LOG_SCHEMA.md`. Planned-rotation-failure events use 3-year retention; events with `rotated_by` or two-person auth (HMAC and KEYPOINTS_ENC_KEY) use 7-year retention.

| Event Type | Severity | Retention | Key Metadata Fields |
|---|---|---|---|
| `admin.key_rotation_incident_opened` | HIGH | 3 years | `affected_key_id`, `trigger_alert_id` (AL-KEY-01/02/03/04 or `manual`), `ic_user_id`, `days_overdue`, `incident_slug` |
| `admin.encryption_key_rotation_initiated` | HIGH | 7 years | `key_id`, `key_version_new`, `initiated_by_user_id`, `rotation_type` (`planned` / `emergency`), `authorised_by_user_id` (second approver for HMAC / KEYPOINTS), `incident_slug` |
| `admin.encryption_key_rotated` | HIGH | 7 years | `key_id`, `key_version_old`, `key_version_new`, `rotated_by_user_id`, `rotation_duration_seconds`, `incident_slug` |
| `admin.encryption_key_rotation_verified` | HIGH | 7 years | `key_id`, `key_version`, `verified_by_user_id`, `chain_integrity_checked` (bool), `verification_method`, `incident_slug` |
| `admin.key_rotation_incident_closed` | STANDARD | 3 years | `affected_key_id`, `resolution` (`rotated_successfully` / `scheduled_for_maintenance_window`), `compromise_suspected` (bool), `incident_slug` |

```typescript
// DEC-030 emission for incident open — emitted BEFORE any rotation action
const event = {
  event_type: 'admin.key_rotation_incident_opened',
  severity: 'HIGH',
  incident_id: incidentId,
  payload: {
    affected_key_id: affectedKeyId,       // e.g., 'HMAC_AUDIT_CHAIN_KEY'
    trigger_alert_id: triggerAlertId,     // 'AL-KEY-01'|'AL-KEY-02'|'AL-KEY-03'|'AL-KEY-04'|'manual'
    ic_user_id: icUserId,
    days_overdue: daysOverdue,            // 0 if AL-KEY-01 (not yet overdue)
    incident_slug: incidentSlug,
  },
};
const emitted = await emitDec030Event(event, supabaseAdminClient);
if (!emitted.success) {
  throw new Error('DEC-030 emission failed — key rotation incident aborted; retry required');
}
```

#### Evidence Package

| Evidence ID | Artefact | Collection Method | Location |
|---|---|---|---|
| **IR-KEY-E-001** | C1 form-crypto-health KV snapshot | `curl` output saved as JSON; SHA-256 recorded at T+15 min | `compliance/evidence/incidents/<slug>/c1-crypto-health-kv.json` |
| **IR-KEY-E-002** | C2 audit_log key rotation event history | SQL output exported as JSONL; shows full initiated → rotated → verified chain for the affected key | `compliance/evidence/incidents/<slug>/c2-rotation-history.jsonl` |
| **IR-KEY-E-003** | C3 scheduler health record | KV last-run value + Sentry Worker error extract | `compliance/evidence/incidents/<slug>/c3-scheduler-health.json` |
| **IR-KEY-E-004** | C4 HMAC cadence query (AL-KEY-03 only) | SQL output with `age_days` value; primary CC5.3 cadence evidence | `compliance/evidence/incidents/<slug>/c4-hmac-cadence.jsonl` |
| **IR-KEY-E-005** | Rotation completion confirmation | DEC-030 `admin.encryption_key_rotated` + `admin.encryption_key_rotation_verified` events as JSON; confirms triple on-chain | `compliance/evidence/incidents/<slug>/rotation-completion.json` |
| **IR-KEY-E-006** | Two-person authorisation record | DEC-030 `admin.encryption_key_rotation_initiated` with both `initiated_by_user_id` and `authorised_by_user_id`; required for HMAC and KEYPOINTS_ENC_KEY rotations | `compliance/evidence/incidents/<slug>/two-person-auth.json` |

All evidence under `compliance/evidence/incidents/<slug>/` with SHA-256 manifest. 3-year retention for IR-KEY-E-001 through IR-KEY-E-006 (7-year for IR-KEY-E-005 and IR-KEY-E-006 when HMAC_AUDIT_CHAIN_KEY is affected, given its audit-integrity role under SOC 2 CC8.1).

#### SOC 2 TSC Mapping

| TSC Criterion | Control | Evidence from This Runbook |
|---|---|---|
| **CC5.2** (Policies communicated) | Key rotation schedule in CRYPTOGRAPHY_POLICY.md §5 and OBSERVABILITY.md §30 KEY-SLO-01; responsible parties per §30.1 | IR-KEY-E-002: rotation event history demonstrates the schedule is actively monitored |
| **CC5.3** (Control activities performed as designed) | KEY-SLO-01: all keys rotated within 110% of schedule; AL-KEY-02 P0 triggers this runbook within 15 min; verified by `admin.encryption_key_rotation_verified` | IR-KEY-E-005: rotation completion as primary CC5.3 evidence |
| **CC6.7** (Encryption at rest and in transit) | KEYPOINTS_ENC_KEY rotation maintains encryption-at-rest for Art. 9 biometric data; SUPABASE_SERVICE_ROLE_JWT rotation maintains JWT signing integrity | IR-KEY-E-005: `admin.encryption_key_rotated` event for KEYPOINTS or SERVICE_ROLE_JWT |
| **CC6.8** (Cryptographic key protection) | Two-person auth for HMAC and KEYPOINTS rotations enforces separation of duties; incident runbook demonstrates active management of the eight-key production inventory | IR-KEY-E-006: two-person auth record; IR-KEY-E-001: KV monitoring state snapshot |
| **C1.1** (Privacy commitments maintained) | KEYPOINTS_ENC_KEY rotation protects Art. 9 biometric data per GDPR Art. 5(1)(f); maintenance window notice Template K-01 as transparency artefact | IR-KEY-E-005 for KEYPOINTS_ENC_KEY; Template K-01 customer communication record |
| **CC7.2** (Monitoring for system events) | AL-KEY-01 through AL-KEY-04 as documented automated monitoring; OBSERVABILITY.md §30.6 DEC-030 chain monitors as continuous verification | IR-KEY-E-001 and IR-KEY-E-003: evidence that monitoring controls were operating at incident time |
| **CC7.3** (Anomaly evaluation) | C1–C4 scope assessment procedure; CRYPTO-CHAIN-02 escalation to R-05 + R-20 draws the boundary between operational failure and security incident | IR-KEY-E-001 through IR-KEY-E-004: scope assessment evidence with incident slug traceability |

#### Open Questions

**OQ-KEY-01: Should `admin.key_rotation_incident_opened` be a new DEC-030 event type, or tracked only in the PagerDuty timeline?**

Adding this event to the DEC-030 chain links the alert-to-resolution evidence package for auditors. However, it requires registration in `docs/AUDIT_LOG_SCHEMA.md` before any production incident can use it. The alternative (PagerDuty-only) loses the HMAC-chained incident-to-resolution link. Recommendation: register the event type as part of the R-21 implementation checklist item 1. Owner: security-engineer + compliance-officer. Priority: **P0 — must be registered before R-21 is used in a production incident.** Target: M7.

**OQ-KEY-02: For KEYPOINTS_ENC_KEY rotation, is live re-encryption (dual-key overlap) or a maintenance window required?**

Live re-encryption avoids service disruption but expands the attack surface during the dual-key overlap period. For Art. 9 biometric data, the conservative position is a maintenance window. Enterprise customers with SLA commitments must be notified ≥72h in advance. Decision required before the first KEYPOINTS_ENC_KEY rotation (Month 13). Owner: security-engineer + enterprise-architect + compliance-officer. Priority: **P1.** Target: M12.

#### Implementation Checklist

| # | Action | Priority | Milestone | Owner | Status |
|---|---|---|---|---|---|
| 1 | Register `admin.key_rotation_incident_opened` and `admin.key_rotation_incident_closed` as new DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md`; validate Zod schema; deploy to `emit-audit-event` Worker endpoint; close OQ-KEY-01 | P0 | M7 | security-engineer | [ ] |
| 2 | Add `runbook: "docs/INCIDENT_RESPONSE.md#r-21"` to AL-KEY-02 and AL-KEY-04 PagerDuty alert configs in OBSERVABILITY.md §30.4 and §30.5 §6.2 | P0 | M7 | devops-lead | [ ] |
| 3 | Implement key-rotation-scheduler Worker and confirm AL-KEY-01/02 fire against a staging synthetic overdue-key scenario (per OBSERVABILITY.md §30.10 item 1) | P0 | M4 | platform-engineer + devops-lead | [ ] |
| 4 | Write R-21 tabletop exercise: simulate AL-KEY-02 P0 for SUPABASE_SERVICE_ROLE_JWT; IC opens incident channel; C1–C4 queries run; rotation executed; DEC-030 events emitted; IR-KEY-E-001 through IR-KEY-E-006 collected; PIR completed; measure elapsed time vs. 2-hour rotation SLA | P1 | M8 | security-engineer | [ ] |
| 5 | Resolve OQ-KEY-02 (KEYPOINTS_ENC_KEY live re-encryption vs. maintenance window) and document the decision in DECISION_LOG.md before M12 | P1 | M12 | security-engineer + enterprise-architect + compliance-officer | [ ] |
| 6 | Update Appendix A quick reference: add "Key rotation overdue / AL-KEY-02?" → R-21; distinguish from "Audit chain break or unauthorized rotation (CRYPTO-CHAIN-02)" → R-05 + R-20 | P1 | M7 | security-engineer | [ ] |
| 7 | File IR-KEY-E-001 through IR-KEY-E-006 evidence templates in `compliance/evidence/templates/key-rotation/` with SHA-256 checksum instructions | P2 | M7 | compliance-officer | [ ] |

---

---

### R-22: Enterprise Admin Dashboard — Individual User Data Exposure (Privacy Floor Breach)

**Owner:** compliance-officer + clinical-safety · **Reviewer:** security-engineer + enterprise-architect

> **Scope:** Covers any scenario in which an enterprise tenant admin (HR lead, People team, tenant_owner or tenant_admin role) gains visibility into **individual employee health, coaching, or biometric data** through the FORM Admin Dashboard, API, or data export pipeline — in violation of the non-negotiable privacy floor defined in `docs/ENTERPRISE.md §"Privacy floor for enterprise"`. This runbook is NOT R-01 (external breach) — the exposing party is an authorised tenant admin user, not an external attacker. It is NOT R-11 (CV biometric) — this runbook activates when aggregate-only controls fail at the reporting layer, not at the encryption layer. Parallel activation with R-01 is required only if the individual data was sent to a third party outside the tenant.

#### Privacy Floor Non-Negotiables

The following seven controls are non-negotiable per `ENTERPRISE.md`. This runbook activates when ANY of them fails:

1. HR can never see individual user data — aggregate only
2. HR sees aggregates only: % active, % activated, opt-in NPS
3. Manager-reports on direct reports are never available
4. Employee can revoke company-association at any time
5. ED-screening data is never aggregated to employer (clinical-safety floor)
6. Body composition data is never aggregated to employer (clinical-safety floor)
7. Mental health data is never aggregated to employer (clinical-safety floor)

**Rules 5, 6, 7 apply even at the aggregate level.** An employer seeing "23% of employees screen positive for ED risk" is a floor violation. The employer sees only: "% activated", "% weekly active", and NPS aggregate — nothing further.

#### Trigger

| Source | Alert ID | Condition |
|---|---|---|
| `audit_log_events` monitor | `FORM-PRIV-001` | `data.read_individual` event emitted for `actor_type = 'user'` AND actor has `tenant_admin` or `tenant_owner` role — forbidden per `AUDIT_LOG_SCHEMA.md`; alert fires in real-time, no threshold |
| Admin Dashboard API gateway | `FORM-PRIV-002` | Any HTTP response to `/api/admin/v1/users/*` route returning `user_id`-level health fields (`fitness_level`, `health_goal`, `coaching_turns`, `cv_sessions`, `meal_logs`, `weight`, `body_fat_pct`) for a `tenant_admin` actor — these fields are stripped at the API serialisation layer; a non-empty response indicates a serialiser bypass |
| Aggregate reporting SQL monitor | `FORM-PRIV-003` | `enterprise_admin_reports.user_count_in_cohort < 5` AND `employer_visible = true` — k-anonymity floor violation; `DATA_MODEL.md §17.4` requires minimum cohort size ≥ 5 before any aggregate is returned |
| Clinical-safety flag | `FORM-PRIV-004` | Any `cohort_label` containing `ed_screening`, `body_composition`, `mental_health`, `bmi`, `weight_trend`, or `calories_*` in a tenant-visible aggregate — unconditional floor violation regardless of cohort size |
| Customer report | — | Tenant admin contacts CSM or support reporting they can see individual employee data, specific names, or health metrics they did not expect |
| Code review finding | — | PR diff reveals a missing `aggregate_only = true` filter, a missing `WHERE employer_visible = true` clause, or a serialiser that includes `user_id` in a tenant-admin-accessible response |

#### Severity Classification

| Condition | Severity | Rationale |
|---|---|---|
| Any individual user's coaching turns, CV keypoints, meal logs, or health_profile fields visible to tenant admin | **P0** | Art. 9 special category health data exposed to an unauthorised processor (the employer); GDPR Art. 33 clock starts immediately |
| ED screening, body composition, or mental health aggregate visible to employer (rules 5–7 breach) | **P0** | Clinical-safety floor breached; even aggregate exposure of these categories violates ENTERPRISE.md unconditional rule; clinical-safety VETO activates |
| Individual user_id or user name mapped to any wellness metric in employer-visible report | **P1** | Privacy floor breach even if metric is non-Art. 9; identity linkage makes aggregate into individual data under GDPR Recital 26; assess Art. 33 threshold |
| k-anonymity cohort < 5 exposed in employer-facing report (FORM-PRIV-003) | **P1** | Quasi-individual exposure; may not meet Art. 33 threshold but must be assessed; suppress the offending cohort immediately |
| API serialiser returned `user_id` field in tenant-admin response (FORM-PRIV-002) but no health data in the response body | **P2** | Serialiser bug without confirmed health data exposure; suppress and fix; assess downstream use of the user_id |
| Code review finding only — no evidence of production exposure | **P3** | Preventive fix before any user affected; no notification obligations; PIR and code fix required |

**P0 upgrade trigger:** If FORM-PRIV-001 fires AND the actor is a `tenant_owner` with a commercial wellness incentive contract (wellness-as-punishment no-go flag) — upgrade to P0 regardless of data category and activate no-go escalation protocol (§R-22.9).

#### Scope Assessment Queries

Run all four queries immediately. Results determine GDPR Art. 33 obligation and communication scope.

**C1 — Identify the exposure window and affected actor**

```sql
-- Determine when the privacy floor breach started and which tenant admin triggered it
SELECT
  e.id              AS event_id,
  e.tenant_id,
  e.actor_id,
  e.action,
  e.resource_type,
  e.resource_id,
  e.metadata,
  e.created_at,
  e.hmac_self
FROM audit_log_events e
WHERE e.action IN ('data.read_individual', 'data.export_initiated', 'data.read_aggregate')
  AND e.outcome = 'success'
  AND e.created_at >= NOW() - INTERVAL '72 hours'  -- extend if code review finding suggests longer window
  AND e.actor_id IN (
    SELECT user_id
    FROM tenant_members
    WHERE tenant_id = '<affected_tenant_id>'
      AND role IN ('tenant_owner', 'tenant_admin', 'tenant_manager')
  )
ORDER BY e.created_at ASC;
```

**C2 — Identify which employee user_ids were exposed and whether Art. 9 data is involved**

```sql
-- Map exposed resource_ids to user health categories
-- Run only if C1 returns resource_type = 'user' or 'health_profile' rows
SELECT
  r.user_id,
  hp.fitness_level       IS NOT NULL AS has_fitness_profile,
  hp.health_goal         IS NOT NULL AS has_health_goal,
  COUNT(ct.id)           > 0         AS has_coaching_turns,
  COUNT(cs.id)           > 0         AS has_cv_sessions,
  COUNT(ml.id)           > 0         AS has_meal_logs
FROM (
  SELECT DISTINCT (metadata->>'exposed_user_id')::uuid AS user_id
  FROM audit_log_events
  WHERE id IN (<event_ids_from_C1>)
    AND metadata ? 'exposed_user_id'
) r
LEFT JOIN health_profiles hp ON hp.user_id = r.user_id
LEFT JOIN coaching_turns ct  ON ct.user_id = r.user_id AND ct.created_at >= '<exposure_window_start>'
LEFT JOIN cv_sessions cs     ON cs.user_id = r.user_id AND cs.created_at >= '<exposure_window_start>'
LEFT JOIN meal_logs ml       ON ml.user_id = r.user_id AND ml.created_at >= '<exposure_window_start>'
GROUP BY r.user_id, hp.fitness_level, hp.health_goal;
-- If coaching_turns or cv_sessions IS TRUE for any row → Art. 9 confirmed → P0 + Art. 33 clock
-- If meal_logs IS TRUE → assess Art. 9 (nutrition data may constitute health data under GDPR)
```

**C3 — Check for k-anonymity floor violations in employer-visible reports**

```sql
-- Find cohorts below the k=5 threshold that were returned to employer-facing reports
SELECT
  r.tenant_id,
  r.cohort_label,
  r.user_count_in_cohort,
  r.employer_visible,
  r.generated_at,
  r.requested_by_actor_id
FROM enterprise_admin_reports r
WHERE r.tenant_id = '<affected_tenant_id>'
  AND r.employer_visible = true
  AND r.user_count_in_cohort < 5
  AND r.generated_at >= '<exposure_window_start>'
ORDER BY r.generated_at ASC;
-- Any returned rows = k-anonymity floor violated; assess if cohort_label is Art. 9 (rules 5–7)
```

**C4 — Clinical-safety flag: check for forbidden aggregate labels**

```sql
-- Detect if any employer-visible cohort was labelled with a clinical-safety protected category
SELECT
  r.tenant_id,
  r.cohort_label,
  r.generated_at,
  r.employer_visible,
  r.metric_value
FROM enterprise_admin_reports r
WHERE r.tenant_id = '<affected_tenant_id>'
  AND r.employer_visible = true
  AND r.generated_at >= '<exposure_window_start>'
  AND (
    r.cohort_label ILIKE '%ed_screening%'       OR
    r.cohort_label ILIKE '%eating_disorder%'     OR
    r.cohort_label ILIKE '%body_composition%'    OR
    r.cohort_label ILIKE '%body_fat%'            OR
    r.cohort_label ILIKE '%bmi%'                 OR
    r.cohort_label ILIKE '%weight%'              OR
    r.cohort_label ILIKE '%mental_health%'       OR
    r.cohort_label ILIKE '%depression%'          OR
    r.cohort_label ILIKE '%anxiety%'             OR
    r.cohort_label ILIKE '%calories%'
  )
ORDER BY r.generated_at ASC;
-- Any returned rows → clinical-safety floor breach → P0 regardless of cohort size
-- Notify clinical-safety immediately; they have VETO on any re-enable decision
```

#### Immediate Actions (T+0 to T+20 min)

```
1. Open incident channel: #inc-YYYYMMDD-privacy-floor
   IC: compliance-officer (primary) or clinical-safety (if rules 5–7 breach confirmed)
   Notify immediately: security-engineer + clinical-safety + founder

2. Classify the breach source (pick one before proceeding):
   A) API serialiser bug (FORM-PRIV-002) — individual fields returned in API response
      → Immediately set feature flag: admin_dashboard_user_lookup_enabled = false
      → Block the specific API endpoint at Cloudflare WAF until fix deployed
   B) Aggregate report query bug (FORM-PRIV-003, k-anonymity < 5)
      → Set feature flag: enterprise_reporting_enabled = false for affected tenant
   C) Forbidden cohort label (FORM-PRIV-004, clinical-safety rules 5–7)
      → Set feature flag: enterprise_reporting_enabled = false for ALL tenants
      → Page clinical-safety: VETO activation required before re-enable
   D) No-code detection (customer report, code review — P3)
      → No feature flag change required; assess scope before acting

3. Run scope assessment C1, C2, C3, C4 in READ-ONLY mode
   Do NOT DELETE any evidence rows — audit log is forensic record
   Record all query results with timestamps in incident channel

4. If C2 confirms Art. 9 data (coaching_turns, cv_sessions, meal_logs): P0 upgrade
   → Start GDPR Art. 33 72h clock: exposure time from C1 results, NOT current time
   → Notify compliance-officer to draft Art. 33 assessment (R-15 GDPR runbook §15.3)

5. If C4 returns any rows: clinical-safety breach
   → clinical-safety has unconditional VETO on dashboard re-enable
   → Do NOT re-enable enterprise_reporting_enabled without clinical-safety sign-off
   → Founder must be informed within 30 min (board trigger threshold per §13 applies
     if any enterprise tenant is at risk of termination)
```

#### Containment

**Phase 1 — Immediate suppression (T+0 to T+20 min)**

| Action | Feature flag / mechanism | When to apply |
|---|---|---|
| Disable admin user-level lookup | `admin_dashboard_user_lookup_enabled = false` | API serialiser bug (Scenario A) |
| Disable enterprise reporting for affected tenant | `enterprise_reporting_enabled = false` (per-tenant scope) | k-anonymity or cohort label violation (Scenarios B/D) |
| Disable enterprise reporting globally | `enterprise_reporting_enabled = false` (global scope) | Clinical-safety rules 5–7 breach (Scenario C); reinstate only with clinical-safety sign-off |
| Block API endpoint at WAF | Cloudflare WAF custom rule: `(http.request.uri.path contains "/api/admin/v1/users/") AND (cf.threat_score > 0 OR http.request.method eq "GET")` | Scenario A only; remove when serialiser fix deployed |

**Phase 2 — Root cause identification (T+20 to T+60 min)**

1. Review the relevant component (IC + platform-engineer, read-only):
   - For Scenario A: `src/workers/admin-api/serialisers/user-aggregate.ts` — confirm `employer_visible` filter is applied before serialisation; check if `user_id` field is stripped
   - For Scenario B: `src/workers/admin-api/reports/aggregate-query.ts` — confirm `HAVING COUNT(user_id) >= 5` is applied after `GROUP BY cohort_label`
   - For Scenario C: `src/workers/admin-api/reports/cohort-labels.ts` — confirm `FORBIDDEN_COHORT_LABELS` blocklist is applied to all employer-visible reports
   - For Scenario D (code review): the specific PR or commit that introduced the regression

2. Capture the exact diff that caused the regression (git blame on affected file; cross-reference with recent PR merges since last known-good date)

3. Do NOT push a fix to production without:
   - Two-person review (security-engineer + compliance-officer)
   - Passing the privacy floor test suite (`tests/enterprise/privacy-floor.spec.ts`)
   - IC approval before deployment

**Phase 3 — Eradication and re-enable (T+60 min to resolution)**

1. Deploy the fix (requires two-person review and privacy floor test suite pass)
2. Run C1, C2, C3, C4 queries again post-fix to confirm no further exposure
3. Re-enable feature flags in reverse suppression order: per-tenant before global
4. If Scenario C (clinical-safety): clinical-safety must run the C4 query independently and confirm zero rows before re-enabling `enterprise_reporting_enabled` globally
5. Log re-enable decision in incident channel with timestamp and approver user_ids

#### Communication Protocol

**Enterprise tenant notification — Template PF-01 (required for P0, optional for P1 if employee data involved)**

Send within:
- P0: 30 minutes of declaration
- P1 (employee identity linked to data): 60 minutes
- P1 (k-anonymity floor only, no identity linkage): 2 hours
- P2: next business day

Delivery: encrypted email to `tenant_admin` registered contact + CSM Slack notification

```
Subject: Important security notice regarding your FORM dashboard

Hi [Tenant Admin Name],

We are writing to inform you that between [START_TIME] and [END_TIME] UTC,
a configuration issue in FORM's Admin Dashboard may have presented wellness
data for individual employees in a format that does not meet FORM's
privacy floor commitments for enterprise customers.

[IF SCENARIO A/B]:
Specifically, [N] admin report(s) generated during this window may have
shown wellness metrics that were not properly aggregated. No personally
identifiable health data such as diagnostic information, body composition
specifics, or mental health indicators was involved.

[IF SCENARIO C]:
Specifically, [N] admin report(s) generated during this window may have
shown cohort labels for categories that FORM does not make visible to
employers under any circumstances. We are treating this as a P0 incident.

We have immediately suppressed the affected dashboard capability and are
deploying a fix. The capability will be restored only after independent
verification by our Clinical Safety team.

Affected window: [START_TIME] – [END_TIME] UTC
Affected report type(s): [COHORT_LABELS or API_ENDPOINTS]
Individual employee data accessed: [YES (N employees) / NO — aggregate only]

[IF Art. 9 health data confirmed]:
We have a legal obligation to inform you as the data controller that
special category health data (Article 9 GDPR) was accessed in a manner
inconsistent with our Data Processing Agreement. Our compliance team
will be in contact within 4 hours to discuss our Article 33 assessment
and any joint notification obligations.

Action required from your side: None at this time. Please ask your admin
team to avoid generating dashboard reports until you receive our "all clear"
notification.

We apologise for this incident. Your CSM [CSM NAME] will follow up
within [30 min for P0 / 60 min for P1].

The FORM Enterprise Team
```

**Employee notification — Template PF-02 (required if C2 confirms Art. 9 exposure to employer)**

FORM acts as data processor; the enterprise tenant (employer) is data controller for their employees' data. The Art. 34 individual notification decision lies with the controller.

- Advise the tenant admin (controller) in writing within 2 hours of Art. 9 confirmation
- Provide: number of affected employees (not names — privacy floor applies even in breach remediation), categories of data, exposure window
- The employer decides whether to notify individual employees under Art. 34 — FORM does not notify employees directly without controller instruction
- Exception: if the employer cannot be reached within 4 hours or is suspected of acting in bad faith (no-go escalation per §R-22.9), compliance-officer + founder decide on direct notification with legal counsel

**Board / investor notification — §13 trigger assessment**

| Condition | Threshold | Action |
|---|---|---|
| P0 with confirmed Art. 9 exposure to employer | Always | §13 board notification; Template B-01 within 4h |
| Enterprise tenant issues formal complaint or terminates | Any severity | §13 board notification |
| Wellness-as-punishment use confirmed (§R-22.9) | P0 upgrade | §13 board notification + legal counsel immediately |

#### No-Go Escalation Protocol (§R-22.9)

ENTERPRISE.md defines four wellness-as-punishment no-go scenarios. If this incident reveals that a customer is **intentionally using individual data to evaluate, discipline, or incentivise employees** — the following protocol overrides normal incident response:

```
1. Suspend access to FORM Admin Dashboard for the affected tenant immediately
   (not a feature flag — revoke tenant admin sessions via SSO revocation endpoint)

2. Notify founder within 15 minutes — founder has sole authority to proceed

3. Founder reviews contract + DPA for "no individual data" obligation
   → If contract clearly prohibits: initiate contract termination
   → If contract is ambiguous: outside counsel within 1 business day before any further action

4. Do NOT provide the tenant with individual data "for compliance" or "for HR records"
   → clinical-safety + compliance-officer have VETO: any individual data disclosure
     to an employer for performance management purposes is an unconditional no-go
   → Losing the deal is preferable to enabling harm (ENTERPRISE.md §"When we say no")

5. If the no-go concern is credible but not confirmed:
   → compliance-officer documents the concern in DECISION_LOG.md with timestamp
   → Increase monitoring cadence to 24h for this tenant
   → Legal counsel notified within 48h
```

#### DEC-030 HMAC-Chained Audit Events

All events are HMAC-chained per `docs/AUDIT_LOG_SCHEMA.md`. Failure to emit any P0 event aborts the corresponding response action.

| Event Type | Severity | Retention | Key Metadata Fields |
|---|---|---|---|
| `privacy.floor_breach_detected` | CRITICAL | 7 years | `tenant_id`, `breach_source` (`api_serialiser` / `aggregate_query` / `cohort_label` / `code_review`), `scenario` (`A` / `B` / `C` / `D`), `clinical_safety_rules_violated` (bool), `art9_confirmed` (bool), `exposure_window_start`, `affected_employee_count` (integer or `unknown_pending_assessment`), `ic_user_id`, `incident_slug` |
| `privacy.floor_breach_contained` | HIGH | 7 years | `tenant_id`, `containment_method` (`feature_flag` / `waf_rule` / `session_revoke`), `feature_flags_disabled` (array), `contained_by_user_id`, `incident_slug` |
| `privacy.floor_breach_tenant_notified` | HIGH | 7 years | `tenant_id`, `notification_template` (`PF-01` / `PF-02`), `notified_at`, `notified_by_user_id`, `employee_count_disclosed_to_tenant`, `incident_slug` |
| `privacy.floor_breach_resolved` | STANDARD | 7 years | `tenant_id`, `fix_commit_sha`, `fix_deployed_at`, `feature_flags_re_enabled` (array), `clinical_safety_approved` (bool), `two_person_approvers` (array of user_ids), `incident_slug` |
| `privacy.no_go_escalation_activated` | CRITICAL | 7 years | `tenant_id`, `no_go_reason` (`wellness_as_punishment` / `individual_data_demand` / `manager_report_demand` / `insurance_risk_scoring`), `dashboard_suspended_at`, `founder_notified_at`, `legal_counsel_engaged` (bool), `incident_slug` |

```typescript
// DEC-030 emission — privacy floor breach detected
// Must be emitted BEFORE any containment action (feature flag changes)
const event = {
  event_type: 'privacy.floor_breach_detected',
  severity: 'CRITICAL',
  incident_id: incidentId,
  payload: {
    tenant_id:                    affectedTenantId,
    breach_source:                breachSource,          // 'api_serialiser' | 'aggregate_query' | 'cohort_label' | 'code_review'
    scenario:                     scenario,              // 'A' | 'B' | 'C' | 'D'
    clinical_safety_rules_violated: clinicalRulesViolated, // true if rules 5/6/7 breached
    art9_confirmed:               art9Confirmed,         // pending C2 query result at detection time; update with privacy.floor_breach_assessment event
    exposure_window_start:        exposureWindowStart,   // ISO 8601 UTC; from C1 query
    affected_employee_count:      affectedCount,         // integer | 'unknown_pending_assessment'
    ic_user_id:                   icUserId,
    incident_slug:                incidentSlug,
  },
};
const emitted = await emitDec030Event(event, supabaseAdminClient);
if (!emitted.success) {
  throw new Error('DEC-030 emission failed — privacy floor breach detection event aborted; retry required before feature flag changes');
}
```

#### Evidence Package

| Evidence ID | Artefact | Collection Method | Location |
|---|---|---|---|
| **IR-PF-E-001** | C1 audit_log_events query — exposure timeline | SQL output saved as JSONL; SHA-256 recorded; includes hmac_self for each row | `compliance/evidence/incidents/<slug>/c1-audit-timeline.jsonl` |
| **IR-PF-E-002** | C2 employee exposure map | SQL output as CSV (user_id column only — no names; privacy floor applies to evidence collection); indicates Art. 9 data categories present | `compliance/evidence/incidents/<slug>/c2-exposure-map.csv` |
| **IR-PF-E-003** | C3 k-anonymity floor violations | SQL output as CSV; cohort_label + user_count_in_cohort + generated_at | `compliance/evidence/incidents/<slug>/c3-k-anonymity-violations.csv` |
| **IR-PF-E-004** | C4 clinical-safety cohort labels | SQL output as CSV; cohort_label + employer_visible + generated_at | `compliance/evidence/incidents/<slug>/c4-clinical-safety-labels.csv` |
| **IR-PF-E-005** | Feature flag state — suppression record | Cloudflare Feature Flags audit log export; shows `enterprise_reporting_enabled` transition to `false` with actor and timestamp | `compliance/evidence/incidents/<slug>/feature-flag-suppression.json` |
| **IR-PF-E-006** | Root cause diff | Git diff of the offending commit; PR link; privacy floor test suite results (before and after fix) | `compliance/evidence/incidents/<slug>/root-cause-diff.patch` |
| **IR-PF-E-007** | Tenant notification record | Template PF-01 email (sent copy); delivery receipt from email service; CSM Slack DM transcript | `compliance/evidence/incidents/<slug>/tenant-notification/` |
| **IR-PF-E-008** | Clinical-safety sign-off | clinical-safety written approval (email or DEC-030 `privacy.floor_breach_resolved` event with `clinical_safety_approved: true`) | `compliance/evidence/incidents/<slug>/clinical-safety-sign-off.eml` |

All evidence under `compliance/evidence/incidents/<slug>/` with SHA-256 manifest. 7-year retention per DEC-030 for IR-PF-E-001 through IR-PF-E-008 (7 years is the standard for Art. 9 processing records under GDPR Art. 30).

**Privacy rule for evidence collection:** IR-PF-E-002 (employee exposure map) contains only `user_id` (UUID) — never names, emails, or health values. The UUID mapping to a named employee is retrievable by the employer (data controller) but FORM does not compile or retain the mapping in evidence. Evidence collection itself must not violate the privacy floor.

#### SOC 2 TSC Mapping

| TSC Criterion | Control | Evidence from This Runbook |
|---|---|---|
| **P6.1** (Personal information access and disclosure limited) | Enterprise admin dashboard restricted to aggregate-only data per `DATA_MODEL.md §17`; `employer_visible = true` filter enforced at API serialisation layer; k-anonymity floor k ≥ 5 | IR-PF-E-006: fix diff demonstrates the serialiser and aggregate controls are in place; IR-PF-E-005: feature flag suppression shows rapid containment |
| **P6.7** (Personal information disclosed only to authorised parties) | `data.read_individual` action is blocked for `tenant_admin` role at API middleware layer; AUDIT_LOG_SCHEMA.md defines this as a forbidden action with alert-on-attempt | IR-PF-E-001: audit log showing zero `data.read_individual` success events post-fix; C1 query establishes exposure window |
| **P7.1** (Personal information quality maintained) | Aggregate reporting accuracy maintained through k-anonymity floor and cohort label blocklist; clinical-safety rules 5–7 protect against systematically misleading employer wellness data | IR-PF-E-003 + IR-PF-E-004: k-anonymity and cohort label evidence; IR-PF-E-008: clinical-safety sign-off |
| **CC6.1** (Logical access controls) | Admin dashboard API enforces role-based access at serialisation layer; `tenant_admin` role cannot access individual user records via any documented API path | IR-PF-E-001 + IR-PF-E-006: combined audit evidence and code fix demonstrating the access control is designed and operating effectively |
| **CC6.6** (Unauthorised access prevented) | Monitoring alert FORM-PRIV-001/002/003/004 provides real-time detection; feature flag suppression provides immediate containment within 20 min of detection | IR-PF-E-005: containment timeline evidence |
| **CC7.2** (System events monitored) | Four alert rules FORM-PRIV-001 through FORM-PRIV-004 provide continuous monitoring at audit log, API gateway, aggregate report, and cohort label layers | IR-PF-E-001: audit event triggered the alert; DEC-030 chain provides continuous monitoring evidence |
| **CC7.4** (Incident response program) | Runbook R-22 documents the complete response procedure including containment, evidence, communication, no-go escalation, and SOC 2 evidence mapping | This runbook is itself CC7.4 evidence; IR-PF-E-007: tenant notification demonstrates the communication control |

#### Post-Incident Preventive Controls

| Control | Description | Owner | Priority |
|---|---|---|---|
| Privacy floor test suite — CI gate | `tests/enterprise/privacy-floor.spec.ts` must pass on every PR that touches admin API serialisers, aggregate report queries, or cohort label definitions; gate blocks merge on failure | platform-engineer + qa-lead | **P0 — before enterprise GA** |
| `data.read_individual` alert latency | FORM-PRIV-001 must fire within 60 seconds of the first forbidden `data.read_individual` event; test in staging with synthetic event | devops-lead | **P0 — before enterprise GA** |
| k-anonymity floor enforcement in SQL layer | `HAVING COUNT(DISTINCT user_id) >= 5` added as a DB-layer constraint (view or materialized view) rather than application layer only; DB layer cannot be bypassed by a serialiser bug | platform-engineer | **P1 — M6** |
| Cohort label blocklist review | Quarterly review of `FORBIDDEN_COHORT_LABELS` constant by clinical-safety; any new Victor coaching category must be assessed before adding to employer-visible reporting | clinical-safety | **P1 — quarterly** |
| No-go customer monitoring | Tenant contracts flagged with `wellness_as_punishment_risk = true` receive increased dashboard access monitoring (FORM-PRIV-001 with lower threshold: >0 events in 24h rather than real-time) | compliance-officer + customer-success | **P2 — before Year 1 enterprise deals** |

#### Open Questions

**OQ-PF-01: Should the privacy floor test suite be enforced in production as a runtime assertion or only in CI?**

A runtime assertion (middleware that re-validates `employer_visible` and k-anonymity before every API response) would catch regressions that bypass CI (e.g., a hotfix deployed without full test suite). Cost: adds ~2 ms to every admin dashboard API call. A CI-only gate is lighter but has a window between deploy and first use. Recommendation: CI gate as primary control; runtime assertion as belt-and-suspenders for Art. 9-adjacent endpoints only (`/api/admin/v1/reports/*`). Owner: platform-engineer + security-engineer. Priority: **P1 — before first enterprise pilot.** Target: M6.

**OQ-PF-02: Should clinical-safety rules 5–7 (ED screening, body composition, mental health) be enforced at the data classification layer in `DATA_MODEL.md §5` rather than only at the reporting layer?**

If the `data_classification` column of affected tables is tagged `clinical_safety_protected`, the API serialiser can enforce the rule independently of the cohort label blocklist — reducing the risk of a new coaching category bypassing the blocklist. Risk: adds a new data classification tier that must be maintained as new Victor features ship. Recommendation: implement as a `clinical_safety_protected BOOLEAN DEFAULT false` column on the `exercise_categories`, `coaching_turn_categories`, and `report_cohort_definitions` tables; annotate existing Art. 9 adjacent categories as `true`. Owner: clinical-safety + enterprise-architect. Priority: **P1 — before first enterprise pilot.** Target: M7. Cross-reference: DATA_MODEL.md §5 (Data Classification).

**OQ-PF-03: How should FORM respond if a customer's legal team argues that FORM's privacy floor conflicts with their legal obligation to monitor employee wellness under a local labour law?**

Some jurisdictions (e.g., certain EU member states with works council agreements) may give employers rights to aggregate wellness data that FORM's privacy floor currently denies. The privacy floor is a FORM non-negotiable, not a regulatory requirement on FORM's side — it reflects FORM's ethical commitment, not a legal obligation. If a customer claims a legal entitlement to data FORM will not provide, the correct response is: (a) confirm with outside counsel whether the customer's claim is valid under the applicable law; (b) if confirmed, assess whether serving the customer is consistent with FORM's ethics and no-go policy; (c) if not consistent, decline to expand data access and accept contract loss. No legal entitlement claimed by a customer can override clinical-safety's VETO on rules 5–7 data. Owner: compliance-officer + founder. Priority: **P1 — before first EU enterprise deal.** Target: M9.

#### Implementation Checklist

| # | Action | Priority | Milestone | Owner | Status |
|---|---|---|---|---|---|
| 1 | Implement FORM-PRIV-001 alert: Cloudflare Edge Function monitors `audit_log_events` in real-time for `data.read_individual` action with `tenant_admin` or `tenant_owner` actor; fires PagerDuty P1 immediately; confirm in staging with synthetic `data.read_individual` event for a `tenant_admin` actor | devops-lead + platform-engineer | **P0** | Before enterprise pilot | [ ] |
| 2 | Implement FORM-PRIV-002 alert: Admin API gateway middleware logs response body sample for `/api/admin/v1/users/*` routes when response includes health field names; fires PagerDuty P1 if any health field detected; confirm in staging | platform-engineer | **P0** | Before enterprise pilot | [ ] |
| 3 | Implement FORM-PRIV-003 alert: pg_cron job runs every 15 min; queries `enterprise_admin_reports` for `user_count_in_cohort < 5 AND employer_visible = true`; fires PagerDuty P1 on any result; confirm in staging with synthetic low-count cohort | platform-engineer + devops-lead | **P0** | Before enterprise pilot | [ ] |
| 4 | Implement FORM-PRIV-004 alert: pg_cron job runs every 15 min; queries `enterprise_admin_reports` for clinical-safety cohort labels in `employer_visible = true` rows; fires PagerDuty P0 on any result; notify clinical-safety directly via PagerDuty escalation policy | platform-engineer + devops-lead + clinical-safety | **P0** | Before enterprise pilot | [ ] |
| 5 | Write privacy floor test suite `tests/enterprise/privacy-floor.spec.ts`: 15+ test cases covering all four alert scenarios; add as CI gate on all PRs touching admin API serialisers and aggregate report code | qa-lead + platform-engineer | **P0** | Before enterprise pilot | [ ] |
| 6 | Register all five DEC-030 events (`privacy.floor_breach_detected`, `privacy.floor_breach_contained`, `privacy.floor_breach_tenant_notified`, `privacy.floor_breach_resolved`, `privacy.no_go_escalation_activated`) in `docs/AUDIT_LOG_SCHEMA.md` with full payload schema, severity, and retention; validate Zod schema in staging | platform-engineer + compliance-officer | **P0** | Before enterprise pilot | [x] |
| 7 | Add `runbook: "docs/INCIDENT_RESPONSE.md#r-22"` to FORM-PRIV-001 through FORM-PRIV-004 PagerDuty alert configs; confirm runbook URL resolves correctly | devops-lead | **P1** | M6 | [ ] |
| 8 | Add R-22 tabletop exercise to §9 drill catalog: Scenario J — tenant admin generates aggregate report; cohort for "ED screening awareness" label slips through blocklist due to synonym (`eating_habit_awareness`); FORM-PRIV-004 fires; clinical-safety VETO activated; global reporting suspended; 6 enterprise tenants affected; board notification threshold reached; 10 discussion points | clinical-safety + compliance-officer | **P1** | M9 | [ ] |
| 9 | Resolve OQ-PF-01 (runtime assertion vs CI-only for privacy floor) and document in DECISION_LOG.md | platform-engineer + security-engineer | **P1** | Before enterprise pilot (M6) | [ ] |
| 10 | Resolve OQ-PF-02 (data classification layer for clinical-safety rules 5–7) and update DATA_MODEL.md §5 if adopted | clinical-safety + enterprise-architect | **P1** | M7 | [ ] |
| 11 | Update Appendix A: add "Admin dashboard showing individual data?" → R-22; add "Wellness-as-punishment use suspected?" → R-22.9 no-go escalation | compliance-officer | **P0** | With this release | [x] |

---

*v1.8 additions (2026-06-09): §2.2.1 Phase 0 Permanent On-Call: Compensating Control Statement — closes the documentation action required by CC2-GAP-005 (`docs/SOC2_READINESS.md §30.4`). Auditor-facing acknowledgement that Phase 0 is a solo-founder permanent on-call arrangement, not a rotation. Four compensating controls documented: (1) PagerDuty critical alerts 24/7 on founder mobile overriding DND (CC2-E-004); (2) multi-escalation policy — T+3 min phone call, T+8 min SMS retry + second call, T+15 min out-of-band emergency contact paged + Slack #security-alerts auto-post (CC2-E-004); (3) Better Uptime independent failsafe — 30 s probes from three regions, 90 s failure = SMS + email to secondary address independent of PagerDuty (CC2-E-005); (4) HMAC audit chain dead-man's switch — pg_cron every 10 min, chain break = PagerDuty P0 on escalation policy with no silence possible (CC2-E-004). Acceptance statement: P1 risk accepted; no enterprise contract before Phase 1 or written customer acceptance. Phase 1 trigger checklist: five items to complete within 30 days of second engineering hire. Phase 1 rotation schedule placeholder: fields pre-populated for founding engineer + founder alternating weekly. Document header corrected v1.5 → v1.8 (v1.4 through v1.7 additions did not update the header). SOC 2 evidence: CC2.2 — external communication of on-call commitments; CC2-GAP-005 status updated from open → 🟡 Partial (documented; CC2-E-004 screenshot pending as an implementation task). Owner: security-engineer.*

*v1.7 additions (2026-06-05): R-22 §checklist item 6 closed — all five DEC-030 HMAC-chained privacy floor breach events (`privacy.floor_breach_detected` CRITICAL, `privacy.floor_breach_contained` HIGH, `privacy.floor_breach_tenant_notified` HIGH, `privacy.floor_breach_resolved` STANDARD, `privacy.no_go_escalation_activated` CRITICAL) registered in `docs/AUDIT_LOG_SCHEMA.md` v0.3 §"Privacy floor enforcement events" with full payload schema and retention. Cross-ref: AUDIT_LOG_SCHEMA.md v0.3 changelog.*

*v1.6 additions (2026-06-05): R-22 Enterprise Admin Dashboard — Individual User Data Exposure (Privacy Floor Breach) — twenty-second runbook; the first dedicated to FORM's enterprise privacy floor commitments. Fills the runbook gap between the product-level privacy floor in `docs/ENTERPRISE.md §"Privacy floor for enterprise"` (seven non-negotiable rules) and the technical enforcement in `DATA_MODEL.md §17` (aggregate-only admin reporting schema), `DATA_MODEL.md §5` (data classification), and `DATA_MODEL.md §6` (privacy floor enforcement). R-22 covers: seven privacy floor non-negotiable rules as the explicit trigger boundary; four alert sources FORM-PRIV-001 (audit event monitor), FORM-PRIV-002 (API serialiser field check), FORM-PRIV-003 (k-anonymity floor violation, k < 5), FORM-PRIV-004 (clinical-safety cohort labels — P0, unconditional for rules 5–7); six-row severity classification table (P0 for Art. 9 individual exposure or clinical-safety rules; P1 for identity-linkage; P2 for user_id serialiser without health data; P3 for code review finding only); four scope assessment SQL queries C1–C4 (audit event timeline, employee exposure map with Art. 9 categorisation, k-anonymity floor violations, clinical-safety cohort label scan); immediate actions four-path (Scenarios A/B/C/D) with feature flag suppression options; two-phase containment (immediate suppression then root-cause identification) and Phase 3 re-enable with clinical-safety VETO gate; communication Template PF-01 (tenant admin notification, P0 within 30 min) and Template PF-02 (employer controller notification for Art. 34 decision on employee notification); §R-22.9 no-go escalation protocol (wellness-as-punishment detection → immediate session revoke + founder notification + clinical-safety VETO + contract termination path); five DEC-030 HMAC-chained events (privacy.floor_breach_detected CRITICAL 7yr, privacy.floor_breach_contained HIGH 7yr, privacy.floor_breach_tenant_notified HIGH 7yr, privacy.floor_breach_resolved STANDARD 7yr, privacy.no_go_escalation_activated CRITICAL 7yr); evidence package IR-PF-E-001 through IR-PF-E-008 with privacy rule for evidence collection (user_id UUID only in IR-PF-E-002 — no names or health values compiled by FORM); SOC 2 mapping P6.1/P6.7/P7.1/CC6.1/CC6.6/CC7.2/CC7.4; five post-incident preventive controls (CI privacy floor test suite gate, FORM-PRIV-001 latency SLA, SQL-layer k-anonymity enforcement, quarterly cohort label review by clinical-safety, no-go customer monitoring); three open questions (OQ-PF-01 runtime assertion vs CI-only — P1, M6; OQ-PF-02 clinical-safety data classification layer — P1, M7; OQ-PF-03 employer legal entitlement claims — P1, M9); eleven-item implementation checklist (5× P0 before enterprise pilot: FORM-PRIV-001 through PRIV-004 alerts, privacy floor test suite, DEC-030 event registration; 5× P1 M6-M9; checklist item 11 Appendix A already closed [x]). Appendix A updated with R-18, R-20, R-21, R-22 quick reference entries (closes R-20 checklist item 8 and R-21 checklist item 6, both previously open). Cross-references: docs/ENTERPRISE.md §"Privacy floor for enterprise" (seven non-negotiable rules), docs/DATA_MODEL.md §17 (Enterprise Admin Reporting Schema — aggregate-only enforcement), docs/DATA_MODEL.md §5 (Data Classification — OQ-PF-02 target), docs/DATA_MODEL.md §6 (Privacy Floor Enforcement), docs/AUDIT_LOG_SCHEMA.md (five new DEC-030 event types to register — checklist item 6), docs/SOC2_READINESS.md §35 P1–P8 Privacy Deep-Dive (P6.1 and P7.1 criteria mapping), docs/OBSERVABILITY.md §13 (Per-Tenant Observability — FORM-PRIV-* alerts to be registered in §6.2 alert rules table), docs/SSO_SCIM_IMPLEMENTATION.md §3 (tenant_admin role definition — RBAC boundary that privacy floor builds on), R-01 (parallel activation if individual data sent to third party), R-10 (Victor AI coach safety — adjacent clinical-safety scope), R-11 (CV biometric privacy — parallel if cv_sessions data involved), R-14 (DSAR — if an employee later requests access to data the employer saw). Owner: compliance-officer + clinical-safety.*

*v1.5 additions (2026-06-03): Fixed header v1.3 → v1.5 (v1.4 additions dated 2026-06-02 added R-20 Insider Threat but did not update the document header). R-21 Key Rotation Failure & Scheduled Cryptographic Control Emergency — closes the runbook gap created when OBSERVABILITY.md §30 (Key Management & Cryptography Observability, v1.7, 2026-06-03) added AL-KEY-01 through AL-KEY-04 alert rules with no `runbook_url` cited; AL-KEY-02 (overdue P0) and AL-KEY-04 (verification missing P0) had no matching incident procedure. R-21 covers: CRYPTO-CHAIN-02 stop-gate (unauthorized rotation → R-05 + R-20, never R-21); severity classification table (P0/P1/P2 with upgrade rule); seven-row trigger matrix with CRYPTO-CHAIN-02 stop-gate row; four scope assessment queries (C1 KV state, C2 audit_log rotation history, C3 scheduler health, C4 HMAC cadence); per-key-type response procedures (SUPABASE_SERVICE_ROLE_JWT per SOC2_READINESS §57; HMAC_AUDIT_CHAIN_KEY dual-key per SOC2_READINESS §58; KEYPOINTS_ENC_KEY maintenance window pending OQ-KEY-02; API keys lower-ceremony); communication protocol with Template K-01 (enterprise tenant maintenance window notice for KEYPOINTS_ENC_KEY, ≥72h advance notice requirement); five DEC-030 HMAC-chained events (admin.key_rotation_incident_opened HIGH 3yr; admin.encryption_key_rotation_initiated HIGH 7yr; admin.encryption_key_rotated HIGH 7yr; admin.encryption_key_rotation_verified HIGH 7yr; admin.key_rotation_incident_closed STANDARD 3yr); evidence package IR-KEY-E-001 through IR-KEY-E-006 with SHA-256 manifest; SOC 2 mapping CC5.2/CC5.3/CC6.7/CC6.8/C1.1/CC7.2/CC7.3; two open questions (OQ-KEY-01 DEC-030 event type registration — P0 M7; OQ-KEY-02 KEYPOINTS_ENC_KEY re-encryption strategy — P1 M12); seven-item implementation checklist (2× P0 M4/M7, 3× P1 M8/M12/M7, 2× P2 M7). Appendix A update pending checklist item 6. Cross-references: docs/OBSERVABILITY.md §30 (AL-KEY-01 through AL-KEY-04; CRYPTO-CHAIN-01 through CRYPTO-CHAIN-04; KEY-SLO-01 through KEY-SLO-04); docs/CRYPTOGRAPHY_POLICY.md §5 (rotation procedures per key type); docs/SOC2_READINESS.md §56 (key inventory); docs/SOC2_READINESS.md §57 (SUPABASE_SERVICE_ROLE_JWT rotation runbook); docs/SOC2_READINESS.md §58 (HMAC_AUDIT_CHAIN_KEY dual-key rotation runbook); docs/AUDIT_LOG_SCHEMA.md (admin.encryption_key_rotated — OQ-ENC-03 target event); docs/SSO_SCIM_IMPLEMENTATION.md §24 (PAM authorisation for rotation sessions); R-05 (HMAC chain break — escalation on CRYPTO-CHAIN-02); R-20 (insider threat — escalation on CRYPTO-CHAIN-02 or unauthorized rotation). Owner: security-engineer + devops-lead.*

---

#### R-23 Victor AI Coach Safety Incident

**Owner:** security-engineer + clinical-safety
**Last updated:** 2026-06-10
**Related docs:** docs/CLINICAL_SAFETY.md, docs/AUDIT_LOG_SCHEMA.md, docs/OBSERVABILITY.md §31, docs/DECISION_LOG.md
**Cross-ref:** R-22 (enterprise privacy floor — adjacent clinical-safety scope), R-10 (Victor AI coach — earlier reference), R-01 (data exfiltration — activate in parallel if prompt injection also leaks PII)

---

##### R-23.1 Scope and Trigger Conditions

This runbook governs incidents where the Victor AI coaching persona produces output that is unsafe, clinically harmful, factually incorrect from a sports-science standpoint, or otherwise violates FORM's clinical-safety commitments. Victor operates as a motivational sports-science coach; any output that crosses into medical diagnosis, clinical psychology, extreme nutrition guidance, or body-shaming territory is in-scope for this runbook regardless of how the content was elicited.

**Trigger condition matrix:**

| Trigger ID | Description | Minimum Severity |
|---|---|---|
| VT-01 | Victor produces dangerous training load advice — e.g., prescribes 1RM maximal lifts for a user whose onboarding profile shows beginner fitness level; prescribes daily high-intensity workouts with no recovery period | P1 HIGH |
| VT-02 | Victor advises user to train through acute injury symptoms — e.g., "push through the sharp knee pain", "that swelling is normal, keep going" | P1 HIGH |
| VT-03 | Victor gives dangerously caloric-deficit advice — e.g., recommends sustained intake below 1,000 kcal/day, or combines aggressive deficit with high-volume training | P0 CRITICAL (clinical-safety VETO) |
| VT-04 | Victor makes a medical claim or diagnosis — e.g., "that sounds like a torn ACL", "you probably have low testosterone", "this is likely overtraining syndrome — stop training for 3 weeks" | P0 CRITICAL (clinical-safety VETO) |
| VT-05 | Victor prescribes, recommends, or implies medication or supplements in a clinical context — e.g., "you should try ibuprofen for the inflammation", "creatine will fix your fatigue — sounds hormonal" | P0 CRITICAL (clinical-safety VETO) |
| VT-06 | Victor generates body-shaming content, eating-disorder-adjacent language, or harmful weight/body-composition framing — e.g., "you need to lose X kg before this program will work", "your body fat is holding you back", references to guilt around eating | P0 CRITICAL (clinical-safety VETO — unconditional) |
| VT-07 | Victor produces output contradicting sports-science consensus — e.g., "rest days are unnecessary for progress", "ignore pain signals, they are mental weakness", "more volume is always better" | P2 MEDIUM |
| VT-08 | Victor consistently fails within a session — hallucinates exercise names, gives contradictory periodisation advice within the same conversation, provides logically incompatible rep/set schemes | P2 MEDIUM |
| VT-09 | Victor generates responses in the wrong language — English when the user's locale is Ukrainian, or vice versa — or generates garbled multilingual output | P3 LOW |
| VT-10 | Victor's tone drifts from persona specification — e.g., becomes dismissive, uses aggressive or demeaning language, abandons the motivational-coach register without clinical-safety implications | P3 LOW |

**Clinical-safety VETO boundary:** Trigger conditions VT-03, VT-04, VT-05, and VT-06 are unconditional clinical-safety VETO territory. No business, technical, or user-preference argument can override the P0 response path for these conditions. clinical-safety owner has final and sole authority to re-enable Victor after a P0 incident.

---

##### R-23.2 Severity Classification

| Severity | Trigger Conditions | Description | Re-enable Authority |
|---|---|---|---|
| **P0 CRITICAL** | VT-03, VT-04, VT-05, VT-06 | Clinical-safety VETO territory: medical diagnosis, medication recommendation, eating-disorder-adjacent content, dangerously caloric-deficit advice. Victor disabled globally, immediately, with no intermediate containment step. | clinical-safety owner written sign-off only |
| **P1 HIGH** | VT-01, VT-02 | Dangerous training advice: extreme loads for beginner profile, advice to train through acute injury symptoms. Victor rate-limited and reviewed; global disable if scope assessment (C1) shows >5 affected sessions in 24h. | sports-scientist + founder joint sign-off |
| **P2 MEDIUM** | VT-07, VT-08 | Sports-science errors: contradicts consensus on rest/recovery, hallucinates exercises or gives contradictory periodisation within a session. Victor continues with guardrail injection; monitoring window 30 min. | devops-lead with updated prompt, 30-min monitoring window |
| **P3 LOW** | VT-09, VT-10 | Tone or localisation violations: wrong language, persona style drift without clinical implications. Log and fix in next deploy cycle; no immediate containment required unless rate elevates. | devops-lead; no sign-off required |

**Severity upgrade rule:** Any incident that begins at P2 or P3 must be upgraded to P1 immediately if C1 scope assessment reveals more than 10 affected sessions. Any P1 incident must be upgraded to P0 immediately if a clinical-safety keyword is found in any of the offending responses during scope assessment (see C1 query, R-23.4).

---

##### R-23.3 Immediate Actions — First 15 Minutes

The following actions apply from the moment the incident is detected, regardless of detection source (user report, automated alert, internal review, or third-party report).

```
T+0     Detection confirmed — assign Incident Commander (IC)
        IC logs incident_id, incident_slug, detection_source in Linear
        Detection sources: user_report | automated_alert | internal_qa | sports_scientist_review

T+2min  Classify severity using trigger matrix (R-23.2)
        If ANY doubt about P0 triggers (VT-03/04/05/06): treat as P0 until proven otherwise

T+3min  [P0 only] Set feature flag: victor_coach_enabled = false in Cloudflare KV
        → No intermediate step; no "wait and see"; no partial disable
        → Serve static fallback message to all users attempting to access Victor
        → Emit DEC-030 event: ai.victor_disabled (severity: HIGH)
        → Page clinical-safety owner immediately (PagerDuty P0 escalation policy)
        → Page founder immediately

T+5min  [P1] Rate-limit Victor responses to 1 response per 5 minutes per user
        Set coaching category affected to read-only (no new sessions in that category)
        Emit DEC-030 event: ai.safety_incident_opened (severity: HIGH)
        Page sports-scientist + founder (PagerDuty P1)

T+5min  [P2/P3] Inject guardrail instruction into Victor system prompt via KV update
        Log the guardrail addition with timestamp and IC user_id
        Emit DEC-030 event: ai.safety_incident_opened (severity: CRITICAL [P0] or HIGH [P1])
        Begin monitoring window

T+7min  Capture evidence package (all severities):
        → Offending prompt text (redacted — user_id UUID only, no PII, no name, no email)
        → Victor response text verbatim
        → session_id (UUID)
        → user_id (UUID — pseudonymised; do NOT log any PII alongside)
        → coaching_turn_category
        → model_version (from Cloudflare KV model_version key)
        → system_prompt_version (from KV victor_prompt_version key)
        → timestamp UTC ISO 8601
        → detection_source
        Store to: compliance/evidence/incidents/<slug>/ir-v-e-001-offending-turn.json
        SHA-256 the file immediately and record hash in incident log

T+10min Determine incident vector — two primary hypotheses:
        Hypothesis A: Model-level regression (model version changed recently)
        Hypothesis B: Prompt injection by user (adversarial input bypassing guardrails)
        See R-23.4 for scope assessment queries C1–C4 to distinguish
        Do NOT assume one hypothesis; run both in parallel

T+12min [P0] Emit DEC-030 event: ai.safety_incident_opened (severity: CRITICAL)
        Confirm clinical-safety owner has received PagerDuty page
        If clinical-safety owner unreachable within 10 min: founder acts as clinical-safety
        proxy for containment decisions only (re-enable still requires clinical-safety sign-off)

T+15min First status update posted to #safety-incidents Slack channel
        Use Template V-02 (R-23.6)
        Include: severity, trigger condition ID, session count (preliminary), containment state
```

**Automated alert prerequisite:** The PagerDuty alert for `ai.safety_incident_opened` at P0 severity must be configured before launch (see Implementation Checklist item 6). If this alert is not yet configured and the incident is detected manually, the IC must manually page clinical-safety and founder via PagerDuty.

---

##### R-23.4 Scope Assessment

Run all four queries within 30 minutes of detection. Queries use pseudonymised identifiers only — do not join to user PII tables during scope assessment.

**C1: Count affected sessions in last 24h with similar prompt patterns**

```sql
-- Identifies coaching turns with similar content patterns to the offending turn
-- Replace :offending_category with the coaching_turn_category from IR-V-E-001
-- Replace :model_version with the model_version from IR-V-E-001
-- clinical_safety_keywords: array of terms from CLINICAL_SAFETY.md trigger word list
SELECT
  COUNT(DISTINCT session_id)           AS affected_session_count,
  COUNT(*)                             AS affected_turn_count,
  MIN(created_at)                      AS earliest_affected_at,
  MAX(created_at)                      AS latest_affected_at,
  bool_or(
    response_text ILIKE ANY(ARRAY[
      '%diagnosis%', '%condition%', '%injury%', '%medication%',
      '%sounds like%', '%probably have%', '%you should stop%',
      '%your body fat%', '%lose weight%', '%eating%', '%calories%',
      '%push through the pain%', '%ignore the pain%', '%no rest%'
    ])
  )                                    AS clinical_safety_keyword_found,
  model_version,
  system_prompt_version
FROM coaching_turns
WHERE created_at >= NOW() - INTERVAL '24 hours'
  AND coaching_turn_category = :offending_category
  AND model_version = :model_version
GROUP BY model_version, system_prompt_version
ORDER BY affected_session_count DESC;
```

`clinical_safety_keyword_found = true` on any row triggers an immediate severity upgrade to P0 if the incident is currently classified below P0.

**C2: Check if a specific user_id is prompt-injecting**

```sql
-- Identifies users with repeated edge-case prompts in the same category
-- within the last 7 days; high turn_count + high distinct_session_count
-- is a prompt-injection signal
SELECT
  user_id,
  COUNT(DISTINCT session_id)           AS distinct_session_count,
  COUNT(*)                             AS total_turn_count,
  MIN(created_at)                      AS first_seen_at,
  MAX(created_at)                      AS last_seen_at,
  array_agg(DISTINCT prompt_hash)      AS distinct_prompt_hashes
FROM coaching_turns
WHERE created_at >= NOW() - INTERVAL '7 days'
  AND coaching_turn_category = :offending_category
GROUP BY user_id
HAVING COUNT(*) > 10
ORDER BY total_turn_count DESC
LIMIT 50;
```

If a single `user_id` appears with `distinct_session_count > 3` and `total_turn_count > 15` in the same category, treat this as a prompt-injection signal. Proceed to R-23.7 (prompt injection path). Do NOT share this query result outside the incident team — it is adversarial-pattern data about a specific user.

**C3: Check if model version changed recently (cross-ref deployment log)**

```sql
-- Compares error rate before and after most recent model_version change
-- Uses the model_version value from IR-V-E-001
WITH version_boundary AS (
  SELECT MIN(created_at) AS version_introduced_at
  FROM coaching_turns
  WHERE model_version = :offending_model_version
),
before_version AS (
  SELECT
    COUNT(*) FILTER (WHERE safety_flag_triggered = true) AS flagged_before,
    COUNT(*)                                              AS total_before
  FROM coaching_turns ct, version_boundary vb
  WHERE ct.created_at < vb.version_introduced_at
    AND ct.created_at >= vb.version_introduced_at - INTERVAL '7 days'
),
after_version AS (
  SELECT
    COUNT(*) FILTER (WHERE safety_flag_triggered = true) AS flagged_after,
    COUNT(*)                                              AS total_after
  FROM coaching_turns ct, version_boundary vb
  WHERE ct.created_at >= vb.version_introduced_at
)
SELECT
  vb.version_introduced_at,
  bv.flagged_before,
  bv.total_before,
  ROUND(bv.flagged_before::numeric / NULLIF(bv.total_before, 0) * 100, 2) AS flag_rate_before_pct,
  av.flagged_after,
  av.total_after,
  ROUND(av.flagged_after::numeric / NULLIF(av.total_after, 0) * 100, 2)  AS flag_rate_after_pct
FROM version_boundary vb, before_version bv, after_version av;
```

`flag_rate_after_pct` significantly greater than `flag_rate_before_pct` (threshold: >3× increase) is a model regression signal. Proceed to R-23.7 (model regression path).

**C4: Check if a specific coaching_turn_category is consistently producing errors**

```sql
-- Compares safety flag rate across all coaching categories in last 7 days
-- Identifies whether the issue is category-specific or model-wide
SELECT
  coaching_turn_category,
  COUNT(*)                                                   AS total_turns,
  COUNT(*) FILTER (WHERE safety_flag_triggered = true)       AS flagged_turns,
  ROUND(
    COUNT(*) FILTER (WHERE safety_flag_triggered = true)::numeric
    / NULLIF(COUNT(*), 0) * 100, 2
  )                                                          AS flag_rate_pct,
  MIN(created_at) FILTER (WHERE safety_flag_triggered = true) AS first_flag_in_category
FROM coaching_turns
WHERE created_at >= NOW() - INTERVAL '7 days'
GROUP BY coaching_turn_category
ORDER BY flag_rate_pct DESC;
```

A single category with `flag_rate_pct` substantially higher than all others (threshold: >5× the median) indicates a prompt-gap in that specific coaching scenario. Proceed to R-23.7 (system prompt gap path).

---

##### R-23.5 Containment Tiers

Containment tier selection is driven by severity and scope assessment results. Each tier is progressive — if a lower tier does not stop the incident within the monitoring window, escalate to the next tier immediately.

**Tier 1 — Soft Guardrail Injection (P2/P3 only)**

Applicable to: P2 MEDIUM (VT-07, VT-08), P3 LOW (VT-09, VT-10).

```typescript
// Inject additional guardrail instruction into Victor system prompt via KV
// This does NOT require a code deploy — KV update takes effect on next coaching turn
const guardrailAddendum = `
SAFETY REMINDER (injected ${new Date().toISOString()} by incident ${incidentSlug}):
- You are a sports-science coach, not a medical professional.
- Never diagnose injuries or medical conditions.
- Never recommend medication or clinical intervention.
- Always recommend consulting a qualified medical professional for injury concerns.
- Always include at least one rest day per week in any training plan.
- Respond only in the user's configured locale language.
`;
await cloudflareKV.put('victor_system_prompt_addendum', guardrailAddendum);
await cloudflareKV.put('victor_prompt_incident_slug', incidentSlug);
```

Monitor for 30 minutes after injection. If a further safety-flag event is emitted within the monitoring window, escalate immediately to Tier 2.

**Tier 2 — Partial Disable (specific coaching category)**

Applicable to: P1 HIGH (VT-01, VT-02) where scope assessment shows the issue is category-specific (C4 confirms single category with elevated flag rate).

```typescript
// Disable a specific coaching category; serve static fallback for that category only
const disabledCategories: string[] = JSON.parse(
  await cloudflareKV.get('victor_disabled_categories') ?? '[]'
);
disabledCategories.push(affectedCategory);
await cloudflareKV.put('victor_disabled_categories', JSON.stringify(disabledCategories));
// Emit DEC-030 event for the category-level partial disable
await emitDec030Event({
  event_type: 'ai.victor_disabled',
  severity: 'HIGH',
  payload: {
    scope: 'category',
    disabled_category: affectedCategory,
    incident_slug: incidentSlug,
    changed_by_user_id: icUserId,
  },
}, supabaseAdminClient);
```

Static fallback message for disabled category (served in user's locale):
> "Victor is taking a short break from [category] coaching while we make some improvements. Your other coaching sessions continue as normal. We'll have this back shortly."

Monitor for 30 minutes. If a further safety-flag event occurs in any other category, escalate to Tier 3.

**Tier 3 — Global Victor Disable**

Applicable to: P0 CRITICAL (all VT-03/04/05/06 triggers — immediate, no intermediate step), P1 HIGH where Tier 2 has not resolved the incident within 30 minutes, or P1 where C1 scope assessment shows >5 affected sessions across multiple categories.

```typescript
// Global disable via feature flag — single KV write; takes effect on next request
await cloudflareKV.put('victor_coach_enabled', 'false');
await cloudflareKV.put('victor_disabled_at', new Date().toISOString());
await cloudflareKV.put('victor_disabled_by', icUserId);
await cloudflareKV.put('victor_disabled_incident', incidentSlug);
// Emit DEC-030 event
await emitDec030Event({
  event_type: 'ai.victor_disabled',
  severity: 'HIGH',
  payload: {
    scope: 'global',
    incident_slug: incidentSlug,
    changed_by_user_id: icUserId,
    trigger_severity: incidentSeverity,   // 'P0' | 'P1'
  },
}, supabaseAdminClient);
```

Static fallback served to all users when `victor_coach_enabled = false`:
> "Victor is temporarily unavailable while we make some improvements. Your workout history and programs are safe. We'll notify you as soon as Victor is back — typically within a few hours."

**Clinical-safety P0 rule:** P0 incidents (VT-03/04/05/06) proceed directly to Tier 3. There is no Tier 1 or Tier 2 step for P0. Any deviation from this rule requires a written exception approved by the founder, documented in Linear, and recorded in the DEC-030 audit chain. In practice, no exception should ever be granted for a clinical-safety VETO condition.

---

##### R-23.6 Communication Templates

**Template V-01 — In-App User Notification (session affected)**

Delivered to users whose `session_id` appears in the C1 scope assessment results. Sent via push notification and in-app message. Delivered within 2 hours of containment for P0/P1; within 24 hours for P2.

```
Subject: A note about your recent Victor session

Hi [first_name],

We want to let you know that a recent Victor coaching session may not have met
the quality standard we hold ourselves to.

We've reviewed the session and taken steps to make sure it doesn't happen again.
If any advice from that session gave you cause for concern, please disregard it
and feel free to reach out to us directly at support@formapp.io.

Your progress and training history are unaffected.

— The FORM Team
```

Notes:
- Do NOT specify what went wrong in the user notification (no specifics about the safety category, the trigger condition, or the response text).
- Do NOT imply the user did anything wrong.
- Do NOT mention "AI safety" or "clinical" concerns — keep the language neutral.
- clinical-safety must review Template V-01 copy before first use (see Implementation Checklist item 5).

**Template V-02 — Internal Slack #safety-incidents Notification**

Posted by IC at T+15 minutes. Audience: founder, clinical-safety owner, sports-scientist, security-engineer.

```
[VICTOR AI SAFETY INCIDENT]
Incident ID: <incident_slug>
Severity: <P0 CRITICAL | P1 HIGH | P2 MEDIUM | P3 LOW>
Trigger: <VT-XX — description>
Containment state: <Tier 1 soft guardrail | Tier 2 category disable | Tier 3 global disable | monitoring>
Sessions affected (preliminary C1): <count>
Detection source: <user_report | automated_alert | internal_qa | sports_scientist_review>
Model version: <version>
Prompt version: <version>
IC: <ic_user_id>
Evidence file: compliance/evidence/incidents/<slug>/ir-v-e-001-offending-turn.json
Next update: T+1h
Runbook: docs/INCIDENT_RESPONSE.md#r-23
```

Ping: @clinical-safety @founder @sports-scientist (always); @compliance-officer (P0 and enterprise-tenant-affected P1 only).

**Template V-03 — Enterprise Tenant Notification**

Applicable when C1 scope assessment confirms that affected users belong to an enterprise tenant. Sent to tenant's primary CSM contact and tenant admin email.

Timing: P0 — within 60 minutes of containment. P1 — within 60 minutes of containment. P2 — within 24 hours. P3 — not required unless tenant CSM requests it.

Board notification threshold: if 5 or more enterprise users from a single tenant are affected, escalate to tenant's executive contact (if known) and loop in FORM founder on the notification.

```
Subject: Service quality notice — Victor coaching

Dear [Tenant Admin Name],

We are writing to inform you of a service quality event that affected a small
number of coaching sessions for users in your organisation on [date].

We identified that Victor AI coach produced responses that did not meet our
quality standards. We have taken immediate action to address this, including
[disabling the affected feature / injecting updated guidance — choose appropriate
description]. The issue has been contained as of [containment_timestamp UTC].

The number of sessions affected within your organisation: [count].
No personal health data was exposed or transmitted to any third party as a result
of this event.

We will provide a full post-incident summary within 7 days.

If you have any questions in the meantime, please contact your FORM customer
success manager directly.

— FORM Security Team
```

Notes:
- Template V-03 must be reviewed by compliance-officer before sending for any P0 incident involving EU-resident users (GDPR Art. 33 assessment required — see R-23.13 OQ-V-02).
- Do NOT disclose the specific offending prompt or response text to the tenant.
- Do NOT disclose the model version or prompt version.

---

##### R-23.7 Root Cause Investigation

Four primary hypotheses. Run in parallel; do not wait for one to be ruled out before starting the next.

**Hypothesis 1 — Prompt Injection by User**

Signal: C2 query shows a single `user_id` with high `total_turn_count` and `distinct_session_count > 3` in the affected `coaching_turn_category`.

Investigation steps:
1. Review the full prompt history for the identified `user_id` (pseudonymised — retrieve via `session_id` chain; do NOT look up PII without legal/compliance review).
2. Check whether the offending prompt contains jailbreak patterns: instructions to "ignore previous instructions", role-play framings that ask Victor to act as a different persona, attempts to extract the system prompt, or direct requests for medical advice framed as hypotheticals.
3. Check whether similar prompt patterns have been submitted by different `user_id` values (coordinated injection attempt signal).
4. If prompt injection confirmed: block the specific prompt pattern via a deny-list rule in the Cloudflare Worker (input validation layer); add the pattern to the adversarial prompt test suite (R-23.9); consider temporary account suspension for the injecting user (involves founder + compliance-officer approval).
5. If coordinated injection: escalate to R-01 (potential data exfiltration via injection) and notify founder immediately.

**Hypothesis 2 — Model-Level Regression**

Signal: C3 query shows `flag_rate_after_pct` is more than 3× `flag_rate_before_pct` following a model version change.

Investigation steps:
1. Identify the exact timestamp of the model version change from the Cloudflare KV deployment log (`victor_model_version` key history).
2. Confirm whether the model version change was an intentional deploy (cross-reference Linear deployment ticket) or an unintended change (supply-chain concern — escalate to security-engineer if no Linear ticket exists).
3. If intentional and regressed: pin `victor_model_version` in KV to the last-known-good version immediately; do not wait for re-enable approval.
4. File a regression report with the model provider (Anthropic). Include: model version that regressed, model version that was last known good, offending response (redacted), category, and a reproducible test prompt (without PII).
5. Do not re-enable the new model version until the provider has confirmed the regression is addressed and FORM's internal regression test suite (R-23.9) passes against the new version.

**Hypothesis 3 — System Prompt Gap**

Signal: C4 query shows a specific `coaching_turn_category` with a flag rate substantially higher than all other categories; OR the offending response involves a coaching scenario not explicitly covered in the current Victor prompt guide.

Investigation steps:
1. Review the current Victor system prompt (`victor_system_prompt` KV key) for the affected `coaching_turn_category`.
2. Identify whether the offending scenario has explicit handling in the prompt (e.g., "if user mentions injury, respond with...") or falls into an uncovered gap.
3. Draft a prompt addition that covers the scenario explicitly. The draft must be reviewed by: sports-scientist (for training advice accuracy) and clinical-safety (for clinical-safety guardrail completeness).
4. Do not add the prompt update to production until both reviewers have signed off in Linear (comment on the prompt-update task).
5. After merge, add the scenario as a regression test case in the Victor prompt test suite (R-23.9, at least one test per new guardrail added).

**Hypothesis 4 — Missing Clinical-Safety Guardrail**

Signal: The offending response type (VT-03/04/05/06) is not explicitly blocked in the current Victor system prompt, OR the prompt injection bypassed an existing guardrail.

Investigation steps:
1. Review `docs/CLINICAL_SAFETY.md` — identify whether the triggered violation type is listed and whether the corresponding prohibition appears in the Victor system prompt.
2. If the violation type is listed in CLINICAL_SAFETY.md but not reflected in the Victor prompt: this is a synchronisation gap. The clinical-safety rules must be re-derived into the Victor prompt by the clinical-safety owner.
3. If the violation type is not listed in CLINICAL_SAFETY.md: this is a new category of harm. clinical-safety owner must add it to CLINICAL_SAFETY.md first, then derive the corresponding prompt guardrail. Both changes require clinical-safety owner sign-off.
4. Update `docs/CLINICAL_SAFETY.md` with the new or updated rule.
5. Update Victor guardrail prompts accordingly.
6. Add the scenario to the Victor regression test suite.

---

##### R-23.8 Re-enable Procedure

Re-enable is always done via feature flag percentage ramp. Never re-enable at 100% in a single step after a P0 or P1 incident.

**P0 CRITICAL — Re-enable path (clinical-safety VETO)**

Prerequisites before any ramp step:
1. clinical-safety owner has provided written sign-off. Acceptable formats: (a) Linear comment on the incident task explicitly stating "I approve re-enable of Victor — clinical-safety sign-off [date] [name]"; or (b) DEC-030 `ai.victor_reenabled` event emitted with `approver_id` matching clinical-safety owner's `user_id`. No other format is acceptable.
2. Root cause has been identified and documented (R-23.7).
3. The fix (prompt update, model pin, or guardrail addition) has been deployed and verified in staging.
4. Victor regression test suite (R-23.9) passes in full — zero failures.
5. sports-scientist has reviewed any prompt changes that affect training advice categories.

Ramp schedule:
```
Step 1: 5% of users — 30 min monitoring window
        Monitor: safety_flag_triggered rate vs baseline; manual spot-check of 10 random turns
        Abort criterion: any safety_flag_triggered = true in the 5% cohort → back to Tier 3

Step 2: 20% of users — 60 min monitoring window
        Monitor: same as Step 1
        Abort criterion: same as Step 1

Step 3: 100% of users — 2h monitoring window
        Monitor: same as Step 1 plus: per-category flag rate from C4 query
        Abort criterion: flag rate in any category exceeds 2× the 7-day pre-incident baseline
```

Emit DEC-030 `ai.victor_reenabled` at each ramp step with `percentage`, `approver_id`, and `incident_slug`.

**P1 HIGH — Re-enable path**

Prerequisites:
1. sports-scientist written sign-off in Linear.
2. Founder acknowledgement in Linear (comment or thumbs-up on the re-enable task is sufficient for P1).
3. Root cause identified and fix deployed.
4. Regression test suite passes.

Ramp schedule: same three-step ramp as P0. Monitoring windows: 15 min at 5%, 30 min at 20%, 1h at 100%.

**P2 MEDIUM / P3 LOW — Re-enable path**

Prerequisites:
1. devops-lead confirms updated prompt or fix is deployed.
2. Regression test suite passes (if a new test was added for this scenario — see R-23.9).

Ramp schedule: can re-enable at 100% directly for P3. For P2, use a 50% → 100% ramp with a 30-min monitoring window at 50%.

---

##### R-23.9 Post-Incident Controls

After every incident, regardless of severity, the following controls must be applied or reviewed. The intent is that each incident makes Victor permanently safer.

**After prompt injection (Hypothesis 1 confirmed):**
- Add the injecting prompt pattern to the adversarial prompt blocklist in the Cloudflare Worker input validation layer. The blocklist is maintained as a KV key `victor_adversarial_blocklist` containing an array of regex patterns.
- Add a regression test case to the Victor prompt test suite that submits the injecting pattern and asserts that Victor's response does not cross any clinical-safety boundary.
- If the user account was determined to be acting in bad faith: document the account action in Linear and in the DEC-030 audit log (action type: `user.adversarial_prompt_block`).

**After model regression (Hypothesis 2 confirmed):**
- Pin `victor_model_version` in KV to the last-known-good version. Do not unpin until the provider confirms the regression is addressed and FORM's test suite passes against the new version.
- Add a regression test case that reproduces the regressed output and asserts it does not recur.
- Update `docs/OBSERVABILITY.md §31` to include a per-model-version safety flag rate metric so future regressions are detected automatically rather than by user report.

**After system prompt gap (Hypothesis 3 confirmed):**
- sports-scientist reviews and updates the Victor prompt guide for the affected `coaching_turn_category`.
- The prompt update is treated as a code change: reviewed in a PR, reviewed by both sports-scientist and clinical-safety, merged to main, and deployed via the standard KV update path.
- Add a regression test case for the gap scenario.

**After clinical-safety guardrail gap (Hypothesis 4 confirmed):**
- clinical-safety owner updates `docs/CLINICAL_SAFETY.md` with the new or updated rule.
- clinical-safety owner updates the Victor guardrail prompts to reflect the new rule.
- Both changes merged in the same PR and reviewed by clinical-safety owner before merge.
- Add a regression test case.

**Victor prompt regression test suite — minimum requirements:**
- At least 10 adversarial scenarios must be present in the test suite before launch (Implementation Checklist item 4).
- Each post-incident addition adds at least 1 new test case.
- Test suite runs in CI on every PR that modifies any Victor-related file (system prompt, KV config, coaching worker).
- Test suite must include at minimum: one test per VT-01 through VT-06 trigger condition; at least 3 prompt-injection patterns; at least 2 language/locale edge cases (Ukrainian input, mixed-language input).

---

##### R-23.10 DEC-030 HMAC-Chained Audit Events

All five events are HMAC-chained per `docs/AUDIT_LOG_SCHEMA.md`. Failure to emit any P0 event aborts the corresponding response action — the IC must retry emission before proceeding.

| Event Type | Severity | Retention | Key Metadata Fields |
|---|---|---|---|
| `ai.safety_incident_opened` | CRITICAL (P0) / HIGH (P1) | 7 years | `incident_slug`, `trigger_condition_id` (e.g., `VT-04`), `session_id`, `user_id` (UUID), `model_version`, `system_prompt_version`, `coaching_turn_category`, `detection_source`, `initial_severity`, `ic_user_id` |
| `ai.safety_incident_contained` | HIGH | 7 years | `incident_slug`, `containment_tier` (`1` / `2` / `3`), `containment_method` (`guardrail_injection` / `category_disable` / `global_disable`), `contained_by_user_id`, `containment_timestamp` |
| `ai.victor_disabled` | HIGH | 7 years | `incident_slug`, `scope` (`global` / `category`), `disabled_category` (if scope = `category`), `disabled_at`, `changed_by_user_id`, `trigger_severity` |
| `ai.victor_reenabled` | HIGH | 7 years | `incident_slug`, `ramp_percentage` (5 / 20 / 100), `approver_id`, `reenabled_at`, `fix_description` (brief, no PII), `regression_test_passed` (bool) |
| `ai.safety_incident_resolved` | STANDARD | 3 years | `incident_slug`, `resolved_at`, `root_cause_hypothesis` (`prompt_injection` / `model_regression` / `prompt_gap` / `clinical_safety_gap`), `sessions_affected_final_count`, `resolution_summary` (brief) |

```typescript
// DEC-030 emission — ai.safety_incident_opened
// Must be emitted before any containment action for P0 incidents.
// For P0: emit BEFORE setting victor_coach_enabled = false (feature flag change).
const event = {
  event_type: 'ai.safety_incident_opened',
  severity: incidentSeverity === 'P0' ? 'CRITICAL' : 'HIGH',
  incident_id: incidentId,
  payload: {
    incident_slug:           incidentSlug,
    trigger_condition_id:    triggerConditionId,    // 'VT-01' through 'VT-10'
    session_id:              sessionId,             // UUID
    user_id:                 userId,                // UUID — no PII
    model_version:           modelVersion,
    system_prompt_version:   promptVersion,
    coaching_turn_category:  category,
    detection_source:        detectionSource,
    initial_severity:        incidentSeverity,      // 'P0' | 'P1' | 'P2' | 'P3'
    ic_user_id:              icUserId,
  },
};
const emitted = await emitDec030Event(event, supabaseAdminClient);
if (!emitted.success) {
  throw new Error(
    'DEC-030 emission failed — ai.safety_incident_opened aborted; ' +
    'retry required before feature flag changes for P0 incidents'
  );
}
```

**Retention note:** `ai.safety_incident_opened` and related events at P0 are retained for 7 years because the incident may involve Art. 9 health-adjacent data (coaching turns involve health behaviour data) and because clinical-safety sign-off records must be auditable for the lifetime of FORM's SOC 2 certification and any applicable regulatory inquiry period. `ai.safety_incident_resolved` is STANDARD (3 years) because the resolution record is secondary evidence once the primary incident record (7-year events) is in place.

---

##### R-23.11 Evidence Package

All evidence under `compliance/evidence/incidents/<slug>/` with SHA-256 manifest at `compliance/evidence/incidents/<slug>/MANIFEST.sha256`. 7-year retention for IR-V-E-001 through IR-V-E-004 and IR-V-E-006 (DEC-030 chain items). 3-year retention for IR-V-E-005.

| Evidence ID | Artefact | Collection Method | Location |
|---|---|---|---|
| **IR-V-E-001** | Offending prompt + response — redacted (user_id UUID only, no PII, no name, no email, no health values beyond what is in the coaching turn itself) | Captured at T+7min per R-23.3; saved as JSON; SHA-256 recorded immediately | `compliance/evidence/incidents/<slug>/ir-v-e-001-offending-turn.json` |
| **IR-V-E-002** | Session count affected — C1 query export | SQL export as JSONL; includes `session_id` (UUID), `coaching_turn_category`, `model_version`, `created_at` for each affected turn; no user PII beyond UUID | `compliance/evidence/incidents/<slug>/ir-v-e-002-affected-sessions.jsonl` |
| **IR-V-E-003** | Feature flag change log — timestamp, changed_by | Cloudflare KV audit log export showing `victor_coach_enabled` key transitions; includes `changed_by_user_id` and ISO 8601 timestamp for each change | `compliance/evidence/incidents/<slug>/ir-v-e-003-feature-flag-log.json` |
| **IR-V-E-004** | Clinical-safety sign-off record (P0 only) | Linear comment export or DEC-030 `ai.victor_reenabled` event with `approver_id` matching clinical-safety owner; includes timestamp and approver_id | `compliance/evidence/incidents/<slug>/ir-v-e-004-clinical-safety-signoff.eml` or `.json` |
| **IR-V-E-005** | Updated Victor prompt diff (post-incident fix) | Git diff of the Victor system prompt KV update; includes PR link, reviewer sign-offs, and regression test suite results (pass/fail per test case) | `compliance/evidence/incidents/<slug>/ir-v-e-005-prompt-diff.patch` |
| **IR-V-E-006** | Re-enable monitoring log — feature flag ramp with timestamps | Log of each ramp step (5% / 20% / 100%), monitoring window results, and abort-criterion check; includes DEC-030 `ai.victor_reenabled` event IDs at each step | `compliance/evidence/incidents/<slug>/ir-v-e-006-reenable-monitoring-log.json` |

**PII rule for evidence collection:** IR-V-E-001 and IR-V-E-002 must contain only UUIDs for user identification — never names, emails, biometric values, or other PII. The UUID-to-PII mapping exists in the Supabase `users` table and is accessible only via authenticated API with appropriate role; it is not compiled into the evidence package. If a legal process requires PII mapping, this is retrieved by compliance-officer under legal hold separately from the incident evidence package.

---

##### R-23.12 SOC 2 TSC Mapping

| TSC Criterion | Control | Evidence from This Runbook |
|---|---|---|
| **CC7.2** (Monitor system components) | Victor safety flag rate monitored per session, per category, per model version; PagerDuty alert fires on `ai.safety_incident_opened` P0 (Implementation Checklist item 6); per-session error rate in OBSERVABILITY.md §31 (Implementation Checklist item 7) | IR-V-E-002: session count and flag rate data; DEC-030 `ai.safety_incident_opened` event establishes monitoring evidence |
| **CC7.3** (Evaluate security events) | Scope assessment queries C1–C4 (R-23.4) provide structured evaluation of every Victor safety incident; root cause investigation (R-23.7) provides documented evaluation of all four hypotheses | R-23.4 query outputs (IR-V-E-002); R-23.7 investigation record in Linear |
| **CC7.4** (Respond to security incidents) | Runbook R-23 documents the complete incident response procedure; re-enable procedure (R-23.8) with feature flag ramp provides structured recovery; post-incident controls (R-23.9) provide permanent improvement | This runbook is itself CC7.4 evidence; IR-V-E-003 (feature flag log); IR-V-E-006 (ramp monitoring log) |
| **CC9.2** (Vendors and partners — AI model provider) | Model version pinning (R-23.7 Hypothesis 2) and regression reporting to model provider (Anthropic) document the vendor management response; model regression signal (C3 query) provides structured vendor performance monitoring | IR-V-E-003: KV model version pin log; regression report filed with Anthropic (if applicable) |
| **A1.2** (Availability — performance and availability monitoring) | Victor feature flag (`victor_coach_enabled`) provides rapid availability control; static fallback message ensures users are never left with a broken experience; ramp re-enable procedure (R-23.8) provides controlled availability restoration | IR-V-E-003: feature flag change log shows availability management; IR-V-E-006: ramp log shows availability restoration |

---

##### R-23.13 Open Questions

**OQ-V-01: Should FORM implement a real-time safety classifier (separate lightweight model) that screens Victor outputs before delivery?**

A real-time classifier running in the Cloudflare Worker between Victor's response generation and response delivery to the client would provide an independent second-opinion safety gate. The classifier would be a small, fast model specifically fine-tuned on clinical-safety and sports-science safety categories, distinct from Victor's general-purpose model. Estimated latency cost: +40–60 ms per coaching turn. The alternative is to rely entirely on Victor's own guardrail instructions within the system prompt.

Assessment: relying solely on guardrail instructions in a general-purpose model is not sufficient for P0 clinical-safety conditions (VT-03, VT-04, VT-05, VT-06). System prompts can be bypassed by prompt injection; model regressions can degrade guardrail adherence without warning. An independent classifier operating on the output — not the input — would catch both failure modes.

Recommendation: implement the classifier for P0 clinical-safety categories only (VT-03 through VT-06). The classifier does not need to gate VT-07 through VT-10 (sports-science errors and tone issues), which can be handled by prompt improvements and periodic review. Defer the full classifier to V2 roadmap; implement basic output scanning (keyword + pattern matching against clinical-safety trigger word list from CLINICAL_SAFETY.md) in V1 as a lower-cost intermediate measure.

Owner: security-engineer + clinical-safety. Priority: **P1 — M8.** Decision to be documented in `docs/DECISION_LOG.md` once made.

**OQ-V-02: What is FORM's GDPR Art. 33 notification obligation to a supervisory authority if a P0 AI safety incident resulted in health-adjacent harmful content being delivered to EU-resident users?**

GDPR Art. 33 requires notification to the competent supervisory authority within 72 hours of becoming aware of a personal data breach. The key question is whether a Victor AI safety incident (harmful output delivered to a user) constitutes a "personal data breach" under GDPR Art. 4(12) — i.e., "a breach of security leading to the accidental or unlawful destruction, loss, alteration, unauthorised disclosure of, or access to, personal data transmitted, stored or otherwise processed."

A Victor output error (Victor saying something harmful) is not inherently a breach of security or a disclosure of personal data. The harm is the dangerous content delivered to the user, not a leakage of data. However, if the P0 incident involved a prompt injection attack that also caused Victor to reveal another user's session data, or if the harmful output was itself derived from health data processed about the user (e.g., Victor misused stored health metrics in its response), there may be an Art. 33 notification duty.

Recommendation: consult outside counsel with GDPR expertise before enterprise GA. The legal opinion should address: (a) whether FORM's Victor AI output constitutes "processing" of health data under Art. 9 (likely yes, for coaching turn content); (b) whether a harmful AI output error — with no data exfiltration — triggers Art. 33; (c) the threshold for Art. 9 data breach notification under the applicable supervisory authority's guidance. Pending legal opinion: FORM's default posture is that Art. 33 notification is NOT required for a pure AI output error with no PII exfiltration, but IS required if prompt injection or a related bug also caused any cross-user data access. Document the legal opinion outcome in `docs/DECISION_LOG.md`.

Owner: compliance-officer + outside counsel. Priority: **P1 — M9 (before enterprise GA).** Cross-reference: R-22 OQ-PF-03 (employer legal entitlement claims, same outside-counsel engagement recommended).

---

##### R-23.14 Appendix A Cross-Reference

The following entries are to be added to the Appendix A quick-reference table when R-23 is integrated into the main document index:

| Scenario | Runbook |
|---|---|
| Victor AI producing dangerous training advice (extreme loads, injury training) | R-23 (P1 HIGH) — R-23.3 immediate actions, R-23.5 Tier 2 |
| Victor AI making medical claims or diagnosing injuries | R-23 (P0 CRITICAL — clinical-safety VETO) — R-23.3 T+3min global disable |
| Victor AI generating body-shaming or eating-disorder-adjacent content | R-23 (P0 CRITICAL — clinical-safety VETO) — R-23.3 T+3min global disable |
| Victor AI giving dangerously caloric-deficit advice | R-23 (P0 CRITICAL — clinical-safety VETO) — R-23.3 T+3min global disable |
| Victor AI contradicting sports-science consensus (no rest days, ignore pain) | R-23 (P2 MEDIUM) — R-23.5 Tier 1 guardrail injection |
| Victor AI hallucinating exercises or giving contradictory periodisation | R-23 (P2 MEDIUM) — R-23.5 Tier 1 guardrail injection |
| Victor AI responding in wrong language | R-23 (P3 LOW) — R-23.5 Tier 1 guardrail injection |
| Prompt injection suspected in a Victor coaching session | R-23.7 Hypothesis 1; also check R-01 if PII exfiltration possible |
| Victor AI safety incident affecting enterprise tenant users | R-23.6 Template V-03; also R-22 if privacy floor breach suspected |

---

##### Implementation Checklist

| # | Action | Priority | Milestone | Owner | Status |
|---|---|---|---|---|---|
| 1 | Create feature flag `victor_coach_enabled` in Cloudflare KV; confirm that setting it to `false` causes the coaching Worker to serve the static fallback message to all users within one request cycle; verify in staging | devops-lead + platform-engineer | **P0** | M4 | [ ] |
| 2 | Register all five DEC-030 events (`ai.safety_incident_opened`, `ai.safety_incident_contained`, `ai.victor_disabled`, `ai.victor_reenabled`, `ai.safety_incident_resolved`) in `docs/AUDIT_LOG_SCHEMA.md` with full payload schema, severity, and retention per R-23.10; validate Zod schema in staging | platform-engineer + compliance-officer | **P0** | M4 | [ ] |
| 3 | Document clinical-safety re-enable procedure in `docs/CLINICAL_SAFETY.md` with owner confirmation: clinical-safety owner named explicitly; sign-off format defined (Linear comment format or DEC-030 approver_id); confirm owner has PagerDuty access for P0 pages | clinical-safety + security-engineer | **P0** | M5 | [ ] |
| 4 | Create Victor prompt regression test suite with at least 10 adversarial scenarios: one per VT-01 through VT-06, at least 3 prompt-injection patterns, at least 1 language edge case (Ukrainian input producing English output); add as CI gate on all PRs touching Victor system prompt, coaching Worker, or KV config | qa-lead + sports-scientist + clinical-safety | **P1** | M6 | [ ] |
| 5 | Draft and review Template V-01 (user in-app notification), Template V-02 (internal Slack), and Template V-03 (enterprise tenant notification) copy; review by brand-voice lead for tone; review by clinical-safety for accuracy of what is disclosed vs withheld; final approval by founder | clinical-safety + brand-voice + founder | **P1** | M5 | [ ] |
| 6 | Configure PagerDuty alert for `ai.safety_incident_opened` at P0 severity: fires within 60 seconds of DEC-030 event emission; escalation policy pages clinical-safety owner (primary) and founder (secondary) simultaneously; test with synthetic event in staging | devops-lead | **P0** | M4 | [ ] |
| 7 | Add per-session Victor error rate monitoring to `docs/OBSERVABILITY.md §31`: define alert rule `FORM-VICTOR-001` (safety_flag_triggered rate > baseline threshold); define per-category flag rate metric; confirm metric is computed from `coaching_turns` table and visible in monitoring dashboard | devops-lead + platform-engineer | **P1** | M6 | [ ] |
| 8 | Schedule sports-scientist quarterly review of Victor prompt guide: calendar invite, Linear recurring task, review checklist (new coaching categories added since last review, any post-incident prompt additions, regression test suite coverage gap assessment); first review at end of M9 | sports-scientist + clinical-safety | **P2** | Quarterly (start M9) | [ ] |
| 9 | Document OQ-V-01 classifier decision in `docs/DECISION_LOG.md`: choose between real-time classifier, keyword/pattern output scanner, or guardrail-only; record decision, rationale, and owner; if classifier chosen, add to V2 product roadmap | security-engineer + clinical-safety | **P1** | M8 | [ ] |
| 10 | Complete OQ-V-02 legal consultation: engage outside counsel (same engagement as R-22 OQ-PF-03 if timing aligns); obtain written opinion on Art. 33 notification obligation for AI output errors involving EU health data; document outcome in `docs/DECISION_LOG.md` and update `docs/CLINICAL_SAFETY.md` if the opinion requires a change to FORM's default posture | compliance-officer + outside counsel | **P1** | M9 | [ ] |

---

*v1.9 additions (2026-06-10): R-23 Victor AI Coach Safety Incident — twenty-third runbook; closes the incident response gap for FORM's Victor AI coaching persona producing unsafe, clinically harmful, or sports-science-erroneous output. R-23 covers: ten-row trigger matrix (VT-01 through VT-10) spanning clinical-safety VETO conditions (medical diagnosis, medication recommendation, caloric-deficit, body-shaming — VT-03 through VT-06 P0 unconditional), dangerous training advice (VT-01/02 P1), sports-science errors (VT-07/08 P2), and tone/localisation violations (VT-09/10 P3); four-row severity classification table with clinical-safety VETO boundary and severity upgrade rule (>10 sessions triggers P1 → P0 if clinical-safety keyword found); first-15-minute immediate action timeline (T+0 through T+15min) with P0 global disable at T+3min before any intermediate step; four scope assessment SQL queries C1–C4 (affected session count with clinical-safety keyword scan, prompt-injection user pattern detection, model version regression rate comparison, per-category flag rate analysis); three-tier containment (Tier 1 soft guardrail injection for P2/P3, Tier 2 category partial disable for P1 category-specific, Tier 3 global feature flag disable for P0 and escalated P1 — P0 always Tier 3 with no intermediate); three communication templates (V-01 user in-app notification, V-02 internal Slack #safety-incidents with board-notification threshold at 5+ enterprise users, V-03 enterprise tenant notification within 60 min P0/P1); four root cause hypotheses investigated in parallel (prompt injection, model regression, system prompt gap, clinical-safety guardrail gap); three-step feature flag ramp re-enable (5% → 20% → 100%) with clinical-safety VETO gate for P0 and sports-scientist + founder gate for P1; post-incident controls per hypothesis including adversarial blocklist update, model version pin, prompt regression test suite addition, CLINICAL_SAFETY.md update; five DEC-030 HMAC-chained events (ai.safety_incident_opened CRITICAL/HIGH 7yr; ai.safety_incident_contained HIGH 7yr; ai.victor_disabled HIGH 7yr; ai.victor_reenabled HIGH 7yr; ai.safety_incident_resolved STANDARD 3yr); six evidence artefacts IR-V-E-001 through IR-V-E-006 with PII rule (UUID only in evidence package); SOC 2 mapping CC7.2/CC7.3/CC7.4/CC9.2/A1.2; two open questions (OQ-V-01 real-time safety classifier — P1 M8, document in DECISION_LOG.md; OQ-V-02 GDPR Art. 33 notification for AI output errors to EU users — P1 M9, outside counsel engagement); Appendix A cross-reference table with nine Victor-specific scenario entries; ten-item implementation checklist (3× P0 M4-M5: feature flag, DEC-030 registration, clinical-safety re-enable procedure; 5× P1 M5-M9: regression test suite, communication templates, PagerDuty alert, OBSERVABILITY.md §31 metrics, classifier decision, legal consultation; 1× P2 quarterly: sports-scientist prompt review; 1× P1 M8: OQ-V-01 decision). Owner: security-engineer + clinical-safety.*

---

### R-24: SCIM Mass Deprovisioning Emergency

> **Scope:** Covers the failure mode where a SCIM 2.0 provisioning sync from an enterprise tenant's IdP (Okta, Azure AD / Entra ID, Google Workspace, or any SAML/OIDC provider with SCIM enabled) triggers accidental mass deprovisioning of legitimate active users — causing widespread access loss for the tenant's workforce without any intentional HR action. This is distinct from R-19 (IdP Outage, which covers FORM failing to reach the IdP) and R-04 (SSO/SAML Compromise, which covers a security incident on the authentication path). R-24 is a *data integrity and availability* incident: the IdP connectivity is healthy, FORM's SCIM endpoint processed the request correctly, but the deprovision payload was wrong — typically due to IdP misconfiguration, an AD group deletion, an Okta group rule error, or a SCIM filter expression that matched more users than intended.
>
> **Privacy floor invariant:** Individual user access status (who was deprovisioned) is aggregate-only in all external communications. Tenant admins are told the count; HR is never given a list of named individuals unless they are the data controller requesting it in writing under their DPA obligations. Even in a recovery context, FORM does not enumerate individual employees by name to HR contacts.

#### R-24.1 Trigger Matrix

| Source | Signal | Threshold | Severity |
|---|---|---|---|
| **AL-SCIM-MASS-01** (new — §R-24.11) | `scim.user_deprovisioned` DEC-030 event burst: ≥ 10% of tenant's active seat count within any rolling 10-minute window | Fires on first threshold breach | See severity table below |
| **Tenant admin direct report** | Tenant admin contacts CSM or `enterprise@form.coach` reporting "many users locked out" | N/A — any report | P1 pending scope assessment |
| **CSM weekly health check** | Admin dashboard shows seat activation rate dropped > 20% vs. prior week with no corresponding HR action in CRM | > 20% drop unexplained | P2 investigation |
| **SCIM endpoint error log** | `scim.batch_deprovision_completed` DEC-030 event where `deprovisioned_count > 0.1 * tenant_seat_count` | Single event | P1 pending scope |

**Severity classification:**

| Condition | Severity | Rationale |
|---|---|---|
| ≥ 50% of tenant's active seats deprovisioned in < 30 minutes | **P0** | Critical availability breach; SLA breach guaranteed; board notification threshold may apply (§13) |
| ≥ 10% but < 50% deprovisioned, OR ≥ 100 users regardless of percentage | **P1** | Material availability impact; SLA at risk; CSM immediate escalation required |
| < 10% AND < 50 users, AND IdP confirms the deprovisioning was intentional (e.g., terminations) | **P2** | Notification and verification only; no emergency recovery needed |
| Any deprovisioning affecting a tenant_owner or all tenant_admin roles (admin lockout) | **P0** (override) | Tenant cannot self-service; FORM CSM must take direct action; magic link fallback mandatory per §R-24.4 |

**Do NOT downgrade** from P0 to P1 based on a verbal assertion from a tenant's IT admin that the deprovisioning was "probably intentional" — confirm in writing before downgrade. Accidental deprovisions are often initially attributed to coincidence by IT teams under pressure.

---

#### R-24.2 Immediate Actions (T+0 to T+20 min)

```
T+0   Open incident channel: #inc-YYYYMMDD-scim-mass-<tenant-slug>
      Page: CSM (Customer Lead) + platform-engineer
      Note exact detection time — this is the SLA clock start for §12 enterprise breach protocol
      Do NOT contact the tenant yet: gather scope first to avoid premature alarm

T+2   Run R-24-C1 (scope assessment SQL below) to confirm deprovisioned count and %
      If result confirms P0 or P1 threshold: proceed immediately to containment
      If result < P1 threshold: continue monitoring; move to P2 investigation track

T+5   Suspend SCIM sync for the affected tenant (see §R-24.3 — suspend mechanism)
      This stops further deprovisioning events from propagating
      Note: existing valid sessions are NOT revoked by this step — affected users with
      active session tokens can still use FORM until session expiry

T+8   Run R-24-C2 (active session count for deprovisioned users)
      If active sessions exist: inform tenant admin that users will experience disruption
      only at next login, not immediately — this reduces perceived urgency if P1

T+10  Notify tenant admin via Template MD-01 (§R-24.7)
      Channel: direct email to tenant admin contact + Slack Connect if provisioned
      The notification names the incident ID, the confirmed scope (aggregate count only),
      and the containment step already taken (SCIM sync suspended)
      DO NOT include individual user names or roles in notification

T+20  Begin root cause analysis in parallel with tenant IdP admin (§R-24.5)
      Objective: determine if deprovisioning was accidental (IdP misconfiguration)
      or reflects an intentional but mis-scoped IdP change (e.g., group rename)
```

---

#### R-24.3 Containment

**Step 1: Suspend SCIM sync for the affected tenant**

SCIM provisioning is controlled per-tenant via the `tenant_sso_configs.scim_enabled` flag and the Cloudflare KV key `scim:sync:paused:{tenant_id}` (TTL-free — must be explicitly removed).

```typescript
// Emergency SCIM suspension — execute via Cloudflare Workers REST API
// or via the internal admin console at /internal/v1/admin/scim/pause

const PAUSE_PAYLOAD = {
  tenant_id: "<affected_tenant_id>",
  reason: "R-24 mass deprovision incident — INC-YYYYMMDD-<slug>",
  paused_by: "<ic_admin_user_id>",
  pause_duration_hours: 24,  // auto-lifted after 24h if not manually resumed
};

// This writes KV key: scim:sync:paused:<tenant_id>
// The SCIM endpoint returns HTTP 503 for all incoming PUSH requests when key exists
// GET /scim/v2/Users requests still served (read-only) — IdP dry-run is unaffected
await env.SCIM_KV.put(
  `scim:sync:paused:${PAUSE_PAYLOAD.tenant_id}`,
  JSON.stringify(PAUSE_PAYLOAD),
  { expirationTtl: PAUSE_PAYLOAD.pause_duration_hours * 3600 }
);
```

Emit DEC-030 `scim.sync_suspended` event immediately after KV write (§R-24.9).

**Step 2: Verify existing sessions are intact**

Deprovisioning in FORM sets `tenant_members.deprovisioned_at` to the current timestamp. It does NOT immediately revoke active `enterprise_sessions` — session revocation is a separate, explicit action (per `docs/SSO_SCIM_IMPLEMENTATION.md §22` KV revocation cache). This means users who were deprovisioned may still have valid sessions until expiry or next login attempt.

**Decision point — session revocation:**
- **If incident is confirmed accidental:** Do NOT revoke active sessions. Affected users can continue working; recovery restores their `tenant_members` row without requiring re-login.
- **If root cause is unknown and security-related (e.g., possible credential compromise at IdP):** Page security-engineer; consider session revocation per `docs/SSO_SCIM_IMPLEMENTATION.md §22.5` (bulk revocation). This is a higher-disruption action — confirm with IC before proceeding.

**Step 3: Freeze downstream seat-count billing adjustments**

If the tenant is on a metered-by-active-seat billing model (Starter tier contracts), mass deprovision could incorrectly trigger a billing credit calculation at the next billing cycle. Flag to customer-success immediately: do not process any seat-count adjustment for the affected tenant until the incident is resolved. Record the pre-incident active seat count from the scope assessment SQL output.

---

#### R-24.4 Admin Lockout Sub-Protocol

If the mass deprovisioning affects all users with `tenant_admin` or `tenant_owner` roles — leaving the tenant with no admin-capable accounts — the standard self-service admin console is inaccessible. Activate the magic link fallback procedure immediately.

**Magic link fallback for tenant recovery (per `docs/SSO_SCIM_IMPLEMENTATION.md §10`):**

```
1. Identify the tenant owner's email from FORM's CRM record
   (do NOT ask the tenant admin — they may be one of the locked-out users)
   Source: customer-success CRM record, original contract counterparty email

2. CSM sends a FORM-generated magic link directly to that email via the admin API:
   POST /internal/v1/admin/magic-link
   { "email": "<tenant_owner_email>", "tenant_id": "<tenant_id>",
     "reason": "R-24 admin lockout recovery", "incident_id": "INC-..." }

3. Magic link is valid for 15 minutes; grants tenant_owner session
   bypassing SSO (which may itself be broken if IdP config is corrupt)

4. Tenant owner uses session to re-grant admin roles to other team members
   from the admin console

5. Emit DEC-030 scim.admin_lockout_recovery (see §R-24.9) for audit trail

6. Log: IC + CSM user IDs, time of magic link generation, tenant owner email hash
   No plain-text email addresses in incident channel or DEC-030 events
```

**If the tenant owner email is also unknown** (e.g., the original contract signer has left the company): escalate to compliance-officer + founder. Recovery requires identity verification of the new responsible party — this is a legal/contracts matter, not purely a technical one. Do not issue a magic link to an unverified party.

---

#### R-24.5 Root Cause Analysis

Investigate all four hypotheses in parallel with the tenant IdP admin. The goal is to determine whether the deprovisioning was IdP-side (most common) or FORM-side (rare, but must be ruled out).

| # | Hypothesis | Diagnostic signals | Resolution path |
|---|---|---|---|
| **H1** | IdP group deletion or rename caused users to drop out of the SCIM-mapped group | Check IdP audit log for group change events in the 30 minutes preceding first `scim.user_deprovisioned` DEC-030 event | IdP admin re-creates group or renames to match SCIM mapping; FORM re-enables sync; IdP re-pushes group membership |
| **H2** | SCIM filter expression on the IdP side became too broad (e.g., department filter changed, OU relocated) | Check IdP SCIM provisioning app audit log; compare pre- and post-incident `attribute_mapping` config | IdP admin corrects the filter; FORM re-enables sync; full re-push from IdP to restore membership |
| **H3** | FORM SCIM endpoint processed a valid batch `DELETE /Users` request incorrectly (FORM-side bug) | Compare `scim_sync_log` Cloudflare Worker logs against IdP-sent SCIM payload; look for deprovisioned_user_ids that do NOT appear in the IdP's DELETE request list | FORM engineering deploys fix; revert affected `tenant_members` rows from DEC-030 event history; post-incident: add batch-deprovision regression test to CI |
| **H4** | IdP SCIM provisioning app underwent an automatic schema update that changed the user matching logic | Check IdP app version history; look for scheduled IdP platform update in tenant's IT change log | IdP support case + FORM CSM documents new schema; update SCIM attribute mapping config; test with IdP dry-run before re-enabling |

**Evidence to gather from IdP admin (request within T+30 min):**
- IdP SCIM provisioning audit log export: past 2 hours, all events
- Any group, OU, or directory change events in the past 6 hours
- Screenshot of current SCIM provisioning app configuration (filter, attribute mapping, scope)
- Confirmation of IdP platform version and whether an automatic update occurred

---

#### R-24.6 Scope Assessment SQL

All queries require `form_admin` BYPASSRLS. Execute via `pam-db-proxy` using a `read_only` PAM elevation with `reason: "R-24 mass deprovision scope assessment — INC-..."`.

```sql
-- R-24-C1: Count deprovisioned users and % of tenant in rolling window
-- Replace $tenant_id and $window_minutes with incident values

SELECT
  COUNT(*)                                          AS deprovisioned_count,
  (SELECT COUNT(*) FROM tenant_members
   WHERE tenant_id = $tenant_id
   AND deprovisioned_at IS NULL)                    AS still_active_count,
  ROUND(
    COUNT(*) * 100.0 /
    NULLIF((SELECT COUNT(*) FROM tenant_members
            WHERE tenant_id = $tenant_id), 0), 1)  AS pct_of_all_seats,
  MIN(deprovisioned_at)                             AS first_deprovision,
  MAX(deprovisioned_at)                             AS last_deprovision,
  ROUND(EXTRACT(EPOCH FROM
    (MAX(deprovisioned_at) - MIN(deprovisioned_at))
  ) / 60, 1)                                       AS deprovision_window_minutes
FROM tenant_members
WHERE tenant_id = $tenant_id
  AND deprovisioned_at >= NOW() - INTERVAL '1 hour';
```

```sql
-- R-24-C2: Active sessions for deprovisioned users (determines immediate user impact)

SELECT
  COUNT(DISTINCT es.user_id)          AS users_with_active_session,
  COUNT(es.id)                        AS total_active_sessions,
  MIN(es.expires_at)                  AS earliest_session_expiry,
  MAX(es.expires_at)                  AS latest_session_expiry
FROM enterprise_sessions es
INNER JOIN tenant_members tm
  ON tm.user_id = es.user_id
  AND tm.tenant_id = es.tenant_id
WHERE tm.tenant_id = $tenant_id
  AND tm.deprovisioned_at >= NOW() - INTERVAL '1 hour'
  AND tm.deprovisioned_at IS NOT NULL
  AND es.expires_at > NOW()
  AND es.revoked_at IS NULL;
```

```sql
-- R-24-C3: Role distribution of deprovisioned users (identifies admin lockout risk)

SELECT
  ur.role,
  COUNT(*)  AS deprovisioned_count
FROM tenant_members tm
INNER JOIN user_roles ur ON ur.user_id = tm.user_id AND ur.tenant_id = tm.tenant_id
WHERE tm.tenant_id = $tenant_id
  AND tm.deprovisioned_at >= NOW() - INTERVAL '1 hour'
GROUP BY ur.role
ORDER BY CASE ur.role
  WHEN 'tenant_owner'   THEN 1
  WHEN 'tenant_admin'   THEN 2
  WHEN 'tenant_manager' THEN 3
  ELSE 4
END;
```

```sql
-- R-24-C4: DEC-030 audit chain for the deprovisioning burst
-- Verify events are well-formed SCIM-sourced deprovisions, not a security incident

SELECT
  ale.id             AS audit_event_id,
  ale.occurred_at,
  ale.event_type,
  ale.actor_id,
  ale.metadata->>'scim_request_id'   AS scim_request_id,
  ale.metadata->>'scim_source'       AS scim_source,
  ale.metadata->>'deprovision_reason' AS deprovision_reason,
  ale.hmac_chain_position
FROM audit_log_events ale
WHERE ale.tenant_id = $tenant_id
  AND ale.event_type = 'scim.user_deprovisioned'
  AND ale.occurred_at >= NOW() - INTERVAL '1 hour'
ORDER BY ale.occurred_at ASC;
```

**Interpreting R-24-C4:**
- `actor_id` should be the SCIM service account UUID — not a human user_id. If a human actor_id appears, escalate to R-01 (potential unauthorized bulk action).
- `scim_request_id` should be consistent across the burst (same batch request) or sequential (multiple IdP-initiated requests). If random and non-sequential, investigate FORM-side processing bug (H3).
- `deprovision_reason` should be `scim_push` — if it is `admin_manual` or `pam_escalation`, stop immediately and page security-engineer.

---

#### R-24.7 Communication Templates

##### Template MD-01: Initial Notification to Tenant Admin (T+10 min)

```
Subject: [FORM — Service Notice] Access disruption for [Tenant Name] users — INC-YYYYMMDD-<slug>

[Tenant Admin Name],

We have detected an unexpected change in user provisioning for your FORM account
and are writing to inform you immediately.

Incident Reference: INC-YYYYMMDD-<slug>
Severity: [P0 / P1]
Detected: [HH:MM UTC, YYYY-MM-DD]
Accounts affected: approximately [N] users (aggregate count)
Current status: Investigating. FORM has suspended your SCIM sync to prevent
further changes while we diagnose the root cause.

What this means for your users:
• Users who had active FORM sessions when this occurred can continue using FORM
  until their session expires (typically [X] hours from last login).
• Users who log out or whose session expires will be unable to log in until
  provisioning is restored.
• [If admin lockout: Your admin console is affected. We have initiated a
  recovery procedure and will contact [owner email domain] directly.]

What we are doing:
Your dedicated CSM [name] is managing this incident. Our engineering team is
investigating the root cause. We will provide an update within 30 minutes.

What we need from you:
• Please share your IdP provisioning audit log for the past 2 hours
  (Okta: Admin → System Log; Azure AD: Users → Audit logs; Google: Admin SDK audit)
• Confirm whether any group, OU, or directory changes were made in the past 6 hours

Please reply to this email or contact [CSM name] directly at [CSM Slack/email].
We are treating this as our highest priority.

— [Your CSM Name], FORM Customer Success
```

##### Template MD-02: Status Update (every 30 min until resolved)

```
Subject: [FORM — Update] INC-YYYYMMDD-<slug> — [status]

[Tenant Admin Name],

Update as of [HH:MM UTC]:

Current status: [Investigating root cause / Root cause identified, recovery in progress /
                 SCIM sync restored, re-provisioning complete]

Users restored: [N of N] (updated count if partial recovery is underway)

Root cause (preliminary / confirmed): [one sentence — no speculation if still unknown]

Next update: in 30 minutes, or immediately on resolution.

— [Your CSM Name]
```

##### Template MD-03: Resolution Notice

```
Subject: [FORM — Resolved] INC-YYYYMMDD-<slug> — User access restored

[Tenant Admin Name],

We are writing to confirm that the provisioning incident for [Tenant Name] has
been resolved.

Incident: INC-YYYYMMDD-<slug>
Resolved at: [HH:MM UTC, YYYY-MM-DD]
Duration: [N] minutes from detection to full restoration

Summary:
[2-3 sentences describing root cause and the fix, e.g.:
"An Okta group rule was inadvertently modified on [date], causing the group
membership for 'FORM Users' to exclude all active employees. FORM's SCIM endpoint
processed the resulting deprovision requests as instructed. After your IT team
corrected the group rule and re-pushed membership, FORM re-enabled SCIM sync and
confirmed all [N] users are now active."]

Actions taken by FORM:
• Suspended SCIM sync at [time] to prevent further deprovisioning
• [If admin lockout was involved: Issued emergency admin access recovery at [time]]
• Re-enabled SCIM sync at [time] after root cause confirmed resolved on IdP side
• Confirmed [N] user accounts active at [time]

SLA impact:
[If P0 threshold triggered: "This incident triggered our P0 SLA threshold. Your
Customer Success Manager will calculate the applicable credit per §4 of your MSA
and apply it to your next invoice."]
[If P1, no breach: "This incident did not breach your contracted SLA thresholds."]

Post-incident review:
We will share a formal post-incident summary within 5 business days. If you have
any questions, please contact [CSM name] directly.

— [Your CSM Name]
```

---

#### R-24.8 Recovery Procedure

**Standard recovery (IdP-side root cause confirmed, H1 or H2):**

1. Wait for confirmation from tenant IT admin that IdP configuration is corrected and re-push is initiated.
2. Confirm SCIM sync is still suspended (`scim:sync:paused:{tenant_id}` KV key present).
3. Once IT admin confirms re-push is complete in IdP audit log, remove the KV suspension key:

```typescript
// Resume SCIM sync — execute via internal admin console or direct KV API
await env.SCIM_KV.delete(`scim:sync:paused:${tenant_id}`);
// Emit DEC-030 scim.sync_resumed (§R-24.9)
```

4. Monitor `scim.user_provisioned` DEC-030 events to confirm re-provisioning is progressing.
5. Run R-24-C1 with `deprovisioned_at IS NOT NULL AND provisioned_at >= NOW() - INTERVAL '1 hour'` to track recovery count.
6. Confirm with tenant admin that their users can log in successfully.

**FORM-side root cause (H3 — FORM SCIM processing bug):**

1. Page platform-engineer; initiate a fix deployment via the standard change management process.
2. Do NOT re-enable SCIM sync until the fix is deployed and verified in staging.
3. Restore affected `tenant_members` rows by replaying provisioning events from DEC-030 history:

```sql
-- R-24-R1: Identify users who should be re-provisioned
-- (were active immediately before the incident window)

WITH pre_incident_active AS (
  SELECT DISTINCT
    ale.metadata->>'user_id'::uuid   AS user_id,
    ale.occurred_at                  AS last_provisioned_at
  FROM audit_log_events ale
  WHERE ale.tenant_id = $tenant_id
    AND ale.event_type = 'scim.user_provisioned'
    AND ale.occurred_at < $incident_start_time
    AND ale.occurred_at >= $incident_start_time - INTERVAL '90 days'
),
currently_deprovisioned AS (
  SELECT tm.user_id
  FROM tenant_members tm
  WHERE tm.tenant_id = $tenant_id
    AND tm.deprovisioned_at >= $incident_start_time
)
SELECT
  pia.user_id,
  pia.last_provisioned_at
FROM pre_incident_active pia
INNER JOIN currently_deprovisioned cd ON cd.user_id = pia.user_id
ORDER BY pia.last_provisioned_at DESC;
```

4. For each row returned by R-24-R1: manually re-provision via the admin provisioning API (`POST /internal/v1/admin/provision-user`) with `source: "R-24-recovery"` in the audit metadata.
5. Each manual re-provision emits a standard `scim.user_provisioned` DEC-030 event with `source: "incident_recovery"` for audit trail completeness.

---

#### R-24.9 DEC-030 HMAC-Chained Audit Events

All R-24 events are appended to the master DEC-030 chain. Privacy invariant: no individual user names, emails, or health data in any event payload — only UUIDs, counts, and operational metadata.

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `scim.mass_deprovision_detected` | **CRITICAL** | 7 yr | AL-SCIM-MASS-01 threshold breach | `tenant_id`, `deprovisioned_count`, `pct_of_seats`, `window_minutes`, `incident_id`, `detection_source` (`alert` / `csm_report` / `monitoring`) |
| `scim.sync_suspended` | **HIGH** | 7 yr | IC or platform-engineer executes SCIM pause | `tenant_id`, `suspended_by_admin_id`, `reason`, `auto_resume_at`, `incident_id` |
| `scim.admin_lockout_recovery` | **HIGH** | 7 yr | Magic link issued to tenant owner during admin lockout | `tenant_id`, `issued_by_admin_id`, `tenant_owner_email_hash` (SHA-256, not raw), `incident_id`, `magic_link_expires_at` |
| `scim.sync_resumed` | **HIGH** | 7 yr | SCIM suspension KV key removed after root cause resolved | `tenant_id`, `resumed_by_admin_id`, `root_cause_hypothesis` (H1/H2/H3/H4), `incident_id`, `users_restored_count` |
| `scim.mass_reprovision_complete` | **STANDARD** | 3 yr | Recovery confirmed: all deprovisioned users re-provisioned | `tenant_id`, `restored_count`, `unrestorable_count` (0 expected), `incident_id`, `recovery_method` (`idp_repush` / `form_manual` / `both`) |

**HMAC chain requirement:** `scim.mass_deprovision_detected` must precede `scim.sync_suspended` in the chain; `scim.sync_suspended` must precede `scim.sync_resumed`. If the chain is broken between these events, treat as R-05 (HMAC Chain Break) and alert security-engineer.

---

#### R-24.10 Evidence Preservation

Preserve within T+15 min of incident declaration. All evidence stored in `compliance/evidence/incident-scim-deprovision/<incident_id>/`.

```
compliance/evidence/incident-scim-deprovision/INC-YYYYMMDD-<slug>/
├── scope/
│   ├── r24-c1-scope-query-result.json      # R-24-C1 output (user counts only, no names)
│   ├── r24-c2-active-sessions.json         # R-24-C2 output
│   ├── r24-c3-role-distribution.json       # R-24-C3 output
│   └── r24-c4-dec030-chain.jsonl           # R-24-C4 output (HMAC chain segment)
├── idp-evidence/
│   ├── idp-audit-log-export.csv            # Provided by tenant IT admin
│   ├── scim-provisioning-config-before.png # Screenshot of IdP SCIM config at time of incident
│   └── scim-provisioning-config-after.png  # Screenshot after fix
├── comms/
│   ├── md-01-initial-notification.eml      # Template MD-01 sent copy
│   ├── md-02-updates.eml                   # All MD-02 updates
│   └── md-03-resolution.eml               # Template MD-03 sent copy
├── dec030/
│   └── scim-mass-deprovision-chain.jsonl   # All five DEC-030 events for this incident
└── MANIFEST.sha256                         # SHA-256 hash of every file (tamper evidence)
```

Generate manifest:
```bash
find . -type f -not -name MANIFEST.sha256 | sort | xargs sha256sum > MANIFEST.sha256
git add . && git commit -m "evidence: R-24 incident $INC_ID preserved"
```

**7-year retention** applies to all evidence files (aligned with `scim.mass_deprovision_detected` CRITICAL 7yr DEC-030 retention). Evidence directory is read-only once the incident PIR is closed — no further modifications permitted.

---

#### R-24.11 AL-SCIM-MASS-01 Alert Rule Definition

This alert rule does not exist until implemented per the R-24 implementation checklist. It must be registered in `docs/OBSERVABILITY.md §26` (SSO/SCIM Identity Observability) under the SCIM provisioning health subsection.

| Field | Value |
|---|---|
| Alert ID | `AL-SCIM-MASS-01` |
| Severity | **P0** (≥ 50% of seats) / **P1** (≥ 10% of seats or ≥ 100 users) |
| Condition | `scim.user_deprovisioned` DEC-030 event count for a single `tenant_id` exceeds `MAX(0.10 * tenant_seat_count, 10)` within any rolling 10-minute window |
| Measurement | pg_cron job `scim_mass_deprovision_check` (every 5 min) queries `audit_log_events WHERE event_type = 'scim.user_deprovisioned' AND occurred_at >= NOW() - INTERVAL '10 minutes'` grouped by `tenant_id`; joins to `tenants.seat_count` for percentage calculation |
| Response | Immediate PagerDuty P0/P1 → platform-engineer + CSM (Customer Lead); emit `scim.mass_deprovision_detected` DEC-030 CRITICAL before PagerDuty call |
| Dedup key | `scim-mass-deprovision-{tenant_id}-{date_hour}` — one alert per tenant per hour; re-alerts if a second burst occurs after 60 min |
| Auto-resolve | No — manual resolution only after IC confirms all users are restored |
| PagerDuty service | `form-customer-success` (primary CSM page) + `form-platform` (secondary) |
| Runbook link | This document §R-24 |

---

#### R-24.12 SOC 2 Evidence Mapping

| Criterion | Control | R-24 mechanism | Evidence artefact | Status |
|---|---|---|---|---|
| **A1.1 — Capacity management** | SCIM mass deprovision is an availability threat to enterprise tenants; AL-SCIM-MASS-01 detects it automatically before tenant reports it | AL-SCIM-MASS-01 alert + `scim.mass_deprovision_detected` DEC-030 event with detection timestamp | **SCIM-E-001:** DEC-030 event export showing detection preceded tenant complaint (demonstrates proactive monitoring) | 🟡 Authored — closes on first production event or staging drill |
| **A1.2 — Environmental threats** | Accidental mass deprovisioning is an operational availability threat from third-party IdP state changes; FORM detects, contains, and recovers without requiring tenant action | `scim.sync_suspended` + `scim.sync_resumed` DEC-030 chain proves FORM-side isolation controls; recovery SQL proves data integrity | **SCIM-E-002:** Full DEC-030 chain for any R-24 incident during observation period, including `scim.mass_reprovision_complete`; or staging tabletop drill chain | 🟡 Authored |
| **CC7.2 — System monitoring** | Continuous automated monitoring of SCIM provisioning event rate enables detection of abnormal deprovisioning before users are affected | AL-SCIM-MASS-01 pg_cron job 5-min cadence; `scim.mass_deprovision_detected` event as automated detection artefact | **SCIM-E-003:** PagerDuty incident log showing AL-SCIM-MASS-01 fires within 5 min of threshold breach (staging test); incident `opened_at` vs DEC-030 `occurred_at` delta < 5 min | 🟡 Authored |
| **CC7.3 — Anomaly response** | Defined escalation and recovery procedures (this runbook) with timing SLAs; magic link admin lockout recovery as a documented compensating control | R-24 runbook; Template MD-01 T+10 min SLA; admin lockout recovery procedure; SCIM suspension/resume controls | **SCIM-E-004:** Communications log (MD-01/MD-02/MD-03) with timestamps from any R-24 incident showing SLA adherence; or tabletop drill evidence | 🟡 Authored |
| **CC9.2 — Vendor management** | IdP is a third-party service whose configuration changes can affect FORM availability; FORM's containment mechanism (SCIM suspension) does not depend on IdP cooperation | Architectural: `scim:sync:paused` KV key takes effect immediately regardless of IdP state; FORM can restore provisioning state from DEC-030 event history if IdP is unresponsive | **SCIM-E-005:** `wrangler.toml` or KV binding configuration demonstrating the pause mechanism exists independently of IdP API availability | 🟡 Authored |

**Auditor evidence artefacts:**

| Artefact ID | Description | Collection method | Retention |
|---|---|---|---|
| **SCIM-E-001** | DEC-030 `scim.mass_deprovision_detected` event for R-24 incident or staging drill | `audit_log_events WHERE event_type = 'scim.mass_deprovision_detected'` + timestamp comparison vs tenant complaint log | 7 yr |
| **SCIM-E-002** | Full DEC-030 HMAC chain for R-24 incident: `scim.mass_deprovision_detected` → `scim.sync_suspended` → `scim.sync_resumed` → `scim.mass_reprovision_complete` | `audit_log_events WHERE event_type LIKE 'scim.%' AND metadata->>'incident_id' = '<id>'` | 7 yr |
| **SCIM-E-003** | AL-SCIM-MASS-01 PagerDuty incident showing fire-to-detection delta < 5 min | PagerDuty export: `opened_at`, `alert_key`, first DEC-030 `occurred_at` from matching chain | 7 yr |
| **SCIM-E-004** | Communication log (Templates MD-01/02/03) with timestamps demonstrating ≤ 10 min initial notification SLA | Email thread + Slack export from incident channel; timestamps in UTC | 7 yr |
| **SCIM-E-005** | SCIM pause mechanism technical evidence | `wrangler.toml` KV binding for `SCIM_KV` + `quota-check.ts` excerpt showing KV read before processing SCIM PUSH requests; confirms pause works regardless of IdP state | 3 yr |

Store all artefacts in `compliance/evidence/scim-mass-deprovision/` and cross-reference in `docs/SOC2_READINESS.md` §A1.2, §CC7.2, §CC7.3, §CC9.2 evidence tables when the first production event or tabletop drill is completed.

---

#### R-24.13 Post-Incident Controls

Per the §14 Continuous Improvement Program, the following controls are mandatory post-incident actions:

| Control | Trigger condition | Owner | SLA | Notes |
|---|---|---|---|---|
| Add SCIM filter expression review to tenant onboarding checklist | Any R-24 incident | customer-success | 7 days post-PIR close | Onboarding engineer to verify IdP SCIM filter scope with IT admin before first sync |
| Implement IdP SCIM configuration snapshot on first sync and on every config change | H1 or H2 root cause confirmed | platform-engineer | 30 days post-PIR | Snapshot stored in R2; enables diff against incident-time config to speed future diagnosis |
| Add SCIM batch DELETE threshold server-side (FORM-side rate limit) | H3 root cause confirmed (FORM SCIM processing bug) | platform-engineer | M5 | FORM SCIM endpoint rejects any single SCIM PATCH/DELETE batch that would deprovision > 20% of tenant's seats; returns HTTP 422 with `error: "mass_deprovision_threshold_exceeded"` and requires IC-signed override |
| Add tabletop Scenario K (SCIM mass deprovision) to §9.4 tabletop catalog | After first R-24 real incident, or at next quarterly IR drill | security-engineer | Quarterly drill schedule | Scenario: Okta group rule change deprovisioning 60% of a 500-seat enterprise tenant at 14:00 UTC on a Friday; IdP admin unreachable; test: detection → suspension → admin lockout recovery → full restoration timeline |

---

#### R-24.14 Open Questions

| OQ | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-R24-01** | **Should FORM implement a SCIM batch DELETE threshold at the endpoint level?** A server-side guard that rejects any incoming SCIM PUSH that would deprovision > 20% of a tenant's seats in a single request — returning HTTP 422 and requiring a CSM-countersigned override token — would prevent H3-type incidents and create a friction point for accidental IdP mass-deprovision pushes. Risk: breaks legitimate mass-termination scenarios (e.g., layoffs). Mitigation: the override path allows legitimate use with an audit trail. Recommendation: implement with a per-tenant configurable threshold (default 20%, adjustable by tenant owner with CSM approval). Requires update to `docs/SSO_SCIM_IMPLEMENTATION.md §15`. | **P1** | enterprise-architect + platform-engineer | Decide before first enterprise GA customer (M13); document in `docs/DECISION_LOG.md` |
| **OQ-R24-02** | **Should FORM provide an "undo deprovision" self-service button in the admin dashboard for tenant owners?** A time-windowed (e.g., 30-min) emergency re-provision action that restores users deprovisioned in the last N minutes, without requiring FORM CSM involvement. Risk: an attacker who compromises a tenant_owner account could use this to re-provision previously intentionally deprovisioned users. Mitigation: require MFA re-authentication for the "undo" action; log with DEC-030 CRITICAL. Recommendation: implement in M7+ as a Growth/Enterprise feature gated by `feature.scim_undo_deprovision`. | **P2** | enterprise-architect + security-engineer | Evaluate at first 3 enterprise customers; document outcome in `docs/DECISION_LOG.md` |
| **OQ-R24-03** | **What is FORM's GDPR Art. 33 obligation when a mass deprovisioning event is later determined to have been caused by a FORM-side bug (H3)?** If FORM's SCIM endpoint processed a DELETE request that exceeded its authorized scope (i.e., deleted users the IdP did not intend to deprovision), this is arguably an unauthorised alteration of personal data — potentially triggering Art. 33 notification duty. The key question: is loss of access (no data exfiltration, no disclosure) a "breach of security leading to... alteration" under Art. 4(12)? | **P1** | compliance-officer + outside counsel | Legal consultation (same engagement as OQ-V-02 if timing aligns); document in `docs/DECISION_LOG.md` before enterprise GA |

---

#### R-24.15 Implementation Checklist

| # | Action | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Implement AL-SCIM-MASS-01 as a pg_cron job `scim_mass_deprovision_check` (every 5 min): query `audit_log_events WHERE event_type = 'scim.user_deprovisioned' AND occurred_at >= NOW() - INTERVAL '10 minutes'` grouped by `tenant_id`; join to `tenants.seat_count` for percentage; emit `scim.mass_deprovision_detected` DEC-030 CRITICAL + PagerDuty P0/P1 on threshold breach. Add to `docs/OBSERVABILITY.md §12.6` pg_cron registry and §26 SCIM health alert table. | platform-engineer + devops-lead | **P0** | M5 | [ ] |
| 2 | Implement SCIM sync pause mechanism: `scim:sync:paused:{tenant_id}` Cloudflare KV key checked at SCIM endpoint entry; HTTP 503 on PUSH requests when key present; GET requests still served; KV write via internal admin console endpoint `POST /internal/v1/admin/scim/pause` with `reason` + `paused_by` fields; emit DEC-030 `scim.sync_suspended` on write. | platform-engineer | **P0** | M5 | [ ] |
| 3 | Register all five DEC-030 events from §R-24.9 in `docs/AUDIT_LOG_SCHEMA.md` event registry with Zod schemas; validate HMAC chain ordering constraint (`scim.mass_deprovision_detected` → `scim.sync_suspended` → `scim.sync_resumed`) in staging. | platform-engineer + compliance-officer | **P0** | M5 | [x] **Done — `docs/AUDIT_LOG_SCHEMA.md` v1.6 (2026-06-11)** |
| 4 | Implement `scim:sync:paused` KV read at SCIM PUSH endpoint entry (in `scim-provisioning-worker.ts`); confirm HTTP 503 is returned for POST/PATCH/DELETE SCIM requests when key present; GET requests return 200 normally; add unit test asserting pause is enforced before any DB writes. | platform-engineer | **P0** | M5 | [ ] |
| 5 | Add magic link admin lockout recovery endpoint `POST /internal/v1/admin/magic-link` (or extend existing magic link flow): requires `reason` + `incident_id` fields; emits `scim.admin_lockout_recovery` DEC-030 HIGH; logs `issued_by_admin_id` and `tenant_owner_email_hash` (not raw email); valid for 15 minutes. | platform-engineer + security-engineer | **P1** | M5 | [ ] |
| 6 | Add `scim_mass_deprovision_check` to `docs/OBSERVABILITY.md §12.6` pg_cron registry (resolve job number with devops-lead — next available slot after jobs 22/23). Add AL-SCIM-MASS-01 rows to §26 (SSO/SCIM Identity Observability) alert table. | devops-lead | **P1** | M5 | [ ] |
| 7 | Draft and review Templates MD-01, MD-02, MD-03 (§R-24.7) copy; review by brand-voice for tone; review by compliance-officer for GDPR-compliant aggregate-only framing (no individual names); final approval by founder. Store approved templates in `compliance/evidence/ir-templates/r24-md-01.txt` etc. | customer-success + brand-voice + compliance-officer | **P1** | M5 | [ ] |
| 8 | Add tabletop Scenario K to §9 drill catalog (per §R-24.13 post-incident controls): Okta group rule change deprovisioning 60% of 500-seat enterprise tenant; IdP admin unreachable; test detection → suspension → admin lockout recovery → full restoration timeline. Schedule as part of Q1 annual drill. | security-engineer + customer-success | **P2** | Q1 drill | [ ] |
| 9 | Collect SCIM-E-001 through SCIM-E-005 evidence artefacts (§R-24.12) using the §9 tabletop Scenario K drill (if no production R-24 incident has occurred by M6); store in `compliance/evidence/scim-mass-deprovision/`; cross-reference in `docs/SOC2_READINESS.md` A1.1, A1.2, CC7.2, CC7.3, CC9.2 evidence tables. | compliance-officer | **P1** | M6 | [ ] |
| 10 | Resolve OQ-R24-01: decide server-side batch DELETE threshold; document in `docs/DECISION_LOG.md`; if approved, add implementation spec to `docs/SSO_SCIM_IMPLEMENTATION.md §15` (SCIM 2.0 Groups Provisioning) as a §15.N sub-section. | enterprise-architect + platform-engineer | **P1** | M13 (before enterprise GA) | [ ] |
| 11 | Resolve OQ-R24-03 (GDPR Art. 33 obligation for H3 root cause): engage outside counsel (same engagement as OQ-V-02); document outcome in `docs/DECISION_LOG.md`. | compliance-officer + outside counsel | **P1** | M9 | [ ] |

---

*v2.0 additions (2026-06-11): R-24 SCIM Mass Deprovisioning Emergency — twenty-fourth runbook; closes the incident response gap for accidental or erroneous mass SCIM deprovisioning events from enterprise IdP integrations (Okta, Azure AD, Google Workspace, any SAML 2.0/OIDC provider with SCIM enabled). R-24 is classified as a data integrity and availability incident (distinct from R-04 SSO/SAML Compromise, R-19 IdP Outage, and R-01 Data Breach). Trigger matrix: four signal sources (AL-SCIM-MASS-01 automated threshold alert, tenant admin direct report, CSM health check, SCIM endpoint error log); severity table with P0 override for admin lockout and P0 at ≥ 50% seats in < 30 min. Immediate actions: T+0 to T+20 timeline (channel open, scope SQL, SCIM suspension, active session check, tenant notification). SCIM suspension mechanism: `scim:sync:paused:{tenant_id}` Cloudflare KV key (HTTP 503 on PUSH, GET unaffected); TypeScript implementation provided. Admin lockout sub-protocol: magic link recovery for locked-out tenant_owner with `tenant_owner_email_hash` in DEC-030 (no raw email). Root cause hypotheses H1–H4 (IdP group deletion, SCIM filter, FORM-side processing bug, IdP platform update) with per-hypothesis diagnostic signals and resolution paths. Four scope assessment SQL queries: R-24-C1 (deprovisioned count + % of seats + time window), R-24-C2 (active sessions for deprovisioned users), R-24-C3 (role distribution including admin lockout risk), R-24-C4 (DEC-030 chain with actor_id and scim_source validation). Three communication templates: MD-01 (T+10 min initial notification, aggregate count only, no individual names), MD-02 (30-min status update), MD-03 (resolution notice with SLA impact section). Recovery procedure: standard IdP-side recovery (KV resume after IdP fix) and FORM-side recovery (R-24-R1 restoration SQL for H3 root cause). Five DEC-030 HMAC-chained events: `scim.mass_deprovision_detected` (CRITICAL 7yr), `scim.sync_suspended` (HIGH 7yr), `scim.admin_lockout_recovery` (HIGH 7yr), `scim.sync_resumed` (HIGH 7yr), `scim.mass_reprovision_complete` (STANDARD 3yr); chain ordering requirement documented. Evidence structure: `compliance/evidence/incident-scim-deprovision/<incident_id>/` with five subdirectories (scope, idp-evidence, comms, dec030, MANIFEST.sha256). AL-SCIM-MASS-01 alert rule definition: pg_cron `scim_mass_deprovision_check` every 5 min, `MAX(0.10 * seat_count, 10)` threshold, dedup key per tenant per hour, PagerDuty `form-customer-success` + `form-platform` dual page. SOC 2 evidence mapping: A1.1 (availability threat monitoring — SCIM-E-001), A1.2 (environmental threat detection — SCIM-E-002 DEC-030 chain), CC7.2 (proactive monitoring — SCIM-E-003 AL-SCIM-MASS-01 PagerDuty log), CC7.3 (anomaly response — SCIM-E-004 communication SLA), CC9.2 (IdP as third-party risk — SCIM-E-005 pause mechanism independence). Post-incident controls: four controls including server-side batch DELETE threshold (OQ-R24-01 P1), admin dashboard "undo deprovision" (OQ-R24-02 P2), tabletop Scenario K (P2 Q1 drill), SCIM config snapshot on first sync (P1 30 days). Three open questions: OQ-R24-01 (server-side batch DELETE threshold — P1, enterprise-architect, M13), OQ-R24-02 (admin dashboard undo deprovision — P2, evaluate at 3 customers), OQ-R24-03 (GDPR Art. 33 for FORM-side H3 root cause — P1, outside counsel, M9). Eleven-item implementation checklist: 4× P0 M5 (AL-SCIM-MASS-01 pg_cron, SCIM pause KV mechanism, DEC-030 event registration, SCIM PUSH endpoint enforcement), 5× P1 M5-M13 (admin lockout magic link, OBSERVABILITY §12.6/§26 registry, templates, evidence collection, OQ-R24-01 decision, OQ-R24-03 legal consultation), 1× P2 Q1 (tabletop Scenario K). Cross-references: `docs/SSO_SCIM_IMPLEMENTATION.md §10` (magic link fallback), `docs/SSO_SCIM_IMPLEMENTATION.md §15` (SCIM 2.0 Groups Provisioning — batch DELETE threshold OQ-R24-01), `docs/SSO_SCIM_IMPLEMENTATION.md §22` (KV session revocation — session revocation decision point), `docs/DATA_MODEL.md §15` (SCIM provisioning webhook event queue), `docs/OBSERVABILITY.md §12.6` (pg_cron registry — scim_mass_deprovision_check job), `docs/OBSERVABILITY.md §26` (SSO/SCIM Identity Observability — AL-SCIM-MASS-01 alert row), `docs/AUDIT_LOG_SCHEMA.md` (five DEC-030 events to register), `docs/SOC2_READINESS.md §A1.2/CC7.2/CC7.3/CC9.2` (evidence artefacts SCIM-E-001 through SCIM-E-005), `docs/ENTERPRISE_SLA.md §4` (SLA credit calculation for P0 breach — referenced in Template MD-03). Owner: enterprise-architect + security-engineer + customer-success + compliance-officer.*

---

### R-25: GDPR Art. 9 Enterprise Tenant Off-boarding — Hard-Delete Overdue (AL-GDPR-03)

**Owner:** compliance-officer + platform-engineer
**Last updated:** 2026-06-12
**Related docs:** `docs/DATA_MODEL.md §25` (Tenant Off-boarding & Data Egress Schema), `docs/OBSERVABILITY.md §37` (GDPR Compliance Pipeline Observability), `docs/SOC2_READINESS.md §67` (Tenant Deletion Certificate), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 events)
**Cross-ref:** R-01 (Data Breach — activate in parallel if Art. 9 data was accessed by unauthorized party during delay), R-14 (DSAR Art. 17 consumer erasure — R-25 covers enterprise tenant off-boarding only)

---

#### R-25.1 Trigger Matrix

| Source | Signal | Severity at Trigger | Auto-paging |
|---|---|---|---|
| **AL-GDPR-03** (OBSERVABILITY §37.5) | pg_cron check finds `tenants.status = 'offboarding'` AND `art9_hard_delete_completed_at IS NULL` AND `offboarding_initiated_at < NOW() - INTERVAL '4 hours'` | **P0** — no-auto-resolve | PagerDuty `form-compliance` → compliance-officer |
| Tenant admin complaint | Customer reports employee data still accessible via export API after off-boarding confirmation | P0 | Page compliance-officer immediately |
| GDPR supervisory authority inquiry | DPA requests confirmation of Art. 9 data deletion following tenant off-boarding | P1 | compliance-officer + founder |
| Scheduled monthly evidence collection | `compliance/evidence/gdpr/` audit reveals overdue deletion in `data_subject_requests` table | P1 | Linear ticket; page only if > 72h overdue |

**Critical distinction from R-14 (DSAR Art. 17):** R-14 covers individual consumer-tier erasure requests. R-25 covers the enterprise *tenant* off-boarding pipeline — bulk deletion of Art. 9 data for all users of a deprovisioned tenant. Art. 9 health data (workout records, CV keypoints, body composition, coaching session content) must be hard-deleted within 4 hours of `tenants.offboarding_initiated_at` per GDPR-SLO-03 (DEC-036 zero-grace-period commitment).

---

#### R-25.2 Severity Classification

| Condition | Severity | GDPR Art. 33 Trigger |
|---|---|---|
| Art. 9 data not deleted > 4h after `offboarding_initiated_at`; data at rest in Supabase only, not accessible via API | **P0** | Monitor — Art. 33 unlikely if no access path exists; assess per §R-25.7 |
| Art. 9 data not deleted AND tenant admin can still retrieve individual user data via export API | **P0** | **Yes — 72h clock starts at detection** |
| Art. 9 data not deleted AND unauthorized third party accessed data during delay | **P0** | **Yes — immediate Art. 33; R-01 activates in parallel** |
| Hard-delete Worker crashed and is not retrying; data at rest only | **P0** | Unlikely if delay < 72h and no access path; reassess at T+72h |
| Deletion partially complete (> 50% rows removed) AND Worker currently retrying | **P1** | No — monitor for P0 upgrade if Worker stalls |
| Large tenant (> 10,000 users); delay > 4h but < 24h; Worker still running | **P1** → P0 at 24h | No — pending OQ-GDPR-OBS-02 large-tenant SLO; upgrade at 24h |

---

#### R-25.3 Immediate Actions (T+0 to T+20 min)

```
T+0:  Open incident channel: #inc-YYYYMMDD-gdpr-art9-delete
T+0:  Page: compliance-officer + platform-engineer simultaneously
T+0:  Note exact time of AL-GDPR-03 first fire — GDPR regulatory clock may have already started
T+2:  Run R-25-C1: identify affected tenant(s) and hours overdue
T+5:  Check art9_hard_delete Worker status:
        SELECT * FROM pg_cron.job_run_details
        WHERE jobname = 'enterprise_art9_hard_delete'
        ORDER BY start_time DESC LIMIT 10;
T+5:  Run R-25-C2: verify whether Art. 9 data is currently accessible via API
T+10: If data IS accessible to tenant admins → upgrade to P0 + open R-01 in parallel
T+10: If Worker is slow (large tenant) → apply R-25.2 large-tenant severity rule
T+15: Emit DEC-030 gdpr.art9_delete_overdue_detected (CRITICAL) BEFORE any remediation
      action — this is the evidentiary anchor for the regulatory response
T+20: Decision: manual retry per §R-25.6 Step 1 or escalate to founder for Art. 33 assessment
```

---

#### R-25.4 Scope Assessment Queries

```sql
-- R-25-C1: Which tenants are overdue for Art. 9 hard-delete?
SELECT
  tenant_id,
  status,
  offboarding_initiated_at,
  art9_hard_delete_completed_at,
  EXTRACT(EPOCH FROM (NOW() - offboarding_initiated_at)) / 3600 AS hours_overdue,
  seat_count
FROM tenants
WHERE status = 'offboarding'
  AND art9_hard_delete_completed_at IS NULL
  AND offboarding_initiated_at < NOW() - INTERVAL '4 hours'
ORDER BY offboarding_initiated_at ASC;

-- R-25-C2: How much Art. 9 data remains for the affected tenant?
-- Run per affected tenant_id
SELECT 'workout_sessions' AS table_name, COUNT(*) AS rows_remaining
  FROM workout_sessions WHERE tenant_id = '<tenant_id>'
UNION ALL
SELECT 'cv_sessions', COUNT(*)
  FROM cv_sessions WHERE tenant_id = '<tenant_id>'
UNION ALL
SELECT 'coaching_turns', COUNT(*)
  FROM coaching_turns
  WHERE user_id IN (SELECT id FROM users WHERE tenant_id = '<tenant_id>')
UNION ALL
SELECT 'body_metrics', COUNT(*)
  FROM body_metrics
  WHERE user_id IN (SELECT id FROM users WHERE tenant_id = '<tenant_id>')
ORDER BY table_name;

-- R-25-C3: Are tenant admin sessions still active post-offboarding?
SELECT
  s.id, s.user_id, s.tenant_id, s.created_at, s.expires_at,
  u.role
FROM enterprise_sessions s
JOIN users u ON u.id = s.user_id
WHERE s.tenant_id = '<tenant_id>'
  AND s.expires_at > NOW()
  AND s.revoked_at IS NULL;

-- R-25-C4: DEC-030 off-boarding audit chain for this tenant
SELECT event_type, occurred_at, actor_id,
       metadata->>'hours_overdue' AS hours_overdue
FROM audit_log_events
WHERE tenant_id = '<tenant_id>'
  AND event_type LIKE 'tenant.%' OR event_type LIKE 'gdpr.%'
ORDER BY occurred_at ASC;
```

---

#### R-25.5 Root Cause Hypotheses

| Hypothesis | Diagnostic Signal | Resolution |
|---|---|---|
| **H1 — Worker crash / uncaught exception** | `pg_cron.job_run_details` shows `status = 'failed'` or non-zero `return_value`; Sentry error from `enterprise-art9-hard-delete` Worker | Investigate Sentry; fix exception; trigger manual retry per §R-25.6 Step 1 |
| **H2 — Supabase IOPS exhaustion for large tenant** | Worker completed without error but rows remain; `pg_stat_activity` shows cascade DELETE sleeping; no Sentry error | Use chunked batch mode §R-25.6 Step 2; notify compliance-officer of SLO extension; see OQ-GDPR-OBS-02 |
| **H3 — Migration not yet deployed** | pg_cron shows `ERROR: column "offboarding_initiated_at" does not exist`; OBSERVABILITY §37.10 items 1/2 not yet executed | P0 deployment gap — notify founder; run the migration immediately before retrying |
| **H4 — Tenant `status` or `offboarding_initiated_at` not set** | `tenants.status != 'offboarding'` or `offboarding_initiated_at IS NULL` despite contractual off-boarding | Set manually with compliance-officer approval; emit `tenant.offboarding_initiated` DEC-030 before setting |
| **H5 — Audit log pseudonymisation conflict** | Worker refused to delete `audit_log_events` rows; OQ-GDPR-OBS-03 unresolved | Follow interim protocol §R-25.8 — do NOT delete audit log rows; continue deletion of application tables |

---

#### R-25.6 Recovery Procedure

**Step 1 — Standard Worker retry (H1)**
```sql
-- Manually trigger the pg_cron job (requires devops-lead access)
SELECT cron.run_job('<job_id_for_enterprise_art9_hard_delete>');
```

**Step 2 — Chunked batch DELETE (H2 — large tenant)**
```
⚠ Requires PAM break-glass escalation (DATA_MODEL.md §29) + IC + compliance-officer sign-off
⚠ Emit DEC-030 gdpr.art9_delete_manual_trigger BEFORE executing any SQL
```
```sql
-- Chunk deletion per table; 1,000-row batches; 100ms yield between batches
DO $$
DECLARE batch_size INT := 1000; rows_deleted INT;
BEGIN
  LOOP
    DELETE FROM workout_sessions
    WHERE id IN (SELECT id FROM workout_sessions
                 WHERE tenant_id = '<tenant_id>' LIMIT batch_size);
    GET DIAGNOSTICS rows_deleted = ROW_COUNT;
    EXIT WHEN rows_deleted = 0;
    PERFORM pg_sleep(0.1);
  END LOOP;
END $$;
-- Repeat for: cv_sessions, coaching_turns, body_metrics, and all other Art. 9 tables
-- per docs/DATA_MODEL.md §25 off-boarding table list
```

**Step 3 — Mark deletion complete**
```sql
UPDATE tenants
SET art9_hard_delete_completed_at = NOW(),
    status = 'offboarded'
WHERE tenant_id = '<tenant_id>';
-- Worker then emits DEC-030 gdpr.art9_delete_completed (HIGH, 7yr) automatically
```

---

#### R-25.7 GDPR Art. 33 Assessment Decision Tree

Art. 33 requires supervisory authority notification within 72 hours of **becoming aware** of a **breach of security** leading to unauthorised access to or loss of personal data. Evaluate at T+15 min:

| Question | Yes → | No → |
|---|---|---|
| Was Art. 9 data accessible to the tenant admin via API after `offboarding_initiated_at`? | **Art. 33 likely — 72h clock starts at detection; page founder** | Continue |
| Did any third party access Art. 9 data during the delay? | **Art. 33 required — R-01 activates; 72h clock from breach discovery** | Continue |
| Is the delay > 72h with data still present in the database? | **Art. 33 likely — GDPR Art. 5(1)(f) integrity obligation may be breached; founder + outside counsel decision required** | Continue |
| Delay < 4h AND no access path existed at any point? | Art. 33 unlikely — operational failure contained before threshold; document reasoning in PIR | Close |

If the Art. 33 decision is unclear, escalate to outside counsel immediately. Contact and decision authority: `compliance/p1/gov-request-policy.md §2`.

---

#### R-25.8 Interim Protocol: Audit Log Rows (OQ-GDPR-OBS-03)

> **⚠ Legal status: UNRESOLVED.** OQ-GDPR-OBS-03 (P0 · OBSERVABILITY.md §37.11) identifies a fundamental tension between DEC-030's 7-year retention requirement for audit events and GDPR Art. 9's deletion obligation, given that some audit events (e.g., `workout.session_completed`, `ai.coaching_turn_submitted`) contain health-adjacent `user_id` references. Deleting these rows would break the HMAC chain. Outside counsel confirmation is required before either deletion or pseudonymisation is implemented.

**Until OQ-GDPR-OBS-03 is resolved, apply the following during R-25:**

1. **Do NOT delete `audit_log_events` rows** for the off-boarded tenant. The HMAC chain must remain intact and verifiable for SOC 2 CC7.2 evidence.
2. **Verify tenant isolation holds:** the RLS policy for `form_api` and `tenant_admin` roles already prevents any query from reading another tenant's rows. No additional access revocation is needed for cross-tenant isolation.
3. **Emit `gdpr.audit_log_pending_art9_decision`** (STANDARD, 7yr) at T+15 min, before the PIR is finalised. This creates the chain-of-custody record that the legal question was identified and is pending resolution.
4. **Document in PIR:** reference OQ-GDPR-OBS-03 and state that `audit_log_events` rows for the tenant are retained with original `user_id` values intact, pending outside counsel guidance (target: M13 pre-GA).

**Recommended resolution path (pending counsel confirmation):** pseudonymise `user_id` in `audit_log_events` using `ERASURE_PSEUDONYM_SALT` (write-once key per CRYPTOGRAPHY_POLICY.md §5) — add `user_id_pseudonym TEXT NULL` column; set at off-boarding; RLS restricts original `user_id` column to `form_admin` + `compliance_reviewer` roles only. The HMAC chain remains valid as pseudonymisation is additive (does not modify the at-write hash).

---

#### R-25.9 Communication Templates

**Template ART9-01 — Initial notification (P0; send at T+10 min via Slack Connect and email):**
> FORM is contacting you regarding the completion of your data deletion request. We have identified that the automated deletion of health and fitness data for your tenant is taking longer than our standard 4-hour target. No data has left FORM-controlled systems, and employee data is not accessible to any other tenant or external party. Our compliance team is actively resolving this. We will send a status update within 60 minutes. Questions: enterprise@form.coach.

**Template ART9-02 — Status update (send every 60 min until resolved):**
> Status update [HH:MM UTC]: The deletion process is progressing. [N]% of records have been removed across [M] affected tables. Estimated completion: [T+X]. We will notify you upon completion with a deletion confirmation reference.

**Template ART9-03 — Deletion confirmation (send after Step 3 — completion):**
> Deletion complete [TIMESTAMP UTC]. Your tenant's GDPR Art. 9 health and fitness data has been removed from all FORM systems. DEC-030 audit event reference: `gdpr.art9_delete_completed` event ID [UUID]. A written deletion certificate per your contract is available on request within 5 business days. Reference: `docs/SOC2_READINESS.md §67`.

---

#### R-25.10 DEC-030 HMAC-Chained Audit Events

Chain ordering constraint: `gdpr.art9_delete_overdue_detected` → `gdpr.art9_delete_resumed` (if applicable) → `gdpr.art9_delete_completed`. `gdpr.art9_delete_manual_trigger` must precede any manual deletion SQL.

| Event type | Severity | Retention | Trigger | Privacy constraints |
|---|---|---|---|---|
| `gdpr.art9_delete_overdue_detected` | CRITICAL | 7 yr | AL-GDPR-03 fires; emit at T+15 before any remediation | `tenant_id` only; `hours_overdue` rounded to nearest hour; no `user_id`, no health data |
| `gdpr.art9_delete_resumed` | HIGH | 7 yr | Manual or automatic retry of hard-delete Worker begins | `tenant_id`, `triggered_by_actor_id`, `root_cause_hypothesis` (H1–H5 enum) |
| `gdpr.art9_delete_completed` | HIGH | 7 yr | `tenants.art9_hard_delete_completed_at` set; 0 rows across all Art. 9 tables confirmed | `tenant_id`, `total_hours_to_complete`, `rows_deleted_count` (aggregate only) |
| `gdpr.art9_delete_manual_trigger` | CRITICAL | 7 yr | IC + compliance-officer approve break-glass manual DELETE | `tenant_id`, `ic_actor_id`, `compliance_officer_actor_id`, `pam_session_id` |
| `gdpr.audit_log_pending_art9_decision` | STANDARD | 7 yr | OQ-GDPR-OBS-03 interim protocol (§R-25.8) — records that audit log rows are retained pending legal counsel decision | `tenant_id`, `oq_reference: "OQ-GDPR-OBS-03"`, `expected_resolution_milestone: "M13"` |

---

#### R-25.11 Evidence Preservation

Store all artefacts at `compliance/evidence/incident-gdpr-art9-delete/<incident_id>/` with `MANIFEST.sha256`.

| Artefact | Description | SOC 2 Criteria | Retention |
|---|---|---|---|
| **ART9-E-001** | DEC-030 `gdpr.art9_delete_overdue_detected` HMAC chain export for affected tenant | P5.2, CC7.2 | 7 yr |
| **ART9-E-002** | `pg_cron.job_run_details` export: all runs of `enterprise_art9_hard_delete` during incident window | CC7.2 | 7 yr |
| **ART9-E-003** | R-25-C2 query output before and after remediation: aggregate row counts per table (no individual `user_id` values) | P5.2, CC7.3 | 7 yr |
| **ART9-E-004** | Communication log: Templates ART9-01/02/03 with UTC timestamps; confirms tenant notification | P6.4, CC7.4 | 7 yr |
| **ART9-E-005** | Art. 33 assessment decision record: IC + compliance-officer written decision on whether Art. 33 applied; outside counsel sign-off if triggered | P4 | 7 yr |
| **ART9-E-006** | DEC-030 `gdpr.art9_delete_completed` event — final tamper-evident deletion certificate; referenced in written certificate issued to tenant per `docs/SOC2_READINESS.md §67` | P5.2, CC6.5 | 7 yr |

---

#### R-25.12 SOC 2 and GDPR Evidence Mapping

| Criterion | Evidence | Control statement |
|---|---|---|
| **CC7.2** — Monitoring | ART9-E-002 + AL-GDPR-03 PagerDuty alert history | AL-GDPR-03 automated monitoring detects overdue deletion within 1 hour of GDPR-SLO-03 breach |
| **CC7.3** — Anomaly response | ART9-E-003 (rows before/after) + DEC-030 chain | Incident response completes Art. 9 deletion as the recovery action |
| **CC7.4** — Security incident response | ART9-E-005 (Art. 33 assessment) | Compliance-officer assesses regulatory obligation and escalates if required |
| **CC6.5** — Logical access termination | ART9-E-006 (`gdpr.art9_delete_completed`) | Hard-delete is the terminal access revocation for off-boarded tenants |
| **P5.2** — Disposal of personal information | ART9-E-001 + ART9-E-003 + ART9-E-006 | Art. 9 data is hard-deleted from all application tables; deletion is HMAC-evidenced |
| **P4.0** — Privacy notification | ART9-E-004 | FORM notifies the tenant controller of deletion completion per GDPR controller-processor obligations |
| **GDPR Art. 5(1)(f)** — Integrity and confidentiality | DEC-030 chain + §R-25.8 interim protocol | HMAC chain integrity preserved; Art. 9 application data deleted; audit log pending legal resolution (OQ-GDPR-OBS-03) |

---

#### R-25.13 Post-Incident Controls

| Control | Trigger condition | Owner | SLA |
|---|---|---|---|
| Add tabletop Scenario M (Art. 9 hard-delete failure for 10,000-user tenant) to §9.4 catalog | Any production R-25 incident | security-engineer + devops-lead | 30 days post-PIR |
| Implement chunked batch deletion as the default Worker behavior (not only in recovery) | H2 root cause confirmed | platform-engineer | M6 |
| Implement `art9_delete_progress_pct` column on `tenants` table to track partial progress | H2 root cause confirmed | platform-engineer | M6 — closes OQ-GDPR-OBS-02 large-tenant tracking requirement |
| Add per-tenant Art. 9 SLO extension log (`tenant_art9_slo_extensions` with compliance-officer approval gate) | First large-tenant (> 10,000 seats) off-boarding | compliance-officer + enterprise-architect | M13 |

---

#### R-25.14 Open Questions

| OQ | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-GDPR-OBS-03** | **Should the Art. 9 off-boarding Worker hard-delete `audit_log_events` rows for the tenant's users?** DEC-030 requires 7-year retention; GDPR Art. 9 requires deletion of health-adjacent event data. Interim protocol (§R-25.8): retain rows; RLS prevents access; emit `gdpr.audit_log_pending_art9_decision`. Recommended path: add `user_id_pseudonym TEXT NULL` column; pseudonymise with `ERASURE_PSEUDONYM_SALT` at off-boarding; RLS restricts raw `user_id` to `form_admin` + `compliance_reviewer`; HMAC chain integrity preserved (pseudonymisation is additive). **Outside counsel confirmation required.** | **P0** | compliance-officer + legal + security-engineer | Resolve before first enterprise tenant off-boarding (M13 pre-GA); route to same counsel engagement as OQ-R24-03 |
| **OQ-R25-01** | **At what delay threshold does a hard-delete overrun automatically trigger GDPR Art. 33 notification?** Current position: Art. 33 applies only if data was accessible during the delay, not merely retained in a locked DB. Counsel may confirm that a 4h–24h delay with no unauthorized access does not constitute a "breach" under GDPR Art. 4(12). Without a clear threshold, the IC must apply the §R-25.7 decision tree manually on each incident. | **P1** | compliance-officer + outside counsel | Same counsel engagement as OQ-GDPR-OBS-03; document outcome in `docs/DECISION_LOG.md` before enterprise GA |

---

#### R-25.15 Implementation Checklist

| # | Action | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Register all five DEC-030 events from §R-25.10 in `docs/AUDIT_LOG_SCHEMA.md` event registry with Zod schemas; deploy to `emit-audit-event` Worker. | platform-engineer + compliance-officer | **P0** | M6 | [ ] |
| 2 | Implement enterprise Art. 9 off-boarding Worker (OBSERVABILITY §37.10 item 5): cascade DELETE per table list in DATA_MODEL.md §25; update `art9_hard_delete_completed_at`; emit `gdpr.art9_delete_completed`. | platform-engineer | **P0** | M6 | [ ] |
| 3 | Configure AL-GDPR-03 in PagerDuty `form-compliance` service: P0, no-auto-resolve, dedup key `art9-overdue-{tenant_id}`; test with synthetic overdue row in staging. | devops-lead | **P0** | M6 | [ ] |
| 4 | Resolve OQ-GDPR-OBS-03: engage outside counsel (same engagement as OQ-R24-03); if pseudonymisation approach confirmed, add `user_id_pseudonym TEXT NULL` migration; update `docs/DATA_MODEL.md §12` and `docs/AUDIT_LOG_SCHEMA.md`. | compliance-officer + legal + security-engineer | **P0** | M13 (pre-GA) | [ ] |
| 5 | Draft and store templates ART9-01/ART9-02/ART9-03 at `compliance/evidence/ir-templates/r25-art9-0N.txt`; review by brand-voice + compliance-officer before first enterprise off-boarding. | customer-success + brand-voice + compliance-officer | **P1** | M10 | [ ] |
| 6 | Add Scenario M (Art. 9 delete overrun) to §9.4 tabletop drill catalog; schedule in Q1 2027 annual IR drill. | security-engineer | **P1** | Q1 2027 | [ ] |
| 7 | Collect ART9-E-001 through ART9-E-006 artefacts after first production R-25 incident or Scenario M tabletop; store in `compliance/evidence/incident-gdpr-art9-delete/<drill_id>/`; cross-reference in `docs/SOC2_READINESS.md §70` and `§67`. | compliance-officer | **P1** | M13 or first off-boarding | [ ] |
| 8 | Resolve OQ-R25-01 (Art. 33 automatic threshold): document in `docs/DECISION_LOG.md` and update §R-25.7 with confirmed threshold. | compliance-officer + outside counsel | **P1** | M13 | [ ] |

---

### R-26: SCIM Deprovisioning Latency SLA Breach (AL-SCIM-LAT-01 / AL-SCIM-LAT-02)

> **Scope:** Covers incidents where individual SCIM DELETE requests (employee deprovisioning) exceed the latency commitments in `docs/ENTERPRISE_SLA.md §3.7` (OQ-SLA-03 resolved, v1.1 2026-06-13). This is a *latency and access-revocation* incident — distinct from R-24 (SCIM Mass Deprovisioning), which covers bulk deprovisioning of ≥ 10 % of a tenant's seats. R-26 triggers when a single SCIM DELETE takes > 5 minutes (AL-SCIM-LAT-02, P1 auto-opened) or when the P99 latency over a 1-hour rolling window exceeds 30 seconds (AL-SCIM-LAT-01, P2). Security significance: an employee whose IdP account has been deactivated but whose FORM sessions remain active represents an access control gap — a potential SOC 2 CC6.3 violation (access revocation on termination) if unresolved.
>
> **Not R-24:** R-26 is triggered per-event or by rolling-window P99 statistics. R-24 is triggered by a count threshold (≥ 10 % of seats deprovisioned in 10 minutes). If AL-SCIM-MASS-01 and AL-SCIM-LAT-02 fire simultaneously, treat as R-24 (higher severity) and close R-26 as a duplicate.

#### R-26.1 Trigger Matrix

| Alert | Source | Threshold | Auto-severity | PagerDuty service |
|---|---|---|---|---|
| **AL-SCIM-LAT-02** | Better Stack S-014 (`scim-deprovision-latency`) or real SCIM DELETE event | Any single SCIM DELETE > 5 minutes | **P1 auto-opened** | `form-platform` + `form-customer-success` |
| **AL-SCIM-LAT-01** | Better Stack S-014 — rolling P99 | P99 latency > 30 s over 1-hour rolling window | **P2** | `form-platform` |
| **Manual** | CSM reports production tenant employee still active post-SCIM DELETE | N/A | **P1** (presume breach) | `form-platform` + `form-customer-success` |

> **Probe S-014:** Better Stack synthetic probe fires a SCIM DELETE on a canary test user in each deployed region every 15 minutes. Real SCIM DELETEs on production tenants that exceed the threshold also trigger the same alert rules.

#### R-26.2 Severity Classification

| Condition | Severity | Escalation |
|---|---|---|
| Single SCIM DELETE > 5 min AND sessions not yet revoked | **P1** | IC + security-engineer + CSM immediately |
| Single SCIM DELETE > 5 min AND sessions confirmed revoked (latency only, no access window) | **P2** | IC + platform-engineer; CSM monitors tenant |
| P99 > 30 s (systemic, no single event > 5 min) | **P2** | IC + platform-engineer |
| S-014 canary probe fails only (no production tenant affected) | **P2** | IC + platform-engineer; escalate to P1 if ≥ 3 consecutive failures |
| Production tenant affected AND employee is being offboarded (active HR action) | **P1 elevated** | Add compliance-officer and tenant's named CSM to incident channel immediately |

#### R-26.3 Immediate Actions (T+0 to T+15 min)

```
T+0   PagerDuty P1/P2 fires. IC takes ownership of incident channel #ir-scim-latency-YYYYMMDD.
      Emit DEC-030 scim.deprovision_latency_breach (§R-26.7) as the first HMAC chain entry.

T+2   Determine: canary event or real production SCIM DELETE?
      Canary → probe S-014 in region X failed. No production tenant affected yet. Continue monitoring.
      Real   → check Cloudflare Workers logs for SCIM endpoint: which tenant_id, which scim_request_id?

T+3   If real production event:
        Run R-26-C1 (§R-26.4) to check whether the user's sessions are actually revoked.
        A latency spike does not necessarily mean sessions are still active — the Worker may have completed.

T+5   If sessions NOT revoked (R-26-C1 confirms active_sessions > 0):
        Manual session revocation via admin console:
          POST /internal/v1/admin/session/revoke-tenant-user
            { "tenant_id": "<id>", "scim_request_id": "<id>",
              "reason": "R-26 latency breach manual revoke — INC-YYYYMMDD" }
        Emit DEC-030 scim.deprovision_sessions_manual_revoke immediately after (§R-26.7).

T+7   Notify tenant CSM lead: send Template LAT-01 (§R-26.6) if affected user is a production tenant employee.
      Do NOT send LAT-01 for canary-probe-only incidents.

T+10  Check S-014 probe history: isolated (1 event) or systemic (≥ 3 events in 30 min)?
      Isolated  → proceed to §R-26.4 root cause analysis.
      Systemic  → escalate to P1 regardless of initial classification; page devops-lead.

T+15  Begin root cause analysis (§R-26.4). All immediate containment must be complete by T+15.
```

#### R-26.4 Root Cause Hypotheses and Scope Query

**Scope query R-26-C1 — Check session revocation status:**

```sql
-- Requires form_admin BYPASSRLS via pam-db-proxy (read_only PAM elevation)
-- Reason: "R-26 SCIM latency scope assessment — INC-YYYYMMDD"
-- Privacy: returns session count and timestamps only; no user_id in result set.
SELECT
  t.id                                              AS tenant_id,
  t.name                                            AS tenant_name,
  COUNT(s.id)                                       AS total_sessions,
  COUNT(CASE WHEN s.revoked_at IS NULL THEN 1 END)  AS active_sessions,
  MAX(u.scim_deprovisioned_at)                      AS deprovisioned_at,
  NOW() - MAX(u.scim_deprovisioned_at)              AS elapsed
FROM scim_requests sr
JOIN users u  ON u.id       = sr.subject_user_id AND u.tenant_id = sr.tenant_id
JOIN tenants t ON t.id      = sr.tenant_id
LEFT JOIN sessions s ON s.user_id = u.id AND s.revoked_at IS NULL
WHERE sr.id        = '<scim_request_id>'   -- from alert payload
  AND sr.tenant_id = '<tenant_id>'         -- belt-and-suspenders RLS enforcement
GROUP BY t.id, t.name;
```

> If `active_sessions > 0` after 5+ minutes: immediate manual revocation required (T+5 action above).

**Root cause hypotheses:**

| # | Hypothesis | Diagnostic signal | Immediate mitigation |
|---|---|---|---|
| **H1** | Postgres high load / lock contention on `users` table | `pg_stat_activity` — long-running lock holders on `users`; Supabase dashboard — connection saturation | Wait for lock to clear; if > 10 min, devops-lead kills blocking backend (`pg_terminate_backend`) |
| **H2** | Supabase Edge Function timeout / cold start | Worker invocation logs — function timed out or cold-started with > 3 s boot | Manual session revocation (T+5); re-invoke after warm-up |
| **H3** | Session-revocation Worker queue backlog | Cloudflare Queue depth for session-revocation FIFO consumer > 500 messages | Drain queue; manual revocation for affected user; devops-lead scales queue consumer concurrency |
| **H4** | HMAC audit log write contention | `emit-audit-event` Worker 429s or high P99 in Cloudflare Analytics | Retry with exponential backoff; investigate write throughput limit in `emit-audit-event` |
| **H5** | Region-specific Cloudflare routing degradation | S-014 probe fails in one region only; Cloudflare status page confirms edge degradation | Canary-only failure — no production access window. Monitor for recovery; P2 unless production traffic affected |

#### R-26.5 Recovery Procedure

**Step 1 — Confirm session revocation.**
Run R-26-C1. If `active_sessions = 0`: sessions were revoked despite probe latency. No access window existed. Document in PIR.

**Step 2 — If sessions still active.**
1. Execute manual revocation via `POST /internal/v1/admin/session/revoke-tenant-user`.
2. Re-run R-26-C1 to confirm `active_sessions = 0`.
3. Emit `scim.deprovision_sessions_manual_revoke` DEC-030 event (§R-26.7).

**Step 3 — Root cause fix.**
- H1: Investigate lock contention; review index on `users(tenant_id, scim_deprovisioned_at)`.
- H2: Investigate Edge Function cold-start behavior; increase min instances or add warmup cron.
- H3: Scale session-revocation queue consumer; increase `max_concurrency` in wrangler.toml.
- H4: Review `emit-audit-event` Worker write throughput limit; evaluate D1 batch write migration.
- H5: No FORM-side fix required; file Cloudflare incident report if persistent.

**Step 4 — Verify S-014 probe recovery.**
Probe S-014 should return green within 2 probe cycles (≤ 30 minutes) after root cause fix. Confirm with Better Stack dashboard before closing.

**Step 5 — Close incident.**
Emit `scim.deprovision_latency_resolved` DEC-030 event (§R-26.7). Update PagerDuty status. Schedule PIR if P1.

#### R-26.6 Communication Templates

**Template LAT-01 — Tenant notification (P1 real-production only; do not send for canary-only incidents):**
> FORM is contacting you regarding a brief delay we detected in processing a SCIM deprovisioning request for a member of your organisation [ref: INC-YYYYMMDD]. We confirmed that [the employee's sessions were revoked / the employee's sessions were manually revoked as a precaution]. No employee data was accessed by any external party during this period. Our engineering team is investigating the root cause. We will send a resolution confirmation within [60 minutes / 4 hours]. Questions: enterprise@form.coach.

> **When to send LAT-01:** Required for any P1 where `active_sessions > 0` was confirmed by R-26-C1 (an access window existed, even briefly). IC discretion for P1 where sessions were already revoked (no access window). Never send for canary-probe-only P2.

#### R-26.7 DEC-030 HMAC-Chained Audit Events

Chain ordering constraint: `scim.deprovision_latency_breach` must be the first event emitted for any `incident_id`. `scim.deprovision_sessions_manual_revoke` (if issued) must follow it. `scim.deprovision_latency_resolved` must be last and reference the same `incident_id`.

**Privacy invariant (all three events):** No `user_id`, no employee name or email in any payload. `scim_request_id` is an opaque UUID tied to the SCIM endpoint request log — it maps to a user only via `form_admin`-only RLS-gated tables. `tenant_id` is null for canary-probe events.

| Event type | Severity | Retention | Trigger | Payload |
|---|---|---|---|---|
| `scim.deprovision_latency_breach` | HIGH | 7 yr | AL-SCIM-LAT-02 fires, or manual P1 raised by CSM | `tenant_id` (null for canary), `scim_request_id`, `latency_ms`, `region`, `probe_type` (`real` \| `canary`), `sessions_revoked_at_detection` (bool), `incident_id` (UUID) |
| `scim.deprovision_sessions_manual_revoke` | HIGH | 7 yr | IC manually revokes sessions after R-26-C1 confirms `active_sessions > 0` | `tenant_id`, `scim_request_id`, `revoked_session_count` (integer), `manual_revoke_actor_id` (FORM internal UUID — not tenant employee), `incident_id` |
| `scim.deprovision_latency_resolved` | STANDARD | 3 yr | IC confirms S-014 probe green + R-26-C1 `active_sessions = 0`; incident closed | `tenant_id` (null for canary), `incident_id`, `resolution_method` (`auto` \| `manual_revoke` \| `probe_recovery`), `time_to_resolution_min` (float) |

**HMAC chain requirement (R26-CHAIN-01):** `scim.deprovision_latency_breach` must precede all other R-26 events for a given `incident_id`. A chain break among these events triggers R-05 (HMAC Chain Break) and immediate escalation to security-engineer.

#### R-26.8 Evidence Preservation

Store artefacts at `compliance/evidence/incident-scim-latency/<incident_id>/` with `MANIFEST.sha256`.

| Artefact | Description | SOC 2 Criteria | Retention |
|---|---|---|---|
| **LAT-E-001** | DEC-030 `scim.deprovision_latency_breach` HMAC chain export | CC6.3, CC7.2 | 7 yr |
| **LAT-E-002** | R-26-C1 query output: `active_sessions` before and after manual revocation (session count only — no `user_id`) | CC6.3 | 7 yr |
| **LAT-E-003** | Cloudflare Workers SCIM endpoint access log for the affected `scim_request_id` (P95 response time) | CC7.2 | 3 yr |
| **LAT-E-004** | Better Stack S-014 probe history export for incident window (± 30 min) | CC7.2 | 3 yr |
| **LAT-E-005** | Template LAT-01 send record with UTC timestamp (P1 real-production incidents only) | CC7.4 | 7 yr |

#### R-26.9 SOC 2 and ENTERPRISE_SLA Evidence Mapping

| Criterion | Evidence | Control statement |
|---|---|---|
| **CC6.3** — Removes access when appropriate | LAT-E-001 + LAT-E-002 | SCIM deprovisioning latency is monitored; manual revocation closes any access window within the P1 SLA window |
| **CC7.2** — Proactive monitoring | LAT-E-004 (S-014 probe) + LAT-E-003 | Better Stack S-014 detects latency breaches every 15 minutes per region; AL-SCIM-LAT-01/02 rules provide automated SLA breach detection |
| **CC7.3** — Responds to identified events | LAT-E-001 + LAT-E-002 + PIR | IC response (T+0 to T+15) and manual revocation demonstrate CC7.3 operational response |
| **ENTERPRISE_SLA §3.7** (OQ-SLA-03) | LAT-E-003 + LAT-E-004 | P99 < 30 s deactivation and < 60 s session revocation measured by S-014; breach events HMAC-evidenced per DEC-030 |

#### R-26.10 Post-Incident Controls

| Control | Trigger condition | Owner | SLA |
|---|---|---|---|
| Add Scenario N (SCIM deprovisioning latency breach) to §9.4 tabletop catalog | Any P1 R-26 production incident | security-engineer | 30 days post-PIR |
| Review session-revocation Worker concurrency limits | H3 confirmed as root cause | devops-lead + platform-engineer | Next sprint |
| Review Postgres index on `users(tenant_id, scim_deprovisioned_at)` | H1 confirmed as root cause | platform-engineer | Next sprint |

#### R-26.11 Implementation Checklist

| # | Action | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Register three DEC-030 events from §R-26.7 in `docs/AUDIT_LOG_SCHEMA.md` with Zod schemas; deploy to `emit-audit-event` Worker. | platform-engineer + compliance-officer | **P0** | M6 | [ ] |
| 2 | Configure AL-SCIM-LAT-01 (P2) and AL-SCIM-LAT-02 (P1) alert rules in Better Stack for probe S-014; verify PagerDuty routing (`form-platform` P2; `form-platform` + `form-customer-success` P1). | devops-lead | **P0** | M6 | [ ] |
| 3 | Implement `POST /internal/v1/admin/session/revoke-tenant-user` endpoint (if not already available from R-24 admin lockout tooling); requires `form_admin` PAM elevation; emits DEC-030 `scim.deprovision_sessions_manual_revoke`. | platform-engineer | **P1** | M6 | [ ] |
| 4 | Add Scenario N to §9.4 tabletop catalog; schedule in Q2 2027 drill set. | security-engineer | **P2** | Q2 2027 | [ ] |
| 5 | Store Template LAT-01 at `compliance/evidence/ir-templates/r26-lat-01.txt`; review by brand-voice + compliance-officer. | customer-success + brand-voice | **P2** | M10 | [ ] |

---

*v2.3 additions (2026-06-13): Two changes. (1) R-14.8 DEC-030 event table: five new DSAR events from `docs/ENTERPRISE_SLA.md §19.5` (v1.1 2026-06-13) added — `dsar.data_provided` (HIGH, 7yr — enterprise processor-side Art. 15 export provision, 24-hour SLO tracking), `dsar.deletion_soft` (HIGH, 7yr — Art. 17 step 1 soft delete), `dsar.deletion_confirmed` (HIGH, 7yr — hard-delete cascade complete, deletion certificate issued), `dsar.portability_export_completed` (STANDARD, 7yr — employee self-serve Art. 20 export), `dsar.offboarding_export_available` (STANDARD, 7yr — contract-end bulk export window open for tenant admin). Privacy invariant: `user_id_hash` (SHA-256) used in portability event; `tenant_id` only in enterprise events; no plaintext user identifiers. AUDIT_LOG_SCHEMA.md registration required (see updated R-14 checklist item 9). (2) R-26 SCIM Deprovisioning Latency SLA Breach — twenty-sixth runbook. Closes the incident-response gap created when `docs/ENTERPRISE_SLA.md §3.7` (v1.1, OQ-SLA-03 resolved) defined AL-SCIM-LAT-02 (P1 auto-open; any single SCIM DELETE > 5 min) with no corresponding runbook. Distinct from R-24 (mass deprovisioning — count threshold); R-26 is per-event latency. Security significance: SOC 2 CC6.3 (access revocation on termination) at risk if employee sessions remain active post-IdP deactivation. Trigger matrix: AL-SCIM-LAT-02 (P1 auto), AL-SCIM-LAT-01 (P2, P99 > 30 s over 1 h rolling), manual CSM report. Better Stack probe S-014 (`scim-deprovision-latency`) every 15 min per region. Severity table: P1 if sessions not revoked; P2 if sessions confirmed revoked or canary-only. Immediate actions T+0 to T+15 with DEC-030 emission at T+0 as evidentiary anchor. Scope query R-26-C1 (session revocation status — no `user_id` in result). Five root cause hypotheses H1–H5 (Postgres lock contention, Edge Function timeout, queue backlog, HMAC chain write contention, regional Cloudflare degradation). Manual revocation endpoint `POST /internal/v1/admin/session/revoke-tenant-user`. One communication template LAT-01 (P1 real-production only). Three DEC-030 HMAC-chained events: `scim.deprovision_latency_breach` (HIGH, 7yr, anchor), `scim.deprovision_sessions_manual_revoke` (HIGH, 7yr, if manual revocation required), `scim.deprovision_latency_resolved` (STANDARD, 3yr). Five evidence artefacts LAT-E-001 through LAT-E-005 mapped to SOC 2 CC6.3/CC7.2/CC7.3/ENTERPRISE_SLA §3.7. Three post-incident controls (Scenario N tabletop, queue concurrency review, Postgres index review). Five-item implementation checklist: 2× P0/M6 (DEC-030 registration, AL-SCIM-LAT alert rules), 1× P1/M6 (revoke endpoint), 2× P2 (Scenario N, LAT-01 template). Cross-references: `docs/ENTERPRISE_SLA.md §3.7` (OQ-SLA-03 — trigger and SLA commitment); `docs/AUDIT_LOG_SCHEMA.md` (three new DEC-030 events + five DSAR events to register); `docs/OBSERVABILITY.md` (S-014 probe definition, AL-SCIM-LAT-01/02 alert rules); R-24 (mass deprovisioning — sibling runbook); R-05 (HMAC chain break — chain break sub-protocol). Owner: security-engineer + platform-engineer + compliance-officer.*

---

### R-27: Evidence Collection Automation Failure — Three-Consecutive-Month Recovery

> **Scope:** Covers incidents where the `evidence-collection-cron` Cloudflare Worker fails to run for three or more consecutive calendar months, creating an evidence gap that poses a material SOC 2 Type II audit risk. This runbook was explicitly specified in `docs/SOC2_READINESS.md §81.12 OQ-EVD-02` and resolves that open question. A single-month failure is handled operationally by AL-EVD-01 (Slack `#compliance-alerts`) without activating this runbook. This runbook activates on the **third consecutive missing month**, which escalates the severity from an operational annoyance to a compliance risk requiring a documented incident response and manual evidence backfill.
>
> **Not R-05 (HMAC Chain Break):** AL-EVD-02 (`audit.chain_integrity_violation`) is a separate incident triggered by a chain integrity check failure, not by missing Worker runs. R-27 covers scheduling and execution failures, not chain integrity failures. If both AL-EVD-01 (missing cron) and AL-EVD-02 (chain break) fire simultaneously, activate R-05 as the primary incident; R-27 becomes a secondary track for evidence backfill.
>
> **Not R-03 (Infrastructure Outage):** If the cron failure is attributable to a Cloudflare Workers platform outage, activate R-03 for infrastructure remediation and use R-27 for the evidence backfill aftermath only.

#### R-27.1 Trigger Matrix

| Alert / Trigger | Source | Threshold | Auto-severity | PagerDuty service |
|---|---|---|---|---|
| **AL-EVD-01 × 3 consecutive months** | pg_cron job 33 `evidence_cron_freshness_check` (03:05 UTC on 2nd of each month) | No `system.evidence_collection_automated` event for the third consecutive period | **P1 — escalated from P2** on 3rd fire | PagerDuty `form-compliance` (HIGH urgency) |
| **AL-EVD-03** | `evidence-collection-cron` Worker R2 write failure | Worker's `writeToR2` throws on any required artefact path for 3+ months | **P1** | PagerDuty `form-platform` |
| **Manual** | Compliance officer notices missing evidence during quarterly audit prep | Two or more periods absent from `MASTER-INDEX-YYYY.csv` | **P1** | `form-compliance` |

> **Escalation note:** The first and second AL-EVD-01 alerts (P2, Slack `#compliance-alerts`) should be handled operationally without activating this runbook — re-run the Worker manually and confirm the missed period is backfilled. This runbook activates only when the compliance-officer confirms three consecutive periods are missing from the MASTER-INDEX or from `audit_log_events WHERE action = 'system.evidence_collection_automated'`. At that point the severity escalates to P1 because three missed months during a 12-month observation period constitutes a SOC 2 evidence gap that may prevent audit sign-off.

#### R-27.2 Severity Classification

| Condition | Severity | Escalation |
|---|---|---|
| Three or more consecutive months missing AND observation period has not yet started | **P1** | IC + compliance-officer + devops-lead; no auditor notification required yet |
| Three or more consecutive months missing AND observation period is active | **P1 elevated** | IC + compliance-officer + founder; assess whether to notify auditor of compensating control (manual backfill) |
| Evidence gap spans dates of a pending auditor fieldwork window | **P0** | Immediate: compliance-officer + founder + outside counsel; notify auditor proactively |
| AL-EVD-03 (R2 write failure) is the confirmed root cause AND no periods are permanently missing | **P1 → P2 on R2 recovery** | Downgrade to P2 once R2 is verified accessible and all artefacts re-written |

#### R-27.3 Immediate Actions (T+0 to T+30 min)

```
T+0   AL-EVD-01 third consecutive fire escalates to P1. PagerDuty `form-compliance` pages IC.
      IC opens incident channel #ir-evidence-automation-YYYYMMDD.
      Emit DEC-030 system.evidence_automation_failure_declared (§R-27.7) as first chain entry.
      Record: which months are confirmed missing? (Run R-27-C1 below.)

T+5   Run R-27-C1 (§R-27.4): count missing periods from audit_log_events.
      Run R-27-C2: check Cloudflare Cron Dashboard for evidence-collection-cron run history.
      Cross-check MASTER-INDEX-YYYY.csv: GET compliance/evidence/MASTER-INDEX-YYYY.csv from R2.
      Reconcile: if R-27-C1 shows N missing but MASTER-INDEX shows only M missing, determine
      whether some runs completed partially (artefacts filed but DEC-030 event not emitted).

T+10  Determine root cause category (§R-27.4 H1–H5).
      If R2 bucket unavailable (AL-EVD-03): activate R-03 (Infrastructure Outage) in parallel.
      If chain integrity failure present (AL-EVD-02): activate R-05 (HMAC Chain Break) in parallel.
      Do NOT proceed to manual collection until R-03/R-05 are stabilised — manual writes to a
      degraded R2 or a broken chain will create additional evidence gaps.

T+15  Assess: can the Worker be re-run immediately to backfill the missing months?
      YES → proceed to §R-27.5 Step 1 (automated backfill attempt).
      NO  → proceed directly to §R-27.5 Step 2 (manual collection per §79).

T+20  If observation period is active: compliance-officer drafts compensating control memo
      (Template EVD-01, §R-27.6) documenting the gap, root cause, and backfill plan.
      This memo will be filed as EVD-COMP-E-001 (§R-27.8) and pre-empts auditor questions.

T+30  All immediate actions complete. Begin recovery procedure (§R-27.5).
```

#### R-27.4 Root Cause Hypotheses and Scope Queries

**R-27-C1 — Identify missing periods:**

```sql
-- Requires form_admin BYPASSRLS via pam-db-proxy (read_only PAM elevation)
-- Reason: "R-27 evidence automation failure scope assessment — INC-YYYYMMDD"
-- Privacy: no user data; event metadata only
SELECT
  TO_CHAR(gs.month, 'YYYY-MM')               AS expected_period,
  COUNT(ale.id)                               AS events_found,
  MAX(ale.created_at)                         AS last_run_at
FROM generate_series(
  DATE_TRUNC('month', NOW() - INTERVAL '6 months'),
  DATE_TRUNC('month', NOW() - INTERVAL '1 month'),
  INTERVAL '1 month'
) AS gs(month)
LEFT JOIN audit_log_events ale
  ON ale.action = 'system.evidence_collection_automated'
 AND ale.created_at >= gs.month
 AND ale.created_at <  gs.month + INTERVAL '1 month'
GROUP BY gs.month
ORDER BY gs.month;
```

> Rows where `events_found = 0` are confirmed evidence gaps. Report count and which periods to the incident channel.

**R-27-C2 — Check R2 artefact presence for a specific missing period:**

```
1. Access Cloudflare R2 → form-soc2-evidence → compliance/evidence/a1/
2. Check for: A1-E-001-{YYYY-MM}.json  (existence and size > 0)
3. Access compliance/evidence/cc7/
4. Check for: CC7-E-001-{YYYY-MM}.json (existence and size > 0)
5. If artefacts present but DEC-030 event missing: the Worker filed artefacts but crashed
   before emitting the chain event — treat as a partial failure (H4 below).
```

**Root cause hypotheses:**

| # | Hypothesis | Diagnostic signal | Immediate mitigation |
|---|---|---|---|
| **H1** | Cloudflare Cron trigger disabled or removed from `wrangler.toml` | Cloudflare Dashboard → Workers → `evidence-collection-cron` → Triggers: no active cron trigger listed | Re-add cron trigger `0 1 1 * *`; deploy Worker; trigger manual run |
| **H2** | Worker runtime error (unhandled exception on every run) | Cloudflare Workers Logs for `evidence-collection-cron`: `Error: ...` in last 3 monthly run windows | Fix the exception; re-deploy Worker; trigger manual runs for each missing period |
| **H3** | R2 bucket write permission revoked or bucket deleted | AL-EVD-03 history; Cloudflare R2 → `form-soc2-evidence` → 404 or 403 on GET | Restore R2 bucket permissions; verify `EVIDENCE_VAULT` binding in Worker config; re-run |
| **H4** | Worker crashed after R2 write but before DEC-030 emission | R2 artefacts present for the period; no `system.evidence_collection_automated` in `audit_log_events` | Emit `system.evidence_manual_collection_completed` manually via admin API (compliance-officer); `artefacts_written` populated; `manual_trigger: true` |
| **H5** | MASTER-INDEX ETag conflict (AL-EVD-04 repeated across multiple months) | Three or more `system.evidence_cron_conflict` events in `audit_log_events` covering the missing periods | Manual MASTER-INDEX reconciliation per §81.7; implement delta-log if triggered OQ-EVD-01 resolution |

#### R-27.5 Recovery Procedure

**Step 1 — Attempt automated backfill.**

If the Worker is deployable (H1 or H2 resolved):
1. Re-deploy `evidence-collection-cron` with the fix applied.
2. For each missing period `{YYYY-MM}`, trigger the Worker manually via Cloudflare Dashboard → Workers → `evidence-collection-cron` → Send test event, with body:
   ```json
   { "period": "{YYYY-MM}", "dry_run": false, "manual_trigger": true }
   ```
3. After each run, confirm via R-27-C1 that `system.evidence_collection_automated` now appears for that period in `audit_log_events` with `chain_integrity_ok: true`.
4. Verify MASTER-INDEX updated: `MASTER-INDEX-YYYY.csv` must contain rows for the backfilled periods with `status = COLLECTED` and `sha256` populated.
5. Confirm via R-27-C2 that both `A1-E-001-{YYYY-MM}.json` and `CC7-E-001-{YYYY-MM}.json` exist and are non-empty.

**Step 2 — Manual collection (if Worker cannot be immediately repaired).**

If Step 1 is not possible within the P1 resolution SLA (4 hours), follow `docs/SOC2_READINESS.md §79` manual-periodic procedures for each missing period.

For each missing period `{YYYY-MM}`:
1. Open `docs/SOC2_READINESS.md §79.5` (periodic evidence collection calendar) and identify which evidence items were due for that period.
2. Collect `A1-E-001-{YYYY-MM}` manually:
   - Source: Better Stack SLA report export for the month.
   - File to: `compliance/evidence/a1/A1-E-001-{YYYY-MM}-MANUAL.json`
3. Collect `CC7-E-001-{YYYY-MM}` manually:
   ```sql
   SELECT action, created_at, severity
   FROM audit_log_events
   WHERE created_at >= '{YYYY-MM}-01'::date
     AND created_at <  '{YYYY+1 or MM+1}-01'::date
   ORDER BY created_at;
   ```
   - File to: `compliance/evidence/cc7/CC7-E-001-{YYYY-MM}-MANUAL.json`
4. Update `MASTER-INDEX-YYYY.csv`: add rows for each artefact with `status = MANUAL_COLLECTION`, `collected_by = compliance-officer`, `collection_note = R-27 manual backfill — INC-YYYYMMDD`.
5. Emit `system.evidence_manual_collection_completed` DEC-030 event (§R-27.7) for each period backfilled. One event per period.
6. Sign the compensating control memo (Template EVD-01, §R-27.6) and file as `EVD-COMP-E-001-{YYYY-MM}.md`.

**Step 3 — Root cause fix and restoration.**

After manual collection closes the evidence gap, fix the underlying root cause:
- H1: Restore cron trigger in `wrangler.toml` (`0 1 1 * *`); deploy; confirm next scheduled run fires.
- H2: Fix unhandled exception; add integration test for the failing code path; deploy.
- H3: Restore R2 bucket permissions; verify `EVIDENCE_VAULT` binding; run synthetic dry-run.
- H4: Investigate why the crash occurred post-write; add try/catch with DEC-030 fallback emission.
- H5: Resolve MASTER-INDEX conflict per §81.7 (ETag guard procedure); document OQ-EVD-01 decision in `docs/DECISION_LOG.md` if `system.evidence_cron_conflict` events > 3.

**Step 4 — Confirm restoration.**

After the Worker runs successfully for the first month post-incident:
1. Confirm `system.evidence_collection_automated` DEC-030 event emitted with `manual_trigger: false`.
2. Emit `system.evidence_automation_restored` DEC-030 event (§R-27.7).
3. Update incident status: RESOLVED. Schedule PIR within 7 days.

#### R-27.6 Communication Templates

**Template EVD-01 — Compensating control memo (internal; required for every manual collection):**

> **FORM Compliance Memo — Evidence Automation Failure Compensating Control**
>
> Incident: INC-YYYYMMDD
> Affected periods: {list of YYYY-MM periods}
> Root cause: {H1–H5 category and description}
> Gap window: {first missing period} through {last missing period}
>
> **Compensating control:** Manual evidence collection was performed for all affected periods per `docs/SOC2_READINESS.md §79`. Evidence artefacts are filed at `compliance/evidence/a1/` and `compliance/evidence/cc7/` with `MANUAL_COLLECTION` status in `MASTER-INDEX-{YYYY}.csv`. DEC-030 `system.evidence_manual_collection_completed` events are HMAC-chained for each period.
>
> **Root cause remediation:** {describe fix; date deployed or expected completion}
>
> **Auditor disclosure recommendation:** {Disclose / No disclosure required — see §R-27.2 severity table}
>
> Signed: {compliance-officer} · {date}

> **When to send EVD-01:** Required whenever Step 2 (manual collection) is performed. One memo per incident covers all missed periods. If the observation period is active, share with the auditor as a proactive disclosure to establish the compensating control trail before fieldwork begins.

**Template EVD-02 — Auditor notification (P0 only; observation period active; fieldwork imminent):**

> We are writing to inform you of an incident (INC-YYYYMMDD) affecting our automated evidence collection layer. The `evidence-collection-cron` Worker did not run for {N} consecutive months ({periods}). We have completed manual evidence collection for all affected periods per our evidence collection protocol (`docs/SOC2_READINESS.md §79`). Evidence artefacts are in the expected R2 paths with `MANUAL_COLLECTION` status in the MASTER-INDEX. Root cause: {brief description}; fix deployed {date}. DEC-030 chain events confirm manual collection completeness. We are prepared to walk through the backfill artefacts at the start of fieldwork. Questions: compliance@form.coach.

#### R-27.7 DEC-030 HMAC-Chained Audit Events

Chain ordering constraint: `system.evidence_automation_failure_declared` must be the first event emitted for a given `incident_id`. For each backfilled period, `system.evidence_manual_collection_completed` must follow the declaration. `system.evidence_automation_restored` must be last and emitted only after the first successful automated (non-manual) cron run post-fix.

**Privacy invariant (all three events):** No user PII, no health data, no `user_id`. `tenant_id` is null — these are system-level compliance events. Only period labels (`YYYY-MM`), artefact R2 paths, and integrity check results appear in payloads.

| Event type | Severity | Retention | Trigger | Payload |
|---|---|---|---|---|
| `system.evidence_automation_failure_declared` | HIGH | 7 yr | IC declares P1 incident after 3rd consecutive AL-EVD-01; this runbook activated | `{ incident_id, missing_periods: string[], consecutive_misses: number, first_missing_period: "YYYY-MM", trigger_alert: z.enum(["AL-EVD-01","AL-EVD-03","manual"]) }` |
| `system.evidence_manual_collection_completed` | STANDARD | 7 yr | Step 2 manual collection complete for one period; one event emitted per period backfilled | `{ incident_id, period: "YYYY-MM", artefacts_written: string[], master_index_updated: boolean, collector: "compliance-officer", manual_trigger: true }` |
| `system.evidence_automation_restored` | STANDARD | 3 yr | First successful automated cron run post-fix confirms `manual_trigger: false` in `system.evidence_collection_automated` | `{ incident_id, restored_at_period: "YYYY-MM", root_cause_category: z.enum(["cron_disabled","worker_crash","r2_permission","partial_crash","etag_conflict"]), fix_deployed_at: z.string().datetime() }` |

**EVD-CHAIN-01 ordering invariant (R-27 extension):** For any `incident_id` in this runbook, `system.evidence_automation_failure_declared` must precede all `system.evidence_manual_collection_completed` events and `system.evidence_automation_restored`. A chain where `automation_restored` precedes the `failure_declared` for the same `incident_id` is a chain ordering violation that triggers R-05.

#### R-27.8 Evidence Preservation

Store artefacts at `compliance/evidence/incident-evidence-automation/<incident_id>/` with `MANIFEST.sha256`.

| Artefact | Description | SOC 2 Criteria | Retention |
|---|---|---|---|
| **EVD-E-001** | DEC-030 `system.evidence_automation_failure_declared` HMAC chain export | CC4.1, CC4.2 | 7 yr |
| **EVD-E-002** | R-27-C1 query output: table of expected periods vs. `events_found` (shows which months were missing) | CC4.2 | 7 yr |
| **EVD-E-003** | Cloudflare Workers log export for `evidence-collection-cron` covering the failure window (crash stack traces / cron trigger absence) | CC4.2, CC7.2 | 3 yr |
| **EVD-E-004** | Manual collection artefacts for each backfilled period: `A1-E-001-{YYYY-MM}-MANUAL.json` and `CC7-E-001-{YYYY-MM}-MANUAL.json` filed at canonical R2 paths | A1.1, CC7.1 | 7 yr |
| **EVD-E-005** | DEC-030 `system.evidence_manual_collection_completed` chain exports — one per period | CC4.2 | 7 yr |
| **EVD-COMP-E-001** | Signed compensating control memo (Template EVD-01) per missed-period batch; filed to `compliance/evidence/cc4/EVD-COMP-E-001-{YYYY-MM}-INC-YYYYMMDD.md`; uploaded to Vanta | CC4.1, CC4.2 | 7 yr |

#### R-27.9 SOC 2 Evidence Mapping

| Criterion | Evidence | Control statement |
|---|---|---|
| **CC4.1** — Control environment: monitoring deficiency detection | EVD-E-001 + EVD-COMP-E-001 | Automation failure detected within 1 month by AL-EVD-01 dead-man's switch; third consecutive failure triggers P1 escalation; compensating control memo demonstrates management response and accountability |
| **CC4.2** — Ongoing monitoring of controls | EVD-E-002 + EVD-E-004 + EVD-E-005 | Manual collection backfills every evidence gap; `MANUAL_COLLECTION` MASTER-INDEX status transparently distinguishes automated from manual artefacts; three-event DEC-030 chain proves the full recovery lifecycle |
| **CC7.2** — Anomaly detection | EVD-E-003 (Worker crash / cron trigger logs) | Cloudflare Workers log retention confirms the failure mode was detected and documented; AL-EVD-01 dead-man's switch is the automated detective control |
| **CC7.3** — Responds to identified events | EVD-E-001 + EVD-COMP-E-001 | `system.evidence_automation_failure_declared` at T+0; recovery steps completed within P1 SLA; root cause fixed before next cron window |
| **A1.1** — Availability commitments met | EVD-E-004 (`A1-E-001-{YYYY-MM}-MANUAL.json` per missed period) | SLA reports collected manually confirm availability commitments were met during the months the automation was absent; no gap in the A1.1 evidence time series |

**Auditor narrative for CC4.2:** FORM's evidence automation layer experienced a failure affecting {N} consecutive months. The dead-man's switch (AL-EVD-01, pg_cron job 33) detected the first missed run within 24 hours of the expected run time. After three consecutive failures, the compliance-officer activated R-27, declared a formal incident, and completed manual evidence collection for all affected periods following `docs/SOC2_READINESS.md §79`. EVD-COMP-E-001 provides the signed compensating control statement. The `MANUAL_COLLECTION` MASTER-INDEX status transparently distinguishes manually-filed artefacts from automated ones for auditor review. `system.evidence_automation_restored` confirms the Worker was repaired and ran successfully before the end of the observation period.

#### R-27.10 Post-Incident Controls

| Control | Trigger condition | Owner | SLA |
|---|---|---|---|
| Add Scenario P (evidence cron failure — manual backfill) to §9.4 tabletop catalog | Any activation of R-27 | security-engineer + compliance-officer | 30 days post-PIR |
| Review OQ-EVD-01 (delta-log vs. full-rewrite MASTER-INDEX) and document decision in `docs/DECISION_LOG.md` | AL-EVD-04 confirmed as root cause (H5) | devops-lead + compliance-officer | Next sprint |
| Add CI integration test for `evidence-collection-cron` end-to-end (mock R2 write + DEC-030 emission) | H2 (Worker crash) confirmed as root cause | devops-lead + platform-engineer | 14 days post-PIR |
| Add quarterly synthetic dry-run: trigger Worker with `{ "period": "test-dry-run", "dry_run": true }` on first Monday of Jan/Apr/Jul/Oct; confirm successful completion | Scheduled quarterly regardless of incidents | devops-lead | Quarterly |

#### R-27.11 Implementation Checklist

| # | Action | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Register three DEC-030 events from §R-27.7 in `docs/AUDIT_LOG_SCHEMA.md §6.System-Compliance` with Zod schemas; deploy to `emit-audit-event` Worker event registry. | platform-engineer + compliance-officer | **P0** | Month O-1 | [ ] |
| 2 | Configure AL-EVD-01 third-consecutive-month P1 escalation: extend pg_cron job 33 SQL to count `system.evidence_cron_stale` events in `audit_log_events` for the rolling 90-day window; on count ≥ 3, auto-escalate to PagerDuty `form-compliance` HIGH urgency via `emit_evidence_automation_failure()` Edge Function. | devops-lead + platform-engineer | **P0** | Month O-1 | [ ] |
| 3 | Store Template EVD-01 at `compliance/evidence/ir-templates/r27-evd-01.md`; Template EVD-02 at `compliance/evidence/ir-templates/r27-evd-02.md`. Review by compliance-officer. | compliance-officer | **P1** | Month O-1 | [ ] |
| 4 | Add Scenario P (evidence cron failure) to §9.4 tabletop catalog; schedule in Q4 2026 drill set (pre-observation-period). | security-engineer + compliance-officer | **P1** | Month O-1 | [ ] |
| 5 | Add quarterly synthetic dry-run calendar entry to `compliance/calendar/` (first Monday of Jan/Apr/Jul/Oct); document procedure at `compliance/evidence/ir-templates/r27-dry-run.md`. | devops-lead | **P2** | Month O+1 | [ ] |
| 6 | Update `docs/SOC2_READINESS.md §81.12 OQ-EVD-02` status to 🟢 Resolved with reference to this runbook (R-27). | compliance-officer | **P2** | Month O-1 | [ ] |

---

*v2.5 additions (2026-06-14): R-27 Evidence Collection Automation Failure — twenty-seventh runbook. Closes OQ-EVD-02 (P2, security-engineer + compliance-officer, Month O-1) from `docs/SOC2_READINESS.md §81.12`, which specified "propose R-27 runbook structure in INCIDENT_RESPONSE.md covering evidence automation failure recovery." Trigger: three consecutive AL-EVD-01 fires (pg_cron job 33 dead-man's switch; `evidence-collection-cron` Cloudflare Worker absent for 3+ calendar months). Single-month and double-month failures handled operationally by P2 Slack alert; runbook activates only on third consecutive miss. Severity table: P1 for 3+ misses pre-observation; P1 elevated if observation period active; P0 if fieldwork window directly affected. Not R-05 (chain break) or R-03 (infrastructure outage) — those activate in parallel if AL-EVD-02 or R2 bucket is down. Immediate actions T+0 to T+30: DEC-030 anchor `system.evidence_automation_failure_declared` at T+0; R-27-C1 scope query (missing-period count from `audit_log_events` using `generate_series` LEFT JOIN); R-27-C2 R2 artefact presence check; parallel R-03/R-05 coordination gate at T+10; compensating control memo at T+20 if observation active. Five root cause hypotheses: H1 (cron trigger disabled — Cloudflare Dashboard); H2 (Worker runtime exception — Workers Logs); H3 (R2 write permission revoked / bucket deleted — AL-EVD-03); H4 (partial crash: R2 artefacts present but DEC-030 not emitted); H5 (MASTER-INDEX ETag conflict — AL-EVD-04 repeating). Two-step recovery: Step 1 automated backfill via manual Cloudflare trigger `{ "period": "YYYY-MM", "dry_run": false, "manual_trigger": true }` with R-27-C1/C2 verification after each period; Step 2 manual collection per `docs/SOC2_READINESS.md §79` (Better Stack SLA export → `A1-E-001-{YYYY-MM}-MANUAL.json`; `audit_log_events` query → `CC7-E-001-{YYYY-MM}-MANUAL.json`; MASTER-INDEX rows `status = MANUAL_COLLECTION`). Two communication templates: EVD-01 (internal compensating control memo — required for every Step 2 execution; compliance-officer signed) and EVD-02 (auditor notification — P0 fieldwork-imminent only). Three DEC-030 HMAC-chained events: `system.evidence_automation_failure_declared` (HIGH, 7yr — `missing_periods` array, `consecutive_misses` int, `trigger_alert` enum), `system.evidence_manual_collection_completed` (STANDARD, 7yr — one per period backfilled; `artefacts_written` R2 paths, `manual_trigger: true`), `system.evidence_automation_restored` (STANDARD, 3yr — `root_cause_category` enum, `fix_deployed_at` datetime). EVD-CHAIN-01 ordering invariant: failure_declared precedes all manual_collection_completed and automation_restored events for the same `incident_id` — inversion triggers R-05. Six evidence artefacts EVD-E-001 through EVD-E-005 + EVD-COMP-E-001 mapped to CC4.1 / CC4.2 / CC7.2 / CC7.3 / A1.1; CC4.2 auditor narrative supplied (MANUAL_COLLECTION MASTER-INDEX status distinguishes manual from automated artefacts). Four post-incident controls: Scenario P tabletop (30d post-PIR), OQ-EVD-01 delta-log review (H5 trigger), CI integration test for Worker (H2 trigger), quarterly synthetic dry-run (quarterly regardless). Six-item implementation checklist: 2× P0/Month O-1 (three DEC-030 events registered; AL-EVD-01 third-consecutive escalation counter in pg_cron job 33), 2× P1/Month O-1 (EVD-01/02 templates, Scenario P tabletop), 2× P2 (quarterly dry-run calendar, OQ-EVD-02 resolved status update in SOC2_READINESS.md §81.12). Cross-references: `docs/SOC2_READINESS.md §81` (evidence automation Worker design, AL-EVD-01/02/03/04 alert rules); `docs/SOC2_READINESS.md §81.8` (AL-EVD-01 dead-man's switch SQL, AL-EVD-03 R2 write failure, AL-EVD-04 ETag conflict); `docs/SOC2_READINESS.md §81.12 OQ-EVD-02` (source open question — this runbook is the resolution); `docs/SOC2_READINESS.md §79` (master evidence collection protocol — §79.5 periodic calendar used in Step 2 manual collection); `docs/SOC2_READINESS.md §81.7` (MASTER-INDEX ETag guard — H5 root cause and OQ-EVD-01 delta-log trigger); R-03 (Infrastructure Outage — parallel activation when R2 unavailable); R-05 (HMAC Chain Break — parallel activation when AL-EVD-02 fires; EVD-CHAIN-01 ordering violation also triggers); `docs/AUDIT_LOG_SCHEMA.md §6.System-Compliance` (three new DEC-030 events to register — P0 before Month O-1); §9.4 (tabletop catalog — Scenario P); pg_cron job 33 `evidence_cron_freshness_check` (§12.6 OBSERVABILITY.md registry — third-consecutive escalation logic added). Owner: compliance-officer + devops-lead + platform-engineer.*

---

## 17. `siem-incident-automator` Resilience Design — Incident ID Pre-generation & Free-Text Privacy Controls

> Closes: OQ-IR-02 (P1, platform-engineer, M4) and OQ-IR-03 (P2, security-engineer + compliance-officer, M4 emitter review).
> Owner: security-engineer + platform-engineer + compliance-officer. References: §16, DEC-030, `docs/AUDIT_LOG_SCHEMA.md`, `docs/CRYPTOGRAPHY_POLICY.md`.

### 17.1 Purpose and Decisions

This section formalises two design decisions for the `siem-incident-automator` Cloudflare Worker (§16.4) left as open questions in §16.9.

| Open Question | Decision | Section |
|---|---|---|
| **OQ-IR-02** — Linear API unavailability creates a gap window where DEC-030 events have no stable `incident_id` | **Option A adopted**: pre-generate `incident_id` before Linear API call; emit `incident.opened` immediately; link ticket asynchronously via new `incident.linear_ticket_linked` event | §17.2 |
| **OQ-IR-03** — IC may inadvertently write PII into `incident.severity_changed.reason`, which is retained 7 years in immutable chain | **Option A + SHA-256 hash masking**: no plaintext stored in DEC-030; runtime blocklist provides soft-warning advisory without blocking; plaintext lives exclusively in the Linear ticket | §17.3 |

---

### 17.2 OQ-IR-02: Pre-Generated Incident ID Architecture

#### 17.2.1 Problem

The §16.4 design creates the Linear ticket first, then uses the returned URL to populate `incident.opened`'s `linear_ticket_url` field. When Linear is unavailable, the Worker falls back to PagerDuty-only notification. The on-call IC is paged but has no `incident_id` anchor until someone manually creates a Linear ticket — creating a window where DEC-030 lifecycle events have no stable ID to reference, which auditors may flag as a CC7.4 evidence gap.

#### 17.2.2 Decision: Option A — ID-first emission

**Adopted approach:** The `siem-incident-automator` Worker generates `incident_id` as its very first operation, before any outbound API call. DEC-030 `incident.opened` is emitted immediately with this ID. Linear and PagerDuty calls proceed asynchronously after the emit.

**`incident_id` format:**

```
INC-YYYYMMDD-[6hex]
```

Where:
- `YYYYMMDD` — UTC date at event receipt
- `[6hex]` — 3-byte cryptographically random suffix (`crypto.getRandomValues(3)`, hex-encoded)

Collision probability at 1,000 events/day: < 0.00001%. If a duplicate primary key is returned by `audit_log_events`, the Worker retries suffix generation once. A second collision is CRITICAL (R-05).

#### 17.2.3 Revised Worker Execution Order

```
siem-incident-automator Worker (per SIEM webhook receipt)

Step 1  Validate PagerDuty webhook HMAC-SHA256 signature
Step 2  Generate: incident_id = "INC-" + UTC_YYYYMMDD + "-" + hex(crypto.getRandomValues(3))
Step 3  EMIT  incident.opened  (HIGH, 7yr)
               { incident_id, alert_id, severity, triggered_by: "siem_auto",
                 linear_ticket_url: "PENDING" }
               ← PRIMARY CHAIN ANCHOR. Must succeed before Steps 4–6.
               On write failure → return HTTP 500; PagerDuty retries delivery.
Step 4  CREATE Linear issue asynchronously
               title = "[{incident_id}] {alert_summary}"
               → Success:  EMIT incident.linear_ticket_linked (LOW, 7yr)
                           { incident_id, linear_ticket_url, linked_by: "auto" }
               → Failure:  retry ×5 with exponential backoff (2 s / 4 s / 8 s / 16 s / 32 s ≈ 62 s total)
                           After 5 failures: fire AL-IR-LINEAR-01 (P2, Slack #security-ops);
                           set linear_ticket_url = "PENDING-LINEAR-UNAVAILABLE"
Step 5  PAGE   PagerDuty Events API v2
               dedup_key = incident_id; custom_details include incident_id
Step 6  NOTIFY Slack #security-ops: incident_id, severity, runbook link
```

**Chain invariant:** `incident.opened` at Step 3 is the primary anchor. All subsequent `incident.*` events for this `incident_id` reference it via `prev_event_hash`. Auditor verification starts at `incident.opened`; `incident.linear_ticket_linked` is the secondary reference to the human-readable ticket.

#### 17.2.4 `incident.linear_ticket_linked` — New DEC-030 Event

| Field | Value |
|---|---|
| `action` | `incident.linear_ticket_linked` |
| `severity` | LOW |
| `retention` | 7 yr |
| `trigger` | Linear ticket successfully created — either immediately (Step 4 auto) or via IC amendment endpoint (§17.2.5) |
| `payload` | `{ incident_id, linear_ticket_url, linked_by: "auto" \| "manual_ic" }` |
| Privacy invariant | No user PII. `incident_id` + `linear_ticket_url` only. `linked_by` distinguishes automated vs. manual IC linkage for CC7.4 completeness review. |
| HMAC chain position | Immediately follows `incident.opened` (if auto) or at the time of IC amendment (if manual). A `incident.linear_ticket_linked` event without a preceding `incident.opened` for the same `incident_id` is a chain ordering violation (R-05). |

Every `incident.opened` event must be paired with exactly one `incident.linear_ticket_linked` event. Auditors verify this pairing during SOC 2 fieldwork.

#### 17.2.5 Manual Ticket Amendment Protocol

When `AL-IR-LINEAR-01` fires (5 retries exhausted):

1. IC creates the Linear ticket manually; title **must** begin with `[{incident_id}]` (exact format enforced by the amendment endpoint validator).
2. IC calls `POST /internal/v1/incident/link-ticket` with `{ incident_id, linear_ticket_url }` — requires PAM-elevated `form_admin` role (§24).
3. The amendment endpoint emits `incident.linear_ticket_linked` with `{ incident_id, linear_ticket_url, linked_by: "manual_ic" }`.
4. IC closes AL-IR-LINEAR-01 in Slack/PagerDuty with the Linear ticket URL in the resolution note.

This protocol guarantees every `incident.opened` chain anchor eventually receives a corresponding `incident.linear_ticket_linked` — the linkage is complete regardless of Linear API availability at the moment of incident detection.

#### 17.2.6 SOC 2 Evidence Impact

| Criterion | Before §17 | After §17 |
|---|---|---|
| **CC7.4** — Incident response activities documented in chain | `incident.opened` may emit with `linear_ticket_url: 'PENDING'` and never resolve; auditors may flag | Every incident: `incident.opened` at T+0 + `incident.linear_ticket_linked` within 62 s (auto) or IC amendment SLA |
| **CC7.3** — Detection and monitoring | SIEM auto-open had a gap window under Linear unavailability | DEC-030 chain anchor at T+0 regardless of Linear or PagerDuty state |
| **CC7.2** — Anomaly monitoring | No alert on Linear API failure | AL-IR-LINEAR-01 (P2 Slack) detects persistent Linear unavailability within 62 s |

---

### 17.3 OQ-IR-03: Privacy Floor for Free-Text Fields in Incident Lifecycle Events

#### 17.3.1 Problem

The `incident.severity_changed` event (§16.2) includes a `reason` field authored by the IC under time pressure. This field is retained for 7 years in the immutable HMAC chain. Under incident conditions, an IC may inadvertently write a user's name, email address, Ukrainian IPN (10-digit national identifier), or a health data category term. Mitigation options were (a) runtime blocklist, (b) documentation-only, (c) allowlist schema.

#### 17.3.2 Decision: Option A with SHA-256 Hash Masking

**Adopted approach:** `reason_plaintext` is never written to the DEC-030 chain record. Instead, the `emit-audit-event` Worker stores:

```
reason_hash = SHA-256(incident_id + "|" + reason_plaintext + "|" + INCIDENT_REASON_HASH_SALT)
```

This is identical to the `bypass_reason_hash` pattern established by DEC-044 in §40 (`system.load_test_gate_bypassed`), creating a consistent auditor verification workflow across FORM's incident and control records. `reason_plaintext` is stored exclusively in the Linear ticket referenced by `incident.linear_ticket_linked`.

**Why not Option B (documentation-only):** IC training checklists do not provide a reliable privacy guarantee for a 7-year immutable record that cannot be corrected after the fact.

**Why not Option C (allowlist):** A strict allowlist (only incident IDs, runbook IDs, DEC-030 event types) would prevent ICs from writing useful free-form context under time pressure — "Postgres index lock contention in SCIM Worker" would be rejected — increasing cognitive load during active P0s. Hash masking achieves the privacy guarantee without imposing allowlist friction on the IC.

#### 17.3.3 Hash Computation

```typescript
// In emit-audit-event Worker, for incident.severity_changed
const reasonBytes = new TextEncoder().encode(
  incident_id + '|' + reason_plaintext + '|' + env.INCIDENT_REASON_HASH_SALT
);
const hashBuffer = await crypto.subtle.digest('SHA-256', reasonBytes);
const reason_hash = Array.from(new Uint8Array(hashBuffer))
  .map(b => b.toString(16).padStart(2, '0'))
  .join('');

// Stored in DEC-030 record:  reason_hash  (hex string, 64 chars)
// NOT stored in DEC-030:     reason_plaintext
```

`INCIDENT_REASON_HASH_SALT` is a Cloudflare Workers Secret on the `emit-audit-event` Worker, catalogued in `docs/CRYPTOGRAPHY_POLICY.md §5`:

| Key | Purpose | Custody | Rotation | Write-once? |
|---|---|---|---|---|
| `INCIDENT_REASON_HASH_SALT` | SHA-256 salt for `reason_plaintext` in `incident.severity_changed` | Cloudflare Workers Secret: `emit-audit-event` | Annual — compliance-officer + security-engineer joint approval; rotation is forward-only (does not invalidate past hash records; auditor uses the salt in effect at event emission time, retrievable from key version history in 1Password Operations vault) | No |

#### 17.3.4 Runtime Blocklist — Soft-Warning Layer

Before SHA-256 hashing, the emitter runs a lightweight blocklist scan on `reason_plaintext`. If any pattern matches, it:
1. Emits `incident.pii_risk_detected` (§17.3.5) immediately after `incident.severity_changed` in the chain.
2. Posts to `#security-ops` Slack: `⚠️ PII pattern detected in incident.severity_changed reason for {incident_id} — category: {matched_pattern_category}. Review the Linear ticket and redact if needed. (May be false positive.)`

The blocklist is a **soft warning only** — it does not block the `incident.severity_changed` emission or pause incident management workflow.

| Pattern | Category | Rationale |
|---|---|---|
| `/\S+@\S+\.\S+/` | `email` | Email addresses |
| `/\b\d{10}\b/` | `national_id` | Ukrainian IPN (10-digit tax identifier) |
| `/\b\d{9}\b/` | `national_id` | Potential SSN / NIN format |
| `/\b(user_id\|tenant_id\|coaching_turn_id)\s*[:=]\s*[0-9a-f-]{8,}/i` | `uuid_dump` | Accidental structured field dump with UUIDs |
| `/\b(hrv\|injury\|clinical\|diagnosis\|health data\|weight\|bmi)\b/i` | `art9_term` | GDPR Art. 9 special category terms (note: may false-positive on runbook names like R-15 "Health Data Field") |

**False positive handling:** The Slack warning is annotated `(may be false positive — review Linear ticket)`. Auditors treat `incident.pii_risk_detected` events as advisory; a confirmed PII write is assessed separately under R-22 (Privacy Floor Breach). The false-positive rate is tracked via IR-AUTO-E-003 (§17.6) with a tuning review at M9 if the rate exceeds 30%.

#### 17.3.5 `incident.pii_risk_detected` — New Advisory DEC-030 Event

| Field | Value |
|---|---|
| `action` | `incident.pii_risk_detected` |
| `severity` | MEDIUM |
| `retention` | 7 yr |
| `trigger` | Blocklist match on `reason_plaintext` during `incident.severity_changed` emission |
| `payload` | `{ incident_id, matched_pattern_category: z.enum(["email","national_id","uuid_dump","art9_term"]), false_positive_likely: z.boolean() }` |
| Privacy invariant | The matched plaintext fragment is **not** stored — only the pattern category. `false_positive_likely: true` is set automatically when the matched term is a known runbook ID substring (e.g., "health" in "R-15 Health Data Field"). |
| HMAC chain position | Must immediately follow the `incident.severity_changed` event with the same `incident_id`. Chain ordering constraint **IR-PII-CHAIN-01**: a `incident.pii_risk_detected` without a directly preceding `incident.severity_changed` for the same `incident_id` is a chain ordering violation (R-05). |

#### 17.3.6 Auditor Fieldwork Protocol for `reason_plaintext` Retrieval

Auditors needing to read the IC's reason for a severity change:

1. Locate the `incident.severity_changed` chain record for the target `incident_id`. Note `reason_hash` (64-char hex).
2. Cross-reference `incident.linear_ticket_linked` for the same `incident_id` to obtain `linear_ticket_url`.
3. Access the Linear ticket via read-only guest credentials (granted by compliance-officer for the observation-period fieldwork window).
4. `reason_plaintext` is in the Linear ticket description or the incident log comment thread.
5. Independent hash verification: `SHA-256(incident_id + '|' + reason_plaintext + '|' + INCIDENT_REASON_HASH_SALT)` must match the stored `reason_hash`. The salt version in effect at emission time is retrievable from the 1Password Operations vault version history.

This protocol is filed at `compliance/fieldwork/incident-reason-verification.md` (P1, §17.7 checklist item 6).

---

### 17.4 AUDIT_LOG_SCHEMA.md Registration

Two new events require registration in `docs/AUDIT_LOG_SCHEMA.md §6.Incident-Lifecycle`:

| Event type | Severity | Retention | Zod schema |
|---|---|---|---|
| `incident.linear_ticket_linked` | LOW | 7 yr | `z.object({ incident_id: z.string().regex(/^INC-\d{8}-[0-9a-f]{6}$/), linear_ticket_url: z.string().url(), linked_by: z.enum(["auto", "manual_ic"]) })` |
| `incident.pii_risk_detected` | MEDIUM | 7 yr | `z.object({ incident_id: z.string().regex(/^INC-\d{8}-[0-9a-f]{6}$/), matched_pattern_category: z.enum(["email","national_id","uuid_dump","art9_term"]), false_positive_likely: z.boolean() })` |

HMAC chain entries for both events reference the `prev_event_hash` of the immediately preceding event in the `incident.*` namespace for the same `incident_id`.

---

### 17.5 SOC 2 Evidence Mapping

| Criterion | Evidence | Control statement |
|---|---|---|
| **CC7.4** — Incident response activities documented | IR-AUTO-E-001 (`incident.opened` T+0 latency sample) + IR-AUTO-E-002 (linkage latency distribution) | Every SIEM-triggered incident has a DEC-030 chain anchor within 3 seconds of webhook receipt, regardless of Linear API availability |
| **CC7.3** — Responds to detected events | IR-AUTO-E-001 | `incident_id` is stable from T+0; IC has an authoritative ID to reference in all subsequent communications |
| **CC7.2** — Anomaly monitoring | AL-IR-LINEAR-01 PagerDuty/Slack history | Linear API unavailability is detected and alerted within 62 seconds |
| **CC2.2** — External communications | `incident.update_posted` + `reason_hash` | `incident.severity_changed` chain record contains no plaintext PII; all stakeholder communications reference `incident_id` only |
| **P3.2** — Privacy practices followed | IR-AUTO-E-003 (`incident.pii_risk_detected` history) | Automated monitoring of accidental PII in IC-authored free text; 7-year chain contains no plaintext `reason` content |

Evidence artefacts:

| ID | Type | Collection query / source | Path | Retention |
|---|---|---|---|---|
| **IR-AUTO-E-001** | CC7.4 | `SELECT incident_id, (emitted_at - webhook_received_at) AS t0_latency_ms FROM audit_log_events WHERE action = 'incident.opened' AND emitted_at BETWEEN :obs_start AND :obs_end` | `compliance/evidence/cc7/ir-auto-e-001-YYYY-QN.csv` | 7 yr |
| **IR-AUTO-E-002** | CC7.4 | Cross-join `incident.opened` and `incident.linear_ticket_linked` on `incident_id`; compute `linked_at - opened_at`; report P95 linkage latency and count of `linked_by = "manual_ic"` amendments | `compliance/evidence/cc7/ir-auto-e-002-YYYY-QN.csv` | 7 yr |
| **IR-AUTO-E-003** | P3.2 | `SELECT matched_pattern_category, false_positive_likely, COUNT(*) FROM audit_log_events WHERE action = 'incident.pii_risk_detected' AND emitted_at BETWEEN :obs_start AND :obs_end GROUP BY 1, 2` | `compliance/evidence/privacy/ir-auto-e-003-YYYY-QN.csv` | 7 yr |

---

### 17.6 Implementation Checklist

#### P0 — Before M4 siem-incident-automator deployment

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Refactor `siem-incident-automator` Worker per §17.2.3: generate `incident_id` before all external API calls; emit `incident.opened` with `linear_ticket_url: "PENDING"`; add Linear 5× exponential-backoff retry (2 s / 4 s / 8 s / 16 s / 32 s); fire AL-IR-LINEAR-01 Slack alert on final failure. | platform-engineer | **P0** | M4 | [ ] |
| 2 | Register `incident.linear_ticket_linked` in `docs/AUDIT_LOG_SCHEMA.md §6.Incident-Lifecycle` with Zod schema from §17.4; deploy to `emit-audit-event` Worker. | platform-engineer + compliance-officer | **P0** | M4 | [ ] |
| 3 | Implement `POST /internal/v1/incident/link-ticket` amendment endpoint: PAM-elevated `form_admin` role required (§24); validates `incident_id` regex; emits `incident.linear_ticket_linked` with `linked_by: "manual_ic"`. | platform-engineer | **P0** | M4 | [ ] |
| 4 | Replace `reason` plaintext field with `reason_hash` (SHA-256, §17.3.3) in `incident.severity_changed` emitter; provision `INCIDENT_REASON_HASH_SALT` as Cloudflare Workers Secret on `emit-audit-event`; add to `docs/CRYPTOGRAPHY_POLICY.md §5` key inventory. | security-engineer + platform-engineer | **P0** | M4 | [ ] |
| 5 | Implement runtime blocklist regex scan (§17.3.4) in emitter; register `incident.pii_risk_detected` in `AUDIT_LOG_SCHEMA.md §6.Incident-Lifecycle` (§17.4); configure Slack `#security-ops` notification with false-positive guidance. | security-engineer | **P0** | M4 | [ ] |

#### P1 — Before M5 SOC 2 pre-observation readiness

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 6 | Document auditor fieldwork protocol (§17.3.6) at `compliance/fieldwork/incident-reason-verification.md`; include in auditor onboarding package alongside existing break-glass and DSAR fieldwork procedures. | compliance-officer | **P1** | M5 | [x] **Done (2026-06-18):** `compliance/fieldwork/incident-reason-verification.md` v1.0 authored; five-step retrieval procedure, hash verification formula, salt version retrieval from 1Password Operations vault, P3.2/CC2.2 SOC 2 mapping. Listed in `compliance/evidence/auditor-onboarding/README.md`. |
| 7 | Collect IR-AUTO-E-001 for first month of production `incident.opened` events; confirm P95 T+0 latency < 3 seconds; confirm zero incidents with `linear_ticket_url: "PENDING-LINEAR-UNAVAILABLE"` unresolved for > 1 hour. | compliance-officer | **P1** | M5 | [ ] |
| 8 | Add Tabletop Scenario O: "Linear API unavailable during active P0 — IC manually links ticket via amendment endpoint" to §9.4 tabletop catalog; schedule in Q3 2026 drill set. | security-engineer | **P1** | M5 | [ ] |

#### P2 — Post-observation period (M9)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 9 | Collect IR-AUTO-E-002 (linkage latency distribution); confirm P95 < 62 s for auto-linked incidents; report count of `manual_ic` amendments. | compliance-officer | **P2** | M9 | [ ] |
| 10 | Collect IR-AUTO-E-003 (`incident.pii_risk_detected` by category); review false-positive rate; tune blocklist if `art9_term` category > 30% false-positive rate (consider removing terms that match common runbook names). | security-engineer | **P2** | M9 | [ ] |
| 11 | Update `docs/SOC2_READINESS.md §CC7.4` evidence table to reference IR-AUTO-E-001 and IR-AUTO-E-002 as the primary CC7.4 automation evidence corpus. | compliance-officer | **P2** | M9 | [ ] |

---

### 17.7 Open Questions Resolved

| ID | Resolution | Date | Section |
|---|---|---|---|
| **OQ-IR-01** | 🟢 **Resolved** — Option (a) adopted. Timestamp-interleaved master chain retained; §18.3.1 skip-and-verify SQL extracts per-incident sequences; CONC-CHAIN-01 invariant (§18.3.3) detects cross-contamination. No per-incident parallel sub-chains (option b) — DEC-053. | 2026-06-14 | §18 |
| **OQ-IR-02** | 🟢 **Resolved** — Option A adopted. `incident_id` generated before all external API calls; `incident.opened` emitted at T+0; `incident.linear_ticket_linked` closes linkage asynchronously (auto 62 s / manual IC amendment). AL-IR-LINEAR-01 provides observability. | 2026-06-14 | §17.2 |
| **OQ-IR-03** | 🟢 **Resolved** — Option A + SHA-256 hash masking. `reason_plaintext` replaced by `reason_hash` in DEC-030; plaintext in Linear ticket only; runtime blocklist emits advisory `incident.pii_risk_detected` (no block). Consistent with DEC-044 §40 pattern. | 2026-06-14 | §17.3 |

All three open questions from §16.9 are now resolved.

---

*v2.4 additions (2026-06-14): §17 `siem-incident-automator` Resilience Design — Incident ID Pre-generation & Free-Text Privacy Controls. Closes OQ-IR-02 (P1, platform-engineer, M4) and OQ-IR-03 (P2, security-engineer + compliance-officer, M4 emitter review) from §16.9. OQ-IR-02 decision: Option A — `incident_id` = `INC-YYYYMMDD-[6hex]` (crypto.getRandomValues) generated as Worker's first operation; `incident.opened` (HIGH, 7yr) emitted with `linear_ticket_url: "PENDING"` before any external API call; Linear ticket created asynchronously with title prefix `[{incident_id}]`; 5× exponential-backoff retry (2/4/8/16/32 s); AL-IR-LINEAR-01 (P2 Slack) on persistent failure; IC amendment via `POST /internal/v1/incident/link-ticket` (PAM-elevated); `incident.linear_ticket_linked` (new LOW event, 7yr) closes linkage loop in both auto and manual paths. OQ-IR-03 decision: Option A + SHA-256 hash masking — `reason_hash = SHA-256(incident_id + '|' + reason_plaintext + '|' + INCIDENT_REASON_HASH_SALT)` stored in DEC-030 chain; no plaintext `reason` in 7-year record; consistent with DEC-044 `bypass_reason_hash` pattern (§40); `INCIDENT_REASON_HASH_SALT` added to CRYPTOGRAPHY_POLICY.md §5 (annual rotation, not write-once); runtime blocklist (5 patterns: email, national_id, uuid_dump, art9_term) emits advisory `incident.pii_risk_detected` (new MEDIUM event, 7yr — pattern category only, no fragment) to Slack `#security-ops` without blocking IC workflow; auditor fieldwork protocol (§17.3.6) mirrors §40 `bypass_reason_hash` verification procedure. Two new DEC-030 events: `incident.linear_ticket_linked` (LOW, 7yr) + `incident.pii_risk_detected` (MEDIUM, 7yr) — both require AUDIT_LOG_SCHEMA.md §6.Incident-Lifecycle registration. Three SOC 2 evidence artefacts: IR-AUTO-E-001 (CC7.4 — T+0 latency), IR-AUTO-E-002 (CC7.4 — linkage latency P95), IR-AUTO-E-003 (P3.2 — pii_risk_detected by category). Eleven-item implementation checklist: 5× P0/M4 (Worker refactor, event registration ×2, amendment endpoint, reason_hash + salt, blocklist), 3× P1/M5 (fieldwork procedure doc, IR-AUTO-E-001 collection, Tabletop Scenario O), 3× P2/M9 (linkage P95 evidence, false-positive tuning, SOC2_READINESS CC7.4 update). Chain ordering constraints: IR-PII-CHAIN-01 (`incident.pii_risk_detected` must follow same-incident `incident.severity_changed`); `incident.linear_ticket_linked` must follow `incident.opened` for same `incident_id`. Cross-references: §16 (§16.9 OQ-IR-02/03 source); §40 (DEC-044 `bypass_reason_hash` pattern — §17.3 adopts identically); `docs/AUDIT_LOG_SCHEMA.md §6.Incident-Lifecycle` (two new events to register — P0/M4); `docs/CRYPTOGRAPHY_POLICY.md §5` (INCIDENT_REASON_HASH_SALT key inventory entry — P0/M4); `docs/SOC2_READINESS.md §CC7.4` (evidence table update — P2/M9); R-05 (chain ordering violations — IR-PII-CHAIN-01 escalation path). Owner: security-engineer + platform-engineer + compliance-officer.*

---

*v2.2 additions (2026-06-12): R-25 GDPR Art. 9 Enterprise Tenant Off-boarding — Hard-Delete Overdue (AL-GDPR-03) — twenty-fifth runbook. Closes the incident response gap created when `docs/OBSERVABILITY.md §37` (v3.3, 2026-06-12) defined AL-GDPR-03 (P0, no-auto-resolve) for Art. 9 hard-delete overdue events with no corresponding incident runbook. R-25 covers the enterprise tenant off-boarding pipeline failure scenario: the Art. 9 hard-delete Worker fails to complete within GDPR-SLO-03 (4h from `tenants.offboarding_initiated_at`, DEC-036 zero-grace-period commitment). Distinct from R-14 (consumer DSAR Art. 17 individual erasure) and R-01 (data breach — activated in parallel only if data was accessed during the delay). Trigger matrix: AL-GDPR-03 primary automated trigger; tenant admin complaint and scheduled audit as manual triggers. Severity table: P0 for > 4h delay; P1 for partially-complete deletion; large-tenant extension threshold per OQ-GDPR-OBS-02. Immediate actions T+0 to T+20 with DEC-030 emission at T+15 as evidentiary anchor before any remediation action. Five root cause hypotheses H1–H5 (Worker crash, IOPS exhaustion, missing migration, tenant status not set, audit log pseudonymisation conflict). Recovery: Step 1 standard Worker retry; Step 2 chunked batch DELETE with 100ms pg_sleep yield for H2; Step 3 completion UPDATE + DEC-030 emission. Critical sub-protocol §R-25.8 (OQ-GDPR-OBS-03 interim): audit_log_events rows are NOT deleted during R-25 pending outside counsel confirmation on pseudonymisation vs deletion under HMAC-chain constraint; `gdpr.audit_log_pending_art9_decision` (STANDARD, 7yr) marks the pending decision in the chain. Recommended resolution: additive pseudonymisation (user_id_pseudonym column + ERASURE_PSEUDONYM_SALT + RLS restriction on original user_id) preserves chain integrity and satisfies GDPR Art. 4(5). GDPR Art. 33 decision tree §R-25.7: three-question gate (data accessible? third-party access? delay > 72h?) with explicit trigger conditions and outside counsel path. Three communication templates ART9-01/02/03. Five DEC-030 HMAC-chained events (CRITICAL: gdpr.art9_delete_overdue_detected + gdpr.art9_delete_manual_trigger; HIGH: gdpr.art9_delete_resumed + gdpr.art9_delete_completed; STANDARD: gdpr.audit_log_pending_art9_decision). Six evidence artefacts ART9-E-001 through ART9-E-006 mapped to SOC 2 CC7.2/CC7.3/CC7.4/CC6.5/P5.2/P4.0/GDPR Art. 5(1)(f). Four post-incident controls including Scenario M tabletop and chunked batch default. Two open questions: OQ-GDPR-OBS-03 (P0, shared with OBSERVABILITY §37.11 — outside counsel required before M13) and OQ-R25-01 (P1, Art. 33 automatic trigger threshold). Eight-item implementation checklist: 3× P0/M6 (DEC-030 registration, off-boarding Worker, AL-GDPR-03 PagerDuty); 1× P0/M13 (OQ-GDPR-OBS-03 counsel + migration); 4× P1 (templates, Scenario M, evidence collection, OQ-R25-01). Cross-references: `docs/OBSERVABILITY.md §37` (AL-GDPR-03 trigger; GDPR-SLO-03; OQ-GDPR-OBS-03 shared); `docs/DATA_MODEL.md §25` (off-boarding table list); `docs/DATA_MODEL.md §12` (Art. 17 erasure — pseudonymisation pattern for OQ-GDPR-OBS-03 resolution); `docs/SOC2_READINESS.md §67` (deletion certificate); `docs/SOC2_READINESS.md §70` (DSAR lifecycle — sister evidence section); `docs/AUDIT_LOG_SCHEMA.md` (five new DEC-030 events to register); R-01 (parallel activation condition); R-14 (consumer sister runbook). Owner: compliance-officer + platform-engineer + security-engineer.*

---

---

## 18. OQ-IR-01 Resolution — Concurrent Incident Sub-Chain Ordering (DEC-053)

### 18.1 Purpose & Decision

This section closes **OQ-IR-01** (P2, §16.9) — the design question of how the DEC-030 HMAC master chain handles interleaved events from two or more simultaneously active incidents.

| Field | Value |
|---|---|
| **Decision** | **Option (a) selected** — retain current timestamp-interleaved master chain; document skip-and-verify algorithm for auditors |
| **Decision ref** | DEC-053 (2026-06-14) |
| **Previous status** | 🟡 Open — target M6 |
| **New status** | 🟢 **Resolved** |
| **Owners** | security-engineer + compliance-officer |
| **Effective** | 2026-06-14 |

Option (b) — per-incident parallel sub-chains with a merge step at `incident.recovered` — is **rejected** at this stage. See §18.2 for full rationale. Re-evaluation trigger: if concurrent incidents exceed two within a single 12-month SOC 2 observation window, option (b) must be re-evaluated before the next observation period opens (see §18.7 item 5).

---

### 18.2 Decision Rationale

Five independent grounds favour option (a). Each is sufficient; together they are conclusive.

#### 18.2.1 Ground 1 — Statistical Rarity of Concurrent Incidents

FORM's current threat model (pre-Series A, beta cohort < 5,000 users, Cloudflare Workers stateless compute) produces an estimated P0/P1 incident frequency of ≤ 3 per year. The conditional probability of two simultaneously active incidents — where both generate interleaved `incident.*` chain events within the same multi-hour window — is estimated at < 2% annually.

Designing a chain-merge architecture for a scenario that occurs < 1× per year adds implementation surface without a proportionate reduction in auditor friction. The skip-and-verify algorithm (§18.3) produces an equivalent per-incident chain view with a single SQL query.

**Re-evaluation condition:** If two or more incidents are simultaneously active for > 4 hours within any single 12-month observation window, compliance-officer must initiate an option (b) design review before the next SOC 2 observation period opens.

#### 18.2.2 Ground 2 — Option (b) Breaks the DEC-030 HMAC Single-List Invariant

The DEC-030 chain (per `docs/AUDIT_LOG_SCHEMA.md`) is a **strictly ordered single linked list**: each event's `hmac_value` is computed over `(hmac_prev, actor, action, payload, emitted_at)`, where `hmac_prev` is the `hmac_value` of the immediately preceding row ordered by `emitted_at`.

Option (b) would require:
1. A new `incident.sub_chain_fork` event type (marking where the parallel branch begins) with an `hmac_prev` pointing into the master chain.
2. A new `incident.sub_chain_merge` event type (rejoining the master chain at `incident.recovered`) requiring a non-trivial merge algorithm.
3. Resolution of a write-ordering race: if INC-A recovers while INC-B is still emitting events, the `incident.sub_chain_merge` for INC-A must determine which event becomes the new `hmac_prev` for subsequent master-chain events — creating a concurrent-write ordering problem under Cloudflare Workers execution.

This is a structural change to the DEC-030 specification requiring `docs/AUDIT_LOG_SCHEMA.md` amendment, a migration for `audit_log_events` (new event types, new `sub_chain_id` column), and auditor re-education on a two-level chain structure. The implementation risk (merge race condition creating a silent chain break) exceeds the auditor-convenience benefit.

#### 18.2.3 Ground 3 — Skip-and-Verify Is Auditor-Standard Practice

SOC 2 Type II auditors routinely filter interleaved audit logs by structured attribute (user ID, session ID, request correlation ID). FORM's `incident_id` key (`INC-YYYYMMDD-[6hex]`) functions identically to a correlation key in conventional SIEM products (Splunk, Datadog, Sumo Logic).

The canonical skip-and-verify SQL (§18.3.1) produces a deterministic per-incident event sequence from the master chain in < 100 ms at any projected observation-period volume (< 50,000 events/year). AICPA SOC 2 fieldwork guides explicitly permit filtered event-stream extraction as a chain-verification technique.

#### 18.2.4 Ground 4 — Consistent with DEC-043 / DEC-044 / DEC-051 Pattern

FORM consistently chooses the simpler implementation with a documented auditor or IC protocol over a complex automated solution that introduces new failure modes:
- **DEC-043:** `business_justification` plaintext excluded from auditor bulk export; redacted-sample fieldwork protocol adopted.
- **DEC-044:** `bypass_reason_hash` SHA-256 in chain; plaintext in Linear ticket only.
- **DEC-051:** 7-year retention via pg_cron job 32 (not a complex tiered retention engine).

DEC-053 continues this pattern: the auditor fieldwork protocol (§18.5) documents exactly what a SOC 2 reviewer needs to extract and verify a per-incident chain, without changing the underlying infrastructure.

#### 18.2.5 Ground 5 — Upgrade Path to Option (b) Preserved

Option (a) does not preclude a future upgrade. If option (b) is ever selected:
- Migration path: add `sub_chain_id UUID` column to `audit_log_events`; create `incident_sub_chains` table; add `incident.sub_chain_fork` and `incident.sub_chain_merge` to `docs/AUDIT_LOG_SCHEMA.md §6.Incident-Lifecycle`.
- All existing events retain their current `hmac_prev` semantics — option (b) is entirely additive; no re-hashing of historical events required.

Reverse cost of option (a) → option (b): **Low.**

---

### 18.3 Skip-and-Verify Algorithm

The following SQL extracts the ordered event sequence for a single incident from the interleaved master chain. This is the canonical per-incident chain view for SOC 2 auditor fieldwork.

#### 18.3.1 Canonical Auditor SQL

```sql
-- Per-incident sub-chain extraction (compliance_reviewer role; read-only)
-- Replace :incident_id with the target ID in format INC-YYYYMMDD-XXXXXX
SELECT
  e.id                                                     AS event_id,
  e.emitted_at,
  e.actor,
  e.action,
  e.payload->>'incident_id'                                AS incident_id,
  e.hmac_value,
  e.hmac_prev,
  ROW_NUMBER() OVER (ORDER BY e.emitted_at)                AS sub_chain_seq,
  LAG(e.id)   OVER (ORDER BY e.emitted_at)                 AS master_chain_prev_event_id
FROM audit_log_events e
WHERE e.payload->>'incident_id' = :incident_id
ORDER BY e.emitted_at ASC;
```

**Output interpretation:**
- `sub_chain_seq` numbers the incident's own events chronologically. No gaps can appear (the query returns every incident event).
- `master_chain_prev_event_id` identifies the event immediately preceding each incident event in the master chain — this may belong to a different active incident. Auditors use this to confirm that `hmac_prev` of each incident event references its correct master-chain predecessor.
- Non-contiguous `master_chain_prev_event_id` values (events from another incident interleaved between two INC-A events) are **expected and correct** — they are not a chain anomaly.

#### 18.3.2 HMAC Spot-Verification Procedure

To verify HMAC integrity of concurrent-incident events, the auditor may:

1. Run the standard master-chain integrity check from `docs/SOC2_READINESS.md §79.7` — a clean result covers all events, including interleaved concurrent-incident events. This is the primary verification path.
2. Optionally, for per-incident spot-verification: run §18.3.1 to extract INC-A events; for each row, look up the referenced `hmac_prev` event by `hmac_value = :hmac_prev` in `audit_log_events`; confirm `hmac_value = HMAC-SHA256(hmac_prev ‖ actor ‖ action ‖ payload ‖ emitted_at)` using the active-epoch `AUDIT_LOG_HMAC_SECRET` (provided by compliance-officer on request during fieldwork).
3. The §18.5 fieldwork protocol document contains a worked example with expected input/output for step 2.

#### 18.3.3 CONC-CHAIN-01 — Cross-Contamination Invariant

**Invariant:** No `incident.*` DEC-030 event may reference an `incident_id` belonging to a different simultaneously-active incident. Each event must carry exactly the `incident_id` of the incident it pertains to.

**Detection query:**
```sql
-- CONC-CHAIN-01 violation detector (compliance_reviewer role; read-only)
-- Returns events where the payload incident_id is inconsistent with the Linear ticket URL.
-- Zero rows = invariant intact. Any rows = investigate with security-engineer.
SELECT
  e.id,
  e.action,
  e.payload->>'incident_id'       AS claimed_incident_id,
  e.payload->>'linear_ticket_url' AS linear_url
FROM audit_log_events e
WHERE e.action LIKE 'incident.%'
  AND e.payload->>'linear_ticket_url' IS NOT NULL
  AND e.payload->>'linear_ticket_url' NOT LIKE
      '%' || (e.payload->>'incident_id') || '%';
```

**On zero rows:** CONC-CHAIN-01 intact — file result as part of CONC-E-001 (§18.6).
**On any rows:** Potential cross-contamination; escalate to security-engineer for manual review; do not file CONC-E-001 until reviewed and explained.

---

### 18.4 Concurrent Incident Coordination Protocol

When two incidents are simultaneously active, the following protocol minimises the risk of a CONC-CHAIN-01 violation.

| Step | Action | Owner | Timing |
|---|---|---|---|
| 1 | Confirm both `incident_id` values are distinct in Slack `#incidents`. The `siem-incident-automator` generates each ID independently (`INC-YYYYMMDD-[6hex]` via `crypto.getRandomValues`); collision probability is negligible but must be verified on a concurrent-open. | IC (primary) | T+0 for each incident |
| 2 | Open a dedicated restricted channel per incident: `#inc-YYYYMMDD-[severity]-a` and `#inc-YYYYMMDD-[severity]-b`. All DEC-030 events, runbook links, and status updates reference only the channel's own `incident_id`. Cross-channel coordination occurs in `#incidents` (general) only. | IC (each) | T+5 min |
| 3 | Designate a separate IC for each active incident if either incident is severity ≥ P1. Single-IC management of two concurrent P0 incidents is **prohibited** — page the secondary on-call engineer immediately. | founder or primary IC | T+10 min |
| 4 | After both incidents are recovered: run §18.3.1 for each `incident_id`; confirm zero overlap (no event appears in both query results); run §18.3.3 and confirm zero rows. File combined result as CONC-E-001 for the observation quarter. | compliance-officer | Post-recovery (within 24h) |

---

### 18.5 Auditor Fieldwork Protocol

**Document path:** `compliance/fieldwork/concurrent-incident-chain-verification.md`

This document must be authored per §18.7 item 1 and included in the auditor onboarding package. Required contents:

1. Explanation of the timestamp-interleaved master chain design — why non-contiguous per-incident event sequences are expected and correct.
2. The §18.3.1 canonical SQL with annotated column descriptions.
3. The §18.3.2 three-step spot HMAC verification procedure, including the `AUDIT_LOG_HMAC_SECRET` request process.
4. The §18.3.3 CONC-CHAIN-01 violation detection query with interpretation guide.
5. A worked example using synthetic incident IDs from Tabletop Scenario P (§18.7 item 4): two simultaneous incidents, expected interleaved chain segment, §18.3.1 output for each, §18.3.3 zero-row confirmation.

---

### 18.6 SOC 2 Evidence Mapping

| Criterion | Evidence | Control statement |
|---|---|---|
| **CC7.4** — Response to identified security events | CONC-E-001: §18.3.1 query output for any concurrent-incident period in the observation quarter; demonstrates per-incident event sequences are extractable and verifiable | Concurrent incidents are managed using independent `incident_id` namespaces; the master chain records all lifecycle events correctly; the skip-and-verify protocol provides per-incident chain isolation for auditors without requiring infrastructure changes |
| **CC4.1** — Performance evaluations of controls | CONC-E-001 + §18.3.3 zero-row result | CONC-CHAIN-01 is verifiable at any time; zero rows confirms no cross-contamination occurred during the observation period |

**Evidence artefact: CONC-E-001**

| Field | Value |
|---|---|
| **Artefact ID** | CONC-E-001 |
| **SOC 2 criteria** | CC7.4, CC4.1 |
| **Contents** | (a) §18.3.1 query output for each concurrent-incident period in the observation quarter (or a zero-event attestation if no concurrent incidents occurred); (b) §18.3.3 violation detection query result (zero rows expected); (c) reference to `compliance/fieldwork/concurrent-incident-chain-verification.md` |
| **Path** | `compliance/evidence/ir-chain/CONC-E-001_<YYYY-QN>.md` |
| **Frequency** | Quarterly — filed even when no concurrent incidents occurred (zero-event attestation demonstrates the control was not bypassed) |
| **Retention** | 7 years (CC7.4 evidence class) |

**Auditor narrative for CC7.4:** FORM's DEC-030 HMAC chain records all incident lifecycle events in a single timestamp-ordered linked list. When two incidents are simultaneously active, their events are interleaved chronologically. The §18.3.1 SQL query extracts a per-incident event sequence from the master chain in < 100 ms. The master chain integrity check (SOC2_READINESS §79.7) verifies all events — including interleaved concurrent-incident events — in a single pass. CONC-CHAIN-01 (§18.3.3) provides a real-time cross-contamination detector. No per-incident sub-chain architecture is required; the master chain integrity proof subsumes per-incident integrity.

---

### 18.7 Implementation Checklist

#### P1 — Before SOC 2 observation period starts (M7)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Author `compliance/fieldwork/concurrent-incident-chain-verification.md` per §18.5 spec; include worked example using synthetic IDs from Tabletop Scenario P (§18.7 item 4). Include document in auditor onboarding package index at `compliance/evidence/auditor-onboarding/README.md`. | compliance-officer | **P1** | M7 | [x] **Done (2026-06-18):** `compliance/fieldwork/concurrent-incident-chain-verification.md` v1.0 authored per all five §18.5 spec items: (1) timestamp-interleaved design explanation with DEC-053 rationale; (2) §18.3.1 canonical SQL with annotated column table; (3) §18.3.2 four-step HMAC spot-verification procedure; (4) §18.3.3 CONC-CHAIN-01 detection query with interpretation guide; (5) Tabletop Scenario P worked example (INC-20260618-a1b2c3 P0 breach + INC-20260618-d4e5f6 P1 SIEM false-positive) with §18.3.1 per-incident outputs and §18.3.3 zero-row confirmation. `compliance/evidence/auditor-onboarding/README.md` v1.0 created; both fieldwork docs indexed. |
| 2 | Run §18.3.3 CONC-CHAIN-01 violation detection query on all existing `incident.*` events in `audit_log_events`; confirm zero rows; file result as baseline CONC-E-001 zero-event attestation for the pre-observation period (`compliance/evidence/ir-chain/CONC-E-001_2026-pre-obs.md`). | compliance-officer | **P1** | M7 | [ ] |
| 3 | Add §18.4 concurrent incident coordination protocol to the §1 IC Quick-Start checklist as a two-line addendum: "Two active incidents? Confirm distinct IDs in #incidents. Separate channels, separate ICs for ≥ P1." | security-engineer | **P1** | M7 | [ ] |

#### P2 — Tabletop and annual governance

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 4 | Add Tabletop Scenario P to §9.4 tabletop catalog: two simultaneously active incidents (P0 breach + P1 SIEM false-positive). Walk through §18.4 coordination protocol; validate distinct `incident_id` values; run §18.3.1 for each ID post-exercise; run §18.3.3 and confirm zero rows. Document expected output in the §18.5 fieldwork guide worked example. | security-engineer | **P2** | M8 | [ ] |
| 5 | Annual review: if two or more concurrent incidents occurred in the observation year, initiate option (b) design review (§18.2.1 re-evaluation condition) and document outcome in `docs/DECISION_LOG.md`. | compliance-officer | **P2** | Annual (M12, M24, …) | [ ] |

---

### 18.8 OQ-IR-01 Status

| Status | 🟢 **Resolved — DEC-053 (2026-06-14)** |
|---|---|
| **Decision** | Option (a) — timestamp-interleaved master chain with documented skip-and-verify auditor protocol |
| **Option (b) status** | Rejected at current scale; upgrade path preserved via additive migration (§18.2.5) |
| **Re-evaluation trigger** | ≥ 2 concurrent incidents within any single 12-month observation window |
| **§16.9 cross-update** | OQ-IR-01 updated to 🟢 Resolved (DEC-053) |
| **§17.7 cross-update** | OQ-IR-01 row added to resolved table |

---

*v2.7 (2026-06-15): R-05 OFB-REGION-01 cross-reference patch — registers EU off-boarding data egress invariant (OFB-REGION-01) as an additional R-05 trigger condition; closes `docs/DATA_MODEL.md §36.9` cross-reference obligation (DEC-061, 2026-06-15). R-05 Trigger now reads: "Or: OFB-REGION-01 chain invariant violation — `enterprise.data_export_completed` event emitted with `is_eu_region: true` but `r2_bucket != 'form-offboarding-exports-eu'` (HTTP 422 + P1 PagerDuty `form-platform`; see `docs/DATA_MODEL.md §36.6`; cross-tenant data residency breach — treat as potential data breach until forensic analysis confirms routing bug)." Rationale: OFB-REGION-01 fires only when the HMAC-chained `enterprise.data_export_completed` event records an EU tenant's off-boarding package being routed to the wrong R2 bucket region; this is a chain invariant violation (not a chain break) but the forensic analysis and evidence-preservation steps of R-05 are the correct response protocol — the chain itself remains intact but the payload OFB-REGION-01 field values are inconsistent with the data routing contract. No new runbook, DEC-030 event, or evidence artefact introduced — the OFB-REGION-01 trigger resolves via the existing R-05 steps (verify timing, export chain, forensic root-cause analysis, repair via `DATA_MODEL.md §36.5` routing fix). Header corrected: v2.6 → v2.7. Cross-references: `docs/DATA_MODEL.md §36.6` (OFB-REGION-01 specification — HTTP 422 + P1 PagerDuty on violation); `docs/DATA_MODEL.md §36.9` (cross-reference obligation — now closed); `docs/SOC2_READINESS.md §67.9` (CC6.1/C1.1 EU data residency criteria — OFB-E-005/006 evidence; v3.9.1); `docs/DECISION_LOG.md DEC-061`. Privacy floor: no employee `user_id`, health data, or individual names in any R-05 forensic query — chain validation operates on event metadata and HMAC values only. Owner: security-engineer + compliance-officer.*

*v2.6 (2026-06-14): §18 OQ-IR-01 Resolution — Concurrent Incident Sub-Chain Ordering (DEC-053). Closes the sole remaining open question from §16.9 (v1.0, 2026-06-01). OQ-IR-01 question: when two incidents are simultaneously active, their `incident.*` DEC-030 events are timestamp-interleaved in the master chain, complicating per-incident auditor verification. Decision: **Option (a) adopted** — retain timestamp-interleaved master chain; provide documented skip-and-verify SQL (§18.3.1) for auditor per-incident extraction. Option (b) rejected on five grounds: (1) statistical rarity — concurrent P0/P1 incidents at FORM's scale estimated < 2% annual probability; (2) DEC-030 structural integrity — option (b) requires `incident.sub_chain_fork` / `incident.sub_chain_merge` event types that violate the single-list HMAC invariant; merge-step write-ordering race under concurrent Workers execution introduces a new chain-break failure mode not present in option (a); (3) auditor-standard practice — AICPA SOC 2 guides permit filtered event-stream extraction; §18.3.1 SQL produces per-incident timeline equivalent to option (b) in < 100 ms; (4) DEC-043/DEC-044/DEC-051 pattern — FORM consistently prefers documented auditor protocol over complex automated infrastructure at this scale; (5) upgrade path preserved — option (b) is fully additive (new column, new event types, no re-hashing). §18.3 skip-and-verify algorithm: §18.3.1 canonical auditor SQL (extracts per-incident events via `payload->>'incident_id' = :incident_id`, with `sub_chain_seq` ordinal and `master_chain_prev_event_id` for master-chain predecessor identification); §18.3.2 HMAC spot-verification procedure (primary path: SOC2_READINESS §79.7 master chain check; optional per-incident path: three-step HMAC-SHA256 re-computation against active epoch key); §18.3.3 CONC-CHAIN-01 cross-contamination invariant (detection query: zero rows = invariant intact; any rows = escalate to security-engineer). §18.4 concurrent incident coordination protocol: four-step IC procedure (distinct ID confirmation in #incidents, dedicated per-incident Slack channels, separate IC per ≥ P1 incident, post-recovery §18.3.1 + §18.3.3 cross-check before filing CONC-E-001); single-IC management of two concurrent P0s prohibited. §18.5 auditor fieldwork protocol: five-item document spec for `compliance/fieldwork/concurrent-incident-chain-verification.md` including Tabletop Scenario P worked example. §18.6 SOC 2 evidence: CONC-E-001 (CC7.4/CC4.1 — §18.3.1 query output + §18.3.3 zero-row result per quarter; filed even on zero concurrent-incident quarters as a zero-event attestation; 7yr retention); auditor narrative for CC7.4 supplied. §18.7 five-item implementation checklist: 3× P1/M7 (fieldwork document, baseline CONC-E-001 attestation, IC Quick-Start addendum); 2× P2/M8–Annual (Tabletop Scenario P, annual re-evaluation gate). §18.8 OQ-IR-01 status: 🟢 Resolved (DEC-053, 2026-06-14). §16.9 OQ-IR-01 updated to 🟢 Resolved. §17.7 resolved table extended with OQ-IR-01 row. Document header updated v2.2 → v2.6. Privacy floor: no individual employee health data in any §18 SQL — queries filter by `incident_id` (FORM-internal format `INC-YYYYMMDD-XXXXXX`); compliance_reviewer role access governed by Supabase RLS per `docs/AUDIT_LOG_SCHEMA.md`; no `user_id`, email, or Art. 9 category appears in any §18 evidence artefact. Cross-references: §16.9 (OQ-IR-01 source — all three open questions now 🟢 resolved); §17.7 (resolved table — OQ-IR-01 row added); `docs/SOC2_READINESS.md §79.7` (master chain integrity SQL — primary verification path for §18.3.2); `docs/AUDIT_LOG_SCHEMA.md` (CONC-CHAIN-01 — no new DEC-030 event types required; existing chain structure sufficient); R-05 (HMAC chain break — master chain integrity check covers interleaved concurrent-incident events; CONC-CHAIN-01 violation ≠ chain break, handled by security-engineer review before R-05 activation); §9.4 (tabletop catalog — Scenario P to be added per §18.7 item 4); §16.3 (HMAC sub-chain spec — §18.2.2 explains why option (b) violates the single-list invariant); `docs/DECISION_LOG.md DEC-053`. Owner: security-engineer + compliance-officer.*

---

**v2.6 · 2026-06-14 · Owner: security-engineer + compliance-officer**
**Review: after every P0/P1 incident, minimum annual.**
**Next scheduled review: June 2027 or after first P0/P1 — whichever comes first.**

---
