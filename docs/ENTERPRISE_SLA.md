# FORM · Enterprise Service Level Agreement v1.2

> Customer-facing SLA Terms. Attached to the Master Service Agreement (MSA) as **Exhibit B — Service Level Agreement**.
> Owner: `compliance-officer` + `enterprise-architect`. Review: annually or after any P0 incident that results in a credit dispute.
> Internal engineering spec for how availability is measured and credits are calculated: `docs/OBSERVABILITY.md §23`.
> Incident response procedures: `docs/INCIDENT_RESPONSE.md`. Pricing tiers: `docs/ENTERPRISE.md §5`.

---

## TL;DR

FORM commits to **99.9% monthly uptime** per tenant, **P0 resolution within 4 hours**, and a **72-hour GDPR breach notification** to the supervisory authority. Customers receive automated credit reports; no claim required for credits ≥ 5% of MRR. HR never sees individual user data — that floor is non-negotiable regardless of contract value.

---

## 1. Definitions

| Term | Meaning |
|---|---|
| **Agreement** | The Master Service Agreement between FORM and the Customer, of which this SLA is Exhibit B. |
| **Availability** | The percentage of calendar minutes in a month during which the Covered Services are reachable and returning correct responses, measured per tenant. |
| **Covered Downtime** | Minutes classified as down or degraded under §3.3, excluding SLA Exclusions (§5). |
| **Monthly Recurring Revenue (MRR)** | The monthly fee invoiced to Customer under the Agreement for the affected month. For annual-contract customers, MRR = Annual Contract Value ÷ 12. |
| **Service Credit** | A credit applied to the following month's invoice, expressed as a percentage of MRR for the month in which the SLA was missed. Credits are not redeemable as cash. |
| **Tenant** | The Customer's isolated environment in the FORM multi-tenant platform, identified by a unique `tenant_id`. |
| **Incident Commander (IC)** | The FORM on-call engineer responsible for coordinating response to a declared incident. |
| **Privacy Floor** | The set of non-negotiable data-access restrictions in §13, which no contract term can override. |
| **SOC 2 Type II** | Independent auditor attestation that FORM's controls operated effectively over a minimum 6-month observation period. Target: Month 12 after enterprise launch. |

---

## 2. Service Tiers

Enterprise customers are assigned to a tier at contract execution. The SLA tier governs credit percentages, maintenance notice periods, and support channel entitlements.

| Tier | Seats | Monthly seat price | SLA tier | Support channel |
|---|---|---|---|---|
| **Starter** | 50–200 | $12/seat | Standard | Email (24h response) |
| **Growth** | 200–1,000 | $9/seat | Standard | Slack Connect + email (4h response business hours) |
| **Enterprise** | 1,000+ | $6–8/seat (negotiated) | Premium (optional addendum) | Dedicated phone + Slack + email (1h response 24×7) |

Pricing is annual contract (month-to-month not available for enterprise). Discounts: 15% for 2-year, 25% for 3-year, 10% for annual upfront (non-2/3-year).

---

## 3. Service Availability SLA

### 3.1 Uptime Commitment

| Tier | Monthly Availability Commitment | Covered Downtime Budget |
|---|---|---|
| **Standard** (Starter, Growth) | **99.9%** | 43.8 minutes per 30-day calendar month |
| **Premium** (Enterprise addendum) | **99.95%** | 21.9 minutes per 30-day calendar month |

Availability is measured **per tenant**. A platform-wide incident affects all tenants' SLA clocks simultaneously; a bug limited to one tenant's SSO configuration is an incident for that tenant only.

### 3.2 Covered Services

All five services must be simultaneously unavailable for a minute to count as full downtime. A single-service outage counts as a **partial outage** at 50% weight against the downtime budget.

| Service | What is measured |
|---|---|
| **API Gateway** | HTTP availability at `GET /health` (Cloudflare Workers) |
| **Authentication** | Successful token exchange (Supabase Auth / WorkOS SSO) |
| **Workout Logging** | HTTP 201 from `POST /api/v1/workouts/sets` |
| **AI Coaching (Victor)** | HTTP 200 from coaching turn probe |
| **Admin Dashboard API** | HTTP 200 from `GET /api/v1/admin/health` |

**Partial outage example:** AI Coaching is unavailable for 20 minutes; API Gateway, Auth, Workout Logging, and Admin Dashboard remain reachable → 10 covered minutes charged against the budget (20 × 50%).

### 3.3 Measurement Methodology

Availability is derived from two independent sources, reconciled at month-end. The more conservative value (lower availability) is used if sources disagree by > 0.05%.

- **Primary:** Better Stack synthetic probes running from three regions (us-east, eu-central, ap-southeast). A minute is **down** when ≥ 2 regions report failure simultaneously; a minute is **degraded** when exactly 1 region fails (50% weight). Probe frequency: 30 seconds per probe.
- **Secondary (corroborating):** Cloudflare Analytics Engine hourly availability ratios, filtered by `tenant_id`.

The monthly reconciliation is recorded as a DEC-030 audit event (`sla.measurement_reconciled`) with both source values. The final measurement is available in the Admin Dashboard → SLA Uptime Report panel (real-time during the month) and delivered as a PDF on the 3rd business day of the following month.

Full technical specification: `docs/OBSERVABILITY.md §23.3`.

### 3.4 Service Credit Schedule

Credits are applied to the **following month's invoice** as a line-item reduction. No formal claim is required for credits ≥ 5% of MRR — FORM applies them automatically.

**Standard tier (99.9% commitment):**

| Monthly Availability | Covered Downtime Used | Service Credit |
|---|---|---|
| ≥ 99.9% | 0 – 43.8 min | **0%** — SLA met |
| 99.0% – < 99.9% | 43.8 min – 7.2 h | **5%** of monthly invoice |
| 95.0% – < 99.0% | 7.2 h – 36 h | **15%** of monthly invoice |
| 90.0% – < 95.0% | 36 h – 72 h | **25%** of monthly invoice |
| < 90.0% | > 72 h | **50%** of monthly invoice |

**Premium tier (99.95% commitment):**

| Monthly Availability | Covered Downtime Used | Service Credit |
|---|---|---|
| ≥ 99.95% | 0 – 21.9 min | **0%** — SLA met |
| 99.5% – < 99.95% | 21.9 min – 3.6 h | **5%** of monthly invoice |
| 99.0% – < 99.5% | 3.6 h – 7.2 h | **10%** of monthly invoice |
| 95.0% – < 99.0% | 7.2 h – 36 h | **20%** of monthly invoice |
| < 95.0% | > 36 h | **50%** of monthly invoice |

