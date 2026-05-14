# FORM · Deployment Runbook v0.1

> Як deploy marketing site на public internet.
> Власник: founder (поки solo). Updates with each platform decision.

---

## Поточний стан

Marketing site — це **static HTML/CSS/JS файли** у repo. Жодного build process. Будь-який static hoster це працює.

### Файли для deploy

```
index.html
about.html
beta.html
pricing.html
faq.html
settings.html
screens.html (cloud-added)
onboarding.html (cloud-added)
workout-live.html
```

Жодного backend поки що. Backend починається тільки коли founding engineer hire'd.

---

## Recommended host: Cloudflare Pages

### Чому Cloudflare

- Free tier generous (500 deployments/mo, 100 builds/mo)
- Global CDN включений
- Custom domain (form.coach) trivial
- Edge runtime для майбутнього (Workers compatibility)
- Жодних cold starts
- Same infrastructure as our future API (Workers)

### Альтернативи (теж okay):
- **Vercel** — найшвидший deploy, але vendor lock-in більший
- **Netlify** — solid, але CDN не такий fast як Cloudflare
- **GitHub Pages** — найдешевший, але немає preview deployments

---

## Step-by-step: First deploy

### 1. Buy domain (10 хв)

- **form.coach** через CloudFlare Registrar
- Цена ~$30/year для .coach TLD
- Or via Namecheap if simpler

### 2. Create Cloudflare account (5 хв)

- Sign up на cloudflare.com
- Verify email
- Enable 2FA з hardware key

### 3. Create Pages project (5 хв)

```
Cloudflare Dashboard → Pages → Create project
→ Connect to Git
→ Choose GitHub
→ Authorize Cloudflare access
→ Select repository: Rbit27/form
→ Select branch: main
→ Production branch: main
→ Build settings:
   - Build command: (leave blank)
   - Build output directory: /
   - Environment variables: (none yet)
→ Save and Deploy
```

First deploy: 1-2 minutes.

### 4. Connect custom domain (5 хв)

```
Pages project → Custom domains → Set up a custom domain
→ form.coach
→ Cloudflare auto-configures DNS
→ SSL cert generates automatically
```

Within 5-10 minutes, https://form.coach live.

### 5. Verify

