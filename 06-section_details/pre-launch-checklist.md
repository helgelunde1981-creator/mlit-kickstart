# Section Detail — Pre-launch sjekk (sikkerhet + readiness)

> **Aldri lanser uten å fullføre denne sjekklisten.** Hvert prosjekt skal ha en dedikert «Pre-launch»-sprint (siste sprint før produksjons-go-live) hvor disse punktene gjennomgås, dokumenteres og signeres av minst to personer (utvikler + Helge / kunde).

---

## Hvordan denne kjøres

1. **Når:** Siste sprint før lansering, etter alle features er ferdig
2. **Hvem:** Utvikler kjører gjennom, Helge verifiserer
3. **Verktøy:** Sjekklisten + automatiserte tester + manuell verifisering
4. **Output:** Pre-launch-rapport committed til repo som `docs/launches/YYYYMMDD-launch-report.md`
5. **Sign-off:** Minst 2 navn + dato. Lanser ALDRI uten dette.

---

## 1. Sikkerhet

### 1.1 Headers

Test med https://securityheaders.com — mål: **A+**

- [ ] **Content-Security-Policy** med nonce (ikke `'unsafe-inline'` på `script-src`)
- [ ] **Strict-Transport-Security** med `max-age=63072000; includeSubDomains; preload`
- [ ] **X-Content-Type-Options: nosniff**
- [ ] **X-Frame-Options: DENY** (eller `SAMEORIGIN` med dokumentert grunn)
- [ ] **Referrer-Policy: strict-origin-when-cross-origin**
- [ ] **Permissions-Policy:** camera, mic, geo locked ned med mindre nødvendig
- [ ] **Cross-Origin-Opener-Policy: same-origin**
- [ ] **X-XSS-Protection** ikke satt (deprecated, kan skape false positives)

### 1.2 Supabase RLS

- [ ] RLS aktivert på **alle** public-tabeller (`select rls.enabled` på hver)
- [ ] **Per tabell:** policy for SELECT, INSERT, UPDATE, DELETE (eksplisitt — ingen wildcard)
- [ ] **Anon-rolle test:** prøv å skrive til hver tabell som anon — skal feile
- [ ] **Auth-rolle test:** prøv å lese/skrive andre brukeres data — skal feile
- [ ] **Service-role aldri eksponert i klient:** `grep -r "SERVICE_ROLE" src/` skal kun matche server-side filer
- [ ] **RLS-policyer dokumentert** i kommentar i SQL-fila

### 1.3 Secrets

- [ ] `.env.local` i `.gitignore` (verifiser: `git check-ignore .env.local`)
- [ ] **Ingen secrets i git history:** kjør `gitleaks detect --source . -v` eller `trufflehog filesystem .`
- [ ] **Alle secrets i Vercel env vars** (Production / Preview / Development)
- [ ] **Alle secrets i GitHub Actions secrets** (`gh secret list`)
- [ ] **Ingen secrets i logs** (verifiser Sentry / Vercel logs for siste 7 dager)
- [ ] **Rotert nøkler de siste 90 dager** (Supabase service role, Resend, Anthropic, etc.)
- [ ] **Sjekkliste for «secrets eksponert»** — hvis ja, roter umiddelbart

### 1.4 Input-validering

- [ ] **Alle Server Actions** har Zod-validering på input
- [ ] **Alle API routes** har Zod-validering
- [ ] **Sanitization** på rich-text (DOMPurify eller TipTap-renderer som ikke aksepterer custom HTML)
- [ ] **SQL-injection-test:** prøv `' OR 1=1--` i hvert search/filter-input
- [ ] **XSS-test:** prøv `<script>alert(1)</script>` i hvert tekstfelt
- [ ] **File-upload-validering:** mime-type allowlist + størrelse-limit + magic-byte sjekk

### 1.5 Anti-spam og abuse

- [ ] **Turnstile** på alle public skjemaer (kontakt, signup, etc.)
- [ ] **Honeypot** i tillegg til Turnstile (gratis ekstra lag)
- [ ] **Rate-limit** på alle endpoints: `/api/contact` 5/min/IP, auth 10/min/IP, etc.
- [ ] **Rate-limit testet** med faktiske rapid requests — skal returnere 429
- [ ] **Audit log** av alle admin-actions

### 1.6 Auth-flyt

