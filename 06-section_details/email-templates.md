# Section Detail — E-postmaler (HTML, designet)

> Brukes når AI fyller ut PROJECT.md. Alle e-poster fra prosjektet (auth + transactional + marketing) MÅ være nydelig designede HTML-eposter som matcher sidens visuelle stil — ikke default Supabase/Resend-text.

---

## Krav

**Alle e-poster skal:**

- Være HTML med samme designspråk som siden (samme palette, samme typografi, samme tone)
- Ha responsive layout (fungerer på iPhone Mail, Outlook, Gmail, Apple Mail)
- Inkludere preheader (skjult tekst som vises i inbox-preview)
- Inkludere klar CTA-knapp med samme stil som siden
- Ha footer med «Utviklet av Myrvoll-Lunde IT Drift» → mlit.no (mandatory, jf. regel 11)
- Følge brand voice fra `05-BRAND_VOICE.md`
- Bruke **react-email** eller liknende for å bygge (komponent-basert + responsive utility-CSS)
- Sendes via Resend (default) eller Supabase Auth SMTP for auth-mails

## Mal-katalog som ALLE prosjekter må ha

Minimum-sett. Marker som «N/A» kun hvis funksjonalitet ikke er i scope.

### Auth / kontoadministrasjon

Disse er Supabase Auth-emails som settes opp i Dashboard (se `supabase-admin-setup.md`):

| Mal | Trigger | Krav |
|-----|---------|------|
| **Magic link** | Bruker ber om innlogging | Stor «Logg inn»-knapp · 1-times utløp nevnt · IP/enhet-info |
| **E-post-verifisering** | Etter registrering | «Bekreft e-post»-knapp · forklarer hvorfor |
| **Tilbakestill passord** | Bruker glemte passord | «Nullstill passord»-knapp · 1-times utløp |
| **Invitasjon** | Superadmin inviterer | Inviterers navn · «Aksepter»-knapp · forklarer rolle |
| **E-post-endring** | Bruker endrer e-post | Bekreftelse til både ny og gammel adresse |

### Transactional (per prosjekt-type)

| Mal | Når | Krav |
|-----|-----|------|
| **Kontaktskjema-kvittering** | Bruker sender kontaktskjema | Bekrefter mottak · «Vi tar kontakt innen X»-løfte |
| **Kontaktskjema-internvarsel** | Til kunde-team | Inneholder hele meldingen + svare-instruks |
| **Bestillingsbekreftelse** | E-handel/booking | Bestillingsdetaljer · pris-breakdown · neste steg |
| **Faktura sendt** | Faktura genereres | PDF vedlagt · «Betal nå»-knapp · forfallsdato |
| **Tilbudsforespørsel** | Bruker ber om tilbud | Bekrefter mottak · responstid · referansenummer |

### Marketing (kun hvis aktuelt)

| Mal | Når | Krav |
|-----|-----|------|
| **Velkommen** | Etter signup | Personlig hilsen · neste-steg-guide · social proof |
| **Newsletter** | Per kampanje | Tydelig avmeldings-lenke · brand-tone |

## Anbefalt teknologi

