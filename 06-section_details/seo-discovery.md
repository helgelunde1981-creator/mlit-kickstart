# Section Detail — SEO + LLM-discovery + Stedssider (alltid 10/10)

> Hvert prosjekt skal være maksimalt funn-bart på Google + LLM-søk (ChatGPT, Claude, Perplexity, Google AI Overviews). Dette gjøres med en bevisst landingside-arkitektur, ikke bare meta-tags. Hvis kunden tilbyr tjenester i flere geografiske områder, MÅ vi lage dedikerte sted × tjeneste-landingssider.

---

## Filosofi

Google + LLM-er finner ikke en hjemmeside med 5 generiske sider. De finner en hjemmeside med **mange spesifikke, høykvalitets-landingssider** som dekker «hva folk søker etter».

For et lokalt firma som leverer mange tjenester i mange områder, betyr det at vi må generere et **kombinasjons-grid**:

```
        tjeneste1   tjeneste2   tjeneste3   tjeneste4
sted-A     /         /          /           /
sted-B     /         /          /           /
sted-C     /         /          /           /
```

Hver celle er en dedikert URL med unikt innhold. Dette dekker både Google long-tail og LLM-spørsmål som «Hvilken [tjeneste]-leverandør er i [sted]?»

---

## Mandatory metadata per side

Hver eneste public-side må ha:

```html
<!-- Standard meta -->
<title>[Side-tittel] | [Kunde] — [USP]</title>
<meta name="description" content="[155 tegn — beskrivelse + CTA]" />
<link rel="canonical" href="https://[domene][path]" />

<!-- Open Graph -->
<meta property="og:title" content="[...]" />
<meta property="og:description" content="[...]" />
<meta property="og:image" content="https://[domene]/og/[unik].png" />  <!-- 1200x630, ikke gjenbrukt fra andre sider -->
<meta property="og:url" content="https://[domene][path]" />
<meta property="og:type" content="website" /> <!-- eller article for blogg -->
<meta property="og:locale" content="nb_NO" />

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="[...]" />
<meta name="twitter:description" content="[...]" />
<meta name="twitter:image" content="[...]" />
```

### OG image-generering (kritisk)

- Per side: **egen OG image** (1200×630px)
- Generert via Next.js `opengraph-image.tsx` route med design-tokens fra valgt retning
- IKKE samme bilde overalt («OG image fra Unsplash»)
- Inneholder: side-tittel, kunde-logo, design-elementer fra retningen
- AVIF-versjon for moderne klienter

### Per side må AI redigere via CMS

- Title (overstyrer site-default)
- Description
- OG image (egen + override)
- Canonical URL (default = self)
- noindex/nofollow flag
- JSON-LD-overstyring

---

## JSON-LD per side-type (struktur)

### Forside
- `Organization` (globalt)
- `LocalBusiness` hvis lokal virksomhet (inkluderer aaddress, opening_hours, geo)
- `WebSite` med `SearchAction` (Google Sitelinks Search Box)

### Sted × tjeneste-sider (kritisk!)
- `Service` med `areaServed` = sted + `provider` = LocalBusiness
- `LocalBusiness` (gjentatt med stedsspecifikt info)
- `BreadcrumbList`
- `FAQPage` med 3-5 sted-spesifikke spørsmål

### Tjeneste-sider (uten sted)
- `Service` med beskrivelse, tilbyder, områder
- `FAQPage`
- `BreadcrumbList`

### Bilbil / produkt-side
- `Product` eller `Vehicle` (spesifikk skjema)
- `Offer` med pris
- `AggregateRating` (hvis anmeldelser)

### Blogg-post
- `Article` med author, datePublished, dateModified
- `BreadcrumbList`
- `ImageObject` for cover-bilde

### Kontakt
- `LocalBusiness` med full kontaktinfo
- `ContactPage`

### Om oss
- `AboutPage`
- `Organization` med ansatte (`employee[]`)

### Person-sider (team)
- `Person` med jobTitle, image, sameAs (LinkedIn)

AI skal **liste hvilken JSON-LD per side** i seksjon 13 av PROJECT.md med fullstendige eksempler — ikke bare nevne typene.

