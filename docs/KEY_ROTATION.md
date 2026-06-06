# FORM · Key Rotation Runbook

**Version:** v1.0
**Status:** IN FORCE
**Effective Date:** June 2026
**Owner:** security-engineer (primary), compliance-officer (dual-custody keys)
**Policy reference:** `docs/CRYPTOGRAPHY_POLICY.md` — defines schedules and standards; this doc implements them step-by-step.
**Failure escalation:** `docs/INCIDENT_RESPONSE.md R-21` — key rotation failure or overdue P0 incident.
**SOC 2 controls:** CC5.2, CC5.3, CC6.7, CC6.8, C1.1

> **What this document is:** Step-by-step rotation procedures for every production secret FORM holds. Run this when AL-KEY-01 or AL-KEY-02 fires, or on the schedule in §2. If a rotation fails mid-procedure, stop and open R-21 — do not improvise.
>
> **What this document is not:** Cryptographic policy (→ `CRYPTOGRAPHY_POLICY.md`), incident handling for compromise (→ `INCIDENT_RESPONSE.md R-03/R-21`), or key monitoring architecture (→ `OBSERVABILITY.md §30`).

---

## 1. Purpose and Scope

This runbook ensures that every FORM cryptographic credential is rotated on schedule, that rotation events are HMAC-chained to the DEC-030 audit log, and that SOC 2 evidence is collected in the compliance artefact store.

**In scope:** All production secrets listed in §2. No development or staging secrets require this procedure unless explicitly noted.

**Out of scope:** OAuth tokens (wearable integrations — auto-refreshed every 2 hours by platform-engineer; revocation on consent withdrawal handled in `DATA_MODEL.md §14.8`). TLS certificates (Cloudflare auto-renews; human procedure only triggered when mobile cert pin must update — §5.12).

**Dual-custody requirement:** `HMAC_AUDIT_CHAIN_KEY` and `BACKUP_ENC_KEY` require both security-engineer and compliance-officer to be present. All other keys may be rotated by security-engineer alone. Dual-custody keys are marked **[DC]** in §2.

---

## 2. Key Inventory

The `form-crypto-health` Cloudflare KV namespace tracks rotation state per key. Each entry stores:

```json
{
  "last_rotated": "2026-04-01T10:00:00Z",
  "next_rotation_due": "2026-07-01T10:00:00Z",
  "version": "v4",
  "days_overdue": 0
}
```

KV key pattern: `key:rotation:{KEY_ID}`.

| KEY_ID | Type | Storage | Rotation Period | Dual Custody | Owner | SOC 2 |
|---|---|---|---|---|---|---|
| `SUPABASE_SERVICE_ROLE_JWT` | HS256 JWT | Supabase dashboard + Cloudflare Workers Secrets | **90 days** | No | security-engineer | CC6.7, C1.1 |
| `HMAC_AUDIT_CHAIN_KEY` | HMAC-SHA256 | 1Password dual-vault (security-engineer + compliance-officer) | **Annual** | **[DC]** | security-engineer + compliance-officer | CC6.7, CC6.8, C1.1 |
| `KEYPOINTS_ENC_KEY` | AES-256-CBC (Supabase Vault / pgsodium) | Supabase Vault | **Annual** | No | security-engineer | CC5.3, C1.1 |
| `CLOUDFLARE_API_TOKEN` | Ed25519 / opaque | 1Password Operations vault + Cloudflare dashboard | **180 days** | No | security-engineer | CC5.3, CC6.7 |
| `ANTHROPIC_API_KEY` | Opaque | 1Password Operations vault + Cloudflare Workers Secrets | **180 days** | No | security-engineer | CC5.3 |
| `ELEVENLABS_API_KEY` | Opaque | 1Password Operations vault + Cloudflare Workers Secrets | **180 days** | No | security-engineer | CC5.3 |
| `STRIPE_LIVE_API_KEY` | Opaque | 1Password Operations vault + Stripe dashboard | **180 days** | No | compliance-officer | CC5.3 |
| `SENTRY_DSN` | Opaque | 1Password Operations vault | **180 days** | No | security-engineer | CC5.3 |
| `SUPABASE_ANON_KEY` | JWT (public-facing) | Supabase dashboard + Cloudflare Workers Secrets | **365 days** | No | security-engineer | CC5.2 |
| `BACKUP_ENC_KEY` | AES-256-GCM | 1Password compliance-officer vault + offline USB | **Annual** | **[DC]** | compliance-officer | CC6.7, C1.1 |
| `API_KEY_HASH_SECRET` | 256-bit random | Cloudflare Workers Secrets only | **180 days** | No | security-engineer | CC5.3, CC6.4 |

