# ⚡ A Major Skills

Simple, minimal, lean skills that people don't think about when shipping apps. Things we use at [A Major](https://amajor.ai).

[![Status](https://shieldcn.dev/badge/status-experimental-orange.svg)](https://github.com/amajorai/skills)
[![Stars](https://shieldcn.dev/github/stars/amajorai/skills.svg)](https://github.com/amajorai/skills)
[![Forks](https://shieldcn.dev/github/forks/amajorai/skills.svg)](https://github.com/amajorai/skills)
[![License](https://shieldcn.dev/github/license/amajorai/skills.svg)](https://github.com/amajorai/skills)
[![Issues](https://shieldcn.dev/github/issues/amajorai/skills.svg)](https://github.com/amajorai/skills/issues)

## Flagship

### 📦 [ship.md](https://github.com/amajorai/ship.md)

The end-to-end skill for shipping features without gaps. Up to 10 phases from interview to final verify. Wraps Claude Code's built-in `/batch`, `/goal`, and `/model` commands into a single quality-gated pipeline.

```bash
npx skills add amajorai/ship.md
```

### 🪅 [vibe.md](https://github.com/amajorai/vibe.md)

The end-to-end skill for spinning up a 24/7 production-ready full-stack dev and deploy environment. One interview, one clean pass: VPS provisioned, Bun installed, GitHub CLI wired, deployment platform running, and your project scaffolded and shipping.

```bash
npx skills add amajorai/vibe.md
```

## Skills

### Core

| Skill | What it does |
|-------|-------------|
| [`edge-cases`](skills/edge-cases/SKILL.md) | Discover and harden edge cases across 8 categories using parallel subagents |
| [`e2e`](skills/e2e/SKILL.md) | End-to-end test authoring and execution. Discovers flows, writes Playwright (web) or Maestro (iOS/Android/React Native/Flutter) tests, fixes failures |
| [`icons`](skills/icons/SKILL.md) | Generate app icons, favicons, and splash screens for Tauri, PWA, Capacitor, Expo, and Electron |
| [`legal-compliance`](skills/legal-compliance/SKILL.md) | Generate production-ready Privacy Policy, Terms, and DPA covering 20+ regulations |
| [`app-store-compliance`](skills/app-store-compliance/SKILL.md) | Audit against Apple App Store and Google Play Store review guidelines with a prioritized fix list |
| [`hardening`](skills/hardening/SKILL.md) | Harden a self-hosted Linux server. Detects state, runs a structured 5-question interview, then implements in one pass: SSH hardening (auto-randomized port), UFW + provider firewall (AWS Lightsail, EC2, Hetzner, DigitalOcean, OVH, or any other), fail2ban, kernel hardening, ClamAV, 2FA, and more. Uses subagent SSH verification instead of "open a new terminal" prompts. Includes rescue-mode docs for all providers. |
| [`youtube-to-skill`](skills/youtube-to-skill/SKILL.md) | Convert a YouTube video into a reusable Claude Code / Codex skill file |
| [`lighthouse`](skills/lighthouse/SKILL.md) | Audit with Lighthouse across performance, accessibility, best practices, and SEO. Fix and re-verify |
| [`seo`](skills/seo/SKILL.md) | Optimize for search. Meta tags, structured data, sitemap, robots.txt, Core Web Vitals, hreflang |
| [`aso`](skills/aso/SKILL.md) | Make a site agent-ready with Cloudflare AI Search, markdown negotiation, MCP card, and agent skills listing |
| [`better-t-stack`](skills/better-t-stack/SKILL.md) | Scaffold a Better T Stack project. Picks frontend, backend, database, ORM, auth, payments, and runs the CLI |
| [`distill-skill`](skills/distill-skill/SKILL.md) | Scan conversation history for repeated workflows and extract them into reusable skill files |
| [`mirror`](skills/mirror/SKILL.md) | Scan past transcripts for recurring patterns and improvement opportunities, then update CLAUDE.md or AGENTS.md |
| [`reflect`](skills/reflect/SKILL.md) | Reflect on the current conversation to extract corrections and learnings, then update CLAUDE.md or AGENTS.md |

### Others

| Skill | What it does |
|-------|-------------|
| [`payments`](skills/others/payments/SKILL.md) | Integrate Stripe, LemonSqueezy, or Polar.sh (web) or Superwall/RevenueCat (mobile). Products, webhooks, subscription portal, billing page, live checklist |
| [`auth`](skills/others/auth/SKILL.md) | Add authentication with Better Auth. OAuth, magic links, passkeys, sessions, route protection, and user model boilerplate |
| [`observability`](skills/others/observability/SKILL.md) | Set up Sentry error tracking, structured logging (pino), and uptime monitoring in one pass |
| [`analytics`](skills/others/analytics/SKILL.md) | Add product analytics with PostHog or Plausible. Event taxonomy, funnels, and a starter dashboard |
| [`email-transactional`](skills/others/email-transactional/SKILL.md) | Wire up Resend (CLI-driven), Postmark, useSend, or Plunk with templates for welcome, reset, notifications, and digest emails |
| [`launch-checklist`](skills/others/launch-checklist/SKILL.md) | Pre-launch audit. SSL, DNS, secrets, env vars, rate limits, backups, monitoring, and a go/no-go report |
| [`ci`](skills/others/ci/SKILL.md) | Set up GitHub Actions. Lint, typecheck, test, build, preview deploys on PRs, and production deploy on merge |
| [`og-images`](skills/others/og-images/SKILL.md) | Dynamic Open Graph images via edge function. No third-party service, cached, 1200x630 |
| [`waitlist`](skills/others/waitlist/SKILL.md) | Waitlist landing page with email capture, referral mechanics, position tracking, and welcome email |
| [`cookie-consent`](skills/others/cookie-consent/SKILL.md) | GDPR/CCPA cookie consent. Blocks non-essential scripts until consent, with a preferences modal |
| [`a11y`](skills/others/a11y/SKILL.md) | WCAG 2.2 AA audit and fixes. Keyboard nav, screen reader support, contrast, focus management |
| [`free-trial`](skills/others/free-trial/SKILL.md) | Add a time-boxed free trial. Expiry tracking, gating, reminder emails, and upgrade prompt flow |
| [`bundle-analysis`](skills/others/bundle-analysis/SKILL.md) | Analyze JS bundle size, identify bloat, implement code splitting, lazy loading, and dependency swaps |
| [`i18n`](skills/others/i18n/SKILL.md) | Add internationalization. Extract strings, locale routing, translation library, locale switcher |
| [`db-migrate`](skills/others/db-migrate/SKILL.md) | Run a production database migration safely. Dry-run, rollback plan, zero-downtime patterns, verification |
| [`load-test`](skills/others/load-test/SKILL.md) | Load test with k6. Baseline, spike, and soak scenarios; find throughput limits before users do |
| [`push-notifications`](skills/others/push-notifications/SKILL.md) | Web push (service worker + VAPID) or Expo mobile push. Preferences UI and server-side sending |
| [`context`](skills/others/context/SKILL.md) | Set up Context7 (live library docs MCP) and opensrc (real package source), plus a project CLAUDE.md |
| [`agent-quality`](skills/others/agent-quality/SKILL.md) | Set up react-doctor, react-scan, react-grab, expect, and agentation. Quality tools for AI-generated React code |

## Quickstart

```bash
npx skills add amajorai/skills
```

Installs all skills and automatically configures them for whichever coding agents you have installed (Claude Code, Codex, Cursor, and 50+ others).

Install a single skill:

```bash
npx skills add amajorai/skills/skills/hardening
```

### Auto-Update

Auto-update is **disabled by default**. Skills do not self-update unless you explicitly opt in — this prevents untrusted code from running automatically during a session (supply chain hygiene).

To update a skill on a single invocation, pass `--update`:

```
/lighthouse https://mysite.com --update
```

Or enable it project-wide in your CLAUDE.md:

```
SKILLS_AUTO_UPDATE: true
```

### Claude Code plugin

```
/plugin marketplace add amajorai/skills
/plugin install amajor-skills@amajorai
```

Invoked as `/amajor-skills:ship <task>`.

## Star History

<a href="https://www.star-history.com/#amajorai/skills&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=amajorai/skills&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=amajorai/skills&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=amajorai/skills&type=Date" />
 </picture>
</a>
