# FORM-DR-005 · Total Vendor Outage — Nuke Scenario Runbook

> **Runbook ID:** FORM-DR-005 · **Version:** 1.0 · **Last tested:** _(pending first drill — see §9)_
> **Classification:** Internal — Enterprise Confidential. Share under NDA only.
> **Owner:** `devops-lead` + `compliance-officer`. Execution: founder + on-call engineer jointly.
> **SOC 2 controls:** CC7.4, CC7.5, A1.2, A1.3, CC9.1.
> **RTO target:** 8 hours (worst-case cold start). **RPO target:** Determined by last cold backup timestamp.
> **Related:** `docs/BUSINESS_CONTINUITY.md §6.4`, `docs/INCIDENT_RESPONSE.md §5.5`, `docs/DATA_MODEL.md §10`.
> **DEC-030:** Audit chain resumes from restored state; breach risk assessment required in §8.
> **Requires joint execution:** Founder approval is mandatory before every irreversible step.

---

## Purpose

This runbook covers the **worst-case multi-vendor failure**: simultaneous or cascading loss of Supabase project data AND Cloudflare Workers configuration AND production credentials. This scenario requires rebuilding FORM's entire production environment from cold storage backups.

**Plausible triggers:**
- Compromised Supabase account (attacker deletes project).
- Supabase catastrophic data loss at provider level (not covered by their SLA).
- Compromised 1Password team vault leading to mass credential deletion/rotation by attacker.
- Extreme operator error affecting multiple systems simultaneously.

**What this is NOT:**
- Supabase platform incident with data intact → use FORM-DR-003.
- Cloudflare Workers bug or platform incident → use FORM-DR-004.
- Data corruption with PITR restore possible → use FORM-DR-003 §3.

**Critical limitations to communicate upfront:**

> Cold backup automation to Backblaze B2 is not yet implemented (BCP-04 known gap). Until it is, PITR via Supabase is the sole backup mechanism. If the Supabase project is fully deleted, cold restore from a manually exported dump is required. If no manual export exists, **there is no recovery path for data created after the last manual export**. This must be communicated immediately to all enterprise customers.

---

## Pre-Execution Checklist

This checklist is the incident gate. Complete before any recovery steps.

```
[ ] 1. I have confirmed the failure scope: Supabase PITR is NOT available (project deleted or inaccessible).
[ ] 2. I have confirmed this is NOT covered by FORM-DR-003 (PITR available) or FORM-DR-004 (Worker-only).
[ ] 3. I have notified the founder. FOUNDER IS REQUIRED for this runbook. Solo execution is not permitted.
[ ] 4. I have access to 1Password team vault → "DR / Cold Storage" entry.
        If 1Password is also compromised: see §2.1 (Emergency Credential Recovery).
[ ] 5. I have declared a P0 incident in PagerDuty and the Linear incident tracker.
[ ] 6. I have started a timestamped incident log (Notion, Google Doc, or plain text file in a safe location).
[ ] 7. compliance-officer has been notified. GDPR 72-hour clock assessment must start immediately (§8.1).
[ ] 8. I have access to GitHub repository (github.com/rbit27/form) — required for schema migrations.
[ ] 9. I have confirmed git clone access: `git clone git@github.com:rbit27/form.git /tmp/form-restore` succeeds.
[ ] 10. I have estimated backup age: last cold backup timestamp = ____________ (from Backblaze B2 or last manual export).
         Estimated data loss window: ____________ minutes/hours.
         RPO breach confirmed: YES / NO. If YES: compliance-officer notified (done in step 7).
```

**Stop here if the founder is not reachable.** Wait up to 30 minutes before proceeding unilaterally to §2, and only if enterprise tenant data loss risk exceeds the risk of waiting.

---

## §1 — Incident Communication (T+0 to T+15 min)

Before any recovery work, communications must go out. A P0 silence is worse than a P0 outage.

### Step 1.1 — Enterprise tenant notification

Send immediately via all available channels (Slack Connect, email, phone if necessary):

