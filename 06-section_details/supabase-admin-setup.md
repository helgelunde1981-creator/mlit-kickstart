# Section Detail — Supabase admin-oppsett (Dashboard-only)

> AI kan ikke endre disse via CLI eller MCP. AI skal eksplisitt instruere kunden (eller Helge) om å gjøre dem i Supabase Dashboard, og inkludere en sjekkliste i PROJECT.md.

---

## Hvorfor dette må flagges

Per Helge's globale CLAUDE.md:
> ### Hva Claude IKKE kan gjøre (krever Dashboard)
> - Endre auth-innstillinger (redirect URLs, providers, email templates)
> - Slette prosjekter
> - Endre billing

Hvis AI ikke flagger disse i PROJECT.md, blir de glemt — og prosjektet vil ikke fungere i prod (auth-flyt feiler, emails kommer ikke frem, etc.).

## Mandatory admin-sjekkliste per prosjekt

PROJECT.md må inkludere denne sjekklisten i seksjon 14 (Sikkerhet) eller egen «Supabase admin-oppsett»-undersksjon:

### 1. Auth → Redirect URLs

Settes i: **Dashboard → Authentication → URL Configuration**

```
Site URL:    https://[domene]
Redirect URLs (legg til):
  https://[domene]/**
  https://[domene]/auth/callback
  https://[domene]/auth/set-password
  http://localhost:3000/**          (kun for dev)
  https://[preview-host].vercel.app/**  (Vercel preview)
```

Uten dette: magic links og e-postbekreftelser redirecter til `localhost:3000` eller feil URL → ødelagt brukeropplevelse.

### 2. Auth → Email Templates (oppdatert med våre HTML-maler)

Settes i: **Dashboard → Authentication → Email Templates**

For hver mal-type (Confirm signup, Invite user, Magic link, Reset password, Change email, Reauthentication), lim inn HTML-en vi har laget i `emails/auth/`-mappen.

Sjekkliste:
- [ ] Confirm signup
- [ ] Invite user
- [ ] Magic link
- [ ] Reset password
- [ ] Change email
- [ ] Reauthentication

NB: Supabase støtter Handlebars-variabler som `{{ .ConfirmationURL }}`, `{{ .Token }}`, `{{ .Email }}`, `{{ .RedirectTo }}`. Vi rendrer react-email til HTML først og erstatter manuelt med disse.

### 3. Auth → SMTP-konfigurasjon

Settes i: **Dashboard → Project Settings → Auth → SMTP Settings**

Default sender via Supabase delt SMTP (begrenset til 4/time). For prod MÅ vi sette opp custom SMTP via Resend:

```
Host:     smtp.resend.com
Port:     587
Username: resend
Password: re_XXX (Resend API key)
Sender:   noreply@[domene] (samme domene som er verifisert i Resend)
```

Sjekkliste:
- [ ] Domene verifisert i Resend (DKIM/SPF/DMARC — se §16 DNS)
- [ ] SMTP-credentials lagret i Supabase Dashboard
- [ ] Test-email sendt (Dashboard har «Send test email»-knapp)
- [ ] Email mottatt + ikke i spam

### 4. Auth → Providers

Settes i: **Dashboard → Authentication → Providers**

Avhenger av prosjektkrav:
- [ ] Email (alltid på)
- [ ] Google (hvis Google SSO i `auth_type`)
- [ ] Magic Link (av eller på basert på krav)
- [ ] Phone (kun hvis aktuelt)

For Google SSO: krever Google Cloud OAuth-app + lim inn Client ID + Secret.

### 5. Database → Backups

Settes i: **Dashboard → Database → Backups**

- [ ] Point-in-Time Recovery aktivert (Pro-plan: 7 dager, Team+: 30 dager)
- [ ] Daglig backup-policy bekreftet

Uten PITR: ved data-tap er det INGEN måte å rulle tilbake.

### 6. API → Network Restrictions (valgfritt, anbefalt for prod)

Settes i: **Dashboard → Project Settings → API → Network Restrictions**

Hvis prosjektet er strengt admin-only:
- [ ] Restrict API to Vercel IP ranges
- [ ] Whitelist team-utviklere

### 7. Storage → Bucket-policyer

Settes i: **Dashboard → Storage**

Per bucket (`media`, `documents`, etc.):
- [ ] Bucket public/private satt riktig
- [ ] Policy: SELECT for authenticated, INSERT for service-role only
- [ ] Mime-type allowlist (eks: `image/*` for media)
- [ ] File-size limit satt

## Sjekkliste-format i PROJECT.md

I seksjon 14 (Sikkerhet), inkluder undersksjonen:

```markdown
### 14.X Supabase admin-oppsett (Dashboard-only)

Disse må kunden eller Helge gjøre manuelt i Supabase Dashboard
før prosjektet kan deployes til prod. AI/Claude kan ikke endre disse.

#### Auth → Redirect URLs
- [ ] Site URL: https://[domene]
- [ ] Redirect URLs lagt til (se mal i `06-SECTION_DETAILS/supabase-admin-setup.md`)

#### Auth → Email Templates
- [ ] Confirm signup limt inn fra `emails/auth/ConfirmEmail.tsx`
- [ ] Magic Link limt inn fra `emails/auth/MagicLink.tsx`
- [ ] Reset Password limt inn fra `emails/auth/ResetPassword.tsx`
- [ ] Invite User limt inn fra `emails/auth/Invitation.tsx`
- [ ] Change Email + Reauthentication limt inn

#### Auth → SMTP
- [ ] Custom SMTP konfigurert (smtp.resend.com)
- [ ] Sender: noreply@[domene]
- [ ] Test-email mottatt og verifisert

#### Auth → Providers
- [ ] Email aktivert
- [ ] [Google SSO konfigurert hvis aktuelt]

#### Database
- [ ] PITR aktivert (Backups)
- [ ] Daglig backup verifisert

#### Storage
- [ ] [Per bucket: public/private + RLS-policyer]
```

## Notat til AI

Når AI genererer PROJECT.md, MÅ den:

1. Liste alle relevante punkter fra denne malen (kun de som er aktuelle for prosjektet)
2. Markere dem som «Dashboard-only — krever manuell gjennomføring»
3. Plassere dem i seksjon 14 (Sikkerhet) som egen undersksjon
4. Inkludere lenke/referanse til denne malen
5. Også referere til disse i suksesskriteriene («Supabase admin-sjekkliste fullført»)
6. Hvis kundedata mangler info (eks. e-post-domene for SMTP), legge spørsmål i «Spørsmål til kunden»-seksjonen
