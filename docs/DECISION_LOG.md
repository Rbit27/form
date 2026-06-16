# FORM · Decision Log v0.1

> Live record of significant decisions з context. Format: Keep it short — what + why + cost-to-reverse.
> Власник: `process-keeper` agent. Founder reviews monthly.
> Newest first.

---

## Table format

| # | Date | Decision | Owner | Why | Reverse cost |
|---|---|---|---|---|---|

---

## 2026-06-15

### DEC-061 · OQ-OFB-02: EU-region R2 bucket routing for enterprise off-boarding data egress adopted

- **Decision:** EU-domiciled enterprise tenants receive a **dedicated EU-jurisdiction Cloudflare R2 bucket** (`form-offboarding-exports-eu`) for all off-boarding egress packages. Routing is computed at row-creation time by `resolveEgressBucket()` in `apps/offboarding-worker`, keyed on `tenants.data_region`: `eu-central-1` and `eu-west-1` → `form-offboarding-exports-eu`; `us-east-1` → `form-offboarding-exports` (unchanged). Correctness is enforced by `chk_r2_bucket_region_consistency` CHECK constraint added in migration `0075_offboarding_r2_routing.sql`, making misrouting a hard schema error rather than an application-layer risk. The DEC-030 HMAC-chain event `enterprise.data_export_completed` is extended with `data_region`, `r2_bucket`, and `is_eu_region` fields; invariant OFB-REGION-01 requires `is_eu_region: true → r2_bucket = 'form-offboarding-exports-eu'` — violation emits HTTP 422 and P1 PagerDuty. Full specification in `docs/DATA_MODEL.md §36`. Closes OQ-OFB-02 (P0 — before any EU enterprise tenant off-boards, from §25.9).
- **Owner:** enterprise-architect + compliance-officer + devops-lead + legal
- **Why:** (1) **GDPR Chapter V compliance.** Personal data of EU data subjects must not leave the EEA without an Art. 44–49 transfer mechanism. Using a US-East R2 bucket for EU tenant egress packages requires Standard Contractual Clauses (SCCs) plus a Transfer Impact Assessment (TIA) under Schrems II — a maintenance burden that grows with each EU customer onboarded and each annual DPA renewal. An EU-jurisdiction bucket eliminates the transfer entirely: GDPR Chapter V ceases to apply, the TIA is not required, and EU DPOs can attest to intra-EEA processing on a factual basis rather than a contractual one. (2) **FISA §702 / CLOUD Act exposure removed.** Cloudflare US-East storage is subject to US government access requests under FISA §702 and the CLOUD Act. EU enterprise customers' legal and DPO teams routinely flag this in security questionnaires; storing egress packages in a Cloudflare EU-jurisdiction bucket (operated by Cloudflare Ireland Ltd, an existing SUBPROCESSORS.md entry) removes this exposure without requiring a new processor relationship. (3) **Options B and C rejected.** Option B (SCCs + single US bucket): requires per-customer TIA maintenance, creates DPO approval friction during contract negotiation, and does not eliminate FISA §702 exposure — the SCC mechanism accepts the legal risk rather than removing it. Option C (EU bucket + SCCs as fallback): same TIA maintenance burden plus DPA Annex B complexity (two transfer mechanisms to maintain); adds no material advantage over the clean Option A routing. (4) **Enforcement via schema constraint.** The CHECK constraint converts a potential misconfiguration into a Postgres hard error at INSERT time — no silent misrouting possible even if `resolveEgressBucket()` logic is accidentally changed. OFB-REGION-01 adds a HMAC-chain double-check: if a row somehow bypassed the constraint, the chain event payload triggers a P1 alert before any data leaves FORM's infrastructure boundary.
- **Reverse cost:** Low–Medium. Reverting to single-bucket (US-East for all tenants) requires: (a) removing `data_region` column and `chk_r2_bucket_region_consistency` CHECK constraint via a new migration; (b) introducing SCCs and TIAs into all EU enterprise DPAs (requires legal review and renegotiation with signed customers); (c) migrating any existing `form-offboarding-exports-eu` objects to `form-offboarding-exports`; (d) removing `is_eu_region` and `r2_bucket` from `enterprise.data_export_completed` chain event (backwards-compatible drop, but requires auditor notification if evidence artefact OFB-E-005 already filed); (e) new DECISION_LOG entry + compliance-officer + legal sign-off. No change to the DEC-030 HMAC chain structure itself. Re-evaluation trigger: if Cloudflare changes EU jurisdiction guarantees for `form-offboarding-exports-eu` in a GDPR-incompatible way, evaluate migration to AWS S3 `eu-central-1` with Object Lock as the leading alternative.

---

### DEC-060 · OQ-SSO-24.1 + OQ-SSO-24.2: `pam-db-proxy` — Supabase Edge Function + direct Postgres session connection (port 5432) adopted

