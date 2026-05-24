---
name: email-transactional
description: Wire up transactional email for any app using Resend, Postmark, useSend, or Plunk. Sets up templates for welcome, password reset, notifications, and digests. Handles domain verification, deliverability, and unsubscribe. Use when the app needs to send emails to users.
argument-hint: <email types needed: welcome | reset | notifications | digest | all>
---

# Email Transactional

You are wiring up transactional email end-to-end. Work through each phase in order.

**Email types:** {{args}}


## Phase 0: Auto-Update

*Skip unless `{{args}}` contains `--update`, or `SKILLS_AUTO_UPDATE: true` is set in your project CLAUDE.md.*

```bash
npx skills update email-transactional -y
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Phase 1: Interview

Ask the user (combine related questions):

- **Provider**: Which provider?
  - **Resend** (default, great DX, CLI-driven setup, hosted)
  - **Postmark** (high deliverability focus, hosted)
  - **useSend** (self-hosted, open-source, SES-backed: https://github.com/usesend/useSend)
  - **Plunk** (self-hosted, open-source, Docker: https://github.com/useplunk/plunk)
- **From address**: What sending domain? Is it already verified with the provider?
- **Email types**: Which of these are needed: welcome, email verification, password reset, notification, weekly digest, billing receipt, team invitation?
- **Templates**: Plain text only, or styled HTML? React Email or MJML for templating?
- **Stack**: Framework and language?


## Phase 2: Explore

Spawn **2 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | Auth flows (signup, password reset), notification trigger points in the codebase |
| 2 | Existing email code, env vars, any current email setup |


## Phase 3: Provider Setup

### Resend (CLI-driven)

```bash
# Install CLI
irm https://resend.com/install.ps1 | iex   # Windows
curl -fsSL https://resend.com/install.sh | bash  # macOS/Linux

# Authenticate
resend login

# Create and verify sending domain
resend domains create --name mail.yourdomain.com --region us-east-1
resend domains list   # get the domain ID
resend domains verify <id>

# Create an API key and copy it to .env
resend api-keys create --name "Production" --permission full_access
```

Install SDK: `bun add resend`

Set `RESEND_API_KEY` in env vars.

### Postmark

Install SDK: `bun add postmark`

1. Create a Postmark account and a Server
2. Add and verify your sending domain in the Postmark dashboard (SPF, DKIM records)
3. Set `POSTMARK_API_KEY` (Server API token) in env vars

### useSend (self-hosted)

Deploy via Docker or Railway (one-click): see https://docs.usesend.com

```bash
# After deploy, configure AWS SES credentials in the useSend dashboard
# Domain setup and DKIM/SPF are managed through the dashboard
```

Install SDK: `bun add usesend-js`  
Set `USESEND_API_KEY` and `USESEND_BASE_URL` in env vars.

### Plunk (self-hosted)

```bash
# Deploy via Docker
docker pull useplunk/plunk
# Follow https://docs.useplunk.com for full self-hosting setup
```

Install SDK: `bun add @plunk/node`  
Set `PLUNK_SECRET_KEY` and `PLUNK_BASE_URL` in env vars.

### DNS Records (all providers)

Guide the user to add these DNS records for the sending domain:
- **SPF**: `v=spf1 include:<provider-include> ~all`
- **DKIM**: TXT record provided by the provider dashboard
- **DMARC**: `v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com`


## Phase 4: Template Setup

Use React Email (default) for HTML templates:

```bash
bun add @react-email/components react-email
```

Create `emails/` directory with one file per template. Each template:
- Uses `<Html>`, `<Head>`, `<Body>`, `<Container>` from `@react-email/components`
- Is mobile-responsive
- Has a plain-text fallback
- Matches the app's brand (colors, fonts, logo)
- Includes a footer with unsubscribe link (required for bulk emails; best practice for all)


## Phase 5: Implement Email Functions

Create a shared `sendEmail(to, template, data)` utility that:
1. Renders the React Email template to HTML
2. Calls the provider API with from, to, subject, html, text
3. Logs the result (success or error) with the user ID
4. Never throws: catches errors and reports to error tracking

For each email type, create a typed wrapper:

```typescript
sendWelcomeEmail(user: { email, name })
sendPasswordResetEmail(user: { email }, resetUrl: string)
sendNotificationEmail(user: { email }, notification: Notification)
```

Wire each wrapper to the correct trigger point in the codebase.


## Phase 6: Deliverability

- [ ] SPF, DKIM, DMARC DNS records verified
- [ ] Sending from a subdomain (`mail.yourdomain.com`) not the root domain
- [ ] From name is recognizable: `"App Name <hello@mail.yourdomain.com>"`
- [ ] Subject lines are clear and not spammy (no ALL CAPS, no excessive punctuation)
- [ ] Unsubscribe link in every email (required by CAN-SPAM, GDPR)
- [ ] Test deliverability at [mail-tester.com](https://www.mail-tester.com): aim for 9+/10


## Phase 7: Verify

- [ ] Welcome email sends on new user signup (check inbox, not spam)
- [ ] Password reset email arrives within 30 seconds
- [ ] Email renders correctly in Gmail, Apple Mail, and Outlook (use Litmus or Email on Acid if available)
- [ ] Plain text fallback is readable
- [ ] Unsubscribe link works
- [ ] Bounced/failed sends are logged and don't crash the app


## Completion Report

- Provider configured and domain verified
- Templates created (list)
- Send functions wired to trigger points (list with file locations)
- Deliverability checks passed
- Env vars required (names only)
