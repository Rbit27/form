# FORM · Privacy Policy (DRAFT)

**Effective Date:** TBD (pending legal review)
**Last Updated:** 14 May 2026
**Version:** 0.1 (draft — not for public)

> ⚠️ **This is a draft.** Before public release, qualified counsel in
> Ukraine + EU must review. Do NOT publish as-is.

---

## 1. Who we are

FORM ("we", "us", "our") is operated by [Legal Entity TBD],
registered at [Address TBD], Ukraine, contact: legal@form.coach.

If you are in the EU, our data controller obligations apply under
the General Data Protection Regulation (GDPR). If you are in the UK,
under UK GDPR. If you are in California, under CCPA.

---

## 2. What information we collect

### 2.1 Account information
- Email address (via Apple Sign In — may be a relay)
- Display name (you choose; optional)
- Age (for compliance — must be 18+)
- Country (for geo-pricing)
- Apple ID (opaque identifier)

### 2.2 Health and fitness data (SPECIAL CATEGORY under GDPR Art. 9)
- Heart Rate Variability (HRV) — read from HealthKit/Health Connect
- Sleep stages and duration
- Resting heart rate
- Body temperature (if available)
- Weight, height (manual entry, optional)
- Workout sessions: sets, reps, weights, tempo
- Food entries (text only — you log what you eat)

**Collection of this data requires your explicit consent**, captured
during onboarding. You can withdraw consent at any time in Settings.

### 2.3 Technical information
- Crash reports (anonymized, via Sentry)
- Product usage analytics (pseudonymous, via PostHog) — opt-out available
- Device model + OS version
- IP address (used only for rate-limiting and fraud prevention,
  truncated within 1 hour)

### 2.4 Communication
- Chat messages with our AI Coach ("Victor")
- Support emails (if you contact us)

### 2.5 What we DO NOT collect
- Raw camera frames (never leave your device)
- Raw audio recordings (transcribed locally; only text stored)
- Precise location (we collect country only)
- Contacts, photos, microphone (without explicit in-context permission)
- Browsing history outside our app
- Social media activity

---

## 3. How we use your information

We use your information to:

| Purpose | Legal basis (GDPR) |
|---|---|
| Provide the product (workouts, nutrition, coaching) | Contract |
| Personalize your training plan | Explicit consent (Art. 9) |
| Improve our models (aggregated, pseudonymous) | Legitimate interest |
| Detect and prevent fraud / abuse | Legitimate interest |
| Comply with legal obligations | Legal obligation |
| Communicate about the product (with opt-out) | Legitimate interest |

We do NOT use your data for advertising. We do not sell or share
your data with data brokers.

---

## 4. Who we share information with

### 4.1 Service providers (processors)

| Provider | Purpose | Data shared | Location |
|---|---|---|---|
| Anthropic | LLM chat & coaching responses | Conversation context (no PII — user ID pseudonymized) | US (with SCCs in place) |
| ElevenLabs | Voice synthesis for coaching cues | TEXT only — no PII | US (with SCCs) |
| Supabase | Database hosting | All user data (encrypted at rest) | EU (eu-central-1) |
| Cloudflare | Edge networking and CDN | Request metadata | Global edge (EU primary) |
| Apple | App Store payments | Billing only (we never see card details) | US/Global |
| Sentry | Crash reporting | Anonymized stack traces | US (with SCCs) |
| PostHog | Product analytics (opt-out available) | Pseudonymous event data | EU (eu.posthog.com) |

All processors are bound by Data Processing Agreements (DPAs) under GDPR.

### 4.2 Legal disclosure
We may disclose your data if required by law (court order,
subpoena, lawful government request). We publish an annual
transparency report (planned post-launch).

### 4.3 Business transfer
If FORM is acquired, your data may transfer to the acquirer.
We will notify you 30 days in advance.

---

## 5. International transfers

Some processors are outside the EU. For transfers outside the EU/UK:
- We rely on Standard Contractual Clauses (SCCs)
- For US transfers, we monitor adequacy decisions and Privacy Shield
  developments
- You can request a copy of our SCCs at legal@form.coach

---

## 6. How long we keep your data

| Data type | Retention |
|---|---|
| Active subscription | While your account is active |
| Cancelled subscription | 30 days grace, then full erasure |
| Crash reports | 90 days |
| Product analytics | 13 months (then aggregated) |
| Chat history | Until account deletion or 24 months inactivity |
| Workout / food data | Until account deletion |
| Health snapshots | Until account deletion |

---

## 7. Your rights (GDPR / UK GDPR)

You have the right to:

- **Access** — request a copy of all your data
- **Rectification** — correct inaccurate data
- **Erasure** — delete your account and all data (30-day grace period)
- **Portability** — export your data in JSON format
- **Restriction** — pause processing (we will freeze your data)
- **Objection** — opt out of product analytics
- **Withdraw consent** — at any time, in Settings
- **Lodge a complaint** — with your local Supervisory Authority

To exercise these rights, use the Settings menu in the app or
contact privacy@form.coach. We respond within 30 days.

---

## 8. Special category data: health information

Health data receives the highest protection under GDPR Art. 9.

- We process only with your **explicit consent**
- You can withdraw consent at any time — this will limit
  personalization but the app will still function in a
  reduced-functionality mode
- We never use this data for advertising or share it for
  commercial purposes

---

## 9. Children

FORM is not for children under 18. If we discover an account
belonging to someone under 18, we close it and delete the data.

If you believe a minor has registered, contact legal@form.coach.

---

## 10. Cookies and tracking (for our marketing site)

Our marketing site (form.coach) uses minimal cookies:
- Strictly necessary (session, preferences) — no consent required
- Analytics (PostHog) — opt-in for EU visitors

The mobile app does not use cookies in the traditional sense
(it uses native iOS/Android storage).

---

## 11. Security

We use industry-standard security:
- TLS 1.3 for all data in transit
- AES-256 encryption at rest (Supabase default)
- Apple Keychain / Android Keystore for local secrets
- Regular dependency audits
- Bug bounty program (planned, post-launch)

**No system is 100% secure.** We will notify you and the relevant
supervisory authority within 72 hours of any breach affecting your data.

---

## 12. Changes to this policy

If we make material changes, we will:
- Notify you 30 days in advance via in-app banner + email
- For changes affecting consent (e.g., new processor), we will
  re-request consent

---

## 13. Contact

- **Privacy questions:** privacy@form.coach
- **DPO (when appointed):** dpo@form.coach
- **General:** hello@form.coach
- **Postal:** [TBD — Ukrainian registered address]

For EU residents who don't reach a satisfactory resolution, you can
lodge a complaint with:
- Ukraine: Парламент Уповноваженого з прав людини
- Your local EU/UK supervisory authority

---

**v0.1 DRAFT · NOT PUBLIC · pending legal review**
