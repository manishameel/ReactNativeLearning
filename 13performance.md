# Performance in React Native

⚠️ **2026 context:** The New Architecture (JSI, Fabric, TurboModules) is no longer optional — the old Bridge was fully removed in RN 0.82, and Hermes V1 became the default engine in RN 0.84. This changes some old performance advice: bridge-serialization tricks matter less now (JSI calls are direct, not JSON-serialized), and **RAM bundles are largely legacy** — Hermes' precompiled bytecode + New Architecture already solve most of what RAM bundles were designed for.

---

## 1. Understanding Frame Rates

**What it is:** Your app needs to redraw the screen 60 times per second (60fps) — or 120fps on high refresh-rate devices — to feel smooth. Each frame has a budget of about **16ms** (at 60fps). If any single frame takes longer, the user sees a stutter/jank.

**Two threads matter:**

| Thread | Job | If it's blocked... |
|---|---|---|
| **JS Thread** | Runs your React/business logic | Touches feel delayed, state updates lag |
| **UI Thread** (aka Main/Native thread) | Actually draws pixels on screen | Scrolling/animations stutter |

With the New Architecture, gesture handling and animations (via Reanimated worklets) can run directly on the UI thread, bypassing the JS thread entirely — this is why modern gesture/animation code feels so much smoother than older `Animated` API code.

**Where to check this:** Use the in-app **Perf Monitor** (Dev Menu → Show Perf Monitor) to watch JS and UI frame rates live while using your app.

---

## 2. Common Problem Sources

| Problem | Cause |
|---|---|
| **Unnecessary re-renders** | Passing new inline functions/objects as props every render, missing memoization |
| **Heavy JS thread work** | Large loops, JSON parsing, unoptimized state updates all blocking the JS thread |
| **Unoptimized lists** | Using `ScrollView` or a poorly configured `FlatList` for large data |
| **Large images** | Loading full-resolution images for small thumbnails |
| **Excessive bridge/JSI traffic** | Very frequent calls between JS and native (e.g. updating native props on every scroll pixel without throttling) |
| **Bloated JS bundle** | Slow app startup because too much JS has to be parsed/executed before first render |
| **Console logs left in production** | `console.log` has real overhead, especially in loops |

**Basic re-render fix example:**
```jsx
import { memo, useCallback } from 'react';

const ListItem = memo(function ListItem({ title, onPress }) {
  return <Pressable onPress={onPress}><Text>{title}</Text></Pressable>;
});

function List({ items }) {
  const handlePress = useCallback((id) => {
    console.log('pressed', id);
  }, []);

  return items.map(item => (
    <ListItem key={item.id} title={item.title} onPress={() => handlePress(item.id)} />
  ));
}
```

### 🧑‍💻 Professional usage:
- Treats `memo`/`useCallback`/`useMemo` as targeted tools, not something to sprinkle everywhere — over-memoizing adds its own overhead and complexity; pros apply it where profiling actually shows a re-render problem
- Uses the React DevTools Profiler (via React Native DevTools) to see exactly which components re-render and why, instead of guessing

---

## 3. Speeding Up Builds

**What it is:** Reducing the time it takes to build and install your app during development (not runtime performance — this is developer experience).

### Practical steps:
```bash
# Use Gradle build caching (Android) - enabled in gradle.properties
org.gradle.caching=true
org.gradle.parallel=true

# Use Metro's cache instead of clearing it every time
npx react-native start --reset-cache   # only when truly needed, it's slow
```

- **Avoid unnecessary full rebuilds**: JS-only changes should hot-reload via Fast Refresh — you only need a native rebuild when native code/dependencies change
- **Split debug vs release builds**: never build a full release/optimized build during everyday development
- **Use a real device over an emulator** when possible for iOS — sometimes faster and closer to real-world performance

### 🧑‍💻 Professional usage:
- Uses **EAS Build** (Expo) or a self-hosted CI cache for native builds, so teammates aren't all independently paying full native build costs locally
- Keeps native dependencies lean — every native module added increases build time and app size; audits `package.json` periodically for unused ones
- Uses **build variants/flavors** properly (e.g. a "dev client" build) so most day-to-day work only needs a JS reload, with native rebuilds reserved for when native code actually changes

---

## 4. Optimizing FlatList Config

**What it is:** FlatList only renders what's near the viewport, but its default settings aren't tuned for every use case — several props exist specifically to control this.

```jsx
<FlatList
  data={data}
  renderItem={renderItem}
  keyExtractor={(item) => item.id}

  // Performance tuning props:
  initialNumToRender={8}        // how many items render on first paint
  maxToRenderPerBatch={8}       // items rendered per batch as user scrolls
  windowSize={5}                // how many "screens" worth of content to keep rendered
  removeClippedSubviews={true}  // unmounts off-screen views (Android benefit mainly)
  getItemLayout={(data, index) => (
    { length: ITEM_HEIGHT, offset: ITEM_HEIGHT * index, index }
  )}  // skips dynamic measurement if item height is fixed/known
/>
```

