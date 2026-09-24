# Spatial Geometry & Typographic Architecture

> **Mandate**: Typography and spatial geometry are the twin pillars of software comprehension. Never pick arbitrary numbers or eyeball visual relationships. Establish mathematical and optical consistency across layout intervals and typographic scales.

---

## 1. The Physics of Negative Space

Negative space (white space) is not empty void; it is the structural canvas that defines reading cadence, visual grouping, and cognitive focus.

### 1.1 The Spacing Scale Ladder
Base all spatial intervals on integer multiples of a single base unit $U$ ($4\text{px}$ for compact tools, $8\text{px}$ for balanced/touch systems):

| Step | Multiplier | $U = 4\text{px}$ (Compact Cockpit) | $U = 8\text{px}$ (Standard / Touch) | Semantic Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **`3xs`** | $0.25U$ | $1\text{px}$ | $2\text{px}$ | Hairline borders, micro-offsets |
| **`2xs`** | $0.5U$ | $2\text{px}$ | $4\text{px}$ | Tight badge padding, icon-text gap |
| **`xs`** | $1U$ | $4\text{px}$ | $8\text{px}$ | Compact component padding, label-to-input gap |
| **`sm`** | $1.5U$ | $6\text{px}$ | $12\text{px}$ | Sibling button gap, item lists |
| **`md`** | $2U$ | $8\text{px}$ | $16\text{px}$ | Standard container padding, form field gap |
| **`lg`** | $3U$ | $12\text{px}$ | $24\text{px}$ | Section gutters, card cluster spacing |
| **`xl`** | $4U$ | $16\text{px}$ | $32\text{px}$ | Screen margins, major module separation |
| **`2xl`** | $6U$ | $24\text{px}$ | $48\text{px}$ | Landing section headers, hero gutters |
| **`3xl`** | $8U$ | $32\text{px}$ | $64\text{px}$ | Landmark view transitions, editorial breathing room |

*Zero Magic Numbers*: Never use uncalibrated numbers like `13px`, `19px`, or `27px`. Every interval must map cleanly to the scale ladder.

### 1.2 The Containment Tension Invariant
The internal padding ($P$) of any container must always be strictly less than the external margin ($M$) separating it from its neighboring sibling containers:

$$P_{\text{internal}} < M_{\text{external}}$$

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       CONTAINMENT TENSION INVARIANT                         │
│                                                                             │
│   ❌ COLLAPSED / BROKEN (P >= M):       ✅ TENSION INTACT (P < M):          │
│   ┌───────────────┐ ┌───────────────┐   ┌───────────────┐   ┌───────────────┐│
│   │ P = 24px      │ │ P = 24px      │   │ P = 16px      │   │ P = 16px      ││
│   │ Content A     │ │ Content B     │   │ Content A     │   │ Content B     ││
│   └───────────────┘ └───────────────┘   └───────────────┘   └───────────────┘│
│           M = 16px (Colliding!)                 M = 32px (Distinct!)        │
└─────────────────────────────────────────────────────────────────────────────┘
```

When $P \ge M$, the visual bounding box leaks into neighboring space, creating optical crowding and confusing Gestalt boundaries.

---

## 2. Typographic Architecture

Typography is 95% of digital interface design. An interface that masters typography rarely needs decorative graphics.

### 2.1 Modular Typographic Scales
Font sizes should not be guessed randomly. Generate them using musical/geometric ratios based on a root body size ($B = 14\text{px}$ or $16\text{px}$):

* **Minor Third ($1.200$)** — Ideal for high-density tools, dashboards, and enterprise cockpits:
  `10px, 12px, 14px (Base), 17px, 20px, 24px, 29px, 35px`
* **Major Third ($1.250$)** — Ideal for standard web, marketing, and mobile applications:
  `10px, 12px, 16px (Base), 20px, 25px, 31px, 39px, 49px`
* **Perfect Fourth ($1.333$)** — Ideal for bold, expressive editorial publications and Swiss posters:
  `9px, 12px, 16px (Base), 21px, 28px, 38px, 50px, 67px`

### 2.2 Hierarchy via Weight and Luminance Before Font Size
Amateur designers rely exclusively on gigantic font sizes to create hierarchy. Professional designers achieve hierarchy through **typographic weight and luminance contrast**:

```text
// ❌ AMATEUR: Everything is oversized and shouts at equal contrast
[48px Bold] PROJECT DASHBOARD
[24px Bold] Total Active Servers
[36px Bold] 1,428
[18px Bold] Last updated 2 minutes ago

// ✅ REFINED: Balanced, rhythmic, high scanability
[20px Semibold, 100% White]    Project Dashboard
[12px Medium, 65% Zinc, Caps]  TOTAL ACTIVE SERVERS
[28px Bold, 100% White]        1,428
[11px Regular, 45% Zinc]       Updated 2m ago · Singapore East
```

### 2.3 The Measure (Line Length) Invariant
The human eye fatigues when reading lines that are either too long (hard to track the return jump to the next line) or too short (frantic saccades):
* **Body Prose Measure**: Strictly **45 to 75 characters per line** (`max-width: 65ch`).
* **Micro-Copy & Tooltips**: Strictly **20 to 35 characters per line**.
* **Data Grids & Code**: Full bleed permitted only when columns align vertically.

### 2.4 Inverse Leading (Line Height) Mechanics
Line height must be inversely proportional to font size:
* **Display Titles ($> 32\text{px}$)**: Tight leading ($1.10 - 1.25\times$). Large letterforms have generous internal counter-spaces; wide leading makes multi-line titles fall apart.
* **Body Copy ($14 - 18\text{px}$)**: Generous leading ($1.45 - 1.65\times$). Ample vertical white space is required between lines so the eye does not accidentally skip or double-read lines.
* **Captions & Micro-Labels ($< 12\text{px}$)**: Standard leading ($1.3 - 1.4\times$).

### 2.5 Tabular & Monospaced Figures
Never display dynamic numbers (stock prices, timers, row counts, telemetry metrics) in proportional fonts:
* **The Failure**: In proportional fonts, the digit `1` is much narrower than the digit `8`. As numbers change, the text box jitters horizontally, creating visual instability.
* **The Fix**: Always enable **tabular figures** (`tabular-nums`, `FontFeature.tabularFigures()`, or dedicated monospaced fonts). Decimal points and dollar signs must stack with laser precision.