- **Decision:** `pam-db-proxy` is implemented as a **Supabase Edge Function** (OQ-SSO-24.1); it connects to Postgres using the **direct session-mode connection string (port 5432)** — not the PgBouncer transaction-mode pooler (port 6543) — eliminating the `SET ROLE form_admin` session-affinity risk and bypassing PgBouncer entirely (OQ-SSO-24.2). `SUPABASE_PAM_DB_URL` is stored as a Supabase Edge Function secret (distinct from `SUPABASE_SERVICE_ROLE_KEY`). No Supabase plan upgrade is required — max 5–6 concurrent PAM direct connections is well within the Pro plan 60-connection ceiling. Full specification in `docs/SSO_SCIM_IMPLEMENTATION.md §31`. Closes OQ-SSO-24.1 (P1, Before M4 — pam-db-proxy placement) and OQ-SSO-24.2 (P0, Before M4 — PgBouncer session mode); both from §24.9.
- **Owner:** security-engineer + platform-engineer + enterprise-architect
- **Why:** (1) **OQ-SSO-24.1 — Supabase Edge Function over Cloudflare Worker + Hyperdrive:** Hyperdrive uses transaction-mode pooling by default; `SET ROLE form_admin` is session-scoped and not guaranteed to persist across connection checkouts in transaction mode — this is a PgBouncer architectural constraint, not a configuration gap. Additionally: `form_admin` credentials stored in Cloudflare Workers Secrets would cross the Supabase trust boundary on every new pool connection; intra-VPC Edge Function → Postgres latency is < 5 ms vs. 20–50 ms for Cloudflare → Hyperdrive → Supabase; no third-party intermediary in the privileged data path. (2) **OQ-SSO-24.2 — Direct port 5432 connection:** Bypasses PgBouncer entirely. `SET ROLE form_admin` is reliable by design. `SUPABASE_PAM_DB_URL` (port 5432) is available on all Supabase paid plans with no additional pool configuration. Connection ceiling analysis: ≤ 5 PAM-eligible engineers (§24.4.1 named list cap), 1 active session per admin (KV enforcement), break-glass uses a separate `BREAK_GLASS_DB_URL` — max simultaneous PAM connections is 5–6, within Pro plan 60-connection limit. `RESET ROLE` + `RESET app.pam_session_id` called in a TypeScript `finally` block; connection closed after each PAM query (not pooled — audit isolation + Edge Function anti-pattern for persistent pools).
- **Reverse cost:** Low. Changing from Edge Function to Cloudflare Worker + Hyperdrive requires: (a) confirming Hyperdrive session-mode GA availability; (b) moving `form_admin` credential from Supabase secrets to Cloudflare Workers Secrets; (c) security-engineer threat model update for the cross-provider privileged connection; (d) new DECISION_LOG entry. Changing from port 5432 to a dedicated PgBouncer session-mode pool (if team exceeds 10 PAM-eligible engineers) requires: (a) provisioning a Supabase dedicated pooler or standalone PgBouncer instance; (b) updating `SUPABASE_PAM_DB_URL` to the pool endpoint; (c) confirming `SET ROLE` session-affinity with the new pooler. No schema or DEC-030 event changes required for either reversal.

---

## 2026-06-14

### DEC-059 · OQ-MOBILE-03: Mobile SSO browser mode — system browser mandated, WebView prohibited

- **Decision:** FORM's React Native mobile application MUST use `WebBrowser.openAuthSessionAsync()` from `expo-web-browser` for all enterprise SSO outbound redirects (`ASWebAuthenticationSession` on iOS; Chrome Custom Tabs on Android). Embedding a `<WebView>` component for any SSO authentication step is prohibited. Universal Links (iOS) and App Links (Android) are preferred over `form://` custom scheme for the SSO callback redirect URI. Azure AD B2C tenants using SAML custom policies must be migrated to OIDC PKCE mode before mobile SSO is enabled for their tenant. Two DEC-030 HMAC-chained events added: `sso.mobile_browser_mode_set` (STANDARD, 7yr — config state, no PII) and `sso.mobile_webview_blocked` (HIGH, 7yr — emitted only if violation detected; firing is itself an incident). Alert AL-SSO-WEB-01 (P1, PagerDuty `form-security`) fires on any `sso.mobile_webview_blocked` event — zero-fire is the expected steady state. CI guard: ESLint `no-restricted-imports` + `no-webview-in-sso.test.ts` grep test block any PR introducing WebView into `apps/mobile/src/auth/`. Full specification in `docs/SSO_SCIM_IMPLEMENTATION.md §30`. Closes OQ-MOBILE-03 (`docs/OBSERVABILITY.md §28.13` — P1, before first enterprise SSO customer M10). Updates OBSERVABILITY §28.13 OQ-MOBILE-03 row to 🟢 Resolved.
- **Owner:** security-engineer + platform-engineer + enterprise-architect
- **Why:** (1) **RFC 9700 §4.1 (OAuth 2.0 Security BCP).** "The authorization server MUST NOT allow the embedding of its authorization endpoint in embedded user agents." WebView violates this by design; system browser satisfies it. Enterprise customers' security teams audit mobile OAuth flows for BCP compliance — a WebView finding would be a Critical in every security assessment. (2) **Process isolation.** `ASWebAuthenticationSession` runs in a separate OS process; the host app cannot access URL, cookies, DOM, or keystrokes during authentication. WebView runs in the host app's process, exposing `shouldOverrideUrlLoading()` / `onReceivedSslError()` interception to any malicious SDK update. (3) **TLS bypass.** On Android, WebView's `onReceivedSslError()` callback can be overridden to silently accept invalid certificates, disabling HTTPS for the SSO flow entirely. System browsers enforce TLS at the OS level with no override hook. (4) **IdP compatibility.** All four FORM-supported IdP families (Okta, Entra, Google Workspace, generic SAML) are compatible with system browser — the SAML ACS callback is server-side, so the mobile client only needs to open the IdP login URL and receive the final deep-link redirect. Azure AD B2C is the one ⚠️ case, resolved by OIDC PKCE mode (no WebView needed). (5) **SOC 2 posture.** CC6.1 evidence for mobile credential handling is substantially cleaner with a documented system-browser mandate + DEC-030 chain record than with a WebView approach that would require auditor justification.
- **Reverse cost:** Low. Reverting to WebView would require: (1) removing the ESLint rule and CI test; (2) a new DECISION_LOG entry with a full security-engineer threat model; (3) founder + compliance-officer sign-off; (4) updating `docs/SSO_SCIM_IMPLEMENTATION.md §30.3` to remove the prohibition. The IdP compatibility constraint is manageable — the only affected case (Azure AD B2C SAML) has a fully documented OIDC PKCE workaround. No schema or DEC-030 event changes are required to revert.

