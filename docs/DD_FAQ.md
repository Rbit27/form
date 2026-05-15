# FORM · Investor Due Diligence FAQ

> Підготовка до DD-питань після успішного intro call.
> Companion до: SEED_NARRATIVE.md (call script), PITCH.md (deck), FINANCIALS.md (model), DATA_ROOM.md (materials).
> Власник: `product-strategist` + founder. Оновлювати після кожного реального investor conversation.

---

## Як користуватися цим документом

Це не скрипт — це підготовка. Кожна відповідь — чесна, коротка, без bullshit.

Правила тону:
- Конкретно, не розмито
- Невизначеність — визнавати, не ховати
- Якщо не знаєш — сказати що і коли дізнаєшся
- Жодних вигаданих цифр

---

## Блок 1 · Команда

**Q: Чому ти? Чому цей продукт, чому зараз?**

Я тренуюся з 16 років. Проблема «немає тренера поряд» — моя власна. Я розумію і користувача, і продукт зсередини. Технічно — будую з Claude AI agent stack, розумію що реалістично в CV і що — research-bet. Зараз — бо Vision API достиг до точки де real-time pose estimation на пристрої стає можливим без cloud latency. Вікно відкрилося.

---

**Q: У тебе немає co-founder. Це ризик?**

Так, і я це визнаю. Mitigation: наймаю Founding Engineer з сильним iOS-background як першого hire, і двох технічних advisors. Це не замінює co-founder, але закриває найбільший execution risk. Якщо правильна людина з'явиться — відкритий до розмови.

---

**Q: Де технічна команда?**

Зараз — я плюс AI agent stack для docs/strategy. Engineering починається з наймом Founding Engineer (iOS). JD готова, розміщується на Djinni і Wellfound. Очікую найняти до [дата]. CV pipeline — після FE на борту. Не маю технічного co-founder, але є конкретний план і реальний JD.

---

**Q: Ти з України. Операційний ризик?**

Реальний ризик. Mitigation: команда розподілена за дизайном, розробка — remote-first. Юридична структура — Delaware entity + UA entity для операцій. Ключові дані і infrastructure — поза UA. Є founders з кращою ситуацією — ми про це чесні. Але ринок UA + Diaspora — $300M+ TAM, і я знаю його краще за будь-кого.

---

## Блок 2 · Ринок і розмір

**Q: Яка реальна TAM?**

Fitness app market globally — $15B+ і росте. Але ми не цілимо в "fitness app market" — ми цілимо в людей, що тренуються самостійно без персонального тренера і вже платять за fitness apps. Консервативний адресний ринок для v1.0 (UA + Eastern EU + Diaspora): ~5M потенційних users. При 2% penetration і $9-19/мо = $10-20M ARR addressable. TAM для Western EU expansion — інший порядок, але це пізніший розмова.

---

**Q: Хто ще в цьому просторі?**

Основні: Whoop, Future, Vi, Freeletics, Caliber. Жоден не робить real-time form correction — вони або coaching-on-demand (дорого), або програмна генерація (дешево але тупо). FORM закриває між ними. Детальна розбивка — COMPETITIVE.md.

---

**Q: Чому не зробить Apple або Google?**

Можуть. Але: (1) вони platform-level, не coach-level, (2) глибока спортивна наука і Victor persona — не їхній продукт, (3) вони побудують generic, ми будуємо opinionated. Якщо Apple зробить щось схоже — це validation, і в нас буде 12-18 місяців lead + відносини з early adopters.

---

## Блок 3 · Продукт і технологія

**Q: CV pipeline — це реально? Accuracy?**

Apple Vision Framework v2 + CoreML — так, реально на сучасних iPhone. Ми обрали 8 базових ліфтів для v1.0 (DEC-017) бо точність critical — краще добре на 8, ніж погано на 30. Accuracy на рівні з Personal Trainer на 3 базових рухах (squat, deadlift, press) — ціль для TestFlight beta. Не обіцяємо 99% — обіцяємо чесний benchmark після beta.

---

**Q: Privacy? Дані тіла — sensitive.**

По дизайну: CV processing — on-device (не відправляємо відео на server). Health data (HRV, sleep) — локально на пристрої, синхронізується через encrypted storage. Юридична framework — GDPR compliant (LEGAL.md). ED screening перед nutrition features — non-skippable (DEC-018). Детальна архітектура — TECHNICAL.md і SECURITY.md.

---

**Q: LLM costs — скільки коштує один Victor-response?**

Поточна модель: ~$0.002-0.008 per interaction з prompt caching. При 15 interactions/day на user і $19/мо — LLM cost ~$0.30-1.20/user/мо, або 1.5-6% від revenue. Прийнятно. Цифри — estimates до реального usage data, оновлю після beta.

---

**Q: Чому iOS-only для v1.0?**

Три причини: (1) Vision Framework — кращий на iOS, Android equivalent requires more workarounds, (2) target user (early adopter, higher LTV) — iOS-first, (3) Founding Engineer hire — iOS specialist. Android — M7+ після UA launch validation.

