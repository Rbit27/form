# FORM · Legal & Compliance Framework v0.1

> Каркас. Не юридична консультація. До public launch — обов'язковий review кваліфікованого юриста.
> Власник: `product-strategist` + майбутній outside counsel.

---

## 1 · Юрисдикції

| Юрисдикція | Чому | Що покриваємо |
|---|---|---|
| **Ukraine** | Команда тут, перший ринок | GDPR-eq (Закон про захист персональних даних), Дiya City податковий статус |
| **EU (GDPR)** | Західна аудиторія, Apple-розповсюдження | Full GDPR compliance, DPO if scale > 250 users |
| **UK** | Високий-LTV ринок | UK GDPR (post-Brexit), Data Protection Act |
| **US (California, NY)** | App Store reaches | CCPA, COPPA (для age-gating), HIPAA-adjacent (не tochaem health data в US definition) |
| **Apple App Store** | Distribution | App Store Review Guidelines, особливо 1.4 (Physical Harm) і 5.1 (Privacy) |

---

## 2 · Privacy Policy (draft outline)

### A. Кого це стосується
- Усіх користувачів FORM
- Beta-учасників (з додатковими положеннями про research)

### B. Які дані ми збираємо

**Категорія 1 · Account (мінімум):**
- Email (Apple Sign In token — pseudo-email якщо relay)
- Display name (опціонально)
- Дата народження (для age-gate)
- Country (для geo-pricing)

**Категорія 2 · Health (special category під GDPR Art. 9):**
- HRV, sleep, heart rate (via HealthKit / Health Connect)
- Weight, height (manual entry, опціонально)
- Workout sessions (sets, reps, weights, tempo)
- Food entries (text only)

**Категорія 3 · Technical:**
- Crash logs (Sentry — анонімізовано)
- Product analytics (PostHog — pseudonymous events, no PII)
- IP-адреса (тимчасово, для rate-limiting)

**Категорія 4 · Communication:**
- Chat messages with Victor (LLM context)

**НЕ збираємо:**
- Raw camera frames (ніколи не залишають девайс)
- Audio/voice recordings (transcribed locally, текст зберігається)
- Location (точну)
- Контакти, фотогалерея, microphone (без явного дозволу-в-контексті)

### C. Як використовуємо
- Надавати продукт (workout, nutrition, coaching)
- Покращувати модель (aggregated, non-PII)
- Comply with legal obligations
- Communicate про продукт (з opt-out)

### D. Кому передаємо
- **Anthropic** (LLM provider) — chat content + program context, **без PII** (ID-обфускація)
- **ElevenLabs** (TTS) — TEXT для генерації voice, **без PII**
- **Supabase** (DB) — encrypted-at-rest на EU region
- **Cloudflare** (edge) — request metadata, не payload
- **Apple / Google** (payment) — billing flow

**НЕ продаємо нікому. Жодних data brokers. Ніколи.**

### E. Retention

| Дані | Retention |
|---|---|
| Active subscription | Зберігаємо все доки активний |
| Cancelled subscription | 30 днів grace, потім видалення (GDPR right to erasure) |
| Crash logs | 90 днів |
| Product analytics | 13 місяців (aggregate далі) |
| Chat history | До видалення акаунту або 24 місяці inactivity |

### F. Права користувача

- **Access** — export всіх своїх даних у JSON через Settings → Export
- **Rectification** — editable в Profile
- **Erasure** — Settings → Delete Account, 3-кліки, без obstrut. 30 днів grace.
- **Portability** — JSON-export універсальний format
- **Objection** — opt-out з product analytics одним toggle
- **Restriction** — pause subscription, дані заморожуються

### G. Children
- Service не для < 18
- Якщо виявимо акаунт дитини — закриваємо, дані видаляємо
- Age verification: self-attestation + reasonable checks

### H. Зміни policy
- 30 днів попередження email + in-app banner
- Якщо matrial change — explicit re-consent

---

## 3 · Terms of Service (draft outline)

### A. Account
- Один акаунт = одна особа
- 18+ years
- Sharing account = termination

### B. Subscription
- Auto-renewing
- Cancel anytime, через Apple/Google або in-app (3 clicks)
- Refunds: per Apple/Google policy (FORM не processes refunds directly)

### C. Acceptable use
- Personal use only
- No reselling, no reverse engineering
- No abuse of AI features (jailbreaking Victor)

### D. Disclaimers (важливо)

> **FORM is not a medical device.** Coaching is informational, not medical advice. If you experience pain, injury, dizziness, or chest discomfort during exercise — stop and consult a licensed healthcare professional. FORM's AI Coach (Victor) is not a substitute for a physician, registered dietitian, physical therapist, or licensed mental health professional.

> **Form analysis through camera is a beta feature.** It detects rep counts, tempo, and gross-deviation patterns on 8 base exercises. It does not replace a qualified coach. It will miss subtle technique issues. Use it as an aid, not as a verdict.

> **No guarantees of results.** Individual outcomes vary. We make no warranties about fitness, weight, or health outcomes.

### E. Limitation of liability

Standard SaaS lim of liab — to extent permitted by jurisdiction.

### F. Termination

