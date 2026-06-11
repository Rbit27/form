# FORM · Decision Log v0.1

> Live record of significant decisions з context. Format: Keep it short — what + why + cost-to-reverse.
> Власник: `process-keeper` agent. Founder reviews monthly.
> Newest first.

---

## Table format

| # | Date | Decision | Owner | Why | Reverse cost |
|---|---|---|---|---|---|

---

## 2026-06-11

### DEC-045 · OQ-RL-02 + OQ-BILL-01: OVERAGE_GRACE = 500; enterprise pilots via comped subscription

- **Decision:** Two related P0 open questions resolved together:
  - **OQ-RL-02 (P0 — before quota enforcement ships):** `OVERAGE_GRACE_REQUESTS = 500` requests. Secondary warn threshold: `QUOTA_SECONDARY_WARN_PCT = 0.95`. Both stored as Cloudflare Workers environment variables, not hard-coded. Migration `0059b` adds `primary_warn_sent_at` and `secondary_warn_sent_at` dedup columns to `api_quota_usage`. New DEC-030 event `security.quota_95pct_warning` (STANDARD, 3-year retention) registered in `docs/AUDIT_LOG_SCHEMA.md §6`.
  - **OQ-BILL-01 (P0 — before enterprise pilot):** Consumer Pro trial is fixed at **14 days** — no per-tenant `trial_duration_days` on `enterprise_contracts`. Enterprise pilots use a comped active subscription: `billing_channel = 'none'`, `price_usd_cents = 0`, `status = 'active'`; pilot window governed by `enterprise_contracts.pilot_start_at / pilot_end_at` (§16). No schema change required.
- **Owner:** enterprise-architect + customer-success (OQ-RL-02); founder + enterprise-architect (OQ-BILL-01)
- **Why:**
  - *OQ-RL-02:* 500 requests is 1.0% of the Starter 50k/month quota — enough headroom to absorb a legitimate month-boundary burst without representing a material contract violation. The secondary 95% warn gives CSMs a final escalation window before the hard block fires; without it, a CSM receiving only the 80% alert has insufficient time to negotiate a quota uplift before the tenant is blocked.
  - *OQ-BILL-01:* "Trial" and "pilot" are distinct commercial concepts. A fixed 14-day consumer trial is simpler to reason about and aligns with App Store norms. Modelling enterprise pilots as comped subscriptions avoids extending the billing state machine with per-tenant trial overrides and prevents accidental production disruption if `pilot_end_at` passes without a contract signed.
- **Reverse cost:** Low (OQ-RL-02). Increasing OVERAGE_GRACE is a single env var change; decreasing it is the same. Per-tier tuning is possible post-GA (§30.7 item 11). Low (OQ-BILL-01). Introducing a `trial_duration_days` column after launch requires a schema migration and billing Worker update; no active enterprise pilots are affected before M5.

---

### DEC-044 · OQ-BILL-05: split-column approach adopted for `subscription_events` GDPR Art. 17 pseudonymization

- **Decision:** Option (b) — split column — is adopted for the `subscription_events.user_id` GDPR Art. 17 erasure FK problem. Migration `0059` makes `user_id` nullable and adds `erased_user_reference TEXT NULL` with a regex CHECK constraint. A mutual-exclusivity CHECK (`chk_sub_events_user_or_erased`) ensures every row has exactly one of the two columns. Pseudonym format: `[ERASED-{sha256(user_id + ERASURE_PSEUDONYM_SALT)}]` — a per-user keyed HMAC satisfying GDPR Art. 4(5). `ERASURE_PSEUDONYM_SALT` is a write-once Cloudflare Secret (see OQ-ERA-02 in `docs/DATA_MODEL.md §30.8`). Erasure Worker gains step SUB-1. See `docs/DATA_MODEL.md §30.2`.
- **Owner:** enterprise-architect + platform-engineer + compliance-officer
- **Why:** Option (a) — sentinel UUID — was rejected because: (a) it contaminates the `users` table with a permanent artefact that must be excluded from every query (COUNT, list, DSAR export, activation metrics); (b) a sentinel UUID is not a true GDPR pseudonym under Art. 4(5) — it is a shared opaque identifier, not an individually keyed HMAC; (c) auditors reviewing the `users` table would encounter the sentinel and require an explanation. The split-column approach confines all erasure evidence to `subscription_events`, which has a clear documented 7-year financial retention basis under Ukrainian Tax Code Art. 44 and EU VAT Directive Art. 245. The query pattern analysis (§30.2.5) confirms zero mandatory query changes for correctness — the mutual-exclusivity constraint is backward-compatible with all existing FK-using queries.
- **Reverse cost:** Medium. Reverting to sentinel UUID requires: (a) migration to make `user_id` non-nullable again; (b) removal of `erased_user_reference` column; (c) insertion of a sentinel row into `users`; (d) re-erasure of any rows already pseudonymized under the split-column approach; (e) update of `billing.user_erased` Zod schema. Cost increases with each Art. 17 erasure processed under the split-column approach.

