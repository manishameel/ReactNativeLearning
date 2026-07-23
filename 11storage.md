# Storage in React Native

Different data needs different storage — small settings, secrets, files, structured data, and large datasets all have a "right tool." Using the wrong one is one of the most common RN mistakes.

---

## 1. AsyncStorage

**What it is:** Simple, unencrypted **key-value** storage — like `localStorage` on the web, but async.

**Where to use:** Small, non-sensitive data — theme preference, "has seen onboarding" flag, last-viewed tab, simple cached settings.

```bash
npm install @react-native-async-storage/async-storage
```

```jsx
import AsyncStorage from '@react-native-async-storage/async-storage';

// Save
await AsyncStorage.setItem('theme', 'dark');

// Read
const theme = await AsyncStorage.getItem('theme');

// Save an object (must stringify)
await AsyncStorage.setItem('settings', JSON.stringify({ notifications: true }));

// Read an object (must parse)
const settings = JSON.parse(await AsyncStorage.getItem('settings'));

// Remove
await AsyncStorage.removeItem('theme');

// Clear everything
await AsyncStorage.clear();
```

⚠️ **Not encrypted.** Never store tokens, passwords, or personal data here.

### 🧑‍💻 Professional usage:
- Wraps AsyncStorage in a small helper module (`storage.js`) with typed get/set functions, instead of calling raw stringify/parse everywhere
- Uses `multiGet`/`multiSet` when reading/writing several keys at once — much faster than multiple individual calls
- Pairs it with a state library (Zustand's `persist` middleware, Redux Persist) to auto-save/restore app state, instead of manually wiring storage calls everywhere

---

## 2. Expo Ecosystem

### expo-secure-store

**What it is:** Encrypted key-value storage — uses iOS Keychain / Android Keystore under the hood.

**Where to use:** Auth tokens, passwords, API keys, anything sensitive.

```bash
npx expo install expo-secure-store
```

```jsx
import * as SecureStore from 'expo-secure-store';

await SecureStore.setItemAsync('accessToken', token);
const token = await SecureStore.getItemAsync('accessToken');
await SecureStore.deleteItemAsync('accessToken');
```

⚠️ Values are limited to ~2KB — it's for small secrets, not large data.

---

### expo-file-system

**What it is:** Reading/writing actual **files** — images, PDFs, downloads, exported data.

**Where to use:** Saving a downloaded file, caching an image, writing/reading a document, exporting user data as a file.

```bash
npx expo install expo-file-system
```

```jsx
import * as FileSystem from 'expo-file-system';

const fileUri = FileSystem.documentDirectory + 'notes.txt';

// Write
await FileSystem.writeAsStringAsync(fileUri, 'Hello, this is saved to disk!');

// Read
const content = await FileSystem.readAsStringAsync(fileUri);

// Download a file from the internet
const download = await FileSystem.downloadAsync(
  'https://example.com/report.pdf',
  FileSystem.documentDirectory + 'report.pdf'
);

// Check if a file exists
const info = await FileSystem.getInfoAsync(fileUri);
console.log(info.exists, info.size);

// Delete
await FileSystem.deleteAsync(fileUri);
```

`FileSystem.documentDirectory` = permanent storage. `FileSystem.cacheDirectory` = temporary, OS can clear it to free space.

---

### expo-sqlite

**What it is:** A real **SQL database** on the device — structured, queryable, relational data.

**Where to use:** Large amounts of structured data — a to-do app with hundreds of tasks, an offline-first app, a notes app with search/filter, anything needing relationships between data (e.g. users → orders → items).

```bash
npx expo install expo-sqlite
```

```jsx
import * as SQLite from 'expo-sqlite';

async function setupDatabase() {
  const db = await SQLite.openDatabaseAsync('app.db');

  await db.execAsync(`
    CREATE TABLE IF NOT EXISTS tasks (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      title TEXT NOT NULL,
      done INTEGER DEFAULT 0
    );
  `);

  return db;
}

async function addTask(db, title) {
  await db.runAsync('INSERT INTO tasks (title) VALUES (?)', title);
}

async function getTasks(db) {
  return await db.getAllAsync('SELECT * FROM tasks');
}
```

---

## 3. Other Storage Options

| Tool | What it is | Use When |
|---|---|---|
| **react-native-mmkv** | Ultra-fast key-value storage (much faster than AsyncStorage) | Frequent reads/writes, performance-critical settings/cache |
| **WatermelonDB** | Reactive, high-performance database built for large datasets | Big offline-first apps (thousands of records, complex sync) |
| **Realm** | Object-based local database with sync capabilities | Apps needing local DB + cloud sync out of the box |
| **react-native-keychain** | Encrypted storage (bare RN equivalent of expo-secure-store) | Non-Expo projects needing secure token storage |

---

## Decision Guide

```
Is it sensitive (token, password)?
  → expo-secure-store (or react-native-keychain)

Is it an actual file (image, PDF, download)?
  → expo-file-system

Is it structured/relational data, or a lot of records?
  → expo-sqlite (or WatermelonDB for very large/complex apps)

Is it just a small setting or flag?
  → AsyncStorage (or MMKV if you need speed)
```

---

## 🧑‍💻 How a Professional Developer Uses This

- **Never mixes concerns** — a token never ends up in AsyncStorage "just for now," and structured relational data never gets crammed into a single giant AsyncStorage JSON blob (this gets slow and error-prone past a few hundred entries)
- **Migrates from AsyncStorage to MMKV** in performance-sensitive apps — MMKV is significantly faster since it's synchronous and memory-mapped, avoiding the async bridge overhead
- **Builds an offline-first architecture** using SQLite/WatermelonDB as the source of truth, syncing with the server in the background — rather than assuming the app always has internet
- **Cleans up expo-file-system cache directory** periodically (or on app start) to avoid unbounded disk usage from cached images/downloads
- **Versions local database schemas** — writes migration logic (e.g. `ALTER TABLE` scripts run on app update) so existing users' local SQLite data doesn't break when the schema changes
- **Encrypts sensitive fields even within SQLite** if storing things like health data or financial details, since SQLite itself isn't encrypted by default (would pair with `expo-secure-store` for the encryption key, and use SQLCipher for encrypted SQLite)
- **Tests storage on both platforms separately** — iOS Keychain and Android Keystore behave slightly differently (e.g. Keychain data can survive app reinstall unless explicitly cleared, Android Keystore data usually doesn't)

---

## Quick Reference

| Storage | Encrypted | Type | Use For |
|---|---|---|---|
| AsyncStorage | No | Key-value | Small non-sensitive settings |
| expo-secure-store | Yes | Key-value (small) | Tokens, passwords, secrets |
| expo-file-system | No (by default) | Files | Images, PDFs, downloads |
| expo-sqlite | No (by default) | Relational DB | Structured/large data, offline-first |
| MMKV | Optional | Key-value | Fast, frequent read/write |
| WatermelonDB / Realm | Optional | Database | Large-scale offline-first apps |