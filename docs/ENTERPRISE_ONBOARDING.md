# FORM · Enterprise Customer Onboarding v0.1

> Operational playbook for every enterprise deal from contract signature through Year 1 renewal.
> Owner: `customer-success` + `enterprise-architect` + `compliance-officer`.
> Activate per account. One named CSM owns each account end-to-end.
> References: `docs/ENTERPRISE.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/SSO_SCIM_IMPLEMENTATION.md`, `docs/DATA_MODEL.md`, `docs/ENTERPRISE_SLA.md`.

---

## 0. Principles

**Privacy floor — non-negotiable at every tier:**
HR and People leads access *aggregate* engagement metrics only. No individual workout history, no per-person progress view, no coaching conversation data. This is enforced at the RLS layer (`DATA_MODEL.md §6`) and cannot be unlocked by contract, price, or feature flag. See §9 for the customer communication script.

**No-go use cases (ENTERPRISE.md §no-go):**
- Insurance risk-scoring or premium adjustment based on fitness data
- Government contracts requiring backdoor or plaintext data access
- Wellness programs where participation is mandatory or tied to insurance pricing
- Sports league contracts seeking athlete performance evaluation data

If a prospect surfaces any of these in the sales process, escalate to founder before advancing. Do not accept the deal.

**HMAC audit chain (DEC-030):**
Every privileged action touching this tenant generates an immutable, HMAC-chained audit event. The chain is append-only and tamper-evident; no actor — including FORM support staff — can modify or delete entries. Tenants with Growth and Enterprise tier contracts can export their own tenant-scoped audit log from the Admin Dashboard at any time.

---

## 1. Pre-launch checklist

Complete before the kick-off call. All items are blockers unless marked optional.

### 1.1 Commercial

- [ ] MSA counter-signed by both parties
- [ ] DPA counter-signed — **required before any employee data enters FORM** (GDPR Art. 28; also required for US HIPAA-adjacent contracts). Template in DocuSign envelope from legal.
- [ ] Order Form confirming: seat count, tier (Starter / Growth / Enterprise), contract length, billing cadence (monthly or annual upfront), SLA tier
- [ ] Invoice or purchase order received if net-terms requested
- [ ] Slack Connect channel created: `form-[customer-slug]-shared`

### 1.2 Security review package delivery

Provide *before* IT/CISO sign-off, not after:

| Document | Delivery method | Notes |
|---|---|---|
| Sub-processor list | Public URL — `form.coach/subprocessors` | No NDA required |
| Security overview | Public URL — `form.coach/security` | No NDA required |
| SOC 2 executive summary | Email with NDA acknowledgement | Available post-audit (Q1 2027); pre-audit: share SOC 2 readiness summary |
| Pentest executive summary | Email under executed NDA | Available for accounts ≥ $50k ACV; scope per `docs/SOC2_READINESS.md §16` |
| Completed security questionnaire | `docs/SECURITY_QUESTIONNAIRE.md` answers exported to PDF | Fill once, reuse for same-tier prospects |
| GDPR DPIA summary | Email | Available for EU-based customers processing Art. 9 health data |

### 1.3 Technical readiness (customer side)

Confirm with the customer's IT lead before Day 0:

- [ ] IdP chosen: Okta / Azure AD / Google Workspace / OneLogin / other SAML 2.0 or OIDC provider
- [ ] IdP admin who will configure the FORM app integration is identified and has FORM contact
- [ ] Group or directory scope defined: which users / groups will be synced via SCIM
- [ ] Initial `tenant_owner` email addresses confirmed (max 2 per contract)
- [ ] IP allowlist required? (optional; forces SSO device trust if enabled)
- [ ] White-label required? (Enterprise tier only — see §8.2 for setup checklist)
- [ ] Custom data residency requirement? (EU region: `eu-west-1` Supabase; default: `us-east-1`)

---

## 2. Contract week — Day 0–7

### 2.1 Kick-off call agenda (60 min)

