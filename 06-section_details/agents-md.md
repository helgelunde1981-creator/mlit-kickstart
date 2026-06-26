# AGENTS.md — kontrakten mellom prosjektet og AI-agenter

Hver gang en ny AI-økt (Claude, Codex, andre) åpner prosjektet, leser den `AGENTS.md` i rot — det er **den første kontrakten**. Standardene er allerede i `~/.claude/CLAUDE.md` globalt, men `AGENTS.md` peker eksplisitt på prosjekt-spesifikke kilder.

## Hvorfor

Vi har sett at AI ofte:
1. Begynner å gjette på arkitektur før den leser `PROJECT.md`
2. Glemmer å lese `~/.claude/projects/.../memory/MEMORY.md`
3. Glemmer å sjekke `docs/standards/` for siste regler
4. Forsøker å bygge nye mønstre fremfor å finne eksisterende

`AGENTS.md` tvinger en lese-rekkefølge før første edit.

## Mal-fil

Se `EXAMPLES/templates/AGENTS.md.example` for komplett startpoint. AI/CLI fyller inn placeholders.

## Innholdskrav

`AGENTS.md` MÅ ha disse seksjonene:

1. **Før du gjør noe** — eksplisitt lese-rekkefølge
2. **Source-of-truth-hierarki** — hvor regler bor, og hvilken som overstyrer hva
3. **Aldri-listen** — handlinger som krasjer prosjektet (dev-server, force-push, etc.)
4. **Sjekk-først-listen** — verifiser disse før du tror noe er nytt
5. **Stack-snapshot** — Next.js version, Tailwind version, key packages
6. **Conventions** — branch-flyt, commit-format, migrasjonsfilnavn

## Placering

- I prosjekt-roten (samme nivå som `package.json`)
- Inkluderes via Codex/Claude Code's automatisk AGENTS.md-discovery
- Committet til repo (ikke `.gitignored`)

## Vedlikehold

Hver gang en ny regel legges til (f.eks. ny global CLAUDE.md-regel, nytt standardfragment), AI oppdaterer `AGENTS.md` slik at den peker dit. Stale `AGENTS.md` er verre enn ingen.