---

### DEC-057 · OQ-ANOM-01: CR-01 Analytics Engine per-tenant aggregation — no architecture change required

- **Decision:** The CR-01 brute-force detection query (`GROUP BY blob4, blob2` — tenant_id, actor_ip_subnet) in `docs/OBSERVABILITY.md §27.3` and §34.3 is confirmed to provide per-tenant isolation with no cross-tenant aggregation blind spot. Analytics Engine GROUP BY evaluation produces independent per-bucket aggregates; each `(tenant_id, actor_ip_subnet)` pair is evaluated against the `HAVING SUM(_sample_interval) >= 10` threshold independently. A high-volume tenant's auth failures cannot suppress or mask a CR-01 alert for a low-volume tenant because the two tenants' events occupy distinct `(blob4, blob2)` buckets. Symmetric ingestion pipeline lag (bounded by SIEM-SLO-BRIDGE-01 ≤ 5 min) does not create per-tenant fairness disparities — all tenants experience the same maximum lag. No query change, schema change, or per-tenant fairness mechanism is required. A mandatory staging verification test (`src/tests/siem/cr01-per-tenant-isolation.test.ts` per `docs/OBSERVABILITY.md §44.3.4`) is required as a CI gate before the M5 SIEM alert activation deployment. On pass, `system.siem_calibration_verified` DEC-030 event (STANDARD, 3yr) is emitted and CALIB-E-002 evidence artefact is filed. Re-evaluation trigger: M9 ClickHouse migration (§34.7) — re-run the §44.3.4 verification against the new MergeTree CR-01 implementation before reactivating alerts. Closes OQ-ANOM-01 (`docs/SOC2_READINESS.md §76.11` — P1, before first enterprise pilot M10). Full analysis in `docs/OBSERVABILITY.md §44.3`.
- **Owner:** security-engineer + devops-lead + compliance-officer
- **Why:** (1) **Architectural confirmation.** GROUP BY semantics in Cloudflare Analytics Engine are equivalent to SQL GROUP BY — aggregation is per-bucket, not fleet-wide. The HAVING clause tests each `(tenant_id, subnet)` bucket independently before returning results. The concern in OQ-ANOM-01 would only be valid if the query omitted `tenant_id` from the GROUP BY key, which it does not. (2) **Empirical verification over inference.** Rather than relying solely on architectural reasoning, a mandatory staging test with a synthetic high-volume tenant (1,000 failures) and a synthetic low-volume tenant (12 failures) on distinct subnets closes the gap between theoretical isolation and demonstrated isolation. (3) **SOC 2 CC7.2 evidence quality.** CALIB-E-002 (the passing test chain event + CI log) gives auditors a direct empirical artefact that CC7.2 anomaly detection operates correctly per-tenant — a stronger claim than an architecture diagram alone. (4) **Consistent with FORM's staged validation pattern.** DEC-053 (IR-01 — concurrent incident ordering) and DEC-054 (WL-OBS-01 — domain hash) both followed the pattern of: identify concern → analyse → document resolution with evidence artefact requirement. DEC-057 applies the same discipline to SIEM alert calibration.
- **Reverse cost:** Low. If a future Analytics Engine schema change or query redesign creates a genuine cross-tenant blind spot, the resolution is: update the GROUP BY key, re-run §44.3.4, emit a new `system.siem_calibration_verified` event, and file updated CALIB-E-002 evidence. No new infrastructure is required. The M9 ClickHouse migration already mandates re-verification (§44.6 item 11), ensuring the decision is re-evaluated at the most relevant architectural transition point.

---

### DEC-056 · OQ-ANOM-02: AL-SIEM-06 dead-man's switch threshold — Option (a) confirmed with graduated activation

