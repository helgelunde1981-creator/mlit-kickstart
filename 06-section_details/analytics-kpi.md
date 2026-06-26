# Section Detail — Analytics + KPI + Post-launch måling

> Brukes når AI fyller ut seksjon 17 (Analytics + KPI) i PROJECT.md.

---

## Minimum required output

Seksjon 17 må inneholde:

1. **Analytics-stack** (hvilke verktøy)
2. **Måleplan** — per side / feature, hva måler vi
3. **KPI-er** — konkrete tall, ikke vagt
4. **Dashboards** — hvor ser kunden tallene
5. **Rapportering** — kadens og format
6. **GDPR-håndtering** — consent-bundet, ingen tracking før samtykke

## Template

```markdown
## 17. Analytics + KPI + post-launch måling

### 17.1 Analytics-stack

| Verktøy | Brukes til | Consent-krav |
|---------|------------|--------------|
| **Vercel Analytics** | Page views, top pages, geo, devices | Cookie-fritt (ingen consent) |
| **Vercel Speed Insights** | RUM for Core Web Vitals | Cookie-fritt |
| **PostHog** | Produktanalyse, funnels, session replay | Krever consent-banner accept |
| **Google Search Console** | SEO-rankings, klikk-rate, indexing | Eier-only |

### 17.2 Måleplan per side

| Side / feature | Hva måles | Hvorfor |
|----------------|-----------|---------|
| Forside | Page views, scroll-depth, kalkulator-engagement, CTA-klikk | Hovedfunnel-inngang |
| Signature moment | Interaksjons-rate, completion-rate, abandonment-step | Avgjørende for produkt-suksess |
| Pris-side | Time-on-page, CTA-klikk-rate | Konvertering |
| Bestillingsskjema | Form-step-completion, drop-off, submission-rate | Critical funnel |
| Blogg | Top posts, scroll-depth, recirculation | Content-strategi |
| Kontakt | Submission-rate | Lead-generering |

### 17.3 Konkrete KPI-er

**Lansering (måned 1):**

- [ ] Lighthouse Performance ≥ 95 vedvarende
- [ ] LCP < 2.5s p75
- [ ] INP < 200ms p75
- [ ] CLS < 0.1 p75
- [ ] Forside bounce-rate < 50%
- [ ] Konvertering forside → CTA-klikk ≥ X% (baselinet etter uke 1)

**Måned 3:**

- [ ] Organic search-trafikk vokser måned-over-måned
- [ ] Top 10 i Google for [3-5 nøkkelfraser]
- [ ] Konvertering CTA → kontakt/bestilling ≥ Y%
- [ ] Email open-rate ≥ 25% (på transactional)

**Måned 6:**

- [ ] [Domene-spesifikke mål]

### 17.4 Dashboards

| Dashboard | Hvor | Hvem ser |
|-----------|------|---------|
| Vercel Analytics | vercel.com/[team]/[prosjekt]/analytics | Myrvoll-Lunde IT Drift-team |
| Speed Insights | vercel.com/[team]/[prosjekt]/speed-insights | Myrvoll-Lunde IT Drift-team |
| PostHog | app.posthog.com | Myrvoll-Lunde IT Drift + kunde |
| Search Console | search.google.com/search-console | Myrvoll-Lunde IT Drift + kunde |
| Custom KPI-dashboard | /admin/insights (hvis bygget) | Kunde |

### 17.5 Rapportering

**Ukentlig (interne, automatisert):**
- Sentry error-rate
- Vercel deploy-status
- Core Web Vitals-trend

**Månedlig (til kunde, manuell sammensatt):**
- Email-rapport med:
  - Topp 5 sider (views, time-on-page)
  - Konverteringstrend
  - Søkeordsbevegelser
  - Performance-snapshot
  - Anbefalte tiltak for neste måned
- Sendt 1. virkedag i måneden via Resend-mal

**Kvartalsvis:**
- Møte med kunde (digitalt eller fysisk)
- Roadmap-justering basert på data

### 17.6 GDPR-håndtering

- Vercel Analytics: cookie-fritt, ingen consent nødvendig (cookies brukes ikke)
- Speed Insights: cookie-fritt
- **PostHog: lastes kun etter Cookiebot-samtykke** til "statistics"-kategori (vi bygger ALDRI egen banner)
- **Google Analytics: brukes ikke** (overlapper med PostHog, krever cookies, dårligere consent-UX)

Consent = Cookiebot. Last trackere mot Cookiebots consent-state (aldri egen banner):

\`\`\`tsx
// Last ikke PostHog før Cookiebot melder samtykke for statistics
useEffect(() => {
  const onConsent = () => {
    if (window.Cookiebot?.consent.statistics) {
      posthog.init(POSTHOG_KEY, { ... })
    }
  }
  window.addEventListener('CookiebotOnConsentReady', onConsent)
  return () => window.removeEventListener('CookiebotOnConsentReady', onConsent)
}, [])
\`\`\`

Samtykke-loggen ligger hos Cookiebot — vi har ingen egen `cookie_consents`-tabell.
```

## Vanlige fallgruver

| Problem | Fix |
|---------|-----|
| Tracker lastet før consent | Bygg lazy init med `useEffect` lyttende på consent |
| Glemmer å re-initialize tracker når consent oppdateres | Lytt på consent-endringer, re-init |
| Måler "page view" men ikke "meaningful engagement" | Bruk scroll-depth, time-on-page, custom events |
| Rapporterer Vanity-tall til kunde (raw views) | Fokuser på konvertering og trend |
| Mangler baseline før lansering | Mål første uke som baseline, sett mål deretter |
