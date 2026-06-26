# Section Detail — Signature moment-trace (§34)

> §5 i PROJECT.md definerer signature moment — det ene som skiller prosjektet fra alle konkurrenter. §5.2 lister hvor den skal være. Container Consult Sprint 4 (mai 2026): AI bygde BookingCalculator-komponenten, men plasserte den kun på `/bestill`. Hele forretningsverdien var gjemt på en underside. Forsiden hadde i stedet et stort hero-bilde + standard tjenester-grid — sterkt anti-template-brudd. **Aldri igjen.**

---

## Hovedregel

Signature moment er **ikke** «en kul interaktiv ting et eller annet sted». Det er **selve grunnen kunden er på siden** — og den MÅ stå der besøkende ikke kan unngå å se den.

| Plassering (fra §5.2) | Hvor i UI | Eksempel (Container Consult) |
|---|---|---|
| **Primær** | Forside, full bredde, over fold (≤800px scroll) | Container-booking-kalkulator direkte under header |
| **Sekundær** | Øverst på alle relaterte produkt-/tjenestesider | Samme kalkulator øverst på `/avfallscontainer`, `/lagercontainer`, osv. |
| **Tertsiær** | Dedikert side med fullscreen-versjon | `/bestill` med maks plass + ekstra forklaring |
| **Embed** | Per sted×tjeneste-side i SEO-grid | Inline widget på alle 72 `/<sted>/<tjeneste>`-URL-er |

**Hvis komponenten finnes men ikke står på primær-plassering → leveransefeil.** Forretningsverdien er borte.

---

## Convention: `data-signature-moment` attribute

Hver implementasjon av signature moment-komponenten MÅ tagges med dette attributtet på rot-elementet:

```tsx
// src/components/booking/BookingCalculator.tsx
export function BookingCalculator(props: Props) {
  return (
    <section
      data-signature-moment="booking-calculator"
      aria-labelledby="booking-calc-heading"
      className="..."
    >
      {/* ... */}
    </section>
  )
}
```

Convention-navn: kebab-case, samme på tvers av plasseringer. Brukes av Playwright-sjekkene for å finne komponenten uavhengig av React-tre-struktur.

---

## Plassering på primær (forside)

```tsx
// src/app/(public)/page.tsx
export default async function HomePage() {
  const containerTypes = await getContainerTypes()
  return (
    <>
      <Hero />
      {/* Signature moment OVER fold — typisk maks 600px scroll fra topp */}
      <BookingCalculator containerTypes={containerTypes} />
      <ServicesBento services={...} />
      <NewsFeed />
    </>
  )
}
```

**Krav:** komponentens topp må være innenfor `viewport.height + 200px` fra top of page. Verifiseres med Playwright `boundingBox().y < 1000`.

## Plassering på sekundær (tjenestesider)

For dynamiske `/[slug]`-routes som henter side fra DB:

```tsx
// src/app/(public)/[slug]/page.tsx
const SECONDARY_SIGNATURE_SLUGS = await getSecondarySignaturePlacements()
// Hentes fra DB-flagg, ikke hardkodet (§33):
// pages.show_signature_moment boolean default false
// services-tabellen kan også markere dette per service

export default async function ServicePage({ params }: PageProps) {
  const { slug } = await params
  const page = await getPageBySlug(slug)
  const showSignature = SECONDARY_SIGNATURE_SLUGS.has(slug)

  return (
    <>
      <PageHero page={page} />
      {showSignature && <BookingCalculator containerTypes={...} variant="embed" />}
      <SectionRenderer sections={page.sections} />
    </>
  )
}
```

**Datamodell-utvidelse** (§12 må inkludere):

```sql
alter table public.pages
  add column if not exists show_signature_moment boolean default false;

-- Sett for relevante sider — kunden kan toggle i admin etter behov
update public.pages set show_signature_moment = true
  where slug in ('/', '/avfallscontainer','/lagercontainer','/kraning',
                 '/maskintransport','/tanktransport','/cctanktransport');
```

Slik unngår vi hardkoding (§33) samtidig som vi har full kontroll fra admin.

## Plassering på sted×tjeneste-grid

