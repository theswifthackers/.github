# iOS Architecture

Layering, state, DI, concurrency, and anti-patterns.  
Sourced from twostraws/SwiftAgents, alpozcan/iOSAgentSkills.

---

## Recommended stack

```
SwiftUI Views
    ↓ binds to / observes
@Observable ViewModels (@MainActor)
    ↓ call
Services / Actors (networking, SwiftData, system APIs)
    ↓ work with
Models (Codable value types / @Model)
```

---

## Layer responsibilities

| Layer | Owns | Must not |
|-------|------|----------|
| View | Layout, animation, user input wiring | Business logic, networking, direct model mutation |
| ViewModel | Screen state, user intents, mapping service data to UI | Import SwiftUI types; call services on background threads |
| Service | I/O, caching, API clients, system framework wrappers | UI state |
| Model | Data shape and invariants | UI or networking details |

---

## @Observable (the modern pattern)

```swift
// RIGHT — iOS 17+
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
```

All `@Observable` classes must be `@MainActor` unless there is a documented reason not to.

Never use `ObservableObject` / `@Published` / `@StateObject` / `@ObservedObject` / `@EnvironmentObject` for new code.

---

## State ownership rules

| Need | SwiftUI construct |
|------|------------------|
| View-local toggle / text field | `@State var` (value type) |
| Owned screen view model | `@State var viewModel = MyViewModel()` |
| Pass binding down | `@Bindable var viewModel` |
| App-wide settings | `@AppStorage` or `@Environment` injected object |
| Async work on view appearance | `.task { await viewModel.load() }` |

---

## Dependency injection

Use initializer injection for testability:

```swift
@MainActor
@Observable
final class HomeViewModel {
    private let service: ItemServiceProtocol

    init(service: ItemServiceProtocol = ItemService.shared) {
        self.service = service
    }
}
```

For app-wide dependencies, use an `AppDependencies` container passed via `.environment`:

```swift
@main
struct MyApp: App {
    @State private var deps = AppDependencies()

    var body: some Scene {
        WindowGroup { ContentView() }
            .environment(deps)
    }
}

// In a view:
@Environment(AppDependencies.self) private var deps
```

---

## Actor-based concurrency

Use `actor` for shared mutable state accessed from multiple concurrent contexts:

```swift
actor DataStore {
    private var cache: [String: Data] = [:]

    func data(for key: String) -> Data? { cache[key] }
    func store(_ data: Data, for key: String) { cache[key] = data }
}
```

Rules:
- `@MainActor` for all UI-bound state (ViewModels).
- `actor` for shared background state (caches, stores).
- Never use `DispatchQueue`, `NSLock`, or `@unchecked Sendable` for new code.
- Domain models crossing actor boundaries must be value types (`struct`, `enum`) conforming to `Sendable`.
- Async service calls from ViewModels: `await service.fetch()` — never blocking the main thread.

---

## Service layer pattern

```swift
protocol ItemServiceProtocol: Sendable {
    func fetchItems() async throws -> [Item]
}

struct ItemService: ItemServiceProtocol {
    static let shared = ItemService()

    func fetchItems() async throws -> [Item] {
        let (data, response) = try await URLSession.shared.data(from: Endpoint.items.url)
        guard let http = response as? HTTPURLResponse, 200..<300 ~= http.statusCode else {
            throw AppError.badResponse
        }
        return try JSONDecoder().decode([Item].self, from: data)
    }
}
```

Centralize base URL, auth headers, and error mapping in the service — not in the view model.

---

## Error handling

Typed errors for every domain:

```swift
enum AppError: Error, LocalizedError {
    case badResponse
    case notFound(id: String)
    case unauthorized

    var errorDescription: String? {
        switch self {
        case .badResponse: "The server returned an unexpected response."
        case .notFound(let id): "'\(id)' could not be found."
        case .unauthorized: "Sign in again to continue."
        }
    }
}
```

Map errors to user-facing state in the view model, not in the view. Always surface three UI states: loading, empty, error. See `templates.md` for the `AsyncContentView` wrapper.

---

## Navigation — value-based routing

```swift
enum Route: Hashable {
    case detail(Item.ID)
    case settings
}

@Observable
final class AppRouter {
    var path = NavigationPath()

    func push(_ route: Route) { path.append(route) }
    func pop() { if !path.isEmpty { path.removeLast() } }
    func popToRoot() { path = NavigationPath() }
}
```

Never pass closures for navigation up through the view hierarchy. Use an injected router or `navigationDestination(for:)`.

---

## Modularisation

Start with feature folders, not Swift packages. Extract a local Swift package only when:
- Two targets share code (app + widget, app + share extension), or
- A domain is large enough to warrant its own tests and public API surface.

When modularising, enforce strict dependency direction: Features → Core → Models. Features must not import other Features directly.

---

## Anti-patterns

| Pattern | Why it's wrong |
|---------|---------------|
| 2000-line `ContentView` | No separation of concerns; impossible to test |
| Singletons for every service | Hidden dependencies; breaks testability |
| View model importing `SwiftUI` types | Leaks UI coupling into business logic |
| Blocking `@MainActor` with synchronous I/O | Freezes the UI |
| Multiple `ObservableObject` conformances | Use `@Observable`; don't mix paradigms |
| `DispatchQueue.main.async` | Use `await MainActor.run {}` or mark callers `@MainActor` |
| Mixing UIKit view controllers without a clear bridging need | Adds complexity; avoid unless integrating a UIKit-only API |
| `AnyView` everywhere | Breaks type inference and hurts performance; use generics or `@ViewBuilder` |
| Computed property sub-views | Creates large body methods; extract into `View` structs |
| Force unwraps in production code | Use `guard let` or safe optional chaining |