```
Subject: FORM Critical Service Outage — [DATE TIME UTC] — PLEASE READ

FORM is currently experiencing a critical infrastructure failure.

Status: Complete service unavailability. Active recovery in progress.
Expected restoration: Within 8 hours ([TARGET TIME UTC]).
Data impact: [Unknown pending assessment / Data created after [TIMESTAMP] may not be recoverable].
Root cause: Infrastructure failure under investigation.

Your dedicated CSM is [NAME] and will contact you directly within 30 minutes.
We will provide updates every hour at minimum.

— FORM Incident Commander
```

### Step 1.2 — Update all status channels

- Better Uptime status page: "Critical outage — full service restoration in progress."
- `form.coach/status` — may be unavailable; update Better Uptime as fallback.

### Step 1.3 — Regulatory assessment

```
[ ] compliance-officer: Start 72-hour GDPR Art. 33 clock if personal data is affected.
    Clock start: [TIMESTAMP] (time of confirmed data loss, not time of discovery).
    DPA notification deadline: [TIMESTAMP + 72 hours].

[ ] Does the data loss involve EU-resident user data? YES / NO.
[ ] Does the data loss involve special category data (health/fitness metrics)? YES / NO.
    If YES: Art. 9 breach — supervisory authority notification is mandatory within 72 hours.

[ ] If notification is required:
    - Supervising authority: [Country] DPA
    - Template: docs/INCIDENT_RESPONSE.md §9 E-EU-01
    - compliance-officer owns the notification.
```

---

## §2 — Credential Recovery

### Step 2.1 — Standard path (1Password accessible)

```
1. Open 1Password → "DR / Cold Storage" entry.
2. Retrieve:
   [ ] Backblaze B2 Application Key (Key ID + Application Key)
   [ ] Backblaze B2 Bucket name: form-dr-backups
   [ ] GPG decryption key (passphrase or key file)
   [ ] Supabase project reference (for new project creation)
   [ ] Cloudflare API token (for Workers deployment)
   [ ] WorkOS API key (for SSO configuration)
   [ ] Stripe API key (for billing)
   [ ] Anthropic API key
   [ ] ElevenLabs API key
   [ ] PostHog write key
   [ ] Sentry DSN

3. Copy all credentials into a temporary secure local file (NOT a shared doc):
   /tmp/form-restore-credentials.env  (chmod 600)
   Delete this file at the end of §7.
```

### Step 2.2 — Emergency path (1Password also compromised)

If 1Password is inaccessible or suspected compromised:

```
[ ] 1. Contact Backblaze B2 via account recovery: storage.backblazeb2.com → "Forgot password".
       Requires access to founder email account.

[ ] 2. Contact Supabase via account recovery: supabase.com → "Forgot password".
       Requires access to founder email account.

[ ] 3. Contact Cloudflare via account recovery: cloudflare.com → "Forgot password".
       Requires access to founder email account.

[ ] 4. If founder email is also compromised: this is a full identity compromise incident.
       Invoke docs/INCIDENT_RESPONSE.md §5.5 (Security Breach) in parallel.
       Contact legal counsel immediately.
       Recovery timeline extends to 24–48 hours.
```

---

## §3 — Database Restore from Cold Backup

**Founder approval required before this section.** Log approval:

```
Founder approval received at: _____________ (timestamp)
Approval method: [ ] In-person [ ] Phone [ ] Slack DM
Statement: "Proceed with cold restore from Backblaze B2 backup."
```

### Step 3.1 — Download backup from Backblaze B2

```bash
# Install B2 CLI if not available
pip install b2 || brew install b2-tools

# Authenticate
b2 authorize-account "$B2_KEY_ID" "$B2_APPLICATION_KEY"

# List available backups (most recent first)
b2 ls --long form-dr-backups/

# Download the most recent backup
BACKUP_FILE="form-backup-2026-06-07T00-00-00Z.dump.gpg"  # Replace with actual filename
b2 download-file-by-name form-dr-backups "$BACKUP_FILE" "/tmp/$BACKUP_FILE"

# Verify download integrity
b2 get-file-info form-dr-backups "$BACKUP_FILE" | grep sha1
sha1sum "/tmp/$BACKUP_FILE"
# SHA1 sums must match. If they do not: backup is corrupted. Try the previous day's backup.
```

