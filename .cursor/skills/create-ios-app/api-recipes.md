# Common Apple API Recipes

Minimal patterns for frequent iOS capabilities. Add Info.plist usage strings when required.

## SwiftData list

```swift
import SwiftData
import SwiftUI

@Model
final class Note {
    var title: String
    var createdAt: Date

    init(title: String, createdAt: Date = .now) {
        self.title = title
        self.createdAt = createdAt
    }
}

// In App:
// WindowGroup { ContentView() }
//     .modelContainer(for: Note.self)

struct NotesView: View {
    @Environment(\.modelContext) private var context
    @Query(sort: \Note.createdAt, order: .reverse) private var notes: [Note]
    @State private var draft = ""

    var body: some View {
        NavigationStack {
            List {
                ForEach(notes) { note in
                    Text(note.title)
                }
                .onDelete { indexSet in
                    indexSet.map { notes[$0] }.forEach(context.delete)
                }
            }
            .navigationTitle("Notes")
            .toolbar {
                ToolbarItem(placement: .primaryAction) {
                    Button("Add", systemImage: "plus") {
                        context.insert(Note(title: draft.isEmpty ? "New Note" : draft))
                        draft = ""
                    }
                }
            }
        }
    }
}
```

## Photos picker

```swift
import PhotosUI
import SwiftUI

struct AvatarPickerView: View {
    @State private var selection: PhotosPickerItem?
    @State private var image: Image?

    var body: some View {
        VStack {
            if let image {
                image
                    .resizable()
                    .scaledToFill()
                    .frame(width: 120, height: 120)
                    .clipShape(Circle())
            }
            PhotosPicker("Choose Photo", selection: $selection, matching: .images)
        }
        .onChange(of: selection) { _, newValue in
            Task {
                guard let data = try await newValue?.loadTransferable(type: Data.self),
                      let uiImage = UIImage(data: data) else { return }
                image = Image(uiImage: uiImage)
            }
        }
    }
}
```

Requires `NSPhotoLibraryUsageDescription` only for broader library access; limited picker often needs no prompt — still document intent in README.

## Location (when in use)

```swift
import CoreLocation

@MainActor
@Observable
final class LocationProvider: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    private(set) var coordinate: CLLocationCoordinate2D?
    private(set) var authorization = CLAuthorizationStatus.notDetermined

    override init() {
        super.init()
        manager.delegate = self
        manager.desiredAccuracy = kCLLocationAccuracyHundredMeters
    }

    func request() {
        manager.requestWhenInUseAuthorization()
        manager.startUpdatingLocation()
    }

    nonisolated func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        Task { @MainActor in
            self.coordinate = location.coordinate
        }
    }

    nonisolated func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        let status = manager.authorizationStatus
        Task { @MainActor in
            self.authorization = status
        }
    }
}
```

Add `NSLocationWhenInUseUsageDescription`.

## Keychain-friendly secrets

- Never store API tokens in source control or `UserDefaults`.
- Prefer the Keychain (via a small wrapper or Apple’s Generic Password APIs) for session secrets.
- Use Xcode schemes / `.xcconfig` (gitignored) for debug-only base URLs when needed.

## Push notifications (high level)

1. Capability: Push Notifications in Xcode
2. Request authorization with `UNUserNotificationCenter`
3. Register for remote notifications; send device token to backend
4. Physical device required for reliable remote push testing

Do not invent APNs keys or certificates in the repo.

## Widgets

Only add a Widget Extension when the user asks. Share models via a small local package or shared group folder; keep the first version read-only timeline based on existing app data.

## App Store readiness checklist

- [ ] Unique bundle identifier placeholder documented in README
- [ ] App icon slots present (even if placeholder)
- [ ] Launch screen / first frame is not a dead white screen
- [ ] Privacy usage strings match real APIs
- [ ] No hardcoded secrets
- [ ] Dark Mode looks intentional
- [ ] Dynamic Type does not clip primary UI
