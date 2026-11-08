# TV form factor

For TV apps: tvOS (SwiftUI or UIKit, both fully supported) and Android TV (Jetpack Compose with `androidx.tv:tv-material3`; the Leanback library is deprecated, migrate to Compose for TV). Load alongside [ios.md](ios.md) or [android.md](android.md); the platform's design language still governs theming and color; this file adds what the ten-foot distance and the D-pad change. See also [form-factors.md](form-factors.md).

## The ten-foot test (the TV slop test)

Could a user across the room, on a couch, operate every screen with only the remote? The tell is ported handheld UI: hover-dependent affordances, tap-sized targets, dense tables, thin type, elements only a touch screen could reach. If anything needs a pointer, a keyboard, or a squint, redesign it.

## Focus is the interface

- **Every interactive element is focusable**, and focus is always visible: tvOS scales and brightens the focused element; Android TV `Surface` draws focus border/glow/scale. An unfocusable button is a broken button.
- **Focus movement is predictable.** Arrow keys must move focus in the direction they imply, with no dead ends and no traps. In Compose use `Modifier.focusGroup()`, `Modifier.focusRestorer()`, and `FocusRequester` for sensible entry points; on tvOS use `focusSection()`, `@FocusState`, and `defaultFocus(_:_:priority:)`.
- **Full-width focus guides** (`UIFocusGuide` on tvOS) route vertical movement between rows whose items don't align into columns; the user's intuition is "down", not "find the diagonally adjacent card".
- **Press feedback is immediate and legible**: the focused state is unmistakable from at least three meters, in both appearances.

## Layout for the living room

- **Overscan-safe areas.** tvOS: inset primary content **60 pt from top and bottom, 80 pt from the sides**. Android TV: keep the same margins in spirit: content hugging screen edges gets cropped or hidden on real panels.
- **Grids with room to breathe.** tvOS full-width columns at 2-col 860 pt · 3-col 560 pt · 4-col 410 pt · 5-col 320 pt · 6-col 260 pt · 7-col 217 pt · 8-col 184 pt · 9-col 160 pt, with **40 pt minimum horizontal spacing and ~100 pt vertical spacing** between rows; focused elements grow, so rows need the headroom. No dense tables, no 12-column web grids.
- **Big, bold, sparse.** Large type, short labels, one primary action per screen. Thin weights and long copy fail at distance.
- **Row-and-item browsing** (cards in horizontal shelves) is the native idiom for content catalogs; keep shelves a consistent height and item aspect so focus glide stays smooth.

## Input beyond the D-pad

Play/pause and media keys, voice search where the platform offers it, and game controllers must not fight the focus system: focus follows whatever input source drives it. Never require a text field for primary navigation; text entry is last resort (dictation or on-screen keyboard).

## Verifying the build

- **tvOS:** Apple TV simulator captured with `xcrun simctl io booted screenshot <path>`, same flow as [ios.md](ios.md). **Android TV:** a TV AVD (leanback system image) captured with `adb exec-out screencap -p > <path>`, picking the device with `adb -s <serial>` when several are attached.
- **Drive through without a pointer:** walk every screen with arrow keys only (the emulator maps arrows to the D-pad); any unreachable element or dead-end focus is a P1.
- Check both appearances and the overscan margins at the screen edges on every capture.

**Avoid**:
- Hover states, cursors, or anything pointer-only
- Focus dead ends or jumps that fight the arrow-key direction
- Content or controls under the overscan margins
- Dense text and thin weights readable only up close
- Shipping Leanback UI in a new Android TV build
