# FORM · HMAC Chain Verification Algorithm v1.0

**Artefact ID:** HMAC-VERIFY-ALGO-001  
**Version:** 1.0 (2026-06-20)  
**Classification:** Enterprise Security — NDA-gated (Tier 2)  
**Canonical source:** `compliance/docs/hmac-chain-verification-algorithm.md`  
**Normative specification:** `docs/OBSERVABILITY.md §50` (DEC-071, 2026-06-19)  
**SOC 2 mapping:** CC1.1 (communication of HMAC chain objectives), C1.1 (confidentiality — demonstrates what data is in the chain and what the privacy floor excludes)  
**Audience:** Enterprise security teams, SIEM integration engineers, SOC 2 auditors  
**Retention:** Permanent (each version superseded by next; archive old versions with date stamp)  
**Owner:** security-engineer + devops-lead + compliance-officer

---

## 1. Purpose

This document specifies the algorithm by which enterprise tenants can independently verify the integrity of their FORM audit log export. FORM's DEC-030 architecture produces a cryptographic chain across `audit_log_events` rows: each event's HMAC-SHA256 signature commits to the preceding event's signature and payload, making silent deletion or modification of any event detectable without access to FORM's internal systems.

This specification enables a security-literate engineer to implement chain verification in under two hours using the pseudocode and test vector below. FORM does not need to provide a library — the algorithm is the artefact.

**Scope:** Tenant-facing verification only. This document does not describe chain *construction* (FORM-internal, governed by DEC-030 and `docs/AUDIT_LOG_SCHEMA.md`) or FORM's internal chain-break monitoring (AL-SIEM-05 P0 alert — `docs/OBSERVABILITY.md §27.7`).

---

## 2. DEC-030 Chain Structure

Each event row in `audit_log_events` carries the following fields relevant to verification:

| Field | Type | Description |
|---|---|---|
| `event_id` | UUID v4 | Unique event identifier |
| `event_type` | TEXT | Event category (e.g., `sso.login_succeeded`) |
| `payload` | JSONB | Event-specific data — privacy floor: no raw PII, no `user_email`, no health data |
| `created_at` | TIMESTAMPTZ | Event timestamp (UTC), ISO 8601 with `Z` suffix |
| `hmac_signature` | TEXT | 64-char lowercase hex — HMAC-SHA256 of this event |
| `previous_event_id` | UUID v4 or NULL | `event_id` of the immediately preceding event in this tenant's chain; NULL for the first event in the chain |

### 2.1 Signature Input Format

The `hmac_signature` of event *N* is:

```
HMAC-SHA256(
  key   = tenant_hmac_verify_key,
  input = "{event_id}:{event_type}:{canonical_payload}:{created_at_unix_ms}:{previous_signature}"
)
```

Where:

- **`canonical_payload`** = `json.dumps(payload, separators=(",", ":"), sort_keys=True)` — keys sorted lexicographically, no whitespace, no trailing comma
- **`created_at_unix_ms`** = UTC epoch milliseconds as an integer (no decimal point, no trailing zeros)
- **`previous_signature`** = `hmac_signature` of the event at `previous_event_id`; for the chain's first event (where `previous_event_id IS NULL`), `previous_signature` = `"0000000000000000000000000000000000000000000000000000000000000000"` (64 zero characters — the chain-start sentinel)

### 2.2 Per-Tenant HMAC Verification Key

The verification key is **not** FORM's internal master secret. FORM derives a per-tenant key using:

```
tenant_hmac_verify_key = HKDF-SHA256(
  IKM  = FORM_AUDIT_HMAC_SECRET,  # internal master secret, never published
  info = tenant_id,               # UTF-8 encoded
  salt = HMAC_KDF_SALT,           # registered in docs/CRYPTOGRAPHY_POLICY.md §5
  len  = 32                       # bytes → 64 hex chars
)
```

Each tenant can verify only their own chain. FORM does not expose the master secret to any tenant.

**Receiving your key:** Admin Dashboard → Security → Audit Export → *HMAC Verification Key*. The key is displayed once with a copy-to-clipboard button. FORM does not retain a copy after display. Store it in your SIEM secrets vault (Splunk Vault, Azure Key Vault, AWS Secrets Manager, HashiCorp Vault).

---

## 3. Verification Pseudocode (Python 3.10+)

