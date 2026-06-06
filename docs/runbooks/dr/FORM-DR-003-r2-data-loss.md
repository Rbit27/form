# FORM-DR-003 · Cloudflare R2 Primary Bucket Data Loss or Corruption

> **Runbook ID:** FORM-DR-003
> **Severity:** P1
> **Owner:** devops-lead · reviewed by: compliance-officer · enterprise-architect
> **Last reviewed:** 2026-06-06
> **SOC 2 TSC:** A1.2, A1.3
> **Cross-refs:** docs/SOC2_READINESS.md §53; docs/INCIDENT_RESPONSE.md §IR-3; docs/OBSERVABILITY.md §OBS-4

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | FORM-DR-003 |
| Severity | P1 |
| Description | Primary R2 bucket (`form-production`) experiences data loss or object corruption. User-uploaded assets, CV session thumbnails, and coaching media are affected. Cold backup bucket (`form-cold-backups`) is a separate bucket and is unaffected. |
| Affected tiers | All tiers for media access; Enterprise prioritised for restore |
| Estimated RTO — partial restore (< 20% objects) | 1–2 h |
| Estimated RTO — full bucket restore | 4–6 h |

Core authentication, workout logging, and AI coaching remain fully operational during this incident. User-facing media is degraded, not absent.

---

## 2. RTO/RPO Targets

| Tier | RTO | RPO | SLA Enforcement |
|---|---|---|---|
| Enterprise | ≤ 2 h | ≤ 15 min | Contractual; enterprise tenant objects restored first |
| Growth | ≤ 4 h | ≤ 1 h | Operational SLA |
| Consumer | ≤ 8 h | ≤ 4 h | Best effort |

RPO note: cold backup is nightly; objects created since the last backup cannot be restored from cold backup. PITR covers the database record of which keys exist but not the objects themselves.

---

## 3. Detection Signals

- **Sentry alert FORM-DR-R2-001** — R2 object reads returning 404 or 500 for known-good keys at a rate > 1% sustained over 5 min
- **Cloudflare R2 dashboard** — object count metric showing unexpected drop or error rate spike
- **User reports** — broken media, failed thumbnail loads, missing coaching assets
- **Better Stack / Grafana Cloud** — `r2_read_success_rate` below 99% for > 5 min
- **https://www.cloudflarestatus.com** — R2 component incident (if platform-caused)

Escalate to devops-lead immediately. IC: platform-engineer for P1 (devops-lead as backup).

---

## 4. Pre-Flight Checklist

- [ ] Confirm alert FORM-DR-R2-001 is genuine: cross-check with Cloudflare R2 dashboard object count
- [ ] Distinguish R2 platform issue from FORM-side misconfiguration: check `CF_R2_ACCOUNT_ID` and bucket name in Workers Secret list
- [ ] Check https://www.cloudflarestatus.com for an R2-specific incident
- [ ] Open Linear incident ticket — P1, tag `r2-data-loss`, note whether Cloudflare platform is involved
- [ ] Confirm devops-lead is reachable as backup IC
- [ ] Confirm `form-cold-backups` bucket is accessible and unaffected (spot-check one known backup manifest)
- [ ] Pull list of enterprise tenant IDs from database for prioritised restore ordering

```bash
# Confirm cold-backups bucket is accessible
wrangler r2 object get \
  "form-cold-backups/manifests/$(date +%Y/%m)/backup-manifest-$(date +%Y-%m-%d).json" \
  --file /tmp/manifest-check.json && echo "Cold backup accessible"
```

---

## 5. Step-by-Step Recovery Procedure

### Step 1 — Scope the damage (devops-lead; T+0 to T+20 min)

