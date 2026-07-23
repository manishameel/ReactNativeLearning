# Testing in React Native

Testing works best as a **pyramid**: lots of fast unit tests at the bottom, some component/integration tests in the middle, and a few slow end-to-end tests at the top.

```
        /\
       /E2E\        ← few, slow, expensive (Detox/Appium/Maestro)
      /------\
     /Component\    ← some, medium speed (RNTL)
    /------------\
   /  Unit (Jest)  \  ← many, fast, cheap
  /------------------\
```

⚠️ **2026 update:** For E2E, **Maestro** has become a strong, faster alternative to Detox for many teams (YAML-based, no native build step needed, much lower CI time). Detox is still excellent and has the tightest React Native integration, but the ecosystem is shifting — many new projects start with Maestro for core flows and add Detox only where its deeper sync capabilities matter.

---

## 1. Jest (Unit Testing)

**What it is:** The default test runner for React Native — tests individual functions, hooks, and logic in isolation, without a real device/simulator.

**Where to use:** Utility functions, custom hooks, business logic, reducers — anything that doesn't need to render actual UI.

Comes pre-configured with React Native's default template.

```jsx
// utils/math.js
export function add(a, b) {
  return a + b;
}
```

```jsx
// utils/math.test.js
import { add } from './math';

test('adds two numbers correctly', () => {
  expect(add(2, 3)).toBe(5);
});
```

**Testing a custom hook:**
```jsx
import { renderHook, act } from '@testing-library/react-native';
import { useCounter } from './useCounter';

test('increments counter', () => {
  const { result } = renderHook(() => useCounter());

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

Run with:
```bash
npx jest
```

### 🧑‍💻 Professional usage:
- Aims for **~70% of the test suite to be unit tests** — they're the fastest and cheapest to run, so most logic gets covered here before it ever touches a UI test
- Mocks external dependencies (API calls, native modules, AsyncStorage) so unit tests are fast, deterministic, and don't depend on network/device
- Runs Jest in CI on **every pull request**, often with a minimum coverage threshold enforced (e.g. `coverageThreshold` in `jest.config.js`)

---

## 2. Component Testing

### React Native Testing Library (RNTL)

**What it is:** Renders your components and lets you interact with them (tap, type) and assert on what the user would actually see — testing behavior, not internal implementation.

**Where to use:** Testing that a component renders correctly, responds to user interaction, and shows the right state — without needing a real device.

```bash
npm install --save-dev @testing-library/react-native
```

```jsx
import { render, screen, fireEvent } from '@testing-library/react-native';
import LoginForm from './LoginForm';

test('shows error when submitting empty email', () => {
  render(<LoginForm />);

  fireEvent.press(screen.getByText('Submit'));

  expect(screen.getByText('Email is required')).toBeTruthy();
});

test('calls onLogin with entered credentials', () => {
  const onLogin = jest.fn();
  render(<LoginForm onLogin={onLogin} />);

  fireEvent.changeText(screen.getByPlaceholderText('Email'), 'test@example.com');
  fireEvent.changeText(screen.getByPlaceholderText('Password'), 'secret123');
  fireEvent.press(screen.getByText('Submit'));

  expect(onLogin).toHaveBeenCalledWith('test@example.com', 'secret123');
});
```

### react-test-renderer

**What it is:** A lower-level tool that renders components to a plain JS object tree (not real interactions) — mostly used for **snapshot testing**.

```jsx
import renderer from 'react-test-renderer';
import ProfileCard from './ProfileCard';

test('matches snapshot', () => {
  const tree = renderer.create(<ProfileCard name="Manisha" />).toJSON();
  expect(tree).toMatchSnapshot();
});
```

⚠️ Considered more "legacy" now — RNTL is preferred for most new tests because it tests things the way a user actually experiences them (by text, role, label) rather than internal component structure.

### 🧑‍💻 Professional usage:
- Queries elements the way a **user would find them** — by visible text, placeholder, or `accessibilityLabel` — instead of by internal implementation details, so tests don't break on harmless refactors
- Uses snapshot tests sparingly, mainly for components with a genuinely stable, simple output (like a design-system Button) — overusing snapshots on complex screens leads to people blindly approving snapshot diffs without reading them, which defeats the purpose
- Adds `testID` props to key interactive elements specifically to make both RNTL and E2E tests (Detox) reliable

---

## 3. E2E Testing (Detox, Appium, Maestro)

**What it is:** Testing the **real, fully assembled app** — running on an actual simulator/emulator/device, simulating a real user tapping through actual screens.

**Where to use:** Critical user flows only (login, checkout, core navigation) — not everything, since E2E tests are slow and comparatively expensive to maintain.

### Detox

Built specifically for React Native — synchronizes with the JS thread so it automatically waits for animations/network calls, reducing flakiness.

```bash
npm install --save-dev detox
```

```js
// e2e/login.test.js
describe('Login flow', () => {
  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should log in successfully', async () => {
    await element(by.id('emailInput')).typeText('test@example.com');
    await element(by.id('passwordInput')).typeText('secret123');
    await element(by.id('loginButton')).tap();

    await expect(element(by.text('Welcome back!'))).toBeVisible();
  });
});
```

### Maestro

YAML-based — no code, very fast to write, minimal setup (no native build step required).

```yaml
# login.yaml
appId: com.myapp
---
- launchApp
- tapOn: "Email"
- inputText: "test@example.com"
- tapOn: "Password"
- inputText: "secret123"
- tapOn: "Login"
- assertVisible: "Welcome back!"
```

Run with:
```bash
maestro test login.yaml
```

### Appium

Cross-platform, WebDriver-based — works beyond just React Native. More setup, more flexible, but generally slower and more maintenance-heavy than Detox/Maestro for RN-only apps.

```js
const loginButton = await driver.$('~loginButton'); // accessibility ID
await loginButton.click();
```

### 🧑‍💻 Professional usage:
- Keeps E2E tests to a **small, critical set (~10% of the suite)** — login, checkout, core happy paths — rather than trying to E2E-test every screen, since these tests are slow and prone to flakiness at scale
- Ensures every interactive component has a stable `testID`/`accessibilityLabel` before writing E2E tests — Detox especially depends on this for reliable element targeting
- Runs E2E tests on a **dedicated CI pipeline** separate from unit tests (needs a macOS runner for iOS, simulator/emulator boot time, etc.) — often nightly or pre-release, not on every single commit, to keep PR feedback fast
- Chooses the tool based on team needs: **Maestro** for fast setup and non-developer-readable tests, **Detox** when deep RN-specific sync/reliability matters and the team is comfortable in JS, **Appium** if the product spans beyond RN (e.g. also has a separate native iOS/Android codebase) or needs maximum device-cloud compatibility
- Adds retry logic and treats occasional E2E flakiness as expected — doesn't block releases on a single flaky E2E failure, but does track flake rate over time and investigates trends

---

## Quick Reference

| Layer | Tool | % of Suite | Speed |
|---|---|---|---|
| Unit | Jest | ~70% | Fast (seconds) |
| Component | React Native Testing Library | ~20% | Medium |
| E2E | Detox / Maestro / Appium | ~10% | Slow (minutes) |