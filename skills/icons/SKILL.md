---
name: icons
description: Generate app icons, favicons, and splash screens for Tauri, PWA, Capacitor (iOS/Android), Expo, Electron, and web projects from a single source image. Detects project type automatically, validates the source image, runs the right CLI command, and wires up the output.
argument-hint: [path/to/source-image.png]
---

# Icons

You are generating all platform icons from a single source image. Work through each step in order.

**Source image:** {{args}}


## Phase 0: Auto-Update

*Skip unless `{{args}}` contains `--update`, or `SKILLS_AUTO_UPDATE: true` is set in your project CLAUDE.md.*

```bash
npx --yes skills update amajorai/skills -y 2>/dev/null || true
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Step 1: Locate the Source Image

If `{{args}}` is empty or no image path was provided, ask the user:

> "Where is your source logo or icon? Ideally a **1024×1024 PNG** (or larger square PNG/SVG). What's the file path?"

Once you have a path, verify it exists and read its dimensions if possible:

```bash
# Check file exists
ls "{{args}}"
```

**Minimum requirements for source image:**
- Format: PNG (preferred), SVG, or JPG
- Size: at least 512×512px; ideally 1024×1024px or larger
- Shape: square (1:1 aspect ratio)

If the image is too small or non-square, warn the user and ask if they want to continue anyway.


## Step 2: Detect Project Type

Check for platform config files (run all checks; missing files are expected and harmless):

```bash
# Check all at once
ls tauri.conf.json src-tauri/tauri.conf.json 2>/dev/null
ls capacitor.config.ts capacitor.config.js capacitor.config.json 2>/dev/null
ls app.json app.config.ts app.config.js 2>/dev/null
ls electron-builder.yml electron-builder.json package.json 2>/dev/null
ls vite.config.ts next.config.ts next.config.js astro.config.mjs 2>/dev/null
ls index.html public/index.html 2>/dev/null
```

Identify which platforms apply: a project can have **multiple** (e.g. Tauri + PWA, Capacitor + PWA):

| File found | Platform |
|---|---|
| `src-tauri/tauri.conf.json` or `tauri.conf.json` | **Tauri** (desktop) |
| `capacitor.config.*` | **Capacitor** (iOS + Android) |
| `app.json` / `app.config.*` with `expo` key | **Expo** (iOS + Android) |
| `electron-builder.*` | **Electron** (desktop) |
| `vite.config.*` with `VitePWA` / `next.config.*` / `astro.config.*` / any web `index.html` | **PWA + Favicon** (web) |
| No matches | Ask user which platform(s) to target |


## Step 3: Generate Icons Per Platform

Run the relevant sections below. If multiple platforms apply, do them in parallel.

Replace `<SOURCE_IMAGE>` with the validated source image path from Step 1 (the value of `{{args}}`). Quote it if it contains spaces.

See [references/platform-commands.md](references/platform-commands.md) for the full generation commands per platform (Tauri, Capacitor, Expo, Electron, PWA/Web).


## Step 4: Verify Output

After generation, list what was created:

```bash
# Show generated files per platform
ls src-tauri/icons/ 2>/dev/null       # Tauri
ls resources/ 2>/dev/null              # Capacitor source
ls ios/App/App/Assets.xcassets/ 2>/dev/null  # Capacitor iOS
ls android/app/src/main/res/ 2>/dev/null     # Capacitor Android
ls assets/ 2>/dev/null                 # Expo
ls build/ icons/ 2>/dev/null           # Electron
ls public/icons/ 2>/dev/null           # PWA/Web
```

Report back:
- Which platforms were processed
- How many icon files were generated per platform
- Any warnings (image too small, missing native folders, etc.)
- What the user should do next (e.g. rebuild the app, run `bunx cap sync`)


## Common Issues

| Problem | Fix |
|---|---|
| `tauri icon` not found | Run `bun add -D @tauri-apps/cli` first |
| Capacitor: no iOS/Android folder | Run `bunx cap add ios` / `bunx cap add android` |
| Image not square | Crop to square before running (ask user to provide a square image) |
| SVG source with `pwa-asset-generator` | Add `--type png` flag |
| Expo: icons look blurry | Source image must be at least 1024×1024px |
| `@capacitor/assets` - "resources/icon.png not found" | Ensure you copied the source image to `resources/icon.png` |
