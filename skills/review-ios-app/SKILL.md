---
name: review-ios-app
description: Review an iOS, iPadOS, or multi-platform Apple app against Apple's App Store Review Guidelines. Use when asked to review an app for App Store submission, check guideline compliance, find rejection risks, audit privacy/IAP/UGC/login, prepare App Review notes, or suggest revisions before submitting. Scan the codebase first, cite guideline numbers, and propose concrete revisions.
---

# Review an iOS App (App Store Guidelines)

Review the current app the way App Review will: **evidence first**, then gaps, then revisions.

**Source of truth:** [App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) (last published update reflected here: **8 June 2026**).

This skill does **not** replace Apple's review, give legal advice, or guarantee approval. It finds likely rejection and delay risks, then proposes revisions.

**Always run: Scan → Profile → Apply guidelines → Report → Revise. Do not edit files until the user asks to implement.**

If the live guidelines page is reachable, fetch it at the start of the review and prefer it when they disagree. Cite the official URL and guideline number on every finding.

The detailed rule map, search patterns, and fix recipes live next to this file:

- [guidelines-map.md](guidelines-map.md)
- [scan-signals.md](scan-signals.md)
- [revision-playbook.md](revision-playbook.md)

---

## 1. Confirm there is an app to review

| What you find | What to do |
|---------------|------------|
| Swift / Xcode / `Info.plist` / entitlements / app target | Continue |
| No app sources | Ask which app to review |
| User wants architecture cleanup, not store review | Stop; that is a refactor, not this skill |
| User wants a new app | Stop; scaffold first, then review |

---

## 2. Scan the codebase

Read-only. Use the search list in [scan-signals.md](scan-signals.md).

```
Scan checklist:
- [ ] App name, bundle ID, platforms, min OS, Xcode / SDK
- [ ] Entry point, screens, features, web-wrapper vs native value
- [ ] Info.plist: usage descriptions, ATS, encryption, background modes
- [ ] Entitlements vs APIs actually called
- [ ] PrivacyInfo.xcprivacy and third-party SDK manifests
- [ ] Account / login / Sign in with Apple / account deletion
- [ ] Payments: StoreKit, subscriptions, Stripe/PayPal, external links, restore
- [ ] UGC, chat, comments, creators, generative AI
- [ ] Kids / child-directed copy, analytics, ads
- [ ] Camera, mic, location, photos, contacts, HealthKit, tracking (ATT)
- [ ] Push, widgets, App Clips, extensions, WebKit
- [ ] Privacy policy, support contact, in-app legal links
- [ ] Secrets, private APIs, placeholder / lorem / TODO shipping copy
- [ ] README, screenshots, listing copy, App Review notes drafts
```

Treat source, project settings, entitlements, and privacy manifests as stronger evidence than README marketing claims.

---

## 3. Build the App Profile

Produce this **before** the findings list. It decides which guidelines apply.

```
## App Profile: <App Name>

### What it is
- Purpose: <one sentence>
- Platforms: <iPhone / iPad / Mac / Watch / …>, min OS <version>
- Audience: <general / kids / regulated / enterprise>
- Completeness: <prototype / TestFlight-ready / submission-ready>

### Capabilities in the binary
- Accounts: <none / optional / required> — providers: <…>
- Payments: <none / IAP / subscriptions / physical goods / reader / P2P>
- UGC or social: <no / yes — kinds>
- Generative AI: <no / yes — what is sent where>
- Sensitive APIs: <camera, mic, location, photos, contacts, HealthKit, tracking>
- Kids-directed: <no / yes / unclear>
- Ads / analytics / third-party SDKs: <list>
- Extensions / widgets / clips: <list>

### Submission surfaces (may be outside the repo)
- Privacy policy URL: <found / missing>
- Support URL / in-app contact: <found / missing>
- App Privacy labels vs manifest: <aligned / mismatch / unknown>
- Demo account or demo mode: <present / needed / N/A>
```

Mark each fact **verified**, **absent**, or **unknown**. Never invent App Store Connect answers.

---

## 4. Apply only the guidelines that fit

Do not dump all five guideline chapters. Load [guidelines-map.md](guidelines-map.md).