---

## Блок 4 · Бізнес-модель і фінанси

**Q: Чому subscription-only, без free tier?**

DEC-016. Free tier у fitness = graveyard of inactive users. Ми будуємо для людей що committed до тренувань. Paywall дає signal наміру. Trial period (7 days) дає можливість попробувати без risk. Детальна обгрунтовка — DECISION_LOG.md DEC-016.

---

**Q: Pricing — звідки $9 UA / $19 Western EU?**

Geo-pricing (DEC-023). UA: $9/мо = менше ніж одне заняття в залі, але в нас daily value. Western EU: $19/мо = менше ніж Coffee + Spotify + щось, але порівнянно з персональним тренером ($200+/тиждень). Pricing тестуватиметься в beta. Якщо wrong — pivot. Детально — FINANCIALS.md.

---

**Q: Break-even при яких числах?**

Модель показує: при ~800 paying users (monthly, $15 blended ARPU) операційний break-even для ранньої команди (founder + 1 FE + infrastructure costs). Це консервативно. Детальна таблиця — FINANCIALS.md. Я не буду говорити ці цифри вголос поки вони не підтверджені реальним usage.

---

**Q: Куди підуть гроші pre-seed?**

1. Founding Engineer hire (найбільша стаття)
2. Design Lead hire
3. Infrastructure (Supabase, Cloudflare, Apple dev accounts)
4. Legal (counsel review privacy policy, ToS, GDPR DPA)
5. User research (30 interviews — деякі з incentives)
6. Operating runway поки не буде перших 200 beta users

Детально — FINANCIALS.md розбивка по категоріях.

---

## Блок 5 · Execution і ризики

**Q: Що може піти не так?**

Чесно — три biggest risks:

1. **CV accuracy не досягне планки** — якщо on-device CV не дасть достатньої точності, або latency занадто висока. Mitigation: ранній proof-of-concept з FE до повного dev. Kill criteria чітко в OKRS_2026.md.

2. **Retention після перших 30 днів** — якщо users churning early, моделі не працюють. Mitigation: AARRR з чіткими thresholds, cohort tracking від дня 1. Детально — METRICS.md.

3. **Founding Engineer не найметься вчасно** — якщо hire займе 3+ місяці, timeline зсувається. Mitigation: JD на Djinni + Wellfound зараз, referral network активований.

---

**Q: Якщо через 6 місяців не буде traction — що робиш?**

Залежить від причини. Kill criteria в OKRS_2026.md конкретні: якщо D30 retention < X%, якщо CV accuracy < Y на benchmark lifts, якщо < Z paying users після 3 місяці beta — pivot або stop. Я не буду тягнути мертвий продукт роками. Ресурси і час дорогі.

---

**Q: Чому ти будуєш в Україні під час війни?**

Бо тут мій ринок, моя команда, і мій контекст. Великий технічний talent pool за значно нижчими costs. Founders що будують в складних умовах — часто більш resilient. І це не перша компанія що успішно будується з UA як home base.

---

## Блок 6 · Раунд і умови

**Q: Скільки піднімаєш і на яку оцінку?**

Pre-seed: $250-500K (конкретна сума залежить від investor conversations). Оцінка — не називаю першим, чекаю на anchor від investor. Готовий до SAFE або convertible note з стандартними умовами (MFN, pro-rata for $250K+).

---

**Q: Чи є інші investor conversations?**

[Відповідь оновлюється в реальному часі]

Якщо так: "Є кілька розмов. Не можу говорити деталі, але готовий до синхронізації по timeline."

Якщо ні: "Ти перший institutional conversation. Є кілька angel розмов."

Правило: ніколи не брехати про FOMO — якщо його немає.

---

**Q: Що ти хочеш від investor крім грошей?**

Конкретно: (1) intro до fitness/wellness founders що підняли pre-seed, (2) intro до iOS engineers з досвідом, (3) connections до sport science advisors. Якщо у вас немає цього network — гроші все одно корисні, але питаю чесно.

---

## Блок 7 · Data Room

Якщо investor хоче більше:

| Матеріал | Де |
|---|---|
| Deck (12 slides) | [send directly] |
| One-pager | investors.html |
| Financial model | docs/FINANCIALS.md |
| Technical architecture | docs/TECHNICAL.md |
| Privacy/legal framework | docs/LEGAL.md + legal/ |
| Competitive analysis | docs/COMPETITIVE.md |
| Team + JDs | hiring/ + docs/HIRING.md |
| OKRs + kill criteria | docs/OKRS_2026.md |

Повна структура data room по tier — docs/DATA_ROOM.md.

---

## Оновлення після розмов

Після кожного investor conversation — записуй:
- Які питання прозвучали вперше
- Де відповідь була слабка
- Що треба підготувати краще

Додавай нові Q&A сюди. Це живий документ.

---

**v0.20.0 · травень 2026**