---

### DEC-043 · OQ-PAM-02: business_justification excluded from SOC 2 auditor bulk export; redacted-sample approach adopted

- **Decision:** `business_justification` (plain-text) in `admin_jit_escalations` is NOT included in any SOC 2 auditor bulk evidence export. Auditors receive `justification_hash` (SHA-256) in DEC-030 events and may request plaintext hash-verification for specific examples during fieldwork. FORM provides a redacted sample set — sanitised to remove any PII or customer-identifying details — stored at `compliance/evidence/pam/justification-sample.md`. This resolves OQ-PAM-02 from `docs/DATA_MODEL.md §29.11` (P1 — before SOC 2 observation period start).
- **Owner:** compliance-officer
- **Why:** Bulk export of `business_justification` to an auditor creates a secondary access path to plain-text justification content that may carry customer-identifying context (e.g. tenant names, support ticket IDs). The hash-in-chain approach already satisfies CC6.2 (authorised JIT escalation) and CC6.3 (confidential data handling): the auditor can confirm the justification existed and was recorded (hash in DEC-030 HMAC chain) and can verify specific examples on request. A redacted sample demonstrates the control operates as intended without a bulk disclosure. Consistent with FORM's existing `abuse_flags.evidence_summary` redaction policy for auditor staging replica access.
- **Reverse cost:** Low before first SOC 2 observation period. If a subsequent auditor requires full plaintext bulk export during fieldwork, FORM can provide it under NDA via a time-limited `security_reviewer` grant on a staging replica — no schema or data destruction decision is required.

---

### DEC-042 · OQ-PAM-01: break-glass audit records require no dual-auth query gate; alert-on-access adopted

- **Decision:** `admin_jit_escalations` rows with `is_break_glass = true` do NOT require a separate dual-role authentication step to query. The `security_reviewer` SELECT RLS policy covers all rows including break-glass. In place of a query-time co-approval gate, FORM registers a `security.break_glass_audit_record_accessed` DEC-030 event (HIGH, 7yr) emitted whenever a query specifically targets `admin_jit_escalations WHERE is_break_glass = true` — every read is itself logged in the immutable HMAC chain. This resolves OQ-PAM-01 from `docs/DATA_MODEL.md §29.11` (P2 — decide before first break-glass activation in production).
- **Owner:** compliance-officer + security-engineer
- **Why:** A query-time approval gate for break-glass audit record access would directly undermine incident response — break-glass exists to enable rapid remediation, and requiring co-approval to read the audit trail that documents the remediation creates a catch-22 when time-critical (T+15min IR SLA, INCIDENT_RESPONSE.md R-20). Alert-on-access achieves the same transparency goal: the access event is logged in the tamper-evident HMAC chain, making any unauthorized or suspicious read of break-glass records detectable in retrospect. This is operationally equivalent to the GDPR Art. 30 records-of-processing standard: access is controlled, but audited rather than blocked.
- **Reverse cost:** Low. Switching to a dual-auth gate later is a PAM architecture addition (wrapping a Postgres query in an elevation-approval workflow) — permissible if SOC 2 auditors specifically require it, but no existing auditor framework mandates it. The `security.break_glass_audit_record_accessed` DEC-030 event registration (platform-engineer, P1) is the only implementation item created by this decision.

---

### DEC-041 · Enterprise AI coaching billing: flat per-seat model adopted; token budgets are internal governance targets only (OQ-COACH-02 resolution)

- **Decision:** Flat per-seat billing is adopted for enterprise AI coaching at launch. Token allowances (`tenant_config.coaching_seat_allowance_tokens`) are set to NULL in production — no per-seat token cap is exposed in contractual terms or the admin dashboard. The `tenant_monthly_coaching_cost` view is retained as a FORM-internal cost-monitoring tool only (never surfaced to tenants). Token budgets defined in `docs/COST_MODEL.md §33.3` (Starter/Growth: 50,000 tokens/seat/month; Enterprise: 75,000 tokens/seat/month) are internal governance thresholds for margin management, not contractual commitments. Consumption billing is explicitly deferred; re-evaluation conditions are documented in `docs/COST_MODEL.md §33.5`.
- **Owner:** enterprise-architect + finance + compliance-officer
- **Why:** OQ-COACH-02 (`DATA_MODEL.md §21.15`) asked whether per-seat token consumption should be tracked, capped, and/or billed contractually. Contractual per-seat token limits were rejected because: (a) they create a two-sided measurement problem (FORM must prove consumption to customers who cannot independently verify token counts); (b) they shift the enterprise sales conversation to technical minutiae about token burn rates rather than coaching outcomes; (c) the fully-loaded coaching COGS at expected consumption rates produces 92.5% gross margin at Starter tier and 93.3% at Enterprise — no material margin risk justifies the sales and legal complexity; (d) the internal monitoring view (`tenant_monthly_coaching_cost`) provides sufficient early-warning signal for FORM to act on consumption outliers without binding customers to token terms. Flat per-seat billing is also consistent with ASC 606 single-PO ratable revenue recognition (no variable-consideration constraint). DEC-030 events `enterprise.ai_cost_monitor_alert` and `enterprise.ai_cost_outlier_flagged` (with tenant-id-only privacy invariant) provide the operational governance layer.
- **Reverse cost:** Medium. Introducing consumption billing after launch requires: MSA amendment with active tenants, 90-day notice under §31.7.2 pricing change governance, and `tenant_config.coaching_seat_allowance_tokens` schema migration from NULL to integer. Increases with number of signed enterprise contracts. Decision can be revisited at first enterprise cohort QBR (§33.10 item 11, P2, M12).