```python
import hmac as hmac_lib
import hashlib
import json
from datetime import timezone, datetime


SENTINEL = "0" * 64  # chain-start sentinel for the first event in a tenant's chain


def canonical_payload(payload: dict) -> str:
    """Deterministic JSON serialisation: sorted keys, no whitespace."""
    return json.dumps(payload, separators=(",", ":"), sort_keys=True)


def created_at_to_ms(created_at: str) -> int:
    """Convert ISO 8601 UTC timestamp (with Z suffix) to epoch milliseconds."""
    dt = datetime.fromisoformat(created_at.replace("Z", "+00:00"))
    return int(dt.astimezone(timezone.utc).timestamp() * 1000)


def compute_signature(
    event_id: str,
    event_type: str,
    payload: dict,
    created_at_ms: int,
    prev_signature: str,
    hmac_secret: bytes,
) -> str:
    message = (
        f"{event_id}:{event_type}:{canonical_payload(payload)}"
        f":{created_at_ms}:{prev_signature}"
    )
    return hmac_lib.new(hmac_secret, message.encode("utf-8"), hashlib.sha256).hexdigest()


def verify_chain(events: list[dict], hmac_secret: bytes) -> dict:
    """
    Verify the HMAC chain integrity of a tenant's audit log export.

    Parameters
    ----------
    events      : List of event dicts from GET /enterprise/v1/audit-events.
                  Must all belong to the same tenant_id.
                  Sorted ascending by created_at (API cursor guarantees this).
    hmac_secret : Per-tenant HMAC verification key (bytes) — from Admin Dashboard
                  → Security → Audit Export. NOT the FORM master secret.

    Returns
    -------
    dict with keys:
      valid          : bool — True if all signatures and chain pointers match
      errors         : list of dicts describing each violation
      events_checked : int — number of events evaluated
    """
    events_sorted = sorted(events, key=lambda e: e["created_at"])
    prev_signature = SENTINEL
    prev_event_id: str | None = None
    errors: list[dict] = []

    for event in events_sorted:
        created_at_ms = created_at_to_ms(event["created_at"])

        expected_sig = compute_signature(
            event["event_id"],
            event["event_type"],
            event["payload"],
            created_at_ms,
            prev_signature,
            hmac_secret,
        )

        if event["hmac_signature"] != expected_sig:
            errors.append(
                {
                    "event_id": event["event_id"],
                    "error": "SIGNATURE_MISMATCH",
                    "expected": expected_sig,
                    "got": event["hmac_signature"],
                }
            )

        if prev_event_id is not None and event.get("previous_event_id") != prev_event_id:
            errors.append(
                {
                    "event_id": event["event_id"],
                    "error": "CHAIN_POINTER_MISMATCH",
                    "expected_previous": prev_event_id,
                    "got_previous": event.get("previous_event_id"),
                }
            )

        prev_signature = event["hmac_signature"]
        prev_event_id = event["event_id"]

    return {
        "valid": len(errors) == 0,
        "errors": errors,
        "events_checked": len(events_sorted),
    }
```

---

## 4. Implementation Notes

### 4.1 Per-Tenant Isolation

Run `verify_chain()` per `tenant_id`. The chain is tenant-scoped — mixing events from different tenants in a single call will produce false `CHAIN_POINTER_MISMATCH` errors.

### 4.2 Cursor Continuity Across API Pages

`GET /enterprise/v1/audit-events` is paginated. Chain verification across page boundaries requires either:

- **Accumulate then verify:** Collect all pages in ascending `created_at` order, then call `verify_chain()` on the full list.
- **Incremental verify:** Call `verify_chain()` per page, carrying `prev_signature` and `prev_event_id` forward between pages. More memory-efficient for high-volume tenants.

### 4.3 SIEM Delivery Gaps Are Not Chain Breaks

If your SIEM experiences a downtime window, the pull API cursor advances past the gap period. Events received after a gap will have a `previous_event_id` pointing to an event not in the current batch — this is a SIEM-side delivery gap, not a tampering event.

To distinguish: FORM's internal AL-SIEM-05 (P0, auto-opens incident R-01) fires within 5 minutes of any chain break on FORM's side. If AL-SIEM-05 did not fire during the gap window, the chain is intact and the gap is purely a SIEM delivery issue.

For incremental verification across gaps, request the gap period separately via the pull API (by timestamp range), verify that sub-chain, then resume normal incremental verification from the last successfully verified event.

### 4.4 HMAC Secret Rotation

FORM rotates `FORM_AUDIT_HMAC_SECRET` annually per `docs/CRYPTOGRAPHY_POLICY.md §5`. When a rotation occurs:

- Events signed with the old key retain their old signatures (immutable — the chain is append-only).
- Events signed after rotation use the new derived key.
- FORM will issue a new `tenant_hmac_verify_key` via the Admin Dashboard and notify tenant security contacts via the enterprise CSM at least 30 days before rotation.
- Verification of a cross-rotation export requires two keys: old key for pre-rotation events, new key for post-rotation events. FORM will document the exact rotation boundary (UTC timestamp of the first post-rotation event) in the Admin Dashboard alongside the new key.

