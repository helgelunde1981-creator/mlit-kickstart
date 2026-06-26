# Section Detail — Monitoring + Alerting + Disaster Recovery

> Brukes når AI fyller ut seksjon 18 (Monitoring + DR) i PROJECT.md.

---

## Minimum required output

Seksjon 18 må inneholde:

1. **Uptime-monitoring** — hvilken tjeneste, hvilke endepunkter
2. **Error tracking** — Sentry config, hvilke errors fanger vi
3. **Performance-monitoring** — Vercel + custom
4. **Alerting** — hvem får varsel når, via hvilken kanal
5. **Backup-strategi** — Supabase PITR, kadens, retensjons-policy
6. **Disaster recovery plan** — RTO, RPO, runbook
7. **Status-page** — har vi en, hvor

## Template

```markdown
## 18. Monitoring + alerting + DR

### 18.1 Uptime-monitoring

**Tjeneste:** Selvhostet **Uptime Kuma** på **https://uptime.mlit.no**. Full operasjonell instruks (hva som skal settes opp for nye prosjekter, og hvordan legge til monitorer/status-sider/grupper): se **[uptime-kuma.md](./uptime-kuma.md)**.

**Endepunkter overvåket (minimum per prosjekt):**

| Endpoint | Forventet | Intervall |
|----------|-----------|-----------|
| `https://[domene]/` | 200 OK, keyword-sjekk | 60s |
| `https://[domene]/api/health` | 200 OK, `"ok":true` | 60s |
| `https://<ref>.supabase.co/auth/v1/health` | 200 / 401 (gateway lever) | 120s |
| Eksterne tjenester appen bruker (Stripe, Resend, …) | «edge lever»-koder | 300s |

Varsling går til Slack (default-notifikasjon arves av nye monitorer). Se [uptime-kuma.md §4](./uptime-kuma.md) for delte tjenester som allerede er overvåket — ikke dupliser dem.

**`/api/health`-endpoint:**