---

### DEC-040 · Feature flag lifecycle: plan downgrade auto-disables Growth+ flags (OQ-FLAG-01); 90-day deprecation sunset policy (OQ-FLAG-02)

- **Decision:** Two decisions for the feature flag lifecycle in `docs/DATA_MODEL.md §19`:
  - **OQ-FLAG-01 (High — before M5):** When a tenant downgrades from Growth to Starter (or from Enterprise to Growth/Starter), **Option (a) is adopted:** all flags whose `feature_flag_registry.minimum_tier IN ('growth', 'enterprise')` are automatically disabled in `tenant_feature_flags` via a plan-downgrade hook wired into the contract lifecycle (`DATA_MODEL.md §16 tenant.lifecycle_status`). Each disabled flag emits a `feature_flag.disabled` DEC-030 HMAC-chained event with `reason = 'plan_downgrade'` and `set_by = 'system'`. Rows are set to `enabled = FALSE` (not deleted) so the disable is explicitly re-enabled on re-upgrade — no silent reactivation. The downgrade hook fires synchronously inside the same transaction as the `tenants.plan` column update and uses `FOR UPDATE SKIP LOCKED` to avoid TOCTOU races with concurrent feature-flag reads.
  - **OQ-FLAG-02 (Medium — before first deprecation):** The flag deprecation sunset process is formalised as: (a) minimum **90 days** between setting `deprecated_at` on a `feature_flag_registry` row and the effective disablement date; (b) customer-success communicates via a non-dismissible admin dashboard banner when a tenant has an enabled flag whose `deprecated_at IS NOT NULL` (banner text: *"Feature [flag_label] will be retired on [deprecated_at + 90d]. Contact your CSM to plan the transition."*); (c) a pg_cron job runs nightly at 03:15 UTC and soft-deletes `tenant_feature_flags` rows where the corresponding registry `deprecated_at + INTERVAL '90 days' <= now()`, emitting `feature_flag.disabled` DEC-030 events with `reason = 'deprecated'` for each row. Hard-delete is explicitly rejected — rows remain as tombstones to preserve the historical audit trail.
- **Owner:** enterprise-architect + compliance-officer
- **Why:**
  - *OQ-FLAG-01:* Without the downgrade hook, a Starter tenant retains `enabled = TRUE` in `tenant_feature_flags` for Growth-only flags after a plan downgrade. `assertFeatureEnabled()` blocks at the Worker tier gate, so there is no functional access regression — but the inconsistent stored rows mislead `form_admin` operators, pollute the audit trail, and would silently re-activate Growth features on any future re-upgrade without a deliberate admin action. This violates SOC 2 CC8.1 (change management requires explicit, logged authorisation for feature enablement). Option (a) produces clean data and a correct audit chain at negligible implementation cost.
  - *OQ-FLAG-02:* Enterprise customers depend on flagged features for production integrations (e.g. `audit.siem_export_enabled`, `admin.cohort_analytics_enabled`). Surprise removal without notice causes customer outages and erodes trust. A 90-day documented sunset policy satisfies SOC 2 CC9.2 (vendor and partner obligations) and creates a defensible change management record. Soft-delete preserves evidence of what features each tenant held and for how long — necessary for SOC 2 audits and GDPR Art. 30 records of processing.
- **Reverse cost:**
  - *OQ-FLAG-01:* Medium. Switching to Option (b) — rely solely on the Worker tier gate — requires: removing the downgrade hook, updating the SOC 2 CC8.1 narrative (losing the "explicit re-enable" guarantee), and notifying any tenant already downgraded under this rule if their flag rows were set to `enabled = FALSE`. Permissible before the first downgrade event is processed; increases in cost with each downgrade processed under this rule.
  - *OQ-FLAG-02:* Low before any 90-day sunset notice is dispatched. Once a sunset banner is shown to a tenant, the date is a commitment — cannot be shortened without a retroactive notification and amended CSM communication.

---

### DEC-039 · SCIM group role conflict: higher-privilege-wins confirmed (OQ-SSO-19.1); group cap 500/tenant (OQ-SSO-19.3)