### Step 3.2 — Verify GPG signature and decrypt

```bash
# Import GPG key from 1Password "DR / Cold Storage" → "GPG Key"
gpg --import /tmp/form-dr-gpg-key.asc

# Verify signature (backup must be signed by the FORM backup key)
gpg --verify "/tmp/$BACKUP_FILE.sig" "/tmp/$BACKUP_FILE" 2>&1
# Expected: "Good signature from FORM Backup Key <devops@form.coach>"
# If signature fails: DO NOT proceed. The backup may be tampered. Escalate to compliance-officer.

# Decrypt
gpg --decrypt "/tmp/$BACKUP_FILE" > /tmp/form-backup-decrypted.dump
```

### Step 3.3 — Create new Supabase project

```
1. Go to dashboard.supabase.com → New Project.
2. Organization: form-production
3. Name: form-production-restored-[DATE]
4. Database password: Generate new 32-character random password. Store in 1Password immediately.
5. Region: Select based on enterprise customer contracts:
   - US customers → us-east-1
   - EU customers → eu-central-1
   - If mixed: create two projects (additional complexity; discuss with founder).
6. Pricing: Pro plan (required for PITR and connection pooling).
7. Wait for project provisioning (2–5 minutes).
8. Note new project reference: ____________________
```

### Step 3.4 — Restore database dump

```bash
# Get connection string from new Supabase project → Settings → Database → Connection string (URI mode)
NEW_DB_URL="postgresql://postgres:[PASSWORD]@[HOST]:5432/postgres"

# Restore dump
pg_restore \
  --format=custom \
  --no-owner \
  --no-privileges \
  --dbname="$NEW_DB_URL" \
  /tmp/form-backup-decrypted.dump \
  2>&1 | tee /tmp/form-restore-output.log

# Check for errors
grep -i "error" /tmp/form-restore-output.log | head -20
# Acceptable: "already exists" errors for system objects
# Not acceptable: "relation does not exist", "permission denied", data errors
```

### Step 3.5 — Apply schema migrations to HEAD

The cold backup may be from an older schema version. Apply all pending migrations:

```bash
# Clone the repository (if not already done)
git clone git@github.com:rbit27/form.git /tmp/form-restore

# Install Supabase CLI
npm install -g supabase

# Link to new project
supabase link --project-ref "$NEW_PROJECT_REF" --password "$NEW_DB_PASSWORD"

# Check current migration state
supabase db diff --schema public 2>&1 | head -50

# Apply pending migrations
supabase db push 2>&1 | tee /tmp/form-migrations-output.log

# Verify: no pending migrations remain
supabase db diff --schema public 2>&1
# Expected: empty diff (no changes needed)
```

---

## §4 — Cloudflare Workers Restore

### Step 4.1 — Re-enter all Worker secrets

```bash
# Authenticate wrangler with the Cloudflare API token from 1Password
export CLOUDFLARE_API_TOKEN="$CF_API_TOKEN"
wrangler whoami  # Verify authentication

# Re-enter every secret (do NOT re-use old values — rotate all secrets during restore)
# Secrets to re-enter (generate new values for each):
wrangler secret put SUPABASE_URL --env production    # New project URL
wrangler secret put SUPABASE_ANON_KEY --env production  # From new Supabase project
wrangler secret put SUPABASE_SERVICE_KEY --env production  # From new Supabase project
wrangler secret put WORKOS_API_KEY --env production  # From 1Password (or new if rotated)
wrangler secret put WORKOS_CLIENT_ID --env production
wrangler secret put ANTHROPIC_API_KEY --env production
wrangler secret put ELEVENLABS_API_KEY --env production
wrangler secret put STRIPE_SECRET_KEY --env production
wrangler secret put POSTHOG_WRITE_KEY --env production
wrantml secret put HMAC_SIGNING_KEY --env production  # MUST be new — do not re-use old key
wrangler secret put IP_HASH_SALT --env production    # MUST be new — do not re-use old value

# Log all secret rotation events in audit_events (once DB is accessible in §5)
```

