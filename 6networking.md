# Networking in React Native

Every app needs to talk to a server — to fetch data, get real-time updates, and check if the internet is available. These three topics cover that.

---

## 1. Connectivity Status

**What it is:** The app should know whether the user's internet is ON or OFF, so it can adjust its behavior accordingly.

**Where to use:** Offline banners, disabling submit buttons when offline, retry logic, caching decisions.

React Native doesn't give internet status by default — for this we use the `@react-native-community/netinfo` library.

```bash
npm install @react-native-community/netinfo
```

```jsx
import NetInfo from '@react-native-community/netinfo';
import { useEffect, useState } from 'react';
import { View, Text } from 'react-native';

function ConnectivityBanner() {
  const [isConnected, setIsConnected] = useState(true);

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      setIsConnected(state.isConnected);
    });
    return () => unsubscribe(); // cleanup — very important!
  }, []);

  if (isConnected) return null;

  return (
    <View style={{ backgroundColor: 'red', padding: 8 }}>
      <Text style={{ color: 'white' }}>No Internet Connection</Text>
    </View>
  );
}
```

### 🧑‍💻 How a professional developer uses it:
- Doesn't just check `isConnected` — also checks `isInternetReachable` (handles the case where WiFi is connected but there's actually no internet)
- Builds a **global Context/Zustand store** (`useNetworkStatus`) for the whole app, so every component doesn't need its own NetInfo listener
- Builds a **queue system** for offline actions — actions done while offline get auto-retried once internet comes back (e.g. `react-query`'s `onlineManager` or a custom queue)
- Debounces the banner (waits 2-3 seconds) so it doesn't flicker on/off on a flaky network

---

## 2. Fetch

**What it is:** The built-in JavaScript way to request or send data to a server (GET, POST, etc.)

**Where to use:** Loading data from APIs — login, fetching lists, submitting forms.

```jsx
import { useEffect, useState } from 'react';
import { Text, ActivityIndicator } from 'react-native';

function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('https://api.example.com/user/1')
      .then(res => res.json())
      .then(data => setUser(data))
      .catch(err => console.error(err))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <ActivityIndicator />;
  return <Text>{user?.name}</Text>;
}
```

**POST request example:**

```jsx
fetch('https://api.example.com/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password }),
})
  .then(res => res.json())
  .then(data => console.log(data));
```

### 🧑‍💻 How a professional developer uses it:
- Never uses raw `fetch` directly everywhere — builds an **API wrapper/service layer** with a common baseURL, headers, and error handling:

```jsx
// api/client.js
const BASE_URL = 'https://api.example.com';

async function apiClient(endpoint, options = {}) {
  const token = await getToken(); // from secure storage

  const response = await fetch(`${BASE_URL}${endpoint}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      Authorization: token ? `Bearer ${token}` : '',
      ...options.headers,
    },
  });

  if (!response.ok) {
    throw new Error(`API Error: ${response.status}`);
  }
  return response.json();
}

export default apiClient;
```

- In production, instead of raw `fetch`, uses **React Query / TanStack Query** or **RTK Query** — because it automatically handles caching, retries, loading/error states, and re-fetching:

```jsx
import { useQuery } from '@tanstack/react-query';

function useUser(id) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => apiClient(`/user/${id}`),
    retry: 2,
    staleTime: 1000 * 60, // 1 min cache
  });
}
```

- Uses **AbortController** to cancel requests (so if the user leaves the screen, the API call gets cancelled):

```jsx
useEffect(() => {
  const controller = new AbortController();
  fetch(url, { signal: controller.signal });
  return () => controller.abort();
}, []);
```

- Also adds timeout handling, exponential backoff retries, and a global error toast inside the wrapper

---

## 3. WebSocket

**What it is:** A **real-time, two-way** connection with the server — used for chat apps, live notifications, live location tracking. Unlike fetch, you don't send a request again and again — the connection stays open.

**Where to use:** Chat messages, live scores, real-time notifications, collaborative apps.

```jsx
import { useEffect, useState } from 'react';
import { Text, View } from 'react-native';

function ChatScreen() {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const ws = new WebSocket('wss://example.com/chat');

    ws.onopen = () => console.log('Connected');
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setMessages(prev => [...prev, data]);
    };
    ws.onerror = (error) => console.log('WebSocket error:', error);
    ws.onclose = () => console.log('Disconnected');

    return () => ws.close(); // cleanup — important
  }, []);

  return (
    <View>
      {messages.map((msg, i) => (
        <Text key={i}>{msg.text}</Text>
      ))}
    </View>
  );
}
```

**Sending a message:**

```jsx
ws.send(JSON.stringify({ type: 'message', text: 'Hello!' }));
```

### 🧑‍💻 How a professional developer uses it:
- Wraps the raw WebSocket object in a **custom hook** (`useWebSocket`) so the logic is reusable
- Adds **auto-reconnect logic** — if the connection drops, it retries with exponential backoff:

```jsx
function connect() {
  const ws = new WebSocket(url);
  ws.onclose = () => {
    setTimeout(connect, reconnectDelay); // retry after delay
    reconnectDelay = Math.min(reconnectDelay * 2, 30000); // backoff cap
  };
}
```

- Sends a **heartbeat/ping-pong** (every 30 sec) to check if the connection is still alive or dead
- Integrates with connectivity status (`NetInfo`) — as soon as internet comes back, it triggers a WebSocket reconnect
- For simpler use cases, uses **Socket.IO client** or managed services (Pusher, Ably, Supabase Realtime) instead of writing raw WebSocket code — since they already handle reconnection, rooms, and fallback
- Closes/reopens the connection when the app goes to background/foreground (to save battery/data), using the `AppState` API

---

## Quick Reference

| Topic | Use When | Production Tool |
|---|---|---|
| Connectivity Status | Detecting internet on/off | NetInfo + global store |
| Fetch | One-time data fetch/submit | API wrapper + React Query |
| WebSocket | Real-time, live data | Custom hook + auto-reconnect + Socket.IO/Ably |