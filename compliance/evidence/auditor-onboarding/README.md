# Auditor Onboarding Package — Fieldwork Protocol Index

**Version:** v1.0 · 2026-06-18
**Owner:** compliance-officer
**Purpose:** Index of fieldwork procedures granted to SOC 2 auditors for the observation-period fieldwork window.

---

## Access

All documents in this package require read-only access granted by the compliance-officer. Credentials and access paths are provided at the start of the fieldwork engagement.

**Database access:** `compliance_reviewer` role (Supabase; read-only; scoped to `audit_log_events` and designated evidence tables).
**Linear access:** Read-only guest credentials for incident ticket lookup (see `incident-reason-verification.md §4.3`).
**1Password:** `AUDIT_LOG_HMAC_SECRET` and `INCIDENT_REASON_HASH_SALT` provided on request during fieldwork via 1Password `form-cloudflare-workers-secrets` vault share.
**R2 evidence bucket:** `compliance-officer` provides pre-signed read URLs for `form-soc2-evidence/` artefacts.

---

## Fieldwork Protocol Documents

| Document | Path | SOC 2 TSC | Purpose |
|---|---|---|---|
| Incident reason hash retrieval | `compliance/fieldwork/incident-reason-verification.md` | P3.2, CC2.2 | Verify that `incident.severity_changed` chain records contain no plaintext PII; retrieve IC reason from Linear ticket and verify SHA-256 hash |
| Concurrent incident chain verification | `compliance/fieldwork/concurrent-incident-chain-verification.md` | CC7.4, CC4.1 | Verify per-incident event sequences when two or more incidents were simultaneously active; explains interleaved master chain design and provides skip-and-verify SQL |
| SCIM Bulk Deprovision Guard | `compliance/fieldwork/scim-bulk-deprovision-guard.md` | CC6.3, A1.2, CC7.2, CC9.2 | Verify the preventive control that blocks mass SCIM deprovisioning above a per-tenant threshold. Includes BDG architecture, three-step audit traceability reconstruction (DEC-069 §35.7), GUARD-E-001 collection SQL (primary + CC6.3 assertion + CC7.2 stale-advisory), zero-event attestation procedure, and privacy floor attestation |

---

## Master Evidence Index

The authoritative evidence artefact registry is at `compliance/evidence/MASTER-INDEX` (SHA-256 manifest; maintained by `evidence-cron.ts`). See `docs/SOC2_READINESS.md §79` for the collection protocol and `§80.3` for R2 folder structure.

---

## Existing Fieldwork Procedures

The following fieldwork procedures are referenced in other compliance documents and may be provided separately during fieldwork:

| Procedure | Source reference | Status |
|---|---|---|
| Break-glass access verification | `docs/SOC2_READINESS.md` | Available — request from compliance-officer |
| DSAR chain retrieval | `docs/GDPR_DPIA.md` | Available — request from compliance-officer |
| Incident reason hash retrieval | `compliance/fieldwork/incident-reason-verification.md` | **Authored v1.0 (2026-06-18)** |
| Concurrent incident chain verification | `compliance/fieldwork/concurrent-incident-chain-verification.md` | **Authored v1.0 (2026-06-18)** |
| SCIM Bulk Deprovision Guard fieldwork guide | `compliance/fieldwork/scim-bulk-deprovision-guard.md` | **Authored v1.0 (2026-06-20)** |

---

*v1.1 (2026-06-20): Added SCIM Bulk Deprovision Guard fieldwork guide to the fieldwork protocol table and existing procedures table. Closes `docs/SSO_SCIM_IMPLEMENTATION.md §35.9` item 5 (P1/M13) and `docs/SOC2_READINESS.md §98.5` item 3. Cross-references: `compliance/fieldwork/scim-bulk-deprovision-guard.md` (v1.0, 2026-06-20); `docs/SOC2_READINESS.md §91` (GUARD-E-001 primary registration); `docs/SSO_SCIM_IMPLEMENTATION.md §35.9` item 5 (marked [x] Done 2026-06-20). Owner: compliance-officer.*

*v1.0 (2026-06-18): Initial authoring. Created to satisfy `docs/INCIDENT_RESPONSE.md §18.7 checklist item 1` (P1/M7) which requires the concurrent-incident fieldwork document to be "included in auditor onboarding package index at `compliance/evidence/auditor-onboarding/README.md`." Owner: compliance-officer.*
