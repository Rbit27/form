# Auditor Fieldwork Protocol — `incident.severity_changed` Reason Verification

**Document path:** `compliance/fieldwork/incident-reason-verification.md`
**Version:** v1.0 · 2026-06-18
**Owner:** compliance-officer
**SOC 2 TSC:** P3.2 (Privacy practices followed), CC2.2 (External communications)
**Source spec:** `docs/INCIDENT_RESPONSE.md §17.3.6`
**Onboarding package:** Included in `compliance/evidence/auditor-onboarding/README.md`

---

## 1. Purpose

This protocol is for SOC 2 auditors verifying that `incident.severity_changed` DEC-030 chain records contain no plaintext personally identifiable information (PII) or Art. 9 special-category data.

FORM's incident response system (DEC-044) stores the Incident Commander's reason for each severity change as a SHA-256 hash in the DEC-030 chain — not as plaintext. The verbatim reason text is held exclusively in the corresponding Linear ticket. This design prevents a 7-year-retained chain record from inadvertently capturing names, email addresses, or health data categories under time pressure.

This is the same pattern applied to `system.load_test_gate_bypassed` via `bypass_reason_hash` (DEC-044, `docs/INCIDENT_RESPONSE.md §40`).

---

## 2. Background: Why `reason_plaintext` Is Not in the Chain

### 2.1 Retention exposure

`incident.severity_changed` records are retained for 7 years (HIGH severity class). Under GDPR Art. 17 and FORM's data-minimisation principle, no `user_id`, employee name, email, or Art. 9 health category may appear in a long-lived audit chain record.

### 2.2 Time-pressure PII risk

During a live P0 incident, an Incident Commander writing a severity-change reason is under significant time pressure. A runtime blocklist (§17.3.4) emits a `incident.pii_risk_detected` advisory event (MEDIUM, 7yr) if the reason matches five pattern categories (email, national_id, uuid_dump, art9_term), but the blocklist is advisory — it does not block incident workflow. The hash-plus-plaintext-in-Linear design eliminates the 7-year exposure even if a blocklist miss occurs.

### 2.3 Design reference

Decision: DEC-044 (`docs/INCIDENT_RESPONSE.md §17`, v2.4, 2026-06-14). Pattern: `reason_hash = SHA-256(incident_id + '|' + reason_plaintext + '|' + INCIDENT_REASON_HASH_SALT)`.

---

## 3. Chain Record Structure

The `incident.severity_changed` Zod schema (authoritative: `docs/AUDIT_LOG_SCHEMA.md §6.Incident-Lifecycle`):

```typescript
z.object({
  incident_id:        z.string().regex(/^INC-\d{8}-[0-9a-f]{6}$/),
  previous_severity:  z.enum(["P0","P1","P2","P3"]),
  new_severity:       z.enum(["P0","P1","P2","P3"]),
  reason_hash:        z.string().length(64),  // SHA-256 hex, 64 chars
  changed_by:         z.string().uuid(),      // IC user UUID (form_ops role)
})
```

**`reason_hash` formula:**

```
reason_hash = SHA-256( incident_id + '|' + reason_plaintext + '|' + INCIDENT_REASON_HASH_SALT )
```

Where `INCIDENT_REASON_HASH_SALT` is a per-environment Cloudflare Workers Secret with annual rotation (catalogued in `docs/CRYPTOGRAPHY_POLICY.md §5`).

---

## 4. Five-Step Retrieval and Verification Procedure

### Step 1 — Locate the chain record

Using the `compliance_reviewer` database role (read-only), retrieve the target `incident.severity_changed` event:

```sql
SELECT
  e.id            AS event_id,
  e.emitted_at,
  e.actor,
  e.action,
  e.payload->>'incident_id'       AS incident_id,
  e.payload->>'reason_hash'       AS reason_hash,
  e.payload->>'previous_severity' AS previous_severity,
  e.payload->>'new_severity'      AS new_severity,
  e.hmac_value,
  e.hmac_prev
FROM audit_log_events e
WHERE e.action = 'incident.severity_changed'
  AND e.payload->>'incident_id' = :incident_id
ORDER BY e.emitted_at ASC;
```

Note `reason_hash` (64-char hex) for each returned row.

### Step 2 — Locate the Linear ticket URL

Retrieve the `incident.linear_ticket_linked` record for the same `incident_id`:

