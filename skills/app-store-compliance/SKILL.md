---
name: app-store-compliance
description: Audits an app against the latest Apple App Store and Google Play Store review guidelines. Fetches current guidelines from official sources, interviews the user about app features, content, monetization, and target audience, then produces a detailed compliance report with pass/fail status for each rule category and a prioritized fix list. Use before submitting a new app, adding a major feature, or after a rejection to understand what needs to change.
argument-hint: [app name and brief description, or paste rejection reason]
---

# app-store-compliance — App Store & Play Store Compliance Audit

You are auditing an app against Apple App Store and Google Play Store review policies. Work through each phase in order. Fetch current guidelines before auditing — policies change frequently and your training data may be stale.

**App:** {{args}}


## Phase 1: Fetch Current Guidelines

Before asking the user anything, fetch the current official guidelines. Run these in parallel:

- Fetch Apple App Store Review Guidelines: `https://developer.apple.com/app-store/review/guidelines/`
- Fetch Google Play Developer Program Policies: `https://play.google.com/intl/en_us/about/developer-content-policy/`
- Fetch Google Play Target API Level Requirements: `https://developer.android.com/google/play/requirements/target-sdk`
- Fetch Apple Human Interface Guidelines overview: `https://developer.apple.com/design/human-interface-guidelines/`

Summarize what version/date of guidelines you retrieved. If a fetch fails, note it and proceed with your training data, flagging it as potentially outdated.


## Phase 2: App Interview

Ask all questions in a single message. Tell the user: detailed answers prevent false positives in the audit.

**Core App Details**
- What does the app do? (describe the primary use case in 2–3 sentences)
- Which platforms are you targeting? (iOS only, Android only, or both)
- Is this a new app submission, an update to an existing app, or are you responding to a rejection?
- If responding to a rejection: paste the exact rejection reason(s) from Apple or Google
- What is the app's primary category? (e.g., Productivity, Games, Health & Fitness, Social Networking, Finance, Education, Entertainment, Shopping, Utilities, Medical)
- What subcategory or secondary category applies?

**Content & Features**
- Does the app contain any of the following? (check all that apply)
  - User-generated content (posts, comments, images, videos, messages)
  - Real-money gambling, poker, or lotteries
  - Dating or social networking features
  - Medical or health advice or diagnosis
  - Financial products (loans, investments, insurance, crypto trading)
  - Virtual currency or in-app tokens (not real money)
  - News or editorial content
  - Political content or voter registration
  - Religious content
  - Explicit or mature content (violence, nudity, strong language)
  - Content involving real people (celebrities, politicians, etc.)
  - Weapons (depictions of guns, purchase/sale/modification of real weapons)
  - Alcohol or tobacco
  - Cannabis or controlled substances
  - VPN or network proxy functionality
  - Device management, MDM, or enterprise features
  - Browser, web engine, or web rendering beyond a WebView
  - Emulation of other platforms or devices
  - Screen recording or accessibility features that interact with other apps
  - Widgets, extensions, keyboard extensions, or share extensions
  - AR features using the camera

**Monetization**
- How does the app make money? (select all that apply)
  - Free with no monetization
  - One-time purchase price
  - In-app purchases (consumable items, e.g. coins, credits)
  - In-app purchases (non-consumable, e.g. unlock a feature)
  - Auto-renewing subscriptions
  - Non-renewing subscriptions
  - Advertising (banner ads, interstitial ads, rewarded video)
  - Selling physical goods or services (restaurant orders, ride-sharing, etc.)
  - Selling digital goods or services (e.g. linking to external purchase)
  - Tipping creators
  - NFT minting or marketplace

- If using subscriptions or IAP: are you using Apple's StoreKit / Google Play Billing, or are you directing users to purchase outside the app?
- Do you offer a web subscription at a lower price than the in-app subscription?
- Do you mention external purchase options, links, or prices inside the app?

