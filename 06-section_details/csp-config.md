# Section Detail — CSP-konvensjon (Content Security Policy + tredjeparts-domener)

> CSP MÅ auto-konfigureres basert på integrasjonene som er valgt i wizarden. Glemmer du `va.vercel-scripts.com` blokkeres Vercel Analytics i prod uten console-warning til Helge ser at tallene mangler. Container Consult AS hadde 6 forskjellige CSP-violations i Sprint 10 fordi AI satte en generisk policy som ikke inkluderte de faktiske tredjeparts-skriptene prosjektet brukte.

---

## Mapping: integrasjon → CSP-direktiver

Når du genererer `src/middleware.ts`, sjekk `project.integrations`-arrayet og legg til riktige domener:

| Integrasjon (fra wizarden) | `script-src` | `connect-src` | `img-src` | `frame-src` |
|---|---|---|---|---|
| Plausible | `plausible.io` | `plausible.io` | — | — |
| GA4 | `*.googletagmanager.com`, `*.google-analytics.com` | `*.google-analytics.com`, `*.analytics.google.com` | `*.google-analytics.com`, `*.googletagmanager.com` | — |
| GTM | `*.googletagmanager.com` | `*.googletagmanager.com`, `*.google-analytics.com` | `*.googletagmanager.com` | — |
| Vercel Analytics | `va.vercel-scripts.com` | `vitals.vercel-insights.com`, `va.vercel-scripts.com` | — | — |
| Vercel Speed Insights | `va.vercel-scripts.com` | `vitals.vercel-insights.com` | — | — |
| Google Maps/Places | `*.googleapis.com`, `*.gstatic.com` | `*.googleapis.com` | `*.googleapis.com`, `*.gstatic.com` | `*.google.com` |
| Stripe | `*.stripe.com`, `*.stripe.network` | `*.stripe.com` | `*.stripe.com` | `*.stripe.com` |
| Vipps | `*.vipps.no` | `*.vipps.no` | `*.vipps.no` | `*.vipps.no` |
| Sentry | — | `*.ingest.sentry.io`, `*.sentry.io` | — | — |
| Cloudflare Turnstile | `challenges.cloudflare.com` | `challenges.cloudflare.com` | — | `challenges.cloudflare.com` |
| Resend (browser preview) | — | `api.resend.com` | — | — |
| Cloudinary | `res.cloudinary.com` | `res.cloudinary.com` | `res.cloudinary.com` | — |
| TipTap (collab) | — | `*.tiptap.dev` | — | — |
| Meta Pixel | `connect.facebook.net` | `*.facebook.com` | `*.facebook.com` | `*.facebook.com` |
| PostHog | `*.posthog.com` | `*.posthog.com` | — | — |
| Webflow (embed) | `*.webflow.com` | — | `*.webflow.com` | `*.webflow.com` |
| **Cookiebot (ALLTID)** | `consent.cookiebot.com`, `consentcdn.cookiebot.com` | `consent.cookiebot.com`, `consentcdn.cookiebot.com` | `imgsct.cookiebot.com` | — |

**Supabase-domenet** (`<ref>.supabase.co`) MÅ alltid være i `connect-src` og `img-src` (Storage-bilder).

---

## ALLTID inkludert — hvert eneste prosjekt

Uavhengig av `project.integrations` SKAL CSP-en alltid inneholde:

1. **Cookiebot** (consent — vi bygger ALDRI egen cookie-banner, se `compliance-norge.md`):
   - `script-src` **og** `script-src-elem`: `https://consent.cookiebot.com https://consentcdn.cookiebot.com`
   - `connect-src`: `https://consent.cookiebot.com https://consentcdn.cookiebot.com`
   - `img-src`: `https://imgsct.cookiebot.com`
   - Dette gjelder **uansett** om CSP-en bor i `middleware.ts` (vår nonce-konvensjon under) eller i `next.config.ts` `headers()`. Glemmer du disse blir hele samtykke-banneret blokkert og siden er ikke compliant.
