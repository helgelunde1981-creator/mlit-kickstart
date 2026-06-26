# Section Detail — Scrape Pipeline (når `requires_scrape: true`)

> Når Helge krysser av "Scrape eksisterende nettside", er **ALT innhold fra dagens side** prosjektets utgangspunkt. AI MÅ levere et komplett, kjørbart, idempotent 3-skripts-mønster + verifisering. **Halv-leveranse er en kritisk leveransefeil** — det fanger ingen opp før Helge ser at sidene er tomme i Sprint 10. Vi har vært der. Aldri igjen.

---

## Hvorfor dette dokumentet finnes (kort historikk)

Container Consult AS (mai 2026): Scraping kjørte og produserte 13 sider + 7 artikler + 100 bilder. Importen ble aldri kjørt fordi:

1. Skriptet leste `process.env.SUPABASE_URL` — men `.env.local` har bare `NEXT_PUBLIC_SUPABASE_URL`. Stille feil.
2. Importen mappet bare 6 av 13 scrape-filer (hardkodet whitelist). 7 sider ble bare ignorert.
3. Et eget bilde-import-skript fantes ikke i det hele tatt — 100 bilder lå som disk-filer.
4. Section-type `rich_text` matchet ikke DB-constraint (`text_block`, `hero`, etc.).
5. Sprint 1 "ferdig"-kriteriet sjekket ikke om DB hadde fått innholdet.

Resultat: prosjektet kom til Sprint 10 før noen oppdaget at innholdet manglet. **Det er målet med dette dokumentet å gjøre umulig.**

---

## Hovedregel: «alt eller ingenting»

Når `requires_scrape: true` er ALL av disse obligatorisk:

1. `scripts/scrape-existing-site.ts` — discover alle URL-er (sitemap.xml → fallback til crawl), scrape og lagre én JSON per side
2. `scripts/import-scraped-content.ts` — leser **ALLE** filer i `data/scraped/*.json` (ikke en hardkodet whitelist), upserter til CMS
3. `scripts/import-scraped-images.ts` — laster alle filer i `data/scraped/images/` til Supabase Storage + media-tabellen
4. `scripts/verify-scrape-import.ts` — sammenligner antall scrape-filer mot DB-rader, exit-code != 0 ved mismatch
5. **CI-jobb** som kjører `verify-scrape-import.ts` på hver PR — feiler PR hvis dekning ikke matcher
6. **Sprint 1 done-criteria** som krever output fra alle 4 skriptene + bestått verifisering

Hopper du over et av disse: vi rygger tilbake hit og fikser det. Det skal ikke skje to ganger.

---

## Skript 1 — `scripts/scrape-existing-site.ts`

### Krav
- Discover URL-er fra `sitemap.xml` først. Hvis ingen sitemap finnes: crawl med depth-limit fra `existing_url`.
- Følg `robots.txt` (rate-limit, blokkerte paths).
- Inkludér alle public-sider — også de som ikke er i meny (om-oss, kontakt, faq, jubileums-sider, blogger, etc.).
- Lagre én `data/scraped/<slug>.json` per side med:
  ```ts
  {
    url, path, scrapedAt,
    title, metaDescription,
    headings: { level, text }[],
    paragraphs: string[],
    images: { src, alt, width, height }[],
    links: { href, text }[]
  }
  ```
- Nyheter/blogg-artikler i egen undermappe: `data/scraped/nyheter/<slug>.json`.
- Bilder: last ned originalfilen til `data/scraped/images/<filename>` (behold filnavn).
- Idempotent — kjør på nytt skal oppdatere eksisterende filer.

### Anti-pattern
- ❌ Hardkodet liste over hvilke URL-er som scrapes
- ❌ Discovery kun via menyer (mister "skjulte" sider)
- ❌ Hoppe over bilder

---

## Skript 2 — `scripts/import-scraped-content.ts`

### Env-konvensjon (kritisk)

```ts
const URL = (process.env.SUPABASE_URL || process.env.NEXT_PUBLIC_SUPABASE_URL)?.trim();
const SERVICE_KEY = process.env.SUPABASE_SERVICE_ROLE_KEY?.trim();
if (!URL || !SERVICE_KEY) {
  console.error("Mangler SUPABASE_URL/NEXT_PUBLIC_SUPABASE_URL eller SUPABASE_SERVICE_ROLE_KEY");
  process.exit(1);
}
```