### Step 4.2 — Deploy Worker from main branch

```bash
cd /tmp/form-restore

# Verify the code is at main HEAD
git log --oneline -3

# Update wrangler.toml to point to new Supabase project URL (if hardcoded)
# OR: rely on the SUPABASE_URL secret set in Step 4.1

# Deploy
wrangler deploy --env production 2>&1 | tee /tmp/form-worker-deploy.log

# Verify deployment
wrangler deployments list --env production | head -5
curl -sf https://api.form.coach/api/health | jq '.status, .version'
# Expected: status "ok" (note: DNS may take a few minutes to propagate if new Workers route)
```

### Step 4.3 — Re-point DNS (if new project required new Worker route)

If the Workers route changed during restore:

```
1. Cloudflare dashboard → DNS → api.form.coach CNAME record.
2. Update target to new Workers route.
3. Wait for DNS propagation: 1–5 minutes (Cloudflare is authoritative, so fast).
4. Verify: curl -sf https://api.form.coach/api/health | jq '.status'
```

---

## §5 — Re-enable Enterprise Services

### Step 5.1 — WorkOS SSO reconfiguration

WorkOS SSO configuration (SAML IdP metadata, OIDC client IDs) is stored in WorkOS, not in the database. If WorkOS account is intact:

```
1. Log in to WorkOS dashboard.
2. Verify all enterprise connections are still active (Connections → each tenant).
3. Test SSO with a test tenant: initiate SP-initiated flow.
4. If WorkOS account is compromised: contact WorkOS support for account recovery.
   Recovery timeline: 24–48 hours (blocking for enterprise SSO).
```

### Step 5.2 — SCIM provisioning

SCIM provisioning uses WorkOS SCIM tokens. If WorkOS is intact, SCIM should resume automatically on the restored environment. Verify:

```bash
curl -sf -H "Authorization: Bearer $TEST_SCIM_TOKEN" \
  "https://api.form.coach/scim/v2/ServiceProviderConfig" | jq '.schemas'
# Expected: non-empty
```

### Step 5.3 — Stripe billing re-integration

```
1. Log in to Stripe dashboard.
2. Verify webhook endpoint: stripe.com → Developers → Webhooks → api.form.coach/api/stripe/webhook.
3. If webhook is misconfigured: update to new Workers URL and re-verify with Stripe.
4. Verify test event delivery from Stripe → FORM endpoint.
```

---

## §6 — Privacy Floor & Data Integrity Verification

**This section is non-negotiable.** Do not proceed to §7 without completing every check here.

### Step 6.1 — Privacy Floor verification

```sql
-- Connect to restored Supabase project
-- Test 1: Admin aggregate view returns no individual identifiers
SELECT column_name
FROM information_schema.columns
WHERE table_name = 'v_team_wellness_aggregate'
  AND column_name IN ('user_id', 'employee_name', 'email', 'user_email');
-- Expected: 0 rows (no individual identifiers in admin view)

-- Test 2: RLS prevents cross-tenant data access
SET LOCAL role = 'form_member_role';
SET LOCAL request.tenant_id = '[TENANT_A_UUID]';
SELECT COUNT(*) FROM user_profiles WHERE tenant_id = '[TENANT_B_UUID]';
-- Expected: 0 rows

-- Test 3: Aggregate query returns only counts, not individual values
SELECT * FROM v_team_wellness_aggregate
WHERE tenant_id = '[TEST_TENANT_UUID]'
LIMIT 1;
-- Expected: aggregate statistics only. No user_id, email, or health metric rows.
```

**If any test fails:**

