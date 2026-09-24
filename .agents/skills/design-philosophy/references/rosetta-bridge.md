# The Rosetta Translation Bridge

> **Mandate**: UI design philosophy must remain completely technology-agnostic in its principles, but razor-sharp and idiomatic in its technical execution. Use this Rosetta Bridge to translate universal visual moves into the native primitives of your target framework.

---

## 1. The Core Translation Matrix

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          THE ROSETTA TRANSLATION MAP                        │
│                                                                             │
│  Universal Principle ──► Idiomatic Implementation in:                       │
│  • Web (CSS / Tailwind)                                                     │
│  • Apple iOS / macOS (SwiftUI)                                              │
│  • Android (Jetpack Compose)                                                │
│  • Cross-Platform (Flutter)                                                 │
│  • Terminal / CLI (Rust Ratatui / Go Bubbletea)                             │
│  • 2D Canvas / Game HUD (HTML5 Canvas / Skia)                               │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Universal Move Translations

### Move 1 · Proximity Grouping (Replacing Nested Boxes with Negative Space)
*Universal Intent*: Group related elements using internal gutters and eliminate enclosing border boxes.

* **Web (Tailwind / CSS)**:
  ```html
  <!-- ✅ Clean proximity grouping without borders -->
  <div class="flex flex-col gap-2">
    <span class="text-xs font-semibold uppercase tracking-wider text-muted-foreground">Cluster Status</span>
    <span class="text-sm font-medium text-foreground">12 Nodes Online</span>
  </div>
  ```
* **iOS (SwiftUI)**:
  ```swift
  VStack(alignment: .leading, spacing: 8) {
      Text("CLUSTER STATUS")
          .font(.caption)
          .fontWeight(.semibold)
          .foregroundStyle(.secondary)
      Text("12 Nodes Online")
          .font(.subheadline)
          .fontWeight(.medium)
  }
  ```
* **Android (Jetpack Compose)**:
  ```kotlin
  Column(verticalArrangement = Arrangement.spacedBy(8.dp)) {
      Text(
          text = "CLUSTER STATUS",
          style = MaterialTheme.typography.labelSmall,
          color = MaterialTheme.colorScheme.onSurfaceVariant
      )
      Text(
          text = "12 Nodes Online",
          style = MaterialTheme.typography.bodyMedium
      )
  }
  ```
* **Cross-Platform (Flutter)**:
  ```dart
  Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    spacing: 8.0,
    children: [
      Text('CLUSTER STATUS', style: Theme.of(context).textTheme.labelSmall?.copyWith(
        color: Theme.of(context).colorScheme.outline,
      )),
      Text('12 Nodes Online', style: Theme.of(context).textTheme.bodyMedium),
    ],
  )
  ```
* **Terminal TUI (Rust Ratatui)**:
  ```rust
  let chunks = Layout::default()
      .direction(Direction::Vertical)
      .constraints([Constraint::Length(1), Constraint::Length(1)])
      .split(area);
  frame.render_widget(Paragraph::new("CLUSTER STATUS").style(Style::default().fg(Color::DarkGray)), chunks[0]);
  frame.render_widget(Paragraph::new("12 Nodes Online").style(Style::default().fg(Color::White)), chunks[1]);
  ```

---

### Move 2 · Tabular Numeric Alignment (Eliminating Horizontal Jitter)
*Universal Intent*: Render changing numbers and timers with monospaced/tabular figures so decimal points and columns align stably.

* **Web (CSS / Tailwind)**:
  ```css
  /* Vanilla CSS */
  .metric-value { font-variant-numeric: tabular-nums; }
  ```
  ```html
  <!-- Tailwind -->
  <span class="tabular-nums font-mono font-medium">$1,482.90</span>
  ```
* **iOS (SwiftUI)**:
  ```swift
  Text("$1,482.90")
      .monospacedDigit()
      .fontWeight(.medium)
  ```
* **Android (Jetpack Compose)**:
  ```kotlin
  Text(
      text = "$1,482.90",
      fontFamily = FontFamily.Monospace,
      fontWeight = FontWeight.Medium
  )
  ```
