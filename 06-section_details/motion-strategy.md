# Section Detail — Motion-strategi (`motion_preference` MÅ implementeres)

> Helge har valgt et `motion_preference` i kickstart-wizarden. Det er et **løfte til kunden** om hvordan siden skal føles. Container Consult (30. mai 2026): wizarden sa «Livlig — scroll-effekter og hover-animasjoner». Sidene endte opp helt statiske. Det er en kontraktbrudd, ikke en stilistisk preferanse. **Aldri igjen.**

---

## Hovedregel

Hvert `motion_preference`-valg i wizarden mapper til en **konkret implementasjon-leveranse**. Det er ikke en «stemning» AI kan vifte med — det er en sjekkliste som MÅ leveres og verifiseres i Playwright.

| `motion_preference` | Bibliotek | Hva som MÅ leveres | Verifiseres med |
|---|---|---|---|
| **Statisk** | (ingen) | Ingen JS-motion. Kun fokus-ringer for tilgjengelighet. | `L1=true` (motion er bevisst fraværende — ingen sjekker kjøres) |
| **Minimal** | CSS transitions | `transition` på hover/focus på interaktive elementer. Ingen scroll-trigger. | L1: hover-transition finnes (≥3 elementer) |
| **Lett** | CSS animations + transitions | Hover/focus-transitions + CSS keyframe-animasjoner ved page-load (fade-in/slide-up på hero). Ingen scroll-trigger. | L1: hover, L2: load-animation |
| **Livlig** | Framer Motion (eller motion.dev) | Hover-animasjoner + scroll-baserte reveals (IntersectionObserver/Framer `whileInView`) + page-transitions | L1: hover, L2: scroll-reveal, L3: page-transition |
| **Avansert scrollytelling** | GSAP + ScrollTrigger | Pin/parallax-seksjoner + scrubbed timelines + horizontal scroll + tekst-reveals | L1-L4: alle motion-typer + minst én pin/parallax-seksjon |
| **Custom** | (Helge velger) | Spesifisert i §9.3 eller separat brief | Per Helges beskrivelse |

**Hvis sprint rapporteres ferdig med `motion_preference: 'Livlig'` men siden ikke har scroll-reveals → sprint er ikke ferdig.** L-kategorien i kvalitetsrubrikken blokkere.

---

## Implementasjon per preferanse

### `Statisk`

Ingen JS-motion. Eneste tillatt CSS:

```css
*:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}
```

Spesifiser i §9.4 i PROJECT.md: *«Bevisst statisk — ingen scroll/hover-motion. Begrunnelse: [eks. eldre målgruppe, reduced-motion-default, ytelse-fokus].»*

### `Minimal`

CSS-only:

```css
a, button {
  transition: color 150ms ease-out, background-color 150ms ease-out;
}

a:hover, button:hover { /* farge-endring */ }
button:hover { transform: translateY(-1px); }

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    transition-duration: 0.01ms !important;
  }
}
```

Ingen JS-bibliotek installeres.

### `Lett`

CSS transitions + keyframe-animasjoner ved load:

```css
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}

.hero h1 { animation: fadeUp 600ms var(--ease-out-expo) both; }
.hero p  { animation: fadeUp 600ms var(--ease-out-expo) 100ms both; }
.hero a  { animation: fadeUp 600ms var(--ease-out-expo) 200ms both; }
```

Ingen bibliotek. Bare velg-styled CSS-animations på hero.

### `Livlig` (Container Consult-valget)

Installer Framer Motion:

```bash
pnpm add motion  # Framer Motion v11+ heter "motion" på npm
```

Implementer minst alle av disse:

**1. Hover på CTA-er og kort:**
```tsx
import { motion } from 'motion/react'

<motion.a
  whileHover={{ scale: 1.02, y: -2 }}
  whileTap={{ scale: 0.98 }}
  transition={{ type: 'spring', stiffness: 400, damping: 17 }}
  href="/bestill"
>
  Bestill container
</motion.a>
```

**2. Scroll-reveal på seksjoner (IntersectionObserver via Framer):**
```tsx
import { motion } from 'motion/react'

export function RevealSection({ children }: { children: React.ReactNode }) {
  return (
    <motion.section
      initial={{ opacity: 0, y: 24 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true, amount: 0.3 }}
      transition={{ duration: 0.6, ease: [0.16, 1, 0.3, 1] }}
    >
      {children}
    </motion.section>
  )
}
```

