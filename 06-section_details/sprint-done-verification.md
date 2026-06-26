# Section Detail — Sprint done-verification (per-side iterasjon til 10/10)

> Hver sprint er IKKE ferdig før AI fysisk har åpnet hver berørte public-side i Playwright, scoret den mot 10/10-rubrikken, og ITERERT på samme sprint til alle sider scorer ≥9.0. «Ferdig» basert på «koden ser riktig ut» er forbudt. Vi har vært i Sprint 10 med tomme sider, manglende logo, blokket CSP, og en theme-toggle som aldri fungerte — fordi ingen åpnet siden.

---

## Hvorfor dette dokumentet finnes

Container Consult AS (mai 2026), observert i Sprint 10:
- Hero-tekst som tar 600px høyde uten bilde, kontekst, eller bento-layout — så ut som en tom Notion-side
- Logo manglet → kun en gul prikk
- Favicon manglet → default Vercel-favicon
- Meny inneholdt 6 tjenester men ikke «Nyheter», «Om oss», «FAQ» som finnes som scrapede sider
- Theme-toggle var et dekorativt ikon — ingen state, ingen DOM-endring
- CSP blokkerte Vercel Analytics + Plausible + Google Maps → silent fail i prod
- TipTap-warn: duplicate `link`-extension
- 100 scrapede bilder lå i `media`-tabellen men var ikke koblet til noen `page_sections`

Alt dette ville Playwright + en 60-sekunds visuell sjekk fanget i Sprint 2. **Det er det dette dokumentet låser inn.**

---

## Hovedregel: «Per-side iterasjon til 10/10»

Etter hver sprint som påvirker en public-side, MÅ AI:

1. Starte dev-serveren (kan kreve at brukeren godkjenner — alternativt kjøre mot deployed preview)
2. Åpne **hver** berørte public-side i Playwright
3. Kjøre `scripts/check-public-pages.ts` som scorer mot 10/10-rubrikken
4. Hvis en eller flere sider scorer <9.0: **gå tilbake til sprinten og fiks**. Ikke gå videre.
5. Iterere til alle berørte sider scorer ≥9.0
6. Lagre rapport i `docs/sprint-reports/sprint-<N>-quality.md`

«Det ser bra ut i koden» er aldri godt nok. Det må åpnes i nettleser. Det er det å være ferdig.

---

## 10/10-rubrikken (54 sjekkpunkter per side + S1-S5 per kombinasjon-side)

Hver sjekk = 1 poeng. Score per side = `(passerte / totale sjekker) × 10`. J5 + J6 + K1-K4 + L1-L4 + W1-W4 er prosjekt-globale (samme verdi på alle sider). **S1-S5 er en EGEN rubrikk (0-5)** som kjøres på hver `/[tjeneste]/[sted]`-kombinasjon-side via `check-seo-grid.ts` — ikke en del av 10/10-totalen, men en HARD blokker for sprint som leverer SEO-grid.

### A) Strukturell (4 poeng)
- [ ] Side laster (HTTP 200, ingen 5xx redirect)
- [ ] `<title>` ikke tom, ikke «Untitled», unik per side
- [ ] `<meta name="description">` finnes, 70-160 tegn
- [ ] H1 finnes, ikke duplikat med andre sider

### B) Visuelt (4 poeng) — screenshot diff
- [ ] Layout fyller viewport (ikke tomme svarte hull >300px)
- [ ] Hero-seksjon har enten bilde, gradient, illustration, eller bento-grid (ikke bare tekst på flat farge)
- [ ] Bilder vises (ingen broken-image-ikoner, ingen 404 på `<img src>`)
- [ ] Type-hierarki tydelig — H1 ≥2.5× kroppstekst, kontrast OK

### C) Identitet (4 poeng)
- [ ] Logo vises i header (img med width/height eller SVG, ikke ren tekst)
- [ ] Favicon laster (HEAD `/favicon.ico` eller `/favicon.svg` = 200)
- [ ] OG-image finnes (`<meta property="og:image">`), URL returnerer 200, bildet er ≥1200×630
- [ ] Footer-credit «Utviklet av Myrvoll-Lunde IT Drift» synlig + lenker til mlit.no