**SAML/IDP certificates (per-tenant):** Stored in Supabase Vault, encrypted column. Managed per-tenant by platform-engineer during SSO onboarding. Not tracked in `form-crypto-health` KV. Rotation triggered by enterprise customer's IdP certificate renewal; see `SSO_SCIM_IMPLEMENTATION.md §24`.

---

## 3. Rotation Calendar

AL-KEY-01 (P1) fires 14 days before `next_rotation_due`. AL-KEY-02 (P0) fires on any overdue key. Do not wait for the alert — calendar reminders must be set by security-engineer when a rotation completes.

**Annual key ceremony (January):** All annual keys (`HMAC_AUDIT_CHAIN_KEY`, `KEYPOINTS_ENC_KEY`, `BACKUP_ENC_KEY`) are reviewed and rotated together where due. Documented in `SOC2_READINESS.md §15` compliance calendar item Q1-Jan.

To update KV state after a successful rotation:

```bash
# Cloudflare Workers KV write (from wrangler CLI with --env production)
wrangler kv:key put \
  --namespace-id <form-crypto-health-ns-id> \
  "key:rotation:SUPABASE_SERVICE_ROLE_JWT" \
  '{"last_rotated":"2026-07-01T10:00:00Z","next_rotation_due":"2026-10-01T10:00:00Z","version":"v5","days_overdue":0}' \
  --env production
```

Replace KEY_ID and dates for each key. The `key-rotation-scheduler` Worker reads this KV within 24 h and resolves the AL-KEY-02 PagerDuty alert.

---

## 4. Universal Pre-Rotation Checklist

Run through this list before starting any rotation. If any item is not clear, stop and consult security-engineer.

- [ ] **Alert context confirmed:** AL-KEY-01 (scheduled) or AL-KEY-02 (overdue) — not AL-KEY-04 or CRYPTO-CHAIN-02. If CRYPTO-CHAIN-02 fired (rotation event without a preceding initiation event), **stop: activate INCIDENT_RESPONSE R-05 + R-20, not this runbook.**
- [ ] **Maintenance window booked** (required for `KEYPOINTS_ENC_KEY`, `SUPABASE_SERVICE_ROLE_JWT` emergency only; optional for all others).
- [ ] **Dual-custody partner available** — required for `HMAC_AUDIT_CHAIN_KEY` and `BACKUP_ENC_KEY`. Confirm both security-engineer and compliance-officer are reachable.
- [ ] **PITR branch created** (required for `KEYPOINTS_ENC_KEY` due to in-place re-encryption migration).
- [ ] **Enterprise tenant notification sent** — `KEYPOINTS_ENC_KEY` rotation requires 72 h advance notice to enterprise tenants per `INCIDENT_RESPONSE.md R-11` protocol.
- [ ] **1Password vault access confirmed** — log in to Operations vault and verify the item for the key being rotated is visible.
- [ ] **Cloudflare Workers deployment access confirmed** — `wrangler whoami` returns the correct account; `wrangler secret list --env production` returns expected secrets.
- [ ] **No active P0 incident in progress** — check `#incident` Slack channel. Simultaneous rotation during active P0 requires founder approval.
- [ ] **Git repo clean** — `git status` shows no uncommitted changes to Workers or Edge Function code that could interfere with a `wrangler deploy`.

---

## 5. Per-Key Rotation Runbooks

### 5.1 SUPABASE_SERVICE_ROLE_JWT — 90 days

> **Full procedure:** `docs/SOC2_READINESS.md §57` (Step-by-step rotation including two-phase Cloudflare Workers Secrets update, maintenance window requirements, and DEC-030 event specification).

**Summary:**

1. Pre-rotation: confirm no active admin migration jobs; open maintenance window.
2. Rotate via Supabase dashboard: **Settings → API → Service Role Key → Rotate**.
3. Copy new key immediately (shown once).
4. Update Cloudflare Workers Secret: `wrangler secret put SUPABASE_SERVICE_ROLE_KEY --env production`.
5. Redeploy affected Workers: `wrangler deploy --env production` for each Worker that reads the secret.
6. Smoke-test: hit `/admin/health` endpoint; confirm 200 with `db: ok`.
7. Emit `system.credential_rotated` DEC-030 event (§6.1).
8. Update `form-crypto-health` KV (§3). Set `next_rotation_due = now + 90d`.
9. File evidence: Supabase key rotation log screenshot to `compliance/evidence/enc/supabase-service-role-rotation-YYYY-MM-DD.png`.

**Emergency rotation:** `SOC2_READINESS.md §57.5` — activate within 1 hour of confirmed compromise; treat as P0 parallel-track with R-03.

