# 3D Integration — Referanser

| Site | URL | Hvorfor |
|------|-----|---------|
| Apple AirPods Pro | https://apple.com/airpods-pro | Reference 3D-produkt-page |
| Tesla | https://tesla.com | 3D bil-konfigurator integrert i kjøp-flyt |
| Spline | https://spline.design | Eier kategorien for "Spline-look" |
| Bruno Simon (portfolio) | https://bruno-simon.com | Three.js portfolio som er hele opplevelsen |
| Linear (older) | https://linear.app | Sublim 3D-integrasjon (når den ble lansert) |
| Stripe (gradient sphere) | https://stripe.com | Subtil 3D-animasjon i hero |
| Vercel (rebrand) | https://vercel.com | 3D-elementer i marketing |

## Hva å se etter

1. Hva er fallback for low-end devices?
2. Hvor mye av siden er 3D vs static?
3. Performance — laster siden under 2s?
4. Hvordan håndteres mobile (forenklet 3D eller poster.jpg)?

## Tekniske noter

- **Spline:** drag-and-drop, raskt for produktvisualisering, gratis-nivå tilstrekkelig for mindre prosjekter
- **Three.js + React Three Fiber:** code-driven, ubegrenset, lengere build-tid
- **Krever:** WebGL-support, GPU-driver-håndtering, fallback til static
- **Bundle-impact:** Three.js er ~150 KB gzipped — vurder dynamic import
