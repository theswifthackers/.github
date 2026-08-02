# SwiftUI Starter Templates

Production-ready SwiftUI patterns. Copy and adapt; rename types to match the feature.

---

## App entry

```swift
import SwiftUI
import SwiftData

@main
struct AppNameApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: [Item.self])  // remove if not using SwiftData
    }
}
```

---

## Tab root (iOS 18+ Tab API; fallback for 17 below)

```swift
// iOS 18+
struct ContentView: View {
    var body: some View {
        TabView {
            Tab("Home", systemImage: "house") { HomeView() }
            Tab("Explore", systemImage: "magnifyingglass") { ExploreView() }
            Tab("Settings", systemImage: "gearshape") { SettingsView() }
        }
    }
}

// iOS 17 fallback
struct ContentView: View {
    var body: some View {
        TabView {
            HomeView()
                .tabItem { Label("Home", systemImage: "house") }
            SettingsView()
                .tabItem { Label("Settings", systemImage: "gearshape") }
        }
    }
}

#Preview { ContentView() }
```

---

## Navigation stack with value-based routing

```swift
enum Route: Hashable {
    case detail(Item.ID)
    case settings
}

struct HomeView: View {
    @State private var path = NavigationPath()
    @State private var viewModel = HomeViewModel()

    var body: some View {
        NavigationStack(path: $path) {
            List(viewModel.items) { item in
                NavigationLink(value: Route.detail(item.id)) {
                    Text(item.title)
                }
            }
            .navigationTitle("Home")
            .navigationBarTitleDisplayMode(.large)
            .navigationDestination(for: Route.self) { route in
                switch route {
                case .detail(let id): DetailView(itemID: id)
                case .settings: SettingsView()
                }
            }
            .toolbar {
                ToolbarItem(placement: .primaryAction) {
                    Button("Settings", systemImage: "gearshape") {
                        path.append(Route.settings)
                    }
                }
            }
            .task { await viewModel.load() }
        }
    }
}

#Preview { HomeView() }
```

---

## Loading / empty / error wrapper

```swift
struct AsyncContentView<Content: View>: View {
    let isLoading: Bool
    let errorMessage: String?
    let isEmpty: Bool
    let retry: () async -> Void
    @ViewBuilder let content: () -> Content

    var body: some View {
        Group {
            if isLoading && isEmpty {
                ProgressView("Loading…")
                    .frame(maxWidth: .infinity, maxHeight: .infinity)
            } else if let message = errorMessage, isEmpty {
                ContentUnavailableView {
                    Label("Something went wrong", systemImage: "exclamationmark.triangle")
                } description: {
                    Text(message)
                } actions: {
                    Button("Try Again") { Task { await retry() } }
                }
            } else if isEmpty {
                ContentUnavailableView(
                    "No Items",
                    systemImage: "tray",
                    description: Text("Pull to refresh or try again later.")
                )
            } else {
                content()
            }
        }
    }
}

// Usage
AsyncContentView(
    isLoading: viewModel.isLoading,
    errorMessage: viewModel.errorMessage,
    isEmpty: viewModel.items.isEmpty,
    retry: viewModel.load
) {
    List(viewModel.items) { item in Text(item.title) }
}
```

---

## Detail view

```swift
struct DetailView: View {
    let itemID: Item.ID
    @State private var viewModel: DetailViewModel

    init(itemID: Item.ID) {
        self.itemID = itemID
        _viewModel = State(initialValue: DetailViewModel(itemID: itemID))
    }

    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 16) {
                if let item = viewModel.item {
                    Text(item.title)
                        .font(.title.bold())
                    Text(item.body)
                        .foregroundStyle(.secondary)
                }
            }
            .frame(maxWidth: .infinity, alignment: .leading)
            .padding(.horizontal, 20)
            .padding(.vertical, 16)
        }
        .navigationTitle(viewModel.item?.title ?? "")
        .navigationBarTitleDisplayMode(.inline)
        .task { await viewModel.load() }
    }
}

#Preview {
    NavigationStack {
        DetailView(itemID: Item.preview.id)
    }
}
```

