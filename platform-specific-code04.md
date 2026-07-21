# Writing Platform-Specific Code in React Native

Sometimes iOS, Android, and Web need slightly different code. React Native gives you 3 ways to handle this.

---

## 1. Platform Module

**What it is:** A simple object that tells you *which OS the app is running on*, so you can write different logic/styles inside the same file.

**Where to use:** Small differences — like a different padding value, or one line of logic that changes per OS.

```jsx
import { Platform, View, Text } from 'react-native';

function MyComponent() {
  return (
    <View style={{ padding: Platform.OS === 'ios' ? 20 : 10 }}>
      <Text>
        {Platform.OS === 'ios' ? 'Hello from iOS' : 'Hello from Android'}
      </Text>
    </View>
  );
}
```

`Platform.OS` can be `'ios'`, `'android'`, or `'web'`.

There's also a cleaner helper — `Platform.select` — for picking values based on OS:

```jsx
const styles = {
  fontSize: Platform.select({
    ios: 18,
    android: 16,
    default: 14, // used for web or anything else
  }),
};
```

---

## 2. File Extensions

**What it is:** Instead of `if/else` inside one file, you make **separate files** for each platform. React Native automatically picks the right one at build time.

**Where to use:** When the component's code is very different per platform (not just a value, but whole logic/UI).

**Naming pattern:**
```
Button.ios.js       → used only on iOS
Button.android.js   → used only on Android
Button.web.js       → used only on Web
Button.native.js    → used for iOS + Android (not web)
```

You just import it normally, without mentioning the extension:

```jsx
import Button from './Button'; 
// RN automatically picks Button.ios.js on iOS, Button.android.js on Android
```

**Example:**

`Button.ios.js`
```jsx
import { Button } from 'react-native';
export default function MyButton(props) {
  return <Button {...props} color="blue" />;
}
```

`Button.android.js`
```jsx
import { Button } from 'react-native';
export default function MyButton(props) {
  return <Button {...props} color="green" />;
}
```

---

## 3. react-native-web

**What it is:** A library that lets your React Native components run in a normal web browser too (same code, one app for mobile + web).

**Where to use:** When you want your RN app to also work as a website, without rewriting everything in plain React.

**Basic setup idea (conceptual):**

```bash
npm install react-native-web react-dom
```

`index.web.js` (entry file for web)
```jsx
import { AppRegistry } from 'react-native';
import App from './App';

AppRegistry.registerComponent('MyApp', () => App);
AppRegistry.runApplication('MyApp', {
  rootTag: document.getElementById('root'),
});
```

Your normal component code stays the same:

```jsx
import { View, Text } from 'react-native';

export default function App() {
  return (
    <View>
      <Text>This works on iOS, Android, AND Web!</Text>
    </View>
  );
}
```

> Most projects today use **Expo** with `expo start --web`, which sets up react-native-web automatically — no manual config needed.

---

## Quick Reference

| Method | Use When |
|---|---|
| `Platform.OS` / `Platform.select` | Small style/logic differences in the same file |
| `.ios.js` / `.android.js` / `.web.js` files | Big differences — separate components per platform |
| react-native-web | Want the whole app to also run in a browser |