- **Decision:** The existing `docs/OBSERVABILITY.md §27.7` AL-SIEM-06 specification — zero `siem_events` rows in any **30-minute** window during **08:00–22:00 UTC** — is formally confirmed as the production specification. No change to the 30-min threshold or the business-hours gate. A **graduated activation policy** is added for the initial production deployment: AL-SIEM-06 runs in **shadow mode** (P3 Slack `#devops-alerts` only, no PagerDuty) for the first 30 calendar days after production deployment, then automatically escalates to **full activation** (P1 PagerDuty `form-devops` HIGH) when `auth_event_avg_per_30min > 5` (30-day rolling average) OR 30 days elapse, whichever comes first. The 30-day cap is absolute — even at zero event volume after 30 days, P1 activation proceeds. On exit from shadow mode, devops-lead emits `system.siem_alert_activated` DEC-030 event (STANDARD, 3yr) and files CALIB-E-001 evidence artefact (`compliance/evidence/cc7/siem-calibration/`). Full analysis and dual-phase alert specification in `docs/OBSERVABILITY.md §44.2`. Closes OQ-ANOM-02 (`docs/SOC2_READINESS.md §76.11` — P1, before M5 PagerDuty deployment).
- **Owner:** devops-lead + security-engineer + compliance-officer
- **Why:** (1) **Threshold confirmed correct.** Event volume baseline analysis (§44.2.1) shows that the 30-min zero-event probability during business hours drops to LOW at pre-beta (≥ 1 active user) and to NEGLIGIBLE at consumer-beta scale. A 60-min threshold (Option (b)) would increase the undetected-stall window from 30 min to 60 min — during which up to 60 min of HMAC chain events could be silently lost. The 30-min threshold minimises detection lag without triggering on transient hiccups. (2) **Business-hours gate already resolves the off-peak concern.** OQ-ANOM-02 raised concern about 02:00–06:00 UTC off-peak false positives. §27.7 already gates AL-SIEM-06 to 08:00–22:00 UTC — the off-peak concern is fully resolved by the existing specification, not a new mechanism. (3) **Graduated activation isolates the remaining false-positive risk.** The only plausible scenario for a genuine false positive (a zero-event 30-min window during business hours) is the staging-only phase before any real user traffic. The 30-day shadow mode eliminates this risk for a one-time bounded period while ensuring P3 Slack alerts demonstrate the control was operational throughout the ramp-up. (4) **SOC 2 CC4.1 continuity.** P3 shadow alerts throughout the calibration window provide daily evidence the control intention was active. `system.siem_alert_activated` provides a tamper-evident chain record of the transition date and the calibration metric that triggered it — giving auditors full evidence coverage across both the shadow and full-activation phases. (5) **Option (c) complexity trade-off.** Adding a rolling 24h `auth_event_count` guard (Option (c)) would require a new pg_cron aggregation table or Better Stack conditional logic not natively supported in the alert rule DSL — implementation overhead disproportionate to the narrow staging-only false-positive risk that the shadow mode already resolves.
- **Reverse cost:** Low. Reverting to immediate P1 activation (removing the shadow mode) requires only updating the Better Stack alert routing on day 1 of deployment and removing the `system.siem_alert_activated` shadow-mode emission step. Increasing the threshold to 60 min requires updating §27.7, a new DECISION_LOG entry (DEC-056-REV), and evidence that the annual false-positive review (§44.6 item 10) triggered the change. Neither change affects the HMAC chain schema, any Postgres migration, or any SOC 2 evidence already filed.

---

### DEC-055 · OQ-CTOOL-01: Cloudflare WAF evidence collection method — Terraform state export adopted

- **Decision:** Cloudflare WAF rule evidence (SOC 2 artefact CTOOL-WAF-E-001) is collected quarterly via **Terraform state export** (`terraform refresh` → `terraform show -json | jq '[.values.root_module.resources[] | select(.type == "cloudflare_ruleset")]'`) rather than a manual Cloudflare Dashboard screenshot. Output is a structured JSON file filed to R2 `compliance/evidence/cc6/waf-rules-YYYY-QN.json`, SHA-256 checksummed into MASTER-INDEX, then mirrored to Vanta manual evidence tab as CTOOL-Cloudflare. Dashboard screenshot (Option B) retained as documented fallback for Terraform unavailability; requires quarterly compliance memo note when used. Full extraction command, quarterly upload procedure, evidence artefact spec, and seven-item implementation checklist in `docs/SOC2_READINESS.md §82`. Closes OQ-CTOOL-01 (§78.11).
- **Owner:** devops-lead + compliance-officer
- **Why:** (1) **Tamper-evidence.** Terraform state is the authoritative deployment record; auditors can cross-reference the quarterly JSON against the GitHub PR that applied the change (CC8.1), providing an end-to-end provenance chain. Dashboard screenshots cannot be independently corroborated against a deployment record and could diverge from the deployed state if a manual console change bypassed Terraform. (2) **Checksum-stable artefact.** `sha256sum` of a JSON file is stable between quarters unless WAF rules actually changed. Screenshot SHA-256 changes whenever the Cloudflare UI updates its rendering, producing spurious checksum divergence that obscures genuine rule modifications — incompatible with the §79.7 MASTER-INDEX integrity protocol. (3) **§79.7 chain-of-custody.** The MASTER-INDEX protocol requires hashing the artefact *before* upload — the hash covers content the devops-lead authored locally, not a rendering artefact produced by a third-party browser. JSON satisfies this; screenshots do not. (4) **Operational simplicity.** FORM already uses Terraform for Cloudflare WAF deployment; extraction is a two-command pipeline. Screenshot collection requires console login, zone navigation, and visual verification — more failure-prone steps with no equivalent provenance link to the deployment state.
- **Reverse cost:** Low. Reverting to dashboard screenshots requires only updating §82.3 procedure and the MASTER-INDEX comment convention; no schema, migration, or DEC-030 changes required. If Terraform is deprecated in favour of another IaC tool (Pulumi, OpenTofu), update the extraction command in §82.3 to use the equivalent `show --json` output of that tool and emit a new DECISION_LOG entry.

