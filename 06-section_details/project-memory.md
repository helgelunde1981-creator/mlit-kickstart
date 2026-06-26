# Project Memory — Claude-lokalt prosjekt-minne

Hvert nytt Myrvoll-Lunde IT Drift-prosjekt skal ha en standardisert **Claude memory-mappe** på utviklerens lokale PC, slik at AI-en husker prosjekt-spesifikke fakta på tvers av økter — uten å duplisere det som allerede står globalt.

## Hvorfor

Uten standard har vi sett to feilmoduser:

1. **For lite memory:** AI glemmer prosjekt-fakta (Supabase-ID, klient, branch-flyt) og må spørre på nytt hver økt.
2. **For mye memory:** Hvert prosjekt duplikerer globale regler («aldri commit til main», «alltid CLI ikke MCP»). Når en global regel endres, må vi oppdatere N prosjekter — som vi aldri rekker.

Standarden under er **kun for det som ikke kan utledes fra koden** og **ikke er dekket av global CLAUDE.md**.

## Mappestruktur

Memory bor lokalt på Helge's Windows-PC, ikke i repoet:

```
~/.claude/projects/c--Github-<ProsjektNavn>/memory/
├── MEMORY.md                          # Indeks — én linje per memory-fil
├── project_stack.md                   # Tier 1 — infra-IDer (Supabase, Vercel)
├── project_klient.md                  # Tier 1 — klient/kontakt/org
├── project_secrets.md                 # Tier 1 — hvor secrets bor
├── project_auth.md                    # Tier 2 — auth, roller, RLS-mønster
├── project_scope.md                   # Tier 2 — multi-tenancy / scope
├── project_integrasjoner.md           # Tier 2 — eksterne tjenester
└── (feedback_*.md / reference_*.md legges til underveis)
```

Stien er strikt: `c--Github-<ProsjektNavn>` (bindestrek erstatter skråstrek, ingen escaping).

## De 4 tiers — hva skal stå i memory

### Tier 1 — Prosjekt-fakta som ikke kan utledes fra koden

Disse oppdateres sjelden (kun ved infra-bytte).

| Memory | Innhold |
|--------|---------|
| `project_stack.md` | Supabase project-ref (prod + dev hvis branching), Vercel team-ID + project-ID, custom domener (prod, staging), repo-URL |
| `project_klient.md` | Klient-navn, primær-kontakt, kontakt-epost, telefon, org.nr (hvis fakturering), Vault-notat-lenke |
| `project_secrets.md` | Hvor hver secret-kategori lever (`.env.local` vs Vercel vs GitHub Secrets vs Supabase Vault), navn på Vercel-team, bash-snippet for å sette en ny secret |

### Tier 2 — Arkitektur-valg som ikke er åpenbare fra koden

Oppdateres ved arkitekturskifter (sjelden).

| Memory | Innhold |
|--------|---------|
| `project_auth.md` | Auth-provider (Supabase Auth? Custom?), MFA på/av, rolle-hierarki, RLS-mønster (`auth.uid()`, helper-funksjoner, security definer), edge-funksjoners JWT-strategi |
| `project_scope.md` | Multi-tenancy-modell: team-scoped vs user-scoped vs single-tenant. Hvilke tabeller har `team_id`, hvilke har `user_id`. Hvordan impersonation fungerer |
| `project_integrasjoner.md` | Resend (domener, fra-adresser), n8n-webhook hvis brukt, Stripe (account-ID, live/test), Tripletex, Vipps, Bring, etc. — én linje per tjeneste med URL + nøkkelfaktum |

### Tier 3 — Feedback (legges til underveis når Claude gjør feil)

`feedback_<kort_navn>.md` — opprettes når Helge korrigerer Claude med noe som er **prosjekt-spesifikt** (ikke globalt).

Format:
```markdown
---
name: feedback-<kort_navn>
description: <én linje for å avgjøre relevans>
metadata:
  type: feedback
---

<Regelen>

**Why:** <Hvorfor — gjerne en hendelse>
**How to apply:** <Når/hvor regelen kicker inn>
```