- **Decision:** Two enterprise-architect decisions to close P0/P1 open questions in `docs/SSO_SCIM_IMPLEMENTATION.md §19` (SCIM Groups Sync & Group-Based Role Mapping):
  - **OQ-SSO-19.1 (P0 — founder sign-off):** When a user belongs to multiple SCIM groups with conflicting role mappings, **higher privilege wins.** The `group_member_effective_role` view uses `MAX()` aggregation with ordinal scoring (`tenant_owner` > `tenant_admin` > `coach` > `member` > `viewer`). `tenant_owner` is explicitly excluded from group assignment — no group mapping may elevate to `tenant_owner`; that role is assigned exclusively at tenant creation or by direct admin action. Rationale (§19.3.2): enterprise IT assigns users to groups by job function, not minimum access level. A user in both "All Staff" (member) and "FORM Coaches" (admin) holds a specialist function and should receive the admin role. Lowest-privilege-wins would require IT to manually exclude specialists from broad groups — a fragile configuration that breaks with standard Active Directory nesting patterns. The decision is already encoded in the `group_member_effective_role` view (§19.4.4); this entry is the required governance record.
  - **OQ-SSO-19.3 (P1 — enterprise-architect decision):** Default maximum groups per tenant is **500.** Configurable per enterprise agreement via `tenants.feature_limits JSONB`. Rationale: 500 covers a 10,000-seat org with granular team groups; exceeding 500 indicates a misconfigured IdP push filter (pushing all org groups rather than FORM-specific groups). B-tree index on `(tenant_id, scim_group_id)` in `tenant_groups` is performant well beyond 500 rows. The cap is already enforced in `resolveJitGroups()` (§19.6.2) and in SCIM Group POST validation.
- **Owner:** founder + enterprise-architect
- **Why:** OQ-SSO-19.1 was a **P0 M4 blocker** in the §19.10 implementation checklist — the feature cannot ship without the founder's decision logged. OQ-SSO-19.3 was a P1 item with a clear recommendation already written in §19.3.4 and §19.5.2. Both items were deferred from v1.1 (2026-05-30) authoring pass; this closes both as a patch.
- **Reverse cost:** Medium (OQ-SSO-19.1). Switching to lowest-privilege-wins requires: (a) one-line view change `MAX()` → `MIN()` in `group_member_effective_role`, (b) §19.3.2 rationale rewrite, (c) notifying enterprise customers whose role configuration relies on higher-privilege resolution. Low (OQ-SSO-19.3): any cap change only requires updating `tenants.feature_limits` default and `resolveJitGroups()` constant; tenants already at 500 groups need data migration.

---

## 2026-06-10

### DEC-038 · Pricing exception approval procedure formalised in COST_MODEL.md §32; closes §31.9 P0 item 3

- **Decision:** The enterprise pricing exception approval procedure is now documented as `docs/COST_MODEL.md §32`. The procedure covers: (a) when exception approval is required (any effective discount exceeding the pre-approved –32.5% maximum); (b) approval authority by band (founder-only for –32.5% to –40%; founder + investor lead for –40% to contractual floor); (c) the seven-step process from approval request through DEC-030 event emission to quote send; (d) floor enforcement protocol for below-floor requests, including CRITICAL-severity DEC-030 event even for denied attempts; (e) pre-CRM record-keeping structure. This is the pre-CRM "written procedure in `docs/DECISION_LOG.md`" required by §31.9 item 3 — the full procedure lives in §32, and this entry serves as the formal DEC record. Closes §31.9 item 3 (P0, M4): two prior P0 items — DEC-030 event type registration (v3.25.1) and calculator price floor enforcement (v3.27.1) — were already closed.
- **Owner:** founder + compliance-officer
- **Why:** Without a documented procedure, the first non-standard discount request has no defined approval chain, no DEC-030 emission trigger, and no audit trail — creating a SOC 2 CC5.2 gap (no policy for pricing commitments). The procedure must exist before the first enterprise quote is sent, not after. DEC-030 event `enterprise.pricing_exception_approved` (HIGH, 7yr) and `enterprise.price_floor_override_requested` (CRITICAL, 7yr) were registered in AUDIT_LOG_SCHEMA.md at v3.25.1; the pricing calculator floor enforcement was added at v3.27.1; the missing piece was the human-facing approval runbook.
- **Reverse cost:** Low (procedure can be revised; the DEC-030 event schema is more costly to change — any field rename requires schema migration and re-audit). The contractual floor levels themselves ($6.00/$4.50/$4.00/seat) cannot be changed without notifying active enterprise customers per §31.7.2.

---

## 2026-06-09

### DEC-037 · privacy.html authored as form.coach/privacy URL target; PRE-01 advances to 🟡 Authored

