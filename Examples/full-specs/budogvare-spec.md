# Skedsmo Bud & Vare — Ny hjemmeside + CMS + Fraktkalkulator

**Status:** Spec — venter på bruker-review
**Dato:** 2026-05-28
**Kunde:** Skedsmo Bud & Vare AS (Industrivegen 22, 2069 Jessheim)
**Domene:** budogvare.no (eksisterende)
**Tech-stack:** Next.js 15 App Router · Supabase Postgres + Auth + Storage · Vercel · Resend
**Eier:** Myrvoll-Lunde IT Drift / Myrvoll-Lunde IT Drift

---

## 1. Visjon

Bygge en **10/10 hjemmeside med 2026-design** for Skedsmo Bud & Vare som:

- **Posisjonerer dem premium** i en bransje preget av generiske WordPress-sider
- **Konverterer på sekunder** via en fraktkalkulator som er kjernen i forsiden
- **Eies 100% av kunden** — alt innhold, priser, soner, biler, bilder, blogg, kalkulator-regler styres fra et egetbygd CMS
- **SEO-presterer minimum 95%** i Google, LLM-søk (Perplexity, ChatGPT, Claude), og bilde-/tekst-søk
- **Er knasterk på sikkerhet** — RLS, CSP, secret-management, GDPR fra dag 1
- **Tester seg selv** A-til-Å før vi sier «ferdig»

### Designretning (valgt)

**Dark Luxury Motion** + **Geist/Geist Mono-typografi** + **Split form + Live Quote**-kalkulator med Lucide-ikoner.

- Primær: `#ff4500` (orange, fra dagens brand)
- Sekundær: `#ed8f0c` (orange-accent)
- Mørke: `#0a0a0a`, `#15100a` (warm dark)
- Suksess: `#00ff88` (live-indikatorer)
- Tekst: `#ffffff`, `rgba(255,255,255,0.7)`, `rgba(255,255,255,0.5)`
- Font heading: **Geist** (variable, 100-900)
- Font body: **Geist** (samme)
- Font detail/monospace: **Geist Mono** (etiketter, breadcrumbs, små tall, "live"-indikatorer)
- Visual: orange-glow radial gradient, glassmorphism (backdrop-blur), motion ved scroll (Framer Motion)

---

## 2. Funksjonell omfang

### 2.1 Public sider (rendret fra CMS)

| Side | URL | Innhold styrt i CMS |
|------|-----|---------------------|
| Forside | `/` | Hero, USPer, fraktkalkulator-widget, biltype-grid, kundereferanser, miljø-blokk, CTA |
| Om oss | `/om-oss` | Innledning + lenker til underseksjoner |
| Om bedriften | `/om-oss/om-bedriften` | Tekst + team-grid + bilder |
| Miljøfyrtårn | `/om-oss/miljofyrtarn` | Tekst + sertifikat-PDF-lenke |
| Fair Transport | `/om-oss/fair-transport` | Tekst + sertifikat-PDF-lenke |
| Referanser | `/om-oss/referanser` | Kundeliste + samarbeidspartnere + logoer |
| Våre biler | `/biler` | Bil-grid med ikoner |
| Bil-detalj | `/biler/[slug]` | Per biltype (varebil, lastebil, kran, semi, flak, skapbil, expotrailer, 0-utslipp) |
| Tjenester / Priser | `/priser` | Prisliste + lenke til kalkulator |
| Kontakt | `/kontakt` | Skjema (→ Resend) + adresse + kart + åpningstider |
| Blogg | `/blogg` | Liste over blogg-innlegg (fra MLIT-webhook + manuelle) |
| Blogg-post | `/blogg/[slug]` | Individuelt innlegg |
| Personvern | `/personvern` | GDPR-tekst (redigeres i CMS) |
| Cookies | `/cookies` | Cookie-policy + samtykke-logg |
| Vilkår | `/vilkar` | Transportvilkår (basert på PDFen i dagens side) |
| Åpenhetsloven | `/apenhetsloven` | Lovpålagt tekst + PDF-lenke |
| 404 / 500 | — | Custom feilside med søk + tilbake-til-hjem |

### 2.2 Fraktkalkulator

Kjerneprodukt. Egen seksjon i specen (§5).

**Plassering:**
- **Forside-hero**: kompakt versjon (biltype + vekt + sone + estimat)
- **`/beregn-pris`**: full versjon (alle tilvalg + breakdown + bestill)
- **Som widget**: kan embedes på spesifikke tjenester-sider

### 2.3 CMS (admin-panel)

URL: `/admin` — krever innlogging, kun **Superadmin** har tilgang i sprint 1.

Funksjoner:
- Innholdsredigering for alle public sider (rik tekst, bilder, seksjoner)
- Bil-administrasjon (legge til, redigere, slette biltyper)
- Pris-administrasjon (soner, vekter, biltyper, tillegg, sone-matrise)
- Sone-administrasjon (postnummer → sone-mapping)
- Mediabibliotek (Supabase Storage)
- Blogg-administrasjon (legge til, redigere, publisere/avpublisere)
- Form-innboks (kontaktforespørsler, tilbudsforespørsler, kalkulator-bestillinger)
- Bruker-administrasjon (invitere, slette superadmin-er senere)
- Innstillinger (kontaktinfo, SoMe, åpningstider, SEO-tags per side)
- Cookie-samtykke-logger (GDPR)
- Audit log (hvem endret hva, når)

### 2.4 Form-håndtering

- Kontaktskjema → Resend (e-post til post@budogvare.no)
- Tilbudsforespørsel (fra kalkulator → "Be om tilbud" når noe er for komplekst) → Resend
- Bestilling fra kalkulator → Resend + lagret i `bookings` tabell

Alle skjemaer:
- Honeypot + Turnstile (Cloudflare) anti-spam
- Server-side validering (Zod)
- Rate-limiting per IP
- Lagres i Supabase + sendes via Resend
- Synlig i CMS-innboks

### 2.5 SEO + Søk

Mål: **95%+ på Lighthouse SEO, 95%+ på Google PageSpeed, fungerer i LLM-søk.**

Implementering:
- **Server-side rendering** (App Router default) for all public innhold
- **Strukturerte data (JSON-LD)** for:
  - `LocalBusiness` på forside + kontakt
  - `Service` per biltype
  - `Article` per blogg-post
  - `BreadcrumbList` på alle dypsider
  - `FAQPage` der det finnes FAQ
  - `Organization` globalt
