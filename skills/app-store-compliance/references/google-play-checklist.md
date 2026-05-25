# Google Play Store Audit Checklist

Mark each item: **PASS** / **FAIL** / **RISK** / **N/A**

## 4.1 Restricted Content

| Check | Status | Notes |
|---|---|---|
| No sexual content beyond what policy permits for designated adult apps | | |
| No content that sexualizes minors in any form | | |
| No gratuitous violence: violence must have redemptive purpose | | |
| No content that incites hatred based on protected characteristics | | |
| No content facilitating illegal activities | | |
| No bullying, harassment, or threatening content | | |
| Dangerous products/substances not promoted (how-to guides for illegal activities) | | |

## 4.2 Privacy & Data Safety

| Check | Status | Notes |
|---|---|---|
| Data Safety section in Play Console accurately filled out | | |
| Prominent disclosure shown before collecting sensitive data (location, contacts, SMS, call logs, camera, microphone, account data, device/app data, financial data, health data) | | |
| Consent obtained before collecting sensitive data | | |
| Data collected is limited to what is necessary for disclosed purposes | | |
| Personal and sensitive user data handled securely (HTTPS) | | |
| Permissions requested are necessary for core functionality | | |
| Permissions requested contextually at time of need — not all at app launch | | |
| App does not silently transmit user data to third parties | | |
| If app targets children (Families Policy): no behaviorally targeted ads, SDKs are Families-certified | | |

## 4.3 Deceptive Behavior

| Check | Status | Notes |
|---|---|---|
| App does exactly what the store listing claims | | |
| App does not impersonate other apps, developers, or entities | | |
| App does not use misleading app icons, titles, or descriptions | | |
| App does not artificially inflate installs, ratings, or reviews | | |
| No unauthorized use of trademarked or copyrighted material | | |
| "Contains ads" label set in Play Console if app displays ads | | |
| Ads within app comply with Google Play Ads Policy: no deceptive ads, no ads that interfere with navigation | | |

## 4.4 Monetization & Payments

| Check | Status | Notes |
|---|---|---|
| Digital goods and services purchased within the app use Google Play Billing | | |
| App does not direct users outside to purchase digital goods usable in-app | | |
| Subscriptions clearly disclose: price, billing period, free trial terms, cancellation policy | | |
| Subscription pricing not misrepresented (e.g., showing "per week" price for annual plan) | | |
| Physical goods/services may use alternative payment methods | | |
| Real-money gambling apps geo-restricted and licensed | | |

## 4.5 Families Policy (if targeting children)

| Check | Status | Notes |
|---|---|---|
| Target audience age group accurately set in Play Console | | |
| No interest-based advertising to users under 13 | | |
| All ad SDKs used are Google Play Families-certified | | |
| App content appropriate for declared age range | | |
| No collection of device serial number, SIM serial, IMEI beyond what's permitted | | |
| Prominent parental consent mechanism before collecting data from children | | |

## 4.6 Technical Requirements

| Check | Status | Notes |
|---|---|---|
| Target SDK level meets current Google Play requirements (verify with Phase 1 fetch) | | |
| 64-bit support included in APK/AAB | | |
| App does not use non-SDK (private) APIs | | |
| App does not download executable code after installation | | |
| App has functional back button behavior on Android | | |
| Notification permission requested contextually — not at first launch (Android 13+) | | |
| QUERY_ALL_PACKAGES permission only if core functionality requires it | | |
| VPN Service apps: primary VPN app, not bundled with unrelated functionality | | |
| Accessibility Services declared only if app is genuinely an accessibility tool | | |

## 4.7 App-Type-Specific Play Policies

Apply only the subsections relevant to this app.

**Financial Services**
- [ ] APR disclosed in app listing description
- [ ] Minimum and maximum repayment period disclosed
- [ ] No personal loan repayment period under 60 days
- [ ] Physical address provided

**Health / Medical**
- [ ] No sale or facilitation of prescription drugs without verified license
- [ ] Prescription drug apps verified by Google

**News Aggregation**
- [ ] Content clearly attributed to original publisher
- [ ] Does not misrepresent aggregated content as original reporting
