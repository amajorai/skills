# Platform Icon Generation Commands

In every command below, replace `<SOURCE_IMAGE>` with the validated source image path from Step 1. Quote it if it contains spaces.

Detect package manager once before running any install commands:
```bash
command -v bun >/dev/null 2>&1 && PM=bun || (command -v pnpm >/dev/null 2>&1 && PM=pnpm || PM=npm)
```

---

### Tauri

```bash
# Install Tauri CLI if not present
$PM add -D @tauri-apps/cli

# Generate all icon sizes (outputs to src-tauri/icons/)
npx tauri icon "<SOURCE_IMAGE>"
```

This generates: `.icns` (macOS), `.ico` (Windows), multiple `.png` sizes (Linux, tray, etc.) all placed in `src-tauri/icons/`. No config changes needed: Tauri reads from that directory automatically.

Verify output:
```bash
ls src-tauri/icons/
```


### Capacitor (iOS + Android)

```bash
# Install the assets package
$PM add -D @capacitor/assets

# Place source image: assets tool expects resources/ directory
mkdir -p resources
cp "<SOURCE_IMAGE>" resources/icon.png
```

Ask the user for background colors (used for adaptive icons and splash screens):

> "What background color should the icon use? (e.g. `#FFFFFF` for white). Do you want a different color for dark mode?"

Then run:
```bash
npx @capacitor/assets generate \
  --iconBackgroundColor '#FFFFFF' \
  --iconBackgroundColorDark '#111111' \
  --splashBackgroundColor '#FFFFFF' \
  --splashBackgroundColorDark '#111111'
```

This generates icons for both iOS (`ios/App/App/Assets.xcassets/`) and Android (`android/app/src/main/res/`).

If `ios/` or `android/` folders don't exist yet, tell the user to run `npx cap add ios` / `npx cap add android` first.


### Expo

```bash
# Ensure expo-splash-screen is installed
npx expo install expo-splash-screen

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
npx expo prebuild --clean
```
(This regenerates native folders with correct icon sizes.)


### Electron

```bash
# Install electron-icon-maker
$PM add -D electron-icon-maker

# Generate all sizes (outputs to ./icons/)
npx electron-icon-maker --input="<SOURCE_IMAGE>" --output=./

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
$PM add -D pwa-asset-generator

# Generate all PWA icons + favicons (outputs to public/icons/)
npx pwa-asset-generator "<SOURCE_IMAGE>" public/icons \
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
