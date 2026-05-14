# FORM · Launch Checklist v0.1

> Усе, що має бути зроблено до public launch. Чесний gate.
> Власник: founder. Stamp перед launch: cross-team review.

---

## Принцип

**Якщо хоч один P0-чекбокс не tick'нуто — НЕ shipпуємо.**

«Move fast and break things» — для toys. Це продукт, який чіпає тіло і психіку людей. Move carefully and ship right.

---

## P0 · Безпека (clinical-safety + legal)

### Mental / ED safety
- [ ] ED-screening flow на онбордингу tested з 30+ users
- [ ] Soft-mode (без calorie tracking) повністю functional
- [ ] Body-comp metrics opt-out працює (hide forever)
- [ ] Suicide-safe messaging guidelines reviewed (NEDA, APA standards)
- [ ] Chat-classifier виявляє ED-trigger language → soft response + resources
- [ ] Calorie floor enforced (≥ 1,200 ккал, з warning якщо approached)
- [ ] No surveillance language у нотифікаціях (jeden audit)
- [ ] Streak-with-grace tested (1 miss/week дозволений)

### Medical safety
- [ ] Pain triage flow protokol → MD recommendation якщо ≥ 5/10 або > 5 днів
- [ ] No AI-generated diagnosis у chat (template-only для injury cases)
- [ ] Pre-existing condition screening at onboarding (PAR-Q-light)
- [ ] Age gate ≥ 18 enforced
- [ ] Pregnancy screening + warning flow
- [ ] Cardiac risk warning для users 40+

### Privacy / GDPR
- [ ] Privacy Policy reviewed by qualified counsel
- [ ] ToS reviewed by qualified counsel
- [ ] Data Processing Agreements signed з усіма processors
- [ ] Standard Contractual Clauses (SCCs) для US-EU transfers
- [ ] DPIA (Data Protection Impact Assessment) для health data
- [ ] Consent management platform live для marketing site
- [ ] Right to erasure (3-tap delete + 30-day grace) tested end-to-end
- [ ] Right to portability (JSON export) tested
- [ ] Cookie banner compliant для EU visitors

---

## P0 · Якість продукту

### Crash + stability
- [ ] Crash-free sessions ≥ 99.5% у last beta cohort (target 99.8%)
- [ ] iOS 17 + 18 + future-проверка
- [ ] iPhone 12, 13, 14, 15, 16 tested
- [ ] iPad: graceful UX (не optimized, але не broken)

### CV quality
- [ ] Rep counting ≥ 90% на squat / bench / deadlift
- [ ] Tempo measurement ≥ 85%
- [ ] False-positive form alerts ≤ 15%
- [ ] Honest disclosure про CV limitations у onboarding + settings

### Integration testing
- [ ] HealthKit data flowing коректно
- [ ] Whoop API tested (якщо launch включає)
- [ ] Oura API tested (якщо launch включає)
- [ ] Apple Watch workout sessions tracked

### Performance
- [ ] App cold-start < 1.5s
- [ ] API P95 latency < 1.5s
- [ ] CV inference < 80ms per frame
- [ ] Battery drain < 30%/hour у workout mode

---

## P0 · Бізнес / монетизація

### Payment flow
- [ ] Apple In-App Purchase configured
- [ ] Subscription products created в App Store Connect
- [ ] Receipt validation working end-to-end
- [ ] Trial → paid conversion tested
- [ ] Cancel flow (3 кліки) tested
- [ ] Refund process documented

### Pricing
- [ ] Geo-pricing configured (UA $9, Western $19)
- [ ] Annual plan з -34% дискаунтом
- [ ] App Store auto-prices set
- [ ] Currency conversion handled by Apple

### Anti-fraud
- [ ] Apple Sign In single source of identity
- [ ] Rate limiting на critical endpoints
- [ ] Subscription validation prevents tampering

---

## P0 · Маркетинг + App Store

### App Store listing
- [ ] Title finalized (≤ 30 chars)
- [ ] Subtitle finalized (≤ 30 chars)
- [ ] 5 screenshots prepared (per device size)
- [ ] App preview video (15-30 sec) prepared
- [ ] Description написана + reviewed
- [ ] Keywords researched + entered
- [ ] Category selected (Health & Fitness)
- [ ] Age rating set (17+ через health data sensitivity)
- [ ] Privacy "nutrition labels" filled correctly