The first scheduled rotation is estimated at M24 post-commercial launch. This document will be updated with cross-rotation handling instructions when that rotation is scheduled.

---

## 5. Test Vector

The following three-event synthetic chain uses the fixed test credentials below. **These are not production secrets.**

```
test_tenant  : tenant-test-001
hmac_secret  : test-secret-key-do-not-use  (bytes: UTF-8 encoding)
```

### Event 1 — chain start

```json
{
  "event_id":          "a1b2c3d4-0001-0001-0001-000000000001",
  "event_type":        "sso.login_succeeded",
  "payload":           {"actor_user_hash": "abc123", "tenant_id": "tenant-test-001"},
  "created_at":        "2026-06-19T10:00:00.000Z",
  "created_at_unix_ms": 1781863200000,
  "previous_event_id": null,
  "previous_signature": "0000000000000000000000000000000000000000000000000000000000000000",
  "hmac_input": "a1b2c3d4-0001-0001-0001-000000000001:sso.login_succeeded:{\"actor_user_hash\":\"abc123\",\"tenant_id\":\"tenant-test-001\"}:1781863200000:0000000000000000000000000000000000000000000000000000000000000000",
  "hmac_signature":    "6926f92c8a76a5986bf1cac4d2de8ff0056fc6cd49b9212d528a77c2af0d56f5"
}
```

### Event 2 — chained to Event 1

```json
{
  "event_id":          "a1b2c3d4-0001-0001-0001-000000000002",
  "event_type":        "auth.session_created",
  "payload":           {"session_type": "sso", "tenant_id": "tenant-test-001"},
  "created_at":        "2026-06-19T10:00:05.000Z",
  "created_at_unix_ms": 1781863205000,
  "previous_event_id": "a1b2c3d4-0001-0001-0001-000000000001",
  "previous_signature": "6926f92c8a76a5986bf1cac4d2de8ff0056fc6cd49b9212d528a77c2af0d56f5",
  "hmac_input": "a1b2c3d4-0001-0001-0001-000000000002:auth.session_created:{\"session_type\":\"sso\",\"tenant_id\":\"tenant-test-001\"}:1781863205000:6926f92c8a76a5986bf1cac4d2de8ff0056fc6cd49b9212d528a77c2af0d56f5",
  "hmac_signature":    "af8620250edfd83b994723921973b2e339a9534259b9f7015d2f8ec2567efaaf"
}
```

### Event 3 — chained to Event 2

```json
{
  "event_id":          "a1b2c3d4-0001-0001-0001-000000000003",
  "event_type":        "sso.login_failed",
  "payload":           {"failure_reason": "invalid_credentials", "tenant_id": "tenant-test-001"},
  "created_at":        "2026-06-19T10:01:00.000Z",
  "created_at_unix_ms": 1781863260000,
  "previous_event_id": "a1b2c3d4-0001-0001-0001-000000000002",
  "previous_signature": "af8620250edfd83b994723921973b2e339a9534259b9f7015d2f8ec2567efaaf",
  "hmac_input": "a1b2c3d4-0001-0001-0001-000000000003:sso.login_failed:{\"failure_reason\":\"invalid_credentials\",\"tenant_id\":\"tenant-test-001\"}:1781863260000:af8620250edfd83b994723921973b2e339a9534259b9f7015d2f8ec2567efaaf",
  "hmac_signature":    "ba05639de41cb1fadecb47931f2b5e2be884848834a0f664c5c6bc35e69a8a0a"
}
```

### Expected result

```json
{
  "valid": true,
  "errors": [],
  "events_checked": 3
}
```

### Verify with the pseudocode

```python
import hmac as hmac_lib, hashlib, json
from datetime import timezone, datetime

# Paste full verify_chain() from §3 above, then:
events = [
    {
        "event_id":          "a1b2c3d4-0001-0001-0001-000000000001",
        "event_type":        "sso.login_succeeded",
        "payload":           {"actor_user_hash": "abc123", "tenant_id": "tenant-test-001"},
        "created_at":        "2026-06-19T10:00:00.000Z",
        "previous_event_id": None,
        "hmac_signature":    "6926f92c8a76a5986bf1cac4d2de8ff0056fc6cd49b9212d528a77c2af0d56f5",
    },
    {
        "event_id":          "a1b2c3d4-0001-0001-0001-000000000002",
        "event_type":        "auth.session_created",
        "payload":           {"session_type": "sso", "tenant_id": "tenant-test-001"},
        "created_at":        "2026-06-19T10:00:05.000Z",
        "previous_event_id": "a1b2c3d4-0001-0001-0001-000000000001",
        "hmac_signature":    "af8620250edfd83b994723921973b2e339a9534259b9f7015d2f8ec2567efaaf",
    },
    {
        "event_id":          "a1b2c3d4-0001-0001-0001-000000000003",
        "event_type":        "sso.login_failed",
        "payload":           {"failure_reason": "invalid_credentials", "tenant_id": "tenant-test-001"},
        "created_at":        "2026-06-19T10:01:00.000Z",
        "previous_event_id": "a1b2c3d4-0001-0001-0001-000000000002",
        "hmac_signature":    "ba05639de41cb1fadecb47931f2b5e2be884848834a0f664c5c6bc35e69a8a0a",
    },
]

result = verify_chain(events, b"test-secret-key-do-not-use")
assert result == {"valid": True, "errors": [], "events_checked": 3}
print("Test vector passed.")
```

