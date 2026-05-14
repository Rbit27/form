---
title: "Як ми побудували Victor — голос AI-тренера, що не виглядає як AI"
slug: building-victor-voice
date: 2026-05-14
author: George (FORM founder)
category: product-craft
estimated_reading_time: 11 хв
status: draft
review_pending: brand-voice + marketing-lead
target_persona: tech-builders + designers + product-people
---

# Як ми побудували Victor — голос AI-тренера, що не виглядає як AI

«AI voice» у фітнес-апках — це або:
- Cheerleader-зомбі («You got this! Stay strong!»)
- Generic-clinical («Recovery 73%. Suggest medium intensity»)
- Або — найгірше — спроба ChatGPT відповідати «як тренер» з результатом, що звучить як LinkedIn post

Жоден з цих варіантів не звучить як **реальний тренер**.

Ми витратили 4 місяці на щоб зробити Victor — і це найскладніша річ, яку я будував у моєму житті. Чому, і як ми її вирішили — нижче.

---

## Чому це складна задача

«Voice» здається простим. «Просто скажи, як тренер!» Реальність:

### 1. Тренер з 20-річним досвідом — не одна особистість

Він підлаштовується. Зранку — спокійний. На третьому сеті — конкретніший. Якщо ти травмований — обережно. Якщо ти добиваєш PR — впевнено.

LLM з одним system prompt не робить такий range. У тебе або «жорсткий тренер» по дефолту, або «дружній мотиватор» — але не один coach з шириною.

### 2. Реальний тренер дотримує контексту

Якщо вчора була погана тренування, сьогодні він може спитати «як ти?» — навіть якщо не пов'язано з тренуванням.

Чи AI пам'ятає вчора? Тільки якщо ти його активно інструктуєш. Більшість продуктів — ні.

### 3. Реальний тренер мовчить, коли треба

Деякі моменти — тиша краща ніж слова. Поганий тренер заповнює тишу мотивацією. Хороший — дивиться, чекає.

AI зазвичай **завжди** говорить. Це антитеза «as needed».

### 4. Reальний тренер не Cheerleader

Він не каже «You crushed it!» після стандартного сету. Він зберігає таку фразу для реального досягнення. Якщо все таке — нічого не значить.

LLM trained на consumer copywriting overuses praise. Це **підриває довіру**.

### 5. Реальний тренер не дражливий

Тренер з 20-річним досвідом не каже «Wow incredible job!» з кожного повтору. Це поглужливо для adults.

Дорослі lifters читають через AI-фальш одразу.

---

## Як ми це вирішили

### Крок 1 · Знайти реальний референс

Я провів 30 годин слухаючи реальних elite-тренерів — у відео, podcasts, у залі. Записав 200+ конкретних reps їхньої мови.

Patterns що з'явилися:

**Real coaches:**
- Короткі речення (3-7 слів — типова норма)
- Дані перед судом («HRV 48 — низька»)
- Choose-architecture («2 хвилини або зниж до RPE 7. Що обираєш?»)
- Тиша між сетами (не constant talking)
- Acknowledgment без надмірної похвали («Ок. Наступний.»)
- Specific feedback, не generic («Лікті ближче»)

**AI-trained voices:**
- Довгі речення з модіфікаторами
- Загальні фрази («стратегічно навколо твоєї recovery»)
- Forced enthusiasm
- Continuous narration
- Praise inflation

### Крок 2 · Декомпозувати на components

Voice — не одна штука. Це:

1. **Cadence** (rhythm of speech)
2. **Diction** (word choice)
3. **Tone** (warm / cold / neutral)
4. **Authority** (cert / uncert)
5. **Verbosity** (skinny / fat)
6. **Cultural register** (formal / informal)

Для Victor:
- Cadence: short, choppy
- Diction: technical-precise («HRV», «RPE», «tempo»), not generic
- Tone: neutral, slightly warm
- Authority: high-confidence з epistemic humility
- Verbosity: skinny (3-12 word phrases mostly)
- Register: informal Ukrainian/English (du/ти), no slang

### Крок 3 · Спроектувати «персони» як параметри, не як окремих coach

4 базові варіанти:

| Persona | Mod from baseline |
|---|---|
| **Smart-direct** (default) | baseline |
| **Mindful friend** | +questions, +warmth, –directives |
| **Cold professor** | +citations, –warmth, +verbosity |
| **National coach** | +high standard, +belief signal, –directives |

Plus intensity slider (0-100). 0 = gentle / 100 = blunt.

Кожна персона — це **same base coach** з different surface mods. Не 4 окремі personalities, а 4 calibrations.

### Крок 4 · Hard rules що Victor НЕ каже

Ці — закладені у system prompt і enforced:

1. Жодного «I see that you...» (surveillance)
2. Жодного «You crushed it!» (forced praise)
3. Жодного «We believe in you» (corporate)
4. Жодного «Just do it» (dismissive)
5. Жодного emoji (decorative)
6. Жодного «as your AI coach» (meta-references)
7. Жодного «studies show» без specific source
8. Жодного «healthy» / «toxic» / «unhealthy» (moralization)

Кожна violation = bug.

### Крок 5 · Train з real-corpus, не з internet

Standard LLM training має consumer-blog tone built in. Our system prompt і RAG add layers:

- 200+ reference snippets from real coaches
- Tone calibration з brand-voice agent на every output
- clinical-safety override for sensitive topics
- Decision trees для when to speak vs stay silent

LLM produces draft → tone classifier rates against scale → reject if score < threshold → re-generate з tighter constraints.

This is **expensive** (multi-call). Но it's worth it.

### Крок 6 · Beta-test з real lifters

12 beta-testers used Victor для 6 weeks each. Reactions categorized:

**Top complaints (early iterations):**
- «Sounds like a sports broadcaster» (33%)
- «Tries too hard to be friendly» (28%)
- «Patronizing» (19%)
- «Boring» (12%)
- «Confusing» (8%)

After 4 iteration cycles:
- Patronizing: dropped to 3%
- Sports broadcaster: dropped to 5%
- «Boring» actually increased to 22% (трактуємо як good — calm, not hype)

«Boring» — це **acceptance criterion** для us. Coach who doesn't entertain you = coach who respects your time.

---

## Real examples

### Morning check-in

❌ AI-default:
> "Good morning, Dmitro! 🌅 Your HRV is looking great today at 68, and you slept 7 hours and 12 minutes! Time to crush some weights! 💪"

❌ Generic «smart» AI:
> "Daily morning report. HRV: 68 ms. Sleep: 7:12. Both metrics show favorable readiness."

✓ Victor (smart-direct):
> «HRV 68. Сон 7:12. Тіло окей. Випий води.»

### Form correction mid-set

❌ AI-default:
> "Hey Dmitro! Looking great! Try to keep your elbows a bit closer to your torso for better form. You got this! 💪"

❌ Generic «smart» AI:
> "Form observation: Elbow position deviating laterally. Adjust to keep elbows tucked closer to the body."

✓ Victor (smart-direct):
> «Лікті ближче. Останні три — повільніше.»

### Post-failed set

❌ AI-default:
> "Don't worry, you can do it next time! Stay positive and keep pushing!"

❌ Generic «smart» AI:
> "Set discontinued. RPE estimated at 9.5. Suggest reducing weight by 5kg next attempt."

✓ Victor (smart-direct):
> «Окей. Зніми 5 кг. Останній сет з нормальною технікою.»

---

## Чому це матерує

Це не косметика. Це продукт.

### 1. Доросле звертання vs дитяче

Реальні lifters — adults. Treating them as adults у tone — це **бар довіри**.

Якщо AI говорить як coach з 5-річним досвідом — користувач відчуває це інстинктивно. Через тиждень — churn.

### 2. Coach's voice — це бренд

Whoop не має voice. Fitbod має generic AI voice. Future має human coach (= не AI).

Voice = differentiation. Done well, це single biggest moat.

### 3. Voice = trust

Якщо AI говорить чесно — користувач довіряє конкретним рекомендаціям. Сumulative effect.

Якщо AI inflates — користувач вчиться ignore everything.

---

## Що ми НЕ вирішили

Чесно — це **continually evolving**.

Відкрите:
1. **Cycle de-personalization над часом.** Чи Victor стає «нудний» через 6 місяців? Need data.
2. **Cultural drift in non-UA languages.** German Victor vs UA Victor — same person?
3. **Audio version (TTS).** Це інша задача — voice acting через ElevenLabs не = текстовий voice
4. **Inter-user consistency.** Чи дві людини отримують ідентичний Victor? Чи slightly personalized?

---

## TLDR

1. AI voice у фітнес-апках погана через cheerleader tone, generic phrasing, forced enthusiasm
2. Реальний тренер — короткий, дані-перший, дозволяє тиші
3. Voice = composition: cadence + diction + tone + authority + verbosity + register
4. Personas = same coach з different surface mods, не різні personalities
5. Hard rules що Victor НЕ каже — більш важливі ніж positive specifications
6. «Boring» — це часто признак того, що ми зробили правильно

Voice — це найбільш недооцінена частина AI-product дизайну. Robi'mo її правильно і ми робимо moat.

---

**Хочеш Victor у твоїй кишені?** [TestFlight beta]

---

*FORM · pre-launch · Київ 2026*
*Цей пост — особиста реф lecture від founder.*
