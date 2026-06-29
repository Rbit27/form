# Q3 2026 Quarterly Access Review — Pre-Review Checklist

**Review due:** 2026-07-31
**Pre-review gate (14 days ahead):** 2026-07-17
**Owner:** compliance-officer (founder)
**SOC 2 criteria:** CC6.2, CC6.3, CC6.5, CC4.2, CC1.2
**Cross-reference:** `docs/SOC2_READINESS.md §23` (procedure), `§65.11` (Q3 forward plan), `§65.13 AR-P1-01` (this calendar gate closes the AR-2026-Q2-01 remediation), `§128` (OQ-AR-01 and OQ-TDD-01 resolved — 2026-06-29)

**Why this file exists:** Q2 2026 access review (executed 2026-06-05) was conducted 36 days late — no calendar gate was in place (finding AR-2026-Q2-01, Low severity). This document is the remediation: a standing pre-review checklist committed to the repo so the Q3 review cannot silently slip.

---

> **OQ-AR-01 and OQ-TDD-01 resolved — see SOC2_READINESS.md §128, v3.53.0, 2026-06-29.** Pilot-tier tenants are now reviewed under §23.2.3 at the same standard as production enterprise tenants (§128.1). Destruction certificate DELETION SCOPE block updated with `Processing activities covered:` boilerplate (§128.3).

## Gate: 2026-07-17 (T-14 days)

Complete all items below by 2026-07-17. If any item is blocked, escalate immediately — the review must not start incomplete.

### Step 0 — Confirm review ownership

- [ ] Confirm reviewer identity: founder (solo-founder compensating control per §23.7 until a second reviewer is hired)
- [ ] If an engineering or compliance hire has occurred since Q2 review: second reviewer required for Step 3 (account verification) per §23.7 expiry trigger. Identify second reviewer now.
- [ ] Block 2026-07-28 to 2026-07-31 in calendar for review execution (3–4 hours total)

### Step 1 — System inventory check

Review the 11 human account systems from the Q2 artifact and confirm the list is still current:

| System | Account | Expected status |
|---|---|---|
| GitHub | @Rbit27 (founder) | Active |
| Cloudflare | founder email | Active |
| Supabase | founder email | Active |
| 1Password | founder email | Active |
| PostHog | founder email | Active |
| Sentry | founder email | Active |
| Stripe | founder email | Active |
| ElevenLabs | founder email | Active |
| Anthropic | founder email | Active |
| Apple Developer | founder email | Active |
| Google Play | founder email | Active |

- [ ] Verify no new SaaS services have been added to the stack since 2026-06-05 that are not on this list. Add any new services before the review date.
- [ ] If any enterprise pilot tenants went live between Q2 and Q3 review: confirm §23.2.3 enterprise tenant query is ready to run against production using `WHERE t.tier IN ('enterprise', 'pilot')` per §128.1 OQ-AR-01 resolution (pilot-tier reviewed at same standard as enterprise)

### Step 2 — Service account / API token inventory

- [ ] Re-enumerate all API tokens and service accounts from the Q2 artifact §65.3.2 table (14 rows)
- [ ] Check for new tokens created since Q2 review. Any new token must be added to the authorized roster (see Step 3)
- [ ] Identify tokens approaching SLA boundaries: `CLOUDFLARE_API_KEY` and `WORKOS_API_KEY` are flagged to reach 180-day SLA by ~2026-09-03 — note in review artifact even if not yet past SLA

### Step 3 — Authorized roster comparison

- [ ] Pull `compliance/access-review/authorized-roster.md` (v1.0 — to be created per AR-P0-03 by 2026-06-07)
- [ ] Compare current account inventory (Steps 1–2) against authorized roster
- [ ] Document any discrepancy. There should be none; if there is, treat as unauthorized access — escalate to INCIDENT_RESPONSE.md R-20 before proceeding

### Step 4 — Sentry DPA status check

- [ ] Check status of AR-2026-Q2-02: has Sentry DPA escalation been sent to legal@sentry.io?
- [ ] If escalation was sent: record response status in review artifact
- [ ] If escalation was not sent: treat as overdue (AR-P1-02 deadline was 2026-06-12) — escalate immediately and document in Q3 artifact

### Step 5 — Credential rotation review

