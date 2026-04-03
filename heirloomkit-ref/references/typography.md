# HeirloomKit Typography Reference

## Table of Contents

- [Font Families](#font-families)
- [View Modifiers](#view-modifiers)
- [Font.custom Access](#fontcustom-access)
- [Variant Enums](#variant-enums)
- [FontScaleManager](#fontscalemanager)
- [Environment Key](#environment-key)

---

## Font Families

Three bundled typefaces registered via `HeirloomFonts.registerAll()`.

| Family | Name | Character |
|---|---|---|
| Sans | Cabin | Humanist sans-serif, warm and legible |
| Serif | CrimsonPro | Old Style serif, Garamond/Minion lineage |
| Mono | Space Mono | 1960s lab-spec monospace |

---

## View Modifiers

All modifiers multiply size by the `\.fontScale` environment value.

```swift
// Sans — Cabin
.heirloomSans(_ size: CGFloat = 14, _ variant: SansVariant = .regular)

// Serif — CrimsonPro (size * 1.18 optical compensation)
.heirloomSerif(_ size: CGFloat = 14, _ variant: SerifVariant = .regular)

// Mono — Space Mono
.heirloomMono(_ size: CGFloat = 12, _ variant: MonoVariant = .regular)
```

The serif modifier multiplies by 1.18 because CrimsonPro's thin strokes appear smaller than Cabin at the same point size. Don't manually compensate.

---

## Font.custom Access

For `Font.custom()` contexts (not view modifiers):

```swift
.font(.heirloomSans(size: 14))
.font(.heirloomSans(size: 14, variant: .bold))
.font(.heirloomSerif(size: 16, variant: .light))
.font(.heirloomMono(size: 12))

// Relative sizing
.font(.heirloomSans(.headline))
.font(.heirloomSerif(.body))
.font(.heirloomMono(.caption))
```

---

## Variant Enums

### SansVariant (Cabin)

| Case | PostScript Name |
|---|---|
| `.regular` | Cabin-Regular |
| `.italic` | Cabin-Italic |
| `.medium` | Cabin-Medium |
| `.mediumItalic` | Cabin-MediumItalic |
| `.bold` | Cabin-Bold |
| `.boldItalic` | Cabin-BoldItalic |

### SerifVariant (CrimsonPro)

Weight names are shifted down from native:

| Case | PostScript Name | Note |
|---|---|---|
| `.light` | CrimsonPro-ExtraLight | shifted |
| `.lightItalic` | CrimsonPro-ExtraLightItalic | shifted |
| `.regular` | CrimsonPro-Light | shifted |
| `.italic` | CrimsonPro-LightItalic | shifted |
| `.medium` | CrimsonPro-Medium | |
| `.mediumItalic` | CrimsonPro-MediumItalic | |
| `.bold` | CrimsonPro-Bold | |
| `.boldItalic` | CrimsonPro-BoldItalic | |

### MonoVariant (Space Mono)

| Case | PostScript Name |
|---|---|
| `.regular` | SpaceMono-Regular |
| `.italic` | SpaceMono-Italic |
| `.bold` | SpaceMono-Bold |
| `.boldItalic` | SpaceMono-BoldItalic |

### Core Text Access

Each variant has a `ctFontName: CFString` property:

```swift
let font = CTFontCreateWithName(SerifVariant.bold.ctFontName, 14, nil)
```

---

## FontScaleManager

Observable manager for user-adjustable font scaling. Persists to UserDefaults.

```swift
@Observable @MainActor class FontScaleManager: Sendable {
    static let minScale: CGFloat = 0.8
    static let maxScale: CGFloat = 2.0
    static let step: CGFloat = 0.1

    var scale: CGFloat  // read-only

    static func restored() -> FontScaleManager  // from UserDefaults
    func increase()  // +0.1, clamped to 2.0
    func decrease()  // -0.1, clamped to 0.8
    func reset()     // back to 1.0
}
```

UserDefaults key: `"HeirloomFonts.fontScale"`

---

## Environment Key

```swift
@Environment(\.fontScale) private var fontScale

// Inject at app level
ContentView()
    .environment(\.fontScale, fontScaleManager.scale)
```

Default value: 1.0
