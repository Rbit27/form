# FORM · Data Processing Agreement Receipt Registry

> **Evidence artifact:** PRE-34-E-004 (C1.1 Third-Party DPAs · SOC 2 Type II)
> **Gap closure:** C1-GAP-003 🔴 → 🟡 Authored (directory + register created; PDF receipts pending collection)
> **Owner:** `compliance-officer`. Co-review: `security-engineer` + `enterprise-architect`.
> **Review cadence:** Quarterly (aligned to §15 compliance calendar) + on any sub-processor addition or removal.
> **Cross-references:** `docs/SUBPROCESSORS.md §2` (canonical SP register), `docs/SOC2_READINESS.md §17/§34/§59` (vendor review programme, C1 deep-dive, VRM tier classification), `docs/GDPR_DPIA.md §4` (processing activities PA-01–PA-10), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 `vendor.*` events).
> **Canonical path:** `compliance/dpa/` — older references to `compliance/c1/dpas/` redirect here.

---

## Purpose

This directory stores:

1. **DPA receipts** — signed, click-through confirmed, or ToS-accepted DPA evidence for every GDPR Art. 28 sub-processor
2. **SCC confirmation records** — Standard Contractual Clauses (Module 2, controller→processor) for US-based sub-processors processing EU/UK personal data
3. **This README** — collection procedure, filing standard, and SOC 2 evidence mapping

GDPR Art. 28 prohibits using a sub-processor without a binding agreement covering the eight obligations in Art. 28(3)(a)–(h). Enterprise customers routinely request proof of these agreements during security questionnaires and before countersigning their own DPA with FORM.

**SOC 2 evidence claim:** Auditors requiring C1.1 third-party DPA evidence are directed to this directory. The combination of filed receipts here + the `docs/SUBPROCESSORS.md §2` register + `docs/SOC2_READINESS.md §17` vendor review programme constitutes the CC9.2 and C1.1 evidence package.

---

## Current DPA Status

> Last updated: 2026-06-12 · Next review: 2026-09-12

| ID | Sub-Processor | DPA Status | SCC Required | Collection Method | Evidence File | SOC 2 / Cert | Gap |
|---|---|---|---|---|---|---|---|
| SP-01 | **Supabase, Inc.** | 🔴 **Pending** | Yes — Module 2 (US tenants) | Supabase Dashboard → Settings → GDPR DPA | `supabase-dpa-YYYY-MM-DD.pdf` | SOC 2 Type I (2024) | VRM-GAP-001 P0 |
| SP-02 | **Cloudflare, Inc.** | 🟡 **ToS DPA active** | Yes — Module 2 (global edge) | Cloudflare Enterprise ToS (auto-accepted); download from Cloudflare Trust Hub | `cloudflare-dpa-tos-confirmation.md` | SOC 2 Type II (2025) + ISO 27001 | Verify download on file |
| SP-03 | **Resend, Inc.** | 🔴 **Pending** | Yes — Module 2 | Resend Dashboard → Settings → GDPR DPA or email legal@resend.com | `resend-dpa-YYYY-MM-DD.pdf` | None | VRM-GAP-001 P0 |
| SP-04 | **RevenueCat, Inc.** | 🔴 **Pending** | Yes — Module 2 | RevenueCat Dashboard → Settings → Data Privacy or email legal@revenuecat.com | `revenuecat-dpa-YYYY-MM-DD.pdf` | SOC 2 Type I (2024) | VRM-GAP-001 P0 |
| SP-05 | **Apple, Inc.** (APNs) | 🟡 **Compensating control** | Yes — Module 2 (US/Global) | Apple Developer Program License Agreement — no standalone Art. 28 DPA available commercially; OQ-VRM-01 compensating control documented | `apple-dpla-compensating-control.md` | None (see OQ-VRM-01) | OQ-VRM-01 open |
| SP-06 | **Sentry / Functional Software** | 🔴 **Pending** | Yes — Module 2 | Sentry Dashboard → Settings → Data Privacy → GDPR DPA | `sentry-dpa-YYYY-MM-DD.pdf` | SOC 2 Type II (2025) | VRM-GAP-001 P0 · AR-2026-Q2-02 |
| SP-07 | **Anthropic, PBC** | 🟢 **Enterprise DPA active** | Yes — Module 2 | Anthropic Console → Settings → Data Processing Agreement | `anthropic-dpa-YYYY-MM-DD.pdf` | SOC 2 Type II (2025) | — |
| SP-08 | **ElevenLabs, Inc.** | 🔴 **Pending** | Yes — Module 2 | ElevenLabs Dashboard → Account → Privacy or email privacy@elevenlabs.io | `elevenlabs-dpa-YYYY-MM-DD.pdf` | None (security questionnaire required) | OQ-SP-01 P1 |
| SP-09 | **PostHog, Inc.** (EU Cloud) | 🔴 **Pending** | No — EU-hosted, adequacy | PostHog Dashboard → Organization → Privacy & DPA | `posthog-dpa-YYYY-MM-DD.pdf` | SOC 2 Type I (2024) | P1 M4 |
| SP-10 | **Stripe, Inc.** | 🟢 **Stripe DPA active** | Yes — Module 2 | Auto-signed via Stripe Business ToS; download from Stripe Dashboard → Settings → Legal | `stripe-dpa-tos-confirmation.md` | SOC 2 Type I (2025) + PCI-DSS Level 1 | — |
| SP-11 | **WorkOS, Inc.** | 🟡 **Pending verification** | Yes — Module 2 | WorkOS Dashboard → Settings → Security; or email security@workos.com | `workos-dpa-YYYY-MM-DD.pdf` | SOC 2 Type II (2024) | Verify; P1 before enterprise GA |

