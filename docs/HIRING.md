# FORM · Hiring Playbook v0.1

> Перші три наймачі вирішують усе. Не наймай швидко. Не наймай під щось окрім роботи.
> Власник: founder. Reviewed by: `product-strategist`.

---

## 1 · Принципи

1. **Hire слабкіших за себе у спеціалізації — фейл.** Кожен наймач має бути найкращим у своїй темі в кімнаті.
2. **Hire тих, кого хочеш на свій 50-річний день народження.** Якщо не хочеться — не наймай, навіть якщо хороший.
3. **Compensation: above-market base + meaningful equity.** Дешеві наймачі — короткострокова економія, довгострокова втрата.
4. **Remote-first, opt-in offsite.** Квартальні зустрічі у Києві, Львові або Berlin/Lisbon.
5. **No prima donnas.** Можеш бути найкращим інженером світу, але якщо токсичний — не у команді.
6. **Bias toward people who use the product.** Якщо людина не може уявити, як буде нею користуватися — це red flag.

---

## 2 · Перші три ролі (до Seed close)

### Role 1 · Founding Engineer (iOS / Mobile)

**Mission:** v1 iOS-застосунок shippable за 6 місяців.

**Чому ця роль перша:** без mobile-products ми не існуємо. Це highest-impact hire у компанії на цій стадії.

**Профіль:**
- 5+ років iOS-розробки (Swift, SwiftUI, ARKit/Vision/AVFoundation)
- Працював на consumer-app з 100k+ MAU
- Має один проєкт у Github / App Store, де ви ВИДНО видно його стиль
- Розуміє ML on-device (CoreML, Apple Vision API). Не обов'язково ML engineer.
- Bonus: знає Android (для phase 2 transition)

**Анти-профіль:**
- Тільки enterprise-iOS (B2B, sluggish UX patterns)
- Не може показати свою роботу публічно
- Хоче «AI fitness» бо це trend
- 100% перформер у попередній команді з мікроменеджментом

**Compensation (target):**
- Base: $90–130k USD (Київ — top of market)
- Equity: 1.5–3.0% (founding range)
- Cash-rich варіант: $130k + 1.0%
- Equity-rich варіант: $90k + 3.0%

**Hiring funnel:**
1. Outreach до 50 кандидатів (LinkedIn, GitHub, Twitter)
2. Async portfolio review (без CV ranking — судимо за роботою)
3. 30-min intro call (motivation, fit)
4. Take-home мала задача (8 годин max) — pose-detection prototype
5. Deep technical session (90 хв)
6. Founder + advisor лекція (60 хв)
7. Reference checks (3 mandatory)
8. Offer

**Source channels:**
- iOS Slack/Discord комюніті (UA + global)
- Apple Worldwide Developers conference 2026 attendees
- People who built indie fitness apps з 10k+ downloads
- Ex-Whoop / Strava / Future / Fitbod engineers

---

### Role 2 · Design Lead

**Mission:** від рівня v0.2 (HTML-прототипи) до production-design system у Figma + Xcode.

**Чому друга:** дизайн — це наш бренд-моат. Без сильного дизайн-лідера ми регресуємо до AI-slop.

**Профіль:**
- 5+ років product design на mobile
- Знає Figma до рівня variables/tokens/auto-layout без compromises
- Працював у консьюмер-продукті з суворим бренд-кодом (Apple, Stripe, Linear, Notion-tier)
- Має typography-чутливість (різниця між Inter і Manrope для нього очевидна)
- Сильна копірайт-сторона (може писати UX-copy без brand-voice supervision)

**Анти-профіль:**
- Тільки design-system maintainer (не дизайнерить features)
- Шукає remote-only без collaboration
- Не використовує продукти, які проектує (test users — конкретний red flag)
- Сильні Dribbble-навички, але без shipped products

**Compensation:**
- Base: $80–115k USD
- Equity: 1.0–2.0%

**Hiring funnel:**
- Як Founding Engineer, але take-home — redesign однієї секції з нашого `index.html`

**Source channels:**
- Mobbin, Dribbble (з фільтром на shipped iOS work)
- Designers, які працювали у nuxt/linear-tier startups
- Українські дизайнери у Berlin/Amsterdam/Lisbon (повертаються до UA на partial basis)

---

### Role 3 · Sport Science Advisor (Part-time)

**Mission:** валідація програм, CV-метрик, нутриційних рекомендацій. Один день на тиждень — 8 годин/місяць консультацій.

**Чому третя:** ми не маємо in-house sport-science. Без зовнішньої експертної валідації наш `sports-scientist` агент — це LLM з книжок без feedback loop.

**Профіль:**
- PhD у sport science / exercise physiology / kinesiology
- Реальний коучинг-досвід (мінімум 5 років з athletes)
- Має публікації або strong track record at federations
- Розуміє AI-products і його обмеження
- Engаging communicator (можемо колись використовувати як content collaborator)

**Анти-профіль:**
- Тільки academic, без хвилі реального coaching
- Анти-AI принципово (не може advise)
- Працює на конкурента (Whoop science team тощо) full-time

**Compensation:**
- Hourly: $200–350/година
- Або retainer: $2,500–4,000/міс за 8-10 годин
- Optional advisory equity: 0.1–0.25% vested over 2 years

**Source channels:**
- Renaissance Periodization affiliated PhDs
- Andy Galpin alumni network
- Українські sport-science кафедри (LNUFK, Дніпровський спорт)
- Strava / Whoop / Strenuous Life advisor networks

---

## 3 · Майбутні ролі (post-Seed, M4–M8)

Не наймаємо доти, доки не доведено потреба:

