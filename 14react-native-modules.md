# Using Native Modules in React Native

**What it is:** Sometimes JavaScript alone can't do something — accessing a native SDK, doing heavy native-only computation, using a platform API with no RN equivalent. A **native module** is a bridge that lets your JS code call real native (Kotlin/Swift) code.

⚠️ **2026 context:** Since the old Bridge was removed (RN 0.82+), all native modules are now written as **TurboModules** using the New Architecture — JSI-based, type-safe, and generated via **Codegen**. The old `NativeModules`/`@ReactMethod`-only bridge style is deprecated. Everything below uses the current TurboModule approach.

---

## How It Works (big picture)

```
1. Define a TypeScript "spec" (the contract)
        ↓
2. Codegen reads the spec, auto-generates native interface code
        ↓
3. You implement that interface in Kotlin (Android) and Swift (iOS)
        ↓
4. JS calls the module directly via JSI — no bridge serialization, no JSON
```

The spec file is the single source of truth — both platforms implement the same interface.

---

## Step 1: Define the Spec (shared, TypeScript)

```ts
// specs/NativeDeviceInfo.ts
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  getBatteryLevel(): Promise<number>;
  getDeviceModel(): string;
}

export default TurboModuleRegistry.getEnforcing<Spec>('DeviceInfo');
```

Codegen reads this and generates the matching native interfaces for both platforms automatically when you build.

---

## For Android (Kotlin)

```kotlin
// android/app/src/main/java/com/myapp/DeviceInfoModule.kt
package com.myapp

import com.facebook.react.bridge.Promise
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.turbomodule.core.interfaces.TurboModule

class DeviceInfoModule(reactContext: ReactApplicationContext) :
    NativeDeviceInfoSpec(reactContext) {

    override fun getName() = "DeviceInfo"

    override fun getBatteryLevel(promise: Promise) {
        try {
            val batteryManager = reactApplicationContext
                .getSystemService(android.content.Context.BATTERY_SERVICE) as android.os.BatteryManager
            val level = batteryManager.getIntProperty(
                android.os.BatteryManager.BATTERY_PROPERTY_CAPACITY
            )
            promise.resolve(level)
        } catch (e: Exception) {
            promise.reject("BATTERY_ERROR", e)
        }
    }

    override fun getDeviceModel(): String {
        return android.os.Build.MODEL
    }
}
```

**Registering the module (Package file):**
```kotlin
// android/app/src/main/java/com/myapp/DeviceInfoPackage.kt
class DeviceInfoPackage : TurboReactPackage() {
    override fun getModule(name: String, reactContext: ReactApplicationContext): NativeModule? {
        return if (name == "DeviceInfo") DeviceInfoModule(reactContext) else null
    }

    override fun getReactModuleInfoProvider() = ReactModuleInfoProvider {
        mapOf("DeviceInfo" to ReactModuleInfo(
            "DeviceInfo", "DeviceInfo", false, false, false, true
        ))
    }
}
```

---

## For iOS (Swift)

```swift
// ios/DeviceInfoModule.swift
import Foundation

@objc(DeviceInfoModule)
class DeviceInfoModule: NSObject, NativeDeviceInfoSpec {

  func getBatteryLevel(_ resolve: @escaping RCTPromiseResolveBlock,
                        reject: @escaping RCTPromiseRejectBlock) {
    UIDevice.current.isBatteryMonitoringEnabled = true
    let level = UIDevice.current.batteryLevel

    if level < 0 {
      reject("BATTERY_ERROR", "Could not read battery level", nil)
    } else {
      resolve(Int(level * 100))
    }
  }

  func getDeviceModel() -> String {
    return UIDevice.current.model
  }
}
```

**Objective-C bridge header (still needed to expose Swift to RN's obj-c runtime):**
```objc
// ios/DeviceInfoModule.m
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(DeviceInfoModule, NSObject)
RCT_EXTERN_METHOD(getBatteryLevel: (RCTPromiseResolveBlock)resolve
                  reject: (RCTPromiseRejectBlock)reject)
RCT_EXTERN_METHOD(getDeviceModel)
@end
```

---

## Using It From JS

```jsx
import DeviceInfo from '../specs/NativeDeviceInfo';

async function checkBattery() {
  const level = await DeviceInfo.getBatteryLevel();
  const model = DeviceInfo.getDeviceModel();
  console.log(`${model} battery: ${level}%`);
}
```

Called directly through JSI — synchronous methods return instantly, no bridge round-trip.

---

## 🧑‍💻 How a Professional Developer Uses This

- **Checks for an existing library first.** Writing a native module is a last resort — pros search npm/React Native Directory for an already-maintained TurboModule (e.g. `react-native-device-info` already does the battery example above) before writing native code, since native code means maintaining it across OS updates forever.
- **Keeps the spec as the contract, not an afterthought** — designs the TypeScript interface carefully (naming, types, sync vs async) before touching native code, since Codegen enforces that both platforms match it exactly.
- **Uses synchronous methods sparingly.** JSI allows synchronous native calls now (unlike the old async-only bridge), but a slow synchronous native call blocks the JS thread directly — pros only make something sync if it's genuinely fast (e.g. reading a cached value), and keep anything with real work (network, disk, computation) as `Promise`-based.
- **Handles errors properly on both sides** — rejects promises with meaningful error codes/messages (like `"BATTERY_ERROR"` above) so JS can handle different failure cases distinctly, instead of a single generic catch-all error.
- **Tests native modules on real devices**, not just simulators — some native APIs (camera, sensors, certain permissions) don't behave the same or don't exist on simulators/emulators.
- **Documents platform differences clearly** — if a feature behaves differently on Android vs iOS (common with permissions, background tasks, notifications), the interface often exposes it uniformly to JS while the pro adds inline comments in native code explaining platform-specific quirks.
- **Runs Codegen as part of the build** (`npx react-native codegen` or automatic via build scripts) and treats generated code as read-only — never hand-edits generated interface files, since they get overwritten.
- **For teams without native expertise**, considers using **Expo Modules API** instead of raw TurboModules — it's a higher-level, more ergonomic wrapper over the same New Architecture primitives, with less boilerplate for common native module patterns.

---

## Quick Reference

| Step | Tool |
|---|---|
| Define contract | TypeScript spec file (`Spec extends TurboModule`) |
| Auto-generate interfaces | Codegen |
| Implement (Android) | Kotlin, extends generated `Spec` class |
| Implement (iOS) | Swift + Objective-C bridge header |
| Call from JS | Import spec, call directly (JSI, no bridge) |
| Simpler alternative | Expo Modules API |