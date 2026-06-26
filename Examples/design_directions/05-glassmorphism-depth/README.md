# Glassmorphism Depth

> Ekte lag, ikke flat dekorasjon. Backdrop-blur brukt målbevisst for å skape z-akse. Ofte kombinert med subtil noise/grain for å unngå "plast"-følelse.

## Use når

- Fintech med trygg modern feel
- Dashboards med kompleks state
- AI-produkter
- SaaS som vil vise dybde i features

## Ikke bruk når

- Lav-end devices er primær målgruppe (krever blur-support)
- Tekst-tunge content sites
- Når performance er ekstra kritisk

## Signaturmoves

- **Glass-kort med faktisk dybde** — backdrop-blur 12-16px, ikke 4px
- **Hover som trekker frem** — z-index shift + blur-økning
- **Gradient mesh i bakgrunn** — animert subtilt
- **Noise/grain overlay** — 2-3% opacity SVG noise
- **Fokus på visuell stabilitet** — ikke for mye motion
- **Floating elements** — sticky elements på scroll med blur-økning

## Filer

- `tokens.css`
- `references.md`
