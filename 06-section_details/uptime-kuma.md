# Uptime-overvåking — Instruks for nye prosjekter

> Operasjonell runbook for **uptime.mlit.no** (selvhostet Uptime Kuma).
> Brukes når et nytt prosjekt settes opp, og når du skal legge til monitorer, status-sider eller grupper.
> Hører sammen med §18.1 i [monitoring-dr.md](./monitoring-dr.md).

---

## 1. Oversikt over oppsettet

| | |
|---|---|
| **URL** | https://uptime.mlit.no (dashboard bak login) |
| **Versjon** | Uptime Kuma 2.x (selvhostet) |
| **Hosting** | Hetzner-boksen (`178.104.164.96`), egen Docker-stack i `~/uptime-kuma/`, isolert fra n8n |
| **Proxy/SSL** | Caddy (deler med n8n) — auto Let's Encrypt |
| **Varsling** | Slack incoming webhook (notifikasjon «Slack uptime», satt som default → arves av nye monitorer) |
| **Dead-man's-switch** | healthchecks.io pinges hvert minutt av cron på boksen — varsler hvis selve Kuma dør |

Dashboardet er **privat** (admin-login). Status-sidene er **offentlige** (men ikke søkemotor-indeksert) og viser kun navn + opp/ned + oppetid-%, aldri URL-er eller secrets.

---

## 2. Hva som skal settes opp for ETHVERT nytt prosjekt

Definition of done for uptime ved lansering:

- [ ] **Nettsiden** — HTTP(s)-monitor på produksjons-URL
- [ ] **Supabase** — health-monitor på prosjektets Supabase-gateway
- [ ] **Eksterne tjenester** — alle tredjeparts-API-er appen er *runtime*-avhengig av (se §3–4)
- [ ] **(Anbefalt for betalingskritiske prosjekter)** — et `/api/health`-endepunkt for ekte ende-til-ende-sjekk (se §8)
- [ ] **Status-side** for kunden hvis relevant (se §6)
- [ ] Alle monitorer har Slack-varsling (arves automatisk fra default-notifikasjonen)

---

## 3. Slik finner du prosjektets avhengigheter

Kjør en 2-veis skann i repoet (lokalt). Begge signaler trengs — `.env` viser hvilke tjenester, koden viser hvilke som faktisk kalles:

```bash
# 1) Tjeneste-nøkler i .env-filene (avslører hvilke tjenester)
cat .env .env.local .env.production 2>/dev/null \
  | grep -ioE '(RESEND|STRIPE|VIPPS|ANTHROPIC|OPENAI|GEMINI|SENDGRID|TWILIO|CLOUDINARY|TRIPLETEX|WEBFLOW|WIX|TURNSTILE|FACEBOOK|META|ZERNIO|BUNDLE|CHATWOOT|ATERA|FIKEN|TEAMVIEWER|SUPABASE)' \
  | tr 'a-z' 'A-Z' | sort -u

# 2) Eksterne domener faktisk kalt i koden (avslører runtime-avhengigheter)
grep -rhoE 'https://[a-zA-Z0-9.-]+\.[a-z]{2,}' src app \
  --exclude-dir=node_modules --exclude-dir=.next 2>/dev/null \
  | sed -E 's#https://##' | sort | uniq -c | sort -rn
```

Filtrer bort: eget domene, bilde-CDN-er (unsplash/pexels), lenke-mål til sosiale medier/lovsider, font-CDN-er. Det som står igjen og faktisk kalles = en avhengighet som skal overvåkes.

**Supabase-ref** hentes fra `NEXT_PUBLIC_SUPABASE_URL` / `VITE_SUPABASE_URL` (subdomenet er prosjekt-ref-en).

---

## 4. Delte tjenester som ALLEREDE er overvåket — ikke dupliser

Disse har én felles monitor hver. Bruker prosjektet en av dem, **koble til den eksisterende** (på status-siden), ikke lag en ny:

`Resend API` · `Stripe API` · `Anthropic API` · `OpenAI API` · `Google Gemini API` · `Twilio API` · `Cloudinary API` · `SendGrid API` · `Tripletex API` · `Cloudflare Turnstile` · `Facebook/Instagram Graph API` · `Webflow API` · `Wix API` · `Zernio API (Publify)` · `bundle.app (Somify)` · `chat.mlit.no` (Chatwoot)

