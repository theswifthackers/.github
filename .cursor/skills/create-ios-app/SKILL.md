---
name: create-ios-app
description: Create or modify iOS apps with Swift and SwiftUI. Handles both new apps (full scaffold from scratch) and existing apps (refactor/recode). Use when the user mentions iOS, SwiftUI, UIKit, Xcode, widgets, or Apple app development. Automatically detects whether the repo already has Swift code and routes to the correct workflow.
---

# Create iOS App

**Step 0 — Detect mode before doing anything else.**

---

## Step 0 — New app or existing app?

Scan the current workspace for Swift source files (`.swift`) before responding.

| What you find | Mode | What to do |
|---------------|------|------------|
| No `.swift` files — empty or non-Swift repo | **New App** | Continue with Phase 1 below |
| `.swift` files exist | **Existing App** | Stop. Use the `refactor-ios-app` skill instead |

If you cannot scan the workspace (e.g. no filesystem access), ask the user directly:
> "Is this a brand-new app or an existing codebase?"

---

## Phase 1 — Discovery (new apps only — ask before writing any code)

Ask all of the following before scaffolding anything:

1. **What does the app do?** One or two sentences. What problem does it solve, and for whom?
2. **What are the core features?** List every screen or major capability (e.g. "list of items, detail view, favorites, settings, push notifications").
3. **What platform(s)?** iPhone only, iPhone + iPad, or all Apple platforms?
4. **What integrations does it need?** (Login, camera, location, maps, payments, widgets, notifications, local data, remote API, etc.)
5. **Is there a design direction?** Colours, brand, any existing screenshots or references?
6. **Minimum iOS version?** Default is iOS 17 unless the user needs iOS 26 / Swift 6 strict concurrency.

Only proceed once you have enough answers to produce a meaningful plan. If the user is vague, make reasonable assumptions and state them explicitly in the plan.

---

## Phase 2 — App Plan (produce before writing any code)

Output a short structured plan in this format:

```
## App Plan: <App Name>

### Purpose
<One sentence>

### Target Platform
<iPhone / iPhone + iPad / etc.>, iOS <version>+

### Features
1. <Feature name> — <what it does>
2. ...

### Screen List
- <Screen name> (<purpose>)
- ...

### Key Integrations
- <Integration> — <why>

### Architecture
- Pattern: MVVM + @Observable
- Navigation: NavigationStack (+ TabView if multi-tab)
- Persistence: <SwiftData / AppStorage / URLSession>
- Concurrency: async/await + @MainActor

### Project Structure (folder tree)
<abbreviated tree>

### Open Questions / Assumptions
- <assumption>
```

Get explicit or implicit confirmation before writing code. You may start immediately after presenting the plan if the user says "go ahead" or similar.

---

## Defaults (override only when user requests)

| Concern | Default |
|---------|---------|
| UI | SwiftUI |
| Architecture | MVVM + `@Observable` |
| Concurrency | `async/await` + `@MainActor` on all UI-bound VMs |
| Min deployment | iOS 17 |
| Persistence | SwiftData (local) / URLSession (remote) |
| Navigation | `NavigationStack` + value-based routing |
| Package manager | Swift Package Manager |
| Third-party deps | None unless user asks |

---

## Swift Language Rules

These come from battle-tested production patterns. Apply them to every file you write:

### Always use modern API