* **Cross-Platform (Flutter)**:
  ```dart
  Text(
    '\$1,482.90',
    style: TextStyle(
      fontFeatures: const [FontFeature.tabularFigures()],
      fontWeight: FontWeight.w500,
    ),
  )
  ```
* **Terminal TUI**:
  * Character cells are natively monospaced. Ensure format string right-aligns figures:
  ```rust
  format!("{:>10.2}", 1482.90);
  ```

---

### Move 3 · Fluid Reading Measure (Preventing Eye Tracking Fatigue)
*Universal Intent*: Constrain longform body text to 45–75 characters per line (`max-width: 65ch`) with centered margins.

* **Web (CSS / Tailwind)**:
  ```html
  <article class="mx-auto max-w-prose px-4 leading-relaxed">
    <p>Body prose flows comfortably...</p>
  </article>
  ```
* **iOS (SwiftUI)**:
  ```swift
  Text("Body prose flows comfortably...")
      .frame(maxWidth: 600, alignment: .leading)
      .lineSpacing(4)
      .padding(.horizontal)
  ```
* **Android (Jetpack Compose)**:
  ```kotlin
  Text(
      text = "Body prose flows comfortably...",
      modifier = Modifier.widthIn(max = 600.dp).padding(horizontal = 16.dp),
      lineHeight = 24.sp
  )
  ```
* **Cross-Platform (Flutter)**:
  ```dart
  ConstrainedBox(
    constraints: const BoxConstraints(maxWidth: 600),
    child: Text('Body prose flows comfortably...', style: TextStyle(height: 1.5)),
  )
  ```
* **Terminal TUI**:
  ```rust
  let prose_area = Layout::default()
      .constraints([Constraint::Max(70)])
      .split(area)[0];
  ```

---

### Move 4 · Accessible Hit Area Expansion (Fitts’s Law)
*Universal Intent*: Ensure small visual icons have a minimum physical interactive touch area of $\ge 44 \times 44\text{pt}$ / $9\text{mm}$.

* **Web (Tailwind / CSS)**:
  ```html
  <!-- Visual icon is 16px, hit area is 44px -->
  <button class="relative p-3 -m-3 inline-flex items-center justify-center">
    <svg class="w-4 h-4" ... />
  </button>
  ```
* **iOS (SwiftUI)**:
  ```swift
  Button(action: deleteItem) {
      Image(systemName: "trash")
          .frame(width: 44, height: 44) // Expands hit area
          .contentShape(Rectangle())
  }
  ```
* **Android (Jetpack Compose)**:
  ```kotlin
  IconButton(
      onClick = { deleteItem() },
      modifier = Modifier.minimumInteractiveComponentSize() // Enforces 48dp minimum
  ) {
      Icon(Icons.Default.Delete, contentDescription = "Delete Item")
  }
  ```
* **Cross-Platform (Flutter)**:
  ```dart
  IconButton(
    icon: const Icon(Icons.delete, size: 18),
    constraints: const BoxConstraints(minWidth: 44, minHeight: 44),
    onPressed: deleteItem,
  )
  ```

---

### Move 5 · Surface Elevation in Dark Mode (Lightness Step-Up)
*Universal Intent*: Elevate floating layers by increasing background lightness, not by casting invisible black shadows.

* **Web (CSS Variables)**:
  ```css
  :root[data-theme="dark"] {
    --bg-ground:   #0d0e12;
    --bg-card:     #16181f; /* Step 1 */
    --bg-popover:  #21242d; /* Step 2 */
    --bg-overlay:  #2b303c; /* Step 3 */
  }
  ```
* **iOS (SwiftUI)**:
  ```swift
  // Uses system semantic elevation materials natively
  .background(.ultraThinMaterial)
  .background(Color(uiColor: .secondarySystemBackground))
  ```
* **Android (Jetpack Compose)**:
  ```kotlin
  // Uses tonal elevation to automatically lighten the surface
  Surface(
      tonalElevation = 4.dp,
      shape = RoundedCornerShape(8.dp)
  ) { ... }
  ```