Kun **prosjekt-unike** integrasjoner skal opprettes på nytt (f.eks. mlit sine `Atera`/`Fiken`/`TeamViewer`). **Supabase** er alltid prosjekt-spesifikk → egen monitor per prosjekt.

---

## 5. Konvensjoner (følg disse for konsistens)

### Navngiving
- Nettside: domenet uten protokoll — `kunde.no`
- Supabase: `Supabase: <prosjekt>`
- Ekstern delt tjeneste: tjenestenavn — `Stripe API`
- Prosjekt-unik ekstern: `<Tjeneste> <PROSJEKT>` — f.eks. `Fiken API MLIT`

### Tags (fargekoding i dashbordet)
| Tag | Farge | Bruk |
|---|---|---|
| `Myrvoll-Lunde IT Drift` | `#3b82f6` blå | Myrvoll-Lunde IT Drift-prosjekter |
| `Gange-Rolv` | `#22c55e` grønn | Gange-Rolv-prosjekter |
| `Supabase` | `#3ecf8e` | Alle Supabase-health-sjekker |
| `Ekstern` | `#a855f7` lilla | Tredjeparts-API-er |

### Intervall og aksepterte statuskoder
| Type | Intervall | Aksepterte koder | Endepunkt |
|---|---|---|---|
| Nettside | 60s | `200-299` (+`401`/`403` bak auth-vegg) | produksjons-URL |
| Supabase | 120s | `200`, `401` | `https://<ref>.supabase.co/auth/v1/health` |
| Ekstern API | 300s | `200-299`,`401`,`403`,`404` (+`421` for OpenAI) | API-base |

> **«Edge lever»-prinsippet:** for tredjeparts-API-er godtar vi alle koder som betyr «serveren svarte» (401/403/404). Da varsler vi kun på reell nedetid (5xx / timeout / DNS-feil), ikke på at API-et krever auth.

### Varsling
Default-notifikasjonen «Slack uptime» har `isDefault=true` → **nye monitorer arver den automatisk**. Du trenger ikke huke den av manuelt i UI-et, men sjekk at den er på.

---

## 6. Slik legger du til en monitor (UI — normal måte)

1. Logg inn på https://uptime.mlit.no
2. Klikk **「+ Legg til ny overvåkning」**
3. Fyll ut:
   - **Type:** HTTP(s) for nettsider/API, TCP Port for rå server-sjekk
   - **Vennlig navn:** følg navnekonvensjonen (§5)
   - **URL:** se endepunkt-tabellen (§5)
   - **Heartbeat-intervall:** 60 / 120 / 300 etter type
   - **Forsøk på nytt:** 2
   - **Aksepterte statuskoder** (under «Avansert»): se §5
   - **Notifikasjoner:** «Slack uptime» (skal være på)
   - **Tags:** sett riktig selskap + `Supabase`/`Ekstern`
4. **Lagre.** Monitoren starter å sjekke umiddelbart.

> For **én eller noen få** monitorer: bruk alltid UI-et. For **bulk** (mange på en gang), se §7.

---

## 7. Bulk-tillegg via SQLite (avansert — kun for mange samtidig)

> ⚠️ `uptime-kuma-api` (Python-wrapperen) virker **kun mot Kuma 1.x**, ikke 2.x. På 2.x legges monitorer inn enten i UI-et eller via direkte SQLite-innsetting. Bruk SQLite kun ved bulk, og ta backup først.

```bash
ssh mlit@178.104.164.96
cd ~/uptime-kuma
# backup
tar czf backup-$(date +%Y%m%d_%H%M%S).tar.gz data docker-compose.yml
docker compose stop uptime-kuma
# ... kjør python-script som kloner en eksisterende top-level http-rad og
#     overstyrer name/url/accepted_statuscodes_json/interval, så insert i
#     monitor + monitor_notification + monitor_tag ...
docker compose start uptime-kuma   # laster nye monitorer ved oppstart
```

**Nøkkelfakta for SQLite-metoden:**
- `monitor`-tabellen har **ingen NOT NULL uten default** → trygt å klone en eksisterende rad og kun overstyre felter
- Koble varsling: insert i `monitor_notification (monitor_id, notification_id)`
- Sett tag: insert i `monitor_tag (monitor_id, tag_id, value)`
- Kjør script i engangs-container: `docker run --rm -v ~/uptime-kuma/data:/data -v script.py:/s.py:ro python:3.11-slim python /s.py`
- Container **uten `-i`** gir tom stdin → monter alltid script som fil, ikke pipe via `/dev/stdin`