1. Introductions — CSM, customer lead (HR / People Ops / IT), technical contact (IT admin)
2. Success criteria definition — agree on 2–3 measurable outcomes at 90 days (e.g. ≥ 30% activation, ≥ 40% D30 retention, NPS ≥ 40 in first survey)
3. Privacy model walkthrough — explain the privacy floor; confirm HR understands they see only aggregates (§9)
4. No-go use case confirmation — brief verbal acknowledgement that data will not be used for insurance pricing, performance evaluation, or policy enforcement (ENTERPRISE.md §no-go)
5. Technical timeline — walk through §3 setup steps; confirm IdP admin availability for Week 2
6. Communication plan — who receives incident notifications, who approves announcements to employees

### 2.2 Post-call

- Send written summary of success criteria to Slack Connect and customer lead email within 24 hours
- Open onboarding tracker (Linear project: `ENT-[customer-slug]`)
- Tag `tenant.created` event in audit log — see §3.1

---

## 3. Technical setup — Day 7–30

### 3.1 Tenant provisioning

The `customer-success` CSM creates the tenant record; `enterprise-architect` confirms RLS policy is active.

**Audit events emitted (DEC-030 HMAC chain):**

| Event | Severity | Retention | Payload fields |
|---|---|---|---|
| `tenant.created` | HIGH | 7 yr | `tenant_id`, `tier`, `seat_count`, `contract_start`, `contract_end`, `sla_tier`, `data_region`, `actor_id` (CSM) |
| `tenant.sla_tier_set` | STANDARD | 7 yr | `tenant_id`, `sla_tier`, `seat_count` |
| `admin.rbac_role_assigned` | HIGH | 7 yr | `tenant_id`, `target_user_id`, `role: 'tenant_owner'` |

All three events are chained. The `tenant.created` entry precedes any data write — this ordering constraint is mandatory per DEC-030.

**Verify tenant isolation after provisioning:**

```sql
-- Run as service role; confirm zero cross-tenant leakage
SELECT count(*) FROM users WHERE tenant_id = '[new-tenant-id]';
-- Expected: 0 (no users yet)

-- Confirm RLS policy is active on all tenant-scoped tables
SELECT schemaname, tablename, policyname, cmd
FROM pg_policies
WHERE tablename IN ('users','workouts','meals','audit_log_events')
  AND policyname LIKE '%tenant_isolation%';
-- Expected: 4 rows, one per table
```

See `docs/DATA_MODEL.md §6` for full RLS policy listing and §4 for schema-per-tenant vs row-level discussion.

### 3.2 SSO configuration

Full protocol reference: `docs/SSO_SCIM_IMPLEMENTATION.md`. Summary for the onboarding call with the customer's IdP admin:

**IdP-initiated flow (most common — Okta, Azure AD tile click):**
1. Customer IdP admin creates SAML 2.0 app integration using SP metadata from FORM Admin Dashboard → SSO Settings
2. Provide: ACS URL, Entity ID, NameID format (`emailAddress`)
3. FORM maps: `email` → user identity, `groups` SAML attribute → FORM roles (`tenant_admin`, `tenant_manager`, or member)
4. Test with the `tenant_owner` account before enabling enforcement

**SP-initiated flow (for bookmark/direct links):**
1. Same metadata exchange as above
2. FORM redirects unauthed users to IdP via AuthnRequest; IdP redirects back with assertion
3. Confirm `RelayState` passes through correctly (link preservation)

**OIDC (Google Workspace, modern IdPs):**
- Provide client ID and secret; FORM handles discovery document fetch
- Scopes required: `openid email profile`

**Enforcement modes (set in Admin Dashboard → SSO Settings):**

| Mode | Behaviour |
|---|---|
| `optional` | SSO available; email/password also works |
| `required` | All new sessions must use SSO; existing sessions survive until expiry |
| `enforced` | All sessions immediately revoked on save; SSO is the only path |

Do not set `enforced` until at least one `tenant_owner` has confirmed a successful SSO login.