`.trim()` er obligatorisk pga. echo-newline-buggen som har bitt oss tre ganger.

### Multi-target-konvensjon (kritisk — alle seed-skript)

**Regel:** Hvis prosjektet har Supabase Branching (`requires_supabase_branching: true`), MÅ alle seed/import-skript støtte `TARGET=prod|dev|both` og defaulte til `both`. Aldri kjør kun mot én branch når begge eksisterer — det skaper drift mellom dev og prod, og preview-deploys blir ubrukelige fordi DB er tom.

**Hardt lært (container-consult-as, 30. mai 2026):** Vi importerte 13 sider + 7 artikler + 100 bilder til prod-ref `kloipbtwptdaywhkfcjy`. Dev-branch `itmvionzfhirslyvqawe` ble laget med `with_data: false` og forble TOM hele veien fra Sprint 1 til Sprint 11. Helge merket det først da han åpnet preview-URL og så blank side. Sprint 11.6 måtte retroaktivt seede dev mot alt vi hadde gjort mot prod.

**Standard env-naming:**
- `SUPABASE_URL_PROD` / `SUPABASE_SERVICE_ROLE_KEY_PROD` (eksisterende settes som default)
- `SUPABASE_URL_DEV` / `SUPABASE_SERVICE_ROLE_KEY_DEV` (NY — hent fra Management API for dev-branchen)

Sett alle 4 i `.env.local` + Vercel + GitHub Secrets.

**Helper-mønster (`scripts/lib/multi-target.ts`):**

```ts
import 'server-only'  // hvis brukt i Next-kontekst; ellers fjern

export type Target = 'prod' | 'dev'

export function resolveTargets(): Target[] {
  const t = process.env.TARGET?.trim()
  if (t === 'prod') return ['prod']
  if (t === 'dev') return ['dev']
  return ['prod', 'dev']  // default
}

export function getSupabaseConfig(target: Target): { url: string; serviceKey: string } {
  const url = target === 'prod'
    ? (process.env.SUPABASE_URL_PROD || process.env.SUPABASE_URL || process.env.NEXT_PUBLIC_SUPABASE_URL)?.trim()
    : process.env.SUPABASE_URL_DEV?.trim()
  const serviceKey = target === 'prod'
    ? (process.env.SUPABASE_SERVICE_ROLE_KEY_PROD || process.env.SUPABASE_SERVICE_ROLE_KEY)?.trim()
    : process.env.SUPABASE_SERVICE_ROLE_KEY_DEV?.trim()

  if (!url || !serviceKey) throw new Error(`Mangler env for target=${target}`)
  return { url, serviceKey }
}
```

**Bruksmønster i seed-skript:**

```ts
import { createClient } from '@supabase/supabase-js'
import { resolveTargets, getSupabaseConfig } from './lib/multi-target'

async function seedTarget(target: 'prod' | 'dev') {
  const { url, serviceKey } = getSupabaseConfig(target)
  const supabase = createClient(url, serviceKey, { auth: { persistSession: false } })
  console.log(`\n→ Seeding ${target} (${url})`)
  // ... gjør seed
  console.log(`✓ ${target} ferdig`)
}

async function main() {
  for (const target of resolveTargets()) {
    await seedTarget(target)
  }
}
```

**Kjøring:**

```bash
# Standard: kjør mot begge (anbefalt)
pnpm tsx scripts/import-scraped-content.ts

# Bare prod (sjelden — typisk når du fikser noe akutt i prod)
TARGET=prod pnpm tsx scripts/import-scraped-content.ts

# Bare dev (typisk når du retroaktivt seeder dev)
TARGET=dev pnpm tsx scripts/import-scraped-content.ts
```

**Verifisering — data-paritet:**

Lag `scripts/verify-data-parity.ts` som sammenligner DB-counts mellom prod og dev:

```ts
import { createClient } from '@supabase/supabase-js'
import { getSupabaseConfig } from './lib/multi-target'

const TABLES = ['pages', 'page_sections', 'articles', 'media', 'settings', 'services', 'locations', 'service_locations']

async function getCounts(target: 'prod' | 'dev') {
  const { url, serviceKey } = getSupabaseConfig(target)
  const supabase = createClient(url, serviceKey, { auth: { persistSession: false } })
  const counts: Record<string, number> = {}
  for (const t of TABLES) {
    const { count } = await supabase.from(t).select('*', { count: 'exact', head: true })
    counts[t] = count ?? 0
  }
  return counts
}

const prod = await getCounts('prod')
const dev = await getCounts('dev')

const mismatches = TABLES.filter((t) => prod[t] !== dev[t])
if (mismatches.length) {
  console.error('❌ Data-paritet brutt:')
  mismatches.forEach((t) => console.error(`   ${t}: prod=${prod[t]} dev=${dev[t]}`))
  process.exit(1)
}
console.log('✅ Dev = prod for alle tabeller')
```

CI-jobb i `.github/workflows/quality-gate.yml` skal kjøre denne — feil = PR-blokk.

**Storage:** husk at bilder også må lastes opp til DEV-branchens storage-bucket. Storage er per-branch i Supabase, ikke delt. `import-scraped-images.ts` skal også iterere over targets.

**Auth-config:** AuthConfigButton i mlit-admin har allerede «Sett på begge»-knapp — bruk den, ikke «kun prod».

### Discovery (kritisk — ingen hardkoding)

```ts
// Auto-discover alle scrape-filer, ikke whitelist
const files = (await readdir(SCRAPED_DIR)).filter((f) => f.endsWith(".json"));
```

For mapping av slug + title: heuristikk basert på filnavn + scraped path. Hvis du må lage en mapping-tabell, MÅ den dekke 100% av filene — sjekkes i verify-skriptet.

### Section-types (kritisk — sjekk DB-constraint)

Før du bruker en `type`-verdi i `page_sections`-insert, sjekk constraint:

```bash
# Lag denne sjekken i Sprint 1 done-checklist
psql ... -c "select pg_get_constraintdef(c.oid) from pg_constraint c
  join pg_class t on t.oid = c.conrelid
  where t.relname = 'page_sections' and c.contype = 'c';"
```

Hvis CMS-modellen tillater `hero`, `text_block`, `cta`, etc. — bruk DISSE, ikke oppfunne navn.

### Idempotent

Upsert på `slug` (pages), `slug` (articles), `key` (settings). Slett-og-erstatt for `page_sections` per page_id (rekkefølge-uavhengig).

### Logging

Skriv én linje per side: `✓ /slug (N avsnitt)`. Mismatch eller skipped må logges som `⚠️` så det er synlig i CI.

---

## Skript 3 — `scripts/import-scraped-images.ts`

### Pattern
- Les alle `data/scraped/images/*.{jpg,jpeg,png,gif,webp,avif,svg}`
- Last opp til Supabase Storage bucket `media` under prefix `scraped/`
- Insert rad i `public.media` med `filename`, `alt_text` (fra filnavn til Helge rydder i CMS), `file_url` (public URL), `mime_type`, `file_size_bytes`, `folder: 'scraped'`
- Idempotent: SELECT på `file_url` før insert; `upsert: false` på Storage og håndter `"already exists"` som "skipped"

### Alt-tekst-fallback

Filnavn renset for tall + bindestreker. Resultat: `alt_text: "10fotlager"` → "10fotlager" (kan endres i CMS).

Pre-launch må kreve at viktige bilder har manuelt redigert alt-tekst (sjekk i [pre-launch-checklist.md](./pre-launch-checklist.md)).

---

## Skript 3b — `scripts/link-images-to-pages.ts` (DATA-drevet, IKKE hardkodet)

Denne bygger broen mellom `media`-tabellen og `page_sections.data.image_id`. Den er en **engangs-seed** — etter første kjøring redigerer kunden i admin via mediavelger-modul.

**KRITISK forbud — ingen hardkodet `KEYWORDS: Record<slug, ...>`-mapping.** Container Consult Sprint 11.5 (mai 2026) fikk dette feil: AI lagde en hardkodet slug → keyword-liste i kode, som betyr at ny tjeneste lagt til via admin aldri ville fått bilder uten en utvikler. Det er stikk motsatt av et CMS-drevet prosjekt. Se `06-SECTION_DETAILS/dynamic-content.md` for full anti-pattern-forklaring.

