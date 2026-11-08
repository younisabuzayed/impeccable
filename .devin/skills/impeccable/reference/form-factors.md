# Form factors

For native products (`ios` / `android` / `adaptive`) that ship beyond a handheld phone: tablet, watch, tv, or several of these. PRODUCT.md's `## Form factors` records which device classes the product ships to; the platform references ([ios.md](ios.md) / [android.md](android.md)) stay the design-language authority, and this file is the shared responsive model that spans them.

Two axes decide every native layout: **platform** (which design language, HIG or Material 3) and **form factor** (which device class). Never let either axis silently absorb the other: a watch app is not a small iPhone app, and a tablet layout is not a stretched phone.

## The size ladder

Structure from **window size**, never from device model or orientation. Each window maps to a width class and a height class independently; both can change at any moment (rotation, multitasking, folding, windowing) without the app restarting.

- **Apple: compact or regular per axis.** Two classes, all four combinations are reachable. Decide from the class, not the device: the same code must survive iPhone portrait, iPhone landscape (compact width, regular height on many models), iPad windowed, and iPad full screen. There is no numeric threshold in the HIG; treat ~600 pt as the folklore boundary and code against the class, not a constant.
- **Android: compact / medium / expanded / large / extra-large.** Width classes break at **600 / 840 / 1200 / 1600 dp**; height classes break at **480 / 900 dp**. Classes come from the *window*, explicitly not from the screen ("not intended for isTablet-style logic"). `large` and `extra-large` are opt-in (`supportLargeAndXLargeWidth = true`) and mostly matter on desktop-form-factor and foldable-inner displays.
- **Web mapping** (for adaptive products sharing a web surface): 640 / 768 / 1024 / 1280 px correspond roughly to compact/medium/expanded/large; content-driven breakpoints still win over dogma.

## Rules that span every class

- **Same functionality, different visibility.** A size change may reveal or hide panes; it may never remove a capability. If a pane hides at compact width, its content must remain reachable (navigation, not deletion).
- **Plan all four Apple combinations** and at least compact/medium/expanded on Android before building. The untested combination is where the defect ships.
- **Phone landscape is the trap.** Compact width with regular height is a real configuration; layouts keyed only to width classes still need height awareness for it.
- **Multitasking is a size, not an edge case.** iPadOS windows resize freely with macOS-like window controls (Split View/Slide Over are retired; apps do not control or get told the window arrangement), and Android multi-window and desktop windowing hand the app fractional screens. Size-class-driven layout handles all of it for free; device-model checks break in all of it.

## Canonical layouts (the three worth reaching for)

- **List-detail.** Two panes: collection on the left, content on the right. Expanded width shows both; medium and compact show one at a time with working Back. Android: `ListDetailPaneScaffold` + `rememberListDetailPaneScaffoldNavigator()` (+ `BackHandler` when only detail shows). iOS: `NavigationSplitView`, which collapses to a stack at compact width.
- **Feed.** Responsive grid that keeps items readable instead of stretching rows. Android: `LazyVerticalGrid` with `GridCells.Adaptive(minSize = 180.dp)` and `maxLineSpan` for full-width items; it behaves like a `LazyColumn` at compact width. iOS: `LazyVGrid` with `GridItem(.adaptive(minimum:), spacing:)`.
- **Supporting pane.** Main content plus auxiliary content (comments, inspector, summary). Android: `SupportingPaneScaffold`, 50/50 at medium width, **70% main / 30% supporting** at expanded, supporting content below main or in a bottom sheet at compact. iOS: secondary column in `NavigationSplitView` or a trailing inspector.

## Navigation changes shape with width

- **Android:** `NavigationSuiteScaffold` switches automatically by its default rule: **navigation bar when width or height is compact or the device is in tabletop posture; navigation rail for everything else**; a drawer fits expanded width via a `layoutType` override. Never leave a phone bottom bar untouched on a tablet.
- **iOS:** tab bar at compact width becomes a sidebar at regular width (`NavigationSplitView` list, `UISplitViewController` sidebar behavior). Keep destinations identical; only the container changes.

## Foldables and posture

React to posture (folded, unfolded, tabletop) through window size classes and hinge insets, not through hinge heuristics. Tabletop posture moves primary content to the upper half and controls to the lower half. Test folded, unfolded, and tabletop; an unhandled hinge inset hides content under the crease.

## Watches and TVs are different worlds

A watch or TV is not a "small/large phone"; their references carry their own rules and slop tests. Load [watch.md](watch.md) or [tv.md](tv.md) when either is a target form factor; the ladder here stops where their interaction models begin.

## Verifying the build

Capture every declared form factor, per the platform reference's Verifying the build commands, and name which class each capture ran at:

| Form factor | What to prove |
|---|---|
| phone | compact width; portrait and landscape; the landscape configuration shows no clipping and no lost function |
| tablet | expanded width both orientations; multitasking/windowing at a compact-width window; foldables through postures where applicable |
| watch | smallest and largest supported sizes; circular and rectangular; [watch.md](watch.md) verification |
| tv | focus traversal without a pointer on every screen; overscan safe areas; [tv.md](tv.md) verification |

**Avoid**:
- Layout decisions keyed to device model, idiom checks, or `isTablet`-style booleans
- A capability that exists only at one width class
- Ship a stretched phone layout on a tablet, or a shrunken tablet layout on a TV
- Testing only the configuration you built in
