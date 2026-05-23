---
name: cookie-consent
description: Add a GDPR/CCPA-compliant cookie consent banner to any web app. Implements consent categories, blocks non-essential scripts until consent is given, and provides an opt-out mechanism. Use when the app uses analytics, advertising, or any non-essential cookies.
argument-hint: <cookie categories used: analytics | marketing | preferences | all>
---

# Cookie Consent

You are implementing a compliant cookie consent system. Work through each phase in order.

**Cookie categories:** {{args}}


## Phase 0: Auto-Update

*Skip if `{{args}}` contains `--no-update`, or if `SKILLS_AUTO_UPDATE: false` is set in your project CLAUDE.md.*

```bash
npx skills update cookie-consent -y
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Phase 1: Audit Cookies

Spawn **1 subagent** to:
- Find all analytics/tracking scripts in the codebase (PostHog, Google Analytics, Hotjar, Meta Pixel, etc.)
- Identify cookies set by the app vs. third-party scripts
- Check if there are existing consent mechanisms

Produce a cookie inventory:

| Cookie | Set by | Category | Purpose | Expiry |
|--------|--------|----------|---------|--------|
| `_ph_*` | PostHog | Analytics | User tracking | 1 year |
| ... | ... | ... | ... | ... |


## Phase 2: Interview

Ask the user (combine related questions):

- **Jurisdictions**: EU/UK (GDPR), California (CCPA), or global?
- **Categories**: Which categories apply: Strictly Necessary, Analytics, Marketing, Preferences?
- **Library**: Use a pre-built library (Cookiebot, CookieYes) or build a lightweight custom banner?
- **Design**: Match the app's design system or use a minimal default?

For most indie apps: recommend building a lightweight custom banner (~50 lines) rather than a heavy third-party widget.


## Phase 3: Implement Consent Store

Create a consent store that:
1. Persists consent decisions to localStorage (`cookie_consent` key)
2. Exposes per-category consent: `{ analytics: boolean, marketing: boolean, preferences: boolean }`
3. Defaults to no consent until the user acts (opt-in model, required for GDPR)
4. Fires a `consent_updated` event when consent changes (so scripts can initialize)
5. Provides `hasConsented()`, `grantConsent(categories)`, `revokeConsent(categories)` functions


## Phase 4: Block Scripts Until Consent

For each non-essential script:

1. **Analytics (e.g., PostHog)**: Wrap initialization in consent check
   ```typescript
   if (consent.analytics) { posthog.init(...) }
   window.addEventListener('consent_updated', () => { if (consent.analytics) posthog.init(...) })
   ```
2. **Third-party scripts**: Change `type="text/javascript"` to `type="text/plain"` and add `data-category="analytics"` so the consent manager can activate them after consent
3. **Server-side tracking**: Check consent header or cookie before processing


## Phase 5: Build the Banner

Create a consent banner component with:

1. **First visit**: Banner at the bottom of the screen with:
   - Brief explanation of what cookies are used for
   - "Accept All" button
   - "Reject Non-Essential" button
   - "Manage Preferences" link
2. **Preferences modal**: Toggles per category with a short description of each
3. **Footer link**: "Cookie Settings" that reopens the preferences modal at any time
4. **No dark patterns**: Accept and Reject buttons must be equal prominence (GDPR requirement)


## Phase 6: CCPA (if applicable)

For California users:

- [ ] "Do Not Sell My Personal Information" link in footer
- [ ] Clicking it sets marketing consent to false and persists the choice
- [ ] No selling/sharing of data when this opt-out is active


## Phase 7: Verify

- [ ] Banner appears on first visit (clear localStorage to test)
- [ ] Banner does not appear on return visits after consent is given
- [ ] Accepting all consent initializes analytics scripts
- [ ] Rejecting non-essential blocks all non-essential scripts (verify in Network tab: no PostHog/GA calls)
- [ ] "Manage Preferences" link in footer opens the modal
- [ ] Revoking consent stops tracking and clears existing tracking cookies
- [ ] No console errors on any consent state


## Completion Report

- Cookie inventory documented
- Consent store implemented
- Non-essential scripts blocked until consent
- Banner and preferences modal built
- CCPA opt-out (if applicable)
- Pages where banner/footer link appears