---

## 8. (Anbefalt) `/api/health`-endepunkt for ekte ende-til-ende-sjekk

Nettside- + Supabase- + delte-monitorer fanger ~90 % av reelle feil. For **betalingskritiske** prosjekter, legg til et health-endepunkt som verifiserer at HELE stacken faktisk virker (DB-spørring + nøkler gyldige), og overvåk **det ene endepunktet**.

**Hold Kuma nøkkelfri:** lag endepunktet **offentlig** (gjør det hemmelige arbeidet server-side, eksponer kun `200 {ok}` / `503`). Da trenger Kuma ingen nøkler.

```ts
// app/api/health/route.ts  (Next.js) — eller Supabase edge function med --no-verify-jwt
export async function GET() {
  const supabase = createServerClient(/* ... */)
  const { error } = await supabase.from('site_settings').select('key').limit(1)
  if (error) return Response.json({ ok: false, error: 'db' }, { status: 503 })
  // valgfritt: verifiser Stripe/Resend-nøkkel her, server-side
  return Response.json({ ok: true })
}
```

Overvåk så `https://<domene>/api/health` med keyword-sjekk på `"ok":true`.

> **Aldri** legg ekte secrets (service-role, Stripe secret, API-nøkler) inn i Kuma. Kun den allerede-offentlige Supabase anon-key er teknisk grei — men unngå den ved å bruke et offentlig health-endepunkt.

---

## 9. Slik lager du en status-side (UI)

1. Toppmenyen → **「Statusside」** → **「Ny statusside」**
2. Skriv **navn** + **slug** (slug blir URL: `uptime.mlit.no/status/<slug>`)
3. I editoren: sett tittel, logo, tema (kan matche kundens farger), og skru på «Vis tags» + «Vis sertifikat-utløp»
4. Legg til **grupper** (§10) og dra monitorene inn
5. **Lagre**

### Gruppe-konvensjon
- **Enkeltprosjekt-side (kunde):** `Din løsning` (nettside + Supabase) + `Eksterne tjenester` (avhengigheter)
- **Oversiktsside (mange prosjekter):** `Nettsteder` + `Backend (Supabase)` + `Eksterne tjenester` (+ `Infrastruktur` for intern stack)

### Egen URL (white-label) per kunde — valgfritt
`status.kunde.no` → CNAME til `uptime.mlit.no` + legg domenet i Caddy (`~/n8n/Caddyfile`) og reload. Da blir status-siden helt på kundens domene.

---

## 10. Slik legger du til / endrer grupper

I status-side-editoren: **「Legg til gruppe」** → navngi → bruk monitor-nedtrekkslisten eller dra monitorer inn i gruppen. Rekkefølge styres ved å dra.

> **Viktig om SQLite-endringer:** Kuma **cacher status-side-innhold i minnet**. Endrer du `status_page`/`group`/`monitor_group` direkte i databasen, må du **restarte containeren** (`docker compose restart uptime-kuma`) for at endringen skal vises. (Nye *monitorer* lastes ved oppstart uansett; UI-endringer trer i kraft umiddelbart.)

DB-struktur: `status_page` → `group (status_page_id)` → `monitor_group (monitor_id, group_id, weight, send_url)`.

---

## 11. Sjekkliste ved lansering av nytt prosjekt

- [ ] Nettside-monitor opprettet (riktig produksjons-URL, keyword/status)
- [ ] Supabase-monitor opprettet (`/auth/v1/health`, tag `Supabase`)
- [ ] Avhengigheter kartlagt (§3) og alle unike eksterne tjenester overvåket
- [ ] Delte tjenester **koblet til**, ikke duplisert (§4)
- [ ] Slack-varsling bekreftet på alle nye monitorer
- [ ] Riktige tags satt (selskap + Supabase/Ekstern)
- [ ] (Hvis betalingskritisk) `/api/health`-endepunkt deployet + overvåket
- [ ] (Hvis kundevendt) status-side opprettet med riktig gruppering
- [ ] Alle nye monitorer verifisert grønne før sign-off
