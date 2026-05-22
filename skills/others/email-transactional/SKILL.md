---
name: email-transactional
description: Wire up transactional email for any app using Resend or Postmark. Sets up templates for welcome, password reset, notifications, and digests. Handles domain verification, deliverability, and unsubscribe. Use when the app needs to send emails to users.
argument-hint: <email types needed: welcome | reset | notifications | digest | all>
---

# email-transactional — Transactional Email

You are wiring up transactional email end-to-end. Work through each phase in order.

**Email types:** {{args}}


## Phase 1: Interview

Ask the user (combine related questions):

- **Provider**: Resend (default, great DX) or Postmark (high deliverability focus)?
- **From address**: What sending domain? Is it already verified with the provider?
- **Email types**: Which of these are needed — welcome, email verification, password reset, notification, weekly digest, billing receipt, team invitation?
- **Templates**: Plain text only, or styled HTML? React Email or MJML for templating?
- **Stack**: Framework and language?


## Phase 2: Explore

Spawn **2 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | Auth flows (signup, password reset), notification trigger points in the codebase |
| 2 | Existing email code, env vars, any current email setup |


## Phase 3: Domain Verification

1. Install the SDK: `bun add resend` or `bun add postmark`
2. Guide the user to add DNS records (SPF, DKIM, DMARC) for the sending domain
   - SPF: `v=spf1 include:amazonses.com ~all` (or provider-specific)
   - DKIM: TXT record provided by the email provider
   - DMARC: `v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com`
3. Verify domain is confirmed in the provider dashboard before sending real emails
4. Set `RESEND_API_KEY` (or `POSTMARK_API_KEY`) in env vars


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
4. Never throws — catches errors and reports to error tracking

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
- [ ] Test deliverability at [mail-tester.com](https://www.mail-tester.com) — aim for 9+/10


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
