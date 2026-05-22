---
name: waitlist
description: Build a waitlist landing page with email capture, referral mechanics, position tracking, and a drip email welcome sequence. Use before launching a product to build an audience and create launch momentum.
argument-hint: <product name and one-line description>
---

# Waitlist

You are building a waitlist system with email capture, referral mechanics, and automated emails. Work through each phase in order.

**Product:** {{args}}


## Phase 1: Interview

Ask the user (combine related questions):

- **Goal**: Pure email list, or viral referral loop (users move up the queue by referring friends)?
- **Email tool**: Resend, Postmark, or ConvertKit for the drip sequence?
- **Existing site**: Adding to an existing app, or a standalone landing page?
- **Design**: Does the user have brand assets (colors, logo, copy)?
- **Launch timeline**: When is the product launching? (affects urgency messaging)


## Phase 2: Plan

Define the components:

1. **Landing page** — hero, value prop, social proof (if any), email capture form
2. **Waitlist table** — `email`, `referral_code`, `referred_by`, `position`, `created_at`
3. **Signup API** — validates email, creates record, sends welcome email, returns position
4. **Referral system** — unique link per user, position bump when referrals sign up
5. **Confirmation page** — position, referral link, share buttons
6. **Welcome email** — confirms signup, shows position, shares referral link
7. **Admin view** — count of signups, referral leaderboard (optional)


## Phase 3: Database

Create the waitlist table:

```sql
CREATE TABLE waitlist (
  id          TEXT PRIMARY KEY DEFAULT gen_random_uuid(),
  email       TEXT UNIQUE NOT NULL,
  referral_code TEXT UNIQUE NOT NULL DEFAULT substr(md5(random()::text), 1, 8),
  referred_by TEXT REFERENCES waitlist(referral_code),
  position    INTEGER NOT NULL,
  created_at  TIMESTAMPTZ DEFAULT now()
);
```

Position is assigned at insert time (use `count(*) + 1` or a sequence). If referred, insert the referrer ahead of new organic signups.


## Phase 4: API

Create `POST /api/waitlist`:
1. Validate email format
2. Check for duplicate — return existing position if already signed up
3. Resolve referral code if `?ref=<code>` param present
4. Insert with calculated position
5. Send welcome email asynchronously (don't block the response)
6. Return `{ position, referral_link, total_count }`


## Phase 5: Landing Page

Build a high-converting landing page:

1. **Hero**: Clear headline (what the product does), subheadline (who it's for), email input + CTA button
2. **Social proof**: Early user count, logos, testimonials (or placeholder if pre-launch)
3. **Features**: 3 key benefits with icons
4. **FAQ**: 3–5 common objections answered
5. **Footer**: Privacy policy link, unsubscribe note

After signup, show a confirmation state:
- "You're #[N] on the waitlist!"
- Referral link with copy button
- Pre-written share text for Twitter/X and LinkedIn
- Progress bar showing how many spots are "reserved"


## Phase 6: Welcome Email

Send immediately after signup:

- Subject: "You're on the list — here's your spot"
- Position in the waitlist
- Referral link with call to action
- What to expect (launch timeline, what early access means)
- Unsubscribe link


## Phase 7: Referral Mechanics

When a referred user signs up:
1. Look up the referrer by `referral_code`
2. Decrease the referrer's position by 1 (move up) for each referral
3. Send the referrer a "you moved up!" email (optional — set a threshold like every 5 referrals)

Cap position at 1 to prevent going to 0 or negative.


## Phase 8: Verify

- [ ] Signup form submits and shows confirmation with position
- [ ] Duplicate emails return the existing position (no error)
- [ ] Referral link works — signing up via it bumps the referrer
- [ ] Welcome email arrives within 30 seconds
- [ ] Share buttons pre-populate correct copy
- [ ] Mobile layout looks correct


## Completion Report

- Database schema created
- API endpoint implemented
- Landing page built (URL)
- Welcome email configured
- Referral mechanics working
- Env vars required (names only)
