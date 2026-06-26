# Section Detail — Brand-assets (logo, favicon, OG-bilder) som Sprint 1-blocker

> Container Consult AS i Sprint 10: header viste en gul prikk i stedet for logo. Favicon var default Vercel-favicon. OG-image manglet. Det er ikke et 10/10-prosjekt. Disse tre må være på plass FØR Sprint 2 starter — og verifiseres av check-public-pages.ts som hard blokker.

---

## Hva som er obligatorisk

| Asset | Path | Format | Krav | Kilde |
|---|---|---|---|---|
| **Hoved-logo** | `public/logo.svg` | SVG (foretrukket) eller PNG ≥240×80 | Vektor eller 2× retina | Scrape fra `existing_url` → finn `<img>` i header / Get from kunden |
| **Logo (lys versjon)** | `public/logo-light.svg` | SVG | For mørke bakgrunner | Hvis logo trenger varianter |
| **Logo (mørk versjon)** | `public/logo-dark.svg` | SVG | For lyse bakgrunner | Hvis logo trenger varianter |
| **Favicon** | `public/favicon.ico` | ICO multi-resolution (16, 32, 48) | Eller bruk `public/icon.svg` (Next 16-konvensjon) | Generert fra logo |
| **Apple touch icon** | `public/apple-icon.png` | PNG 180×180 | Avrundede hjørner OK | Generert fra logo |
| **Android/PWA-ikon** | `public/icon-192.png` + `public/icon-512.png` | PNG | Brukes i manifest.json | Generert fra logo |
| **OG-image (default)** | `public/og-image.png` | PNG/JPG 1200×630 | Logo + tagline mot brand-bakgrunn | Designet, ikke screenshot |
| **OG-image (dynamisk)** | `src/app/opengraph-image.tsx` | Next 16 generated | Per-side OG med tittel + brand | Code, ikke fil |

---

## Sprint 1 done-criteria (blocker)

`scripts/check-public-pages.ts` MÅ gi grønt på:

- `C1 Logo i header` — `<header img[alt*="logo"]>` eller `<header svg>` returneres
- `C2 Favicon 200` — HEAD til `/favicon.ico` eller `/favicon.svg` returnerer 200, ikke default Next-favicon
- `C3 OG image laster` — `<meta property="og:image">` peker til URL som returnerer 200 + bildet er ≥1200×630

Hvis noen av disse er røde: sprint 1 er ikke ferdig.

---

## Hvor logoen kommer fra

Prioritetsrekkefølge AI skal følge:

1. **Wizard-input** — hvis Helge har lastet opp logo i wizarden, bruk den
2. **Scraping** — hvis `existing_url` finnes og logoen kan utvinnes:
   ```ts
   // del av scripts/scrape-existing-site.ts
   const logoSrc = await page.locator('header img[alt*="logo" i], header img:first-of-type').first().getAttribute('src')
   if (logoSrc) await downloadTo(`public/logo.${ext(logoSrc)}`, logoSrc)
   ```
3. **Plassholder** — generer SVG med initialer + brand-primary-color:
   ```tsx
   // public/logo.svg (fallback hvis ingenting kan hentes)
   <svg viewBox="0 0 240 64" xmlns="http://www.w3.org/2000/svg">
     <rect width="64" height="64" rx="12" fill="{{color_primary}}"/>
     <text x="32" y="42" font-size="28" font-family="system-ui" font-weight="700"
           text-anchor="middle" fill="white">{{initials}}</text>
     <text x="80" y="40" font-size="20" font-family="system-ui" font-weight="600"
           fill="currentColor">{{client_name}}</text>
   </svg>
   ```

**Plassholder må flagges i PROJECT.md §6 (Brand) som «midlertidig — kunden leverer logo innen Sprint 2».** Helge kan ikke ha en plassholder i prod.

---

## Favicon-genering (automatisk i Sprint 1)

```bash
# scripts/generate-favicons.ts — kjøres etter logo er på plass
pnpm dlx pwa-asset-generator public/logo.svg public/ \
  --icon-only \
  --favicon \
  --opaque false \
  --background "transparent"
```

Output: `favicon.ico`, `apple-icon.png`, `icon-192.png`, `icon-512.png`.

Hvis logo ikke finnes ennå → bruk plassholder-SVG-en over.

---

## OG-image

### Statisk (minimum)

`public/og-image.png` 1200×630 med:
- Logo
- Tagline / kjernebudskap
- Brand-bakgrunn (primary color eller gradient)
- IKKE bare en skjermdump av forsiden — det ser amatørmessig ut i sosiale medier

### Dynamisk (anbefalt — Next 16 ImageResponse)

```tsx
// src/app/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const size = { width: 1200, height: 630 }
export const contentType = 'image/png'

export default async function OG() {
  return new ImageResponse(
    (
      <div style={{ display: 'flex', flexDirection: 'column', height: '100%', width: '100%',
                    background: 'linear-gradient(135deg, #0b0b0b 0%, #1a1a1a 100%)', padding: 80 }}>
        <div style={{ fontSize: 32, color: '#FFD400', fontWeight: 600 }}>CONTAINER CONSULT AS</div>
        <div style={{ fontSize: 64, color: '#fff', fontWeight: 700, marginTop: 40, lineHeight: 1.1 }}>
          Container-utleie i Ålesund og omegn
        </div>
        <div style={{ fontSize: 24, color: '#9a9a9a', marginTop: 'auto' }}>
          50 års erfaring · Booking på 30 sek
        </div>
      </div>
    ),
    size,
  )
}
```

Hver side med `generateMetadata` får automatisk per-side OG-image.

---

## §6 Brand i PROJECT.md (mal — fyll ut fra wizarden)

```markdown
## 6. Brand-identitet

**Klient:** {{client_name}}
**Primary color:** {{color_primary}}
**Secondary color:** {{color_secondary}}

### Brand-assets (Sprint 1-leveranser)

| Asset | Status | Kilde |
|---|---|---|
| `public/logo.svg` | [ ] | [Wizard-input \| scraped fra `existing_url` \| plassholder] |
| `public/logo-light.svg` | [ ] | (hvis trenger varianter) |
| `public/favicon.ico` | [ ] | Generert fra logo med pwa-asset-generator |
| `public/apple-icon.png` | [ ] | Generert |
| `public/icon-192.png` | [ ] | Generert |
| `public/og-image.png` | [ ] | Designet — eller dynamisk via `src/app/opengraph-image.tsx` |

**Blocker:** Sprint 2 starter ikke før alle 6 har status ✓.
```

---

## For AI

Sprint 1 oppgaveliste i §20 MÅ inneholde:

- [ ] Skaff logo (wizard → scrape → plassholder, i denne rekkefølgen)
- [ ] Plasser i `public/logo.svg`
- [ ] Generer favicons med `pnpm dlx pwa-asset-generator`
- [ ] Lag `src/app/opengraph-image.tsx` (dynamisk) eller `public/og-image.png` (statisk)
- [ ] Sett `<link rel="icon">` + manifest.json
- [ ] Sett `<meta property="og:image">` (statisk fallback hvis ingen dynamic)
- [ ] Kjør `check-public-pages.ts` — C1/C2/C3 må alle være grønne
- [ ] Hvis plassholder brukt: flag i §6 + risikoliste at kunden må levere ekte logo innen Sprint 2