- **OpenGraph + Twitter Card** per side (autogenert fra CMS-feltene)
- **Sitemap.xml** (autogenert fra DB)
- **robots.txt** (riktig oppsatt)
- **Norsk locale (`nb-NO`)**, hreflang-tagger
- **Per-side editerbar `<title>`, `meta description`, `OG image`** i CMS
- **Alt-tekst** påkrevd på alle bilder i CMS-uploaden
- **Strukturerte sider med h1 → h2 → h3** (h1 er låst til side-tittel)
- **Bildeoptimalisering**: Next.js `<Image>` + AVIF/WebP, lazy loading, eksplisitte dimensjoner
- **Core Web Vitals-mål**: LCP <2.5s, INP <200ms, CLS <0.1, FCP <1.5s
- **LLM-optimalisering**: ren semantisk HTML, sider eksponerer hovedinnhold i `<main>`, ingen viktig info i kun bilde-form, alle priser/tjenester også som tekst
- **llms.txt** og `llms-full.txt` ved roten (for LLM-crawlere)
- **AEO (Answer Engine Optimization)**: FAQ-blokker på alle service-sider, korte svar formet for direct quote i LLM-svar

### 2.6 Lovpålagt

- **GDPR**: personvernpolicy redigerbar, cookie-banner med granulært samtykke (essentials, analytics, marketing), samtykker logges i Supabase
- **Cookies-policy**: liste over alle cookies (genereres fra config)
- **Vilkår**: transportvilkår (kan basere på dagens PDF)
- **Åpenhetsloven**: dedikert side + lenke til PDF
- **Tilgjengelighet (WCAG 2.2 AA)**: kontrast, keyboard-nav, ARIA, redusert bevegelse-respekt
- **Cookie-banner** vises før noen analytics laster (consent-first)
- **Slett-min-data-forespørsel**-endpoint (GDPR Art. 17)

### 2.7 Integrasjoner

- **Resend** — all e-post (kontakt, tilbud, bestilling)
- **MLIT-innhold** — webhook fra deres verktøy publiserer blogg-innlegg
- **Supabase Storage** — bilder, PDFer, sertifikater
- **Cloudflare Turnstile** — anti-spam på alle skjemaer
- **Vercel Analytics** + **Vercel Speed Insights** — performance-monitoring
- **PostHog (cookie-consent-bundet)** — produktanalyse (eller utelat hvis ikke ønsket)

---

## 3. Tech-stack

| Lag | Teknologi |
|-----|-----------|
| Framework | **Next.js 15 App Router** (React Server Components default) |
| Språk | **TypeScript strict** |
| Styling | **Tailwind CSS v4** + CSS-variabler for tokens |
| Komponentbibliotek | **shadcn/ui** (selektiv import, Lucide-ikoner) |
| Animasjon | **Framer Motion** + view-transitions API der støttet |
| Forms | **react-hook-form** + **Zod** |
| Database | **Supabase Postgres** (med RLS overalt) |
| Auth | **Supabase Auth** (magic link + e-post/passord), Superadmin-only sprint 1 |
| Storage | **Supabase Storage** (bilder, PDFer) |
| E-post | **Resend** (transactional + bekreftelser) |
| Hosting | **Vercel** (Myrvoll-Lunde IT Drift-team) |
| Anti-spam | **Cloudflare Turnstile** |
| Testing | **Vitest** (unit), **Playwright** (E2E), **axe-core** (a11y) |
| CMS | **Egetbygd** i Next.js App Router (Server Actions + RSC) |
| Rich text | **TipTap** med Supabase-lagret JSON, render via custom RSC-renderer |
| Bildeoptimalisering | **Next.js Image** + **sharp** på Vercel |
| Linting | ESLint + Prettier + Stylelint |
| Type-sjekk | tsc med `--incremental` og 60s timeout (per ECC-hooks) |
| Pakkebehandler | **pnpm** |
| Package manager | pnpm |

### Hvorfor disse valgene

- **Next.js 15** gir RSC, partial prerendering, beste Vercel-integrasjon, beste SEO out-of-the-box
- **Supabase** gir oss DB + auth + storage + realtime + RLS i én pakke (Helge har erfaring)
- **Egetbygd CMS** matcher kundens ønske om full kontroll og 100% tilpasset UX
- **TipTap** er beste rich-text-editor for å lagre strukturert JSON som vi kan rendre RSC-side (raskere enn HTML-blob)
- **Vercel** er kundens prod-target og gir bygde-tid, CDN, ISR/PPR gratis

---

## 4. Datamodell (Supabase Postgres)

Alle tabeller har:
- `id uuid primary key default gen_random_uuid()`
- `created_at timestamptz default now()`
- `updated_at timestamptz default now()` (oppdateres via trigger)
- RLS aktivert

### 4.1 Auth / brukere

```sql
-- auth.users er Supabase-managed.
-- public.profiles utvider med role + display_name
create table public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  email text not null,
  display_name text,
  role text not null default 'superadmin' check (role in ('superadmin','admin','editor','customer')),
  avatar_url text,
  is_active boolean default true,
  last_login_at timestamptz,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- Invitasjoner (sprint 1: kun superadmin kan invitere)
create table public.invitations (
  id uuid primary key default gen_random_uuid(),
  email text not null,
  role text not null default 'superadmin',
  token text unique not null,
  invited_by uuid references public.profiles(id),
  expires_at timestamptz not null,
  accepted_at timestamptz,
  created_at timestamptz default now()
);
```

### 4.2 CMS — sider og seksjoner

