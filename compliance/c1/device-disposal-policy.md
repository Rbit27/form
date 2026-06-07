# FORM · Media and Device Disposal Policy · POL-013

| Field | Value |
|---|---|
| **Policy ID** | POL-013 |
| **Version** | v1.0 |
| **Status** | IN_FORCE |
| **Effective date** | 2026-06-07 |
| **Owner** | `compliance-officer` + `security-engineer` |
| **Approved by** | founder (git-commit attestation, §7.3) |
| **Next review** | 2027-06-07 (annual; accelerated on fleet change or hire) |
| **Gap closures** | C1-GAP-002 · C1-GAP-004 · PRE-06 |
| **SOC 2 criteria** | C1.2 · CC6.5 · CC6.7 · CC1.4 |
| **Evidence artifact** | PRE-34-E-008 |
| **Source section** | `docs/SOC2_READINESS.md §66` |

> **Scope:** All physical devices — laptops, mobile devices, external storage media — that hold or have held FORM production credentials, user health data, debugging artifacts, or compliance evidence. Applies during the solo-founder phase and scales without architectural change to a multi-person team.
>
> **Privacy floor:** No device serial number, no health data excerpt, and no credential value may appear in disposal log entries, DEC-030 event payloads, or evidence artifacts. Use the `device_asset_id` identifier (FORM-DEV-XXX) in all records.

---

## 1. Asset Inventory

FORM maintains a device register at `compliance/assets/device-register.csv`. Every physical device that holds or has held FORM production credentials or health data must have a row in this register before it is used for FORM work.

Serial numbers are stored in 1Password under the vault entry **"FORM Asset Register — Device Serials"** and are never written into git-tracked files. The 1Password vault item hash is recorded in MDD-E-001.

**Current device register (solo-founder phase):** see `compliance/assets/device-register.csv`.

---

## 2. Data Wiping Standards

FORM follows **NIST SP 800-88 Rev. 1** for media sanitization. The required standard per storage category:

| Media Type | Standard | Method | Verification artifact |
|---|---|---|---|
| **SSD / NVMe (internal — Apple Silicon)** | NIST SP 800-88 §2.4 Purge | FileVault 2 EACS — cryptographic erasure of the volume encryption key via Secure Enclave discard | MDD-E-003: EACS completion screenshot + `diskutil` output |
| **USB / Flash storage** | NIST SP 800-88 §2.4 Purge | 7-pass Disk Utility Secure Erase (macOS); physical destruction if > 3 years old or wear-count > 1,000 P/E cycles | Disk Utility log; or vendor destruction certificate (MDD-E-004) |
| **Mobile device (iOS)** | NIST SP 800-88 §2.4 Purge | Settings → General → Transfer or Reset iPhone → Erase All Content and Settings | Screenshot of "iPhone is erased" confirmation; MDM removal confirmation |
| **Mobile device (Android)** | NIST SP 800-88 §2.4 Purge | Factory Reset + Google account removal + MDM device wipe | MDM wipe event log |
| **HDD (spinning, legacy only)** | NIST SP 800-88 §2.4 Clear + physical destruction | DoD 5220.22-M 7-pass overwrite, then certified vendor shredding | Overwrite log + MDD-E-004 |
| **Paper printouts** | N/A | Cross-cut shredding, DIN 66399 P-4 minimum | Log entry in MDD-E-003 |

**Cryptographic erasure note:** FileVault 2 AES-XTS-256 EACS discards the volume encryption key from the Secure Enclave. NIST SP 800-88 §2.4 explicitly endorses this method for encrypted SSD/NVMe storage. It is faster than multi-pass overwrite and produces a verifiable EACS completion receipt in System Settings.

---

## 3. Disposal Procedures by Device Category

### 3.1 Active development laptop (production credentials + health data)

1. **T−72 h — credential revocation.** Revoke all production credentials assigned to the device: GitHub OAuth apps and tokens, Cloudflare API tokens, 1Password device deauthorization, Anthropic API key (if device-specific), Cloudflare Workers (`wrangler logout`), PostHog personal API keys. Each revocation must emit a DEC-030 `admin.credential_revoked` event with `reason: "device_disposal"`.

2. **T−24 h — session invalidation.** Sign out of all FORM-related accounts while connected to the internet (ensures remote session invalidation, not just local token deletion).