**DPA collection summary (2026-06-12):**

| Status | Count | Sub-Processors |
|---|---|---|
| 🟢 Active — receipt filed | 0 of 2 ready | SP-07 Anthropic, SP-10 Stripe (DPAs active; PDF downloads pending filing) |
| 🟡 ToS / compensating control — confirmed | 0 of 3 | SP-02 Cloudflare, SP-05 Apple, SP-11 WorkOS (agreements active; download confirmations pending) |
| 🔴 Execution pending | 6 | SP-01, SP-03, SP-04, SP-06, SP-08, SP-09 |

---

## Collection Procedure

### Step 1 — Click-through / ToS-accepted DPAs (SP-02, SP-07, SP-10)

Most sub-processors offer DPA acceptance via a dashboard click-through or auto-include it in their Business ToS. For these:

1. Log in to the vendor dashboard using the FORM founder account.
2. Navigate to the DPA or Data Privacy section (paths listed in the table above).
3. If a DPA click-through is required, accept and download the signed confirmation PDF.
4. If auto-signed via ToS, download the current DPA version from the vendor's Trust Portal or legal page.
5. Save as `{vendor}-dpa-YYYY-MM-DD.pdf` in this directory.
6. Update the table above and add a row to `compliance/policy-approval-log.csv`.

### Step 2 — Standard DPA request (SP-01, SP-03, SP-04, SP-06, SP-08, SP-09)

For vendors requiring explicit DPA execution or dashboard activation:

1. Navigate to the vendor's data privacy dashboard page (paths listed above).
2. Activate the DPA if it requires a button/toggle click.
3. Download or request the signed confirmation document.
4. If no self-service path exists, send an email using the template in §5 below.
5. Save the PDF with filename pattern `{vendor}-dpa-YYYY-MM-DD.pdf`.
6. If the vendor requires FORM to supply a DPA rather than counter-signing theirs: attach `docs/SUBPROCESSORS.md §5` template; route to outside counsel if the vendor proposes material changes.

### Step 3 — SCC Module 2 verification

For US-based sub-processors (SP-01, SP-02, SP-03, SP-04, SP-06, SP-07, SP-08, SP-10, SP-11):

1. Confirm the DPA or ToS incorporates EU Standard Contractual Clauses (SCCs) Module 2 (controller → processor) per European Commission Decision 2021/914.
2. Look for the SCC annex in the vendor's DPA PDF. If absent, request an SCC addendum.
3. Note the SCC incorporation confirmation in the evidence file.
4. For UK data subjects: verify the DPA includes the UK IDTA or UK Addendum to EU SCCs.

### Step 4 — Evidence filing

After a DPA is collected:

1. File the PDF to this directory (`compliance/dpa/{vendor}-dpa-YYYY-MM-DD.pdf`).
2. Update the DPA status column in the table above.
3. Add a row to `compliance/policy-approval-log.csv` with:
   - `document_id`: `DPA-{VENDOR}-001`
   - `document_name`: `{Vendor} Data Processing Agreement`
   - `version`: vendor's current DPA version
   - `status`: `IN_FORCE`
   - `effective_date`: date of signing / click-through confirmation
   - `owner`: `compliance-officer`
4. Emit a `vendor.dpa_executed` DEC-030 HMAC-chained audit event (see §3 below).
5. Update `docs/SUBPROCESSORS.md §2` DPA Status column and the compliance dashboard.

### Step 5 — Remediation deadlines