```tsx
// src/app/(public)/[sted]/[tjeneste]/page.tsx
// Signature moment alltid embed i sted×tjeneste — det er nettopp HER folk søker
return (
  <>
    <LocalServiceHero ... />
    <BookingCalculator containerTypes={...} variant="embed" prefilledLocation={sted} />
    <ContentBlocks ... />
  </>
)
```

`prefilledLocation` gir konvertering-boost — adressen er allerede satt, brukeren trenger bare å velge container + dato.

---

## Polymorf signature moment (for prosjekter med flere tjenestetyper)

**Anti-pattern (Container Consult 30. mai 2026):** AI embed'et samme «velg container»-kalkulator på alle 7 tjenestesider, inkludert maskintransport og kraning. Resultat: kalkulatoren spør om container-størrelse selv om brukeren er på en kran-side. Helt feil signal til besøkende — *«de skjønner ikke hva tjenesten min er»*.

**Hovedregel:** Hvis `services`-tabellen har **flere distinkte tjenestetyper** (container vs transport vs konsultasjon vs SaaS-tier vs e-handel-kategori), MÅ signature moment være **polymorf**. Aldri embed én generisk kalkulator som lyver om hva den faktisk løser.

### Mønster: trinn 0 + dynamiske trinn fra DB

**Trinn 0 — «Hva trenger du?»** Vises kun på generiske plasseringer (forsiden). På tjenestespesifikke sider hopper den direkte til trinn 1 (tjenesten er pre-valgt).

**Trinn 1-N — drives av `booking_flows`-tabell per tjenestetype.**

### Datamodell

```sql
-- Hver tjeneste har sin egen step-flyt
create table public.booking_flows (
  service_slug text primary key references public.services(slug) on delete cascade,
  steps jsonb not null,                  -- array av {type, label, required?, options?}
  inquiry_only boolean default false,    -- true → vis bare "Be om tilbud"-skjema (ingen sanntidspris)
  updated_at timestamptz default now()
);

-- Eksempel:
insert into booking_flows (service_slug, steps) values
  ('avfallscontainer', '[
     {"type":"container_size","label":"Containerstørrelse"},
     {"type":"address","label":"Adresse"},
     {"type":"date","label":"Leveringsdato"}
   ]'::jsonb),
  ('maskintransport', '[
     {"type":"machine_type","label":"Maskintype"},
     {"type":"weight","label":"Vekt"},
     {"type":"address_from","label":"Fra-adresse"},
     {"type":"address_to","label":"Til-adresse"}
   ]'::jsonb),
  ('kraning', '[
     {"type":"crane_hours","label":"Antall timer"},
     {"type":"reach","label":"Høyde/rekkevidde"},
     {"type":"address","label":"Adresse"},
     {"type":"date","label":"Dato"}
   ]'::jsonb),
  -- inquiry_only: for prosjekter som krever manuell pris
  ('cctanktransport', '[]'::jsonb);

update booking_flows set inquiry_only = true where service_slug = 'cctanktransport';
```

**Kunden redigerer i admin** — kan endre rekkefølge, legge til/fjerne steg, eller toggle `inquiry_only` per tjeneste. Aldri kode-endring. (Matcher §33 hardkoding-forbud.)

### Komponent-skjema