### D) Navigasjon (4 poeng)
- [ ] Hovedmeny inneholder alle sider med `is_published=true AND show_in_nav=true`
- [ ] Alle menylenker returnerer 200 (klikket gjennom)
- [ ] Mobilmeny åpner/lukker på viewport ≤768px
- [ ] Aktiv-state markert på nåværende side

### E) Interaktivitet (4 poeng)
- [ ] Alle skjemaer kan submittes (validering trigger på tom input)
- [ ] Alle CTA-knapper navigerer dit de skal
- [ ] Theme-toggle (hvis `requires_dark_mode`) faktisk endrer `data-theme` eller `class="dark"` på `<html>`
- [ ] Søkefelt/filter (hvis finnes) returnerer resultater

### F) Console (4 poeng)
- [ ] **Zero console.error** i nettleser-console etter full sidelast
- [ ] **Zero CSP-violations** i console (alle tredjeparts-skripter må være whitelisted)
- [ ] **Zero failed network requests** (alle `<img>`, `<script>`, `<link>` returnerer 2xx)
- [ ] **Zero React/Next.js warnings** (hydration mismatch, key-warning, deprecated API)

### G) Performance (4 poeng) — Lighthouse mobile
- [ ] LCP < 2.5s
- [ ] CLS < 0.1
- [ ] Performance score ≥ 90
- [ ] No render-blocking resources > 200KB

### H) Accessibility (4 poeng) — axe-core
- [ ] 0 violations (WCAG 2.2 AA)
- [ ] Tab-navigasjon fungerer (visible focus på alle interaktive)
- [ ] Alle bilder har meningsfull `alt` (ikke filnavn, ikke «image»)
- [ ] Kontrast ≥ 4.5:1 normal tekst, 3:1 stor tekst

### I) SEO (4 poeng)
- [ ] JSON-LD finnes (`Organization`/`LocalBusiness`/`Service`/`Article` per kontekst), validerer på schema.org
- [ ] Canonical-tag korrekt
- [ ] OG-tags komplett (title, description, image, type, url)
- [ ] Hreflang hvis flerspråklig

### J) Innhold + CMS-dynamikk (6 poeng — 2 prosjekt-globale)
- [ ] J1: Ingen «Lorem ipsum», «TODO», «TBD», «coming soon»
- [ ] J2: Tekst på norsk bokmål (med mindre spec sier annet)
- [ ] J3: Kontaktinfo stemmer med faktiske kundeverdier fra wizarden
- [ ] J4: Bilder fra mediabibliotek faktisk koblet til seksjoner (`page_sections.data.image_id` satt — ikke bare uploaded til Storage)
- [ ] **J5: Ingen hardkoding av CMS-innhold (§33)** — `scripts/check-no-hardcoding.sh` exit 0. Ingen `Record<slug, ...>`-mappings, ingen `slug === "/..."` conditionals, ingen hardkodet meny/footer/tjeneste-liste i kode. Se [`dynamic-content.md`](./dynamic-content.md)
- [ ] **J6: Admin mediavelger finnes** (drag-drop fra mediabibliotek, ikke URL-input). Verifiseres ved å åpne `/admin/sider/[id]` med ADMIN_AUTH_COOKIE og lete etter `[data-media-picker]` eller «Velg fra mediabibliotek»-knapp

### L) Motion-strategi (4 poeng — prosjekt-globale, basert på `motion_preference`)

Helge har valgt en motion-strategi i wizarden. Den MÅ være faktisk implementert, ikke bare nevnt i §9. Sjekkene tilpasses preferansen — se [`motion-strategy.md`](./motion-strategy.md).

- [ ] **L1: Hover-transitions** — hover på første CTA endrer `transform`/`background-color`/`color` innen 200ms (alle preferenser unntatt `Statisk`)
- [ ] **L2: Scroll-reveal** — minst én seksjon midt på siden går fra opacity <0.5 til >0.9 (eller transform endres) når brukeren scroller til den (`Lett`, `Livlig`, `Avansert`)
- [ ] **L3: Motion-bibliotek installert + brukt** — `motion`/`framer-motion` for `Livlig`, `gsap` for `Avansert`, ingen krav for `Minimal`/`Statisk`. `import` finnes i ≥3 komponenter.
- [ ] **L4: prefers-reduced-motion respekteres** — under `page.emulateMedia({ reducedMotion: 'reduce' })` er alle transitions <100ms

### V) Visuell kvalitet (5 sjekk — AI-vision, §36)

