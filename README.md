# A Major Skills

A collection of reusable AI coding agent skills for **Claude Code** and **Codex**. Each skill is a single unified `SKILL.md` file with common phases up top and platform-specific notes inline.

```
skills/
  <skill-name>/
    SKILL.md
.claude-plugin/
  plugin.json
  marketplace.json
install.sh
```

## Skills

### Core

| Skill | What it does |
|-------|-------------|
| [`ship`](skills/ship/SKILL.md) | Full-cycle workflow: interview, explore, plan (`/model opusplan`), implement (`/batch`), verify (`/goal`), edge cases, simplify, security, final verify |
| [`ship-simple`](skills/ship-simple/SKILL.md) | Quick implementation for simple features that don't need the full pipeline — no security review, edge cases, or simplify pass |
| [`vibe`](skills/vibe/SKILL.md) | Set up a full-stack dev and deployment environment — VPS, Bun, GitHub CLI, Dokploy or Coolify |
| [`edge-cases`](skills/edge-cases/SKILL.md) | Discover and harden edge cases across 8 categories using parallel subagents |
| [`e2e`](skills/e2e/SKILL.md) | End-to-end test authoring and execution — discovers flows, writes Playwright/Cypress tests, fixes failures |
| [`icons`](skills/icons/SKILL.md) | Generate app icons, favicons, and splash screens for Tauri, PWA, Capacitor, Expo, and Electron |
| [`legal-compliance`](skills/legal-compliance/SKILL.md) | Generate production-ready Privacy Policy, Terms, and DPA covering 20+ regulations |
| [`app-store-compliance`](skills/app-store-compliance/SKILL.md) | Audit against Apple App Store and Google Play Store review guidelines with a prioritized fix list |
| [`hardening`](skills/hardening/SKILL.md) | Harden a self-hosted Linux server — SSH, firewall, fail2ban, and optional 2FA |
| [`youtube-to-skill`](skills/youtube-to-skill/SKILL.md) | Convert a YouTube video into a reusable Claude Code / Codex skill file |
| [`lighthouse`](skills/lighthouse/SKILL.md) | Audit with Lighthouse across performance, accessibility, best practices, and SEO — then fix and re-verify |
| [`seo`](skills/seo/SKILL.md) | Optimize for search — meta tags, structured data, sitemap, robots.txt, Core Web Vitals, hreflang |
| [`cloudflare-ai-search`](skills/cloudflare-ai-search/SKILL.md) | Make a site agent-ready with Cloudflare AI Search, markdown negotiation, MCP card, and agent skills listing |

### Others

| Skill | What it does |
|-------|-------------|
| [`payments`](skills/others/payments/SKILL.md) | Integrate Stripe or LemonSqueezy — products, webhooks, subscription portal, billing page, live checklist |
| [`auth`](skills/others/auth/SKILL.md) | Add authentication — OAuth, magic links, sessions, route protection, and the full user model boilerplate |
| [`observability`](skills/others/observability/SKILL.md) | Set up Sentry error tracking, structured logging (pino), and uptime monitoring in one pass |
| [`analytics`](skills/others/analytics/SKILL.md) | Add product analytics with PostHog or Plausible — event taxonomy, funnels, and a starter dashboard |
| [`email-transactional`](skills/others/email-transactional/SKILL.md) | Wire up Resend or Postmark with templates for welcome, reset, notifications, and digest emails |
| [`launch-checklist`](skills/others/launch-checklist/SKILL.md) | Pre-launch audit — SSL, DNS, secrets, env vars, rate limits, backups, monitoring, and a go/no-go report |
| [`ci`](skills/others/ci/SKILL.md) | Set up GitHub Actions — lint, typecheck, test, build, preview deploys on PRs, and production deploy on merge |
| [`og-images`](skills/others/og-images/SKILL.md) | Dynamic Open Graph images via edge function — no third-party service, cached, 1200×630 |
| [`waitlist`](skills/others/waitlist/SKILL.md) | Waitlist landing page with email capture, referral mechanics, position tracking, and welcome email |
| [`cookie-consent`](skills/others/cookie-consent/SKILL.md) | GDPR/CCPA cookie consent — blocks non-essential scripts until consent, with a preferences modal |
| [`a11y`](skills/others/a11y/SKILL.md) | WCAG 2.2 AA audit and fixes — keyboard nav, screen reader support, contrast, focus management |
| [`free-trial`](skills/others/free-trial/SKILL.md) | Add a time-boxed free trial — expiry tracking, gating, reminder emails, and upgrade prompt flow |
| [`bundle-analysis`](skills/others/bundle-analysis/SKILL.md) | Analyze JS bundle size, identify bloat, implement code splitting, lazy loading, and dependency swaps |
| [`i18n`](skills/others/i18n/SKILL.md) | Add internationalization — extract strings, locale routing, translation library, locale switcher |
| [`db-migrate`](skills/others/db-migrate/SKILL.md) | Run a production database migration safely — dry-run, rollback plan, zero-downtime patterns, verification |
| [`load-test`](skills/others/load-test/SKILL.md) | Load test with k6 — baseline, spike, and soak scenarios; find throughput limits before users do |
| [`push-notifications`](skills/others/push-notifications/SKILL.md) | Web push (service worker + VAPID) or Expo mobile push — with preferences UI and server-side sending |
| [`context-tools`](skills/others/context-tools/SKILL.md) | Set up Context7 (live library docs MCP) and opensrc (real package source), plus a project CLAUDE.md |
| [`agent-quality`](skills/others/agent-quality/SKILL.md) | Set up react-doctor, react-scan, react-grab, expect, and agentation — quality tools for AI-generated React code |

## Installation

### skills.sh (recommended)

```bash
npx skills add amajorai/skills
```

Installs all skills and automatically configures them for whichever coding agents you have installed (Claude Code, Codex, Cursor, and 50+ others).

Install a single skill:

```bash
npx skills add amajorai/skills/skills/ship
```

### Claude Code plugin

```
/plugin marketplace add amajorai/skills
/plugin install amajor-skills@amajorai
```

Invoked as `/amajor-skills:ship <task>`.

### install.sh (one-liner)

```bash
curl -fsSL https://raw.githubusercontent.com/amajorai/skills/main/install.sh | bash
```

```bash
# Codex
curl -fsSL https://raw.githubusercontent.com/amajorai/skills/main/install.sh | bash -s -- --codex
```

Or clone and run manually:

```bash
git clone https://github.com/amajorai/skills.git
cd skills

./install.sh           # Claude Code, copies to ~/.claude/skills/, invoke as /ship
./install.sh --codex   # Codex, copies to ~/.codex/skills/, invoke as $ship
```

### Copy a single skill

```bash
# Claude Code
cp skills/ship/SKILL.md ~/.claude/skills/ship.md

# Codex
mkdir -p ~/.codex/skills/ship && cp skills/ship/SKILL.md ~/.codex/skills/ship/SKILL.md
```
