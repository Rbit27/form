# FORM · Enterprise Tier v0.1

> Як FORM продається у компанії і чим він відрізняється від consumer-tier.
> Не активується до того, як consumer-tier має PMF (D90 ≥ 65% retention, NPS ≥ 50, ARR ≥ $500k).

---

## TL;DR

Enterprise FORM — це **same product, different operating model**:

- Multi-tenant SaaS з SSO/SCIM
- Admin dashboard для HR / People leads
- Aggregated wellness metrics (privacy-first)
- White-label опціонально
- SOC 2 / GDPR compliant
- Annual contracts $50k–$500k
- Dedicated CSM від customer-success agent

---

## Why enterprise

### Market
- 50k+ companies у US/EU з wellness budget $50–500/employee/year
- Engineering / product / tech-forward orgs = our sweet spot
- Average deal size for similar SaaS wellness: $100k+ ARR

### Business case
- Higher LTV (annual contracts, lower churn)
- Better margins (no Apple/Google fees)
- Reference customers help consumer brand
- Different acquisition cost (sales-led vs marketing-led)

### Strategic
- Distribution at scale (one contract = 1000 users)
- Insulates from consumer fitness fad cycles
- Investors love B2B revenue mix

---

## Org structure

### Existing team (consumer-tier)
- product-strategist, research-lead, marketing-lead, growth-lead
- sports-scientist, nutrition-coach, clinical-safety
- platform-engineer, design-craft, brand-voice, brand-system, ux-flow
- content-strategist, process-keeper

### NEW for enterprise
- **enterprise-architect** — multi-tenancy, SSO, RBAC, white-label
- **compliance-officer** — SOC 2, HIPAA, GDPR, ISO 27001
- **devops-lead** — CI/CD, IaC, observability, cost ops
- **data-engineer** — analytics warehouse, ETL, BI dashboards
- **qa-lead** — test automation, performance, accessibility
- **security-engineer** — threat modeling, pen-testing, IR
- **ml-engineer** — CV pipeline, MLOps, model lifecycle
- **customer-success** — onboarding, expansion, retention
- **product-manager** — daily delivery coordination

**Total team:** 23 specialized agents + founder.

---

## Enterprise stack additions

### Architecture
- Multi-tenant Postgres з Row-Level Security (RLS)
- Tenant-scoped feature flags
- Cross-tenant test in CI (block bad migrations)
- Audit log per tenant (tamper-evident)

### Identity
- SAML 2.0 SSO (Okta, Azure AD, Google Workspace, OneLogin)
- OIDC for modern providers
- SCIM 2.0 for user provisioning
- MFA enforcement at tenant level
- Session policies (max duration, IP whitelist)

### Admin Dashboard (NEW product surface)
- Web app, React + TS, hosted on Cloudflare Pages
- 3 personas: tenant_owner, tenant_admin, tenant_manager
- Aggregate-only health metrics (no individual PII)
- User provisioning (manual + bulk CSV + SCIM)
- Billing self-serve via Stripe Customer Portal
- Audit log viewer

### Compliance
- SOC 2 Type II within first 12 months of enterprise launch
- DPIA для health data processing
- DPAs signed з усіма processors
- SCC для US-EU transfers
- Penetration tests annually
- Continuous compliance via Vanta/Drata

### Branding
- White-label tier above $50k ARR
- Custom domain (CNAME → form.coach)
- Tenant logo replaces FORM
- Color override з accessibility floor
- "Powered by FORM" footer non-removable < $50k ARR

---

## Pricing

| Tier | Seats | Price | Notes |
|---|---|---|---|
| **Starter** | 50–200 | $12/seat/mo (annual) | Self-serve, no CSM |
| **Growth** | 200–1,000 | $9/seat/mo | Dedicated CSM, QBR quarterly |
| **Enterprise** | 1,000+ | $6–8/seat/mo (negotiated) | Multi-year, white-label option, custom SLA |

Discounts:
- 15% for 2-year contract
- 25% for 3-year contract
- 10% for paid annual upfront

**Billing currency:** All enterprise contracts are denominated and invoiced in **USD** (DEC-096). For Ukraine-based customers, a non-binding UAH reference price is included in the Order Form Addendum for internal budgeting purposes; the binding payment obligation is the USD amount. A UAH conversion clause with a ±10% floor is available upon written request and requires a counter-signed amendment (MSA §4.7). EU contracts (Poland M12, Germany/Netherlands M18–M20) are also invoiced in USD at GA; EUR billing is evaluated at Series A based on EU entity decision (OQ-GEO-03, counsel M8).

