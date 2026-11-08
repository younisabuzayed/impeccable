# Jetpack Compose framework

The declarative way to build Android surfaces: phones, tablets, foldables, Wear OS, and TV. Load when PRODUCT.md's Stack names Compose and the platform is `android` or `adaptive`; read with [android.md](android.md) (design language) and, where declared, [form-factors.md](form-factors.md), [watch.md](watch.md), [tv.md](tv.md). Compose's adaptive libraries make form-factor response a composition decision, not a device check.

## Size classes drive structure

- `currentWindowAdaptiveInfo(supportLargeAndXLargeWidth = true)` (`androidx.compose.material3.adaptive`) yields the window's width/height classes: the [ladder](form-factors.md) at 600/840/1200/1600 dp width, 480/900 dp height. Branch structure on it; never on screen size, density, or an `isTablet` boolean.
- Helpers like `isWidthAtLeastBreakpoint(WindowSizeClass.WIDTH_DP_EXPANDED_LOWER_BOUND)` read intent-first; `BoxWithConstraints` delays composition and answers a question the class already answered; reserve it for genuinely constraint-dependent drawing.

## Canonical layouts, off the shelf

- **List-detail:** `ListDetailPaneScaffold` + `rememberListDetailPaneScaffoldNavigator()` (`listPane` / `detailPane`; opt-in `ExperimentalMaterial3AdaptiveApi`). Both panes at expanded width, one at a time below it, `BackHandler` when only detail is visible. State survives size changes if pane state is remembered (`rememberSaveable`) with the navigator's `scaffoldDirective`/`scaffoldValue`.
- **Supporting pane:** `SupportingPaneScaffold` + `rememberSupportingPaneScaffoldNavigator()`, 50/50 at medium, 70/30 at expanded, stacked or bottom-sheeted at compact.
- **Feed:** `LazyVerticalGrid(columns = GridCells.Adaptive(minSize = 180.dp))` with `maxLineSpan` for full-width items; it degrades to `LazyColumn` behavior at compact width.

## Navigation that changes shape

`NavigationSuiteScaffold` (`material3-adaptive-navigation-suite`) renders bar, rail, or drawer by the default rule (bar when width or height is compact or tabletop posture, rail otherwise), with a `layoutType` override for expanded-width drawers. The destinations are one list; the container adapts.

## State & performance

- `collectAsStateWithLifecycle`, `rememberSaveable` across configuration changes, stable model classes (`@Immutable`), and `key` in lazy-item lambdas keep recomposition honest; the differences a large screen exposes are recomposition storms written on a phone.
- Insets: edge-to-edge is the default posture; consume `WindowInsets` via Material scaffolds' padding rather than manual `statusBarPadding` patches.

## Wear and TV, in Compose

- **Wear OS:** `ScreenScaffold`/`AppScaffold` + `TimeText`, `TransformingLazyColumn` for scaling lists, `SwipeDismissableNavHost` for navigation, `SwipeToReveal` for actions; see [watch.md](watch.md).
- **TV:** `androidx.tv:tv-material3` `Surface` gives focus border/glow/scale; `Modifier.focusGroup()`, `focusRestorer()`, and `FocusRequester` keep D-pad traversal sane; see [tv.md](tv.md).

## Anti-patterns

- Fixed-dp widths sized to one phone (`width = 360.dp`) that shatter on an expanded window; constrain to a max content width instead.
- Recreating pane scaffolds by hand (a `Row` with two `LazyColumn`s and a hope) when the adaptive scaffolds encode the size-change behavior.

## Verifying the build

Compose `@Preview` specifications across compact/medium/expanded widths catch structure cheaply; shipped evidence still comes from the emulator captures in [android.md](android.md) (and [watch.md](watch.md) / [tv.md](tv.md) where declared): every declared form factor, dark theme, a raised font scale. Name which class each capture ran at.
