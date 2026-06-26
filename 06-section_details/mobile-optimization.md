# Mobil-kvalitet 100 % (§39)

Mobil er primær viewport for 60-80 % av trafikken på norske kundeprosjekter.
«Det ser bra ut på desktop» er ikke en akseptabel rapport. Hver public-side i
hver sprint MÅ verifiseres på minst 3 mobile viewports via Playwright +
Claude Sonnet 4.6 vision-evaluator, og AI MÅ iterere fortløpende til snitt
9.0+ per side.

---

## Hva som testes

### Viewports (3 obligatoriske)

| Device | Viewport | Hvorfor |
|---|---|---|
| iPhone SE | 375×667 | Smallest moderne iPhone — fanger overflow tidligst |
| iPhone 13 | 390×844 | Vanligste iPhone-modell 2024-2026 |
| Galaxy S20 | 360×800 | Smaleste vanlige Android — fanger overflow |

(Pixel 6 / iPhone 11 kan legges til for ekstra dekning, men er sjelden
strengt nødvendig.)

### Per side: screenshot av HVER seksjon/modul

Ikke nok å ta én full-page screenshot. Scroll til hver `<section>`,
hver hero, hver CMS-modul, hver FAQ-accordion, hver pris-tabell, hver
booking-widget — og capture per modul. Vision-evaluatoren scorer per
modul, ikke per side. Slik fanger vi at modul 3 har horizontal overflow
selv om modul 1+2 ser fine ut.

### Scoring-rubrikk (M1-M6, 1-10 per dimensjon)

| Dim | Hva som testes | Auto-fail threshold |
|---|---|---|
| **M1 Layout** | Ingen horizontal scroll. Ingen overflow utenfor viewport. | M1 < 8 = critical |
| **M2 Tap-targets** | Alle knapper/lenker ≥44×44 px (iOS guideline). Padding rundt små ikoner. | M2 < 7 |
| **M3 Lesbarhet** | Brødtekst ≥14 px. Headinger skalert ned (ikke desktop-størrelse). Linjebredde 45-75 tegn. | M3 < 7 |
| **M4 Nav** | Mobilmeny åpner+lukker med touch. Hamburger synlig. Ingen desktop-dropdown synlig. | M4 < 8 = critical |
| **M5 Hero+CTA** | Primær CTA over fold (synlig uten scroll) i hver av de 3 viewports. Hero-bilde fyller ikke 100 % høyde. | M5 < 7 |
| **M6 Skjemaer** | Input-felt minst 16 px font (hindrer iOS zoom). Touch-keyboard skjuler ikke knappen. Autofill-attributter satt. | M6 < 7 |

**Snitt < 9.0 ELLER kritisk M1/M4-feil = sprint ikke ferdig.**

---

## Iterer fortløpende — det er regelen

AI MÅ kjøre `check-mobile-quality.ts` → lese rapport → fikse hvert
problem-modul → kjøre på nytt → fortsette til alle er 9+. Ikke
"rapporter feil og gå videre". Ikke "denne er nesten 9". Fix til 9+
før sprint kan rapporteres ferdig.

Når et problem identifiseres:
- M1 horizontal scroll: typisk overflow-x: hidden mangler, eller en
  hardkodet width større enn viewport, eller image uten max-width:100%
- M2 små tap-targets: legg til `min-h-[44px] min-w-[44px]` på ikon-knapper
- M3 tekst-overflow: bryt lange ord med `overflow-wrap: anywhere`,
  reduser font-size på h1/h2 for mobil
- M4 nav fungerer ikke: sjekk `'use client'`-direktiv, sjekk z-index,
  sjekk at sheet/drawer-komponent rendres på riktig sted
- M5 CTA under fold: reduser hero-height, flytt CTA opp
- M6 skjema-feil: `text-base` på input (gir 16px), `autoComplete`-attributter

---

## Skript-mal

Se [`EXAMPLES/scripts/check-mobile-quality.ts.example`](../EXAMPLES/scripts/check-mobile-quality.ts.example) for kjørbar mal.

### Kjør-flyt

```bash
# Lokalt mot dev-server
PLAYWRIGHT_BASE_URL=http://localhost:3000 \
ANTHROPIC_API_KEY=sk-ant-... \
  pnpm tsx scripts/check-mobile-quality.ts

# Mot Vercel preview
PLAYWRIGHT_BASE_URL=https://preview-xyz.vercel.app \
VERCEL_AUTOMATION_BYPASS_SECRET=$VERCEL_BYPASS \
ANTHROPIC_API_KEY=sk-ant-... \
  pnpm tsx scripts/check-mobile-quality.ts
```

### Rapport

Skrives til `docs/sprint-reports/sprint-<N>-mobile.md`:

```markdown
# Sprint N — Mobile Quality Report

Generated: 2026-05-31T08:00:00Z
Viewports: iPhone SE (375×667), iPhone 13 (390×844), Galaxy S20 (360×800)

## Sammendrag

- Sider testet: 12
- Moduler testet: 47
- Snitt M1-M6: 9.3 ✅
- Kritiske feil: 0 ✅

## Per side

### / (forside)
- iPhone SE: 9.5 ✅
- iPhone 13: 9.7 ✅
- Galaxy S20: 9.2 ✅

#### Modul: hero
- Score: 9.5
- Issues: ingen

#### Modul: services-grid
- Score: 8.8 ⚠
- Issues: M3 tekst (12px på iPhone SE → bør være 14+)
- Fix: økt til text-sm (14px) i ServicesGrid.tsx:42
```

---

## CI-integrasjon

Legges til `.github/workflows/quality-gate.yml`:

```yaml
- name: Mobile quality check
  run: pnpm tsx scripts/check-mobile-quality.ts
  env:
    PLAYWRIGHT_BASE_URL: ${{ steps.vercel.outputs.preview-url }}
    VERCEL_AUTOMATION_BYPASS_SECRET: ${{ secrets.VERCEL_AUTOMATION_BYPASS_SECRET }}
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

Exit 1 fra skriptet = PR-blokk.

---

## Sprint-readiness-integrasjon

`scripts/check-sprint-readiness.ts` krever output-stempel fra
mobile-rapporten i `docs/sprint-reports/sprint-<N>-final.md`:

```markdown
## Check-skript-output

### check-mobile-quality.ts
- Status: ✅ Pass (snitt 9.3, 0 kritiske)
- Rapport: docs/sprint-reports/sprint-N-mobile.md
- Sider: 12, Moduler: 47, Viewports: 3
```

Manglende mobile-output = `check-sprint-readiness.ts` exit 1.

---

## Hvorfor dette er nødvendig

- 60-80 % av trafikk på norske B2C-sider er mobil
- Lighthouse-mobile gir ikke modulær score per seksjon
- Desktop-fokusert utvikling skaper overflow + små tap-targets som
  Lighthouse ikke alltid fanger
- AI-vision-evaluatoren kan se hva en bruker faktisk opplever — ikke
  bare hva DOM sier

Tidligere prosjekter har "lansert" med ikoner som ikke kunne tappes,
hamburger-meny som ikke åpnet, eller priser som forsvant utenfor
skjermen. Med §39 skal det aldri skje igjen.