---

## Stedssider × tjenestesider (KRITISK for lokale virksomheter)

### Når dette er påkrevd

Hvis prosjektet er for et lokalt firma som tilbyr flere tjenester i flere geografiske områder (uansett størrelse), **MÅ** vi lage kombinasjons-landingssider.

Eksempler:
- Transport: tjenester (varebil, kranbil, semi) × områder (Lillestrøm, Skedsmo, Jessheim, Eidsvoll, Lørenskog, ...)
- Bokseklubb: tjenester (treninger, prøvetime, leie sal) × bydeler (Ålesund sentrum, Spjelkavik, Borgund, ...)
- Snekker: tjenester (kjøkken, bad, terrasse) × kommuner i omkrets

### URL-struktur

```
/[tjeneste]                       → hovedtjeneste-side (generisk)
/[tjeneste]/[sted]                → kombinasjon (sterkest SEO)
/[sted]                           → stedsside som lister alle tjenester der
/[sted]/[tjeneste]                → alternativ rekkefølge (canonical til /tjeneste/sted)
```

Eksempel for budogvare.no:

```
/varebil                          → generell varebil-info
/varebil/lillestrom               → varebil til/fra Lillestrøm
/varebil/jessheim
/varebil/eidsvoll
/kranbil
/kranbil/lillestrom
/kranbil/jessheim
...
/lillestrom                       → alle tjenester vi gjør på Lillestrøm
/jessheim                         → alle tjenester vi gjør på Jessheim
```

### Innholdsmal per kombinasjons-side

Hver `/[tjeneste]/[sted]`-side skal ha **minst 800 ord unikt innhold**, ikke template-tekst med byer-erstatning:

```markdown
# [Tjeneste] i [Sted] — [Kunde]

Hero: «[Tjeneste] til/fra [Sted] med levering på X minutter»
CTA: «Beregn pris» eller «Be om tilbud»

## Hvorfor velge oss for [tjeneste] i [sted]?
- [3-5 punkter, noen sted-spesifikke]

## Prisindikasjon for [tjeneste] [sted]
- Tabell med faktiske startpriser fra/til området

## Vanlige oppdrag i [sted]
- [3-5 case-eksempler — uten persondata]
- Geografiske detaljer (industrigater, kjøpesentre, etc.)

## Slik bestiller du
- 3-stegs forklaring

## Områder vi dekker fra [sted]
- Liste over omkrets-områder (krysslink-strategi)

## Vanlige spørsmål om [tjeneste] i [sted]
- 5+ Q&A (mater FAQPage JSON-LD + LLM-svar)
- «Hvor lang tid tar [tjeneste] i [sted]?»
- «Hva koster [tjeneste] i [sted]?»
- «Hvilke områder kan dere [tjeneste] i?»

## Andre tjenester vi tilbyr i [sted]
- Krysslink til /[andre-tjenester]/[sted]

## Andre steder vi tilbyr [tjeneste]
- Krysslink til /[tjeneste]/[andre-steder]

Footer-CTA + kart med markør i området
```

### Disse sidene er ofte «skjulte» — det er HENSIKTEN

**Viktig:** Sted × tjeneste-sider trenger **ikke** være i hoved-navigasjonen. De er ofte «skjulte» for besøkende som kommer via forsiden, men funn-bare via Google + LLM-søk. Det er hele poenget.

| Brukerflyt | Hvordan finner de siden? |
|------------|--------------------------|
| Direkte besøkende | Klikker meny → tjeneste-hovedside → ev. lenke ned til sted |
| Google-søk «varebil jessheim» | Lander **direkte** på `/varebil/jessheim` (best UX, høyest konvertering) |
| LLM-svar «Hvem leverer varebil-transport i Jessheim?» | LLM siterer fra `/varebil/jessheim` + lenker dit |

**Konsekvenser av at disse er «skjulte» fra hoved-meny:**

- ✓ Hovedmenyen forblir ren (5-7 items, ikke 48)
- ✓ Brukere som kommer fra Google går **direkte** til mest relevante side (høyere konvertering)
- ✓ Kunden trenger ikke selv bygge meny-system for hver kombinasjon
- ✓ Vi kan ha flere sider enn det som er menyvennlig

