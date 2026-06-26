# Section Detail — Compliance (Norge)

> Brukes når AI fyller ut seksjon 15 (GDPR + lovpålagt) i PROJECT.md. Norge har egne lover utover EU-GDPR.

---

## Minimum required output

Seksjon 15 må inneholde:

1. **GDPR-implementering** (alltid)
2. **Cookie-policy + samtykke** (alltid hvis cookies)
3. **E-handelsloven** (hvis salg)
4. **Åpenhetsloven** (hvis omsetning > 70 mill NOK eller > 50 ansatte)
5. **Universell utforming-loven** (hvis offentlig sektor eller rettet mot allmennheten)
6. **Markedsføringsloven** (hvis markedsføring)
7. **Andre bransjespesifikke** (eks: hvitvaskingsloven for finans)

AI **må** identifisere hvilke som gjelder basert på kundedata og inkludere ALLE som gjelder.

## Per-lov-detaljer

### GDPR (alltid)

**Personvernpolicy** må inkludere:

- Hvilke data samles inn (kategorier + spesifikt)
- Formål (lovlig grunnlag per behandling)
- Oppbevaringstid per kategori
- Hvem som har tilgang (intern + databehandlere)
- Brukerens rettigheter:
  - Innsyn (Art. 15)
  - Retting (Art. 16)
  - Sletting / "right to be forgotten" (Art. 17)
  - Begrensning (Art. 18)
  - Dataportabilitet (Art. 20)
  - Innsigelse (Art. 21)
- Klage til Datatilsynet (kontakt-info)
- Behandlingsansvarlig (org.nr + adresse)
- Databeskyttelsesombud (DPO) hvis aktuelt

**Endpoints:**

- `/api/gdpr/export` — generer JSON med all bruker-data
- `/api/gdpr/delete` — slett / anonymiser ved verifisert forespørsel

**Implementasjon:**

```ts
// app/api/gdpr/delete/route.ts
export async function POST(req: Request) {
  const { email, token } = await req.json()
  // 1. Verifiser at token matcher e-post + nylig sendt
  // 2. Anonymiser brukerens form_submissions (slett PII, behold ID for audit)
  // 3. Anonymiser bookings (slett PII, behold pris/historikk for regnskap)
  // 4. (cookie-samtykke ligger hos Cookiebot — ikke vår tabell, ingenting å slette her)
  // 5. Logg i audit_log
  // 6. Send bekreftelse via Resend
}
```

### Cookie-policy + samtykke (alltid hvis cookies) — **KUN Cookiebot**

> 🔒 **HARDT KRAV: Vi bygger ALDRI egen consent-/cookie-banner. Samtykke håndteres ALLTID av Cookiebot.**
> Egen banner, egen samtykke-modul og egen `cookie_consents`-tabell er **forbudt** — Cookiebot
> dekker banner, kategorisering, samtykke-logg, IAB TCF og auto-generert cookie-deklarasjon. Å
> rulle vårt eget gir oss vedlikeholds- og compliance-risiko vi ikke skal ta.

**AI MÅ spørre Helge om Cookiebot-koden** før consent settes opp — den kan ikke gjettes:

- Be eksplisitt om **Cookiebot-CBID** (domene-gruppe-ID) / `<script>`-snutten fra Cookiebot Manager.
- Legg dette som et eget punkt under «## Spørsmål til kunden/Helge» i PROJECT.md.
- Ikke implementer consent før koden er mottatt — marker som blokkert til den finnes.

**Oppsett (når CBID er mottatt):**

- Cookiebot-scriptet lastes **først i `<head>`** med `data-cbid="<CBID>"` og
  `data-blockingmode="auto"` slik at det auto-blokkerer alle ikke-essensielle scripts før samtykke.
- `/cookies` bruker Cookiebots **auto-genererte deklarasjons-script** (`CookieDeclaration`) — ikke en
  håndskrevet cookie-tabell.
- Footer-lenke «Informasjonskapsler» åpner Cookiebots dialog på nytt
  (`Cookiebot.renew()`), så bruker kan endre samtykke når som helst.
- Analytics/marketing-scripts (PostHog, GA4, Pixel …) lastes **kun** etter at Cookiebot melder
  samtykke for riktig kategori (`window.Cookiebot.consent.statistics` / `.marketing`).
- **CSP:** `consent.cookiebot.com` + `consentcdn.cookiebot.com` i BÅDE `script-src` og
  `connect-src` (+ `imgsct.cookiebot.com` i `img-src`). Se `csp-config.md`.

Samtykke-logg ligger hos Cookiebot — vi lagrer den **ikke** selv.

