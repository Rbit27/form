# FORM · Investor Data Room v0.1

> Map of materials available to share з investors. Tiered by trust level.
> Власник: founder.

---

## Tier 0 — Public

**Anyone can access. Linked from website / shared freely.**

| Document | Status | Location |
|---|---|---|
| Marketing site | live | [index.html](../index.html) |
| Manifesto / About | live | [about.html](../about.html) |
| Pricing | live | [pricing.html](../pricing.html) |
| Blog posts (4 published) | live | `content/post-*.md` |
| Privacy Policy (draft) | draft | `legal/privacy-policy-draft.md` |
| Terms of Service (draft) | draft | `legal/terms-of-service-draft.md` |
| App Store listing draft | working | `docs/APP_STORE.md` |
| Open-source agents | live | `.claude/agents/*.md` |

---

## Tier 1 — Light NDA (для cold-introduced investors)

**Shared after one-pager intro accepted. Light NDA via DocuSign.**

| Document | Status | Updated |
|---|---|---|
| 12-slide investor pitch (PDF) | rendered from `docs/PITCH.md` | weekly |
| One-pager (PDF) | rendered from `docs/PITCH.md` § One-pager | weekly |
| Founder bio + LinkedIn | `docs/PRESS.md` § Founder | as needed |
| Competitive teardown | `docs/COMPETITIVE.md` | monthly |
| Technical architecture overview | `docs/TECHNICAL.md` (high-level only) | quarterly |
| Brand book preview | `docs/BRAND.md` excerpt | as needed |

---

## Tier 2 — Strong NDA (для serious due-diligence)

**Shared after first founder call + signed strong NDA.**

| Document | Status | Format |
|---|---|---|
| Full 3-year financial model | `docs/FINANCIALS.md` | Excel + PDF |
| Detailed unit economics | `docs/FINANCIALS.md` § 1-2 | Excel breakdown |
| User research synthesis | `docs/RESEARCH.md` + synthesis when ready | PDF |
| GTM strategy | `docs/MARKETING.md` | PDF |
| Hiring plan + JDs | `docs/HIRING.md` | PDF |
| 3-year team structure | extended from `docs/HIRING.md` | Slide |
| Sport-science advisory list | who's signed | List |
| Roadmap + milestones | extracted from OKRs | Roadmap doc |
| API design overview | `docs/API.md` high-level | PDF |
| Security architecture | `docs/SECURITY.md` | PDF |

---

## Tier 3 — Closing-stage (для term sheet under discussion)

**Shared during diligence after verbal interest.**

| Document | Status | Notes |
|---|---|---|
| Cap table current + post-money scenarios | Build in Carta / equivalent | Confidential |
| Reference list (founder) | 3-5 contacts | Confidential |
| Customer reference list | beta cohort with consent | Confidential |
| Detailed metrics dashboard | from product analytics | live access читання |
| Cohort retention curves | full data | Excel |
| Vendor contracts (Anthropic, ElevenLabs, etc.) | upon request | PDF |
| Existing employment agreements | upon request | PDF |
| IP assignments | upon request | PDF |
| Bank statements / financial controls | upon request | PDF |
| Insurance certificates | upon request | PDF |

---

## Tier 4 — Personal (founder-only)

**Never shared. Internal context only.**

| Document |
|---|
| [`docs/FOUNDER.md`](FOUNDER.md) — founder narrative, red lines, anti-motivators |
| Personal financial planning |
| Burn rate personal contingency |
| Health and wellness journal |
| Therapy / mental-health notes (private) |

---

## Process

### Initial inquiry from investor

1. **Email response** within 24 hours
2. **Share Tier 0** — link to marketing site
3. **15-min screening call** if interested
4. **If yes** → send Tier 1 materials + NDA

### After first deep call

1. **Light NDA signed** → Tier 1 + 2 materials
2. **30-min follow-up call** scheduled within 7 days
3. **Reference exchange** (their portfolio companies, our advisors)

### Diligence stage

1. **Strong NDA signed** → Tier 2 access
2. **Founder + investor work session** (2-3h)
3. **Q&A list received** → batch response within 48h
4. **Tier 3 materials prepared** for term-sheet conversation

### Term sheet → close

1. **Term sheet review** by founder counsel (essential, не оптіонал)
2. **Negotiation** — typically 1-2 weeks
3. **Definitive agreements** drafted by counsel
4. **Closing** target within 30 days post-term sheet

---

## Anti-patterns

- ❌ Share Tier 2+ materials before founder call complete
- ❌ Send confidential metrics через unsecured email
- ❌ Allow investor to share materials with their team без consent
- ❌ Provide reference contacts before they've been briefed
- ❌ Talk numbers без context (always include cohort definition, time period)
- ❌ Use round-trip cohort metrics (вибрати best month) — always be honest about variance

---

## Documents to prepare (status checklist)

### Ready (як of v0.3.8)
- [x] Marketing site
- [x] Manifesto (about.html)
- [x] 12-slide pitch (markdown, потрібно PDF render)
- [x] Founder bio (3 lengths)
- [x] Competitive analysis
- [x] Technical architecture
- [x] Brand book
- [x] User personas (research-validated TBD)
- [x] GTM strategy
- [x] Hiring plan
- [x] Financial model (markdown, потрібно Excel render)
- [x] Privacy + ToS drafts
- [x] Security architecture
- [x] OKRs Q3-Q4 2026

### To prepare (before serious diligence)
- [ ] Pitch deck rendered to PDF (Figma або Keynote)
- [ ] One-pager rendered to PDF
- [ ] Financial model в Excel format
- [ ] Cap table в Carta
- [ ] References briefed
- [ ] Reference customers identified (post-beta)
- [ ] Demo video (2-min screen capture)
- [ ] Annual report draft (Q4 2026 retrospective)

### To prepare (post-Seed close)
- [ ] Board materials template
- [ ] Quarterly board reports
- [ ] Annual investor letter
- [ ] Portfolio updates (post-Series A)

---

## Sharing tools

| Tool | Use case |
|---|---|
| **DocSend** | Sharing pitch deck з analytics (who opened, time spent) |
| **Notion** | Diligence portal з selective access |
| **Carta** | Cap table source of truth |
| **Loom** | Founder intro / demo videos |
| **Calendly** | Meeting scheduling |
| **DocuSign** | NDAs, term sheets, employment offers |

---

## Privacy of investor relationships

- We don't disclose які funds are in conversation publicly
- We don't disclose other investors' terms у negotiation
- We respect investor confidentiality unless they ask publicity
- Once closed — investor list може стати public (з consent)

---

## Bias guards (founder, не investor)

- Don't ego-share Tier 2+ з anyone who's «just curious»
- Don't share confidential metrics на masse — це leaks
- Don't pitch fund whose thesis clearly doesn't fit (wastes everyone's time)
- Don't accept money з incompatible partners just because the cheque is offered

---

**v0.1 · травень 2026 · оновлюється з кожним etapом**