**Partial-outage multiplier:** Credits earned during partial-outage windows are issued at 50% of the tier rate. Example: Growth tier customer (Standard), 2-hour AI Coaching-only outage → 1h counted against budget (50% weight) → if that pushes into the 99.0–99.9% tier → 5% × 50% = 2.5% credit.

**Credit calculation example:**

```
Customer: Growth tier, 300 seats at $9/seat = $2,700 MRR
Incident: May, 95 minutes full-platform outage
  → 95 minutes > 43.8-minute budget
  → Availability: (43,200 − 95) / 43,200 = 99.78%
  → Tier: 99.0–99.9% → 5% credit
  → Credit: 5% × $2,700 = $135.00 applied to June invoice
```

### 3.5 Credit Limitations

- **Cap:** Total credits in any calendar month ≤ 50% of that month's invoice.
- **Form:** Credits are applied against future invoices only; they are not redeemable as cash, bank transfer, or offset against other agreements.
- **Stacking:** Credits from multiple incidents in the same month are aggregated, subject to the 50% cap.
- **Exclusivity:** Service Credits are the Customer's sole and exclusive remedy for SLA breaches. They do not affect indemnification rights under the Agreement for data breaches.
- **Dispute window:** Customers may dispute a credit calculation within **30 days** of the month-end SLA report delivery date. Disputes submitted after 30 days are not eligible for review.

### 3.6 Credit Claim Process

| Step | Actor | Action | Timeline |
|---|---|---|---|
| 1 | FORM (automated) | Month-close Worker reconciles sources; writes `sla_monthly_reports` row | 00:30 UTC on 1st of each month |
| 2 | FORM (automated) | If `credit_tier > 0`: emits `sla.credit_calculated` DEC-030 event; sends email to `tenant_billing_email` | Same day |
| 3 | FORM (`compliance-officer`) | Verifies calculation; approves or adjusts within 5 business days; emits `sla.credit_approved` | ≤ 5 business days |
| 4 | FORM (billing) | Applies credit line item to following month's invoice | Next invoice cycle |
| 5 | Customer (optional) | Disputes via Admin Dashboard → SLA Reports → "Dispute this report" | Within 30 days |
| 6 | FORM (`compliance-officer` → founder) | Escalates unresolved disputes; final determination | ≤ 10 business days from dispute receipt |

Credits are **auto-applied** by default across all tiers. Starter and Growth tier customers who prefer a manual-claim workflow (for example, to negotiate credits as contract goodwill) may switch to Manual Credit Mode in Admin Dashboard → Billing → Credit Preferences. In Manual Credit Mode, a `sla.credit_pending_claim` notification is sent to `tenant_billing_email`; the credit expires unclaimed after 90 days. Default remains auto-apply; Growth/Enterprise auto-apply cannot be disabled. *(OQ-SLA-02 resolved — see §18.)*

---

### 3.7 SCIM Deprovisioning and Provisioning Latency

SCIM deprovisioning latency is a security-critical commitment. An employee who retains active FORM sessions after their IdP account is deactivated represents an access control gap. FORM commits to the following latency targets, measured from the moment FORM's SCIM API receives the request.

| Operation | P99 Target | SLA-Breach Threshold | Measurement source |
|---|---|---|---|
| SCIM DELETE / PATCH `active: false` → user deactivated | < 30 seconds | > 2 minutes | `user.scim_deprovisioned` DEC-030 timestamp − SCIM request receipt timestamp |
| SCIM DELETE / PATCH `active: false` → all active sessions revoked | < 60 seconds | > 5 minutes | `user.scim_deprovisioned.session_revocation_completed_at` DEC-030 field |
| SCIM POST → user account created and accessible | < 60 seconds | > 10 minutes | `user.scim_created` DEC-030 timestamp − SCIM request receipt timestamp |

**SLA-breach consequence:** A deprovisioning event that exceeds the breach threshold for **session revocation** (> 5 minutes) is automatically classified as a **P1 incident** per §6.1 regardless of any broader platform outage. The Customer's `tenant_security_contact` is notified within 30 minutes. This ensures security-critical deprovisioning failures are never silently absorbed as low-severity anomalies.

**Measurement probe:** Better Stack synthetic probe **S-014** (`scim-deprovision-latency`) fires a SCIM DELETE on a canary test user in each deployed region every 15 minutes.
- **AL-SCIM-LAT-01:** P99 > 30 seconds over a 1-hour rolling window → PagerDuty P2 (`form-platform`).
- **AL-SCIM-LAT-02:** Any single event > 5 minutes → PagerDuty P1 (`form-platform` + `form-customer-success`); P1 incident auto-opened.

**Exclusions:** The SCIM latency SLA does not apply when delay is caused by Customer-side SCIM credential expiry, Customer IdP rate limiting on outbound SCIM requests, or Customer-side network connectivity issues between the IdP and FORM's SCIM endpoint (all §5 SLA Exclusions).

*(OQ-SLA-03 resolved — see §18.)*

---

### 3.8 CAEP/SSF Real-Time Session Control

Available as an opt-in commitment for tenants whose Identity Provider supports CAEP/RISC PUSH delivery. Incorporated via **Addendum 6 — CAEP/SSF Real-Time Session Control SLA Addendum**, selected in the Order Form.

**Eligible IdPs:** Okta, Microsoft Entra ID, Google Workspace OIDC. OneLogin and generic SAML 2.0 providers are **not** eligible — they receive the baseline JWT TTL (≤ 15 minutes) session-revocation guarantee with no addendum.

**Revocation latency commitment (Addendum 6 tenants only):**

| Trigger event types | Commitment | Measurement |
|---|---|---|
| `session-revoked`, `credential-change`, `account-disabled`, `account-enabled`, `account-purged`, `device-compliance-change`, Google RISC `hijacking` | < 60 seconds from IdP emitting the Security Event Token (SET) to FORM revoking access | `sso.caep_event_received` DEC-030 timestamp → `sso.caep_session_revoked` DEC-030 timestamp delta |

**Baseline (non-CAEP) revocation — all tenants:** Session revocation occurs at the next JWT validation check, within the maximum JWT TTL of **≤ 15 minutes**. This is SOC 2 CC6.3 compliant and requires no addendum. The CAEP addendum is additive — it does not replace the JWT TTL floor.

