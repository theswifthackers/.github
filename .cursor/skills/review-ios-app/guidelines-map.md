# Guideline map

Official text: [App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/). This file is a **reviewer map**, not a substitute. Summaries are short so you can match evidence; quote the live page when a finding is contested.

Last published update reflected here: **8 June 2026**.

How to use a row:

- **Look for** — signals in the repo or listing
- **Fail if** — what usually triggers rejection
- **Revise** — the default fix (see [revision-playbook.md](revision-playbook.md))

---

## Before You Submit

These are process checks. Missing any of them delays review even when the binary is fine.

| ID | Rule | Look for | Fail if | Revise |
|----|------|----------|---------|--------|
| BYS-1 | Test for crashes and bugs | No tests, force-unwraps, empty error paths, `fatalError` in launch | Known crash, placeholder screens, broken first launch | Stabilize launch + core path; say what you tested |
| BYS-2 | Accurate complete metadata | README, screenshots, draft listing | Lorem, “coming soon”, Android chrome, prices in name | Rewrite listing to match the shipping build |
| BYS-3 | Contact information | Support URL, in-app help, `mailto` | 404, social-only, no way to reach the developer | Add a real support page and in-app contact (1.5) |
| BYS-4 | Reviewer access | Demo account, demo mode, QR sample | Login wall with no credentials in notes | Demo account **or** full demo mode; backend live |
| BYS-5 | Live backend | API base URLs, feature flags | Staging-only, expired tokens, localhost | Production (or review) environment that works |
| BYS-6 | Review notes | Non-obvious hardware, IAP, regulated claims | Reviewer cannot find a featured feature | Draft notes: where to tap, what to buy, what to attach |
| BYS-7 | Other Apple docs | HIG, trademark, Apple Pay, Wallet | Fake system UI, misused Apple marks | Fix branding and HIG-breaking chrome |

---

## 1. Safety

| ID | Rule | Look for | Fail if | Revise |
|----|------|----------|---------|--------|
| 1.1 | Objectionable content | Copy, assets, generated output, “hot or not”, prank/fake-tracker features | Hate, porn, realistic torture, fake device data, prank calls, disaster-profiteering | Remove or gate; do not hide behind “for entertainment” |
| 1.2 | User-generated content | Posts, comments, chat, profiles, creator feeds, reports | UGC with no filter, report, block, or published contact | Add all four: filter, report + timely response, block, contact |
| 1.2.1 | Creator content | In-app creators, UGC games, tips | Unmoderated catalog; paid extras not disclosed | Treat as UGC + 3.1.1; age-gate mature creator content |
| 1.3 | Kids Category | Kids category, “For Kids”, under-13 features | Links, IAP, or ads in the kids experience without a parental gate; third-party analytics/ads; PII to third parties | Parental gate; strip third-party tracking; keep the commitment in later updates |
| 1.4.1 | Medical accuracy | Health claims, sensor-only vitals | “This iPhone measures blood pressure / glucose / SpO2 / x-ray” without validated method | Remove unverifiable claims; disclose method; doctor disclaimer; attach clearance if any |
| 1.4.2 | Drug dosage | Dosage calculators | Unaffiliated dosage app | Only from an approved health entity / regulator |
| 1.4.3–5 | Harmful activities | Alcohol, tobacco, DUI, reckless challenges | Encourages minors, illegal sales, drunk driving, physical-harm bets | Remove encouragement; DUI data only from law enforcement |
| 1.5 | Developer information | Settings, support, Wallet passes | No contact path; stale classroom-app contact | In-app + Support URL contact; Wallet issuer contact |
| 1.6 | Data security | Keychain vs UserDefaults, HTTPS, logging | Tokens in UserDefaults/logs, cleartext, leaked secrets | Keychain, TLS, no secrets in repo |
| 1.7 | Crime-reporting apps | Tip-line / crime-report features | No local law-enforcement involvement | Restrict regions; involve law enforcement |

---

## 2. Performance

