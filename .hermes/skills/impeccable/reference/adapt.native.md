> **Additional context needed**: target platforms/devices and usage contexts.

Adapt an existing **native** design (`ios` / `android` / `adaptive`) to a different context: another device class, orientation, platform, or origin. The trap is treating adaptation as scaling. The job is rethinking the experience for the new context, inside the platform conventions of [ios.md](ios.md) / [android.md](android.md); read the target platform's reference before planning if Setup hasn't already. When the adaptation crosses device classes, [form-factors.md](form-factors.md) is the shared size-class model, and [watch.md](watch.md) / [tv.md](tv.md) carry the target's own rules and slop test.

## Assess Adaptation Challenge

1. **Source context**: what was it designed for, and what assumptions did it make? (Phone-only? Portrait-only? One platform's idioms? A website?)
2. **Target context**: which device class (phone, tablet, watch, tv, foldable), orientation, platform, and usage posture (one-handed on the go vs two-handed at rest)?
3. **What breaks**: navigation that doesn't fit the target, layouts that stretch instead of restructure, gestures or controls that don't exist there (a watch has no room for stacked navigation; a TV has no touch at all)?

## Adaptation Strategies

### Phone → Tablet (iPad / large screens)

- **Restructure, don't stretch.** A scaled-up phone UI on a tablet is the failure mode. Use size classes (iOS) / window size classes (Android) to switch structure.
- **Navigation changes shape**: tab bar stays or becomes a sidebar on iPad; Android navigation bar becomes a rail or drawer on expanded width.
- **Use the width**: split view / master-detail (list + detail side by side), multi-column grids, popovers where phones used sheets.
- **Multitasking is a size, not an edge case**: iPad Split View and Android multi-window can hand you a phone-width window on a tablet; size-class-driven layout handles both for free.

### Phone → Watch (companion apps especially)

- **Extract the job, don't shrink the screen.** A watch surface answers one question or does one job; pick the task worth having on the wrist and rebuild it list-first; [watch.md](watch.md)'s glance test is the acceptance bar.
- **Depth collapses to two screens.** Replace stacks and tab bars with a vertical page or swipe navigation; primary action reachable in one tap from the entry screen.
- **Input changes hands**: crown/bezel scrolls, swipes dismiss, voice replaces keyboards. Anything requiring precision taps or text entry stays on the phone.

### Phone / handheld → TV (ten-foot UI)

- **Focus replaces touch.** Every element becomes focusable with unmistakable focused state; arrow-key traversal must be predictable with no dead ends; [tv.md](tv.md)'s ten-foot test is the acceptance bar.
- **Density drops an order of magnitude.** One primary action per screen, short labels, large type; a phone grid becomes rows and shelves at the platform's grid specs, inside the overscan insets.
- **Media keys and voice** replace scroll-and-tap patterns; text entry is a last resort, not a primary path.

### Orientation & foldables

- Landscape restructures (side-by-side panes, repositioned controls); never clip or letterbox. Lock orientation only when the task truly demands it.
- Foldables (Android): react to posture and hinge via window size classes; test folded, unfolded, and tabletop.

### Platform → platform (iOS ↔ Android)

Translate idioms; never transplant them:

| iOS | Android |
|---|---|
| Tab bar | Navigation bar / rail / drawer |
| Edge-swipe back, back chevron | Predictive Back gesture / button |
| Switch, segmented control, system pickers | Material switch, chips, Material pickers |
| Action sheet | Bottom sheet / Material dialog |
| SF Symbols, SF Pro, Dynamic Type | Material Symbols, Roboto, sp scaling |
| Semantic system colors, materials | Material color roles, tonal elevation |
| System push/sheet transitions | Container transform, shared-axis, fade-through |

Rebuild navigation and controls in the target's vocabulary; carry over the brand's expressive layer (palette intent, type accent, motion personality) through the target's theming system.

### Web → native (porting a website or web app)

Reconform, don't reflow. Replace web navigation with the platform's model, HTML-shaped controls with platform controls, hover affordances with touch-first ones, and px-based type with Dynamic Type / sp. Then treat the result to the full platform reference; the slop test there is the acceptance bar.

## Implement & Verify

- Drive structure from **size classes / window size classes**, never from device-model checks.
- Respect safe areas and window insets in every new configuration (notch, hinge, status bar, keyboard, overscan).
- Test on simulators for breadth, then real hardware for truth: at least one phone and one tablet per shipped platform, both orientations, split-screen where supported; every declared watch size or TV device captured per its reference's verification section.

When the adaptation feels native to each context, hand off to `/impeccable polish` for the final pass.

**Avoid**:
- Ship a stretched phone layout on a tablet
- Port one platform's controls or navigation onto the other
- Hide core functionality on smaller devices (if it matters, make it work)
- Lock orientation to dodge a layout bug
- Trust simulators alone (posture, gestures, and performance need hardware)
