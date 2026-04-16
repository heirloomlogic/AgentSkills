---
name: heirloomkit-ref
description: "HeirloomKit component and modifier reference for all Heirloom Logic SwiftUI projects. Use whenever writing, reviewing, or modifying UI code in any project that imports HeirloomKit — including glass frames, capsule bars, typography, color tokens, spacing, toolbar chrome, content fade, noise overlays, cursor handling, and button styles. Triggers on: HeirloomKit, glassFrame, GlassSection, CapsuleBar, heirloomSans, heirloomSerif, heirloomMono, contentFade, floatingToolbar, noiseOverlay, pointingHandCursor, HeirloomButtonStyle, surfacePrimary, textPrimary, FontScaleManager, AppearanceManager, selectionPopover, SelectionPopover, locationSearchPopover, LocationSearchPopover, LocationResult, KeyboardSearchField. Also use when the project's Package.swift depends on HeirloomKit, even if the user doesn't mention it by name."
---

# HeirloomKit Reference

Quick-reference for building UI with HeirloomKit. Organized by what you need to do, not by file.

## When to Read the Detailed References

This file covers conventions, gotchas, and common patterns. For full API signatures and parameter lists, read the relevant reference file:

- `references/components.md` — GlassFrame, GlassSection, HeirloomButtonStyle (`.heirloom`), CapsuleBar, NoiseOverlay, ContentFade, FloatingToolbar, PointingHandCursor, DividedVStack, SelectionPopover, LocationSearchPopover, LocationResult, KeyboardSearchField
- `references/typography.md` — Fonts, view modifiers, font scale, variant enums, Core Text access
- `references/tokens.md` — Colors, spacing constants, padding helpers

Read the reference file when you need exact parameter names, defaults, or enum cases. Read this file for how things fit together.

---

## Setup

Register fonts at app launch. Everything else is modifiers and views.

```swift
@main struct MyApp: App {
    init() { HeirloomFonts.registerAll() }
}
```

Inject font scale and appearance into the environment:

```swift
@State private var fontScaleManager = FontScaleManager.restored()
@State private var appearanceManager = AppearanceManager.restored()

var body: some Scene {
    WindowGroup {
        ContentView()
            .environment(\.fontScale, fontScaleManager.scale)
            .onAppear { appearanceManager.applyPlatformAppearance() }
    }
}
```

---

## Gotchas

These will bite you if you don't know about them. Read this section first.

### Modifier Ordering

`.contentFade` must come BEFORE `.inspector()`. The header is placed via overlay, so applying it after `.inspector()` causes the header to span the full window width, overlapping the inspector panel.

```swift
// Correct
ZStack { content }
    .contentFade { CapsuleBar(...) }
    .inspector(isPresented: $show) { ... }
    .toolbar { ... }
    .floatingToolbar()

// Wrong — header overlaps inspector
ZStack { content }
    .inspector(isPresented: $show) { ... }
    .floatingToolbar()
    .contentFade { CapsuleBar(...) }
```

### CapsuleBar Needs a Stable Container

When the CapsuleBar sits above content that switches views based on the bar's state, wrap that content in a `ZStack`. Without it, SwiftUI recreates the view on every branch change, replacing the capsule's spring animation with a hard cut.

```swift
// Correct — ZStack provides stable identity
ZStack {
    switch currentTab {
    case .photos:  PhotosView()
    case .albums:  AlbumsView()
    }
}
.contentFade {
    CapsuleBar(currentTab: currentTab, ...)
}

// Wrong — switch directly in NavigationStack body
NavigationStack {
    switch currentTab { ... }  // recreated on each change
}
```

### ContentFade: Apply to the ScrollView, Not a Parent

`.contentFade` masks the entire view it's attached to and uses `.contentMargins` to offset scroll content below the header. Both behaviors require it to be applied directly to a `ScrollView`. Attaching it to a parent container (e.g., `NavigationSplitView`, outer `VStack`) fades non-scrolling children (sidebars, fixed panels) and breaks the header offset since `.contentMargins` is a no-op on non-scroll views.

