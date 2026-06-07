# FORM · CC2 Communication and Information — Evidence Index

> SOC 2 Type II · Trust Service Criteria CC2 (Communication and Information).
> Owner: compliance-officer + security-engineer. Review: annual (Q1) + on any material policy change.
> Reference: `docs/SOC2_READINESS.md §1 (CC2)`, `docs/INCIDENT_RESPONSE.md`, `docs/ACCEPTABLE_USE_POLICY.md`, `docs/PRIVACY_POLICY.md`, `docs/SECURITY_QUESTIONNAIRE.md`.

---

## CC2 Criteria Coverage

| Criterion | Description | Status |
|---|---|---|
| CC2.1 | The entity obtains or generates and uses relevant, quality information to support the functioning of internal control. | 🟡 Partial — Policies documented; formal information quality review cadence not yet initiated. Policy approval log (`compliance/policy-approval-log.csv`) tracks version, owner, and effective date for all active policies. First annual policy review: Q1 2027. |
| CC2.2 | The entity internally communicates information, including objectives and responsibilities for internal control, necessary to support the functioning of internal control. | 🟡 Partial — AUP v1.0 authored; IRP distributed; BCP authored; security training log template live. Internal communication is founder-to-self at pre-hire stage — compensating control accepted per `docs/SOC2_READINESS.md §22`. Post-first-hire: onboarding checklist (`compliance/cc1/`) triggers policy acknowledgment within 5 business days. |
| CC2.3 | The entity communicates with external parties regarding matters affecting the functioning of internal control. | 🟡 Partial — Privacy policy authored (`docs/PRIVACY_POLICY.md`); sub-processor list authored (`docs/SUBPROCESSORS.md`); incident notification templates live (IRP §8 E-01 through E-04). **Critical gap:** both documents must be published to live URLs before first enterprise pilot. Security questionnaire template (`docs/SECURITY_QUESTIONNAIRE.md`) enables enterprise customer due diligence. |

---

## Evidence Files

