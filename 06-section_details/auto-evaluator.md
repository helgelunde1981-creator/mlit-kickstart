# Auto-evaluator — målbar 10/10 på hver PR

«10/10» i specen er en påstand. Auto-evaluatoren gjør det til et **fakta** — den kjører på hver PR til `main` og blokkerer merge hvis terskelen ikke nås.

## Hvorfor

Vi har sett at AI sniker seg gjennom selv med Prinsipp 4 og 19 regler. Når det ikke finnes en uavhengig sjekk, blir «ferdig» definert av samme AI som skrev koden. Auto-evaluatoren gir oss en uavhengig dommer.

## Pakken (8 leveranser per prosjekt)

Hver nye PROJECT.md sier eksplisitt at disse 8 filene skal være på plass:

1. `.github/workflows/quality-gate.yml` — GitHub Action som kjører ved push + PR
2. `scripts/check-lighthouse.ts` — Lighthouse ≥ 95 alle 4 kategorier på alle public-ruter
3. `scripts/check-a11y.ts` — axe-core på samme ruter, 0 critical/serious
4. `scripts/check-cms-coverage.ts` — grep for hardkodet kunde-tekst, scorer per side
5. `scripts/check-footer-credit.ts` — verifiserer «Utviklet av Myrvoll-Lunde IT Drift» på alle public-ruter
6. `scripts/check-email-rendering.ts` — render hver react-email-mal, verifiser DOM
7. `scripts/check-broken-links.ts` — interne lenker + JSON-LD URL-er + sitemap-koherens
8. `scripts/score-delivery.ts` — samler resultatene, scorer 0–10 per dimensjon, lagrer i `delivery_scores`-tabellen

## Terskler (per `04-QUALITY_GATES.md`)

| Dimensjon | Terskel | Hvis under |
|-----------|---------|------------|
| Lighthouse Performance | ≥ 95 | BLOKKER merge |
| Lighthouse Accessibility | ≥ 95 | BLOKKER merge |
| Lighthouse Best Practices | ≥ 95 | BLOKKER merge |
| Lighthouse SEO | ≥ 95 | BLOKKER merge |
| axe-core critical | 0 | BLOKKER merge |
| axe-core serious | 0 | BLOKKER merge |
| CMS-coverage (hardkodet kunde-tekst) | 0 funn | BLOKKER merge |
| Footer-credit på alle public-ruter | 100% | BLOKKER merge |
| Email-templates som rendrer uten feil | 100% | BLOKKER merge |
| Broken internal links | 0 | BLOKKER merge |
| JSON-LD valid schema | 100% | WARN |

## GitHub Action (skjelett)

Se `EXAMPLES/scripts/quality-gate.yml` for komplett mal. Kjørerekkefølge:

1. `pnpm build` (statisk eksport eller `next start` i bakgrunn)
2. `pnpm check:cms-coverage` (raskest — feiler tidlig)
3. `pnpm check:footer-credit`
4. `pnpm check:broken-links`
5. `pnpm check:email-rendering`
6. `pnpm check:lighthouse` (treigest — siste)
7. `pnpm check:a11y`
8. `pnpm score:delivery` (oppsumering)

Action poster delivery-score som PR-kommentar (markdown-tabell med ✓/✗ per dimensjon).

## Lokal kjøring

Hver utvikler skal kunne kjøre:

```bash
pnpm quality:check    # Kjører alle 7 sjekker mot lokalt bygg
pnpm quality:score    # Viser score-tabell
```

## Failure-håndtering

Når Action feiler, må PR-en fikse hver enkelt sjekk:
- **CMS-coverage**: Flytt hardkodet tekst til DB + admin-CRUD
- **Footer-credit**: Sjekk at `<MLITCredit />` er importert i alle layouts
- **A11y**: Fiks WCAG-feil (axe-output gir konkrete selectors)
- **Lighthouse**: Profil rapporten i workflow-artifacts
- **Email**: Sjekk at hver react-email-mal har default-export + valid props

## Hva auto-evaluatoren IKKE dekker

- Visuell kvalitet (krever menneske eller AI-design-review)
- Signature moment-implementering (krever spec → kode-diff-agent — egen seksjon)
- Brand voice (krever LLM-eval)

Disse dekkes manuelt i Sprint 2 og pre-launch.

## Anti-juks-mekanisme

Hver av scriptene skal **logge konkrete bevis** (URL-er, screenshots, lighthouse-rapport-link). Hvis et script returnerer ✓ uten å logge bevis, anses det som juks. CI fanger dette ved å sjekke at hver passert sjekk har minst 1 logget bevis-linje.