---

### 5.2 HMAC_AUDIT_CHAIN_KEY — Annual [DC]

> **Full procedure:** `docs/SOC2_READINESS.md §58` (dual-key design, chain continuity, `key_transition` metadata, historical verification).

**Summary:**

Requires security-engineer and compliance-officer to be simultaneously present (call or co-located). The old key is **never deleted** — it is moved to read-only custody for verifying historical chain entries.

1. Both parties log in to their respective 1Password vaults.
2. Generate new 256-bit HMAC key via CSPRNG: `openssl rand -hex 32`.
3. Assign new key ID: `HMAC_AUDIT_CHAIN_KEY_v{N+1}`.
4. Store in both 1Password vaults (security-engineer vault entry + compliance-officer vault entry).
5. Update the HMAC chain verifier function to accept both old and new key ID for historical vs. new entries. New entries from this moment use the new key ID.
6. Deploy updated chain verifier: `wrangler deploy --env production`.
7. Emit `admin.signing_key_rotated` DEC-030 event with `key_id`, `rotation_trigger: 'scheduled'`, `rotated_by` for both parties (§6.3).
8. Run full chain integrity re-verification against last 30 days: confirm no `admin.hmac_chain_break` events appear.
9. Emit `admin.hmac_key_rotation_verified` DEC-030 event (required within 24 h for KEY-SLO-03; triggers AL-KEY-04 if missing).
10. Update `form-crypto-health` KV.
11. File evidence: dual-vault 1Password audit log export (both entries) to `compliance/evidence/enc/hmac-chain-key-rotation-YYYY.pdf`.

---

### 5.3 KEYPOINTS_ENC_KEY — Annual

This key encrypts `cv_sessions.keypoints_enc` (AES-256-CBC via pgsodium). Rotation requires a re-encryption migration over existing rows — **always run in a maintenance window with a PITR branch**.

1. **Notify enterprise tenants 72 h in advance** via CSM channel: "Scheduled CV data key rotation on [date]. CV coaching will be briefly unavailable for ~15 min during migration window."
2. Open PITR branch on Supabase: **Dashboard → Backups → New Branch**. Record branch ID.
3. Disable CV globally: set feature flag `cv_enabled = false` in tenant_feature_flags (all tenants). This prevents new writes to `keypoints_enc` during migration.
4. Wait 60 seconds for in-flight Workers to drain.
5. Rotate Vault key: execute in Supabase SQL editor —
   ```sql
   SELECT pgsodium.rotate_key('KEYPOINTS_ENC_KEY');
   ```
   Record new `key_id` returned.
6. Run re-encryption migration. Execute in a transaction:
   ```sql
   BEGIN;
   UPDATE cv_sessions
   SET keypoints_enc = pgsodium.crypto_aead_det_encrypt(
     pgsodium.crypto_aead_det_decrypt(keypoints_enc, additional, old_key_id),
     additional, new_key_id
   )
   WHERE keypoints_enc IS NOT NULL;
   -- Verify row count matches pre-migration count:
   SELECT COUNT(*) FROM cv_sessions WHERE keypoints_enc IS NOT NULL;
   COMMIT;
   ```
   If row counts do not match, **ROLLBACK immediately** and activate INCIDENT_RESPONSE R-21.
7. Spot-check: decrypt 5 random rows with new key; confirm `keypoints_enc IS NOT NULL` and no error.
8. Re-enable CV: set `cv_enabled = true` in tenant_feature_flags.
9. Emit `admin.encryption_key_rotated` DEC-030 event (§6.2).
10. Update `form-crypto-health` KV.
11. Discard PITR branch only after 48 h of stable operation.
12. File evidence: migration row count before/after, pgsodium key version log, to `compliance/evidence/enc/keypoints-enc-key-rotation-YYYY.md`.

---

### 5.4 CLOUDFLARE_API_TOKEN — 180 days

1. Log in to Cloudflare dashboard → **My Profile → API Tokens**.
2. Locate the `form-production` token. Record its current permissions (do **not** modify scope during rotation).
3. Click **Roll** (or create new token with identical permissions if Roll is unavailable for the token type).
4. Copy new token value (shown once).
5. Update 1Password Operations vault item `Cloudflare API Token - Production`.
6. Update any CI/CD secrets that reference the token (GitHub Actions secret `CLOUDFLARE_API_TOKEN` — update via GitHub repository settings).
7. Verify: `curl -X GET "https://api.cloudflare.com/client/v4/user/tokens/verify" -H "Authorization: Bearer <new-token>"` — expect `{"success": true}`.
8. Revoke old token via Cloudflare dashboard.
9. Emit `system.credential_rotated` DEC-030 event (§6.1).
10. Update `form-crypto-health` KV.
11. File evidence: Cloudflare API token audit log (dashboard → **Audit Log** filter by API token events) to `compliance/evidence/enc/cloudflare-api-token-rotation-YYYY-MM-DD.png`.

