# Publishing React Native Apps

Publishing has two separate jobs: **Build** (turn your code into a signed native binary) and **Submit** (upload that binary + store listing for review). Both platforms require accounts, signing, and review.

⚠️ **2026 requirements to know:**
- **Android**: new apps/updates must target **Android 16 (API level 36)** as of August 31, 2026
- **iOS**: apps must be built with the **iOS/iPadOS 26 SDK** or later as of April 2026
- Both stores now enforce stricter **privacy manifest / data safety** disclosures than a couple years ago

Most modern RN/Expo teams use **EAS Build + EAS Submit** to handle both platforms from one command line, without needing a Mac for iOS builds.

---

## Apple App Store

### Requirements
- Apple Developer Program account — **$99/year**
- A Mac with Xcode (or use EAS Build to skip this)
- App built with the current iOS SDK (iOS 26+ SDK as of April 2026)
- App must be fully functional — no beta/placeholder content
- Should support iPad reasonably well if it's an iPhone app (universal support expectation)

### Steps

**1. Configure app identity** (`app.json` for Expo):
```json
{
  "expo": {
    "ios": {
      "bundleIdentifier": "com.mycompany.myapp",
      "buildNumber": "1"
    }
  }
}
```

**2. Build:**
```bash
eas build --platform ios
```
This produces a signed `.ipa` file. EAS handles certificates/provisioning profiles automatically (or you manage them yourself in Xcode for a bare workflow).

**3. Submit to TestFlight first (internal testing):**
```bash
eas submit --platform ios
```

**4. Fill in App Store Connect listing:**
- Screenshots for required device sizes
- App description, keywords, category
- Privacy details (what data you collect — App Store's "Privacy Nutrition Label")
- Support URL, marketing URL (optional)

**5. Submit for review.** Apple's review typically takes **24–48 hours** in 2026, sometimes faster.

### Common rejection reasons
- Missing/vague privacy usage descriptions (`NSCameraUsageDescription`, etc.)
- App crashes or has broken flows during review
- Login-required apps without a demo account provided to reviewers
- Requesting permissions with no clear, working feature behind them

---

## Google Play Store

### Requirements
- Google Play Console account — **one-time $25 fee**
- App target SDK = **Android 16 (API 36)** as of Aug 31, 2026 deadline
- App must be uploaded as an **AAB (Android App Bundle)**, not APK — Google generates optimized per-device installs from it
- Privacy Manifest / Data Safety form completed
- Signed with a valid keystore

### Steps

**1. Configure app identity** (`app.json` for Expo):
```json
{
  "expo": {
    "android": {
      "package": "com.mycompany.myapp",
      "versionCode": 1
    }
  }
}
```

**2. Build:**
```bash
eas build --platform android
# or, bare workflow:
./gradlew bundleRelease
```
This produces an `.aab` file.

**3. First submission is manual.** Google Play's API can't submit to an app that doesn't exist yet — the very first version has to be uploaded manually through Play Console (Production → Create new release). After that, subsequent releases can go through:
```bash
eas submit --platform android
```

**4. Fill in Play Console listing:**
- Screenshots, feature graphic, icon
- Short + full description
- Content rating questionnaire
- Data Safety section (what data collected, shared, why)

**5. Submit for review.** Google's review is generally quicker than Apple's — often **hours rather than days**.

### Common rejection reasons
- Incomplete Data Safety form, or form doesn't match what the app actually does
- Missing content rating
- Broken core functionality (must be genuinely testable, not a stub)
- Permissions declared but unused/unexplained

---

## 🧑‍💻 How a Professional Developer Handles This

- **Uses EAS Build + EAS Submit (or an equivalent CI pipeline)** rather than manually building locally every release — this makes releases reproducible, and lets any team member trigger a release without needing a personally-configured Mac/Android SDK setup.
- **Tests on internal tracks before public release**: TestFlight for iOS, Play Console's Internal/Closed testing track for Android — catches crashes and UI bugs (a label too long on a narrow screen, an icon that doesn't read clearly) before real users see them.
- **Provides reviewer demo accounts** for any app with login, since Apple/Google reviewers can't sign up with real credentials and a broken login flow is a common instant rejection.
- **Keeps a release checklist** before every submission: version/build numbers bumped, target SDK current, privacy manifest accurate, signing verified, screenshots up to date, tested on real devices (not just simulators).
- **Separates "code freeze" from "submit"** — doesn't submit the moment a build finishes; runs a manual smoke test pass first, especially for AI-generated or heavily-templated screens where subtle layout bugs are easy to miss.
- **Sets up staged rollouts** on Google Play (e.g. release to 10% of users first) to catch crash spikes before a full rollout, rather than pushing to 100% immediately.
- **Uses OTA updates (EAS Update / CodePush-style tools) for JS-only fixes** post-release — lets small bug fixes ship instantly without going through a full app store review cycle again, while reserving full store submissions for native changes.
- **Automates the parts that don't need a human**: CI triggers on merge to `main` → build → auto-submit to internal testing track, with public release still gated behind a manual approval step.
- **Tracks store policy changes proactively** — subscribes to Apple/Google developer newsletters, since SDK target deadlines (like the Android 16 / iOS 26 SDK requirements) are hard cutoffs that block submissions if missed.

---

## Quick Reference

| | Apple App Store | Google Play Store |
|---|---|---|
| Account cost | $99/year | $25 one-time |
| Build output | `.ipa` | `.aab` |
| Review time | ~24–48 hrs | Often hours |
| Current SDK requirement | iOS 26 SDK (Apr 2026) | API 36 / Android 16 (Aug 2026) |
| First submission | Can be automated | Must be manual |
| Build/submit tool | `eas build/submit --platform ios` | `eas build/submit --platform android` |