```sql
SELECT
  e.payload->>'linear_ticket_url' AS linear_ticket_url,
  e.payload->>'linked_by'         AS linked_by,
  e.emitted_at
FROM audit_log_events e
WHERE e.action = 'incident.linear_ticket_linked'
  AND e.payload->>'incident_id' = :incident_id
ORDER BY e.emitted_at ASC
LIMIT 1;
```

If `linked_by = 'manual_ic'`, the IC used the amendment endpoint (`POST /internal/v1/incident/link-ticket`) after the initial `incident.opened` emission — this is expected when the Linear API was unavailable at T+0.

### Step 3 — Access the Linear ticket

Request read-only guest credentials from the compliance-officer. These are granted for the duration of the SOC 2 fieldwork window.

In the Linear ticket:
- The description contains the IC's severity-change reason (labelled "Severity change reason" or in the incident log comment thread).
- The Linear ticket title begins with `[{incident_id}]`, confirming the association.

### Step 4 — Read `reason_plaintext`

Locate the reason text in the Linear ticket description or incident log comment thread. This is `reason_plaintext` for the verification step. The text should not contain user names, email addresses, or Art. 9 health categories (the runtime blocklist warns on these, but the hash design ensures the chain is clean regardless).

### Step 5 — Independent hash verification

Compute the expected hash and compare to the chain record:

```
expected_hash = SHA-256( incident_id + '|' + reason_plaintext + '|' + INCIDENT_REASON_HASH_SALT )
```

**Obtaining `INCIDENT_REASON_HASH_SALT`:** Request the salt from the compliance-officer during fieldwork. The salt is stored in the 1Password Operations vault (`form-cloudflare-workers-secrets` item). Annual rotation dates and prior-version values are accessible via the vault's version history — use the version active at the time of the chain event's `emitted_at` timestamp.

**Expected outcome:** `expected_hash` must match the `reason_hash` stored in the `audit_log_events` row exactly (case-insensitive hex comparison).

**On mismatch:** Escalate to the compliance-officer. Possible causes: (a) wrong salt version; (b) whitespace difference in `reason_plaintext` (trim leading/trailing whitespace and retry); (c) salt rotation occurred between event emission and this verification (check vault version history for the exact rotation timestamp).

---

## 5. Privacy Constraints on Fieldwork Artefacts

When filing observations from this procedure:

- Do **not** include `reason_plaintext` in any written SOC 2 observation. Reference the Linear ticket URL only.
- Do **not** include the `INCIDENT_REASON_HASH_SALT` value in any filing — note only that "salt verified per 1Password Operations vault."
- `changed_by` (user UUID) and `incident_id` may appear in artefact files.
- The `compliance_reviewer` role does not expose `user_id` values linked to health data — the incident chain contains only `incident_id` and DEC-030 metadata.

---

## 6. SOC 2 Criteria Mapping

| Criterion | How this protocol supports it |
|---|---|
| **P3.2** — Privacy practices followed | Demonstrates that `incident.severity_changed` chain records contain no plaintext PII; `reason_hash` + Linear ticket separation is the operational privacy control |
| **CC2.2** — External communications | Confirms that stakeholder communications during incidents reference only `incident_id`; no personal data leaks through the DEC-030 record |

---

## 7. Cross-References

| Document | Relevance |
|---|---|
| `docs/INCIDENT_RESPONSE.md §17.3.6` | Source specification for this protocol |
| `docs/INCIDENT_RESPONSE.md §17.3.3` | `reason_hash` computation formula and Salt rotation policy |
| `docs/INCIDENT_RESPONSE.md §17.3.4` | Runtime blocklist — `incident.pii_risk_detected` advisory events |
| `docs/AUDIT_LOG_SCHEMA.md §6.Incident-Lifecycle` | `incident.severity_changed` and `incident.linear_ticket_linked` Zod schemas |
| `docs/CRYPTOGRAPHY_POLICY.md §5` | `INCIDENT_REASON_HASH_SALT` key inventory entry (annual rotation) |
| `docs/INCIDENT_RESPONSE.md §40` | DEC-044 `bypass_reason_hash` pattern — the same design used in `system.load_test_gate_bypassed` |
| `compliance/evidence/auditor-onboarding/README.md` | Auditor onboarding package index |

---

*v1.0 (2026-06-18): Initial authoring per `docs/INCIDENT_RESPONSE.md §17.7 checklist item 6` (P1/M5). Closes the documentation obligation from `docs/INCIDENT_RESPONSE.md §17.3.6`. Owner: compliance-officer.*
