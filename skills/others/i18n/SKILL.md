---
name: i18n
description: Add internationalization to any web or mobile app. Extracts strings, sets up locale routing, wires up a translation library, and connects to a translation workflow. Use when the app needs to support more than one language.
argument-hint: <target locales, e.g. "en, es, fr, de">
---

# i18n

You are adding internationalization support. Work through each phase in order.

**Target locales:** {{args}}


## Phase 0: Auto-Update

*Skip if `{{args}}` contains `--no-update`, or if `SKILLS_AUTO_UPDATE: false` is set in your project CLAUDE.md.*

```bash
npx skills update i18n -y
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Phase 1: Interview

Ask the user (combine related questions):

- **Locales**: Which languages? Is one the default/fallback?
- **Library**: `next-intl` (Next.js), `react-i18next`, `i18next`, `paraglide` (Inlang, recommended for type-safety)?
- **Routing**: Locale in URL (`/en/about`, `/fr/about`) or subdomain (`fr.yourdomain.com`)?
- **Translation workflow**: Self-translated, Google Translate as a starting point, or connecting to Crowdin/Lokalise?
- **RTL**: Any right-to-left locales (Arabic, Hebrew)?

Recommend: **Paraglide** for new projects (compile-time, zero runtime overhead, fully type-safe). **next-intl** for Next.js projects that want ecosystem familiarity.


## Phase 2: Explore

Spawn **2 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | All hardcoded user-facing strings in UI components, page titles, error messages, email templates |
| 2 | Routing structure, layout files, any existing i18n setup |

Produce a string inventory: estimated count of strings, location of most-used ones.


## Phase 3: Setup

### Paraglide (recommended)

```bash
bun add @inlang/paraglide-js
bunx paraglide-js init
```

Creates:
- `messages/en.json` (and one per locale)
- `src/lib/i18n.js` - the compiled message functions

### next-intl

```bash
bun add next-intl
```

Creates:
- `messages/en.json`
- `i18n.ts` - locale configuration
- Middleware for locale routing

### Locale routing

Set up locale-aware routing so `/en/about` and `/fr/about` work. Configure:
- Default locale redirect (e.g., `/` → `/en/`)
- Locale detection from `Accept-Language` header
- Locale switcher component


## Phase 4: Extract Strings

For each hardcoded string found in Phase 2:

1. Add it to `messages/en.json` with a descriptive key:
   ```json
   { "auth.login.title": "Welcome back", "auth.login.cta": "Sign in" }
   ```
2. Replace the hardcoded string with the translation function:
   ```typescript
   // Paraglide
   import * as m from '$lib/paraglide/messages'
   <h1>{m.auth_login_title()}</h1>
   
   // next-intl
   const t = useTranslations('auth.login')
   <h1>{t('title')}</h1>
   ```

Work through files systematically. Do not leave hardcoded strings: they will show in English to non-English users.


## Phase 5: Translate

For each non-English locale:

1. Copy `messages/en.json` to `messages/<locale>.json`
2. Translate all values (keep keys in English)
3. For machine translation as a starting point:
   - Use the DeepL API or Google Translate API to generate initial translations
   - Flag all machine-translated strings for human review
4. Preserve format strings: `"greeting": "Hello, {name}!"` → `"greeting": "Hola, {name}!"`


## Phase 6: Plurals & Formatting

Handle edge cases:

- **Plurals**: `{ count, plural, one {# item} other {# items} }` (ICU format)
- **Numbers**: Use `Intl.NumberFormat` with the current locale
- **Dates**: Use `Intl.DateTimeFormat` with the current locale
- **Currency**: Use `Intl.NumberFormat` with `style: 'currency'`
- **RTL**: Add `dir="rtl"` to `<html>` for RTL locales; verify layout doesn't break


## Phase 7: Locale Switcher

Build a locale switcher component:
- Dropdown or flag buttons showing available locales
- Switching preserves the current page (changes locale prefix in URL)
- Persists preference to localStorage or cookie
- Place in the app's navigation


## Phase 8: Verify

- [ ] Switching locale changes all UI strings
- [ ] Default locale loads without redirect loop
- [ ] All user-facing strings are translated (no English leaking into French UI)
- [ ] Format strings with variables render correctly
- [ ] Plurals work correctly in each locale
- [ ] Locale switcher persists across navigation
- [ ] Page `<title>` and `<meta>` tags are also translated


## Completion Report

- Locales supported (list)
- String count extracted and translated
- Library and routing strategy used
- Machine-translated locales flagged for human review
- Locale switcher location
