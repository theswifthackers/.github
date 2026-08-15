# Common Apple API Recipes

Drop-in patterns for frequent iOS capabilities.

---

## SwiftData (local persistence)

```swift
import SwiftData

@Model
final class Note {
    var title: String
    var body: String
    var createdAt: Date

    init(title: String, body: String = "", createdAt: Date = .now) {
        self.title = title
        self.body = body
        self.createdAt = createdAt
    }
}

// App entry
@main
struct NoteApp: App {
    var body: some Scene {
        WindowGroup { ContentView() }
            .modelContainer(for: Note.self)
    }
}

// View
struct NotesView: View {
    @Environment(\.modelContext) private var context
    @Query(sort: \Note.createdAt, order: .reverse) private var notes: [Note]

    var body: some View {
        List {
            ForEach(notes) { note in Text(note.title) }
                .onDelete { indexSet in indexSet.map { notes[$0] }.forEach(context.delete) }
        }
        .toolbar {
            ToolbarItem(placement: .primaryAction) {
                Button("Add Note", systemImage: "plus") {
                    context.insert(Note(title: "New Note"))
                }
            }
        }
    }
}
```

CloudKit + SwiftData notes:
- Never use `@Attribute(.unique)` when CloudKit sync is enabled.
- All model properties must have default values or be optional.
- All relationships must be optional.

---

## Photos picker

```swift
import PhotosUI
import SwiftUI

struct AvatarPickerView: View {
    @State private var selection: PhotosPickerItem?
    @State private var image: Image?

    var body: some View {
        VStack(spacing: 16) {
            if let image {
                image
                    .resizable()
                    .scaledToFill()
                    .frame(width: 120, height: 120)
                    .clipShape(Circle())
            } else {
                Circle()
                    .fill(Color(.secondarySystemBackground))
                    .frame(width: 120, height: 120)
                    .overlay { Image(systemName: "person.fill").foregroundStyle(.tertiary) }
            }
            PhotosPicker("Choose Photo", selection: $selection, matching: .images)
        }
        .onChange(of: selection) { _, newValue in
            Task {
                guard let data = try? await newValue?.loadTransferable(type: Data.self),
                      let uiImage = UIImage(data: data) else { return }
                image = Image(uiImage: uiImage)
            }
        }
    }
}
```

---

## Location (when in use)

Info.plist: `NSLocationWhenInUseUsageDescription`

```swift
import CoreLocation

@MainActor
@Observable
final class LocationProvider: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    private(set) var coordinate: CLLocationCoordinate2D?
    private(set) var authorizationStatus = CLAuthorizationStatus.notDetermined

    override init() {
        super.init()
        manager.delegate = self
        manager.desiredAccuracy = kCLLocationAccuracyHundredMeters
    }

    func requestPermission() {
        manager.requestWhenInUseAuthorization()
    }

    func start() { manager.startUpdatingLocation() }
    func stop() { manager.stopUpdatingLocation() }

    nonisolated func locationManager(_ manager: CLLocationManager,
                                     didUpdateLocations locations: [CLLocation]) {
        guard let loc = locations.last else { return }
        Task { @MainActor in self.coordinate = loc.coordinate }
    }

    nonisolated func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        let status = manager.authorizationStatus
        Task { @MainActor in self.authorizationStatus = status }
    }
}
```

---

## Push notifications (local scheduling)

```swift
import UserNotifications

func scheduleNotification(title: String, body: String, seconds: TimeInterval) async throws {
    let center = UNUserNotificationCenter.current()
    let granted = try await center.requestAuthorization(options: [.alert, .sound, .badge])
    guard granted else { return }

    let content = UNMutableNotificationContent()
    content.title = title
    content.body = body
    content.sound = .default

    let trigger = UNTimeIntervalNotificationTrigger(timeInterval: seconds, repeats: false)
    let request = UNNotificationRequest(identifier: UUID().uuidString,
                                        content: content, trigger: trigger)
    try await center.add(request)
}
```