Tekniske rubrikker er blinde for «føles dette 10/10?». `check-visual-quality.ts` tar screenshots av ≥5 sider og sender til Claude Sonnet 4.6 vision-evaluator. Snitt <7/10 = sprint ikke ferdig. Full rubrikk i [`visual-quality.md`](./visual-quality.md).

- [ ] **V1: Hero visuell tyngde** — Claude scorer 7+ på dominerende visuelt anker (bilde/video/widget ≥40% av viewport)
- [ ] **V2: Layout-hierarki** — bento/grid har klar visuell ledelse, ikke alt likestort
- [ ] **V3: Direction-fidelitet** — screenshots matcher karakteristikkene for valgt `design_style`
- [ ] **V4: Side-variasjon** — minst 3 distinkte layout-mønstre på tvers av sidene
- [ ] **V5: WOW-faktor** — Claude svarer 7+ på «ville en profesjonell designer publisert dette?»

Container Consult-feilen (mai 2026): 10/10 i alle tekniske rubrikker, men «ser ut som begrenset modulsystem» i Helges vurdering. V1-V5 fanger nettopp det.

### S) SEO-grid (5 sjekk PER kombinasjon-side — egen rubrikk, §18)

Hvis prosjektet har sted×tjeneste-grid (lokal virksomhet): `scripts/check-seo-grid.ts` scorer hver `/[tjeneste]/[sted]`-side mot 5 obligatoriske krav. Snitt <4.5 = sprint som leverte grid er ikke ferdig. Full spec i [`seo-discovery.md`](./seo-discovery.md).

- [ ] **S1: ≥800 ord faktisk innhold** i `<main>` (ikke bare CTA + brødsmuler)
- [ ] **S2: JSON-LD Service + LocalBusiness** validerer på schema.org
- [ ] **S3: Unikt innhold** (Jaccard <60% overlapp mot andre kombinasjon-sider)
- [ ] **S4: ≥3 FAQ-blokker** med direkte-siterbare svar (AEO for LLM-søk)
- [ ] **S5: ≥3 internal cross-links** til andre `/[tjeneste]/[sted]`-URL-er

Container Consult-feilen (mai 2026): `/avfallscontainer/alesund` var en tom stub med brødsmuler + H1 + CTA + kalkulator. Ingen 800 ord, ingen JSON-LD, ingen FAQ. Hele formålet med 72-sider-grid kastet bort.

### W) Wizard-coverage (4 poeng — prosjekt-globale, §35)

Hvert satt wizard-felt MÅ ha sin leveranse i koden. Se [`wizard-coverage.md`](./wizard-coverage.md) for full kravsmatrise.

- [ ] **W1: Tech-stack-coverage** — hvert element i `frontend_stack` + `backend_stack` + `hosting_infra` har sin pakke i `package.json` + minst én `import` i `src/`
- [ ] **W2: Integrasjon-coverage** — hver `integrations[]`-verdi har pakke installert + ENV-vars konfigurert + minst én faktisk bruk (ikke død import)
- [ ] **W3: Krav-coverage** — hvert `requires_*=true`-flagg har sin verifisering grønn (sitemap for SEO, Cookiebot for GDPR, audit_log-tabell for audit, etc.)
- [ ] **W4: Design-coverage** — `color_primary`/`color_secondary` i tokens.css + brukt på CTA-er, `font_preference` lastet via `next/font`, `design_style` synlig (anti-template-grep grønn — overlapper med K4)

### K) Signature moment + anti-template (4 poeng — prosjekt-globale, §34)

§5 i PROJECT.md definerer signature moment som forretningsverdien. Hvis det er gjemt på en underside i stedet for å være visuelt dominant der specen sier — er ingenting annet 10/10 verdt. Disse 4 sjekkene er HARDE blokkere.

- [ ] **K1: Signature moment-komponenten eksisterer** — minst én komponent er tagget med `data-signature-moment="<name>"` i kodebasen
- [ ] **K2: Signature moment vises på primær plassering over fold** — Playwright åpner `/` (eller hva §5.2 sier primær er), tar screenshot av første viewport (1280×800), `[data-signature-moment]` MÅ være synlig i den
- [ ] **K3: Signature moment på alle sekundær-plasseringer** — Playwright sjekker hver URL listet i §5.2 sekundær (typisk hver tjenesteside) for `[data-signature-moment]`
- [ ] **K4: Anti-template-grep grønn** — `bash scripts/check-anti-template.sh` exit 0. Fanger 6+ identiske bento-kort uten hierarki, sentrert hero uten visuelt anker, manglende `data-signature-moment` på forsiden, default Tailwind/shadcn-utseende uten brand-tilpasning

