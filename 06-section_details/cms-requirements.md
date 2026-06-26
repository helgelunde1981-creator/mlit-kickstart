# Section Detail — CMS-krav (alt redigerbart, 10/10 UX)

> **Brukes når prosjektet inkluderer noe public-innhold som kunden skal kunne endre selv.** AI skal ikke levere et CMS som «delvis kan redigere noe» — det blir aldri brukt. Enten leverer vi 10/10 UX som dekker ALT, eller så er det ikke et CMS.

---

## Hovedregel

**ALL tekst, alle bilder, alle lenker, alle SEO-felter, alle CTA-er, alle prislister, alle åpningstider, alt vist på public-siden MÅ kunne redigeres i CMS av en ikke-teknisk bruker.**

Hvis noe er hardkodet i koden og ikke endrebart fra CMS — det er en bug, ikke en feature.

Den eneste lovlige unntak: tekniske ting som routes-strukturen, footer-credit «Utviklet av Myrvoll-Lunde IT Drift» (jf. regel #11), juridiske disclaimers som krever utvikler-review.

## Hva «alt redigerbart» betyr konkret

For HVER side på public-siden må CMS-en ha redigerings-tilgang til:

### Tekstinnhold
- [ ] Hero-headline (h1)
- [ ] Hero-subtittel
- [ ] Hero-CTA-tekst + lenke
- [ ] Alle h2/h3-overskrifter
- [ ] All brødtekst (rich-text editor)
- [ ] Lister, anførselstegn, lenker innenfor brødtekst
- [ ] Knapper i alle seksjoner (tekst + lenke + variant)
- [ ] Footer-tekst (kontaktinfo, sosiale medier, åpningstider)

### Bilder og media
- [ ] **Opplastingsmodul** (drag-and-drop + multi-file) — IKKE URL-felt
- [ ] Alt-tekst påkrevd ved upload (validert)
- [ ] Bildetittel + caption (valgfritt)
- [ ] Beskjæring/crop direkte i nettleseren før save
- [ ] Auto-konvertering til AVIF + WebP
- [ ] Auto-generering av thumbnail (per breakpoint)
- [ ] Eksplisitte dimensjoner lagret i DB (for CLS=0)
- [ ] Mediabibliotek med søk på filnavn og alt-tekst
- [ ] Mappestruktur (eks: «hero/», «produkter/», «team/»)
- [ ] Filtre på dato, type, brukt-på-side
- [ ] Erstatt-bilde-flyt (ny upload eller velg fra bibliotek)
- [ ] «Hvor er dette bildet brukt?»-info
- [ ] Bulk-sletting med confirm

### Seksjoner og layout
- [ ] Legg til ny seksjon (av definerte typer: hero, image-text, gallery, cta, faq, testimonials, etc.)
- [ ] Slett seksjon (med soft-delete + recovery)
- [ ] Drag-and-drop reordering
- [ ] Vis/skjul seksjon uten å slette
- [ ] Endre seksjon-type uten å miste innhold

### SEO + Open Graph
- [ ] Meta title per side (overstyrer default fra site-config)
- [ ] Meta description per side
- [ ] OG image per side (med generator hvis ikke valgt)
- [ ] OG title overstyring
- [ ] OG description overstyring
- [ ] Twitter Card-type (summary / large_image)
- [ ] Canonical URL overstyring (default = self)
- [ ] noindex/nofollow flag per side
- [ ] Structured data (JSON-LD) — auto-generert basert på side-type

### Site-config (global)
- [ ] Brand: logo (light + dark), favicon, app icons (16/32/180/192/512)
- [ ] Primær + sekundær accent-farge
- [ ] Header-meny (drag-drop nav-items, multi-level)
- [ ] Footer-kolonner (overskrifter + lenker)
- [ ] Sosiale medier (icon + URL + rekkefølge)
- [ ] Kontaktinfo (e-post, telefon, adresse, åpningstider)
- [ ] GA4/PostHog/Plausible-IDer
- [ ] Cookie-policy-tekst
- [ ] Analytics consent-tekster

### Datadrevne lister
For prosjekter med tabeller (priser, biler, tjenester, ansatte, etc.):
- [ ] Legg til/redigér/slett rader (CRUD)
- [ ] Sorterbar tabell
- [ ] Bulk-import via CSV
- [ ] Bulk-eksport
- [ ] Synkronisert med public-side umiddelbart (eller med «publiser»-knapp)

### Blogg / nyheter (hvis aktuelt)
- [ ] Rich-text editor (TipTap) med h2/h3, bold/italic, lister, lenker, bilder, embed (YouTube/Vimeo)
- [ ] Coverbilde med opplastingsmodul
- [ ] Slug auto-generert + redigerbar
- [ ] Tags / kategorier
- [ ] Forfatter (velg fra brukere)
- [ ] Publiseringsstatus (draft/published/scheduled)
- [ ] Schedulert publisering (dato + klokkeslett)
- [ ] SEO per post
- [ ] Forhåndsvisning av post FØR publisering

## 10/10 UX-krav

### Forhåndsvisning før lagring
- [ ] **Live preview** mens du redigerer (split-screen eller toggle)
- [ ] Preview viser EKSAKT slik public-siden vil se ut
- [ ] Preview oppdateres innen 500ms etter input

### Auto-save
- [ ] Endringer auto-lagres som draft hvert 5. sekund eller ved blur
- [ ] «Lagrer …» / «Lagret kl 14:32» indikator alltid synlig
- [ ] Aldri miste innhold ved cras eller stengt fane
- [ ] Recovery: «Du har en ulagret kladd fra X minutter siden — gjenopprett?»

### Publiseringsflyt
- [ ] Draft → Review (valgfritt) → Published
- [ ] «Forhåndsvis upublisert»-knapp på public-siden
- [ ] «Tilbakerull til versjon X»
- [ ] Schedulert publisering (publiseres automatisk på gitt dato)
- [ ] Diff-view: hva endret seg siden forrige versjon

### Versjonering
- [ ] Alle endringer lagres som revisjon
- [ ] Liste over alle revisjoner per side / post
- [ ] Rollback til hvilken som helst revisjon
- [ ] Hvem endret + når + diff

### Roller og tilganger
- [ ] **Superadmin**: alt
- [ ] **Editor**: redigere innhold + media, ikke endre struktur eller brukere
- [ ] **Viewer**: lese + kommentere (valgfritt)
- [ ] Tydelig indikasjon på hva din rolle kan / ikke kan

### Audit log
- [ ] Hver endring logges: hvem + når + hva (entity, action, before/after)
- [ ] Søkbar audit-log for superadmin
- [ ] Eksport til CSV

### Onboarding
- [ ] Førstegangs-bruker får en 1-minutt guidet tur
- [ ] Hver redigerings-side har en «hjelp»-ikon med tooltip
- [ ] Tomme states har klar CTA («Last opp ditt første bilde»)

### Mobile / responsive admin
- [ ] CMS fungerer like bra på iPad og 13"+ skjerm
- [ ] Mobile (telefon): minimum read + status-endring (godkjenne posts, svare på meldinger)

### Performance
- [ ] Bilde-opplasting: progress bar + thumbnail-preview umiddelbart
- [ ] Save: aldri blokkerer UI > 200ms (optimistisk update + revert ved feil)
- [ ] Liste-sider: virtualisering hvis > 100 rader
- [ ] Søk: debounced 200ms, instant feedback

### Tilbakemeldinger
- [ ] Toast-notification ved hver save / publish / error
- [ ] Loading-spinner kun hvis > 500ms
- [ ] Confirm-dialoger for destruktive actions («Er du sikker?»)
- [ ] Aldri silent failure — alle feil får synlig melding

## Implementasjons-pattern

### Datamodell

For sider med strukturerte seksjoner (forside, om-oss, etc.):

```sql
create table pages (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,
  title text not null,
  status text default 'draft' check (status in ('draft','published','archived')),
  meta_title text,
  meta_description text,
  og_image_url text,
  noindex boolean default false,
  -- ...
);

create table page_sections (
  id uuid primary key default gen_random_uuid(),
  page_id uuid references pages(id) on delete cascade,
  position int not null,
  section_type text not null,   -- 'hero', 'image_text', 'gallery', etc.
  data jsonb not null,          -- type-spesifikk innhold
  is_visible boolean default true
);

create table page_revisions (
  id uuid primary key default gen_random_uuid(),
  page_id uuid references pages(id) on delete cascade,
  snapshot jsonb not null,      -- hele sidens state
  author_id uuid references profiles(id),
  created_at timestamptz default now()
);

create table media (
  id uuid primary key default gen_random_uuid(),
  storage_path text not null unique,
  url text not null,            -- CDN URL
  filename text not null,
  mime_type text not null,
  size_bytes bigint,
  width int,                    -- påkrevd
  height int,                   -- påkrevd
  alt_text text not null,       -- PÅKREVD ved insert
  caption text,
  folder text,                  -- mappestruktur
  uploaded_by uuid references profiles(id),
  used_in jsonb,                -- {pages: ['hero-1'], posts: ['post-2']} — auto-oppdatert
  created_at timestamptz default now()
);
```

### Image upload-flyt

```tsx
// components/admin/ImageUpload.tsx
// 1. Drag-and-drop (react-dropzone)
// 2. Multi-file
// 3. Validering: maks-størrelse, mime-type
// 4. Crop/resize i nettleseren (react-image-crop)
// 5. Auto-konvertering til AVIF + WebP via sharp på server
// 6. Progress per fil
// 7. Alt-tekst-input PÅKREVD per fil
// 8. På save: opprett media-rad + last opp til Supabase Storage
// 9. Returner media.id til parent

interface UploadResult {
  id: string
  url: string
  width: number
  height: number
  alt_text: string
}

function ImageUpload({ onUploaded, folder, maxFiles = 1 }: Props) {
  // ...
}
```

### Rich-text editor (TipTap)

```tsx
// Lagrer som strukturert JSON i DB (ikke HTML)
// Renderes server-side via custom RSC-renderer for SEO + ytelse
// Editor: H2/H3, bold, italic, lenke, lister, bilde (via ImageUpload), embed
// Aldri inline-style — kun semantiske marks
```

### Seksjon-system

```tsx
// Hver seksjon-type har:
// 1. En Zod-skjema for data
// 2. En editor-komponent (lever inputs + ImageUpload)
// 3. En render-komponent (public-side)
// 4. Et thumbnail (vises i «legg til seksjon»-dialog)

// SectionTypeRegistry:
const SECTIONS = {
  hero: { schema: HeroSchema, Editor: HeroEditor, Render: HeroRender },
  image_text: { ... },
  gallery: { ... },
  cta: { ... },
  faq: { ... },
}
```

### Live preview

```tsx
// Bruk shadcn Resizable Panel:
// Venstre: editor (form med alle felter)
// Høyre: preview iframe som loader /preview/[id] med draft-data
// På input-endring: debounce 300ms, send oppdatert state til iframe via postMessage
// Preview-route bruker draft-data hvis ?draft=true
```

## Anti-patterns vi avviser

- ❌ Bilde-URL-input («lim inn URL til bildet ditt»)
- ❌ «Endre dette i koden» for noen public-tekst
- ❌ Statiske mockups i CMS som ikke matcher faktisk side
- ❌ «Lagre»-knapp som tar > 1 sek (ingen feedback)
- ❌ Mister innhold ved navigering uten advarsel
- ❌ Tekstfelt der det burde vært rich-text
- ❌ Rich-text der det burde vært enkel input
- ❌ Endring som krever full refresh av siden for å se
- ❌ Endring som krever utvikler-deploy for å gå i prod
- ❌ Mediabibliotek uten søk
- ❌ Liste-sider med > 50 rader uten paginering eller virtualisering
- ❌ Sletting uten confirm
- ❌ Audit-log fraværende

## Anti-hardkoding-mandate (kritisk — AI klarer ikke dette uten eksplisitt krav)

**Problemet:** AI bygger gjentatte ganger CMS som "halvveis dekker" innholdet, og resten er hardkodet i komponenter. Når kunden tar over, oppdager de at de ikke kan endre overskriften, ikke kan bytte hero-bilde, ikke kan oppdatere åpningstider osv.

**Løsningen:** Per side i §11 (Sider og URL-struktur), AI MÅ produsere en **CMS-coverage-tabell** som lister HVER tekstklynge, HVERT bilde, HVER lenke, HVER CTA — og dokumenterer presis hvilken DB-rad/felt det kommer fra. Hvis et element finnes på siden men ikke i CMS-mappingen → det er hardkodet → det er en bug.

### Obligatorisk CMS-coverage tabell per side

For HVER public-side i prosjektet, skriv en tabell som denne — den er **ikke valgfri**:

```markdown
### CMS-coverage for `/` (Forside)

| Element på siden | Type | DB-kilde (tabell.felt eller seksjon) | Redigeringsplassering | Status |
|------------------|------|--------------------------------------|----------------------|--------|
| Hero overskrift | Tekst | `page_sections[type=hero].data.headline` | /admin/sider/forside → Hero | ✓ CMS |
| Hero underoverskrift | Tekst | `page_sections[type=hero].data.subtitle` | /admin/sider/forside → Hero | ✓ CMS |
| Hero CTA-tekst | Tekst | `page_sections[type=hero].data.cta_label` | /admin/sider/forside → Hero | ✓ CMS |
| Hero CTA-lenke | URL | `page_sections[type=hero].data.cta_href` | /admin/sider/forside → Hero | ✓ CMS |
| Hero-bilde | Bilde | `media[id=...]` via `page_sections[type=hero].data.image_id` | /admin/sider/forside → Hero (opplastingsmodul) | ✓ CMS |
| Hero-bilde alt-tekst | Tekst | `media[id=...].alt_text` | Opplastingsmodul (påkrevd) | ✓ CMS |
| USP-strip overskrift | Tekst | `page_sections[type=usps].data.headline` | /admin/sider/forside → USPer | ✓ CMS |
| USP-strip items[] | Liste | `page_sections[type=usps].data.items` (icon + tekst per item) | /admin/sider/forside → USPer | ✓ CMS |
| Service-grid items[] | Liste | `services` (slug + name + icon + description) | /admin/tjenester | ✓ CMS |
| Service-grid CTA | Tekst+URL | `site_settings[key='services_cta']` | /admin/innstillinger | ✓ CMS |
| Testimonial-strip items[] | Liste | `testimonials` (person + tekst + image) | /admin/anbefalinger | ✓ CMS |
| Kontakt-CTA-tekst | Tekst | `page_sections[type=contact_cta].data.headline` | /admin/sider/forside | ✓ CMS |
| Kontakt-CTA-knapp | Tekst+URL | `page_sections[type=contact_cta].data.cta_label/href` | /admin/sider/forside | ✓ CMS |
| Footer-kolonne overskrifter | Tekst | `site_settings[key='footer_columns']` | /admin/innstillinger → Footer | ✓ CMS |
| Footer-lenker | Liste | `site_settings[key='footer_columns'].links` | /admin/innstillinger → Footer | ✓ CMS |
| Sosiale medier (icon + URL) | Liste | `site_settings[key='social_media']` | /admin/innstillinger → SoMe | ✓ CMS |
| Kontaktinfo (tlf/epost/adresse) | Tekst | `site_settings[key='contact_info']` | /admin/innstillinger → Kontakt | ✓ CMS |
| Åpningstider | Liste | `site_settings[key='opening_hours']` | /admin/innstillinger → Kontakt | ✓ CMS |
| Logo (light + dark) | Bilde | `site_settings[key='brand_logo']` | /admin/innstillinger → Brand (opplastingsmodul) | ✓ CMS |
| Favicon | Bilde | `site_settings[key='favicon']` | /admin/innstillinger → Brand (opplastingsmodul) | ✓ CMS |
| Meta title | Tekst | `pages[slug='/'].meta_title` | /admin/sider/forside → SEO | ✓ CMS |
| Meta description | Tekst | `pages[slug='/'].meta_description` | /admin/sider/forside → SEO | ✓ CMS |
| OG image | Bilde | `pages[slug='/'].og_image_url` | /admin/sider/forside → SEO (opplastingsmodul) | ✓ CMS |
| Footer-credit «Utviklet av Myrvoll-Lunde IT Drift» | Tekst | (HARDKODET — jf. regel #11 eneste unntak) | — | ✗ Hardkodet (lovlig) |
```

**Krav:**
- Hver tekstklynge på siden = én rad
- Hvert bilde på siden = én rad
- Hver lenke / URL på siden = én rad
- Hver CTA-knapp på siden = to rader (tekst + URL)
- Hver liste / items[] = én rad som dokumenterer items-strukturen
- Hver SEO-felt (title, description, OG image) = én rad

**Tillatte ✗-statuser (kun disse):**
- Footer-credit «Utviklet av Myrvoll-Lunde IT Drift» (regel #11)
- Juridiske disclaimers som krever utvikler-review
- Tekniske routes-strukturen

Alt annet skal være ✓ CMS.

### Verifisering — anti-hardkoding-grep

Lag en sjekk-script som leter etter hardkodede strenger i public-komponenter:

```bash
# scripts/check-cms-coverage.ts
# Grep gjennom src/app/(public)/**/*.tsx for hardkodede strenger
# Whitelist: Tailwind-klasser, design-tokens, route-paths, dev-tools-strenger
# Output: liste over potensielt hardkodet tekst som BØR komme fra CMS
```

Kjør som en sprint-oppgave i Sprint 3 (CMS-skall) og i Pre-launch.

### Sprint-oppgaver som MÅ være med

I Sprint 3 (eller CMS-sprint), AI må eksplisitt liste:

- [ ] **Bygg CMS-coverage tabell for HVER public-side** (forside, om-oss, tjenester, hver sted×tjeneste, kontakt, blogg, etc.) — kopier malen over
- [ ] **For hver rad i CMS-coverage**: bekreft at det er en faktisk redigerings-side i admin som setter den verdien
- [ ] **For hver redigerings-side**: bekreft at form-feltene faktisk PERSISTERER til DB (Server Action testet)
- [ ] **For hvert bilde**: bekreft at det går via opplastingsmodul, ikke URL-input
- [ ] **For hvert tekst-felt**: bekreft at det er enten input eller rich-text editor (ikke hardkodet)
- [ ] **Anti-hardkoding-grep**: kjør check-script og fiks alle funn
- [ ] **Manuell verifisering**: gå gjennom hver side i prod-build og endre en tekst + verifiser at det vises på public-siden

I Pre-launch sprint (§8 CMS), AI må eksplisitt:

- [ ] **For hver side i §11**: gå inn i CMS, endre minst 3 elementer, verifiser på public-siden
- [ ] **For hver bilde-felt**: erstatt med ny opplasting + verifiser nytt bilde vises
- [ ] **For hver SEO-felt**: endre meta title, deploy, verifiser i page source
- [ ] **Anti-hardkoding-grep**: kjør sjekk og dokumenter at 0 ulovlige hardkodinger gjenstår

### Self-review-krav (utvidet)

I self-review-seksjonen, AI må krysse av:

- [ ] **§11 har CMS-coverage-tabell for HVER public-side** i prosjektet (forside + om-oss + alle tjeneste-sider + alle sted-sider + alle sted×tjeneste-sider + kontakt + blogg + lovpålagt)
- [ ] **Hver rad i hver tabell har et faktisk DB-mappingfelt + redigeringsplass**
- [ ] **Antall ✗-statuser ≤ 3 per side** (kun footer-credit + ev. juridiske disclaimers)
- [ ] **Anti-hardkoding-grep-script er listet som Sprint 3-oppgave og Pre-launch-oppgave**

> Hvis AI leverer en spec uten CMS-coverage-tabeller per side, er det en juks (jf. Prinsipp 4). Self-review skal fange dette FØR levering.

---

## Sjekkliste for AI

Når AI lager PROJECT.md som inkluderer CMS, må §10 (Funksjonell spec) og §11 (Sider og URL-struktur) inkludere:

- [ ] **Per side: liste over redigerbare felter** (alt skal være med — hvis ikke nevnt, antas det å være redigerbart)
- [ ] **Bilde-opplastingsmodul** spesifisert med alle krav (drag-drop, multi-file, alt-tekst, crop, AVIF/WebP)
- [ ] **Rich-text editor** spesifisert (TipTap eller liknende, JSON-lagring)
- [ ] **Seksjon-system** spesifisert hvis sider har strukturerte seksjoner
- [ ] **Live preview** nevnt og hvordan den fungerer
- [ ] **Auto-save** + recovery
- [ ] **Versjonering / revisjonshistorikk**
- [ ] **Roller** (minimum superadmin, ofte editor)
- [ ] **Audit log**
- [ ] **Mediabibliotek** med søk + filtre
- [ ] **SEO per side** redigerbar
- [ ] **Datamodell** for pages + page_sections + page_revisions + media

I §12 (Datamodell) må SQL inkludere alle CMS-tabellene over.

I §20 (Sprintplan) må CMS-byggingen ha sin egen dedikerte sprint (eller 2 sprinter for større prosjekter).

---

## Anti-pattern fra container-consult-as (mai 2026 — låst inn av disse reglene)

### Meny må drives av `pages`-tabellen, ikke `services`

Container Consult-menyen viste 6 tjenester (avfallscontainer, lagercontainer, kraning, maskintransport, tanktransport, kontakt) — men manglet «Nyheter», «Om oss», «FAQ» som finnes som scrapede sider. Det skjedde fordi AI bygde hovedmenyen ved å iterere `services`-tabellen.

**Regel:** Hovedmeny SKAL drives av `pages`-tabellen med `show_in_nav=true` + `nav_order`. Tjenester kan vises som dropdown under «Tjenester», men «Om oss», «Nyheter», «FAQ», «Kontakt» — alt som er en `pages`-rad — skal være tilgjengelig fra hovedmeny eller footer.

```sql
-- pages-tabellen MÅ ha:
alter table public.pages
  add column if not exists show_in_nav boolean default true,
  add column if not exists nav_order integer default 100,
  add column if not exists nav_label text;  -- hvis tittel skal forkortes i menyen
```

```tsx
// src/components/layout/MainNav.tsx — KORREKT mønster
const { data: navPages } = await supabase
  .from('pages')
  .select('slug, title, nav_label, nav_order')
  .eq('is_published', true)
  .eq('show_in_nav', true)
  .order('nav_order')
```

`check-public-pages.ts` sjekker `D1: Header har ≥4 lenker` + `D2: Header-lenker returnerer 200`. Hvis menyen mangler scrapede sider → sprint ikke ferdig.

### Bilder MÅ kobles til seksjoner, ikke bare lastes opp

Container Consult-databasen hadde 100 bilder i `media`-tabellen etter scrape — men ZERO `page_sections.data` refererte til dem. Resultat: sidene hadde ingen hero-bilder, ingen galleri, ingen visuelle elementer.

**Regel:** Når `import-scraped-content.ts` skriver `page_sections`, MÅ den også koble bilder fra `media`-tabellen til riktig sted i `data`-jsonb:

```ts
// Korrekt mønster
const heroImage = await supabase
  .from('media')
  .select('id, file_url, alt_text')
  .ilike('filename', `%${slug}%`)
  .limit(1)
  .single()

await supabase.from('page_sections').insert({
  page_id, type: 'hero', sort_order: 0,
  data: {
    headline: heroHeading,
    subline: metaDescription,
    image_id: heroImage.data?.id,         // ← KRITISK
    image_url: heroImage.data?.file_url,
    image_alt: heroImage.data?.alt_text,
  },
})
```

`check-public-pages.ts B2: Hero har bilde/gradient/illustration` fanger dette — sprint ikke ferdig hvis hero er kun tekst på flat farge.

### `force-dynamic` på public layout som leverer CMS-data (KRITISK)

Container Consult Sprint 11: AI la til migrasjon for `show_in_nav`/`nav_order`-felter, kjørte den mot prod, oppdaterte verdier i DB — men menyen viste fortsatt kun «Tjenester ▼». Årsak: Next.js cachet HTML fra build-tid. Hver kunde-redigering i CMS-en + hver DB-migrasjon ble usynlig før neste full prod-deploy.

**Regel:** Hver Server Component som leverer CMS-drevet data (meny, settings, sider, footer-info) MÅ enten:

```ts
// Alternativ A: ferskt på hver request (anbefalt for layouts med kunde-redigering)
export const dynamic = "force-dynamic"
export const revalidate = 0

// Alternativ B: ISR med kort revalidate (anbefalt for tunge layouts)
export const revalidate = 60  // sekunder

// Alternativ C: on-demand revalidation fra CMS-Save-Action
// I CMS-handlerne: revalidatePath('/') eller revalidateTag('nav')
```

For `(public)/layout.tsx` som leverer meny + settings + footer → **alternativ A**. Performance-tap er minimalt fordi React Suspense + Vercel edge cache fortsatt hjelper, og kunden ser endringene umiddelbart i stedet for å vente på ny deploy.

For sider med mye sjelden-redigert innhold (artikler, statiske sider) → **alternativ C** med `revalidateTag` i CMS-save-actions.

**Verifisering:** `check-public-pages.ts` kjørt 2 ganger med 30 sekunders mellomrom mot dev — DOM må vise oppdaterte CMS-data uten ny deploy.

### TipTap StarterKit v3+ inkluderer Link — IKKE legg til separat uten å disable først

Container Consult Sprint 11 hadde console-warning «Duplicate extension names found: ['link']» fordi StarterKit + `@tiptap/extension-link` ble lagt til parallelt. StarterKit v3 inkluderer Link as default.

**Regel:** Hvis du trenger custom Link-konfigurasjon (typisk `openOnClick: false` + `HTMLAttributes`-overrides), MÅ du disable den i StarterKit først:

```ts
import StarterKit from '@tiptap/starter-kit'
import Link from '@tiptap/extension-link'

const extensions = [
  StarterKit.configure({
    heading: { levels: [2, 3] },
    link: false,  // ← KRITISK: ellers duplikatfeil med vår egen Link.configure
  }),
  Link.configure({
    openOnClick: false,
    HTMLAttributes: { rel: 'noopener noreferrer' },
  }),
]
```

Samme gjelder enhver annen StarterKit-extension du vil overstyre (e.g. Heading, Bold, History). Disable først, custom-konfigurer etter.

### Theme-toggle MÅ være funksjonell, ikke dekorativ

Container Consult-headeren hadde en `☀`-knapp som ikke gjorde noe ved klikk. Det er verre enn å ikke ha en — det skader tilliten.

**Regel:** Hvis `requires_dark_mode=true`:

```tsx
// src/components/ThemeToggle.tsx
'use client'
import { useEffect, useState } from 'react'

export function ThemeToggle() {
  const [theme, setTheme] = useState<'light' | 'dark'>('light')

  useEffect(() => {
    const saved = localStorage.getItem('theme') as 'light' | 'dark' | null
    const initial = saved ?? (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light')
    setTheme(initial)
    document.documentElement.dataset.theme = initial
  }, [])

  const toggle = () => {
    const next = theme === 'light' ? 'dark' : 'light'
    setTheme(next)
    document.documentElement.dataset.theme = next
    localStorage.setItem('theme', next)
  }

  return <button onClick={toggle} aria-label="Bytt tema">{theme === 'light' ? '🌙' : '☀'}</button>
}
```

`check-public-pages.ts E3: Theme-toggle endrer DOM` verifiserer at klikk faktisk endrer `data-theme` på `<html>`. Hvis dette returnerer rødt: knappen er dekorativ og må fikses.

Hvis `requires_dark_mode=false`: ikke vis knappen i det hele tatt — ikke ha en knapp som ikke gjør noe.