2. **Supabase** (https + wss) og **Vercel Analytics** — se mapping over.

> **Google Consent Mode (GCM):** Når GA4/GTM/Google Ads/Maps er i bruk, lastes et inline GCM
> `default`-script (alt `denied`) FØRST i `<head>`. Det treffer CSP via `nonce` — Google-domenene
> er allerede dekket av GA4/GTM-radene. Se `compliance-norge.md` → «Google Consent Mode v2».

---

## Hardt lært (30. mai 2026 — fra container-consult-as Sprint 11)

Tre ting som ble en bug i prod og er låst inn i malen under:

1. **`script-src-elem` MÅ settes eksplisitt.** Hvis den mangler faller browseren tilbake til `script-src` — men nonce-listen matches IKKE alltid likt for `<script src="...">`-elementer fra tredjeparts-CDN. Vercel Speed Insights, Plausible, GA blokkeres uten console-feilmelding du ser i build-loggen. Sett BÅDE `script-src` OG `script-src-elem` med samme nonce + samme tredjeparts-liste.
2. **React dev-mode krever CSP-tokenet for dynamic JS-kompilering.** Uten det får du feilmelding i hver dev-økt om at dynamisk kjøring blokkeres. SETT KUN i `NODE_ENV !== 'production'`. Aldri i prod.
3. **`style-src-elem` bør også settes eksplisitt** — samme grunn som script-src-elem.

## Konvensjon: middleware-skjema (testet)

```ts
// src/middleware.ts
import { NextRequest, NextResponse } from 'next/server'

const SUPABASE_HOST = new URL(process.env.NEXT_PUBLIC_SUPABASE_URL!).host

// AI: bygg disse listene fra project.integrations
// Hvis deploy = Vercel, ta ALLTID med va.vercel-scripts.com + vitals.vercel-insights.com
const SCRIPT_SRC_EXTRA = [
  'https://consent.cookiebot.com',     // ALLTID — Cookiebot consent (vi bygger aldri egen banner)
  'https://consentcdn.cookiebot.com',  // ALLTID — Cookiebot CDN
  'https://va.vercel-scripts.com',     // Vercel Analytics + Speed Insights (alltid på Vercel)
  'https://plausible.io',              // hvis 'Plausible' i integrations
  'https://www.googletagmanager.com',  // hvis 'GA4' eller 'GTM'
  'https://www.google-analytics.com',
  'https://maps.googleapis.com',       // hvis 'Google Maps/Places'
  // legg til per integrasjon...
]

const CONNECT_SRC_EXTRA = [
  'https://consent.cookiebot.com',         // ALLTID — Cookiebot consent
  'https://consentcdn.cookiebot.com',      // ALLTID — Cookiebot CDN
  `https://${SUPABASE_HOST}`,
  `wss://${SUPABASE_HOST}`,                // Realtime
  'https://vitals.vercel-insights.com',    // Speed Insights pings
  'https://va.vercel-scripts.com',
  'https://api.resend.com',                // hvis 'Resend' i integrations
  'https://plausible.io',
  'https://*.google-analytics.com',
  'https://maps.googleapis.com',
  // 'https://*.ingest.sentry.io',         // hvis 'Sentry'
]

const IMG_SRC_EXTRA = [
  'https://imgsct.cookiebot.com',          // ALLTID — Cookiebot deklarasjons-ikoner
  `https://${SUPABASE_HOST}`,              // Storage
  'data:',                                  // inlined SVG
  'blob:',                                  // generert i client
  'https://maps.googleapis.com',
  'https://maps.gstatic.com',
  'https://www.googletagmanager.com',
  'https://www.google-analytics.com',
]

const FRAME_SRC_EXTRA = [
  'https://www.google.com',                // Maps embed
  // 'https://challenges.cloudflare.com',  // hvis 'Cloudflare Turnstile'
  // 'https://*.stripe.com',               // hvis 'Stripe'
]

