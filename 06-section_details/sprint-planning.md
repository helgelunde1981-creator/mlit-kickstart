# Section Detail — Sprintplan (alle oppgaver eksplisitt, ingen glemt)

> Sprintplanen er den **operasjonelle ryggraden** i PROJECT.md. Hvis en oppgave ikke står i sprintplanen, blir den glemt i implementering. AI MÅ liste hver eneste oppgave fra alle 17 regler + alle seksjon-detail-fragmenter som eksplisitte sprint-oppgaver.

---

## Hovedregel

Hvis det er nevnt i en sjekkliste, en regel, en seksjon, eller en kunde-spec — det skal være en **bullet** i en sprint med checkbox. Ikke implisitt. Ikke «se §X». Ikke «som standard». **Eksplisitt.**

Anti-pattern AI gjør (du gjør ikke):

- ❌ «Sprint 4 — Vanlige features. Implementer alt som er listet.»
- ❌ «Sprint 8 — Standard sikkerhets-oppsett.»
- ❌ «Sprint 3 — Resten av komponentene.»

Riktig:

- ✓ «Sprint 4 — Fraktkalkulator. Bygg pris-engine (50 testscenarier). Bygg sone-velger. Bygg tilvalgs-toggles. Bygg bestillingsflyt. Bygg «be om tilbud»-flyt. Bygg CMS for redigering av prislister.»

---

## Standard-mal per prosjekt (sprinter må eksplisitt nevne ALT)

### Sprint 0 — Grunnmur (1 dag)

**Mål:** Tom Next.js-app deployer automatisk til Vercel + Supabase Branching + e-post + secrets klar.

- [ ] Lag GitHub-repo `helgelunde1981-creator/[prosjekt]` (privat, dev som default branch)
- [ ] `pnpm dlx create-next-app@latest [prosjekt] --typescript --tailwind --app --turbopack`
- [ ] Installer dependencies: shadcn/ui, Lucide-react, Framer Motion, react-hook-form, zod, @supabase/ssr, resend, @react-email/components, @tiptap/react, @tiptap/starter-kit
- [ ] Opprett Supabase-prosjekt (Helge i Dashboard)
- [ ] Aktiver Supabase Branching i Dashboard (Settings → Branching)
- [ ] Opprett Vercel-prosjekt i `myrvoll-lunde-it-drift-s-projects`-team + koble til GitHub-repo
- [ ] Konfigurer auto-deploy: main → prod, dev → preview
- [ ] Opprett Resend-konto + lag API-key
- [ ] Domain-verifisering for Resend (DKIM/SPF/DMARC i DNS — gjør i Sprint 1)
- [ ] Opprett Turnstile-keys (site + secret)
- [ ] Opprett Upstash Redis (eller verify Vercel KV) for rate-limit
- [ ] **Secrets-flyt** — kjør initial-setup-script (jf. §14.5): sett ALLE secrets 3 steder (`.env.local` + Vercel × 3 + GH)
- [ ] Bekreft `.gitignore` har `.env*`
- [ ] `.github/workflows/supabase-migrations.yml` opprettet
- [ ] `.github/workflows/ci.yml` (lint + typecheck + vitest + build + lighthouse + axe)
- [ ] `.github/workflows/sync-auth-templates.yml` (synker email-maler ved push)
- [ ] Sett opp ECC-hooks per `web/hooks.md` (format/lint/typecheck/build)
- [ ] **Obsidian-vault auto-registrering**: `C:\Github\vault\prosjekter\[Prosjekt].md` + oppdater Prosjektoversikt + klient-notat
- **Leveranse:** Tom Next.js-app deployer automatisk til Vercel; Supabase-prosjekt koblet; alle CI-jobber grønne på første commit; alle 3 secret-stores synket.

### Sprint 1 — Datagrunnlag, auth, og admin-skall (2-4 dager)

**Mål:** Database-skjema klar, auth fungerer, admin-route eksisterer.

- [ ] Skriv alle migrasjonsfiler i `supabase/migrations/YYYYMMDDHHMMSS_navn.sql` for HVER tabell i §12
  - For HVER tabell: explicit list — pages, page_sections, page_revisions, media, profiles, ..., osv.