#### Manuell registrering av cookies/trackere scanneren ikke finner (OBLIGATORISK)

Cookiebots automatiske scanner crawler kun **offentlige** sider. Den finner IKKE cookies/trackere som
bare settes:

- **bak innlogging** (dashboard, admin, medlemssider, «min side»),
- **etter en brukerhandling** (skjema-submit, checkout, booking, chat åpnet),
- **dynamisk injisert** av tredjeparts-scripts som lastes betinget,
- **server-satte / first-party app-cookies** (auth/session, CSRF, rate-limit, preferanser).

Disse MÅ legges inn manuelt i Cookiebot → **«Manually added cookies and trackers»**, med riktig
kategori (Nødvendig / Statistikk / Markedsføring / Preferanser), navn, formål, varighet og leverandør.
Ufullstendig cookie-deklarasjon = ikke compliant.

**Slik gjør AI det per prosjekt:**

1. **Kartlegg ALLE cookies/trackere** — også på innloggede sider. Kjør appen, logg inn, gå gjennom
   dashboard/admin/skjema-flyter og les `Application → Cookies` + nettverksforespørsler. Sjekk koden:
   Supabase auth-cookies (`sb-*`), CSRF/session, rate-limit, Turnstile (`cf_*`), preferanse-cookies,
   evt. PostHog/GA/Pixel som lastes bak auth.
2. **Sammenlign mot Cookiebots auto-scan** — alt scanneren ikke fant, føres manuelt.
3. **Legg inn manuelt** i Cookiebot Manager med korrekt kategori + metadata.
4. **Lever en cookie-oversikt i PROJECT.md** (§15.2): tabell med hver cookie/tracker — navn, type
   (1st/3rd party), kategori, formål, varighet, hvor den settes (offentlig/innlogget), og om den ble
   funnet automatisk eller må legges inn manuelt.

**Vedlikehold:** hver gang et nytt tracker/cookie-settende script legges til (særlig bak innlogging),
MÅ det manuelt registreres i Cookiebot i samme slengen. Står i AGENTS.md «sjekk først» + pre-launch.

#### Google Consent Mode v2 (OBLIGATORISK når en Google-tjeneste brukes)

Hvis `project.integrations` inneholder **GA4, GTM, Google Ads eller Google Maps**, MÅ Google Consent
Mode v2 (GCM) settes opp sammen med Cookiebot. Uten GCM blir Google-tagger enten helt blokkert av
Cookiebot, eller de fyrer uten gyldig samtykke (lovbrudd). GCM lar Google-tagger laste i «consent-aware»
modus: ingen cookies/ingen PII før samtykke, modellert data etterpå.

**Rekkefølge i `<head>` (kritisk):**

1. **GCM default-script FØRST** — setter alt til `denied` før noe Google laster. Scriptet MÅ ha
   `data-cookieconsent="ignore"` (ellers blokkerer Cookiebot selve bootstrap-en) **og** prosjektets
   CSP-`nonce`. I Next 16 rendres det via `next/script` (`strategy="beforeInteractive"`) med `nonce`
   fra middleware-headeren:

   ```html
   <!-- FØRST i <head>, før Cookiebot og før gtag/GTM -->
   <script data-cookieconsent="ignore" nonce="{NONCE}">
     window.dataLayer = window.dataLayer || [];
     function gtag(){ dataLayer.push(arguments); }
     gtag('consent','default',{
       ad_storage:'denied', ad_user_data:'denied', ad_personalization:'denied',
       analytics_storage:'denied', functionality_storage:'denied',
       personalization_storage:'denied', security_storage:'granted',
       wait_for_update: 500
     });
     gtag('set','ads_data_redaction', true);
     gtag('set','url_passthrough', false);
   </script>
   ```

2. **Cookiebot-script** (`data-cbid` + `data-blockingmode="auto"`).
3. **gtag.js / GTM** etterpå.

Skru på **«Google Consent Mode»** i Cookiebot Manager (Settings → Templates) — da pusher Cookiebot
`consent('update', …)` automatisk når brukeren velger, og du slipper egen update-kode.

**CSP:** GCM bruker Google-domenene som allerede ligger i GA4/GTM-radene i `csp-config.md`
(`*.googletagmanager.com`, `*.google-analytics.com`). Det inline GCM-scriptet treffer CSP via `nonce`
— ingen nye domener trengs utover Cookiebot- + Google-radene.

#### Oppsett-flyt: env-vars + placeholder + guard