**Why `getItemLayout` matters:** without it, FlatList has to measure each item as it renders, which is expensive. If your items are a fixed height, always provide this.

### 🧑‍💻 Professional usage:
- For any list beyond roughly a screen's worth of content, **defaults to FlashList (v2)** over FlatList — it uses cell recycling (like native `RecyclerView`/`UITableView`) instead of FlatList's mount/unmount virtualization, avoiding blank-cell flashing and dropped frames on large/complex lists
- Keeps `renderItem` components memoized (`memo`) and avoids creating new inline functions/objects inside `renderItem` on every render
- Uses stable, unique `keyExtractor` values (never array index for lists that reorder/filter) — unstable keys cause React to unnecessarily unmount/remount items
- Lazy-loads images inside list items and uses a proper caching image component (`expo-image` or `react-native-fast-image`) rather than the default `Image`

---

## 5. RAM Bundles + Inline Requires

**What they were:** Older optimizations from the pre-Hermes/pre-New-Architecture era.

- **RAM Bundle**: instead of loading the entire JS bundle into memory upfront, it split the bundle so modules load lazily as needed — helped startup time on the old JSC engine
- **Inline Requires**: deferring `require()` calls until a module is actually used (rather than all imports resolving at app start), reducing initial load work

```js
// Inline require example (deferring cost until actually needed)
function openSettings() {
  const Settings = require('./Settings').default; // loaded only when this runs
  navigation.navigate(Settings);
}
```

⚠️ **Current relevance:** With Hermes as the default engine, JS is precompiled to bytecode ahead of time and loaded far more efficiently than the old JSC setup RAM bundles were built to fix. Most modern RN projects **no longer need RAM bundles**. Inline requires can still occasionally help defer loading of genuinely heavy, rarely-used modules (e.g. a big charting library only used on one screen), but it's a minor, situational optimization now rather than a default best practice.

### 🧑‍💻 Professional usage:
- Doesn't reach for RAM bundles on a modern (0.84+) Hermes/New Architecture project — it's solving a problem that's already been solved at the engine level
- Still uses **code-splitting / lazy imports** at the React level (`React.lazy` equivalent patterns, or conditionally requiring rarely-used heavy libraries) for genuinely large, rarely-touched features
- Focuses bundle-size effort on **Metro tree-shaking**, removing unused dependencies, and checking bundle composition with `npx react-native-bundle-visualizer` instead of legacy bundle-splitting tricks

---

## 6. Profiling

**What it is:** Measuring where time is actually being spent, instead of guessing what's slow.

### Tools:

**Perf Monitor** (built-in, quick check)
Dev Menu → Show Perf Monitor — shows live JS/UI FPS.

**React Native DevTools** (default debugger since 0.76)
Press `j` in the Metro terminal. Includes a **Profiler** panel — records a session and shows which components rendered, how often, and why (which prop/state changed).

**Hermes Sampling Profiler** (bundled with Hermes)
Captures a low-overhead CPU trace of actual JS execution — useful for finding exactly which function is eating time.
```bash
# Trigger via Dev Menu -> "Start/Stop Sampling Profiler"
# Then open the generated trace in Chrome DevTools' Performance tab
```

**Native tools:**
- **Xcode Instruments** (iOS) — Time Profiler, Allocations (memory leaks)
- **Android Studio Profiler** — CPU, memory, network on Android

### A simple profiling workflow:
1. Reproduce the slow interaction with Perf Monitor open — is it JS thread or UI thread that's dropping frames?
2. If JS thread: open React Native DevTools Profiler, record the interaction, find the component doing unnecessary work
3. If still unclear: run the Hermes Sampling Profiler for a precise CPU trace
4. If it's native-side (image decode, native module call): drop into Xcode Instruments / Android Studio Profiler

### 🧑‍💻 Professional usage:
- **Never optimizes blind** — always profiles first to confirm where the actual bottleneck is (JS thread vs UI thread vs native) before writing any "optimization" code, since intuition about performance is often wrong
- Profiles on a **real, mid-tier device**, not just a powerful development machine or simulator — simulators/high-end phones hide real-world performance problems
- Tracks key metrics over time in CI (cold start time, bundle size, list scroll FPS) so regressions get caught automatically instead of discovered by users
- Treats performance work as targeted: fixes the single worst offending component/screen first (usually found via profiling, not code review), rather than broadly "optimizing everything"

---

## Quick Reference

| Topic | Key Tool/Setting | Priority |
|---|---|---|
| Frame rates | Perf Monitor, understand JS vs UI thread | Foundational |
| Common problems | memo/useCallback, avoid heavy JS thread work | High |
| Build speed | Gradle caching, Fast Refresh, dev client builds | Dev experience |
| FlatList | `getItemLayout`, tuning props, or switch to FlashList | High for lists |
| RAM bundles / inline requires | Mostly legacy now (Hermes handles this) | Low, situational |
| Profiling | React Native DevTools Profiler, Hermes Sampling Profiler | Do this first, always |