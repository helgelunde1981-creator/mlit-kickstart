# Section Detail — Hardkoding-forbud (CMS-drevet innhold)

> Hver hardkodet liste av sider, tjenester, menypunkter eller bilde-koblinger i kode er en kritisk leveransefeil. Kunden MÅ kunne legge til, endre og slette CMS-drevet innhold uten å åpne kode-editor. Container Consult AS (mai 2026): AI lagde `KEYWORDS: Record<slug, string[]>` i `link-images-to-pages.ts` for å koble bilder til sider — så en ny tjeneste lagt til via admin ville ikke fått bilde uten at en utvikler oppdaterte skriptet. **Aldri igjen.**

---

## Hovedregel

Kunden skal kunne gjøre disse handlingene i admin uten å be om utvikler-hjelp:

| Handling | Hvor det skjer | Hva som må fungere automatisk |
|---|---|---|
| Opprette ny side | `pages`-tabellen (admin → ny side) | Vises i meny, footer, sitemap, har redigerbar SEO + seksjoner |
| Slette/skjule side | `is_published = false` eller `show_in_nav = false` | Forsvinner fra meny + sitemap, 404 håndteres |
| Endre meny-rekkefølge | `pages.nav_order` (drag-drop i admin) | Header oppdateres ferskt |
| Opprette ny tjeneste | `services`-tabellen | Vises i «Tjenester»-dropdown, evt egen dynamisk side `/[tjeneste-slug]` |
| Bytte hero-bilde | `page_sections.data.image_id` (mediavelger) | Bildet vises uten å kalle utvikler |
| Endre tekst i hero | `page_sections.data.headline/subline` (rich-text-editor) | Vises ferskt |
| Bytte mengde bilder i galleri | `page_sections.data.image_ids[]` | Galleri reflekterer endringen |
| Legge til/endre tjenestested | `service_locations`-tabellen | sted×tjeneste-grid genererer ny URL |

**Hvis en av disse krever kode-endring → leveransefeil. Send PR for å fikse data-modellen, ikke send «opplæring» til kunden.**

---

## Anti-patterns (forbudte mønstre)

### ❌ Hardkodet slug → konfigurasjon

```ts
// FORBUDT
const HERO_IMAGE_KEYWORDS: Record<string, string[]> = {
  "/": ["container", "scania"],
  "/om-oss": ["container", "bil"],
  "/avfallshandtering": ["avfall", "10fot"],
}

const HERO_TITLES: Record<string, string> = {
  "/": "Velkommen til Container Consult",
  "/kontakt": "Kontakt oss",
}

function getServiceColor(slug: string) {
  if (slug === "/kraning") return "yellow"
  if (slug === "/lagercontainer") return "blue"
  return "gray"
}
```

**Hvorfor det er feil:** Ny tjeneste lagt til i CMS → vises i meny → klikker på sida → tomt hero. Krever kode-endring. Bryter kjernen i et CMS-prosjekt.

### ❌ Hardkodet meny / footer

```tsx
// FORBUDT
<nav>
  <Link href="/tjenester">Tjenester</Link>
  <Link href="/om-oss">Om oss</Link>
  <Link href="/kontakt">Kontakt</Link>
</nav>

// FORBUDT
const FOOTER_LINKS = [
  { href: "/personvern", label: "Personvern" },
  { href: "/vilkar", label: "Vilkår" },
]
```

**Hvorfor det er feil:** Ny side lagt til i CMS → må manuelt redigeres inn i meny/footer-komponenten.

### ❌ Hardkodet liste av tjenester / sted-grid / kategorier

```ts
// FORBUDT
const SERVICES = ["Avfallscontainer", "Kraning", "Maskintransport"]
const LOCATIONS = ["Ålesund", "Molde", "Kristiansund"]
```

**Hvorfor det er feil:** Kunden har en CRUD i admin men det reflekteres ikke i koden — to sannheter.

### ❌ Side-spesifikk logikk i komponenter

```tsx
// FORBUDT
function HeroSection({ slug }: { slug: string }) {
  if (slug === "/jubileum") {
    return <JubileumSpecialHero />
  }
  return <DefaultHero />
}
```

