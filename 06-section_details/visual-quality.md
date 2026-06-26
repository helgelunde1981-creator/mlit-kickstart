# Section Detail — Visuell kvalitet (V1-V5, AI-vision)

> Container Consult AS (mai 2026): score 9.8/10 i deterministisk audit + 10/10 i `check-public-pages.ts` + 5/5 i SEO-grid. Helge: «sidene ser ut som begrenset modulsystem». Vi målte tekniske ting — IKKE visuell kvalitet. Hele rubrikken vår var blind for den faktiske brukeropplevelsen. **Aldri igjen.**

---

## Hovedregel

Tekniske sjekker (DOM-struktur, JSON-LD, ord-telling, dependencies) er **nødvendige men ikke tilstrekkelige**. Et 10/10-prosjekt MÅ også passere visuell kvalitet-rubrikken — vurdert av **Claude vision** mot screenshot, ikke av en grep eller en pixel-teller.

V1-V5 er prosjekt-globale sjekker som blokker sprint hvis snitt <7/10.

---

## V1-V5 rubrikken

### V1: Hero visuell tyngde
Hero (første viewport, 1280×800) MÅ ha et dominerende visuelt anker: bilde, video, signature widget eller illustrasjon som tar **≥40% av viewport-arealet**. Ren tekst + CTA = fail. Claude scorer 0-10 basert på screenshot.

### V2: Layout-hierarki
Ingen seksjon kan være alle elementer like-store. Bento må ha minst ett element 2× større. Grid-seksjoner må ha varierende dybde/farge/typografi. Claude vurderer screenshot mot «alt likt» (fail) vs «klar visuell ledelse» (pass).

### V3: Direction-fidelitet
`design_style` fra wizarden er en kontrakt om hva sidene skal LIKNE. Claude sammenligner screenshot mot karakteristikkene for valgt direction:

| Direction | Kjennetegn Claude leter etter |
|---|---|
| Editorial Bento | Asymmetrisk grid, store typografiske kontraster, redaksjonell rytme, hvit-rom som strukturelement |
| Dark Luxury Motion | Mørk default-palette, glanseffekter, gull/kobber-accents, langsom motion |
| Swiss Minimal Refined | Grid-disiplin, monospace-/serif-blanding, mye hvit-rom, presisjon |
| Brutalism Refined | Tunge skyggevirkninger, kanter, asymmetri, monospace dominerer |
| Glassmorphism Depth | Frostete bakgrunner, depth via backdrop-blur, layering |
| Retro-Futurism | 70/80-tall-paletter, scanlines, neon-accents, retro typografi |
| Scrollytelling Editorial | Pin-seksjoner, scrubbed reveals, story-flow |
| 3D Integration | WebGL/Three.js, perspektiv-effekter, lys-skygge-arbeid |
| Industrial / Robust | Tunge fotografi, mørke metalliske tokens, rotert grid, byggebrans-jefølelse |

Hvis screenshot ikke ser ut som direction → fail.

### V4: Side-variasjon
Claude sammenligner screenshots av ≥5 ulike sider (forside + 3 service-sider + en sted×tjeneste). Hvis alle ser ut som samme template med byttet tekst: fail. Minst 3 distinkte layout-mønstre kreves.

### V5: WOW-faktor (designer-bedømmelse)
Claude svarer på spørsmålet: *«Ville en profesjonell designer publisert dette under sitt navn?»* med 0-10. Vurderer kombinasjonen av typografi, palette, layout, mikroanimasjoner, kontrast, hvit-rom-disiplin.

---

## Implementasjon: `scripts/check-visual-quality.ts`

```ts
import { chromium } from 'playwright'
import Anthropic from '@anthropic-ai/sdk'

const claude = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY!.trim() })

const VISUAL_RUBRIC_TOOL = {
  name: 'submit_visual_score',
  description: 'Score en side mot V1-V5 visuell kvalitet-rubrikk',
  input_schema: {
    type: 'object' as const,
    properties: {
      v1_hero_weight: {
        type: 'object',
        properties: { score: { type: 'number' }, reasoning: { type: 'string' } },
        required: ['score', 'reasoning'],
      },
      v2_layout_hierarchy: {
        type: 'object',
        properties: { score: { type: 'number' }, reasoning: { type: 'string' } },
        required: ['score', 'reasoning'],
      },
      v3_direction_fidelity: {
        type: 'object',
        properties: { score: { type: 'number' }, reasoning: { type: 'string' } },
        required: ['score', 'reasoning'],
      },
      v5_wow_factor: {
        type: 'object',
        properties: { score: { type: 'number' }, reasoning: { type: 'string' } },
        required: ['score', 'reasoning'],
      },
      top_issues: {
        type: 'array',
        items: { type: 'string' },
        description: 'Topp 3 konkrete ting som bør fikses',
      },
    },
    required: ['v1_hero_weight', 'v2_layout_hierarchy', 'v3_direction_fidelity', 'v5_wow_factor', 'top_issues'],
  },
}

async function scoreOne(page: Page, url: string, direction: string) {
  await page.goto(url, { waitUntil: 'networkidle' })
  await page.waitForTimeout(1000)
  const screenshot = await page.screenshot({ fullPage: false }) // første viewport

  const result = await claude.messages.create({
    model: 'claude-sonnet-4-6',
    max_tokens: 2000,
    tools: [VISUAL_RUBRIC_TOOL],
    tool_choice: { type: 'tool', name: VISUAL_RUBRIC_TOOL.name },
    messages: [{
      role: 'user',
      content: [
        { type: 'image', source: { type: 'base64', media_type: 'image/png', data: screenshot.toString('base64') } },
        { type: 'text', text: `Vurder denne forsiden mot design-direction "${direction}".

