# iOS Design Reference

Full HIG and visual design rules. Sourced from wshobson/mobile-ios-design, arjitj2/swiftui-design-principles.

---

## Core philosophy

**Restraint over decoration.** Every pixel must earn its place. Polished apps use fewer colors, fewer font sizes, fewer spacing values — but use them consistently. Native components and semantic system colors create harmony; custom gradients and decorative borders create noise.

**Native first.** Follow Apple HIG. Users already know how iOS works; surprise them with your content, not your navigation patterns.

---

## 1. Spacing — 4/8-pt grid

**Never use arbitrary values.** Every padding and spacing must come from this set:

```
Allowed: 4  8  12  16  20  24  32  40  48
```

Standard assignments:

| Use case | Value |
|----------|-------|
| Outer horizontal content padding | 16–20 |
| Between major sections | 24–32 |
| Within grouped components | 4–12 |
| Card / row internal (horizontal) | 16 |
| Card / row internal (vertical) | 12–16 |

```swift
// RIGHT
.padding(.horizontal, 20)
.padding(.vertical, 12)
HStack(spacing: 8)

// WRONG
.padding(.bottom, 26)     // arbitrary
HStack(spacing: 18)       // arbitrary
```

---

## 2. Typography — hierarchy, not chaos

Use **five or fewer** distinct font sizes. Lighter weights for larger display sizes; regular/medium for smaller UI text.

### Recommended scale (data-focused UI)

| Role | Size | Weight | Notes |
|------|------|--------|-------|
| Hero number | 36–42 | `.light` | Elegant, not heavy |
| Secondary stat | 20–24 | `.light` | Same weight family |
| Body / toggle label | 15 | `.regular` | Standard iOS body |
| Section header | 11 | `.medium` | Uppercase + tracking |
| Caption / subtitle | 11–13 | `.regular` | Secondary info |

### Rules

- Pick **one** font `design` and use it everywhere — app and widgets.
- Use at most **two** `tracking` values, only on uppercase labels (e.g. `1.5` for section headers, `3` for toolbar titles).
- Never mix monospaced in app with rounded in widgets.
- Prefer Dynamic Type semantic styles (`.body`, `.caption`) for body text; custom sizes only for display/hero elements.
- For years and identifiers: `Text(String(year))` or `Text(year, format: .number.grouping(.never))` to prevent locale comma-grouping ("2,026").
- Never use `fontWeight(.bold)` — use `bold()`.
- Don't force specific sizes; use Dynamic Type.

```swift
// RIGHT — 5 sizes, clear system
.font(.system(size: 42, weight: .light, design: .monospaced))   // hero
.font(.system(size: 24, weight: .light, design: .monospaced))   // stat value
.font(.system(size: 15, weight: .regular, design: .monospaced)) // body
.font(.system(size: 11, weight: .medium, design: .monospaced))  // label
.font(.system(size: 11, weight: .regular, design: .monospaced)) // caption
```

---

## 3. Colors — semantic system colors

Never hardcode hex values or build a soup of `Color.white.opacity(0.32)` variants.

```swift
// RIGHT — adapts to light/dark/accessibility
Color(.systemBackground)           // main background
Color(.secondarySystemBackground)  // card/group backgrounds
Color(.separator)                  // dividers
Color.primary                      // primary text and UI
.foregroundStyle(.secondary)       // secondary text
.foregroundStyle(.tertiary)        // labels, captions
.tint(.accentColor)                // interactive elements
```

If you need opacity at all, limit to 2–3 values:
```swift
.opacity(0.15)  // subtle background strokes
.opacity(0.3)   // separator lines
```

### Materials (depth)

```swift
.background(.ultraThinMaterial)   // floating panels, sheets
.background(.regularMaterial)     // cards with blur
```

### Dark Mode

Use semantic colors — they adapt automatically. Never toggle between hardcoded light and dark palettes.

---

## 4. Component sizing — proportional

### Cards and groups

```swift
// RIGHT — native grouped style
VStack(spacing: 0) {
    row1
    Divider().padding(.leading, 16)
    row2
}
.background(Color(.secondarySystemBackground))
.clipShape(.rect(cornerRadius: 10))
```

Rules:
- **Corner radius**: 10 pt for standard cards/groups (matches iOS style). Only go higher for pill or branded surfaces.
- **Dividers**: `Divider().padding(.leading, 16)`. Never build custom divider views.
- **Background**: `Color(.secondarySystemBackground)`. Never custom gradients for standard cards.

### Progress rings

