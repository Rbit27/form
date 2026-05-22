# C1 — Confidentiality Evidence

This directory contains evidence artifacts for SOC 2 C1 (Confidentiality) criteria.

**SOC 2 criteria:** C1.1 (identify and maintain confidential information), C1.2 (dispose of confidential information)  
**Owner:** compliance-officer + security-engineer  
**Reference doc:** `docs/SOC2_READINESS.md §34`

## Evidence Index

| Artifact ID | Description | Status | Path |
|---|---|---|---|
| **PRE-34-E-001** | Confidential Data Asset Inventory — all systems with Confidential/Sensitive/Restricted data; classification, retention, access controls; founder-signed | 🟡 Authored — pending founder signature | `compliance/c1/data-asset-inventory.md` |
| **PRE-34-E-002** | Data Classification Policy cross-reference | ✅ Done — `docs/SOC2_READINESS.md § Data Classification Policy` | In-document |
| **PRE-34-E-003** | NDA / employment confidentiality agreement template — founder-signed as template approval | 🔴 To create (C1-GAP-001) | `compliance/c1/nda-template.md` |
| **PRE-34-E-004** | DPA filing receipts for all sub-processors | 🔴 To collect (C1-GAP-003) | `compliance/c1/dpas/` |
| **PRE-34-E-005** | Supabase encryption screenshot (AES-256 at-rest confirmation) | 🔴 To file | `compliance/evidence/c1/supabase-encryption.png` |
| **PRE-34-E-006** | Column-level encryption DDL commit reference | 🟡 Partial — DATA_MODEL.md §6 authored; implementation commit pending | `docs/DATA_MODEL.md §6` |
| **PRE-34-E-007** | Sample DEC-030 `privacy.erasure_completed` event (staging) | 🔴 Pre-launch staging gate | `compliance/evidence/c1/sample-erasure-event.json` |
| **PRE-34-E-008** | Media/device disposal policy | 🔴 To create (C1-GAP-004) | `compliance/c1/device-disposal-policy.md` |

## C1 Gap Status

| Gap ID | Description | Priority | Status |
|---|---|---|---|
| **C1-GAP-001** | NDA / employment confidentiality agreement template not yet drafted | P0 — pre-hire gate | 🔴 Open |
| **C1-GAP-002** | Confidential Data Asset Inventory (PRE-34-E-001) | P1 | 🟡 AUTHORED — pending founder signature |
| **C1-GAP-003** | DPA receipts not yet collected and filed in `compliance/c1/dpas/` | P0 — pre-enterprise gate | 🔴 Open (Sentry + Better Stack DPAs outstanding) |
| **C1-GAP-004** | Media/device disposal policy not yet drafted | P1 | 🔴 Open |

## Signature Procedure for PRE-34-E-001

When the founder signs the Data Asset Inventory:

1. Founder adds name, signature confirmation, and date to §10 of `data-asset-inventory.md`
2. Commit with message: `compliance: PRE-34-E-001 signed · C1-GAP-002 closed`
3. Tag: `c1-inv-001-signed-YYYY-MM-DD`
4. Update `compliance/policy-approval-log.csv` — add C1-INV-001 row
5. Update `docs/SOC2_READINESS.md §34` — C1-GAP-002 🟡 → 🟢, PRE-34-E-001 🟡 → ✅

---

*Owner: compliance-officer · Created: 2026-05-22*