Bruk på minimum: hero, signature moment, tjenester-bento, nyheter-feed, kontakt-seksjon.

**3. Stagger på lister:**
```tsx
<motion.ul
  initial="hidden"
  whileInView="visible"
  viewport={{ once: true }}
  variants={{
    visible: { transition: { staggerChildren: 0.08 } }
  }}
>
  {items.map((item) => (
    <motion.li
      key={item.id}
      variants={{
        hidden: { opacity: 0, y: 16 },
        visible: { opacity: 1, y: 0 },
      }}
    >
      {item.name}
    </motion.li>
  ))}
</motion.ul>
```

**4. Page-transitions (Next.js):**
```tsx
// src/app/(public)/template.tsx
'use client'
import { motion } from 'motion/react'

export default function Template({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      transition={{ duration: 0.3 }}
    >
      {children}
    </motion.div>
  )
}
```

**5. Reduced motion-respekt (kritisk):**
```tsx
import { useReducedMotion } from 'motion/react'

const shouldReduceMotion = useReducedMotion()
const animationProps = shouldReduceMotion
  ? {}
  : { whileHover: { scale: 1.02 }, transition: { duration: 0.3 } }
```

Eller globalt i `globals.css`:
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

### `Avansert scrollytelling`

Installer GSAP + ScrollTrigger:

```bash
pnpm add gsap
```

Implementer minst:
- 1+ pin/parallax-seksjon (innholdet henger fast mens man scroller)
- 1+ scrubbed timeline (animasjon-progress følger scroll-posisjon)
- Tekst-reveal med SplitText (eller manuell char-splitting)
- Horizontal scroll-seksjon (valgfritt)

Følg `06-SECTION_DETAILS/scrollytelling.md` (hvis den finnes — ellers se Framer Motion-dokumentasjon).

### `Custom`

Helge spesifiserer i `design_references`-feltet eller §9.4. AI implementerer etter brief.

---

## Verifisering (Playwright)

`scripts/check-public-pages.ts` får ny L-kategori basert på `project.motion_preference`. AI leser den fra wizarden via en `motion_preference.json`-fil eller fra `project_motion`-tabell i Supabase.

### L-sjekker (per `motion_preference`)

```ts
// scripts/check-motion-strategy.ts (inkluderes i check-public-pages.ts)
async function checkMotion(page: Page, pref: MotionPreference): Promise<L>
```

| Sjekk | Hva | Når kjøres |
|---|---|---|
| **L1: Hover-transitions** | Hover på første CTA → får `transform`/`background-color`/`color`-endring innen 200ms. Bruk Playwright `page.locator('a,button').first().hover()` + sammenlign computed styles før/etter. | Alle preferences unntatt `Statisk` |
| **L2: Scroll-reveal** | Scroll til midten av siden, vent 500ms, sjekk at minst én seksjon har `opacity` gått fra <0.5 til >0.9 (eller `transform` har endret seg). | `Lett`, `Livlig`, `Avansert` |
| **L3: Motion-bibliotek installert** | `package.json` har `motion`/`framer-motion` (Livlig+) eller `gsap` (Avansert). | `Livlig`, `Avansert` |
| **L4: prefers-reduced-motion respekteres** | Sett `page.emulateMedia({ reducedMotion: 'reduce' })`, reload, verifiser at animations er <50ms varighet. | Alle preferences med JS-motion |

### Implementasjon-skjelett

