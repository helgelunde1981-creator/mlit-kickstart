# Dark Luxury Motion — Font-pairings

## Primær: Geist + Geist Mono

**Hvorfor:** Geist er Vercel sin moderne sans med stor variabel-range (100-900). Smelter sammen "tech" og "luxury" — moderne nok til å føles 2026, men har vekt-flexibilitet for store typografi-moments. Geist Mono gir det monospace-aksenten som dark luxury trenger for "live"-indikatorer og metadata.

**Bruk:**
- Display og body: Geist (700 for hero, 500 for cards/nav, 400 for body)
- Mono labels, metadata, "live"-indikatorer: Geist Mono (uppercase, tracking-wide)

**Installer:**

```bash
pnpm add geist
```

```tsx
// app/layout.tsx
import { GeistSans } from 'geist/font/sans'
import { GeistMono } from 'geist/font/mono'

export default function RootLayout({ children }) {
  return (
    <html lang="nb" className={`${GeistSans.variable} ${GeistMono.variable}`}>
      <body>{children}</body>
    </html>
  )
}
```

```css
/* tokens.css */
:root {
  --font-sans: var(--font-geist-sans), system-ui, sans-serif;
  --font-mono: var(--font-geist-mono), ui-monospace, monospace;
}
```

## Alternativ 1: Cal Sans + Inter

**Hvorfor:** Cal Sans gir mer karakter (litt rounded, friendly) — fungerer hvis kunden er premium men "approachable". Inter for body fordi det er pålitelig.

**Bruk:**
- Display: Cal Sans (kun headlines, store moments)
- Body: Inter

**Installer:**

```bash
pnpm add @vercel/font
```

Cal Sans krever lokal font-fil eller `next/font/local`.

## Alternativ 2: Migra + Söhne

**Hvorfor:** Migra Display gir editorial-luksus-feel. Söhne (Klim) gir Swiss-stramhet i body. Begge er premium-typografi som signaliserer kvalitet.

**Bruk:**
- Display: Migra Display Bold
- Body: Söhne Regular

**Lisens:** Söhne fra Klim type foundry, krever lisens. Migra fra Pangram Pangram (gratis for personlig, betalt for kommersielt).

**Bruk kun hvis budsjettet tåler lisens-kost.**

## Aldri bruk for denne retningen

- **Default system fonts** alene — føles uferdig
- **Comic Sans / playful fonts** — bryter luxury-vibe
- **Tunge slab-serifs** — kolliderer med dark sleek aesthetic
- **For mange fonter** — maks 2 familier, helst 1 + mono
