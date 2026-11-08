# UIKit framework

The imperative way to build Apple surfaces: iOS, iPadOS, and tvOS. Load when PRODUCT.md's Stack names UIKit and the platform is `ios` or `adaptive`; read with [ios.md](ios.md) (design language) and, where declared, [form-factors.md](form-factors.md) and [tv.md](tv.md). Imperative adaptivity is event-driven: the system announces trait changes, the app re-resolves layout in response. Getting this right is discipline about *where* layout decisions live. <!-- rule:uikit-trait-driven -->

## React to traits, not to devices

- Read `traitCollection.horizontalSizeClass` (and `verticalSizeClass`): `UITraitCollection`, not `UIDevice`. Adopt `UITraitChangeObservable` (`traitCollectionDidChange(_:)` or the `UITraitChangeCallback` API) and re-resolve layout there; `willTransition(to:with:)` covers coordinated rotations. <!-- rule:uikit-trait-change-observable -->
- One constraint set per class, activated/deactivated on change, beats one constraint set wrestled to fit all classes. Keep the swap in the view controller that owns the layout, and store active constraints so the swap is cheap and reversible. <!-- rule:uikit-constraint-sets -->

## Layout anchors that adapt for you

- Constraint to `safeAreaLayoutGuide` and `readableContentGuide`, not superview edges; insets, bars, and rounded corners then cost zero code. `UILayoutGuide` objects replace invisible spacer views. <!-- rule:uikit-layout-guides -->
- `UIScreen.main.bounds` is retired from layout decisions: read `view.window.windowScene` / trait context instead. Screens lie in multitasking and multi-scene apps. <!-- rule:uikit-no-screen-bounds -->

## Presentations that differ by idiom

- `UIAlertController.style(.actionSheet)` on regular width presents as a **popover and crashes without an anchor**: set `popoverPresentationController`'s `sourceView` + `sourceRect` (or `barButtonItem`) before presenting; at compact width it is a bottom sheet. `.alert` presents identically everywhere. <!-- rule:uikit-actionsheet-popover-anchor -->
- `UISplitViewController` gives [list-detail](form-factors.md) for free: primary collapses to a stack at compact width. `UISheetPresentationController` detents for partial-height sheets. <!-- rule:uikit-split-controller -->

## tvOS, in UIKit

The focus engine replaces touch: make views focusable, keep focus movement predictable, and use `UIFocusGuide` (an `isEnabled` layout guide) to bridge rows whose items don't align; see [tv.md](tv.md). `preferredFocusEnvironments` decides where focus lands on entry. <!-- rule:uikit-tvos-focus-engine -->

## Interop & anti-patterns

- Mixing SwiftUI into UIKit: `UIHostingConfiguration` for cells, `UIHostingController` for surfaces, a good path for incremental adoption, not a license to skip trait handling in the remaining UIKit. <!-- rule:uikit-hosting-interop -->
- Layout decisions in `viewDidLayoutSubviews` by arithmetic (`x = bounds.width * 0.5 …`) when constraints or size classes would express the intent; it's the imperative GeometryReader. Reserve manual math for genuinely measured drawing. <!-- rule:uikit-no-arithmetic-layout -->
- Device checks (`userInterfaceIdiom`, model names) for layout. Idiom only where behavior truly differs (e.g., pointer interactions); layout always keys to traits. <!-- rule:uikit-no-device-checks -->

## Verifying the build

Exercise the trait changes, not just the launch state: rotate, enter Split View/Stage Manager on iPad (or resize the window), and confirm the constraint swap fires and reverts; popover anchoring on iPad is the classic crash. Evidence comes from the simulator captures in [ios.md](ios.md) (and [tv.md](tv.md) where declared): every declared form factor, both appearances, a large Dynamic Type size.
