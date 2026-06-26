# Section Detail — Vercel CLI + Secrets-flyt (3 steder samtidig)

> AI/Claude har full Vercel CLI-tilgang via Helge's auth-token. AI skal kjøre alt selv — ikke be brukeren om å gå i Dashboard. Secrets skal **alltid** settes tre steder samtidig: `.env.local` + Vercel + GitHub.

---

## 🚨 KRITISK regel — bruk `printf`, IKKE `echo`, når du setter secrets

`echo` legger til `\n` på slutten. Den newline-en blir med inn i Vercel env-vars og GitHub Secrets. Når koden så leser `process.env.MY_TOKEN`, får den `"ghp_abc123\n"` i stedet for `"ghp_abc123"`. API-er avviser dette med kryptiske feilmeldinger:

- Supabase: `400 Invalid organization slug`
- GitHub: `401 Bad credentials`
- Vercel: silent rejection eller `403`

**Bugen har skapt mye trøbbel** (advisors-cron, wizard-bootstrap, etc.) fordi feilmeldingene ikke peker på årsaken.

### ❌ ALDRI

```bash
echo "ghp_abc123" | npx vercel env add MY_TOKEN production --token "$TOKEN"
```

### ✅ ALLTID

```bash
# printf — ingen newline
printf "ghp_abc123" | npx vercel env add MY_TOKEN production --token "$TOKEN"

# Alternativ: echo -n
echo -n "ghp_abc123" | npx vercel env add MY_TOKEN production --token "$TOKEN"
```

### Verifisering (alltid etter setting)

```bash
npx vercel env pull /tmp/check --token "$TOKEN" --environment=production --yes
VAL=$(grep "^MY_TOKEN=" /tmp/check | cut -d= -f2- | tr -d '"')
echo "Lengde: ${#VAL}"  # Skal matche forventet, IKKE +1
rm /tmp/check
```

Hvis lengde er +1 → trailing newline. Slett og re-sett med `printf`.

### Defense-in-depth i koden

Trim env-vars når de leses:

```typescript
const token = process.env.MY_TOKEN?.trim()
```

Slik tåler koden også gamle env-vars som har newline fra tidligere oppsett.

---

## Vercel CLI — AI skal bruke direkte

