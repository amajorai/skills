---
name: payments
description: Integrate payments into any web or mobile app. Sets up Stripe, LemonSqueezy, or Polar.sh for web; Superwall or RevenueCat for mobile. Covers products, webhooks, subscription portal, billing page, and a test-to-live checklist. Use when adding a paywall, subscription plan, or one-time purchase to an app.
argument-hint: <billing model: subscription | one-time | usage-based>
---

# Payments

You are wiring up a complete payment integration. Work through each phase in order.

**Billing model:** {{args}}


## Phase 1: Interview

Ask the user (combine related questions):

- **Platform**: Web or mobile (iOS/Android)?
- **Provider**:
  - Web: Stripe, LemonSqueezy, or Polar.sh?
    - Stripe — direct card processing, full control, best ecosystem
    - LemonSqueezy — built-in VAT/tax handling, good for solo/EU projects
    - Polar.sh — open-source/developer-first, built-in sponsorships, benefits, and issue funding; great for OSS or dev tools
  - Mobile: Superwall or RevenueCat?
    - Superwall — dynamic paywalls configurable without deploys; iOS-first, Android in beta
    - RevenueCat — abstracts App Store + Google Play billing; best for cross-platform subscriptions and analytics
- **Products**: What plans/products exist? Prices, billing intervals, trial periods?
- **Gates**: Which features are paywalled? What happens when a user exceeds their plan?
- **Existing auth**: How are users identified? What is the user model (table/schema)?
- **Stack**: Framework, language, existing API layer?

Confirm the product catalog and gating rules before proceeding.


## Phase 2: Explore

Spawn **3 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | User model, auth flow, session handling |
| 2 | Existing API routes, middleware, webhook handling patterns |
| 3 | Frontend component patterns, routing, protected pages |

Synthesize: data model changes needed, API route plan, frontend gating strategy.


## Phase 3: Plan

Define the full implementation surface based on provider:

### Web (Stripe / LemonSqueezy / Polar.sh)

1. **Database changes** — add `customer_id`, `subscription_status`, `plan` fields to user model
   - Stripe: `stripe_customer_id`
   - LemonSqueezy: `lemon_customer_id`
   - Polar.sh: `polar_customer_id`
2. **API routes** — checkout session, billing portal, webhook handler
3. **Webhook events**:
   - Stripe: `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.deleted`, `invoice.payment_failed`
   - LemonSqueezy: `subscription_created`, `subscription_updated`, `subscription_cancelled`, `subscription_payment_failed`
   - Polar.sh: `subscription.created`, `subscription.updated`, `subscription.cancelled`, `order.created`
4. **Frontend** — pricing page, upgrade prompt, billing management page, plan-gated components
5. **Env vars** — keys needed in dev and prod

### Mobile (Superwall / RevenueCat)

1. **No backend billing routes needed** — App Store / Google Play handle the transaction
2. **RevenueCat**: configure entitlements and offerings in dashboard; sync user ID on login
3. **Superwall**: create paywall templates in dashboard; register triggers in code
4. **Database changes** — optionally mirror subscription state server-side via webhooks for backend gating
5. **Env vars** — SDK API keys per platform

Present the plan and confirm before implementing.


## Phase 4: Implement

### Stripe (Web)

1. Install: `bun add stripe`
2. Create customer on signup (or lazily on first checkout)
3. Checkout session endpoint — returns hosted checkout URL
4. Billing portal endpoint — returns portal URL for plan changes/cancellation
5. Webhook handler:
   - Verify signature: `stripe.webhooks.constructEvent`
   - Handle events, update user record in DB
   - Return `200` fast; do async work after acknowledging
6. Middleware to protect gated routes — check `subscription_status === 'active'`

### LemonSqueezy (Web)

1. Install: `bun add @lemonsqueezy/lemonsqueezy.js`
2. Create checkout via `createCheckout()` with variant ID
3. Webhook handler — verify with `X-Signature` header using HMAC-SHA256
4. Handle `subscription_created` / `subscription_updated` / `subscription_cancelled`
5. Store `lemon_customer_id` and `subscription_status` on user