### 3.3 SCIM provisioning

Reference: `docs/SSO_SCIM_IMPLEMENTATION.md §8`.

1. Generate SCIM bearer token from Admin Dashboard → SSO Settings → SCIM
2. Provide to IdP admin: SCIM base URL, bearer token
3. Map IdP groups to FORM roles in IdP SCIM app configuration
4. Run test provisioning with 5 users before opening to full directory

**SCIM audit events (DEC-030):**

| Event | When |
|---|---|
| `tenant.scim_provisioned` | First successful SCIM sync completing > 0 users |
| `user.scim_created` | Each user created via SCIM |
| `user.scim_deprovisioned` | User removed from IdP group → deactivated in FORM |

SCIM deprovisioning sets `status = 'inactive'` and revokes all active sessions within 30 seconds (session KV TTL). Health and coaching data are retained for the contractual data retention period unless a deletion request is filed under the DPA.

> **SCIM IP enforcement (self-hosted proxy customers only):** Customers running a self-hosted SCIM proxy with stable IP ranges may enable `scim_ip_enforcement_enabled` at Admin Dashboard → SSO Settings → SCIM → IP Restriction. This restricts inbound SCIM provisioning to the specified CIDRs; calls from other IPs are rejected (403) and logged as `scim.ip_allowlist_blocked` DEC-030 events. Enable only after a successful full-directory sync and only for tenants with confirmed static proxy IPs. Cloud-hosted IdP tenants (Okta SaaS, Azure AD) should not use this control without confirming static CIDR support with their IdP.
>
> Implementation reference: `docs/SSO_SCIM_IMPLEMENTATION.md §32.3` (DEC-062) and `§33.2` (onboarding protocol).

### 3.4 Initial admin accounts

After SSO is live:

1. Confirm `tenant_owner` can log in via SSO
2. Assign `tenant_admin` role to HR / People Ops lead (aggregate metrics only — no individual data)
3. Assign `tenant_manager` role to wellness program manager if applicable
4. Walk `tenant_owner` through Admin Dashboard sections: User Management, Analytics (aggregate), Billing, SSO Settings, Audit Log, SLA Reports

**Roles reference (`ENTERPRISE.md §admin-dashboard`):**

| Role | Sees | Cannot see |
|---|---|---|
| `tenant_owner` | All admin views, billing, SSO config, audit log export | Individual user coaching content, individual workout history |
| `tenant_admin` | Aggregate analytics, user list (names/emails only), provisioning | Individual user data, billing, SSO config |
| `tenant_manager` | Aggregate analytics for assigned group/department | Individual data, billing, SSO, user management |

The individual-data prohibition is enforced at RLS — it is not a UI restriction that can be bypassed. Any request from any role for individual user data returns zero rows at the database layer.

---

## 4. Employee rollout — Day 30–60

### 4.1 Pilot cohort launch (Day 30)

Recommended pilot size: 50–100 employees, self-selected volunteers.