**Hvorfor det er feil:** Spesial-rendering er kode-koblet til en spesifikk slug. Hvis slug endres, må kode endres. Bedre: `page_sections.type === 'jubileum_hero'` med matchende renderer.

### ❌ Seed-skript som hardkoder hva som seedes

```ts
// FORBUDT (Container Consult Sprint 11.5-eksempel)
const KEYWORDS: Record<string, string[]> = {
  "/om-oss": ["container", "bil"],
  // 10+ flere linjer...
}
```

**Hvorfor det er feil:** Seed er en engangs-jobb, men hardkodingen overlever som permanent kobling. Hver gang noen sletter eller endrer en slug må mappingen oppdateres.

---

## Korrekte mønstre

### ✅ Bilde-binding via CMS-felt

**Datamodell:**

```sql
-- page_sections.data jsonb:
-- hero:    { headline, subline, image_id, image_url, image_alt, cta_label, cta_href }
-- gallery: { title, image_ids[], image_urls[], image_alts[] }
```

**Admin-UI:** «Velg fra mediabibliotek»-modal-komponent. Drag-drop fra Storage. Aldri URL-input.

**Rendering:**

```tsx
function HeroBlock({ data }: { data: HeroData }) {
  if (!data.image_id) return <TextOnlyHero {...data} />
  return <ImageHero src={data.image_url} alt={data.image_alt} {...data} />
}
```

### ✅ Førstegangs-seed via DATA-drevet heuristikk

```ts
// scripts/link-images-to-pages.ts — engangsjobb. Ingen hardkodede slug → keyword maps.
const allMedia = await supabase.from("media").select("id, filename, alt_text")
const allPages = await supabase
  .from("pages")
  .select("id, slug, title, meta_description")
  .eq("is_published", true)

function scoreMatch(page: Page, media: Media): number {
  const haystack = `${page.title} ${page.slug} ${page.meta_description ?? ""}`.toLowerCase()
  const needle = `${media.filename} ${media.alt_text ?? ""}`.toLowerCase()
  // Token-overlap score (Jaccard / TF-IDF / embedding cosine — velg én)
  return jaccardOverlap(tokenize(haystack), tokenize(needle))
}

for (const page of allPages) {
  const ranked = allMedia
    .map((m) => ({ media: m, score: scoreMatch(page, m) }))
    .sort((a, b) => b.score - a.score)

  const best = ranked[0]
  if (best.score > MIN_SCORE) {
    await upsertHeroImage(page.id, best.media)
    console.log(`✓ ${page.slug} → ${best.media.filename} (score ${best.score.toFixed(2)})`)
  } else {
    console.log(`⏭  ${page.slug} — ingen god match (best ${best.score.toFixed(2)}), kunden velger i admin`)
  }
}
```

**Etter første kjøring:** kunden tar over via admin-mediavelger. Skriptet skal IKKE re-kjøres uten `--reset`-flagg (for å unngå at det overskriver kunde-redigeringer).

For prosjekter med rik scraped-data: bruk Claude API som scorer (les scraped innhold per side + filnavn → returner bestemte par via tool-use). Resultat er ENGANGS-bilde-tildeling, ikke kode-mapping.

### ✅ Meny + tjenester drives av tabeller

```tsx
// MainNav.tsx
const { data: navPages } = await supabase
  .from("pages")
  .select("slug, nav_label, nav_order")
  .eq("is_published", true)
  .eq("show_in_nav", true)
  .order("nav_order")

const { data: services } = await supabase
  .from("services")
  .select("slug, name")
  .eq("is_active", true)
  .order("sort_order")
```

Ny side i admin → vises i meny på neste request (krever `force-dynamic` — se `cms-requirements.md`).

### ✅ Footer-lenker via egen tabell

For footer-lenker som ikke matcher pages-tabellen (juridisk, sosiale medier, ekstern info):

```sql
create table public.footer_items (
  id uuid primary key default gen_random_uuid(),
  column_id text not null,        -- 'company' | 'services' | 'legal' | 'social'
  label text not null,
  href text not null,
  sort_order integer default 100,
  is_external boolean default false
);
```

Admin-CRUD per kolonne. Footer-komponenten itererer.

### ✅ Seksjons-typer i stedet for slug-spesifikk komponent

