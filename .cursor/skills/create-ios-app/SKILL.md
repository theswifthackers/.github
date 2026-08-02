---
name: create-ios-app
description: Create production-ready iOS apps with Swift and SwiftUI using MVVM, modern Apple APIs, and Human Interface Guidelines. Use when scaffolding a new iOS app, adding SwiftUI screens, setting up Xcode project structure, or when the user mentions iOS, SwiftUI, UIKit, Xcode, or Apple app development.
---

# Create iOS App

Build iOS apps with **Swift + SwiftUI** by default. Prefer UIKit only when the user asks or when a required API has no SwiftUI equivalent.

## Quick start checklist

```
Task Progress:
- [ ] Clarify app purpose, platforms (iPhone / iPad / Mac), and min iOS version
- [ ] Scaffold project structure (App → Features → Core → Resources)
- [ ] Define data models and navigation
- [ ] Implement screens with SwiftUI + MVVM
- [ ] Wire networking / persistence / permissions as needed
- [ ] Add previews, basic tests, and accessibility
- [ ] Verify build settings, Info.plist keys, and assets
```

## Defaults (unless user overrides)

| Choice | Default |
|--------|---------|
| UI | SwiftUI |
| Architecture | MVVM + unidirectional data flow |
| Min deployment | iOS 17+ |
| Concurrency | `async` / `await` + `@MainActor` for UI state |
| Persistence | SwiftData for local; URLSession for remote |
| Navigation | `NavigationStack` + value-based routing |
| Package manager | Swift Package Manager |
| Previews | `#Preview` on every screen |

## Discovery questions

Ask only what blocks scaffolding. Infer the rest:

1. What does the app do in one sentence?
2. iPhone only, or iPhone + iPad?
3. Needs login, networking, maps, camera, or offline storage?
4. Any existing design / brand constraints?

If answers are missing, scaffold a clean SwiftUI starter with a home screen, settings stub, and clear TODO markers.

## Project structure

```
AppName/
├── AppNameApp.swift          # @main entry
├── ContentView.swift         # Root shell / tab host
├── Features/
│   ├── Home/
│   │   ├── HomeView.swift
│   │   └── HomeViewModel.swift
│   └── Settings/
│       └── SettingsView.swift
├── Core/
│   ├── Models/
│   ├── Services/             # Networking, persistence, system APIs
│   ├── Components/           # Reusable UI pieces
│   └── Theme/                # Colors, typography, spacing
├── Resources/
│   ├── Assets.xcassets
│   └── Localizable.xcstrings
└── Info.plist                # Only when custom keys are required
```

Keep feature folders vertical: view + view model + feature-specific models stay together.

## Implementation rules

### SwiftUI views

- Views are declarative and side-effect free; put logic in the view model or services.
- Prefer small composed views over one giant body.
- Use `@State` for view-local UI; `@StateObject` / `@Observable` for owned view models; `@Environment` for shared dependencies.
- Prefer `@Observable` (Observation framework) over `ObservableObject` for new code on iOS 17+.
- Always provide `#Preview` with sample data.

```swift
import SwiftUI

struct HomeView: View {
    @State private var viewModel = HomeViewModel()

    var body: some View {
        NavigationStack {
            List(viewModel.items) { item in
                Text(item.title)
            }
            .navigationTitle("Home")
            .task { await viewModel.load() }
        }
    }
}

#Preview {
    HomeView()
}
```

### View models

- Mark UI-facing view models `@MainActor`.
- Expose ready-to-render state, not raw networking types.
- Handle loading / empty / error states explicitly.

```swift
import Foundation

@MainActor
@Observable
final class HomeViewModel {
    private(set) var items: [Item] = []
    private(set) var isLoading = false
    private(set) var errorMessage: String?

    func load() async {
        isLoading = true
        errorMessage = nil
        defer { isLoading = false }
        do {
            items = try await ItemService.shared.fetchItems()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}
```

### Networking

- Use `URLSession` + `async/await`.
- Decode with `Codable`.
- Centralize base URL, auth headers, and error mapping in a service layer.
- Never block the main thread.

### Persistence

- **SwiftData** for local models and queries.
- **AppStorage** / `UserDefaults` for lightweight preferences only.
- Do not invent a Core Data stack unless the user asks or SwiftData cannot cover the need.

### Navigation

```swift
enum Route: Hashable {
    case detail(Item.ID)
    case settings
}

NavigationStack(path: $path) {
    HomeView()
        .navigationDestination(for: Route.self) { route in
            switch route {
            case .detail(let id): DetailView(itemID: id)
            case .settings: SettingsView()
            }
        }
}
```

### Permissions & Info.plist

Add usage descriptions only for APIs you actually call:

| Capability | Info.plist key |
|------------|----------------|
| Camera | `NSCameraUsageDescription` |
| Photo library | `NSPhotoLibraryUsageDescription` |
| Location when in use | `NSLocationWhenInUseUsageDescription` |
| Microphone | `NSMicrophoneUsageDescription` |
| Face ID | `NSFaceIDUsageDescription` |

Write clear, human purpose strings — App Review rejects vague copy.

### Accessibility & polish

- Use `Label` / SF Symbols where possible.
- Support Dynamic Type; avoid fixed font sizes for body text.
- Add `.accessibilityLabel` when icons lack text.
- Respect safe areas; test on a small phone width (e.g. iPhone SE) and a large phone.
- Prefer system colors / semantic colors unless brand tokens are provided.

### Testing

- Unit-test view models and services.
- Use SwiftUI previews as the first visual check.
- Add a UI test only for critical flows (launch → primary action).

## Design direction

When building UI from scratch (no existing design system):

- Follow Apple HIG: clarity, deference, depth.
- Use SF Pro via system fonts; pair with SF Symbols.
- Prefer native controls (`List`, `Form`, `NavigationStack`, `TabView`) over custom chrome.
- One primary action per screen.
- Avoid generic “AI slop” aesthetics (purple gradients, glowing cards, emoji-as-icons).

For brand / marketing surfaces inside the app, still keep native iOS patterns for interactive flows (forms, settings, lists).

## Xcode / tooling notes

- Target a single app product first; add widgets, App Clips, or watch later.
- Use SPM for dependencies; pin versions intentionally.
- Prefer Xcode 15+ project format and folder-synced groups when available.
- Simulator is fine for most UI work; note when device-only APIs (push, camera, NFC) need a physical device.
- This environment may lack macOS/Xcode — still produce complete Swift sources and project layout the user can open on a Mac.

## Deliverables for a new app

When asked to create an app, ship:

1. Folder structure above with compiling Swift sources
2. `README.md` with open-in-Xcode steps, min iOS version, and features
3. Asset placeholders (AppIcon set structure) when relevant
4. Clear TODOs for signing, bundle ID, and team ID (user-specific)

Do **not** invent fake certificates, provisioning profiles, or API keys.

## Additional resources

- Architecture patterns and layering: [architecture.md](architecture.md)
- Starter screen templates: [templates.md](templates.md)
- Common Apple API recipes: [api-recipes.md](api-recipes.md)
