# React Native Core Components — Explained Simply

---

## Views

### SafeAreaView
**What:** Keeps content away from notches, camera cutouts, and home indicator.
**Where to use:** Wrap the root of every screen.

⚠️ Built-in version is deprecated — use `react-native-safe-area-context`.

```jsx
import { SafeAreaView, SafeAreaProvider } from 'react-native-safe-area-context';

export default function App() {
  return (
    <SafeAreaProvider>
      <SafeAreaView style={{ flex: 1 }} edges={['top']}>
        <Text>Hello</Text>
      </SafeAreaView>
    </SafeAreaProvider>
  );
}
```

---

### KeyboardAvoidingView
**What:** Pushes content up when the keyboard opens, so inputs don't get hidden.
**Where to use:** Screens with forms/text inputs, especially near the bottom of the screen.

```jsx
import { KeyboardAvoidingView, TextInput, Platform } from 'react-native';

<KeyboardAvoidingView
  behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
  style={{ flex: 1 }}
>
  <TextInput placeholder="Type here" style={{ borderWidth: 1, padding: 8 }} />
</KeyboardAvoidingView>
```

---

## Text & Input

### Text
**What:** Displays text. It's the *only* component that can hold raw text.
**Where to use:** Any label, heading, paragraph.

```jsx
import { Text } from 'react-native';

<Text style={{ fontSize: 18, color: 'black' }}>Welcome back!</Text>
```

---

### TextInput
**What:** A text field for user input.
**Where to use:** Forms, search bars, login screens.

```jsx
import { TextInput } from 'react-native';
import { useState } from 'react';

function Search() {
  const [text, setText] = useState('');
  return (
    <TextInput
      value={text}
      onChangeText={setText}
      placeholder="Search..."
      style={{ borderWidth: 1, padding: 8 }}
    />
  );
}
```

---

### Button
**What:** A simple built-in clickable button.
**Where to use:** Quick actions where custom styling isn't needed.

```jsx
import { Button } from 'react-native';

<Button title="Submit" onPress={() => alert('Submitted!')} />
```

> Limited styling (just `color`). For custom-looking buttons, use `Pressable` instead.

---

## Media & Feedback

### Image
**What:** Displays an image — local or from the internet.
**Where to use:** Profile pictures, icons, banners.

```jsx
import { Image } from 'react-native';

<Image
  source={{ uri: 'https://example.com/photo.jpg' }}
  style={{ width: 100, height: 100 }}
/>
```

---

### ImageBackground
**What:** Like `Image`, but lets you put other components on top of it.
**Where to use:** Hero sections, cards with a background photo + text overlay.

```jsx
import { ImageBackground, Text } from 'react-native';

<ImageBackground
  source={{ uri: 'https://example.com/bg.jpg' }}
  style={{ width: '100%', height: 200 }}
>
  <Text style={{ color: 'white' }}>Overlay Text</Text>
</ImageBackground>
```

---

### Switch
**What:** An on/off toggle.
**Where to use:** Settings screens (dark mode, notifications on/off).

```jsx
import { Switch } from 'react-native';
import { useState } from 'react';

function DarkModeToggle() {
  const [enabled, setEnabled] = useState(false);
  return <Switch value={enabled} onValueChange={setEnabled} />;
}
```

---

### ActivityIndicator
**What:** A spinning loader.
**Where to use:** While data is loading (API calls, page transitions).

```jsx
import { ActivityIndicator } from 'react-native';

<ActivityIndicator size="large" color="blue" />
```

---

### StatusBar
**What:** Controls the phone's top status bar (time, battery icons area).
**Where to use:** Once per screen, to match your app's theme (light/dark).

```jsx
import { StatusBar } from 'react-native';

<StatusBar barStyle="dark-content" backgroundColor="white" />
```

---

## Overlays & Touch

### Modal
**What:** Shows content in a popup layer above the current screen.
**Where to use:** Confirmation dialogs, pop-up forms, image previews.