- [ ] **Magic link** funker og redirecter til riktig URL (ikke localhost)
- [ ] **Password reset** funker og redirecter riktig
- [ ] **Invite user** funker (hvis aktuelt)
- [ ] **Sign-out** fjerner all session-state
- [ ] **Brute-force-prevention** på passord-innlogging
- [ ] **JWT-utløp** verifisert (default 1 time)
- [ ] **Refresh token rotation** aktivert

### 1.7 Penetrasjonstesting (light)

- [ ] **Manuell sjekk** av alle authentication bypass: prøv å hoppe over login-sjekk
- [ ] **CSRF-test** på alle state-changing endpoints
- [ ] **Direct object reference test:** kan du se andre brukeres data ved å endre ID i URL?
- [ ] **Open redirect test:** redirecter siden til ekstern URL fra query-param?

### 1.8 Security review-agent

- [ ] Kjør `security-reviewer`-agent på hele kodebasen
- [ ] Fix alle CRITICAL og HIGH funn
- [ ] Dokumenter alle MEDIUM funn som er bevisst akseptert

---

## 2. Performance

### 2.1 Lighthouse CI

- [ ] **Performance:** ≥ 95 på alle hovedsider (forside, kontakt, viktig landing)
- [ ] **Accessibility:** ≥ 95
- [ ] **Best Practices:** ≥ 95
- [ ] **SEO:** ≥ 95
- [ ] **PWA** (hvis aktuelt): ≥ 90

### 2.2 Core Web Vitals (CrUX + lab)

- [ ] **LCP** < 2.5s p75
- [ ] **INP** < 200ms p75
- [ ] **CLS** < 0.1 p75
- [ ] **FCP** < 1.5s
- [ ] **TBT** < 200ms
- [ ] **TTFB** < 800ms

### 2.3 Bundle-budsjett (gzipped)

- [ ] **Landing page** < 150 KB JS
- [ ] **App-side** < 300 KB JS
- [ ] **Admin** < 400 KB JS
- [ ] **CSS** < 50 KB per side
- [ ] Verifisert med `next build` + `@next/bundle-analyzer`

### 2.4 Bilder

- [ ] **Alle bilder** har eksplisitte `width` og `height`
- [ ] **AVIF + WebP** konvertering aktiv via Next.js Image
- [ ] **Lazy loading** på alt under fold
- [ ] **Eager loading** kun på hero-bilde med `fetchpriority="high"`
- [ ] **Ingen src URL-er** for bilder som ikke finnes i `next.config.ts` `remotePatterns`

### 2.5 Fonts

- [ ] **Max 2 font-familier** (med mindre eksplisitt unntak)
- [ ] **`font-display: swap`**
- [ ] **Preload** kun den kritiske vekten
- [ ] **Subsetting** der mulig

### 2.6 Tredjeparts-scripts

- [ ] **Asynkron lasting** (`async` eller `defer`)
- [ ] **Kun etter consent** for analytics/marketing
- [ ] **SRI (Subresource Integrity)** for CDN-scripts hvor mulig

---

## 3. Accessibility (WCAG 2.2 AA)

- [ ] **axe-core** kjørt på alle public-sider — 0 violations
- [ ] **Manuell keyboard-test** av alle interaktive flater
- [ ] **Tab-rekkefølge** logisk på hver side
- [ ] **Focus visible** på alle elementer
- [ ] **ARIA-labels** der ikoner brukes uten tekst
- [ ] **`prefers-reduced-motion`** respektert overalt
- [ ] **Kontrast** ≥ 4.5:1 body, ≥ 3:1 stort tekst (verifisert med Lighthouse)
- [ ] **Touch-targets** ≥ 44×44 px på mobil
- [ ] **Språk-attributt** `<html lang="nb">` satt
- [ ] **Skjema-feilmeldinger** lest av screen readers (NVDA + VoiceOver test)
- [ ] **Skip-til-innhold-lenke** for keyboard-brukere
- [ ] **Tilgjengelighetserklæring** publisert på `/tilgjengelighet` (hvis universell utforming-loven gjelder)

---

## 4. SEO