| ID | Rule | Look for | Fail if | Revise |
|----|------|----------|---------|--------|
| 2.1 | Completeness | Placeholders, `#TODO` UI, login, IAP products | Crash, empty website, missing IAP, no demo access | Ship a finished first-run; demo account or approved demo mode |
| 2.2 | Beta testing | “Beta”, “test”, “WIP” in listing or UI | Production listing for a demo/trial | TestFlight for betas; strip beta wording from release |
| 2.3.1 | Hidden / misleading features | Debug menus, review bypass, fake antivirus | Undocumented features; marketing a capability the app lacks | Document in review notes or remove; match marketing to binary |
| 2.3.2 | IAP disclosure | StoreKit products in screenshots/description | Featured items that are actually paid, with no disclosure | Show that levels/subs require purchase |
| 2.3.3–4 | Screenshots and previews | Marketing assets in repo | Title-art-only shots; preview that is not the app | Capture the running app; previews = screen recordings |
| 2.3.5–7 | Category, age, name, keywords | 30-char name, competitor names, prices in metadata | Keyword stuffing, competitor marks, unverifiable claims | Unique name; honest age answers; no prices in name/subtitle |
| 2.3.8 | 4+ metadata; “For Kids” reserved | Icon, screenshots, “for children” copy | Violent icon; kids wording outside Kids Category | 4+ safe store art; drop reserved kids terms or join Kids |
| 2.3.9–13 | Rights, other platforms, What’s New, events | Android/Play icons; real people’s data in shots; event deep links | Other-OS branding; real accounts in screenshots; vague What’s New | Fictional demo data; Apple-platform focus; specific What’s New |
| 2.4.1–2 | iPad + power | iPhone-only layout, tight loops, mining | Unusable iPad letterbox; heat/battery abuse; in-app mining | Basic iPad layout; no unrelated background work |
| 2.4.4 | System settings | Copy that says “turn off Wi-Fi / disable security” | Requires unrelated system changes | Remove; only ask for settings tied to the feature |
| 2.5.1 | Public APIs, intended use | HealthKit, HomeKit, private selectors | Private API; HealthKit used as a generic database | Public APIs only; use frameworks for their purpose |
| 2.5.2 | Self-contained binary | Code download, JS bundles that add features | Downloading executable features after review | Keep features in the reviewed binary (see 4.7 exceptions) |
| 2.5.4 | Background modes | `UIBackgroundModes` | Location/audio/VoIP mode with no matching feature | Remove unused background modes |
| 2.5.5 | IPv6 | Hardcoded IPv4, custom networking | Breaks on IPv6-only | Use system URLSession / happy eyeballs |
| 2.5.6 | WebKit | `WKWebView`, custom engines | Non-WebKit browser without entitlement | Use WebKit unless an approved engine entitlement exists |
| 2.5.9 | Native controls | Hijacked volume/silent switch, blocked outgoing links | Breaks expected system behavior | Don’t override standard switches or expected links |
| 2.5.11 | Siri / Shortcuts | Intent definitions | Unrelated intents; ads before fulfillment | Only intents the app can finish; no ads in the path |
| 2.5.13 | Face auth | ARKit face login | Face login not using LocalAuthentication; no under-13 fallback | LocalAuthentication + alternate auth under 13 |
| 2.5.14 | Recording indicator | Camera, mic, screen record | Silent recording | Consent + visible/audible indicator |
| 2.5.15 | File picker | Document picker | Files/iCloud documents omitted | Include Files and iCloud docs |
| 2.5.16 | Widgets / extensions / clips | WidgetKit, App Clips, notifications | Unrelated widget; ads in clip/widget | Keep them on-app; no ads in clips; clip features ⊆ app |
| 2.5.18 | Ads | Ad SDKs | Ads in widgets/clips/watch; no report/close; health/kids targeting | Ads only in main app; skip control; report path; no sensitive targeting |

---

## 3. Business