| ❌ Old / wrong | ✅ Modern |
|----------------|-----------|
| `foregroundColor()` | `foregroundStyle()` |
| `cornerRadius()` | `clipShape(.rect(cornerRadius:))` |
| `tabItem()` | `Tab` initializer (iOS 18+) |
| `NavigationView` | `NavigationStack` |
| `ObservableObject` + `@Published` | `@Observable` class |
| `@StateObject` / `@ObservedObject` | `@State` + `@Observable` |
| `DispatchQueue.main.async` | `await MainActor.run` or `@MainActor` |
| `DateFormatter` / `NumberFormatter` | `.formatted()` / `FormatStyle` |
| `Task.sleep(nanoseconds:)` | `Task.sleep(for:)` |
| `UIScreen.main.bounds` | `GeometryReader` → prefer `containerRelativeFrame()` |
| `GeometryReader` | `containerRelativeFrame()` or `visualEffect()` when sufficient |
| `onTapGesture` for actions | `Button` (reserve `onTapGesture` for location/count needs) |
| `onChange(of:) { v in }` (1-param) | `onChange(of:) { old, new in }` |
| `AnyView` | Concrete `View` types or generics |
| `fontWeight(.bold)` | `bold()` |
| `showsIndicators: false` in ScrollView | `.scrollIndicators(.hidden)` |
| `Array(seq.enumerated())` in ForEach | `ForEach(seq.enumerated(), id: \.element.id)` |
| `UIGraphicsImageRenderer` for SwiftUI | `ImageRenderer` |

### Concurrency
- All `@Observable` classes must be `@MainActor` unless you have a specific reason.
- Never use `DispatchQueue`, `NSLock`, or other GCD primitives for new code.
- Use `actor` for shared mutable state accessed from multiple concurrent contexts.
- Use `.task { }` modifier for view-lifecycle async work, not `onAppear`.

### Formatting & strings
- Never use `String(format:)` for numbers — use `Text(value, format: .number)`.
- Filter user-input text with `localizedStandardContains()`, not `contains()`.
- Use `URL.documentsDirectory` and `appending(path:)`.
- Avoid force unwraps; use `guard let` or `try?` with explicit fallbacks.

### Code organisation
- One type per file.
- No computed-property sub-views — break into `View` structs.
- Add `#Preview` to every screen.

---

## Project Structure

```
AppName/
├── AppNameApp.swift           # @main, WindowGroup, modelContainer
├── ContentView.swift          # Root tab host or NavigationStack shell
├── Features/
│   ├── FeatureName/
│   │   ├── FeatureNameView.swift
│   │   └── FeatureNameViewModel.swift
│   └── Settings/
│       └── SettingsView.swift
├── Core/
│   ├── Models/                # @Model / Codable value types
│   ├── Services/              # Networking, persistence, system APIs
│   ├── Components/            # Reusable SwiftUI pieces
│   └── Theme/                 # Color tokens, Typography, Spacing
├── Resources/
│   ├── Assets.xcassets
│   └── Localizable.xcstrings
└── Info.plist                 # Only for custom capability keys
```

Keep features vertical: view + view model + feature-specific models stay together.

---

## MVVM Pattern

```swift
// ViewModel
@MainActor
@Observable
final class HomeViewModel {
    private(set) var items: [Item] = []
    private(set) var isLoading = false
    private(set) var errorMessage: String?
    private let service: ItemServiceProtocol

    init(service: ItemServiceProtocol = ItemService.shared) {
        self.service = service
    }

    func load() async {
        isLoading = true
        errorMessage = nil
        defer { isLoading = false }
        do { items = try await service.fetchItems() }
        catch { errorMessage = error.localizedDescription }
    }
}

// View
struct HomeView: View {
    @State private var viewModel = HomeViewModel()

    var body: some View {
        List(viewModel.items) { item in Text(item.title) }
            .task { await viewModel.load() }
    }
}
```

Always expose three states on data screens: **loading**, **empty**, **error**. See `templates.md` for the `AsyncContentView` wrapper.

---

## Design Rules (summary — see design.md for full details)

