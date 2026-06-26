# Signature Moment — Ambient Data Visualization

> Eksempel-mønster: hero-område viser live data fra produktet — uptime, transactions, requests, users — som beviser at "vi er i drift akkurat nå".

## Hva

Hero-seksjonen har ikke statisk copy. Den viser bevegelige tall: "1.247 leveringer i dag", "98.7% on-time", "Siste levering: 3 minutter siden". Tall oppdateres live (websocket eller polling). Krydder med subtile animasjoner som signaliserer "live".

## Hvor

- **Forside-hero:** dominerer over the fold
- **Status-page:** mer detaljert versjon

## Oppførsel

### States

- **Initial:** placeholder med skeleton (subtle pulse), ingen tall ennå
- **Live:** tall vises, oppdaterer hver 10-30 sek
- **Disconnected:** websocket droppet — viser cached tall + advarsel "Forsinket data"
- **Error:** API down — viser fallback static data + "Status-page →"

### Transitions

- Tall-endring: count-up med ease-out-circ
- Status-indikator (grønn prikk): pulserer for "live"
- Nye items: slide-in fra topp med fade

### Edge cases

- **Reduced-motion:** ingen pulse, ingen count-up — tall vises plain
- **Lav-bandwidth:** drop til polling-mode i stedet for websocket
- **Server overload:** rate-limit data-pull med exponential backoff

## Hvorfor signature

Det viser at produktet er levende. Vanlig "trust signal" som "100% uptime" føles tørt — live tall som beveger seg foran øynene dine er fundamentalt mer overbevisende. Egnet for SaaS, marketplace, transport, fintech der "vi opererer på skala" er kjernemeldingen.

Eksempel: Stripe sin "$X processed today", Vercel sine deploy-counters, Linear sin issue-counter.

## Suksessmetrikk

- **Time-to-first-CTA-click:** kortere = signaliserer at trust ble etablert raskt
- **Scroll-rate:** scroller folk forbi hero raskere når data er live? (engagement-test)
- **Mobile engagement:** funker live data på mobil uten å spise batteri?

## Tekniske highlights

- **Server-sent Events** eller Websocket for live data
- **`useDeferredValue`** for å unngå hyper-frequent re-renders
- **Cache-busting** med stale-while-revalidate
- **Server-side first render** av "snapshot" tall — klient tar over for live
- **Fallback til polling** hvis WS feiler
- **Background tab throttling** — pause oppdateringer når sida ikke er synlig