```tsx
// src/components/booking/BookingWidget.tsx
'use client'
import { StepIndicator } from './StepIndicator'
import { ServicePicker } from './steps/ServicePicker'
import { stepComponents } from './steps'  // map: step.type → renderer

interface BookingWidgetProps {
  /** Hvis satt: hopper over trinn 0 (tjeneste-valg) */
  prefilledServiceSlug?: string
  /** hero | embed | fullscreen */
  variant: 'hero' | 'embed' | 'fullscreen'
}

export function BookingWidget({ prefilledServiceSlug, variant }: BookingWidgetProps) {
  const [serviceSlug, setServiceSlug] = useState(prefilledServiceSlug ?? null)
  const [stepIndex, setStepIndex] = useState(prefilledServiceSlug ? 0 : -1)  // -1 = trinn 0
  const [data, setData] = useState<BookingDraft>({})

  const flow = useBookingFlow(serviceSlug)  // henter steps fra booking_flows-tabell

  // Inquiry-only fallback (prosjektbasert pris)
  if (flow?.inquiry_only) {
    return <InquiryForm serviceSlug={serviceSlug} variant={variant} />
  }

  return (
    <section data-signature-moment="booking-widget" className={cn(variantStyles[variant])}>
      <StepIndicator steps={['Tjeneste', ...flow.steps.map(s => s.label), 'Pris', 'Bekreft']} current={stepIndex + 1} />

      {stepIndex === -1 && (
        <ServicePicker onPick={(slug) => { setServiceSlug(slug); setStepIndex(0) }} />
      )}

      {stepIndex >= 0 && stepIndex < flow.steps.length && (() => {
        const step = flow.steps[stepIndex]
        const Renderer = stepComponents[step.type]
        return <Renderer data={data} onComplete={(v) => { setData({...data, ...v}); setStepIndex(stepIndex + 1) }} />
      })()}

      {stepIndex === flow.steps.length && (
        <PriceReveal data={data} serviceSlug={serviceSlug} onConfirm={() => setStepIndex(stepIndex + 1)} />
      )}

      {stepIndex === flow.steps.length + 1 && (
        <ConfirmAndSubmit data={data} serviceSlug={serviceSlug} />
      )}
    </section>
  )
}
```

### Step-renderer-bibliotek

Hver step-type har sin egen renderer-komponent — gjenbrukbar på tvers av tjenester:

```tsx
// src/components/booking/steps/index.ts
export const stepComponents: Record<string, ComponentType<StepProps>> = {
  container_size: ContainerSizeStep,
  machine_type:   MachineTypeStep,
  weight:         WeightStep,
  crane_hours:    CraneHoursStep,
  reach:          ReachStep,
  tank_volume:    TankVolumeStep,
  tank_content:   TankContentStep,
  lease_period:   LeasePeriodStep,
  address:        AddressStep,       // gjenbrukes for både enkel og fra/til
  address_from:   AddressFromStep,
  address_to:     AddressToStep,
  date:           DateStep,
}
```

Ny step-type krever: ny renderer + DB-CHECK-constraint-update på `booking_flows.steps` (eller helt åpent jsonb). Kunden velger eksisterende steg fra admin-dropdown — IKKE hardkodet liste i kode.

### Embed-logikk per plassering

| Plassering | `prefilledServiceSlug` | Trinn 0 vises? |
|---|---|---|
| Forsiden (`/`) | `undefined` | Ja — bruker velger tjeneste |
| `/avfallscontainer` | `"avfallscontainer"` | Nei — direkte til trinn 1 |
| `/maskintransport` | `"maskintransport"` | Nei — direkte til trinn 1 |
| `/kraning` | `"kraning"` | Nei — direkte til trinn 1 |
| `/<sted>/<tjeneste>` | `<tjeneste>` + prefilledLocation | Nei — direkte til trinn 1 med adresse pre-fylt |
| `/bestill` | `undefined` | Ja — full wizard |

### Inquiry-only fallback (når sanntidspris ikke gir mening)

For tjenester der pris MÅ regnes manuelt (prosjekt-basert, komplekse spesialtransporter):

```ts
// booking_flows: inquiry_only = true
// Widget rendrer InquiryForm i stedet for step-flyt:
//   - Telefon-input
//   - Adresse
//   - Beskrivelse av oppdrag
//   - "Vi kontakter deg innen 24t"-løfte
//   - Submit → settings.contact_email + Resend
```

Brukerflyt er identisk på overflaten (samme widget, samme styling) — bare med Inquiry-form i stedet for sanntidspris. Konsistent UX, riktig forventning per tjeneste.

### Verifisering

`check-public-pages.ts` får ny K5:
- **K5:** Hver tjenesteside har riktig step-flyt-type. Heuristikk: åpne `/<tjeneste-slug>`, scroll til `[data-signature-moment]`, sjekk at første visible step matcher forventet step-type fra `booking_flows[slug].steps[0]`. Hvis container-velger vises på en maskintransport-side → failed.

### Anti-patterns

- ❌ Hardkode `if (slug === '/avfallscontainer') showContainerStep()` — bruker `booking_flows`-tabell i stedet
- ❌ Én komponent som spør «velg container» på alle tjenestesider
- ❌ Skille widget i 6+ helt separate komponenter — bruk én polymorf med step-bibliotek
- ❌ Hardkodet liste av steps i frontend-koden — alt fra DB
- ❌ Ingen inquiry-fallback for tjenester der pris-flyt ikke gir mening