```
[ ] DO NOT lift maintenance mode or allow any customer access.
[ ] Emit: privacy.floor_breach_detected (CRITICAL, DEC-030) once audit_events table is writable.
[ ] Notify compliance-officer and clinical-safety immediately.
[ ] Investigate whether the restore introduced a schema migration error affecting RLS policies.
[ ] Re-apply RLS policies from docs/DATA_MODEL.md §4 if needed.
```

### Step 6.2 — Row count sanity check

```sql
SELECT
  tablename,
  n_live_tup AS row_count
FROM pg_stat_user_tables
WHERE schemaname = 'public'
ORDER BY n_live_tup DESC;
```

Compare against pre-incident baseline in `compliance/evidence/cc7/baseline-row-counts.csv`. Document any delta — this delta is the data loss to report to enterprise customers.

### Step 6.3 — HMAC audit chain assessment (DEC-030)

The audit chain in the restored database ends at the last event in the cold backup. All events between the backup timestamp and the incident are permanently lost.

```sql
-- Find the last event in the restored chain
SELECT id, event_type, created_at, hmac_self
FROM audit_events
ORDER BY created_at DESC
LIMIT 1;
```

Document:
```
Last audit event in restored chain: [created_at timestamp]
Audit log gap: from [^above timestamp] to [incident timestamp]
Duration of audit gap: _______ hours/minutes
Compliance risk: SOC 2 auditors must be informed of the gap window.
```

This gap is an expected consequence of the nuke scenario and does not constitute a chain break. The new chain resumes from the last restored event. Document this in the post-mortem.

---

## §7 — Full Smoke Test Suite

All tests must pass before lifting maintenance mode.

```bash
# F1: Health
curl -sf https://api.form.coach/api/health | jq '.status'
# Expected: "ok"

# F2: Auth
curl -sf -H "Authorization: Bearer $TEST_REFRESH_TOKEN" \
  https://api.form.coach/api/auth/refresh | jq '.access_token'
# Expected: non-null JWT

# F3: Workout read
curl -sf -H "Authorization: Bearer $TEST_ACCESS_TOKEN" \
  https://api.form.coach/api/workouts?limit=1 | jq '.data | length'
# Expected: >= 0

# F4: Workout write
curl -sf -X POST -H "Authorization: Bearer $TEST_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"strength","duration_sec":1}' \
  https://api.form.coach/api/workouts | jq '.id'
# Expected: UUID

# F5: SSO health
curl -sf "https://api.form.coach/api/sso/health?tenant_id=$TEST_ENTERPRISE_TENANT" \
  | jq '.saml_configured'
# Expected: true

# F6: Admin aggregate (Privacy Floor)
curl -sf -H "Authorization: Bearer $TEST_ADMIN_TOKEN" \
  "https://api.form.coach/api/admin/wellness-aggregate?tenant_id=$TEST_ENTERPRISE_TENANT" \
  | jq 'has("user_id")'
# Expected: false

# F7: Audit log write (new chain must start correctly)
curl -sf -X POST -H "Authorization: Bearer $TEST_ACCESS_TOKEN" \
  https://api.form.coach/api/debug/audit-ping | jq '.hmac_valid'
# Expected: true

# F8: SCIM health
curl -sf -H "Authorization: Bearer $TEST_SCIM_TOKEN" \
  "https://api.form.coach/scim/v2/ServiceProviderConfig" | jq '.schemas'
# Expected: non-empty
```

---

## §8 — Lift Maintenance Mode & Customer Communication

```bash
# Remove maintenance mode flag
wrangler kv key delete --binding=SYSTEM_KV "system:maintenance" --env production

# Final verification
curl -sf https://api.form.coach/api/health | jq '.status'
# Expected: "ok"
```

### Step 8.1 — Enterprise customer data impact notification

This is the most sensitive communication in any DR event. Be specific about data loss.

