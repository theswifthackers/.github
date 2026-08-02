---
name: refactor-ios-app
description: Refactor, modernize, or make targeted changes to an existing iOS Swift/SwiftUI codebase. Use when Swift files already exist and the user wants to recode, modernize architecture, fix patterns, add features, or make any changes to an existing app. Always scans the codebase first before touching anything.
---

# Refactor iOS App

For **existing codebases** — Swift files are already present.

**Always run: Scan → Assess → Clarify → Plan → Implement. Never edit files before completing the plan.**

---

## Step 1 — Scan the codebase

Before asking the user anything, explore the project:

```
Scan checklist:
- [ ] Find the @main entry point (App struct)
- [ ] List all feature folders and their view/viewmodel files
- [ ] Identify the architecture pattern in use (MVC, MVVM, TCA, VIP, mixed?)
- [ ] Check for ObservableObject vs @Observable usage
- [ ] Check for NavigationView vs NavigationStack
- [ ] Check for UIKit view controllers mixed into SwiftUI
- [ ] Check the minimum deployment target (Info.plist or project settings)
- [ ] Check for third-party dependencies (Package.swift, Podfile, Cartfile)
- [ ] Note any obviously deprecated API (see deprecated API table in assessment)
- [ ] Find existing tests (XCTest, Swift Testing)
```

Read broadly — file names and class/struct names are often enough to map the architecture without reading every line.

---

## Step 2 — Assessment report

After scanning, produce this report **before asking the user what they want**:

```
## Codebase Assessment: <App Name>

### Current state
- Entry point: <file>
- Architecture: <what pattern is being used>
- iOS target: <version>
- Swift version / concurrency model: <old/new>
- UI framework: <SwiftUI / UIKit / mixed>
- Dependencies: <list or "none">
- Test coverage: <yes/partial/none>

### What's working well
- <observation>

### Modernisation opportunities
- <issue> — <recommended fix>
  e.g. "NavigationView → NavigationStack (deprecated iOS 16+)"
  e.g. "ObservableObject + @Published → @Observable (simpler, iOS 17+)"
  e.g. "DispatchQueue.main.async → @MainActor / async/await"
  e.g. "DateFormatter → .formatted() / FormatStyle"
  e.g. "cornerRadius() → clipShape(.rect(cornerRadius:))"

### Architectural concerns
- <concern> — <recommended fix>
  e.g. "Massive ContentView (400+ lines) — split into feature View structs"
  e.g. "Services called directly from views — extract to ViewModels"
  e.g. "No error state handling on data screens"

### Files touched estimate
~<N> files to reach requested goal
```

---

## Step 3 — Clarify the scope

After presenting the assessment, ask:

1. **What's the goal?** Choose one:
   - **A — Modernise/clean up**: update deprecated API, improve patterns, no new features
   - **B — Add features**: build on top of what exists
   - **C — Recode a specific area**: rewrite one or more screens/modules from scratch
   - **D — Full recode**: replace everything while keeping the same feature set
   - **E — Mix**: describe the combination

