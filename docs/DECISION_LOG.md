# FORM · Decision Log v0.1

> Live record of significant decisions з context. Format: Keep it short — what + why + cost-to-reverse.
> Власник: `process-keeper` agent. Founder reviews monthly.
> Newest first.

---

## Table format

| # | Date | Decision | Owner | Why | Reverse cost |
|---|---|---|---|---|---|

---

## 2026-05-16

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