---

### DEC-054 · OQ-WL-OBS-01: White-label custom domain in DEC-030 chain — SHA-256 hash adopted

- **Decision:** `custom_domain` appears as **`custom_domain_hash` = SHA-256(`custom_domain`)** in all `tenant.white_label_*` DEC-030 HMAC-chain payloads. Plaintext `custom_domain` is retained in `tenant_white_label_domains.custom_domain` (RLS-gated; `compliance_reviewer` + `tenant_admin` SELECT own-tenant). Migration 0072 DDL extended with `custom_domain_hash TEXT GENERATED ALWAYS AS (encode(sha256(custom_domain::bytea), 'hex')) STORED`. `docs/AUDIT_LOG_SCHEMA.md §WhiteLabel` payload spec for `tenant.white_label_provisioned` updated to use `custom_domain_hash`. Design in `docs/OBSERVABILITY.md §42.13`. Closes OQ-WL-OBS-01.
- **Owner:** compliance-officer + security-engineer
- **Why:** (1) **Consistent redaction pattern.** DEC-043 hashes `business_justification_reason` in PAM events; DEC-054 applies the same principle to custom domain names in white-label events. Auditor bulk chain exports should not expose commercial domain names of enterprise customers even though the domain is DNS-resolvable — the chain export is a compliance artefact, not a public DNS query. (2) **Matching §43 pattern.** §43 (Webhook Delivery Observability, v4.0) uses `endpoint_url_hash` in `integration.webhook_*` events for identical reasons; adopting `custom_domain_hash` in §42 creates a consistent cross-section design. (3) **Privacy by default.** White-label tenants consider their custom domain commercially sensitive during contract negotiations; early exposure in audit exports could create a business-sensitivity breach even absent health data. Counter-argument addressed: the domain being browser-visible (DNS CNAME) does not obligate FORM to include it in the immutable 7-year audit chain — the chain records operational events, not a DNS directory.
- **Reverse cost:** Low. Changing from hash to plaintext requires (a) updating the `custom_domain_hash` column definition or adding a separate `custom_domain_plaintext` column; (b) amending `docs/AUDIT_LOG_SCHEMA.md §WhiteLabel`; (c) no re-hashing of historical events (plaintext is always available in `tenant_white_label_domains` for any lookup). Reversal would require a DECISION_LOG entry and compliance-officer sign-off to confirm auditor bulk export privacy is still satisfied.

---

### DEC-053 · OQ-IR-01: Concurrent incident sub-chain ordering — Option (a) adopted; timestamp-interleaved master chain with skip-and-verify auditor protocol

- **Decision:** When two or more incidents are simultaneously active, their `incident.*` DEC-030 HMAC-chained events are recorded in the **existing timestamp-interleaved master chain** without modification. No per-incident parallel sub-chains (option b) are introduced. A documented skip-and-verify SQL query (`docs/INCIDENT_RESPONSE.md §18.3.1`) extracts per-incident event sequences from the master chain for auditor fieldwork. CONC-CHAIN-01 invariant established: no `incident.*` event may reference an `incident_id` belonging to a different active incident; cross-contamination detection query in §18.3.3. Full design in `docs/INCIDENT_RESPONSE.md §18`.
- **Owner:** security-engineer + compliance-officer
- **Why:** (1) **Statistical rarity** — concurrent P0/P1 incidents at FORM's current scale (< 5k users, pre-Series A) estimated at < 2% annual probability; a chain-merge architecture would be deployed < 1× per year. (2) **DEC-030 structural integrity** — option (b) requires `incident.sub_chain_fork` / `incident.sub_chain_merge` event types that violate the single-list HMAC invariant in `docs/AUDIT_LOG_SCHEMA.md`; a merge-step write-ordering race under concurrent Cloudflare Workers execution introduces a new chain-break failure mode. (3) **Auditor-standard practice** — SOC 2 fieldwork guides permit filtered event-stream extraction; §18.3.1 SQL produces an equivalent per-incident timeline in < 100 ms. (4) **Consistent with DEC-043/DEC-044/DEC-051 pattern** — FORM consistently chooses documented auditor protocol over complex automated infrastructure at this scale. (5) **Upgrade path preserved** — option (b) remains available as an additive migration (new column, new event types, no re-hashing of historical events required) if concurrent-incident frequency increases post-Series A.
- **Reverse cost:** Low. Option (a) makes no schema changes. Upgrading to option (b) requires adding two new DEC-030 event types, a `sub_chain_id` column to `audit_log_events`, and a merge algorithm in `siem-incident-automator` — all additive; historical events unaffected.

---

### DEC-052 · OQ-EC-01: Evidence vault architecture — R2-primary, Vanta-mirror adopted