| Vendor | Deadline | Priority | Blocker |
|---|---|---|---|
| SP-01 Supabase | **Before enterprise GA** | P0 | VRM-GAP-001 — blocks SOC 2 CC9.2 and enterprise onboarding |
| SP-06 Sentry | **2026-06-20** | P0 | AR-2026-Q2-02 escalation deadline |
| SP-03 Resend | **Before enterprise GA** | P0 | VRM-GAP-001 |
| SP-04 RevenueCat | **Before enterprise GA** | P0 | VRM-GAP-001 |
| SP-07 Anthropic | **2026-06-19** (download) | P1 | DPA is active; PDF receipt not yet filed |
| SP-10 Stripe | **2026-06-19** (download) | P1 | DPA is active via ToS; confirmation not yet filed |
| SP-02 Cloudflare | **2026-06-19** (download) | P1 | ToS DPA active; confirmation not yet filed |
| SP-09 PostHog | M4 (pre-launch) | P1 | Dashboard DPA click-through not yet activated |
| SP-08 ElevenLabs | M5 (pre-launch) | P1 | OQ-SP-01 — ElevenLabs DPA process clarification pending |
| SP-11 WorkOS | Before enterprise GA | P1 | Pending verification |
| SP-05 Apple | Pre-Series A | P2 | OQ-VRM-01 compensating control accepted for pre-Series A |

---

## DEC-030 Audit Events

All DPA lifecycle actions must be recorded as HMAC-chained audit events per DEC-030.

| Event | Severity | Retention | Trigger |
|---|---|---|---|
| `vendor.dpa_executed` | HIGH | 7 years | Sub-processor DPA signed, click-through accepted, or ToS-based DPA confirmed |
| `vendor.dpa_renewed` | HIGH | 7 years | Annual DPA confirmation or re-signature when vendor updates terms |
| `vendor.dpa_lapsed` | CRITICAL | 7 years | DPA confirmation not renewed within 13 months of last execution (dead-man trigger) |
| `vendor.dpa_removed` | HIGH | 7 years | Sub-processor offboarded; DPA no longer operative |

**Emitter assignment:** `compliance-officer` via admin console (manual entry at DPA execution). The `emit-audit-event` Cloudflare Worker validates the Zod schema before chain-appending.

**Payload schema (vendor.dpa_executed):**
```json
{
  "actor_id": "compliance-officer",
  "actor_role": "compliance_officer",
  "resource_type": "vendor_dpa",
  "resource_id": "SP-{NN}",
  "action": "vendor.dpa_executed",
  "severity": "HIGH",
  "retention_years": 7,
  "payload": {
    "vendor_name": "string",
    "vendor_id": "SP-{NN}",
    "dpa_version": "string",
    "effective_date": "ISO8601",
    "scc_module": "Module_2 | Module_3 | none",
    "scc_effective_date": "ISO8601 | null",
    "uk_addendum": true | false,
    "collection_method": "dashboard_click_through | tos_auto_accept | negotiated | email_request",
    "evidence_file": "compliance/dpa/{vendor}-dpa-YYYY-MM-DD.pdf",
    "gap_closures": ["C1-GAP-003", "VRM-GAP-001"],
    "soc2_criteria": ["C1.1", "CC9.2", "P6.2", "P6.3"]
  },
  "prev_hash": "HMAC_PREV"
}
```

---

## SOC 2 Evidence Mapping

| Artifact | SOC 2 Criterion | GDPR Article | Evidence ID | Status |
|---|---|---|---|---|
| Filed DPA receipts (all SP-01–SP-11) | C1.1 — Third-party confidentiality obligations | Art. 28(3) | PRE-34-E-004 | 🔴 Pending collection |
| DPA receipts for SP-07, SP-10 | C1.1 | Art. 28(3) | PRE-34-E-004 partial | 🟡 Active; PDF filing pending |
| SCC Module 2 confirmations | P6.3 — International data transfers | Art. 46(2)(c) | PRE-35-E-002 | 🔴 Pending collection |
| `vendor.dpa_executed` DEC-030 chain | CC9.2 — Third-party risk assessment | Art. 28(1) | CC9-E-DPA | 🔴 Pending first event |
| This README + register | C1.1, CC9.2, P6.2–P6.3 | Art. 28, Art. 30 | PRE-34-E-004 (structural) | 🟡 **Authored** (C1-GAP-003 🔴 → 🟡) |

