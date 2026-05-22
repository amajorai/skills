---
name: auth
description: Add authentication to any web or mobile app using Better Auth. Sets up OAuth (Google/GitHub/Discord), magic links, passkeys, session management, and route protection. Handles the full boilerplate — signup, login, logout, protected routes, and user model. Use when starting a new app or adding auth to an existing one.
argument-hint: <auth methods: oauth | magic-link | password | passkeys | all>
---

# auth — Authentication with Better Auth

You are implementing a complete auth system using [Better Auth](https://better-auth.com). Docs: https://better-auth.com/docs

**Auth methods:** {{args}}


## Setup: Better Auth MCP

Before starting, add the Better Auth MCP so you have live, accurate docs throughout:

```json
{
  "mcpServers": {
    "better-auth": {
      "command": "npx",
      "args": ["-y", "better-auth-mcp"]
    }
  }
}
```

Add to `.claude/settings.json` (project) or `~/.claude/settings.json` (global), then restart Claude Code.


## Phase 1: Interview

Ask the user (combine related questions):

- **Methods**: OAuth (which providers — Google, GitHub, Discord?), magic link, username/password, or passkeys?
- **Stack**: Framework, database, ORM?
- **User model**: What fields beyond email/name are needed? Roles? Teams/orgs?
- **Session strategy**: Cookie-based sessions or JWT tokens? Expiry preferences?


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

```bash
bun add better-auth
```

Reference the Better Auth MCP and docs at https://better-auth.com/docs throughout.

1. Create `lib/auth.ts` — configure providers, session strategy, database adapter
2. Create the catch-all API route: `app/api/auth/[...all]/route.ts`
3. Run schema generation: `bunx better-auth generate` and apply migration
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

1. `/login` — provider buttons, magic link input, or password form
2. `/signup` — same as login if using OAuth/magic link; separate form if password
3. Post-auth redirect to the intended destination (save `?next=` param)


## Phase 5: Security Hardening

- [ ] Session tokens are cryptographically random (not predictable IDs)
- [ ] Sessions expire (default: 30 days, shorter for sensitive apps)
- [ ] CSRF protection on auth endpoints (Better Auth handles this; verify if custom)
- [ ] OAuth state parameter validated to prevent CSRF
- [ ] Rate limit: max 10 login attempts per IP per 15 min
- [ ] Secure + HttpOnly + SameSite=Lax cookies for session tokens
- [ ] No user enumeration — same error message for "user not found" and "wrong password"


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
