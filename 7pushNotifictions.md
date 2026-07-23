# Push Notifications in React Native

There are two different things people call "push notifications" — it's important to know the difference first.

| Type | What it means | Needs a server? |
|---|---|---|
| **Local notification** | App itself schedules/shows a notification (e.g. "reminder in 10 mins") | No |
| **Remote/Push notification** | A server sends a notification to the device, even if the app is closed | Yes (via FCM/APNs) |

⚠️ **Note on libraries in 2026:** Notifee (the most popular local-notification library) was archived by its maintainers. The community fork **`react-native-notify-kit`** is now the recommended drop-in replacement with the same API. For most apps, **Expo Notifications** + **Firebase Cloud Messaging (FCM)** is the standard, well-supported path.

---

## How it works (the big picture)

```
Your Server → Firebase Cloud Messaging (FCM) → Apple APNs (iOS) / Google FCM (Android) → User's Device → Your App
```

FCM acts as the universal relay — you send one request to FCM, and it delivers to both iOS and Android for you.

---

## 1. Setup — Getting a Device Token

Every device gets a unique token. Your server needs this token to send that specific device a notification.

```bash
npx expo install expo-notifications expo-device expo-constants
```

```jsx
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import { useEffect, useState } from 'react';

async function registerForPushNotifications() {
  if (!Device.isDevice) {
    alert('Must use a physical device for push notifications');
    return null;
  }

  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== 'granted') {
    alert('Failed to get push token — permission denied');
    return null;
  }

  const tokenData = await Notifications.getExpoPushTokenAsync();
  return tokenData.data; // send this to your backend, save it against the user
}
```

**Where to use:** Call this once when the user logs in (or on app start), then send the token to your backend so it can target that device later.

---

## 2. Receiving & Displaying Notifications

```jsx
import * as Notifications from 'expo-notifications';
import { useEffect } from 'react';

// Controls how notification looks when app is in FOREGROUND
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: false,
  }),
});

function App() {
  useEffect(() => {
    // Fires when a notification arrives while app is open
    const receivedListener = Notifications.addNotificationReceivedListener(notification => {
      console.log('Notification received:', notification);
    });

    // Fires when user TAPS a notification
    const responseListener = Notifications.addNotificationResponseReceivedListener(response => {
      const data = response.notification.request.content.data;
      console.log('User tapped notification:', data);
      // e.g. navigate to a specific screen based on data.screen
    });

    return () => {
      receivedListener.remove();
      responseListener.remove();
    };
  }, []);

  return null; // rest of your app
}
```

**Where to use:** Set this up once at the root of your app (e.g. `App.js`), so it's always listening.

---

## 3. Local Notifications (scheduled by the app itself)

```jsx
import * as Notifications from 'expo-notifications';

async function scheduleReminder() {
  await Notifications.scheduleNotificationAsync({
    content: {
      title: "Don't forget!",
      body: 'You have a task due today.',
      data: { screen: 'Tasks' },
    },
    trigger: { seconds: 10 }, // fires 10 seconds from now
  });
}
```

**Where to use:** Reminders, to-do alerts, "cart abandoned" nudges — anything that doesn't need a server.

---

## 4. Sending from a Server (Node.js example using FCM)

```js
// backend/sendNotification.js
const admin = require('firebase-admin');
admin.initializeApp({ credential: admin.credential.cert(serviceAccount) });

async function sendPushNotification(deviceToken, title, body, data = {}) {
  await admin.messaging().send({
    token: deviceToken,
    notification: { title, body },
    data, // custom payload, e.g. { screen: 'OrderDetails', orderId: '123' }
  });
}
```

**Where to use:** Order updates, chat message alerts, promotional campaigns — anything triggered by a backend event.

---

## 🧑‍💻 How a Professional Developer Uses This

- **Handles token refresh properly.** Tokens change on reinstall, backup restore, or rotation — a pro re-uploads the token to the backend on *every* app launch, not just first login.
- **Handles all 3 app states separately**: foreground (in-app banner), background (system tray), and killed/quit (system tray + deep-link on tap) — each behaves differently and needs testing.
- **Uses `data` payloads for navigation**, not just display text — so tapping a notification deep-links straight into the right screen (e.g. a specific chat or order), using React Navigation's `navigate()` inside the tap listener.
- **Separates concerns**: FCM/APNs handles delivery, but for rich, well-styled local notifications (custom icons, grouping, channels on Android), pairs it with `react-native-notify-kit` (the Notifee successor) instead of relying on default OS styling.
- **Sets up Android Notification Channels** properly (e.g. "Messages", "Promotions", "Order Updates") so users can control which categories they want, rather than dumping everything into one generic channel.
- **For large-scale apps**, avoids maintaining push infrastructure manually and uses a managed service like **OneSignal** — which adds a dashboard for targeting, segmentation, A/B testing, and delivery analytics on top of FCM/APNs.
- **Always asks for permission at the right moment** — not immediately on app open (which gets auto-denied a lot), but contextually (e.g. right after the user takes an action that benefits from notifications, like adding an item to a wishlist).
- **Tests badge counts and silent/data-only notifications** separately, since these are used for background syncing (e.g. refreshing content without alerting the user).

---

## Quick Reference

| Task | Tool |
|---|---|
| Get device token, listen for taps | `expo-notifications` |
| Rich local notification styling | `react-native-notify-kit` (Notifee successor) |
| Send from backend | Firebase Admin SDK (FCM) |
| Full dashboard/analytics/targeting | OneSignal |