---

### 5.5 ANTHROPIC_API_KEY — 180 days

1. Log in to Anthropic console → **API Keys**.
2. Create a new key (same scope as current key). Copy value immediately.
3. Update Cloudflare Workers Secret: `wrangler secret put ANTHROPIC_API_KEY --env production`.
4. Redeploy affected Workers: `wrangler deploy --env production`.
5. Smoke-test: send a minimal Victor coaching prompt via the `/api/coaching/test` endpoint. Confirm 200 and a valid response.
6. Update 1Password Operations vault item `Anthropic API Key - Production`.
7. Delete old key from Anthropic console.
8. Emit `system.credential_rotated` DEC-030 event.
9. Update `form-crypto-health` KV.
10. File evidence: Anthropic console key creation/deletion timestamps to `compliance/evidence/enc/anthropic-key-rotation-YYYY-MM-DD.txt`.

---

### 5.6 ELEVENLABS_API_KEY — 180 days

1. Log in to ElevenLabs dashboard → **Profile → API Key**.
2. Generate new API key. Copy value.
3. Update Cloudflare Workers Secret: `wrangler secret put ELEVENLABS_API_KEY --env production`.
4. Redeploy affected Workers.
5. Smoke-test: hit the voice synthesis endpoint with a test phrase; confirm audio returned.
6. Update 1Password Operations vault.
7. Revoke old key from ElevenLabs dashboard.
8. Emit `system.credential_rotated` DEC-030 event.
9. Update `form-crypto-health` KV.
10. File evidence: ElevenLabs key event log to `compliance/evidence/enc/elevenlabs-key-rotation-YYYY-MM-DD.txt`.

---

### 5.7 STRIPE_LIVE_API_KEY — 180 days

> **Owner:** compliance-officer (Stripe credentials are financial-system access — treat as P0 credential per `CRYPTOGRAPHY_POLICY.md §5`).

1. Log in to Stripe dashboard → **Developers → API keys**.
2. Click **Roll key** on the live secret key. Stripe generates a new key and immediately rotates.
3. Copy new value (shown once).
4. Update 1Password Operations vault item `Stripe Live Secret Key`.
5. Update Cloudflare Workers Secret: `wrangler secret put STRIPE_SECRET_KEY --env production`.
6. Redeploy affected Workers.
7. Smoke-test: retrieve a known test subscription object via the Stripe API using the new key; confirm 200.
8. Stripe automatically invalidates the old key after the roll.
9. Emit `system.credential_rotated` DEC-030 event.
10. Update `form-crypto-health` KV.
11. File evidence: Stripe API key event log (dashboard → **Developers → Events** filter by `customer.*` or API key events) to `compliance/evidence/enc/stripe-key-rotation-YYYY-MM-DD.txt`.

---

### 5.8 SENTRY_DSN — 180 days

The Sentry DSN is public-facing (embedded in the mobile app build) and is low-risk by itself — it only allows error reporting ingest. Rotation is still required for SOC 2 credential hygiene.

1. Log in to Sentry dashboard → **Settings → Projects → form-mobile → Client Keys (DSN)**.
2. Create a new DSN key.
3. Update the `SENTRY_DSN` environment variable in the React Native build configuration (`.env.production` or EAS Secret: `eas secret:create --scope project --name SENTRY_DSN --value <new-dsn>`).
4. Trigger a new EAS build with the updated DSN.
5. Once the build is live, deactivate the old DSN key in Sentry.
6. Update 1Password Operations vault.
7. Emit `system.credential_rotated` DEC-030 event.
8. Update `form-crypto-health` KV.
9. File evidence: Sentry key creation/deactivation timestamps.

---

### 5.9 SUPABASE_ANON_KEY — 365 days

The `anon` key is public-facing — embedded in the mobile app and admin dashboard. Row-Level Security policies govern what the anon role can access. Rotation requires a coordinated app release.

1. Log in to Supabase dashboard → **Settings → API → Anon Key → Rotate**.
2. Copy new key.
3. Update the React Native build: `eas secret:create --scope project --name SUPABASE_ANON_KEY --value <new-key>` — or update `.env.production`.
4. Update the admin dashboard Cloudflare Pages environment variable.
5. Trigger an EAS build + submit to App Store and Play Store. The old key remains valid until the next step.
6. Once ≥ 90% of active users have migrated to the new build (check PostHog `app_version` cohort), rotate-disable the old key via Supabase dashboard.
7. Emit `system.credential_rotated` DEC-030 event.
8. Update `form-crypto-health` KV.
9. File evidence: Supabase key rotation log + PostHog version cohort screenshot showing >90% on new build.

