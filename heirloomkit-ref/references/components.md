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

---

## GlassFrame

Clips content to a shape, fills with material, renders an optional etched border, and applies a depth effect.

```swift
.glassFrame(
    shape: GlassBackgroundShape = .roundedRectangle(),
    tint: Color? = nil,
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
- `tint` — Optional color overlay. Nil uses an adaptive dark/light default.
- `etched` — Thin border for edge visibility on iOS 26+. Default true. Set false when floating over a contrasting surface.
- `contentInset` — Clips content 1.25pt inside the glass edge. Default true. Set false for edge-to-edge content like images.
- `depth` — Shadow or glow behind the frame. Default `.shadow(.soft)`.

### Platform behavior

- iOS 26+ / macOS 26+: Uses `.glassEffect(.regular)` with optional etched border
- Earlier: Falls back to `.ultraThinMaterial` with stroke border

---

## HeirloomButtonStyle

ButtonStyle wrapping labels in a glass frame with cursor, press feedback, and hover glow.

```swift
// Default (static property)
.buttonStyle(.heirloom)

// Customized (static method)
.buttonStyle(.heirloom(
    shape: GlassBackgroundShape = .capsule,
    tint: Color? = nil,
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

### Feedback

- Press: scale 0.97, opacity 0.85 (0.15s snappy)
- Hover: brightness +0.05
- Content shadow: dark mode black 0.25, light mode white 0.45
- Includes pointing-hand cursor automatically

---

## GlassSection

Titled section container over a glass panel. Replaces `Form`/`Section` for full control over margins and styling. Dividers between children inserted automatically via DividedVStack.

```swift
GlassSection(
    _ title: String? = nil,
    spacing: CGFloat = .standard,
    etched: Bool = true,
    @ViewBuilder content: @escaping () -> Content
)
```

### Parameters

- `title` — Optional uppercase header rendered inside the glass panel with `.heirloomSans(13, .medium)`, 1pt letter spacing, background fill at textPrimary 0.04 with 0.25 noise.
- `spacing` — Vertical spacing between content items. Default `.standard` (8pt).
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