```sql
-- Hver public-side har en "page"-rad
create table public.pages (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,            -- "om-oss", "biler/varebil"
  parent_id uuid references public.pages(id),  -- for hierarki
  title text not null,                  -- vises som h1
  status text not null default 'draft' check (status in ('draft','published','archived')),
  -- SEO
  meta_title text,                      -- <title>
  meta_description text,
  og_image_url text,
  canonical_url text,
  noindex boolean default false,
  -- Innhold er enten TipTap JSON eller seksjoner (se page_sections)
  content jsonb,                        -- TipTap JSON for "simple"-sider
  use_sections boolean default false,   -- hvis true, bruk page_sections i stedet
  -- Audit
  published_at timestamptz,
  author_id uuid references public.profiles(id),
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- For sider med strukturerte seksjoner (forside, bil-detalj)
create table public.page_sections (
  id uuid primary key default gen_random_uuid(),
  page_id uuid references public.pages(id) on delete cascade,
  position int not null,                -- rekkefølge
  section_type text not null,           -- 'hero', 'calculator', 'cars_grid', 'usp_strip', 'cta', 'testimonials', 'image_text', 'faq', 'rich_text'
  data jsonb not null,                  -- typespesifikk JSON (validert i app)
  created_at timestamptz default now()
);

-- Versjonering (revisjonshistorikk)
create table public.page_revisions (
  id uuid primary key default gen_random_uuid(),
  page_id uuid references public.pages(id) on delete cascade,
  snapshot jsonb not null,
  author_id uuid references public.profiles(id),
  created_at timestamptz default now()
);
```

### 4.3 Bilpark

```sql
create table public.vehicles (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,                -- 'varebil', 'kranbil'
  name text not null,                       -- 'Varebiler'
  short_description text,
  description jsonb,                        -- TipTap JSON
  icon_name text,                           -- Lucide icon, e.g. 'truck'
  hero_image_url text,
  gallery_image_urls text[],
  details jsonb,                            -- nyttelast, lengde, palleplasser etc.
  position int default 0,
  is_published boolean default true,
  meta_title text,
  meta_description text,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);
```

### 4.4 Fraktkalkulator — kjernemodell

Dette er hjertet av prosjektet. Hele prislogikken er datadrevet og kan endres fra CMS.

```sql
-- Geografiske områder (Oppaker, Vormsund, Dal, Jessheim, ...)
create table public.zones (
  id uuid primary key default gen_random_uuid(),
  code text unique not null,                -- 'OPPAKER', 'JESSHEIM'
  name text not null,                       -- 'Oppaker / Årnes'
  description text,
  postal_codes text[],                      -- ['2055','2056'] for autocomplete
  position int default 0,
  is_active boolean default true,
  created_at timestamptz default now()
);

-- Prisgrupper / soner (Sone 1, 2, 3, 4)
create table public.price_zones (
  id uuid primary key default gen_random_uuid(),
  code text unique not null,                -- 'SONE_1', 'SONE_2', 'SONE_3', 'SONE_4'
  name text not null,                       -- 'Nærområde (24 km)', 'Eidsvoll-Drammen'
  description text,
  position int not null,
  created_at timestamptz default now()
);

-- Sone-matrise (fra → til → prissone)
create table public.zone_matrix (
  id uuid primary key default gen_random_uuid(),
  from_zone_id uuid not null references public.zones(id),
  to_zone_id uuid not null references public.zones(id),
  price_zone_id uuid not null references public.price_zones(id),
  unique (from_zone_id, to_zone_id)
);

-- Biltype-kategorier for kalkulator (synkronisert med vehicles)
create table public.vehicle_categories (
  id uuid primary key default gen_random_uuid(),
  code text unique not null,                -- 'VAREBIL', 'SKAPBIL_LIFT', 'EURO6_18PLL', 'KRANBIL_32TM' etc.
  name text not null,
  vehicle_id uuid references public.vehicles(id),
  icon_name text,
  pricing_model text not null check (pricing_model in ('zone_weight','hourly','hourly_km','custom')),
  position int default 0,
  is_active boolean default true
);

-- Vekt-baserte prisgrupper (kun for pricing_model='zone_weight')
create table public.weight_tiers (
  id uuid primary key default gen_random_uuid(),
  category_id uuid references public.vehicle_categories(id),
  min_kg int not null,
  max_kg int not null,
  name text not null,                       -- '0-49 kg'
  position int default 0,
  unique (category_id, position)
);

-- Hovedprismatrise: kategori + vekt-tier + sone → pris
create table public.zone_weight_prices (
  id uuid primary key default gen_random_uuid(),
  category_id uuid not null references public.vehicle_categories(id),
  weight_tier_id uuid not null references public.weight_tiers(id),
  price_zone_id uuid not null references public.price_zones(id),
  price_nok int not null,                   -- i hele kroner eks. mva
  unique (category_id, weight_tier_id, price_zone_id)
);

-- Timepriser + km-priser (for pricing_model='hourly' eller 'hourly_km')
create table public.hourly_rates (
  id uuid primary key default gen_random_uuid(),
  category_id uuid references public.vehicle_categories(id),
  rate_type text not null check (rate_type in ('hourly_only','hourly_plus_km')),
  hourly_rate_nok int not null,
  km_rate_nok numeric(8,2),                 -- 5.70, 12.00 etc.
  km_threshold int default 100,             -- "etter 100 km"
  min_hours int default 2,                  -- "minimum 2 timer"
  notes text,
  created_at timestamptz default now()
);

-- Tilleggsregler (express, helg, ekstra levering, ventetid, ADR, hjelpemann)
create table public.pricing_rules (
  id uuid primary key default gen_random_uuid(),
  code text unique not null,                -- 'EXPRESS', 'KVELD_LORDAG', 'SONDAG', 'HELLIGDAG', 'TILHENGER', 'ADR'
  name text not null,
  description text,
  rule_type text not null check (rule_type in ('percentage','flat_amount','per_km','per_hour','per_unit')),
  amount numeric(10,2) not null,
  applies_to text not null check (applies_to in ('all','category','time_window')),
  category_codes text[],                    -- hvilke biltyper påvirkes
  time_window_start time,                   -- f.eks. '17:00:00'
  time_window_end time,
  weekdays int[],                           -- [1,2,3,4,5,6,7] (1=mandag)
  is_active boolean default true,
  position int default 0
);

-- Ventetid, ekstra levering, admin-omkostninger, dieseltillegg etc.
create table public.pricing_extras (
  id uuid primary key default gen_random_uuid(),
  code text unique not null,                -- 'WAITING_TIME', 'EXTRA_DELIVERY', 'ADMIN_FEE'
  name text not null,
  amount numeric(10,2) not null,
  unit text,                                -- 'per 15 min', 'per levering', 'per faktura'
  notes text,
  is_active boolean default true,
  position int default 0
);

-- Bestillinger / forespørsler fra kalkulator
create table public.bookings (
  id uuid primary key default gen_random_uuid(),
  reference text unique not null,           -- 'BVO-2026-00123'
  status text not null default 'new' check (status in ('new','accepted','quoted','completed','cancelled')),
  type text not null check (type in ('booking','quote_request')),
  -- Snapshot av kalkulator-input (for audit + uavhengig av senere prisendringer)
  inputs jsonb not null,                    -- {category, weight_tier, from_zone, to_zone, options[]}
  -- Snapshot av pris-breakdown
  price_breakdown jsonb not null,           -- [{label, amount}, ...]
  total_nok int not null,
  -- Kontaktinfo
  customer_name text not null,
  customer_email text not null,
  customer_phone text,
  customer_company text,
  pickup_address text,
  delivery_address text,
  scheduled_at timestamptz,
  notes text,
  -- Audit
  ip_address inet,
  user_agent text,
  resend_message_id text,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);
```