\`\`\`ts
// app/api/health/route.ts
export async function GET() {
  // Sjekk Supabase-tilkobling
  const supabase = createServerClient(...)
  const { error } = await supabase.from('site_settings').select('key').limit(1)
  if (error) return Response.json({ ok: false, error: 'db' }, { status: 503 })
  return Response.json({ ok: true, timestamp: Date.now() })
}
\`\`\`

### 18.2 Error tracking (Sentry)

**Installasjon:**

\`\`\`bash
pnpm add @sentry/nextjs
npx @sentry/wizard@latest -i nextjs
\`\`\`

**Konfig:**

- DSN i env vars (`SENTRY_DSN`)
- Source maps lastet opp ved deploy (Vercel-integrasjon)
- Sample rate: 1.0 (alle errors) i prod, 1.0 dev
- Performance sample rate: 0.1 i prod (10% av transactions)
- Ignorer kjente non-issues (browser-quirks, hydration-warnings vi vet er ikke-kritiske)

**Hvilke errors fanger vi:**

- Alle unhandled exceptions (client + server)
- Alle 5xx-responser fra API routes
- Alle Supabase-feil (manuelt logget i Server Actions)
- Alle Resend-feil ved e-post-send
- Custom errors fra rate-limit, validering

**Hvilke filtrerer vi:**

- Browser-extension errors (Chrome ext)
- ResizeObserver loop warnings
- Aborted fetch (bruker navigerer bort)
- Bot user-agents

### 18.3 Performance-monitoring

- **Vercel Speed Insights** — RUM for CWV
- **Sentry Performance** — server-side latency
- **Custom metrics i logs** — Supabase query-tid for slow queries (> 500ms)

### 18.4 Alerting

| Trigger | Kanal | Mottaker | Når |
|---------|-------|----------|-----|
| Uptime check failed 3x i rad | Slack + SMS | Helge | Umiddelbart |
| Sentry error-rate > 10/min | Slack | Helge | Innen 5 min |
| Lighthouse-score faller under 90 | Slack | Helge | Daglig sjekk |
| Supabase storage > 80% kvota | E-post | Helge | Daglig |
| Resend bounce-rate > 5% | E-post | Helge | Ukentlig |

**Slack-integrasjon:** Sentry har native Slack-app. Uptime Kuma varsler til Slack via incoming webhook (allerede konfigurert som default-notifikasjon).

### 18.5 Backup-strategi (Supabase)

| Backup-type | Kadens | Retensjon | Hvor |
|-------------|--------|-----------|------|
| **Point-in-Time Recovery (PITR)** | Kontinuerlig | 7 dager (Pro-plan) eller 30 dager (Team+) | Supabase-managed |
| **Daglig full backup** | Hver natt 03:00 UTC | 30 dager | Supabase-managed |
| **Schema dump** | Per migrasjon (via git) | Forever | GitHub repo |
| **Data export (manuell)** | Før breaking changes | Forever | `gh secret`-protected S3 bucket |

**Aktiver PITR i Supabase dashboard:** Settings → Database → Backups.

### 18.6 Disaster Recovery Plan

**RTO (Recovery Time Objective):** 4 timer for full restore
**RPO (Recovery Point Objective):** 5 minutter (PITR-granularitet)

**Scenarier:**

#### Scenario 1: Vercel-deploy korrupt
1. Revert til siste kjente good deploy i Vercel UI (10 sek)
2. Identifiser root cause i logs
3. Fiks, redeploy

#### Scenario 2: Supabase-data korrupt eller slettet
1. Identifiser tidspunkt før hendelsen
2. PITR-restore til det tidspunktet (Supabase support eller dashboard)
3. Verifiser dataintegritet
4. Vurder om bestillinger/transactions etter PITR-punktet må re-applies manuelt

#### Scenario 3: Domene utilgjengelig (DNS-issue)
1. Sjekk DNS-registrar status
2. Failover til backup-DNS hvis konfigurert
3. Oppdater status-page

#### Scenario 4: Resend-leveranse-issue
1. Sjekk Resend status-page
2. Failover til backup-provider (Postmark) hvis konfigurert
3. Endre `RESEND_API_KEY` til `POSTMARK_TOKEN` med wrapper

### 18.7 Status-page

Status-side lages i **Uptime Kuma** — se [uptime-kuma.md §9–10](./uptime-kuma.md) for full fremgangsmåte.

- **URL:** `https://uptime.mlit.no/status/<slug>` (eller white-label `status.[domene]` via CNAME + Caddy)
- **Gruppering:** `Din løsning` (nettside + Supabase) + `Eksterne tjenester` (avhengigheter)
- **Innhold:** oppetid-%, heartbeat-historikk, SSL-utløp per tjeneste — auto-oppdatert fra monitorene

For ikke-kritiske prosjekter: ignorer status-page.

### 18.8 Incident response runbook

Når monitoring trigger:

1. **Acknowledge** alarmen i Slack/SMS (stopper re-notify)
2. **Triager** — er det ekte issue eller falsk alarm?
3. **Mitiger** — apply quick fix hvis kjent issue (revert, restart, bypass)
4. **Kommuniser** — oppdater status-page hvis kunde-vendt
5. **Root cause** — etter hot-fix, finn rotårsak
6. **Postmortem** — for store incidents (>1 time nedetid), lag postmortem-notat i Obsidian-vault
```

## Vanlige fallgruver

| Problem | Fix |
|---------|-----|
| Sentry overrapporterer (alt går til Sentry) | Filtrer kjente non-issues, sett sample rate lavere |
| Uptime-monitor sjekker bare `/` → admin/api utilgjengelig uten varsel | Sjekk flere endepunkter |
| PITR ikke aktivert i Supabase | Aktiver i dashboard, kost ca $15/mnd ekstra på Pro |
| Status-page glemmes når deployment går ned | Auto-integrert med deploy-system |
| Postmortems blir glemt | Maler i Obsidian + sjekkliste i CI |