V1: Hero visuell tyngde — har hero et dominerende visuelt anker (bilde/video/widget) som tar ≥40% av viewport? Ren tekst + CTA = 0-3. Stort bilde med karakter = 7-10.

V2: Layout-hierarki — er ALT like-stort (fail) eller er det klar visuell ledelse (pass)? Bento må ha minst én tung-vekt-komponent.

V3: Direction-fidelitet — ser dette ut som ${direction}? Score basert på palette, typografi, rytme, motion-cues, layout-arketype.

V5: WOW-faktor — ville en designer publisert dette? Vurder helhet (typografi+palette+layout+mikroanimasjoner+hvit-rom).

Inkluder 3 konkrete ting som bør fikses i top_issues.

Returner via tool-call.` },
      ],
    }],
  })

  const tool = result.content.find((b) => b.type === 'tool_use')
  if (!tool || tool.type !== 'tool_use') throw new Error('Ingen vision-score')
  return tool.input as {
    v1_hero_weight: { score: number; reasoning: string }
    v2_layout_hierarchy: { score: number; reasoning: string }
    v3_direction_fidelity: { score: number; reasoning: string }
    v5_wow_factor: { score: number; reasoning: string }
    top_issues: string[]
  }
}

// V4 (side-variasjon) krever sammenligning av flere screenshots — egen funksjon:
async function scoreVariation(screenshots: Buffer[]): Promise<{ score: number; reasoning: string }> {
  const result = await claude.messages.create({
    model: 'claude-sonnet-4-6',
    max_tokens: 1000,
    tools: [{ name: 'submit_variation', input_schema: { type: 'object', properties: { score: { type: 'number' }, reasoning: { type: 'string' } }, required: ['score', 'reasoning'] } }],
    tool_choice: { type: 'tool', name: 'submit_variation' },
    messages: [{
      role: 'user',
      content: [
        ...screenshots.map((s) => ({ type: 'image' as const, source: { type: 'base64' as const, media_type: 'image/png' as const, data: s.toString('base64') } })),
        { type: 'text', text: 'Disse ${screenshots.length} screenshots er fra ulike sider i samme prosjekt. Score 0-10: ser alle sider ut som samme template med byttet tekst (fail, 0-3) eller er det minst 3 distinkte layout-mønstre (pass, 7-10)? Returner via tool-call.' },
      ],
    }],
  })
  const tool = result.content.find((b) => b.type === 'tool_use')
  return (tool as { input: { score: number; reasoning: string } }).input
}

async function main() {
  // ... iterer pages, kjør scoreOne, samle screenshots til V4, lagre rapport
  // Snitt-score (v1+v2+v3+v4+v5)/5 må være ≥7 for grønt
}
```

Komplett kjørbar mal i `EXAMPLES/scripts/check-visual-quality.ts.example`.

---

## Vercel deployment protection (kritisk når TARGET_URL er preview)

**Container Consult 30. mai 2026:** vision-test mot dev-preview-URL returnerte snitt 1.25/10 — men Claude scoret faktisk **Vercel login-skjermen**, ikke prosjektet. Deployment protection er på som default på preview-deploys.

**Symptom:** alle sider scorer 1/V1, 2/V2, 1/V3, 1/V5 + Claude snakker om «Continue with Email», «GitHub OAuth», «Log in to Vercel».

**Fix (allerede i mal-skriptet):**

1. `check-visual-quality.ts` har en GUARD som kaster feil hvis HTML inneholder Vercel-login-strings:
   ```ts
   if (/Log in to Vercel|Continue with Email/i.test(html)) {
     throw new Error(`Vercel deployment protection blokkerer ...`);
   }
   ```

