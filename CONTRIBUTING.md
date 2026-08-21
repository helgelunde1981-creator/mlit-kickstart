# Slik jobber vi i dette repoet

## Branch

Standardbranch er `main`. Arbeid på egen branch og merge inn:

    <type>/<kort-beskrivelse>

`type` er en av `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `wip`.

## Commit

Conventional Commits: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`.
Én logisk endring per commit. Meldingen forklarer **hvorfor** — diffen viser hva.
Meldinger skrives på norsk, som resten av repoet.

Stage filer eksplisitt med full sti. Aldri `git add -A` eller `git add -u` —
i et repo der flere økter kan jobbe samtidig sluker de hverandres ucommittede filer.

Ingen commit med røde sjekker.

## Før du melder ferdig

- [ ] Endringen er lest gjennom i sin helhet
- [ ] Ingen hemmeligheter i diffen

Dette repoet har ingen automatiske sjekker — innholdet er tekst, ikke kode.
Da er gjennomlesing selve porten. Er du i tvil om en endring er riktig, si det
i stedet for å la den passere.

## Beslutninger med konsekvenser

Bibliotek, arkitektur eller mønstre andre må følge: skriv en ADR i [docs/adr/](./docs/adr/).
