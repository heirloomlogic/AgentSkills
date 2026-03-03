# The Heirloom Standard: Visual Codex

**Version:** 3.0
**Status:** Living Document

## I. The Aesthetic Synthesis

Heirloom Logic builds software that feels *built*, not just programmed. We reject
disposable tech aesthetics. We do not create "retro" apps; we create native apps
with the structural integrity of the past.

**The Equation:**

> Heirloom = Arts & Crafts (Truth to Materials) + Art Deco (Optimism of Structure)

1. **Truth to Materials (1880–1920):** In software, the "material" is the pixel,
   the font, and the screen. We do not simulate leather, wood, or glass.
2. **The Streamline Ethos (1920–1940):** We embrace verticality, geometric
   boldness, and the elegance of machinery.

---

## II. The Heirloom Stack: Typography

We ship a standardized font package
([HeirloomFonts](https://github.com/heirloomlogic/HeirloomFonts)) across all apps.
All structural UI uses this stack. Individual apps may add supplementary display
faces for user-facing content (e.g., Adagio's selectable quote fonts), but the
chrome always speaks Heirloom.

### The Three Voices

**Cabin** — *The Structure* (`.heirloomSans()`)
- Role: Navigation, buttons, headings, window titles, labels.
- Character: Humanist sans-serif with geometric underpinnings. Warmer than
  Futura, more grounded than Avenir. Carries the Art Deco structure without
  clinical coldness.
- Weights: Regular, Medium, SemiBold, Bold (all with italic).
- Guidance: Slightly increase tracking on headings. Let it breathe.

**CrimsonPro** — *The Soul* (`.heirloomSerif()`)
- Role: Body text, long-form content, quotes, input fields.
- Character: Low-contrast, high x-height serif. Sits on screen like ink without
  the brittle "webview" look of system serifs. The warmth counterpoint to
  Cabin's geometry.
- Weights: ExtraLight through Bold.
- Guidance: Pair with generous line height. It rewards slow reading.

**Space Mono** — *The Logic* (`.heirloomMono()`)
- Role: Timestamps, coordinates, tabular data, technical readouts.
- Character: 1960s laboratory spec sheet aesthetic. Bridges the craftsman's
  workshop and the engineer's lab.
- Weights: Regular, Bold.
- Guidance: Use for anything measured. Dates, degrees, percentages, codes.

### macOS Font Scaling

macOS has no native Dynamic Type. We built `FontScaleManager` into the package
to provide CMD+/CMD- scaling (0.8×–2.0×). All `.heirloom*()` modifiers
automatically read the `fontScale` environment value. This is non-negotiable
for accessibility.

---

## III. Shared Design Components

### The `.glassFrame()` Modifier

Our signature container treatment, used across Adagio and Edict for thumbnails,
content sections, and inspectors. Provides configurable:

- **Shape**: `.roundedRectangle()`, `.capsule`, `.circle`
- **Fill**: `.solid()`, `.gradient()`, `.material()`
- **Depth**: `.shadow(.soft)`, `.shadow(.hard)`, `.glow(color:)`, `.none`

Default appearance: rounded rectangle with soft shadow on a dark background.
This modifier replaces ad-hoc `clipShape` + `shadow` stacking throughout the
codebase. Use it.

### Custom SF Symbols (Edict)

Edict's astrology domain required a purpose-built symbol library — standard
SF Symbols don't cover planetary glyphs or zodiac signs. These were produced
via the `swiftdraw` CLI tool and cover:

- **Planets:** Sun, Moon, Mercury, Venus, Mars, Jupiter, Saturn, Uranus,
  Neptune, Pluto
- **Signs:** All 12 zodiac signs (Aries through Pisces)
- **Symbols:** Ascendant, Descendant, Midheaven, Imum Coeli, Nodes, aspects,
  dignities
- **Utility:** Confirm, Caution, Edit, Focus, Intent, Edict, Scan, and
  others

These follow Apple's SF Symbol design guidelines for weight and optical size
consistency and integrate with SwiftUI's `Image(systemName:)` pattern.

This is not a studio-wide convention. Most apps use standard SF Symbols. Custom
symbols are a significant investment and only warranted when a domain genuinely
lacks representation in Apple's library.

---

## IV. The Color System

The palette is warm and charcoal-based — deliberately not blue-black or
true-black. The warmth keeps things grounded and avoids the "dev tool" coldness
that pure greys produce.

### Semantic Tiers

These colors adapt between light and dark mode automatically.

| Token | Light | Dark | Role |
|-------|-------|------|------|
| SurfacePrimary | `#FAF7F3` | `#1A1A1C` | Main background |
| SurfaceSecondary | `#F2EFE9` | `#2C2C2E` | Sidebars, panels |
| SurfaceTertiary | `#E8E4DD` | `#48484A` | Cards, grouped sections |
| TextPrimary | `#1C1B18` | `#E5E5E5` | Headlines, body text |
| TextSecondary | `#6B6860` | `#8E8E93` | Supporting labels |
| TextTertiary | `#9E9A90` | `#636366` | Captions, timestamps |
| Scrim | `#000` @ 8% | `#000` @ 15% | Overlay dimming |

### Named Accents

These carry semantic meaning across all apps.

| Name | Light | Dark | Feeling |
|------|-------|------|---------|
| BurnishedBrass | `#9A7B3A` | `#C5A059` | Warm gold — opportunity, selection, the brand accent |
| BurntSienna | `#B5433A` | `#CD5C5C` | Warm red — tension, hazard, retrograde |
| SiennaDark | `#8C2D2D` | `#A63C3C` | Deep red — critical warning, strong negative |
| AntiqueEmerald | `#237045` | `#2E8B57` | Deep green — favorable conditions, confirmation |
| MutedViridian | `#36654F` | `#448062` | Soft green — secondary positive, growth |
| MediumSeaGreen | `#2D8B58` | `#3CB371` | Bright green — strong positive signal |
| DustedAmethyst | `#7B6B9E` | `#A89AC2` | Soft purple — neutral accent, mystical register |
| RawParchment | `#6E6250` | `#8C7C62` | Warm tan — tertiary accent, earthiness |

### Adagio-Only Colors

| Name | Value | Role |
|------|-------|------|
| CinemaCharcoal | `#262626` | Deep black for cinematic card backgrounds |
| CinemaSlate | `#333439` | Blue-tinted dark for sidebar/navigation |
| SilverBronze | `#CCA661` | Lighter gold for secondary metallic accents |

### Design Notes

- **No blues in the core palette.** The original "Midnight Blue" was abandoned
  early — it made the apps look amateurish. The warm charcoal (SurfacePrimary
  dark: `#1A1A1C`) is nearly black but with a warm undertone that pairs with
  the brass/sienna accents.
- **Signal colors come in pairs.** Red uses BurntSienna (moderate) / SiennaDark
  (severe). Green uses MutedViridian (soft) / AntiqueEmerald (strong) /
  MediumSeaGreen (bright). This lets data-dense views like Edict's telemetry
  express gradient severity.
- **Both apps share the same core palette.** This is intentional — a user who
  uses both Adagio and Edict should recognize them as siblings.

---

## V. Per-App Design Notes

These are not visual metaphors to chase — they are observations about what
each app has become through months of development. Use them as ground truth,
not aspiration.

### Adagio (Quote Slideshows)

- **What it is:** A card-based quote display with photo backgrounds, user-
  selectable typography, and deck organization. It is image-forward.
- **UI character:** Calm, spacious, cinematic. The sidebar is minimal; the
  card preview dominates. The inspector panel uses `.glassFrame()` sections.
- **Typography in content:** Users choose their own display fonts (Cochin,
  serif system fonts, etc.). The *app chrome* uses the Heirloom Stack.
- **Interaction:** Unhurried. Ken Burns panning on photos. No snap or urgency.

### Edict (Strategic Astrology)

- **What it is:** A campaign-based electional astrology engine with a four-phase
  flow: Intent → Scan → Focus → Edict.
- **UI character:** Data-dense but not cluttered. Dark backgrounds with signal
  colors (coral/sage/gold) for chart data. The constellation visualization in
  Scan phase uses blue wireframe on dark void. Terminal-style status messages
  during processing ("preparing re-scan," "consulting the ephemeris").
- **Typography:** Heavy use of Space Mono for planetary degrees, timestamps,
  and telemetry. CrimsonPro for the written strategic briefings. Cabin for
  navigation and labels.
- **Custom glyphs:** The extensive SF Symbol library is a first-class design
  element, not decoration. Planets, signs, and aspects appear in tables and
  charts as integrated data, not ornament.

### Recite (Mental Math Cards)

- **What it is:** A voice-driven math tutor for kids.
- **UI character:** Sharp interaction physics. We want the feeling of a
  mechanical "thunk" — a letterpress strike, not a fluid ripple.
- **Palette:** Deep Indigo, Cream, Heirloom Bronze.

### Tempo (Sourdough Planner)

- **What it is:** A bake planner built around sleep and work schedules.
- **UI character:** Laboratory precision applied to biological process.
- **Palette:** Flour-dusted Slate, Terracotta, Sage.
- **Typography:** Space Mono is the hero (temperatures, times, percentages).

---

## VI. Slow Tech UX Principles

These principles come from the company's founding philosophy and should inform
every design decision:

1. **No attention traps.** Our apps do a thing and stay out of the way. No
   feeds, no streaks, no daily nudges engineered to pull users back.
2. **Analog skills, digital tools.** Bread, math, quotes, astrology — these are
   all practices that predate screens. The software enhances; it doesn't replace.
3. **Silence is a feature.** Empty states should feel restful, not broken.
   Loading states should feel like patience, not failure.
4. **Respect the platform.** We build for Apple. We follow Apple's conventions
   for window management, accessibility, and interaction. Our personality comes
   through typography, color, and restraint — not by reinventing the platform.
