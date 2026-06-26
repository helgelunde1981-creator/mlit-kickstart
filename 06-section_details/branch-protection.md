# Branch protection på `main`

Hvert nytt repo opprettet via mlit-kickstart får automatisk branch
protection på `main`. Dette er en hard sikkerhetsgaranti — ikke en
preferanse — og er bygget for å hindre destruktive uhell fra både utvikler og
agent.

---

## Hva som settes

```json
{
  "required_status_checks": null,
  "enforce_admins": false,
  "required_pull_request_reviews": {
    "required_approving_review_count": 0,
    "dismiss_stale_reviews": false,
    "require_code_owner_reviews": false
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false,
  "required_linear_history": false,
  "required_conversation_resolution": false,
  "lock_branch": false,
  "allow_fork_syncing": false
}
```

### Hva dette gir

| Beskyttelse | Effekt |
|---|---|
| `allow_force_pushes: false` | Ingen kan force-push til main (heller ikke admin med mindre de slår av protection først) |
| `allow_deletions: false` | Branch kan ikke slettes |
| `required_pull_request_reviews` | Direkte `git push origin main` blokkeres — endringer MÅ gå via PR |
| `required_approving_review_count: 0` | Helge kan merge sin egen PR uten reviewer (solo-dev workflow) |
| `enforce_admins: false` | Admins (Helge) kan bypasse hvis kritisk hotfix |

### Hva som IKKE settes (med vilje)

- **`required_status_checks`**: Check-context-navn varierer per repo
  (`CodeQL`, `Vercel Preview Comments`, `Lighthouse`, `Build`, `Lint`...) og
  endrer seg over tid. Å hardkode dem her ville låst Helge ute hvis et workflow
  ble endret. Helge kan legge til per repo via Dashboard hvis ønskelig.
- **`required_linear_history`**: Helge merger ofte med merge-commit
  (PR-default). Krav om linear history ville tvinge rebase som er friksjon
  uten gevinst for solo-dev.
- **`require_code_owner_reviews`**: Ingen CODEOWNERS-fil i prosjektene.

---

## Hvorfor (mai 2026-sveip)

Sveip 30. mai 2026 viste at **25 av 26 helgelunde1981-creator-repos manglet protection
på `main`**. Bare `Somify` hadde det. Det betyr at:

- Force-push til main var mulig på 25 repos (kan ødelegge historie + alle
  åpne PRs)
- Branch-delete var mulig på 25 repos (kan slette `main` ved et uhell)
- Direkte push til main var mulig (ingen PR-krav)

Risiko var akutt etter at vi observerte at agenter (Claude, andre AI-verktøy)
har full GitHub-tilgang via `BOOTSTRAP_GITHUB_TOKEN` — et enkelt `git push
--force` kommando-uhell ville rammet hele orgen.

Alle 25 ble retrofitted i samme sveip. Fra og med denne datoen får hvert nytt
repo det automatisk via `applyBranchProtection()` i
`src/lib/kickstart/bootstrap/github.ts`.

---

## Implementasjon

Funksjonen er definert i [github.ts](../../src/lib/kickstart/bootstrap/github.ts)
og kalles som siste steg i `bootstrapGithub` etter `setDefaultBranch('dev')`:

```ts
// Branch protection på main: force-push + delete blokkert, krev PR
try {
  onProgress('Anvender branch protection på main…')
  await applyBranchProtection(slug, 'main')
  onProgress('main beskyttet: force-push + delete blokkert, PR påkrevd')
} catch (e) {
  // Ikke fail bootstrap hvis protection feiler — logg og fortsett
  const msg = e instanceof Error ? e.message : String(e)
  onProgress(`Advarsel: kunne ikke anvende branch protection (${msg})…`)
}
```

### Fallback-strategi

Hvis API-kallet feiler (typisk pga manglende PAT-permissions: PAT må ha
`administration: write` på orgen), faller bootstrap IKKE — vi logger en
advarsel og fortsetter. Pre-launch-sjekklisten verifiserer at protection
faktisk er på plass før prosjektet kan lanseres.

---

## Verifikasjon

### Manuelt

```bash
gh api /repos/helgelunde1981-creator/<slug>/branches/main/protection \
  --jq '{
    force_push: .allow_force_pushes.enabled,
    deletion: .allow_deletions.enabled,
    pr_required: .required_pull_request_reviews.required_approving_review_count
  }'
```

Forventet output:

```json
{ "force_push": false, "deletion": false, "pr_required": 0 }
```

### Sveipe-script (for å sjekke alle repos under orgen)

```bash
for repo in $(gh repo list helgelunde1981-creator --limit 100 --json name --jq '.[].name'); do
  status=$(gh api "/repos/helgelunde1981-creator/$repo/branches/main/protection" --silent 2>&1 | head -1)
  if echo "$status" | grep -q "Branch not protected\|HTTP 404"; then
    echo "MANGLER: $repo"
  fi
done
```

### Retrofit-script (for å anvende på et eksisterende repo)

```bash
PAYLOAD='{"required_status_checks":null,"enforce_admins":false,"required_pull_request_reviews":{"required_approving_review_count":0,"dismiss_stale_reviews":false,"require_code_owner_reviews":false},"restrictions":null,"allow_force_pushes":false,"allow_deletions":false,"required_linear_history":false,"required_conversation_resolution":false,"lock_branch":false,"allow_fork_syncing":false}'

echo "$PAYLOAD" | gh api -X PUT "/repos/helgelunde1981-creator/<slug>/branches/main/protection" --input -
```

---

## Pre-launch-sjekkliste (legg til)

```markdown
- [ ] `main` har branch protection (force-push + delete blokkert, PR påkrevd)
      Verifiser: `gh api /repos/helgelunde1981-creator/<slug>/branches/main/protection`
      returnerer 200 og `allow_force_pushes.enabled = false`
```

---

## PAT-permissions

`BOOTSTRAP_GITHUB_TOKEN` (PAT) trenger følgende scopes for at
`applyBranchProtection` skal funke:

- `repo` (full kontroll over private repos — allerede satt)
- `admin:org` (eller `administration: write` på finere granuleringsnivå)

Verifiser at PATen har `admin:org`-scope ved å sjekke
`https://github.com/settings/tokens` — eller via API:

```bash
curl -sI -H "Authorization: Bearer $BOOTSTRAP_GITHUB_TOKEN" \
  https://api.github.com/user | grep -i 'x-oauth-scopes'
```
