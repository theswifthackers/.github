# Revision playbook

Concrete revisions for the gaps this skill finds most often. Adapt names to the app. Do not paste unused APIs “just in case.”

Each recipe: **when**, **change**, **done when**.

---

## Privacy policy and in-app legal (5.1.1(i), 1.5)

**When:** No policy URL, 404, or Settings has no legal links.

**Change:**

- Add Settings rows: Privacy Policy, Support (and Terms if you have them).
- Open the policy in `SafariViewController` or an in-app `WebView` that is fully visible.
- Tell the user the App Store Connect Privacy Policy field must use the same live URL.
- Policy body must name: data collected, how, why, third parties (analytics, ads, AI), retention, and how to delete / withdraw consent.

**Done when:** A reviewer can open a real policy from the app and from the listing without leaving a dead page.

Do not invent a hosted URL. If none exists, mark the finding **Needs human** and draft the policy outline only.

---

## Purpose strings (5.1.1(ii))

**When:** An API is called and the usage key is missing, or the string is “for a better experience.”

**Change:** One sentence, specific feature, no leftover keys for unused APIs.

```
NSCameraUsageDescription = "Take a photo of your receipt so the app can attach it to an expense."
NSLocationWhenInUseUsageDescription = "Show nearby recycling points on the map."
NSUserTrackingUsageDescription = "We'll use this to measure ad effectiveness across other apps. You can use the app without tracking."
```

**Done when:** Every called privacy API has a matching string, and unused keys/entitlements are gone.

---

## Photos and contacts minimization (5.1.1(iii))

**When:** `PHPhotoLibrary.requestAuthorization` or `CNContactStore.requestAccess` for a one-off pick.

**Change:** Prefer `PhotosPicker` / `PHPickerViewController` and a share sheet. If the user denies location, offer a typed address. Never require mic access to upload a photo.

**Done when:** The core task works with the system picker or a manual fallback.

---

## Forced login vs guest + account deletion (5.1.1(v))

**When:** First screen is Sign In and most features are local, **or** users can register but cannot delete.

**Change:**

1. Let people use non-account features without an account.
2. Put **Delete Account** in Settings (not buried, not email-only).
3. Confirm, re-authenticate, then delete the account **and** associated personal data.
4. If deletion is delayed, say when it completes.
5. If Sign in with Apple was used, revoke Apple tokens in that flow.
6. If some deletion happens on the web, the in-app button must open that specific form.

```swift
// Settings — required when the app creates accounts
Button("Delete Account", role: .destructive) {
    pendingDeletion = true
}
```

**Done when:** A reviewer can create (or use the demo account) and finish deletion inside the app.

---

## Sign in with Apple (4.8)

**When:** Google / Facebook / X / WeChat / Amazon / LinkedIn is offered as a **primary** account login.

**Change:** Offer an equivalent option that: collects only name + email, can hide the email, and does not track app activity for ads. Sign in with Apple is the usual implementation. Place it with the other login buttons, not behind a secondary menu.

Skip this if an official 4.8 exception applies (first-party auth only, enterprise/education SSO, government eID, or a client that must sign into that exact third-party account).

**Done when:** The reviewer can create the primary account without a tracking-heavy social login.

---

## Demo account and review notes (2.1, Before You Submit)

**When:** Anything behind auth, IAP, hardware, or a QR/code gate.

**Change:** Provide a full-privilege demo account **or** a built-in demo mode that exercises every featured flow. Put this in App Review notes:

```
Demo account
Username: review@example.com
Password: <do not invent — ask the owner>

How to review
1. Launch → skip onboarding with “Reviewer demo” (Settings → Reviewer, if present)
2. Home → <core feature path>
3. Settings → Privacy Policy, Delete Account

In-app purchases
- <product id> — <where it appears>

Hardware / extras
- Sample QR is attached
- Backend is production, region <x>
```

**Done when:** Notes are specific. Generic “please test the app” is a rejection risk.

---

## StoreKit for digital goods (3.1.1, 3.1.2)

**When:** Stripe/PayPal/license key/QR unlocks a digital feature, or IAP exists without restore / subscription disclosure.

**Change:**

- Move digital unlocks to StoreKit 2 (`Product.purchase`, `Transaction.currentEntitlements`).
- Keep a Restore Purchases control.
- Before subscribe: what they get, length, price, auto-renew until canceled, how to cancel.
- Loot boxes: show odds before purchase.
- Do not expire IAP currency.
- Grandfather customers who already paid for a permanent unlock if you later add subscriptions.
- Physical goods stay on Apple Pay / card — do **not** force those through IAP.

**Done when:** A reviewer can find, buy (or sandbox-buy), and restore every digital product named in the listing.