We can suspend or terminate for:
- TOS violations
- Repeated abuse of AI features
- Behavior harmful to user or others (e.g., expressed self-harm intent — we'll redirect to support resources first)

### G. Dispute resolution

- First — direct contact (legal@form.coach)
- Then — mediation in Ukraine or user's residence (whichever law applies)
- Class-action waiver (US users specifically)

### H. Governing law
- Ukraine for UA users
- EU GDPR Default for EU
- User's residence for else

---

## 4 · App Store Guidelines compliance map

### Apple App Store Review Guidelines

| Rule | Our compliance |
|---|---|
| **1.1.6 Inaccurate Info** | Cohort-goal badges, no fake stats on website. ASO opis чесний. |
| **1.4 Physical Harm** | "Not medical advice" disclaimers, ED-screening, pain triage |
| **5.1.1 Data Collection** | Privacy policy chiseled, opt-in for analytics, no surveillance |
| **5.1.1(viii) Personal information** | No social hooks без consent |
| **5.1.5 Location** | Не запитуємо location |
| **3.1.2 Subscription** | Cancel у 3 кліки, free trial з ясною умовою |
| **2.5.1 Approved frameworks** | Vision, HealthKit, CoreML — все Apple-supported |

### Google Play Health Apps Policy

- Disclosure про health data — yes
- No medical claims — yes
- No diagnose/treatment claims — yes
- Privacy policy + ToS — required

---

## 5 · Special category data (GDPR Art. 9)

HRV, sleep, weight — **special category** під GDPR. Потребують explicit consent.

**Consent flow:**
1. Onboarding screen: "FORM uses your sleep and HRV to personalize your plan. We don't sell or share this. [Read more] [I agree] [Skip]"
2. Documented: timestamp, version of policy, consent capture
3. Withdrawable at any time (Settings → Privacy → Revoke)
4. If withdrawn — продукт працює без personalization, не блокується повністю

---

## 6 · ED і clinical-safety legal framework

**Position (для legal review):**

FORM is NOT a medical or therapeutic tool. Coaches are not licensed clinicians. We do not screen for or treat ED.

**What we do:**
- ED-screening at onboarding (SCOFF-light, 3 questions)
- If user shows ED markers — present resources (NEDA, regional equivalents), enable soft-mode (no calorie tracking, no body comp metrics)
- We do not store the screening result as "user has ED" — we set a UX-only flag

**What we don't do:**
- Refer users to specific clinicians (liability)
- Track recovery (not our scope)
- Promise we'll detect ED — disclaimer

**Mandatory legal review areas:**
- Risk of mis-classified ED-screening (false negative → harm)
- Jurisdictional differences (mental health regulation in EU vs US)
- Liability waivers (do they hold in EU? — likely no for health-related claims)

---

## 7 · IP

### What we own
- FORM brand, logo, "Victor" persona, voice persona variants
- All copy, design assets, code
- Custom CV heuristics (trade secret, не patent поки)

### What we license
- Anthropic Claude (use via API, our policy)
- ElevenLabs Voice (commercial license потрібен для production)
- Google Fonts (Manrope, Fraunces, JBMono — SIL OFL)
- Open Food Facts / USDA data (CC-BY for OFF, public domain for USDA)

### Trade secrets
- CV heuristics per-exercise (specific angles, thresholds)
- Cohort retention data
- Pricing experiment results

### What we won't trademark prematurely
- "Victor" — generic name, hard to defend
- "FORM" — application in fitness pending, but expensive to defend globally

---

## 8 · Data breach response plan

If breach detected:
1. Within **1 hour** — internal containment (revoke keys, isolate compromised system)
2. Within **24 hours** — assessment: which data, how many users
3. Within **72 hours** (GDPR requirement) — notify Supervisory Authority + affected users
4. Public statement on form.coach/security with timeline
5. Post-mortem published within 30 days

---

## 9 · Compliance checklist (pre-launch)

- [ ] Privacy Policy reviewed by EU lawyer
- [ ] Terms of Service reviewed by EU + UA lawyer
- [ ] Cookie policy (для marketing site)
- [ ] Apple App Privacy disclosures (App Store Connect)
- [ ] Google Play Data Safety form
- [ ] GDPR Data Processing Agreement з усіма processors (Anthropic, ElevenLabs, Supabase, Cloudflare)
- [ ] Standard Contractual Clauses (SCC) для US-EU data transfers
- [ ] DPIA (Data Protection Impact Assessment) для health data
- [ ] Consent management платформа на сайті (для EU)
- [ ] Age-verification flow tested
- [ ] ED-screening flow review by clinical advisor (paid consultation pre-launch)
- [ ] Refund/cancellation flow tested на iOS і Android
- [ ] Account deletion flow tested end-to-end (30-day grace, full data removal)
- [ ] Right to portability (JSON export) implemented
- [ ] DPO appointed (if scale demands — 250+ users з health data в EU)

---

## 10 · Open Legal Questions

1. **Чи FORM-camera-analysis can be classified як medical device у EU MDR?**
   - Hypothesis: No, бо ми не diagnose/treat. Але EU MDR borderline cases — потрібна консультація.

2. **HIPAA-adjacent для US?**
   - Hypothesis: No, бо ми не covered entity (не insurer, не provider). Але PHI definition в деяких штатах ширша.

3. **Ukrainian Дiя City — як впливає на data residency?**
   - Hypothesis: Більшість данних можна тримати в EU regions, але tax/operational rules треба перевірити.

4. **Дiти 13-17 — accidentally реєструються?**
   - Зараз: self-attestation + видалення при виявленні
   - Більш строгий вибір: payment-method check (більшість 13-17 не мають credit card)

5. **B2B SaaS varianta — інший contract framework?**
   - Wellness programs у компаніях — окремі MSAs, окремий DPAs
   - Не до Series A

---

**v0.1 · травень 2026 · MANDATORY review by qualified counsel before public launch**
