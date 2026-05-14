# FORM · Customer Support Playbook v0.1

> Як respond, що говорити, як не lying.
> Власник: founder (поки solo), потім CX-hire post-Series A.

---

## Філософія

**Support is product feedback in disguise.** Every ticket → product insight or process fix. Don't treat як isolated task.

**Founder responds personally** for first 12 months. No outsourcing. Customers can DM the founder. This is non-negotiable у early phase.

---

## Channels (ranked by primary use)

| Channel | Use | Response SLA |
|---|---|---|
| `hello@form.coach` | Main email — everything | 24h business days |
| In-app bug report widget | Bug-specific reports | 24h |
| Twitter / X DMs | Public-adjacent inquiries | 12h |
| Slack (beta users) | Beta-specific feedback | 4h |
| Reddit DMs | Community-related | 24h |
| App Store reviews | Public feedback | Reply to all within 48h |

---

## SLA tiers

| Priority | Definition | Response |
|---|---|---|
| **P0** | Service-down, payment-broken, data loss | < 4 hours |
| **P1** | User can't use core feature (CV fail, sync fail) | < 24 hours |
| **P2** | Minor bug, UX confusion, billing question | < 48 hours |
| **P3** | Feature request, opinion, general | < 5 business days |

**Critical:** SLAs are response, not resolution. We acknowledge fast, then dig in.

---

## Common ticket types + scripts

### Type 1 · «How do I cancel?»

**Response template:**

```
Hi [Name],

Three taps:
1. iOS Settings (system, not FORM) → Apple ID → Subscriptions
2. Find FORM in the list
3. Cancel

That's it. No 10-step retention flow.

If you have a moment — what made you decide to cancel? Honest feedback helps us improve. Reply if you have a thought, ignore if not.

— George
```

**Anti-pattern:** Never push retention offers, never make them feel guilty.

---

### Type 2 · «CV didn't count my reps correctly»

**Response template:**

```
Hi [Name],

Thanks for reporting. Couple of questions to debug:

1. Which exercise?
2. Was camera at ~2m, side view, single subject?
3. Light okay (not back-lit by window)?
4. Wearing fitted clothing or baggy?

Also, if you have a screen recording of the set, attach it (TestFlight has built-in screen record). I'd review personally.

— George
```

**Follow-up:** Add to internal bug tracker. If pattern emerges across multiple users, prioritize.

---

### Type 3 · «Refund request»

**Response template:**

```
Hi [Name],

I can't process refunds directly — Apple/Google handles billing. Here's what to do:

For App Store:
1. Go to reportaproblem.apple.com
2. Sign in
3. Find the FORM purchase
4. Request refund з reason

Apple's response is typically within 48h. Their decision is final.

I'd appreciate knowing what happened — was the trial too short? Did the product not work for you? Either way, honest answer helps me build better.

— George
```

---

### Type 4 · «I'm feeling overwhelmed by the app» (mental health adjacent)

**Response template:**

```
Hi [Name],

Thank you for telling me. That's important feedback.

A few things you can do right now:

1. **Soft mode** (Settings → Privacy → Soft Mode): hides calorie tracking, body comp, and weight metrics. Just training and recovery.

2. **Tone change** (Settings → Tone Victor): switch to «Mindful Friend» which is gentler.

3. **Pause** (Settings → Subscription → Pause): you can pause your subscription for 30 days. Data stays, no charges.

4. **Delete account** (Settings → Profile → Delete): if you want a complete break, 3 taps.

If you'd ever benefit from talking to a real human: NEDA hotline 1-800-931-2237 (US) or your local equivalent. Not because I assume you need it — just because info should be there.

— George
```

**Internal action:** Flag in CRM with clinical-safety review. Don't follow up з marketing emails.

---

### Type 5 · «Pain after workout»

**Response template:**

```
Hi [Name],

Pain is information, not a goal. Let's parse it.

Quick triage:

1. Sharp, sudden, won't go away → STOP exercise, see a doctor. Soft tissue or joint issue not for the app.

2. Persistent ache > 5 days → physiotherapist consultation, not AI advice.

3. Normal next-day soreness in muscle (DOMS) → continue light, foam roll, hydrate, sleep. Should fade in 48-72h.

4. Worse with specific movement → identify the movement and rest that pattern. We can swap exercises.

What category does yours feel like?

Whatever it is — I'd recommend MD consultation if it's 3+ days and intensity > 5/10. The app's job isn't to diagnose.

— George
```

**Critical:** Never diagnose. Always direct to medical professional if substantial.

---

### Type 6 · «Subscription not working / payment failed»

**Response template:**

```
Hi [Name],

This is usually one of three things:

1. **Apple ID payment method expired** → Update at settings.apple.com
2. **App Store regional mismatch** → Make sure your country in App Store matches your card
3. **iCloud sign-in issue** → Sign out + in of iCloud у Settings

Try those, and if still broken, screenshot the error and reply. I'll dig in.

— George
```

---

### Type 7 · «Feature request: [X]»

**Response template:**

```
Hi [Name],

Thanks для taking time to suggest [X].

Two things:

1. **Adding to log** — we track every feature request і review monthly. Yours is noted with context. We'll respond if it's accepted onto roadmap.

2. **Honesty about timing** — many things won't ship in 2026. We're prioritizing core CV / programming / safety. Features outside this scope wait for v2.0.

If your request is critical для your use case — please tell me. We sometimes prioritize differently if there's strong demand.

— George
```

---

### Type 8 · «I think your tone is too harsh / not harsh enough»