Cookiebot-CBID og GTM-ID skaffer **Helge manuelt** per nytt prosjekt (ny Cookiebot-konto + ny
GTM-container). De er offentlige `NEXT_PUBLIC_`-IDer — ikke hemmeligheter. Scaffold med **placeholder**
fra start så appen bygger lokalt, og **guard** scriptene så en fake ID aldri når prod:

```bash
# .env.example + .env.local
NEXT_PUBLIC_COOKIEBOT_CBID="REPLACE_WITH_CBID"   # Helge limer inn ekte CBID
NEXT_PUBLIC_GTM_ID="GTM-XXXXXXX"                 # kun hvis GTM/GA4 brukes
```

```tsx
// src/lib/consent.ts — guard: ekte verdi satt?
const PLACEHOLDERS = new Set(['', 'REPLACE_WITH_CBID', 'GTM-XXXXXXX'])
export const cbid = process.env.NEXT_PUBLIC_COOKIEBOT_CBID
export const gtmId = process.env.NEXT_PUBLIC_GTM_ID
export const cookiebotEnabled = !!cbid && !PLACEHOLDERS.has(cbid)
export const gtmEnabled = !!gtmId && !PLACEHOLDERS.has(gtmId)

// I layout: render Cookiebot/GTM/GCM-scriptene KUN hvis *Enabled === true.
// Mangler ekte CBID → ingen scripts lastes (ingen knekt consent i prod).
```

**Helge får beskjed når:** generert PROJECT.md ber eksplisitt om CBID + GTM-ID i consent-/Sprint 0-steget
(«Spørsmål til kunden»), og markerer consent som blokkert til de er mottatt. **Pre-launch blokkerer**
lansering hvis verdien fortsatt er placeholder (`check`-steg verifiserer at `cookiebotEnabled === true`
i prod). Slik kan det aldri glemmes.

### GDPR art. 30-behandlingsprotokoll (Myrvoll-Lunde IT Drift master) — OBLIGATORISK registrering

> Myrvoll-Lunde IT Drift har **én felles** GDPR art. 30-behandlingsprotokoll. Hvert nytt prosjekt MÅ registreres
> i den — og holdes oppdatert når behandlingen endres. Dette er en **Sprint 0-leveranse** og en
> **pre-launch-blocker**.

**Master (single source of truth — ALDRI dupliser inn i prosjekt-repoet):**
`~/Documents/GitHub/mlit/docs/compliance/Behandlingsprotokoll-MLITDrift.md`
Seksjoner: `Forside` · `Art.30(1) Ansvarlig` (egne produkter) · `Art.30(2) Databehandler` (klientprosjekter)
· `Underdatabehandlere` · `Prosjektregister`.

**Omfang:** kun prosjekter under Supabase-org «Myrvoll-Lunde IT Drift» og/eller GitHub-org helgelunde1981-creator.
(GangeRolv og private helgelunde1981-repoer uten Myrvoll-Lunde IT Drift-tilknytning holdes utenfor.)

PROJECT.md MÅ inneholde en konkret oppgave (Sprint 0) som kjører denne registreringen, slik at lokal
Claude/Codex utfører den i det nye repoet:

```text
Registrer dette prosjektet i GDPR-behandlingsprotokollen for Myrvoll-Lunde IT Drift.
Master: ~/Documents/GitHub/mlit/docs/compliance/Behandlingsprotokoll-MLITDrift.md

1. ROLLE: «Egen (behandlingsansvarlig)» eller «Klient (databehandler)»?
2. KARTLEGG: les package.json + src for databehandlere (Supabase, Vercel, Resend, Stripe, Vipps,
   Anthropic/OpenAI, Google, Hetzner, Cookiebot, Sentry, Cloudflare/Turnstile, Upstash) og funksjoner
   (auth, kontaktskjema, nyhetsbrev, betaling, medlem/påmelding, booking, e-signering, chat, analyse).
   Les .env(.local) → Supabase-ref + region; bekreft org «Myrvoll-Lunde IT Drift».
3. Legg til rad i «Prosjektregister».
4. Legg til rad i «Art.30(1) Ansvarlig» (egen) eller «Art.30(2) Databehandler» (klient): formål,
   rettslig grunnlag (art. 6), registrerte, persondata-kategorier, databehandlere, tredjelands-
   overføring, lagringssted, sletterutine, sikkerhetstiltak (+ kunde/DPA for klient).
5. Ny underdatabehandler? Legg til i seksjonen «Underdatabehandlere» + speil i personvernerklæringen.
6. Sett Cookiebot-status. Marker ukjent med [FYLL INN ...].
7. Oppdater versjon/dato på «Forside». Commit til mlit-repoet. Ikke dupliser protokollen.
```

