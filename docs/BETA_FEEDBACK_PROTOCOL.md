# FORM · Beta Feedback Protocol

> Як обробляти, класифікувати і використовувати фідбек від 200 бета-юзерів.

**Owner:** process-keeper (intake, routing) · research-lead (synthesis, insights)  
**Triggered by:** TestFlight launch · перша хвиля запрошень  
**Rhythm:** Daily triage · Weekly synthesis · Monthly report  
**Companion docs:** BETA_PLAYBOOK.md · COMMUNITY.md · RESEARCH.md · ANALYTICS_SETUP.md

---

## 1. Навіщо цей документ

Коли 200 людей починають тестувати FORM — фідбек приходить одночасно з 6+ каналів у різних форматах. Без системи є ризики:

- P0 safety issue губиться в шумі Slack
- Один голосний юзер тягне продукт у неправильний бік
- Реальні патерни не видно через суб'єктивне сприйняття
- Рішення приймаються без прив'язки до даних

Цей документ закриває loop між «у нас є бета-юзери» і «ми реально вчимося від них».

---

## 2. Канали фідбеку

| Канал | Тип | Очікуваний обсяг | Затримка |
|---|---|---|---|
| In-app кнопка «Feedback» | Structured + free text | Високий | Real-time |
| Slack / Discord бета-ком'юніті | Unstructured, soсial | Середній | Години |
| TestFlight crash reports | Bug/crash | Низький | Дні |
| Інтерв'ю-дзвінки (RESEARCH.md) | Deep qualitative | За графіком | За графіком |
| Email beta@form.coach | Edge cases, sensitive | Низький | Дні |
| PostHog behavioral data | Поведінковий | Безперервний | Real-time |

Пріоритет уваги: **safety concern будь-де > P0 bug > interview insight > все інше**.

---

## 3. Таксономія фідбеку

### Категорії

| Категорія | Приклади |
|---|---|
| **UX** | «Не розумію як знайти X», «Не очевидна кнопка», «Заплутав навігацію» |
| **Victor voice** | «Звучить занадто грубо», «Занадто generic», «Саме те що треба» |
| **Якість контенту** | Неправильна техніка, погана cue, помилка в програмі |
| **Performance** | Lag камери, затримка CV, неправильне розпізнавання |
| **Feature request** | «Хочу X», «Чому немає Y» |
| **Safety concern** | Травма, болі, ED-тригер, ментальне здоров'я → **негайна ескалація** |
| **Bug** | Краш, втрата даних, неочікувана поведінка |

### Рівні severity

| Рівень | Визначення | Час відповіді |
|---|---|---|
| **P0** | Safety concern АБО краш що блокує core use | ≤ 24 годин |
| **P1** | Core flow зламано для >10% юзерів | ≤ 3 дні |
| **P2** | Friction point — є workaround, але UX страждає | Наступний спринт |
| **P3** | Полірування, nice-to-have, feature request | Backlog review |

---

## 4. Процес intake

### Щодня (15 хв)

```
1. Перевір in-app feedback queue
2. Проскануй Slack/Discord на P0/P1 сигнали
3. Перевір email beta@form.coach
4. Кожен item тегни: Категорія + Severity + Канал
```

### Safety escalation — окремий трек

Будь-який item з тегом **Safety concern**:

1. НЕ намагайся вирішити самостійно
2. Негайно залучи clinical-safety для review
3. Задокументуй у DECISION_LOG.md (навіть якщо рішення «нічого не міняємо»)
4. Якщо юзер повідомляє про фізичну травму або психологічний дистрес — відповідь через 24h з warm referral на professional resources (формат у SUPPORT.md)

### Bug routing

P0/P1 bug:
1. GitHub Issue із повним repro (device model, iOS version, кроки)
2. Notify Founding Engineer
3. Не закривай issue поки фікс не верифіковано на TestFlight

---

## 5. Щотижнева синтеза (п'ятниця, 30 хв)

```
1. Порахуй items per category — кількісний сигнал
2. Перечитай всі P1+ items
3. Знайди патерни: "3+ юзери сказали X"
4. Запиши 3-5 bullets: "Сигнал цього тижня"
5. Порівняй з минулим тижнем: покращення чи деградація?
6. Онови DECISION_LOG.md якщо патерн тригерить рішення
```

