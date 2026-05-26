---
name: push-notifications
description: Add push notifications to any web or mobile app. Covers web push (service worker), Expo push for React Native, and a notification preferences UI. Use when the app needs to re-engage users outside of their active session.
argument-hint: <platform: web | expo | both>
---

# Push Notifications

You are implementing push notifications end-to-end. Work through each phase in order.

**Platform:** {{args}}


## Phase 1: Interview

Ask the user (combine related questions):

- **Platform**: Web push (PWA/browser), Expo (React Native), or both?
- **Notification types**: Which events trigger notifications? (new message, activity on your post, billing reminder, product update)
- **Opt-in strategy**: Ask permission immediately, or wait for a meaningful moment (after user completes first action)?
- **Backend**: Does a backend exist for sending notifications server-side?


## Phase 2: Explore

Spawn **1 subagent** to:
- Find the app's trigger points (where events occur that should send notifications)
- Check for existing service worker setup (web) or Expo config
- Identify the user model for storing push tokens


## Phase 3: Web Push Setup

```bash
command -v bun >/dev/null 2>&1 && PM=bun || (command -v pnpm >/dev/null 2>&1 && PM=pnpm || PM=npm)
```

#### Service Worker

Create `public/sw.js`:

```javascript
self.addEventListener('push', (event) => {
  const data = event.data.json()
  event.waitUntil(
    self.registration.showNotification(data.title, {
      body: data.body,
      icon: '/icon-192.png',
      badge: '/badge-72.png',
      data: { url: data.url },
    })
  )
})

self.addEventListener('notificationclick', (event) => {
  event.notification.close()
  event.waitUntil(clients.openWindow(event.notification.data.url))
})
```

Register the service worker on app start and keep the registration reference (you need it to subscribe later):
```typescript
let swRegistration: ServiceWorkerRegistration | undefined
if ('serviceWorker' in navigator) {
  swRegistration = await navigator.serviceWorker.register('/sw.js')
}
```

#### VAPID Keys

Generate VAPID keys (one-time setup):
```bash
npx web-push generate-vapid-keys
```

Store as env vars: `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `VAPID_SUBJECT` (mailto:you@domain.com)

#### Permission & Subscription

`applicationServerKey` must be a `Uint8Array`, not the raw base64 string. Convert the VAPID public key first:

```typescript
function urlBase64ToUint8Array(base64: string): Uint8Array {
  const padding = '='.repeat((4 - (base64.length % 4)) % 4)
  const normalized = (base64 + padding).replace(/-/g, '+').replace(/_/g, '/')
  const raw = atob(normalized)
  return Uint8Array.from([...raw].map((c) => c.charCodeAt(0)))
}

const permission = await Notification.requestPermission()
if (permission === 'granted' && swRegistration) {
  const sub = await swRegistration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY),
  })
  await fetch('/api/push/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(sub),
  })
}
```

Save subscription to DB linked to the user.


## Phase 4: Expo Push Setup

```bash
command -v bun >/dev/null 2>&1 && PM=bun || (command -v pnpm >/dev/null 2>&1 && PM=pnpm || PM=npm)
$PM add expo-notifications
```

In `app.json`:
```json
{ "expo": { "plugins": ["expo-notifications"] } }
```

Get and store the push token:
```typescript
const token = (await Notifications.getExpoPushTokenAsync()).data
await fetch('/api/push/subscribe', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ token }),
})
```

Request permission on iOS (Android grants automatically):
```typescript
await Notifications.requestPermissionsAsync()
```


## Phase 5: Database

Store push subscriptions:

```sql
CREATE TABLE push_subscriptions (
  id         TEXT PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id    TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  platform   TEXT NOT NULL,  -- 'web' or 'expo'
  token      TEXT NOT NULL,  -- JSON for web push, string for Expo
  created_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, token)
);
```


## Phase 6: Server-Side Sending

Install: `$PM add web-push` (for web). Expo uses their own HTTP API directly, so no extra package is required.

Create `lib/notifications.ts`:

```typescript
async function sendPushNotification(userId: string, notification: {
  title: string
  body: string
  url?: string
}) {
  const subs = await db.query('SELECT * FROM push_subscriptions WHERE user_id = $1', [userId])
  for (const sub of subs) {
    if (sub.platform === 'web') {
      await webpush.sendNotification(JSON.parse(sub.token), JSON.stringify(notification))
    } else {
      // Expo send API
      await fetch('https://exp.host/--/api/v2/push/send', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Accept: 'application/json',
        },
        body: JSON.stringify({ to: sub.token, ...notification }),
      })
    }
  }
}
```

Handle expired/invalid tokens: catch `410 Gone` errors and delete the subscription from DB.


## Phase 7: Notification Preferences UI

Build a preferences page where users can:
- See which notification types are enabled
- Toggle individual notification types on/off
- See which devices have push enabled
- Revoke push permission per device

Store preferences in a `notification_preferences` table linked to the user.


## Phase 8: Permission UX

Do not ask for push permission on page load: this results in high denial rates.

Instead:
1. Wait until the user completes a meaningful action (posts content, receives a message)
2. Show an in-app prompt explaining the value: "Get notified when someone replies to your post"
3. Only call `Notification.requestPermission()` after the user clicks "Enable notifications"


## Phase 9: Verify

- [ ] Service worker registers without errors (check Application tab in DevTools)
- [ ] Permission prompt appears at the right moment
- [ ] Subscribing saves to the database
- [ ] Sending a test notification from the server delivers to the device
- [ ] Clicking the notification opens the correct URL
- [ ] Preference toggles correctly suppress/allow notifications
- [ ] Expired tokens are cleaned up automatically


## Completion Report

- Platforms supported (web/Expo)
- Notification types implemented (list)
- Permission prompt placement
- Server send utility created
- Preference UI built
- Env vars required (names only)