**Target Audience & Age Rating**
- What age group is your primary target audience?
- Does the app allow users under 13? under 16? under 18?
- Does the app contain: violence, realistic violence, cartoon violence, sexual content (none / suggestive / explicit), profanity, drug/alcohol/tobacco references, horror/fear themes, gambling?
- Is the app intended for the Kids category on either store?

**Privacy & Data**
- Does the app collect any personal data? (refer to user's existing privacy policy if available)
- Does the app use any of these device capabilities: Camera, Microphone, Location (precise), Location (approximate), Contacts, Calendar, Reminders, Photos/Media Library, Motion/Fitness data, Health data (HealthKit), HomeKit, Bluetooth, Local Network, Face ID/Touch ID, Tracking (IDFA)?
- Does the app use any third-party SDKs for analytics, advertising, or tracking?
- Does the app share any data with third parties for advertising purposes?

**Technical**
- What is the minimum iOS version supported? (iOS apps)
- What is the minimum Android API level targeted? (Android apps)
- Does the app use any private or undocumented APIs? (iOS)
- Does the app load remote code or scripts after installation?
- Does the app require an account to access any core functionality?
- If account required: can users sign in with Apple (iOS) / sign in with Google (Android)?
- Does the app have a working demo mode or test account you can provide to reviewers?
- What is the reviewer login / demo account? (do not share real user data — provide a sandboxed test account)

Do not proceed to Phase 3 until all Phase 2 questions are answered.


## Phase 3: Apple App Store Audit

Work through each guideline category. For each item, mark as:
- PASS — compliant based on the information provided
- FAIL — clear violation that will cause rejection
- RISK — potential issue that needs attention or clarification
- N/A — not applicable to this app

### 3.1 Safety

| Check | Status | Notes |
|---|---|---|
| App does not encourage or enable violence against real people or animals | | |
| User-generated content has moderation, reporting, and blocking mechanisms | | |
| App does not encourage self-harm, suicide, or eating disorders | | |
| App that connects strangers uses commercially reasonable safeguards against predatory behavior | | |
| Apps using location share location only to connected users, with opt-out | | |
| App does not enable anonymous or prank calls to emergency services | | |
| Medical/health apps cite reputable sources and include medical disclaimers | | |
| Apps for minors do not use targeted advertising | | |

### 3.2 Performance

| Check | Status | Notes |
|---|---|---|
| App is complete — no placeholder content, broken links, dummy data in production build | | |
| App performs advertised function on the target device and iOS version | | |
| App does not use excessive battery, storage, or network bandwidth | | |
| App does not crash or display obvious technical problems | | |
| App provides reviewer credentials / demo mode for gated content | | |
| App has sufficient lasting value — not a single use or trivially simple app | | |
| App is not a duplicate of another app from the same developer (spam) | | |
| App name, keywords, and description accurately reflect the app's content and features | | |
| Screenshots and preview video accurately reflect the app's actual UI | | |
| App does not artificially inflate ratings (no prompting for positive reviews only) | | |
| Rating prompts use the standard StoreKit requestReview API (max 3 times per year) | | |

### 3.3 Business (Payments & Monetization)

| Check | Status | Notes |
|---|---|---|
| In-app purchases of digital goods/services use Apple's In-App Purchase (StoreKit) | | |
| App does not link to or mention external purchase options for digital goods (unless reader app exception) | | |
| App does not use confusing language to prevent users from understanding they are buying a subscription | | |
| Subscription free trial terms are clearly disclosed | | |
| App does not charge users for features they have a right to access for free (e.g., required platform features) | | |
| Real-money gambling apps have necessary local licenses and are geo-restricted to permitted regions | | |
| Cryptocurrency exchange/wallet apps submitted by established financial institutions | | |
| NFT apps do not enable purchasing NFTs with IAP; may allow viewing/browsing | | |
| Physical goods and services (e.g., Uber, Amazon) may use their own payment systems | | |
| Freemium apps make it clear which features are free vs. paid before download | | |
| "Reader" apps (Netflix, Spotify) may omit IAP if they only access previously purchased content — but cannot link to sign-up | | |
| Apps using EntitlementLink / Account Deletion compliance: if app allows account creation, it must also allow account deletion | | |

### 3.4 Design

| Check | Status | Notes |
|---|---|---|
| App follows Apple Human Interface Guidelines — does not mimic iOS system UI deceptively | | |
| App does not use custom keyboard extensions to track keystrokes without disclosure | | |
| Apple Sign In offered if app supports any third-party sign-in (Google, Facebook, etc.) | | |
| App does not use Push Notifications for advertising or promotions without user opt-in | | |
| App does not spam or send unsolicited messages | | |
| Widgets display timely, relevant information from the parent app | | |
| Extensions do not include marketing materials or upsells | | |
| App Clips: under 15MB uncompressed, work without full app, no excessive data collection | | |

### 3.5 Legal & Privacy

| Check | Status | Notes |
|---|---|---|
| App has a privacy policy URL set in App Store Connect | | |
| Privacy policy is accessible from within the app (not just the App Store listing) | | |
| App Privacy Nutrition Labels in App Store Connect accurately reflect data collected | | |
| Usage description strings (NSCameraUsageDescription, NSLocationWhenInUseUsageDescription, etc.) present for every permission requested | | |
| Usage descriptions explain WHY the permission is needed (not just THAT it is needed) | | |
| App does not request permissions not needed for core functionality | | |
| App uses App Tracking Transparency (ATT) framework before tracking users across apps/websites | | |
| Privacy Manifest file included if app uses Required Reason APIs (UserDefaults, file timestamps, disk space, active keyboard, system boot time) | | |
| SDK Privacy Manifests included for all third-party SDKs that require them | | |
| App does not collect device fingerprinting data to uniquely identify devices | | |
| App does not enable user profiling without explicit consent | | |
| App does not violate export compliance laws (encryption export) — declare encryption use in App Store Connect | | |
| App contains only content the developer has rights to use | | |
| App does not defame, stalk, bully, or harass real people | | |
| Apps with mature content use correct age rating in App Store Connect | | |
| Kids Category apps do not contain ads, do not link outside the app, do not collect personal data beyond app functionality | | |

### 3.6 App-Type-Specific Rules

Apply only the sections relevant to this app based on Phase 2 answers:

**If user-generated content:**
- [ ] Content moderation system in place (automated + human review path)
- [ ] In-app mechanism to report offensive content
- [ ] Block/mute mechanism for specific users
- [ ] Ability for developer to remove content and ban users
- [ ] Privacy policy explains UGC moderation

**If dating/social:**
- [ ] Sign in with Apple required
- [ ] Age verification for 18+ features
- [ ] User blocking and reporting
- [ ] No full nudity

**If health/medical:**
- [ ] Data shared with HealthKit only used for health/fitness purposes
- [ ] Does not use HealthKit data for advertising or third-party data mining
- [ ] Medical disclaimers present
- [ ] Does not make specific medical claims

**If finance:**
- [ ] Licensed/regulated status disclosed
- [ ] Geo-restricted to permitted jurisdictions
- [ ] Does not offer margin trading or leveraged financial products without compliance evidence (for crypto apps)

**If Kids Category:**
- [ ] No behavioral advertising
- [ ] No links outside the app
- [ ] No social networking features
- [ ] No purchase prompts directed at children
- [ ] Parental gate on any external links or purchases


## Phase 4: Google Play Store Audit

### 4.1 Restricted Content

| Check | Status | Notes |
|---|---|---|
| No sexual content beyond what policy permits for designated adult apps | | |
| No content that sexualizes minors in any form | | |
| No gratuitous violence — violence must have redemptive purpose | | |
| No content that incites hatred based on protected characteristics | | |
| No content facilitating illegal activities | | |
| No bullying, harassment, or threatening content | | |
| Dangerous products/substances not promoted (how-to guides for illegal activities) | | |

### 4.2 Privacy & Data Safety

| Check | Status | Notes |
|---|---|---|
| Data Safety section in Play Console accurately filled out | | |
| Prominent disclosure shown before collecting sensitive data (location, contacts, SMS, call logs, camera, microphone, account data, device/app data, financial data, health data) | | |
| Consent obtained before collecting sensitive data | | |
| Data collected is limited to what is necessary for disclosed purposes | | |
| Personal and sensitive user data handled securely (HTTPS) | | |
| Permissions requested are necessary for core functionality | | |
| Permissions not requested in bulk at app launch — requested contextually at time of need | | |
| App does not silently transmit user data to third parties | | |
| If app targets children (Families Policy): no behaviorally targeted ads, no collection beyond what policy permits, SDKs are Families-certified | | |

### 4.3 Deceptive Behavior

| Check | Status | Notes |
|---|---|---|
| App does exactly what the store listing claims | | |
| App does not impersonate other apps, developers, or entities | | |
| App does not use misleading app icons, titles, or descriptions | | |
| App does not artificially inflate installs, ratings, or reviews | | |
| App does not use unauthorized use of trademarked or copyrighted material | | |
| App discloses if it contains ads — "Contains ads" label in Play Console if applicable | | |
| Ads within app comply with Google Play Ads Policy — no deceptive ads, no ads that interfere with navigation | | |

### 4.4 Monetization & Payments

| Check | Status | Notes |
|---|---|---|
| Digital goods and services purchased within the app use Google Play Billing | | |
| App does not direct users outside to purchase digital goods that can be used in-app | | |
| Subscriptions clearly disclose: price, billing period, free trial terms, cancellation policy | | |
| Subscription pricing is not misrepresented (e.g., showing "per week" price for annual plan) | | |
| Physical goods/services may use alternative payment methods | | |
| Real-money gambling apps geo-restricted and licensed | | |

### 4.5 Families Policy (if targeting children)

| Check | Status | Notes |
|---|---|---|
| Target audience age group accurately set in Play Console | | |
| No interest-based advertising to users under 13 | | |
| All ad SDKs used are Google Play Families-certified | | |
| App content appropriate for declared age range | | |
| No collection of device serial number, SIM serial, IMEI beyond what's permitted | | |
| Prominent parental consent mechanism before collecting data from children | | |

### 4.6 Technical Requirements

| Check | Status | Notes |
|---|---|---|
| Target SDK level meets Google Play requirements (check Phase 1 fetch for current minimum) | | |
| 64-bit support included in APK/AAB | | |
| App does not use non-SDK (private) APIs | | |
| App does not download executable code after installation | | |
| App has a functional back button behavior on Android | | |
| Notification permission requested contextually (not at first launch) — Android 13+ | | |
| If using Android permissions: QUERY_ALL_PACKAGES only if core functionality requires (else use explicit package names) | | |
| VPN Service apps: must be the VPN's primary app, not bundled with unrelated apps | | |
| Accessibility Services: declared only if app is genuinely an accessibility tool | | |

### 4.7 App-Type-Specific Play Policies

**If financial services app:**
- [ ] Discloses annual percentage rate (APR) in app listing description
- [ ] Minimum and maximum repayment period disclosed
- [ ] No apps allowing personal loan repayment period under 60 days
- [ ] Must provide a physical address

**If health/medical:**
- [ ] Cannot sell or facilitate sale of prescription drugs without verified license
- [ ] Prescription drug apps must be verified by Google

**If aggregating news:**
- [ ] Clearly attribute content to original publisher
- [ ] Does not misrepresent sources as original reporting


## Phase 5: Compliance Report

Generate a structured report saved to `/app-store-compliance-report.md`.

### Report Structure

```
# App Store Compliance Report
## App: [App Name]
## Generated: [today's date]
## Guidelines Version: [version/date retrieved in Phase 1]


## Executive Summary

[2–3 sentence summary of overall compliance status and most urgent issues]

## Apple App Store: [PASS / CONDITIONAL / FAIL]

### Critical Issues (will cause rejection)
[Numbered list — empty if none]

### Risks (may cause rejection or future policy action)
[Numbered list — empty if none]

### Recommendations (best practice, not blocking)
[Numbered list — empty if none]

### Checklist Summary
| Category | Status | Issues |
|---|---|---|
| Safety | PASS/FAIL/RISK | |
| Performance | PASS/FAIL/RISK | |
| Business | PASS/FAIL/RISK | |
| Design | PASS/FAIL/RISK | |
| Legal & Privacy | PASS/FAIL/RISK | |


## Google Play Store: [PASS / CONDITIONAL / FAIL]

### Critical Issues (will cause rejection or removal)
[Numbered list — empty if none]

### Risks (may trigger policy warning or future removal)
[Numbered list — empty if none]

### Recommendations
[Numbered list — empty if none]

### Checklist Summary
| Category | Status | Issues |
|---|---|---|
| Restricted Content | | |
| Privacy & Data Safety | | |
| Deceptive Behavior | | |
| Monetization | | |
| Families Policy | N/A or status | |
| Technical Requirements | | |


## Priority Fix List

Ordered by urgency (blocking → high → medium → low):

| # | Issue | Platform | Severity | Fix |
|---|---|---|---|---|
| 1 | [issue] | iOS/Android/Both | Blocking | [specific action] |
...


## Metadata to Complete in App Store Connect / Play Console

List every metadata field the user still needs to fill in:

### App Store Connect
- [ ] Privacy policy URL
- [ ] App Privacy labels (data types, purposes, linked/not linked, tracking)
- [ ] Age rating questionnaire
- [ ] Encryption compliance declaration
- [ ] Review notes / demo account credentials

### Google Play Console
- [ ] Data Safety section (all questions)
- [ ] Content rating questionnaire (IARC)
- [ ] Target audience and content settings
- [ ] Financial features declaration (if applicable)
- [ ] News app declaration (if applicable)


## Before You Submit

- [ ] Test the app on a real device (not just simulator/emulator)
- [ ] Test every IAP and subscription flow end-to-end in sandbox
- [ ] Verify all external URLs in the app are live
- [ ] Confirm reviewer demo account works and provides access to all reviewed features
- [ ] Remove any test/debug code, console logs, and placeholder content
- [ ] Check app binary does not include unused permission strings (iOS: remove from Info.plist)
- [ ] Verify Privacy Manifest and SDK manifests are included (iOS 17+)
- [ ] Run through the app as a first-time user — onboarding must be clear without external context


## Resources

- Apple rejection appeals: https://developer.apple.com/app-store/review/#common-app-rejections
- Apple review guidelines: https://developer.apple.com/app-store/review/guidelines/
- Google Play policy center: https://support.google.com/googleplay/android-developer/topic/9858052
- Google Play appeals: https://support.google.com/googleplay/android-developer/answer/9899234
```


## Phase 6: Rejection Response (if applicable)

If the user provided a rejection reason in Phase 2, add a dedicated section to the report:

**Rejection Analysis**
1. Quote the exact rejection reason
2. Identify the specific guideline violated (with section number)
3. Explain what the reviewer likely saw that triggered the rejection
4. Provide step-by-step instructions to resolve it
5. Draft the appeal message text (if the rejection appears incorrect or overly broad)

**Appeal message template:**
```
Subject: Appeal — [App Name] — [Rejection Reason Code]

Dear App Review Team,

Thank you for reviewing [App Name]. We would like to respectfully appeal the rejection under guideline [X.X].

[Explain what the app does and why it does not violate the cited guideline, OR explain exactly what change was made to resolve the issue]

[If changes were made] We have made the following changes in version [X.X.X]:
- [Change 1]
- [Change 2]

[If providing additional context] We would be happy to provide additional information or a screen recording demonstrating [specific feature].

Thank you for your time and consideration.

[Name]
[Developer/Company Name]
```


## Ongoing Compliance Note

App store policies update frequently — Apple typically updates guidelines in the fall alongside iOS releases; Google updates Play policies on a rolling basis. Recommended practice:
- Subscribe to Apple Developer news and Google Play policy update emails
- Re-run this audit before every major version submission
- Watch for policy sunset dates when using deprecated APIs or SDKs