```bash
# Export R2 object list from form-production bucket
wrangler r2 object list form-production --json > /tmp/r2-current-objects.json

# Query database for all expected R2 keys
psql "${DATABASE_URL}" -c \
  "SELECT thumbnail_r2_key FROM cv_sessions WHERE thumbnail_r2_key IS NOT NULL
   UNION ALL
   SELECT media_r2_key FROM coaching_turns WHERE media_r2_key IS NOT NULL" \
  --csv -o /tmp/expected-keys.csv

# Compute missing and present keys
python3 - <<'EOF'
import json, csv
with open('/tmp/r2-current-objects.json') as f:
    present = {o['key'] for o in json.load(f).get('objects', [])}
with open('/tmp/expected-keys.csv') as f:
    expected = {row[0] for row in csv.reader(f) if row[0]}
missing = expected - present
print(f"Expected: {len(expected)}, Present: {len(present)}, Missing: {len(missing)}")
print(f"Affected %: {len(missing)/max(len(expected),1)*100:.1f}%")
with open('/tmp/missing-keys.txt', 'w') as f:
    f.write('\n'.join(sorted(missing)))
EOF
```

**Decision gate:**
- < 20% objects missing or corrupted → partial restore (Step 2).
- ≥ 20% objects missing or corrupted → full bucket restore (Step 3).

---

### Step 2 — Partial restore: targeted recovery from cold backup (< 20% of objects)

**2a. Retrieve cold backup manifest**

```bash
MANIFEST_DATE=$(date +%Y-%m-%d)
wrangler r2 object get \
  "form-cold-backups/manifests/$(date +%Y/%m)/backup-manifest-${MANIFEST_DATE}.json" \
  --file /tmp/restore-manifest.json
```

**2b. Prioritise enterprise tenant objects**

```bash
# Get enterprise tenant asset keys
psql "${DATABASE_URL}" -c \
  "SELECT thumbnail_r2_key FROM cv_sessions cs
   JOIN tenants t ON cs.tenant_id = t.id
   WHERE t.tier = 'enterprise' AND cs.thumbnail_r2_key = ANY(
     SELECT unnest(string_to_array(pg_read_file('/tmp/missing-keys.txt'), E'\n'))
   )" --csv -o /tmp/enterprise-missing-keys.csv
```

**2c. Restore missing objects by SHA-256 checksum match**

```bash
# For each missing key, find its record in the manifest and restore
while IFS= read -r key; do
  CHECKSUM=$(jq -r --arg k "$key" '.objects[] | select(.key == $k) | .sha256' /tmp/restore-manifest.json)
  if [ -n "$CHECKSUM" ]; then
    wrangler r2 object get "form-cold-backups/objects/${key}" --file "/tmp/restore/${key##*/}"
    ACTUAL=$(sha256sum "/tmp/restore/${key##*/}" | awk '{print $1}')
    if [ "$ACTUAL" = "$CHECKSUM" ]; then
      wrangler r2 object put "form-production/${key}" --file "/tmp/restore/${key##*/}"
      echo "Restored: ${key}"
    else
      echo "CHECKSUM MISMATCH for ${key} — skip and log"
    fi
  fi
done < /tmp/missing-keys.txt
```

**2d. Verify re-upload checksums**

```bash
# Spot-check 10 restored objects
head -10 /tmp/missing-keys.txt | while IFS= read -r key; do
  wrangler r2 object head "form-production/${key}" && echo "OK: $key" || echo "MISSING: $key"
done
```

---

### Step 3 — Full bucket restore from cold backup (≥ 20% of objects affected)

**3a. Retrieve manifest and verify**

```bash
wrangler r2 object get \
  "form-cold-backups/manifests/$(date +%Y/%m)/backup-manifest-$(date +%Y-%m-%d).json" \
  --file /tmp/full-restore-manifest.json

# Manifest structure: {"backup_date":"...","backup_file":"...","sha256_checksum":"...","file_size_bytes":...}
cat /tmp/full-restore-manifest.json
```

**3b. Restore enterprise tenant objects first**

Execute Step 2b–2d scoped to enterprise tenant keys before running bulk restore. This satisfies Enterprise RTO ≤ 2 h for the highest-priority objects.

**3c. Bulk restore all remaining objects**

