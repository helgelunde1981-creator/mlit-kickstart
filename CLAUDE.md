# mlit-kickstart

Kildematerialet til kickstart-verktøyet: promptene, malene og kvalitetskravene som
brukes til å generere en komplett `PROJECT.md` for et nytt kundeprosjekt hos
Myrvoll-Lunde IT Drift.

**Dette er ikke et kode-repo.** Det finnes ingen `package.json`, ingen bygg og ingen
tester. Innholdet er prompt-tekst som leses av et verktøy og av mennesker.

## Struktur

Filene er nummererte fordi rekkefølgen er meningsbærende — de leses i denne rekkefølgen:

- `00-SYSTEM_PROMPT.md` — system-prompten til Claude API. Setter rollen og de tre
  fundamentale prinsippene som overstyrer alt annet
- `01-SPEC_TEMPLATE.md` — malen for den genererte `PROJECT.md`
- `02-DESIGN_DIRECTIONS.md` — designretningene det kan velges mellom
- `03-TECH_STACK_CANON.md` — hvilken stack som er standard, og når man avviker
- `04-QUALITY_GATES.md` — hva som må være på plass før noe regnes som ferdig
- `05-BRAND_VOICE.md` — språk og tone
- `06-section_details/` — utdypning per seksjon i specen (datamodell, CSP, compliance,
  analytics, branch protection m.m.)
- `Examples/` — ferdige specer, designretninger, workflows og anti-mønstre å lære av

## Konvensjoner

- Alt innhold på norsk bokmål.
- Endrer du et prinsipp i `00-SYSTEM_PROMPT.md`, sjekk om `04-QUALITY_GATES.md`
  fortsatt stemmer. De to henger tett sammen.
- Et nytt tema hører hjemme som egen fil i `06-section_details/`, ikke som et nytt
  avsnitt i en fil som allerede er lang.
- `Examples/anto-patterns` viser hva som *ikke* skal gjøres. Ikke rydd den bort —
  den er der med vilje.

## Fallgruver

- Filene ser ut som dokumentasjon, men er produksjonstekst: en endring her endrer
  hva verktøyet genererer for neste kunde. Behandle det som kode.
- Numrene i filnavnene er ikke bare sortering. Setter du inn en ny fil midt i,
  må du vurdere om rekkefølgen fortsatt gir mening når de leses etter hverandre.

## Referanser

- @03-TECH_STACK_CANON.md — standardstacken
- @04-QUALITY_GATES.md — definisjonen av ferdig
- @CONTRIBUTING.md — branch og commit