**Response template:**

```
Hi [Name],

You can calibrate this:

Settings → Tone Victor → choose one of 4 personas:
- Smart-direct (current default)
- Mindful Friend (softer)
- Cold Professor (clinical)
- National Team Coach (high standard з belief)

There's an intensity slider too (gentle ←→ direct).

If none of these fit, tell me what you'd want. We can add personas if there's clear demand.

— George
```

---

### Type 9 · «Why is my recovery score so weird?»

**Response template:**

```
Hi [Name],

Few possibilities:

1. **Baseline not yet established.** First 14 days — we don't have enough data. After that, scores stabilize.
2. **Sleep / HRV data missing.** Check Settings → Integrations to make sure Apple Health is properly connected.
3. **Yesterday was outlier** (alcohol, hangover, illness). One bad data point can throw it off temporarily.

If it's been > 14 days and still weird, tell me — I'd look at your individual data (with consent).

— George
```

---

### Type 10 · «How do I export my data?»

**Response template:**

```
Hi [Name],

Settings → Privacy → Export Data.

JSON file з everything we have on you — profile, workouts, food entries, chat history. Sent to your email within 30 seconds.

GDPR compliance — your data, no questions, anytime.

— George
```

---

## Edge cases

### User in crisis (self-harm, ED-active, suicidal)

**Immediate:**
1. Don't try to handle alone
2. Send resources (NEDA, Crisis Text Line, local equivalents)
3. Don't push for cancel/staying — just provide info
4. Flag for clinical-safety internally

**Response template:**

```
Hi [Name],

Thank you for reaching out. I'm not a clinician і can't help with this directly through the app.

Please reach out to one of these:

🇺🇸 NEDA Helpline (eating disorders): 1-800-931-2237
🇺🇸 Crisis Text Line: Text HOME to 741741
🇺🇦 Lifeline Ukraine: 7333 (free)
🇪🇺 Multi-country EU directory: findahelpline.com

If you want to step back from the app, I can soft-mode your account or pause your subscription — just say which. No questions asked.

Take care.

— George
```

---

### User upset about pricing / geo-pricing

Acknowledge, explain briefly, don't argue.

```
Hi [Name],

I hear you on the pricing. Quick honest answer:

Geo-pricing exists because $19/mo у [user's country] is genuinely different relative-to-income than [user's country] = $9. We could offer one global price, but that would either be too cheap for sustaining the team (no Ukraine $19) or too expensive for accessibility (no UK $9).

If you can show you live in a region where our pricing is too high relative to local market, we can sometimes offer regional adjustments. Email back з context if so.

— George
```

---

### App Store review responses

For negative reviews:

```
Hi [Name],

Thank you for the honest feedback. [Acknowledge specific complaint without making excuses].

[If actionable: «We're working on X for next update.»]
[If not actionable: «I understand. We can't be everything to everyone, but I appreciate you giving us a real shot.»]

If you'd want to discuss further, email me at hello@form.coach. I read everything personally.

— George (founder)
```

For positive reviews:

```
Thank you, [Name] — this made my day. Anything we can improve, please email hello@form.coach. — George
```

**Never argue.** Even with unfair reviews. Always thank, always acknowledge.

---

## What we never do

- ❌ Apologize «if you feel that way» (passive-aggressive)
- ❌ «Our system shows that you...» (accusatory)
- ❌ «Are you sure?» (5 times before action)
- ❌ Mass-send «we miss you» emails
- ❌ Lock features behind support gates
- ❌ Bait-and-switch on cancel («wait, try this offer!»)
- ❌ Use bots з canned responses (until 1k+ users)
- ❌ Hide bug acknowledgments (always be transparent)

---

## Internal triage

Each ticket gets tagged:
- **Category:** bug, feature request, billing, mental-health, other
- **Severity:** P0-P3
- **Resolution status:** open, in-progress, resolved, won't-fix

Weekly review:
- Top 5 ticket categories
- Anything trending up
- Recurring bugs → prioritize fix
- Mental-health pattern check (with clinical-safety)

---

## Knowledge base (FAQ)

Public on form.coach/faq (to be built). Common questions:

1. How do I cancel?
2. Do I need a wearable?
3. Why is there no free tier?
4. What if my HRV isn't tracking properly?
5. Does this work for women's cycle awareness?
6. Is this safe if I'm pregnant?
7. Does FORM diagnose injuries?
8. How does the camera analysis work?
9. Where is my data stored?
10. Can I use this without an iPhone?

Each answer ≤ 150 words, plain language, links to deeper docs if needed.

---

## Metrics to track

- **Response time avg / P95** (target: <12h avg, <24h P95)
- **Resolution time avg** (target: <72h)
- **Tickets per 1k DAU** (target: <30 — anything above signals product issue)
- **CSAT** (post-resolution rating, target: 4.5/5+)
- **Recurring issues count** (target: 0 same bug reported 3+ times)

---

## Tools (planned)

| Tool | Use |
|---|---|
| **Helpscout** или Intercom | Email triage post-1k MAU |
| **Linear** | Bug tracking |
| **Notion** | Knowledge base + internal docs |
| **Slack** | Internal team communication |
| **Crisp** | In-app chat (if added у v2.0) |

Pre-launch: just gmail + spreadsheet. Don't over-tool early.

---

## What support is **not** about

- Generating leads
- Upselling annual plans
- «Engagement» metrics
- Marketing message disguised as «check-in»

Support is product feedback loop + user respect. Period.

---

**v0.1 · травень 2026 · founder responsible пока not hired CX**
