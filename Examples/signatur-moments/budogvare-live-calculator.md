# Signature Moment — Budogvare Live Fraktkalkulator

> Faktisk levert eksempel fra budogvare.no.

## Hva

Live fraktkalkulator med Lucide-ikoner og real-time pris-breakdown. Bruker velger biltype, vekt, fra/til-sone, og tilvalg — pris oppdateres innen 200ms med detaljert linje-for-linje breakdown.

## Hvor

- **Forside-hero:** kompakt versjon (biltype + vekt + sone + estimat) — over the fold
- **`/beregn-pris`:** full versjon med alle tilvalg, breakdown, bestillingsflyt
- **Som widget:** kan embedes på spesifikke tjenester-sider

## Oppførsel

### States

- **Initial:** ingen input gitt — viser placeholder med "Velg biltype for å begynne"
- **Partial:** noen inputs gitt — viser "Velg flere felt for pris"
- **Calculating:** input nylig endret, debounced 200ms — viser pulserende grønn prikk + "Beregner..."
- **Ready:** pris klar — viser stor pris-tall + breakdown
- **Edge:** ugyldig kombinasjon (eks: vekt > 999 kg) — viser "Be om tilbud" med pre-fylt skjema
- **Submitting:** bruker klikker "Bestill" — viser loading + venter på server
- **Success:** bestilling mottatt — viser bekreftelse + e-post-besked
- **Error:** server-feil — viser "Kunne ikke sende, prøv igjen" + retry-knapp

### Transitions

- Input-endring: pris fade out → calculate → fade in (300ms)
- Mobile: breakdown kollapser til sticky bottom-sheet
- Reduced-motion: alle transitions skrudd av, pris bytter umiddelbart

### Edge cases

- **Bom/ferge:** vises som "faktiske utlegg kommer i tillegg" — kan ikke estimeres
- **Express:** krever km — hvis ikke fylt inn, vis "Express: 31 kr/km på toppen"
- **Helgetillegg-kombinasjon:** bruker høyeste prosent (eks: kveld + lørdag = 25%, ikke 50%)
- **Minimum timer:** alltid 2 timer minimum for timepris-biler
- **Ukjent kombinasjon:** "Be om tilbud"-flyt åpnes automatisk
- **HVO 100:** info-checkbox som flagger "tillegg på forespørsel"

## Hvorfor signature

Skedsmo Bud & Vare er ikke et generisk transportfirma — de leverer raskt på Romerike. Kjernen i forretningen er "vet du hva det koster før du bestiller?". 90% av konkurrentene har skjulte priser eller "kontakt oss for tilbud". En kalkulator som er ærlig, rask og fungerer på mobil signaliserer hele brand-løftet i ett interaktivt øyeblikk. Det er det som gjør produktet skille seg fra "yet another WordPress transport-site".

## Suksessmetrikk

- **Engagement rate:** % besøkende som interagerer med kalkulatoren (mål: ≥ 60%)
- **Completion rate:** % som fullfører alle felt og ser pris (mål: ≥ 40% av engagement)
- **Conversion rate:** % som klikker "Bestill" eller "Be om tilbud" etter å ha sett pris (mål: ≥ 15%)
- **Drop-off-step:** hvilket steg taper vi flest brukere på (instrumenter hvert steg-bytte)
- **Median tid:** fra første interaksjon til submit (mål: < 90 sekunder)
- **Mobile vs desktop ratio:** målsetning >50% mobile completions (alle skal kunne bestille mens de er på farten)

## Tekniske highlights

- **Pris-engine i `lib/pricing.ts`** med 100% test-dekning på alle PDF-eksempler
- **Server-side validering** av alle prisberegninger (klient kan ikke endre pris)
- **Audit-log** av alle inputs ved bestilling (snapshot av prisliste på det tidspunktet)
- **Alle prisendringer i CMS** reflekteres umiddelbart i kalkulatoren