3. **Disposal day.**
   - If MDM enrolled: trigger remote wipe from MDM console; wait for wipe-confirmation event; screenshot and file as MDD-E-003.
   - If MDM not yet enrolled (solo-founder pre-MDM phase): perform macOS EACS (System Settings → General → Transfer or Reset → Erase All Content and Settings). Confirm device boots to Setup Assistant. Screenshot and file as MDD-E-003.
   - For physical disposal (charity / resale): device must complete EACS and boot to Setup before leaving FORM possession. File chain-of-custody note per §7.
   - For secure third-party destruction: contact a certified vendor per §9. File vendor certificate as MDD-E-004.

4. **Post-disposal.** Update `device-register.csv` with `disposal_date`, `disposal_method`, `mdd_evidence_id`. Emit `asset.disposal_completed` DEC-030 event. Remove device from MDM enrollment.

### 3.2 Mobile test devices

1. Revoke TestFlight device registration (App Store Connect → Devices).
2. Revoke any device-specific developer certificate.
3. iOS: Erase All Content and Settings. Android: Factory Reset + MDM wipe.
4. If the device touched live FORM production data during debugging: apply §5 (Test Device Handling) before disposal.
5. File MDD-E-003; update device register.

### 3.3 External storage (USB drives, SSDs)

1. Determine whether the drive ever held: FORM codebase, production database exports, health data, compliance evidence, or credentials.
2. If yes to any: apply §2 Purge-level wipe.
3. If too old to wipe reliably (> 3 years, connector corrosion, SSD wear-count > 1,000 P/E cycles): certified third-party physical destruction per §9. File MDD-E-004.
4. Document in MDD-E-003; update device register.

---

## 4. Remote-Work Device Policy

FORM is fully remote. The following controls apply to all devices holding FORM credentials or health data:

| Control | Implementation | Evidence |
|---|---|---|
| Full-disk encryption | FileVault 2 on all macOS; iOS hardware-encrypted by default | MDD-E-001: FileVault status screenshot; iOS ADP enabled |
| Screen lock | Auto-lock ≤ 5 minutes; password-required on wake | MDD-E-001: System Preferences screenshot |
| MDM enrollment | Jamf Now (≤ 3 devices free) or Kandji when team grows | CC6-GAP-010; MDD-E-002 enrollment record |
| No credentials in plain text | All secrets in 1Password; `.env` files git-ignored | TruffleHog CI gate; SOC2_READINESS §56 key inventory |
| VPN not required | Cloudflare Access secures all admin surfaces (network-layer MFA) | SOC2_READINESS §26.2.3 |
| Jailbroken / rooted devices | Prohibited for FORM dev use; MDM compliance policy blocks enrollment | Explicit policy; audit at each device registration event |

**Post-hire extension:** MDM enrollment becomes mandatory before any production credential is issued to a new hire. The 14-day disposal SLA (§6) applies from the final employment day.

---

## 5. Test Device Handling — Devices That Touched Production Data

A device that accessed live FORM production data during incident response, debugging, or data export requires heightened disposal treatment.

**Trigger events** (any one classifies a device as a "production-data device"):
- Connected to Supabase production with `service_role` credentials.
- Ran a query against `cv_sessions`, `coaching_turns`, `wearable_readings`, `meal_log`, or `users` (production) that returned real user data.
- Received a webhook payload containing real user PII or health data.
- Held `SUPABASE_SERVICE_ROLE_JWT` or `HMAC_AUDIT_CHAIN_KEY` in memory or on disk.
- Used in a PAM break-glass session.

**Additional steps beyond standard §3 procedures:**

1. **Audit log search.** Before wiping, query `audit_log_events` (or Cloudflare Access logs) for sessions originating from the device during the suspect window. File summary as MDD-E-005.
2. **Two-person sign-off.** Requires a second authorized witness, or the solo-founder compensating control in §7.2 (time-delayed self-attestation with HMAC-chained DEC-030 emission).
3. **Health data cache verification (macOS).** Before EACS, inspect `~/Library/Application Support/FORM/`, `~/Downloads/`, and `~/Documents/form-data/`. Confirm nothing remains:
   ```
   find ~ -name "*.json" -newer [provisioning-date] | grep -i "health\|workout\|session\|coaching"
   ```
   Results (empty or reviewed-and-deleted) filed as MDD-E-006.
