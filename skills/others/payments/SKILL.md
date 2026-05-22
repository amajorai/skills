---
name: payments
description: Integrate payments into any web or mobile app. Sets up Stripe or LemonSqueezy with products, webhooks, subscription portal, billing page, and a test-to-live checklist. Use when adding a paywall, subscription plan, or one-time purchase to an app.
argument-hint: <billing model: subscription | one-time | usage-based>
---

# payments — Payment Integration

You are wiring up a complete payment integration. Work through each phase in order.

**Billing model:** {{args}}

---

## Phase 1: Interview

Ask the user (combine related questions):

- **Provider**: Stripe or LemonSqueezy? (Default: Stripe for direct card processing; LemonSqueezy for VAT/tax handling in solo projects)
- **Products**: What plans/products exist? Prices, billing intervals, trial periods?
- **Gates**: Which features are paywalled? What happens when a user exceeds their plan?
- **Existing auth**: How are users identified? What is the user model (table/schema)?
- **Stack**: Framework, language, existing API layer?

Confirm the product catalog and gating rules before proceeding.

---

## Phase 2: Explore

Spawn **3 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | User model, auth flow, session handling |
| 2 | Existing API routes, middleware, webhook handling patterns |
| 3 | Frontend component patterns, routing, protected pages |

Synthesize: data model changes needed, API route plan, frontend gating strategy.

---

## Phase 3: Plan

Define the full implementation surface:

1. **Database changes** — add `stripe_customer_id`, `subscription_status`, `plan` fields to user model
2. **API routes** — checkout session, billing portal, webhook handler
3. **Webhook events** — `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.deleted`, `invoice.payment_failed`
4. **Frontend** — pricing page, upgrade prompt, billing management page, plan-gated components
5. **Env vars** — keys needed in dev and prod

Present the plan and confirm before implementing.

---

## Phase 4: Implement

### Backend

1. Install SDK: `bun add stripe` or `bun add @lemonsqueezy/lemonsqueezy.js`
2. Create customer on user signup (or lazily on first checkout)
3. Checkout session endpoint — returns a hosted checkout URL
4. Billing portal endpoint — returns a portal URL for plan changes/cancellation
5. Webhook handler:
   - Verify signature (`stripe.webhooks.constructEvent`)
   - Handle each event by updating the user record in the database
   - Return `200` fast; do async work after acknowledging
6. Add middleware to protect gated routes — check `subscription_status === 'active'`

### Frontend

1. Pricing page with plan cards and a CTA that hits the checkout endpoint
2. Upgrade prompt component for paywalled features
3. Billing page (link to portal endpoint)
4. Show current plan in account settings

---

## Phase 5: Test Mode Verification

Before going live, verify end-to-end in test mode:

- [ ] Checkout flow completes with Stripe test card `4242 4242 4242 4242`
- [ ] Webhook fires and updates user record in DB
- [ ] Gated routes block free users and allow paid users
- [ ] Billing portal loads and allows plan changes
- [ ] Cancellation webhook downgrades the user correctly
- [ ] Failed payment webhook (`4000 0000 0000 0341`) triggers correct state

Use the Stripe CLI to forward webhooks locally: `stripe listen --forward-to localhost:3000/webhooks/stripe`

---

## Phase 6: Live Checklist

Before switching to live API keys:

- [ ] Products and prices created in Stripe dashboard (live mode)
- [ ] Webhook endpoint registered in Stripe dashboard with correct events
- [ ] Live API keys set in production env vars (never committed to git)
- [ ] Idempotency: webhook handler is safe to receive the same event twice
- [ ] Error logging on webhook failures
- [ ] Customer emails enabled in Stripe for receipts and payment failures
- [ ] Test a real $0.50 charge with your own card before launch

---

## Completion Report

- Products and prices configured
- API routes created (checkout, portal, webhook)
- Webhook events handled and tested
- Frontend: pricing page, upgrade prompts, billing page
- Gating: which routes/features are protected
- Env vars required (names only)
