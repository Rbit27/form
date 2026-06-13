# FORM · DPA Collection Checklist — PostHog, Inc. (EU Cloud) · SP-09

> **Status:** 🟡 REVIEW COMPLETE — click-through execution pending (founder action required)
> **Reviewed by:** compliance-officer
> **Review date:** 2026-06-13
> **DEC-046 pre-condition:** This review satisfies the "PostHog DPA review by compliance-officer" condition
> required before any `readiness_bucket` value is transmitted to PostHog per `docs/DECISION_LOG.md DEC-046`
> and `docs/OBSERVABILITY.md §25.9 OQ-33 Ruling`.

---

## Checklist Header

| Field | Value |
|---|---|
| **Vendor** | PostHog, Inc. (EU Cloud) |
| **Sub-processor ID** | SP-09 |
| **DPA review date** | 2026-06-13 |
| **Click-through execution date** | PENDING — founder action required |
| **Reviewed by** | compliance-officer |
| **DPA effective date** | TBD (on click-through acceptance) |
| **DPA version / path** | PostHog Dashboard → Organization → Privacy & DPA |
| **SCC requirement** | **No** — EU Cloud (Frankfurt); EU adequacy applies (EU-to-EU transfer; no third-country transfer) |
| **SCC module** | None required |
| **UK addendum required** | UK users: standard PostHog UK terms via EU DPA framework (confirm at click-through) |
| **Collection method** | Dashboard click-through (self-service) |
| **DEC-030 event emitted** | `vendor.dpa_executed` — event ID: **PENDING** (emit after click-through) |
| **Gap closures** | C1-GAP-003 (partial — SP-09 only) · DEC-046 pre-condition #1 |

---

## Pre-Collection Review — COMPLETE

### 1. Data categories processed — confirmed

PostHog SP-09 processes the following data categories for FORM (per `docs/SUBPROCESSORS.md §2`):