- **Decision:** Create `privacy.html` as the deployable HTML page at `form.coach/privacy`, rendering all 14 sections of `docs/PRIVACY_POLICY.md` v0.1.0-draft in FORM brand style. The page carries a prominent "DRAFT — Pending Counsel Review" banner and is not legally effective until counsel sign-off. This advances SOC 2 gap PRE-01 / P-GAP-001 from 🔴 Open to 🟡 Authored. Closes to 🟢 on: (a) outside counsel reviews the policy across EU/UA/US jurisdictions; (b) page is deployed to Cloudflare Pages at form.coach/privacy; (c) counsel sign-off is filed as CC2-E-001 in `compliance/cc2/`.
- **Owner:** compliance-officer
- **Why:** PRE-01 was the highest remaining critical gap after PRE-16 (cookie banner) was advanced to 🟡 Authored in v3.8.2. The privacy policy URL must exist and be stable before any enterprise pilot data flows and before the SOC 2 observation period starts. Creating the HTML page now unblocks the form.coach/privacy link required in the cookie consent banner (PRE-16 §74.3) even while counsel review is pending — the draft banner makes the pre-legal status transparent to any user who visits the URL.
- **Reverse cost:** Low (page can be updated or retracted before counsel sign-off at negligible cost; the legal commitment attaches at counsel sign-off and publication, not at HTML authoring)

---

### DEC-036 · Art. 9 health data: no-grace period is an absolute invariant during enterprise off-boarding

- **Decision:** During enterprise tenant off-boarding, Art. 9 health data (`keypoints_enc`, `user_health_profiles`, `coaching_turns`, `meal_logs`, `body_metrics`) is hard-deleted immediately with no grace period. This rule is an absolute invariant — it cannot be overridden by an enterprise MSA, even if the contract includes a mutual notice period or a DPA clause purporting to allow a recovery window. Enterprise customers who require a data export of individual health data must instruct their employees to exercise the right to data portability (DATA_MODEL.md §12.5) independently and before the off-boarding trigger date. Any enterprise MSA clause requesting a health-data recovery window must be refused at contract review stage. Resolves OQ-OFB-03 (DATA_MODEL.md §25.8).
- **Owner:** compliance-officer + legal
- **Why:** GDPR Recital 51 identifies health data as requiring a higher level of protection. A grace period for Art. 9 data after the employer relationship ends creates unnecessary exposure with no legitimate recovery justification; the 30-day soft-delete window (DATA_MODEL.md §12.1) exists for accidental individual-user deletion, not for employer off-boarding scenarios. If the first enterprise contract is signed before this is logged, the legal baseline is ambiguous — the decision must precede any contract signature.
- **Reverse cost:** Cannot reverse without renegotiating the privacy policy, DPA template, and notifying existing enterprise customers (constitutive data handling promise)

---

### DEC-035 · API key expiry: soft enforcement (age alerts + CSM escalation) not automatic revocation

- **Decision:** The `expires_at` field on `tenant_api_keys` (DATA_MODEL.md §26.4) is enforced as a soft control — age-alert UI warnings (amber at 365 days, red at 730 days) plus CSM escalation — not as hard automatic revocation at the `expires_at` timestamp. This default is explicitly provisional: if any key exceeds 365 days without rotation at the first SOC 2 observation period start, the soft control is demonstrably not operating effectively and hard enforcement must be added before the observation period ends. Resolves OQ-SSO-26.2 (DATA_MODEL.md §26.12).
- **Owner:** enterprise-architect + compliance-officer
- **Why:** Hard automatic revocation at 365 days provides a stronger SOC 2 CC6.4 posture (system control vs. procedural control). However, enterprise customers with CI/CD pipelines on dynamic-IP runners (GitHub Actions, Buildkite cloud) cannot use static IP allowlists and may not notice key age until revocation silently breaks their integration. Soft enforcement with CSM escalation avoids unplanned production outages for early enterprise pilots while providing documented evidence that the policy is operating. The provisional nature is a hard commitment: if evidence at observation-period start shows the soft control is insufficient, hard enforcement ships immediately.
- **Reverse cost:** Low initially (provisionally soft); escalates to High if soft control is retained past observation period without documented operating effectiveness evidence

---

## 2026-06-01

### DEC-034 · D7 activation metric definition: full coaching session with CV enabled within 7 days

- **Decision:** FORM's primary pre-Series A product health signal is the D7 activation rate — the percentage of registered users who complete at least one full coaching session with CV pose feedback enabled within 7 days of registration. Target: ACT-SLO-01 ≥ 40% [ESTIMATE]. Activation tied to CV (not to a softer proxy such as app open or exercise logged) because CV feedback is the core value proposition that drives long-term retention. Documented in OBSERVABILITY.md §25.2 (DEC-034).
- **Owner:** product-strategist + devops-lead
- **Why:** Activation definitions anchored to the core wedge (DEC-006: camera-coaching) create a higher-quality cohort signal than soft metrics. Enterprise customers evaluate adoption by this metric in the 30/60/90-day review (ENTERPRISE.md §Implementation timeline). Changing the definition mid-cohort invalidates historical comparisons, so the definition must be fixed now and version-tracked in analytics schema.
- **Reverse cost:** Medium (changing the activation definition invalidates historical cohort comparisons and requires a schema migration on `victor_quality_daily` and PostHog cohort definitions; permissible only with a major analytics version bump and re-baseline of ACT-SLO-01 target)

---

## 2026-05-23

### DEC-033 · Blog milestone post-100 reached through sports-science only (no clinical-safety-blocked content)