- Open form.coach у browser
- Click through all 9 pages
- Check на mobile
- Verify no broken links
- Check SSL cert valid
- Check robots.txt accessible (we'll create)

---

## Pre-deployment checklist

Before pushing to live:

### Content
- [ ] All links between pages tested
- [ ] All external links open у new tab
- [ ] No placeholder text remaining
- [ ] Email addresses use real domain (hello@form.coach)
- [ ] All copy reviewed з brand-voice agent

### SEO
- [ ] Each page has `<title>` and `<meta description>`
- [ ] OG tags для social sharing
- [ ] Sitemap.xml created (next step)
- [ ] robots.txt created

### Performance
- [ ] All fonts loaded efficiently (preconnect)
- [ ] Images optimized (none yet, but for future)
- [ ] Tailwind CSS не bloated (we use CDN, fine for static)
- [ ] No console errors у production build

### Privacy / Compliance
- [ ] Cookie banner для EU visitors
- [ ] Privacy Policy linked у footer
- [ ] ToS linked у footer
- [ ] Email subscription gets explicit opt-in (when implemented)

### Analytics
- [ ] Cloudflare Analytics enabled (privacy-friendly, free)
- [ ] Optional: PostHog tracking (з opt-out cookie banner)
- [ ] NO Google Analytics (privacy concerns)

---

## Sitemap.xml (потрібно створити)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://form.coach/</loc>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://form.coach/about.html</loc>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
  <url>
    <loc>https://form.coach/pricing.html</loc>
    <changefreq>monthly</changefreq>
    <priority>0.9</priority>
  </url>
  <url>
    <loc>https://form.coach/beta.html</loc>
    <changefreq>weekly</changefreq>
    <priority>0.9</priority>
  </url>
  <url>
    <loc>https://form.coach/faq.html</loc>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>
  <url>
    <loc>https://form.coach/workout-live.html</loc>
    <changefreq>monthly</changefreq>
    <priority>0.6</priority>
  </url>
  <url>
    <loc>https://form.coach/screens.html</loc>
    <changefreq>monthly</changefreq>
    <priority>0.6</priority>
  </url>
  <url>
    <loc>https://form.coach/settings.html</loc>
    <changefreq>monthly</changefreq>
    <priority>0.4</priority>
  </url>
  <url>
    <loc>https://form.coach/onboarding.html</loc>
    <changefreq>monthly</changefreq>
    <priority>0.5</priority>
  </url>
</urlset>
```

---

## robots.txt

```
User-agent: *
Allow: /

Sitemap: https://form.coach/sitemap.xml

# Block AI scrapers from training on our content
User-agent: GPTBot
Disallow: /

User-agent: ClaudeBot
Disallow: /

User-agent: anthropic-ai
Disallow: /

User-agent: CCBot
Disallow: /
```

---

## Email infrastructure

### Custom domain emails

Setup через Cloudflare Email Routing (free):

```
hello@form.coach     → personal-gmail@gmail.com
hiring@form.coach    → personal-gmail@gmail.com
press@form.coach     → personal-gmail@gmail.com
invest@form.coach    → personal-gmail@gmail.com
privacy@form.coach   → personal-gmail@gmail.com
advisor@form.coach   → personal-gmail@gmail.com
beta@form.coach      → personal-gmail@gmail.com
support@form.coach   → personal-gmail@gmail.com
```

Outbound через Gmail з alias send-as setup.

### Newsletter

Pre-launch: Beehiiv або Substack (free tiers acceptable)
Post-launch: ConvertKit чи Loops.so

---

## Continuous deployment

### Branch strategy

```
main          → production form.coach
preview/*     → preview deploys at preview-X.pages.dev
hotfix/*      → expedited preview + manual promote
```

### Auto-deploy

Every push to `main`:
1. Cloudflare Pages detects
2. Builds (in our case = просто copy files)
3. Deploys globally у ~60 seconds
4. Old version cached for 24h (rollback window)

### Rollback

```
Cloudflare Dashboard → Pages → form-marketing
→ View Deployment History
→ Click previous successful deployment
→ Rollback to this deployment
```

~30 seconds to revert.

---

## Monitoring (post-deploy)

### Cloudflare Analytics (free)

- Daily visitors
- Geographic breakdown
- Top pages
- Bandwidth

### PostHog (optional, opt-in)

- Funnel: index → pricing → beta-signup
- Time-on-page per article
- Scroll depth
- Click maps

### Uptime monitoring

UptimeRobot (free) — pings form.coach every 5 min.
Alerts on 2+ consecutive failures.

---

## Cost projection

| Service | Cost |
|---|---|
| Domain (form.coach .coach TLD) | $30/year |
| Cloudflare Pages | $0 (free tier sufficient) |
| Cloudflare Email Routing | $0 |
| UptimeRobot | $0 (free tier) |
| Cloudflare Analytics | $0 |
| PostHog (optional, post-1k users) | $0 (free tier до 1k MAU) |
| **Total через рік 1** | **~$30** |

Це cost для marketing-сайту. Mobile app — окремий budget (Apple Developer $99/year + infra).

---

## Deploy timeline

| Step | Time |
|---|---|
| Buy domain | 10 хв |
| Create Cloudflare account | 5 хв |
| Create Pages project | 5 хв |
| Configure DNS | 5 хв |
| Test live site | 30 хв |
| Setup email routing | 15 хв |
| Setup sitemap + robots.txt | 15 хв |
| **Total time** | **~85 хв** |

One-time effort. After this — push to main = auto-deploy.

---

## Open deployment questions

1. **Cloudflare Workers vs Pages для future API?**
   - Pages primary now, Workers для future backend
   - Same platform, easy migration

2. **CDN strategy для images?**
   - Cloudflare Images $5/mo if we add many assets
   - Або just store у /assets/ folder
   - Decide post-launch based on volume

3. **Pre-rendering blog posts?**
   - Currently blog posts MD files only
   - Consider Astro or similar для proper blog (post-launch)
   - For now, static HTML enough

4. **Internationalization deploy strategy?**
   - Subdomain (uk.form.coach) чи path (form.coach/uk/)?
   - Path simpler, subdomain better SEO
   - Decide closer to phase 1 i18n

---

## Anti-patterns

- ❌ Custom server VPS («I want full control»)
- ❌ Build-step для static site (over-engineering)
- ❌ Google Analytics (privacy concerns)
- ❌ Auto-deploy from feature branches to production
- ❌ Skipping HTTPS

---

**v0.1 · травень 2026 · founder**
**One-time setup. Auto-deploy після.**