```
Subject: FORM Service Restored — Data Impact Assessment — [DATE TIME UTC]

FORM services have been restored as of [TIME UTC].
Total outage duration: [X hours].

DATA IMPACT SUMMARY:
- Service restored from backup dated: [BACKUP TIMESTAMP UTC]
- Data created between [BACKUP TIMESTAMP] and [INCIDENT TIMESTAMP] could not be recovered.
- Estimated data affected: [X workout sessions / Y user records / Z audit events] for your organisation.
- Individual user notifications: [Will be / Will not be] required per GDPR Art. 34.

SLA CREDIT:
- Applicable credit: [X%] of monthly fee per docs/ENTERPRISE_SLA.md §5.
- Credit will be applied automatically to your next invoice.

NEXT STEPS:
- Full post-incident report with root cause: within 5 business days.
- Your CSM [NAME] will contact you within 1 hour to discuss impact and recovery.

We are deeply sorry for this disruption.
— FORM Incident Commander
```

### Step 8.2 — Regulatory notifications (if applicable)

```
[ ] GDPR Art. 33 (if EU personal data affected):
    Notify relevant DPA within 72 hours of data loss confirmation.
    Template: docs/INCIDENT_RESPONSE.md §9 E-EU-01.
    Owner: compliance-officer.

[ ] GDPR Art. 34 (if high risk to data subjects):
    Notify affected individuals if data loss creates high risk.
    Template: docs/INCIDENT_RESPONSE.md §9 E-EU-02.
    Owner: compliance-officer.

[ ] If no personal data was affected: document this determination and file to compliance/evidence/.
```

### Step 8.3 — Post-restore credential cleanup

```bash
# Securely delete temporary credentials file
shred -vzu /tmp/form-restore-credentials.env

# Securely delete decrypted backup
shred -vzu /tmp/form-backup-decrypted.dump

# Remove GPG temporary key material
gpg --delete-secret-and-public-key form-dr-backup@form.coach

# Emit all credential rotation events to audit_events (new chain)
# One INSERT per rotated credential (HMAC_SIGNING_KEY, IP_HASH_SALT, SUPABASE_*, etc.)
```

### Step 8.4 — Emit restore completion audit event

```sql
INSERT INTO audit_events (event_type, severity, actor_type, metadata)
VALUES ('system.cold_restore_completed', 'HIGH', 'system',
  '{"incident_ref": "FORM-DR-005-[DATE]",
    "backup_timestamp": "[ISO8601]",
    "restore_completed_at": "[ISO8601_NOW]",
    "rto_minutes": [X],
    "rpo_breach": true,
    "data_loss_window_minutes": [Y],
    "smoke_tests_passed": 8,
    "privacy_floor_verified": true,
    "gdpr_notification_required": true | false,
    "new_supabase_project_ref": "[REF]",
    "lifted_by": "[founder_user_id]"}');
```

---

## §9 — Evidence Filing & Post-Mortem

### Mandatory evidence package

```
[ ] Running incident log (timestamped actions from T+0 to restoration).
[ ] 1Password access log (if available — when did DR credentials get accessed).
[ ] Backblaze B2 download receipt (file name, timestamp, size, SHA1).
[ ] GPG signature verification output.
[ ] pg_restore output log (/tmp/form-restore-output.log).
[ ] Supabase migration output (/tmp/form-migrations-output.log).
[ ] Worker deployment log (/tmp/form-worker-deploy.log).
[ ] Privacy Floor verification SQL output (screenshot or text).
[ ] Row count delta (pre vs post restore).
[ ] Smoke test results (F1–F8 timestamps and outputs).
[ ] Customer notification timestamps (channel, recipient, time sent).
[ ] GDPR notification filing confirmation (if applicable).
[ ] Credential rotation log (list of all rotated secrets with timestamps).
```

File to: `compliance/evidence/cc7/incidents/FORM-DR-005-[DATE]/`

### Post-mortem (mandatory within 5 business days)

