---
name: launch-checklist
description: Pre-launch audit for any web app. Checks SSL, DNS, env vars, secrets, rate limits, backups, error tracking, analytics, performance, and staging/prod parity. Produces a prioritized fix list. Use the day before going live.
argument-hint: <app URL or domain>
---

# Launch Checklist

You are running a pre-launch audit. Work through each category systematically and produce a prioritized fix list.

**Target:** {{args}}


## Phase 1: Gather Context

Ask the user (one batch of questions):

- **URL**: What is the production domain?
- **Stack**: Hosting platform, framework, database?
- **Auth**: Is there user auth? What provider?
- **Payments**: Is there a payment integration? Test mode or live?
- **Team size**: Solo or team? (affects backup/runbook requirements)

Spawn **2 parallel subagents** to explore:

| Subagent | Focus |
|----------|-------|
| 1 | Env var files, config files, secrets in code (grep for hardcoded keys/tokens) |
| 2 | Package.json scripts, deployment config, infrastructure files |


## Phase 2: Security

- [ ] **HTTPS enforced** — HTTP redirects to HTTPS, HSTS header present
- [ ] **No secrets in git** — grep for `sk_live`, `api_key`, `password`, `token` in committed files; check `.gitignore` covers `.env`
- [ ] **Env vars set in production** — every `process.env.*` reference has a corresponding production value
- [ ] **Auth hardened** — session secrets are random and long, no default credentials
- [ ] **Rate limiting** — auth endpoints (login, signup, password reset) are rate-limited
- [ ] **CORS** — production origin whitelist is set, not `*`
- [ ] **CSP headers** — Content-Security-Policy set, at minimum `default-src 'self'`
- [ ] **Dependencies** — run `bun audit` or equivalent; no critical CVEs


## Phase 3: Infrastructure

- [ ] **DNS** — A/CNAME records point to production; TTL lowered before launch
- [ ] **SSL certificate** — valid, not expiring within 30 days, auto-renewal configured
- [ ] **Health endpoint** — `GET /health` returns 200 with status info
- [ ] **Error tracking** — Sentry or equivalent configured and receiving test events
- [ ] **Uptime monitoring** — at least one monitor watching the health endpoint
- [ ] **Log access** — production logs are accessible (not just local stdout)
- [ ] **Backups** — database has automated backups with tested restore procedure


## Phase 4: Application

- [ ] **Staging/prod parity** — staging environment exists and mirrors production config
- [ ] **Seed data removed** — no test accounts, demo data, or debug routes in production
- [ ] **Feature flags** — any in-development features are behind flags and disabled by default
- [ ] **Email sending** — transactional emails send correctly from a real domain (not localhost)
- [ ] **File uploads** — uploads go to cloud storage (S3/R2), not local disk
- [ ] **Cron jobs / workers** — background jobs are running and monitored
- [ ] **404 / error pages** — custom error pages exist and don't expose stack traces


## Phase 5: Performance

- [ ] **Page speed** — run Lighthouse; score > 70 on mobile
- [ ] **Images optimized** — no uncompressed images > 200KB; next-gen formats used
- [ ] **Bundle size** — JS bundle not egregiously large; code splitting in place
- [ ] **CDN** — static assets served from CDN, not the origin server
- [ ] **Database indexes** — queries on user-facing paths have appropriate indexes


## Phase 6: Legal & Compliance

- [ ] **Privacy policy** — exists and linked in footer
- [ ] **Terms of service** — exists and linked in footer
- [ ] **Cookie consent** — if using cookies beyond strictly necessary, banner present
- [ ] **GDPR/CCPA** — data deletion mechanism exists for user data


## Phase 7: Payments (if applicable)

- [ ] **Live mode keys** — Stripe/LemonSqueezy switched from test to live keys
- [ ] **Webhook registered** — production webhook endpoint registered in provider dashboard
- [ ] **Test purchase** — a real card charge completes end-to-end
- [ ] **Receipt emails** — customer receives a receipt after purchase


## Phase 8: Go / No-Go Report

Classify every finding:

- **BLOCKER** — must fix before launch (security holes, broken auth, no backups on user data)
- **HIGH** — fix in first week (missing monitoring, no error tracking, perf < 50)
- **MEDIUM** — fix in first month (legal pages missing, minor perf issues)
- **LOW** — nice to have (cosmetic, minor improvements)

Present the full list, highlight blockers, and confirm the user is ready to proceed.