---

## Scoring

| Score | Tolkning | Handling |
|---|---|---|
| 9.0–10.0 | 10/10-nivå | Sprint klar |
| 7.5–8.9 | God, men flere mangler | Iterer på sprinten, fiks topp 5 punkter |
| 5.0–7.4 | Stor jobb gjenstår | Iterer minst 3 ganger til |
| <5.0 | Ikke nær ferdig | Sprint er ikke startet på alvor — fundamental re-arbeid |

**AI skal aldri rapportere «sprint ferdig» med score <9.0. Det er en kontrakt-feil og en kvalitetstabbe.**

---

## Verktøy

### `scripts/check-public-pages.ts` (kjøres etter hver sprint)

```ts
import { chromium, type Page } from 'playwright'
import { createClient } from '@supabase/supabase-js'
import { writeFile } from 'node:fs/promises'

const supabase = createClient(
  (process.env.SUPABASE_URL || process.env.NEXT_PUBLIC_SUPABASE_URL)!.trim(),
  process.env.SUPABASE_SERVICE_ROLE_KEY!.trim(),
  { auth: { persistSession: false } },
)

const BASE_URL = process.env.TARGET_URL?.trim() || 'http://localhost:3000'

interface PageScore {
  slug: string
  url: string
  score: number  // 0-10
  passed: string[]
  failed: string[]
}

async function scoreOnePage(page: Page, slug: string): Promise<PageScore> {
  const url = `${BASE_URL}${slug}`
  const errors: string[] = []
  const cspViolations: string[] = []
  const failedReqs: string[] = []
  const consoleWarnings: string[] = []

  page.on('console', (msg) => {
    if (msg.type() === 'error') errors.push(msg.text())
    if (msg.type() === 'warning') consoleWarnings.push(msg.text())
  })
  page.on('pageerror', (err) => errors.push(err.message))
  page.on('requestfailed', (req) => failedReqs.push(`${req.url()}: ${req.failure()?.errorText}`))
  page.on('response', (res) => {
    if (res.status() >= 400) failedReqs.push(`${res.url()} → ${res.status()}`)
  })

  const resp = await page.goto(url, { waitUntil: 'networkidle', timeout: 30000 })
  await page.waitForTimeout(800)

  const passed: string[] = []
  const failed: string[] = []
  const check = (label: string, cond: boolean) =>
    cond ? passed.push(label) : failed.push(label)

  // A) Strukturell
  check('A1 HTTP 200', resp?.ok() ?? false)
  const title = await page.title()
  check('A2 Title finnes', title.length > 3 && !/untitled/i.test(title))
  const metaDesc = await page.locator('meta[name="description"]').getAttribute('content')
  check('A3 Meta description', (metaDesc?.length ?? 0) >= 70 && (metaDesc?.length ?? 0) <= 160)
  check('A4 H1 finnes', (await page.locator('h1').count()) === 1)

  // C) Identitet
  const logoSelectors = ['header img[alt*="logo" i]', 'header svg', 'header [role="banner"] img', '[data-logo]']
  let hasLogo = false
  for (const sel of logoSelectors) {
    if ((await page.locator(sel).count()) > 0) { hasLogo = true; break }
  }
  check('C1 Logo i header', hasLogo)

  const faviconResp = await page.request.head(`${BASE_URL}/favicon.ico`).catch(() => null)
    ?? await page.request.head(`${BASE_URL}/favicon.svg`).catch(() => null)
  check('C2 Favicon 200', faviconResp?.ok() ?? false)

  const ogImage = await page.locator('meta[property="og:image"]').getAttribute('content')
  let ogOk = false
  if (ogImage) {
    const r = await page.request.head(ogImage).catch(() => null)
    ogOk = r?.ok() ?? false
  }
  check('C3 OG image', ogOk)
  check('C4 Footer credit', await page.getByText(/Myrvoll-Lunde IT Drift/i).isVisible().catch(() => false))

  // F) Console
  check('F1 Zero console.error', errors.length === 0)
  check('F2 Zero CSP-violations',
    !errors.some((e) => /Content Security Policy/i.test(e)) &&
    !cspViolations.length)
  check('F3 Zero failed network', failedReqs.length === 0)
  check('F4 Zero React warnings',
    !consoleWarnings.some((w) => /hydrat|key.*prop|deprecated/i.test(w)))

  // B) Visuelt — sjekker bilder
  const imgs = await page.locator('img').all()
  let brokenImgs = 0
  for (const img of imgs) {
    const naturalWidth = await img.evaluate((el: HTMLImageElement) => el.naturalWidth)
    if (naturalWidth === 0) brokenImgs++
  }
  check('B3 Bilder vises (ingen broken)', brokenImgs === 0)

  // ... legg til resten av sjekkene per behov

  const score = (passed.length / (passed.length + failed.length)) * 10
  return { slug, url, score, passed, failed }
}

async function main() {
  const { data: pages } = await supabase
    .from('pages')
    .select('slug, title')
    .eq('is_published', true)
    .order('slug')

  if (!pages?.length) {
    console.error('Ingen published pages i DB')
    process.exit(1)
  }

  const browser = await chromium.launch()
  const context = await browser.newContext({ viewport: { width: 1280, height: 800 } })
  const results: PageScore[] = []

  for (const p of pages) {
    const page = await context.newPage()
    console.log(`Sjekker ${p.slug} …`)
    const r = await scoreOnePage(page, p.slug)
    results.push(r)
    console.log(`  → ${r.score.toFixed(1)}/10  (${r.failed.length} mangler)`)
    await page.close()
  }

  await browser.close()

  const avg = results.reduce((s, r) => s + r.score, 0) / results.length
  const failing = results.filter((r) => r.score < 9.0)

  const report = [
    `# Sprint Quality Report`,
    `Generert: ${new Date().toISOString()}`,
    ``,
    `**Snitt:** ${avg.toFixed(2)}/10`,
    `**Sider:** ${results.length}`,
    `**Under 9.0:** ${failing.length}`,
    ``,
    `## Per-side`,
    ``,
    ...results.map((r) => [
      `### ${r.slug} — ${r.score.toFixed(1)}/10`,
      ``,
      r.failed.length ? `**Mangler:**\n${r.failed.map((f) => `- ❌ ${f}`).join('\n')}` : '✅ Alle sjekker bestått',
      ``,
    ].join('\n')),
  ].join('\n')

  await writeFile(`docs/sprint-reports/sprint-quality-${Date.now()}.md`, report)

  if (failing.length > 0) {
    console.error(`\n❌ ${failing.length} sider scorer under 9.0 — sprint er IKKE ferdig`)
    failing.forEach((r) => console.error(`   ${r.slug}: ${r.score.toFixed(1)} (${r.failed.length} mangler)`))
    process.exit(1)
  }

  console.log(`\n✅ Alle ${results.length} sider scorer ≥9.0 — sprint klar`)
}

