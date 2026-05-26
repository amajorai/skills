---
name: app-store-compliance
description: Audits an app against the latest Apple App Store and Google Play Store review guidelines. Fetches current guidelines from official sources, interviews the user about app features, content, monetization, and target audience, then produces a detailed compliance report with pass/fail status for each rule category and a prioritized fix list. Use before submitting a new app, adding a major feature, or after a rejection to understand what needs to change.
argument-hint: [app name and brief description, or paste rejection reason]
---

# App Store & Play Store Compliance Audit

You are auditing an app against Apple App Store and Google Play Store review policies. Work through each phase in order. Fetch current guidelines before auditing: policies change frequently and your training data may be stale.

**App:** {{args}}


## Phase 0: Auto-Update

*Skip unless `{{args}}` contains `--update`, or `SKILLS_AUTO_UPDATE: true` is set in your project CLAUDE.md.*

```bash
npx --yes skills update amajorai/skills -y 2>/dev/null || true
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.


## Phase 1: Fetch Current Guidelines

Before asking the user anything, fetch the current official guidelines in parallel:

- Apple App Store Review Guidelines: `https://developer.apple.com/app-store/review/guidelines/`
- Google Play Developer Program Policies: `https://play.google.com/intl/en_us/about/developer-content-policy/`
- Google Play Target API Level Requirements: `https://developer.android.com/google/play/requirements/target-sdk`
- Apple Human Interface Guidelines overview: `https://developer.apple.com/design/human-interface-guidelines/`

Summarize what version/date of guidelines you retrieved. If a fetch fails, note it and proceed with training data, flagging it as potentially outdated.


## Phase 2: App Interview

Ask all questions in a single message. Tell the user: detailed answers prevent false positives in the audit.

**Core App Details**
- What does the app do? (2–3 sentences)
- Which platforms? (iOS only / Android only / both)
- New submission, update, or responding to a rejection? If rejection: paste the exact reason(s).
- Primary category + subcategory?

**Content & Features** — check all that apply:
User-generated content / real-money gambling / dating or social networking / medical or health advice / financial products / virtual currency / news or editorial / political content / explicit or mature content / content involving real people / weapons / alcohol or tobacco / cannabis / VPN or network proxy / device management or MDM / browser or web engine / platform emulation / screen recording or accessibility features / widgets or extensions / AR camera features

**Monetization** — check all that apply:
Free / one-time purchase / IAP consumable / IAP non-consumable / auto-renewing subscription / non-renewing subscription / advertising / physical goods/services / digital goods via external link / tipping / NFT

- Using Apple StoreKit / Google Play Billing, or directing users outside the app?
- Web subscription at a lower price than in-app?
- External purchase options, links, or prices mentioned inside the app?

**Target Audience & Age Rating**
- Primary target age group?
- Users allowed under 13? under 16? under 18?
- Content: violence / sexual content (none/suggestive/explicit) / profanity / drugs-alcohol / horror / gambling?
- Intended for Kids category on either store?

**Privacy & Data**
- Personal data collected? (refer to existing privacy policy if available)
- Device capabilities used: Camera / Microphone / Precise Location / Approximate Location / Contacts / Calendar / Reminders / Photos / Motion-Fitness / HealthKit / HomeKit / Bluetooth / Local Network / Face ID-Touch ID / IDFA Tracking?
- Third-party SDKs for analytics, advertising, or tracking?
- Data shared with third parties for advertising?

**Technical**
- Minimum iOS version? Minimum Android API level?
- Private or undocumented APIs? (iOS)
- Remote code or scripts loaded after installation?
- Account required for core functionality? If yes: Sign in with Apple / Sign in with Google available?
- Demo mode or test account for reviewers?

Do not proceed to Phase 3 until all Phase 2 questions are answered.


## Phase 3: Apple App Store Audit

Work through each guideline category using the audit tables in [references/apple-checklist.md](references/apple-checklist.md).

For each item mark: **PASS** / **FAIL** / **RISK** / **N/A**

Cover all six sections:
- 3.1 Safety
- 3.2 Performance
- 3.3 Business (Payments & Monetization)
- 3.4 Design
- 3.5 Legal & Privacy
- 3.6 App-Type-Specific Rules (apply only relevant subsections based on Phase 2)


## Phase 4: Google Play Store Audit

Work through each guideline category using the audit tables in [references/google-play-checklist.md](references/google-play-checklist.md).

For each item mark: **PASS** / **FAIL** / **RISK** / **N/A**

Cover all seven sections:
- 4.1 Restricted Content
- 4.2 Privacy & Data Safety
- 4.3 Deceptive Behavior
- 4.4 Monetization & Payments
- 4.5 Families Policy (if targeting children)
- 4.6 Technical Requirements
- 4.7 App-Type-Specific Play Policies (apply only relevant subsections)


## Phase 5: Compliance Report

Generate `app-store-compliance-report.md` in the current working directory using the full report structure in [references/report-template.md](references/report-template.md).

Use exact values from the audit — no placeholder text.


## Phase 6: Rejection Response (if applicable)

If the user provided a rejection reason in Phase 2, append a Rejection Response section to the report using the analysis structure and appeal template in [references/rejection-response.md](references/rejection-response.md).


## Ongoing Compliance Note

App store policies update frequently: Apple typically updates in the fall alongside iOS releases; Google updates on a rolling basis. Recommended practice:
- Subscribe to Apple Developer news and Google Play policy update emails
- Re-run this audit before every major version submission
- Watch for policy sunset dates when using deprecated APIs or SDKs
