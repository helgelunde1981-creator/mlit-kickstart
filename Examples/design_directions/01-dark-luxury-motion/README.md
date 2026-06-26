# Dark Luxury Motion

> Warm darks, gold/amber/orange accents, glassmorphism med ekte dybde, scroll-baserte motion-reveals, store typografi-moments.

## Use når

- Premium tjenester (juridisk, finans, eiendom)
- Transport og logistikk med premium-posisjonering
- High-end SaaS
- Konsulent-firma med "trusted advisor"-rolle
- Eiendomsmegling
- Restauranter / hotell

## Ikke bruk når

- Lekent, barnevennlig
- Lavpris-volum (DM, butikk-kjede)
- Sterk lys-tema-brand allerede etablert
- Offentlig sektor (for "luksuriøst")

## Signaturmoves

- **Orange-glow radial gradients** i hero-bakgrunn — `radial-gradient(circle at 30% 20%, rgba(255,69,0,0.15), transparent 50%)`
- **Glass-cards med backdrop-blur** — ekte dybde, ikke flat dekorasjon
- **Monospace "live"-indikatorer** med pulserende grønn prikk
- **Scroll-triggered fade-up med stagger** — Framer Motion `whileInView` med 100ms stagger
- **Tall som "telles opp"** ved første viewport-entry
- **Hover på cards: lift + shadow-expand** — translate-y-[-4px] + shadow-lg
- **Geist + Geist Mono** som hovedfont-par

## Inspirasjon (URLs)

Se `references.md` for navngitte URL-er.

## Filer i denne mappen

- `tokens.css` — komplett CSS custom properties
- `tailwind.config.snippet.ts` — Tailwind v4-config
- `font-pairings.md` — anbefalte font-pairings
- `motion.example.ts` — Framer Motion-vokabular
- `hero.example.tsx` — ferdig 10/10 hero-komponent
- `card.example.tsx` — ferdig glass-card med states
- `nav.example.tsx` — ferdig navigation med dark luxury vibe
- `references.md` — navngitte URL-eksempler
- `screenshots/` — egne screenshots (legges til etterhvert)