**Prerequisite — active CAEP stream:** The < 60-second commitment applies only while the CAEP stream is in `active` status (visible in Admin Dashboard → SSO Settings → CAEP Status). If the stream enters `error` or `inactive`, FORM's revocation falls back to the ≤ 15-minute JWT TTL baseline until stream health is restored. FORM monitors stream health via dead-man's switch alert:

- **AL-CAEP-03:** No CAEP events received on an active stream for 4 consecutive hours during business hours (08:00–22:00 UTC) → PagerDuty P2 (`form-platform`) + CSM notification to `tenant_security_contact`. FORM investigates and notifies Customer of stream status within 4 hours.

After SAML certificate rotation, FORM automatically re-registers the CAEP stream within 5 minutes (pg_cron `caep_reregister_sweep`, `*/5 * * * *`) per DEC-072. Re-registration does not interrupt the baseline JWT TTL revocation path during the re-registration window.

**PUSH delivery only:** SSF PULL/poll mode is not included in this addendum. During an IdP-side PUSH delivery outage (not caused by FORM), revocation falls back to the baseline JWT TTL. No SLA credit accrues for IdP-side PUSH delivery failures — this is an SLA Exclusion under §5.

**SLA credit for CAEP breach:** Revocation latency breaches attributable solely to FORM-side processing failures (not IdP delivery latency or IdP PUSH outages) may be raised as SLA disputes per §3.5 and MSA Article 18.3. Credits are calculated under §3.4 and remain the Customer's sole remedy per MSA §15.4.

**DEC-030 audit events (HMAC-chained):**

| Event | Severity | Retention | When emitted |
|---|---|---|---|
| `sso.caep_event_received` | STANDARD | 7 years | FORM's CAEP receiver endpoint receives a SET from IdP |
| `sso.caep_session_revoked` | HIGH | 7 years | FORM completes session revocation following the SET |
| `sso.caep_stream_health_breach` | HIGH | 3 years | AL-CAEP-03 fires (stream silent for 4h during business hours) |
| `sso.caep_reregistration_queued` | STANDARD | 7 years | Cert-rotation-triggered stream re-registration queued |

SOC 2 evidence artefact: **CC6-E-CAEP-002** (CC6.3 — revocation latency measurements; quarterly export). Full specification: `docs/SSO_SCIM_IMPLEMENTATION.md §23`. MSA Addendum 6 template: `docs/MSA_TEMPLATE.md §Addendum 6`. CAEP-eligible IdP onboarding checklist: `docs/ENTERPRISE_ONBOARDING.md §2.3`.

*(Addendum 6 selection required — see §17 checklist item 11.)*

---

### 3.9 SSO Login Availability (Per-Tenant SSO-SLO-01)

Per DEC-073 (2026-06-20), SSO login success is measured **per tenant** as a distinct SLA dimension supplementing the overall uptime commitment in §3.1. Implementation spec: `docs/OBSERVABILITY.md §49`.

**SSO-SLO-01 — per-tenant login success rate:**

| Metric | Target | Measurement window | SLA credit impact |
|---|---|---|---|
| SSO login success rate (`sso.login_succeeded` / (`sso.login_succeeded` + `sso.login_failed`) per `tenant_id`) | ≥ 99% | 15-minute rolling window | Per-tenant SLA credit under §3.4; measured per tenant per §3.1 |

An SSO-SLO-01 breach is a separate incident from a broader platform availability breach. The Customer's `tenant_security_contact` is notified within 30 minutes if the breach is FORM-side (not IdP-side). Customer-side IdP misconfiguration (e.g., expired SAML certificate, incorrect ACS URL, SCIM token expiry) that degrades the tenant's login success rate below 99% is classified under §5 SLA Exclusions and does not generate credits.

**SSO-SLO-01b — fleet-wide companion signal (operational only — no SLA credit):**

When ≥ 3 distinct tenants with active SSO simultaneously breach SSO-SLO-01 within any 15-minute rolling window, FORM treats this as a potential platform-level regression:

| Signal | Response | SLA credit impact |
|---|---|---|
| AL-SSO-FLEET-01: ≥ 3 tenants breach SSO-SLO-01 simultaneously | PagerDuty P1 (`form-platform`) → IC + security-engineer escalation; P1 incident opened per §6 | **None** — `sla_credit_impact: 'none'` hard invariant encoded in DEC-030 event payload (DEC-073) |

SSO-SLO-01b distinguishes correlated platform failure (e.g., Cloudflare Worker bug, WorkOS upstream incident) from coincident independent IdP issues. Any proposal to attach SLA credits to SSO-SLO-01b requires a new DECISION_LOG entry and compliance-officer sign-off per DEC-073 hard invariant.

**DEC-030 audit event:**

| Event | Severity | Retention | When emitted |
|---|---|---|---|
| `siem.sso_fleet_health_breach` | HIGH | 3 years | AL-SSO-FLEET-01 fires (≥ 3 tenants breach SSO-SLO-01 simultaneously) |

SOC 2 evidence artefact: **SSO-FLEET-E-001** (CC7.2/CC7.3 — quarterly PagerDuty export of AL-SSO-FLEET-01 firings; first filing M11). Full specification: `docs/OBSERVABILITY.md §49`. Implementation: pg_cron job 36 (`sso_fleet_health_check`, `*/5 * * * *`, `form_audit` role).

*(OQ-SLA-01 resolved — see §18.)*

---

## 4. Planned Maintenance

FORM performs planned maintenance during low-traffic windows (typically 02:00–06:00 UTC Sunday). Planned maintenance is **excluded** from Covered Downtime provided:

| Requirement | Standard tier | Premium tier |
|---|---|---|
| Advance notice to `tenant_ops_email` | ≥ 72 hours | ≥ 7 days |
| Maximum window per occurrence | 4 hours | 2 hours |
| Maximum windows per calendar month | 2 | 1 |
| Total excluded time per calendar month | ≤ 8 hours | ≤ 4 hours |

Maintenance windows exceeding these limits are **not excluded** — the excess time counts as Covered Downtime.

Emergency maintenance required to address a P0 security incident is excluded from Covered Downtime regardless of notice period, provided FORM notifies the Customer within 1 hour of the emergency maintenance start.

---

## 5. SLA Exclusions

The following do not count toward Covered Downtime:

| Category | Notes |
|---|---|
| **Upstream provider outage** | Cloudflare, Supabase, Anthropic, ElevenLabs, WorkOS outages outside FORM's control. FORM provides the provider's status page link in the post-incident report. |
| **Planned maintenance** | Per §4 above. |
| **Customer-side IdP misconfiguration** | Failures traceable to the Customer's Okta/Azure AD misconfiguration, firewall rules blocking FORM IP ranges, or SCIM credential expiry not renewed by the Customer. Evidence: SSO error code `idp_configuration_error` in the Admin Dashboard audit log. |
| **Customer network** | Failures in the Customer's own network, VPN, or DNS infrastructure. |
| **Force majeure** | Internet backbone failures, national infrastructure incidents, acts of war or terrorism, natural disasters. |
| **Beta and preview features** | Features explicitly labelled `[BETA]` in the Admin Dashboard carry no SLA coverage. |
| **Probe false positives** | If synthetic probes report failure but Cloudflare Analytics shows zero real-user errors in the same 5-minute window, FORM may reclassify the minute as a false positive. Reclassification requires `security-engineer` approval and is recorded as a DEC-030 `sla.probe_false_positive` event. |
| **Customer-requested configuration changes** | Downtime caused by configuration changes requested and approved in writing by the Customer. |

---

## 6. Incident Response SLAs

These SLAs govern how quickly FORM acknowledges, escalates, and resolves incidents that affect Customer tenants. Full classification criteria and runbooks: `docs/INCIDENT_RESPONSE.md`.

### 6.1 Incident Classification

| Severity | Scope | Data exposure | Platform impact |
|---|---|---|---|
| **P0** (Critical) | Any confirmed user harm | Any PII or health data confirmed or suspected exposed | Full platform or critical-path down |
| **P1** (High) | Significant user cohort impacted; Enterprise admin blocked | No confirmed exposure but plausible access path | Partial outage; Enterprise feature unavailable |
| **P2** (Medium) | Degraded experience; non-critical path | No data exposure path identified | Degraded performance; SLA at risk |
| **P3** (Low) | Cosmetic; single user; edge case | None | None |

### 6.2 Response Time Commitments

| Severity | Acknowledge | Initial assessment | Customer notification (Enterprise tenants) | Target resolution |
|---|---|---|---|---|
| **P0** | < 15 min | < 1 h | < 1 h (direct phone + email) | < 4 h |
| **P1** | < 30 min | < 2 h | < 2 h (Slack Connect + email) | < 8 h |
| **P2** | < 2 h | < 4 h | < 4 h (email) | < 24 h |
| **P3** | Next business day | — | On request | Sprint cycle |

*Resolution time* is the target, not a guarantee; complex incidents may require longer resolution while FORM provides hourly status updates.

**GDPR breach notification:** If a P0 incident involves confirmed exposure of personal data, FORM notifies the applicable supervisory authority within **72 hours** of FORM's confirmation of the breach. Affected data subjects are notified without undue delay if the breach is likely to result in a high risk to their rights. Customer receives notification within **1 hour** of FORM's breach confirmation regardless of GDPR notification timing.

### 6.3 Communication Channels by Tier

| Tier | P0 channel | P1 channel | P2/P3 channel | Status page |
|---|---|---|---|---|
| **Starter** | Email + status page | Email | Email | status.form.coach |
| **Growth** | Slack Connect + email | Slack Connect + email | Email | status.form.coach |
| **Enterprise** | Direct phone + Slack + email | Slack + email | Email | status.form.coach |

All enterprise tenants must designate a `tenant_ops_email` and `tenant_security_contact` in the Admin Dashboard before the SLA takes effect. FORM cannot guarantee notification timelines if these are not configured.

---

## 7. Support Tier SLAs

Support SLAs apply during business hours unless otherwise stated. Business hours: Monday–Friday 09:00–18:00 CET / EET.

| | Starter | Growth | Enterprise |
|---|---|---|---|
| **Seats** | 50–200 | 200–1,000 | 1,000+ |
| **Support channel** | Email (`support@form.coach`) | Slack Connect + email | Dedicated Slack + direct phone + email |
| **First response — P0** | < 1 h (24×7) | < 1 h (24×7) | < 30 min (24×7) |
| **First response — P1** | < 4 h (biz hours) | < 2 h (biz hours) | < 1 h (24×7) |
| **First response — P2** | < 24 h | < 8 h | < 4 h |
| **First response — P3** | Next business day | Next business day | Same business day |
| **Dedicated CSM** | No | Yes | Yes |
| **Quarterly Business Review (QBR)** | No | Yes | Yes (+ monthly health check) |
| **Escalation path** | support → on-call | CSM → IC | CSM → IC → Founder (P0) |
| **Pentest summary** | On request (NDA) | On request (NDA) | Proactively shared annually |
| **Compliance attestation** | On request | Quarterly | Quarterly |

---

## 8. Data Residency Commitments

FORM processes and stores data in the region selected at tenant creation. Region cannot be changed post-deployment without a full data migration (scheduled maintenance, separate SoW).

| Region label | Supabase region | Cloudflare PoP | Notes |
|---|---|---|---|
| **EU (default)** | `eu-central-1` (Frankfurt) | EU-Central | Satisfies GDPR Art. 44; recommended for all EU customers |
| **EU-West** | `eu-west-1` (Dublin) | EU-West | Available for UK/IE customers; Brexit SCC required |
| **US** | `us-east-1` (N. Virginia) | US-East | Available for US customers; not available for EU-only DPA |

**Cross-region prohibition:** Customer data is never replicated across region boundaries for production purposes. Disaster recovery replicas remain within the same region. Multi-region failover is on the post-Series A roadmap; customers requiring cross-region redundancy should discuss this as a contract term.

**Sub-processor region commitments:** All sub-processors that handle Customer personal data are required to process data in the contracted region. Exceptions for inherently global services (e.g., Anthropic API, ElevenLabs) are documented in `docs/SUBPROCESSORS.md` with applicable transfer mechanism (SCC Module 2).

---

## 9. Security Commitments

| Commitment | Frequency | Shared with Customer |
|---|---|---|
| **Penetration test** | Annual (external firm) | Executive summary (finding counts by severity + remediation plan) under NDA |
| **SOC 2 Type II report** | Annual (target: Month 12 post-enterprise launch) | Available under NDA on request for contracts > $50k ACV |
| **Vulnerability disclosure** | Continuous | Material vulnerabilities affecting Customer data disclosed within 72 h of FORM confirmation |
| **Key rotation** | Per schedule in `docs/KEY_ROTATION.md` | Rotation events logged in DEC-030 audit trail; Customer can verify via audit log export |
| **MFA enforcement** | Always | Tenant-level MFA policy configurable in Admin Dashboard; FORM enforces for all internal staff |
| **Audit log availability** | 7-year retention | HMAC-chained, tamper-evident; exportable via webhook or hourly R2/S3/GCS batch |

