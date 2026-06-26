# Signature Moment — Scroll Product Tour

> Eksempel-mønster: scroll-triggered walk-through av et produkt med pinned visualization.

## Hva

En hero-til-bunn scroll-opplevelse der hovedvisualisering (typisk 3D-modell, video-loop, eller stor produkt-screenshot) sitter pinned mens tekst-stages glir forbi. Hver stage forteller en del av historien. Stage-bytte trigger animasjon i visualiseringen (rotasjon, zoom, feature-highlight).

## Hvor

- **Forside:** dominerer øvre halvdel av sida — første 3-5 viewports er scrollytelling
- **Produkt-detalj-side:** dypere versjon med 8-12 stages

## Oppførsel

### States

- **Initial:** stage 1 visible, visualization i default-tilstand
- **Scrolling:** progress-indikator viser hvor i fortellingen brukeren er
- **Stage active:** ved stage-trigger (snap-point), tekst får fokus, visualization animerer
- **Reduced-motion:** alle stages vises som vanlig vertical liste uten parallax/pin

### Transitions

- Stage-bytte: tekst fade out → ny tekst fade in (500ms)
- Visualization-animasjon: GSAP timeline med ease-out-expo
- Mobile: pin disabled, hver stage blir vanlig hero-seksjon

### Edge cases

- **Lav FPS:** detect via `requestAnimationFrame` timing — degrader til simpler transitions
- **No GPU:** disable 3D, fall back til static images per stage
- **Browser back:** scroll-restore må håndteres (deactivate ScrollTrigger ved unmount)

## Hvorfor signature

Egnet når produktet er komplekst eller har flere features som krever forklaring. Vanlig "feature list" føles flatt — scrolly-tour lar brukeren oppdage funksjonalitet i en kontrollert rytme. Skaper "wow"-moment som er memorable.

Eksempel: Apple iPhone-launch sider, Stripe Atlas, Linear features.

## Suksessmetrikk

- **Scroll-completion-rate:** % som scroller hele tour-en (mål: ≥ 50%)
- **Stage-engagement:** tid per stage (mål: 4-8 sekunder hver)
- **Drop-off-stage:** identifiser hvor folk slutter å scrolle
- **Conversion post-tour:** CTA-klikk etter tour vs uten tour (A/B-test første måned)

## Tekniske highlights

- **GSAP ScrollTrigger** (krever lisens for kommersielt — kjøp eller bruk Framer Motion `useScroll`)
- **Pin-elements** for sticky visualization
- **Intersection Observer** for trigger-points (lazy load av stages)
- **`prefers-reduced-motion`** fallback obligatorisk