### 4.5 Blogg

```sql
create table public.blog_posts (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,
  title text not null,
  excerpt text,
  content jsonb,                            -- TipTap JSON
  cover_image_url text,
  status text not null default 'draft' check (status in ('draft','published','archived')),
  source text default 'manual' check (source in ('manual','mlit')),
  source_id text,                           -- ID fra MLIT
  author_name text,
  meta_title text,
  meta_description text,
  og_image_url text,
  tags text[],
  published_at timestamptz,
  author_id uuid references public.profiles(id),
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- Webhook-payloads fra MLIT (audit + retry)
create table public.mlit_events (
  id uuid primary key default gen_random_uuid(),
  payload jsonb not null,
  signature text,
  status text not null default 'received' check (status in ('received','processed','failed')),
  error text,
  blog_post_id uuid references public.blog_posts(id),
  created_at timestamptz default now()
);
```

### 4.6 Forms / kontakt

```sql
create table public.form_submissions (
  id uuid primary key default gen_random_uuid(),
  form_type text not null check (form_type in ('contact','quote','newsletter','custom')),
  data jsonb not null,
  email text,
  phone text,
  status text not null default 'new' check (status in ('new','read','replied','archived','spam')),
  ip_address inet,
  user_agent text,
  resend_message_id text,
  created_at timestamptz default now(),
  read_at timestamptz,
  read_by uuid references public.profiles(id)
);
```

### 4.7 Media

```sql
-- Lagres i Supabase Storage. Metadata i DB.
create table public.media (
  id uuid primary key default gen_random_uuid(),
  storage_path text not null unique,        -- 'media/uuid.jpg'
  url text not null,                        -- public CDN URL
  filename text not null,
  mime_type text not null,
  size_bytes bigint,
  width int,
  height int,
  alt_text text,                            -- påkrevd ved bruk (validert ved insert i CMS)
  caption text,
  uploaded_by uuid references public.profiles(id),
  tags text[],
  created_at timestamptz default now()
);
```

### 4.8 Innstillinger og GDPR

```sql
-- Site-wide innstillinger
create table public.site_settings (
  key text primary key,                     -- 'contact_email', 'phone', 'address', 'opening_hours', 'social_facebook'
  value jsonb not null,
  description text,
  updated_at timestamptz default now(),
  updated_by uuid references public.profiles(id)
);

-- Cookie-samtykker (GDPR-revisjon)
create table public.cookie_consents (
  id uuid primary key default gen_random_uuid(),
  consent_id text not null,                 -- klient-generert UUID i cookie
  categories jsonb not null,                -- {essential: true, analytics: false, marketing: false}
  user_agent text,
  ip_hash text,                             -- hashet for personvern
  created_at timestamptz default now()
);

-- Audit log (hvem endret hva)
create table public.audit_log (
  id uuid primary key default gen_random_uuid(),
  actor_id uuid references public.profiles(id),
  action text not null,                     -- 'create','update','delete','login','publish'
  entity_type text not null,                -- 'page','vehicle','price','booking'
  entity_id uuid,
  before jsonb,
  after jsonb,
  ip_address inet,
  created_at timestamptz default now()
);
```

### 4.9 RLS-policyer (oppsummert)

**Sprint 1 — kun superadmin:**

- `pages`, `vehicles`, `zones`, `price_zones`, `zone_matrix`, `vehicle_categories`, `weight_tiers`, `zone_weight_prices`, `hourly_rates`, `pricing_rules`, `pricing_extras`, `blog_posts`, `site_settings`, `media`, `invitations`:
  - SELECT public (anon) for status='published' rader; alle for superadmin
  - INSERT/UPDATE/DELETE bare for superadmin
- `bookings`, `form_submissions`, `cookie_consents`, `mlit_events`, `audit_log`:
  - INSERT public via service-role (server-side actions)
  - SELECT/UPDATE bare for superadmin
- `profiles`:
  - SELECT egen rad for innlogget bruker
  - UPDATE egen rad (egne ikke-role felt)
  - Alt for superadmin
- `page_revisions`:
  - INSERT trigget av app
  - SELECT bare for superadmin

**Framtidig (sprint 2+):** admin / editor / customer policyer legges til uten å bryte sprint 1-data.

---

## 5. Fraktkalkulator — detaljert spesifikasjon

### 5.1 Inputs

1. **Biltype-kategori** (vehicle_categories): klikkbare chips med Lucide-ikoner
   - Varebil (truck)
   - Skapbil m/lift (truck)
   - Termobil/Kjølebil (snowflake)
   - Kranbil (construction)
   - Semitrailer (truck)
   - Flakbil (truck)
   - 0-utslipp (zap)
2. **Vekt-tier** (weight_tiers): pillss "0–49kg", "50–499kg", "500–999kg" (filtrert per kategori)
3. **Fra-sone** og **Til-sone** (zones): combobox med søk på navn + postnummer
   - Default: "Fra Jessheim" (sentralen)
4. **Dato/tid** (date picker): for å beregne kveld/helg/helligdag-tillegg
   - Default: nå
5. **Tilvalg** (toggles, fra pricing_rules):
   - Express (Hentes innen 1 time — flat 31 kr/km)
   - Tilhenger (+10%)
   - ADR-gods (+10%)
   - Hjelpemann (+506 kr/time)
6. **Antall leveringer** (default 1, ekstra: 291 kr per ekstra)
7. **Ventetid** (timer/min, beregnes etter 15 min)
8. **Bom/ferge** (text-input som vises som "Faktiske utlegg")

### 5.2 Beregningsalgoritme

