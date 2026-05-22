---
name: icons
description: Generate app icons, favicons, and splash screens for Tauri, PWA, Capacitor (iOS/Android), Expo, Electron, and web projects from a single source image. Detects project type automatically, validates the source image, runs the right CLI command, and wires up the output.
argument-hint: [path/to/source-image.png]
---

# Icons

You are generating all platform icons from a single source image. Work through each step in order.

**Source image:** {{args}}


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

Spawn parallel subagents to check for platform config files:

```bash
# Check all at once
ls tauri.conf.json src-tauri/tauri.conf.json 2>$null
ls capacitor.config.ts capacitor.config.js capacitor.config.json 2>$null
ls app.json app.config.ts app.config.js 2>$null
ls electron-builder.yml electron-builder.json package.json 2>$null
ls vite.config.ts next.config.ts next.config.js astro.config.mjs 2>$null
ls index.html public/index.html 2>$null
```

Identify which platforms apply — a project can have **multiple** (e.g. Tauri + PWA, Capacitor + PWA):

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


### Tauri

```bash
# Install Tauri CLI if not present
bun add -D @tauri-apps/cli

# Generate all icon sizes (outputs to src-tauri/icons/)
bunx tauri icon "<SOURCE_IMAGE>"
```

This generates: `.icns` (macOS), `.ico` (Windows), multiple `.png` sizes (Linux, tray, etc.) all placed in `src-tauri/icons/`. No config changes needed — Tauri reads from that directory automatically.

Verify output:
```bash
ls src-tauri/icons/
```


### Capacitor (iOS + Android)

```bash
# Install the assets package
bun add -D @capacitor/assets

# Place source image — assets tool expects resources/ directory
# Create resources/ with icon and splash source images
mkdir -p resources
cp "<SOURCE_IMAGE>" resources/icon.png
```

Ask the user for background colors (used for adaptive icons and splash screens):

> "What background color should the icon use? (e.g. `#FFFFFF` for white). Do you want a different color for dark mode?"

Then run:
```bash
bunx @capacitor/assets generate \
  --iconBackgroundColor '#FFFFFF' \
  --iconBackgroundColorDark '#111111' \
  --splashBackgroundColor '#FFFFFF' \
  --splashBackgroundColorDark '#111111'
```

This generates icons for both iOS (`ios/App/App/Assets.xcassets/`) and Android (`android/app/src/main/res/`).

If `ios/` or `android/` folders don't exist yet, tell the user to run `bunx cap add ios` / `bunx cap add android` first.


### Expo

```bash
# Ensure expo-splash-screen is installed
bunx expo install expo-splash-screen

# Create assets directory
mkdir -p assets

# Copy source as the app icon (Expo uses 1024x1024)
cp "<SOURCE_IMAGE>" assets/icon.png
# Copy as adaptive icon foreground (Android)
cp "<SOURCE_IMAGE>" assets/adaptive-icon.png
# Copy as splash screen (recommend a centered logo on white bg)
cp "<SOURCE_IMAGE>" assets/splash-icon.png
```

Then update `app.json` / `app.config.ts` to reference these:

```json
{
  "expo": {
    "icon": "./assets/icon.png",
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      }
    },
    "splash": {
      "image": "./assets/splash-icon.png",
      "resizeMode": "contain",
      "backgroundColor": "#FFFFFF"
    }
  }
}
```

For actual icon resizing across all required sizes, run:
```bash
bunx expo prebuild --clean
```
(This regenerates native folders with correct icon sizes.)


### Electron

```bash
# Install electron-icon-maker
bun add -D electron-icon-maker

# Generate all sizes (outputs to ./icons/)
bunx electron-icon-maker --input="<SOURCE_IMAGE>" --output=./

# Or if using electron-builder, place icon at:
cp "<SOURCE_IMAGE>" build/icon.png   # Linux
# electron-builder auto-generates .icns / .ico from build/icon.png
```

If using `electron-builder`, verify `package.json` has:
```json
{
  "build": {
    "icon": "build/icon.png"
  }
}
```


### PWA + Favicon (Web)

```bash
# Install pwa-asset-generator
bun add -D pwa-asset-generator

# Generate all PWA icons + favicons (outputs to public/icons/)
bunx pwa-asset-generator "<SOURCE_IMAGE>" public/icons \
  --favicon \
  --manifest public/manifest.json \
  --index public/index.html \
  --padding "10%"
```

This generates:
- `favicon.ico` + multiple `favicon-*.png` sizes
- PWA manifest icons at 192×192, 512×512, maskable variants
- Injects `<link rel="icon">` and `<link rel="apple-touch-icon">` into your HTML
- Updates `manifest.json` with icon entries

If the project uses Vite + `vite-plugin-pwa`, also update `vite.config.ts`:
```ts
VitePWA({
  manifest: {
    icons: [
      { src: '/icons/pwa-192x192.png', sizes: '192x192', type: 'image/png' },
      { src: '/icons/pwa-512x512.png', sizes: '512x512', type: 'image/png' },
      { src: '/icons/maskable-icon-512x512.png', sizes: '512x512', type: 'image/png', purpose: 'maskable' }
    ]
  }
})
```

For **Next.js**, place `favicon.ico` in `app/` (App Router) or `public/` (Pages Router) and add to `layout.tsx`:
```ts
export const metadata = {
  icons: {
    icon: '/icons/favicon.ico',
    apple: '/icons/apple-touch-icon.png',
  }
}
```


## Step 4: Verify Output

After generation, list what was created:

```bash
# Show generated files per platform
ls src-tauri/icons/ 2>$null       # Tauri
ls resources/ 2>$null              # Capacitor source
ls ios/App/App/Assets.xcassets/ 2>$null  # Capacitor iOS
ls android/app/src/main/res/ 2>$null     # Capacitor Android
ls assets/ 2>$null                 # Expo
ls build/ icons/ 2>$null           # Electron
ls public/icons/ 2>$null           # PWA/Web
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
| `@capacitor/assets`: "resources/icon.png not found" | Ensure you copied the source image to `resources/icon.png` |
