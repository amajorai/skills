---
name: observability
description: Set up error tracking, structured logging, and uptime monitoring for any app. Wires up Sentry (errors), structured logs with context, and an uptime monitor (BetterStack or UptimeRobot). Use when an app goes to production without visibility into what breaks.
argument-hint: <app type: web | api | mobile | all>
---

# Observability

You are setting up production observability. Work through each phase in order.

**Target:** {{args}}


## Phase 1: Interview

Use `AskUserQuestion` for every question below — **one call per question**, not markdown. Ask questions one at a time and wait for each answer before proceeding.

**Question 1: Error tracking** (single select):
| Value | Description |
|---|---|
| `sentry` | Sentry (default) |
| `highlight` | Highlight.io |
| `bugsnag` | Bugsnag |

**Question 2: Log sink** (single select):
| Value | Description |
|---|---|
| `console` | Console only (dev / simple setups) |
| `betterstack-logs` | BetterStack Logs |
| `axiom` | Axiom |
| `file` | File-based logging |

**Question 3: Uptime monitoring** (single select):
| Value | Description |
|---|---|
| `betterstack` | BetterStack (default) |
| `uptimerobot` | UptimeRobot |

**Question 4: Alerting destination** (multi-select):
| Value | Description |
|---|---|
| `email` | Email |
| `slack` | Slack |
| `pagerduty` | PagerDuty |

After alerting, ask as free text:
> What is your frontend framework, backend runtime, and do you have any background jobs?

Confirm tools and alert destinations before proceeding.


## Phase 2: Explore

Spawn **2 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | Existing logging, error handling, try/catch patterns across the codebase |
| 2 | Entry points: server bootstrap, client root, API route structure |

Identify: where to initialize SDKs, what's unhandled, what context (user ID, request ID) is available.


## Phase 3: Error Tracking (Sentry)

Detect the package manager:
```bash
command -v bun >/dev/null 2>&1 && PM=bun || (command -v pnpm >/dev/null 2>&1 && PM=pnpm || PM=npm)
```

1. Install: `$PM add @sentry/node @sentry/browser` (or framework-specific: `@sentry/nextjs`, `@sentry/react`)
2. Initialize at the earliest possible entry point with:
   - DSN from env var (`SENTRY_DSN`)
   - Environment tag (`production` / `staging`)
   - Release identifier (use git SHA: `process.env.COMMIT_SHA`)
3. Attach user context on login: `Sentry.setUser({ id, email })`
4. Capture unhandled promise rejections and uncaught exceptions
5. For API routes: wrap handlers with Sentry request tracing
6. Verify: trigger a test error, confirm it appears in Sentry dashboard


## Phase 4: Structured Logging

1. Install: `$PM add pino` (default) or use existing logger if present
2. Create a shared logger instance with:
   - JSON output in production, pretty-print in development
   - Default fields: `service`, `env`, `version`
3. Replace bare `console.log` / `console.error` calls throughout the codebase
4. Add request context middleware (request ID, user ID, route) for API layers
5. Log levels: `error` for exceptions, `warn` for degraded state, `info` for key events, `debug` for dev only
6. Never log sensitive fields: add a redact list to the logger config for passwords, tokens, PII


## Phase 5: Uptime Monitoring

1. Identify all public endpoints to monitor (at minimum: root `/`, health check `/health`)
2. Create a lightweight health endpoint if one doesn't exist:
   ```
   GET /health → { status: "ok", version: "...", uptime: ... }
   ```
3. Configure BetterStack or UptimeRobot:
   - Check interval: 1 minute
   - Alert after: 2 consecutive failures
   - Alert destination: the channel confirmed in Phase 1
4. Add a status page if the user wants one (BetterStack provides this free)


## Phase 6: Alerting Rules

Set up alert fatigue prevention:

- **Critical** (page immediately): service completely down, error rate > 10% in 5 min
- **Warning** (Slack/email): error spike, latency P95 > threshold, disk/memory pressure
- **Noise suppression**: ignore known flaky endpoints, bot traffic, health check errors


## Phase 7: Verify

- [ ] Sentry receives a test exception from production or staging
- [ ] Structured logs appear in the log sink with correct fields
- [ ] Health endpoint responds correctly
- [ ] Uptime monitor fires an alert when the endpoint is manually taken down (test mode)
- [ ] Alert reaches the correct destination


## Completion Report

- Error tracking: SDK installed, initialized, user context attached
- Logging: logger configured, console calls replaced, sensitive fields redacted
- Uptime: endpoints monitored, health route created, alert channel confirmed
- Env vars required (names only)