```ts
// Pseudokode
function calculatePrice(input: CalcInput): PriceBreakdown {
  const category = await db.vehicle_categories.find(input.categoryCode)

  if (category.pricing_model === 'zone_weight') {
    // 1. Slå opp pris-sonen fra zone_matrix
    const priceZone = await db.zone_matrix.find({
      from_zone_id: input.fromZoneId,
      to_zone_id: input.toZoneId
    }).price_zone

    // 2. Slå opp grunnpris
    const basePrice = await db.zone_weight_prices.find({
      category_id: category.id,
      weight_tier_id: input.weightTierId,
      price_zone_id: priceZone.id
    }).price_nok

    let lines = [{ label: `Grunnpris (${category.name}, ${weightTier.name}, ${priceZone.name})`, amount: basePrice }]
    let runningTotal = basePrice

    // 3. Tidsbaserte tillegg
    const timeRules = computeTimeAddons(input.scheduledAt) // hverdag etter 17, lørdag, søndag, helligdag, 1./17.mai
    for (const rule of timeRules) {
      const addon = Math.round(runningTotal * (rule.amount / 100))
      lines.push({ label: rule.name, amount: addon })
      runningTotal += addon
    }

    // 4. Tilvalg
    if (input.options.express) {
      // Express er 31 kr/km, krever distanse — be om "ca km" eller fast tillegg?
      // PDF sier: "Hentes innen 1 time, leveres direkte Kr. 31,- pr km."
      // For kalkulator-prototyp: la bruker skrive inn forventet km, eller flat antakelse
      const expressAddon = input.expectedKm * 31
      lines.push({ label: `Express (${input.expectedKm} km × 31 kr)`, amount: expressAddon })
      runningTotal += expressAddon
    }

    if (input.options.trailer) {
      const addon = Math.round(runningTotal * 0.10)
      lines.push({ label: 'Tilhenger +10%', amount: addon })
      runningTotal += addon
    }

    if (input.options.adr) {
      const addon = Math.round(runningTotal * 0.10)
      lines.push({ label: 'ADR-gods +10%', amount: addon })
      runningTotal += addon
    }

    // 5. Per-enhet
    if (input.extraDeliveries > 0) {
      const addon = input.extraDeliveries * 291
      lines.push({ label: `Ekstra levering × ${input.extraDeliveries}`, amount: addon })
      runningTotal += addon
    }

    if (input.waitingMin > 15) {
      const blocks = Math.ceil((input.waitingMin - 15) / 15)
      const addon = blocks * 152
      lines.push({ label: `Ventetid (${blocks} × 15 min)`, amount: addon })
      runningTotal += addon
    }

    if (input.helper) {
      const addon = input.helperHours * 506
      lines.push({ label: `Hjelpemann (${input.helperHours} t)`, amount: addon })
      runningTotal += addon
    }

    // 6. Administrasjon (faktura)
    lines.push({ label: 'Adm. omkostning (faktura)', amount: 49, note: 'tillegges ved fakturering' })

    return { lines, total: runningTotal, vatExclusive: true, addons: ['Bom/ferge faktiske utlegg', 'HVO 100 på forespørsel'] }
  }

  if (category.pricing_model === 'hourly' || category.pricing_model === 'hourly_km') {
    // Tilsvarende for timepris-modeller
    // ...
  }
}
```

### 5.3 Edge cases

- **Bom/ferge**: Vi viser «kommer i tillegg som faktiske utlegg» — kan ikke estimeres.
- **Express**: Krever km-input. Hvis bruker ikke fyller inn, viser vi grunnpris + «Express: 31 kr/km på toppen». For B2B-kunder med faste ruter kan vi senere ha lagrede ruter.
- **Helgetillegg-kombinasjon**: hvis det er både kveld og lørdag — bruker høyeste prosent (25%). Hellig + søndag = 100% (regelen i PDFen er at helligdager og 1/17. mai er +100%, søndag er +50%).
- **Minimum timer (timepris-biler)**: alltid minimum 2 timer på Sone 1.
- **Vekter over 999 kg**: kalkulator viser «Be om tilbud» og åpner kontakt-skjema med pre-fylte felter.
- **Ukjente kombinasjoner** (timer-modus, lange ruter): «Be om tilbud»-flyt.
- **HVO 100**: vises som info-checkbox med tekst «Tillegg på forespørsel — vi kontakter deg».

### 5.4 UX-detaljer

- **Live-oppdatering** på hver input-endring (debounced 200ms)
- **Grønn "live"-prikk** når prisen oppdateres
- **Breakdown alltid synlig** — også på mobil (kollapser til drawer)
- **«Bestill nå»** = full bestillingsflyt med kontaktinfo
- **«Be om tilbud»** = enklere form, sender forespørsel
- **«Del/skriv ut tilbud»** = genererer PDF (sprint 2)
- **Lagre kalkulasjon** (sprint 2): lenke til pre-fylt kalkulator

### 5.5 Validering

- Server-side validering av alle prisberegninger (klient kan ikke endre pris)
- Bestilling lagrer både input og beregnet pris (snapshot) — endring av prisliste senere endrer ikke historiske bestillinger
- Audit log: alle prisendringer logges med før/etter

---

## 6. Innholds-scrape og migrering

Når siden er bygget kjøres en **migrerings-job** som scraper dagens budogvare.no og fyller CMS:

### 6.1 Hva som scrapes

1. **Sider**: alle sider fra sitemap.xml (15+ sider) → `pages` + `page_sections`
2. **Bil-detaljer**: hver `/vare-biler/*` → `vehicles`
3. **Bilder**: alle bilder fra `/wp-content/uploads/2025/01/` → Supabase Storage → `media` med alt-tekst (autogenerert fra filnavn, redigerbar)
4. **Tekstinnhold**: h1, h2, h3, p, ul-grupper → strukturert TipTap JSON
5. **Logo**: `skedsmo-bud-vare-hovedlogo-sort-orange.png` → Storage
6. **Favicon**: 32x32, 180x180, 192x192 → public/
7. **PDFer**: Miljøfyrtårn-sertifikat, Fair Transport-sertifikat, prisliste, soner, Åpenhetsloven → Storage
8. **Prisdata**: parses fra PDF (allerede gjort i denne specen) → `zones`, `price_zones`, `zone_matrix`, `vehicle_categories`, `weight_tiers`, `zone_weight_prices`, `pricing_rules`, `pricing_extras`
9. **Kontaktinfo**: Industrivegen 22, 2069 Jessheim, 63 94 38 80, post@budogvare.no → `site_settings`
10. **Team-medlemmer**: Hans Petter, Svein Arne, Maya, Ellen, Anna, Vidar, Stig → `pages.team` JSON eller egen tabell (kall det `team_members`)

