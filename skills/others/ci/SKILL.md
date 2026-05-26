---
name: ci
description: Set up a GitHub Actions CI/CD pipeline for any project. Configures lint, typecheck, test, build, preview deploys on PRs, and production deploy on main. Use when setting up a new project or when CI is missing or broken.
argument-hint: <deploy target: vercel | fly | dokploy | cloudflare | none>
---

# CI

You are setting up a complete CI/CD pipeline with GitHub Actions. Work through each phase in order.

**Deploy target:** {{args}}


## Phase 0: Auto-Update

*Skip unless `{{args}}` contains `--update`, or `SKILLS_AUTO_UPDATE: true` is set in your project CLAUDE.md.*

```bash
npx --yes skills update amajorai/skills -y 2>/dev/null || true
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Phase 1: Explore

Spawn **2 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | Package.json scripts (test, lint, typecheck, build), existing CI config in `.github/` |
| 2 | Deploy platform config files (vercel.json, fly.toml, Dockerfile, etc.) |

Identify: which checks to run, how to run them, what secrets the deploy needs.


## Phase 2: Interview

Use `AskUserQuestion` for every question below — **one call per question**, not markdown. Ask questions one at a time and wait for each answer before proceeding.

**Question 1: CI checks** (multi-select; pre-select based on Phase 1 exploration):
| Value | Description |
|---|---|
| `lint` | Lint (e.g. ESLint, Biome) |
| `typecheck` | Type-check (tsc) |
| `test` | Unit tests |
| `e2e` | E2E tests (Playwright, Cypress) |

**Question 2: Preview deploys** (single select):
| Value | Description |
|---|---|
| `yes` | Yes — post a preview URL on every PR |
| `no` | No |

**Question 3: Production deploy** (single select):
| Value | Description |
|---|---|
| `yes` | Yes — auto-deploy to production on merge to main |
| `no` | No |

**Question 4: Monorepo** (single select):
| Value | Description |
|---|---|
| `no` | Single package |
| `yes` | Monorepo — specify which packages need CI |

After the monorepo question, ask as free text:
> What API keys or secrets does the deploy step need? (names only, e.g. "VERCEL_TOKEN, DATABASE_URL")


## Phase 3: CI Workflow (checks on every PR)

Create `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest
      - run: bun install --frozen-lockfile
      - run: bun run lint        # if exists
      - run: bun run typecheck   # if exists
      - run: bun run test        # if exists
      - run: bun run build       # always
```

**Important:** Only include the steps whose scripts actually exist in `package.json`. `bun run <script>` exits non-zero ("Script not found") if the script is missing, which would fail CI. From the Phase 1 exploration, delete the steps for any script the project does not define (do not rely on the `# if exists` comments to skip them).

Rules:
- Use `--frozen-lockfile` so lockfile drift fails CI
- Cache `~/.bun/install/cache` keyed on the lockfile, e.g. `hashFiles('**/bun.lock', '**/bun.lockb')` (Bun 1.2+ uses the text lockfile `bun.lock`; older projects use the binary `bun.lockb`)
- Fail fast: if build fails, no point running deploy


## Phase 4: Preview Deploy (optional, per PR)

**Vercel**:
```yaml
- uses: amondnet/vercel-action@v25
  with:
    vercel-token: ${{ secrets.VERCEL_TOKEN }}
    vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
    vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
```

**Cloudflare Pages**:
```yaml
- uses: cloudflare/wrangler-action@v3
  with:
    apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    command: pages deploy ./dist
```

**Fly.io / Dokploy**: deploy to a staging app with a unique name derived from the PR number.

Post the preview URL as a PR comment using `peter-evans/create-or-update-comment`.


## Phase 5: Production Deploy (on merge to main)

Create `.github/workflows/deploy.yml`. To guarantee the deploy only runs after CI passes, gate it on the CI workflow completing successfully via `workflow_run` (note: `needs:` only chains jobs within the *same* file, so it cannot reference the `check` job in `ci.yml`):

```yaml
name: Deploy
on:
  workflow_run:
    workflows: ["CI"]        # must match the `name:` in ci.yml
    types: [completed]
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    # Only deploy if the CI run succeeded
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest
      - run: bun install --frozen-lockfile
      - run: bun run build
      # Platform-specific deploy step here
```

Alternatively, keep the deploy job in the same `ci.yml` file and use `needs: check` so it runs only after the `check` job succeeds. Either way: never deploy a broken build.


## Phase 6: Secrets Setup

List all secrets needed and guide the user to add them:

1. Go to GitHub repo → Settings → Secrets and variables → Actions
2. Add each secret (names only, never values in code):
   - Platform tokens (VERCEL_TOKEN, FLY_API_TOKEN, etc.)
   - App secrets needed at build time
3. Reference in workflow as `${{ secrets.SECRET_NAME }}`


## Phase 7: Branch Protection

Recommend enabling branch protection on `main`:

- Require status checks to pass before merging
- Select the `check` job as a required status check
- Require at least 1 approval (optional for solo projects)
- Do not allow force pushes


## Phase 8: Verify

Push a PR and confirm:

- [ ] CI workflow triggers on PR open
- [ ] All check steps run and pass
- [ ] Preview deploy URL appears in PR comment (if configured)
- [ ] Merging to main triggers the deploy workflow
- [ ] Production deploy completes successfully
- [ ] A broken build (introduce a syntax error) correctly fails CI and blocks merge


## Completion Report

- Workflows created (list files)
- Checks configured (lint/typecheck/test/build)
- Preview deploys: yes/no
- Production deploy target and trigger
- Secrets needed (names only)
- Branch protection rules recommended