```swift
// Correct — fade only affects the scrolling detail
NavigationSplitView {
    Sidebar()
} detail: {
    ScrollView { DetailContent() }
        .contentFade { CapsuleBar(...) }
}

// Wrong — fades sidebar, header offset broken
NavigationSplitView {
    Sidebar()
} detail: {
    DetailContent()
}
.contentFade { CapsuleBar(...) }
```

### Toolbar Buttons: No Custom Styles

Do not apply `.heirloom` or any custom ButtonStyle to buttons inside `ToolbarItem`. macOS wraps toolbar buttons in its own chrome. A custom style produces a double-layered look. Use `Label` and let the system handle feedback.

```swift
.toolbar {
    ToolbarItem(placement: .primaryAction) {
        Button { action() } label: {
            Label("New", systemImage: "plus")
        }
        // No .buttonStyle here
    }
}
```

### Large Glass Frames Glitch

The liquid glass material (`glassEffect`) can produce visual artifacts on very large views — full-page documents, tall scrollable content panes, etc. When a `.glassFrame` view is too large for the material to render cleanly, switch to `.solid(color)`:

```swift
// Before — glitches on large content
documentContent.glassFrame(depth: .shadow(.soft))

// After — same clipping, border, and shadow, no material
documentContent.glassFrame(fill: .solid(.surfaceSecondary))
```

`.solid` is a drop-in replacement: shape clipping, etched borders, content inset, and depth effects all work identically.

### Cursor: Do Not Rewrite

`pointingHandCursor()` uses `.onContinuousHover` + `NSCursor.push()/pop()` on macOS. Do NOT replace this with `resetCursorRects`/`addCursorRect` via NSViewRepresentable. The NSViewRepresentable approach has a ~10% failure rate when clicking and moving between adjacent cursor views due to timing gaps during SwiftUI layout passes.

### CrimsonPro Optical Compensation

The `.heirloomSerif()` modifier multiplies size by 1.18 because CrimsonPro's thin strokes make it appear smaller than Cabin at the same point size. Don't manually compensate — the modifier handles it.

### SerifVariant Weight Shift

CrimsonPro variant names are shifted down from native PostScript names:
- `.light` maps to CrimsonPro-ExtraLight
- `.regular` maps to CrimsonPro-Light

This is intentional — the design system's "regular" is visually lighter than the font's native "regular" weight.

---

## Common Patterns

### Glass Frame on Text

Padding goes on the text, then glassFrame wraps it:

```swift
Text("Label")
    .heirloomMono(11)
    .padding(horizontal: .spacious, vertical: .standard)
    .glassFrame(shape: .capsule)
```

### Glass Frame on Images (Edge-to-Edge)

Set `contentInset: false` so the image reaches the glass edges:

```swift
Image("photo")
    .resizable()
    .scaledToFill()
    .frame(height: 80)
    .glassFrame(
        shape: .roundedRectangle(cornerRadius: 10),
        contentInset: false
    )
```

### Interactive Elements

For buttons, use HeirloomButtonStyle instead of manually stacking `.glassFrame()` + `.pointingHandCursor()`:

```swift
Button("Save") { save() }
    .buttonStyle(.heirloom)

Button { toggle() } label: {
    Image(systemName: "plus").frame(width: 32, height: 32)
}
.buttonStyle(.heirloom(shape: .disc))
```

For tinted or solid fill buttons, foreground color and content shadow are set automatically based on fill luminance — no need to set `.foregroundStyle(...)` manually:

```swift
// Dark fill → white text, dark shadow (automatic)
Button("Cast") { cast() }
    .buttonStyle(.heirloom(fill: .tinted(Color(red: 0.6, green: 0.45, blue: 0.25))))

// Light fill → textPrimary text, light shadow (automatic)
Button("Light") { action() }
    .buttonStyle(.heirloom(fill: .solid(.surfacePrimary)))
```

For non-button interactive views, add `.contentShape()` before `.pointingHandCursor()`:

```swift
Text("Clickable")
    .padding(horizontal: .spacious, vertical: .standard)
    .contentShape(Capsule())
    .glassFrame(shape: .capsule)
    .pointingHandCursor { isHovering = $0 }
```

### Sections with Labeled Content

Font styling goes on both the LabeledContent and its nested content:

```swift
GlassSection("Settings") {
    LabeledContent("Theme") {
        Text("Dark").heirloomSans(14)
    }
    .heirloomSans(14)
}
```

