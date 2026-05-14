# FORM · команда субагентів

Семеро. Кожен — одна зона. Координатор — Claude.

## Ролі

| Агент | Зона | Сила |
|---|---|---|
| `product-strategist` | scope, moat, pricing, claims | VETO на feature creep і вигадані цифри |
| `sports-scientist` | програми, HRV, періодизація, відновлення | джерела + рамки прогресії |
| `nutrition-coach` | план харчування, макроси, food-логіка | rd-перспектива з ED-обережністю |
| `clinical-safety` | ED, ментальне, травми, юридика | **HARD VETO** на все, що шкодить |
| `platform-engineer` | CV, носимі, мобільна архітектура, latency | каже що реально, а що research-bet |
| `brand-voice` | копірайтинг, голос Victor, UA/EN | проходить через clinical-safety |
| `design-craft` | типографіка, колір, motion, UI | VETO на AI-slop |

## Як вони працюють разом

Default-pipeline на нову фічу:

```
product-strategist  →  scope, moat, чи варто взагалі
        ↓
sports-scientist / nutrition-coach / platform-engineer
        →  що по змісту, що по техніці
        ↓
clinical-safety  →  VETO або список фіксів
        ↓
brand-voice  →  копірайт (UA + EN)
        ↓
clinical-safety  →  ще раз перевіряє копірайт
        ↓
design-craft  →  візуал
        ↓
ship
```

## Правила

- **`clinical-safety` має право зупинити будь-що.** Без винятків. Це не консультант — це блок.
- **`product-strategist` ріже scope.** Якщо команда хоче 12 фіч у v0.1 — він залишає 3.
- **Жодна цифра не з'являється на лендингу без джерела.** `product-strategist` цей пункт тримає.
- **Кожен текст про їжу / тіло / травму** проходить через `clinical-safety` перед публікацією.
- **Кожна технічна обіцянка** (приватність, точність CV, latency) — через `platform-engineer` на правдивість.

## Що ще можна додати пізніше

Не зараз — після того як v0.1 знайде сигнал:

- `growth-analyst` — CAC/LTV, ретеншн-метрики, A/B-тести
- `legal-compliance` — GDPR, FDA medical device, gambling-adjacent в gamification
- `community-lead` — соціальні фічі, модерація, безпека
- `ops-finance` — собівартість LLM-викликів, маржа, runway
