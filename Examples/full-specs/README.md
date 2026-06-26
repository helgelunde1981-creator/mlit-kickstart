# Full Specs — Referanse-PROJECT.md

> Komplette `PROJECT.md`-filer fra reelle prosjekter. AI kan inkludere disse som few-shot eksempler i Claude API-call for å forsterke output-kvalitet og struktur.

## Filer

| Fil | Prosjekt | Designretning | Hvorfor referanse |
|-----|----------|---------------|-------------------|
| `budogvare-spec.md` | Skedsmo Bud & Vare | Dark Luxury Motion | Master example — har feature-deep-dive (kalkulator), full datamodell med RLS, sprintplan, alt |

## Bruksmønster

Når kickstart-verktøyet genererer ny PROJECT.md, kan det inkludere relevante utdrag (ikke hele filer) fra disse som few-shot. Strategi:

- **Hvis prosjekt har kalkulator-feature:** inkluder §5 fra budogvare-spec.md
- **Hvis prosjekt har CMS:** inkluder §4 fra budogvare-spec.md (datamodell)
- **Hvis sprintplan har > 8 sprinter:** inkluder §9 fra budogvare-spec.md

Token-kost vurderes mot value: budogvare-spec.md er ~12k tokens. Inkluder kun relevante seksjoner, ikke hele.

## Vedlikehold

- Legg til nye full-specs etter hvert som vi leverer nye prosjekter med ulike designretninger
- Mål: minst én full-spec per av de 9 design-retningene
- Når vi har 3+ full-specs, lag et utvelgelses-heuristikk i `generate.ts`

## Note for AI

Disse er **referanser**, ikke templates som skal kopieres ordrett. Strukturen kan og bør tilpasses kundens behov. Det viktige er fullstendigheten og kvaliteten — alle seksjoner adressert, ingen vagt språk, alle terskler målbart.
