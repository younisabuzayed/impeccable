# React Native framework

Cross-platform declarative UI in JavaScript/TypeScript: React Native and Expo shipping to iOS and Android from one codebase (tvOS via the community forks). Load when PRODUCT.md's Stack names React Native or Expo; read with [ios.md](ios.md) and [android.md](android.md), which stay the design-language authority per platform, plus [form-factors.md](form-factors.md) and the device references where declared. React Native's trap is "one app that looks the same everywhere": the platform references' slop tests still apply per OS. <!-- rule:rn-crossplatform-stance -->

## Size from the live window, never a snapshot

- `useWindowDimensions()` is the size-class source: it updates on rotation, split screen, and fold posture, and carries `fontScale`. `Dimensions.get()` returns a snapshot that goes stale the moment the window changes; reserve it for one-shot constants outside React. <!-- rule:rn-use-window-dimensions -->
- There are no built-in size classes. Map `width` from `useWindowDimensions()` onto the [ladder](form-factors.md) (600 / 840 / 1200 / 1600 dp) in one shared `useBreakpoint()` hook, and branch structure on it. `Platform.OS` / `Platform.select` decide idiom differences (back button, haptics, control skins), never layout structure. <!-- rule:rn-breakpoint-hook -->

## Canonical layouts in React Native terms

- **List-detail:** a `flexDirection: 'row'` two-pane at expanded width with one pane rendered at compact width, keeping both panes' state in a common parent; or a split-capable navigator (react-navigation's drawer + nested stacks) whose expanded form shows master and detail side by side. The pane switch keys off the breakpoint hook. <!-- rule:rn-list-detail -->
- **Feed:** `FlatList`/`SectionList` with `numColumns` derived from the window width (aim ≈180 dp minimum per cell) and `columnWrapperStyle` for gutters; it degrades to a single column at compact width. Virtualization is not optional: a `ScrollView` of `.map()`-generated rows is the audit's unvirtualized-list finding. <!-- rule:rn-feed-numcolumns -->
- **Insets:** `SafeAreaView` from `react-native-safe-area-context` (the core `SafeAreaView` is iOS-only and deprecated in practice); consume insets per edge so content clears notches, system bars, and the keyboard in every configuration. <!-- rule:rn-safe-area-context -->

## Platform conformance per OS

- **System controls, not branded clones:** render the platform's controls (native stack headers, system switches and pickers via community bindings or hoisted native views); a Bootstrap-styled button in a native app is the web-shaped control the platform references ban. Touch targets stay 44 pt (iOS) / 48 dp (Android). <!-- rule:rn-system-controls -->
- **Back must work both ways:** Android's predictive Back and hardware back via `BackHandler` and native-stack integration; iOS edge-swipe via native-stack gesture enabled. Never build a JS-only back button as the only path. <!-- rule:rn-back-paths -->
- **Type follows the user:** `allowFontScaling` stays on, sizes map to Dynamic Type / sp buckets (read `fontScale` from `useWindowDimensions()` for custom scales), and nothing clips at the largest accessibility size. <!-- rule:rn-dynamic-type -->

## TV and watch

- **tvOS:** the react-native-tvos fork ships the focus engine ( `isFocusable`, focusable `Pressable` variants, Apple TV parallax); keep [tv.md](tv.md)'s rules: visible focus, predictable D-pad traversal, overscan insets. <!-- rule:rn-tvos-fork -->
- **watchOS:** there is no official React Native watch target; build the companion as a native SwiftUI watch app and follow [watch.md](watch.md). <!-- rule:rn-watch-native -->

## Anti-patterns

- Pixel-identical UI on both platforms ("write once, look the same") as a goal; each platform owes its users its own navigation, gestures, and controls. <!-- rule:rn-no-lookalike-goal -->
- `Dimensions.get('window')` in render, fixed pixel widths sized to one phone, and styles that ignore `fontScale`. <!-- rule:rn-no-dimensions-in-render -->

## Verifying the build

Evidence comes from the platform captures in [ios.md](ios.md) and [android.md](android.md), both appearances, a raised font scale, and a size change: rotate and enter split screen so `useWindowDimensions` proves it restructures, not rescales. Name which class each capture ran at.
