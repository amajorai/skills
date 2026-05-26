---
name: better-t-stack
description: Scaffolds a Better T Stack project via a structured interview. Uses AskUserQuestion to let the user pick frontend, backend, runtime, database, ORM, API layer, auth, payments, addons, example templates, and where to create the project. Assembles and runs the exact create-better-t-stack CLI command with no manual prompts.
argument-hint: [project name]
---

# Better T Stack

You are scaffolding a Better T Stack project. Run a structured interview to lock in all choices, then build and execute the `bun create better-t-stack@latest` command in one clean pass.

**Project name (if provided):** {{args}}


## Phase 1: Project Basics

Use `AskUserQuestion` for every question below — **one call per question**, not markdown. Ask questions one at a time and wait for each answer before proceeding.

**Question 1: Project name** (skip if {{args}} is set):
> What is your project name?

**Question 2: Where to create it:**
> Where should the project be created? Provide a full absolute path (e.g. `C:\Code`, `/home/user/projects`, `~/projects`). Leave blank to use the current directory.

**Question 3: Package manager** (single select):
Options: `bun` (Recommended: fastest), `npm`, `pnpm`

**Question 4a: Git init** (single select):
- Initialize a git repository? Yes / No

**Question 4b: Auto-install** (single select):
- Install dependencies automatically? Yes / No


## Phase 2: Stack Interview

Use `AskUserQuestion` for every question below — **one call per question**. Use `multiSelect: true` for array options.

### Batch A: Frontend & Backend

**Question: Frontend framework** (multi-select):
| Value | Description |
|---|---|
| `tanstack-router` | React + TanStack Router: lightweight SPA (Recommended) |
| `react-router` | React + React Router v7 |
| `tanstack-start` | React + TanStack Start: SSR/SSG |
| `next` | Next.js: full-stack React |
| `nuxt` | Nuxt: Vue full-stack |
| `svelte` | SvelteKit |
| `solid` | SolidStart |
| `astro` | Astro: content/hybrid |
| `native-bare` | React Native (bare) |
| `native-uniwind` | React Native + NativeWind (Tailwind) |
| `native-unistyles` | React Native + Unistyles |
| `none` | No frontend |

**Question: Backend framework** (single select):
| Value | Description |
|---|---|
| `hono` | Hono: ultra-fast, edge-ready (Recommended) |
| `express` | Express: classic Node.js |
| `fastify` | Fastify: fast Node.js |
| `elysia` | Elysia: Bun-native |
| `convex` | Convex: reactive backend-as-a-service |
| `self` | Self-hosted / fullstack (no separate backend server) |
| `none` | No backend |

**Question: Runtime** (single select):
| Value | Description |
|---|---|
| `bun` | Bun: fastest JS runtime (Recommended) |
| `node` | Node.js |
| `workers` | Cloudflare Workers: edge runtime |

**Question: API layer** (single select):
| Value | Description |
|---|---|
| `trpc` | tRPC: end-to-end type-safe RPC (Recommended) |
| `orpc` | oRPC: alternative type-safe RPC |
| `none` | No API layer (REST or custom) |


### Batch B: Data Layer

**Question: Database** (single select):
| Value | Description |
|---|---|
| `sqlite` | SQLite / LibSQL: lightweight, zero-config (Recommended) |
| `postgres` | PostgreSQL |
| `mysql` | MySQL |
| `mongodb` | MongoDB |
| `none` | No database |

**Question: ORM** (single select: show only compatible options based on DB choice):
| Value | Compatible with |
|---|---|
| `drizzle` | SQLite, PostgreSQL, MySQL (Recommended) |
| `prisma` | PostgreSQL, MySQL, MongoDB, SQLite |
| `mongoose` | MongoDB only |
| `none` | No ORM |

**Question: Database setup / hosting** (single select: skip if database is `none`):
| Value | Description |
|---|---|
| `turso` | Turso: hosted LibSQL (best for SQLite) |
| `neon` | Neon: serverless PostgreSQL |
| `supabase` | Supabase: PostgreSQL + extras |
| `prisma-postgres` | Prisma Postgres: managed |
| `mongodb-atlas` | MongoDB Atlas: hosted MongoDB |
| `d1` | Cloudflare D1: edge SQLite |
| `docker` | Local Docker: self-hosted |
| `none` | Manual / skip |