**Test for om en feedback hører hjemme her:**
- Er regelen dekket av `~/.claude/CLAUDE.md` eller `~/Documents/GitHub/CLAUDE.md`? → Ikke skriv her, oppdater global.
- Er regelen avledbar fra koden (lint, types, struktur)? → Ikke skriv, fiks koden.
- Er regelen kun for dette prosjektet? → Skriv her.

### Tier 4 — Reference (eksterne pekere)

`reference_<navn>.md` — pekere til Linear, Notion, Grafana, dashboards, runbooks, etc.

## Hva som IKKE skal være i memory

**Disse er forbudte:**

1. **Status-rapporter** («Sprint 3 ferdig», «Phase 2 90% complete») — disse ligger i PROJECT.md, repo-issues, eller PR-beskrivelser. Memory blir foreldet umiddelbart.
2. **Code patterns / file paths** («ConfirmDialog props er X», «Reviews-tabell har Y-kolonner») — disse leses fra koden direkte hver gang.
3. **Hva som er ferdig / hva som gjenstår** — bruk TodoWrite eller plan-dokumenter.
4. **Globale regler** — disse bor i `~/.claude/CLAUDE.md` og `~/Documents/GitHub/CLAUDE.md`.

**Tommelfingerregel:** Hvis det endres oftere enn én gang per måned, hører det ikke i memory.

## MEMORY.md-formatet

`MEMORY.md` er en **index**, ikke en memory. Maks ~25 linjer. Ingen frontmatter.

```markdown
# <ProsjektNavn> — Project Memory Index

> Globale regler ligger i ~/.claude/CLAUDE.md + ~/Documents/GitHub/CLAUDE.md.
> Her står kun det som er spesifikt for dette prosjektet.

## Tier 1 — Prosjekt-fakta
- [Stack & infra](project_stack.md) — Supabase ref, Vercel team+project, domener
- [Klient & kontakt](project_klient.md) — kunde, kontakt, org.nr hvis relevant
- [Secrets-strategi](project_secrets.md) — hvor secrets bor

## Tier 2 — Arkitektur
- [Auth & roller](project_auth.md) — Supabase Auth, MFA, roller, RLS
- [Scope](project_scope.md) — multi-tenancy, team_id vs user_id
- [Integrasjoner](project_integrasjoner.md) — Resend, n8n, Stripe, etc.

## Tier 3 — Feedback
<legges til når Helge korrigerer noe prosjekt-spesifikt>

## Tier 4 — Reference
<Linear, Notion, Grafana, runbooks>
```

## Memory-bootstrap i nye prosjekter

`PROJECT.md` skal ha en egen seksjon (§21 — se `01-SPEC_TEMPLATE.md`) som inneholder:

1. Ferdig utfylt MEMORY.md (med ekte Supabase-ID, Vercel-team, klient-navn fra wizarden)
2. Ferdig utfylt `project_stack.md`
3. Ferdig utfylt `project_klient.md`
4. Ferdig utfylt `project_secrets.md` med default-strategi
5. Skjelett for `project_auth.md`, `project_scope.md`, `project_integrasjoner.md` som lokal Claude fyller inn i Sprint 0

I første lokale Claude-økt på prosjektet, skal Claude:

1. Sjekke om `~/.claude/projects/c--Github-<ProsjektNavn>/memory/MEMORY.md` finnes
2. Hvis ikke: opprett mappa og kopier alle seks filene fra §21 i PROJECT.md
3. Bekreft til Helge med én linje

## Sjekkliste — er memory-mappa OK?

- [ ] `MEMORY.md` finnes og er under 25 linjer
- [ ] Tier 1-filene (stack, klient, secrets) er fylt med konkrete verdier — ikke `<TBD>`
- [ ] Ingen entry duplikerer global CLAUDE.md
- [ ] Ingen entry inneholder kode-patterns eller filstier
- [ ] Ingen entry inneholder status-rapporter eller phase-tracking
- [ ] Hver feedback-entry har **Why:** og **How to apply:**