4. **Proceed with standard wipe** per §3.

---

## 6. Timeline Requirements

| Event | Deadline | Owner | Evidence |
|---|---|---|---|
| Pre-disposal credential revocation (planned) | ≤ 72 h before device leaves possession | compliance-officer | DEC-030 `admin.credential_revoked` events with `reason: "device_disposal"` |
| Emergency revocation (compromise suspected) | Immediate | security-engineer | Follows INCIDENT_RESPONSE.md R-16 |
| Device wipe completion — departing personnel | ≤ 14 calendar days from final employment day | compliance-officer | MDD-E-003 + `asset.disposal_completed` DEC-030 |
| Device wipe completion — active device retirement | ≤ 7 calendar days from end-of-life decision | compliance-officer | MDD-E-003 + `asset.disposal_completed` DEC-030 |
| Third-party destruction handoff | ≤ 30 calendar days from disposal initiation (if in-house wipe not possible) | compliance-officer | Chain-of-custody form + MDD-E-004 |
| Device register update | Same day as disposal completion | compliance-officer | `device-register.csv` git commit SHA in DEC-030 payload |
| Annual disposal audit | Q2 (June) per §15 compliance calendar | compliance-officer | MDD-E-007 annual audit log |

**Departing personnel flow (post-hire):**

```
T-14 days: Offboarding initiated (SOC2_READINESS §22.5 checklist)
T-5 days:  SCIM deprovisioning or manual account removal
T-0 (last day): All production credentials revoked; Cloudflare Access device cert removed
T+14 days: Device wipe completed; MDD-E-003 filed; device-register.csv updated
T+30 days: If device not returned: legal notice; MDM remote wipe triggered
```

---

## 7. Chain of Custody

### 7.1 Internal disposal (founder wipes device personally)

The `device-register.csv` entry with `disposal_method`, `disposal_date`, and a reference to the wipe-completion screenshot (MDD-E-003) constitutes the complete chain of custody for internal disposals.

### 7.2 Solo-founder compensating control (two-person requirement)

During the solo-founder phase, the two-person witness requirement is satisfied by:

1. Emit `asset.disposal_initiated` DEC-030 event **before** beginning the wipe, with `device_asset_id` and `authorized_by_self: true`.
2. After wipe completion, emit `asset.disposal_completed` with `wipe_verification_screenshot_sha256` (SHA-256 of MDD-E-003 screenshot file) in the payload.
3. The HMAC chain continuity between these two events — verifiable by the chain verification function in SOC2_READINESS §58.3 — serves as tamper-evident evidence of the two-event sequence, satisfying the chain-of-custody requirement in the absence of a human witness.
4. This compensating control is disclosed in the auditor narrative (SOC2_READINESS §38 Management Assertion Letter).

### 7.3 Physical transfer chain-of-custody form (third-party handoff)

**Policy:** No device leaves FORM's possession before completing a §2 Purge-level wipe or being handed directly to a certified destruction vendor.

**Chain-of-custody form for third-party handoffs:**

```
FORM DEVICE DISPOSAL — CHAIN OF CUSTODY
Date: [ISO 8601]
Asset ID: [FORM-DEV-XXX]
Handoff from: [Name, title, FORM]
Handoff to: [Vendor name, agent name, license/certification number]
Transport method: [Courier / in-person / secure postal]
Device sealed: [Yes / No] — confirm tamper-evident bag applied
Expected destruction date: [date]
Certificate promised by: [date]
Founder signature: _______________
```

**Founder attestation** (by committing this policy to the `form` repository):
1. All currently active devices in `compliance/assets/device-register.csv` are enrolled in, or are pending MDM enrollment per CC6-GAP-010.
2. All currently active devices have FileVault 2 / iOS hardware encryption enabled.
3. Upon disposal of any registered device, this policy will be followed.
4. This policy will be reviewed at each annual Q1 policy review (§15 compliance calendar) and updated if the device fleet changes.

*Attested: 2026-06-07. Founder.*

---

## 8. DEC-030 HMAC-Chained Audit Events

All disposal events are emitted via the `emit-audit-event` Cloudflare Worker and registered in `docs/AUDIT_LOG_SCHEMA.md`. Events are HMAC-chained and 7-year retention.

