---
name: free-trial
description: Add a time-boxed free trial to any app with a payment integration. Handles trial start, expiry tracking, gating, expiry emails, and the upgrade prompt flow. Use when adding a trial period to an existing subscription or paywall.
argument-hint: <trial length in days, e.g. "14">
---

# Free Trial

You are implementing a free trial with proper expiry, gating, and upgrade flow. Works on top of an existing payment integration.

**Trial length:** {{args}} days


## Phase 1: Interview

Ask the user (combine related questions):

- **Scope**: Full product access during trial, or limited feature set?
- **Credit card required**: Require a card upfront (higher conversion to paid, lower trial starts) or no card (lower friction, more trials)?
- **Reminder emails**: Send reminders at trial start, midpoint, and 2 days before expiry?
- **Behavior at expiry**: Hard block (can't use app), soft block (read-only mode), or grace period?
- **Existing setup**: Is there already a payment integration (Stripe/LemonSqueezy)?


## Phase 2: Explore

Spawn **2 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | User model, subscription status fields, any existing plan gating |
| 2 | Auth flow, onboarding steps, where trial would start |


## Phase 3: Database Changes

Add trial fields to the user model:

```sql
ALTER TABLE users ADD COLUMN trial_started_at TIMESTAMPTZ;
ALTER TABLE users ADD COLUMN trial_ends_at    TIMESTAMPTZ;
ALTER TABLE users ADD COLUMN trial_expired     BOOLEAN DEFAULT false;
```

Or use Stripe's built-in trial support: when creating a subscription, set `trial_end: Math.floor(Date.now() / 1000) + 14 * 86400`. Stripe will handle the trial period and send a `customer.subscription.trial_will_end` webhook 3 days before expiry.


## Phase 4: Trial Start

Trigger trial start at the right moment (signup or first meaningful action):

1. Set `trial_started_at = now()`, `trial_ends_at = now() + N days`
2. If using Stripe: create a subscription with a trial period (no charge until trial ends)
3. If no card required: just set the database fields: prompt for card when trial expires
4. Send welcome email with trial end date prominently displayed


## Phase 5: Access Gating

Create a `getAccessLevel(user)` utility that returns one of:
- `'full'` - active paid subscriber
- `'trial'` - within trial period
- `'expired'` - trial ended, no payment
- `'free'` - on free plan (if applicable)

The middleware/guard checks this, not raw dates:

```typescript
if (['full', 'trial'].includes(getAccessLevel(user))) {
  // allow
} else {
  // redirect to /upgrade
}
```

Update all existing plan-gated routes to use `getAccessLevel`.


## Phase 6: Trial Expiry

Run a background job (cron, every hour) that:
1. Finds users where `trial_ends_at < now()` and `trial_expired = false`
2. Sets `trial_expired = true`
3. Sends the trial-expired email with a direct upgrade link

Alternatively use Stripe webhooks: `customer.subscription.trial_will_end` (3 days before) and `customer.subscription.updated` (when trial converts or cancels).


## Phase 7: Upgrade Prompt

Build an upgrade prompt component shown when a trial user hits a paywall:

1. Show days remaining in the trial (or "your trial has ended")
2. List what they'll lose access to
3. Clear CTA: "Upgrade to [Plan]: $X/mo"
4. Secondary: "Remind me later" (snooze for 24h, not available at expiry)

Show a persistent banner in the header for the last 3 days of trial.


## Phase 8: Emails

Send these emails automatically:

| Trigger | Subject | Content |
|---------|---------|---------|
| Trial starts | "Your 14-day trial has started" | Features, trial end date, support link |
| 7 days before expiry | "Your trial ends in 7 days" | What you've built, upgrade CTA |
| 2 days before expiry | "2 days left in your trial" | Urgency, upgrade CTA |
| Trial expired | "Your trial has ended" | What you lose, upgrade CTA, FAQ |


## Phase 9: Verify

- [ ] New user gets trial fields set on signup
- [ ] All gated features accessible during trial
- [ ] Trial expiry check runs and sets `trial_expired = true`
- [ ] Expired user sees upgrade prompt, not the gated feature
- [ ] Trial reminder emails send at correct intervals
- [ ] Upgrading during trial converts correctly (immediate access, no double charge)
- [ ] Trial expiry does not affect already-paying users


## Completion Report

- Trial length and scope configured
- Database fields added
- `getAccessLevel` utility created
- Gated routes updated
- Expiry job configured
- Emails wired up (list triggers)
- Upgrade prompt location(s)