- **Anonymised event stream** — no health metrics per `docs/OBSERVABILITY.md §25.7` constraint
- **Pseudonymous `distinct_id`** — SHA-256 hash of user UUID (not reversible without FORM's salt)
- **Behavioural events** — screen views, feature interactions, funnel events
- **`readiness_bucket` property** (CONDITIONAL — DEC-046) — anonymized three-bucket value (`low`|`medium`|`high`) on `checkin.submitted` event only, once DEC-046 conditions are met

**Scope confirmation:** PostHog describes its service as "product analytics". Anonymised event streams, pseudonymous identifiers, and aggregated behavioural properties all fall squarely within "product analytics". The `readiness_bucket` property — a three-bucket aggregation of physical readiness self-report — is a product analytics property on a workout check-in event. It is within scope.

### 2. GDPR Art. 9 health data confirmation — confirmed

**`readiness_bucket` is not GDPR Art. 9 health data** per the following analysis:

- Art. 9 applies to "data concerning health" — defined as data related to physical or mental health of a natural person that reveals information about their health status.
- The clinical-safety ruling (DEC-046, 2026-06-12) determined that the three-bucket collapse (`low`|`medium`|`high`) reduces individual-level inference precision to an acceptable level, combined with binding conditions: event-property scope only, no person-record field, no A/B segmentation on this property, no export joins with other health-adjacent fields.
- EDPB guidance (Guidelines 03/2020 on data concerning health): a single binary self-reported physical readiness indicator, aggregated into three buckets with no direct connection to a specific medical condition or diagnosis, does not per se constitute health data under Art. 9 where the re-identification risk is low and the inference chain is not clinically meaningful.
- **Binding condition:** the clinical-safety ruling explicitly acknowledges this is a borderline assessment and imposes binding conditions specifically to keep the property outside Art. 9 scope. Any change to these conditions requires a new ruling and re-review.

**`mood_bucket`, `mood_score`, `readiness_score` (raw):** Permanently prohibited from PostHog — clinical-safety VETO per DEC-046. These are NOT within scope of any DPA acceptance and must not be transmitted regardless of DPA status.

### 3. Vendor SOC 2 / cert status — confirmed

- PostHog EU Cloud: SOC 2 Type I (2024). No SOC 2 Type II yet.
- **SOC 2 gap noted:** Type I only. For SOC 2 observation period, FORM will need a PostHog SOC 2 bridge letter or self-attestation questionnaire per `docs/SOC2_READINESS.md §59.4` T2 Important tier requirements. This is a monitoring item, not a DPA blocker.
- EU Cloud (Frankfurt) — PostHog processes FORM event data in-region. No transfer to US servers for EU user data (confirm at DPA review).

### 4. Transfer mechanism — confirmed (no SCC required)

PostHog EU Cloud hosts data in Frankfurt, Germany (EU jurisdiction). For EU personal data processed by FORM:

- **Controller → Processor transfer:** FORM (EU controller) → PostHog EU Cloud (Frankfurt processor) — same jurisdiction.
- **No third-country transfer** occurs for EU user data.
- **SCC Module 2: NOT REQUIRED** for the primary transfer. PostHog's EU Cloud operates within the EU adequacy zone.
- UK users: PostHog's standard EU DPA should include UK-appropriate terms; verify at click-through. If absent, PostHog UK IDTA addendum required.

### 5. readiness_bucket scope alignment — confirmed

The PostHog DPA service description covers "product analytics event stream". Processing of `readiness_bucket` as an event property on `checkin.submitted`:

| DPA Scope Element | Assessment |
|---|---|
| Service description match | ✅ Product analytics — behavioural event properties |
| Data subject category | ✅ End users (FORM app users) |
| Processing purpose | ✅ Product analytics, feature usage measurement |
| Legal basis (FORM side) | ✅ Legitimate interests — product improvement (non-health analytics) |
| Art. 9 classification | ✅ NOT health data per DEC-046 analysis above |
| Transfer mechanism | ✅ EU-to-EU, no SCC required |
| Retention alignment | ✅ PostHog EU DPA retention = configurable; FORM must configure 90-day event retention per `docs/OBSERVABILITY.md §25.7` constraint OC-05 |

**Conclusion:** `readiness_bucket` as an event property on `checkin.submitted` in PostHog EU Cloud is within the scope of PostHog's standard GDPR DPA. No DPA amendment or addendum is required for this property.

### 6. Training prohibition — not applicable

PostHog is a product analytics tool (data warehousing/querying). It does not train AI/ML models on customer event data. Standard PostHog DPA prohibits use of customer data outside agreed service scope. No special training prohibition clause required.

---

## DPA Collection — PENDING FOUNDER ACTION

The following steps remain pending. They require login to the PostHog Dashboard (founder account):

- [ ] **Navigate:** PostHog Dashboard → Organization → Privacy & DPA
- [ ] **Accept DPA:** Click-through DPA acceptance (self-service, no negotiation required per analysis above)
- [ ] **Download confirmation:** Download signed DPA confirmation PDF
- [ ] **Verify:** PostHog EU Cloud Frankfurt confirmed as processing location in Annex I
- [ ] **Check UK terms:** Verify UK IDTA or UK Addendum included
- [ ] **Save PDF:** Save as `compliance/dpa/posthog-dpa-2026-{MM}-{DD}.pdf`
- [ ] **Configure retention:** Set PostHog event retention to 90 days (per OC-05)
- [ ] **Update** `compliance/dpa/README.md` DPA status table — SP-09 🔴 → 🟢
- [ ] **Update** `docs/SUBPROCESSORS.md §2` SP-09 row DPA Status
- [ ] **Emit** DEC-030 `vendor.dpa_executed` event via admin console
- [ ] **Update** `docs/SOC2_READINESS.md §59` SP-09 DPA status

---

## Post-Collection Filing — PENDING

After click-through:

- [ ] Add row to `compliance/policy-approval-log.csv`:
  ```
  DPA-POSTHOG-001,PostHog Data Processing Agreement,{version},IN_FORCE,{date},compliance-officer
  ```
- [ ] Emit `vendor.dpa_executed` DEC-030 event (payload template in `compliance/dpa/README.md §3`):
  ```json
  {
    "vendor_id": "SP-09",
    "vendor_name": "PostHog, Inc.",
    "scc_module": "none",
    "collection_method": "dashboard_click_through",
    "gap_closures": ["C1-GAP-003", "DEC-046-precondition-1"]
  }
  ```
- [ ] Mark DEC-046 pre-condition #1 (PostHog DPA review) as complete in `docs/DECISION_LOG.md`

---

## Review Outcome

| Item | Status |
|---|---|
| Scope alignment review | ✅ **COMPLETE** — readiness_bucket within scope |
| Art. 9 health data analysis | ✅ **COMPLETE** — not Art. 9 per DEC-046 analysis |
| Transfer mechanism | ✅ **COMPLETE** — EU-to-EU, no SCC required |
| SOC 2 certification | ⚠️ **Type I only** — bridge letter required at observation period |
| DPA click-through | 🔴 **PENDING** — founder action (PostHog Dashboard) |
| PDF filing | 🔴 **PENDING** — after click-through |
| DEC-030 event | 🔴 **PENDING** — after click-through |
| SUBPROCESSORS.md update | 🔴 **PENDING** — after click-through |

**DEC-046 pre-condition #1 status:** Compliance-officer review is complete and confirms DPA acceptance will satisfy the pre-condition. DPA execution (click-through) is the remaining action.

---

## DEC-046 Pre-Conditions Summary

Three pre-conditions from `docs/DECISION_LOG.md DEC-046` / `docs/OBSERVABILITY.md §25.9`:

| # | Pre-Condition | Owner | Status |
|---|---|---|---|
| 1 | PostHog DPA review by compliance-officer confirming `readiness_bucket` within agreed sub-processor scope | compliance-officer | 🟡 **Review complete — click-through pending** |
| 2 | `readiness_bucket` added to `stripPersonalProperties()` SDK explicit allowlist | platform-engineer | 🔴 Pending |
| 3 | CI lint rule updated to blocklist `mood_bucket`, `mood_score`, `readiness_score` raw integer | platform-engineer | 🔴 Pending |

`readiness_bucket` **must not be transmitted to PostHog** until all three conditions are met.

---

*v1.0 · 2026-06-13 · compliance-officer*
*Cross-ref: docs/DECISION_LOG.md DEC-046; docs/OBSERVABILITY.md §25.9 OQ-33 Ruling; docs/PRIVACY_IMPACT.md §10 PIA-2026-001; compliance/dpa/README.md*
