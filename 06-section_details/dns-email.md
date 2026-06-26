# Section Detail — DNS + Email Deliverability

> Brukes når AI fyller ut seksjon 16 (DNS + email-deliverability) i PROJECT.md.

---

## Minimum required output

Seksjon 16 må inneholde:

1. **Domain setup** — hvilket domene, hvem eier DNS, hvor er det registrert
2. **DNS-records** — tabell med type/host/value
3. **Email-deliverability** — DKIM, SPF, DMARC, MX
4. **Redirect-strategi** — gamle URL-er → nye URL-er (hvis migrering)
5. **www-canonical** — www vs apex, hvilken er kanon
6. **TLS / cert** — Vercel auto, eller egen?
7. **Robots / sitemap / llms.txt** — riktig sti, oppdatert dynamisk

## Template

```markdown
## 16. DNS + email-deliverability

### 16.1 Domain setup

- **Hoveddomene:** [eksempel.no]
- **DNS-registrar:** [Domeneshop / Loopia / etc.]
- **DNS-host:** [Vercel / Cloudflare / registrar]
- **Eier av DNS-tilgang:** [Kunde / Myrvoll-Lunde IT Drift]

### 16.2 DNS-records

| Type | Host | Value | TTL | Formål |
|------|------|-------|-----|--------|
| A | @ | 76.76.21.21 | 3600 | Vercel apex (hvis ikke ALIAS støttet) |
| CNAME | www | cname.vercel-dns.com | 3600 | Vercel www |
| TXT | @ | v=spf1 include:_spf.resend.com -all | 3600 | SPF |
| TXT | resend._domainkey | [DKIM-verdi fra Resend] | 3600 | DKIM |
| TXT | _dmarc | v=DMARC1; p=quarantine; rua=mailto:dmarc@[domene]; pct=100; adkim=s; aspf=s | 3600 | DMARC |
| MX | @ | [evt. Google Workspace / Microsoft 365] | 3600 | Innkommende e-post |

### 16.3 Email-deliverability (Resend)

**Domain verification:**
1. Legg til domene i Resend dashboard
2. Resend gir DKIM-records (3 stk CNAME) som må legges til DNS
3. SPF: `v=spf1 include:_spf.resend.com -all`
4. Vent på "Verified" status (typisk 5-60 min)

**DMARC-startoppsett:**
- Start med `p=quarantine` (ikke `p=reject`) første 30 dager
- Sjekk DMARC-rapporter ukentlig
- Når deliverability er stabil → øk til `p=reject`

**Sender:**
- **From**: `[no-reply | post]@[domene]`
- **Reply-To**: `[support | post]@[domene]`
- **Aldri** send fra Gmail/Outlook-adresser i prod

### 16.4 Redirect-strategi

Hvis prosjektet erstatter en eksisterende side:

| Gammel URL | Ny URL | Status |
|------------|--------|--------|
| /tjenester/varebil | /biler/varebil | 301 |
| /pris.pdf | /priser | 301 |
| /kontakt-oss | /kontakt | 301 |

Implementert i `next.config.ts`:

\`\`\`ts
async redirects() {
  return [
    { source: '/tjenester/varebil', destination: '/biler/varebil', permanent: true },
    // ...
  ]
}
\`\`\`

### 16.5 www-canonical

**Default:** apex (uten www) er kanon. www → 301 til apex.

Implementer i Vercel ved å sette "Primary Domain" til apex, og legg til www som redirect.

Hvis kunden har eksisterende SEO-investering i www, snu det.

### 16.6 TLS / cert

- **Vercel auto-provisions** Let's Encrypt cert
- Sjekk at HTTPS er enforced (Vercel default)
- HSTS preload header sendt fra `middleware.ts` (se seksjon 14)

### 16.7 Robots / sitemap / llms.txt

| Fil | Sti | Innhold |
|-----|-----|---------|
| `robots.txt` | `/robots.txt` | User-agent: *, Allow: /, Sitemap: https://[domene]/sitemap.xml |
| `sitemap.xml` | `/sitemap.xml` | Auto-generert fra `pages` + `blog_posts` (status='published') |
| `llms.txt` | `/llms.txt` | Index av hovedinnhold + lenker til llms-full.txt |
| `llms-full.txt` | `/llms-full.txt` | Full tekst-export for LLM-konsum |

Implementert som Next.js route handlers i `app/robots.ts`, `app/sitemap.ts`.
```

## Vanlige fallgruver

| Problem | Fix |
|---------|-----|
| DMARC `p=reject` for tidlig → e-post bouncer | Start med `p=quarantine` |
| SPF inkluderer flere providers uten `redirect=` | Konsolider eller bruk `redirect=` |
| DKIM-record for lang for DNS UI | Splittes automatisk av Resend |
| Apex CNAME ikke støttet hos registrar | Bruk A-record til Vercel IP |
| Gamle URL-er ikke 301-redirectet → SEO-tap | Sjekk Google Search Console for 404-er etter cutover |
| www og apex begge serverer → duplicate content | Velg én som kanon, redirect den andre |
