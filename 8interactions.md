# Interactions in React Native

This covers how users *touch, drag, scroll, and move between screens* — plus how to animate all of it.

⚠️ **What's changed recently:** `react-native-gesture-handler` just hit **v3** (rebuilt for the New Architecture, hook-based API, new `Touchable` component). `react-native-reanimated` is on **v4** with worklet-based animations running fully on the UI thread. `react-navigation` is on **v7**. If you started a project in the last year, you're almost certainly already on the New Architecture — these tools assume that.

---

## 1. Touchable

**What it is:** Making something respond to a tap.

**Where to use:** Buttons, cards, list items, icons — anything tappable.

```jsx
import { Pressable, Text } from 'react-native';

<Pressable
  onPress={() => console.log('Tapped!')}
  style={({ pressed }) => ({
    padding: 12,
    backgroundColor: pressed ? '#ddd' : '#eee',
    borderRadius: 8,
  })}
>
  <Text>Tap Me</Text>
</Pressable>
```

`Pressable` is the modern standard — it replaced the older `TouchableOpacity` / `TouchableHighlight` / `TouchableWithoutFeedback` components. It also supports `onLongPress`, `onPressIn`, `onPressOut`, and `delayLongPress`.

### 🧑‍💻 Professional usage:
- Adds `hitSlop` to increase the tappable area without changing visual size (helps small icons meet accessibility touch-target guidelines: 44x44pt minimum)
- Wraps `Pressable` in a shared `AppButton` component across the app, so press animations, disabled states, and haptics are consistent everywhere instead of copy-pasted
- Adds haptic feedback (`expo-haptics` or `react-native-haptic-feedback`) on press for a more native feel

---

## 2. Gesture Handling

**What it is:** Detecting drags, pinches, swipes, and multi-touch — beyond a simple tap.

**Where to use:** Draggable cards (like Tinder swipe), pinch-to-zoom images, custom sliders, bottom sheets.

Uses **`react-native-gesture-handler`** — the official native-backed gesture library. It runs gesture detection on the native/UI thread, not the JS thread, so it stays smooth even under heavy JS load.

```bash
npx expo install react-native-gesture-handler
```

Wrap your app root once:
```jsx
import { GestureHandlerRootView } from 'react-native-gesture-handler';

export default function App() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      {/* rest of app */}
    </GestureHandlerRootView>
  );
}
```

**Example — draggable box:**
```jsx
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated, { useSharedValue, useAnimatedStyle } from 'react-native-reanimated';

function DraggableBox() {
  const translateX = useSharedValue(0);
  const translateY = useSharedValue(0);

  const pan = Gesture.Pan()
    .onChange((event) => {
      translateX.value += event.changeX;
      translateY.value += event.changeY;
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }, { translateY: translateY.value }],
  }));

  return (
    <GestureDetector gesture={pan}>
      <Animated.View style={[{ width: 80, height: 80, backgroundColor: 'tomato' }, animatedStyle]} />
    </GestureDetector>
  );
}
```

### 🧑‍💻 Professional usage:
- Pairs Gesture Handler with **Reanimated worklets** (like above) so the gesture *and* the resulting animation both run on the UI thread — zero JS bridge involvement, no dropped frames even during heavy JS work
- Uses `.activeOffsetX([-10, 10])` on a `Pan` gesture so it only "wins" after enough horizontal movement — this stops it from fighting a parent `ScrollView` for vertical scrolls
- Composes gestures with `Gesture.Simultaneous()`, `Gesture.Race()`, or `Gesture.Exclusive()` for complex interactions (e.g. pinch-to-zoom + pan at the same time)
- Builds reusable interaction patterns (swipeable list row, draggable bottom sheet) as their own components instead of repeating gesture logic in every screen

---

## 3. Scrolling & Swiping

**What it is:** Moving through content that's larger than the screen, or swiping between items/cards.

**Where to use:** Feeds, image carousels, onboarding screens, swipe-to-delete list items.

**Basic scroll:**
```jsx
import { ScrollView, View } from 'react-native';

<ScrollView horizontal showsHorizontalScrollIndicator={false}>
  <View style={{ width: 200, height: 200, backgroundColor: 'skyblue', marginRight: 8 }} />
  <View style={{ width: 200, height: 200, backgroundColor: 'salmon' }} />
</ScrollView>
```