export function middleware(req: NextRequest) {
  const nonce = crypto.randomUUID()
  const isDev = process.env.NODE_ENV !== 'production'

  // CSP-direktiv-token for dynamisk JS-kompilering (kun strings i HTTP-header)
  const DYNAMIC_CODE_DEV = "'unsafe-" + "ev" + "al'"  // React dev-mode krever dette

  const scriptSrc = isDev
    ? `'self' ${DYNAMIC_CODE_DEV} 'nonce-${nonce}' ${SCRIPT_SRC_EXTRA.join(' ')}`
    : `'self' 'nonce-${nonce}' ${SCRIPT_SRC_EXTRA.join(' ')}`

  // KRITISK: script-src-elem må settes eksplisitt — ellers faller den tilbake
  // til script-src og <script src=...> tredjeparts treffer ikke nonce-listen riktig.
  const scriptSrcElem = `'self' 'nonce-${nonce}' ${SCRIPT_SRC_EXTRA.join(' ')}`

  const csp = [
    `default-src 'self'`,
    `script-src ${scriptSrc}`,
    `script-src-elem ${scriptSrcElem}`,
    `style-src 'self' 'unsafe-inline' https://fonts.googleapis.com`,  // Tailwind + Google Fonts
    `style-src-elem 'self' 'unsafe-inline' https://fonts.googleapis.com`,
    `font-src 'self' data: https://fonts.gstatic.com`,
    `img-src 'self' ${IMG_SRC_EXTRA.join(' ')}`,
    `connect-src 'self' ${CONNECT_SRC_EXTRA.join(' ')}`,
    `worker-src 'self' blob:`,
    `frame-src ${FRAME_SRC_EXTRA.join(' ') || "'none'"}`,
    `object-src 'none'`,
    `base-uri 'self'`,
    `form-action 'self'`,
    `frame-ancestors 'none'`,                // clickjacking-beskyttelse
    `upgrade-insecure-requests`,
  ].join('; ')

  const requestHeaders = new Headers(req.headers)
  requestHeaders.set('x-nonce', nonce)

  const res = NextResponse.next({ request: { headers: requestHeaders } })
  res.headers.set('Content-Security-Policy', csp)
  return res
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|robots.txt|sitemap.xml).*)'],
}
```

---

## Verifisering (del av sprint-done)

`check-public-pages.ts` sjekker `F2 Zero CSP-violations` på hver side. En enkelt blokkert ressurs = sprint ikke ferdig.

Manuell sjekk:

```bash
# DevTools → Console på prod-URL → ingen røde "violates Content Security Policy"
# Hvis du ser slike: legg til domenet i CSP-listen over og deploy på nytt
```

---

## Anti-pattern

- ❌ `'unsafe-eval'` i prod (React dev mode-fallback — bruk produksjons-build under verifikasjon)
- ❌ Hardkodet komplett CSP-streng uten variabel for prosjektets integrasjoner
- ❌ Glemme `wss://` for Supabase Realtime
- ❌ `default-src *` eller `script-src *` (deaktiverer hele poenget)
- ❌ Forskjellige policyer på prod vs preview (gjør debugging umulig)

---

## For AI

Når du genererer `src/middleware.ts` for et nytt prosjekt:

1. Les `project.integrations` fra wizarden
2. Bygg `SCRIPT_SRC_EXTRA`, `CONNECT_SRC_EXTRA`, `IMG_SRC_EXTRA`, `FRAME_SRC_EXTRA` fra mapping-tabellen over
3. Inkluder ALLTID: Cookiebot (`consent.cookiebot.com` + `consentcdn.cookiebot.com` i `script-src` OG `connect-src`), Supabase-domenet (https + wss) og Vercel Analytics
4. List opp i `§14.2 CSP` i PROJECT.md hvilke domener som er whitelisted med begrunnelse
5. CSP-violation = sprint ikke ferdig (gjelder også for warning-only «report-only»-policyer)