- **react-email** (https://react.email) — komponent-bibliotek for designede emails i React
- **Tailwind CSS for email** (inline) — bruker `tailwindcss-cli` for å inline klasser
- **Resend** — Send via Resend's React-email-integrasjon

```tsx
// emails/contact-receipt.tsx — eksempel
import {
  Body, Button, Container, Head, Heading, Hr, Html, Img, Link,
  Preview, Section, Tailwind, Text,
} from '@react-email/components'

interface Props {
  name: string
  message: string
}

export default function ContactReceiptEmail({ name, message }: Props) {
  return (
    <Html lang="nb">
      <Head />
      <Preview>Takk for henvendelsen, {name} — vi tar kontakt innen 24 timer</Preview>
      <Tailwind>
        <Body className="bg-[#f7f9fc] font-sans">
          <Container className="mx-auto max-w-[560px] rounded-xl bg-white p-8 shadow-sm">
            <Img
              src="https://[domene]/email-logo.png"
              width={120}
              height={32}
              alt="[Kunde]"
            />
            <Heading className="mt-6 text-2xl font-medium text-[#0b0b0b]">
              Takk, {name}!
            </Heading>
            <Text className="mt-2 text-base leading-relaxed text-[#5a5a5a]">
              Vi har mottatt henvendelsen din og kommer tilbake til deg
              innen 24 timer (ofte raskere).
            </Text>

            <Section className="mt-6 rounded-md border border-[#e2e2e2] bg-[#faf6f2] p-4">
              <Text className="m-0 text-xs uppercase tracking-wider text-[#5a5a5a]">
                Din melding
              </Text>
              <Text className="mt-2 whitespace-pre-line text-sm text-[#0b0b0b]">
                {message}
              </Text>
            </Section>

            <Button
              href="https://[domene]"
              className="mt-6 inline-block rounded-md bg-[#4a805f] px-5 py-3 text-sm font-medium text-white no-underline"
            >
              Tilbake til [domene]
            </Button>

            <Hr className="my-8 border-[#e2e2e2]" />
            <Text className="text-xs text-[#5a5a5a]">
              Utviklet av{' '}
              <Link href="https://mlit.no" className="text-[#4a805f]">
                Myrvoll-Lunde IT Drift
              </Link>
            </Text>
          </Container>
        </Body>
      </Tailwind>
    </Html>
  )
}
```

## Designsystem-tilpasning per direction

Bruk tokens fra valgt design-direction. Eksempler:

- **Dark Luxury Motion**: dark bg (`oklch(8% 0.01 30)`), orange CTA, Geist Mono i footer
- **Swiss Minimal Refined**: hvit bg, blå accent, hairline-borders
- **Industrial / Robust**: warm grey bg, safety-orange CTA, monospace metadata
- **Editorial Bento**: hvit bg, deep-red accent, serif headings

Email-CSS er begrenset (no `gap`, ingen CSS Grid, ingen modern features). Bruk tabeller for layout der nødvendig.

## Testing

Hver mal MÅ være testet på:

- iPhone Mail (mest brukt i Norge)
- Apple Mail (macOS)
- Gmail web + Gmail iOS app
- Outlook web + Outlook Windows
- Dark mode (varier per klient)

Verktøy: https://mailtrap.io, https://www.litmus.com, eller https://react.email/components-test

## Mappestruktur

```
emails/
├── components/
│   ├── EmailHeader.tsx
│   ├── EmailFooter.tsx     ← med "Utviklet av Myrvoll-Lunde IT Drift"
│   └── EmailButton.tsx
├── auth/
│   ├── MagicLink.tsx
│   ├── ConfirmEmail.tsx
│   ├── ResetPassword.tsx
│   └── Invitation.tsx
├── transactional/
│   ├── ContactReceipt.tsx
│   ├── ContactInternalAlert.tsx
│   ├── OrderConfirmation.tsx
│   └── QuoteRequest.tsx
└── marketing/
    └── Welcome.tsx
```

## Send-flyt

```ts
// lib/email.ts
import { render } from '@react-email/render'
import { Resend } from 'resend'
import ContactReceiptEmail from '@/emails/transactional/ContactReceipt'

const resend = new Resend(process.env.RESEND_API_KEY!)

export async function sendContactReceipt(to: string, name: string, message: string) {
  const html = await render(ContactReceiptEmail({ name, message }))
  return resend.emails.send({
    from: process.env.RESEND_FROM_EMAIL!,
    to,
    subject: `Takk for henvendelsen, ${name}`,
    html,
  })
}
```

## Sjekkliste for AI

Når AI lager PROJECT.md, må email-seksjonen inkludere:

- [ ] Liste over alle email-maler som skal lages (per matrise over)
- [ ] Mappestruktur i `emails/`
- [ ] Anbefalt verktøy (react-email)
- [ ] Hvordan auth-emails sync-es til Supabase Dashboard (lenker til `supabase-admin-setup.md`)
- [ ] Resend domain-verifisering nevnt (krever DKIM/SPF/DMARC fra §16)
- [ ] Testing-strategi (iPhone Mail, Gmail, Outlook, dark mode)
- [ ] Footer-credit «Utviklet av Myrvoll-Lunde IT Drift» i hver mal

---

## Utvidet mal-katalog — analyser kundedata og lag ALLE som trengs

AI skal **aktivt identifisere** hvilke eposter prosjektet trenger basert på funksjonalitet i kundedata. **Aldri stopp ved «default-listen»** — gå gjennom hver feature og spør «trenger denne en email?»

### Per feature-type — eposter som MÅ være med

| Feature i prosjektet | Eposter som må lages |
|----------------------|----------------------|
| Kontaktskjema | Bekreftelse til avsender + internvarsel til admin |
| Bestilling / kjøp | Bestillingsbekreftelse (kunde) + intern ordre-melding + faktura (med PDF vedlagt) + leveringsbekreftelse + spørrelse om anmeldelse 7 dager etter |
| Booking / reservasjon | Bekreftelse + påminnelse 24t før + endring/avbestilling + takk-for-besøket etterpå |
| Medlemskap / abonnement | Velkomst + faktura per periode + utløp-varsel 14d før + fornyelse-bekreftelse + oppsigelse-bekreftelse |
| Tilbudsforespørsel | Bekreftelse til kunde + intern lead-varsel + oppfølging hvis ikke svar etter 3d |
| Nyhetsbrev signup | Double opt-in bekreftelse + velkomst + ukentlig/månedlig newsletter |
| Brukerkonto | Magic link + confirm email + reset password + invitasjon + change email + reauthentication |
| Multi-tenant invitasjoner | Invitasjon m/aksepter-knapp + onboarding etter accept |
| Kommentarer / messaging | Varsel om svar + ukentlig oppsummering (opt-in) |
| Pris-endring | Varsel 30 dager før (lovkrav for abonnement) |
| Schedulert event | Event-bekreftelse + påminnelse 24t/1t før + oppsummering etterpå |
| Cron-job feiler | Intern alert til admin |
| Sikkerhetshendelse | Varsel til berørte brukere (GDPR Art. 34) |

### Hvis prosjektet er en transport- / bookingbedrift

Eks. budogvare.no — full epost-suite:

- `auth/MagicLink.tsx`
- `auth/ConfirmEmail.tsx`
- `auth/ResetPassword.tsx`
- `auth/Invitation.tsx`
- `auth/ChangeEmail.tsx`
- `auth/Reauthentication.tsx`
- `transactional/ContactReceipt.tsx`
- `transactional/ContactInternalAlert.tsx`
- `transactional/QuoteRequestReceipt.tsx`
- `transactional/QuoteFromAdmin.tsx`
- `transactional/BookingConfirmation.tsx`
- `transactional/BookingReminder24h.tsx`
- `transactional/BookingChange.tsx`
- `transactional/BookingCancellation.tsx`
- `transactional/InvoiceSent.tsx`
- `transactional/PaymentReceived.tsx`
- `transactional/ReviewRequest.tsx` (7d etter levering)
- `marketing/Welcome.tsx`
- `marketing/MonthlyNewsletter.tsx`

AI skal være eksplisitt om hver mal: tittel + når den trigges + hvem som mottar + hvilke variabler + hvilken Server Action / cron som sender den.

---

## Sette Supabase Auth-templates programmatisk (ikke bare Dashboard)

Supabase har Management API for å sette email-templates uten å gå inn i Dashboard. AI skal bruke dette + dokumentere i README.

### Via REST API

```bash
SUPABASE_ACCESS_TOKEN=sbp_xxx
PROJECT_REF=abc123xyz

# Render react-email til HTML først
npx render-email auth/MagicLink.tsx > /tmp/magic-link.html

# Sett magic-link template
HTML=$(cat /tmp/magic-link.html | python3 -c "import json,sys; print(json.dumps(sys.stdin.read()))")
curl -X PATCH \
  "https://api.supabase.com/v1/projects/$PROJECT_REF/config/auth" \
  -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"mailer_templates_magic_link_content\": $HTML, \"mailer_subjects_magic_link\": \"Logg inn på Skedsmo Bud & Vare\"}"
```

Templates som kan settes via API:

| Type | API-feltnavn |
|------|--------------|
| Confirm signup | `mailer_templates_confirmation_content` + `mailer_subjects_confirmation` |
| Magic link | `mailer_templates_magic_link_content` + `mailer_subjects_magic_link` |
| Reset password | `mailer_templates_recovery_content` + `mailer_subjects_recovery` |
| Invite user | `mailer_templates_invite_content` + `mailer_subjects_invite` |
| Change email | `mailer_templates_email_change_content` + `mailer_subjects_email_change` |
| Reauthentication | `mailer_templates_reauthentication_content` + `mailer_subjects_reauthentication` |

### Via npm-script per prosjekt

```ts
// scripts/sync-supabase-auth-templates.ts
import { render } from '@react-email/render'
import MagicLink from '@/emails/auth/MagicLink'
import ConfirmEmail from '@/emails/auth/ConfirmEmail'
// ...

const SUPABASE_ACCESS_TOKEN = process.env.SUPABASE_ACCESS_TOKEN!
const PROJECT_REF = process.env.SUPABASE_PROJECT_REF!

async function syncTemplate(field: string, subjectField: string, html: string, subject: string) {
  await fetch(`https://api.supabase.com/v1/projects/${PROJECT_REF}/config/auth`, {
    method: 'PATCH',
    headers: {
      Authorization: `Bearer ${SUPABASE_ACCESS_TOKEN}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ [field]: html, [subjectField]: subject }),
  })
}

async function main() {
  const magicLinkHtml = await render(MagicLink({ confirmationUrl: '{{ .ConfirmationURL }}' }))
  await syncTemplate(
    'mailer_templates_magic_link_content',
    'mailer_subjects_magic_link',
    magicLinkHtml,
    'Logg inn på {{ .SiteName }}'
  )
  // ... resten av malene
}

main()
```

Kjør: `npx tsx scripts/sync-supabase-auth-templates.ts` på Sprint 0 + ved hver oppdatering av maler.

### Gjør den til en del av CI

```yaml
# .github/workflows/sync-auth-templates.yml
on:
  push:
    branches: [main]
    paths: ['emails/auth/**']

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: pnpm install
      - run: npx tsx scripts/sync-supabase-auth-templates.ts
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
          SUPABASE_PROJECT_REF: ${{ secrets.SUPABASE_PROJECT_REF }}
```

Da synkroniseres maler automatisk ved hver push til main. Aldri «glem å oppdatere Dashboard» mer.

---

## Sjekkliste for AI (utvidet)

- [ ] **Analysert kundedata** og identifisert ALLE eposter prosjektet trenger (ikke bare default)
- [ ] **Hver mal listet** med navn, trigger, mottaker, variabler, Server Action / cron
- [ ] **Mappestruktur** med komponenter, auth, transactional, marketing
- [ ] **`scripts/sync-supabase-auth-templates.ts`** spesifisert og inkludert i Sprint 0
- [ ] **CI-workflow** for automatisk synk ved endring av maler
- [ ] **react-email + Resend** valgt som default-stack
- [ ] **Footer-credit i hver mal**
- [ ] **Design-tokens fra valgt 2026-retning** brukt
- [ ] **Testing**: Litmus/Mailtrap + Mail-Tester ≥ 9/10 + ekte test til personlig e-post
- [ ] **Resend domain verifisert** (DKIM/SPF/DMARC fra §16)
- [ ] **Supabase SMTP** konfigurert med Resend