> **Note:** Do not disable the old anon key before the 90% threshold — doing so will break the app for users on old builds. If a force-update is required, coordinate with product-manager for an App Store expedited review.

---

### 5.10 BACKUP_ENC_KEY — Annual [DC]

The backup encryption key protects Cloudflare R2 and Backblaze B2 WORM backups. Requires both security-engineer and compliance-officer to be present.

1. Both parties present — security-engineer and compliance-officer confirm identity via 1Password vault access.
2. Generate new 256-bit key: `openssl rand -hex 32`. **Do not save to any computer file system.**
3. Immediately store in compliance-officer 1Password vault (item: `Backup Encryption Key - Production`).
4. Write to offline encrypted USB (founder-held). Confirm the USB is unlocked and the key is successfully written before proceeding.
5. Update Cloudflare R2 bucket encryption key via Cloudflare dashboard (R2 → Settings → Encryption → Customer-Managed Key → Upload new key).
6. Update Backblaze B2 bucket server-side encryption key via B2 dashboard.
7. Emit `system.credential_rotated` DEC-030 event.
8. Update `form-crypto-health` KV.
9. File evidence: 1Password audit log entry (both vault entries) + USB write confirmation to `compliance/evidence/enc/backup-enc-key-rotation-YYYY.pdf`.
10. Shred any paper notes used during the ceremony.

---

### 5.11 API_KEY_HASH_SECRET — 180 days

This secret is used to hash tenant API keys at rest in `tenant_api_keys.key_hash`. Rotation requires re-hashing all active key rows — schedule a maintenance window.

1. Open maintenance window. Set `api_keys_maintenance = true` feature flag — new key creation and validation will return 503 during migration.
2. Generate new 256-bit secret: `openssl rand -base64 32`.
3. Update Cloudflare Workers Secret: `wrangler secret put API_KEY_HASH_SECRET --env production`.
4. Run re-hashing migration (execute as `form_system` role):
   ```sql
   BEGIN;
   UPDATE tenant_api_keys
   SET key_hash = hmac(raw_key_bytes, new_secret, 'sha256')
   WHERE revoked_at IS NULL;
   SELECT COUNT(*) FROM tenant_api_keys WHERE revoked_at IS NULL; -- verify row count
   COMMIT;
   ```
   If row counts diverge from pre-migration count, **ROLLBACK and activate R-21**.
5. Redeploy Workers: `wrangler deploy --env production`.
6. Disable maintenance flag.
7. Smoke-test: issue a test API request from a tenant sandbox key; confirm 200.
8. Update 1Password Operations vault.
9. Emit `admin.encryption_key_rotated` DEC-030 event (§6.2) with `kms_provider: 'cloudflare_workers_secrets'`.
10. Update `form-crypto-health` KV.
11. File evidence: pre/post migration row count, Workers Secrets deployment audit log.

---

### 5.12 TLS Certificate + Mobile Certificate Pin

**TLS certificates** are Cloudflare-managed and auto-renew. No manual action required. Better Stack probe S-012 monitors expiry (fires at T-30 days). If S-012 fires, check Cloudflare dashboard — it may indicate a Cloudflare service issue, not an action item.

**Mobile certificate pin** must be updated manually when the Cloudflare leaf certificate rotates. Procedure:

1. security-engineer detects upcoming rotation via Better Stack S-012 alert (T-30 days).
2. Compute SHA-256 of new Cloudflare leaf certificate public key:
   ```bash
   openssl x509 -in new-leaf.crt -pubkey -noout | openssl pkey -pubin -outform DER | openssl dgst -sha256 -binary | base64
   ```
3. Open a PR to add the new hash as a **backup pin** in the React Native TLS pinning configuration. Keep the existing primary pin active.
4. Release the app build with the backup pin. Include a note in the release description: "Updated TLS backup pin for planned certificate rotation on [date]."
5. Wait for ≥ 90% of users to upgrade (PostHog `app_version` cohort).
6. After certificate rotation completes and old certificate is retired, promote backup pin to primary in the next release.
7. No DEC-030 event required (TLS certs are platform-managed, not a FORM credential). File evidence: Better Stack S-012 alert history + App Store/Play Store submission timestamps to `compliance/evidence/enc/tls-cert-pin-rotation-YYYY-MM-DD.md`.

---

