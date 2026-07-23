# Deep Linking in React Native

**What it is:** Opening your app directly to a *specific screen* using a link — instead of just opening to the home screen. Like clicking a link and landing straight on a product page, not the store's homepage.

**Where to use:** Push notification taps, shared links (WhatsApp/email), marketing campaigns, "Open in App" buttons on your website, password reset emails.

---

## Types of Deep Links

| Type | Example | Notes |
|---|---|---|
| **Custom URL Scheme** | `myapp://profile/42` | Simple, but only works if app is installed; browsers may warn/block it |
| **Universal Link (iOS) / App Link (Android)** | `https://myapp.com/profile/42` | Looks like a normal web link. If app is installed → opens app. If not → opens website. This is the modern standard. |

---

## 1. Basic Setup

**Expo (easiest):**
```json
// app.json
{
  "expo": {
    "scheme": "myapp",
    "ios": {
      "associatedDomains": ["applinks:myapp.com"]
    },
    "android": {
      "intentFilters": [
        {
          "action": "VIEW",
          "data": [{ "scheme": "https", "host": "myapp.com", "pathPrefix": "/" }],
          "category": ["BROWSABLE", "DEFAULT"]
        }
      ]
    }
  }
}
```

**Bare React Native:** you configure this natively — `Info.plist` + associated domain file for iOS, `AndroidManifest.xml` intent filters for Android.

---

## 2. Wiring it to React Navigation

```jsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

const linking = {
  prefixes: ['myapp://', 'https://myapp.com'],
  config: {
    screens: {
      Home: '',
      Profile: 'profile/:userId',       // myapp://profile/42
      Post: 'post/:postId',             // myapp://post/99
      Settings: 'settings',
    },
  },
};

export default function App() {
  return (
    <NavigationContainer linking={linking}>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
        <Stack.Screen name="Post" component={PostScreen} />
        <Stack.Screen name="Settings" component={SettingsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

Now `myapp://profile/42` or `https://myapp.com/profile/42` will automatically open `ProfileScreen` with `route.params.userId === '42'`.

```jsx
function ProfileScreen({ route }) {
  const { userId } = route.params;
  return <Text>Viewing profile {userId}</Text>;
}
```

---

## 3. Manually Listening for Links (without navigation config)

Useful if you want custom handling instead of automatic routing.

```jsx
import * as Linking from 'expo-linking';
import { useEffect } from 'react';

function useDeepLinkListener() {
  useEffect(() => {
    // App was opened FROM a link (cold start)
    Linking.getInitialURL().then(url => {
      if (url) handleDeepLink(url);
    });

    // App was already open in background, link came in
    const subscription = Linking.addEventListener('url', ({ url }) => {
      handleDeepLink(url);
    });

    return () => subscription.remove();
  }, []);
}

function handleDeepLink(url) {
  const { path, queryParams } = Linking.parse(url);
  console.log('Path:', path, 'Params:', queryParams);
}
```

---

## 4. Triggering a Link Yourself (testing / sharing)

```jsx
import * as Linking from 'expo-linking';

// Build a link
const url = Linking.createURL('profile/42');
// → myapp://profile/42

// Open it (for testing)
Linking.openURL(url);
```

**Testing on device/simulator via terminal:**
```bash
npx uri-scheme open myapp://profile/42 --ios
npx uri-scheme open myapp://profile/42 --android
```

---

## 🧑‍💻 How a Professional Developer Uses This

- **Always sets up Universal/App Links, not just custom schemes.** A custom scheme (`myapp://`) breaks the flow when the app isn't installed — the link just fails silently. A Universal Link gracefully falls back to a web page (often a smart landing page prompting "Download the app").
- **Hosts the required verification files** (`apple-app-site-association` for iOS, `assetlinks.json` for Android) on their own domain server — this proves to iOS/Android that the app is allowed to handle that domain's links. Without this, associated domains don't work.
- **Handles auth-gated deep links properly**: if a link points to a screen that needs login (e.g. `myapp://order/55`), the pro stores the intended destination, sends the user through login first, then navigates to the original target after auth succeeds — instead of just dropping the link.
- **Centralizes link → screen mapping** in one config file/module, so marketing/backend teams have one source of truth for what links exist, instead of deep link routes being scattered across the codebase.
- **Uses `navigationRef`** (a ref set up outside of React Navigation's component tree) so deep links arriving from push notifications or other non-UI code can still trigger navigation, even before the app has fully mounted.
- **Tests all 3 app states** — link tapped while app is: closed (cold start), backgrounded, and already open — since each goes through a different code path (`getInitialURL` vs the `url` event listener).
- **Adds analytics/attribution** on deep link opens (which campaign, which link, which screen) — often via a service like Branch.io or Firebase Dynamic Links successor tools, especially for marketing-driven apps where tracking link performance matters.
- **Falls back gracefully** — if a deep link points to content that's deleted or doesn't exist (e.g. a post that was removed), it navigates to a friendly "not found" screen instead of crashing.

---

## Quick Reference

| Task | Tool |
|---|---|
| Basic setup (scheme + domains) | `app.json` (Expo) or native config (bare) |
| Auto-route links to screens | React Navigation `linking` config |
| Manual link handling | `expo-linking` / `Linking` API |
| Test links locally | `npx uri-scheme open <url>` |
| Attribution/analytics on links | Branch.io or similar |