### 6.2 Implementering

- Egen `pnpm run scrape` script som kjøres etter at DB-skjema er på plass
- Skriptet er **idempotent** — kan kjøres flere ganger uten duplikater
- Lagrer audit-log over hva som ble migrert
- Etter migrering kan superadmin redigere alt i CMS

---

## 7. Sikkerhet

### 7.1 Secrets-håndtering (per Helge-CLAUDE.md)

- Alle secrets i `.env.local` (gitignored)
- Speil til **Vercel env vars** (production, preview, development)
- Speil til **GitHub secrets** for CI/CD
- Aldri commit secrets
- Bruker `gh secret set` automatisk når nye secrets opprettes

Nødvendige secrets:

```bash
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
SUPABASE_SERVICE_ROLE_KEY=...
SUPABASE_JWT_SECRET=...
RESEND_API_KEY=...
RESEND_FROM_EMAIL=post@budogvare.no
TURNSTILE_SITE_KEY=...
TURNSTILE_SECRET_KEY=...
MLIT_WEBHOOK_SECRET=...
NEXT_PUBLIC_SITE_URL=https://budogvare.no
CRON_SECRET=...
```

### 7.2 RLS-overalt

Alle tabeller har RLS aktivert. Service-role brukes kun fra server-side handlers (Next.js Server Actions, API routes). Aldri eksponert til klient.

### 7.3 CSP (Content Security Policy)

Nonce-basert CSP i `middleware.ts`:

```
default-src 'self';
script-src 'self' 'nonce-{RANDOM}' https://vercel.live https://challenges.cloudflare.com;
style-src 'self' 'unsafe-inline';
img-src 'self' data: https://*.supabase.co https://budogvare.no;
font-src 'self' data:;
connect-src 'self' https://*.supabase.co https://api.resend.com https://challenges.cloudflare.com;
frame-src https://challenges.cloudflare.com;
object-src 'none';
base-uri 'self';
```

### 7.4 Andre sikkerhets-headers

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

### 7.5 Rate-limiting

- Alle public form-endpoints: 5 req/min per IP (via Vercel KV eller Supabase edge function med leaky bucket)
- Auth-endpoints: 10 req/min per IP
- Kalkulator: 60 req/min per IP

### 7.6 Anti-spam

- **Turnstile** på alle public skjemaer
- Honeypot-felt
- Server-side validering med Zod

### 7.7 Input-validering

- Alle inputs gjennom Zod-skjema
- Aldri rå SQL — alltid Supabase JS-client eller forberedte queries
- Sanitization av rich-text før render (DOMPurify eller TipTap-renderer som ikke aksepterer custom HTML)

---

## 8. Lovpålagt (GDPR + Norge)

### 8.1 Cookies

Cookie-banner med tre nivåer:
- **Nødvendige** (alltid på, ikke valgbart) — auth, CSRF, session
- **Analyse** (opt-in) — Vercel Analytics, ev. PostHog
- **Markedsføring** (opt-in) — pixel-cookies hvis brukt

Banner er fullstendig blokkerende for analytics-scripts til samtykke gis. Logger samtykker i `cookie_consents` med hashet IP.

### 8.2 Personvern-side

- Hvilke data samles inn
- Hvorfor (formål)
- Hvor lenge (oppbevaringstid)
- Hvem som har tilgang
- Brukerens rettigheter (innsyn, sletting, klage til Datatilsynet)
- Behandlingsansvarlig (Skedsmo Bud & Vare AS) + DPO-kontakt

### 8.3 Slettings-flyt

Endpoint `/api/gdpr/delete` der bruker kan be om sletting med e-post-verifisering. Sletter:
- `form_submissions` med matching e-post (anonymiserer eldre enn 30 dager)
- `bookings` (anonymiserer kontaktinfo, beholder pris/historikk for regnskapsår)
- `cookie_consents` med samme consent_id

### 8.4 Tilgjengelighet (WCAG 2.2 AA)

- Kontrast minimum 4.5:1 for body, 3:1 for stort
- Tab-rekkefølge logisk
- Focus visible på alle interaktive elementer
- ARIA-labels der ikoner brukes uten tekst
- `prefers-reduced-motion`-respekt — alle animasjoner kan skrus av
- Skjema-feilmeldinger lest av screen readers

---

## 9. Sprintplan

10 sprinter, hver leveres ferdig og testet før neste starter.

### Sprint 0 — Grunnmur (1 dag)
- [ ] Lag GitHub-repo `myrvoll-lunde-it-drift/budogvare` (Helge-organisasjon: myrvoll-lunde-it-drift)
- [ ] `pnpm dlx create-next-app@latest budogvare --typescript --tailwind --app --turbopack`
- [ ] Installer shadcn/ui, Lucide-react, Framer Motion, react-hook-form, zod, @supabase/ssr, resend, @tiptap/react
- [ ] Sett opp Supabase-prosjekt i UI (Helge gjør dette — gir deretter `NEXT_PUBLIC_SUPABASE_URL`, anon key, service-role key)
- [ ] Bekreft Supabase auto-deploy-fra-GitHub er på (per Helges instruks: "update schema in code, push to GitHub, Supabase deploys")
- [ ] Sett opp Vercel-prosjekt i `myrvoll-lunde-it-drift-s-projects`-team + GitHub-integrasjon (auto-deploy fra main)
- [ ] Sett opp Resend-domene-verifisering for `budogvare.no` (DKIM/SPF/DMARC i DNS)
- [ ] Sett opp Turnstile-konto + site/secret-keys
- [ ] Konfigurer `.env.local` (gitignored), Vercel env vars, GitHub secrets via `gh secret set`
- [ ] Obsidian-vault auto-registrering: `C:\Github\vault\prosjekter\Budogvare.md` + oppdater Prosjektoversikt + klient-notat for Skedsmo Bud & Vare i `03-Klienter/`
- [ ] CI/CD: GitHub Actions for lint + typecheck + Vitest + Playwright + Lighthouse CI
- [ ] Sett opp ECC-hooks (format/lint/typecheck/build) per coding-style.md
- **Leveranse:** Tom Next.js-app deployer automatisk til Vercel + Supabase auto-deployer migrasjoner fra GitHub