- **Spacing**: use a 4/8-pt grid only. Allowed values: `4 8 12 16 20 24 32 40 48`. Never arbitrary values.
- **Typography**: five or fewer distinct font sizes; one `design` variant throughout (including widgets).
- **Colors**: use semantic system colors (`Color(.systemBackground)`, `.secondary`, `.tertiary`). Avoid hardcoded hex or `Color.white.opacity(0.32)` soup.
- **Cards/groups**: `Color(.secondarySystemBackground)` + `clipShape(.rect(cornerRadius: 10))`. Never gradients or decorative borders on standard cards.
- **Corner radius**: 10 pt for cards/groups. Only go higher for explicitly pill-shaped or branded surfaces.
- **Dividers**: `Divider().padding(.leading, 16)`. Never build custom divider views.
- **NavigationStack**: always — never bare `ZStack` as a navigation root.
- **Dynamic Type**: don't force fixed font sizes for body text. Use `.body`, `.caption`, etc.
- **Dark Mode**: use semantic colors; they adapt automatically.
- **SF Symbols**: prefer over custom icons; pair with text in buttons (`Button("Label", systemImage: "icon")`).

---

## Interface Writing Rules (summary — see writing-for-interfaces.md for full details)

- **Purpose first**: every screen has one primary message; everything else is secondary.
- **Anticipate next steps**: after an error → tell them how to fix it. After a success → point forward.
- **Be specific**: "Can't open 'Report.pdf'" not "Can't open this file."
- **Remove filler**: "Simply tap…" → "Tap…". No "Oops!", no "Successfully saved".
- **Consistent terminology**: pick one word per concept, use it everywhere (not "alias" then "username").
- **Button copy**: name the action — "Cancel Subscription", not "Yes".
- **Error messages**: lead with what went wrong, then how to recover.

---

## Accessibility

- Add `.accessibilityLabel` where icon-only buttons lack text.
- Support Dynamic Type; test at largest accessibility size.
- Provide `.accessibilityElement(children: .combine)` for composite cells.
- Respect `@Environment(\.accessibilityReduceMotion)` in animations.
- Color is not the only differentiator for status (add shape or label).

---

## Widgets (WidgetKit) — see widgets.md for full details

Only add a Widget extension when explicitly requested. Key rules at a glance:

- Use an App Group for data shared between app and widget.
- Never read `UserDefaults.standard` in the extension — always `UserDefaults(suiteName:)`.
- Never use `onTapGesture` in widgets — only `Button(intent:)` / `Toggle(intent:)`.
- Always call `WidgetCenter.shared.reloadTimelines(ofKind:)` after mutations.
- Always add `containerBackground(.fill.tertiary, for: .widget)`.
- Lock Screen circular widgets → use `Gauge` with `.accessoryCircular`, not manual circles.
- Match timeline refresh rate to data granularity (midnight for daily data; not every minute).

---

## Permissions & Info.plist

Add usage descriptions only for APIs you call:

| Capability | Key |
|------------|-----|
| Camera | `NSCameraUsageDescription` |
| Photo library | `NSPhotoLibraryUsageDescription` |
| Location when in use | `NSLocationWhenInUseUsageDescription` |
| Microphone | `NSMicrophoneUsageDescription` |
| Face ID | `NSFaceIDUsageDescription` |

Write plain human-readable purpose strings. App Review rejects vague copy.

---

## Deliverables Checklist

When asked to create an app, always produce:

- [ ] `README.md` with: open-in-Xcode steps, min iOS version, feature list, required permissions, API key / bundle ID TODOs
- [ ] Full folder structure with compiling Swift sources
- [ ] `#Preview` on every screen
- [ ] `Assets.xcassets` with AppIcon placeholder structure
- [ ] Explicit `// TODO: replace with your Bundle ID / Team ID` markers
- [ ] No hardcoded secrets, API keys, or provisioning profiles

---

## Supporting files

| File | What's inside |
|------|---------------|
| [architecture.md](architecture.md) | MVVM layering, DI, state patterns, anti-patterns |
| [templates.md](templates.md) | Starter SwiftUI screens (tabs, list/detail, forms, async states) |
| [api-recipes.md](api-recipes.md) | SwiftData, Photos, location, keychain, push, App Store checklist |
| [design.md](design.md) | Full HIG, spacing grid, typography, colors, components, widget design |
| [widgets.md](widgets.md) | Full WidgetKit reference: providers, families, Controls, Live Activities |
| [writing-for-interfaces.md](writing-for-interfaces.md) | Voice, tone, copy principles, editing craft |
