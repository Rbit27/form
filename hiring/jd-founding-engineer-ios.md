# Founding Engineer · iOS

**Location:** Remote (Київ / Львів / EU preferred; UTC±3 preferred)
**Type:** Full-time, founding team
**Compensation:** $90-130k base + 1.5-3.0% equity
**Start:** ASAP, by Q3 2026 latest

---

## What FORM is

FORM is an AI personal trainer for serious self-coached lifters. We're building a product that combines real-time computer-vision form analysis with HRV-aware programming and a coach voice that respects users.

Closed beta in summer 2026. Public launch Q4 2026. Stage: pre-product, raising Seed.

[Read more →](https://form.coach) · [About us →](https://form.coach/about.html) · [Manifesto →](https://github.com/Rbit27/form/blob/main/docs/FOUNDER.md)

---

## What you'll own

You'll be the first engineering hire. As such, you own the iOS app end-to-end:

- **Architecture decisions** — what's on-device vs cloud, how Vision pipeline integrates з SwiftUI, app state management
- **CV pipeline** — Apple Vision integration, pose-keypoint smoothing, custom heuristics per exercise (rep counting, tempo, form deviation)
- **HealthKit integration** — HRV, sleep, heart rate, workout data flow
- **Performance** — 30fps CV inference, < 80ms latency, battery optimization
- **Build pipeline** — TestFlight, App Store releases, crash reporting
- **Quality** — testing strategy, observability, debugging

You'll work directly з founder. No buffer, no management layer.

---

## Who you are

### Required

- 5+ years iOS development з Swift
- Shipped а consumer-facing iOS app to App Store з 100k+ users (yours or company's)
- Comfortable з SwiftUI (or migrated UIKit codebase recently)
- Working knowledge of Apple Vision, AVFoundation, CoreML
- Comfortable з performance profiling (Instruments, Time Profiler)
- Self-directed — can take a vague spec and ship a working feature

### Strongly preferred

- Experience з real-time camera processing (AR apps, video filters, fitness apps)
- HealthKit integration experience
- StoreKit 2 subscription experience
- Eng leadership experience (mentored juniors, ran code review)
- iOS app you're proud of, that we can look at

### Nice to have

- Knowledge of Android (для phase 2 transition)
- Sport science interest або lifting experience
- Ukrainian or Russian language (для UA team integration; not required if EU based)

---

## What you're NOT (we'll skip you for these)

- Looking для «AI startup» because it's hype
- Backend-only engineer who hasn't shipped consumer iOS
- Manager more than builder
- Someone who needs hand-holding на ambiguous problems
- Someone who treats fitness as «soft» domain — це serious technical challenge

---

## Tech stack

- **iOS:** Swift 6, SwiftUI, SwiftData / Core Data
- **CV:** Apple Vision framework (VNDetectHumanBodyPoseRequest)
- **Camera:** AVFoundation
- **Health:** HealthKit
- **Payments:** StoreKit 2
- **Networking:** URLSession (no Alamofire bloat)
- **Backend:** Cloudflare Workers, Supabase (you'll touch lightly)
- **Observability:** Sentry, PostHog
- **Build:** Xcode 16+, fastlane для releases

---

## What success looks like

### Month 1
- Shipped first user-visible improvement to TestFlight beta
- Understood codebase end-to-end
- Established personal flow з founder (sync cadence, communication)

### Month 3
- Owns CV pipeline (architecture + 1-2 new exercises shipped)
- Independently shipping features without founder review on minor items
- Has identified 2-3 system-level improvements + roadmap'd them

### Month 6
- Quality bar maintained (crash-free ≥ 99.5%)
- Hired (з founder) the 2nd engineer
- Drove App Store public launch successfully

### Month 12
- Promoted to Engineering Lead (depending on hiring trajectory)
- Owns iOS roadmap
- Mentoring 2nd-3rd engineer

---

## Compensation in detail

### Salary

**Base range:** $90,000 – $130,000 USD (annualized)

Final number depends on:
- Your experience (5 years vs 10 years matters)
- Cash vs equity preference (sliding scale)
- Geographic cost (Київ — top of band; EU — adjusted)

### Equity

**Range:** 1.5% — 3.0%

Standard 4-year vest, 1-year cliff. Refresh every 2 years for high-performers.

### Benefits

- Unlimited PTO з 20-day minimum
- Health insurance ($150/міс stipend)
- $1,500/рік edu budget (books, courses, conferences)
- $1,000/рік health & wellness budget (gym, therapy, sport)
- M3 MacBook Pro + chair + monitor + accessories на your choice
- $2,000 home-office setup budget
- Quarterly team offsites (Київ, Львів, Berlin, Lisbon — rotating)

---

## Application process

1. **Send us:**
   - Your CV / LinkedIn
   - Link to 1 iOS project you're proud of (App Store link)
   - Short note: «why FORM?» (≤ 200 words)
   - Email to: hiring@form.coach (subject: «FE iOS — [Your Name]»)

2. **Within 5 days:**
   - Founder will respond personally
   - If fit → 30-min intro call

3. **Take-home:**
   - 8-hour budget max
   - Build a basic pose-detection prototype з Apple Vision
   - We pay $500 for completed take-homes (whether hired or not)

4. **Deep technical session:**
   - 90 min з founder + future Tech Lead
   - Walk through your take-home + architecture discussion
   - Coding pair-programming (light)

5. **Cultural fit:**
   - 60 min з founder
   - Discuss values, working style, expectations

6. **References:**
   - 3 mandatory (direct manager + peer + report if applicable)

7. **Offer:**
   - Within 7 days of references complete
   - Offer letter via DocuSign
   - Negotiable on cash/equity split

**Total timeline:** 3-5 weeks from first email to offer.

---

## Working with us

### Schedule
- **Core hours:** 11:00-15:00 Київ (overlap для real-time collab)
- **Outside core hours:** async (Slack, Linear, GitHub)
- **No mandatory daily standups** — weekly sync + async updates
- **Workhours:** ~40h/week typical, more during launch

### Tools
- Slack (chat)
- Linear (tasks, roadmap)
- GitHub (code)
- Figma (design collaboration)
- Notion (docs)

### Communication
- Async-first, sync when needed
- Direct з founder daily (Slack DM or 15-min call)
- Decisions logged у Decision Log (process-keeper agent)
- Critical decisions з founder + relevant agent

---

## The honest truth

### Things you should know

1. **We're early.** Pre-revenue, pre-product-market-fit. There's not 5 years of legacy code, but there's also not 5 years of validation.

2. **CV is a research bet.** We claim 90%+ rep counting accuracy. Sometimes it's not. You'll have to navigate uncertainty.

3. **Ukraine context.** We're building from Kyiv during ongoing war. Infrastructure works. Team is committed. But there's real-world variance.

4. **Founder is solo eng for now.** You're not joining a 5-engineer team. You're joining a 1-engineer team and helping make it a 5-engineer team.

5. **Pay isn't FAANG.** $90-130k is solid для UA market and competitive globally for early-stage, but not Meta-level. Equity is meant to bridge that — but it's not guaranteed.

6. **Hours flex з launch cycle.** Some weeks 30h, some 50h. We don't believe у 80h hustle culture but we also don't believe у strict 9-5 у first year.

7. **You'll get full context.** Everything we discuss internally is shared з first hire. No layers of obfuscation.

### Things we can promise

- Honesty in every interaction (including post-hire feedback)
- Your work will be visible and impactful (you're not invisible)
- Crash-free sessions for users are everyone's job, not just yours
- We'll fight for your equity to mean something at exit
- We'll respect your time outside work hours
- We won't ask you to do things that compromise your professional ethics

---

## Contact

- **Email:** hiring@form.coach
- **Subject:** «FE iOS — [Your Name]»
- **Founder LinkedIn:** [TBD]
- **Public roadmap:** https://github.com/Rbit27/form

---

**v0.1 · травень 2026 · revisit перед public posting**