**Riktig mønster (DATA-drevet):**

```ts
import { createClient } from "@supabase/supabase-js";

// Engangs-seed. Etter dette: kunden bruker admin-mediavelger.
const supabase = createClient(URL, KEY, { auth: { persistSession: false } });

interface Page { id: string; slug: string; title: string; meta_description: string | null }
interface Media { id: string; filename: string; alt_text: string; file_url: string }

const MIN_SCORE = 0.15; // empirisk, juster per prosjekt

function tokenize(s: string): Set<string> {
  return new Set(
    s.toLowerCase()
      .replace(/[^\w\sæøå]/g, " ")
      .split(/\s+/)
      .filter((t) => t.length >= 3)
  );
}

function jaccard(a: Set<string>, b: Set<string>): number {
  const intersection = [...a].filter((t) => b.has(t)).length;
  const union = new Set([...a, ...b]).size;
  return union === 0 ? 0 : intersection / union;
}

function scoreMatch(page: Page, media: Media): number {
  const haystack = `${page.title} ${page.slug} ${page.meta_description ?? ""}`;
  const needle = `${media.filename} ${media.alt_text}`;
  return jaccard(tokenize(haystack), tokenize(needle));
}

async function main() {
  const { data: pages } = await supabase.from("pages").select("id, slug, title, meta_description").eq("is_published", true);
  const { data: allMedia } = await supabase.from("media").select("id, filename, alt_text, file_url");
  if (!pages || !allMedia) throw new Error("Mangler data");

  for (const page of pages) {
    const ranked = allMedia
      .map((m) => ({ media: m, score: scoreMatch(page, m) }))
      .sort((a, b) => b.score - a.score);

    const best = ranked[0];
    if (best.score >= MIN_SCORE) {
      // Upsert hero-seksjon med image_id
      const { data: heroRow } = await supabase
        .from("page_sections")
        .select("id, data")
        .eq("page_id", page.id)
        .eq("type", "hero")
        .maybeSingle();

      const newData = { ...(heroRow?.data ?? {}), image_id: best.media.id, image_url: best.media.file_url, image_alt: best.media.alt_text };

      if (heroRow) {
        await supabase.from("page_sections").update({ data: newData }).eq("id", heroRow.id);
      } else {
        await supabase.from("page_sections").insert({ page_id: page.id, type: "hero", sort_order: 0, data: newData });
      }

      console.log(`✓ ${page.slug} → ${best.media.filename} (score ${best.score.toFixed(2)})`);
    } else {
      console.log(`⏭  ${page.slug} — ingen god match (best ${best.score.toFixed(2)}, ${best.media.filename}) — kunden velger i admin`);
    }
  }
}

main().catch((e) => { console.error(e); process.exit(1); });
```

**Avansert (Claude API tool-use):** For bedre matching kan engangs-skriptet kalle Claude med all scraped innhold + alle bilder, og få strukturert tilbake `{ page_slug, hero_media_id, gallery_media_ids[] }` per side. Resultatet er fortsatt DB-rader, ingen kode-mapping.

**Sikring mot re-kjøring:** skriptet skal hoppe over sider som allerede har `image_id` satt (kunden har tatt over). Bruk `--reset`-flagg for å overstyre eksplisitt.

```ts
if (heroRow?.data?.image_id && !process.argv.includes("--reset")) {
  console.log(`⏭  ${page.slug} — image_id allerede satt, hopper over (bruk --reset for å overskrive)`);
  continue;
}
```

**Admin-side:** mediavelger-modul med drag-drop fra `media`-tabellen er obligatorisk leveranse (jf. `cms-requirements.md`). Hvis den mangler er prosjektet ikke 10/10.

---

## Skript 4 — `scripts/verify-scrape-import.ts`