**Swipe-to-delete row (using Gesture Handler's Swipeable):**
```jsx
import { Swipeable } from 'react-native-gesture-handler';
import { Text, View, Button } from 'react-native';

function SwipeableRow({ onDelete }) {
  return (
    <Swipeable
      renderRightActions={() => (
        <Button title="Delete" color="red" onPress={onDelete} />
      )}
    >
      <View style={{ padding: 16, backgroundColor: 'white' }}>
        <Text>Swipe me left</Text>
      </View>
    </Swipeable>
  );
}
```

### 🧑‍💻 Professional usage:
- For long lists, always uses `FlatList`/`FlashList` (virtualized) instead of `ScrollView` — mentioned in an earlier note, but critical for scroll performance
- Tracks scroll position with `onScroll` + `useAnimatedScrollHandler` (Reanimated) to drive effects like a collapsing header or parallax, without causing re-renders on every scroll frame
- Uses `react-native-reanimated-carousel` or `@gorhom/bottom-sheet` for polished carousels/bottom sheets instead of building swipe logic from scratch — these handle edge cases (momentum, snapping, orientation change) that are easy to get wrong manually

---

## 4. Screen Navigation

**What it is:** Moving between screens in the app (Home → Profile → Settings, etc.)

**Where to use:** Every multi-screen app. The standard library is **React Navigation** (v7).

```bash
npm install @react-navigation/native @react-navigation/native-stack
npx expo install react-native-screens react-native-safe-area-context
```

```jsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { Button, Text, View } from 'react-native';

const Stack = createNativeStackNavigator();

function HomeScreen({ navigation }) {
  return (
    <View>
      <Text>Home</Text>
      <Button title="Go to Profile" onPress={() => navigation.navigate('Profile', { userId: 42 })} />
    </View>
  );
}

function ProfileScreen({ route }) {
  return <Text>Profile of user {route.params.userId}</Text>;
}

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

Other common navigator types: `createBottomTabNavigator` (tab bar), `createDrawerNavigator` (side drawer).

### 🧑‍💻 Professional usage:
- Structures navigation as **nested navigators** (e.g. a Tab Navigator, where one tab is itself a Stack Navigator) instead of one flat stack — mirrors how real apps are organized
- Uses **TypeScript param lists** (`type RootStackParamList = { Profile: { userId: number } }`) so navigation calls are type-checked, catching typos in route names/params at compile time
- Sets up **deep linking** so notification taps or external URLs open the correct screen directly, instead of always opening the home screen
- Separates navigation logic from UI — a dedicated `navigationRef` lets you `navigate()` from outside React components (like from a push notification handler or an API error interceptor)
- Uses `useFocusEffect` (fires when a screen comes into focus) instead of `useEffect` when logic should re-run every time a user returns to a screen, not just on first mount

---

## 5. Animations

**What it is:** Making UI elements move, fade, scale, or transition smoothly rather than jumping instantly.

**Where to use:** Button press feedback, loading skeletons, screen transitions, list item entrance, progress bars.

Uses **`react-native-reanimated`** (v4) — animations run via "worklets" directly on the UI thread, so they stay smooth (60/120fps) even if the JS thread is busy.

```bash
npx expo install react-native-reanimated
```

**Simple fade-in animation:**
```jsx
import Animated, { useSharedValue, useAnimatedStyle, withTiming } from 'react-native-reanimated';
import { useEffect } from 'react';
import { Text } from 'react-native';

function FadeInText() {
  const opacity = useSharedValue(0);

  useEffect(() => {
    opacity.value = withTiming(1, { duration: 500 });
  }, []);

  const animatedStyle = useAnimatedStyle(() => ({
    opacity: opacity.value,
  }));

  return (
    <Animated.View style={animatedStyle}>
      <Text>Hello, I faded in!</Text>
    </Animated.View>
  );
}
```

**Spring animation on button press:**
```jsx
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';
import { Pressable } from 'react-native';

function BouncyButton() {
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  return (
    <Pressable
      onPressIn={() => (scale.value = withSpring(0.9))}
      onPressOut={() => (scale.value = withSpring(1))}
    >
      <Animated.View style={[{ padding: 20, backgroundColor: 'coral' }, animatedStyle]} />
    </Pressable>
  );
}
```

### 🧑‍💻 Professional usage:
- Prefers `withSpring` for interactive gestures (feels natural, physics-based) and `withTiming` for predictable UI transitions (feels precise, controlled)
- Uses **Reanimated 4's CSS-style animation API** (`animatedStyle` transitions) for simpler cases instead of manually wiring shared values, when the New Architecture is available
- Keeps animation logic in worklets (functions marked to run on UI thread) and avoids touching React state inside them — mixing JS-thread state updates into an animation loop reintroduces the exact jank Reanimated exists to avoid
- Uses `Layout` animations (`entering`/`exiting`/`layout` props on `Animated.View`) for automatic list item enter/exit animations, instead of manually animating opacity/height on add/remove
- For complex custom graphics/animations (charts, custom shapes, shaders), reaches for **React Native Skia** rather than fighting Reanimated + SVG for things it wasn't built for

---

## Quick Reference

| Topic | Core Library | Use When |
|---|---|---|
| Touchable | `Pressable` (built-in) | Any tap interaction |
| Gesture Handling | `react-native-gesture-handler` v3 | Drag, pinch, swipe, multi-touch |
| Scrolling & Swiping | `FlatList`/`FlashList`, `Swipeable` | Feeds, carousels, swipe actions |
| Screen Navigation | `react-navigation` v7 | Moving between screens |
| Animations | `react-native-reanimated` v4 | Smooth motion, transitions, gestures-driven UI |