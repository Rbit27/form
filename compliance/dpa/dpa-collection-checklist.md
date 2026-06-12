# FORM · DPA Collection Checklist Template

> One checklist per sub-processor DPA collection event.
> Copy and rename: `dpa-collection-checklist-{vendor}-YYYY-MM-DD.md`.
> Archive completed checklists in this directory alongside the PDF receipt.
> Evidence artifact: PRE-34-E-004 (one row per vendor).
> Owner: `compliance-officer`.

---

## Checklist Header

| Field | Value |
|---|---|
| **Vendor** | [Vendor name] |
| **Sub-processor ID** | SP-{NN} |
| **DPA collection date** | YYYY-MM-DD |
| **Collected by** | compliance-officer |
| **DPA effective date** | YYYY-MM-DD |
| **DPA version / URL** | [Vendor DPA page URL or document version] |
| **SCC requirement** | Yes / No |
| **SCC module** | Module 2 (controller → processor) / None |
| **UK addendum required** | Yes / No |
| **Collection method** | Dashboard click-through / ToS auto-accept / Email request / Negotiated |
| **DEC-030 event emitted** | `vendor.dpa_executed` — event ID: [UUID] |
| **Gap closures** | C1-GAP-003 / VRM-GAP-001 / — |

---

## Pre-Collection

- [ ] **Confirm data categories processed** — verify the vendor processes data categories listed in `docs/SUBPROCESSORS.md §2` (PA-01–PA-10 as applicable)
- [ ] **Confirm GDPR Art. 9 health data handling** — verify health data pseudonymisation / exclusion per DPIA PA constraints
- [ ] **Check vendor SOC 2 / cert status** — verify against `docs/VENDOR_REGISTRY.md` and `docs/SOC2_READINESS.md §59`
- [ ] **Check current DPA URL** — vendor DPA page may have changed since last review; confirm URL is current
- [ ] **Confirm SCC requirement** — US-based processors require SCC Module 2 for EU/UK personal data

## DPA Collection

- [ ] **Navigate to vendor DPA page** — see path in `compliance/dpa/README.md` table
- [ ] **Accept / activate DPA** — click through confirmation if dashboard self-service
- [ ] **Download confirmation document** — PDF, screenshot, or email confirmation
- [ ] **Verify SCC Module 2** is included in the DPA or as an annex
  - [ ] Annex I (data processing description) populated
  - [ ] Annex II (technical and organisational measures) populated
  - [ ] Annex III (sub-processors) — vendor's own sub-processors listed
- [ ] **Verify UK Addendum or IDTA** if vendor processes UK personal data
- [ ] **Confirm training prohibition** — DPA must prohibit vendor from using FORM data to train AI/ML models (critical for Anthropic, ElevenLabs)

## Post-Collection Filing

- [ ] **Save PDF** as `{vendor}-dpa-YYYY-MM-DD.pdf` in `compliance/dpa/`
- [ ] **Update `compliance/dpa/README.md`** DPA status table — change 🔴 → 🟢
- [ ] **Update `docs/SUBPROCESSORS.md §2`** DPA Status column
- [ ] **Add row to `compliance/policy-approval-log.csv`** with `document_id: DPA-{VENDOR}-001`
- [ ] **Emit DEC-030 event** `vendor.dpa_executed` via admin console
  - Record event UUID in Checklist Header above
- [ ] **Update `docs/SOC2_READINESS.md §59` VRM tier entry** — update DPA status column
- [ ] **If this closes VRM-GAP-001** — update gap register in `docs/SOC2_READINESS.md §51`

## Verification

- [ ] **SHA-256 hash of PDF** — compute and note below for chain of custody:
  ```
  sha256sum compliance/dpa/{vendor}-dpa-YYYY-MM-DD.pdf
  # [paste hash here]
  ```
- [ ] **PDF authenticity check** — document is from vendor domain / trust portal, not a template
- [ ] **Effective date confirmed** — DPA effective date is on or before FORM's first data transfer to this vendor

---

## Notes

[Record any deviations, vendor-specific considerations, or open questions here]

---

*Template v1.0 · 2026-06-12 · compliance-officer*