FORM commits to maintaining the security controls documented in `docs/SOC2_READINESS.md` throughout the contract term and to notifying Customers of any material deviation within 5 business days.

---

## 10. Change Management

### 10.1 Maintenance Notice

See §4. Planned maintenance notices are sent to `tenant_ops_email`.

### 10.2 Sub-processor Changes (GDPR Art. 28)

FORM notifies Customers **30 days in advance** of any new sub-processor addition or material change to an existing sub-processor's role, data types processed, or processing location. Customers have the right to object per GDPR Art. 28(2). If a Customer objects and FORM cannot accommodate the objection without the sub-processor, either party may terminate the affected services without penalty.

Sub-processor removals are notified promptly (no 30-day advance requirement) along with confirmation of data deletion.

Current sub-processor register: `docs/SUBPROCESSORS.md`. Published at `form.coach/legal/sub-processors` (target: before first enterprise pilot).

### 10.3 Material Platform Changes

Material changes to the FORM platform that may affect Customer workflows (e.g., Admin Dashboard API v-breaking changes, SSO metadata URL changes, SCIM endpoint changes) are communicated with **minimum 60 days notice** via:
1. Email to `tenant_ops_email` and `tenant_it_contact`.
2. In-app banner in the Admin Dashboard.
3. Entry in the FORM changelog (`CHANGELOG.md` — public at `github.com/Rbit27/form`).

Cosmetic changes, performance improvements, and backward-compatible additions do not require advance notice.

---

## 11. Customer Obligations (CUECs)

For FORM's controls to be effective, Customers must implement the following. These will appear as CUECs in the issued SOC 2 Type II report.

| Obligation | Owner at Customer | Rationale |
|---|---|---|
| Designate and maintain `tenant_ops_email`, `tenant_security_contact`, and `tenant_billing_email` in the Admin Dashboard | IT / Security | FORM cannot guarantee notification SLAs without these |
| Enforce MFA for all users accessing the Admin Dashboard | IT | FORM enforces MFA at the platform level; Customer must configure IdP-side MFA policy for SSO |
| Rotate SCIM API tokens at least annually (or immediately on suspected compromise) | IT | Expired or compromised SCIM tokens are excluded from SLA coverage per §5 |
| Review and action FORM security advisories within the timelines specified in the advisory | Security | FORM is not liable for incidents caused by Customer's failure to act on advisories within specified timelines |
| Notify FORM promptly of any suspected compromise of Customer-side IdP credentials | Security | Required to initiate the coordinated incident response in `docs/INCIDENT_RESPONSE.md R-12` |
| Maintain accurate seat count; do not share individual accounts across employees | HR / People | Per-seat licensing; shared accounts create audit trail integrity issues |
| Do not attempt to access other tenants' data | Legal / Security | Any attempt is logged as a HIGH-severity audit event and may result in immediate suspension |

---

## 12. Audit Rights

**Customer audit rights:**

- Customers may request a copy of the FORM SOC 2 Type II report (when available) and the most recent penetration test executive summary (under NDA) once per contract year.
- Customers with contracts > $100k ACV may request a meeting with `compliance-officer` to walk through the SOC 2 report and controls evidence, once per contract year.
- Customers may export their tenant's HMAC-chained audit log at any time via the Admin Dashboard → Audit Log Export, the webhook integration, or the R2/S3/GCS batch export pipeline.
- Customers may verify the audit log HMAC chain integrity using the chain verification utility in `docs/AUDIT_LOG_SCHEMA.md §HMAC chaining`.

**FORM audit rights:**

- FORM may audit Customer's compliance with the Acceptable Use Policy (`docs/ACCEPTABLE_USE_POLICY.md`) and Customer Obligations (§11) with 5 business days notice.
- FORM may immediately suspend a tenant that is found to be in material breach of the Agreement pending investigation.

---

## 13. Privacy Floor (Non-Negotiable)

These restrictions apply regardless of contract value, customer request, or contractual language. They are floors set by `clinical-safety` and `compliance-officer` and cannot be overridden by any commercial agreement.

| Restriction | Rationale |
|---|---|
| **HR cannot see individual user data** | FORM shows HR/People leads aggregated metrics only (% active, % activated, opt-in NPS) |
| **Manager-direct-report drill-down is never available** | Would constitute workplace surveillance; creates legal exposure for Customer |
| **ED-screening data is never aggregated to the employer** | GDPR Art. 9 special category (health); clinical-safety VETO |
| **Body composition data is never aggregated to the employer** | GDPR Art. 9 special category; potential body-image harm |
| **Mental health data is never aggregated to the employer** | GDPR Art. 9 special category; workplace discrimination risk |
| **Employee can revoke company association at any time** | Right to data portability and erasure; GDPR Arts. 17, 20 |

A Customer who requests removal of any Privacy Floor restriction will be declined. FORM will lose the deal rather than create exceptions to these controls. The floor is documented in `docs/ENTERPRISE.md §7`.

---

## 14. SLA Remedies and Liability Limitations

- **Sole remedy:** Service Credits under §3.4–3.6 are the Customer's sole and exclusive remedy for SLA availability breaches. Credits do not waive or limit Customer's rights under the Agreement's indemnification, IP, or confidentiality provisions.
- **Liability cap:** FORM's total liability for SLA-related claims in any 12-month period is capped at 12 months of MRR paid by the Customer.
- **No cascading damages:** FORM is not liable for lost revenue, lost profits, or consequential damages arising from SLA breaches unless caused by FORM's gross negligence or willful misconduct.
- **Dispute resolution:** SLA disputes not resolved within 20 business days escalate to binding arbitration per the governing law clause of the Agreement.

---

## 15. DEC-030 Audit Events (SLA Lifecycle)

All SLA-lifecycle events are recorded in the HMAC-chained `audit_log` table per `docs/AUDIT_LOG_SCHEMA.md`. Customers can verify these events via their Admin Dashboard audit log export.

