# Contributing to FORM

> Інженерні стандарти команди. Cloud-агенти і люди — однакові правила.

---

## Версіювання

Формат: **`MAJOR.MINOR.PATCH`** — semver-light.

| Bump | Коли | Приклад |
|---|---|---|
| **PATCH** | Кожна cloud-ітерація. Дрібний крок. | `0.2.1 → 0.2.2` |
| **MINOR** | Нова фіча, новий розділ доків, помітна зміна. | `0.2.5 → 0.3.0` |
| **MAJOR** | Брейкінг, pivot бренду чи скоупу. Не до релізу. | `0.x → 1.0.0` |

[`VERSION`](VERSION) — single source of truth.
Bump у тому самому коміті, що й зміна.

---

## CHANGELOG-дисципліна

Кожен коміт, що чіпає user-facing файли або документи — оновлює [`CHANGELOG.md`](CHANGELOG.md).

Секції (за Keep a Changelog):
- **Added** — нове
- **Changed** — зміна існуючого
- **Removed** — видалено
- **Fixed** — фікс бага
- **Security** — security-related

Правила запису:
- 1 буллет = 1 зміна
- ≤ 80 символів
- Жодних маркетингових формулювань
- Звичайна мова (UA або EN)
- Якщо може зашкодити — позначай (наприклад, ED-related → відмітити що clinical-safety review зроблено)

---

## Commit messages

Формат:

```
<scope>: <one-line summary> · v<new-version>

<optional body — що + чому, ніколи "як">

Co-Authored-By: ...
```

**Scopes:**

| Scope | Що чіпає |
|---|---|
| `site` | `index.html`, маркетинг-сайт |
| `docs` | `docs/*.md` (BRAND, FLOWS, MARKETING) |
| `agents` | `.claude/agents/*` |
| `brand` | bookkeeping у `BRAND.md` (токени, voice rules) |
| `meta` | `README.md`, `CHANGELOG.md`, `VERSION`, `CONTRIBUTING.md` |
| `flow` | онбординг, юзер-флоу прототипи |

**Приклад:**

```
site: add ED-screening to onboarding flow · v0.2.3

Adds 3-question SCOFF-light screen as non-skippable step
before any nutrition feature. Per clinical-safety review.

Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## Тегування

Після кожного push на `main` — тегни версію:

```bash
git tag v0.2.3 -m "FORM v0.2.3 · ED-screening in onboarding"
git push origin v0.2.3
```

Це створює видиму історію релізів на GitHub: https://github.com/Rbit27/form/releases

---

## Принципи команди (з [README](README.md))

1. **Жодних вигаданих цифр.** Цілі — окей. Результати — тільки коли є дані.
2. **`clinical-safety` має HARD VETO.** Все про їжу/тіло/травму/ментальне — проходить через нього.
3. **Якщо фразу не можна сказати другу в гімі — її не каже Victor.**
4. **Не пропускай два поспіль** — і для продукту, і для команди.

---

## Workflow для cloud-агента

Pre-commit checklist:

- [ ] Прочитано README.md, BRAND.md, FLOWS.md (релевантні розділи)
- [ ] Зміна не порушує жоден з 4 принципів
- [ ] VERSION bumped
- [ ] CHANGELOG.md оновлений у правильній секції
- [ ] Commit message слідує формату
- [ ] Якщо чіпає `food/body/injury/mental` — додано note що clinical-safety принципи дотримано
- [ ] `git tag vX.Y.Z` + `git push origin vX.Y.Z`

Post-commit:

- [ ] Push на `main`
- [ ] Перевірити що GitHub показує новий tag

---

## Workflow для людини (коли долучається)

1. Створи feature branch: `feat/short-description`
2. Дотримуйся всіх правил вище
3. Відкрий PR з описом, що матчить CHANGELOG entry
4. Один з агентів (зазвичай clinical-safety + product-strategist) рев'юіть
5. Squash-merge у main

---

## Чого НЕ робимо

- Force-push у main — **ніколи**
- Skip pre-commit hooks (якщо колись будуть) — **ніколи**
- Видалення тегів — **ніколи**
- Зміна минулих CHANGELOG-записів — **ніколи** (тільки додавання нових)
- Commit без bumping VERSION — **ніколи**, якщо чіпає user-facing

---

**v0.2.1 · травень 2026**