**De må fortsatt være:**

- ✓ I `sitemap.xml` (Google finner dem)
- ✓ Indexed (ikke `noindex`)
- ✓ Lenket fra footer eller dedikert «Alle områder vi dekker»-side
- ✓ Krysslinket fra tjeneste-side og sted-side (sann hub-spoke)
- ✓ Lenket fra hverandre (relevante andre kombinasjoner under «Andre tjenester i [sted]»)

### Internal linking-arkitektur for skjulte landingssider

Disse sidene trenger en eksplisitt linking-strategi for at Google skal forstå strukturen:

```
Forside (/)
  ↓ lenker til hovedtjenester
/varebil (hovedside)
  ↓ lenker til alle sted-versjoner
/varebil/jessheim ← (kombinasjons-side, skjult fra hoved-meny)
  ↑ lenker oppover til /varebil og /jessheim
  → krysslinker til andre tjenester på samme sted: /kranbil/jessheim, /flakbil/jessheim
  → krysslinker til samme tjeneste på andre steder: /varebil/lillestrom, /varebil/dal
/sitemap (HTML-sitemap, valgfritt men anbefalt)
  ↓ lenker til alle sider inkludert hver kombinasjon
```

### Footer-strategi (uten å fylle hovedmenyen)

```tsx
// components/Footer.tsx
<footer>
  <section>
    <h3>Tjenester</h3>
    <ul>
      {services.map((s) => (
        <li><Link href={`/${s.slug}`}>{s.name}</Link></li>
      ))}
    </ul>
  </section>

  <section>
    <h3>Områder vi dekker</h3>
    <ul>
      {locations.map((l) => (
        <li><Link href={`/${l.slug}`}>{l.name}</Link></li>
      ))}
    </ul>
  </section>

  <section className="text-xs">
    <details>
      <summary>Se alle våre lokale tjenester</summary>
      <ul>
        {services.flatMap((s) =>
          locations.map((l) => (
            <li>
              <Link href={`/${s.slug}/${l.slug}`}>
                {s.name} i {l.name}
              </Link>
            </li>
          ))
        )}
      </ul>
    </details>
  </section>
</footer>
```

Da har Google og LLM-er full oversikt, men menneskelige besøkende ser ikke en overveldende meny.

### Dedikert «Alle områder»-side (anbefalt)

Lag en side `/dekningsomrade` eller `/alle-tjenester` som lister hele grid-en i en pen visualisering:

```
Tjenester vi tilbyr i ditt område

[Søkefelt: skriv postnummer eller stedsnavn]

Lillestrøm
  • Varebil i Lillestrøm
  • Kranbil i Lillestrøm
  • Flakbil i Lillestrøm
  ...

Jessheim
  • Varebil i Jessheim
  ...
```

Denne sida lenker til alle kombinasjoner uten å forurense hovedmenyen.

### Auto-generering vs manuell

For 10/10 SEO:
- **Innhold generert** kan brukes som startpunkt
- **Manuelt redigert** for hver kombinasjon i CMS (krever lokal kontekst)
- AI skal sette opp **datamodellen + admin-flow** slik at kunden kan redigere hver kombinasjon

```sql
-- Datamodell for sted × tjeneste-grid
create table locations (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,            -- 'lillestrom'
  name text not null,                   -- 'Lillestrøm'
  county text,                          -- 'Akershus'
  postal_codes text[],                  -- for autocomplete
  lat decimal(9,6),
  lng decimal(9,6),
  position int default 0,
  is_active boolean default true
);

create table services (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,            -- 'varebil'
  name text not null,                   -- 'Varebil'
  description jsonb,                    -- TipTap JSON
  icon text,                            -- Lucide
  position int default 0
);

create table service_locations (
  id uuid primary key default gen_random_uuid(),
  service_id uuid references services(id) on delete cascade,
  location_id uuid references locations(id) on delete cascade,
  -- Unik innhold per kombinasjon
  hero_headline text,
  hero_subtitle text,
  hero_image_url text,
  intro_text jsonb,                     -- TipTap JSON
  pricing_note text,
  faq jsonb,                            -- [{q, a}]
  case_examples jsonb,
  -- SEO per kombinasjon
  meta_title text,
  meta_description text,
  og_image_url text,
  -- Status
  is_published boolean default false,
  unique (service_id, location_id)
);
```