## 6. DEC-030 Event Sequences

Every rotation must emit at least one HMAC-chained DEC-030 event within 4 hours of completion. Missing events constitute a SOC 2 evidence gap.

### 6.1 `system.credential_rotated` — standard third-party credentials

Emitted for: `CLOUDFLARE_API_TOKEN`, `ANTHROPIC_API_KEY`, `ELEVENLABS_API_KEY`, `STRIPE_LIVE_API_KEY`, `SENTRY_DSN`, `SUPABASE_SERVICE_ROLE_JWT` (scheduled rotation).

```json
{
  "event_type": "system.credential_rotated",
  "severity": "STANDARD",
  "actor": "security-engineer",
  "credential_name": "ANTHROPIC_API_KEY",
  "rotation_trigger": "scheduled",
  "days_since_last_rotation": 182,
  "rotated_by": "security-engineer@form.coach",
  "hmac_chain_id": "<chain-id from previous event>",
  "timestamp": "2026-06-01T10:30:00Z"
}
```

### 6.2 `admin.encryption_key_rotated` — encryption keys

Emitted for: `KEYPOINTS_ENC_KEY`, `API_KEY_HASH_SECRET`, `SUPABASE_SERVICE_ROLE_JWT` (emergency rotation), `BACKUP_ENC_KEY`.

```json
{
  "event_type": "admin.encryption_key_rotated",
  "severity": "HIGH",
  "actor": "security-engineer",
  "key_id": "KEYPOINTS_ENC_KEY",
  "key_version_old": "v3",
  "key_version_new": "v4",
  "rotation_trigger": "scheduled",
  "rotated_by": "security-engineer@form.coach",
  "kms_provider": "supabase_vault",
  "hmac_chain_id": "<chain-id from previous event>",
  "timestamp": "2026-06-01T10:30:00Z"
}
```

### 6.3 `admin.signing_key_rotated` — HMAC audit chain key

Emitted only for `HMAC_AUDIT_CHAIN_KEY`. Must be followed by `admin.hmac_key_rotation_verified` within 24 hours (KEY-SLO-03). AL-KEY-04 fires if the verification event is absent.

```json
{
  "event_type": "admin.signing_key_rotated",
  "severity": "HIGH",
  "actor": "security-engineer",
  "key_id": "HMAC_AUDIT_CHAIN_KEY_v5",
  "rotation_trigger": "scheduled",
  "rotated_by": ["security-engineer@form.coach", "compliance-officer@form.coach"],
  "key_transition": {
    "old_key_id": "HMAC_AUDIT_CHAIN_KEY_v4",
    "old_key_retained": true,
    "old_key_purpose": "historical_verification_only"
  },
  "hmac_chain_id": "<chain-id signed with old key — last event under old key>",
  "timestamp": "2026-06-01T10:30:00Z"
}
```

---

## 7. Post-Rotation Verification

Run these checks after every rotation before closing the rotation record.

| Check | How | Pass criterion |
|---|---|---|
| Service health | `curl -s https://api.form.coach/health` | `200 OK`, `{"db":"ok","workers":"ok"}` |
| Auth health | Issue a test JWT via Supabase Auth test user; hit a protected endpoint | `200 OK`, correct user data returned |
| Audit log event present | Query `audit_log_events WHERE event_type = 'system.credential_rotated' AND created_at > now() - INTERVAL '1h'` | At least 1 row with correct `credential_name` |
| KV state updated | `wrangler kv:key get --namespace-id <ns> "key:rotation:<KEY_ID>" --env production` | `days_overdue = 0`, `next_rotation_due` is a future date |
| AL-KEY-02 resolved | Check PagerDuty — incident for this KEY_ID | Incident auto-resolved (resolver: `key-rotation-scheduler`) within 24 h |
| HMAC chain integrity (HMAC_AUDIT_CHAIN_KEY only) | Run chain verifier: `SELECT form.verify_audit_chain(last_100_events)` | No `admin.hmac_chain_break` events in next 24 h |

---

## 8. SOC 2 Evidence Collection

After each rotation, file evidence to the compliance artefact store. Evidence must be immutable once filed.

