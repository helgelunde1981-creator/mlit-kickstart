# Section Detail — Wizard-coverage (HVERT felt → leveranse → verifisering)

> Helge krysser av/skriver inn ting i kickstart-wizarden. Hvert valg er et **løfte til kunden** om leveranse. Container Consult AS (mai 2026): Helge valgte «Livlig motion» → sidene ble statiske. Han valgte «Signature moment» → ble gjemt på /bestill. Han valgte 7 tjenester → bare container-flow ble bygget. **AI tolket wizarden som «forslag», ikke som kontrakt. Aldri igjen.**

---

## Hovedregel: Wizard = Kontrakt

Hvert ikke-info-felt i `kickstart_projects`-raden MÅ leveres OG verifiseres. Hvis et felt er satt og leveransen mangler:
- **Sprint som skulle levert det er ikke ferdig.**
- **PR mot main blokkere.**
- **Pre-launch blokkere.**

Info-felter (`client_name`, `vision`, `org_nr`, `contact_person`) er kontekst — ikke noe å verifisere i kode. Alt annet er leveranse.

---

## Felt-for-felt mapping

### Steg 1: Kunde

| Felt | Type | Leveranse | Verifisering |
|---|---|---|---|
| `client_name` | info | Brukes i logo-alt, footer, OG-tags, meta-title | Grep prod-build for forekomst |
| `industry` | info | Påvirker tone i specs | Ingen kode-sjekk |
| `org_nr` | info | I footer + e-postsignatur | Grep footer for forekomst |
| `contact_person` | info | I admin-bootstrap (første bruker) | DB-sjekk: `auth.users` har e-post |
| `existing_url` | trigger | Hvis satt + `requires_scrape=true` → full scrape-pipeline | Se `scrape-pipeline.md` |
| `new_domain` | leveranse | DNS + Vercel-domene + SSL + 301-redirects fra preview | `curl -I https://<new_domain>` returnerer 200 |

### Steg 2: Prosjekttype

| Felt | Type | Leveranse | Verifisering |
|---|---|---|---|
| `project_types` | trigger | Hver type aktiverer ulike krav (saas → multi-tenant + billing; ecommerce → produkt-katalog + checkout; cms → admin) | Per type: relevante tabeller + sider eksisterer |
| `core_product` | info | I §5.1 av spec | Ingen kode-sjekk |
| `vision` | info | I §1 + OG-description | Grep meta |

### Steg 3: Tech-stack

| Felt | Type | Leveranse | Verifisering |
|---|---|---|---|
| `frontend_stack[]` | leveranse | Hver valgt teknologi MÅ være installert + brukt | `package.json` + grep i `src/` for import-statement |
| `backend_stack[]` | leveranse | Hver valgt installert + brukt | Samme |
| `integrations[]` | leveranse | Hver integrasjon installert + ENV-konfigurert + faktisk brukt (ikke død import) | Se mapping under |
| `mobile_stack[]` | leveranse | Hvis satt: Expo/RN-app i monorepo eller separat repo | Sjekke `apps/mobile/` eller eget repo |
| `hosting_infra[]` | leveranse | Vercel: `.vercel/project.json` + `vercel.json`. GitHub Actions: `.github/workflows/*.yml` | Filer eksisterer |

### Integrasjons-mapping (per `integrations[]`-verdi)