### Sprint 1 — Databasegrunnlag + Auth + CMS-skall (3 dager)
- [ ] Alle Supabase-migrasjoner skrevet og kjørt (alle tabeller i §4)
- [ ] RLS-policyer skrevet og enhetstestet
- [ ] TypeScript-typer generert (`pnpm gen-types`)
- [ ] Supabase Auth: magic link + e-post/passord
- [ ] First superadmin opprettes (Helge/Skedsmo)
- [ ] Admin-route `/admin` med beskyttet layout
- [ ] CMS-skall: side-liste, oppretting, sletting, søk
- [ ] Audit-log skrives ved alle endringer
- [ ] Cookie-banner + samtykke-logger
- **Leveranse:** Innlogget superadmin kan lage og slette tomme sider; cookie-banner virker

### Sprint 2 — Public side-skall + design-system (3 dager)
- [ ] Design-system: tokens, fargevariabler, typografi, shadcn-komponenter
- [ ] Layout: header, footer, mobil-nav, dark-mode (default)
- [ ] Page-renderer (TipTap JSON → RSC)
- [ ] Section-renderer (hero, image_text, cta, faq, testimonials, rich_text)
- [ ] 404 / 500-sider
- [ ] sitemap.xml + robots.txt + llms.txt
- [ ] JSON-LD: Organization, LocalBusiness, WebSite (globalt)
- [ ] OG image-generering (Next.js OG-image-route)
- **Leveranse:** Side rendret fra CMS-data ser premium ut i dark luxury motion-stil

### Sprint 3 — Sider og innhold (4 dager)
- [ ] Forside-design med alle seksjoner (hero, kalkulator-widget, biler, USPer, miljø, kontakt-CTA)
- [ ] Om oss-hub + 4 underseksjoner
- [ ] Biler-side + 8 underseksjoner (per biltype)
- [ ] Kontakt-side med skjema + kart
- [ ] Blogg-side + blogg-post-side (uten innhold ennå)
- [ ] Personvern, Cookies, Vilkår, Åpenhetsloven (statisk innhold redigerbart)
- [ ] Innholdsscrape-script (§6) — populerer DB med all data fra dagens budogvare.no
- [ ] Mediabibliotek (Supabase Storage + CMS-uploader)
- [ ] Team-medlemmer-seksjon på Om bedriften
- **Leveranse:** Alle dagens sider eksisterer i ny design med innhold

### Sprint 4 — Fraktkalkulator (5 dager)
- [ ] Pris-engine i `lib/pricing.ts` med 100% test-dekning (alle PDF-eksempler)
- [ ] Kalkulator-komponent (split form + live quote) med Lucide-ikoner
- [ ] Sone-velger med postnummer-søk
- [ ] Tilvalg (express, helg, tilhenger, ADR, hjelpemann, ventetid, ekstra)
- [ ] Bestillingsflyt (kontaktinfo → Resend e-post → lagret i bookings)
- [ ] "Be om tilbud"-flyt
- [ ] CMS for kalkulator: rediger soner, prisgrupper, biltyper, prisregler — alle endringer reflekteres umiddelbart
- [ ] Kompakt versjon for forside-hero
- [ ] Full versjon på `/beregn-pris`
- [ ] PDF-eksport av tilbud (sprint 5 hvis tid)
- **Leveranse:** Fraktkalkulator fungerer 100% mot dagens prisliste, alt redigerbart i CMS

### Sprint 5 — SEO + Performance + Accessibility (3 dager)
- [ ] Strukturerte data per side-type (LocalBusiness, Service, Article, FAQPage, BreadcrumbList)
- [ ] Per-side meta-title, description, OG image i CMS
- [ ] Bildeoptimalisering: alle bilder gjennom Next.js Image + AVIF/WebP
- [ ] Font-optimalisering (subsetting, preload)
- [ ] Core Web Vitals: LCP <2.5s, INP <200ms, CLS <0.1
- [ ] Lighthouse 95%+ på Performance, SEO, Accessibility, Best Practices
- [ ] axe-core a11y-tester per side
- [ ] WCAG 2.2 AA-sjekkliste fullført
- [ ] `prefers-reduced-motion`-respekt overalt
- [ ] llms.txt + llms-full.txt
- [ ] AEO-blokker (FAQ, "What is...", "How much does...") på service-sider
- **Leveranse:** 95%+ på alle Lighthouse-mål, AA-tilgjengelig

### Sprint 6 — Forms + Resend + Anti-spam (2 dager)
- [ ] Kontaktskjema → Resend-mal → e-post + lagret i form_submissions
- [ ] Tilbudsforespørsel-skjema
- [ ] Bestillingsbekreftelse (e-post til kunde + intern e-post)
- [ ] Turnstile på alle skjemaer
- [ ] Rate-limiting (Vercel KV)
- [ ] CMS-innboks for alle skjema-innsendinger
- [ ] Innstillinger for mottaker-e-poster i CMS
- **Leveranse:** Alle skjemaer fungerer, kunder mottar bekreftelse, ingen spam slipper gjennom

### Sprint 7 — MLIT-integrasjon (2 dager)
- [ ] Webhook-endpoint `/api/webhooks/mlit`
- [ ] Signatur-validering
- [ ] Idempotens (lagrer event-id)
- [ ] Mapping: MLIT-payload → blog_posts
- [ ] CMS: vis kilde (MLIT / manuell), tillat redigering etter mottak
- [ ] Test webhook med mock-payloads
- **Leveranse:** MLIT kan publisere blogger som dukker opp i CMS

### Sprint 8 — Sikkerhet + GDPR (2 dager)
- [ ] CSP med nonce
- [ ] Security headers (HSTS, X-Frame, Permissions-Policy)
- [ ] Input-sanitization (DOMPurify på rich-text-render)
- [ ] Slettings-endpoint (`/api/gdpr/delete`)
- [ ] Innsyns-endpoint (`/api/gdpr/export`)
- [ ] Cookie-policy autogenerert fra config
- [ ] Personvern-policy redigerbar i CMS
- [ ] Sikkerhets-review (security-reviewer agent)
- **Leveranse:** Bestått sikkerhets-review + alle GDPR-rettigheter implementert