2. **Are there files that must not be touched?** (e.g. a payment flow that's certified, a third-party integration wrapper)

3. **Is there a design or brand to match?** (share screenshots, a Figma link, or describe the style)

4. **Any new capabilities to add?** (e.g. widgets, push, Apple Watch, iPad layout)

If the user already stated the goal clearly before the skill was invoked, skip questions that are already answered.

---

## Step 4 — Change plan

Produce a targeted plan before writing a single line of code:

```
## Change Plan: <App Name>

### Goal
<A / B / C / D / E — one sentence>

### Approach
<keep existing structure and update in place / restructure folders / incremental feature-by-feature>

### Changes

#### Modernisation (do first — low risk)
- [ ] <file or pattern> → <what to change>

#### Architecture changes (do second)
- [ ] <file> — <what and why>

#### New feature work (do last)
- [ ] <feature> — <screens/files to create>

#### Do NOT touch
- <file / module> — <reason>

### Estimated file count
~<N> files modified, ~<M> files created

### Breaking changes or risks
- <risk and mitigation>
```

Wait for explicit confirmation ("looks good", "go ahead", "yes") before coding.

---

## Step 5 — Implement

Work through the plan top-to-bottom. After every logical group of changes:
- State what was changed and why
- Flag anything that diverged from the plan

Apply all the same modern Swift/SwiftUI rules as a new app. See the reference table below.

---

## Modern API rules (apply to all touched files)

Same rules as `create-ios-app`. Key table:

| ❌ Old / wrong | ✅ Modern |
|----------------|-----------|
| `foregroundColor()` | `foregroundStyle()` |
| `cornerRadius()` | `clipShape(.rect(cornerRadius:))` |
| `NavigationView` | `NavigationStack` |
| `tabItem {}` | `Tab` initializer (iOS 18+); keep `tabItem` for iOS 17 |
| `ObservableObject` + `@Published` | `@Observable` class |
| `@StateObject` / `@ObservedObject` | `@State` + `@Observable` |
| `DispatchQueue.main.async` | `await MainActor.run {}` or `@MainActor` |
| `DateFormatter` / `NumberFormatter` | `.formatted()` / `FormatStyle` |
| `Task.sleep(nanoseconds:)` | `Task.sleep(for:)` |
| `UIScreen.main.bounds` | `containerRelativeFrame()` or `GeometryReader` |
| `GeometryReader` | `containerRelativeFrame()` / `visualEffect()` when sufficient |
| `onTapGesture` (for actions) | `Button` |
| `onChange(of:) { v in }` (1-param) | `onChange(of:) { old, new in }` |
| `fontWeight(.bold)` | `bold()` |
| `showsIndicators: false` | `.scrollIndicators(.hidden)` |
| `String(format: "%.2f", x)` | `Text(x, format: .number.precision(...))` |

Additional rules:
- One type per file.
- No computed-property sub-views — break into `View` structs.
- All `@Observable` classes → `@MainActor`.
- Never `DispatchQueue`, `NSLock`, or GCD for new code.
- Avoid `AnyView`; use generics or `@ViewBuilder`.
- Add `#Preview` to every view that was created or substantially rewritten.

---

## Migration recipes

### ObservableObject → @Observable

```swift
// Before
class HomeViewModel: ObservableObject {
    @Published var items: [Item] = []
    @Published var isLoading = false
}

// After — iOS 17+
@MainActor
@Observable
final class HomeViewModel {
    private(set) var items: [Item] = []
    private(set) var isLoading = false
}
```

View site change:
```swift
// Before: @StateObject var vm = HomeViewModel()
// After:  @State private var vm = HomeViewModel()
```

### NavigationView → NavigationStack

```swift
// Before
NavigationView {
    content
        .navigationTitle("Home")
}

// After
NavigationStack {
    content
        .navigationTitle("Home")
}
```

For link-based navigation, replace `NavigationLink(destination:)` with value-based routing:

```swift
// Before
NavigationLink(destination: DetailView(item: item)) { Text(item.title) }

// After
NavigationLink(value: item) { Text(item.title) }
// + .navigationDestination(for: Item.self) { item in DetailView(item: item) }
```

### DispatchQueue → async/await

```swift
// Before
DispatchQueue.main.async {
    self.items = result
}

// After — if the type is @MainActor
self.items = result  // already on main actor

// Or inside a non-isolated context
await MainActor.run { self.items = result }
```

### DateFormatter → FormatStyle

```swift
// Before
let formatter = DateFormatter()
formatter.dateStyle = .medium
formatter.timeStyle = .short
label.text = formatter.string(from: date)

// After
Text(date, format: .dateTime.day().month().year().hour().minute())
// or
Text(date.formatted(date: .abbreviated, time: .shortened))
```

---

## Scope guidance

| User says | Scope it as |
|-----------|-------------|
| "Minor change" / "small fix" | Touch only the files needed; don't modernise unrelated code |
| "Clean it up" / "modernise" | Full deprecated-API sweep + architecture improvements; no new features |
| "Recode the home screen" | Rewrite that feature folder; keep everything else unchanged |
| "Add [feature]" | Assess impact on existing architecture; add with minimal disruption |
| "Full recode" | Treat like a new app built on the same feature spec; preserve brand/assets |

For a full recode: produce a feature inventory first, confirm with the user, then use `create-ios-app` skill patterns to rebuild from scratch.

---

## Supporting references

The following files from the `create-ios-app` skill apply equally here:

| Reference | Path |
|-----------|------|
| Architecture patterns | `../create-ios-app/architecture.md` |
| Design rules (HIG, spacing, colors) | `../create-ios-app/design.md` |
| SwiftUI templates | `../create-ios-app/templates.md` |
| Common API recipes | `../create-ios-app/api-recipes.md` |
| WidgetKit | `../create-ios-app/widgets.md` |
| Interface writing | `../create-ios-app/writing-for-interfaces.md` |