**Announcement email template (send from customer's People Ops email, not FORM):**

---
Subject: We're launching FORM — AI fitness coaching, private by design

Hi [Name],

Starting [date], [Company] is rolling out FORM — an AI fitness coach for your phone.

What it does: personalised training programmes, real-time coaching using your phone camera, progress tracking.

What it doesn't do: share your personal data with [Company]. Your fitness data and coaching conversations belong to you. [Company] receives only aggregate, anonymised participation metrics — no individual results, no per-person reports, ever.

To get started: download FORM on iOS or Android and sign in with your [Company] account ([SSO provider]).

Questions? Reply to this email or message [People Ops contact].

---

### 4.2 Full seat rollout (Day 45–60)

Once pilot adoption reaches ≥ 25% (checkpoint from Admin Dashboard → Analytics → Activation):

1. Open SCIM sync to full directory (or CSV import for remaining groups)
2. Send company-wide announcement (adapt template above, reference pilot results if permission granted)
3. Communicate IT helpdesk process for SSO login issues
4. Confirm support escalation path: employee → IT helpdesk → CSM (not directly to FORM support)

### 4.3 Communication assets

| Asset | Owner | Notes |
|---|---|---|
| Welcome email template | CSM provides, customer sends | Do not include personal endorsements or fabricated testimonials |
| IT helpdesk FAQ | CSM provides draft, IT edits | Covers: SSO login, app download, privacy questions |
| Manager briefing (optional) | CSM provides deck | Aggregate-only metrics explainer; emphasise privacy floor |

---

## 5. Admin dashboard training

Deliver as 45-minute walkthrough (screen share) with `tenant_owner` and `tenant_admin`.

### 5.1 Training agenda

1. **Analytics** — what the aggregate metrics show (activation rate, D7/D30 retention, session frequency by cohort); what they explicitly do not show (individual names, individual data)
2. **User management** — viewing user list, manually assigning roles, triggering re-provisioning for SSO errors
3. **Audit log** — exporting the tenant-scoped audit log; what events are captured; why each entry is immutable (DEC-030 HMAC chain explainer at business level)
4. **SLA reports** — reading the monthly uptime report, how to dispute a calculation (30-day window per `ENTERPRISE_SLA.md §6`)
5. **Billing** — seat count, invoices, self-serve seat expansion
6. **SSO settings** — enforcement modes, SCIM token rotation, session policy
7. **Incident notification** — confirm the P0/P1 notification contact and channel (Slack Connect + email)

### 5.2 Privacy floor script (verbatim)

> "You'll notice the Analytics section shows your company's overall activation rate, session frequency, and cohort retention — but never a per-person breakdown. That's intentional. FORM's architecture prevents any admin role, including us, from querying individual user fitness data without that user's explicit action. If someone on your HR team asks us to show them how often a specific employee works out, the answer is: we cannot, by design. This isn't a policy — it's enforced at the database layer."

Deliver this during the Analytics section of training. Record the session (with consent) and store in the account CRM record.

---

## 6. Success criteria and milestones

Success criteria are set on the kick-off call (§2.1). Standard defaults by tier:

| Milestone | Metric | Starter threshold | Growth threshold | Enterprise threshold |
|---|---|---|---|---|
| 30-day activation | % employees who logged ≥ 1 session | ≥ 20% | ≥ 25% | ≥ 30% |
| 60-day D30 retention | % of activated users active in Day 31–60 | ≥ 35% | ≥ 40% | ≥ 45% |
| 90-day NPS | Net Promoter Score from in-app survey | ≥ 30 | ≥ 35 | ≥ 40 |
| 90-day CSM NPS | Customer-level NPS (HR / IT lead survey) | ≥ 40 | ≥ 45 | ≥ 50 |

### 6.1 30-day checkpoint

- Pull Admin Dashboard → Analytics → Activation (last 30 days)
- If activation < 50% of threshold: schedule intervention call; review announcement channel reach, SSO login friction, device compatibility issues
- Send 30-day summary to customer lead (plain text email; no fabricated benchmarks)

### 6.2 60-day full deployment review

- Confirm all contracted seats are provisioned
- Review SCIM sync health (Admin Dashboard → SSO → SCIM Sync Log)
- Confirm no open P0/P1 incidents for this tenant in the past 30 days (check `INCIDENT_RESPONSE.md` log)
- Share D30 retention cohort from Admin Dashboard

### 6.3 90-day QBR

Attendees: CSM, `tenant_owner`, `tenant_admin`, (optional) People Ops / Benefits lead.
Duration: 60 minutes.

**Agenda:**

1. Results vs. success criteria (§6 table) — data from Admin Dashboard
2. Employee feedback themes (anonymised, aggregate) — from in-app NPS comments
3. Support case review — any open tickets, SLA adherence per `ENTERPRISE_SLA.md §6`
4. Roadmap preview — upcoming features relevant to this customer
5. Expansion discussion (if D30 retention ≥ threshold) — additional seats, white-label, multi-year pricing
6. Year 1 renewal timeline — renewal date, contract review process

**QBR output:** CSM sends written summary within 48 hours. Include one-page data snapshot (PDF from Admin Dashboard). File in account CRM.

---

## 7. Ongoing cadence

| Cadence | Activity | Owner |
|---|---|---|
| Monthly | SLA uptime report delivered (auto — Admin Dashboard + PDF email) | System |
| Monthly | CSM reviews Admin Dashboard anomalies (activation drop > 10pp MoM) | CSM |
| Quarterly | QBR call (§6.3) | CSM |
| Quarterly | Access review — confirm all `tenant_admin` and `tenant_owner` roles are still appropriate | `tenant_owner` + CSM |
| Annually | DPA and security review refresh — re-confirm sub-processor list, provide updated SOC 2 attestation | compliance-officer + CSM |
| Annually | Renewal call (60 days before contract end) | CSM |

---

## 8. Expansion playbook

### 8.1 Seat expansion

Expansion triggers:
- Organic: `tenant_owner` requests more seats via Admin Dashboard → Billing
- CSM-triggered: activation rate ≥ threshold for 2 consecutive months, team growth signal from customer

Pricing at expansion: same per-seat rate as original contract if within contract period. Re-negotiated at annual renewal.

Seat expansion emits:
- `tenant.seat_count_changed` (STANDARD severity, 7yr retention) — DEC-030 HMAC chain
- Updated Order Form addendum required for > 20% seat increase

### 8.2 White-label upgrade

Available at Enterprise tier (1,000+ seats or $50k+ ACV). Activate when requested:

- [ ] Custom subdomain confirmed: `[company].form.coach` (CNAME provided by FORM, customer creates DNS record)
- [ ] Tenant logo uploaded to Admin Dashboard → Branding (PNG/SVG, ≤ 200 KB; stored in Cloudflare R2 under tenant-scoped path)
- [ ] Primary accent colour confirmed (hex value; WCAG 2.1 AA contrast check against `#0B0A07` background is enforced automatically)
- [ ] Email sender name for system notifications: `[Company] Fitness` (FORM appears in email footer per legal requirement)
- [ ] White-label addendum to MSA signed — confirms FORM branding requirements (footer attribution, no removal of privacy disclosure language)

White-label does not affect the privacy floor. The data model, RLS policies, and audit chain are identical regardless of branding.

### 8.3 Multi-year renewal

Year 1 renewal call 60 days before contract end. If moving to 2- or 3-year term:
- 15% discount on 2-year, 25% on 3-year (per `COST_MODEL.md §pricing-tiers`)
- Updated MSA with new contract end date
- Updated Order Form

---

## 9. Privacy floor — customer communication reference

Use these lines when explaining the privacy model to HR, legal, or executive stakeholders.

**What HR *can* see:**
- Company-wide activation rate (% of seats with ≥ 1 session in 30 days)
- D7/D14/D30/D60/D90 retention cohorts (company-wide)
- Session frequency distribution (anonymised histogram — no names)
- Feature adoption (% using strength vs. cardio vs. coaching)
- Department-level or group-level aggregates if the group has ≥ 10 active users (minimum cohort size enforced at query layer)

**What HR *cannot* see, ever:**
- Individual user session history
- Individual user progress or performance data
- Individual coaching conversation content
- Individual body metrics, weight, or injury history
- Per-person export of any kind

**Why this is architectural, not contractual:**
The database enforces row-level security by role. The `tenant_admin` and `tenant_manager` roles are granted access only to aggregate views (pre-computed, no underlying user rows accessible). FORM engineers with production access also cannot query individual user data across tenant boundaries — each query is constrained by the JWT-supplied `tenant_id` claim.

**If a manager asks "can you show me how active Sarah is?":**
The answer is no. FORM will not build, enable, or contract for any individual-level reporting feature. This is FORM's design principle, stated in the enterprise contract, and enforced at the database layer. Any enterprise deal that requires this capability is declined.

---

## 10. Incident communication to enterprise customers

Reference: `docs/INCIDENT_RESPONSE.md` (full runbook).

**CSM responsibilities during an active P0/P1:**

1. **Within 15 min of P0 declaration:** Contact `tenant_owner` on Slack Connect with: "We're investigating a platform incident. Status at form.statuspage.io. You'll hear from us every 30 minutes until resolved."
2. **Every 30 min:** Send update even if there's no new information — "Investigation ongoing; no customer data impacted so far; next update in 30 min."
3. **On resolution:** Send root cause summary within 24 hours. If any data exposure is suspected, immediately loop in `compliance-officer` — GDPR breach notification clock starts at FORM's confirmation.
4. **Post-mortem:** Share the public post-mortem (sanitised version) with the customer within 5 business days of incident close.

SLA credit calculation is automatic (Admin Dashboard → SLA Reports). CSM does not calculate credits manually.

---

## 11. Contract termination and data offboarding

Triggered by: non-renewal, early termination, or FORM exercising contract termination rights (no-go use case discovered post-signature).

### 11.1 Timeline

| Day | Action | Owner |
|---|---|---|
| 0 (notice received) | Confirm termination date; confirm data export window (30 days) | CSM |
| 0 | Disable new SCIM provisioning | enterprise-architect |
| 0–30 | Customer exports data via Admin Dashboard → Data Export (includes all user-consented data in JSON; excludes audit log which is FORM's compliance artefact) | `tenant_owner` |
| 30 | SSO revocation: all active sessions terminated; SCIM tokens revoked | enterprise-architect |
| 30 | **API key revocation:** all active rows in `tenant_api_keys` for this tenant are wiped; `api_key.revoked` with `reason: 'tenant_offboarding'` is emitted per active key (DEC-030, HIGH severity, 7-year retention, HMAC-chained in insertion order). Keys are revoked after SCIM tokens to prevent race between deprovisioning and API access. | enterprise-architect |
| 30 | Tenant soft-delete: user accounts deactivated, all RLS-governed data becomes inaccessible | enterprise-architect |
| 30–37 | Hard-delete cascade per `DATA_MODEL.md §12`: workouts, meals, coaching sessions, user profiles, media in R2 | enterprise-architect |
| 37 | Deletion confirmation letter sent (email + DocuSign) | compliance-officer |
| Post-termination | `audit_log_events` for this tenant are retained for 7 years (WORM, tamper-evident) per DEC-030 and SOC 2 requirements — this is non-negotiable and disclosed in the DPA |

**Exceptions to hard-delete:**
- `audit_log_events` — 7-year WORM retention (compliance requirement)
- Financial records (invoices, Order Forms) — 7-year retention per tax law
- DPA, MSA, and Order Form — 7-year retention per contract law

### 11.2 Audit events at termination (DEC-030 HMAC chain)

| Event | Severity | When |
|---|---|---|
| `api_key.revoked` (one per active key) | HIGH | Day 30 — after SCIM token revocation, before soft-delete; `reason: 'tenant_offboarding'`; `revoked_by: 'system/tenant_offboarding_job'`; see `docs/AUDIT_LOG_SCHEMA.md api_key.*` |
| `tenant.data_deleted` | HIGH | After hard-delete cascade completes |
| `tenant.deletion_confirmed` | STANDARD | After deletion confirmation letter sent |
| `admin.tenant_deprovisioned` | HIGH | When SSO/SCIM revoked and tenant set to inactive |

### 11.3 Early termination for no-go use case

If a no-go use case is discovered after contract execution:
1. Founder notified immediately
2. Contract termination notice issued per MSA termination-for-cause clause
3. Data offboarding proceeds on accelerated 14-day timeline
4. No service credits apply for the termination period
5. Incident documented in the compliance record — `privacy.no_go_escalation_activated` audit event (DEC-030, CRITICAL severity, 7yr retention)

---

## 12. CSM handoff (if CSM changes)

If the named CSM changes during an active contract:

1. Outgoing CSM documents in account CRM: current seat count, active success criteria, open issues, last QBR date, renewal date, any pending expansion discussions
2. Introductory email to customer from new CSM within 2 business days
3. 30-minute handoff call within 1 week
4. `admin.rbac_role_assigned` audit event updated if CSM had an admin role on the tenant

---

## Appendix A — Checklist templates

### A.1 Pre-launch checklist (customer copy)

```
FORM Enterprise — Pre-Launch Checklist
Account: [Company name]
CSM: [Name]
Contract start: [Date]

□ MSA signed
□ DPA signed
□ Order Form received
□ IdP admin identified: [Name / email]
□ IdP chosen: [Okta / Azure AD / Google Workspace / other]
□ FORM SAML or OIDC metadata shared with IdP admin
□ tenant_owner email(s) confirmed: [emails]
□ Pilot cohort identified: [approx. count]
□ Employee announcement plan agreed
□ Slack Connect channel opened: form-[slug]-shared
□ Success criteria agreed (90-day): [metric 1], [metric 2]
□ Privacy model walkthrough completed
```

### A.2 90-day QBR data pull (Admin Dashboard queries)

Pull these before the QBR call and paste into the summary doc:

| Metric | Admin Dashboard path |
|---|---|
| Total activated seats | Analytics → Activation → Last 90 days |
| D30 retention | Analytics → Cohorts → D30 (most recent cohort) |
| In-app NPS score | Analytics → NPS → Last 30 days |
| Sessions per active user (median) | Analytics → Engagement → Sessions/user |
| Feature split | Analytics → Features → Last 30 days |
| Open support tickets | Support → Open Cases |
| SLA uptime (last 3 months) | SLA Reports → Monthly |

### A.3 No-go use case escalation script

If a customer asks for individual-level data or proposes an excluded use case:

> "I want to flag that what you're describing falls outside FORM's permitted use cases, which are defined in our MSA and are a product principle we don't negotiate on. I'll need to escalate this to our founder before we can continue. I'll follow up within one business day."

Do not offer workarounds, do not say "we can look into it," do not escalate to engineering. Escalate to founder only.

---

*v0.2 (2026-06-09): §11.1 Contract termination timeline — API key revocation step added at Day 30 (MDD-P1-04): all active `tenant_api_keys` rows for the departing tenant are wiped immediately after SCIM token revocation; `api_key.revoked` (reason: `'tenant_offboarding'`) is emitted per active key (DEC-030 HIGH, 7-year retention, HMAC-chained in insertion order). §11.2 DEC-030 event table updated with `api_key.revoked` row. `docs/AUDIT_LOG_SCHEMA.md api_key.revoked` reason enum updated to include `tenant_offboarding`. Closes MDD-P1-04 in `docs/SOC2_READINESS.md`. SOC 2 CC6.4 (credential lifecycle — departed tenant API keys no longer valid post-30d). enterprise-architect + compliance-officer.*

*v0.1 (2026-06-07): Initial onboarding playbook. Covers: pre-launch checklist (commercial + security review delivery + technical readiness), contract week kick-off, tenant provisioning with DEC-030 HMAC chain events (`tenant.created`, `tenant.sla_tier_set`, `admin.rbac_role_assigned`), SSO SAML/OIDC + SCIM provisioning steps, pilot rollout + announcement templates, admin dashboard training with privacy floor script, 30/60/90-day success criteria by tier, QBR agenda, expansion (seat expansion, white-label upgrade checklist, multi-year renewal), incident communication protocol, contract termination data offboarding timeline with hard-delete cascade per DATA_MODEL.md §12 and DEC-030 termination audit events, CSM handoff procedure, Appendix A checklists. Privacy floor enforcement language throughout. No-go use case escalation path (ENTERPRISE.md §no-go). Owner: customer-success + enterprise-architect + compliance-officer.*