| Role | When | Why |
|---|---|---|
| Backend Engineer | M4 | Якщо Supabase + Cloudflare Workers не масштабуються |
| Android Engineer | M5 | iOS launched, активний D7 ≥ 30% |
| Head of Growth | M6 | Якщо CAC проблема або шанс масштабувати paid |
| Customer Support Lead | M5 | Якщо support volume > 20 tickets/день |
| ML Engineer (CV) | M7 | Якщо CV-quality потребує real research, не heuristics |
| Content Producer | M6 | Якщо contentstrategist agent + outsource не масштабується |

**Кожний hire потребує:**
- Чіткий мандат (одна рядкова mission)
- Метрику, яку він буде рухати
- Кваліфікаційні питання для interview
- Альтернативу (чи можемо без?)

---

## 4 · Interview rubric (для всіх ролей)

### Block 1 · Motivation (15 хв)
- Why FORM?
- What did you understand about us before this call?
- What's interesting about this for you personally?

**Red flags:**
- "I'm looking for senior role" (про себе)
- "I love AI" (генерики)
- "Anywhere remote is fine" (не зацікавлений у роботі — у lifestyle)

**Green flags:**
- "I tried your prototype and... [specific observation]"
- "I've been thinking about [related problem]"
- "I want to work on something I'd use myself"

### Block 2 · Craft demonstration (45 хв)
- Walk us through a project of yours
- Why did you make this choice over that one?
- What would you do differently now?

**Red flags:**
- Speaks only at "what" level, не "why"
- Cannot point to a single decision they regret
- Bigger team — bigger story (over-credits self)

**Green flags:**
- Specific. Concrete. "I changed X because Y data showed Z."
- Self-critical
- Mentions team contributions naturally

### Block 3 · Take-home review (60 хв)
- Walk through your solution
- Where did you cut corners and why?
- What would v2 of this look like?

### Block 4 · Cultural fit (30 хв)
- Tell me about a time you disagreed with a teammate
- Tell me about a project that failed — what did you learn?
- What does your ideal workweek look like?

### Block 5 · Their questions (15 хв)
- (We expect strong, specific questions back. If "no questions" — that's a red flag.)

---

## 5 · References (3 mandatory)

1. **Direct manager** at recent role
2. **Direct report** (if has managed)
3. **Peer** at same level

**Questions:**
- How would you describe their work?
- What did you find frustrating about working with them?
- Would you hire them again? Why or why not?
- What's one thing they'd be amazing at, and one thing they'd struggle with?

If reference відмовляється від "What would you find frustrating?" — це сигнал.

---

## 6 · Onboarding (first 30 / 60 / 90 days)

### Day 0 (pre-start)
- Send laptop (M3 MacBook Pro для engineers і designers)
- GitHub / Figma / Slack / Linear access
- Buddy assigned

### Week 1
- Read all docs (README, BRAND, FLOWS, PERSONAS, TECHNICAL, PITCH)
- Meet founder for 90-хв deep dive
- Meet sport-science advisor (if applicable)
- Use the prototype daily

### 30 days
- First shipped contribution
- One user interview attended (any role)
- Personal POV on one product decision documented

### 60 days
- Owns a piece (feature, system, surface)
- Has built reference relationships with all roles
- Submitted one process improvement

### 90 days
- Independent contributor
- Has shipped 2+ user-visible things
- Has helped onboard the next hire OR proposed one

If 90-day mark is not met:
- Direct conversation
- Possible role-adjustment
- Worst case: termination з generous severance, без drama

---

## 7 · Compensation philosophy

### Salary
- **Above-market base** за роль і регіон
- Доступний bands документ (transparent)
- Annual review October, raise effective January
- Equity refresh every 2 years for high-performers

### Equity
- Founding (first 5 hires): 0.5–3.0%
- Early (M4–M12): 0.25–1.0%
- Post-Seed: 0.1–0.5%
- 4-year vest, 1-year cliff (стандарт)
- Optional: shorter cliff (6 міс) для proven hires

### Benefits
- Unlimited PTO (з minimum 20 днів — "use it or lose it" structure для здоров'я)
- $1,500/рік на edu (книги, курси, conferences)
- $1,000/рік на здоров'я (gym, therapy, sport)
- Mac + chair + screen + accessories на ваш вибір
- $2,000 home-office setup budget

---

## 8 · No-go (хто нам НЕ підходить)

- Хто хоче «AI startup» бо це hype
- Хто шукає cushy remote-job
- Хто не використовує консумер-продукти серйозно
- Хто не може показати конкретний приклад своєї роботи
- Хто токсично говорить про попередні команди (про себе на 1-on-1)
- Хто очікує hand-holding на роботі senior-level
- Хто думає, що fitness — це «м'яка» галузь, де можна без серйозності

---

## 9 · Diversity та inclusion

Активно шукаємо різноманітну команду:
- Жінок у technical ролях (історично недопредставлені)
- Не-UA EU-talent (для геолокаційного балансу)
- People з non-traditional backgrounds (self-taught engineers, kicker-careers)

Це НЕ означає quotas. Це означає **proactive sourcing** з пулів, де sponsors або не дивляться, або не вкладають зусилля.

**Red flag:** якщо весь pool кандидатів виглядає однаково — sourcing process поламано.

---

## 10 · Open hiring questions

1. **Чи варто наймати team-lead раніше за Series A?**
   - Hypothesis: ні. До 15 людей founder + dotted-lines працює. Lead скоріше bottleneck.

2. **Чи UA-only команда обмежує us?**
   - Currency advantage real, language ok (всі senior tech говорять English)
   - Risk: під час війни — relocations можливі, geographic disruption

3. **Чи приймати кандидатів які лише дистанційно, без offsite?**
   - Hypothesis: ні до 12 людей. Personal connection важливіше за convenience.

---

**v0.1 · травень 2026 · правки приймає founder**