2. **Tre løsninger** for å gjøre vision-testen kjørbar:
   - **A) Kjør mot prod-URL** — `TARGET_URL=https://<domene>.no` eller `<prosjekt>.vercel.app`. Krever at koden er på `main`.
   - **B) Slå av deployment protection** for preview i Vercel Project Settings → Deployment Protection → "Standard Protection" → Off (kun preview, behold for production)
   - **C) Bruk bypass-token:**
     ```bash
     # Vercel Project → Deployment Protection → Protection Bypass for Automation
     export VERCEL_AUTOMATION_BYPASS_SECRET="<token>"
     pnpm tsx scripts/check-visual-quality.ts
     ```
     Skriptet legger automatisk til `x-vercel-protection-bypass`-header når denne env er satt.

3. CI-workflow (`.github/workflows/quality-gate.yml`) bruker alternativ C — tokenet er stored som GitHub secret.

---

## Workflow-integrering

`.github/workflows/quality-gate.yml` får ny jobb:

```yaml
visual-quality:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: pnpm/action-setup@v3
    - run: pnpm install --frozen-lockfile
    - run: pnpm playwright install --with-deps chromium
    - run: pnpm tsx scripts/check-visual-quality.ts
      env:
        TARGET_URL: ${{ secrets.VERCEL_PREVIEW_URL }}
        ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        DESIGN_STYLE: ${{ vars.DESIGN_STYLE }}  # fra kickstart-wizard
    - uses: actions/upload-artifact@v4
      if: always()
      with:
        name: visual-quality-screenshots
        path: docs/sprint-reports/visual-*.png
```

PR mot main blokkere hvis snitt <7.

---

## Sprint-spesifikasjon (kopier inn i sprinten som leverer designsystem)

```markdown
**Visuell kvalitet-leveranse (§36 — V1-V5)**

- [ ] Bygg hero med dominerende visuelt anker (bilde/video/widget ≥40% av viewport)
- [ ] Bento/grid har visuell hierarki (minst 1 element 2× større)
- [ ] Komponenter matcher valgt design_style (palette, typografi, layout)
- [ ] Minst 3 distinkte layout-mønstre på tvers av sidene
- [ ] Mikroanimasjoner, hvit-rom-disiplin, typografisk personlighet
- [ ] `pnpm tsx scripts/check-visual-quality.ts` snitt ≥7
- [ ] Hvis snitt <7: les Claude's top_issues, fiks topp-3, kjør på nytt
```

---

## Hero-konsistens på tvers av rute-typer (kritisk)

**Container Consult 30. mai 2026:** Hero-fix traff bare `<SectionRenderer type="hero">`-ruter (forside + tjenester + faq). `/kontakt` og `/om-oss` hadde egne hero-implementasjoner med flat farge — fix-en påvirket dem ikke. `/[tjeneste]/[sted]` hadde INGEN hero — bare brødsmuler + H1. Resultat: V1/V3 sprer fra 1 til 6 på tvers av sider.

**Krav:** Hele prosjektet skal bruke ÉN delt hero-komponent (`<PageHero>`, `<HeroSection>`, eller hva prosjektet kaller den). Den henter direction-tokens fra `tokens.css` og motion-bibliotek fra `motion/react`/`gsap`. Hver rute-type konfigurerer den med props — IKKE bygger egen.

**Sjekk:** `check-anti-template.sh` teller unike hero-implementasjoner (grep `<Hero[A-Z]\\w*` + `<section className="..hero..">`). Hvis >3 unike implementasjoner: fail.

**Mal:** `EXAMPLES/templates/service-location-page.tsx.example` viser hvordan sted×tjeneste-ruter MÅ kobles til samme `<PageHero>` som resten av prosjektet.

## V5 (WOW) krever motion i hero-komponenten (ikke bare i kodebasen)

**Container Consult-funn:** Motion-pakke (`motion`) var installert + brukt i page-transitions, men HERO-komponenten hadde ingen `whileInView`/`initial`/`animate`. V5 scoret konsekvent <5 fordi heroene «føles flate».

**Krav:** Hero-komponenten(e) MÅ importere `motion/react` (eller `gsap`) og bruke det på minst hero-tittel + CTA. Sjekk i `check-anti-template.sh` sjekk 7: grep `motion/react|framer-motion|gsap` i hero-komponent-fil.

## Direction → font-mapping (anbefalt, kan overstyres)

For at V3 (direction-fidelitet) skal score ≥7 må fonten matche direction-arketypen. Container Consult-bug: brukte Inter_Tight (soft tech) på Industrial-direction → V3 = 2-4/10.