### Batch C: Auth, Payments & Extras

**Question: Authentication** (single select):
| Value | Description |
|---|---|
| `better-auth` | Better Auth: self-hosted, full-featured (Recommended) |
| `clerk` | Clerk: managed auth with UI components |
| `none` | No auth |

**Question: Payments** (single select):
| Value | Description |
|---|---|
| `polar` | Polar: open-source payments & billing |
| `none` | No payments |

**Question: Web deployment target** (single select):
| Value | Description |
|---|---|
| `cloudflare` | Cloudflare Workers: edge deployment |
| `none` | No web deploy config |

**Question: Example template** (single select):
| Value | Description |
|---|---|
| `none` | Start from scratch (Recommended) |
| `todo` | Todo app template |
| `ai` | AI app template |


### Batch D: Addons

**Question: Addons** (multi-select: pick any):
| Value | Description |
|---|---|
| `biome` | Biome: fast linter + formatter (Recommended) |
| `turborepo` | Turborepo: monorepo build system |
| `nx` | Nx: monorepo tooling |
| `pwa` | PWA: Progressive Web App support |
| `tauri` | Tauri: cross-platform desktop app |
| `electrobun` | Electrobun: Bun-native desktop app |
| `lefthook` | Lefthook: git hooks manager |
| `husky` | Husky: git hooks (alternative to lefthook) |
| `starlight` | Starlight: Astro-based docs site |
| `fumadocs` | Fumadocs: Next.js docs framework |
| `ultracite` | Ultracite: opinionated config preset |
| `oxlint` | OxLint: Rust-based linter |
| `mcp` | MCP: Model Context Protocol server |
| `opentui` | OpenTUI: terminal UI framework |
| `wxt` | WXT: browser extension framework |
| `none` | No addons |


## Phase 3: Assemble the Command

After all answers are collected, assemble the command. Show it to the user for confirmation before running.

### Command template

```
bun create better-t-stack@latest <project-name> \
  [--frontend <val> <val>...] \
  [--backend <val>] \
  [--runtime <val>] \
  [--database <val>] \
  [--orm <val>] \
  [--api <val>] \
  [--auth <val>] \
  [--payments <val>] \
  [--addons <val> <val>...] \
  [--examples <val>] \
  [--package-manager <val>] \
  [--db-setup <val>] \
  [--web-deploy <val>] \
  (--git | --no-git) \
  (--install | --no-install)
```

### Assembly rules

- **Omit** any flag where the user chose `none` or skipped (except boolean flags below — pass those explicitly to avoid interactive prompts)
- **Array flags** (`--frontend`, `--addons`): list each value as a separate space-separated argument after the flag
- **Boolean flags**: always pass explicitly so the CLI never prompts. Use `--git` if git init is yes, `--no-git` if no. Use `--install` if auto-install is yes, `--no-install` if no.
- **Directory**: if a custom path was given, `cd` there before running the command

### Example assembled command

```bash
cd ~/projects
bun create better-t-stack@latest my-app \
  --frontend tanstack-router \
  --backend hono \
  --runtime bun \
  --database sqlite \
  --orm drizzle \
  --api trpc \
  --auth better-auth \
  --addons biome turborepo \
  --package-manager bun \
  --db-setup turso \
  --git \
  --install
```

Display the exact command you're about to run. Ask: "Run this command?"


## Phase 4: Execute

1. `cd` to the specified directory (if provided and not CWD)
2. Run the assembled `bun create better-t-stack@latest` command
3. After scaffolding completes, `cd` into the new project directory
4. Respect the install choice: if the user chose to auto-install (`--install`), dependencies are already installed; if they chose not to (`--no-install`), leave them uninstalled and remind the user to run `bun install` before starting
5. Print the dev start command:

```
cd <project-name>
bun dev
```


## Completion Checklist

- [ ] Project name confirmed
- [ ] Target directory confirmed and navigated to
- [ ] All stack choices captured via interview
- [ ] Assembled command shown and confirmed
- [ ] Project scaffolded successfully
- [ ] Dependencies installed (or user reminded to run `bun install` if they declined auto-install)
- [ ] Dev start command shown to user