| Integrasjon | Pakke | Env-vars | Brukt-sjekk |
|---|---|---|---|
| `Stripe` | `stripe` + `@stripe/stripe-js` | `STRIPE_SECRET_KEY`, `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | `src/lib/stripe.ts` finnes + brukt i checkout |
| `Vipps` | `@vippsmobilepay/*` | `VIPPS_CLIENT_ID/SECRET/SUBSCRIPTION_KEY` | `src/lib/vipps.ts` + checkout-rute |
| `Resend` | `resend` | `RESEND_API_KEY` | `src/lib/email.ts` + minst én `sendEmail`-bruk |
| `Twilio SMS` | `twilio` | `TWILIO_ACCOUNT_SID/AUTH_TOKEN/PHONE` | `src/lib/sms.ts` + bruk |
| `Claude API` | `@anthropic-ai/sdk` | `ANTHROPIC_API_KEY` | Minst én Anthropic-call |
| `OpenAI` | `openai` | `OPENAI_API_KEY` | Minst én OpenAI-call |
| `Google Maps/Places` | `@googlemaps/js-api-loader` | `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` | Maps-komponent + CSP `maps.googleapis.com` |
| `Cloudflare Turnstile` | `@marsidev/react-turnstile` | `TURNSTILE_SITE_KEY/SECRET_KEY` | Skjema-validering |
| `PostHog` | `posthog-js` + `posthog-node` | `NEXT_PUBLIC_POSTHOG_KEY/HOST` | `posthog.init()` + minst én `capture()` |
| `Plausible` | `next-plausible` | `NEXT_PUBLIC_PLAUSIBLE_DOMAIN` | `<PlausibleProvider>` i layout |
| `GA4` | `next/script` med gtag | `NEXT_PUBLIC_GA_ID` | gtag-script + `dataLayer.push`-events |
| `Sentry` | `@sentry/nextjs` | `SENTRY_DSN/AUTH_TOKEN/ORG/PROJECT` | `sentry.client.config.ts` + `sentry.server.config.ts` |
| `Tripletex` | `axios` + custom | `TRIPLETEX_CONSUMER_TOKEN/EMPLOYEE_TOKEN` | `src/lib/tripletex.ts` + faktisk API-call |
| `Cloudinary` | `next-cloudinary` | `CLOUDINARY_*` | `<CldImage>` + minst én upload |
| `TipTap` | `@tiptap/*` | (ingen) | `RichTextEditor` brukt i CMS |
| `n8n` | webhook (ingen pakke) | `N8N_WEBHOOK_URL` | minst én webhook-trigger |
| `Meta Pixel` | `next/script` | `NEXT_PUBLIC_META_PIXEL_ID` | Pixel-snippet + events |
| `Vercel Analytics` | `@vercel/analytics` | (ingen) | `<Analytics />` i layout |

### Steg 4: Krav

| Felt | Type | Leveranse | Verifisering |
|---|---|---|---|
| `auth_type[]` | leveranse | Per type: Supabase Auth konfigurert med riktig provider | `src/app/login`-rute + provider-imports |
| `features` (tekst) | leveranse | HVER linje i features = oppgave i sprintplanen + minst én commit | Manuell + AI-rapport per feature |
| `pages_structure` (tekst) | leveranse | Hver listed URL returnerer 200 | Playwright HEAD-sjekk per linje |
| `requires_seo` | bool | sitemap.xml + robots.txt + llms.txt + JSON-LD på alle sider + meta-tags | `check-public-pages.ts` I1-I4 + grep sitemap/robots |
| `requires_gdpr` | bool | **Cookiebot** consent (ALDRI egen banner/`cookie_consents`-tabell) + /personvern + /cookies (Cookiebot `CookieDeclaration`) + GDPR-endpoints (slett/innsyn) + Cookiebot-domener i CSP. Spør Helge om CBID. | Grep Cookiebot-script + CSP-domener + ruter + Playwright |
| `requires_a11y` | bool | axe-core 0 violations + keyboard-test + reduced-motion | `check-public-pages.ts` H-kategori |
| `requires_scrape` | bool | scrape-pipeline (egen doc) | Se `scrape-pipeline.md` + K3 i rubrikk |
| `requires_supabase_branching` | bool | Dev-branch finnes + data-paritet (egen doc) | Se §32 + `verify-data-parity.ts` |
| `requires_e2e` | bool | Playwright E2E-tester for ≥3 kritiske flyter (booking/login/kontakt) | `tests/e2e/`-mappa har ≥3 spec-filer + CI kjører dem |
| `requires_rate_limiting` | bool | Upstash/Vercel KV rate-limiter på POST-endepunkter + Turnstile på public skjemaer | Grep `ratelimit.limit(` + Turnstile i skjema-imports |
| `requires_rls` | bool | RLS aktiv på ALLE public-tabeller + minst én policy per tabell | SQL: `select tablename from pg_tables where schemaname='public' and rowsecurity=false` → tom |
| `requires_audit_log` | bool | `audit_log`-tabell + triggers på write-tabeller + admin-UI for å lese | Tabell finnes + minst én trigger + `/admin/audit`-rute |
| `requires_dark_mode` | bool | Theme-toggle funksjonell (data-theme veksles) + lys/mørk tokens i tokens.css | `check-public-pages.ts` E3 + grep `[data-theme="dark"]` i CSS |

### Steg 5: Design

| Felt | Type | Leveranse | Verifisering |
|---|---|---|---|
| `design_style[]` | leveranse | Valgt direction MÅ være synlig i koden (ikke bare nevnt i §6) | Anti-template-grep + visuell stikkprøve |
| `color_primary` | leveranse | I `tokens.css` som `--color-primary` + brukt på CTA-er (`bg-primary`) | Grep tokens + grep `bg-primary` ≥ 3 ganger |
| `color_secondary` | leveranse | I tokens som `--color-secondary` + brukt | Grep tokens + bruk |
| `font_preference[]` | leveranse | Hver valgt font lastes via `next/font` + brukt på riktig element | `src/app/layout.tsx` har font-import + body/heading bruker variable |
| `motion_preference[]` | leveranse | Se egen doc `motion-strategy.md` (L1-L4) | `check-public-pages.ts` L-kategori |
| `design_references` (tekst) | info | Påvirker direction-valg | Ingen kode-sjekk |

### Steg 6: Scope

| Felt | Type | Leveranse | Verifisering |
|---|---|---|---|
| `budget` / `deadline` / `sprint_estimate` | info | I §20-header | Manuell |
| `out_of_scope` (tekst) | dokumentasjon | Hver linje SKAL være i §17 «Ikke i scope» + ikke implementert | Grep PROJECT.md §17 |
| `open_questions` (tekst) | trigger | Hver linje SKAL være i §18 «Spørsmål til kunden» + adresseres innen Sprint 1 | Manuell |
| `extra_notes` (tekst) | info | Påvirker context | Ingen kode-sjekk |

---

## W-kategori i kvalitetsrubrikken (4 poeng — prosjekt-globale)

- **W1: Tech-stack-coverage** — hvert element i `frontend_stack` + `backend_stack` + `mobile_stack` + `hosting_infra` er installert OG brukt i `src/`
- **W2: Integrasjon-coverage** — hver `integrations[]`-verdi har pakke + env-var + ≥1 bruk i kode
- **W3: Krav-coverage** — hvert `requires_*=true` har sin verifisering grønn (se mapping over)
- **W4: Design-coverage** — `color_primary` + `color_secondary` i tokens + brukt, `font_preference` lastet via `next/font`, `motion_preference` implementert (L1-L4 grønn), `design_style` synlig (anti-template-grep grønn)

Alle 4 prosjekt-globale (samme på alle sider). Failure på én = score dropper for HELE prosjektet.

---

## `scripts/check-wizard-coverage.ts` — verifikator-mal

```ts
import { createClient } from "@supabase/supabase-js";
import { readFile } from "node:fs/promises";

// Henter kickstart_projects-raden fra mlit-DB
const kickstartSupabase = createClient(
  process.env.KICKSTART_SUPABASE_URL!.trim(),
  process.env.KICKSTART_SUPABASE_SERVICE_KEY!.trim(),
  { auth: { persistSession: false } },
);

const KICKSTART_ID = process.env.KICKSTART_PROJECT_ID!.trim();

const { data: project } = await kickstartSupabase
  .from("kickstart_projects")
  .select("*")
  .eq("id", KICKSTART_ID)
  .single();

if (!project) { console.error("Fant ikke kickstart-prosjekt"); process.exit(1); }

const pkg = JSON.parse(await readFile("package.json", "utf-8"));
const allDeps = { ...pkg.dependencies, ...pkg.devDependencies };

const failures: string[] = [];

// ─── W1: Tech-stack ──────────────────────────────────────────
const TECH_PKG_MAP: Record<string, string[]> = {
  "Next.js 15": ["next"],
  "React": ["react"],
  "TypeScript": ["typescript"],
  "Tailwind CSS": ["tailwindcss"],
  "shadcn/ui": ["@radix-ui/react-dialog"], // shadcn er copy-paste, sjekker for Radix-grunnlag
  "Zod": ["zod"],
  "React Hook Form": ["react-hook-form"],
  "Framer Motion": ["motion", "framer-motion"],
  "GSAP": ["gsap"],
  "Supabase": ["@supabase/supabase-js", "@supabase/ssr"],
  "PostgreSQL": ["@supabase/supabase-js"], // implisitt via Supabase
  "Prisma": ["@prisma/client"],
  // ... utvid per behov
};

const allTechFields = [...project.frontend_stack, ...project.backend_stack, ...project.hosting_infra];
for (const tech of allTechFields) {
  const expectedPkgs = TECH_PKG_MAP[tech];
  if (!expectedPkgs) continue; // ukjent tech, skip
  const installed = expectedPkgs.some((p) => allDeps[p]);
  if (!installed) failures.push(`W1: ${tech} ble valgt men er ikke installert (${expectedPkgs.join(" / ")})`);
}

// ─── W2: Integrasjoner ───────────────────────────────────────
const INTEGRATION_MAP: Record<string, { pkg: string; envVars: string[] }> = {
  "Stripe": { pkg: "stripe", envVars: ["STRIPE_SECRET_KEY"] },
  "Vipps": { pkg: "@vippsmobilepay/checkout-sdk", envVars: ["VIPPS_CLIENT_ID"] },
  "Resend": { pkg: "resend", envVars: ["RESEND_API_KEY"] },
  "Claude API": { pkg: "@anthropic-ai/sdk", envVars: ["ANTHROPIC_API_KEY"] },
  "OpenAI": { pkg: "openai", envVars: ["OPENAI_API_KEY"] },
  "Plausible": { pkg: "next-plausible", envVars: ["NEXT_PUBLIC_PLAUSIBLE_DOMAIN"] },
  "Sentry": { pkg: "@sentry/nextjs", envVars: ["SENTRY_DSN"] },
  "Google Maps/Places": { pkg: "@googlemaps/js-api-loader", envVars: ["NEXT_PUBLIC_GOOGLE_MAPS_API_KEY"] },
  "Cloudflare Turnstile": { pkg: "@marsidev/react-turnstile", envVars: ["TURNSTILE_SITE_KEY"] },
  "PostHog": { pkg: "posthog-js", envVars: ["NEXT_PUBLIC_POSTHOG_KEY"] },
  "Vercel Analytics": { pkg: "@vercel/analytics", envVars: [] },
  "Sentry": { pkg: "@sentry/nextjs", envVars: ["SENTRY_DSN"] },
  // ... utvid
};

for (const integration of project.integrations) {
  const map = INTEGRATION_MAP[integration];
  if (!map) continue;
  if (!allDeps[map.pkg]) failures.push(`W2: ${integration} valgt men ${map.pkg} ikke installert`);
  for (const env of map.envVars) {
    if (!process.env[env]) failures.push(`W2: ${integration} mangler env ${env}`);
  }
}

// ─── W3: requires_*-flagg ────────────────────────────────────
async function fileExists(path: string): Promise<boolean> {
  try { await readFile(path); return true; } catch { return false; }
}

async function grep(pattern: string, paths: string[]): Promise<boolean> {
  const { spawnSync } = await import("node:child_process");
  const result = spawnSync("grep", ["-rE", pattern, ...paths], { encoding: "utf-8" });
  return result.status === 0;
}

if (project.requires_seo) {
  if (!await fileExists("src/app/sitemap.ts") && !await fileExists("public/sitemap.xml"))
    failures.push("W3: requires_seo men ingen sitemap");
  if (!await fileExists("src/app/robots.ts") && !await fileExists("public/robots.txt"))
    failures.push("W3: requires_seo men ingen robots");
}

if (project.requires_gdpr) {
  // Consent = Cookiebot (ALDRI egen banner). Sjekk at Cookiebot finnes — ikke en CookieBanner.
  if (grepCount("cookiebot\\.com|data-cbid|Cookiebot", ["src/"]) === 0)
    failures.push("W3: requires_gdpr men Cookiebot mangler (spør Helge om CBID)");
  if (grepCount("CookieBanner|cookie_consents", ["src/"]) > 0)
    failures.push("W3: egen cookie-banner/consent funnet — FORBUDT, bruk Cookiebot");
  if (grepCount("consent\\.cookiebot\\.com", ["src/middleware.ts", "next.config.ts"]) === 0)
    failures.push("W3: Cookiebot-domener mangler i CSP (script-src + connect-src)");
}

if (project.requires_e2e) {
  const e2eDirs = ["tests/e2e", "e2e", "playwright/tests"];
  const hasE2E = (await Promise.all(e2eDirs.map(fileExists))).some(Boolean);
  if (!hasE2E) failures.push("W3: requires_e2e men ingen tests/e2e-mappe");
}

if (project.requires_rate_limiting) {
  const hasRateLimit = await grep("ratelimit.*limit\\(|@upstash/ratelimit", ["src/"]);
  if (!hasRateLimit) failures.push("W3: requires_rate_limiting men ingen ratelimit-bruk");
}

if (project.requires_audit_log) {
  // sjekk DB-tabell finnes — krever Supabase-call
  // ... implementer
}

if (project.requires_dark_mode) {
  const hasThemeToggle = await grep("data-theme|ThemeToggle", ["src/"]);
  if (!hasThemeToggle) failures.push("W3: requires_dark_mode men ingen theme-toggle");
}

// ─── W4: Design ──────────────────────────────────────────────
if (project.color_primary) {
  const tokensFile = await readFile("src/styles/tokens.css", "utf-8").catch(() => "");
  if (!tokensFile.includes("--color-primary"))
    failures.push("W4: color_primary valgt men --color-primary mangler i tokens.css");
}

if (project.font_preference?.length > 0) {
  const layoutFile = await readFile("src/app/layout.tsx", "utf-8").catch(() => "");
  if (!layoutFile.includes("next/font"))
    failures.push("W4: font_preference satt men next/font ikke brukt i layout.tsx");
}

if (project.motion_preference?.[0] && project.motion_preference[0] !== "Statisk") {
  // L1-L4 sjekkes i check-public-pages.ts — bare advarsel her
  // ...
}

// ─── Rapport ─────────────────────────────────────────────────
if (failures.length > 0) {
  console.error(`\n❌ ${failures.length} wizard-leveranser MANGLER:`);
  failures.forEach((f) => console.error(`   ${f}`));
  process.exit(1);
}

console.log("✅ Alle wizard-leveranser verifisert");
```

CI-jobb i `.github/workflows/quality-gate.yml` kjører dette — rødt = PR-blokk.

---

## For AI

Når du genererer eller jobber på et Myrvoll-Lunde IT Drift-prosjekt:

1. **Les `kickstart_projects`-raden FØR du planlegger sprintene.** Hvert satt felt = oppgave.
2. **For hvert `requires_*=true`-flagg**: legg til konkret implementasjon-oppgave i sprintplanen. Ikke bare nevn det i §14-tekst — bygg det.
3. **For hver `integrations[]`-verdi**: installer pakken + sett ENV + bruk det minst én gang. Ingen «installert men aldri brukt»-pakker.
4. **For `motion_preference !== 'Statisk'`**: følg `motion-strategy.md` — installer bibliotek + implementer L1-L4.
5. **For `design_style[]`**: bruk valgt direction i koden — anti-template-grep blokkere generiske templates.
6. **For `pages_structure` (tekst)**: hver linje = ny side. Bygg dem alle.
7. **For `features` (tekst)**: hver linje = oppgave i sprintplanen + minst én commit.
8. **Pre-launch krever `scripts/check-wizard-coverage.ts` exit 0.**
