# Anti-Patterns — Hva 10/10 IKKE er

> Pek-og-pek-på pedagogikk. Beskrivelser av designs som ofte havner i AI-output men som vi avviser. AI MÅ sjekke output mot disse før levering.

## Anti-patterns vi alltid avviser

### 1. Generic SaaS Hero

**Symptom:** Sentrert headline + subtitle + gradient blob bak + to standard knapper ("Get started" + "Learn more"). Bilde av en MacBook med dashboard.

**Hvorfor 8/10, ikke 10/10:** Du har sett denne 1000 ganger. Den signaliserer "vi har ingen distinkt brand-personlighet". Gradient blob er et 2022-cliché.

**Fix:** Velg én av de 9 designretningene. Bygg hero med retningens signaturmoves. Ingen gradient blob.

---

### 2. Default shadcn Card Grid

**Symptom:** 3-kolonne grid med kort som har lik padding, samme border-radius, samme shadow, samme hover (opacity-shift). Bilde-ikon på toppen.

**Hvorfor 8/10:** shadcn out-of-the-box er en god start, ikke et ferdig design. Uten customisering (variants, tokens, motion) signaliserer det "vi installerte et bibliotek og ble ferdige".

**Fix:**
- Customiser med design-direction tokens
- Bryt grid med minst én "feature-cell" som er 2x1
- Hover-state med karakter (lift + shadow + border-glow)
- Variants for ulike kort-typer

---

### 3. Gray-on-White Safe Styling

**Symptom:** Hvit bakgrunn, grå tekst, én accent-farge brukt sparsomt. Ingen tekstur, ingen depth, ingen motion. "Profesjonelt" men dødt.

**Hvorfor 8/10:** Trygg ≠ god. Trygg signaliserer "ikke noe spennende skjer her". For 95% av prosjektene er dette underdelivery.

**Fix:**
- Velg én av de 9 retningene som har mer karakter
- Hvis "trygg" virkelig er kravet: gå for Swiss Minimal Refined eller Industrial Robust, ikke generic gray
- Bruk farge semantisk (suksess/feil/info)
- Tekstur eller subtle motion for liv

---

### 4. Dashboard-by-Numbers

**Symptom:** Venstre sidebar + topp-header + 4 stat-cards + en chart + en tabell. Default shadcn-dashboard-template.

**Hvorfor 8/10:** Det er funksjonelt, men det er ingen "wow"-moment. Brukeren får tilgang til data, men det føles ikke produkt-tilpasset.

**Fix:**
- Brytt layout — bento, ikke 12-col-grid
- En "primary widget" som er signature moment
- Mikro-interaksjoner (hover på chart, drill-down)
- Egne loading states (skeleton som matcher faktisk layout)

---

### 5. Stock Unsplash Hero Image

**Symptom:** Bilde av et tilfeldig glade arbeidere, et tomt kontor, en abstract pattern. Tatt fra Unsplash. Ingen relevans til kunden.

**Hvorfor 8/10:** Det fyller plass uten å si noe. Verre: brukeren har sett samme bilde på 20 andre sites.

**Fix:**
- Egne foto/illustrasjoner relevant til produktet
- Eller: ingen bilde, men sterk typografi-moment
- Eller: signature moment (kalkulator, live data) i stedet for bilde

---

### 6. "Velkommen til [firma]" h1

**Symptom:** H1-en er bare firmanavnet eller "Velkommen til X". Ingen verdi-løfte.

**Hvorfor 8/10:** Du har 100ms til å fortelle hva produktet gjør. "Velkommen" sløser den tiden.

**Fix:**
- H1 beskriver hva kunden får (ikke hva firmaet gjør)
- Konkret påstand eller spørsmål
- Maks 8 ord

Eks: "Frakt på Romerike levert på 60 minutter" >> "Velkommen til Skedsmo Bud & Vare"

---

### 7. Hover = opacity-skift

**Symptom:** Alle interaktive elementer reagerer på hover med `opacity-80`. Ingen motion, ingen feedback.

**Hvorfor 8/10:** Det signaliserer "klikkbar" på laveste nivå, men gir ingen design-kvalitet.

**Fix:**
- Hover med karakter: lift + shadow, scale + glow, border-shift
- Mikro-motion (translate-x-0.5 på arrow-ikon)
- Active-state også designet (scale-95)
- Focus-ring som ikke er default browser-blå

---

### 8. Alle knapper ser like ut

**Symptom:** Én Button-variant brukt overalt. Ingen visuell hierarki mellom primær/sekundær/destruktiv.

**Hvorfor 8/10:** Brukeren skjønner ikke hva som er hovedhandlingen.

**Fix:**
- Minst 3 button-variants (primary, secondary, ghost)
- Variant for destructive (delete) tydelig
- Konsekvent bruk: kun én primær per side-skjerm

---

### 9. Lorem ipsum eller placeholder-text i lev

**Symptom:** PROJECT.md eller wireframes har "Lorem ipsum dolor sit amet" eller "[Description goes here]".

**Hvorfor 8/10:** Det er ikke ferdig. Vi leverer aldri uferdig.

**Fix:**
- Skriv kunde-spesifikk tekst basert på `core_product` + `vision`
- Hvis tekst ikke er kjent: skriv plassholderne som «Kommer fra kunde — se §X», ikke lorem
- AI skal flagge dette eksplisitt, ikke skjule

---

### 10. Egen cookie-banner i stedet for Cookiebot

**Symptom:** AI bygger en egen `CookieBanner`-komponent + `cookie_consents`-tabell.

**Hvorfor kritisk:** Vi bygger ALDRI egen consent-modul. Egen banner gir compliance- og vedlikeholdsrisiko (IAB TCF, "Avvis alle"-likestilling, cookie-deklarasjon, samtykke-logg) som Cookiebot allerede løser. Egen banner = kontraktbrudd (regel 40).

**Fix:**
- Bruk **Cookiebot** — alltid. Spør Helge om CBID/kode (kan ikke gjettes).
- Script i `<head>` med `data-cbid` + `data-blockingmode="auto"`.
- `/cookies` bruker Cookiebots `CookieDeclaration`-script.
- Cookiebot-domener i CSP (script-src + connect-src).
- Footer-lenke åpner Cookiebot-dialogen på nytt (`Cookiebot.renew()`).

---

## Hvordan AI bruker denne lista

Som del av self-check (§00-SYSTEM_PROMPT.md), verifiser at PROJECT.md ikke har noen av disse anti-patterns implisitt eller eksplisitt foreslått.

Hvis du oppdager at du var i ferd med å foreslå en av disse, regenerer den seksjonen.
