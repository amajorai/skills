# ⚡ A Major Skills

Simple, minimal, lean skills that people don't think about when shipping apps. Things we use at [A Major](https://amajor.ai).

[![Stars](https://shieldcn.dev/github/stars/amajorai/skills.svg)](https://github.com/amajorai/skills)
[![Forks](https://shieldcn.dev/github/forks/amajorai/skills.svg)](https://github.com/amajorai/skills)
[![License](https://shieldcn.dev/github/license/amajorai/skills.svg)](https://github.com/amajorai/skills)
[![Issues](https://shieldcn.dev/github/issues/amajorai/skills.svg)](https://github.com/amajorai/skills/issues)

> [!NOTE]
> These skills have been built and tested with **Claude Code**. Codex support is untested. If you try them on Codex, we'd love your help. [Open an issue](https://github.com/amajorai/skills/issues) to share what works and what doesn't.

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

### 🎉 [party.md](https://github.com/amajorai/party.md)

The 24/7 autonomous build agent. Use GitHub issues and a Projects kanban board as your interface — drop in issues, party.md picks them up, delegates to `/ship`, opens PRs, and moves cards automatically. Works on a server, a Pi, or GitHub Actions while you sleep.

```bash
npx skills add amajorai/party.md
```

### 🎬 [replay.md](https://github.com/amajorai/replay.md)

Record a live video of your running app and share the link — straight from chat. Detects your environment, lets you choose a recording approach (Playwright, VNC + ffmpeg, or Computer Use API) and a storage provider (Cloudflare R2, Hetzner, YouTube, or local). Auto-detects a vibe.md server for zero-setup cloud recording.

```bash
npx skills add amajorai/replay.md
```

## Skills

### Core

| Skill | Status | What it does |
|-------|--------|-------------|
| [`edge-cases`](skills/edge-cases/SKILL.md) | ![beta](https://shieldcn.dev/badge/status-beta-blue.svg) | Discover and harden edge cases across 8 categories using parallel subagents |
| [`e2e`](skills/e2e/SKILL.md) | ![beta](https://shieldcn.dev/badge/status-beta-blue.svg) | End-to-end test authoring and execution. Discovers flows, uses agent-browser (web), Playwright (desktop/complex web), or Maestro (iOS/Android/React Native/Flutter), writes tests, fixes failures |
| [`icons`](skills/icons/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Generate app icons, favicons, and splash screens for Tauri, PWA, Capacitor, Expo, and Electron |
| [`legal-compliance`](skills/legal-compliance/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Generate production-ready Privacy Policy, Terms, and DPA covering 20+ regulations |
| [`app-store-compliance`](skills/app-store-compliance/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Audit against Apple App Store and Google Play Store review guidelines with a prioritized fix list |
| [`hardening`](skills/hardening/SKILL.md) | ![beta](https://shieldcn.dev/badge/status-beta-blue.svg) | Harden a self-hosted Linux server. Detects state, runs a structured 5-question interview, then implements in one pass: SSH hardening (auto-randomized port), UFW + provider firewall (AWS Lightsail, EC2, Hetzner, DigitalOcean, OVH, or any other), fail2ban, kernel hardening, ClamAV, 2FA, and more. Uses subagent SSH verification instead of "open a new terminal" prompts. Includes rescue-mode docs for all providers. |
| [`youtube-to-skill`](skills/youtube-to-skill/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Convert a YouTube video into a reusable Claude Code / Codex skill file |
| [`lighthouse`](skills/lighthouse/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Audit with Lighthouse across performance, accessibility, best practices, and SEO. Fix and re-verify |
| [`seo`](skills/seo/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Optimize for search. Meta tags, structured data, sitemap, robots.txt, Core Web Vitals, hreflang |
| [`aso`](skills/aso/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Make a site agent-ready with Cloudflare AI Search, markdown negotiation, MCP card, and agent skills listing |
| [`better-t-stack`](skills/better-t-stack/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Scaffold a Better T Stack project. Picks frontend, backend, database, ORM, auth, payments, and runs the CLI |
| [`distill-skill`](skills/distill-skill/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Scan conversation history for repeated workflows and extract them into reusable skill files |
| [`mirror`](skills/mirror/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Scan past transcripts for recurring patterns and improvement opportunities, then update CLAUDE.md or AGENTS.md |
| [`reflect`](skills/reflect/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Reflect on the current conversation to extract corrections and learnings, then update CLAUDE.md or AGENTS.md |

### Security

| Skill | Status | What it does |
|-------|--------|-------------|
| [`owasp-web-top10`](skills/owasp-web-top10/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Audit a web application against OWASP Top 10:2025. Parallel subagents per category, prioritized findings report, optional fixes |
| [`llm-top10`](skills/llm-top10/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Audit an LLM-powered application against OWASP LLM Top 10. Covers prompt injection, data leakage, supply chain, excessive agency, and more |
| [`ai-agent-security`](skills/ai-agent-security/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Implement or audit AI agent security controls across 9 pillars: tool least-privilege, input validation, memory security, HITL, output guardrails, monitoring, multi-agent trust, data protection, adversarial testing |
| [`prompt-injection`](skills/prompt-injection/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Audit and fix LLM prompt injection vulnerabilities. Covers direct, indirect, encoding, typoglycemia, Best-of-N, RAG poisoning, and agent-specific attacks |
| [`agentic-skills-top10`](skills/agentic-skills-top10/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Review AI agent skills/plugins against OWASP Agentic Skills Top 10. Covers malicious skills, supply chain, over-privilege, insecure metadata, and governance |
| [`agentic-ai-threats`](skills/agentic-ai-threats/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Threat model an agentic AI system against OWASP Agentic AI Threats and Mitigations. Produces STRIDE-style threat model with mitigations and implementation roadmap |

### Others

| Skill | Status | What it does |
|-------|--------|-------------|
| [`payments`](skills/others/payments/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Integrate Stripe, LemonSqueezy, or Polar.sh (web) or Superwall/RevenueCat (mobile). Products, webhooks, subscription portal, billing page, live checklist |
| [`auth`](skills/others/auth/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Add authentication with Better Auth. OAuth, magic links, passkeys, sessions, route protection, and user model boilerplate |
| [`observability`](skills/others/observability/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Set up Sentry error tracking, structured logging (pino), and uptime monitoring in one pass |
| [`analytics`](skills/others/analytics/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Add product analytics with PostHog or Plausible. Event taxonomy, funnels, and a starter dashboard |
| [`email-transactional`](skills/others/email-transactional/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Wire up Resend (CLI-driven), Postmark, useSend, or Plunk with templates for welcome, reset, notifications, and digest emails |
| [`launch-checklist`](skills/others/launch-checklist/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Pre-launch audit. SSL, DNS, secrets, env vars, rate limits, backups, monitoring, and a go/no-go report |
| [`ci`](skills/others/ci/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Set up GitHub Actions. Lint, typecheck, test, build, preview deploys on PRs, and production deploy on merge |
| [`og-images`](skills/others/og-images/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Dynamic Open Graph images via edge function. No third-party service, cached, 1200x630 |
| [`waitlist`](skills/others/waitlist/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Waitlist landing page with email capture, referral mechanics, position tracking, and welcome email |
| [`cookie-consent`](skills/others/cookie-consent/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | GDPR/CCPA cookie consent. Blocks non-essential scripts until consent, with a preferences modal |
| [`a11y`](skills/others/a11y/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | WCAG 2.2 AA audit and fixes. Keyboard nav, screen reader support, contrast, focus management |
| [`free-trial`](skills/others/free-trial/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Add a time-boxed free trial. Expiry tracking, gating, reminder emails, and upgrade prompt flow |
| [`bundle-analysis`](skills/others/bundle-analysis/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Analyze JS bundle size, identify bloat, implement code splitting, lazy loading, and dependency swaps |
| [`i18n`](skills/others/i18n/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Add internationalization. Extract strings, locale routing, translation library, locale switcher |
| [`db-migrate`](skills/others/db-migrate/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Run a production database migration safely. Dry-run, rollback plan, zero-downtime patterns, verification |
| [`load-test`](skills/others/load-test/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Load test with k6. Baseline, spike, and soak scenarios; find throughput limits before users do |
| [`push-notifications`](skills/others/push-notifications/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Web push (service worker + VAPID) or Expo mobile push. Preferences UI and server-side sending |
| [`context`](skills/others/context/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Set up Context7 (live library docs MCP) and opensrc (real package source), plus a project CLAUDE.md |
| [`agent-quality`](skills/others/agent-quality/SKILL.md) | ![experimental](https://shieldcn.dev/badge/status-experimental-orange.svg) | Set up react-doctor, react-scan, react-grab, expect, and agentation. Quality tools for AI-generated React code |

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