| Direction | Display-font (anbefalt) | Body-font | Mono | Hvorfor |
|---|---|---|---|---|
| Industrial / Robust | **Barlow Condensed Black**, Bebas Neue, Anton | Inter | JetBrains Mono | Kondensert tung sans = industri-merking |
| Editorial Bento | **Tiempos Headline**, GT Sectra, Migra, PP Editorial New | Söhne, Inter | — | Serif display = redaksjonell autoritet |
| Dark Luxury Motion | **Söhne Mono**, GT Sectra, Editorial New | Inter | Söhne Mono | Sjelden serif + tung sans = premium |
| Swiss Minimal Refined | **Söhne**, Helvetica Now, Inter Tight | Söhne | JetBrains Mono | Grid-disiplin krever nøytrale, presise grotesque |
| Brutalism Refined | **VT323**, JetBrains Mono, Space Grotesk | JetBrains Mono | Space Mono | Monospace dominerer |
| Glassmorphism Depth | **Inter Tight**, SF Pro Display | Inter | — | Soft + lett — frostet glass-feel |
| Retro-Futurism | **VT323**, Press Start 2P, Major Mono | Space Mono | Space Mono | Pixel/CRT-fonts = retro-future |
| Scrollytelling Editorial | **Tiempos Headline**, Migra, Söhne | Söhne | — | Story-driven serif |
| 3D Integration | **Söhne**, Inter Tight + variabel-font tricks | Inter | — | Nøytral for å la 3D dominere |

Disse er START-anbefalinger. Hvis kunden har spesifisert font i wizard (`font_preference`), bruk det først. Men FAIL hvis font kombinert med direction ikke matcher karakteristikkene.

## Hero-anti-pattern (Container Consult-fellen, mai 2026)

Den hyppigste V1-feilen vi har sett:

### ❌ Mørk overlay som gjemmer bildet
```tsx
// SCORE V1 = 4-5/10 fordi bildet underordnes teksten
<Image src={bg} className="opacity-55" />
<div className="absolute inset-0 bg-gradient-to-t from-black/80 via-black/40 to-black/30" />
```

Selv om bildet er der teknisk, blir det "bakgrunns-mood". Hero-budskapet blir teksten + CTA, ikke bildet.

### ✅ Vignett venstre + subtil bunn-fade
```tsx
// SCORE V1 = 7-9/10 — bildet ER ankeret
<Image src={bg} className="object-cover" />  {/* FULL opacity */}
{/* Vignett venstre (tekstområde leselig) */}
<div className="absolute inset-0 bg-gradient-to-r from-black/85 via-black/50 to-transparent" />
{/* Subtil bunn-fade for footer-tekst-area */}
<div className="absolute inset-x-0 bottom-0 h-32 bg-gradient-to-t from-black/70 to-transparent" />
```

Tekstområdet leselig på venstre side; bildet dominerer høyre side. Heroens visuelle budskap er bildet, teksten støtter.

## Anti-patterns

### ❌ «Begrenset modulsystem»-utseende
Alle seksjoner er like padding + samme kort-struktur + uniform farge. Sidens layout føles som om en bygger har dratt 6 av samme komponent inn på rad. Claude ser dette med en gang.

### ❌ Hero = stor tekst + CTA på flat farge
Selv om hero har et bilde i bakgrunnen, hvis bildet er underordnet teksten (mørk overlay + senterstilt tekst) → fail på V1. Heroen må FØLES som det er noe der.

### ❌ Default Tailwind-look
`shadow-md`, `rounded-lg`, `bg-slate-50`, `text-slate-900` — alt sammen = stock shadcn-template. Direction-fidelitet faller umiddelbart.

### ❌ Samme layout på alle sider
Hvis V4-screenshot-array viser fem sider med nesten identiske wireframes, bare med byttet tekst og bilder: side-variasjon fail. Hver hovedside bør ha et signaturlayout.

### ❌ Static «just enough» motion
Hover-transitions er der teknisk, men ingen scroll-effekter, ingen entry-animasjoner, ingen parallax. Side-følelsen er «litt død». V3-direction-fidelitet trekkes ned.

---

## For AI

Når du leverer designsystem eller layout:

1. **Kjør `check-visual-quality.ts` mot deployed preview FØR du rapporterer sprint ferdig.** Ikke spør — bare gjør det.
2. **Hvis snitt <7:** les Claude's `top_issues`, fiks topp 3, kjør på nytt. Itererer.
3. **`design_style` er en kontrakt** — hvis valgt direction er «Editorial Bento» skal sidene LIKNE Editorial Bento, ikke «Generic Tailwind Template».
4. **Hero MÅ ha visuell tyngde.** Tekst-CTA-på-flat-farge er aldri 10/10.
5. **Hver hovedside fortjener et signaturlayout.** Ikke kopier-lim samme template på 5 sider.
6. **Bygg motion samtidig som layout** — ikke som en ettertanke. L1-L4 i motion-rubrikken og V3 i visuell-rubrikken henger sammen.