| KEY_ID | Artefact | Location |
|---|---|---|
| `SUPABASE_SERVICE_ROLE_JWT` | Supabase dashboard rotation confirmation screenshot + Cloudflare Workers Secrets deployment log | `compliance/evidence/enc/supabase-service-role-rotation-YYYY-MM-DD/` |
| `HMAC_AUDIT_CHAIN_KEY` | 1Password dual-vault audit log export (both vaults) + `admin.signing_key_rotated` + `admin.hmac_key_rotation_verified` event JSON | `compliance/evidence/enc/hmac-chain-key-rotation-YYYY/` |
| `KEYPOINTS_ENC_KEY` | pgsodium key version before/after, migration row count, maintenance window ticket | `compliance/evidence/enc/keypoints-enc-key-rotation-YYYY/` |
| `CLOUDFLARE_API_TOKEN` | Cloudflare Audit Log filtered by API token events, 1Password vault entry timestamp | `compliance/evidence/enc/cloudflare-api-token-rotation-YYYY-MM-DD/` |
| `ANTHROPIC_API_KEY` | Anthropic console key creation/deletion timestamps | `compliance/evidence/enc/anthropic-key-rotation-YYYY-MM-DD/` |
| `ELEVENLABS_API_KEY` | ElevenLabs key event log | `compliance/evidence/enc/elevenlabs-key-rotation-YYYY-MM-DD/` |
| `STRIPE_LIVE_API_KEY` | Stripe dashboard key roll confirmation + compliance-officer sign-off | `compliance/evidence/enc/stripe-key-rotation-YYYY-MM-DD/` |
| `SENTRY_DSN` | Sentry key creation/deactivation timestamps | `compliance/evidence/enc/sentry-dsn-rotation-YYYY-MM-DD/` |
| `SUPABASE_ANON_KEY` | Supabase key rotation log + PostHog version cohort screenshot | `compliance/evidence/enc/supabase-anon-key-rotation-YYYY/` |
| `BACKUP_ENC_KEY` | 1Password dual-vault audit log + USB write confirmation | `compliance/evidence/enc/backup-enc-key-rotation-YYYY/` |
| `API_KEY_HASH_SECRET` | Pre/post row counts + Workers Secrets deployment log | `compliance/evidence/enc/api-key-hash-secret-rotation-YYYY-MM-DD/` |
| All keys | DEC-030 event JSON (query from `audit_log_events`) | Part of each per-key directory above |

SOC 2 auditors will request artefacts from the above directories during the observation period. File artefacts on the day of rotation, not retroactively.

---

## 9. Escalation to Incident Response

| Condition | Action |
|---|---|
| Rotation fails mid-procedure (e.g., Workers deploy error, migration rollback) | Stop. Activate `INCIDENT_RESPONSE.md R-21`. Do not retry without opening an incident. |
| AL-KEY-02 fires and key is > 7 days overdue | R-21 auto-escalates to founder + compliance-officer. Follow R-21 2-hour rotation SLA. |
| CRYPTO-CHAIN-02 fires during rotation (rotation event without a preceding initiation event within 1 hour) | **Stop immediately.** Do not continue rotation. Activate R-05 + R-20 (suspected unauthorized access). |
| Key is suspected compromised before rotation | Activate R-03 (credential compromise) immediately. Rotation is remediation, not the root response. |
| `admin.hmac_key_rotation_verified` is not emitted within 24 h of `admin.signing_key_rotated` | AL-KEY-04 fires (P0). Activate R-21. Diagnose chain verifier worker health before re-running verification. |
| `KEYPOINTS_ENC_KEY` migration row count mismatch | Rollback transaction. Activate R-21 + R-11 (CV biometric data incident) in parallel. |

---

## 10. Annual Rotation Audit Procedure

Each January (Q1-Jan in compliance calendar, `SOC2_READINESS.md §15`):

1. Pull `form-crypto-health` KV state for all 11 keys. Confirm `days_overdue = 0` for every key.
2. Pull `audit_log_events` for the past 12 months, filtering on `event_type IN ('system.credential_rotated', 'admin.encryption_key_rotated', 'admin.signing_key_rotated')`. Verify every key has at least one rotation event matching its documented schedule.
3. Pull 1Password Operations vault audit log. Confirm every item's last-modified date aligns with a rotation event in step 2.
4. Review open questions (OQ-KEY-01, OQ-KEY-02) documented in `SOC2_READINESS.md §56`. Update status.
5. Confirm `tenant_api_keys` annual review (§5.11 step per `DATA_MODEL.md §26.6`): rotate any active tenant API key older than 365 days; notify affected tenant CSM.
6. File audit report to `compliance/evidence/annual-key-rotation-audit-YYYY/` with: KV state export, audit log query result, 1Password audit log, any exceptions and mitigating actions.
7. Bump `KEY_ROTATION.md` version and update document history if any procedures changed.

---

## 11. Two-Person Authorization

The following operations require two authorized individuals to be simultaneously present:

