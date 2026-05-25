# Apple App Store Audit Checklist

Mark each item: **PASS** / **FAIL** / **RISK** / **N/A**

## 3.1 Safety

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

## 3.2 Performance

| Check | Status | Notes |
|---|---|---|
| App is complete: no placeholder content, broken links, or dummy data in production build | | |
| App performs its advertised function on the target device and iOS version | | |
| App does not use excessive battery, storage, or network bandwidth | | |
| App does not crash or display obvious technical problems | | |
| App provides reviewer credentials or demo mode for gated content | | |
| App has sufficient lasting value: not a single-use or trivially simple app | | |
| App is not a duplicate of another app from the same developer (spam) | | |
| App name, keywords, and description accurately reflect content and features | | |
| Screenshots and preview video accurately reflect the app's actual UI | | |
| App does not artificially inflate ratings (no prompting for positive reviews only) | | |
| Rating prompts use the standard StoreKit requestReview API (max 3 times per year) | | |

## 3.3 Business (Payments & Monetization)

| Check | Status | Notes |
|---|---|---|
| In-app purchases of digital goods/services use Apple's In-App Purchase (StoreKit) | | |
| App does not link to or mention external purchase options for digital goods (unless reader app exception) | | |
| App does not use confusing language to prevent users from understanding they are buying a subscription | | |
| Subscription free trial terms are clearly disclosed | | |
| App does not charge users for features they have a right to access for free | | |
| Real-money gambling apps have necessary local licenses and are geo-restricted | | |
| Cryptocurrency exchange/wallet apps submitted by established financial institutions | | |
| NFT apps do not enable purchasing NFTs with IAP; may allow viewing/browsing | | |
| Physical goods and services may use their own payment systems | | |
| Freemium apps make it clear which features are free vs. paid before download | | |
| "Reader" apps may omit IAP but cannot link to sign-up from within the app | | |
| App allows account deletion if it allows account creation | | |

## 3.4 Design

| Check | Status | Notes |
|---|---|---|
| App follows Apple Human Interface Guidelines: does not mimic iOS system UI deceptively | | |
| App does not use custom keyboard extensions to track keystrokes without disclosure | | |
| Apple Sign In offered if app supports any third-party sign-in (Google, Facebook, etc.) | | |
| App does not use Push Notifications for advertising or promotions without user opt-in | | |
| App does not spam or send unsolicited messages | | |
| Widgets display timely, relevant information from the parent app | | |
| Extensions do not include marketing materials or upsells | | |
| App Clips: under 15 MB uncompressed, work without full app, no excessive data collection | | |

## 3.5 Legal & Privacy

| Check | Status | Notes |
|---|---|---|
| App has a privacy policy URL set in App Store Connect | | |
| Privacy policy is accessible from within the app (not just the App Store listing) | | |
| App Privacy Nutrition Labels in App Store Connect accurately reflect data collected | | |
| Usage description strings present for every permission requested (NSCameraUsageDescription, etc.) | | |
| Usage descriptions explain WHY the permission is needed, not just that it is needed | | |
| App does not request permissions not needed for core functionality | | |
| App uses App Tracking Transparency (ATT) framework before tracking users across apps/websites | | |
| Privacy Manifest file included if app uses Required Reason APIs (UserDefaults, file timestamps, disk space, active keyboard, system boot time) | | |
| SDK Privacy Manifests included for all third-party SDKs that require them | | |
| App does not collect device fingerprinting data to uniquely identify devices | | |
| App does not enable user profiling without explicit consent | | |
| Encryption export compliance declared in App Store Connect | | |
| App contains only content the developer has rights to use | | |
| App does not defame, stalk, bully, or harass real people | | |
| Apps with mature content use correct age rating in App Store Connect | | |
| Kids Category apps: no ads, no links outside the app, no personal data collected beyond app functionality | | |

## 3.6 App-Type-Specific Rules

Apply only the subsections relevant to this app.

**User-Generated Content**
- [ ] Content moderation system in place (automated + human review path)
- [ ] In-app mechanism to report offensive content
- [ ] Block/mute mechanism for specific users
- [ ] Ability for developer to remove content and ban users
- [ ] Privacy policy explains UGC moderation

**Dating / Social**
- [ ] Sign in with Apple required
- [ ] Age verification for 18+ features
- [ ] User blocking and reporting
- [ ] No full nudity

**Health / Medical**
- [ ] HealthKit data used only for health/fitness purposes
- [ ] HealthKit data not used for advertising or third-party data mining
- [ ] Medical disclaimers present
- [ ] No specific medical claims or diagnoses

**Finance**
- [ ] Licensed/regulated status disclosed
- [ ] Geo-restricted to permitted jurisdictions
- [ ] No margin trading or leveraged financial products without compliance evidence (crypto apps)

**Kids Category**
- [ ] No behavioral advertising
- [ ] No links outside the app
- [ ] No social networking features
- [ ] No purchase prompts directed at children
- [ ] Parental gate on any external links or purchases
