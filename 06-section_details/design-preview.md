# Design Preview — kundegodkjenning FØR Sprint 1

I dag ser kunden ingenting før uke 3–4 og masse kode er bygget. Hvis designretningen er feil oppdager vi det FOR sent.

Design Preview er en statisk HTML-side med **hero + nav + footer i valgt 2026-retning** med ekte palette + typografi. Deployes til `preview-<slug>.mlit.no` etter Sprint 0. Kunden godkjenner stilen FØR Sprint 1 starter.

## Hvorfor

- **Tidlig signal:** Hvis kunden hater Editorial Bento, lær det i uke 1, ikke uke 4.
- **Reduserer scope-creep:** Når stilen er godkjent skriftlig, kan ikke kunden bytte til Brutalism halvveis ut i Sprint 3.
- **Konkret beslutningspunkt:** «Godkjent» = vi har milepælen vi kan referere til.
- **Lav teknisk kost:** Én statisk HTML/CSS-side, ingen DB, ingen Auth — 30 min å lage.

## Hva preview-en SKAL ha

1. **Hero-seksjon** (full viewport-høyde)
   - Overskrift fra `vision` eller AI-generert med riktig brand voice
   - Underoverskrift med samme tone
   - 1-2 CTA-er
   - Visuell stil i valgt retning (gradient, bento, brutalism, etc.)
2. **Nav** med 5-7 hovedlenker
3. **3 «mid-page»-seksjoner**
   - Feature/USP-grid
   - Et statement (sitat eller storverdi)
   - Et «proof»-element (testimonial eller logo-rad eller stat-counter)
4. **Footer**
   - Kontaktinfo
   - Sosiale lenker
   - **«Utviklet av Myrvoll-Lunde IT Drift»** → mlit.no
5. **Light/dark toggle** hvis retningen har begge varianter
6. **Responsivt** (mobile + desktop)

## Hva preview-en IKKE har

- Routing (kun én side)
- Auth, CMS, DB
- Faktisk innhold fra kunde (bruk «Lorem ipsum» eller placeholder — dette er om STIL, ikke INNHOLD)
- Backend-integrasjoner

## Teknisk

- Ren HTML + CSS (eventuelt med en script-tag for theme-toggle)
- Ingen build-step nødvendig
- Deploys som static asset til Vercel under subdomene `preview-<slug>.mlit.no`
- Maks 200 kB total (inkl. en hero-bilde-AVIF)

## Mal-fil

Se `EXAMPLES/templates/design-preview.html.example` for komplett startpoint per designretning. AI velger riktig variant basert på `design_style[0]`.

## Godkjennings-flyt

1. Sprint 0 CLI deployer preview
2. Kickstart-modulen sender mail til `contact_email` med link + tekst:
   > Hei [navn], her er den foreslåtte stilen for [prosjekt]. Klikk på lenken og gi tilbakemelding — endringer på dette tidspunktet koster ingenting, men senere blir det dyrere. Godkjenn ved å svare «Godkjent» på denne mailen.
3. Helge følger opp om kunden ikke svarer innen 48 timer
4. Når kunden svarer «Godkjent», lagrer kickstart-DB `design_approved_at` + `design_approved_by`
5. Sprint 1 kan ikke startes uten `design_approved_at` (CLI-sjekk)

## Iterasjonsregel

Kunden får **2 runder med endringer gratis** på preview-en. Etter det er det betalt arbeid (faktureres som design-konsulent-timer). Dette står i PROJECT.md §7 (Kunde og kontekst).

## Anti-pattern

- Skip preview-en for å «spare tid» → garantert dyrere senere
- Bygg preview i hele Next.js-rammeverket → tar 4 timer i stedet for 30 min
- Bruk default Tailwind-klasser uten å vise valgt retning → kunde tror «modern» er standard