---

## 6. Security Questionnaire Standard Response

*For use in enterprise security questionnaires under "Audit log integrity and independent verification":*

> FORM's audit log uses a DEC-030 HMAC-SHA256 chain: each event's signature cryptographically commits to the preceding event's signature and payload, making silent deletion or modification of any event detectable. FORM publishes the full chain-verification algorithm specification and test vector as Data Room artefact HMAC-VERIFY-ALGO-001 (`compliance/docs/hmac-chain-verification-algorithm.md`). Enterprise customers implement verification in under two hours using the provided Python pseudocode. FORM also monitors chain integrity internally via alert AL-SIEM-05 (P0, auto-opens incident R-01 within 5 minutes of any detected chain break). Per-tenant HMAC verification keys are derived using HKDF-SHA256 and surfaced once via Admin Dashboard → Security → Audit Export; FORM does not retain a copy after display. A FORM-authored Python reference implementation is available to enterprise customers who request it via their CSM.

*SOC 2 mapping: CC1.1 (entity communicates HMAC chain purpose and objectives), C1.1 (algorithm spec demonstrates what data is in the chain and what the privacy floor excludes).*

---

## 7. Privacy Floor

The following invariants hold for all chain artefacts and this specification:

- `tenant_hmac_verify_key` is a HKDF-SHA256 derived key (32 bytes / 64 hex chars). FORM does not log, retain in a database, or transmit it in cleartext after Admin Dashboard display.
- No `user_email`, `user_id`, `employee_id`, body composition data, biometric data, or GDPR Art. 9 special category data appears in any event payload at the chain level. The DEC-030 privacy floor (payload design enforced at emit-time) precedes this artefact.
- The test vector in §5 uses fully synthetic, non-PII values. No real tenant, user, or session data was used.
- `canonical_payload` serialisation sorts keys deterministically; it does not expose identity fields — payload design is governed by DEC-030, not by the verification algorithm.

---

## 8. Cross-References

| Reference | Relevance |
|---|---|
| `docs/OBSERVABILITY.md §50` | Normative DEC-071 decision and full specification source |
| `docs/OBSERVABILITY.md §27.4` | Pull API `X-FORM-HMAC-Verify` header + push webhook `X-FORM-Signature` (batch-level signatures; §50 adds per-event chain verification on top) |
| `docs/OBSERVABILITY.md §27.7` | AL-SIEM-05 P0 chain-break alert (FORM-internal monitoring) |
| `docs/AUDIT_LOG_SCHEMA.md §DEC-030` | Canonical chain construction spec (FORM-internal; source of truth for chain structure) |
| `docs/CRYPTOGRAPHY_POLICY.md §5` | HMAC_KDF_SALT registration and annual rotation schedule |
| `docs/ENTERPRISE_ONBOARDING.md §3.5` | CSM protocol for sharing this artefact and confirming tenant key receipt |
| `docs/SECURITY_QUESTIONNAIRE.md LOG-04` | Standard questionnaire response for audit log integrity |
| `docs/DATA_ROOM.md §Technical Security` | HMAC-VERIFY-ALGO-001 data room entry |
| `docs/DECISION_LOG.md DEC-071` | Formal adoption decision (Option B — documentation-first) |

---

*v1.0 (2026-06-20): Initial publication. Artefact HMAC-VERIFY-ALGO-001. Closes `docs/OBSERVABILITY.md §50.10` item 1 (P0/M9). Test vector §5 computed values verified against Python pseudocode §3 with `hmac_secret = b"test-secret-key-do-not-use"` and `canonical_payload = json.dumps(payload, separators=(",", ":"), sort_keys=True)`. Sig1 = 6926f9…d56f5; Sig2 = af8620…efaaf; Sig3 = ba0563…a8a0a. Owner: security-engineer + devops-lead + compliance-officer.*
