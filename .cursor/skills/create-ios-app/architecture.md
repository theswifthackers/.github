# iOS Architecture

Use this when the app grows past a single screen or needs clear boundaries.

## Recommended stack

```
SwiftUI Views
    ↓ binds to
@Observable ViewModels (@MainActor)
    ↓ call
Services (networking, SwiftData, system APIs)
    ↓ use
Models (Codable / SwiftData @Model)
```

## Layer responsibilities

| Layer | Owns | Must not |
|-------|------|----------|
| View | Layout, animation, user input wiring | Networking, business rules |
| ViewModel | Screen state, user intents, mapping to UI | Know SwiftUI types beyond basics |
| Service | I/O, caching, API clients | UI state |
| Model | Data shape and invariants | UI or networking details |

## Dependency injection

Prefer initializer injection for testability:

```swift
@MainActor
@Observable
final class HomeViewModel {
    private let itemService: ItemServiceProtocol

    init(itemService: ItemServiceProtocol = ItemService.shared) {
        self.itemService = itemService
    }
}
```

For app-wide dependencies, create a small `AppDependencies` type and pass via `.environment`.

## State patterns

| Need | Tool |
|------|------|
| View-local toggle / field | `@State` |
| Owned screen model | `@State` + `@Observable` class |
| Shared graph from parent | pass property or `@Bindable` |
| App-wide settings | `@AppStorage` or environment object |
| Async load on appear | `.task { await ... }` |

## Error handling

Surface three UI states on data screens:

1. **Loading** — redacted placeholders or `ProgressView`
2. **Empty** — short explanation + optional action
3. **Error** — message + retry

Keep errors user-readable; log technical details separately.

## Modularization

Start modular with folders, not packages. Extract a local Swift package only when:

- Two targets share code (app + widget), or
- A domain is large enough to own its own tests and API surface

## Anti-patterns

- Massive `ContentView` that owns all features
- View models that import and build complex SwiftUI views
- Singletons for everything (OK for one app session service; prefer injection at call sites)
- Mixing UIKit view controllers without a clear bridge need
- Blocking main actor with synchronous disk/network work