main().catch((e) => { console.error(e); process.exit(1) })
```

### Kjør den hver sprint

```bash
# Mot localhost (lokal verifikasjon)
TARGET_URL=http://localhost:3000 pnpm tsx scripts/check-public-pages.ts

# Mot deployed Vercel preview (sprint done-check)
TARGET_URL=https://<repo>-git-dev-<team>.vercel.app pnpm tsx scripts/check-public-pages.ts
```

Exit-code != 0 hvis noen side scorer <9.0. Loop på sprinten til den passerer.

---

## Auto-evaluator-workflow (PR-blokker)

```yaml
# .github/workflows/quality-gate.yml
name: Quality Gate
on:
  pull_request:
    branches: [main]
jobs:
  per-page-score:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - run: pnpm install --frozen-lockfile
      - run: pnpm playwright install --with-deps chromium
      - name: Score every public page
        run: pnpm tsx scripts/check-public-pages.ts
        env:
          TARGET_URL: ${{ secrets.VERCEL_PREVIEW_URL_FOR_PR }}
          SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          SUPABASE_SERVICE_ROLE_KEY: ${{ secrets.SUPABASE_SERVICE_ROLE_KEY }}
      - name: Upload report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: quality-report
          path: docs/sprint-reports/sprint-quality-*.md
