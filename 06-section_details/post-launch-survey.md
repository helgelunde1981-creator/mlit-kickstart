# Post-launch Survey + Monitoring — vi vet hva som faktisk skjer etter lansering

«Kundeopplevelse er hovedparameter» (Prinsipp 1). Men hvordan vet vi at vi traff?

Vi har sett at AI-spec + Lighthouse-score sier 10/10, men vi får ingen tilbakemelding fra ekte brukere. Etter 14 dager: blackbox.

Post-launch-pakken løser dette med 3 leveranser i hvert prosjekt.

## Leveranse 1 — Plausible Analytics (eller liknende personvernvennlig)

Hver prosjekt får Plausible-script i `app/layout.tsx`:

```tsx
<script
  defer
  data-domain="{{prod_domain}}"
  src="https://plausible.io/js/script.outbound-links.js"
/>
```

- Personvernvennlig (ingen cookies)
- GDPR-compliant uten cookie-banner
- Lett (< 1KB)
- Custom events for kritiske flows (kontakt-skjema, kjøp, etc.)

Standard custom events vi måler:
- `cta_clicked` — primær-CTA på hero
- `contact_submitted` — kontakt-skjema
- `signup_completed` — hvis prosjekt har Auth
- `purchase_completed` — hvis prosjekt har e-commerce

## Leveranse 2 — Customer Satisfaction Survey (14 dager etter lansering)

Innebygget cron som etter 14 dager sender en HTML-mail til klienten:

```
Hei [navn],

Det har gått 14 dager siden vi lanserte [prosjekt-navn].
Vi vil veldig gjerne høre hvordan det fungerer for dere.

Tre raske spørsmål (tar 2 min):
  1. Hvor sannsynlig er det at du anbefaler oss? (1-10)
  2. Hva er det BESTE med løsningen?
  3. Hva er det første du ønsker forbedret?

[Svar nå →]

— Myrvoll-Lunde IT Drift
```

Lenken går til `survey.mlit.no/[token]` med skjema. Data lagres i `customer_surveys`-tabellen i mlit-DB.

### Datamodell

```sql
create table if not exists public.customer_surveys (
  id uuid primary key default gen_random_uuid(),
  created_at timestamptz not null default now(),
  responded_at timestamptz,

  kickstart_project_id uuid references public.kickstart_projects(id) on delete cascade,
  project_slug text not null,
  client_name text not null,
  client_email text not null,

  -- Token (signed JWT)
  token text not null unique,
  sent_at timestamptz not null,
  reminder_sent_at timestamptz,

  -- Svar
  nps_score integer check (nps_score between 0 and 10),
  best_thing text,
  improvement_wanted text,

  -- Auto-tags fra LLM
  tags text[] default array[]::text[],
  sentiment text check (sentiment in ('positive', 'neutral', 'negative')),

  unique (kickstart_project_id, sent_at)
);
```

## Leveranse 3 — Real User Monitoring (RUM) via Vercel Speed Insights

Vercel Speed Insights enables med én linje i `next.config.ts`:

```ts
export default {
  experimental: {
    speedInsights: true,
  },
}
```

Gir oss reelle CWV-tall (LCP, INP, CLS) per route fra ekte besøkende — ikke bare lab-tall fra Lighthouse.

Etter 14 dager kan vi sammenligne:
- Lab Lighthouse: 98 / 100 / 100 / 95
- Real CWV (P75): LCP 2.1s, INP 180ms, CLS 0.05

Hvis store gap, vet vi hvor.

## Sprint-integrasjon

Sprintplanens **siste sprint (Pre-launch)** MÅ inneholde disse oppgavene:
- [ ] Plausible-script installert + custom events satt opp
- [ ] Vercel Speed Insights aktivert
- [ ] Customer survey-cron registrert med korrekt `kickstart_project_id` + `client_email`
- [ ] Survey-mailtemplate testet (sender til Helge som dry-run)

## Anti-pattern

- **Generic NPS uten tags:** Bruk LLM til auto-tagging av svar (perf, design, support, etc.) — ellers er det støy
- **Spør for ofte:** ÉN survey 14 dager etter lansering, en til 90 dager. Punkt.
- **Ikke svar-knapp:** Hvis kunden ber om noe i `improvement_wanted`, MÅ vi follow-up. Innebygget i admin-modulen som task.

## Vault-integrasjon

Hver survey-respons opprettes som notat i `C:\Github\vault\klienter\<KlientNavn>.md` med:
```markdown
## NPS-respons 2026-06-15
- Score: 9/10
- Beste: «Designet ser premium ut, vi har fått tre nye henvendelser»
- Forbedring: «Kunne lett funksjon for å booke direkte»
- Auto-tags: design, lead-gen, booking
```

Slik husker vi hva hver klient sa, uten å lete i DB-en.
