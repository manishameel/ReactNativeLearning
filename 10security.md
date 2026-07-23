# Security in React Native

Mobile apps handle sensitive data — passwords, tokens, personal info. This covers how to keep it safe across 4 areas.

---

## 1. Authentication

**What it is:** Verifying who the user is, and keeping them logged in safely (without storing passwords insecurely).

**Where to use:** Login flows, protected screens, API calls that need to identify the user.

### Basic token-based login flow

```jsx
import * as SecureStore from 'expo-secure-store';

async function login(email, password) {
  const response = await fetch('https://api.example.com/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  });

  const { accessToken, refreshToken } = await response.json();

  // Store tokens securely (NOT in plain AsyncStorage!)
  await SecureStore.setItemAsync('accessToken', accessToken);
  await SecureStore.setItemAsync('refreshToken', refreshToken);
}
```

### Biometric login (Face ID / Fingerprint)

```jsx
import * as LocalAuthentication from 'expo-local-authentication';

async function loginWithBiometrics() {
  const hasHardware = await LocalAuthentication.hasHardwareAsync();
  const isEnrolled = await LocalAuthentication.isEnrolledAsync();

  if (!hasHardware || !isEnrolled) {
    alert('Biometric login not available on this device');
    return;
  }

  const result = await LocalAuthentication.authenticateAsync({
    promptMessage: 'Login with Face ID / Fingerprint',
  });

  if (result.success) {
    // proceed — usually to unlock a previously-stored token
  }
}
```

### 🧑‍💻 Professional usage:
- Never stores raw passwords, ever — only short-lived **access tokens** + long-lived **refresh tokens**
- Implements **automatic token refresh**: when an API call returns 401 (unauthorized), silently calls the refresh endpoint and retries, instead of forcing a re-login
- Uses **biometrics to unlock a locally stored token**, not as the sole auth method — biometric success just decrypts/reveals the already-issued token
- Adds **certificate/SSL pinning** (see Networking below) especially for banking/fintech apps, so auth tokens can't be intercepted even on compromised networks
- Logs the user out and clears all stored tokens on logout — including calling a backend "revoke token" endpoint, not just clearing local storage

---

## 2. Networking (Security Side)

**What it is:** Making sure data traveling between your app and your server can't be read or tampered with by anyone in between.

**Where to use:** Every API call — but especially for sensitive data (payments, personal info, auth).

### Always use HTTPS

```jsx
// ❌ Never
fetch('http://api.example.com/data');

// ✅ Always
fetch('https://api.example.com/data');
```

Android blocks plain HTTP by default (`cleartextTraffic`) unless explicitly allowed — don't turn this off for production.

### Adding an auth header safely

```jsx
async function apiClient(endpoint, options = {}) {
  const token = await SecureStore.getItemAsync('accessToken');

  return fetch(`https://api.example.com${endpoint}`, {
    ...options,
    headers: {
      ...options.headers,
      Authorization: `Bearer ${token}`,
    },
  });
}
```

### 🧑‍💻 Professional usage:
- Uses **certificate pinning** (`react-native-ssl-pinning` or native config) for high-security apps — the app only trusts a specific certificate, so even a fake "trusted" certificate on a hacked network can't intercept traffic (defends against man-in-the-middle attacks)
- Never hardcodes API keys/secrets directly in the JS bundle — JS code in a mobile app can be decompiled and read by anyone. Sensitive secrets stay on the backend; the app only holds short-lived tokens
- Validates and sanitizes all server responses before using them (never blindly trusts data, even from your own backend, in case of a compromised response)
- Sets sensible request timeouts and doesn't leak sensitive error details (stack traces, internal IDs) in error messages shown to users

---

## 3. Storage

**What it is:** Where and how you save data on the device — and the difference between storage that's safe for secrets vs storage that isn't.

**Where to use:** Tokens, passwords, PII → secure storage. Preferences, cached non-sensitive data → regular storage.

| Storage Type | Encrypted? | Use For |
|---|---|---|
| `AsyncStorage` | ❌ No (plain text) | Non-sensitive data: theme preference, onboarding-seen flag |
| `expo-secure-store` / `react-native-keychain` | ✅ Yes (uses device Keychain/Keystore) | Tokens, passwords, sensitive personal data |

```jsx
import AsyncStorage from '@react-native-async-storage/async-storage';
import * as SecureStore from 'expo-secure-store';

// ❌ Wrong — token in plain, unencrypted storage
await AsyncStorage.setItem('accessToken', token);

// ✅ Right — encrypted storage
await SecureStore.setItemAsync('accessToken', token);

// Reading back
const token = await SecureStore.getItemAsync('accessToken');

// Non-sensitive data is fine in AsyncStorage
await AsyncStorage.setItem('theme', 'dark');
```

### 🧑‍💻 Professional usage:
- Draws a clear line in code review: **anything that identifies or authenticates a user never touches `AsyncStorage`**
- Uses `react-native-keychain` (bare RN) or `expo-secure-store` (Expo) — both use the OS-level Keychain (iOS) / Keystore (Android), which is hardware-backed encryption, not just app-level obfuscation
- Clears secure storage completely on logout and on detecting a compromised/jailbroken device (using a library like `jail-monkey`)
- Avoids storing more than necessary — e.g. doesn't cache full user profile with sensitive fields locally "just in case"; fetches fresh from server when needed

---

## 4. Permissions

**What it is:** Asking the user for access to device features — camera, location, contacts, notifications, etc.

**Where to use:** Any feature that needs sensitive device access.

```jsx
import * as ImagePicker from 'expo-image-picker';

async function pickImage() {
  const { status } = await ImagePicker.requestCameraPermissionsAsync();

  if (status !== 'granted') {
    alert('Camera permission is required to take a photo');
    return;
  }

  const result = await ImagePicker.launchCameraAsync();
  // use result
}
```

**Checking without prompting (to adjust UI):**
```jsx
import * as ImagePicker from 'expo-image-picker';

const { status } = await ImagePicker.getCameraPermissionsAsync();
// status: 'granted' | 'denied' | 'undetermined'
```

### 🧑‍💻 Professional usage:
- **Asks for permission contextually**, right when the feature is used — not all at once on app launch (which tanks grant rates and feels invasive)
- Shows a **custom pre-permission explainer screen** before the native OS prompt appears ("We use your camera to scan receipts") — because once a user hits "Don't Allow" on the native prompt, you usually can't ask again programmatically; you only get one shot
- Handles the **"denied forever"** case gracefully — detects when a permission was permanently denied and shows a message with a button that opens the device Settings app (`Linking.openSettings()`) instead of repeatedly failing silently
- Declares only the permissions actually used in `app.json`/`Info.plist`/`AndroidManifest.xml` — app stores (especially Apple) reject or flag apps requesting unused/unexplained permissions
- Writes clear, honest permission usage descriptions (`NSCameraUsageDescription`, etc.) — vague or missing descriptions are a common App Store rejection reason

---

## Quick Reference

| Area | Key Rule | Tool |
|---|---|---|
| Authentication | Never store raw passwords; auto-refresh tokens | Token-based auth + `expo-local-authentication` |
| Networking | Always HTTPS; pin certificates for high-security apps | `fetch` + SSL pinning |
| Storage | Secrets go in encrypted storage, never AsyncStorage | `expo-secure-store` / `react-native-keychain` |
| Permissions | Ask contextually, handle denial gracefully | Platform-specific permission APIs |