| If the app… | Prioritize |
|-------------|------------|
| Has login or account creation | 5.1.1(v), 4.8, 2.1 demo access |
| Collects any data or uses SDKs | 5.1.1, 5.1.2, Privacy Manifest, purpose strings |
| Sells digital features | 3.1.1, 3.1.2, restore, trial disclosure |
| Sells physical goods / real-world services | 3.1.3(e), Apple Pay 4.9 |
| Has UGC, chat, or generative AI | 1.2, 1.1, 4.7, age rating honesty |
| Is child-directed or says “for kids” | 1.3, 2.3.8, 5.1.4 |
| Uses location, health, camera, mic | 5.1.5, 5.1.3, 2.5.14, 1.4 |
| Looks thin, templated, or web-wrapped | 4.2, 4.3, 2.1 |
| Mentions Android / other stores in UI | 2.3.10 |
| Uses push, widgets, Siri, WebKit | 4.5.4, 2.5.16, 2.5.6, 2.5.11 |

Always run **Before You Submit** and **2.1 Completeness**.

| Severity | Meaning |
|----------|---------|
| **Blocker** | Very likely rejection or upload failure if submitted as-is |
| **High** | Common rejection or serious delay |
| **Medium** | Should fix before review; may pass depending on reviewer |
| **Low** | Quality / polish; unlikely to reject alone |
| **Needs human** | Legal, licensing, age rating, or business-model judgment you cannot prove from code |

---

## 5. Write the review report

Lead with the answer: can they submit, and what will likely bounce.

```
## App Store Review: <App Name>

**Guideline source:** https://developer.apple.com/app-store/review/guidelines/ (as of <date>)
**Verdict:** Ready / Not ready / Ready with notes
**Highest risk:** <one sentence>

### App Profile
<compact version of step 3>

### Findings

#### Blockers
- **[Guideline X.X]** <title>
  - Evidence: <file, key, or missing surface>
  - Why it fails: <what App Review will see>
  - Revision: <concrete change>
  - Owner: <code / App Store Connect / legal / content>

#### High
- …

#### Medium / Low
- …

#### Needs human
- …

### What looks solid
- <guideline-aligned things already present>

### Suggested revision plan
1. <highest-leverage fix>
2. …

### App Review notes draft
<ready-to-paste notes: demo account, hardware, non-obvious features, IAP, regulated docs>

### Out of scope / not verified
- <listing screenshots, live backend, on-device crash pass, licenses>
```

Rules for findings:

- Cite a **guideline number** on every issue. If it is an upload rule (privacy manifest, SDK version), say so and still give the fix.
- One finding per defect. Quote evidence. Do not guess hidden server behavior.
- Omit guidelines that do not apply. A short “Not applicable” list is fine for tempting false positives (Kids, IAP, VPN, gambling).
- Distinguish **code revisions** from **App Store Connect / metadata** work.

---

## 6. Suggest revisions, then implement only if asked

For each finding, propose a revision the user can accept. Use [revision-playbook.md](revision-playbook.md).

A good revision has: what to change, why (guideline + reviewer action), how, and done-when.

Order colliding fixes:

1. Completeness and crashes (2.1)
2. Privacy, permissions, account deletion (5.1)
3. Payments (3.1)
4. Safety / UGC / kids (1.x)
5. Design minimum functionality and spam (4.2, 4.3)
6. Metadata and review notes (2.3, Before You Submit)

Do **not** implement until the user says to apply the revisions (or a subset). If they say “fix the blockers,” implement only blockers.

When implementing:

- Touch only files needed for the accepted findings.
- Keep the same architecture; this is a compliance pass, not a recode.
- Never add unused permission strings, entitlements, or tracking to “look complete.”
- Write purpose strings in plain language that match the actual API use.
- Do not invent a privacy policy URL, support email, license, or demo password. Use `// TODO:` or ask.
- After edits, re-scan the changed surfaces and update the report.

---

## Reviewer posture

- If the binary cannot do it, do not recommend listing copy that claims it.
- “For Kids” / “For Children” in name, subtitle, icon, or screenshots is a Kids Category commitment (2.3.8, 1.3).
- Flag `#if DEBUG` store bypasses, undocumented admin panels, and review-only unlocks (2.3.1).
- Analytics, ads, crash, and AI SDKs are in scope for 5.1 and 1.3.
- External purchase links are not the same worldwide (3.1.1(a), 3.1.3). State the region assumption.
- If they only need notarization / alternative distribution, say which rules still apply and which are store-only.
