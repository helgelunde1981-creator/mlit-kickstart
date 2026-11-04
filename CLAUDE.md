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

## Felles sikkerhetsstandard (innført 04.11.2026)

Gjelder alle Helges repoer. Selve workflowene (Gitleaks, Supabase-sjekker) og
migrasjonene kommer som egne PR-er fra den som eier standarden. Ikke lag dem
selv, og ikke skriv om dem når de er kommet.

**Gjelder allerede nå:**

- **Actions pinnes til full SHA.** Hver ny `uses:` i en workflow skrives med
  full 40-tegns commit-SHA og versjonskommentar, f.eks.
  `uses: actions/checkout@<40 tegn> # v5`. Aldri `@v5`, `@main` eller kort SHA.
  Renovate pinner eksisterende linjer. Pin-PR-ene automerges og skal ikke
  lukkes. En GitHub-regel som feiler workflows med upinnede actions kommer senere.
- **Genererte filer redigeres ikke for hånd.** Filer med headeren «generert»
  (bl.a. `supabase/tests/database/0N-*.test.sql`) endres bare av generatoren.
  Drift-sjekken feiler hvis de er endret.

**Kommer med standard-PR-en:**

- **Gitleaks på alle PR-er.** Finner den en hemmelighet i en ny commit, blir
  PR-en rød. Da: roter nøkkelen først (den kan være live), fjern den fra koden,
  og si fra til Helge. Offentlige verdier (`NEXT_PUBLIC_*`, `VITE_*`,
  anon-nøkkelen, Sentry DSN) gir ikke varsel.
- Sjekkene rapporterer bare i en prøveperiode, og blir deretter påkrevde i
  branch protection. Helge får beskjed før det slås på.

**Uendret:**

- Grenflyten slik den står beskrevet for repoet: PR, grønn CI, merge til `dev`,
  verifiser, så `main` (der repoet har `dev`). Ingen force push til `main` eller
  `dev`.
