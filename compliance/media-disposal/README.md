# FORM · Media Disposal Evidence — C1-E-008

> **SOC 2 evidence artifact:** C1.2 (Dispose of Confidential Information) · CC6.3 (Termination of Access) · CC6.5 (Removal of Access to Protected Information Assets)
> **Owner:** compliance-officer + security-engineer
> **Policy source:** `compliance/c1/device-disposal-policy.md` (POL-013 IN_FORCE)
> **Full procedure:** `docs/SOC2_READINESS.md §40`

---

## Purpose

This directory holds the **device disposal log** (C1-E-008) — the primary SOC 2 auditor evidence for C1.2. Every time a FORM-managed or BYOD device is sanitized or destroyed, a row is appended to `device-disposal-log.csv`.

The log is **append-only**. No existing row may be modified. Corrections are made by appending a new row with `reason = 'correction'` and referencing the original `disposal_id` in the `notes` field.

---

## Artifact Index

| Artifact ID | File | Description | Status |
|---|---|---|---|
| **C1-E-008** | `device-disposal-log.csv` | Append-only log of all device sanitization and destruction events | 🟢 Active — seed row on first disposal event |
| **C1-E-008-cloud** | `cloud-disposal/YYYY-MM-{instance}.md` | Cloud instance decommission records (Class E devices) | 🔴 Create on first cloud instance decommission |
| **C1-E-008-backup** | `backups/YYYY-MM-key-rotation-log.md` | Cryptographic Erase log for B2 WORM cold storage | 🔴 Create on first backup key rotation for disposal purposes |

---

## Log Schema (C1-E-008)

`device-disposal-log.csv` columns (per `docs/SOC2_READINESS.md §40.9`):

| Column | Type | Description |
|---|---|---|
| `disposal_id` | TEXT | UUID v4 — unique per event |
| `date` | DATE | ISO 8601 date of sanitization or destruction |
| `device_class` | TEXT | A (laptop), B (mobile), C (removable media), D (BYOD), E (cloud instance) |
| `make_model` | TEXT | e.g., `Apple MacBook Pro 14-inch M3 Pro` |
| `serial_number` | TEXT | Device serial; `UNKNOWN` if BYOD with no physical access |
| `asset_owner` | TEXT | Full name of individual; `FORM` if company-issued |
| `reason` | TEXT | `offboarding`, `end_of_life`, `theft_or_loss`, `tenant_contract_end`, `correction` |
| `sanitization_method` | TEXT | NIST method (e.g., `Cryptographic Erase — Apple Silicon`, `DoD 5220.22-M 3-pass`, `Physical destruction`) |
| `certificate_ref` | TEXT | Shredder certificate filename or `N/A` for software methods |
| `performed_by` | TEXT | Full name of security-engineer executing the procedure |
| `witness` | TEXT | compliance-officer confirming; `SOLO` during solo-founder phase |
| `notes` | TEXT | Exceptions, BYOD attestation reference, compensating controls |

---

## Retention

The `device-disposal-log.csv` is retained **7 years** from the date of the last appended row, per `docs/SOC2_READINESS.md §40.9` and DEC-030 event `asset.device_disposal_logged` (STANDARD, 7yr).

---

## DEC-030 Event

Each row append triggers a `asset.device_disposal_logged` DEC-030 HMAC-chained event with payload:

```json
{
  "event_type": "asset.device_disposal_logged",
  "disposal_id": "<uuid>",
  "device_class": "A|B|C|D|E",
  "reason": "offboarding|end_of_life|theft_or_loss|tenant_contract_end",
  "performed_by": "<security-engineer-id>",
  "sanitization_method": "<NIST-method-string>"
}
```

---

*Owner: compliance-officer · Created: 2026-06-12 · Next review: annual (Q4) or on fleet change*