| Event | Severity | Retention | When emitted |
|---|---|---|---|
| `sla.incident_opened` | HIGH | 7 years | When an incident affecting SLA coverage is declared |
| `sla.incident_updated` | STANDARD | 7 years | Each status update during an open incident |
| `sla.incident_closed` | HIGH | 7 years | When an incident is resolved; includes duration and root cause link |
| `sla.measurement_reconciled` | STANDARD | 7 years | Month-end: both source values recorded; conservativer value selected |
| `sla.credit_calculated` | HIGH | 7 years | When month-close confirms credit is owed |
| `sla.credit_approved` | HIGH | 7 years | When `compliance-officer` approves the credit amount |
| `sla.credit_applied` | HIGH | 7 years | When credit line item appears on invoice |
| `sla.dispute_opened` | HIGH | 7 years | When Customer submits a credit dispute |
| `sla.dispute_resolved` | HIGH | 7 years | Final determination on a dispute |
| `sla.probe_false_positive` | HIGH | 7 years | When a synthetic probe minute is reclassified; requires `security-engineer` approval |
| `sla.maintenance_window_opened` | STANDARD | 3 years | When a planned maintenance window begins |
| `sla.maintenance_window_closed` | STANDARD | 3 years | When a planned maintenance window ends |

All events carry `tenant_id`, `actor_id`, `actor_type`, timestamp, and HMAC chain fields per the schema in `docs/AUDIT_LOG_SCHEMA.md`.

---

## 16. SOC 2 Evidence Mapping

| SOC 2 criterion | Control covered by this document |
|---|---|
| **A1.1** — Availability commitments documented | §3.1 (uptime commitment), §3.3 (measurement methodology), §3.4 (credit schedule) |
| **A1.2** — Availability monitored | §3.3 (Better Stack + Cloudflare dual-source), §3.6 (automated month-close) |
| **A1.3** — Recovery and continuity | §4 (maintenance policy), §6.2 (P0 resolution target < 4 h) |
| **CC6.3** — Logical access terminated on security events | §3.8 (CAEP/RISC real-time revocation < 60 s for PUSH-capable IdPs; baseline JWT TTL ≤ 15 min for all tenants; CC6-E-CAEP-002 quarterly evidence export) |
| **CC6.4** — Access revocation timely on personnel changes | §3.7 (SCIM deprovisioning P99 < 60 s; > 5 min breach → auto P1 incident) |
| **CC7.2** — Incidents detected and responded to | §6 (incident response SLAs), §15 (DEC-030 audit trail), §3.7 (AL-SCIM-LAT-01/02), §3.9 (SSO-SLO-01 per-tenant + SSO-SLO-01b fleet alert AL-SSO-FLEET-01; SSO-FLEET-E-001 quarterly evidence) |
| **CC7.3** — Incidents communicated | §6.2–6.3 (customer notification timelines), §6.2 (GDPR 72h), §3.8 (AL-CAEP-03 stream health → CSM notification within 4h), §3.9 (AL-SSO-FLEET-01 P1 → IC + security-engineer escalation) |
| **CC9.1** — Vendor/sub-processor management | §8 (data residency), §10.2 (30-day sub-processor notice) |
| **P4.0** — Privacy notice and communication | §13 (Privacy Floor), `docs/PRIVACY_POLICY.md` |
| **P8.0** — Privacy requests handled (DSARs) | §19 (DSAR SLAs: 24 h provision, 30-day erasure cert, self-serve portability) |
| **P8.1** — Personal info provided on request | §19.1–19.2 (data export and erasure commitments), §19.5 (DEC-030 DSAR events) |

---

## 17. Implementation Checklist

#### P0 — Before first enterprise pilot goes live (M4)