---

## Form screen

```swift
struct ProfileFormView: View {
    @State private var name = ""
    @State private var email = ""
    @State private var notificationsEnabled = true

    var body: some View {
        NavigationStack {
            Form {
                Section("Profile") {
                    TextField("Name", text: $name)
                        .textContentType(.name)
                    TextField("Email", text: $email)
                        .textContentType(.emailAddress)
                        .keyboardType(.emailAddress)
                        .textInputAutocapitalization(.never)
                }
                Section("Preferences") {
                    Toggle("Notifications", isOn: $notificationsEnabled)
                }
                Section {
                    Button("Save Changes", action: save)
                        .disabled(name.isEmpty || email.isEmpty)
                }
            }
            .navigationTitle("Edit Profile")
        }
    }

    private func save() {
        // persist
    }
}

#Preview { ProfileFormView() }
```

---

## Observable view model (full pattern)

```swift
import Foundation

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

    var isEmpty: Bool { !isLoading && items.isEmpty }

    func load() async {
        isLoading = true
        errorMessage = nil
        defer { isLoading = false }
        do {
            items = try await service.fetchItems()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}
```

---

## Networking service

```swift
import Foundation

protocol ItemServiceProtocol: Sendable {
    func fetchItems() async throws -> [Item]
}

struct ItemService: ItemServiceProtocol {
    static let shared = ItemService()

    private let baseURL = URL(string: "https://api.example.com")!

    func fetchItems() async throws -> [Item] {
        let url = baseURL.appending(path: "items")
        let (data, response) = try await URLSession.shared.data(from: url)
        guard let http = response as? HTTPURLResponse, 200..<300 ~= http.statusCode else {
            throw URLError(.badServerResponse)
        }
        return try JSONDecoder().decode([Item].self, from: data)
    }
}
```

---

## Confirmation / destructive action

```swift
struct ItemRowView: View {
    let item: Item
    let onDelete: () -> Void

    @State private var showDeleteConfirmation = false

    var body: some View {
        HStack {
            Text(item.title)
            Spacer()
            Button("Delete", systemImage: "trash", role: .destructive) {
                showDeleteConfirmation = true
            }
            .labelStyle(.iconOnly)
        }
        .confirmationDialog(
            "Delete '\(item.title)'?",
            isPresented: $showDeleteConfirmation,
            titleVisibility: .visible
        ) {
            Button("Delete", role: .destructive, action: onDelete)
            Button("Cancel", role: .cancel) { }
        } message: {
            Text("This action can't be undone.")
        }
    }
}
```

---

## Pill / capsule filter bar

```swift
struct FilterBar<T: Hashable & CustomStringConvertible>: View {
    let options: [T]
    @Binding var selection: T

    var body: some View {
        ScrollView(.horizontal, showsIndicators: false) {
            HStack(spacing: 8) {
                ForEach(options, id: \.hashValue) { option in
                    Button(option.description) {
                        selection = option
                    }
                    .buttonStyle(.bordered)
                    .tint(selection == option ? .accentColor : .secondary)
                }
            }
            .padding(.horizontal, 16)
            .padding(.vertical, 8)
        }
    }
}
```

---

## README template

Every new app ships with this file:

```markdown
# AppName

Brief one-line description.

## Requirements
- Xcode 15+
- iOS 17+

## Getting started
1. Clone the repo
2. Open `AppName.xcodeproj` in Xcode
3. Select your team in Signing & Capabilities
4. Replace `com.example.appname` with your Bundle ID
5. Run on simulator or device

## Features
- Feature A
- Feature B

## Configuration
- `TODO: Add your API base URL in Core/Services/Endpoint.swift`
- `TODO: Add push notification entitlements if needed`

## Permissions used
| Permission | Why |
|------------|-----|
| Camera | ... |
```