```ts
// I check-public-pages.ts main()
const motionPref = await getMotionPreference()  // fra DB eller config

async function checkMotionOnPage(page: Page): Promise<{ l1: boolean; l2: boolean; l3: boolean; l4: boolean }> {
  if (motionPref === 'Statisk') return { l1: true, l2: true, l3: true, l4: true }

  // L1: hover-transition
  const firstCta = page.locator('header a[href*="bestill"], header a.cta, main a.cta, button[type="submit"]').first()
  const beforeBox = await firstCta.evaluate((el) => getComputedStyle(el).backgroundColor + ' ' + getComputedStyle(el).transform)
  await firstCta.hover()
  await page.waitForTimeout(250)
  const afterBox = await firstCta.evaluate((el) => getComputedStyle(el).backgroundColor + ' ' + getComputedStyle(el).transform)
  const l1 = beforeBox !== afterBox

  // L2: scroll-reveal
  const beforeOpacity = await page.locator('main > :nth-child(3)').evaluate((el) => parseFloat(getComputedStyle(el).opacity))
  await page.evaluate(() => window.scrollBy(0, window.innerHeight))
  await page.waitForTimeout(700)
  const afterOpacity = await page.locator('main > :nth-child(3)').evaluate((el) => parseFloat(getComputedStyle(el).opacity))
  const l2 = motionPref === 'Minimal' ? true : (beforeOpacity < 0.5 && afterOpacity > 0.9) || beforeOpacity === 1 // accept already-visible
  // (juster heuristikk per behov)

  // L3: bibliotek installert
  const pkg = await readFile('package.json', 'utf-8').then(JSON.parse)
  const deps = { ...pkg.dependencies, ...pkg.devDependencies }
  const l3 =
    (['Livlig'].includes(motionPref) && (deps['motion'] || deps['framer-motion'])) ||
    (motionPref === 'Avansert scrollytelling' && deps['gsap']) ||
    (['Minimal', 'Lett'].includes(motionPref))  // CSS-only, ingen bibliotek-krav

  // L4: reduced-motion
  await page.emulateMedia({ reducedMotion: 'reduce' })
  await page.reload({ waitUntil: 'networkidle' })
  const durationOk = await page.locator('main').evaluate((el) => {
    const d = getComputedStyle(el).transitionDuration
    return parseFloat(d) < 0.1 || d === '0s'
  })
  const l4 = durationOk

  return { l1, l2, l3, l4 }
}
```

---

## Datamodell

`motion_preference` lagres i `kickstart_projects`-tabellen (allerede der i mlit-wizarden). Bootstrap-CLI eksporterer den til prosjekt-repoets `motion.config.json`:

```json
{
  "preference": "Livlig",
  "library": "motion",
  "verify_levels": ["L1", "L2", "L3", "L4"]
}
```

`scripts/check-public-pages.ts` leser denne ved oppstart.

---

## Anti-patterns

### ❌ Nevne motion i §9 men ikke implementere
> «Framer Motion brukes for scroll-reveals og hover-animasjoner.» — så ingen `import { motion }` noe sted i koden. Container Consult-feilen.

### ❌ Hardkodet animasjon-overstyring uten reduced-motion-respekt
> CSS-animation kjører i 2 sekunder selv om brukeren har slått på «Reduce motion» i OS-innstillinger. Tilgjengelighet-fail.

### ❌ Motion-bibliotek installert men kun brukt på én komponent
> `framer-motion` er i `package.json`, men eneste `<motion.div>` er på 404-siden. Brukeren ser ingen forskjell. L1-L2-rød.

### ❌ Velge `Livlig` og levere `Statisk`
> Container Consult-feilen. Kontraktbrudd.

---

## Sprint-spesifikasjon (kopier inn i sprinten som leverer designsystem)

```markdown
**Motion-leveranse (§9.4 — basert på motion_preference fra wizarden)**

- [ ] Installer riktig bibliotek (se mapping over)
- [ ] Implementer L1: hover-transitions på CTA-er + kort (≥3 elementer)
- [ ] Implementer L2: scroll-reveal på minst 5 seksjoner (hero, signature, tjenester, nyheter, kontakt)
- [ ] Implementer L3: stagger på lister (tjenester-bento, nyheter-feed)
- [ ] Implementer L4: page-transitions (`(public)/template.tsx`)
- [ ] Verifiser reduced-motion-respekt med `page.emulateMedia({ reducedMotion: 'reduce' })`
- [ ] `check-public-pages.ts` L1-L4 grønne for valgt preference
```

---

## For AI

1. **Les `motion_preference` fra wizard-input FØRST.** Det er prioritet 2 etter signature moment.
2. **Map til konkret implementasjon** (se tabellen over). Ikke improviser.
3. **Implementer på ALLE sider, ikke bare forsiden.** Hover-transitions må gjelde alle CTA-er, scroll-reveals må gjelde alle seksjoner.
4. **Verifiser i Playwright** før du rapporterer sprint ferdig. L1-L4 grønne = OK.
5. **Hvis du fristes til å droppe motion fordi «det er stresende»** — det er nettopp kontraktbruddet vi fikser med denne docen. Gjør jobben.