- [ ] Push migrations via GitHub Actions til dev-branch (verifiser `supabase migration list`)
- [ ] Skriv RLS-policyer per tabell (egne migration-filer hvis mange)
- [ ] Generer TypeScript-typer: `npx supabase gen types typescript --project-id [ref] > src/types/database.types.ts`
- [ ] Bygg `src/lib/supabase/server.ts` + `client.ts` + `admin.ts`
- [ ] Supabase Auth: magic link + e-post/passord
- [ ] Opprett første superadmin (Helge)
- [ ] Bygg `AdminAuthGuard` + `AdminLayoutShell`
- [ ] Admin route `/admin` med beskyttet layout
- [ ] Cookiebot consent satt opp (ALDRI egen banner): script i `<head>` med `data-cbid` + `data-blockingmode="auto"`, Cookiebot-domener i CSP, `/cookies` med `CookieDeclaration`. Krever CBID fra Helge.
- [ ] **Anti-spam**: Turnstile + honeypot + Zod + rate-limit på alle public-skjemaer
- [ ] **Sikkerhets-headers**: middleware med CSP (nonce), HSTS, X-Frame-Options, etc.
- **Leveranse:** Superadmin kan logge inn på `/admin`; Cookiebot consent virker (auto-blokkerer pre-consent); sikkerhets-headers verifisert med securityheaders.com (A+).

### Sprint 2 — Design-system + layout + design-direction (2-3 dager)

**Mål:** Visuell identitet på plass, header/footer/nav fungerer.

- [ ] Implementer designtokens fra valgt 2026-retning (`src/styles/tokens.css`)
- [ ] Tailwind-config med tokens (jf. §9)
- [ ] Skriv inn shadcn-komponenter (Button, Input, Textarea, Card, Dialog, Toast via Sonner, Tabs, Badge, etc.)
  - For HVER komponent: nevn variants og hovedendringer (jf. §9.5)