- **Decision:** Cloudflare R2 bucket `form-soc2-evidence` (EU region, Object Lock Governance mode, versioning locked) is the **primary tamper-evident evidence store** for all SOC 2 Type II artefacts. Vanta is the **auditor-facing mirror** only — evidence is authored on R2 first, uploaded to Vanta within 48 hours. Full architecture in `docs/SOC2_READINESS.md §80`. Closes OQ-EC-01 (§79.10). Also closes OQ-EC-02 (operational decision, no separate DEC required): `compliance/evidence/pre-obs/` folder with README.md protocol for design-phase evidence separation.
- **Owner:** compliance-officer + devops-lead
- **Why:** (1) **WORM tamper-evidence.** R2 Object Lock prevents modification or deletion of evidence files during the retention window — Vanta has no equivalent write-once guarantee. A Vanta-primary model cannot credibly claim tamper-evident storage to a SOC 2 auditor. (2) **Direct control.** R2 is under FORM's direct infrastructure control. A Vanta outage during fieldwork would block auditor access if Vanta were primary; R2 provides the offline fallback (presigned URL in under 5 minutes). (3) **Checksum-before-upload discipline.** §79.7 requires compliance-officer to hash each artefact locally before upload and record the hash in MASTER-INDEX. This only works cleanly if R2 is primary — the hash is computed before the file leaves FORM's control. A Vanta-first model would require hashing *after* upload, which is a weaker integrity model. (4) **DEC-030 chain separation.** HMAC chain lives in Supabase; human-readable evidence lives in R2. Keeping them on separate infrastructure prevents a single compromise from affecting both the tamper-evident log and the supporting evidence. Vanta-as-mirror is correct because: auditors expect a portal checklist interface; Vanta automated connectors (§78) already produce evidence in Vanta directly; unified auditor view is operationally necessary. Dual-primary (both authoritative) rejected: two sources of truth break the §79.7 SHA-256 checksum discipline. Access control: `r2:form-api` has NO ACCESS (same zero-access principle as Supabase Art. 9 tables). `pre-obs/` separation (OQ-EC-02): design-phase artefacts stay on R2 `pre-obs/` only — never uploaded to Vanta to prevent confusion with observation-window evidence.
- **Reverse cost:** Medium. Migrating from R2-primary to a different store would require: (a) transferring all existing evidence files with SHA-256 hash verification; (b) updating MASTER-INDEX to reflect new paths; (c) re-filing VAULT-E-001 (bucket policy evidence) with the new store's configuration; (d) notifying the audit firm of the evidence location change. The R2 choice is low-cost infrastructure-wise (Cloudflare R2 pricing is minimal at evidence volumes) and aligns with FORM's existing Cloudflare Workers infrastructure. Re-evaluation trigger: if Cloudflare R2 changes EU data residency guarantees in a GDPR-incompatible way, migrate to AWS S3 eu-central-1 Object Lock as the leading alternative.

---

### DEC-051 · OQ-SSO-28.2: `tenant_users_role_history` retention period — 7 years adopted

- **Decision:** `tenant_users_role_history` rows are retained for **7 years** from `changed_at`, enforced by pg_cron job 32 (`turh-retention-purge`, 04:00 UTC daily). Safety gate: Phase 1 hard-deletes only rows where `user_id IS NULL AND user_id_pseudonym IS NOT NULL` (GDPR Art. 17 pseudonymisation already complete); Phase 2 emits `system.turh_stale_active_user_detected` HIGH DEC-030 event + AL-TURH-01 (P2 Slack `#compliance-alerts`) if any row with `user_id IS NOT NULL` exceeds the 7yr boundary. Full design and legal basis in `docs/SSO_SCIM_IMPLEMENTATION.md §29`.
- **Owner:** compliance-officer + enterprise-architect + security-engineer
- **Why:** Three grounds — each conclusive: (1) **Audit-chain continuity (TURH-CHAIN-01):** `scim.user_updated` DEC-030 events (the HMAC chain anchors for role changes) are retained 7yr; purging `tenant_users_role_history` at 3yr creates an irremediable SOC 2 CC6.3 evidence gap where auditors see chain events but find no corroborating table rows. (2) **Legal obligation:** Ukrainian Tax Code Art. 44 (7yr business records) + EU audit-record doctrine (access-control records retained at least as long as data they protect). (3) **SOC 2 multi-cycle evidence:** 3-cycle Type II programme (IPO-readiness standard) requires longitudinal CC6.3/CC6.6 data across 5+ years. Legal basis for personal data retention: GDPR Art. 6(1)(c) + Art. 17(3)(b) — `user_id` column is pseudonymous personal data; retention is justified by legal obligation; pseudonymisation on employee departure preserves the audit record without re-identification; full hard-delete at 7yr boundary. Consumer privacy policy: no change (enterprise-only table). Enterprise DPA Annex B: disclosure row required before first EU enterprise DPA execution (§14.3.2 template — see §29.4).
- **Reverse cost:** Medium. Reducing retention below 7yr would require: (a) amending the DPA template and notifying all signed enterprise customers; (b) confirming with AICPA/auditor that shorter retention satisfies SOC 2 multi-cycle evidence requirements; (c) updating pg_cron job 32 DDL. The 7yr choice is conservative and aligns with the DEC-030 chain event retention — no downside to the current decision absent regulatory change.

---

### DEC-050 · OQ-TURH-01 + OQ-TURH-02: `form_role_enum` & `role_change_source_enum` Postgres ENUM types adopted

