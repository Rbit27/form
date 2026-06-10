# FORM · Decision Log v0.1

> Live record of significant decisions з context. Format: Keep it short — what + why + cost-to-reverse.
> Власник: `process-keeper` agent. Founder reviews monthly.
> Newest first.

---

## Table format

| # | Date | Decision | Owner | Why | Reverse cost |
|---|---|---|---|---|---|

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
