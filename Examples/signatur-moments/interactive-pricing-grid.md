# Signature Moment — Interactive Pricing Grid (Bento)

> Eksempel-mønster: Bento-grid der hver celle er en interaktiv pris-konfigurator for én feature.

## Hva

Pris-siden er ikke en passiv tabell. Hver feature-celle i en bento-grid har egen toggle/slider som påvirker totalpris. Bruker bygger sin egen plan ved å aktivere/deaktivere features, og total oppdateres live øverst i gridet.

## Hvor

- **`/priser`:** dedikert pris-side
- **Som widget** på feature-sider (forenklet — kun den feature-en)

## Oppførsel

### States

- **Default:** anbefalt plan er pre-konfigurert
- **Customizing:** bruker toggler features, total animerer mellom verdier
- **Empty:** ingen features valgt — total = 0 + "Velg minst én feature"
- **Conflict:** feature A krever feature B — viser advarsel + auto-aktiverer dep
- **Max:** alle features valgt — viser "Enterprise"-banner med "Kontakt oss"-CTA

### Transitions

- Toggle: cell-state shift (off → on) med 300ms fade + lift
- Total: count-up-animasjon mellom verdier
- Dependency-auto-aktiver: subtle highlight på avhengig celle

### Edge cases

- **Mobile:** grid kollapser til vertical stack, total blir sticky bunn-bar
- **Reduced-motion:** ingen count-up, total bytter umiddelbart

## Hvorfor signature

Pris-sider er ofte den kjedeligste delen. Å gjøre den interaktiv signaliserer "modular fleksibilitet" — kunden ser bokstavelig talt at de kan tilpasse. Skiller seg fra "standard Pro/Team/Enterprise"-mønsteret som alle SaaS bruker.

## Suksessmetrikk

- **Customization rate:** % som endrer fra default config (mål: ≥ 40%)
- **Conversion til signup:** mål: ≥ 8% av interactions
- **Avg config:** hvilke features velges sammen (informasjons-arkitektur-input)
- **Drop-off ved total:** identifiser pris-grenser hvor folk slutter

## Tekniske highlights

- **Pris-engine** med all logikk client-side først, så server-side validert
- **URL-state:** `?features=a,b,c` lar bruker dele config
- **Dependency-graph** definert i config — ikke hardkodet
- **A/B-test** ulike default-configs
