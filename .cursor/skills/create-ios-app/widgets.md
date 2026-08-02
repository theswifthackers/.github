# WidgetKit Reference

Comprehensive WidgetKit patterns covering iOS 17–26 and common LLM mistakes.  
Sourced from n0an/Widgets-Agent-Skill, arjitj2/swiftui-design-principles, alpozcan/iOSAgentSkills.

Only add a Widget extension when the user explicitly requests it.

---

## Decision tree: which surface?

| User needs | Use |
|------------|-----|
| Glanceable data on Home Screen | Home Screen widget (`systemSmall/Medium/Large`) |
| Glanceable data on Lock Screen | Accessory widget (`.accessoryCircular/Rectangular/Inline`) |
| Quick action from Control Center / Action Button | WidgetKit Control |
| Ongoing activity (navigation, workout, delivery) | Live Activity + Dynamic Island |
| Siri / Shortcuts action | App Intent (no widget needed) |

---

## Extension setup

```swift
@main
struct AppWidgets: WidgetBundle {
    var body: some Widget {
        StatusWidget()
        // add more Widget types here
    }
}

struct StatusWidget: Widget {
    let kind = "StatusWidget"

    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: StatusProvider()) { entry in
            StatusWidgetView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
        .configurationDisplayName("Status")
        .description("Shows your current status.")
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}
```

---

## App Group — always use for shared data

```swift
// Both app and widget extension use this
let defaults = UserDefaults(suiteName: "group.com.example.app")!

// WRONG — widget extension is a separate process
let defaults = UserDefaults.standard  // reads empty sandbox
```

Enable the App Groups capability in both the app target and the widget extension target.

---

## Timeline provider

```swift
struct StatusEntry: TimelineEntry {
    let date: Date
    let value: Int
}

struct StatusProvider: TimelineProvider {
    func placeholder(in context: Context) -> StatusEntry {
        StatusEntry(date: .now, value: 0)   // fast, synchronous, no I/O
    }

    func getSnapshot(in context: Context, completion: @escaping (StatusEntry) -> Void) {
        completion(StatusEntry(date: .now, value: loadFromAppGroup()))
    }

    func getTimeline(in context: Context, completion: @escaping (Timeline<StatusEntry>) -> Void) {
        let entry = StatusEntry(date: .now, value: loadFromAppGroup())
        // Refresh at midnight for daily data
        let tomorrow = Calendar.current.startOfDay(
            for: Calendar.current.date(byAdding: .day, value: 1, to: .now)!
        )
        completion(Timeline(entries: [entry], policy: .after(tomorrow)))
    }
}
```

### Reload policies

| Data granularity | Policy |
|------------------|--------|
| Daily (date changes at midnight) | `.after(startOfTomorrow)` |
| Sub-daily live progress | `.after(15-minute interval)` |
| User-triggered only | `.never` + call `WidgetCenter.shared.reloadTimelines(ofKind:)` |
| End of current batch | `.atEnd` |

**Never** poll with `.after(60s)` for static daily data — exhausts the daily reload budget.

### Reload from app

After any mutation that should be visible on the widget:
```swift
WidgetCenter.shared.reloadTimelines(ofKind: "StatusWidget")
// or reload everything:
WidgetCenter.shared.reloadAllTimelines()
```

---

## Widget view families

```swift
struct StatusWidgetView: View {
    @Environment(\.widgetFamily) var family
    let entry: StatusEntry

    var body: some View {
        switch family {
        case .systemSmall:  SmallView(entry: entry)
        case .systemMedium: MediumView(entry: entry)
        default:            SmallView(entry: entry)
        }
    }
}
```

Supported families — declare only what you design for:
```swift
.supportedFamilies([
    .systemSmall, .systemMedium, .systemLarge,
    .accessoryCircular, .accessoryRectangular, .accessoryInline
])
```

Medium and large Home Screen widgets must share the same visual hierarchy (header / progress / footer). Don't reinvent layout per family without a hard size constraint.

Add explicit padding on Home Screen widgets to avoid clipping near rounded edges:
```swift
.padding(.horizontal, 12)
.padding(.vertical, 12)
```

---

## Lock Screen widgets — use Gauge

```swift
// RIGHT — Gauge is purpose-built for lock screen circular
Gauge(value: entry.fraction) {
    Text("")
} currentValueLabel: {
    Text("\(Int(entry.percentage))%")
        .font(.system(size: 12, weight: .medium, design: .monospaced))
}
.gaugeStyle(.accessoryCircular)
.containerBackground(.fill.tertiary, for: .widget)

// WRONG — manual circle drawing
ZStack {
    Circle().stroke(Color.primary.opacity(0.18), lineWidth: 4)
    Circle().trim(from: 0, to: progress).stroke(...)
}
```

Rectangular lock screen → `Gauge` with `.linearCapacity`:
```swift
Gauge(value: fraction) { Text("") }
    .gaugeStyle(.linearCapacity)
    .tint(.primary)
```

