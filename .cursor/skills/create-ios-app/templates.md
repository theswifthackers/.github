# SwiftUI Starter Templates

Copy and adapt these when scaffolding screens. Rename types to match the feature.

## App entry

```swift
import SwiftUI

@main
struct AppNameApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

## Tab root

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        TabView {
            Tab("Home", systemImage: "house") {
                HomeView()
            }
            Tab("Settings", systemImage: "gearshape") {
                SettingsView()
            }
        }
    }
}

#Preview {
    ContentView()
}
```

## List → detail

```swift
import SwiftUI

struct Item: Identifiable, Hashable {
    let id: UUID
    let title: String
    let detail: String
}

struct ItemListView: View {
    let items: [Item]
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            List(items) { item in
                NavigationLink(value: item) {
                    Text(item.title)
                }
            }
            .navigationTitle("Items")
            .navigationDestination(for: Item.self) { item in
                ItemDetailView(item: item)
            }
        }
    }
}

struct ItemDetailView: View {
    let item: Item

    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 12) {
                Text(item.title)
                    .font(.largeTitle.bold())
                Text(item.detail)
                    .foregroundStyle(.secondary)
            }
            .frame(maxWidth: .infinity, alignment: .leading)
            .padding()
        }
        .navigationTitle("Detail")
        .navigationBarTitleDisplayMode(.inline)
    }
}
```

## Form screen

```swift
import SwiftUI

struct ProfileFormView: View {
    @State private var name = ""
    @State private var email = ""
    @State private var notify = true

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
                    Toggle("Notifications", isOn: $notify)
                }
            }
            .navigationTitle("Edit Profile")
            .toolbar {
                ToolbarItem(placement: .confirmationAction) {
                    Button("Save") {
                        // persist
                    }
                    .disabled(name.isEmpty || email.isEmpty)
                }
            }
        }
    }
}
```

## Loading / empty / error wrapper

```swift
import SwiftUI

struct AsyncContentView<Content: View>: View {
    let isLoading: Bool
    let errorMessage: String?
    let isEmpty: Bool
    let retry: () async -> Void
    @ViewBuilder let content: () -> Content

    var body: some View {
        Group {
            if isLoading && isEmpty {
                ProgressView("Loading")
            } else if let errorMessage, isEmpty {
                ContentUnavailableView {
                    Label("Something went wrong", systemImage: "exclamationmark.triangle")
                } description: {
                    Text(errorMessage)
                } actions: {
                    Button("Try Again") {
                        Task { await retry() }
                    }
                }
            } else if isEmpty {
                ContentUnavailableView(
                    "No Data",
                    systemImage: "tray",
                    description: Text("Pull to refresh or try again later.")
                )
            } else {
                content()
            }
        }
    }
}
```

## Observable view model + service

```swift
import Foundation

protocol ItemServiceProtocol: Sendable {
    func fetchItems() async throws -> [Item]
}

struct ItemService: ItemServiceProtocol {
    static let shared = ItemService()

    func fetchItems() async throws -> [Item] {
        let url = URL(string: "https://example.com/api/items")!
        let (data, response) = try await URLSession.shared.data(from: url)
        guard let http = response as? HTTPURLResponse, 200..<300 ~= http.statusCode else {
            throw URLError(.badServerResponse)
        }
        return try JSONDecoder().decode([Item].self, from: data)
    }
}

@MainActor
@Observable
final class ItemListViewModel {
    private(set) var items: [Item] = []
    private(set) var isLoading = false
    private(set) var errorMessage: String?
    private let service: ItemServiceProtocol

    init(service: ItemServiceProtocol = ItemService.shared) {
        self.service = service
    }

    var isEmpty: Bool { items.isEmpty }

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
