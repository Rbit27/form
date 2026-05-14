# FORM · Engineering Runbook v0.1

> Як реально operate engineering в production.
> Власник: `platform-engineer` + founding engineer (after hire).

---

## 1 · On-call

### Phase 0-1 (pre-launch, до 200 beta users)

**On-call:** founder, all hours.

Tools:
- Sentry alerts → personal email + Slack (founder's phone)
- Cloudflare Workers logs accessible через dashboard
- Supabase status page bookmarked
- Apple Developer alerts → personal email

Response targets:
- P0: < 2 hours acknowledge
- P1: < 8 hours acknowledge
- P2: same day

### Phase 2 (post-launch до Founding Engineer)

**On-call:** founder primary, Founding Engineer fallback.

Adopt PagerDuty або similar. Rotation week-by-week. Compensated $200/week on-call.

### Phase 3+ (team of 3+)

Rotation across full team. No solo on-call > 2 weeks at a time.

---

## 2 · Deployment

### Mobile (iOS)

**Production releases:**
1. Build via Xcode Cloud
2. TestFlight push з internal testers (founder + Eng + Design)
3. 48-hour soak time
4. Promote to external TestFlight (beta cohort)
5. 72-hour soak time mid-beta
6. App Store submission
7. Apple Review (~24-72h)
8. Manual release when approved

**Hotfix process:**
1. Branch from `main` → `hotfix/short-description`
2. Fix + test
3. Expedited TestFlight з ALL testers immediately
4. 12-hour soak з beta cohort
5. App Store з expedited review request
6. Manual release ASAP

**Rollback strategy для mobile:**
- Can't truly rollback (App Store immutable)
- Hotfix forward з minimal version increment
- Communicate transparently если bug shipped

### Backend (Cloudflare Workers + Supabase)

**Standard release:**
1. PR → review → merge до `main`
2. Auto-deploy до staging via GitHub Actions
3. Smoke tests pass
4. Manual promote to production
5. Monitor metrics 30 min post-deploy

**Hotfix:**
1. Direct push to `main` allowed for P0
2. Cloudflare deploy via wrangler
3. Watch error rates real-time

**Rollback backend:**
- `git revert` previous commit
- Auto-redeploy
- < 5 min recovery

---

## 3 · Monitoring

### Dashboards (live, monitor daily)

**Sentry:**
- Crash-free sessions: target > 99.5%
- Top 10 errors by frequency
- Errors by iOS version
- Errors by device model

**PostHog:**
- DAU / MAU
- Activation funnel (install → onboard → first session → 3 sessions)
- Feature adoption (camera coaching, chat, integrations)
- Cancel-flow events

**Cloudflare Analytics:**
- Request volume per endpoint
- Error rate per endpoint
- P95 / P99 latency
- Bandwidth usage

**Supabase:**
- Active connections
- Query latency
- Storage usage
- API request rate

### Alerts

**P0 (immediate page):**
- Crash rate > 1% in 1-hour window
- API error rate > 10% in 15-min window
- Database unreachable for > 1 min
- Anthropic API down for > 5 min
- Sign-up endpoint returning errors

**P1 (Slack notify, respond within hour):**
- Crash rate > 0.5%
- API error rate > 5%
- Single-user payment errors (cohort of 1+ within 1h)
- Wearable sync failures у > 10% of users for > 1h

**P2 (daily digest):**
- Crash rate > 0.2%
- Slow endpoints (P95 > 2s)
- Feature flag anomalies

---

## 4 · Incident response

### Severity classification

| Level | Examples | First-response time |
|---|---|---|
| **SEV-0** | Data loss, security breach, paid sign-ups blocked | < 30 min |
| **SEV-1** | Major feature broken (CV, chat down) | < 2 hours |
| **SEV-2** | Minor feature degraded | Same day |
| **SEV-3** | UI bugs, single-user issues | < 5 days |

### SEV-0 runbook

```
T+0  Detected (via alert or user report)
T+5  Incident channel #incident-YYYY-MM-DD created
T+10 Containment: revert / disable feature / scale up
T+15 Status page updated
T+30 First user-facing communication (Twitter + status page)
T+60 Internal root-cause hypothesis
T+2h Decision: fix-forward або extended rollback
T+4h External update з timeline
T+24h Resolution OR detailed status
T+48h Post-mortem draft published
T+7d Post-mortem with action items final
```

### Communication during incident

**Internal (Slack #incident channel):**
- Time-stamped updates every 30 min
- Single «Incident Commander» (usually on-call)
- Don't speculate publicly
- All hands welcome if SEV-0

**External (status page + Twitter):**
- First update within 30 min of detection
- Update every 60 min minimum
- Plain English, не technical jargon
- «What happened» before «what we're doing»
- No fault-projection («Apple API broke») without certainty

---

## 5 · Code review

### Pull request requirements

- [ ] Description explains what + why (не how)
- [ ] Tests added або «no tests required because…»
- [ ] Self-reviewed first (catch obvious mistakes)
- [ ] Linked to issue або decision log entry
- [ ] CI green

### Review responsibilities

- One reviewer minimum (two for backend security-touching code)
- Reviewer responds within 4 hours during business
- Author follows up на comments within 24 hours
- Don't ship з unresolved blocker comments

### Reviewer checklist

- Code correctness
- Error handling
- Security implications
- Performance implications
- Test coverage
- Readability for next person
- Не over-engineering

---

## 6 · Secrets management

### What goes where

| Secret | Storage |
|---|---|
| Anthropic API key | Cloudflare KV (encrypted) |
| ElevenLabs API key | Cloudflare KV |
| Supabase service key | Cloudflare KV |
| Apple App Store secret | Cloudflare KV |
| Database connection string | Cloudflare KV |
| Apple Sign In public keys | Auto-fetched, cached 24h |

### Access control

- Solo founder: 1Password vault з hardware key
- Post-hire: AWS-IAM-style role separation via Cloudflare Access
- No shared accounts
- Rotation: quarterly для API keys, semi-annual для DB password

### What we never do

- ❌ API keys у client code
- ❌ Secrets у commit messages або logs
- ❌ Plaintext secrets у Slack
- ❌ Email-attached secrets

---

## 7 · Database operations

### Schema migrations

- Backwards-compatible only (additive, нікого не break)
- Migrations checked into version control
- Run via Supabase migrations CLI
- Test у staging first ALWAYS

### Data backups

- Supabase automatic backups (point-in-time recovery 30 days)
- Weekly full export to R2 (cold storage)
- Monthly verification of backup integrity

### Data destruction (для GDPR delete)

```
T+0 User clicks Delete Account
T+1 min Mark as deleted у DB (soft-delete with timestamp)
T+30 days Hard delete:
  - User row removed
  - All foreign-key associated rows removed
  - Anonymize у audit log (keep action, lose user-id)
T+45 days Backup containing user data expires (rolling)
```

---

## 8 · Performance

### Targets

| Metric | Target |
|---|---|
| App cold start | < 1.5s |
| API P95 | < 1.5s |
| CV inference per frame | < 80ms |
| Battery drain (workout mode) | < 30% / hour |
| Memory peak | < 250 MB |

### Optimization process

1. Measure first
2. Identify P95 outliers
3. Profile (Instruments для iOS, Cloudflare Analytics для API)
4. Fix
5. Re-measure
6. Repeat

Don't optimize prematurely. Don't ship unmeasured «improvements».

---

## 9 · Feature flags

We use feature flags для:
- Gradual rollouts (1% → 10% → 50% → 100%)
- A/B tests
- Kill-switches for problematic features

Tool: PostHog feature flags (already integrated)

Flag lifecycle:
1. Created з expiration date
2. Documented у Decision Log
3. Cleaned up post-experiment (30-day decay)

**Don't accumulate flags.** Flag-debt = tech-debt.

---

## 10 · Tech debt

### Definition (для context)

Technical debt = code/design choices that work now but will slow us later. Sometimes worth it, sometimes mistakes.

### Tracking

Use Linear label `tech-debt` для:
- Hacks that need cleanup
- Performance bottlenecks deferred
- Test coverage gaps known
- Library upgrades pending

### Repayment

- 20% of each sprint reserved for tech debt
- Quarterly reviews: top-3 debt items addressed
- «No new feature без paying down 1 debt item» rule

---

## 11 · Documentation

### What gets documented

- Architecture decisions (з context, in `docs/`)
- Public API (auto-generated)
- Setup guides (для new engineers)
- Runbooks (this file, incident-specific)
- Decision log

### What doesn't

- Code-level comments (good naming > comments)
- Tutorials covered by external resources
- README-bloat repetition

### Format

- Markdown у GitHub
- Headers logical (H1 = doc, H2 = section)
- Examples > prose where possible
- Update timestamp at top

---

## 12 · Hiring engineering process

(Pulled з `docs/HIRING.md`)

For engineering roles:
1. Resume + GitHub review
2. 30-min intro call
3. Take-home (~8 hours, $500 compensated)
4. Technical session (90 min)
5. Cultural fit (60 min)
6. References (3 mandatory)
7. Offer

Don't lower bar for «we need to hire someone». Better to wait.

---

## 13 · Open engineering questions

1. **Frame processing: 30fps vs 15fps?**
   - Need to test battery impact on real devices
   - Currently 30fps default, considering adaptive

2. **Where to host CV models — bundle or server-fetched?**
   - Bundle: bigger app size, instant inference
   - Server-fetch: smaller app, network dependency
   - Currently bundle

3. **Cohort experimentation tooling?**
   - PostHog has built-in
   - Consider Statsig для better A/B test rigor

4. **Apple Watch app — when?**
   - Phase 2 priority (M4-M6)
   - Want to validate hand-held first

5. **Multi-tenancy для B2B pilot?**
   - Not yet — single-tenant per user
   - B2B comes post-PMF

---

**v0.1 · травень 2026 · `platform-engineer`**
**Update with each significant operational learning.**