**Поріг патерну:** Поведінка згадана 3+ непов'язаними юзерами = патерн для дослідження.  
Один запит, хоч як голосний — не патерн.

---

## 6. Щомісячний звіт (30-хв документ)

Наприкінці кожного бета-місяця:

```
## Місяць [N] · [дата] · [кількість активних юзерів]

### Кількість (що надійшло)
- Всього items: X
- По категоріях: UX X / Victor X / Performance X / Feature X / Safety X / Bug X
- По severity: P0 X / P1 X / P2 X / P3 X

### Якість (що почули)
- Топ-5 цитат (анонімізовані)
- Несподіванки: ...
- Підтверджені гіпотези: ...
- Спростовані гіпотези: ...

### Рішення (що зробили)
- Прийнято рішень: X (лінки до DECISION_LOG)
- Відкладено: X (причина)
- Відмовлено: X (причина)

### На наступний місяць
- Відкриті питання: ...
- Фокус дослідження: ...
```

Формат — внутрішній markdown. Не публікується. Використовується в INVESTOR_UPDATE_TEMPLATE.md.

---

## 7. Фільтр: фідбек → продуктове рішення

Не весь фідбек стає фічею. Фільтр:

```
Це патерн (3+ юзери)?
  НІ → логуй, моніторь. Поки не діяй.
  ТАК ↓
Суперечить core design principles FORM?
  ТАК → задокументуй чому НІ у DECISION_LOG.md
  НІ ↓
Потребує clinical-safety review?
  ТАК → clinical-safety першочергово, потім рішення
  НІ ↓
Вписується у скоуп v0.2 (PRODUCT_SPEC.md)?
  НІ → backlog, out of scope — чесно скажи юзеру
  ТАК ↓
Створи GitHub issue → спринт-беклог
```

---

## 8. Victor voice — окремий трекер

Голос Віктора це core product. Voice feedback — окремий bucket щотижня:

| Тип | Що фіксуємо |
|---|---|
| «Занадто жорстко» | Частота + контекст (яка вправа, який момент) |
| «Занадто generic» | Яка саме ситуація, якого коучингу бракувало |
| «Off-brand» | Що Viktor сказав vs що мав сказати |
| «Саме те» | Зберігаємо як positive examples для brand-voice |

Агрегуємо щомісяця → передаємо brand-voice для рефайну.

---

## 9. Чого НЕ робимо

- **Не обіцяємо** feature requests індивідуально
- **Не відповідаємо** «подумаємо» — тільки «дякуємо, важливо»
- **Не шерімо** індивідуальний фідбек публічно (privacy)
- **Не даємо** одному голосному юзеру визначати roadmap
- **Не шипимо** фікс без розуміння root cause
- **Не змінюємо** Victor's principles під тиском негативного фідбеку без pattern validation

---

## 10. Privacy rules

- Жодних імен юзерів у внутрішніх feedback-документах
- Цитати — тільки анонімізовані
- Цитати для публічного використання (кейс-стаді, testimonials) — тільки з явним письмовим consent
- Behavioral data (PostHog) — агрегати у звітах, не individual traces

---

## 11. Інтеграція з іншими документами

| Документ | Для чого |
|---|---|
| COMMUNITY.md | Slack/Discord channel setup і модерація |
| RESEARCH.md | Методологія інтерв'ю, synthesis framework |
| ANALYTICS_SETUP.md | PostHog event reference (65 events, W-ACSU) |
| BETA_PLAYBOOK.md | TestFlight операції |
| DECISION_LOG.md | Формат запису рішень |
| PRODUCT_SPEC.md | Скоуп v0.2 (що in/out) |
| SUPPORT.md | Відповідь юзерам при safety concern |
| EMAIL_SEQUENCE.md | Beta welcome + follow-up templates |

---

## 12. Відкриті питання

- [ ] Чи будувати lightweight in-app feedback UI у v0.2, чи покладатися на email + ком'юніті? (→ рішення Founding Engineer)
- [ ] Що робити якщо юзер повідомляє про фізичну травму під час вправи що рекомендував FORM? (→ LEGAL.md + clinical-safety протокол, розробити до TestFlight launch)
- [ ] Яка мінімальна частота фідбеку від юзера per month вважається «активним» бета-тестером? (→ BETA_PLAYBOOK.md)

---

*Власник: process-keeper · Оновлювати при зміні бета-процесу · Версія v0.22.0*