**Vedlikehold (kritisk):** Hver gang prosjektet får en **ny underdatabehandler/tredjepartstjeneste**,
**ny datatype/formål**, eller **ny funksjon som behandler persondata** (auth, skjema, nyhetsbrev,
betaling, medlem/påmelding, booking, e-signering, chat), MÅ protokoll-raden + ev. «Underdatabehandlere»
oppdateres i samme slengen — og speiles i personvernerklæringen. AGENTS.md skal liste dette i «sjekk
først»-lista så lokal agent ikke glemmer det. Pre-launch verifiserer at prosjektet finnes i
Prosjektregister med korrekt ref/region/rolle + at alle aktive underdatabehandlere er ført.

### E-handelsloven (hvis salg av varer/tjenester)

Krav:

- **Pristransparens**: priser inkl. mva, lever klart frem
- **Leveringsbetingelser**: tid, kostnad, geografisk område
- **Angrerett**: 14 dager for B2C, formular tilgjengelig
- **Tilstrekkelig info før kjøp**: produktbeskrivelse, totalpris, leveringsdato
- **Vilkår-side** redigerbar i CMS

### Åpenhetsloven (terskel: > 70 mill NOK omsetning ELLER > 50 ansatte ELLER total balanse > 35 mill NOK)

Krav:

- **Aktsomhetsvurdering** publisert på nettsiden
- **Årlig redegjørelse** publisert innen 30. juni hvert år
- **Kontaktpunkt** for informasjonsforespørsler (e-post)
- **5 virkedagers responstid** på informasjonsforespørsler

Implementasjon:

- Dedikert side: `/apenhetsloven`
- Last opp PDF med aktsomhetsvurdering + årsrapport
- Skjema eller e-post: `apenhet@[domene]`

### Universell utforming-loven (offentlig sektor + private mot allmennheten)

Krav:

- **WCAG 2.1 AA minimum** (forskrift fra 2014)
- **WCAG 2.2 AA anbefalt** (vi leverer 2.2 AA som default)
- **Tilgjengelighetserklæring** publisert
- **Rapportering** til Diskrimineringsnemnda hvis offentlig

Implementasjon:

- Verifisert via axe-core i CI
- Tilgjengelighetserklæring: `/tilgjengelighet` — beskriver WCAG-status, kjente avvik, kontakt
- Statisk innhold redigerbart

### Likestillings- og diskrimineringsloven (alltid)

Universell utforming-prinsipper. Dekkes implisitt av WCAG 2.2 AA.

### Markedsføringsloven (hvis markedsføring)

- **Ingen villedende markedsføring**
- **Pristransparens**: ingen "fra" uten kontekst
- **Sammenligninger** må være ærlige
- **Anmeldelser** kan ikke kjøpes/lages opp
- **Influencer-merket innhold** må merkes "annonse"

### Bransjespesifikke (eksempler)

| Lov | Når gjelder | Krav |
|-----|-------------|------|
| Hvitvaskingsloven | Finans, eiendomsmegling | KYC, kunde-tiltak |
| Helsepersonelloven | Helsesektoren | Taushetsplikt, journalføring |
| Bokføringsloven | Salg | Lagre transactional data 5 år |
| Personalsikkerhetsloven | Statlig | Sikkerhetsklarering hvis aktuelt |

## Default side-struktur

Hvert prosjekt skal ha disse sidene (redigerbart i CMS):

| URL | Innhold |
|-----|---------|
| `/personvern` | Personvernpolicy (GDPR) |
| `/cookies` | Cookie-policy med liste |
| `/vilkar` | Salgsvilkår + angrerett (hvis salg) |
| `/apenhetsloven` | Åpenhetsloven-info (hvis terskel) |
| `/tilgjengelighet` | Tilgjengelighetserklæring (hvis universell utforming) |

Lenker fra footer: alltid synlig, samme posisjon på alle sider.

## Implementasjons-sjekkliste for AI

For hvert prosjekt:

- [ ] Identifisert hvilke lover som gjelder basert på kundedata
- [ ] Listet hver gjeldende lov med konkret implementasjonskrav
- [ ] Consent = Cookiebot (ALDRI egen banner); spurt Helge om Cookiebot-CBID/kode
- [ ] Inkludert GDPR-endpoints (/api/gdpr/export, /api/gdpr/delete)
- [ ] Listet alle påkrevde sider (personvern, cookies, etc.)
- [ ] Notert footer-lenker som alltid synlig
- [ ] Identifisert om Åpenhetsloven gjelder (sjekk org.nr → Brønnøysund hvis usikker)
- [ ] Identifisert om Universell utforming-loven gjelder (offentlig eller B2C)
