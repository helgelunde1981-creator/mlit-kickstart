# Section Detail — Datamodell (Supabase Postgres)

> Brukes når AI fyller ut seksjon 12 (Datamodell) i PROJECT.md.

---

## Minimum required output

Seksjon 12 må inneholde:

1. **Komplett SQL** for alle tabeller (CREATE TABLE)
2. **RLS aktivert** på alle public tabeller
3. **RLS-policyer per tabell** med tydelig SELECT/INSERT/UPDATE/DELETE-separasjon
4. **Triggers** (updated_at, etc.)
5. **Indekser** for kjente query-mønstre
6. **Migrasjonsfil-format** følger: `supabase/migrations/YYYYMMDDHHMMSS_navn.sql`

## Felles patterns

### Standard kolonner per tabell

```sql
create table public.example (
  id uuid primary key default gen_random_uuid(),
  -- ... domene-kolonner ...
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

### updated_at-trigger

```sql
create trigger example_updated_at
  before update on public.example
  for each row execute function moddatetime(updated_at);
```

(Krever `create extension if not exists moddatetime;` i en tidlig migrasjon.)

### RLS-policy patterns

**Public read for published rader:**

```sql
alter table public.pages enable row level security;

create policy "Public can read published pages"
  on public.pages
  for select
  to anon, authenticated
  using (status = 'published');

create policy "Authenticated can do all"
  on public.pages
  for all
  to authenticated
  using (true)
  with check (true);
```

**Eier-only (per profile.id):**

```sql
create policy "Users can read own data"
  on public.user_data
  for select
  to authenticated
  using (user_id = (select auth.uid()));

create policy "Users can update own data"
  on public.user_data
  for update
  to authenticated
  using (user_id = (select auth.uid()))
  with check (user_id = (select auth.uid()));
```

**Server-side only (INSERT via service-role):**

```sql
create policy "Server can insert"
  on public.bookings
  for insert
  to service_role
  with check (true);

create policy "Superadmin can read all"
  on public.bookings
  for select
  to authenticated
  using ((select auth.jwt() ->> 'role') = 'superadmin');
```

## Typiske tabeller per prosjekt-type

### Public-site / CMS

- `profiles` (extends auth.users)
- `pages` + `page_sections` + `page_revisions`
- `media`
- `blog_posts`
- `site_settings`
- `form_submissions`
- `audit_log`

> Ingen `cookie_consents`-tabell: consent håndteres av **Cookiebot** (se `compliance-norge.md`). Vi bygger aldri egen samtykke-logg.

### E-handel

- `products` + `product_variants` + `product_images`
- `categories`
- `orders` + `order_items`
- `customers`
- `discounts`
- `inventory_movements`

### Booking / kalender

- `services`
- `availability`
- `bookings` + `booking_status_history`
- `payments`

### SaaS multi-tenant

- `organizations`
- `memberships` (user × organization × role)
- `invitations`
- `subscription_plans`
- `subscriptions`
- `usage_metrics`
- Per-feature tabeller med `organization_id` på hver rad

## Indekser

Standard indekser å vurdere:

```sql
-- For slugs (lookup)
create unique index pages_slug_idx on public.pages (slug);

-- For listing med sortering
create index pages_status_created_idx on public.pages (status, created_at desc);

-- For foreign keys
create index page_sections_page_id_idx on public.page_sections (page_id);

-- For søk (gin på norsk fulltekst)
create index pages_title_search_idx on public.pages using gin (to_tsvector('norwegian', title));

