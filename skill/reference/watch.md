# Watch form factor

For watch apps: watchOS (SwiftUI is the only path for new apps; WatchKit storyboards are deprecated) and Wear OS (Jetpack Compose with `androidx.wear.compose:compose-material3`, or the View system's `androidx.wear.widget`). Load alongside [ios.md](ios.md) or [android.md](android.md); the platform's design language still governs theming, type, and color; this file adds what the wrist changes. See also [form-factors.md](form-factors.md).

## The glance test (the watch slop test)

Would a user get the point in a two-second wrist raise, or must they study the screen? The tell is a ported phone app: stacked navigation, settings-form screens, text paragraphs, controls clustered tighter than a fingertip. Every screen answers one question or does one job; anything deeper belongs in the phone or tablet app. <!-- rule:watch-glance-test -->

## Structure & layout

- **Depth of two screens, maximum.** One primary screen per task plus one detail at most. `TabView` with `.tabViewStyle(.verticalPage)` (watchOS) or `SwipeDismissableNavHost` (Wear OS) for paging; swipe back must always dismiss. <!-- rule:watch-depth-two -->
- **List-first.** Vertical scrolling lists are the backbone: `TransformingLazyColumn` (Wear OS Material 3, scaling and morphing rows) or SwiftUI `List`. Design for the Digital Crown / bezel as the primary scroll driver; scrolling must never depend on touching the screen. <!-- rule:watch-list-first -->
- **Circular-safe layout.** Round displays clip corners: keep content inside the safe margins (`scenePadding` on SwiftUI, `BoxInsetLayout` on the Wear View system). Square and rectangular sizes must stay first-class, not an afterthought. <!-- rule:watch-circular-safe -->
- **At most three glyph buttons, or two text buttons, in a row.** Controls are large (44 pt+ targets), few, and thumb-reachable. No grids of tiny actions, no hover, no drag-to-reveal essentials. <!-- rule:watch-button-density -->

## Screen sizes

Design across the full range, not one hero device. Layout dimensions in points: 40 mm 324×394 · 41 mm 352×430 · 42 mm (Series 10/11) 374×446 · 44 mm 368×448 · 45 mm 396×484 · 46 mm 402×474 · 49 mm (Ultra) 410×502. An app letterboxed on newer sizes was built against an old SDK; build against the current one. Default Dynamic Type differs by size (Large on the smaller cases, XLarge on the larger); nothing may clip at either. <!-- rule:watch-screen-sizes -->

## Typography & color

- **System text styles / sp roles only**, so text follows the user's size setting. Short labels, never paragraphs; numbers and status big, support text minimal. <!-- rule:watch-typo-system -->
- **True black background** (OLED wrist battery), semantic system colors, and full contrast in both appearances. <!-- rule:watch-dark-oled -->
- **Always-on / ambient is a mode, not a screenshot.** With `isLuminanceReduced` (watchOS) or ambient mode (Wear OS), drop to a dimmed palette, remove full-field bright areas and animation, and show only time-relevant data; return to full color on wrist raise. <!-- rule:watch-alwayson-restraint -->

## Beyond the app screen

Most watch usage never opens the app: complications, tiles, and notifications are the product's primary surface. Design them first: glanceable data, one action each, on-platform styles. <!-- rule:watch-complications-first -->

## Verifying the build

- **watchOS:** watch simulators via the same `xcrun simctl io booted screenshot <path>` flow as [ios.md](ios.md) (`xcrun simctl list devices` lists them). Capture the smallest and largest supported sizes plus Ultra, and check a circular and a rectangular device. Flip appearance and a large Dynamic Type size in the pass.
- **Wear OS:** round and square Wear AVDs, captured with `adb exec-out screencap -p > <path>`. Exercise ambient mode and a raised font scale (`adb shell settings put system font_scale 1.3`, restore after).
- **Hardware truth:** wrist raise/lower for always-on behavior, crown/bezel feel, and real-gesture timing need the watch on a wrist, not the simulator. <!-- rule:watch-verify-captures -->

**Avoid**:
- A phone screen shrunk to the wrist
- Reading-time content (paragraphs, tables, settings forms)
- Scroll or actions that require two-handed precision
- Ignoring always-on/ambient until it burns the screen in
- One hero watch size with everything else "close enough"
