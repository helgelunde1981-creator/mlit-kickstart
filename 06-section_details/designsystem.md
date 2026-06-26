# Section Detail — Designsystem som kode

> Brukes når AI fyller ut seksjon 9 (Designsystem) i PROJECT.md. Krever **konkrete tokens**, ikke prosa.

---

## Minimum required output

Seksjon 9 må inneholde:

1. **Komplett `tokens.css`** — alle CSS custom properties
2. **Tailwind v4-config-utdrag** — som mapper tokens til klasser
3. **shadcn-komponenter brukt** — eksplisitt liste med begrunnelse
4. **Ikon-system** — typisk Lucide React med liste over hyppige ikoner
5. **Motion-tokens** — durations, eases, transition-presets
6. **Type-skala** — concrete sizes med clamp() for fluid type
7. **Spacing-skala** — base unit + skalaen brukt
8. **Component variants** — hver shadcn-komponent som er customisert: hva er endret

## Template

```markdown
## 9. Designsystem som kode

### 9.1 Tokens (`src/styles/tokens.css`)

\`\`\`css
:root {
  /* Color — surface */
  --color-bg: oklch(...);
  --color-surface-1: oklch(...);
  --color-surface-2: oklch(...);

  /* Color — text */
  --color-text-primary: oklch(...);
  --color-text-secondary: oklch(...);
  --color-text-muted: oklch(...);

  /* Color — brand */
  --color-accent-primary: oklch(...);
  --color-accent-secondary: oklch(...);

  /* Color — semantic */
  --color-success: oklch(...);
  --color-warning: oklch(...);
  --color-error: oklch(...);
  --color-info: oklch(...);

  /* Type */
  --font-sans: 'Geist', system-ui, sans-serif;
  --font-mono: 'Geist Mono', ui-monospace, monospace;
  --font-display: 'Geist', sans-serif;

  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: clamp(1rem, 0.92rem + 0.4vw, 1.125rem);
  --text-lg: clamp(1.125rem, 1rem + 0.625vw, 1.375rem);
  --text-xl: clamp(1.5rem, 1.2rem + 1.5vw, 2rem);
  --text-2xl: clamp(2rem, 1.4rem + 3vw, 3rem);
  --text-hero: clamp(3rem, 1rem + 7vw, 8rem);

  /* Spacing — base 4px */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-12: 3rem;
  --space-16: 4rem;
  --space-24: 6rem;
  --space-section: clamp(4rem, 3rem + 5vw, 10rem);

  /* Radius */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 1rem;
  --radius-xl: 1.5rem;
  --radius-full: 9999px;

  /* Shadow (refer til design-direction) */
  --shadow-sm: ...;
  --shadow-md: ...;
  --shadow-lg: ...;

  /* Motion */
  --duration-fast: 150ms;
  --duration-normal: 300ms;
  --duration-slow: 500ms;
  --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-out-back: cubic-bezier(0.34, 1.56, 0.64, 1);
}
\`\`\`

### 9.2 Tailwind v4-config

\`\`\`ts
// tailwind.config.ts (utdrag)
import type { Config } from 'tailwindcss'

export default {
  theme: {
    extend: {
      colors: {
        bg: 'var(--color-bg)',
        'surface-1': 'var(--color-surface-1)',
        'surface-2': 'var(--color-surface-2)',
        'text-primary': 'var(--color-text-primary)',
        'text-secondary': 'var(--color-text-secondary)',
        'accent-primary': 'var(--color-accent-primary)',
        'accent-secondary': 'var(--color-accent-secondary)',
        success: 'var(--color-success)',
        warning: 'var(--color-warning)',
        error: 'var(--color-error)',
      },
      fontFamily: {
        sans: ['var(--font-sans)'],
        mono: ['var(--font-mono)'],
        display: ['var(--font-display)'],
      },
      borderRadius: {
        DEFAULT: 'var(--radius-md)',
        lg: 'var(--radius-lg)',
        xl: 'var(--radius-xl)',
      },
      transitionDuration: {
        fast: 'var(--duration-fast)',
        normal: 'var(--duration-normal)',
        slow: 'var(--duration-slow)',
      },
      transitionTimingFunction: {
        'out-expo': 'var(--ease-out-expo)',
        'out-back': 'var(--ease-out-back)',
      },
    },
  },
} satisfies Config
\`\`\`

### 9.3 shadcn-komponenter brukt (custom-tuned)

| Komponent | Customisering | Hvor brukt |
|-----------|---------------|------------|
| Button | Nye variants: `accent`, `ghost-accent`, `mono`. Hover: scale 1.02 + shadow-lg. Active: scale 0.98. | Alle CTA-er |
| Card | Default shadow byttet til `shadow-md`, hover: `shadow-lg` + translate-y-[-2px] | Card-grid på forside |
| Input | Border-radius `lg`. Focus-ring: 2px accent-primary med 0.2 opacity. | Forms |
| Badge | Nye varianter for status (draft/published/archived) | Admin |
| Dialog | Backdrop blur økt til 12px. Animation: scale 0.95 → 1.0 med fade. | Confirm-dialoger |
| Sonner (toast) | Posisjon: top-right. Stil matcher card-tokens. | Notifications |
| Tabs | Underline-style i stedet for pills | Admin |

### 9.4 Ikon-system

**Lucide React** — konsistent stroke-width: 1.5px.

Vanlige ikoner i prosjektet:
[Liste ikonene som faktisk brukes basert på funksjonalitet]

### 9.5 Component variants (eksempel: Button)

\`\`\`tsx
// src/components/ui/button.tsx (utdrag)
const buttonVariants = cva(
  'inline-flex items-center justify-center font-medium transition-all duration-fast ease-out-expo',
  {
    variants: {
      variant: {
        accent: 'bg-accent-primary text-white hover:scale-[1.02] hover:shadow-lg active:scale-[0.98]',
        ghost: 'bg-transparent text-text-primary hover:bg-surface-1',
        mono: 'font-mono text-xs uppercase tracking-wider text-text-secondary hover:text-text-primary',
      },
      size: {
        sm: 'h-9 px-3 text-sm rounded',
        md: 'h-11 px-5 rounded-lg',
        lg: 'h-14 px-7 text-lg rounded-xl',
      },
    },
  }
)
\`\`\`
```

## Avvik fra retning

Hvis kunden krever spesifikke farger som ikke matcher valgt design-direction perfekt, gjør AI dette:

1. Bruk kundens primær-farge som `--color-accent-primary`
2. Tilpass andre tokens for å beholde retningens "vibe" (f.eks. Dark Luxury Motion beholder dark surfaces selv om accent er rød)
3. Dokumenter trade-offs eksplisitt