```bash
# Download and decrypt full backup archive
BACKUP_FILE=$(jq -r '.backup_file' /tmp/full-restore-manifest.json)
wrangler r2 object get "form-cold-backups/${BACKUP_FILE}" --file /tmp/full-backup.tar.enc

openssl enc -d -aes-256-gcm -in /tmp/full-backup.tar.enc -out /tmp/full-backup.tar \
  -pass env:COLD_BACKUP_ENC_KEY -pbkdf2

# Verify archive checksum
EXPECTED=$(jq -r '.sha256_checksum' /tmp/full-restore-manifest.json)
ACTUAL=$(sha256sum /tmp/full-backup.tar | awk '{print $1}')
[ "$EXPECTED" = "$ACTUAL" ] && echo "Checksum OK" || { echo "CHECKSUM MISMATCH — HALT"; exit 1; }

# Extract and re-upload to form-production
tar -xf /tmp/full-backup.tar -C /tmp/restore-objects/
for file in /tmp/restore-objects/*; do
  KEY=$(basename "$file")
  wrangler r2 object put "form-production/${KEY}" --file "$file"
done
```

**3d. Confirm application serving is restored**

```bash
# Sample 10 objects from each tier using database-stored keys
psql "${DATABASE_URL}" -c \
  "SELECT thumbnail_r2_key FROM cv_sessions
   WHERE thumbnail_r2_key IS NOT NULL ORDER BY random() LIMIT 10" \
  --tuples-only | while IFS= read -r key; do
    wrangler r2 object head "form-production/${key}" && echo "OK: $key" || echo "FAIL: $key"
  done
```

---

## 6. Rollback / Abort Procedure

- If restored objects fail SHA-256 checksum verification: do not upload; flag key in evidence log; continue with remaining objects.
- If cold backup manifest is missing for today's date: fall back to yesterday's manifest and note the additional RPO gap in the Linear ticket.
- If R2 platform issue is still active (Cloudflare incident ongoing): pause restore attempts; objects will arrive corrupted regardless. Wait for Cloudflare R2 to stabilise before re-uploading.
- If restore operation is causing excessive R2 API costs (monitor via Cloudflare dashboard): throttle upload rate with `sleep 0.1` between objects; notify devops-lead.

---

## 7. Evidence Capture Instructions

Upload all evidence to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`:

- [ ] R2 dashboard screenshot: object count before and after restore
- [ ] Terminal output: scoping script results (expected count, missing count, affected percentage)
- [ ] Manifest JSON used for restore (copy to incident folder)
- [ ] SHA-256 checksum verification output (all `OK` lines and any `MISMATCH` lines)
- [ ] Terminal output: `wrangler r2 object head` spot-check for 10 objects per tier
- [ ] Screenshot: https://www.cloudflarestatus.com if platform incident was involved
- [ ] DEC-030 `system.dr_drill_completed` event if this was a drill:

```sql
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.dr_drill_completed',
  'INFO',
  'system',
  jsonb_build_object(
    'runbook',            'FORM-DR-003',
    'drill_started_at',   '<ISO8601>',
    'drill_completed_at', now()::text,
    'objects_restored',   '<COUNT>',
    'restore_path',       'partial|full'
  ),
  now()
);
```

Privacy floor: no plaintext user data in captured outputs. Key names (R2 keys) are opaque UUIDs by design. Log only `actor_uuid` and `client_ip_hash` in any audit event payloads.

---

## 8. Post-Incident Actions

### Customer Communication

| Tier | Channel | Timing | Frequency |
|---|---|---|---|
| Enterprise | Email | Within 15 min of P1 declaration | Every 60 min until restored |
| Growth | Email | Within 30 min of P1 declaration | At resolution |
| Consumer | Status page | Within 60 min of P1 declaration | At resolution |

Note: confirm in all communications that no data was permanently lost (cold backup coverage) and that core features remain functional.

### SLA Breach Assessment

- If Enterprise tier media restore exceeds 2 h: compliance-officer reviews MSA applicability.
- Document restore timeline and per-tenant impact in Linear ticket.

### Post-Mortem

- Trigger: always for data-loss events
- Timeline: within 48 h of resolution
- Owner: devops-lead
- Include: root cause (FORM misconfiguration vs. R2 platform), scope of loss, RPO gap for objects created after last backup, preventive actions (e.g., object versioning, cross-bucket replication)

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| A1.2 | Cold backup restore procedure tested via DR drills; SHA-256 verification required at every restore step |
| A1.3 | Enterprise tenant objects prioritised; checksum verification prevents corrupted-data propagation; audit events emitted per DEC-030 |

---

*v1.0 · 2026-06-06 · devops-lead + compliance-officer*
