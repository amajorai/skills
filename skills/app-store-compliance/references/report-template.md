# Compliance Report Template

Save output as `app-store-compliance-report.md` in the current working directory.

Use exact values from the audit — no placeholder text. Fill every section; mark "None" if a list is empty.

---

```markdown
# App Store Compliance Report
## App: [App Name]
## Generated: [today's date]
## Guidelines Version: [version/date retrieved in Phase 1]


## Executive Summary

[2–3 sentence summary of overall compliance status and most urgent issues]


## Apple App Store: [PASS / CONDITIONAL / FAIL]

### Critical Issues (will cause rejection)
[Numbered list — "None" if empty]

### Risks (may cause rejection or future policy action)
[Numbered list — "None" if empty]

### Recommendations (best practice, not blocking)
[Numbered list — "None" if empty]

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
[Numbered list — "None" if empty]

### Risks (may trigger policy warning or future removal)
[Numbered list — "None" if empty]

### Recommendations
[Numbered list — "None" if empty]

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
| 1 | | iOS/Android/Both | Blocking | |


## Metadata to Complete

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
- [ ] Run through the app as a first-time user: onboarding must be clear without external context


## Resources

- Apple rejection appeals: https://developer.apple.com/app-store/review/#common-app-rejections
- Apple review guidelines: https://developer.apple.com/app-store/review/guidelines/
- Google Play policy center: https://support.google.com/googleplay/android-developer/topic/9858052
- Google Play appeals: https://support.google.com/googleplay/android-developer/answer/9899234
```