| Operation | Required parties | Authorization record |
|---|---|---|
| `HMAC_AUDIT_CHAIN_KEY` rotation | security-engineer + compliance-officer | Both names in `admin.signing_key_rotated` `rotated_by` array; 1Password dual-vault access timestamps as evidence |
| `BACKUP_ENC_KEY` rotation | security-engineer + compliance-officer | Both names in `system.credential_rotated` event; USB write confirmation signed by founder |
| `SUPABASE_SERVICE_ROLE_JWT` emergency rotation | founder + security-engineer | R-03 incident ticket + PAM `destructive` elevation log |

For dual-custody operations: if one party is unavailable (PTO, health, emergency), the founder may substitute for compliance-officer. The substitution must be noted in the DEC-030 event payload (`rotated_by` array) and in the compliance artefact.

---

## 12. Implementation Checklist

| # | Task | Owner | Priority | Target |
|---|---|---|---|---|
| 1 | Register `admin.hmac_key_rotation_verified` event type in `docs/AUDIT_LOG_SCHEMA.md` (required before HMAC chain key rotation can produce KEY-SLO-03-compliant evidence) | security-engineer + compliance-officer | **P0** | M7 |
| 2 | Create `form-crypto-health` KV namespace in Cloudflare account; seed with initial state for all 11 keys using actual last-rotated dates from 1Password vault audit logs | security-engineer | **P0** | M5 |
| 3 | Implement `key-rotation-scheduler` Worker that reads KV daily and emits AL-KEY-01 / AL-KEY-02 PagerDuty alerts | platform-engineer | **P0** | M5 |
| 4 | Implement `key-rotation-verifier` Edge Function: reads `admin.signing_key_rotated` events; emits `admin.hmac_key_rotation_verified` within 24 h | platform-engineer | **P1** | M7 |
| 5 | Execute first `KEYPOINTS_ENC_KEY` rotation using this runbook on staging; document elapsed time vs. maintenance window; update §5.3 if procedure deviates | security-engineer | **P1** | M6 |
| 6 | Run R-21 tabletop exercise: simulate AL-KEY-02 P0 for `SUPABASE_SERVICE_ROLE_JWT`; measure elapsed time vs. 2-hour rotation SLA | security-engineer | **P1** | M8 |
| 7 | Add `API_KEY_HASH_SECRET` to 1Password Operations vault if not already present (may be in Cloudflare Workers Secrets only) | security-engineer | **P0** | M5 |
| 8 | Confirm `BACKUP_ENC_KEY` offline USB copy exists and is readable by founder; document USB location in compliance-officer 1Password vault | compliance-officer | **P0** | M5 |

---

## 13. SOC 2 Control Mapping

| SOC 2 Control | How this runbook satisfies it |
|---|---|
| **CC5.2** — Cryptographic controls are defined and enforced | Key inventory (§2) documents all production keys and their rotation periods |
| **CC5.3** — Access to cryptographic material is restricted and monitored | KV state tracking (§3) + AL-KEY-* alerts (OBSERVABILITY §30.4) provide continuous monitoring; 1Password vault audit logs provide access restriction evidence |
| **CC6.4** — Access is removed when credentials change | `SUPABASE_SERVICE_ROLE_JWT` rotation invalidates old token; `CLOUDFLARE_API_TOKEN` old token is explicitly revoked; `API_KEY_HASH_SECRET` re-hash migration ensures old hash secret is replaced across all active rows |
| **CC6.7** — Data at rest and in transit is protected through defined key lifecycle | `KEYPOINTS_ENC_KEY` annual rotation (§5.3); `SUPABASE_SERVICE_ROLE_JWT` 90-day rotation (§5.1); `BACKUP_ENC_KEY` annual rotation (§5.10) |
| **CC6.8** — Dual-custody controls prevent unauthorized key rotation | Two-person authorization table (§11) covers `HMAC_AUDIT_CHAIN_KEY` and `BACKUP_ENC_KEY`; dual-vault 1Password architecture enforces this at the custody layer |
| **C1.1** — Health data confidentiality maintained | `KEYPOINTS_ENC_KEY` and `HMAC_AUDIT_CHAIN_KEY` rotation procedures protect Art. 9 data at rest and the audit chain for health data events |

---

## 14. Document History

| Version | Date | Author | Changes |
|---|---|---|---|
| v1.0 | 2026-06-06 | security-engineer + enterprise-architect | Initial document. Covers 11-key inventory, per-key step-by-step runbooks, DEC-030 event sequences, post-rotation verification, annual audit procedure, dual-custody rules. Resolves CC6.7 evidence gap in `DATA_MODEL.md §7047`. Closes `INCIDENT_RESPONSE.md R-21` implementation checklist item 4 dependency. |

---

**v1.0 · June 2026 · security-engineer + compliance-officer + enterprise-architect**