---

## Variant-konvensjon

Signature moment-komponenten skal støtte minst 3 variants:

```ts
type SignatureMomentVariant = 'hero' | 'embed' | 'fullscreen'

interface BookingCalculatorProps {
  variant: SignatureMomentVariant
  prefilledLocation?: string
  prefilledServiceId?: string
  containerTypes: ContainerType[]
}
```

- `hero`: forsiden — max bredde, dominant, scroll-cue
- `embed`: tjenestesider + sted×tjeneste — kompakt, samme funksjon, mindre dominant
- `fullscreen`: dedikert `/bestill` — full skjerm, ekstra hjelp, lengre forklaring

---

## Sprint-spesifikasjon (kopier inn i sprinten som leverer signature moment)

```markdown
**Sprint X — Signature moment leveranse (§5)**

Implementer signature moment iht. PROJECT.md §5. Komponenten MÅ være på ALLE plasseringer fra §5.2 før sprinten kan rapporteres ferdig.

- [ ] Bygg komponenten med `data-signature-moment="<navn>"` attribute på rot-element
- [ ] Implementer 3 variants: `hero`, `embed`, `fullscreen`
- [ ] Plasser `hero`-variant på forsiden over fold (boundingBox.y < 1000)
- [ ] Plasser `embed`-variant på alle sekundær-sider fra §5.2 (DB-drevet via `show_signature_moment`-flagg, IKKE hardkodet)
- [ ] Plasser `fullscreen`-variant på dedikert side fra §5.2 (typisk `/bestill`)
- [ ] Embed på alle sted×tjeneste-sider (hvis aktuelt — typisk hele §SEO-discovery-griden)
- [ ] Implementer alle Plausible-events fra §5.5 — faktisk fyrer, ikke bare definert
- [ ] Sprint done-criteria: K1-K4 i `check-public-pages.ts` grønne
```

---

## Verifisering

`scripts/check-public-pages.ts` (rubrikkens K-kategori) gjør 4 sjekker:

```ts
// K1: komponenten eksisterer
const componentExists = (await page.locator('[data-signature-moment]').count()) > 0

// K2: synlig på primær over fold
await page.goto(BASE_URL)
const sigBox = await page.locator('[data-signature-moment]').first().boundingBox()
const onPrimaryAboveFold = sigBox && sigBox.y < (viewport.height + 200)

// K3: på alle sekundær-plasseringer (hentes fra DB-flagg show_signature_moment)
const { data: secondaryPages } = await supabase
  .from('pages')
  .select('slug')
  .eq('show_signature_moment', true)
let allSecondaryOk = true
for (const p of secondaryPages) {
  await page.goto(`${BASE_URL}${p.slug}`)
  if ((await page.locator('[data-signature-moment]').count()) === 0) {
    allSecondaryOk = false
    break
  }
}

// K4: anti-template-grep
const { passed } = await runShellCheck('scripts/check-anti-template.sh')
```

---

## For AI

Når en sprint leverer signature moment:

1. **Bygg komponenten ÉN gang, plasser den FLERE steder.** Bruk `variant`-prop for å justere størrelse/oppførsel.
2. **Tagg ALLTID med `data-signature-moment="<navn>"`** på rot-elementet. Dette er hva Playwright leter etter.
3. **Plasser på primær FØR du gjør noe annet.** Ikke bygg «/bestill» først og glem forsiden — det er det Container Consult-feilen.
4. **Sekundær-plasseringer via DB-flagg, ikke hardkoding.** `pages.show_signature_moment` toggles i admin.
5. **Aldri rapporter sprint ferdig** uten å se K1-K4 grønne i rapporten.
6. **Hvis prosjektet har flere tjenestetyper (services.length > 1)**: signature moment MÅ være polymorf. Trinn 0 = tjeneste-valg (skip ved embed med `prefilledServiceSlug`). Trinn 1-N drives av `booking_flows`-tabell per tjeneste. Inquiry-only fallback for tjenester som krever manuell pris. Aldri embed én generisk kalkulator som spør om feil ting på feil side.