| # | Task | Owner | Priority | Definition of Done |
|---|---|---|---|---|
| 1 | Register all 12 SLA DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md`; deploy `emit-audit-event` Worker endpoint with Zod schema validation | data-engineer + platform-engineer | **P0** | Event types in AUDIT_LOG_SCHEMA.md; events emittable from Workers; test events in staging audit log |
| 2 | Implement `POST /api/v1/admin/sla/current` and `GET /api/v1/admin/sla/history` Admin Dashboard API endpoints with tenant RLS isolation (see `docs/OBSERVABILITY.md §23.6`) | platform-engineer | **P0** | Endpoints return correct per-tenant data; RLS verified by cross-tenant query test |
| 3 | Confirm `tenant_ops_email`, `tenant_security_contact`, `tenant_billing_email` fields exist in `tenants` table and are required during SSO onboarding | enterprise-architect | **P0** | Fields in schema; Admin Dashboard onboarding wizard enforces completion; blank → onboarding blocked |
| 4 | Configure Better Stack probes S-001 through S-013 for enterprise tenants; verify probe labels carry `tenant_id` | devops-lead | **P0** | Probes running; `tenant_id` label present in Better Stack monitor metadata |

#### P1 — Before SLA goes into effect for first customer (M5)

| # | Task | Owner | Priority | Definition of Done |
|---|---|---|---|---|
| 5 | Deploy month-close Worker (`workers/sla/month-close.ts`) with reconciliation logic and automated `sla.credit_calculated` event emission | platform-engineer + devops-lead | **P1** | Worker runs on 1st of each month at 00:30 UTC; dry-run on staging confirms correct credit-tier output for test tenant |
| 6 | Implement monthly PDF SLA report generation via pg_cron + Resend; confirm delivery to `tenant_billing_email` by 3rd business day | platform-engineer | **P1** | PDF delivered to test tenant billing email; SHA-256 hash of Better Stack report included |
| 7 | Legal review of §13 Privacy Floor and §14 Liability Limitations — confirm alignment with MSA template | compliance-officer + outside counsel | **P1** | Counsel-approved MSA template includes this SLA as Exhibit B; Privacy Floor clauses in MSA |
| 8 | Publish sub-processor list at `form.coach/legal/sub-processors` (closes SOC2 gap P1 item in `docs/SOC2_READINESS.md §25`) | compliance-officer | **P1** | Page live with current sub-processor list; "last updated" date; 30-day notice mechanism confirmed |

#### P2 — Post-Series A / before enterprise GA at scale

| # | Task | Owner | Priority | Definition of Done |
|---|---|---|---|---|
| 9 | Implement Slack Connect integration for Growth and Enterprise tier customer support channels | customer-success + platform-engineer | **P2** | Growth tier pilot customer onboarded to Slack Connect; support SLA response within 2h confirmed |
| 10 | Add Premium SLA tier (99.95%) configuration to `tenants.slo_tier` enum and Admin Dashboard; update month-close Worker for Premium credit schedule | enterprise-architect + devops-lead | **P2** | Premium tier selectable at contract; month-close Worker applies correct credit schedule |

#### P1 — Before first Addendum 6 (CAEP SLA) customer execution

| # | Task | Owner | Priority | Definition of Done |
|---|---|---|---|---|
| 11 | Configure CAEP stream registration for pilot tenant; validate AL-CAEP-03 dead-man's switch fires on stream inactivity (4h threshold); confirm `sso.caep_session_revoked` DEC-030 event emits within 60s of test SET injection in staging | platform-engineer + security-engineer | **P1** | Stream in `active` status in Admin Dashboard; AL-CAEP-03 PagerDuty P2 alert confirmed in staging; DEC-030 timestamps satisfy < 60s delta; CC6-E-CAEP-002 evidence template created in `compliance/evidence/sso/` |
| 12 | Confirm pg_cron job 36 (`sso_fleet_health_check`, `*/5 * * * *`) running per `docs/OBSERVABILITY.md §49`; validate AL-SSO-FLEET-01 PagerDuty routing to IC + security-engineer; create SSO-FLEET-E-001 evidence template (`compliance/evidence/sso/SSO-FLEET-E-001_<YYYY-QN>.csv`); confirm `sla_credit_impact: 'none'` hard invariant in DEC-030 `siem.sso_fleet_health_breach` payload | devops-lead + compliance-officer | **P1** | pg_cron job 36 confirmed running in production; AL-SSO-FLEET-01 routing verified; SSO-FLEET-E-001 template ready for first quarterly filing (M11) |

---

## 18. Open Questions

**OQ-SLA-01: How does the per-tenant SLA measurement interact with fleet-wide alerting thresholds in PagerDuty? — 🟢 RESOLVED v1.2**

Resolved by DEC-073 (2026-06-20): SSO-SLO-01 (login success ≥ 99%, 15-minute rolling window) is evaluated **per tenant** (§3.9), and per-tenant breaches feed per-tenant SLA credits under §3.1. PagerDuty SSO alert rules (AL-CERT-*, AL-SSO-GDIR-*) are scoped per `tenant_id` — a single-tenant IdP misconfiguration does not trigger platform-level escalation. Fleet-wide companion signal SSO-SLO-01b (≥ 3 tenants breach SSO-SLO-01 simultaneously within 15 minutes → AL-SSO-FLEET-01 P1) closes the systemic platform-failure detection gap without conflating fleet failures with individual-tenant credits. `sla_credit_impact: 'none'` for SSO-SLO-01b is a hard invariant encoded in the DEC-030 `siem.sso_fleet_health_breach` event payload (DEC-073). Full specification: `docs/OBSERVABILITY.md §49`. See §3.9.

**OQ-SLA-02: Should credits be auto-applied for all tiers or require opt-in claim for Starter tier? — 🟢 RESOLVED v1.1**

Credits auto-apply for all tiers. Starter and Growth tier customers may switch to Manual Credit Mode in Admin Dashboard → Billing → Credit Preferences (growth/enterprise auto-apply is not disableable). Manual-claim mode holds the credit as `sla.credit_pending_claim` for 90 days. This eliminates compliance-officer overhead while giving procurement-conscious Starter customers the option to negotiate credits as contract goodwill. See §3.6.

**OQ-SLA-03: What is the SLA commitment for SCIM provisioning latency? — 🟢 RESOLVED v1.1**

SCIM deprovisioning latency SLA added to §3.7. Commitments: SCIM DELETE → user deactivated P99 < 30 s; sessions revoked P99 < 60 s; breach threshold > 5 minutes triggers auto P1 incident. Better Stack probe S-014 (`scim-deprovision-latency`) defined with alerts AL-SCIM-LAT-01 (P2) and AL-SCIM-LAT-02 (P1). Provisioning (SCIM POST → account created) P99 < 60 s; breach threshold > 10 minutes.

---

## 19. DSAR and Data Portability SLAs

FORM's processor-side obligations for Data Subject Access Requests (DSARs), Right to Erasure (Art. 17), and Data Portability (Art. 20) are time-bound commitments documented here as standalone SLA terms. The Customer remains the data controller responsible for the 30-day window to respond to the data subject.

### 19.1 Subject Access Request — Data Provision (FORM to Customer)

When a Customer forwards an Art. 15 DSAR to FORM via `privacy@form.coach` or the Admin Dashboard DSAR flow:

| Obligation | FORM's Commitment | Customer's Responsibility |
|---|---|---|
| Export package available | Within **24 hours** of FORM receiving the forwarded request | Respond to data subject within 30 days (GDPR Art. 15(3)) |
| Export scope | All FORM-held user data: workout history, CV session keypoints, coaching transcripts, biometric trends, wearable sync metadata, account settings | Combine FORM export with data from other Customer systems |
| Export format | Signed JSON package; SHA-256 hash included for verification | |
| Fulfillment record | `dsar.data_provided` DEC-030 event (HIGH, 7yr retention) — `request_id`, `user_id`, `requester_tenant_id`, `export_sha256` | |

Legal maximum: 30 days from FORM receiving the Customer's forwarded request. The 24-hour commitment is a service target; the 30-day window is the binding legal maximum.

### 19.2 Right to Erasure (GDPR Art. 17)

When a Customer initiates deletion of a user's personal data via the Admin API or Admin Dashboard:

| Step | FORM's Commitment | Timeline |
|---|---|---|
| Soft delete (access revoked, data masked) | Immediate on `DELETE /v1/users/{user_id}` | Within 30 seconds |
| Hard-delete cascade (workouts, CV keypoints, coaching transcripts, media in R2) | Per `docs/DATA_MODEL.md §12` retention schedule | Within 30 days of soft delete |
| Audit log pseudonymisation | `user_id` → keyed HMAC pseudonym per DEC-044 (`DATA_MODEL.md §30.2`); HMAC chain integrity preserved | Within 30 days |
| Deletion certificate | Written certificate issued to `tenant_ops_email` (`dsar.deletion_confirmed` DEC-030 event, HIGH, 7yr); attests scope, cascade timestamp, and DEC-030 chain reference | Within 30 days of soft delete |

**Exception for audit records:** Audit log entries are pseudonymised, not erased, to preserve HMAC chain integrity (SOC 2 CC6.1 / CC7.2 requirement). This exception is disclosed in the DPA (MSA Exhibit C §4.3) and documented in `docs/DATA_MODEL.md §30.2`. The deletion certificate explicitly notes the pseudonymisation exception.

### 19.3 Data Portability — Employee Self-Serve (GDPR Art. 20)

Employees exercise Art. 20 directly, without involving the Customer or FORM support:

| Metric | Commitment |
|---|---|
| Export processing | Typically < 24 hours; legal maximum 30 days |
| Export format | Machine-readable JSON per `docs/DATA_MODEL.md §25` |
| Availability | Self-serve 24×7 at Settings → Privacy → Export; no IT ticket or Customer action required |
| Customer visibility | None — employee's personal data portability request is invisible to the Customer tenant admin |

### 19.4 Offboarding Bulk Data Export (Tenant Termination)

At contract termination, FORM provides a window for the Customer to export tenant-level data:

| Metric | Commitment |
|---|---|
| Export window opens | Within 24 hours of contract termination notice |
| Export window duration | 30 days from termination date |
| Export scope | Tenant configuration, aggregate analytics history, admin audit log for the contract period |
| Individual employee personal data | NOT included — employees export personal data independently (§19.3) |
| Post-window purge | Tenant data purged within 30 days of window close; deletion certificate issued on request |

### 19.5 DEC-030 DSAR Events

All DSAR-lifecycle events are recorded in the HMAC-chained `audit_log` per `docs/AUDIT_LOG_SCHEMA.md`.

| Event | Severity | Retention | When emitted |
|---|---|---|---|
| `dsar.request_received` | HIGH | 7 years | FORM receives a Customer-forwarded DSAR |
| `dsar.data_provided` | HIGH | 7 years | Export package made available to Customer |
| `dsar.deletion_soft` | HIGH | 7 years | Soft delete executed; access revoked |
| `dsar.deletion_confirmed` | HIGH | 7 years | Hard-delete cascade complete; certificate issued |
| `dsar.portability_export_completed` | STANDARD | 7 years | Employee self-serve export fulfilled |
| `dsar.offboarding_export_available` | STANDARD | 7 years | Offboarding bulk export package ready |

All events carry `tenant_id`, `actor_id`, `actor_type`, timestamp, and HMAC chain fields per `docs/AUDIT_LOG_SCHEMA.md`.

---

*v1.0 · 2026-06-06 · enterprise-builder cloud worker*

*v1.1 · 2026-06-13 · enterprise-builder cloud worker — §3.6 update (OQ-SLA-02 resolved: auto-apply for all tiers; Starter Manual Credit Mode opt-in added); §3.7 new (OQ-SLA-03 resolved: SCIM deprovisioning latency SLA — P99 < 60 s session revocation; breach threshold > 5 min triggers P1 incident; probe S-014 with AL-SCIM-LAT-01/02); §16 updated (CC6.4, P8.1 added); §19 new (DSAR and Data Portability SLAs: 24-hour DSAR provision commitment, 30-day erasure certificate, self-serve Art. 20 portability, offboarding export window, five DEC-030 DSAR events). References: `docs/DATA_MODEL.md §30.2` (pseudonymisation on erasure), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 schema), `docs/ENTERPRISE.md` (Privacy Floor), `docs/INCIDENT_RESPONSE.md` (P1 incident classification), `docs/OBSERVABILITY.md §23` (SLA measurement spec). SOC 2 CC6.4 + P8.0 + P8.1 evidence.*

*v1.2 · 2026-06-26 · enterprise-builder cloud worker — §3.8 new (CAEP/SSF Real-Time Session Control SLA: < 60 s commitment for PUSH-capable IdPs Okta/Entra ID/Google Workspace; Addendum 6 opt-in via Order Form; baseline JWT TTL ≤ 15 min for all tenants — no addendum required; AL-CAEP-03 dead-man's switch 4h; DEC-030 events sso.caep_event_received STANDARD 7yr + sso.caep_session_revoked HIGH 7yr + sso.caep_stream_health_breach HIGH 3yr + sso.caep_reregistration_queued STANDARD 7yr; CC6-E-CAEP-002 SOC 2 evidence artefact CC6.3 quarterly; excludes IdP PUSH delivery outages per §5; auto cert-rotation stream re-registration per DEC-072); §3.9 new (SSO Login Availability per-tenant SSO-SLO-01: ≥ 99% success rate 15-min rolling window with per-tenant SLA credit; fleet companion SSO-SLO-01b ≥ 3 tenants breach simultaneously → AL-SSO-FLEET-01 P1 IC + security-engineer — operational only, sla_credit_impact: 'none' hard invariant DEC-073; DEC-030 siem.sso_fleet_health_breach HIGH 3yr; SSO-FLEET-E-001 artefact CC7.2/CC7.3 quarterly M11+); §16 updated (CC6.3 added for CAEP/RISC revocation; CC7.2 updated for SSO-SLO-01b + SSO-FLEET-E-001; CC7.3 updated for AL-CAEP-03 + AL-SSO-FLEET-01 escalation paths); §17 updated (items 11–12 P1: CAEP pilot stream validation + AL-CAEP-03 confirmation, pg_cron job 36 validation + AL-SSO-FLEET-01 routing + SSO-FLEET-E-001 template); §18 OQ-SLA-01 marked 🟢 RESOLVED (per DEC-073 2026-06-20: per-tenant SSO-SLO-01, fleet SSO-SLO-01b, docs/OBSERVABILITY.md §49). Cross-references: docs/MSA_TEMPLATE.md §Addendum 6 (CAEP SLA contract language); docs/SSO_SCIM_IMPLEMENTATION.md §23 (CAEP/SSF architecture); docs/ENTERPRISE_ONBOARDING.md §2.3 (CAEP IdP prerequisite checklist); docs/OBSERVABILITY.md §49 (SSO-SLO-01/SSO-SLO-01b implementation); docs/DECISION_LOG.md DEC-072 (CAEP PUSH mandatory); docs/DECISION_LOG.md DEC-073 (per-tenant SSO-SLO-01 adoption). compliance-officer + enterprise-architect.*

*This document fills the gap identified by `docs/OBSERVABILITY.md OQ-SSO-OBS-01` ("Confirm this is consistent with enterprise contract language before the first SLA review") and provides the customer-facing counterpart to the internal SLA engineering spec in `docs/OBSERVABILITY.md §23`. It supplies SOC 2 A1.1 evidence ("Availability commitments documented") and serves as Exhibit B to the FORM Master Service Agreement. References: `docs/ENTERPRISE.md` (tier pricing, Privacy Floor), `docs/INCIDENT_RESPONSE.md` (P0–P3 classification), `docs/OBSERVABILITY.md §23` (measurement spec), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 event schema), `docs/SUBPROCESSORS.md` (sub-processor register), `docs/SOC2_READINESS.md` (SOC 2 criterion mapping). DEC-030 anchored per DEC-030 decision.*