- **Decision:** Posts 98–100 complete the first century of FORM content strictly from sports-science corpus. Posts 73, 74, 75, 76, 87, 88 remain blocked or deferred pending correct clinical-safety review; we do not ship those to hit a number.
- **Owner:** content-strategist + clinical-safety
- **Why:** Content quality floor > content velocity. Having 100 posts with no clinical-safety violations is a stronger signal than 100 posts that include shortcuts on food/body topics.
- **Reverse cost:** Low (deferred posts can ship after proper CS review)

### DEC-032 · Law enforcement no-backdoor policy formalized (INCIDENT_RESPONSE R-13)

- **Decision:** FORM will never install programmatic surveillance hooks, passive intercept mechanisms, or backdoor access at any government entity's request. All law enforcement data access must follow formal compelled legal process; founder sign-off required before any disclosure; Art. 9 health data is never voluntarily produced.
- **Owner:** security-engineer + compliance-officer + founder
- **Why:** Enterprise trust requires unconditional commitment. Any backdoor weakens security posture for all tenants. Anchored in PRIVACY_POLICY.md §6.4 and ENTERPRISE.md no-backdoor clause. First Government Transparency Report commitment signals seriousness to enterprise DPOs.
- **Reverse cost:** Cannot reverse (constitutive promise; reversing requires public announcement, customer notification, and re-audit)

---

## 2026-05-16

### DEC-031 · Agent team expanded from 14 → 24 agents

- **Decision:** Розширити команду AI-агентів з 14 (v0.1) до 24, додавши 10 нових ролей: `enterprise-architect`, `compliance-officer`, `devops-lead`, `data-engineer`, `security-engineer`, `ml-engineer`, `qa-lead`, `qa-walker`, `customer-success`, `product-manager`.
- **Owner:** process-keeper + founder
- **Why:** Початкові 14 агентів покривали product/design/content/growth. Enterprise-tier і technical maturity вимагають ролей з compliance authority (SOC 2, GDPR), security threat modeling, infrastructure-as-code, і систематичного QA після кожного ship-у. `qa-walker` runs mentally after every meaningful change — це підлога якості, не опція.
- **Reverse cost:** Low (агенти — текстові файли; можна remove/merge у будь-який момент)

---

### DEC-030 · Audit log: append-only HMAC-chained, 7-year retention для admin events, support actions auto-notify tenant

- **Decision:** Кожна privileged action логуєтьcz у `audit_log` table з HMAC chain (tamper-evident). Retention 7 років для tenant/financial events, 3 роки для auth, 30 днів для high-volume reads. `support.*` actions (FORM employee touches tenant data) тригерять email-notification до tenant admin within 24h. Break-glass debug access requires 2-person approval + time-bounded role.
- **Owner:** compliance-officer + security-engineer
- **Why:** SOC 2 Type II requires immutable audit trail. GDPR Art. 30 requires processing records. Trust differentiation: most B2B SaaS логує "що ми робили з вашими даними" в чорну скриню; ми робимо це візибл і authenticated.
- **Reverse cost:** High (schema і retention policies формують backbone compliance posture; зміна після audit = повторний audit)

---

## 2026-05-15

### DEC-029 · Growth loops: sharing назовні, no in-app social, clinical-safety gate на весь share-контент

- **Decision:** Всі viral/growth mechanics у FORM — це sharing на зовнішні платформи (Instagram, Twitter, Telegram). Жодного in-app social (leaderboards, друзі, порівняння). Весь share-контент проходить clinical-safety checklist перед ship: blocked — вага тіла, калорії, before/after, порівняння між юзерами. Allowed — form score, lift name, streak count, Victor cue.
- **Owner:** growth-lead + clinical-safety
- **Why:** DEC-002 (no social features) і клінічна безпека вимагають жорсткого розмежування між «шеримо досягнення» і «порівнюємо тіла». Зовнішній sharing дає organic acquisition без токсичного social comparison всередині продукту.
- **Reverse cost:** Low-Medium (додати in-app social — рефакторинг, але можливо у v2.0 з повним clinical review)

---

## 2026-05-14

### DEC-028 · Hiring platform priority: Djinni-first для FE, LinkedIn-first для Design Lead

- **Decision:** Founding Engineer (iOS) постимо спочатку на Djinni + DOU + Wellfound; Design Lead — спочатку LinkedIn + Wellfound + Djinni. Бюджет: $0 на старті, boost (до $150) тільки якщо немає відгуків за тиждень.
- **Owner:** founder (George)
- **Why:** iOS розробники в Україні концентровані на Djinni/DOU; дизайнери — на LinkedIn. Wellfound потрібен обом як equity-signal платформа. Починати безкоштовно — зберігаємо runway. Paid boost тільки як fallback.
- **Reverse cost:** Low (платформи незалежні; можна додати/прибрати будь-яку без наслідків)

### DEC-027 · Assumptions Register як передумова до research sprint