-- For JSONB-felter som queries hyppig
create index page_sections_data_type_idx on public.page_sections ((data->>'type'));
```

## Migrasjons-format og navngivning (KRITISK)

Per Helge's globale memory:

### Filnavn-format

**Alltid** `supabase/migrations/YYYYMMDDHHMMSS_kort_beskrivelse.sql`

| Del | Format | Eksempel | Hvorfor |
|-----|--------|----------|---------|
| `YYYY` | 4-sifret år | `2026` | Sortert kronologisk |
| `MM` | 2-sifret måned (01-12) | `05` | Padded, leksikografisk sortering |
| `DD` | 2-sifret dag (01-31) | `28` | Padded |
| `HHMMSS` | 2-sifret time + min + sek (24t UTC) | `143052` | Sikrer unik rekkefølge ved flere migrasjoner samme dag |
| `_kort_beskrivelse` | `snake_case`, ASCII-only, ≤ 40 tegn | `_add_kickstart_projects` | Lesbar i `git log` og `supabase migration list` |
| `.sql` | filendelse | `.sql` | Supabase CLI gjenkjenner kun `.sql` |

**Gyldige eksempler:**
- `20260528120030_add_kickstart_projects.sql`
- `20260528143200_add_rls_policy_for_bookings.sql`
- `20260528160000_alter_users_add_role.sql`

**Ugyldige:**
- ❌ `add_kickstart.sql` (ingen timestamp)
- ❌ `2026-05-28_add_kickstart.sql` (separator-tegn, ikke YYYYMMDDHHMMSS)
- ❌ `20260528_add_kickstart.sql` (mangler HHMMSS)
- ❌ `20260528120030_add-kickstart-projects.sql` (kebab-case, bruk snake_case)
- ❌ `20260528120030_legger_til_kickstart_prosjekter_med_full_RLS_og_indekser.sql` (for langt)
- ❌ Flate SQL-filer i `supabase/` (må være i `migrations/`)

### Genere ny migrasjon

**Alltid** bruk CLI for å sikre korrekt timestamp:

```bash
npx supabase migration new add_kickstart_projects
# → Creates supabase/migrations/YYYYMMDDHHMMSS_add_kickstart_projects.sql
```

Bruk UTC for timestamp (CLI gjør dette automatisk). Aldri lokal tid.

### SQL-innhold

- **Idempotent der mulig:** `create table if not exists`, `create policy if not exists`
- **Reversibel der mulig:** legg til en kommentar med tilbakerullings-SQL øverst
- **En logisk endring per fil:** ikke samle 5 ulike skjema-endringer i én migrasjon
- **Kommentar på toppen:** `-- Migrasjon: <kort beskrivelse>`
- **Aldri `DROP TABLE`** uten eksplisitt godkjenning fra Helge — bruk `RENAME` til `_deprecated_*` først

## Migrasjons-deployment — KUN via GitHub Actions + Supabase Branching

**Aldri** kjør disse mot prod-DB:

- ❌ `npx supabase db push` lokalt
- ❌ `psql` direkte mot prod-connection
- ❌ MCP-Supabase `apply_migration`
- ❌ Dashboard SQL Editor for skjema-endringer
- ❌ Endre tabeller manuelt i Studio

**Alltid** via Supabase Branching + GitHub Actions:

### Flyt for hver skjema-endring

```
1. LOKAL UTVIKLING
   $ npx supabase migration new add_kickstart_projects
   → supabase/migrations/20260528143052_add_kickstart_projects.sql
   $ # Skriv SQL i filen
   $ git checkout -b feat/kickstart-tabell
   $ git commit && git push

2. PR mot dev
   → GitHub Actions: Supabase Branching workflow kjører
   → Oppretter eller bruker eksisterende dev-branch-DB
   → Kjører ALLE migrasjoner i `supabase/migrations/` mot dev-branch
   → Vercel preview-deploy linkes til dev-branch-DB
   → Test i preview med ekte DB-state

3. PR merget til dev
   → Migrasjoner allerede applied til dev-branch-DB
   → dev-branch er nå «truth» for staging

4. PR fra dev → main
   → Workflow kjører migrasjonene mot PROD-DB
   → main-deploy linkes til prod-DB
   → Verifiser med `npx supabase migration list` at prod og repo matcher