### Sprint 9 — A-Å testing (3 dager)
- [ ] Playwright E2E: alle public sider laster
- [ ] Playwright E2E: kalkulator gir riktig pris for 20+ scenarier (alle fra PDF)
- [ ] Playwright E2E: kontaktskjema, tilbudsforespørsel, bestilling
- [ ] Playwright E2E: superadmin-login + alle CMS-handlinger
- [ ] Playwright E2E: invitasjonsflyt (selv om bare superadmin i sprint 1)
- [ ] Playwright E2E: RLS-policyer (anon vs. innlogget)
- [ ] Visual regression: alle hovedsider på 320, 768, 1024, 1440
- [ ] Axe a11y-tester: alle sider 100% passing
- [ ] Cross-browser: Chrome, Firefox, Safari, Edge
- [ ] Mobil-test: alle sider på iPhone SE, iPhone 15 Pro, Samsung Galaxy, iPad
- [ ] Performance: Lighthouse CI i pipeline
- [ ] Last-test: kalkulator 100 samtidige brukere
- **Leveranse:** Testdekning >80%, alle critical paths grønne

### Sprint 10 — Mobiltilpasning + finpuss (3 dager)
- [ ] Audit hver side på mobil — fiks alle overflow, alle touch-targets <44px, alle font-sizes <14px
- [ ] Test kalkulator-UX på mobil (sticky breakdown? bottom-sheet?)
- [ ] Test CMS-UX på iPad
- [ ] Motion-polish: scroll-baserte animasjoner, view-transitions
- [ ] Mikro-interaksjoner (button-hover, input-focus, success-confetti)
- [ ] Final QA-pass av kunde
- [ ] DNS-cutover-plan (peke budogvare.no til Vercel)
- [ ] 301-redirects fra gamle URL-er
- **Leveranse:** Klar for lansering — alle sider perfekte på alle skjermstørrelser

---

## 10. Hva er IKKE i scope (sprint 1)

Disse er bevisst utsatt eller utelatt for å holde fokus:

- **Kundeweb** (innlogget kundeportal med bestillingshistorikk, fakturaer) — sprint 11+ hvis ønsket
- **EDI/API for integrasjon** med kundenes systemer — sprint 11+
- **AI-fritekst i kalkulator** — sprint 12 (etter prod-lansering)
- **Booking-kalender for sjåfører** — utenfor scope (separat system)
- **Lagerstyring / WMS** — utenfor scope
- **App for sjåfører** — utenfor scope
- **Live-sporing av leveranser på public-side** — sprint 12+
- **Stripe/Vipps-integrasjon for forskuddsbetaling** — sprint 11+
- **Flerspråk (engelsk)** — sprint 11+ hvis ønsket
- **Admin/Editor-roller** — modellen støtter det, men UI/policyer bygges når faktisk behov

---

## 11. Risiko og åpne spørsmål

| Risiko | Sannsynlighet | Mitigering |
|--------|---------------|------------|
| Sone-matrise i PDF er ikke fullstendig (noen ruter mangler) | Middels | Vi spør Skedsmo Bud & Vare om hva som skjer for manglende kombinasjoner — default fallback til "be om tilbud" |
| Express-prising krever km — bruker vet ikke alltid | Høy | Vi viser estimat "fra X kr" med info, eller bruker autocomplete med Google Distance Matrix (krever API-key) |
| MLIT-webhook-format ukjent | Høy | Avklares med MLIT-teamet i sprint 7 — vi designer fleksibelt |
| Skedsmo Bud & Vare har innspill på design vi ikke ser i dag | Middels | Vi ber om kundes godkjenning etter sprint 2 og 3 før vi går videre |
| Eksisterende e-poster (post@budogvare.no) har deliverability-issues | Lav | Verifiserer Resend-domene før sprint 6, setter opp SPF/DKIM/DMARC |
| GDPR-personvern-policy må juridisk gjennomgås | Lav | Vi leverer best-practice template, kunde får juridisk review før lansering |

**Åpne spørsmål til Helge/Skedsmo:**

1. **Express-km**: hvordan beregner dere distanse i dag? Google Maps, fast antakelse, eller bruker?
2. **Bestilling vs. tilbud**: skal kalkulatoren ende med en bindende bestilling, eller alltid med en tilbudsforespørsel?
3. **Kundeweb**: skal vi linke til eksisterende kundeweb i header (`/kundeweb` lenke), eller bygges det fra null senere?
4. **Hjelpemann og spesialkjøretøy**: er prising 506 kr/time fortsatt riktig?
5. **MLIT**: hvem på MLIT-siden kan vi koordinere med for webhook-spec?
6. **Domeneoverføring**: når sprint 10 er ferdig — kjører vi DNS-cutover en bestemt dag, eller A/B parallelt en periode?

---

## 12. Suksesskriterier

Prosjektet er ferdig når **ALLE** disse er sant:

- [ ] Lighthouse SEO ≥ 95% på alle hovedsider
- [ ] Lighthouse Performance ≥ 95% på forside og kalkulator-side
- [ ] Lighthouse Accessibility ≥ 95% på alle sider
- [ ] Lighthouse Best Practices ≥ 95% på alle sider
- [ ] Core Web Vitals "good" på alle sider (LCP <2.5s, INP <200ms, CLS <0.1)
- [ ] WCAG 2.2 AA-pass via axe-core
- [ ] All innhold redigerbart i CMS (verifisert ved sprint 9-test)
- [ ] Kalkulator gir riktig pris for alle 20+ PDF-test-scenarier
- [ ] Alle skjemaer leverer e-post (verifisert mottatt)
- [ ] RLS testet med anon-rolle: kan ikke endre noe
- [ ] Cookie-samtykke logges, banner blokkerer analytics før samtykke
- [ ] Personvern, cookies, vilkår, åpenhetsloven-sider eksisterer og er korrekte
- [ ] Alle sider ser perfekte ut på 320, 375, 768, 1024, 1440, 1920
- [ ] Cross-browser: Chrome, Firefox, Safari, Edge alle grønne
- [ ] CSP, HSTS, security headers på plass
- [ ] Test-dekning ≥ 80% (Vitest)
- [ ] Playwright E2E grønn i CI
- [ ] Kunde har gitt skriftlig godkjenning på design og funksjon