| Event type | Severity | Trigger | Key payload fields |
|---|---|---|---|
| `asset.disposal_initiated` | STANDARD | Before any wipe or handoff step begins | `device_asset_id`, `disposal_method`, `authorized_by_user_id`, `authorized_by_self` (bool) |
| `asset.disposal_completed` | STANDARD | After wipe verification or vendor certificate received | `device_asset_id`, `disposal_method`, `wipe_verification_screenshot_sha256`, `vendor_cert_id` (if third-party) |
| `asset.device_sanitized` | HIGH | After EACS or MDM wipe confirmation | `device_asset_id`, `sanitization_standard: "NIST-SP-800-88-Purge"`, `sanitized_by`, `mdm_wipe_event_id` (if MDM) |
| `asset.chain_of_custody_transferred` | HIGH | When device leaves FORM possession for third-party destruction | `device_asset_id`, `recipient_vendor`, `chain_of_custody_form_sha256`, `transport_method` |
| `asset.disposal_log_filed` | STANDARD | When `device-register.csv` is updated with disposal record | `device_asset_id`, `register_commit_sha`, `mdd_evidence_id` |

**Privacy invariant:** No event payload may contain device serial numbers in plain text (use `device_asset_id`), health data recovered during §5 cache verification, or any credential value.

**HMAC chain requirement:** `asset.disposal_initiated` must be followed by `asset.disposal_completed` within 30 days. A gap wider than 30 days triggers alert MDD-AL-01 (PagerDuty P2, assignee: compliance-officer).

---

## 9. Third-Party Destruction Vendor Criteria

For devices that cannot be wiped in-house (hardware failure, screen damage preventing boot):

| Criterion | Minimum requirement |
|---|---|
| Certification | NAID AAA Certified, e-Stewards, or R2/RIOS certified |
| Destruction standard | NIST SP 800-88 Rev. 1 Purge; or physical shredding to ≤ 6 mm particle size for HDDs |
| Certificate of destruction | Issued within 5 business days; includes serial number, destruction date, method |
| Chain of custody | Signed receipt at pickup; tamper-evident transport bags |
| EU data residency | Destruction within EU/EEA for devices that held EU user health data |
| GDPR Art. 28 | DPA signed with vendor before first handoff |

Vendor register: `compliance/c1/destruction-vendors.md` (MDD-P1-01).

**Prohibited:** Retail recycling programs (Apple Trade-In, Best Buy Recycling) — no chain of custody, no certificate of destruction.

---

## 10. SOC 2 Evidence Mapping

| SOC 2 criterion | This policy satisfies | Evidence artifact |
|---|---|---|
| **C1.2** Confidential information disposal | §2 NIST SP 800-88 wipe standards; §3 per-category procedures; §9 vendor criteria | MDD-E-003 (per-device wipe log), MDD-E-004 (vendor certs) |
| **CC6.5** Logical access termination before device decommission | §3.1 Step 1 (72 h revocation sequence); §6 departure timeline | DEC-030 `admin.credential_revoked` events; offboarding log |
| **CC6.7** Health data disposal restrictions | §5 production-data device handling; §4 no-local-health-data rule | MDD-E-005 (audit log search), MDD-E-006 (cache verification) |
| **CC1.4** Policy exists and is communicated | This document; §7.3 founder attestation | git commit timestamp; new-hire onboarding checklist (SOC2_READINESS §22.5 Step 1) |

---

*v1.0 (2026-06-07): Extracted from `docs/SOC2_READINESS.md §66` per MDD-P0-04. Closes C1-GAP-002 (🔴 → 🟡 Authored), C1-GAP-004 (P1 Open → 🟡 Authored), PRE-06 (🔴 → 🟡 Authored). Registered as PRE-34-E-008 evidence artefact. Policy enters 🟢 upon MDD-P0-01 (device-register.csv baseline) + MDD-P0-02 (founder FileVault attestation) + first disposal event with MDD-E-003 evidence filed. Owner: compliance-officer + security-engineer. Cross-references: docs/AUDIT_LOG_SCHEMA.md (asset.* DEC-030 events — MDD-P0-03), compliance/assets/device-register.csv (MDD-P0-01), compliance/c1/destruction-vendors.md (MDD-P1-01).*
