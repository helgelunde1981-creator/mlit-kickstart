# Delivery Score — målbar trend per prosjekt over tid

`score-delivery.ts` (se `auto-evaluator.md`) skriver én rad per PR til `delivery_scores`-tabellen i mlit-Supabase. Dette gir oss historikk på tvers av prosjekter og tid.

## Hvorfor

Uten historikk har vi ingen anelse om:
- Blir leveransene bedre over tid?
- Hvilken dimensjon (a11y, perf, CMS-coverage) sliter vi mest med?
- Hvilke prosjekter trekker snittet ned?
- Når et nytt prosjekt startes — er Sprint 0 raskere nå enn for 6 måneder siden?

Med historikk: ja-svar med data.

## Datamodell

```sql
create table if not exists delivery_scores (
  id uuid primary key default gen_random_uuid(),
  created_at timestamptz not null default now(),

  -- Prosjekt-kobling
  kickstart_project_id uuid references kickstart_projects(id) on delete set null,
  project_slug text not null,
  client_name text not null,

  -- Git-kontekst
  commit_sha text not null,
  pr_number integer,
  branch text,

  -- Score
  overall_score numeric(3,1) not null,  -- 0.0 - 10.0
  passed boolean not null,

  -- Per-dimensjon (JSONB)
  dimensions jsonb not null,
  -- Eksempel:
  -- [
  --   {"dimension": "lighthouse-performance", "score": 9.8, "passed": true, "detail": "avg 98.2 on 5 routes"},
  --   {"dimension": "lighthouse-accessibility", "score": 10.0, "passed": true, "detail": "avg 100 on 5 routes"},
  --   ...
  -- ]

  -- Workflow-kontekst
  workflow_run_url text,

  unique (project_slug, commit_sha)
);

create index if not exists delivery_scores_project_idx on delivery_scores(project_slug, created_at desc);
create index if not exists delivery_scores_passed_idx on delivery_scores(passed) where passed = false;
```

## Dashboard

I admin-modulen (`/admin/delivery`):
- Sortert liste over prosjekter med siste score
- Trend-graf per prosjekt (siste 30 PRs)
- Heatmap: prosjekter × dimensjoner — rødt på celler hvor en dimensjon konsekvent er under 9.0
- Filter: «kun feilende» / «siste 7 dager» / «alle»

Eksempel-spørring:
```sql
-- Topp 10 prosjekter med dårligst snitt-score siste 30 dager
select
  project_slug,
  client_name,
  round(avg(overall_score)::numeric, 1) as avg_score,
  count(*) as runs,
  count(*) filter (where passed = false) as failed_runs
from delivery_scores
where created_at > now() - interval '30 days'
group by project_slug, client_name
order by avg_score asc
limit 10;
```

## RLS

```sql
alter table delivery_scores enable row level security;

create policy "admin_read"
  on delivery_scores for select
  using (auth.jwt() ->> 'role' = 'admin');

create policy "ci_insert"
  on delivery_scores for insert
  with check (auth.role() = 'service_role');
```

## Hva tabellen IKKE skal inneholde

- Lighthouse-HTML-rapporter (de er GitHub-artifacts)
- A11y-violation-detaljer (de ligger i `reports/a11y/`)
- Kundefokus-data — ingenting personlig

Dette er **kvantitativ trend-data**. Detaljene ligger i CI-artifacts som er knyttet via `workflow_run_url`.

## Bruksområder

1. **Salg:** «Vi har levert 23 prosjekter siste 12 mnd, snitt-score 9.7/10»
2. **Intern læring:** Hvis a11y-dimensjonen ligger på 8.5 mens andre er 9.8 → vi har en blindspot
3. **Klient-rapport:** Etter lansering kan vi sende klienten siste score med bevis
4. **Forutsigelse:** Hvis Sprint 0 setter score 6.5 og første lansering må være 9.5, vet vi hvor mye arbeid som gjenstår
