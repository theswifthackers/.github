# Scan signals

Use these searches first. File names and imports are usually enough to decide which guidelines apply. Read the matching files; do not infer a feature from a single string if the call site is dead or `#if DEBUG`.

Prefer ripgrep over opening every Swift file.

---

## Project and identity

| Look in | Why |
|---------|-----|
| `*.xcodeproj/project.pbxproj`, `*.xcworkspace`, `Package.swift`, `Podfile` | Targets, packages, min OS, SDK |
| `PRODUCT_BUNDLE_IDENTIFIER`, `CFBundleDisplayName`, `CFBundleShortVersionString` | Identity; reject `com.example.*` for submission |
| `IPHONEOS_DEPLOYMENT_TARGET`, `SDKROOT` | Compatibility vs current shipping OS (2.5.1) |
| `README.md`, `AppStore/`, `fastlane/metadata/`, `*.storekit` | Listing copy and IAP config |

---

## Privacy, permissions, manifests

Search:

```
NSCameraUsageDescription|NSMicrophoneUsageDescription|NSPhotoLibraryUsageDescription|NSPhotoLibraryAddUsageDescription|NSLocationWhenInUseUsageDescription|NSLocationAlways
NSContactsUsageDescription|NSFaceIDUsageDescription|NSMotionUsageDescription|NSHealthShareUsageDescription|NSHealthUpdateUsageDescription
NSUserTrackingUsageDescription|NSBluetoothAlwaysUsageDescription|NSLocalNetworkUsageDescription|NSCalendarsUsageDescription
PrivacyInfo.xcprivacy|NSPrivacyAccessedAPITypes|NSPrivacyCollectedDataTypes|NSPrivacyTracking
ITSAppUsesNonExemptEncryption|NSAppTransportSecurity|NSAllowsArbitraryLoads
ATTrackingManager|AppTrackingTransparency|requestTrackingAuthorization
PHPicker|PhotosPicker|PHPhotoLibrary|CNContactStore|CLLocationManager
HealthKit|HKHealthStore|HomeKit|ClassKit|ARKit
```

Also open every `*.entitlements` and compare to APIs that are actually called. Unused HealthKit, Push, Wallet, or Personal VPN entitlements are a 5.1 / 2.5.4 smell.

Required-reason API categories to flag if `PrivacyInfo.xcprivacy` is missing or incomplete:

- UserDefaults (`UserDefaults`, `NSUserDefaults`)
- File timestamp (`creationDate`, `contentModificationDate`, `getattrlist`, `stat`)
- Disk space (`volumeAvailableCapacity`, `systemFreeSize`)
- System boot time (`systemUptime`, `kern.boottime`)
- Active keyboards (`activeInputModes`)

Do not invent reason codes. If usage is unclear, mark **Needs human** and point at [Describing use of required reason API](https://developer.apple.com/documentation/bundleresources/privacy_manifest_files/describing_use_of_required_reason_api).

---

## Accounts and login

```
SignInWithApple|ASAuthorizationAppleID|AuthenticationServices
GIDSignIn|GoogleSignIn|FBSDKLogin|FacebookLogin|OAuth
LoginView|SignInView|createAccount|signUp|deleteAccount|accountDeletion
revoke|ASAuthorizationAppleIDProvider.revoke
```

Decision tree:

1. Significant account features? If no → forced login is 5.1.1(v).
2. Account creation? → in-app **Delete Account** required (not a mailto-only path).
3. Third-party social login as the primary account? → 4.8 equivalent (usually Sign in with Apple) unless an exception applies.
4. Sign in with Apple + deletion? → token revocation belongs in the deletion flow.

---

## Payments

```
StoreKit|Product.purchase|Transaction.currentEntitlements|SKPayment|restorePurchases
SubscriptionStoreView|StoreView|RevenueCat|Purchases.shared
Stripe|PayPal|Braintree|checkout|license key|unlockFullVersion
ExternalLinkAccount|StoreKit External Purchase|music streaming entitlement
ApplePay|PKPaymentAuthorization
```

Classify the good:

- **Digital feature / content / subscription / coins** → must be IAP unless a written 3.1.3 or entitled 3.1.1(a) path applies.
- **Physical goods or real-world appointment** → must **not** use IAP (3.1.3(e)).
- **Reader / multiplatform / enterprise / P2P** → allowed other methods; still flag in-app “buy on the website” CTAs outside allowed storefronts.

Always check restore for non-consumables and subscriptions.

---

## UGC, social, generative AI

```
comment|reportContent|blockUser|moderat|UGC|chatRoom|MessageThread
OpenAI|Anthropic|Gemini|LLM|generateImage|ChatGPT
WKWebView|loadHTMLString|mini.?app|emulator
```

If users can publish text, photos, audio, or model output that other users see, require 1.2’s four controls. If user content is sent to a third-party model, require 5.1.2(i) disclosure + explicit permission.

---

## Kids

```
For Kids|For Children|Kids Category|parentalGate|ageGate|COPPA|under 13
```

Reserved store wording is enough to apply 1.3 + 2.3.8 even if the binary is a general app.

---

## Web, wrappers, spam, other platforms

```
WKWebView|SFSafariViewController|loadRequest\(URLRequest
Android|Google Play|Play Store|apk|alternative marketplace
lorem ipsum|TODO:|coming soon|placeholder|sample@|test@example
UIWebView|evaluateJavaScript
```

A root screen that is only a `WKWebView` of a marketing site is 4.2 until native value is proven.

`SFSafariViewController` that is zero-size, offscreen, or covered is 5.1.1(vii).

---

## Media, recording, background

```
AVCaptureSession|AVAudioRecorder|RPScreenRecorder|isRecording
UIBackgroundModes|audio|location|voip|fetch|processing
UNUserNotificationCenter|requestAuthorization|Push Notifications
CallKit|NEVPNManager|ManagedSettings|DeviceActivity
```

Match each background mode and recording API to a user-visible feature and indicator (2.5.4, 2.5.14).

---

## Review prompts and discovery

```
SKStoreReviewController|requestReview|RequestReviewAction
Rate us|leave a review|unlock.*rate|download our other app
```

Custom review chrome is 5.6.1. Gating features on a rating is 3.2.2(x).

---

## Secrets and completeness

```
sk_live_|sk-proj-|AIza|BEGIN PRIVATE KEY|password\s*=\s*"|apiKey\s*=
fatalError\(|preconditionFailure\(|TODO: replace|com\.example
```

Hardcoded live keys are a 1.6 / 5.1 issue and a shipping defect. `com.example` bundle IDs are not submission-ready.

---

## Evidence ranking

1. Entitlements, Info.plist, privacy manifest, StoreKit config
2. Call sites in app targets (not demo snippets in comments)
3. Package.swift / resolved SDK list
4. In-app Settings / legal screens
5. README and marketing copy (weakest; use to find over-claims)

If a signal exists only in a test target or `#if DEBUG`, say so. Do not file it as a store blocker unless a release compilation path still includes it.
