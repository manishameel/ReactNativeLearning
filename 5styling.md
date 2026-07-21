# Styling in React Native

React Native doesn't use CSS files. Instead, you write styles as JavaScript objects, and layout works with Flexbox by default.

---

## 1. StyleSheet

**What it is:** A way to define your styles in one place (like CSS-in-JS), instead of writing inline style objects everywhere.

**Where to use:** Almost every component — it's the standard way to style in RN.

```jsx
import { View, Text, StyleSheet } from 'react-native';

function Card() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello World</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    backgroundColor: '#f2f2f2',
    borderRadius: 8,
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
  },
});
```

**Why use `StyleSheet.create` instead of a plain object?**
- Slightly better performance (styles get an ID, sent to native side once)
- Catches typos/errors in style names
- Keeps code clean — style definitions separate from JSX

You can also combine multiple styles using an array:

```jsx
<Text style={[styles.title, { color: 'red' }]}>Combined Style</Text>
```

---

## 2. Layouts & Flexbox

**What it is:** React Native uses Flexbox (same idea as CSS Flexbox) to arrange components on screen. By default, everything is `flexDirection: 'column'` (stacked top to bottom) — unlike web, where default is `row`.

**Where to use:** Every screen layout — positioning items, centering things, spacing rows/columns.

### Key properties

```jsx
<View
  style={{
    flex: 1,                      // take up all available space
    flexDirection: 'row',         // 'column' (default) or 'row'
    justifyContent: 'center',     // aligns along main axis (space-between, center, flex-start...)
    alignItems: 'center',         // aligns along cross axis
    flexWrap: 'wrap',             // wrap items to next line if needed
  }}
>
  <View style={{ width: 50, height: 50, backgroundColor: 'red' }} />
  <View style={{ width: 50, height: 50, backgroundColor: 'blue' }} />
</View>
```

### Simple example: 3 boxes in a row, evenly spaced

```jsx
import { View } from 'react-native';

function Row() {
  return (
    <View style={{ flexDirection: 'row', justifyContent: 'space-between' }}>
      <View style={{ width: 60, height: 60, backgroundColor: 'tomato' }} />
      <View style={{ width: 60, height: 60, backgroundColor: 'gold' }} />
      <View style={{ width: 60, height: 60, backgroundColor: 'skyblue' }} />
    </View>
  );
}
```

### Cheat sheet

| Property | Meaning |
|---|---|
| `flex: 1` | Fill remaining space |
| `flexDirection` | `column` (default) or `row` |
| `justifyContent` | Spacing along main axis (`center`, `flex-start`, `flex-end`, `space-between`, `space-around`) |
| `alignItems` | Alignment along cross axis (`center`, `flex-start`, `flex-end`, `stretch`) |
| `flexWrap` | Whether items wrap to next line |

---

## 3. Accessibility

**What it is:** Making your app usable for people with disabilities — e.g. screen reader users (VoiceOver on iOS, TalkBack on Android).

**Where to use:** Buttons, images, inputs — anything interactive or meaningful that a screen reader user needs to understand.

### Key props

```jsx
import { Pressable, Text, Image } from 'react-native';

<Pressable
  onPress={() => alert('Liked!')}
  accessible={true}
  accessibilityLabel="Like this post"
  accessibilityRole="button"
  accessibilityHint="Adds this post to your liked list"
>
  <Text>❤️ Like</Text>
</Pressable>

<Image
  source={{ uri: 'https://example.com/dog.jpg' }}
  accessibilityLabel="A brown dog playing in a park"
/>
```

### What each prop does

| Prop | Meaning |
|---|---|
| `accessible` | Marks this component (and its children) as a single accessible unit |
| `accessibilityLabel` | What the screen reader reads out loud |
| `accessibilityRole` | Tells the screen reader what kind of element it is (`button`, `link`, `image`, `header`, etc.) |
| `accessibilityHint` | Extra spoken info about what happens on interaction |

> Simple rule: if a sighted user understands something just by looking at it (an icon, an image, a colored dot), a screen reader user needs `accessibilityLabel` to understand the same thing by hearing it.

---

## Quick Reference

| Topic | Use When |
|---|---|
| StyleSheet | Styling any component — the standard way to write styles |
| Flexbox | Arranging/positioning components on screen |
| Accessibility | Making buttons, images, inputs usable with screen readers |