External web checkout: only recommend it where 3.1.1(a) / 3.1.3 and the storefront allow it. Default advice is IAP.

---

## UGC moderation (1.2)

**When:** Users can post, comment, chat, or publish creator content.

**Change:** Ship all four:

1. Filter / hide objectionable material (including a default-off NSFW switch if the content is incidental from the web)
2. Report with a path you will actually handle
3. Block a user
4. Published contact

Plus a terms/community standard. Creator catalogs also need age rating labels and an age gate for mature items (1.2.1, 4.7.5).

**Done when:** The reviewer can report and block from a piece of user content without leaving the app.

---

## Kids-directed experience (1.3, 2.3.8, 5.1.4)

**When:** Kids Category, or “For Kids” / “For Children” in name, subtitle, icon, or screenshots.

**Change:**

- Parental gate before outbound links, IAP, or other distractions.
- No third-party analytics/ads unless they meet the narrow 1.3 exceptions (no IDFA / child identifiers; contextual ads with human review).
- Privacy policy; no “for kids” wording unless you stay in Kids Category in later updates.

If the app is **not** for kids, delete that wording from metadata and UI.

**Done when:** A child path cannot reach commerce or third-party tracking without a parent gate, **or** the kids claims are gone.

---

## Tracking and third-party AI (5.1.2)

**When:** Ad networks, IDFA, or personal data sent to an external model.

**Change:**

- ATT only after a factual pre-prompt; never on a cold first launch with no context.
- Gate IDFA / tracking reads on authorization status.
- Do not require tracking, push, or location to use the app or to get rewards.
- Before sending personal data to a third-party AI: name the provider, state the purpose, get explicit permission, and mirror that in the privacy policy.

**Done when:** Declining ATT still leaves the app usable, and AI sharing is opt-in.

---

## Privacy manifest (upload + 5.1)

**When:** No `PrivacyInfo.xcprivacy`, or UserDefaults / file timestamps / disk space / boot time / keyboards are used with no reason.

**Change:** Add an app-target `PrivacyInfo.xcprivacy` via Xcode’s editor. Declare only APIs **this** target uses, with approved reason codes. Update SDKs so they ship their own manifests. Then align App Privacy labels with the merged Privacy Report.

**Done when:** Archive → Privacy Report matches the label, and no required-reason API is undeclared.

Do not copy random reason codes from another app.

---

## Web wrapper / minimum functionality (4.2, 4.3)

**When:** The app is a site in a `WKWebView`, a brochure, or a category clone.

**Change:** Add native value that cannot be a bookmark: offline store, widgets, SharePlay, system share, camera pipeline, on-device data, or unique content. One worldwide app beats one bundle ID per city.

If there is no path to “app-like,” recommend Safari / a web app instead of arguing with 4.2.

**Done when:** A reviewer who ignores the website still gets a complete native task.

---

## Other-platform branding (2.3.10)

**When:** UI or listing mentions Android, Google Play, APKs, or other store chrome.

**Change:** Remove names, icons, and screenshots of other mobile platforms unless there is approved interactive functionality. Keep metadata about *this* app’s Apple-platform experience.

**Done when:** Store art and in-app copy are Apple-platform specific.

---

## Push used as marketing (4.5.4)

**When:** Push is required to enter the app, or promo blasts have no opt-in.

**Change:** The app must work with notifications off. Marketing / promo pushes need in-app consent language and an in-app opt-out. Do not put secrets in notification payloads.

**Done when:** Core features work after denying notification permission.

---

## Custom review prompt (5.6.1, 3.2.2(x))

**When:** A home-built “enjoying the app? Rate us” modal, or a feature locked behind a review.

**Change:** Use `requestReview(in:)` / `RequestReviewAction` only. Never require a rating to unlock content.

**Done when:** No custom review UI remains.

---

## Recording and location (2.5.14, 5.1.5)

**When:** Camera, mic, or screen capture starts without UI, or location is incidental.

**Change:** Ask in context, show the system indicator, and explain location in the app UI—not only in the plist. Drop Always location if When In Use is enough. Do not market the app as emergency dispatch.

**Done when:** A user can tell they are being recorded, and location maps to a real feature.

---

## Health claims (1.4.1, 5.1.3)

**When:** Sensor-only vitals, HealthKit writes, or health data used for ads.

**Change:** Remove unverifiable device-only medical claims. Disclose methodology. Remind users to consult a clinician. Do not write fake samples to HealthKit. Do not store PHI in iCloud. Do not send health data to ad/analytics SDKs.

**Done when:** Remaining claims are supportable, and health data stays in the health context.

---

## After implementing

Re-run the scan on touched files. Update the review report: move fixed items to **What looks solid**, keep anything still **Needs human** (live URLs, licenses, age questionnaire, trader status, sandbox IAP).