| ID | Rule | Look for | Fail if | Revise |
|----|------|----------|---------|--------|
| 3.1.1 | In-app purchase | StoreKit vs Stripe/PayPal/license keys/QR unlocks | Digital features unlocked outside IAP (except entitled/US-link cases) | Use IAP for digital unlocks; restore non-consumables; disclose loot-box odds |
| 3.1.1(a) | External purchase links | External-link entitlement, “buy on web” buttons | Web checkout CTA outside allowed storefronts/entitlements | Region-gate; do not recommend worldwide web checkout |
| 3.1.2 | Subscriptions | Auto-renew products | No ongoing value; no pre-purchase disclosure; trapped upgrades; paid users lose prior unlocks | 7+ day period; clear terms; seamless up/downgrade; grandfather paid unlocks |
| 3.1.3 | Other purchase methods | Reader, multiplatform, enterprise, P2P, physical goods | Physical goods forced through IAP; digital goods on Stripe; “buy on web” nag | Physical/P2P/enterprise/reader exceptions only; no in-app steering except allowed regions |
| 3.1.4 | Hardware unlocks | Accessory pairing | Unrelated product required to unlock software | Hardware-tied features OK; still offer IAP when the product is optional |
| 3.1.5 | Crypto | Wallets, mining, exchanges | On-device mining; individual-dev wallet; task-for-coins | Org enrollment; off-device mining only; licensed exchanges |
| 3.2.1–2 | Other business | Charity, loans, binary options, forced ratings | In-app charity (non-nonprofit); APR > 36% or ≤60-day full repayment; forced review to unlock | Follow the acceptable list; never gate features on a rating |
| 3.2.2(iii) | Ad farms | Ad-heavy shells | App exists mainly to show ads | Add real utility; don’t fake clicks |

---

## 4. Design

| ID | Rule | Look for | Fail if | Revise |
|----|------|----------|---------|--------|
| 4.1 | Copycats | Clone UI, another app’s name/icon | Impersonation or trivial reskin | Distinct name, icon, and product idea |
| 4.2 | Minimum functionality | WKWebView-only, brochure, song/book, AR “drop a model” | Repackaged website; no lasting utility | Native value: offline, widgets, system integration, unique content |
| 4.2.3 | Standalone + download size | Large on-demand resources | Requires another app; silent huge download | Work alone; disclose size and prompt first |
| 4.2.6 | Template / generator apps | White-label leftovers | Commercial template submitted by the agency | Content owner submits; or one picker binary for many clients |
| 4.3 | Spam | City-per-bundle, flashlight/timer clones | Duplicate bundle IDs; me-too category with no new experience | One app + IAP variations; ship a meaningfully different experience |
| 4.4 | Extensions | Keyboards, Safari extensions | Keyboard needs full access; Safari over-broad site access; ads/IAP in extension | Follow extension rules; disclose in marketing |
| 4.5.1–6 | Apple services | Scraping apple.com; MusicKit; push; Game Center IDs | Scraped rankings; monetized Apple Music; required push; marketing push without opt-in | Official APIs only; push optional; marketing push = opt-in + in-app opt-out |
| 4.7 | Mini apps / emulators / chatbots | Embedded HTML5, streaming games, ROM download | No UGC tools, no IAP for digital goods, no software index/age gate | Host is responsible; apply 1.2, 3.1, 5.1 to embedded software |
| 4.8 | Login services | Google/Facebook/X/WeChat login | Third-party social login without an equivalent privacy-preserving option | Add Sign in with Apple (or equivalent: name+email only, hide email, no ad tracking) unless an exception applies |
| 4.9 | Apple Pay | PassKit | Missing term/price/cancel info on recurring Apple Pay | Disclose term, what they get, charges, how to cancel |
| 4.10 | Monetizing built-ins | Paywall on camera, push, iCloud, Apple Music | Charging for OS capabilities themselves | Charge for *your* content/service, not the sensor or Apple service |

**4.8 exceptions (do not demand Sign in with Apple):** first-party login only; education/enterprise SSO; government eID; client that must log into that specific third-party account (mail/social); alternative marketplace login.

---

## 5. Legal