Remote push (high-level):
1. Add Push Notifications capability.
2. Call `UIApplication.shared.registerForRemoteNotifications()`.
3. Receive device token in `AppDelegate.application(_:didRegisterForRemoteNotificationsWithDeviceToken:)`.
4. Send token to your backend.
5. Physical device required — simulator has limited push support.

Do not include APNs auth keys or certificates in the repository.

---

## Keychain storage (secrets)

Never store tokens in `UserDefaults`. Use Keychain via a simple wrapper:

```swift
import Security

enum KeychainError: Error { case saveFailed, loadFailed, deleteFailed }

struct Keychain {
    static func save(_ value: String, for key: String) throws {
        let data = Data(value.utf8)
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecValueData as String: data
        ]
        SecItemDelete(query as CFDictionary)
        guard SecItemAdd(query as CFDictionary, nil) == errSecSuccess else {
            throw KeychainError.saveFailed
        }
    }

    static func load(for key: String) throws -> String {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        var result: AnyObject?
        guard SecItemCopyMatching(query as CFDictionary, &result) == errSecSuccess,
              let data = result as? Data,
              let string = String(data: data, encoding: .utf8) else {
            throw KeychainError.loadFailed
        }
        return string
    }

    static func delete(for key: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key
        ]
        guard SecItemDelete(query as CFDictionary) == errSecSuccess else {
            throw KeychainError.deleteFailed
        }
    }
}
```

API keys for debug builds: use `.xcconfig` files (gitignored), not hardcoded constants.

---

## Date & number formatting (modern API only)

```swift
// Dates — never use DateFormatter
Date.now.formatted(date: .abbreviated, time: .shortened)        // "Aug 2, 2026 at 9:00 AM"
Date.now.formatted(.relative(presentation: .named))             // "2 hours ago"
Date(someString, strategy: .iso8601)                            // parse

// Numbers — never use NumberFormatter
let price = 12.99
Text(price, format: .currency(code: "USD"))                     // "$12.99"
Text(1_234_567, format: .number)                                // "1,234,567"
Text(0.752, format: .percent.precision(.fractionLength(1)))     // "75.2%"
Text(count, format: .number)                                    // never String(format: "%d", count)
```

---

## Haptic feedback

```swift
import UIKit

enum HapticFeedback {
    static func impact(_ style: UIImpactFeedbackGenerator.FeedbackStyle) {
        UIImpactFeedbackGenerator(style: style).impactOccurred()
    }

    static func notification(_ type: UINotificationFeedbackGenerator.FeedbackType) {
        UINotificationFeedbackGenerator().notificationOccurred(type)
    }

    static func selection() {
        UISelectionFeedbackGenerator().selectionChanged()
    }
}

// Usage
HapticFeedback.impact(.medium)
HapticFeedback.notification(.success)
```

---

## App Store readiness checklist

Before shipping (quality bar). For a full guideline review with citations and revisions, use the `review-ios-app` skill.

- [ ] Unique bundle identifier (not `com.example.*`)
- [ ] App icon — all required slots filled (no missing sizes)
- [ ] Launch screen is not blank white on first frame
- [ ] Privacy usage description strings match APIs actually used
- [ ] Privacy policy linked in-app; `PrivacyInfo.xcprivacy` present if required-reason APIs are used
- [ ] Account creation includes in-app account deletion; third-party social login includes Sign in with Apple when 4.8 applies
- [ ] No hardcoded API keys, tokens, or certificates
- [ ] Dark Mode renders intentionally (not broken)
- [ ] Dynamic Type does not clip primary content at accessibility sizes
- [ ] VoiceOver traversal order makes sense for key screens
- [ ] All Info.plist capability keys have human-readable purpose strings
- [ ] `NSAppTransportSecurity` exceptions only for domains that require them
- [ ] No calls to private/undocumented Apple APIs
- [ ] Crash-free on oldest supported device / iOS version
- [ ] Demo account or demo mode ready if anything is behind login