URL-handler:

```ts
// app/[service]/[location]/page.tsx
// 1. Slå opp service + location fra DB
// 2. Hent service_locations-row hvis finnes
// 3. Hvis ikke published → 404
// 4. Render med unik innhold
```

### Sitemap-strategi

```ts
// app/sitemap.ts
// Auto-generer URL-er for:
// - / (forside)
// - /[hovedside] (alle public-pages med published=true)
// - /[service] for hver published service
// - /[location] for hver published location
// - /[service]/[location] for hver published service_locations
// - /blogg/[slug] for hver published blog_post
// - /[lov-paalagt] (personvern, cookies, vilkar, etc.)
```

### Hvor mange URLs?

For et lokalt firma med:
- 6 tjenester
- 8 områder

Det blir minimum **6 + 8 + 48 + 15 = 77 unike URL-er** med spesifikt innhold. Hvis sitatet ditt sier «10 sider», så har AI ikke skjønt SEO-strategien.

---

## LLM-discovery (ChatGPT, Claude, Perplexity, Google AI)

### `/llms.txt` (root)

```
# [Kunde] — llms.txt

> [Kort beskrivelse av virksomheten — 1-2 setninger]

## Tjenester
- [Tjeneste 1]: https://[domene]/tjeneste1
- [Tjeneste 2]: https://[domene]/tjeneste2
...

## Områder vi dekker
- [Sted 1]: https://[domene]/sted1
...

## Kontakt
- Telefon: ...
- E-post: ...
- Adresse: ...

## Vanlige spørsmål
- Spørsmål? Svar.
...
```

### `/llms-full.txt` (root)

Full markdown-eksport av alle public-sider, kombinert. Genereres automatisk fra `pages` + `service_locations` + `blog_posts`.

### AEO-blokker (Answer Engine Optimization)

På hver sted × tjeneste-side, inkluder en seksjon med kort, sitatbare svar:

```markdown
## Hva koster varebil-transport i Jessheim?

Varebil-transport på Jessheim koster fra kr 850 inkl. mva for opptil 50 kg.
Pris avhenger av vekt, avstand og tidspunkt. Bruk vår [fraktkalkulator](/beregn-pris)
for nøyaktig pris.
```

Korte, faktiske avsnitt som LLM-er kan klippe direkte inn i svaret sitt.

### Spesifikt språk for LLM-discovery

- Skriv «[Vår tjeneste] er …» i stedet for «Vi tilbyr …»
- Inkluder årstall der relevant («I 2026 …»)
- Inkluder nummer / data der mulig («2.500 leveranser i 2025»)
- Bruk konkrete by-navn, ikke «hele Romerike»

---

## Bilde-SEO

- [ ] **Filnavn beskrivende:** `varebil-jessheim-lasting.jpg`, ikke `IMG_1234.jpg`
- [ ] **Alt-tekst** påkrevd ved upload (jf. CMS-krav)
- [ ] **Alt-tekst beskrivende:** «Varebil fra Skedsmo Bud & Vare laster pakker på Jessheim» — ikke «varebil»
- [ ] **Caption** der det legger verdi (vises i Google Image Search)
- [ ] **Filstørrelse** under 200kb før Next.js Image optimaliserer
- [ ] **Eksplisitte dimensjoner** lagret i DB
- [ ] **AVIF + WebP** auto-konvertering
- [ ] **`ImageObject` JSON-LD** for hovedbilder med `caption` + `creator`
- [ ] **Geolocation** i EXIF for lokale bilder (Google Image Search foretrekker dette)

---

## Performance for SEO

Google bruker Core Web Vitals som ranking-signal:

| Metrikk | Mål | Hvorfor for SEO |
|---------|-----|-----------------|
| LCP | < 2.5s p75 | Page Experience-signal |
| INP | < 200ms p75 | Page Experience-signal |
| CLS | < 0.1 p75 | Page Experience-signal |
| Mobile-friendly | ✓ | Mobile-first indexing |
| HTTPS | ✓ | Tillit-signal |
| Structured data valid | ✓ | Rich snippet-tilgang |

