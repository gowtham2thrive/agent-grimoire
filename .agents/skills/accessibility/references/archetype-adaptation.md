# Universal Archetype Adaptation Matrix (Accessibility)

Translate universal accessibility moves directly into idiomatic primitives for your target technology:

```
Universal Accessibility Move:
"Implement an interactive disclosure dialog that traps focus, announces its title on open, dismisses on Escape, and restores focus to the triggering element."
```

| Technology Platform | Native Primitive & Semantics | Keyboard & Focus Trapping | Live Annunciation & State | Reflow & Forced Colors |
| :--- | :--- | :--- | :--- | :--- |
| **Modern Web** (HTML / React / Vue) | Native `<dialog>` or `role="dialog"` + `aria-labelledby` | Focus trap loop; listen `keydown` for `Escape` | `aria-modal="true"`; `aria-live="polite"` for dynamic updates | `rem` font units; `@media (forced-colors: active)` border styling |
| **Mobile iOS** (SwiftUI / UIKit) | `.accessibilityElement()`, `.accessibilityAddTraits(.isModal)` | VoiceOver rotor order; `.accessibilityFocused($isFocused)` | `UIAccessibility.post(notification: .screenChanged, ...)` | Dynamic Type (`@ScaledMetric`); Reduce Motion query |
| **Mobile Android** (Compose / Views) | `Modifier.semantics { heading(); paneTitle = "..." }` | `FocusRequester`; `Modifier.focusProperties()` | `Modifier.liveRegion(LiveRegionMode.Polite)` | Sp font units; `LocalDensity.current` reflow |
| **Desktop Native** (WinUI / macOS) | `AutomationPeer` (WinUI) / `NSAccessibilityProtocol` | Window modal pump; UIA modal pattern; Tab sequence | UIA `LiveSetting.Polite`; `NSAccessibilityPostNotification` | Windows High Contrast brushes; macOS display scaling |
| **Terminal TUI / CLI** (Rust / Go / Python) | High-contrast ANSI brackets `[ > Button < ]`, clear text | Direct key bindings (`Tab`, `Esc`); explicit cursor cell | Status bar text banner; optional terminal bell `\a` | SIGWINCH terminal resize; fallback `--plain` stream mode |
| **Canvas / WebGL / Spatial** (2D/3D / Games) | Parallel off-screen virtual accessibility tree | Virtual hit-test grid; arrow key spatial navigation | Synthesized speech / audio cue hook in game loop | Vector UI scaling; 1-click **Tabular Alternative Mode** |
| **AI / Agentic UI** (Generative UI / Chat) | Semantic card landmarks with level 2/3 headers | Keyboard cancel shortcut (`Esc` / `Cmd+.`); focus on prompt | Debounced lifecycle announcements; zero per-token spam | Scalable container flex layout; responsive prompt bar |
