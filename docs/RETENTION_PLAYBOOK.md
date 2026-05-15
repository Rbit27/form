# FORM · Retention Playbook v0.1

> Як утримати користувача після першого coached workout — і чому «утримати» означає «надати цінність», а не «уникнути cancel».
> Власник: `growth-lead` + `clinical-safety` review. Правки через PR.

Читай разом з:
- [`docs/METRICS.md`](METRICS.md) — W-ACSU definition, retention targets, anti-metrics
- [`docs/ANALYTICS_SETUP.md`](ANALYTICS_SETUP.md) — instrumentation для retention events
- [`docs/BETA_FEEDBACK_PROTOCOL.md`](BETA_FEEDBACK_PROTOCOL.md) — як читати якісний сигнал у beta
- [`docs/PRODUCT_SPEC.md`](PRODUCT_SPEC.md) — технічна база для нотифікацій і streak-логіки
- [`docs/EMAIL_SEQUENCE.md`](EMAIL_SEQUENCE.md) — email complement до in-app retention
- [`docs/BETA_PLAYBOOK.md`](BETA_PLAYBOOK.md) — загальна операційна структура beta

---

## 1 · Навіщо цей документ

**W-ACSU** (Weekly Active Coached Sessions per User) — North-Star Metric FORM. Детально у `METRICS.md`. Коротко: сесія рахується лише якщо Victor доставив ≥ 1 coaching cue або ≥ 1 plan adjustment. Тобто retention не «юзер відкрив app», а «юзер отримав коуча».

Цей документ відповідає на питання: **як ми побудуємо продукт так, щоб користувачі хотіли повертатись до coached sessions**, а не «як ми утримаємо їх у додатку довше за рахунок нотифікацій і streak-anxiety».

Різниця не семантична — вона визначає кожне дизайн-рішення у цьому документі.

### Хто використовує цей документ

| Роль | Навіщо |
|---|---|
| Founding Engineer | Implementing streak logic, notification system, pause/cancel flows |
| Product (post-hire) | Prioritizing retention features, feature gating schedule |
| Growth Lead | Defining notification copy, re-engagement protocol, experiment design |
| Clinical Safety | Review of all tone constraints — hard veto authority on any shame language |

Clinical safety reviewer має право заблокувати будь-яке retention copy без appeal. Це не bottleneck — це якість контролю.

---

## 2 · Критичні вікна утримання

> **Клінічна безпека:** Усі фрази Віктора у цьому розділі проходять клінічний review перед вбудовуванням у продукт. Жодної згадки ваги, тіла, зовнішнього вигляду, калорійного дефіциту. Тільки performance, відновлення, відчуття, сила, прогрес.

### Day 0 · Перший запуск

**Що відчуває користувач:** Curiosity + mild skepticism. «Подивимось що вміє цей AI-тренер». Порогова ситуація — перше враження визначає наступні 72 години.

**Що FORM робить:**
- Onboarding завершується coached session preview — не просто введення даних, а відчуття того, як Victor говорить.
- ED-screening (SCOFF-light) вбудований у onboarding як non-skippable. Ніякого способу обійти.
- Перший план генерується до кінця onboarding — юзер бачить «свій» workout до виходу з onboarding flow.
- Push notification permissions запитуємо після першого план-preview, не до.

**Victor може сказати (приклади):**
> «Добре. Я бачу у тебе [N] місяців стажу і ти тренуєшся [X] разів на тиждень. Ось план на цей тиждень — він не загальний, він починається з тебе.»

> «Перше тренування з FORM — найважливіше. Не через результат — через те, що ми побачимо, як ти рухаєшся. Записуй за звичним планом, або спробуй перший у моєму списку. Питання — питай.»

**Метрика вікна:** % юзерів, що завершили onboarding і запустили першу сесію впродовж 24 годин.

---

### Day 1-3 · Активація або смерть

**Що відчуває користувач:** Новизна знижується. З'являється перше питання — «це змінює щось, чи просто ще один log-app?». Це вікно де FORM має довести що Victor ≠ chatbot.

**Що FORM робить:**
- Одна notification на D1: не «поверніться», а результат першої сесії — конкретна coaching observation від Victor.
- Якщо перша coached session відбулась: Victor ready to follow up.
- Якщо першої сесії не було до кінця D2: одне нагадування про план (не «де ти?»).
- D3: якщо ще немає жодної сесії — тиша. Не спам. Email від George (beta sequence B-1) запитує що заважає.