| File | Description | SOC 2 Criteria | Collection Cadence | Status |
|---|---|---|---|---|
| `docs/ACCEPTABLE_USE_POLICY.md` | Acceptable Use Policy v1.0 (POL-001) — defines authorised and prohibited use of FORM systems for all personnel and contractors. Establishes internal control objectives for access, data handling, and professional conduct. | CC2.1, CC2.2 | Annual review (Q1); update on material system change | 🟡 Authored — pending counsel review + founder signature |
| `docs/INCIDENT_RESPONSE.md` | Incident Response Plan v1.3 (POL-003) — P0–P3 severity classification, escalation tree, communication templates (E-01 through E-04 enterprise; E-EU-01 through E-EU-05 GDPR regulatory), post-mortem template. Establishes internal and external communication responsibilities for security incidents. | CC2.2, CC2.3 | Annual review (Q1); update on material incident or architecture change | 🟢 Authored — IN_FORCE |
| `docs/BUSINESS_CONTINUITY.md` | Business Continuity and Disaster Recovery Plan (POL-012) — RTO/RPO targets, service tier classification, escalation paths, vendor substitution procedures, DR runbook references. Communicates availability commitments and recovery responsibilities. | CC2.1, CC2.2 | Annual review (Q1); update on architecture change | 🟢 Authored — IN_FORCE |
| `docs/PRIVACY_POLICY.md` | Privacy policy — GDPR Art. 13/14 disclosure, data categories, lawful basis, retention periods, data subject rights. Required for CC2.3 external communication. | CC2.3 | Annual review; update on new data category | 🟡 Authored — **gap: not yet published at `form.coach/privacy`** (CC2-GAP-001) |
| `docs/SECURITY.md` | Security architecture and controls documentation — encryption (AES-256-GCM at rest, TLS 1.3 in transit), authentication (JWT 1h + 30d session rotation), RLS policy, dependency security, incident handling. Internal control reference. | CC2.1, CC2.2 | Update on architecture change; reviewed annually | 🟢 Authored — IN_FORCE |
| `docs/SUBPROCESSORS.md` | Sub-processor disclosure list — all processors with data categories, DPA status, EU transfer mechanism. GDPR Art. 13(1)(e) and Art. 28(3) compliance. 30-day advance notice for additions. | CC2.3 | Monthly currency check; publish refresh Q4 | 🟡 Authored — **gap: not yet published at `form.coach/legal/sub-processors`** (CC2-GAP-002, shared with CC9-GAP-004) |
| `docs/SECURITY_QUESTIONNAIRE.md` | Standard security questionnaire template — covers data residency, encryption, access controls, incident notification, SOC 2 posture, AI model training policy, sub-processor list. Enables enterprise customer due diligence and supports CC2.3 external communication. | CC2.3 | Update on major control change; reviewed before each new enterprise pilot | 🟢 Authored |
| `docs/ENTERPRISE_SLA.md` | Enterprise SLA — 99.9% uptime commitment, P0/P1/P2/P3 response SLAs, credit terms, measurement methodology. External communication of system availability commitments. | CC2.3 | Annual review; update on contract structure change | 🟢 Authored — IN_FORCE |
| `docs/ENTERPRISE.md` | Enterprise feature specification — SSO/SCIM capabilities, RBAC, audit log delivery, white-label, privacy floor, data residency options. Primary reference for enterprise customer technical due diligence. | CC2.3 | Update on feature launch | 🟢 Authored |
| `compliance/cc1/security-training-log.md` | Founder annual security training completion record (CC1-E-001). Evidence of internal security awareness communication. Supporting artifact for CC2.2 (communicating security responsibilities). | CC2.2 | Annual (January); triggered by compliance calendar §15 | 🟡 Template live — pending founder self-attestation execution |
| `compliance/policy-approval-log.csv` | Policy approval and version control log — all active policies with version, owner, effective date, and review date. Evidence of structured information quality process (CC2.1). | CC2.1 | Updated on each policy creation or revision | 🟢 Active |
| `security.html` | Public security page at `form.coach/security` — encryption, SOC 2 posture, responsible disclosure channel, pen test cadence, data residency, GDPR compliance. Supports CC2.3 external communication. | CC2.3 | Update on major security posture change | 🟢 Published |
| `docs/ONBOARDING_SECURITY.md` | Security onboarding checklist for new employees and contractors — 1Password setup, MFA enrollment, AUP acknowledgment, access provisioning. Operationalises CC2.2 internal communication at each new hire. | CC2.2 | Update on tooling change | 🟢 Authored — pre-hire; activates on first hire |
| _(pending)_ `compliance/cc2/policy-communication-log.md` | Running log of policy distributions — date, policy version, distribution channel (email / Slack / onboarding), recipient scope (all-personnel / contractor / founder-only). Evidence that CC2.2 internal communication actually occurred. | CC2.2 | Updated on each policy distribution event | 🔴 Not yet created — required post-first-hire (CC2-GAP-003) |
| _(pending)_ `compliance/cc2/breach-notification-exercise.md` | Tabletop exercise record — simulating Art. 33 72-hour GDPR DPA notification and Art. 34 data subject notification. Tests CC2.3 external communication procedures in practice. | CC2.3 | Annual (Q2); tied to DR tabletop (§15.5) | 🔴 First execution Q2 2027 (CC2-GAP-004) |

---

## Internal Communication Channels

CC2.2 requires that control objectives, roles, and responsibilities are communicated internally. At the pre-hire solo-founder stage, internal communication is documented as follows:

| Channel | Scope | Evidence Location |
|---|---|---|
| Policy authoring and version control | All policies in `docs/` and `compliance/` | `compliance/policy-approval-log.csv` — tracks author, reviewer, effective date |
| Git commit history | All changes to control documentation; constitutes internal communication of control intent | GitHub repository — referenced in `compliance/cc8/change-management-policy.md` |
| Security training self-attestation | Annual security awareness communication to self | `compliance/cc1/security-training-log.md` |
| Compliance calendar execution log | Recurring control activities; each execution is evidence of control objective communication | `compliance/calendar/q3-2026-access-review.md` and future calendar artifacts |

**Post-first-hire escalation:** Within 5 business days of any new hire, the following must be executed:
1. Security onboarding checklist (`docs/ONBOARDING_SECURITY.md`) completed and signed
2. AUP v1.0 acknowledgment signed and filed to `compliance/cc1/aup/`
3. New hire added to `compliance/access-review/authorized-roster.md`
4. Policy communication event logged to `compliance/cc2/policy-communication-log.md`

---

## External Communication Obligations

CC2.3 external communication requirements and current status:

| Obligation | Trigger | Owner | Status |
|---|---|---|---|
| Privacy policy published at `form.coach/privacy` | Before first user data collected | compliance-officer | 🔴 CC2-GAP-001 — authored; not yet live |
| Sub-processor list at `form.coach/legal/sub-processors` | Before first enterprise pilot | compliance-officer | 🔴 CC2-GAP-002 — authored; not yet live |
| 30-day advance notice for new sub-processor | On each sub-processor addition | compliance-officer | 🟢 Procedure defined (`docs/SUBPROCESSORS.md`) |
| Art. 33 DPA notification (72h) | On confirmed personal data breach | compliance-officer + founder | 🟢 Procedure defined (IRP §9 + SOC2_READINESS.md §13) |
| Art. 34 data subject notification | On confirmed high-risk breach | compliance-officer + founder | 🟢 Procedure defined (IRP §9 + SOC2_READINESS.md §13) |
| Enterprise breach notification (E-01 template) | P0/P1 incident with enterprise data impact | IC (Incident Commander) | 🟢 Template live (IRP §8 E-01 through E-04) |
| SOC 2 Type II report share (Enterprise tier) | On audit completion; under NDA | compliance-officer | 🟡 Pending audit completion (est. 12 mo post-launch) |
| Security questionnaire response | On enterprise prospect request | compliance-officer | 🟢 Template live (`docs/SECURITY_QUESTIONNAIRE.md`) |
| Pen test summary share | Annual; Enterprise tier customers | security-engineer | 🟡 Pending first annual test (PRE-21) |

---

## Gap Status

| Gap ID | Item | Priority | Owner | Status |
|---|---|---|---|---|
| **CC2-GAP-001** | Privacy policy published at `form.coach/privacy` | P0 — required before first user data collection; without this, GDPR Art. 13 is violated and CC2.3 is unmet | compliance-officer | 🔴 Open — `docs/PRIVACY_POLICY.md` authored; publish gate is PRE-12 in SOC2_READINESS.md |
| **CC2-GAP-002** | Sub-processor list published at `form.coach/legal/sub-processors` | P0 — required before enterprise GA; GDPR Art. 13(1)(e); shared with CC9-GAP-004 | compliance-officer | 🔴 Open — `docs/SUBPROCESSORS.md` authored; publication pending legal sign-off |
| **CC2-GAP-003** | Policy communication log (`compliance/cc2/policy-communication-log.md`) | P1 — activates on first hire; pre-hire compensating control is policy-approval-log.csv | compliance-officer | 🔴 Open — required within 5 business days of first hire |
| **CC2-GAP-004** | Annual breach notification tabletop exercise | P2 — first execution Q2 2027; tests Art. 33/34 notification procedures under time pressure | compliance-officer + security-engineer | 🔴 Planned — `compliance/cc2/breach-notification-exercise.md` pending first execution |
| **CC2-GAP-005** | Security awareness training completion for all employees (not just founder) | P1 — activates on first hire; `compliance/cc1/security-training-log.md` is the per-person template | compliance-officer | 🔴 Open — pre-hire compensating control: founder self-attestation (CC1-GAP-002) |
| **CC2-GAP-006** | AUP formal acknowledgment from all personnel | P1 — activates on first hire; pre-hire: founder is sole personnel, compensating control accepted | compliance-officer | 🟡 Partial — AUP authored (CC1-GAP-001); acknowledgment pending counsel review + signature |

---

## SOC 2 Auditor Notes

**Evidence this section closes or advances:**
- `docs/SOC2_READINESS.md §1 (CC2)`: All four CC2 control rows addressed here
- PRE-12 (privacy policy live URL): CC2-GAP-001 closes this when executed
- PRE-16 (consent management before health data): Supported by Privacy Policy (CC2.3) — separate from this section

**Compensating controls accepted at solo-founder stage (per SOC2_READINESS.md §22):**
1. *Single-person internal communication:* CC2.2 requires communicating control objectives to personnel. At one person, all communication is self-directed. Evidence: policy authoring commits with dates, policy-approval-log.csv, security-training-log.md. **Acceptance expires at first hire.**
2. *Informal distribution channel:* No LMS, no email distribution system. Evidence channel is git commit history and policy-approval-log.csv. **Acceptance expires at first hire** (post-hire: onboarding checklist + cc2/policy-communication-log.md take over).

---

*Owner: compliance-officer + security-engineer · Created: 2026-06-07 · Next review: Q1 2027*