```sql
-- page_sections.type CHECK-constraint utvides ved behov
type IN ('hero', 'text_block', 'gallery', 'services', 'cta', 'jubileum_hero', ...)
```

Hver type har sin renderer:

```tsx
function SectionRenderer({ section }: { section: PageSection }) {
  switch (section.type) {
    case "hero":          return <HeroBlock data={section.data} />
    case "jubileum_hero": return <JubileumHero data={section.data} />
    case "gallery":       return <Gallery data={section.data} />
    // ...
  }
}
```

Kunden velger seksjons-type fra dropdown i admin. Ny type krever migrasjon + renderer — men det er feature-arbeid, ikke side-spesifikk hardkoding.

---

## Tillatt hardkoding (unntak)

Kort liste over hva som faktisk MÅ være hardkodet i kode:

- **Design-tokens** (`tokens.css`) — palette, fonter, spacing
- **Komponent-arkitektur** — Header/Footer/Layout-struktur
- **Tekniske ruter** — `/api/health`, `/sitemap.xml`, `/robots.txt`-generatorer
- **404-fallback-tekst** (kun hvis ikke i CMS)
- **System-ikoner** (lucide-react osv.)
- **Juridiske disclaimers** som krever advokat-review før endring (markeres med `// LEGAL: ikke endre uten review`)
- **«Utviklet av Myrvoll-Lunde IT Drift»**-footer-credit

Alt annet er CMS-drevet eller en bug.

---

## Verifisering

### Grep-script (kjøres av check-public-pages.ts som J5-sjekk)

```bash
# scripts/check-no-hardcoding.sh
set -e

# Slug → konfig-mappings
if grep -rnE '^\s*"/[a-z][a-z0-9-]*"\s*:\s*' src/ --include='*.ts' --include='*.tsx' \
    --exclude-dir=node_modules; then
  echo "❌ Hardkodet slug → konfig-mapping funnet (se over)"
  exit 1
fi

# if/switch (slug)
if grep -rnE 'slug\s*===\s*"/[a-z]' src/ --include='*.ts' --include='*.tsx' \
    --exclude-dir=node_modules; then
  echo "❌ slug === '/...' funnet — bruk page_sections.type i stedet"
  exit 1
fi

# Hardkodet menyelementer (Link href="/...")
HARDCODED_NAV=$(grep -rnE '<Link[^>]*href="/(om-oss|kontakt|tjenester|faq|nyheter)"' \
  src/components/layout/ 2>/dev/null | wc -l)
if [ "$HARDCODED_NAV" -gt 0 ]; then
  echo "❌ Hardkodet meny-Link i layout-komponent — bruk pages-tabellen"
  exit 1
fi

echo "✅ Ingen hardkodet CMS-innhold funnet"
```

CI-jobben kjører dette på hver PR. Rødt = blokk.

### Manuell sanity-test (kunde-perspektivet)

Etter Sprint X som påvirker CMS:

1. Logg inn i admin
2. Lag en helt ny side (slug: `/test-ny-side`, tittel: «Test»)
3. Hak av `show_in_nav = true`
4. Forfrisk forsiden — verifiser at «Test» kommer i meny
5. Klikk på den — verifiser at sida laster (404 hvis ingen route, OK hvis dynamic `[slug]`)
6. Slett sida i admin → forfrisk forsiden — verifiser at den forsvinner fra meny

Hvis noen av stegene krever kode-endring: leveransen er ufullført.

---

## For AI

Når du genererer eller endrer kode på et Myrvoll-Lunde IT Drift-prosjekt:

1. **Sjekk først om dette er CMS-drevet innhold.** Hvis ja: bruk tabell + Server Component-fetch. Ikke kode-konstant.
2. **Tekst, bilder, lenker, lister, rekkefølger, kategorier** er ALLTID CMS-drevet på 10/10-prosjekter.
3. **Engangs-seed-skript er OK** — men de skal generere DB-data, ikke etterlate hardkodede mappings i kode som overlever.
4. **Hvis du blir fristet til å skrive `Record<slug, ...>` eller `if (slug === '...')`** — stopp. Dette er nesten alltid en data-modell som mangler et felt. Send forslag til migrasjon i stedet.
5. **`check-no-hardcoding.sh` SKAL grep CI-blokke** disse mønstrene. Rødt = sprint ikke ferdig.
