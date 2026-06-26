# Sprint 0 Bootstrap — fra «PROJECT.md ferdig» til «klar for Sprint 1» på 5 minutter

Sprint 0 er den repeterende infra-jobben Claude (eller Helge) gjør i begynnelsen av hvert prosjekt. Den er identisk hver gang: GitHub repo, Vercel project, Supabase project, secrets på 3 steder, første migrasjon, basic CI, AGENTS.md, og memory-bootstrap.

Hvis vi gjør den manuelt, tar den 60–90 minutter og er sårbar for menneskelig feil. CLI-en gjør den deterministisk på 5 minutter.

## CLI: `create-mlit-project`

Ligger i `scripts/create-mlit-project.ts` i mlit-repoet. Helge kjører:

```bash
pnpm tsx scripts/create-mlit-project.ts --kickstart-id <uuid>
```

CLI-en gjør (i rekkefølge):

### Steg 1 — Les kickstart-data
Henter `kickstart_projects`-raden fra Supabase, validerer at PROJECT.md er ferdig generert.

### Steg 2 — Opprett GitHub repo
- Via `gh repo create helgelunde1981-creator/<slug> --public --description "..." --clone`
- Kopierer `EXAMPLES/templates/.gitignore`, `EXAMPLES/templates/README.md`
- Kopierer AGENTS.md med fylte placeholders
- Første commit + push

### Steg 3 — Opprett Supabase prosjekt
- Via Supabase Management API (`POST /v1/projects`)
- Region `eu-central-1`, instance size `nano`
- Lagrer `project-ref` lokalt
- Lager dev-branch hvis `requires_supabase_branching=true`

### Steg 4 — Kobler Vercel project
- Via Vercel CLI `vercel link --yes`
- Team: myrvoll-lunde-it-drift-s-projects (`<Vercel-team-ID>`)
- Setter custom domene fra wizarden
- Setter framework til Next.js

### Steg 5 — Distribuer secrets på 3 steder
For hver secret fra §14.5 i PROJECT.md:
- `.env.local` (lokalt)
- Vercel env (production + preview + development)
- GitHub Actions secret (`gh secret set`)

### Steg 6 — Første migrasjon + GitHub Actions
- Lager `supabase/migrations/<timestamp>_init.sql` basert på §12 i PROJECT.md
- Kopierer `EXAMPLES/github-workflows/supabase-migrations.yml`
- Kopierer `EXAMPLES/github-workflows/quality-gate.yml`

### Steg 7 — Lokal Claude memory bootstrap
- Oppretter `~/.claude/projects/c--Github-<slug>/memory/`
- Kopierer alle 7 filer fra §21 i PROJECT.md (med fylte placeholders)

### Steg 8 — Vault-registrering
- Oppretter `C:\Github\vault\prosjekter\<ProsjektNavn>.md`
- Oppretter eller oppdaterer `C:\Github\vault\klienter\<KlientNavn>.md`
- Oppdaterer `C:\Github\vault\Prosjektoversikt.md`

### Steg 8b — GDPR-behandlingsprotokoll (Myrvoll-Lunde IT Drift master)
- Registrer prosjektet i master-protokollen (single source of truth — ALDRI dupliser inn i repoet):
  `~/Documents/GitHub/mlit/docs/compliance/Behandlingsprotokoll-MLITDrift.md`
- Bestem rolle (egen=behandlingsansvarlig / klient=databehandler); kartlegg databehandlere + funksjoner fra `package.json`+`src`+`.env`
- Legg til rad i «Prosjektregister» (Supabase-ref + region + repo + rolle + Cookiebot-status) + riktig art.30-seksjon
- Før nye underdatabehandlere i «Underdatabehandlere» (+ speil i personvern); oppdater Forside-dato; commit til mlit-repoet
- Full prompt + mapping: `06-SECTION_DETAILS/compliance-norge.md` → «GDPR art. 30-behandlingsprotokoll»

### Steg 9 — Design preview
- Genererer statisk `preview.html` (se `06-SECTION_DETAILS/design-preview.md`)
- Deployer til `preview-<slug>.mlit.no`
- Sender link til kunde for godkjenning

### Steg 10 — Endrapport
Returnerer en oppsumering:
```
✅ Bootstrap fullført for <ProsjektNavn>
   GitHub:    https://github.com/helgelunde1981-creator/<slug>
   Vercel:    https://vercel.com/myrvoll-lunde-it-drift-s-projects/<slug>
   Supabase:  <project-ref>
   Preview:   https://preview-<slug>.mlit.no
   Memory:    ~/.claude/projects/.../memory/ (7 filer)
   Vault:     C:\Github\vault\prosjekter\<slug>.md

Neste steg: Helge godkjenner design-preview, så starter Sprint 1.
```

## Idempotens

CLI-en skal være **idempotent** — hvis en seksjon allerede er gjort (eksisterende repo, eksisterende prosjekt), hopper den over uten å feile. Krasj midt i bootstrap betyr at man kan kjøre på nytt og fortsette der det stoppet.

## Rollback

CLI-en lager en `bootstrap-state.json` underveis som logger hva som ble gjort. Hvis Helge vil rulle tilbake:

```bash
pnpm tsx scripts/create-mlit-project.ts --rollback <kickstart-id>
```

Sletter GitHub repo (med ekstra bekreftelse), pauser Supabase prosjekt, fjerner Vercel project. Memory + Vault beholdes alltid (de er dokumentasjon).

## Sikkerhetskrav

- CLI-en MÅ verifisere at den jobber mot riktig Vercel-team før den setter env vars
- CLI-en MÅ aldri push til `main` direkte — alltid via `dev`-branch
- CLI-en MÅ logge hver destruktive operasjon med tidsstempel
- Supabase service-role-keys MÅ aldri lagres i `bootstrap-state.json`