Write using `docs/INCIDENT_RESPONSE.md §8` template. Additional required sections for nuke scenarios:
- **Root cause:** How did multi-system failure occur? What attacker/operator path enabled this?
- **Detection gap:** How long between failure and detection? How to reduce detection time?
- **Backup automation gap:** When will automated cold backup to Backblaze B2 be implemented? (BCP-04)
- **Data loss disclosure:** Exact data loss window; individual user impact; regulatory implications.
- **Future prevention:** Key recommendations (access controls, backup frequency, credential hygiene).

### DR drill evidence (if planned drill)

```
[ ] File: enterprise/runbooks/test-reports/FORM-DR-005-drill-[DATE].md
[ ] Document: sections that could be executed vs sections that were simulated only.
[ ] Update §1 header: "Last tested: [DATE] — [Engineer name]"
[ ] Update: compliance/cc9/README.md CC9-GAP-005 status.
```

---

## §10 — Known Gaps & Limitations

| Gap ID | Description | Impact | Resolution Path |
|---|---|---|---|
| **BCP-04** | No automated nightly cold backup to Backblaze B2. | RPO in nuke scenario = age of last manual export. Enterprise RPO SLA breached. | Automate pg_dump → GPG encrypt → B2 upload as nightly cron. Q3 implementation roadmap. |
| **BCP-03** | Sole operator (founder only) pre-Series A. | Nuke scenario requires joint execution; founder unavailability is a compounding risk. | First engineering hire: full production access + joint DR training within 30 days. |
| Supabase data recovery | Supabase Pro plan does not guarantee PITR availability in project deletion scenarios. | Nuke scenario RTO is 8 hours, exceeding 4-hour enterprise contractual RTO. | Documented in ENTERPRISE_SLA.md §3.1 as a carved-out exception for multi-provider failures. |
| WorkOS SSO continuity | WorkOS SSO connections survive the Supabase restore but depend on WorkOS account access. | If WorkOS is also compromised, enterprise SSO restoration adds 24–48 hours. | WorkOS account MFA + emergency contact on file with WorkOS. |

---

## §11 — Quick Reference Card

| Condition | Action |
|---|---|
| 1Password accessible, Supabase deleted | §2.1 → §3 cold restore → §4 Worker restore |
| 1Password also compromised | §2.2 emergency credential recovery (24–48h extra) |
| GPG signature fails on backup | Stop. Escalate to compliance-officer. Try previous backup. |
| Privacy Floor test fails post-restore | Stop. Call compliance-officer + clinical-safety. Do not open to customers. |
| Audit chain gap found (expected) | Document and file. Not a chain break — document in post-mortem. |
| All F1–F8 pass + Privacy Floor verified | §8 lift maintenance → data impact communication |
| EU personal data affected | §8.2 GDPR Art. 33 notification within 72h of data loss confirmation |

---

## §12 — Contact Escalation Tree

| Role | Contact | When |
|---|---|---|
| **Founder** | 1Password → "Emergency Contacts" | T+0 — required before any restore step |
| **compliance-officer** | 1Password → "Emergency Contacts" | T+0 — 72h GDPR clock; Privacy Floor failures; SLA credit |
| **clinical-safety** | 1Password → "Emergency Contacts" | Privacy Floor failure involving health data |
| **Supabase Support** | dashboard.supabase.com → Support | Account recovery; understanding root cause |
| **WorkOS Support** | workos.com → Support | SSO connection recovery |
| **Legal counsel** | 1Password → "Emergency Contacts" | Regulatory notification filings; attacker-driven compromise |
| **Enterprise CSM contacts** | Linear → "Customer Contacts" | Customer communication within 30 minutes |

---

*v1.0 · 2026-06-07 · Owner: devops-lead + compliance-officer*
*Approved by: compliance-officer · Joint execution required: founder + on-call engineer*
*Next review: after first drill execution or any infrastructure architecture change*
*SOC 2 evidence: CC7.4, CC7.5, A1.2, A1.3, CC9.1 · HMAC audit chain: DEC-030*
*Known gap: BCP-04 (cold backup automation) — RPO breach is expected in this scenario until resolved*