- **Decision:** Two named Postgres ENUM types are created in migration `0069_role_enum_types.sql`: (1) `form_role_enum` with five values (`tenant_owner`, `tenant_admin`, `tenant_manager`, `member`, `support_readonly`) — matching `docs/SSO_SCIM_IMPLEMENTATION.md §5.1`; (2) `role_change_source_enum` with four values (`scim_sync`, `admin_ui`, `jit_provisioning`, `form_system`). Columns migrated: `tenant_users.form_role`, `scim_group_role_mappings.form_role`, `tenant_users_role_history.old_role`, `tenant_users_role_history.new_role` (all → `form_role_enum`); `tenant_users_role_history.changed_by` (→ `role_change_source_enum`). Full DDL and rationale in `docs/DATA_MODEL.md §34`.
- **Owner:** platform-engineer + enterprise-architect
- **Why:** (1) **Type safety at storage layer** — Postgres ENUMs reject invalid role values without application-level validation, eliminating a silent data-quality risk. (2) **Gap closure** — the original `tenant_users.form_role` CHECK constraint listed only 4 values; `support_readonly` was documented in SSO_SCIM §5.1 as valid but absent from the CHECK, creating a silent inconsistency. `form_role_enum` includes all five §5.1 values, closing the gap definitively. (3) **Single-place definition** — `ALTER TYPE ... ADD VALUE` is the only migration path for new roles, making the enum the canonical authority. The `scim_group_role_mappings` table retains a companion CHECK restricting group-mappable values to `('tenant_admin', 'tenant_manager', 'member')` — `tenant_owner` and `support_readonly` remain policy-prohibited from SCIM group assignment.
- **Reverse cost:** Medium. Rolling back requires reversing the `USING` casts in `0069_role_enum_types.sql` and dropping the ENUM types — a Postgres TYPE DROP requires no dependent columns, so the migration must be reversed in exact sequence. Any code referencing enum values as typed constants must also revert to string literals. No cross-tenant or DPA impact.

---

### DEC-049 · OQ-SSO-27.2: SCIM role change audit trail — `tenant_users_role_history` table approach adopted

- **Decision:** Role change values (old/new role) produced by SCIM `PUT /Users` full-replace and group PATCH operations are stored in a separate `tenant_users_role_history` append-only table, **not** in the DEC-030 HMAC chain. The `scim.user_updated` event continues to record `fields_changed: ['role']` (attribute name only) as the tamper-evident chain anchor; `tenant_users_role_history.scim_request_id` cross-references that anchor to the actual role values. Full design in `docs/SSO_SCIM_IMPLEMENTATION.md §28` (migration `0068_tenant_users_role_history.sql`).
- **Owner:** security-engineer + compliance-officer + enterprise-architect
- **Why:** Storing `old_role`/`new_role` in the DEC-030 payload would make role assignments visible in any auditor bulk chain export. An auditor reviewing chain events for CC6.1/CC6.3 evidence receives the entire export for the observation period; if role names carry business sensitivity (e.g. distinguishing `manager` from `member` in an org chart context), a cross-tenant accidental export would expose confidential organisational data. The separate RLS-gated table (`compliance_reviewer` SELECT; `tenant_admin` SELECT own tenant; `form_system` INSERT; no `form_api` access) confines role history to queries that require active authentication and tenant scope. The `chk_turh_role_changed` constraint prevents no-op inserts; the `chk_turh_user_xor_pseudonym` constraint ensures GDPR Art. 17 erasure path is preserved (user_id SET NULL on user delete, user_id_pseudonym populated at erasure).
- **Reverse cost:** Low. Removing the table and storing role values in the chain instead would require a HMAC chain schema migration (new field in `scim.user_updated` payload) and a retroactive decision on whether existing chain events are re-emitted. Any auditor evidence already collected from the table would need to be reframed. The separate-table approach is additive — the chain remains unchanged.

---

### DEC-048 · OQ-SSO-27.3: SCIM deprovisioning KV failure — option (c) adopted; AL-REVOKE-01 added

- **Decision:** When the Cloudflare KV write fails during SCIM PATCH deprovisioning (step: `env.SCIM_KV.put(revoke:user:{tenant_id}:{user_id})`), the SCIM Worker responds HTTP 200 to the IdP (maintains idempotency per §27.9) and immediately falls back to the Supabase blocklist path (§22 `kv_session_revocations` INSERT). A `scim.session_revocation_kv_fallback` DEC-030 event (HIGH, 7yr) is emitted, and AL-REVOKE-01 (P1 PagerDuty `form-platform`) fires. Full design in `docs/SSO_SCIM_IMPLEMENTATION.md §28`. The DPA G-013 template clause is updated to: "Session revocation on deprovisioning uses best-effort synchronous Cloudflare KV (< 3 ms under normal conditions). In the event of a regional KV failure, a guaranteed Supabase fallback path achieves revocation within 30 seconds, within the contracted 60-second P99 SLA commitment."
- **Owner:** platform-engineer + devops-lead + compliance-officer
- **Why:** Option (a) — return HTTP 503 on KV failure — breaks the SCIM idempotency guarantee: the IdP would retry the DELETE, triggering a second deprovisioning flow that may conflict with a concurrent reactivation. Option (b) — silently accept the fallback — provides no operational visibility into KV degradation, which is itself a CC7.2 gap. Option (c) gives the SOC 2 auditor evidence of detection (AL-REVOKE-01 history) while preserving idempotency and the SLA commitment. The Supabase fallback path achieves < 30 s revocation (confirmed by §22 architecture: `SELECT` on `kv_session_revocations` fires on every Worker request for revocation check), meaning the 60-second P99 is met by the fallback alone.
- **Reverse cost:** Low. The fallback path already exists (§22 architecture); the only net-new implementation is the try/catch wrapper, the DEC-030 event, and AL-REVOKE-01. Removing this in favour of option (a) or (b) would require removing the event type from AUDIT_LOG_SCHEMA.md (schema migration) and re-negotiating DPA language with affected customers.

