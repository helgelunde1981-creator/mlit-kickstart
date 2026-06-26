# Design-direction-templates

Konkrete kode-eksempler per `design_style` fra kickstart-wizarden. AI MÅ kopiere/tilpasse fra riktig direction-mappe i stedet for å defaulte til generic shadcn-template.

**Hvorfor dette finnes (Container Consult, mai 2026):** Helge valgte «Industrial / Robust» i wizard. AI bygde sider med Inter_Tight + rounded-lg + shadow-md + sentrert hero med mørk overlay = stock shadcn-template. Claude vision scoret V3 (direction-fidelitet) til 2-4/10. Snitt 3.1/10. Helge: «ser ut som begrenset modulsystem». **Aldri igjen.**

## Per direction: 5 obligatoriske filer

| Fil | Hva |
|---|---|
| `tokens.css` | Palette + fonts + spacing + shadows specific to direction |
| `hero.tsx` | Referanse-hero med riktig visuell tyngde (V1=7+) |
| `cta-button.tsx` | CTA-stil som matcher direction (ikke default-rounded-shadow) |
| `bento-grid.tsx` | Layout med ekte hierarki/asymmetri (V2=7+) |
| `anti-patterns.md` | Konkrete fail-mønstre AI MÅ unngå for denne direction |

## Direction-mapping (fra `02-DESIGN_DIRECTIONS.md`)

| Direction | Mappe | Karakteristikker (V3 sjekker disse) |
|---|---|---|
| Industrial / Robust | `industrial-robust/` | Kondenserte tunge sans (Barlow Condensed, Bebas Neue), hard-shadow uten rounding, fotografi-tunge heros, gul/rød accents på sort/grå, gjentakende bilde-motivkomponenter |
| Editorial Bento | `editorial-bento/` | Asymmetrisk grid, store typografiske kontraster, redaksjonell rytme, hvit-rom som strukturelement, serif+sans-mix |
| Dark Luxury Motion | `dark-luxury-motion/` | Mørk default, gull/kobber-accents, glanseffekter, langsom motion, høy-kontrast typografi |
| Swiss Minimal Refined | `swiss-minimal-refined/` | Grid-disiplin, monospace/serif-blanding, mye hvit-rom, presisjon |
| Brutalism Refined | `brutalism-refined/` | Tunge skyggevirkninger, kanter, asymmetri, monospace dominerer |
| Glassmorphism Depth | `glassmorphism-depth/` | Frostete bakgrunner, backdrop-blur, layering |
| Retro-Futurism | `retro-futurism/` | 70/80-tall-paletter, scanlines, neon-accents, retro typografi |
| Scrollytelling Editorial | `scrollytelling-editorial/` | Pin-seksjoner, scrubbed reveals, story-flow |
| 3D Integration | `3d-integration/` | WebGL/Three.js, perspektiv-effekter, lys-skygge |

## For AI

1. **Les `design_style` fra wizarden** før du bygger noe UI.
2. **Kopier/tilpass fra `EXAMPLES/design-directions/<slug>/`** for hero, CTA, bento.
3. **Aldri default til shadcn-base-komponenter** uten å først applikere direction-tokens.
4. **`check-visual-quality.ts` blokker** hvis V3 (direction-fidelitet) <6 — Claude vision sammenligner mot karakteristikkene over.
5. **Anti-patterns per direction MÅ leses** før du skriver kode — det er konkrete feller AI har gått i.
