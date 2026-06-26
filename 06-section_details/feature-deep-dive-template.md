# Section Detail — Feature Deep-Dive Template

> Brukes når AI fyller ut seksjon 10 (Funksjonell spec + deep-dives) i PROJECT.md. Hver kjernefeature MÅ ha en deep-dive.

---

## Definisjon: kjernefeature

En feature regnes som kjerne hvis minst én av disse er sann:

- Den er **signature moment** (definert i seksjon 5)
- Den driver hovedkonverteringen (eks: kalkulator, bestilling, signup-flyt)
- Den er nevnt eksplisitt i kundens visjon
- Den involverer minst 3 forskjellige UI-flater eller stater
- Den krever egen datamodell (dedikerte tabeller)

For hver kjernefeature, lag én deep-dive med strukturen under.

## Mal

```markdown
### 10.X [Feature-navn]

#### 10.X.1 Hva er det

[Én setning beskrivelse av feature-en for en utenforstående.]

#### 10.X.2 Hvor finnes den

| Flate | URL | Versjon |
|-------|-----|---------|
| Kompakt | / | Forside-widget |
| Full | /beregn-pris | Full-versjon |
| Embed | (eksternt) | Widget-versjon |

#### 10.X.3 Inputs

For hver input:

| Felt | Type | Required | Default | Validering |
|------|------|----------|---------|------------|
| [...] | [...] | [...] | [...] | [...] |

#### 10.X.4 Outputs

Hva ser brukeren? Hvilken state oppstår?

- Resultat-visning (live-oppdatering eller submit-respons?)
- Breakdown (hvis pris/kalkulasjon)
- CTA(er) som vises etter resultat

#### 10.X.5 Algoritme / forretningslogikk

Pseudokode eller flytdiagram. Vis hvordan inputs blir outputs.

\`\`\`ts
function calculate(input: Input): Output {
  // 1. ...
  // 2. ...
  // 3. ...
}
\`\`\`

**Edge cases:**

- [Hva skjer ved tom input?]
- [Hva skjer ved ekstreme verdier?]
- [Hva skjer ved nettverksfeil?]
- [Hva skjer ved ugyldig kombinasjon?]

#### 10.X.6 States

Alle states feature-en kan være i:

- `initial` — første render, ingen input
- `loading` — input mottatt, prosesserer
- `success` — resultat klart, brukeren ser output
- `error` — feil oppstod, vis feilmelding
- `empty` — input gyldig men ingen treff (hvis aktuelt)
- `submitting` — bruker har klikket "bestill", venter på server

For hver state:
- Hvilke elementer vises?
- Hvilke skjules?
- Hva er focus?
- Hvilke transitions inn/ut?

#### 10.X.7 Datamodell

Tabeller som feature-en skriver/leser fra. Refererer til seksjon 12.

\`\`\`sql
-- Snapshot ved hver bestilling for audit
create table public.feature_submissions (
  id uuid primary key default gen_random_uuid(),
  inputs jsonb not null,
  outputs jsonb not null,
  customer_id uuid references public.profiles(id),
  created_at timestamptz default now()
);
\`\`\`

#### 10.X.8 UX-detaljer

- **Live-oppdatering** debounced 200ms ved input-endring
- **Mobil-tilpasning**: breakdown kollapser til bottom-sheet
- **Tilgjengelighet**: keyboard-nav gjennom alle inputs, ARIA-live på resultat
- **Reduced motion**: animasjoner skrus av; resultat oppdateres uten transition
- **Loading-indikator**: "live"-prikk endrer farge ved beregning

#### 10.X.9 Validering

- **Client-side**: Zod-skjema (felter, kombinasjoner)
- **Server-side**: re-validering ved Server Action
- **Audit**: alle inputs lagres som snapshot (uavhengig av senere endringer i prisliste/regler)

#### 10.X.10 Suksessmetrikk

Hvordan måler vi at feature-en fungerer?

- **Conversion rate**: fra "feature loaded" til "bestilling sendt"
- **Drop-off-step**: hvilke steg taper vi brukere på?
- **Tid**: median tid fra første interaksjon til submit
- **Feilrate**: % submissions som ender i error-state

#### 10.X.11 Test-scenarier (Playwright E2E)

Minst:

- [ ] Happy path: full kalkulasjon → bestilling → bekreftelse
- [ ] Edge: tom input gir riktig empty state
- [ ] Edge: ugyldig kombinasjon gir korrekt feilmelding
- [ ] Edge: mobil-versjon — breakdown kollapser riktig
- [ ] Edge: reduced-motion — animasjoner skrudd av
- [ ] Edge: nettverksfeil — retry + feilmelding
- [ ] a11y: tab gjennom alle inputs i logisk rekkefølge
```

## Eksempel: budogvare fraktkalkulator

Se `EXAMPLES/full-specs/budogvare-spec.md` §5 for et utfylt eksempel på denne malen.