---

## 2026-06-13

### DEC-047 · Vanta selected as continuous compliance tooling over Drata (PRE-25 pathway)

- **Decision:** Vanta is selected as FORM's continuous compliance automation platform for the SOC 2 Type II observation period. Drata is not adopted at this stage. Full rationale in `docs/SOC2_READINESS.md §78.2`. Re-evaluation trigger: Series A close or initiation of HIPAA / ISO 27001 scope expansion.
- **Owner:** compliance-officer + security-engineer
- **Why:** (a) SOC 2-first product roadmap — Vanta's core evidence automation is tightly scoped to SOC 2; Drata adds HIPAA/ISO 27001 surface area that increases DPA scope without value pre-Series A. (b) Solo-team UX — Vanta's control library is simpler to manage before a dedicated compliance engineer is hired. (c) Supabase PostgreSQL connector available natively; Cloudflare and Better Uptime remain manual-upload paths for both tools. (d) Auditor portal adoption — Vanta's auditor portal is more widely adopted among mid-market SOC 2 audit firms likely to be engaged for FORM's first Type II. (e) Entry-tier pricing is lower pre-Series A. Privacy invariant enforced by `vanta_readonly` Postgres role (`migration 0065_vanta_readonly_role.sql`): explicit REVOKE on all Art. 9 / health-data tables; `vanta_subscription_summary` view exposes only aggregate billing-tier counts. DEC-030 event `system.compliance_tool_connected` (STANDARD, 3yr, CTOOL-E-006) emitted by compliance-officer after DPA execution.
- **Reverse cost:** Medium. Switching to Drata or another tool post-activation requires: (a) Drata DPA execution and `vanta_readonly` role audit; (b) evidence re-upload to Drata for any artefacts already filed in Vanta; (c) auditor portal handoff if observation period has begun. No schema changes required — `system.compliance_tool_connected` payload supports `tool_name: 'drata'` already.

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

### DEC-046 · OQ-33 clinical-safety ruling: readiness_bucket CONDITIONAL PASS; mood_bucket VETO

- **Decision:** SPLIT ruling on OQ-33 (2026-06-12). `readiness_bucket` (low|medium|high) is **CONDITIONALLY PERMITTED** in PostHog as a property on the `checkin.submitted` event only, subject to all conditions in `docs/OBSERVABILITY.md §25.9 OQ-33 Ruling`. `mood_score` and `mood_bucket` in any form are **PERMANENTLY PROHIBITED** from PostHog — clinical-safety VETO. `mood_captured` (boolean) may replace `mood_score` as a completion signal without revealing the score value. The raw `readiness_score` (1–5 integer) remains prohibited from PostHog regardless of this ruling. The ruling expires at SOC 2 observation period start (mandatory re-review) and must be re-issued immediately if FORM integrates biometric readiness signals into the check-in score computation. `docs/PRIVACY_IMPACT.md §2.2` OC-08 constraint updated to reflect the split ruling. A Lightweight PIA (PIA-YYYY-NNN) must be filed by compliance-officer before any `readiness_bucket` value is transmitted to PostHog (per `docs/PRIVACY_IMPACT.md §6.2` OQ-PIA-03 requirement). DPA with PostHog must be reviewed by compliance-officer to confirm `readiness_bucket` is within agreed sub-processor scope — blocking pre-condition on the CONDITIONAL PASS.
- **Owner:** clinical-safety (ruling) + compliance-officer (PIA filing + PostHog DPA review) + platform-engineer (SDK allowlist/blocklist + CI lint rule)
- **Why:** `readiness_score` (physical readiness self-report) at the boundary of GDPR Art. 9 health data; the three-bucket collapse reduces individual-level inference precision to an acceptable level when combined with the binding conditions (event-property-only, no person-record fields, no A/B segmentation, no export joins with other health-adjacent fields). `mood_score` is mental health data — EDPB and multiple EU supervisory authorities (Irish DPC, CNIL) have consistently held that subjective mood data in a digital wellness context constitutes Art. 9 special category data. PostHog's individual-level person model means bucketed mood properties persist on pseudonymous person records, creating longitudinal mental health state profiles. Combined with FORM's `ed_path` super property, a "low" mood bucket for a user on the `soft` ED path constitutes particularly high-risk inference in a population that includes users with eating disorders, depression, and body dysmorphia. The harm risk is not reducible by bucketing alone. Full ruling text: `docs/OBSERVABILITY.md §25.9`.
- **Reverse cost:** Medium for the CONDITIONAL PASS (removing `readiness_bucket` from PostHog requires SDK update + PIA amendment + stakeholder communication, but no schema migration). High for the VETO (overturning a clinical-safety VETO requires a new ruling with fresh DPIA-level evidence and explicit founder sign-off — not a product team decision alone).

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
