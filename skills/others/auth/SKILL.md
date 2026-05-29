---
name: auth
description: Add authentication to any web or mobile app using Better Auth. Sets up OAuth (Google/GitHub/Discord), magic links, passkeys, session management, and route protection. Handles the full boilerplate: signup, login, logout, protected routes, and user model. Use when starting a new app or adding auth to an existing one.
argument-hint: <auth methods: oauth | magic-link | password | passkeys | all>
---

# Auth

You are implementing a complete auth system using [Better Auth](https://better-auth.com). Docs: https://better-auth.com/docs

**Auth methods:** {{args}}


## Setup: Better Auth MCP

Before starting, add the Better Auth MCP so you have live, accurate docs throughout:

```bash
npx @better-auth/cli@latest mcp --claude-code
```

This command auto-configures the remote Better Auth docs MCP server in Claude Code. Restart Claude Code after running it.


## Phase 1: Interview

Use `AskUserQuestion` for every question below — **one call per question**, not markdown. Ask questions one at a time and wait for each answer before proceeding.

**Question 1: Auth methods** (multi-select):
| Value | Description |
|---|---|
| `oauth` | OAuth via social providers (Google, GitHub, Discord) |
| `magic-link` | Magic link (passwordless email) |
| `password` | Username / password |
| `passkeys` | Passkeys (WebAuthn) |

**Question 2: OAuth providers** (multi-select; skip if OAuth not selected):
| Value | Description |
|---|---|
| `google` | Google |
| `github` | GitHub |
| `discord` | Discord |

**Question 3: Stack** — ask as free text:
> What is your framework, database, and ORM? (e.g. "Next.js, PostgreSQL, Drizzle")

**Question 4: User model & session** (multi-select):
| Value | Description |
|---|---|
| `roles` | Role-based access control (admin / user / etc.) |
| `teams-orgs` | Teams or organizations per user |
| `jwt` | JWT tokens instead of cookie-based sessions |
| `custom-expiry` | Custom session expiry (shorter than default 30 days) |


## Phase 2: Explore

Spawn **2 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | Existing user model, database schema, ORM setup |
| 2 | Existing route structure, middleware patterns, protected pages |


## Phase 3: Plan

Define:
1. Database tables needed (users, sessions, accounts, verification tokens)
2. API routes (login, logout, callback, verify, me)
3. Middleware for protected routes
4. Frontend: login page, signup page, redirect-after-auth flow
5. Env vars needed (OAuth client ID/secret, session secret)

Confirm before implementing.


## Phase 4: Implement

Detect the package manager from the lockfile and install Better Auth:

- `bun.lockb` or `bun.lock` present - run `bun add better-auth`
- `pnpm-lock.yaml` present - run `pnpm add better-auth`
- otherwise - run `npm install better-auth`

The snippet below detects this automatically, but it is bash-only. On Windows run it via the Bash tool / Git Bash, or just run the matching install command directly:

```bash
command -v bun >/dev/null 2>&1 && PM=bun || (command -v pnpm >/dev/null 2>&1 && PM=pnpm || PM=npm)
$PM add better-auth
```

Reference the Better Auth MCP and docs at https://better-auth.com/docs throughout.

1. Create `lib/auth.ts` - configure providers, session strategy, database adapter
2. Create the catch-all API route: `app/api/auth/[...all]/route.ts`
3. Run schema generation: `npx @better-auth/cli generate` and apply migration
4. Create auth client: `lib/auth-client.ts` for frontend
5. Add session middleware that attaches `ctx.user` to every request

### OAuth setup (per provider)

- **Google**: Create OAuth app at console.cloud.google.com, get client ID + secret
- **GitHub**: Create OAuth app at github.com/settings/developers
- **Discord**: Create app at discord.com/developers/applications
- Set callback URL: `https://yourdomain.com/api/auth/callback/<provider>`

### Route protection

- **Server**: middleware checks session, redirects to `/login` if missing
- **Client**: `useSession()` hook, redirect in `useEffect` or use route guards
- **API routes**: return `401` if no valid session

### Auth pages

1. `/login` - provider buttons, magic link input, or password form
2. `/signup` - same as login if using OAuth/magic link; separate form if password
3. Post-auth redirect to the intended destination (save `?next=` param)


## Phase 5: Security Hardening

- [ ] Session tokens are cryptographically random (not predictable IDs)
- [ ] Sessions expire (default: 30 days, shorter for sensitive apps)
- [ ] CSRF protection on auth endpoints (Better Auth handles this; verify if custom)
- [ ] OAuth state parameter validated to prevent CSRF
- [ ] Rate limit: max 10 login attempts per IP per 15 min
- [ ] Secure + HttpOnly + SameSite=Lax cookies for session tokens
- [ ] No user enumeration: same error message for "user not found" and "wrong password"


## Phase 6: Verify

- [ ] Sign up with each configured provider works end-to-end
- [ ] Log out clears the session and redirects correctly
- [ ] Protected routes redirect to login when unauthenticated
- [ ] Protected routes are accessible after login
- [ ] Session persists across page refreshes
- [ ] OAuth callback handles errors gracefully (user cancels, invalid state)


## Completion Report

- Auth methods implemented (list)
- Database tables created
- Routes protected (list)
- Env vars required (names only)