```

### Supabase Branching (per prosjekt)

Aktiveres i Supabase Dashboard → Settings → Branching:

- **Persistent dev-branch:** felles for hele teamet (én branch-DB for `dev`-Git-branch)
- **Ephemeral preview-branches** (valgfritt): per Pull Request → egen branch-DB som auto-slettes ved merge
- **Hoved-branch (prod):** main → prod-DB

Hver branch er en ekte Postgres-database med eget skjema + RLS-policyer.

### `.github/workflows/supabase-migrations.yml` (obligatorisk)

```yaml
name: Supabase Migrations
on:
  push:
    branches: [main, dev]
    paths: ['supabase/migrations/**']
  pull_request:
    branches: [main, dev]
    paths: ['supabase/migrations/**']

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: supabase/setup-cli@v1
        with:
          version: latest

      - name: Link Supabase project
        run: supabase link --project-ref ${{ secrets.SUPABASE_PROJECT_REF }}
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}

      - name: Push migrations to PROD (main only)
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: supabase db push --include-all
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}

      - name: Push migrations to DEV-branch (dev only)
        if: github.ref == 'refs/heads/dev' && github.event_name == 'push'
        run: supabase db push --db-url ${{ secrets.SUPABASE_DEV_DB_URL }}
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}

      - name: Validate migrations (PR only — dry-run)
        if: github.event_name == 'pull_request'
        run: supabase db diff --use-migra --schema public
```

### Påkrevde GitHub secrets

Settes via `gh secret set` på repo-nivå:

```bash
gh secret set SUPABASE_PROJECT_REF --body "abc123xyz"      # prod project ref
gh secret set SUPABASE_ACCESS_TOKEN --body "sbp_..."       # personal access token
gh secret set SUPABASE_DEV_DB_URL --body "postgres://..."  # dev-branch connection
```

### Hvorfor denne flyten er kritisk

- **Audit-trail:** git-history viser nøyaktig HVEM endret HVA og NÅR
- **Reversibel:** revert PR + ny PR = rollback
- **Konsistent state:** lokal, dev, og prod er ALLTID i sync med `supabase/migrations/`
- **Ingen drift:** umulig å glemme å pushe en migrasjon — workflow kjører automatisk
- **Forebygger feil DB:** workflow leser `SUPABASE_PROJECT_REF` fra secrets — kan ikke gå mot feil prosjekt
- **PR-validering:** dry-run i PR sjekker at SQL er gyldig før merge

`.github/workflows/supabase-migrations.yml`:

```yaml
name: Supabase Migrations
on:
  push:
    branches: [main, dev]
    paths: ['supabase/migrations/**']
jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: supabase/setup-cli@v1
      - name: Link Supabase
        run: supabase link --project-ref ${{ secrets.SUPABASE_PROJECT_REF }}
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
      - name: Push migrations (prod)
        if: github.ref == 'refs/heads/main'
        run: supabase db push
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
      - name: Push migrations (dev-branch)
        if: github.ref == 'refs/heads/dev'
        run: supabase db push --db-url ${{ secrets.SUPABASE_DEV_DB_URL }}
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
```

GitHub secrets som må settes (via `gh secret set`):
- `SUPABASE_PROJECT_REF`
- `SUPABASE_ACCESS_TOKEN` (fra Supabase Dashboard → Account → Access Tokens)
- `SUPABASE_DEV_DB_URL` (dev-branch DB connection string)

## Typing

Etter migrasjon:

```bash
npx supabase gen types typescript --project-id $PROJECT_ID > src/types/database.types.ts
```

Lagre `Database`-typen i `src/types/database.types.ts`. Bruk via:

```ts
import type { Database } from '@/types/database.types'

type Page = Database['public']['Tables']['pages']['Row']
type PageInsert = Database['public']['Tables']['pages']['Insert']
type PageUpdate = Database['public']['Tables']['pages']['Update']
```

## RLS testing

I PROJECT.md §20 (Sprintplan), inkluder denne testen i test-sprint:

```ts
// test/rls.test.ts
import { createClient } from '@supabase/supabase-js'

const anonClient = createClient(URL, ANON_KEY)

it('anon cannot insert into bookings', async () => {
  const { error } = await anonClient.from('bookings').insert({ ... })
  expect(error?.code).toBe('42501') // permission denied
})
```

## Supabase branching

- Push migration til `dev` → auto-deploy til dev-branch
- Push til `main` → auto-deploy til prod
- Verifiser med `npx supabase migration list` etter hver push