- [ ] Bygg `<Header />` og `<Footer />` (med Myrvoll-Lunde IT Drift-credit jf. regel #11)
- [ ] Bygg mobil-nav
- [ ] Implementer dark mode hvis prosjektet skal ha det
- [ ] Bygg `<PageRenderer />` (TipTap JSON → RSC) — eller per-section renderer
- [ ] Bygg `<SectionRenderer />` med hver section-type (hero, image_text, gallery, faq, cta, testimonials, rich_text)
- [ ] 404 / 500 / not-found / error / loading-sider med design-tokens
- [ ] OG image-generator (Next.js OG-image-route)
- [ ] sitemap.xml + robots.txt + llms.txt + llms-full.txt (basis)
- [ ] JSON-LD `Organization` + `WebSite` (globalt)
- **Leveranse:** Hjemmesiden ser premium ut i valgt retning; Lighthouse-baseline tatt.

### Sprint 3 — CMS-skall + alle sider (3-5 dager)

**Mål:** Hver public-side eksisterer + er fullt redigerbar fra admin (jf. regel #15).

For HVER public-side i §11 (URL-tabellen) — eksplisitt list:

- [ ] **Forside** (`/`): bygg public-render fra `pages` + `page_sections`; bygg CMS-redigering med live preview, alle seksjoner edit-bare
- [ ] **Om oss** (`/om-oss`) + alle underseksjoner: …
- [ ] **Tjeneste-/bil-/produkt-side** (per hovedtype): …
- [ ] **Sted × tjeneste-grid** (hvis lokal virksomhet): list HVER kombinasjon (jf. SEO-discovery)
- [ ] **Blogg** (`/blogg`) + post (`/blogg/[slug]`): …
- [ ] **Kontakt** (`/kontakt`): skjema + kart + åpningstider
- [ ] **Personvern** (`/personvern`)
- [ ] **Cookies** (`/cookies`)
- [ ] **Vilkår** (`/vilkar`)
- [ ] **Åpenhetsloven** (`/apenhetsloven`) — hvis terskel
- [ ] **Tilgjengelighet** (`/tilgjengelighet`) — hvis universell utforming-loven
- [ ] **404 / 500**: custom feilside med søk
- [ ] **CMS-komponenter**: rich-text editor (TipTap), ImageUpload (drag-drop, multi-file, alt-tekst, crop, AVIF/WebP), MediaBibliothek (søk, mapper, «brukt på»), SectionEditor per section-type
- [ ] **CMS-flyt**: auto-save hvert 5s, recovery, draft → published, schedulert publisering, versjonering, rollback
- [ ] **Audit log**: hver endring logges i `audit_log`
- [ ] **Roller**: superadmin + editor minimum
- [ ] **Innholdsscrape-script** (`pnpm run scrape`) hvis kunden har eksisterende side
- **Leveranse:** Alle sider eksisterer, alle innhold redigerbart i CMS, alle bilder via opplastingsmodul.

### Sprint 4+ — Per signature feature

Per kjernefeature i §10 — egen sprint med EKSPLISITTE oppgaver:

- [ ] Bygg datamodell-tabeller (egne migrasjons-filer)
- [ ] Bygg pris-/forretningslogikk i `lib/[feature]/`
- [ ] **100% test-dekning** på pris-/forretningslogikk (Vitest, alle test-scenarier)
- [ ] Bygg signature-komponent
- [ ] Bygg admin-side for CRUD på feature-data
- [ ] Bygg public-routes
- [ ] Bygg Server Actions med Zod-validering + rate-limit
- [ ] Bygg E2E-tester (Playwright) for alle critical paths
- [ ] **Resend-eposter** for feature-relevante hendelser (bekreftelser, varsler)

### SEO-sprint (eget sprint hvis lokal virksomhet)

- [ ] Bygg `locations`-tabell + admin-CRUD
- [ ] Bygg `services`-tabell + admin-CRUD
- [ ] Bygg `service_locations`-tabell + admin-CRUD per kombinasjon
- [ ] Bygg `/[service]`, `/[location]`, `/[service]/[location]`-routes
- [ ] Bygg redaktørverktøy for å fylle innhold per kombinasjon (forhindrer template-tekst-spam)
- [ ] JSON-LD per side-type (jf. §13.2 med fullstendige eksempler)
- [ ] Per-side OG image-generator
- [ ] AEO-blokker på alle service-sider
- [ ] Internal linking-strategi implementert
- [ ] Bilde-SEO: filnavn-konvensjon, alt-tekst-validering, captions, EXIF
- [ ] Test alle sider i Search Console Rich Results Test
- [ ] llms.txt + llms-full.txt route-handlers
- [ ] Lighthouse SEO ≥ 95 på alle sider

### E-post-sprint (eget sprint eller del av forms-sprint)

For HVER mal i §15.5 — eksplisitt:

- [ ] Bygg `emails/auth/MagicLink.tsx` med design-tokens
- [ ] Bygg `emails/auth/ConfirmEmail.tsx`
- [ ] Bygg `emails/auth/ResetPassword.tsx`
- [ ] Bygg `emails/auth/Invitation.tsx`
- [ ] Bygg `emails/auth/ChangeEmail.tsx`
- [ ] Bygg `emails/auth/Reauthentication.tsx`
- [ ] Bygg `emails/transactional/ContactReceipt.tsx`
- [ ] Bygg `emails/transactional/ContactInternalAlert.tsx`
- [ ] [Liste ALLE prosjekt-spesifikke transactionale eposter — bestilling, booking, faktura, anmeldelse-forespørsel, osv.]
- [ ] Bygg `emails/marketing/Welcome.tsx`
- [ ] Bygg `emails/components/EmailHeader.tsx` + `EmailFooter.tsx` (med Myrvoll-Lunde IT Drift-credit)
- [ ] Bygg `lib/email.ts` med send-funksjoner per mal
- [ ] Bygg `scripts/sync-supabase-auth-templates.ts` (synker via Management API)
- [ ] CI-workflow synker maler ved push
- [ ] Test hver mal i Mailtrap + Litmus + ekte iPhone Mail + Gmail + Outlook + dark mode
- [ ] Mail-Tester score ≥ 9/10 verifisert

### Forms-sprint

For HVERT skjema i prosjektet:

- [ ] Bygg skjema-komponent med react-hook-form + Zod
- [ ] Bygg Server Action med validering
- [ ] Lagre i `form_submissions`-tabell
- [ ] Send bekreftelses-email + intern alert via Resend
- [ ] Turnstile + honeypot
- [ ] Rate-limit per IP
- [ ] Admin-inbox-side
- [ ] E2E-test for skjemaet
- [ ] Test rate-limit (rapid requests → 429)
- [ ] Test Turnstile uten token (403)

### Forrige siste sprint — Mobile + finpuss + DNS-prep

- [ ] Mobile audit hver side: 320, 375, 414, 768 — ingen overflow, touch-targets ≥ 44px
- [ ] Visual polish: scroll-baserte animasjoner, view-transitions, mikrointeraksjoner
- [ ] Cross-browser test: Chrome, Firefox, Safari, Edge desktop + iPhone, Android, iPad
- [ ] DNS-prep: opprett alle records (A, CNAME, MX, DKIM, SPF, DMARC) med lang TTL ennå
- [ ] Test Resend SMTP: send mail-test til Mail-Tester (https://mail-tester.com) — score ≥ 9/10
- [ ] Initial Lighthouse CI-run: alle scores ≥ 95
- [ ] Alle E2E-tester grønne
- [ ] Code review fra Helge
- **Leveranse:** Klar for pre-launch-sprint.

### Siste sprint — Pre-launch (sikkerhet + readiness, jf. regel #16)

Full sjekkliste fra `06-SECTION_DETAILS/pre-launch-checklist.md` — 15 områder, ALLE skal være krysset av.

Hver sjekkliste-bullet derfra blir en egen oppgave her. Eksempel:

- [ ] **§1.1** securityheaders.com viser A+
- [ ] **§1.2** RLS aktivert på alle tabeller (`select rowsecurity from pg_tables where schemaname='public'`)
- [ ] **§1.2** Anon-rolle test: prøv INSERT/UPDATE/DELETE som anon — alle skal feile
- [ ] **§1.3** `gitleaks detect --source . -v` returnerer 0 funn
- [ ] **§1.3** Alle secrets i Vercel + GH (kjør sjekk-script fra §14.5)
- [ ] **§1.4** Zod-validering verifisert på alle Server Actions (grep + manuell sjekk)
- [ ] **§1.6** Magic link testet ende-til-ende på prod
- [ ] **§1.8** `security-reviewer`-agent kjørt, alle CRITICAL/HIGH fikset
- [ ] **§2.1** Lighthouse Performance ≥ 95 på forside og topp-5 dyp-sider
- [ ] **§2.2** Core Web Vitals «good» p75 (verifiser med Vercel Speed Insights)
- [ ] **§3** axe-core: 0 violations på alle sider
- [ ] **§4** sitemap.xml + robots.txt + llms.txt validert
- [ ] **§4** JSON-LD validert i Rich Results Test for alle side-typer
- [ ] **§5** GDPR endpoints testet: `/api/gdpr/export` + `/api/gdpr/delete`
- [ ] **§6** Mail-Tester ≥ 9/10
- [ ] **§7** Supabase admin-sjekkliste fra §14.7 kryssa av
- [ ] **§8** CMS: alle felter testet redigert + bilde-opplasting testet
- [ ] **§9** Alle skjemaer testet med ekte data
- [ ] **§10** Sentry test-event mottatt
- [ ] **§10.1** PITR-restore testet i staging
- [ ] **§11** Cross-browser-matrise gjennomgått
- [ ] **§12** Lorem/TODO grep returnerer 0 funn
- [ ] **§13** Alle suksesskriterier fra PROJECT.md krysset av
- [ ] **§14** Kunde har testet CMS + signert akseptanse
- [ ] **§15** DNS-cutover-plan signert, backup tatt, 24t vakt avtalt
- [ ] **Skriv lanseringsrapport** `docs/launches/YYYYMMDD-launch-report.md` med 2-personers sign-off
- [ ] **Cutover** utført

---

## Hvordan AI skal skrive sprintplanen

For HVER sprint:

1. **Sprint-nummer + navn** («Sprint 4 — Fraktkalkulator»)
2. **Estimert varighet** («5 dager»)
3. **Mål** (én setning som forklarer hva som er ferdig etter sprinten)
4. **Oppgaver** (checkboxes) — hver eneste oppgave fra alle relevante regler og fragmenter, eksplisitt
5. **Avhengigheter** (hvilke sprinter må være ferdig først)
6. **Test-kriterier** (hva må være verifisert før sprinten regnes som «done»)
7. **Leveranse** (én setning som beskriver hva som er klart å vise kunden)

### Anti-pattern

❌ «Sprint 4: bygg kalkulatoren med alle nødvendige features.»

### Riktig

✓ ```
Sprint 4 — Fraktkalkulator (5 dager)

Mål: Brukere kan beregne pris og bestille direkte fra forsiden og /beregn-pris.

Avhengigheter: Sprint 1 (datagrunnlag), Sprint 2 (design-system).

Oppgaver:
- [ ] Bygg `lib/pricing.ts` med ZoneWeightCalculator + HourlyCalculator + AddonCalculator
- [ ] 100% test-dekning på lib/pricing.ts (Vitest, alle 23 PDF-eksempler)
- [ ] Bygg `<Calculator />` med split-form + live-quote-layout
- [ ] Sone-velger med autocomplete på postnummer
- [ ] Vekt-tier-velger
- [ ] Tilvalgs-toggles (express, helg, tilhenger, ADR, hjelpemann, ventetid, ekstra leveringer)
- [ ] Live pris-breakdown (oppdaterer hver 200ms, debounced)
- [ ] Bestillingsflyt: kontaktinfo → Server Action → Resend bekreftelse + lagring i `bookings`
- [ ] «Be om tilbud»-flyt for kompliserte case
- [ ] Kompakt versjon for forside-hero
- [ ] Full versjon på `/beregn-pris`
- [ ] PDF-eksport av tilbud (jspdf)
- [ ] CMS for redigering: soner, prisgrupper, biltyper, prisregler (alle endringer reflekteres umiddelbart)
- [ ] E2E-test: 20 PDF-test-scenarier verifiserer riktig pris
- [ ] E2E-test: hele bestillingsflyten end-to-end
- [ ] Resend-mal: BookingConfirmation.tsx + bygges + testes

Test-kriterier:
- Vitest: 100% statement coverage på lib/pricing.ts
- Playwright: 20+ scenarier grønne
- Manuell sjekk av PDF-eksport på 3 forskjellige bestillinger

Leveranse: Fraktkalkulator fungerer 100% mot prisliste; alt redigerbart i CMS; bestilling sender ekte e-post.
```

---

## Sjekkliste for AI (sprintplan-seksjon)

- [ ] **Hver regel fra 00-SYSTEM_PROMPT er adressert** med eksplisitte sprint-oppgaver
- [ ] **Hver sjekkliste fra 04-QUALITY_GATES er adressert** med eksplisitte sprint-oppgaver
- [ ] **Hver side i §11** har eksplisitte oppgaver i Sprint 3 (eller egne sprinter for komplekse sider)
- [ ] **Hver sted × tjeneste-kombinasjon** har oppgaver
- [ ] **Hver epost-mal** har oppgaver i e-post-sprinten
- [ ] **Hvert skjema** har oppgaver i forms-sprinten
- [ ] **Hver Supabase-tabell** har oppgave (migration-fil + RLS-policy)
- [ ] **Hver secret** har oppgave i Sprint 0 (`gh secret set` + `vercel env add` × 3 + `.env.local`)
- [ ] **Hver kjernefeature** har egen sprint med konkrete oppgaver
- [ ] **Pre-launch-sjekklisten** (15 områder) er kopiert som eksplisitte oppgaver i siste sprint
- [ ] **Ingen «osv.»** noe sted i sprintplanen
- [ ] **Test-kriterier** + **leveranse** per sprint