Sjekk hver kombinasjons-side i Lighthouse + Search Console.

---

## Internal linking (kritisk for crawl-budsjett)

For et stort kombinasjons-grid, må internal linking være systematisk:

- Hver tjeneste-side lenker til ALLE sted-versjoner av samme tjeneste
- Hver sted-side lenker til ALLE tjeneste-versjoner på samme sted
- Forside lenker til topp-5 tjenester + topp-3 områder
- Footer har full liste over alle hovedtjenester + alle områder
- Brødsmuler på hver dyp-side

Bygg en `Sitemap`-komponent i admin som visualiserer link-strukturen.

---

## Sjekkliste for AI i PROJECT.md

Seksjon 13 (SEO + AEO + LLM-søk) MÅ inneholde:

- [ ] **Komplett URL-grid** for kombinasjons-sider (sted × tjeneste) hvis lokal virksomhet
- [ ] **JSON-LD per side-type** med fullstendige eksempler (ikke bare type-navn)
- [ ] **OG image-strategi** med per-side generering
- [ ] **Datamodell** for locations + services + service_locations (eller tilsvarende)
- [ ] **`/sitemap.xml`** auto-generering fra alle published rows
- [ ] **`/robots.txt`** + `/llms.txt` + `/llms-full.txt` spesifikasjon
- [ ] **AEO-blokker** på alle service-sider med direct-quote-format
- [ ] **Bilde-SEO**: filnavn, alt-tekst, caption, dimensjoner, AVIF/WebP
- [ ] **Internal linking-strategi** dokumentert
- [ ] **Per side: redigerbar SEO i CMS** (title, description, OG image, canonical, noindex)
- [ ] **Lighthouse SEO ≥ 95** på alle hovedsider + alle kombinasjons-sider
---

## Verifisering: S-kategori (5 sjekkpunkter per side i sted×tjeneste-grid)

> Container Consult AS (mai 2026): Helge åpnet `/avfallscontainer/alesund` og fant en TOM stub — brødsmuler, H1, CTA, kalkulator. Ingen 800 ord innhold, ingen JSON-LD, ingen AEO-blokker, ingen unik tekst. Sprint 6 ble rapportert ferdig. Helge: «TERNINGKAST 1 PÅ ALT». Hele SEO-formålet med 72 sider gikk tapt. **Aldri igjen — låses inn i rubrikken.**

`scripts/check-seo-grid.ts` åpner ALLE sted×tjeneste-URL-er og scorer dem mot 5 obligatoriske krav:

### S1: Min 800 ord faktisk innhold

Tell ord i `<main>` ekskludert nav/footer/skripter. Trim, fjern dobbelt-mellomrom, count token. Krav: ≥800.

```ts
const wordCount = await page.evaluate(() => {
  const main = document.querySelector('main')
  if (!main) return 0
  const text = main.textContent ?? ''
  return text.trim().split(/\s+/).filter(Boolean).length
})
// S1: wordCount >= 800
```

### S2: JSON-LD `Service` + `LocalBusiness` validerer

Hver kombinasjon-side MÅ ha minst:
- `Service`-objekt med `name`, `provider` (Organization), `areaServed` (Place), `description`
- `LocalBusiness`-objekt med `name`, `address`, `geo`, `telephone`, `openingHours`

Begge må passere schema.org validator.

```ts
const jsonLdScripts = await page.locator('script[type="application/ld+json"]').allTextContents()
const types = jsonLdScripts.flatMap((s) => {
  try {
    const parsed = JSON.parse(s)
    return Array.isArray(parsed) ? parsed.map((p) => p['@type']) : [parsed['@type']]
  } catch { return [] }
})
// S2: types.includes('Service') && types.includes('LocalBusiness')
```

### S3: Unikt innhold (ikke duplikat av andre sider)

Sammenlign side X med ALLE andre kombinasjon-sider. Bruk fuzzy match på paragraf-tekst. Maks 60% overlapp tillatt — over det er det template-stuffing.