### Marketing site
- [ ] index.html final review
- [ ] About page (about.html)
- [ ] Pricing page (pricing.html)
- [ ] Beta closed → public-launch landing
- [ ] Privacy Policy + ToS pages live
- [ ] Footer links все correct
- [ ] SEO basics (meta tags, OG images)
- [ ] Analytics tracking working

---

## P1 · Customer support readiness

- [ ] Email support channel (hello@form.coach) live
- [ ] Auto-responder set up для first-line
- [ ] FAQ doc (внутрішня) — 30 common questions answered
- [ ] Bug-report flow in-app working
- [ ] Founder available для first 48h (DND-mode off)
- [ ] Triage rules для P0-P3 bug levels

---

## P1 · Analytics + monitoring

- [ ] Sentry crash reporting live + alerts configured
- [ ] PostHog product analytics live + funnel tracked
- [ ] Cloudflare logs flowing
- [ ] Status page (status.form.coach) live
- [ ] Pingдom або similar uptime monitoring
- [ ] Weekly metrics dashboard prepared (Excel або Notion)

---

## P1 · Content

- [ ] 6+ blog posts published (current: 5)
- [ ] Newsletter sign-up form live
- [ ] First newsletter sent до waitlist
- [ ] Twitter / X account active з 10+ posts
- [ ] Reddit account established (для community participation)

---

## P2 · Legal / corporate

- [ ] Company incorporated (Ukraine + Delaware C-Corp якщо raising)
- [ ] Bank account opened
- [ ] Accounting software set up
- [ ] First financial statement
- [ ] Trademark filing initiated для "FORM."
- [ ] Domain transfers complete (form.coach)
- [ ] Email forwarding live
- [ ] Insurance (technology errors & omissions) considered

---

## P2 · Operational

- [ ] Decision log started (process-keeper)
- [ ] Slack workspace (internal) set up
- [ ] GitHub repo permissions reviewed
- [ ] Backup procedures tested (Supabase point-in-time recovery)
- [ ] Incident response playbook reviewed
- [ ] Secrets vault audited

---

## Day-1 launch sequence

```
T-24h
├── Final go/no-go review
├── Last bug-fixes если any P0/P1
└── Team sync: communications plan

T-12h
├── App Store submission status verified
├── Marketing site live check
├── Email sequences armed
└── Monitor dashboards online

T-0 (launch hour)
├── App Store goes live
├── Twitter announcement thread posted
├── Newsletter sent to waitlist
├── Reddit posts go up (relevant subs)
├── Founder available for first 4h на Slack/X DMs
└── Team monitors all dashboards

T+24h
├── First-day metrics review
├── Any P0 bugs → emergency patch sprint
├── Press response (if coverage)
└── Customer support triage

T+72h
├── 3-day post-launch report
├── Decision Log update
├── Priority adjustments for week 1
└── Investor email update sent
```

---

## Roll-back plan

Якщо щось critical:

| Severity | Action |
|---|---|
| Privacy breach | Pull from sale через App Store, public statement within 6h |
| Critical bug (crash > 5%) | Hotfix immediately, push update within 12h |
| ED-trigger incident | Soft-mode for entire user base, fix і communicate |
| Payment broken | Disable trial → paid; pause acquisition; fix |
| Negative press | Founder responds publicly + factually within 24h |

**Critical rule:** ніколи не приховуємо. Завжди transparent. Worst PR is hidden PR.

---

## Defining «launched»

«Public launch» = App Store live + marketing announced.

**Don't conflate з successfully launched.** Success metrics:
- Week 1: 500+ installs, < 1% crash rate, 1+ press coverage
- Week 4: 1500+ paid trials started, conversion ≥ 10%
- Quarter: $50k+ MRR, NPS ≥ 35

Якщо Week 1 misses — pause і analyze, don't push paid acquisition into broken funnel.

---

## What we explicitly do NOT do at launch

- ❌ Massive marketing blitz (organic-first, paid stays off)
- ❌ Influencer-shilling campaign
- ❌ "Limited time" launch deals (we don't do that)
- ❌ Public stunt PR
- ❌ Over-promise у communications
- ❌ Hide P0 bugs у last-minute "look good" patch

---

**v0.1 · травень 2026 · review pre-launch**