- **Decision:** Зафіксувати всі 30 product/market/tech assumptions явно ДО проведення інтерв'ю
- **Owner:** research-lead + product-strategist
- **Why:** RESEARCH.md описує як проводити інтерв'ю, але не що ми перевіряємо. Без explicit assumptions важко відстежити що змінилося після research і що треба pivotувати.
- **Reverse cost:** Low (документ, не продуктове рішення)

### DEC-026 · PostHog EU Cloud як analytics platform

- **Decision:** PostHog (EU Cloud, Frankfurt) для всієї аналітики продукту
- **Owner:** platform-engineer
- **Why:** Cost (self-hostable fallback), privacy (GDPR EU-resident data), open-source (no vendor lock-in), iOS SDK зрілий. Mixpanel/Amplitude дорожчі і US-hosted за замовчуванням.
- **Reverse cost:** Medium (65-event taxonomy і dashboards прив'язані до PostHog schema; міграція ≈ 2 спринти)

### DEC-025 · FIRST_30_DAYS як primary execution sequencing framework

- **Decision:** docs/FIRST_30_DAYS.md — єдина точка входу для першого місяця виконання
- **Owner:** process-keeper + founder
- **Why:** 40+ стратегічних документів без пріоритизації = paralysis. FIRST_30_DAYS синтезує RESEARCH + INVESTOR_OUTREACH + DEPLOYMENT + HIRING в послідовність тижнів.
- **Reverse cost:** Low (framework, не зобов'язання; адаптується до реальних подій)

### DEC-024 · Twitter thread у Ukrainian або English?
- **Decision:** English thread first, UA версія в наступний день
- **Owner:** marketing-lead
- **Why:** Founder + project mostly hiring internationally. UA audience smaller for launch reach. UA thread sent as follow-up to UA community.
- **Reverse cost:** Low (можемо змінити порядок наступних threads)

### DEC-023 · Geo-pricing $9 UA / $19 Western
- **Decision:** Two-tier geo-priced
- **Owner:** marketing-lead + growth-lead
- **Why:** Western $19 needed для unit econ. UA $19 = unaffordable у local context. $9 UA preserves accessibility з reasonable margin.
- **Reverse cost:** Medium (changing prices on subscribers requires careful communication)

### DEC-022 · v0.4.0 MINOR bump
- **Decision:** Bump to 0.4.0 as milestone marker
- **Owner:** founder
- **Why:** End of strategic-thinking documentation phase, start of execution-ready. Visible markers help.
- **Reverse cost:** Cannot reverse (versioning is monotonic)

### DEC-021 · No Russian language UI у v1.0
- **Decision:** Не shipping Russian-language version
- **Owner:** founder
- **Why:** UA-speaking lifters can use UA UI. RF market not target (ethical + practical). Diaspora can use English UI.
- **Reverse cost:** Low (could add later if context changes)

### DEC-020 · Founding Engineer comp range $90-130k
- **Decision:** Above Ukrainian market base, plus 1.5-3% equity
- **Owner:** founder
- **Why:** Top-of-market UA pulls best people. Equity range competitive but not Silicon Valley level (we're not raising $5M+).
- **Reverse cost:** Low (range, не set offer)

### DEC-019 · Cancel-flow 3 кліки maximum
- **Decision:** No retention-hoops, even one offer
- **Owner:** ux-flow + founder
- **Why:** Trust > short-term retention. Bad-cancel UX hurts brand perception more than saves customer.
- **Reverse cost:** Medium (changing established cancel-flow can shock users)

### DEC-018 · ED-screening non-skippable
- **Decision:** Required step at onboarding для anyone using nutrition features
- **Owner:** clinical-safety (HARD VETO)
- **Why:** Risk of ED-trigger у fitness app users is real. Soft-mode без friction is rare; screening prevents harm.
- **Reverse cost:** Cannot reverse (safety floor)

### DEC-017 · 8 base lifts CV-coaching scope
- **Decision:** Camera coaches only 8 base movements у v1.0
- **Owner:** platform-engineer + sports-scientist
- **Why:** Honest about CV-quality. Adding more without proper accuracy hurts trust. 8 covers 80% of serious lifter's program.
- **Reverse cost:** Low (can add more lifts incrementally)

### DEC-016 · No free tier
- **Decision:** Trial only, no permanent free tier
- **Owner:** growth-lead
- **Why:** MyFitnessPal economics убивають LTV для coaching products. Trial removes commitment barrier while preserving signal.
- **Reverse cost:** Medium (introducing free tier later means restructuring monetization)

### DEC-015 · iOS first, Android phase 2
- **Decision:** Ship iOS only у v1.0, Android Q1 2027
- **Owner:** platform-engineer + product-strategist
- **Why:** Team focus, CV-pipeline reliability. Android adds variance що hurts launch quality.
- **Reverse cost:** Cannot meaningfully reverse без major rework

### DEC-014 · Bullshit-detector у tone (Victor)
- **Decision:** Cannot say things like «Тіло пам'ятає все», «Дисципліна — свобода», «Just do it»
- **Owner:** brand-voice + clinical-safety
- **Why:** Marketing-poster tone alienates target audience (serious lifters). Editorial tone differentiates.
- **Reverse cost:** Low (can re-train tone, але redoing copy = effort)

### DEC-013 · Streak with grace
- **Decision:** Streak ламається після 2 misses, не 1
- **Owner:** clinical-safety + growth-lead
- **Why:** «Не пропускай. Ніколи.» = perfectionist trap. Single-miss break = burnout machine (Duolingo pattern).
- **Reverse cost:** Low (changing breakage rule = small update)

### DEC-012 · Cloud-routine з 2-hour cadence
- **Decision:** Sonnet 4.6 cloud-agent iterates every 2 hours during build phase
- **Owner:** founder
- **Why:** Maintains forward motion. Founder reviews changes. ~12 iterations/day → significant volume of work.
- **Reverse cost:** Low (can pause/cancel routine)

### DEC-011 · Versioning з MAJOR.MINOR.PATCH від v0.2
- **Decision:** Adopt semver-light for all releases
- **Owner:** founder
- **Why:** Engineering discipline. Visible progress. Enables tagged releases on GitHub.
- **Reverse cost:** Cannot reverse (versioning is monotonic)

### DEC-010 · Public GitHub repo
- **Decision:** Open repository (Rbit27/form) як build-in-public
- **Owner:** founder
- **Why:** Trust signal (we hide nothing). Recruiting signal (devs see how we think). Investor signal (transparency).
- **Reverse cost:** High (closing public repo = bad look)

---

## 2026-05-13

### DEC-009 · 14 specialized AI agents як team structure
- **Decision:** Build operationally на agent-team model
- **Owner:** founder
- **Why:** Cost-effective. Specialized expertise w/o hiring. Forces clear role separation.
- **Reverse cost:** Cannot reverse (operating model defines us)

### DEC-008 · Voice persona «Victor»
- **Decision:** Single named persona з 4 calibration tones
- **Owner:** brand-voice + brand-system
- **Why:** Brand differentiation. Memorable. Calibratable.
- **Reverse cost:** High (re-branding voice = months of work)

### DEC-007 · Editorial-brutalist design aesthetic
- **Decision:** Fraunces + Manrope + JBMono. Warm-black + cream + acid lime. Editorial layouts.
- **Owner:** design-craft + brand-system
- **Why:** Stand out з saturated AI-fitness market dominated by purple gradients і generic SaaS look.
- **Reverse cost:** High (re-design = major effort)

### DEC-006 · Camera-coaching як wedge
- **Decision:** Real-time pose-analysis primary differentiator
- **Owner:** product-strategist + platform-engineer
- **Why:** Nobody else does this well. Technical bet, but defensible if works.
- **Reverse cost:** Medium-high (if CV doesn't work, pivot to AI-program-coach exists)

### DEC-005 · UA-built, Western-targeted
- **Decision:** Build in Kyiv, primary markets EU/UK/US
- **Owner:** founder
- **Why:** UA tech cost-base advantage. Western market revenue potential. Geographic arbitrage.
- **Reverse cost:** Low strategic (можемо move team), but cultural cost real

### DEC-004 · Target persona = Дмитро (self-coached intermediate)
- **Decision:** Primary user — male 25-40, 1-3+ years training, no PT, has wearable
- **Owner:** product-strategist + research-lead
- **Why:** Most likely to pay $19, sustainable churn, addressable through Reddit/YouTube
- **Reverse cost:** Low (can shift focus if data says otherwise)

### DEC-003 · Closed beta перед public launch
- **Decision:** 200-user TestFlight beta Q3 2026 перед App Store
- **Owner:** founder + growth-lead
- **Why:** Catches P0 bugs. Generates real metrics. Trust signal у launch communications.
- **Reverse cost:** Cannot reverse (timing locked)

---

## 2026-05-12

### DEC-002 · No social features у v1.0
- **Decision:** Не shipping social, leaderboards, friend-system
- **Owner:** product-strategist
- **Why:** Social fitness apps trigger negative comparison. Retention-harming. Adds complexity без core-value increase.
- **Reverse cost:** Low (можемо add later if signals exist)

### DEC-001 · FORM brand name + .coach domain
- **Decision:** Commit до «FORM» as brand
- **Owner:** founder + brand-system
- **Why:** 4-letter, double-meaning (body form + technique form), pronounceable UA/EN, available .coach TLD
- **Reverse cost:** Very high (renaming established product = years of recovery)

---

## Format guidelines

- **One decision per row.** Never bundle.
- **Owner mandatory.** Who can be cited if questioned.
- **"Why" in plain language.** Not «strategic alignment»; the actual reason.
- **Reverse cost honest.** «Low» / «Medium» / «High» — what does undoing cost?

### Don't log:

- Daily micro-decisions (what color text to use)
- Personal preferences без impact
- Decisions still у flux (log when final)

### Do log:

- Anything affecting roadmap
- Anything affecting safety or ethics
- Anything affecting hiring or org
- Anything that future-you might question

---

**v0.1 · травень 2026 · live document, append-only**