```swift
// RIGHT
.frame(width: 200, height: 200)
Circle().stroke(background, lineWidth: 3)          // background
Circle().trim(from: 0, to: fraction).stroke(fill, lineWidth: 3)  // fill — SAME lineWidth

// WRONG
.frame(width: 260, height: 260)   // too large
Circle().stroke(background, lineWidth: 9)
Circle().trim(...).stroke(fill, lineWidth: 8)  // different — creates misalignment
```

Always use the same `lineWidth` for background and foreground strokes of the same element.

### List rows and toggles

```swift
// RIGHT — native Toggle with label
Toggle(isOn: $value) {
    Text(title).font(.system(size: 15, weight: .regular))
}
.padding(.horizontal, 16)
.padding(.vertical, 12)
.tint(.green)

// WRONG
HStack {
    Text(label).font(.system(size: 18))  // too big
    Spacer()
    Toggle("", isOn: $isOn).labelsHidden()
}
.frame(height: 70)  // too tall, unnecessary
```

---

## 5. Navigation

Always use `NavigationStack`. Never use a bare `ZStack` as a navigation root.

```swift
NavigationStack(path: $path) {
    ContentView()
        .navigationTitle("Title")
        .navigationBarTitleDisplayMode(.inline)
        .navigationDestination(for: Route.self) { route in
            switch route {
            case .detail(let id): DetailView(id: id)
            case .settings: SettingsView()
            }
        }
}
```

Multi-tab apps:
```swift
TabView {
    Tab("Home", systemImage: "house") { HomeView() }
    Tab("Settings", systemImage: "gearshape") { SettingsView() }
}
```
Use the `Tab` API (iOS 18+). For iOS 17 targets, use `tabItem {}`.

---

## 6. Interactive elements

### Mutually exclusive options

Use one selected value, not multiple toggles:

```swift
// RIGHT
enum Cadence: String, CaseIterable { case daily, weekly, monthly }
@State private var cadence: Cadence = .daily

// WRONG — allows contradictory state
Toggle("Daily", isOn: $daily)
Toggle("Weekly", isOn: $weekly)
Toggle("Monthly", isOn: $monthly)
```

### Animated numeric transitions

```swift
Text(value, format: .number)
    .contentTransition(.numericText())
```

### Buttons

- Always use `Button` (not `onTapGesture`) unless you need tap location or count.
- Always include text alongside icon buttons: `Button("Label", systemImage: "plus", action: action)`.

---

## 7. Layout — advanced

### Safe areas

- Respect safe areas; test on small (iPhone SE) and large (iPhone Pro Max) screen widths.
- Use `safeAreaInset(edge: .bottom)` for custom bottom chrome (not `overlay` or `ZStack`).
- If multiple layers need `safeAreaInset`, merge them into a **single** `safeAreaInset` block at the outermost level with conditional content per tab.

### Interactive editors (pan/zoom/crop)

- Present from payload state (`@State var activeCropRequest: CropRequest?`), not a separate `Bool`.
- Use one shared geometry model for preview and export — never duplicate math.
- Budget editor layout top-down: header → canvas → settings → toolbar. Keep sizing in one place.
- Custom headers: do not add `safeAreaInsets.top` if the parent already handles it.

### GeometryReader

Avoid unless truly needed. Prefer `containerRelativeFrame()`, `visualEffect()`, or layout modifiers.

---

## 8. Accessibility

- Support Dynamic Type. Never clamp body text with `minimumScaleFactor` hacks — fix the layout.
- Use `@ScaledMetric(relativeTo:)` for icon/image sizes that should grow with text.
- Use `@Environment(\.accessibilityReduceMotion)` to disable/simplify animations.
- Use `@Environment(\.accessibilityDifferentiateWithoutColor)` for color-only states (add shape/label fallback).
- Enforce VoiceOver order with `.accessibilityElement(children: .combine)` and `.accessibilitySortPriority()` for custom layouts.
- Add `performAccessibilityAudit()` in UI tests for automated WCAG checks.

---

## 9. iPad & multitasking

- Use `.containerRelativeFrame()` and adaptive `LazyVGrid` columns for split-view compatibility.
- Test in slide-over width (~320 pt) and full iPad width.
- Do not hard-code phone-specific widths.
- Use `@Environment(\.horizontalSizeClass)` to adapt layouts at the compact/regular boundary.

---

## 10. Design don'ts (AI slop checklist)

- No purple gradients as a default aesthetic
- No glowing card shadows as decoration
- No emoji as primary navigation icons
- No 6+ different opacity values on the same color
- No arbitrary corner radii (22, 26, 30) on standard cards
- No decorative border strokes on every surface
- No forced dark backgrounds when the system handles it
- No oversized hero typography that crowds content below it