---

## Sales process

| Phase | Duration | Owner | Deliverables |
|---|---|---|---|
| Discovery | Week 0–2 | marketing-lead → customer-success | Discovery notes, ICP fit |
| Demo | Week 3–4 | customer-success + product-manager | Custom demo, ROI projection |
| Pilot | Month 2–3 | customer-success | 90-day free pilot з success criteria |
| Contract | Month 4–5 | founder + outside counsel | MSA, DPA, signed contract |
| Implementation | Month 5–6 | customer-success + devops-lead | SSO setup, branding, training |
| Adoption | Month 6+ | customer-success | QBR cycle, expansion |

Average sales cycle: **5–7 months** for first enterprise deal. Target: 2 deals in Year 1, 12 in Year 2.

---

## What we promise customers

### Hard commitments
- 99.9% uptime SLA (with credits)
- Response time SLAs (P0 < 1h, P1 < 4h, P2 < 24h)
- Data residency (EU customers stay in EU)
- DPA signed before any pilot data flows
- Annual penetration test summary shared
- Quarterly compliance attestation

### Soft commitments
- Adoption rate ≥ 30% within 60 days
- Average engagement ≥ 50% weekly active among activated
- Aggregate wellness improvements measurable

### What we DON'T promise
- Specific health outcomes (weight, BP, etc. — too many variables)
- Individual user data access for employer (privacy floor)
- White-label below $50k ARR
- Custom features for single customer (only at $500k+ ARR)

---

## Privacy floor for enterprise

These are **non-negotiable**, regardless of customer demands:

1. HR can never see individual user data
2. HR sees aggregates only (% active, % activated, opt-in NPS)
3. Manager-reports on direct reports never available
4. Employee can revoke company-association anytime
5. ED-screening data never aggregated to employer
6. Body composition data never aggregated to employer
7. Mental health data never aggregated to employer

These are clinical-safety + compliance-officer floors. Customer cannot override.

---

## When we say no

Enterprise opportunities we decline:
- **Insurance company asking for risk-scoring data on employees** → declined
- **Government contract requiring backdoor or data access** → declined
- **Wellness program tied to insurance pricing** → declined
- **Sports league wanting performance data for evaluation** → declined
- **Wellness budget that punishes non-participants** → declined

These create legal/ethical risk and brand damage.

---

## Implementation timeline (per customer)

```
Day 0       Contract signed
Day 1–7     Kickoff calls (customer success + IT + HR)
Day 7–14    SSO/SCIM configuration in test environment
Day 14–21   Branding setup, custom domain
Day 21–28   Pilot user list, internal comms drafted
Day 28      Soft launch to pilot cohort (50–100 users)
Day 28–60   Pilot phase, weekly check-ins
Day 60      Full rollout to all seats
Day 60–90   First-month adoption report
Day 90      30/60/90 review
```

---

## Open questions для enterprise tier

1. **Custom mobile builds per tenant or single app з branding?**
   - Likely single app with dynamic branding by tenant ID at login
   - Custom build only for top-tier ($250k+ ARR)

2. **Дані residency для EU customers — Frankfurt vs Dublin?**
   - Frankfurt (Cloudflare EU-Central) for German customers
   - Dublin (Cloudflare EU-West) для UK/IE customers
   - Multi-region post-Series A

3. **Support tier differentiation?**
   - Starter: email only, 24h response
   - Growth: Slack Connect, 4h response
   - Enterprise: dedicated phone + Slack, 1h response

4. **Когда наймати першого Account Executive?**
   - Після 1st enterprise close (proves we can sell)
   - Hire pattern: founder closes 1–3, then AE for the rest

---

## Risks

1. **CV-quality not enterprise-ready** — corporate users may have worse environments (private offices, weird lighting). Mitigation: feature flag CV off if accuracy poor.

2. **Sales cycle too long** — risk of cash crunch. Mitigation: keep consumer revenue strong, don't bet runway on enterprise alone.

3. **Compliance audit drags** — SOC 2 first audit can stretch to 9 months. Mitigation: engage audit firm at month 3, not month 9.

4. **Customer asks to bypass safety floor** — pressure to remove ED-screening or share data. Mitigation: clinical-safety has VETO. Lose deal if necessary.

---

**v0.1 · травень 2026 · enterprise-architect + customer-success + product-strategist**