- [ ] Verify `CLOUDFLARE_API_KEY` rotation status: last rotated date vs 180-day SLA
- [ ] Verify `WORKOS_API_KEY` rotation status: last rotated date vs 180-day SLA
- [ ] Verify any credentials identified at Q2 review (AR-2026-Q2-03/04/05) are recorded in `form-crypto-health` KV namespace (AR-P0-05)
- [ ] Identify any credentials that will reach SLA boundary before Q4 review (2026-10-31)

### Step 6 — DEC-030 event preparation

- [ ] Confirm `system.access_review_completed` event is registered in `docs/AUDIT_LOG_SCHEMA.md` (closed AR-P1-03, 2026-06-05 ✅)
- [ ] Confirm `system.credential_rotated` event is registered in `docs/AUDIT_LOG_SCHEMA.md` (closed AR-P1-04, 2026-06-05 ✅)
- [ ] Prepare event payload template for post-review DEC-030 emission (copy from Q2 artifact §65.9, update quarter/date/artifact_sha256 fields)

---

## Review Execution: 2026-07-28 to 2026-07-31

Follow procedure `docs/SOC2_READINESS.md §23` (Steps 1–6) verbatim. Use the Q2 artifact `compliance/access-review/2026-q2/access-review-2026-Q2.md` as the template for the Q3 artifact.

**Q3 artifact path:** `compliance/access-review/2026-q3/access-review-2026-Q3.md`
(Commit to private `form-compliance` repository, not this product repo)

---

## Post-Review: by 2026-07-31 (same day or next business day)

- [ ] Compute SHA-256 of Q3 artifact: `sha256sum access-review-2026-Q3.md`
- [ ] Emit `system.access_review_completed` DEC-030 event with `artifact_sha256` field and `review_quarter: "2026-Q3"`; record HMAC chain event ID in this file
- [ ] Update `docs/SOC2_READINESS.md §51` gap register: add Q3 review date to CC6-GAP-001 closure evidence
- [ ] Create `compliance/calendar/q4-2026-access-review.md` (same format as this file) with Q4 due date 2026-10-31 and pre-review gate 2026-10-17

**DEC-030 event ID (to be filled post-review):** `[PENDING — fill after Q3 review execution]`
**Artifact SHA-256 (to be filled post-review):** `[PENDING — fill after Q3 review execution]`
**Q3 review executed by (to be filled):** `[PENDING]`
**Q3 review date (to be filled):** `[PENDING]`

---

## What Changes vs Q2

Per `docs/SOC2_READINESS.md §65.11`:

| Item | Q2 2026 | Q3 2026 |
|---|---|---|
| Calendar gate | ❌ Not in place (root cause of AR-2026-Q2-01) | ✅ This file — AR-2026-Q2-01 remediation |
| Review latency | 36 days late | Target: within 3 days of 2026-07-31 deadline |
| Enterprise tenants | 0 (pre-launch) | Check for pilot tenants; run §23.2.3 query if any |
| Sentry DPA | 🟡 Pending escalation | Target: closed before this review (AR-P1-02) |
| CLOUDFLARE_API_KEY | ✅ Within 180-day SLA | 🟠 Will reach SLA by ~2026-09-03 — flag in artifact |
| WORKOS_API_KEY | ✅ Within 180-day SLA | 🟠 Will reach SLA by ~2026-09-03 — flag in artifact |
| Second reviewer | N/A (solo-founder compensating control) | Activate if any hire by 2026-07-17 |

---

*v1.1 (2026-06-29): §128 cross-reference added. OQ-AR-01 and OQ-TDD-01 resolved — see SOC2_READINESS.md §128, v3.53.0, 2026-06-29. Cross-reference header updated to include §128. Step 1 pilot-tenant bullet updated to reference §128.1 OQ-AR-01 resolution and `WHERE t.tier IN ('enterprise', 'pilot')` query extension. Closes SOC2_READINESS.md §128.6 item 6 (P0, 2026-07-17 pre-gate). Owner: compliance-officer.*

*v1.0 (2026-06-05): Created as remediation for AR-2026-Q2-01 (Q2 2026 access review latency finding). Closes SOC2_READINESS.md §65.13 AR-P1-01. Owner: compliance-officer.*