- [ ] **`/sitemap.xml`** auto-generert + valid (https://www.xml-sitemaps.com/validate-xml-sitemap.html)
- [ ] **`/robots.txt`** korrekt + tillater crawling
- [ ] **`/llms.txt`** + `/llms-full.txt` for LLM-crawlere
- [ ] **JSON-LD** validert på alle relevante sider (https://search.google.com/test/rich-results)
  - [ ] `Organization` globalt
  - [ ] `LocalBusiness` på forside + kontakt (hvis lokal virksomhet)
  - [ ] `Service` per tjeneste-side
  - [ ] `Article` per blogg-post
  - [ ] `BreadcrumbList` på alle dyp-sider
  - [ ] `FAQPage` der FAQ finnes
- [ ] **Per side:** `<title>` + `<meta description>` + `<meta og:image>` satt
- [ ] **Canonical URLs** korrekt (ingen duplikate canonical)
- [ ] **hreflang** hvis flerspråklig
- [ ] **Open Graph + Twitter Card** validert (https://www.opengraph.xyz/)
- [ ] **Indexering aktivert** for prod (verifiser `noindex` ikke satt feil)

---

## 5. Compliance (Norge)

### 5.1 GDPR

- [ ] **Personvernpolicy** publisert på `/personvern`
- [ ] **Cookiebot consent** aktiv (ALDRI egen banner) — `data-blockingmode="auto"` blokkerer ikke-essensielle scripts FØR samtykke
- [ ] **`/cookies`** bruker Cookiebots auto-genererte `CookieDeclaration` (ikke håndskrevet liste)
- [ ] **Cookiebot-domener i CSP**: `consent.cookiebot.com` + `consentcdn.cookiebot.com` i script-src OG connect-src
- [ ] **Cookiebot-CBID + GTM-ID** mottatt fra Helge og satt i prod — **ikke placeholder** (`REPLACE_WITH_CBID` / `GTM-XXXXXXX`); guard `cookiebotEnabled === true` i prod
- [ ] **Google Consent Mode** aktiv hvis Google brukes (GCM default-script før gtag/GTM, «Google Consent Mode» på i Cookiebot Manager)
- [ ] **Manuelle cookies/trackere ført i Cookiebot** — alt scanneren ikke fant (bak innlogging, etter brukerhandling, server-/first-party: Supabase `sb-*`, CSRF, rate-limit, Turnstile) lagt inn under «Manually added cookies and trackers» med riktig kategori; cookie-oversikt i §15.2 er komplett
- [ ] **`/api/gdpr/export`** fungerer (test med faktisk bruker)
- [ ] **`/api/gdpr/delete`** fungerer + anonymiserer i stedet for slette der regnskapsår krever
- [ ] **DPO-kontakt** oppgitt i personvernpolicy

### 5.2 Bransje-spesifikt

- [ ] **Åpenhetsloven** (hvis terskel — omsetning > 70 mill NOK eller > 50 ansatte): side `/apenhetsloven` med aktsomhetsvurdering
- [ ] **Universell utforming-loven** (hvis offentlig sektor eller B2C): tilgjengelighetserklæring + WCAG 2.1 AA verifisert
- [ ] **E-handelsloven** (hvis salg): pris-transparens + leveringsbetingelser + angrerett (14 dager B2C)
- [ ] **Hvitvaskingsloven** (hvis finans / eiendomsmegling): KYC-flyt
- [ ] **Markedsføringsloven:** ingen villedende markedsføring, anmeldelser ekte

### 5.3 Vilkår

- [ ] **`/vilkar`** publisert med salgsvilkår (hvis salg)
- [ ] **Versjon + dato** på vilkår
- [ ] **Lenker fra footer** på alle public-sider

---

## 6. DNS + E-post

- [ ] **A-record** prod-domene → Vercel IP
- [ ] **CNAME** www → cname.vercel-dns.com
- [ ] **DKIM** verifisert i Resend (https://dnschecker.org/)
- [ ] **SPF** record: `v=spf1 include:_spf.resend.com -all`
- [ ] **DMARC** record: `p=quarantine` (oppgrader til `p=reject` etter 30 dager)
- [ ] **MX-records** (hvis innkommende e-post)
- [ ] **www-canonical** (apex eller www — én redirecter til den andre)
- [ ] **HTTPS enforced** (HTTP → HTTPS redirect verifisert)
- [ ] **TLS-cert** auto-provisjonert av Vercel + ikke utløpt
- [ ] **Mail-Tester score** ≥ 9/10 for sender (https://www.mail-tester.com/)
- [ ] **Test-email mottatt** + ikke i spam i Gmail, Outlook, Apple Mail

---

## 7. Supabase admin-oppsett (jf. §14.7)

- [ ] **Auth → URL Configuration:** Site URL + Redirect URLs satt for prod
- [ ] **Auth → Email Templates:** alle 6 HTML-maler limt inn fra `emails/auth/`
- [ ] **Auth → SMTP Settings:** Custom SMTP via Resend, sender: `noreply@[domene]`
- [ ] **Auth → Providers:** Email + ev. Google SSO konfigurert
- [ ] **Database → Backups:** Point-in-Time Recovery aktivert (Pro: 7 dager)
- [ ] **Storage → Bucket-policyer:** per bucket public/private + RLS + mime-type allowlist
- [ ] **Test-email mottatt** og verifisert

---

## 8. CMS (hvis aktuelt — jf. regel #15)

- [ ] **ALT redigerbart** verifisert: gå gjennom hver public-side og endre minst ett element fra CMS
- [ ] **Bilde-opplastingsmodul** fungerer (drag-drop, multi-file, alt-tekst-krav)
- [ ] **Mediabibliotek** med søk + mapper fungerer
- [ ] **Live preview** oppdateres innen 500ms
- [ ] **Auto-save** fungerer + recovery av kladd testet
- [ ] **Publiseringsflyt** testet (draft → published)
- [ ] **Schedulert publisering** testet
- [ ] **Versjonering** testet — rollback fungerer
- [ ] **Roller** testet (editor kan ikke gjøre superadmin-actions)
- [ ] **Audit log** har entries for alle test-endringer
- [ ] **Ingen hardkodet innhold** i public-koden (eneste unntak: footer-credit + juridiske disclaimers)

---

## 9. Forms

- [ ] **Hvert skjema** sendt med testdata + bekreftet mottak via Resend
- [ ] **Validering** på client + server (Zod)
- [ ] **Tom innsending** håndteres grasiøst
- [ ] **Ugyldig e-post** blokkeres med klar feilmelding
- [ ] **Rate-limit** testet (rapid submissions blokkeres)
- [ ] **Turnstile** verifisert (uten korrekt token = 403)
- [ ] **Honeypot** testet (bot-felt utfylt = 403)
- [ ] **Inbox** i admin viser submission
- [ ] **Bekreftelse-email** mottatt av avsender

---

## 10. Monitoring + DR

- [ ] **Uptime-monitoring** (UptimeRobot / Better Stack) sjekker prod hvert minutt
- [ ] **`/api/health`-endpoint** returnerer 200 + sjekker DB-tilkobling
- [ ] **Sentry** mottar errors fra prod (test ved å throw)
- [ ] **Sentry alerts** konfigurert til Slack + e-post
- [ ] **Vercel Speed Insights** aktivert
- [ ] **Vercel Analytics** aktivert (cookie-fritt)
- [ ] **PostHog** (hvis brukt) kun lastet etter consent
- [ ] **Status-page** publisert (hvis forretningskritisk)

### 10.1 DR

- [ ] **PITR-restore** testet i staging — verifiser at vi kan restore
- [ ] **Backup-prosedyre** dokumentert
- [ ] **Runbook for de 4 scenariene** (Vercel down, Supabase down, DNS issue, Resend issue) skrevet og lagret i `docs/runbooks/`
- [ ] **RTO** (4 timer) + **RPO** (5 min via PITR) bekreftet

---

## 11. Cross-browser + responsive

- [ ] **Chrome desktop** (Mac + Windows) — alle critical paths
- [ ] **Firefox desktop** — alle critical paths
- [ ] **Safari macOS** — alle critical paths
- [ ] **Edge Windows** — alle critical paths
- [ ] **iPhone Safari** (iOS 17+)
- [ ] **iPhone Chrome**
- [ ] **Samsung Browser** (Android)
- [ ] **iPad** (Safari)
- [ ] **Bredder:** 320, 375, 414, 768, 1024, 1280, 1440, 1920 — ingen overflow

---

## 12. Innhold

- [ ] **Ingen lorem ipsum** noe sted i prosjektet (`grep -r -i "lorem"`)
- [ ] **Ingen «TODO», «TBD», «FIXME», «coming soon»** i live-tekst
- [ ] **Norsk korrektur lest på HVER public-side** (jf. regel #42 + `05-BRAND_VOICE.md` «Korrektur og ærlighet») — rettskriving, bøying, tegnsetting, norske typografiregler («», –, kr-format); ingen AI-typos eller rå-oversatt engelsk
- [ ] **Ingen overdrivelse/oppdiktede fakta** — hver konkret påstand (tall, årstall, kundeantall, erfaring, sertifiseringer, garantier, responstid, utmerkelser) er forankret i kundedata; uverifiserte påstander er flagget til kunden, ikke gjettet
- [ ] **Norsk bokmål** (med mindre kunde-spec sier annet)
- [ ] **Alle bilder** har meningsfull alt-tekst (ikke «image1.jpg»)
- [ ] **Ingen placeholder-bilder** (Unsplash uten relevans, generic avatars)
- [ ] **Forbudte fraser** fra `05-BRAND_VOICE.md` ikke brukt
- [ ] **Footer-credit «Utviklet av Myrvoll-Lunde IT Drift»** synlig på alle public-sider med fungerende lenke
- [ ] **Kontakt-info** korrekt (test-ring telefonnummer, send test-email)
- [ ] **Sosiale medier-lenker** fungerer
- [ ] **Åpningstider** stemmer

### Hvis `requires_scrape: true`

- [ ] `pnpm tsx scripts/verify-scrape-import.ts` mot **prod-DB** — exit 0
- [ ] Spot-sjekk **5 tilfeldige scrapede sider** i prod: innhold, bilder, lenker, meta-description
- [ ] **Alt-tekst manuelt redigert** på minimum hero-bilder + de 20 mest brukte bildene (fallback fra filnavn er ikke godt nok for lansering)
- [ ] **301-redirects** fra gamle URL-er satt opp i `next.config.js` eller Vercel-redirects, mapping bygget fra scraped `url`-felter
- [ ] **`.github/workflows/verify-scrape.yml`** har kjørt grønt på siste merge til main

---

## 13. Sprintplan-leveranser

- [ ] **Alle suksesskriterier** fra PROJECT.md krysset av
- [ ] **Alle planlagte features** for sprint 1 implementert (eller eksplisitt utsatt og dokumentert i «Ikke i scope»)
- [ ] **Test-dekning** ≥ 80% statement coverage (Vitest)
- [ ] **E2E-tester** grønne i CI (Playwright)
- [ ] **Lighthouse CI** grønn i pipeline

---

## 14. Kunde-sign-off

- [ ] **Kunde har sett alle hovedsider** på prod-domene
- [ ] **Kunde har testet CMS** (hvis aktuelt) og redigert minst én side
- [ ] **Kunde har bekreftet skriftlig** at de aksepterer leveransen
- [ ] **Kunde har fått opplæring** i CMS / admin (video eller live)
- [ ] **Kunde har fått dokumentasjon** (CLAUDE.md + README + admin-guide)
- [ ] **Vedlikeholdsavtale** signert (hvis aktuelt)
- [ ] **Kunde har egen tilgang** til Vercel, Supabase, Resend, Domain registrar

---

## 15. Lansering

- [ ] **DNS TTL redusert til 300s** 24 timer før cutover (lett rollback hvis problemer)
- [ ] **Backup tatt** rett før cutover
- [ ] **Cutover-tid avtalt** med kunde (utenfor peak)
- [ ] **Rollback-plan** dokumentert
- [ ] **Status-meldinger forberedt** for kunde-team + sosiale medier
- [ ] **Etter cutover:** sjekk alle critical paths innen 30 min
- [ ] **24-timers vakt** etter lansering — utvikler på beredskap
- [ ] **Lansering-rapport** skrevet og committed til `docs/launches/`

---

## Pre-launch-rapport (mal)

Skrives som `docs/launches/YYYYMMDD-launch-report.md`:

```markdown
# Lanseringsrapport — [Kunde] — YYYY-MM-DD

**Domene:** [domene.no]
**Cutover-tid:** YYYY-MM-DD HH:MM
**Utvikler:** [navn]
**Sign-off av Helge:** [navn + dato]
**Sign-off av kunde:** [navn + dato]

## Status per sjekkliste

### 1. Sikkerhet: ✓ / ✗ / N/A
- [Tabell med hver bullet og status]

### 2. Performance: ✓ / ✗ / N/A
- [...]

[osv. for alle 15 seksjoner]

## Kjente issues (ikke blokkerende for lansering)

- [Liste]

## Post-launch oppfølging

- Dato for 7-dagers retrospective: YYYY-MM-DD
- Dato for 30-dagers review: YYYY-MM-DD
```

---

## For AI

Når AI lager PROJECT.md, må §20 (Sprintplan) ha en dedikert siste sprint kalt «Pre-launch» med ALLE punktene over som oppgaver. Denne sprinten kan ikke hoppes over — den er en del av 10/10-leveransen.

Suksesskriterier-seksjonen må også referere til pre-launch-rapporten som obligatorisk leveranse.
