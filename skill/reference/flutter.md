# Flutter framework

Cross-platform declarative UI in Dart: Flutter shipping to iOS and Android (plus web and desktop where declared). Load when PRODUCT.md's Stack names Flutter; read with [ios.md](ios.md) and [android.md](android.md), which stay the design-language authority per platform, plus [form-factors.md](form-factors.md) and the device references where declared. Flutter renders its own pixels, so platform conformance is a choice the code must make explicitly: the OS hands you nothing for free. <!-- rule:flutter-renders-own-pixels -->

## Size from constraints and MediaQuery, read cheaply

- `MediaQuery.sizeOf(context)` (not `MediaQuery.of(context).size`) subscribes to size changes without rebuilding on every metric; `LayoutBuilder` answers "what fits *here*" for a subtree. There are no built-in size classes: map width onto the [ladder](form-factors.md) (600 / 840 / 1200 / 1600 dp) in one shared helper and branch structure on it. <!-- rule:flutter-sizeof-layoutbuilder -->
- `MediaQuery.textScalerOf(context)` carries the user's text scale; every custom size multiplies through it or uses `TextTheme` roles so text follows the OS setting. Hard-coded `fontSize: 14` with `textScaler: NoScaling` is the audit's text-scaling finding. <!-- rule:flutter-text-scaler -->

## Canonical layouts, Material-adaptive

- **Navigation changes shape:** `NavigationBar` at compact width becomes `NavigationRail` (or a drawer) at expanded width, same destinations, switched from the breakpoint helper; `AdaptiveLayout`-style branches stay in one widget tree, not two screens. <!-- rule:flutter-nav-bar-rail -->
- **List-detail:** a `Row` of two panes at expanded width, one pane below it at compact, with the selected state lifted to a shared controller; `GridView` with `SliverGridDelegateWithMaxCrossAxisExtent(maxCrossAxisExtent: 180)` gives the [feed](form-factors.md) pattern and degrades to a single column. <!-- rule:flutter-list-detail-grid -->
- **Insets and posture:** `SafeArea` (or `MediaQuery.paddingOf`) keeps content clear of notches and system bars; `MediaQuery.displayFeaturesOf(context)` exposes hinges and folds, so table-top postures split content instead of hiding it under the crease. <!-- rule:flutter-safearea-displayfeatures -->

## Platform conformance is explicit

- **Adaptive widgets where the platform differs:** `.adaptive` constructors (`Switch.adaptive`, `Slider.adaptive`, `SwitchListTile.adaptive`) and platform-appropriate transitions; a Material chrome surprise on iOS or Cupertino chrome on Android without the brief asking for it is the off-platform-control slop both platform references name. An app that ships Material everywhere still owes each OS its guarantees: safe areas, Back behavior, Reduce Motion, text scaling. <!-- rule:flutter-adaptive-widgets -->
- **Back and gestures:** predictive Back on Android (`PopScope` may defer but never trap), edge-swipe back intact on iOS, and the scrolling feel per platform (`BouncingScrollPhysics` vs material physics) via `ScrollBehavior`. <!-- rule:flutter-back-gestures -->

## Watch and TV are not Flutter targets

Flutter has no official watchOS or tvOS deliverable. Keep those surfaces native: a SwiftUI watch app per [watch.md](watch.md), a SwiftUI or tv-material3 TV app per [tv.md](tv.md), and let the Flutter app stay where Flutter is strong. <!-- rule:flutter-native-companions -->

## Anti-patterns

- `MediaQuery.of(context)` sprinkled through build methods (rebuild storms the profiler attributes to whole subtrees) and `EdgeInsets.all` layouts that ignore insets. <!-- rule:flutter-no-of-in-build -->
- Fixed-width screens from one emulator, and `SizedBox`-driven type that fights `textScaler` at large accessibility sizes. <!-- rule:flutter-no-fixed-widths -->

## Verifying the build

Evidence comes from the platform captures in [ios.md](ios.md) and [android.md](android.md), both appearances, `textScaleFactor` raised (`adb shell settings put system font_scale 1.3`, restore after), and a size change: rotate, split screen, and a fold posture so the breakpoint branches prove they restructure. Name which class each capture ran at.
