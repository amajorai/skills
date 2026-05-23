---
name: analytics
description: Add product analytics to any web or mobile app. Sets up PostHog or Plausible with a proper event taxonomy, page views, funnel events, and a dashboard. Use when you need to understand how users actually use the app.
argument-hint: <tool: posthog | plausible | mixpanel>
---

# Analytics

You are wiring up analytics so the team can understand user behavior. Work through each phase in order.

**Tool:** {{args}} (default: PostHog)


## Phase 1: Interview

Ask the user (combine related questions):

- **Tool**: PostHog (default, self-hostable, product analytics + feature flags), Plausible (privacy-first, no cookies, page views only), or Mixpanel?
- **Key questions to answer**: What actions matter most? Where do users drop off? What does activation look like?
- **Existing tracking**: Is there any analytics already in place to migrate or extend?
- **Privacy**: Any GDPR/CCPA constraints on tracking? Cookie consent required?
- **Stack**: SPA, SSR, or mobile?


## Phase 2: Define Event Taxonomy

Before writing any code, define events to track. Structure:

```
<noun>_<verb>  (e.g., user_signed_up, project_created, upgrade_clicked)
```

Identify:
1. **Lifecycle events**: signed_up, activated, churned, reactivated
2. **Core product actions**: the 3–5 actions that define value delivery
3. **Funnel events**: each step a user takes from landing to activation
4. **Revenue events**: trial_started, plan_upgraded, plan_cancelled
5. **Error events**: payment_failed, auth_error, onboarding_abandoned

Confirm the taxonomy with the user before implementing. Over-tracking is as bad as under-tracking.


## Phase 3: Explore

Spawn **2 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | User flows: auth, onboarding, core features, upgrade path |
| 2 | Existing analytics code, env vars, and where events would fire |


## Phase 4: Install & Initialize

### PostHog

```bash
bun add posthog-js  # frontend
bun add posthog-node  # backend (server-side events)
```

- Initialize with `POSTHOG_API_KEY` from env var
- Set `person_profiles: 'identified_only'` to avoid anonymous profile bloat
- Enable session recording only if the user confirmed no PII in UI

### Plausible

- Add the script tag to the HTML head (no npm package needed)
- Enable custom events via `plausible('event_name', { props: {...} })`


## Phase 5: Instrument

For each event in the taxonomy:

1. Find the code location where the action occurs (server-side preferred for reliability)
2. Fire the event with consistent properties:
   - `user_id` (always)
   - `plan` / `role` (if applicable)
   - Context-specific props (e.g., `project_id`, `template_name`)
3. Use server-side events for revenue and auth events (not spoofable)
4. Use client-side events only for UI interactions (button clicks, modal opens)

Identify events and add them throughout the codebase: do not leave stubs.


## Phase 6: Dashboards

Set up a minimum viable dashboard:

1. **Activation funnel**: steps from signup to first core action
2. **Daily/weekly active users**
3. **Revenue events** (if applicable)
4. **Top drop-off points**

Document the dashboard URL and share it with the user.


## Phase 7: Verify

- [ ] Page views tracked correctly (open network tab, confirm event fires)
- [ ] Each taxonomy event fires exactly once when the action occurs
- [ ] No PII in event names or properties (email, passwords, tokens)
- [ ] Anonymous users are not creating unnecessary profiles
- [ ] Dashboard shows data within 5 minutes of events firing


## Completion Report

- Events instrumented (list with locations)
- Dashboard created
- Any events deferred (with reason)
- Env vars required (names only)
