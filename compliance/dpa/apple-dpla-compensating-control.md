# Apple APNs — GDPR Art. 28 Compensating Control

> **Sub-processor:** Apple, Inc. (SP-05 — Apple Push Notification service)
> **Issue:** Apple does not offer a standalone GDPR Art. 28 DPA for APNs; the Developer Program License Agreement (DPLA) contains partial Art. 28 commitments but does not satisfy all eight Art. 28(3)(a)–(h) obligations.
> **Classification:** Compensating control — accepted for pre-Series A; legal review required pre-Series A
> **Owner:** `compliance-officer` + `platform-engineer`
> **Evidence artifact:** PRE-34-E-004 partial (compensating control documentation for SP-05)
> **SOC 2:** C1.1 (compensating control), OQ-VRM-01

---

## Gap Description

GDPR Art. 28(1) prohibits engaging a sub-processor without a binding DPA covering Art. 28(3)(a)–(h). Apple does not provide a standalone Art. 28 DPA for use of its APNs infrastructure by app developers. The contractual basis is the Apple Developer Program License Agreement (DPLA), which:

- §3.3.2 includes data handling obligations covering notification delivery
- Implicitly restricts Apple's use of notification content to delivery purposes only
- Does **not** explicitly address all eight Art. 28(3)(a)–(h) obligations (particularly: sub-processing disclosure, audit rights, deletion confirmation, instruction compliance)

Apple's posture is that APNs is a conduit service (similar to a postal carrier) and its own Privacy Policy covers the processing adequately under Art. 6(1)(f) (legitimate interests for service delivery).

---

## FORM's Compensating Controls

### 1. Data minimisation (primary control)

Push notification payloads sent through APNs are restricted to non-health text content only. The `notifications-worker` Cloudflare Worker validates payload content before passing to APNs:

- **Prohibited in APNs payload:** workout metrics, body measurements, heart rate/HRV values, food log data, CV session data, GDPR Art. 9 health categories, or any value that could identify individual health status
- **Permitted:** coaching reminder text, streak count, app badge numbers, generic motivational cues
- **Enforcement:** Zod schema on `notifications-worker` input; any health-value fields trigger payload rejection with `notifications.health_data_in_push_blocked` DEC-030 alert (CRITICAL severity)

Because health data under GDPR Art. 9 never enters APNs, Apple's role as a sub-processor is limited to processing non-special-category personal data (device token, notification text). The Art. 28 compliance risk is materially reduced.

### 2. DPLA acceptance record

Apple Developer Program License Agreement version accepted by FORM:
- **Account:** founder Apple Developer account (apple-id@form.coach)
- **DPLA acceptance date:** [DATE — to be recorded on DPLA re-acceptance during annual program renewal]
- **DPLA version:** Available at developer.apple.com/support/terms/

Screenshot of active developer program membership to be filed alongside this document as `apple-developer-program-active-YYYY-MM-DD.png`.

### 3. Transfer mechanism

Under GDPR Art. 45, the EU-US adequacy framework (EU-US Data Privacy Framework, adequacy decision of 10 July 2023) provides a transfer mechanism for personal data to Apple Inc. (US). Apple is self-certified under the EU-US DPF as of 2023. For UK data subjects, the UK-US adequacy arrangement applies.

| Transfer basis | Applies to | Status |
|---|---|---|
| EU-US DPF adequacy (Apple) | EU user device tokens | 🟢 Apple DPF-certified |
| UK adequacy (US DPF) | UK user device tokens | 🟡 Verify Apple UK extension |

### 4. Legal counsel assessment (pending)

**Open question OQ-VRM-01:** Does the DPLA, combined with data minimisation and the EU-US DPF adequacy decision, constitute sufficient Art. 28 compliance for APNs? FORM's outside counsel to assess pre-Series A.

Options if the assessment finds the DPLA insufficient:
1. **Alternative push provider** with GDPR DPA — Firebase Cloud Messaging (Google) has a GDPR DPA; migration cost is moderate (FCM SDK integration)
2. **Direct APNS DPA negotiation** — available only to large-volume developers; not feasible pre-Series A
3. **No-push architecture for EU health notifications** — route health-sensitive notifications through in-app inbox only, use APNs only for badge/sound alerts with no payload content

Outside counsel engagement target: pre-Series A (Month O-8 per §15 compliance calendar).

---

## SOC 2 Auditor Narrative

For SOC 2 C1.1 purposes: FORM does not have a formal Art. 28 DPA with Apple for APNs. The compensating control is data minimisation — no GDPR Art. 9 health data is transmitted via APNs. Apple's APNs role is therefore limited to processing non-special-category personal data (device token + notification text), which reduces the Art. 28 obligation. FORM will obtain outside counsel confirmation of this compensating control's adequacy pre-Series A (OQ-VRM-01). Until then, SP-05 is classified as 🟡 compensating control rather than 🔴 gap, because the health data is demonstrably excluded from APNs payloads by architectural constraint, not just policy.

---

*v1.0 · 2026-06-12 · compliance-officer + platform-engineer*
*This document is PRE-34-E-004 partial for SP-05 Apple*
*Upgrade to 🟢: outside counsel confirmation + DPLA download filed alongside this doc*
