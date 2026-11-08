# Android Views framework

The imperative way to build Android surfaces: XML layouts, View classes, and their Wear OS/TV libraries. Load when PRODUCT.md's Stack names Views (or is silent and the codebase is XML/View-based) and the platform is `android` or `adaptive`; read with [android.md](android.md) (design language) and, where declared, [form-factors.md](form-factors.md), [watch.md](watch.md), [tv.md](tv.md). Imperative adaptivity lives in two places: **resource qualifiers** (the system picks the right layout) and **runtime window-class branches** (the code picks the right structure). <!-- rule:views-two-mechanisms -->

## Resource qualifiers: the system's own size classes

- `layout-sw600dp/` is the tablet breakpoint (smallest width ≥600 dp, orientation-independent); `layout-w600dp/` branches on available width so multi-window gets the compact layout; `layout-land/` only for genuinely landscape-specific structure. Legacy `large`/`xlarge` buckets are retired; smallest-width replaced them. <!-- rule:views-sw600dp-qualifier -->
- Every qualifier variant must stay a faithful superset of the base layout: same IDs, same functionality, same state handling. A `sw600dp` layout that drops features is a defect, not a variant. <!-- rule:views-variant-parity -->

## Runtime branches: window size classes in code

- `WindowSizeClass.compute(width / density, height / density)` (Jetpack WindowManager) fed by `WindowMetricsCalculator.getOrCreate().computeCurrentWindowMetrics(activity)` gives the same [ladder](form-factors.md) Compose reads. Branch *structure* on it (swapping fragments, showing two panes), not pixel math. <!-- rule:views-window-size-class-compute -->
- Re-compute on configuration changes and multi-window resizes (`onConfigurationChanged`, `WindowLayoutInfo` for folds); a layout computed once at launch breaks the moment the window shrinks. <!-- rule:views-recompute-on-resize -->

## Canonical structure, View-style

- **List-detail:** two panes at `sw600dp`/expanded width: `SlidingPaneLayout` for swipeable list+detail, or fragment dual-pane with shared `ViewModel` state; single-pane + navigation below the breakpoint. <!-- rule:views-list-detail-panes -->
- **Feed:** `RecyclerView` + `GridLayoutManager` with span count derived from available width (aim ≈180 dp minimum per cell) and `SpanSizeLookup` for full-span headers; it must degrade to a 1-column list, not stretch rows. <!-- rule:views-grid-span-count -->
- **ConstraintLayout** carries responsive structure: percent constraints, `Barrier`, `Group`, and `Guideline`, instead of nested `LinearLayout` chains that snap at one width. <!-- rule:views-constraintlayout-percent -->

## Insets, folds, and edge-to-edge

Apply window insets with `ViewCompat.setOnApplyWindowInsetsListener` so content clears status/nav bars, cutouts, and IME in every configuration, the edge-to-edge rule of [android.md](android.md) in imperative form. Fold posture arrives via `WindowLayoutInfo`/hinge bounds; keep content out of the hinge. <!-- rule:views-insets-listener -->

## Wear OS and TV, in Views

- **Wear OS:** `BoxInsetLayout` for round-safe margins, `WearableRecyclerView` + `WearableLinearLayoutManager` (curved scrolling), `SwipeDismissFrameLayout` for back; see [watch.md](watch.md). <!-- rule:views-wear-widgets -->
- **TV:** the Leanback library is deprecated: new TV UI is Compose for TV (`androidx.tv:tv-material3`); maintain legacy Leanback screens to the same [tv.md](tv.md) focus rules while migrating. <!-- rule:views-leanback-deprecated -->

## Anti-patterns

- Density- or pixel-based branching (`displayMetrics.densityDpi`, screen-inch math) standing in for size classes. <!-- rule:views-no-density-branches -->
- Fixed dp text that ignores font scale; text carries `sp`, never `dp`. <!-- rule:views-sp-for-text -->
- New Leanback UI, and `layout-land` copies that drift out of sync with their base layout. <!-- rule:views-no-leanback-new -->

## Verifying the build

Install on a tablet AVD (or a phone at `adb shell wm size`-changed dimensions) and on the phone: both must exercise their qualifier variants; evidence from the captures in [android.md](android.md), plus dark theme and `font_scale 1.3`. Name which class each capture ran at.