```ts
// Pseudo: hentet allPageBodies for alle 72 sider
const thisBody = stripHtmlAndNormalize(html)
const overlaps = otherBodies.map((other) => jaccardSimilarity(thisBody, other))
const maxOverlap = Math.max(...overlaps)
// S3: maxOverlap < 0.6
```

### S4: AEO-blokker (FAQ + svar i 2-3 setninger)

Hver kombinasjon-side MÅ ha minst 3 FAQ-spørsmål med direkte-siterbare svar (LLM-vennlig format).

```ts
const faqCount = await page.locator('[itemtype*="FAQPage"] [itemtype*="Question"], section.faq dt, section.faq h3').count()
// S4: faqCount >= 3
```

### S5: Internal linking til ≥3 andre kombinasjons-sider

Hver side MÅ lenke til minst 3 andre `/[tjeneste]/[sted]`-URL-er (krysslinks for SEO-link-flyt).

```ts
const crossLinks = await page.locator('a[href*="/avfallscontainer/"], a[href*="/kraning/"], ...').count()
// (Bruk regex som matcher alle tjeneste-slugs)
// S5: crossLinks >= 3 (eks. "Andre tjenester i [sted]" + "Andre steder vi tilbyr [tjeneste]")
```

### Rapport-output

```
🔍 SEO-grid-verifisering — 72 sider

✅ /avfallscontainer/alesund  S1:✓ S2:✓ S3:✓ S4:✓ S5:✓ (5/5)
❌ /avfallscontainer/molde    S1:✗ (320 ord) S2:✓ S3:✗ (78% overlap m/alesund) S4:✗ (0 FAQ) S5:✗ (1 cross-link) (1/5)
❌ /kraning/alesund           S1:✗ (180 ord) S2:✗ (mangler Service) S3:✓ S4:✗ S5:✓ (2/5)
...

Snitt: 2.1/5  ·  Sider <4/5: 65/72
❌ SEO-grid FAILED — sprint ikke ferdig
```

CI-jobb i `.github/workflows/quality-gate.yml` blokker PR hvis snitt <4.5.

---

## CMS-felt for unik innhold per kombinasjon

For at AI ikke skal generere duplikat-tekst, krever vi `service_locations`-tabellen utvidet:

```sql
alter table public.service_locations
  add column if not exists hero_intro text,            -- 60-100 ord, unik per kombinasjon
  add column if not exists why_us_section jsonb,        -- {title, bullets[]}
  add column if not exists price_indication text,       -- 80-150 ord
  add column if not exists common_jobs_section text,    -- 150-200 ord, lokale eksempler
  add column if not exists booking_steps_section text,  -- 80-120 ord
  add column if not exists coverage_areas text,         -- 100-150 ord
  add column if not exists faq_items jsonb,             -- [{q, a}] minst 5
  add column if not exists other_services_html text,    -- internal-links seksjon
  add column if not exists other_locations_html text;   -- internal-links seksjon
```

Bootstrap-skript fyller disse via Claude API tool-use (genererer unikt innhold per kombinasjon basert på scraped data + service-beskrivelse + sted-info). Kunden redigerer i admin etterpå.

---

## For AI

Når PROJECT.md spesifiserer sted×tjeneste-grid (typisk lokal virksomhet):

1. **Ikke bare bygg URL-strukturen — bygg INNHOLDET.** Hver kombinasjon MÅ ha ≥800 ord faktisk unikt innhold i DB.
2. **Generer innholdet via Claude API tool-use** med scraped data + tjeneste-beskrivelse + sted-fakta som kontekst. Ikke template med byer-erstatning.
3. **JSON-LD `Service` + `LocalBusiness`** på HVER kombinasjon, validert mot schema.org.
4. **Min 3 FAQ-blokker** med direkte-siterbare svar (LLM-AEO).
5. **Internal linking**: minst 3 cross-links til andre kombinasjon-sider.
6. **Verifiser med `check-seo-grid.ts` FØR sprint rapporteres ferdig.** Snitt <4.5 = sprint ikke ferdig.
7. **Sprint som leverer SEO-grid har S1-S5 som hardt blokker.** Kan ikke gå til pre-launch uten.