```ts
import { createClient } from "@supabase/supabase-js";
import { readdir } from "node:fs/promises";
import { join } from "node:path";

const supabase = createClient(
  (process.env.SUPABASE_URL || process.env.NEXT_PUBLIC_SUPABASE_URL)!.trim(),
  process.env.SUPABASE_SERVICE_ROLE_KEY!.trim(),
  { auth: { persistSession: false } },
);

const SCRAPED = join(process.cwd(), "data", "scraped");

const pageFiles = (await readdir(SCRAPED)).filter((f) => f.endsWith(".json"));
const articleFiles = await readdir(join(SCRAPED, "nyheter")).catch(() => []);
const imageFiles = (await readdir(join(SCRAPED, "images")).catch(() => []))
  .filter((f) => /\.(jpe?g|png|gif|webp|avif|svg)$/i.test(f));

const { count: pagesCount } = await supabase.from("pages").select("*", { count: "exact", head: true });
const { count: articlesCount } = await supabase.from("articles").select("*", { count: "exact", head: true });
const { count: mediaCount } = await supabase.from("media").select("*", { count: "exact", head: true });

const mismatches: string[] = [];
if ((pagesCount ?? 0) < pageFiles.length) {
  mismatches.push(`pages: ${pagesCount}/${pageFiles.length}`);
}
if ((articlesCount ?? 0) < articleFiles.length) {
  mismatches.push(`articles: ${articlesCount}/${articleFiles.length}`);
}
if ((mediaCount ?? 0) < imageFiles.length) {
  mismatches.push(`media: ${mediaCount}/${imageFiles.length}`);
}

if (mismatches.length > 0) {
  console.error("❌ Scrape-import er ufullstendig:");
  mismatches.forEach((m) => console.error(`   ${m}`));
  process.exit(1);
}

console.log(`✅ Scrape-dekning OK — pages ${pagesCount}, articles ${articlesCount}, media ${mediaCount}`);
```

Eksempel CI-jobb (`.github/workflows/verify-scrape.yml`):

```yaml
name: Verify scrape coverage
on: pull_request
jobs:
  verify:
    if: hashFiles('data/scraped/**') != ''  # kun hvis prosjektet har scrape
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - run: pnpm install --frozen-lockfile
      - run: pnpm tsx scripts/verify-scrape-import.ts
        env:
          SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          SUPABASE_SERVICE_ROLE_KEY: ${{ secrets.SUPABASE_SERVICE_ROLE_KEY }}
```

---

## Sprint 1 done-criteria (kopier inn i §20 Sprintplan)

Når `requires_scrape: true`, MÅ Sprint 1 ha disse oppgavene + sjekk:

- [ ] `scripts/scrape-existing-site.ts` skrevet og kjørt → `data/scraped/*.json` finnes med ALLE sider fra sitemap
- [ ] `scripts/import-scraped-content.ts` skrevet og kjørt → pages/articles/settings populated
- [ ] `scripts/import-scraped-images.ts` skrevet og kjørt → media-tabellen + storage populated
- [ ] `scripts/verify-scrape-import.ts` skrevet og kjørt → exit-code 0
- [ ] CI-workflow `.github/workflows/verify-scrape.yml` er på plass
- [ ] DB-count matcher fil-count (vis output fra verify-skriptet)
- [ ] Manuell sjekk: åpne 3 tilfeldige scrapede sider i CMS-admin, verifiser at innhold er der

**Ingen sprint 2 før alle 7 punktene er huket av.**

---

## Pre-launch (når scrape er del av prosjektet)

I `pre-launch-checklist.md` skal denne legges til under «Innhold»:

- [ ] Scrape-verifisering: kjør `pnpm tsx scripts/verify-scrape-import.ts` mot prod-DB — exit 0
- [ ] Spot-check minst 5 scrapede sider i prod: innhold, bilder, lenker, meta-description
- [ ] Alt-tekst gjennomgått og manuelt redigert på minimum hero-bilder + de 20 mest brukte bildene
- [ ] 301-redirects fra gamle URL-er satt opp (mapping bygget fra scraped `url`-felter)

---

## For AI (system-prompt-utdrag)

Når PROJECT.md har `requires_scrape: true`:

- Sprint 1 MÅ inneholde alle 4 skript-leveransene + CI-workflow + verifisering
- Sprint 1 done-criteria MÅ liste opp DB-count vs fil-count som hard sjekk
- Import-skript MÅ auto-oppdage filer (ikke whitelist), bruke `(SUPABASE_URL || NEXT_PUBLIC_SUPABASE_URL)?.trim()`, og bruke section-types som finnes i DB-constraint
- Pre-launch-sjekklisten i §16 MÅ ha de 4 scrape-spesifikke punktene over