```jsx
import { Modal, View, Text, Pressable } from 'react-native';
import { useState } from 'react';

function InfoModal() {
  const [visible, setVisible] = useState(false);
  return (
    <>
      <Pressable onPress={() => setVisible(true)}>
        <Text>Open Modal</Text>
      </Pressable>
      <Modal visible={visible} animationType="slide" transparent>
        <View style={{ marginTop: 100, backgroundColor: 'white', padding: 20 }}>
          <Text>This is a modal!</Text>
          <Pressable onPress={() => setVisible(false)}>
            <Text>Close</Text>
          </Pressable>
        </View>
      </Modal>
    </>
  );
}
```

---

### Pressable
**What:** A flexible touchable wrapper — modern replacement for `TouchableOpacity`/`TouchableHighlight`.
**Where to use:** Custom buttons, cards, list items — anything tappable that needs custom styling.

```jsx
import { Pressable, Text } from 'react-native';

<Pressable
  onPress={() => alert('Pressed!')}
  style={({ pressed }) => ({
    padding: 12,
    backgroundColor: pressed ? '#ccc' : '#eee',
  })}
>
  <Text>Tap Me</Text>
</Pressable>
```

---

## Lists / Scrolling

### ScrollView
**What:** Renders ALL its children at once, and lets you scroll through them.
**Where to use:** Small, fixed amount of content (a settings page, a form).

```jsx
import { ScrollView, Text } from 'react-native';

<ScrollView>
  <Text>Item 1</Text>
  <Text>Item 2</Text>
  <Text>Item 3</Text>
</ScrollView>
```

⚠️ Don't use for long lists (100s+ items) — it renders everything upfront, which is slow.

---

### FlatList
**What:** A performance-friendly list — only renders items visible on screen.
**Where to use:** Any medium/long list: chat messages, product lists, feeds.

```jsx
import { FlatList, Text } from 'react-native';

const data = [{ id: '1', name: 'Apple' }, { id: '2', name: 'Banana' }];

<FlatList
  data={data}
  keyExtractor={(item) => item.id}
  renderItem={({ item }) => <Text>{item.name}</Text>}
/>
```

---

### SectionList
**What:** Like `FlatList`, but groups items under section headers.
**Where to use:** Contacts list (grouped by A-Z), grouped settings, categorized menus.

```jsx
import { SectionList, Text } from 'react-native';

const sections = [
  { title: 'Fruits', data: ['Apple', 'Banana'] },
  { title: 'Veggies', data: ['Carrot', 'Potato'] },
];

<SectionList
  sections={sections}
  keyExtractor={(item) => item}
  renderItem={({ item }) => <Text>{item}</Text>}
  renderSectionHeader={({ section }) => (
    <Text style={{ fontWeight: 'bold' }}>{section.title}</Text>
  )}
/>
```

---

### RefreshControl
**What:** Adds "pull down to refresh" to a scrollable list.
**Where to use:** Any list that fetches from a server (feeds, inboxes).

```jsx
import { FlatList, RefreshControl } from 'react-native';
import { useState, useCallback } from 'react';

function List() {
  const [refreshing, setRefreshing] = useState(false);

  const onRefresh = useCallback(() => {
    setRefreshing(true);
    setTimeout(() => setRefreshing(false), 1000); // fetch new data here
  }, []);

  return (
    <FlatList
      data={[{ id: '1' }, { id: '2' }]}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <Text>Item {item.id}</Text>}
      refreshControl={<RefreshControl refreshing={refreshing} onRefresh={onRefresh} />}
    />
  );
}
```

---

## Quick Reference

| Component | Use When |
|---|---|
| SafeAreaView | Avoiding notches/status bar overlap |
| KeyboardAvoidingView | Keyboard covers your inputs |
| Text | Displaying any text |
| TextInput | User needs to type something |
| Button | Simple, default-styled action |
| Image | Showing a picture |
| ImageBackground | Text/content over an image |
| Switch | On/off setting |
| ActivityIndicator | Something is loading |
| StatusBar | Styling the top system bar |
| Modal | Popup dialog/overlay |
| Pressable | Custom-styled tappable element |
| ScrollView | Small amount of scrollable content |
| FlatList | Long list of similar items |
| SectionList | Long list grouped into sections |
| RefreshControl | Pull-to-refresh on a list |