Vercel CLI er allerede autentisert via `npx vercel login`. For scripts som trenger eksplisitt token, sett `VERCEL_TOKEN` som Windows-miljøvariabel (fra https://vercel.com/account/tokens).

### Teams

| Team | Bruk |
|------|------|
| `myrvoll-lunde-it-drift-s-projects` (<Vercel-team-ID>) | Klient-prosjekter |
| `Helge Myrvoll-Lunde's projects` (team_N9xGTOXnz2UqFmjm2yMlOSgB) | Personlige prosjekter |

### Typiske kommandoer

```bash
TOKEN="$env:VERCEL_TOKEN"  # Sett VERCEL_TOKEN som Windows-miljøvariabel

# Liste prosjekter
npx vercel ls --token "$TOKEN" --scope myrvoll-lunde-it-drift-s-projects

# Sjekke deploy-status
npx vercel ls --token "$TOKEN" --scope myrvoll-lunde-it-drift-s-projects | head -10

# Hent runtime-logger (bruk MCP-Vercel — get_runtime_logs)
# Hent build-logger (bruk MCP-Vercel — get_deployment_build_logs)

# Pull env-vars fra Vercel til lokal fil
npx vercel env pull /tmp/vercel-env --environment=production --token "$TOKEN"

# Legg til ny env-var (3 environments)
printf "%s" "$VALUE" | npx vercel env add MY_VAR production --token "$TOKEN"
printf "%s" "$VALUE" | npx vercel env add MY_VAR preview --token "$TOKEN"
printf "%s" "$VALUE" | npx vercel env add MY_VAR development --token "$TOKEN"
```

### Hva AI IKKE gjør med Vercel CLI

- ❌ `vercel deploy` manuelt — bruk auto-deploy fra GitHub-push
- ❌ Slette prosjekter
- ❌ Endre billing

### Hva AI gjør med Vercel CLI

- ✓ Sette env-vars (alle 3 environments)
- ✓ Hente env-vars (`vercel env pull`)
- ✓ Hente runtime-/build-logger via MCP
- ✓ Liste deployments + sjekke status
- ✓ Inspisere mislykkede deploys

---

## Secrets-flyt — ALLTID 3 steder

For hver ny secret som introduseres i prosjektet, AI skal sette den **samtidig** tre steder:

1. **`.env.local`** (gitignored, kun lokal utvikling)
2. **Vercel env vars** (Production + Preview + Development)
3. **GitHub Actions secrets** (`gh secret set`)

Manglende noen av disse = secret virker ikke i den miljøet.

### Sjekkliste for hver ny secret

```bash
SECRET_NAME=ANTHROPIC_API_KEY
SECRET_VALUE='sk-ant-...'
REPO=helgelunde1981-creator/[prosjekt-repo]

# 1) .env.local (lokal)
echo "$SECRET_NAME=\"$SECRET_VALUE\"" >> .env.local
# Verifiser i .gitignore: grep ".env.local" .gitignore || echo ".env.local" >> .gitignore

# 2) Vercel (prod + preview + dev)
TOKEN="$env:VERCEL_TOKEN"  # Sett VERCEL_TOKEN som Windows-miljøvariabel
for ENV in production preview development; do
  printf "%s" "$SECRET_VALUE" | npx vercel env add "$SECRET_NAME" "$ENV" --token "$TOKEN"
done

# 3) GitHub Actions secrets
gh secret set "$SECRET_NAME" --body "$SECRET_VALUE" -R "$REPO"

# Verifiser at alle 3 har den
echo "✓ .env.local: $(grep "^$SECRET_NAME=" .env.local | head -1 | cut -c1-30)..."
echo "✓ Vercel: $(npx vercel env ls --token "$TOKEN" 2>&1 | grep "$SECRET_NAME" | head -1)"
echo "✓ GH: $(gh secret list -R "$REPO" | grep "$SECRET_NAME")"
```

### `.gitignore`-krav

Hvert prosjekt MÅ ha disse i `.gitignore`:

```
.env
.env.local
.env.*.local
.env.production
.env.development
```

AI skal verifisere dette ved oppstart. Hvis mangler → legg til og commit.

### Standard secrets-katalog per prosjekt

| Secret | Bruk | Hvor lagres |
|--------|------|-------------|
| `NEXT_PUBLIC_SUPABASE_URL` | Klient-side | .env.local + Vercel (alle env) + GH |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Klient-side | .env.local + Vercel + GH |
| `SUPABASE_SERVICE_ROLE_KEY` | Server-only | .env.local + Vercel + GH |
| `SUPABASE_JWT_SECRET` | Custom JWT | .env.local + Vercel + GH |
| `SUPABASE_ACCESS_TOKEN` | CLI for migrasjoner | GH only (workflow) |
| `SUPABASE_PROJECT_REF` | Workflow identifier | GH only |
| `SUPABASE_DEV_DB_URL` | Dev-branch connection | GH only |
| `RESEND_API_KEY` | Email-send | .env.local + Vercel + GH |
| `RESEND_FROM_EMAIL` | Sender-adresse | .env.local + Vercel + GH |
| `TURNSTILE_SITE_KEY` | Anti-spam (public) | .env.local + Vercel + GH |
| `TURNSTILE_SECRET_KEY` | Anti-spam (server) | .env.local + Vercel + GH |
| `UPSTASH_REDIS_REST_URL` | Rate-limit | .env.local + Vercel + GH |
| `UPSTASH_REDIS_REST_TOKEN` | Rate-limit | .env.local + Vercel + GH |
| `CRON_SECRET` | Vercel Cron auth | .env.local + Vercel + GH |
| `NEXT_PUBLIC_SITE_URL` | Absolutt URLs | .env.local + Vercel + GH |
| `SENTRY_DSN` (hvis Sentry) | Error tracking | .env.local + Vercel + GH |
| `SENTRY_AUTH_TOKEN` (hvis Sentry) | Source maps | GH only |
| `ANTHROPIC_API_KEY` (hvis AI) | Claude API | .env.local + Vercel + GH |
| `NEXT_PUBLIC_COOKIEBOT_CBID` | Cookiebot consent (offentlig ID) | .env.local + Vercel (alle env) |
| `NEXT_PUBLIC_GTM_ID` (hvis GTM/GA4) | Google Tag Manager (offentlig ID) | .env.local + Vercel (alle env) |

> **Cookiebot-CBID + GTM-ID er offentlige `NEXT_PUBLIC_`-IDer (ikke hemmeligheter)** — trengs ikke i GH
> secrets. **Helge skaffer dem manuelt per nytt prosjekt** (ny Cookiebot-konto → CBID, ny GTM-container →
> `GTM-XXXXXXX`). Workflow: scaffold med **placeholder** i `.env.example` + `.env.local`
> (`NEXT_PUBLIC_COOKIEBOT_CBID="REPLACE_WITH_CBID"`, `NEXT_PUBLIC_GTM_ID="GTM-XXXXXXX"`) slik at appen
> bygger lokalt fra dag 1. **Guard i koden**: Cookiebot-/GTM-scriptene rendres KUN hvis env-varen er satt
> og ikke er placeholder — aldri en fake CBID i prod. Generert PROJECT.md ber Helge eksplisitt om begge
> verdiene i consent-/Sprint 0-steget («Spørsmål til kunden»). **Pre-launch blokkerer** hvis verdien
> fortsatt er placeholder. Se `06-SECTION_DETAILS/compliance-norge.md` → «Google Consent Mode v2».

### AI-sjekkliste når ny secret introduseres

- [ ] Lagt til i `.env.local`
- [ ] `.env.local` er i `.gitignore`
- [ ] Lagt til i Vercel Production
- [ ] Lagt til i Vercel Preview
- [ ] Lagt til i Vercel Development
- [ ] Lagt til i GitHub Actions secrets
- [ ] Dokumentert i seksjon 14.5 av PROJECT.md
- [ ] Verifisert at appen faktisk leser secret riktig

### Vanlige fallgruver

| Problem | Fix |
|---------|-----|
| Secret virker lokalt, men ikke i prod | Mangler i Vercel prod env |
| GitHub Actions feiler | Mangler i GH secrets |
| Preview-deploy bryter | Mangler i Vercel preview env |
| `.env.local` committed til git | Sjekk `.gitignore` + `git filter-repo` |
| Annet team-medlem mangler secret | Bruk `vercel env pull` for å hente alle |
| Secret rotert et sted, ikke andre | Bruk denne sjekklisten igjen |

---

## For AI: krav i PROJECT.md seksjon 14.5

I seksjonen «Sikkerhet → Secrets-håndtering», AI skal liste:

1. **Hver secret** med navn, bruk, hvor lagres (alle 3 steder eller GH-only)
2. **Initial-setup-script** for å sette alle secrets én gang (bash med kommentarer)
3. **`.gitignore`-bekreftelse** at `.env*` er ekskludert
4. **Vercel CLI-bash-snutter** for å pulle/oppdatere
5. **Rotasjons-policy** (hvor ofte, hvilken prosedyre)

Det er ikke nok å si «sett secrets i Vercel og GitHub» — AI må gi konkrete bash-kommandoer som kan kjøres direkte.
