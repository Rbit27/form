# FORM · Incident Response Runbook v0.1

> Owner: security-engineer + devops-lead. Review: after every P0/P1 incident, minimum annual. SOC 2 evidence: CC7.2–CC7.5, CC9.2.

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

**Trigger:** Weekly HMAC chain integrity cron fails and pages security-engineer. Or: anomaly detected in chain validation during a separate investigation.

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

## Appendix A — Quick Reference Card

For use at 3am. IC: read §1 to classify, open the incident channel, then jump to the relevant runbook in §5.

```
P0?  → Open #inc-YYYYMMDD-slug immediately. Page security-engineer + IC. Start GDPR clock.
P1?  → Open #inc-YYYYMMDD-slug. Page IC. Monitor for upgrade.
P2?  → Track in Linear. Handle in business hours unless escalating.

Data breach?                → R-01
Outage?                     → R-02
Creds compromised?          → R-03
SSO down?                   → R-04
Chain break?                → R-05 (always P0)
DDoS?                       → R-06
Processor notified us?      → R-07 (72h clock starts NOW — page compliance-officer)
Compromised npm package?    → R-08 (freeze deploys, check Workers env surface)
Ransomware / destruction?   → R-09 (ISOLATE FIRST — do not pay)

GDPR Art. 33 clock: 72h from first awareness, not from confirmation.
  Sub-processor breach: clock starts when FORM receives notification, not when
  processor discovered the incident. File partial notification at T+48h maximum.
R-09 exception: Isolation BEFORE evidence — stop spread first.
Evidence first, containment second — unless live exfiltration (R-01) or live destruction (R-09).
Never modify production logs. Read-only copies only.
Log every action in incident channel with timestamp and your name.
Sub-processor registry: §11.2 — check which data categories the vendor holds.
Enterprise tenant comms: §12 — Customer Lead owns; IC approves all external statements.
  Template E-01 within 30 min (P0) / 60 min (P1) of declaration.
```

---

**v0.3 · May 2026 · Owner: security-engineer + devops-lead**
**Review: after every P0/P1 incident, minimum annual.**
**Next scheduled review: May 2027 or after first P0/P1 — whichever comes first.**

*v0.2 additions: R-07 Sub-processor Breach Notification Received (new runbook — vendor-originated incident protocol, GDPR Art. 33 clock management when breach is at processor level, enterprise tenant notification, evidence package, post-incident vendor management review). §11 Sub-processor Incident Management (processor registry for Supabase/Anthropic/ElevenLabs/Cloudflare/PostHog/Sentry/Stripe with Art. 9 classification and severity floors; known DPA gaps including Anthropic notification SLA and ElevenLabs unsigned DPA; Art. 28(3)(f) contractual requirements and DPA signing checklist; notification SLA tracker; SOC 2 CC9.2 evidence integration). Appendix A updated to include R-07 quick reference and sub-processor context for GDPR clock.*

*v0.3 additions: R-08 Supply Chain Attack / Compromised Dependency — trigger matrix by package location and runtime access surface; immediate isolation protocol (freeze deploys, EAS revocation); blast radius assessment checklist; dependency tree audit with clean-build verification; preventive CI controls (Dependabot, npm audit, Socket.dev); SOC 2 CC6.1/CC6.8/CC7.2/CC7.4 evidence package. R-09 Ransomware / Destructive Payload — always-P0 protocol with explicit isolation-first ordering (overrides the standard evidence-first rule); no-pay policy with decision authority constraints; Supabase PITR recovery to new project; full secrets rotation sequence; enterprise tenant validation before re-enabling traffic; law enforcement preservation guidance; GDPR note on destruction as a reportable breach. §9.3 Scenarios E and F — supply chain (backdoored npm package in Workers env) and ransomware (staging-to-production lateral movement) tabletop scenarios with discussion point frameworks; both include enterprise customer-success participation requirement. §9.5 Year 2 Testing Schedule — 6-scenario rotation covering all runbook types; cross-functional drills (customer-success joins E and F). §12 Enterprise Tenant SLA Breach & Incident Communication Protocol — SLA tier definition and breach thresholds for API availability, P95 latency, and SSO; credit schedule (10%/25%/50% by uptime band); contact responsibility matrix with timing SLAs; privacy floor enforcement for tenant communications; Templates E-01 (initial notification), E-02 (60-min status update), E-03 (resolution), E-04 (SLA credit); SOC 2 evidence mapping (CC9.2, A1.1, A1.2, CC7.3, CC2.2). Appendix A updated to include R-08, R-09, and §12 enterprise comms reference.*
