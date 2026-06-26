# Signature Moment — Engaging Onboarding First-Run

> Eksempel-mønster: en første-gangs-opplevelse som setter tonen for hele produktet.

## Hva

Når bruker logger inn for første gang, møter hen en kuratert mini-opplevelse — ikke en kjedelig "5-steg-tutorial". Eks: et animert oppdrag, et personlig spørsmål med visuell feedback, eller et "wow"-øyeblikk der produktet beviser sin verdi i 30 sekunder.

## Hvor

- **Etter første signup:** dedikert `/welcome`-rute (eller modal)
- **Re-trigget:** bruker kan re-se ved å gå til `/welcome` igjen

## Oppførsel

### States

- **Step 1:** velkomst med animert brand-element
- **Step 2:** kort spørsmål om bruker (eks: "Hva vil du gjøre først?") med store visuelle valg
- **Step 3:** produktet demonstrerer sin verdi basert på valget — animert demo, ikke statisk
- **Step 4:** CTA til første reell handling
- **Skippable:** alltid en "Hopp over"-link nederst (men ikke prominent)

### Transitions

- Steg-bytte: full-page fade + custom motion per steg
- Brand-element: dynamisk respons på cursor/touch
- Mobile: optimalisert for portrait, ingen forventning om mus

### Edge cases

- **Returning user:** redirect til dashboard hvis allerede onboarded
- **Reduced-motion:** simplere transitions, ingen "wow"-animasjoner
- **Slow connection:** preload alle assets, ingen layout shift mid-transition

## Hvorfor signature

Onboarding er ofte det første reelle møtet med produktet. En generisk "tutorial" signaliserer "vi er bare nok et SaaS-verktøy". En kuratert opplevelse signaliserer "noen tenkte på dette". Sterkt verdiløfte for premium-produkter, B2B med høy CAC, kreative verktøy.

Eksempel: Notion sin onboarding med personlig touch, Linear sin keyboard-shortcut intro, Figma sin "explore Figma"-flyt.

## Suksessmetrikk

- **Completion-rate:** % som fullfører alle 4 steg (mål: ≥ 70%)
- **Time-to-first-action:** etter onboarding, hvor lang tid til første reell handling (mål: < 30 sek)
- **Day-1 retention:** % som returnerer dag 2 (mål: ≥ 50%)
- **Day-7 retention:** % som returnerer 7 dager senere (mål: ≥ 30%)
- **Re-watch-rate:** % som re-besøker `/welcome` (signaliserer at det var en bra opplevelse)

## Tekniske highlights

- **Server Action** logger hvert steg + bruker-valg
- **Personalization** lagres i `profiles`-tabell (eks: `preferred_workflow`)
- **Resumable** — hvis bruker forlater midt i, fortsett der hen sluttet
- **A/B-test** ulike velkomst-meldinger
- **Analytics events** per steg for å identifisere drop-off
