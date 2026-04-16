# HeirloomKit Components Reference

## Table of Contents

- [GlassFrame](#glassframe)
- [HeirloomButtonStyle](#heirloombuttonstyle)
- [GlassSection](#glasssection)
- [DividedVStack](#dividedvstack)
- [CapsuleBar](#capsulebar)
- [ContentFade](#contentfade)
- [FloatingToolbar](#floatingtoolbar)
- [NoiseOverlay](#noiseoverlay)
- [PointingHandCursor](#pointinghandcursor)
- [SelectionPopover](#selectionpopover)
- [LocationSearchPopover](#locationsearchpopover)
- [LocationResult](#locationresult)
- [KeyboardSearchField](#keyboardsearchfield)

---

## GlassFrame

Clips content to a shape, fills with material or a solid color, renders an optional etched border, and applies a depth effect.

```swift
.glassFrame(
    shape: GlassBackgroundShape = .roundedRectangle(),
    fill: GlassFrameFill = .glass,
    etched: Bool = true,
    contentInset: Bool = true,
    depth: GlassDepthStyle = .shadow(.soft)
) -> some View
```

### GlassBackgroundShape

```swift
enum GlassBackgroundShape: Sendable {
    case roundedRectangle(cornerRadius: CGFloat = .relaxed)  // default 12pt
    case disc       // circle
    case capsule    // stadium
    case rectangle  // sharp corners
}
```

Static constant: `GlassBackgroundShape.contentInsetAmount = 1.25` pt

### GlassFrameFill

```swift
enum GlassFrameFill: Sendable {
    case glass              // liquid glass with adaptive auto-tint
    case tinted(Color)      // liquid glass with custom tint
    case solid(Color)       // flat color, no material
}
```

The liquid glass material can glitch visually on very large views (e.g., full-page document layouts). Use `.solid(color)` as a drop-in replacement — it preserves shape clipping, etched borders, and depth effects without the material.

### GlassDepthStyle

```swift
enum GlassDepthStyle: Sendable {
    case none
    case shadow(_ intensity: ShadowIntensity)
    case glow(color: Color, radius: CGFloat = .relaxed)  // default 12pt
}

enum ShadowIntensity: Sendable {
    case hard    // radius: 2, y: 2
    case medium  // radius: 4, y: 3
    case soft    // radius: 8, y: 4
}
```

### Parameters

- `shape` — Clipping shape. Default rounded rectangle with 12pt corners.
- `fill` — Background fill style. Default `.glass`. Use `.tinted(color)` for custom glass tints, `.solid(color)` for large views where the glass material glitches.
- `etched` — Thin border for edge visibility on iOS 26+. Default true. Set false when floating over a contrasting surface.
- `contentInset` — Clips content 1.25pt inside the glass edge. Default true. Set false for edge-to-edge content like images.
- `depth` — Shadow or glow behind the frame. Default `.shadow(.soft)`.

### Platform behavior

- **`.glass` / `.tinted`:** iOS 26+ / macOS 26+ uses `.glassEffect(.regular)` with optional etched border. Earlier falls back to `.ultraThinMaterial` with stroke border.
- **`.solid`:** All platforms use a simple color fill. No material, no platform-specific behavior.

---

## HeirloomButtonStyle

ButtonStyle wrapping labels in a glass frame with cursor, press feedback, and hover glow.

```swift
// Default (static property)
.buttonStyle(.heirloom)

// Customized (static method)
.buttonStyle(.heirloom(
    shape: GlassBackgroundShape = .capsule,
    fill: GlassFrameFill = .glass,
    padding: HeirloomButtonPadding = .spaciousRelaxed,
    contentShadow: Bool = true
))
```

### HeirloomButtonPadding

```swift
enum HeirloomButtonPadding: Sendable {
    case zero                          // 0 all edges
    case standard                      // 8 x 8
    case relaxedStandard               // 12h x 8v
    case relaxed                       // 12 x 12
    case spaciousRelaxed               // 16h x 12v (default)
    case custom(top:leading:bottom:trailing:)
}
```

### Auto-Contrast for Tinted and Solid Fills

When `fill` is `.tinted(color)` or `.solid(color)`, the button automatically selects foreground text color and content shadow based on the fill color's luminance:

- **Dark fill** (luminance < 0.5) → white text, dark shadow (black 0.25)
- **Light fill** (luminance ≥ 0.5) → `.textPrimary` text, light shadow (white 0.45)

For `.glass` fills, no automatic foreground is applied — the caller controls the text color, and the content shadow follows `colorScheme` as usual.

### Feedback

- Press: scale 0.97, opacity 0.85 (0.15s snappy)
- Hover: brightness +0.05
- Content shadow: follows auto-contrast for tinted/solid fills; dark mode black 0.25 / light mode white 0.45 for glass
- Includes pointing-hand cursor automatically

---

## GlassSection

Titled section container over a glass panel. Replaces `Form`/`Section` for full control over margins and styling. Dividers between children inserted automatically via DividedVStack.

```swift
GlassSection(
    _ title: String? = nil,
    spacing: CGFloat = .standard,
    fill: GlassFrameFill = .glass,
    etched: Bool = true,
    @ViewBuilder content: @escaping () -> Content
)
```

### Parameters

- `title` — Optional uppercase header rendered inside the glass panel with `.heirloomSans(13, .medium)`, 1pt letter spacing, background fill at textPrimary 0.04 with 0.25 noise.
- `spacing` — Vertical spacing between content items. Default `.standard` (8pt).
- `fill` — Background fill style forwarded to the internal `.glassFrame()`. Default `.glass`.
- `etched` — Glass edge border. Default true.

When titled: relaxed padding + content inset. When untitled: content only, no inset.

---

## DividedVStack

VStack that inserts Divider views between children. Used internally by GlassSection.

```swift
DividedVStack(
    spacing: CGFloat = .standard,
    @ViewBuilder content: @escaping () -> Content
)
```

Dividers have `.standard / 4` (2pt) vertical spacing.

---

## CapsuleBar

Segmented capsule bar with sliding glass indicator and elastic bounce.

```swift
CapsuleBar<Tab: Hashable & CaseIterable & RawRepresentable>(
    currentTab: Tab,
    navigableTabs: Set<Tab>,
    capsulePadding: CGFloat = 2,
    onTabSelected: @escaping (Tab) -> Void
)
// where Tab.RawValue == String
```

### Parameters

- `currentTab` — Currently selected tab.
- `navigableTabs` — Tabs the user can select. Others render in `.textTertiary` and ignore taps.
- `capsulePadding` — Inset of the sliding indicator from the bar edge. Default 2pt.
- `onTabSelected` — Called when a navigable tab is tapped.

### Animation

Spring: response 0.35, dampingFraction 0.6. Elastic bounce at edges via ClampedOffset (elasticity 0.6).

### ClampedOffset

Public ViewModifier for elastic-clamped offsets:

```swift
ClampedOffset(offset: CGFloat, minX: CGFloat, maxX: CGFloat)

// Static helper
ClampedOffset.elasticClamp(_ value: CGFloat, minX: CGFloat, maxX: CGFloat, elasticity: CGFloat = 0.5) -> CGFloat
```

---

## ContentFade

Masks scrolling content to transparent near the top edge. Optionally pins a fixed header view on top.

```swift
// Fade only
.contentFade(height: CGFloat = 48) -> some View

// With header
.contentFade<Header: View>(
    height: CGFloat = 48,
    @ViewBuilder header: () -> Header
) -> some View
```

### How it works

- Masks content with a gradient (clear at top, opaque at bottom of fade region)
- No `.ignoresSafeArea()` on the mask — content behind the toolbar is naturally masked out
- Header placed via `.overlay(alignment: .top)` (not masked)
- Scroll content offset via `.contentMargins(.top, headerHeight, for: .scrollContent)`
- Scroll indicators offset via `.contentMargins(.top, fadeHeight, for: .scrollIndicators)`
- Header height measured with `onGeometryChange`

### Placement

Apply `.contentFade` directly to the `ScrollView` that needs fading, not to a parent container. The mask covers the entire view — attaching it to a `NavigationSplitView` or outer `VStack` will fade non-scrolling content (sidebars, fixed panels) as well.

The header offset uses `.contentMargins(.top:for: .scrollContent)`, which only affects `ScrollView`. On non-scroll views, the header overlay sits on top of content with no offset.

```swift
// Correct — applied to the scrolling detail column
NavigationSplitView {
    Sidebar()
} detail: {
    ScrollView { DetailContent() }
        .contentFade {
            CapsuleBar(currentTab: tab, ...)
                .padding(.horizontal, .spacious)
                .padding(.vertical, .standard)
        }
}

// Wrong — fades the sidebar and breaks header offset
NavigationSplitView {
    Sidebar()
} detail: {
    DetailContent()
}
.contentFade { CapsuleBar(...) }
```

### Modifier ordering

Must come BEFORE `.inspector()`. See Gotchas in SKILL.md.

---

## FloatingToolbar

Hides the toolbar background for floating title and traffic lights on macOS.

```swift
.floatingToolbar() -> some View
```

- macOS: `.toolbarBackground(.hidden, for: .windowToolbar)`
- Other platforms: no-op

Safe to use unconditionally in cross-platform code.

---

## NoiseOverlay

GPU-generated static noise via Metal shader, applied as a SwiftUI color effect.

```swift
.noiseOverlay(amount: Double = 0.15) -> some View
```

- `amount` — Noise intensity 0...1. Default 0.15 (15%).
- Passing 0 skips the shader entirely.
- Apply before `.clipShape()` — noise needs the full view bounds.

---

## PointingHandCursor

Cross-platform pointing-hand cursor for interactive elements.

```swift
.pointingHandCursor(onHover: ((Bool) -> Void)? = nil) -> some View
```

### Platform behavior

- macOS: `.onContinuousHover` + `NSCursor.pointingHand.push()/pop()`
- visionOS: `.pointerStyle(.link)` + optional `.onHover`
- iOS: Optional `.onHover` only

### Important

Do NOT rewrite to use NSViewRepresentable with `resetCursorRects`/`addCursorRect`. That approach causes ~10% cursor failure rate. See Gotchas in SKILL.md.

---

## SelectionPopover

Generic popover for choosing one item from a list. Highlights the current selection, supports full keyboard navigation, and auto-dismisses on selection.

```swift
.selectionPopover<Item: Identifiable & Equatable>(
    isPresented: Binding<Bool>,
    items: [Item],
    current: Item,
    accentColor: Color = .blue,
    displayName: KeyPath<Item, String>,
    onSelect: @escaping (Item) -> Void
) -> some View
```

### Parameters

- `isPresented` — Binding that controls popover visibility.
- `items` — The items to display as selectable rows.
- `current` — The currently selected item (highlighted with a tinted fill).
- `accentColor` — Tint color for the selected/highlighted item. Default `.blue`.
- `displayName` — Key path to the display string on each item.
- `onSelect` — Called when the user taps or confirms an item.

### Keyboard Navigation

- **Up/Down arrows** — Move highlight through items (clamped to bounds).
- **Tab / Shift-Tab** — Move highlight down / up.
- **Return / Space** — Confirm the highlighted item.
- **Escape** — Dismiss the popover.

Highlight starts on the currently selected item.

### Fill Tiers

Each row uses `HeirloomButtonStyle` with a fill determined by its state:

| State | Fill |
|---|---|
| Highlighted (keyboard focus) | `.tinted(accentColor.opacity(0.15))` |
| Selected (current item, not highlighted) | `.tinted(accentColor.opacity(0.08))` |
| Normal | `.glass` |

### Layout

- Width: 280pt
- Uses `.presentationCompactAdaptation(.popover)` for consistent popover behavior on all platforms.

---

## LocationSearchPopover

Self-contained location search popover. Handles MapKit typeahead search, CoreLocation current-location resolution, keyboard navigation, and dismissal. The client receives a `LocationResult` without managing any search state.

```swift
.locationSearchPopover(
    isPresented: Binding<Bool>,
    accentColor: Color = .blue,
    onSelect: @escaping (LocationResult) -> Void
) -> some View
```

### Parameters

- `isPresented` — Binding that controls popover visibility.
- `accentColor` — Tint color for the highlighted result row and search field caret. Default `.blue`.
- `onSelect` — Called when the user selects a location (via search result or current location).

### Features

- **Typeahead search** — Uses `MKLocalSearchCompleter` for city/region autocomplete.
- **Current location** — "Use Current Location" button resolves coordinates via `CLLocationManager` and reverse geocodes a display name.
- **Keyboard navigation** — Arrow keys, tab, return, and escape via an internal `KeyboardSearchField`.
- **Auto-dismiss** — Popover closes and search state resets when a location is selected or the popover disappears.

### Info.plist Requirement

Apps that use the "Use Current Location" feature must include `NSLocationWhenInUseUsageDescription` in their Info.plist.

### Layout

- Width: 280pt
- Uses `.presentationCompactAdaptation(.popover)`.

---

## LocationResult

A geocoded location returned by `locationSearchPopover`. Plain struct — clients do not need to import CoreLocation or MapKit.

```swift
struct LocationResult: Sendable, Equatable {
    let name: String                    // e.g. "Denver, CO"
    let latitude: Double
    let longitude: Double
    let timeZoneIdentifier: String?     // IANA identifier, e.g. "America/Denver"

    init(
        name: String,
        latitude: Double,
        longitude: Double,
        timeZoneIdentifier: String? = nil
    )
}
```

### Properties

- `name` — Display name (e.g. "Denver, CO").
- `latitude` — Latitude of this location.
- `longitude` — Longitude of this location.
- `timeZoneIdentifier` — IANA time zone identifier (e.g. "America/Denver"), or `nil` when the system cannot determine the time zone.

---

## KeyboardSearchField

A text field that intercepts keyboard navigation events at the platform level, routing them to SwiftUI callbacks.

```swift
KeyboardSearchField(
    text: Binding<String>,
    placeholder: String = "Search",
    accentColor: Color = .blue,
    focusOnAppear: Bool = true,
    onSubmit: @escaping () -> Void = {},
    onEscape: @escaping () -> Void = {},
    onUpArrow: @escaping () -> Void = {},
    onDownArrow: @escaping () -> Void = {},
    onTab: @escaping () -> Void = {},
    onShiftTab: @escaping () -> Void = {}
)
```

### Parameters

- `text` — Binding to the search query string.
- `placeholder` — Placeholder text shown when the field is empty. Default `"Search"`.
- `accentColor` — Color for the insertion point caret (macOS only). Default `.blue`.
- `focusOnAppear` — Whether the field becomes first responder on appear. Default `true`.
- `onSubmit` — Called when the user presses Return.
- `onEscape` — Called when the user presses Escape.
- `onUpArrow` — Called when the user presses the up arrow key.
- `onDownArrow` — Called when the user presses the down arrow key.
- `onTab` — Called when the user presses Tab.
- `onShiftTab` — Called when the user presses Shift-Tab.

### Platform behavior

- **macOS:** `NSViewRepresentable` wrapping `NSTextField`. Captures arrow keys, tab, shift-tab, escape, and return at the AppKit level via `NSTextFieldDelegate.control(_:textView:doCommandBy:)`.
- **iOS:** Plain `TextField` with `.onSubmit`. Arrow key, tab, and escape callbacks are unavailable on this platform.
