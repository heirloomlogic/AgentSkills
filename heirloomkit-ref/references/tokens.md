# HeirloomKit Tokens Reference

## Table of Contents

- [Color Tokens](#color-tokens)
- [Spacing Constants](#spacing-constants)
- [Padding Helpers](#padding-helpers)
- [AppearanceManager](#appearancemanager)

---

## Color Tokens

Adaptive light/dark colors defined in sRGB. No asset catalog needed.

### Text Colors

| Token | Light | Dark | Use |
|---|---|---|---|
| `.textPrimary` | `#1C1B18` | `#E5E5E5` | Body text |
| `.textSecondary` | `#6B6860` | `#8E8E93` | Captions, labels |
| `.textTertiary` | `#9E9A90` | `#636366` | Disabled, hints |

### Surface Colors

| Token | Light | Dark | Use |
|---|---|---|---|
| `.surfacePrimary` | `#FAF7F3` | `#1A1A1C` | Page background |
| `.surfaceSecondary` | `#F2EFE9` | `#2C2C2E` | Cards, panels |
| `.surfaceTertiary` | `#E8E4DD` | `#48484A` | Grouped rows |

### Adaptive Color Init

For custom adaptive colors outside the built-in palette:

```swift
Color(light: .red, dark: .orange)
```

Platform implementation:
- iOS: `UIColor { traits in ... }`
- macOS: `NSColor(name:) { appearance in ... }`
- Other: Falls back to light color

---

## Spacing Constants

Static properties on `CGFloat`:

| Constant | Value | Use |
|---|---|---|
| `.tight` | 4 pt | Dense layouts, tight groupings, small insets |
| `.standard` | 8 pt | General-purpose spacing, padding, small corner radii |
| `.relaxed` | 12 pt | Content margins, card padding, default corner radii |
| `.spacious` | 16 pt | Section gaps, layout regions, generous insets |

```swift
VStack(spacing: .relaxed) { ... }
.padding(.standard)
.padding(.horizontal, .spacious)
```

---

## Padding Helpers

Two View extensions for shorthand padding:

### Per-Edge

```swift
view.padding(
    top: CGFloat = 0,
    right: CGFloat = 0,
    bottom: CGFloat = 0,
    left: CGFloat = 0
)
```

`right` maps to `.trailing`, `left` maps to `.leading`. All default to 0.

### Symmetric

```swift
view.padding(
    horizontal: CGFloat,
    vertical: CGFloat
)
```

---

## AppearanceManager

Observable manager for light/dark mode with UserDefaults persistence.

```swift
@Observable @MainActor class AppearanceManager: Sendable {
    var mode: AppearanceMode  // read-only

    static func restored() -> AppearanceManager  // from UserDefaults
    func setMode(_ newMode: AppearanceMode)      // persists + applies
    func applyPlatformAppearance()               // applies via native API
}

enum AppearanceMode: String, CaseIterable, Sendable {
    case system  // colorScheme: nil
    case light   // colorScheme: .light
    case dark    // colorScheme: .dark
}
```

UserDefaults key: `"HeirloomKit.appearanceMode"`

### Platform behavior

- macOS: Sets `NSApp.appearance`
- iOS: Sets window `overrideUserInterfaceStyle`