### Noise on Shapes

Apply noise before clipShape:

```swift
RoundedRectangle(cornerRadius: 10)
    .fill(gradient)
    .noiseOverlay(amount: 0.15)
    .clipShape(RoundedRectangle(cornerRadius: 10))
```

### Content Fade with Header

```swift
ScrollView { content }
    .contentFade {
        SearchField(text: $query)
            .padding(.horizontal, .spacious)
            .padding(.vertical, .standard)
    }
```

Without a header (fade only):

```swift
ScrollView { content }
    .contentFade()
```

In a split-view layout, apply `.contentFade` to the `ScrollView` inside the detail column:

```swift
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
```

### Selection Popover

Attach to a button that shows the current value. The popover handles keyboard navigation and auto-dismisses:

```swift
Button(currentLevel.displayName) { showPicker = true }
    .selectionPopover(
        isPresented: $showPicker,
        items: FastingLevel.allCases,
        current: currentLevel,
        accentColor: .burnishedBrass,
        displayName: \.displayName
    ) { selected in
        currentLevel = selected
    }
```

`Item` must conform to `Identifiable & Equatable`.

### Location Search Popover

Self-contained location picker with MapKit typeahead and current-location support. Returns a `LocationResult` struct:

```swift
Button(locationName) { showSearch = true }
    .locationSearchPopover(
        isPresented: $showSearch,
        accentColor: .burnishedBrass
    ) { location in
        locationName = location.name
        latitude = location.latitude
        longitude = location.longitude
        timeZone = location.timeZoneIdentifier
    }
```

Requires `NSLocationWhenInUseUsageDescription` in Info.plist for "Use Current Location".

### Appearance Switcher

```swift
ForEach(AppearanceMode.allCases, id: \.self) { mode in
    Button { appearanceManager.setMode(mode) } label: {
        Text(mode.rawValue.capitalized)
            .foregroundStyle(
                appearanceManager.mode == mode
                    ? Color.textPrimary
                    : Color.textTertiary
            )
    }
    .buttonStyle(.heirloom)
}
```

### Font Scale Controls

```swift
Button { fontScaleManager.decrease() } label: {
    Image(systemName: "minus").frame(width: 20, height: 20)
}
.buttonStyle(.heirloom(shape: .disc))

Text("\(Int(fontScaleManager.scale * 100))%")
    .heirloomMono(18)

Button { fontScaleManager.increase() } label: {
    Image(systemName: "plus").frame(width: 20, height: 20)
}
.buttonStyle(.heirloom(shape: .disc))
```

---

## Text Hierarchy

| Role | Modifier | Color |
|---|---|---|
| Heading | `.heirloomSans(18, .bold)` or larger | `.textPrimary` |
| Body | `.heirloomSans(14)` or `.heirloomSerif(15)` | `.textPrimary` |
| Label | `.heirloomSans(13, .medium)` | `.textSecondary` |
| Caption | `.heirloomMono(11)` or `.heirloomMono(10)` | `.textTertiary` |
| Code | `.heirloomMono(13)` | `.textPrimary` |

---

## Spacing Quick Reference

| Constant | Value | Typical use |
|---|---|---|
| `.tight` | 4 pt | Dense groupings, small insets |
| `.standard` | 8 pt | General spacing, default padding |
| `.relaxed` | 12 pt | Card padding, default corner radius |
| `.spacious` | 16 pt | Section gaps, generous insets |
| `.generous` | 24 pt | Page margins, large card gaps |
| `.expansive` | 32 pt | Major section dividers, hero spacing |
| `.vast` | 48 pt | Full-bleed regions, page-level separation |

```swift
VStack(spacing: .relaxed) { ... }
.padding(.horizontal, .spacious)
.padding(horizontal: .spacious, vertical: .standard)  // symmetric helper
.padding(top: .relaxed, right: .standard, bottom: .relaxed, left: .standard)  // per-edge
```

---

## Surface Hierarchy

| Token | Use |
|---|---|
| `.surfacePrimary` | Page/window background |
| `.surfaceSecondary` | Cards, panels |
| `.surfaceTertiary` | Grouped rows, inset areas |