---

## Rendering modes and tinting

```swift
@Environment(\.widgetRenderingMode) var renderingMode

var body: some View {
    switch renderingMode {
    case .accented:
        // Tinted Home Screen (iOS 26): two accent groups
        // Use .widgetAccentable() to mark primary elements
        AccentedView(entry: entry)
    case .vibrant:
        // Lock Screen: luminance to alpha
        VibrantView(entry: entry)
    case .fullColor:
        FullColorView(entry: entry)
    @unknown default:
        FullColorView(entry: entry)
    }
}
```

Mark elements that should receive the accent color:
```swift
Image(systemName: "star.fill")
    .widgetAccentable()     // gets accent color in tinted mode
```

**Always design for tinted mode.** A full-color-only widget turns into a white blob on the tinted Home Screen.

---

## Widget background

```swift
// RIGHT
.containerBackground(.fill.tertiary, for: .widget)

// WRONG — hardcoded, breaks tinted Lock Screen and Standby
.containerBackground(.black, for: .widget)
```

For accessory widgets that should be transparent on the Lock Screen, use `.containerBackground(.clear, for: .widget)`.

---

## Interactive widgets (iOS 17+)

Interactivity requires `AppIntent`. Widget views cannot use closures or gesture handlers.

```swift
struct ToggleFavoriteIntent: AppIntent {
    static var title: LocalizedStringResource = "Toggle Favorite"

    @Parameter(title: "Item ID") var itemID: String

    func perform() async throws -> some IntentResult {
        // Mutate the App Group store
        AppGroupStore.shared.toggleFavorite(id: itemID)
        // Always reload after mutation
        WidgetCenter.shared.reloadAllTimelines()
        return .result()
    }
}

// In the widget view:
Button(intent: ToggleFavoriteIntent(itemID: entry.id)) {
    Image(systemName: entry.isFavorite ? "star.fill" : "star")
}
```

**Never** use `onTapGesture` or button closures in widgets — they compile but do nothing.

---

## Dense visualisations — use Canvas

Widget extensions have a ~30 MB memory budget. Dense grids of subviews get killed by `EXC_RESOURCE`.

```swift
// RIGHT — one Canvas draw pass for 365 dots
Canvas { context, size in
    for day in 1...365 {
        let rect = dotRect(for: day, in: size)
        context.fill(Path(ellipseIn: rect), with: .color(colorFor(day)))
    }
}

// WRONG — hundreds of nested subviews
LazyVGrid(columns: columns) {
    ForEach(1...365, id: \.self) { day in
        Circle().fill(colorFor(day))
    }
}
```

---

## Shared data model between app and widget

```swift
// RIGHT — one model used by both ContentView and TimelineProvider
struct YearProgress {
    let fraction: Double
    let dayOfYear: Int
    let totalDays: Int

    static func current() -> YearProgress { /* shared logic */ }
}

// Include time-of-day when UI implies live progress
let dayProgress = elapsedInCurrentDay / totalSecondsInDay
let fraction = (Double(dayOfYear - 1) + dayProgress) / Double(totalDays)

// WRONG — separate duplicate structs
struct AppYearProgress { ... }
struct WidgetYearProgress { ... }  // duplicated math = diverging results
```

---

## Live Activities (iOS 16.2+)

```swift
struct DeliveryAttributes: ActivityAttributes {
    struct ContentState: Codable, Hashable {
        var status: String
        var eta: Date
    }
    var orderId: String
}

// Start
let activity = try Activity.request(
    attributes: DeliveryAttributes(orderId: "123"),
    content: .init(state: .init(status: "Preparing", eta: .now.addingTimeInterval(1800)),
                   staleDate: .now.addingTimeInterval(3600))
)

// Update
await activity.update(.init(state: .init(status: "On the way", eta: .now.addingTimeInterval(600)),
                             staleDate: .now.addingTimeInterval(1200)))

// End
await activity.end(nil, dismissalPolicy: .after(.now.addingTimeInterval(300)))
```

Info.plist: add `NSSupportsLiveActivities = YES`.

---

## Anti-patterns checklist

- [ ] `UserDefaults.standard` in extension → use App Group suite
- [ ] `onTapGesture` in widget → use `Button(intent:)`
- [ ] Intent mutates data but never calls `WidgetCenter.shared.reloadTimelines` → widget stays stale
- [ ] `.after(60s)` reload policy for daily data → exhausts budget
- [ ] Missing `containerBackground` → broken on iOS 17+ Home Screen
- [ ] Accessory widget with opaque background → wrong on Lock Screen
- [ ] Hardcoded background color → wrong in tinted/Standby mode
- [ ] Manual circle drawing for lock screen progress → use `Gauge`
- [ ] Hundreds of subview circles in a grid → use `Canvas`
- [ ] Duplicate date math in app and widget → share one model
- [ ] Designing only for `.fullColor` → looks broken in tinted mode
