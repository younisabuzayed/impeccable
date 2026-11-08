# SwiftUI framework

The declarative way to build Apple surfaces: iOS, iPadOS, watchOS, tvOS, from iOS 16-era APIs onward. Load when PRODUCT.md's Stack names SwiftUI and the platform is `ios` or `adaptive`; read with [ios.md](ios.md) (design language) and, where declared, [form-factors.md](form-factors.md), [watch.md](watch.md), [tv.md](tv.md). SwiftUI's job is to make adaptivity a description, not an if-tree: state what the layout means and let the framework resolve it.

## Structure from size classes, not devices

- Read `@Environment(\.horizontalSizeClass)` (and `verticalSizeClass` where height matters, iPhone landscape). Branching on `UIDevice.current.userInterfaceIdiom` for layout is the failure this replaces.
- `NavigationSplitView` is the adaptive container: columns at regular width, a stack at compact. Put a `NavigationStack` inside the detail column. Prefer it over hand-rolled two-pane logic for [list-detail](form-factors.md) and sidebar morphs.
- `ViewThatFits` picks between variants (horizontal vs vertical controls, full label vs short) without reading geometry yourself. `AnyLayout` switches whole layout strategies (HStack→VStack, Grid→List) from a size class.

## Adapting composition

- **Grids:** `LazyVGrid`/`LazyHGrid` with `GridItem(.adaptive(minimum:), spacing:)` keeps items readable across widths, the [feed](form-factors.md) pattern.
- **Sidebar morph:** a `List` with `.listStyle(.sidebar)` in `NavigationSplitView` renders tab-bar-like navigation at compact width and a sidebar at regular. Destinations stay identical; only the container changes.
- **Sheets vs popovers:** `.sheet` everywhere; `.popover` is regular-width-only and needs a fallback at compact (`ViewThatFits` or a size-class branch), never `.popover` alone.

## Type & metrics that follow the user

- System text styles only (`Font.title2`, `.body`, …); never `Font.system(size:)` for UI text. Pair custom spacing and icon sizes with `@ScaledMetric` so they track Dynamic Type with the text they accompany.
- Safe areas are the default: use `ignoresSafeArea` only for backgrounds, never to squeeze content under bars or corners.

## Watch and TV, in SwiftUI

- **watchOS:** `TabView` + `.tabViewStyle(.verticalPage)`, crown-driven scroll (custom `digitalCrownRotation` only for non-scroll values), `scenePadding` for circular margins, and the `\.isLuminanceReduced` environment with `TimelineView` for always-on restraint; see [watch.md](watch.md).
- **tvOS:** `focusable(_:interactions:)`, `focusSection()`, `defaultFocus(_:_:priority:)`, and `@FocusState` carry the focus engine; see [tv.md](tv.md).

## Anti-patterns

- `GeometryReader` scaffolding to answer a question a size class already answers; it delays layout and inverts ownership. Reserve it for genuinely measured cases.
- Fixed frames that fight Dynamic Type (`frame(height: 44)` on text containers), and `.frame(maxWidth: .infinity)` used as a layout plan instead of stacks.
- Re-deriving navigation by hand instead of `NavigationStack`/`NavigationSplitView` (the deprecated `NavigationView` is not a substitute).

## Verifying the build

SwiftUI previews across size-class combinations catch structure cheaply, but shipped evidence still comes from the simulator captures in [ios.md](ios.md) (and [watch.md](watch.md) / [tv.md](tv.md) where declared): every declared form factor, both appearances, a large Dynamic Type size. Name which class each capture ran at.