**Gap tracker delta:**
- C1-GAP-003: 🔴 Open → 🟡 Authored (directory structure + register created; evidence artefacts pending collection)
- VRM-GAP-001: unchanged 🔴 (requires execution of SP-01/SP-03/SP-04/SP-06 DPAs; this README defines the collection procedure)
- PRE-34-E-004: 🔴 To collect → 🟡 Structural artefact filed (PDF receipts pending)

---

## DPA Request Email Template

Use when a vendor has no self-service DPA path:

```
Subject: Data Processing Agreement Request — FORM (form.coach)

Hi [Vendor Legal Team],

My name is [Name], and I'm the compliance officer at FORM (form.coach), a fitness
technology company based in [jurisdiction]. We process personal data of EU/UK users
in our application and use [Vendor Name] for [service description].

Under GDPR Article 28, we are required to execute a Data Processing Agreement (DPA)
with all sub-processors before processing personal data on our users' behalf.

Could you please:
1. Provide your standard DPA for our review, OR
2. Point us to a self-service DPA acceptance option in your dashboard, OR
3. Confirm if your Terms of Service already incorporate a GDPR Art. 28 DPA with
   Standard Contractual Clauses (SCCs Module 2)?

For transfers involving EU personal data to a US recipient, we additionally require
SCCs (Annex I, II, and III) incorporated into or appended to the DPA.

If you require a DPA counter-signature from FORM, please send your standard form
and we will review with our outside counsel.

Best regards,
[Name]
compliance@form.coach
```

---

## Compensating Control: Apple APNs (SP-05)

Apple does not offer a standalone Art. 28 DPA for APNs access. Compensating controls:

1. **Data minimisation:** Push notification payloads contain non-health text only (coaching cues, reminder text). No health values, workout metrics, or GDPR Art. 9 data appear in APNs payloads — enforced by `notifications-worker` payload schema validation.
2. **Apple Developer Program License Agreement (DPLA):** The DPLA §3.3.2 and Apple's Privacy Policy contain data processing commitments covering APNs that serve a partial Art. 28 function.
3. **Legal assessment:** Outside counsel (pre-Series A) to confirm whether the DPLA satisfies GDPR Art. 28 for APNs, or whether an alternative push delivery mechanism is required for EU enterprise customers.
4. **Evidence:** Screenshot of DPLA acceptance date + legal counsel memo (when obtained) filed as `apple-dpla-compensating-control.md` in this directory.

---

## Placeholders for PDF Receipts

The following files are expected here once DPAs are collected:

```
compliance/dpa/
├── README.md                                ← this file
├── dpa-collection-checklist.md              ← per-step checklist (one per DPA collection)
├── anthropic-dpa-YYYY-MM-DD.pdf             ← SP-07 · 🔴 download pending
├── stripe-dpa-tos-confirmation.md           ← SP-10 · 🔴 download pending
├── cloudflare-dpa-tos-confirmation.md       ← SP-02 · 🔴 download pending
├── supabase-dpa-YYYY-MM-DD.pdf              ← SP-01 · 🔴 execution pending (P0)
├── resend-dpa-YYYY-MM-DD.pdf               ← SP-03 · 🔴 execution pending (P0)
├── revenuecat-dpa-YYYY-MM-DD.pdf           ← SP-04 · 🔴 execution pending (P0)
├── sentry-dpa-YYYY-MM-DD.pdf               ← SP-06 · 🔴 execution pending (P0)
├── posthog-dpa-YYYY-MM-DD.pdf              ← SP-09 · 🔴 click-through pending (P1)
├── elevenlabs-dpa-YYYY-MM-DD.pdf           ← SP-08 · 🔴 pending (P1 · OQ-SP-01)
├── workos-dpa-YYYY-MM-DD.pdf               ← SP-11 · 🟡 pending verification (P1)
└── apple-dpla-compensating-control.md      ← SP-05 · compensating control doc (P2)
```

Files are named `{vendor}-dpa-YYYY-MM-DD.{ext}` where the date is the effective date of the DPA or the date of click-through confirmation.

---

*v1.0 · 2026-06-12 · Owner: compliance-officer + security-engineer + enterprise-architect*
*Created: compliance/dpa/ directory — closes structural gap (C1-GAP-003 🔴 → 🟡 Authored)*
*Gap advance: PRE-34-E-004 from 🔴 To collect → 🟡 Structural artefact filed*
*Remaining to 🟢: collect PDF receipts for SP-01/03/04/06/07/08/09/10/11; emit vendor.dpa_executed DEC-030 events*
*Cross-ref: docs/SOC2_READINESS.md §17/§34/§59; docs/SUBPROCESSORS.md §2; docs/AUDIT_LOG_SCHEMA.md (DEC-030 vendor.* events)*