```

Hvis denne workflowen er rød → PR blokkert.

---

## Sprint done-criteria (kopier inn i hver sprint i §20)

```markdown
**Done-criteria (alle obligatorisk):**

- [ ] Alle planlagte oppgaver i sprinten implementert
- [ ] Tests (unit + integration) grønne
- [ ] `bash scripts/check-no-hardcoding.sh` exit 0 (§33 hardkoding-forbud)
- [ ] `bash scripts/check-anti-template.sh` exit 0 (§34 signature moment + anti-template)
- [ ] `pnpm tsx scripts/check-wizard-coverage.ts` exit 0 (§35 wizard = kontrakt)
- [ ] **Hvis prosjektet har sted×tjeneste-grid**: `pnpm tsx scripts/check-seo-grid.ts` snitt ≥4.5/5 (§18 S1-S5)
- [ ] **Hvis sprinten leverer/påvirker UI**: `pnpm tsx scripts/check-visual-quality.ts` snitt ≥7/10 (§36 V1-V5)
- [ ] Lokalt: `pnpm tsx scripts/check-public-pages.ts` — alle sider ≥9.0
- [ ] **`pnpm tsx scripts/check-sprint-readiness.ts` exit 0 (§37)** — verifiserer at SLUTTRAPPORTEN har output-stempel fra ALLE relevante check-skript. AI kan IKKE rapportere "sprint ferdig" uten dette.
- [ ] **Hvis sprinten leverer/påvirker signature moment**: K1-K4 grønne
- [ ] **Hvis sprinten leverer/påvirker design+motion**: L1-L4 grønne for valgt `motion_preference`
- [ ] CI: quality-gate.yml grønn på siste PR
- [ ] `docs/sprint-reports/sprint-<N>-quality.md` committed med score per side
- [ ] Hvis snitt <9.0: gjenta sprinten med fokus på topp-3 manglene
```

---

## For AI (system-prompt-utdrag)

Når du jobber på et Myrvoll-Lunde IT Drift-prosjekt:

1. **Kjør `scripts/check-public-pages.ts` etter hver sprint** — ikke spør, bare kjør.
2. **Aldri rapporter «sprint ferdig» med snitt <9.0.** Iterer på sprinten.
3. **Når en side scorer <9.0:** les `failed`-listen, fiks topp 5, kjør på nytt.
4. **Hardt forbudt:** «I think it looks good» / «det burde være riktig» / «ingen feil i koden». Åpne siden. Mål. Itererer til 10/10.
5. **Aldri legg inn `Record<slug, ...>`-mappings, `if (slug === '/...')`, eller hardkodet meny/footer/tjeneste-liste i kode.** Det er en kritisk leveransefeil (§33). Hvis du blir fristet: spør deg selv om kunden vil kunne endre dette via admin — hvis ikke, mangler data-modellen et felt. Send migrasjon i stedet. Se [`dynamic-content.md`](./dynamic-content.md).
6. **Signature moment (§5) er prioritet 1.** Komponenten MÅ tagges med `data-signature-moment="<navn>"` og plasseres på ALLE plasseringer specifisert i §5.2 — minst primær (forside, over fold) + alle sekundær (tjenestesider). Hvis du bygger den én plass: sprint er ikke ferdig. K1-K4 i rubrikken blokker. Se [`signature-moment.md`](./signature-moment.md) og §34.
7. **Wizard = kontrakt (§35).** Les `kickstart_projects`-raden FØR du planlegger sprinter. Hvert satt felt = oppgave. Hvis du tolker noe som «forslag» eller dropper det stille — du har brutt kontrakten. Bygg det, eller flagg eksplisitt i «Spørsmål til kunden». W1-W4 i rubrikken + `check-wizard-coverage.ts` blokker. Se [`wizard-coverage.md`](./wizard-coverage.md) og [`motion-strategy.md`](./motion-strategy.md).
8. **Visuell kvalitet er ikke en tekniske sjekk (§36).** 10/10 i DOM-rubrikken betyr ingenting hvis siden ser ut som «begrenset modulsystem». `check-visual-quality.ts` bruker Claude vision til å score hero-tyngde, layout-hierarki, direction-fidelitet, side-variasjon og WOW-faktor. Snitt <7 = sprint ikke ferdig. Les Claude's `top_issues` og fiks dem. Se [`visual-quality.md`](./visual-quality.md).