### Polar.sh (Web)

1. Install: `bun add @polar-sh/sdk`
2. Create checkout session via `polar.checkouts.custom.create()`
3. Webhook handler — verify with `validateEvent()` from the SDK
4. Handle `subscription.created`, `subscription.updated`, `subscription.cancelled`, `order.created`
5. Store `polar_customer_id` and subscription state on user
6. Use Polar's customer portal URL for billing management
7. Optionally configure **Benefits** (e.g. license keys, Discord roles, downloads) in Polar dashboard — no extra code needed

### RevenueCat (Mobile)

1. Install: `bun add react-native-purchases` (or native pod/gradle)
2. Configure with `Purchases.configure({ apiKey })` on app launch
3. Identify user: `Purchases.logIn(userId)` after auth
4. Fetch offerings: `Purchases.getOfferings()` — display in paywall UI
5. Purchase: `Purchases.purchasePackage(package)`
6. Check entitlements: `customerInfo.entitlements.active['pro']` to gate features
7. Set up RevenueCat webhooks to mirror subscription state to your backend (optional but recommended)

### Superwall (Mobile)

1. Install: `bun add react-native-superwall` (or native SDK)
2. Configure: `Superwall.configure({ apiKey })` on app launch
3. Identify user: `Superwall.shared.identify(userId)` after auth
4. Register trigger: `Superwall.shared.register('campaign_trigger')` at paywall entry points
5. Handle purchase result via `SuperwallDelegate` or subscription handler
6. Paywalls are managed in Superwall dashboard — no app update needed to change copy/design/pricing

### Frontend (Web)

1. Pricing page with plan cards and CTA hitting checkout endpoint
2. Upgrade prompt component for paywalled features
3. Billing page (link to portal endpoint)
4. Show current plan in account settings


## Phase 5: Test Mode Verification

### Stripe

- [ ] Checkout flow completes with test card `4242 4242 4242 4242`
- [ ] Webhook fires and updates user record in DB
- [ ] Failed payment webhook (`4000 0000 0000 0341`) triggers correct state
- [ ] Use Stripe CLI: `stripe listen --forward-to localhost:3000/webhooks/stripe`

### LemonSqueezy

- [ ] Checkout completes in test mode (toggle in LemonSqueezy dashboard)
- [ ] Webhook fires and subscription state updated in DB

### Polar.sh

- [ ] Checkout completes in sandbox mode
- [ ] `order.created` and `subscription.created` webhooks fire correctly
- [ ] Benefits activated on purchase (if configured)
- [ ] Use Polar CLI or ngrok to test webhook locally

### RevenueCat

- [ ] Sandbox purchase completes on simulator/device
- [ ] Entitlement activates and gates feature correctly
- [ ] Restore purchases works
- [ ] RevenueCat dashboard shows test subscriber

### Superwall

- [ ] Paywall appears on trigger in debug mode (`Superwall.shared.debugMode = true`)
- [ ] Sandbox purchase through paywall completes
- [ ] Free user blocked, paid user passes gate


## Phase 6: Live Checklist

### Web (all providers)

- [ ] Products and prices created in provider dashboard (live mode)
- [ ] Webhook endpoint registered with correct events
- [ ] Live API keys set in production env vars (never committed to git)
- [ ] Webhook handler is idempotent (safe to receive same event twice)
- [ ] Error logging on webhook failures
- [ ] Customer receipt emails enabled
- [ ] Test a real charge with your own card before launch

### Mobile (RevenueCat / Superwall)

- [ ] App Store Connect / Google Play products approved and linked in RevenueCat
- [ ] Production API keys configured per platform
- [ ] Entitlements verified with a real sandbox purchase before App Store review
- [ ] RevenueCat webhooks wired to backend (if mirroring state server-side)
- [ ] Superwall paywalls reviewed and published in dashboard


## Completion Report

- Products and prices configured
- API routes / SDK integration created
- Webhook events handled and tested
- Frontend: pricing page, upgrade prompts, billing management
- Gating: which routes/features/screens are protected
- Env vars required (names only)