**Victor може сказати (D1 notification після першої сесії):**
> «Перша сесія є. Я бачив [ліфт] — є одна деталь про постановку, покажу завтра на тренуванні. Відпочивай.»

> «[Name], план на завтра готовий. [Exercise] — 4×6, виходячи з того що бачив сьогодні.»

**Victor може сказати (якщо немає сесії до D2):**
> «Ще не починав — все добре. Твій перший workout чекає, коли будеш готовий. Ніякого тиску.»

**Метрика вікна:** % юзерів, що провели першу coached session в D0-D3 (activation gate).

---

### Day 7 · Перший справжній сигнал утримання

**Що відчуває користувач:** Або є ранній habit («звик планувати тиждень через FORM»), або перший пропуск вже стався і юзер оцінює чи повертатись.

**Що FORM робить:**
- Weekly review нотифікація: агрегований огляд тижня від Victor.
- Якщо юзер active: Victor підводить підсумок, дає preview наступного тижня.
- Якщо юзер пропустив 3+ дні: протокол re-engagement (секція 6).
- HRV baseline починає складатись — перший момент де Victor може зробити справді personalized план adjustment.

**Victor може сказати (weekly review, активний юзер):**
> «Тиждень один: [N] сесій, [N] coached cues. Я бачу що [конкретне спостереження — наприклад, ти сильніший на вечірніх тренуваннях]. Тиждень два: продовжуємо або міняємо — твій вибір.»

> «Цього тижня [exercise] пішов краще. Не через магію — через [конкретна корекція]. Наступного тижня спробуємо [прогресія].»

**Метрика вікна:** D7 retention rate — % юзерів активних на D7 (launched ≥ 1 coached session in day 5-9 window, ±2 days grace).

---

### Day 14 · Петля звичок або відтік

**Що відчуває користувач:** Якщо habit не склався до D14 — ймовірність D30 retention різко падає. Якщо склався — є сигнал довгострокового залишення.

**Що FORM робить:**
- D14 Progress summary — перша «14-day sparkline» нотифікація з реальними даними (не «ти молодець», а «ось що змінилось»).
- Якщо є ≥ 2 coached sessions за тиждень — activation confirmed. FORM може розблокувати наступний рівень Victor's personalization.
- Якщо є менше 2 sessions за тижень — один re-engagement attempt (секція 6), потім тиша.

