---
name: ci
description: Set up a GitHub Actions CI/CD pipeline for any project. Configures lint, typecheck, test, build, preview deploys on PRs, and production deploy on main. Use when setting up a new project or when CI is missing or broken.
argument-hint: <deploy target: vercel | fly | dokploy | cloudflare | none>
---

# CI

You are setting up a complete CI/CD pipeline with GitHub Actions. Work through each phase in order.

**Deploy target:** {{args}}


## Phase 0: Auto-Update

*Skip if `{{args}}` contains `--no-update`, or if `SKILLS_AUTO_UPDATE: false` is set in your project CLAUDE.md.*

```bash
npx skills update ci -y
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

Ask the user (combine related questions):

- **Checks**: Which of these exist: lint, typecheck, unit tests, E2E tests, build?
- **Preview deploys**: Should PRs get preview deploy URLs?
- **Deploy on merge**: Auto-deploy to production when PR merges to main?
- **Secrets needed**: What API keys does the deploy step need?
- **Monorepo**: Is this a monorepo? Which packages need CI?


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

Rules:
- Use `--frozen-lockfile` so lockfile drift fails CI
- Cache `~/.bun/install/cache` with a cache key on `bun.lockb`
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

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    needs: []   # reference the check job if in same file
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
      - run: bun install --frozen-lockfile
      - run: bun run build
      # Platform-specific deploy step here
```

Deploy only runs when checks pass. Never deploy a broken build.


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