| ID | Rule | Look for | Fail if | Revise |
|----|------|----------|---------|--------|
| 5.1.1(i) | Privacy policy | In-app link + listing URL | Missing, 404, or not a real policy | Policy that lists data, third parties, retention, deletion |
| 5.1.1(ii) | Permission | Purpose strings, ATT, consent | Vague “to improve experience”; paid features require tracking | Specific purpose strings; withdrawable consent; paid ≠ data ransom |
| 5.1.1(iii–iv) | Minimization | Photo library vs picker; contacts | Full Photos/Contacts when a picker would do; forced permission | PhotosPicker / share sheet; alternatives if the user says no |
| 5.1.1(v) | Account sign-in | Login gate, delete-account | Forced login with no significant account features; create but no in-app delete | Guest path when possible; **in-app account deletion** (not mailto-only) |
| 5.1.1(vii) | SafariViewController | `SFSafariViewController` | Hidden/offscreen Safari used to track | Visible Safari only |
| 5.1.1(viii) | Scraped personal data | People-search, public-DB compile | Compiling personal info without consent | Don’t ship people-dossiers |
| 5.1.1(ix) | Regulated fields | Banking, health, gambling, cannabis, crypto, aviation | Individual developer submitting a bank/clinic/exchange app | Legal entity; geo-restrict cannabis |
| 5.1.2 | Use and sharing | Analytics, ads, third-party AI | Sharing personal data / sending it to an AI provider without explicit permission; tracking without ATT | Disclose + consent; ATT before tracking; no gating the app on ATT/push/location |
| 5.1.2(iv–v) | Contacts / installed apps | Contact upload, “invite all” | Building a contact DB; Select All invites; scanning other apps | Individual user-initiated invites; preview the outgoing message |
| 5.1.3 | Health | HealthKit, research | Health data to ads; false HealthKit writes; health data in iCloud; research without IRB/consent | Health data for care/research only; no iCloud for PHI; research consent + ethics board |
| 5.1.4 | Kids privacy | Age gate, COPPA/GDPR-K | Child PII + no policy; “for kids” outside Kids Category | Policy + parental consent as required; reserved wording only in Kids |
| 5.1.5 | Location | CoreLocation | Location not core; used as emergency service; no in-app explanation | When-in-use first; purpose in UI; no emergency-dispatch claims |
| 5.2 | Intellectual property | Third-party brands, YouTube/Music downloaders | Unlicensed media, Apple-lookalike UI, implied Apple endorsement | License or remove; no Apple-product clones; WeatherKit attribution |
| 5.3 | Gaming / gambling | Real-money games, raffles | IAP used as casino chips; unlicensed betting; Apple implied as sponsor | License + geo-restrict + free app; official rules; Apple is not the sponsor |
| 5.4 | VPN | NEVPNManager | Individual-dev VPN; data sold; no pre-purchase data screen | Org only; on-screen data declaration; no third-party sale of VPN data |
| 5.5 | MDM | MDM / config profiles | Unapproved MDM; data sold | Apple capability + eligible org; same data-screen rules as VPN |
| 5.6 | Code of conduct | Custom review prompts, review-gating | Custom “rate us” UI; buy/fake reviews; bait-and-switch prices | `SKStoreReviewController` / `RequestReviewAction` only; no discovery fraud |

---

## Upload and privacy-declaration rules (not always numbered)

These fail in App Store Connect **before** a human reviewer, or they fail 5.1 when the three privacy surfaces disagree.

| Signal | Fail if | Revise |
|--------|---------|--------|
| `PrivacyInfo.xcprivacy` | Required-reason APIs (`UserDefaults`, file timestamp, disk space, boot time, active keyboards) used with no approved reason — ITMS-91053 | Declare the **actual** reason codes in the app target |
| Third-party SDK manifest | Listed SDK missing its own manifest — ITMS-91061 | Update or replace the SDK |
| App Privacy label | Label says “not collected” but an SDK collects | Align label, manifest, and runtime |
| `ITSAppUsesNonExemptEncryption` | Missing, so every upload re-asks — or answered wrong | Set the key; file classification if not exempt |
| Unused entitlements | HealthKit/Push/Wallet on but unused | Delete unused entitlements |
| Purpose strings | API used, string missing or generic | Human-readable string that matches the feature |
| Nutrition vs policy | Policy, label, and code tell three stories | One inventory of data types and purposes |

---

## High-frequency first-submission set

If time is short, review these first. They dominate real rejection letters:

1. **2.1** — crash, incomplete, no demo account
2. **5.1.1** — privacy policy, purpose strings, account deletion, forced login
3. **4.2 / 4.3** — thin template or web wrapper
4. **3.1.1** — digital goods outside IAP
5. **4.8** — Google/Facebook login and no Sign in with Apple
6. **1.2** — UGC without report/block
7. **2.3** — screenshots/metadata that do not match the app
8. **Privacy manifest** — missing required-reason declarations
9. **1.5** — dead Support URL
10. **5.1.2** — third-party AI or tracking without consent