**Victor може сказати (D14 summary):**
> «14 днів. [N] coached sessions. Я помітив [pattern — наприклад, ти пропускаєш понеділки але strong у середу-п'ятницю]. Хочеш план під цей ритм?»

> «Два тижні з FORM. Ось де є реальна різниця: [конкретний metric або ліфт]. Не кидаємо.»

**Метрика вікна:** % активованих юзерів (≥ 3 coached sessions у D0-D14), кореляція з D30 retention.

---

### Day 30 · Перший trial-to-paid decision point

**Що відчуває користувач:** Якщо trial — рішення платити. Якщо beta — перший місяць позаду. Раціональна оцінка «чи варте це грошей».

**Що FORM робить:**
- Ніяких нагадувань про ціну до D28 (не тиснемо на тлі retention logic).
- D28: транзакційний email (EMAIL_SEQUENCE.md, D-2) — нейтральний тон, без FOMO.
- Якщо юзер не subscribed і не скасував — одна in-app message від Victor про progress. Не про ціну.
- Monthly progress report: перший автоматичний звіт — sessions, coaching moments, observable trends.

**Victor може сказати (D30 in-app, non-paying user):**
> «Місяць. [N] сесій, [N] coaching cues. Ось твій звіт — не маркетинг, просто дані. Далі — твій вибір.»

**Метрика вікна:** Trial-to-paid conversion rate, D30 retention rate (окремо для converted vs churned).

---

### Day 90 · Sticky або churned

**Що відчуває користувач:** Або FORM — частина routine (high LTV), або юзер «оплачує але не використовує» (churn risk). Другий тип небезпечніший за явний churn — він дає false revenue signal.

**Що FORM робить:**
- Quarterly progress report: Victor підводить 90-денні підсумки. Real data, zero fluff.
- Якщо юзер paid but low-engagement (< 1 session/тиждень за останні 3 тижні): одне пряме питання від Victor — не «ми сумуємо», а «що заважає?».
- Якщо відповіді немає — пропонуємо Pause (секція 7).

**Victor може сказати (D90 low-engagement check-in):**
> «[Name], три місяці. Але останні тижні — тиша. Не питаю для галочки: що змінилось? Іноді plan просто не fits. Це можна виправити.»

**Метрика вікна:** D90 paid retention rate, engagement score for paid cohort (sessions/week trend).

---

## 3 · Notification Strategy

> **Клінічна безпека:** Жодна нотифікація не містить: ваги, calorie deficit, «ти пропустив», «не лінуйся», shame subtext будь-якого виду. Порушення — блок до clinical review.

### Типи нотифікацій

| Тип | Тригер | Приклад copy | Частота |
|---|---|---|---|
| **Reminder** | Scheduled workout / user-set window | «Сьогодні є [exercise]. Готовий — або дай знати якщо день не той.» | Max 1/день |
| **Celebration** | PR, streak milestone, first coached session | «PR на [ліфт]. Victor це бачив. Добре зроблено.» | Event-triggered |
| **Insight** | Weekly review, HRV-based plan adjustment | «HRV сьогодні нижче — план адаптовано: легша версія на сьогодні.» | 1/тиждень (review) |
| **Question** | Re-engagement trigger, 14-day check-in | «Давно не бачились. Що змінилось? (відповідь в один дотик)» | Max 1/event, max 3 total |

### Ліміти

- **Hard cap: 1 notification/день** для всіх нон-emergency типів.
- **Push max total: 4/день** (3 coaching + 1 system) — ніколи не досягаємо у нормальному використанні.
- **Quiet hours:** user-defined (default: 22:00-07:00 local time). FORM ніколи не надсилає за межею quiet window.
- **Opt-out:** одним кроком у Settings → Notifications. Reminder/Celebration/Insight вмикаються незалежно. Question-type (re-engagement) — єдиний mandatory non-zero.
- **Unsubscribe reminder:** якщо юзер disable-ував reminder notifications і не активний > 7 днів — FORM не надсилає нічого, не намагається «обійти».

### Notification copy rules (для того хто пише копі)

**CAN:**
- Посилатись на конкретний ліфт, конкретну дату, конкретний coaching cue
- Називати ім'я (якщо user надав)
- Давати preview наступного тренування
- Ставити питання без очікуваної відповіді
- Зазначати що Victor чекає (не що Victor сумує)

**CANNOT:**
- Вага, калорії, тіло, зовнішній вигляд
- «Ти пропустив», «Знову пропустив», «Де ти?»
- FOMO субтекст («усі тренуються крім тебе»)
- Emoji streak-flames або loss-signaling icons
- «Не зупиняйся», «Не здавайся» (передбачає відмову)
- «Ми сумуємо» (маніпулятивний субтекст)
- Капслок, знаки оклику більш ніж один

---

## 4 · Streak Mechanics

### Дизайн-філософія

**FORM рахує сесії, не consecutive days.**

Причина: запрограмований відпочинок є частиною програми. Користувач, який правильно відпочиває у понеділок-середу, не повинен «ламати streak» через те, що FORM рахує дні, а не sessions. Streak на основі consecutive days — це механіка яка карає фізіологічно правильну поведінку.

### Правило «Не пропускай два поспіль»

> «Не пропускай два поспіль» — це **floor**, не ціль. Це мінімальна норма самоповаги до тренувального процесу. Не мотивація, не гра — просто спортивна педагогіка.

Технічно: якщо юзер пропустив заплановану сесію — Victor зазначає це без коментарів. Якщо пропустив дві заплановані поспіль — Victor питає (одне питання, без тиску). Якщо три — re-engagement protocol (секція 6).

Логіка не «зламай streak» — логіка «є патерн, який варто зрозуміти».

### Weekly streaks > daily streaks

| Підхід | Що рахуємо | Проблема |
|---|---|---|
| Daily streak | Consecutive calendar days | Карає rest days, вихідні, хворобу |
| Session streak | Consecutive scheduled sessions | Кращий, але ignores weekly rhythm |
| **Weekly streak (FORM)** | **Тижні де ≥ N sessions** | **Синхронізується з реальним тренувальним циклом** |

**Реалізація:**
- Weekly streak: кількість тижнів підряд, де юзер провів ≥ свою цільову кількість сесій (встановлена в onboarding, default = 3).
- Grace: 1 тиждень зі скороченою активністю (≥ 1 coached session замість цільових N) не ламає streak — зменшує його якість, але не обнуляє.
- Reset: streak обнуляється якщо тиждень = 0 coached sessions.

### Що Victor робить коли streak переривається

Victor ніколи не:
- Каже «ти зламав streak»
- Використовує числові нагадування про streak loss
- Пропонує «відновити» streak через бонусну сесію

Victor може:
> «Минулий тиждень не склався — нормально. Цього тижня повертаємось до [N] сесій. Перша — [ліфт] у [день].»

> «Тижень пропустили. Питань немає — новий тиждень, новий старт. Твій план готовий.»

### Streak display у UI

- Показуємо streak (тижневий) у Progress screen — не на главному екрані.
- Не показуємо streak у push notifications (щоб уникнути anxiety).
- Streak milestone celebrations: 4 тижні, 8 тижнів, 12 тижнів — одна in-app card від Victor, без fanfare.

---

## 5 · Victor Personalization Arc

> **Клінічна безпека:** Victor ніколи не коментує зміни у зовнішності, вазі або фігурі — навіть якщо юзер просить підтвердження такого роду. Victor redirects до performance markers.

### Week 1 · Establishes Tone

Victor у першому тижні:
- Говорить мало, по суті.
- Задає питання і запам'ятовує відповіді.
- Не дає загальних порад — тільки ті що відносяться до конкретної сесії.
- Встановлює що він є: не cheerleader, не nutritionist, не терапевт — тренер з досвідом.

**Приклад Week 1 coaching cue:**
> «Коліна на squat — трохи всередину в нижній точці. Спробуй focus на push outward. Наступний сет — подивлюсь.»

**Приклад Week 1 plan note:**
> «Додав легку варіацію до [ліфта] на п'ятницю — хочу бачити як реагуєш. Якщо важко — скажи.»

### Month 1 · References Past Sessions

До кінця першого місяця Victor починає referencing попередній контекст.

**Progressive disclosure:** після 3+ coached sessions Victor «вмикає» memory layer — починає формувати coach context window з попередніх observations.

**Приклад Month 1 reference:**
> «Минулого разу ми говорили про hip hinge на deadlift. Сьогодні — перевіримо. Покажи мені перший сет.»

> «Ти казав що ліктьовий болів після bench. Сьогодні міняю кут — дай знати якщо знову є дискомфорт. Тренування через біль — не те що ми будуємо.»

### Month 3 · Pattern-Aware Coaching

До кінця третього місяця Victor має достатньо даних для справжнього pattern recognition.

**Що Victor може робити у Month 3:**
- Помічати day-of-week performance patterns («ти стабільно сильніший у вівторок і п'ятницю — підлаштовуємо важкі сети під ці дні»).
- Корелювати HRV-trends з тренувальною готовністю.
- Прогнозувати стрес-точки (перед відрядженням, у зимовий період) і проактивно адаптувати план.
- Пропонувати деblock тренувань («схоже що deadlift стагнує — є ідея, спробуємо?»).

**Приклад Month 3 pattern coaching:**
> «Три місяці даних. Ось що я бачу: [конкретний pattern]. Зважаючи на це — наступні 4 тижні будуємо так: [конкретна зміна плану]. Питання?»

### Progressive Feature Disclosure Schedule

| Тиждень | Що відкривається |
|---|---|
| W1 | Workout logging + CV form check (3 lifts) |
| W2 | HRV-based plan adjustments (якщо є HealthKit data) |
| W3 | Victor chat (питання між сесіями) |
| W4 | Weekly review notification + streak tracking |
| M2 | 14-day progress sparkline |
| M3 | Pattern-aware coaching cues + seasonal adaptation hints |

Нові фічі не анонсуються через push notifications — вони з'являються контекстуально, коли Victor має підставу їх використати.

---

## 6 · Re-Engagement Protocol

**Принцип:** максимум 3 спроби. Потім тиша. Спам не повертає — він руйнує бренд.

### Stage 1 · Passive (2 дні idle, пропущена заплановна сесія)

**Тригер:** юзер пропустив 1 заплановану сесію і не відкривав app 2+ дні.
**Дія:** 1 push notification від Victor.
**Tone:** neutral, informational — не emotional.

> «[Name], план на [день] готовий. Якщо день не підходить — просто перенесемо.»

Якщо немає реакції: тиша до Stage 2.

---

### Stage 2 · At-Risk (4+ дні idle)

**Тригер:** 4+ дні без відкриття app, ≥ 2 пропущених сесій.
**Дія:** 1 push + 1 email (якщо email enabled).

Push:
> «Чотири дні. Не питаю де ти — питаю що треба змінити. Відповісти можна прямо тут.»

Email (subject: «Питання від Victor»):
> «[Name], кілька днів без тренувань. Іноді це — правильне рішення (відновлення, хвороба, life). Іноді — сигнал що план треба змінити.
>
> Хочу знати що у тебе. Без тиску — один reply з парою слів.
>
> Victor (через George)»

---

### Stage 3 · Pre-Churn Warning (7 днів idle)

**Тригер:** 7 днів без активності.
**Дія:** 1 фінальний push (або email якщо push disabled).
**Tone:** graceful, не desperate.

> «Тиждень без сесій. Ми не зникаємо — але не будемо нагадувати, якщо не хочеш. Коли (і якщо) повернешся — план чекає.»

Після Stage 3: автоматичне mute на 30 днів. Жодних нагадувань.

---

### Stage 4 · Churned (14+ днів idle)

**Тригер:** 14 днів idle, paid user.
**Дія:** одна email від George (не від Victor) — не про продукт, а про причину.

Subject: «Що трапилось?»

> «[Name], [N] днів без активності. Я не намагаюсь тримати тебе — я хочу зрозуміти що пішло не так.
>
> Якщо план не підходив — скажи.
> Якщо Victor не ті нотатки — скажи.
> Якщо life просто заважало — теж скажи.
>
> Без subscriptions pitch. Просто reply.
>
> George»

Після цього email: тиша. Якщо юзер не відреагував — не ping. Якщо скасував — секція 7.

---

## 7 · Pause > Cancel Design

### Принцип

Юзер, що скасовує підписку у важкий момент (травма, хвороба, відрядження, фінансовий тиск) — часто повертається. Юзер що скасував через погано розроблений cancel-flow, з feeling of guilt — ніколи.

**Pause завжди пропонується першим.** Cancel вимагає 3 кліки. Обидва — без friction-traps.

### Pause Flow

1. Settings → Subscription → Pause
2. Вибір тривалості: 2 тижні / 1 місяць / 3 місяці
3. Confirmation: «Підписка на паузі до [date]. Billing не нараховується. Дані збережено.»
4. Victor надсилає одне повідомлення: graceful, без маніпуляцій.

**Victor при паузі:**
> «Узяли паузу. Розумно — іноді це правильне рішення. Коли повернешся — весь твій контекст збережений. Нічого не треба починати спочатку.»

### Cancel Flow

1. Settings → Subscription → Cancel (3 кліки від Settings)
2. **Крок 1 — Pause offer:** «Перш ніж скасувати — поставити на паузу? (без оплати до [date])» → Skip / Pause
3. **Крок 2 — Exit survey:** 2 питання (не більше):
   - «Основна причина скасування:» (multiple choice, max 6 варіантів + «інше»)
   - «Що б змінило рішення?» (open text, optional)
4. **Крок 3 — Confirmation:** «Підписку скасовано. Доступ до [end date]. Export даних: Settings → Privacy.»

**Жодного:**
- Confirmshaming («Так, я хочу відмовитись від тренування»)
- Progress-loss warnings («Ти втратиш свій streak!»)
- Retention discount pop-ups без прохання
- Повторних «ти впевнений?» екранів

### Victor при скасуванні

Одне повідомлення. Не після, а в момент confirmation screen:

> «Дякую що був тут. Дані збережені на [N] днів якщо зміниш думку. Якщо повернешся — пам'ять тренера залишається. Тримайся.»

Жодного follow-up push після cancel. Email від George — один (EMAIL_SEQUENCE.md, D-4), без closing pitch.

### Post-Cancel: Повернення

- Grace period: 30 днів. Реактивація одним кліком, без повторного onboarding.
- Після 30 днів: soft-delete (дані видалені, але account shell залишається 60 днів).
- Після 90 днів: повне видалення.
- Повернення після видалення: новий onboarding, Victor не пам'ятає (і не говорить про попередній досвід).

---

## 8 · Clinical Safety Constraints

> **Ця секція — операційний стандарт.** Хто пише notification copy, email copy, in-app messages, або будь-яке Victor copy — читає цей список перед відправкою на review.

### CANNOT (hard veto, без виключень)

- Вага, BMI, body fat percentage у будь-якому контексті
- Калорії (дефіцит, надлишок, «скільки ти з'їв»)
- Порівняння зовнішності — будь-яке, навіть позитивне («ти краще виглядаєш»)
- Shame language: «ти пропустив», «знову не дійшов», «de ти був?»
- Loss aversion manipulation: «не втрать прогрес», «streak зламається», «всі рухаються далі»
- Urgency tactics на тлі health context: «останній шанс», «скоро буде пізно»
- Body transformation promises: «до літа», «до весілля», «бажане тіло»
- Порівняння з іншими юзерами (індивідуальне або aggregate)
- Мова залежності: «твій тренер сумує», «FORM потребує тебе»

### CAN (approved categories)

- Performance metrics: сила, витривалість, рухомість, техніка
- Recovery markers: HRV, sleep duration, рested/fatigued signal
- Session data: кількість сесій, coaching cues, plan adjustments
- Personalized observations: конкретний ліфт, конкретна дата, конкретний патерн
- Future-plan statements: що заплановано, що Victor рекомендує
- Neutral check-ins: питання без передбачуваної відповіді
- Graceful exits: мова яка не карає за cancel, pause, або відсутність

### Gray zone (потрібен clinical review кейс-по-кейсу)

- «Ти сильніший» — OK якщо є конкретні дані (PR). Not OK як загальне заохочення.
- «Гарно виглядаєш» — завжди blocked.
- «Краще себе почуваєш?» — OK як питання. Not OK як твердження.
- Згадка хвороби або травми — Victor може acknowledge, але не diagnose і не prescribe.
- Менструальний цикл у context of HRV — можливо після консультації з clinical team.

### ED-Screening Protocol reminder

SCOFF-light screening — non-skippable в onboarding. Якщо screening видає ED-risk signal:
- Victor auto-enters soft-mode (без будь-яких food/calorie logging features)
- Ніяких нагадувань про їжу або вагу — назавжди у цьому акаунті
- Retention logic для цього юзера: тільки workout-based retention, нульова food-related комунікація
- Clinical safety reviewer отримує анонімізований alert

---

## 9 · Metrics to Track

> **Важливо:** жодних цільових benchmarks нижче не сформовано як факти. Це метрики, що ми будемо вимірювати під час beta, щоб встановити власні baseline. Порівняння з industry benchmarks проводиться після того, як є ≥ 2 повних когорти.

### Retention rates

| Метрика | Що вимірює | Де інструментовано |
|---|---|---|
| D1 retention | % юзерів з ≥ 1 app open на D1 | PostHog `app_opened` event |
| D7 retention | % юзерів активних в D5-D9 window | PostHog retention cohort |
| D30 retention | % юзерів з ≥ 1 session у week 4 | PostHog retention cohort |
| D90 retention (paid) | % paid users active in month 3 | PostHog + Stripe webhooks |
| Weekly streak distribution | Histogram: 0, 1, 2-3, 4-7, 8-12 тижнів | PostHog streak events |

### Notification effectiveness

| Метрика | Що вимірює | Healthy signal |
|---|---|---|
| Open rate by type | Reminder vs Celebration vs Insight vs Question | Встановлюємо з beta data |
| Tap-through rate | Notification → relevant screen | Встановлюємо з beta data |
| Unsubscribe rate by type | % юзерів що disable конкретний тип | > 30% = type needs redesign |
| Opt-out timing | В який день юзери disable push | Ранній opt-out = cadence issue |

### Streak health

| Метрика | Що вимірює |
|---|---|
| Weekly streak distribution | Скільки юзерів мають 1, 2, 3+ тижневий streak |
| Streak break rate | % streak breaks per week — monitor для burnout signal |
| Post-break return rate | % юзерів що повертаються після streak reset |
| Grace week usage | Скільки юзерів використали grace (1 сесія замість N) і продовжили |

### Pause and cancel

| Метрика | Що вимірює |
|---|---|
| Pause offer acceptance rate | % юзерів що вибрали Pause замість Cancel |
| Pause-to-resume rate | % paused юзерів що повернулись до платної підписки |
| Cancel step drop-off | Де юзери виходять з cancel flow (Pause offer / Survey / Confirm) |
| Exit survey completion rate | % що дали хоч одну відповідь |
| Exit survey top reasons | Категоризовані причини cancel — щотижнева review |

### Re-engagement

| Метрика | Що вимірює |
|---|---|
| Re-engagement response rate by stage | Stage 1/2/3 — % що повернулись після кожного attempt |
| Re-engagement churn after silence | % що не повернулись після Stage 3 → мовчання |
| Time-to-return distribution | Скільки днів між last active та re-engagement |

---

## 10 · Integration Map

Цей документ не існує окремо. Кожна секція посилається на або отримує input від інших docs:

| Документ | Що бере звідси | Що дає сюди |
|---|---|---|
| `METRICS.md` | W-ACSU definition, D1/D7/D30 targets, anti-metrics table | Retention rate raw data, notification open rates |
| `ANALYTICS_SETUP.md` | Instrumentation guide для всіх retention events | Event taxonomy для streak, re-engagement, cancel flows |
| `BETA_FEEDBACK_PROTOCOL.md` | Qualitative сигнали від 200 beta юзерів — доповнюють кількісні | Retention patterns що вимагають qualitative exploration |
| `PRODUCT_SPEC.md` | Technical scope: push notification system, StoreKit cancel flow, streak storage | Retention requirements що FE має імплементувати |
| `EMAIL_SEQUENCE.md` | Email complement до in-app retention (B-sequence, D-4, D-5) | Timing rules для email re-engagement (max 3 attempts) |
| `BETA_PLAYBOOK.md` | Beta operational structure, communication cadence, crisis protocols | Retention windows що визначають beta checkpoint emails |

### Потоки між документами

```
METRICS.md (what to measure)
    ↓
RETENTION_PLAYBOOK.md (how to intervene)
    ↓
ANALYTICS_SETUP.md (how to instrument)
    ↓
BETA_FEEDBACK_PROTOCOL.md (what qualitative data fills gaps)
    ↓
EMAIL_SEQUENCE.md + PRODUCT_SPEC.md (implementation)
```

---

## 11 · Open Questions для Founder + FE / Design Hire

### Q1 · Weekly streak vs session streak — що показуємо UI-користувачу?

Тижневий streak правильний методологічно, але менш інтуїтивний для юзера ніж «X сесій поспіль». Є ризик що складна метрика не мотивує взагалі — або що юзери інтерпретують неправильно і злаються на «баг» коли streak не росте. Питання для FE + design hire: як ми подаємо weekly streak у UI щоб він був одночасно точним і зрозумілим? Чи потрібні дві метрики (total sessions + weekly streak) або одна?

### Q2 · Наскільки aggressive може бути Victor Memory у Month 3?

Чим більше Victor referencing попередніх сесій — тим більше сприйняття «розумного тренера». Але є граничний ефект де це починає відчуватись як surveillance. Де межа? Наприклад: «ти казав що коліно болить» — OK. «Я помітив що ти тренуєшся частіше після важких тижнів на роботі» — можливо занадто. Founder + clinical safety мають вирішити де ця межа до того як Victor memory виходить у M2 feature.

### Q3 · Як обробляти паузу для beta users (безплатні 90 днів)?

Pause logic побудована для paid users. Beta users не платять. Якщо beta user «пропадає» на 3 тижні — чи застосовуємо re-engagement protocol так само? Чи є ризик що aggressive re-engagement у beta відштовхне саме тих юзерів чий feedback найцінніший (skeptics, low-engagement)? Founder + process-keeper мають вирішити перед Wave 1 launch.

### Q4 · Exit survey design: чи показуємо результати юзеру?

Поточний дизайн: 2 питання, відповіді йдуть тільки до team. Альтернатива: після відповіді показуємо агрегований anonymized результат («X% юзерів скасували через ціну — ми чуємо це»). Це може збільшити survey completion rate і відчуття «мене почули». Але додає complexity у UI і може збентежити якщо aggregate не відповідає конкретному кейсу. FE + design hire вирішують чи worth it.

---

**v0.1 · травень 2026 · `growth-lead` + `clinical-safety`**
