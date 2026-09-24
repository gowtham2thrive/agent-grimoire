# Chromatic Semantics & Luminance Physics

> **Mandate**: Treat color as contextual, semantic, and hierarchical information—never as superficial decoration. Ground all chromatic decisions in human luminance perception, APCA contrast science, and strict operational state reservation.

---

## 1. The Physics of Vision: Luminance vs. Hue

The human retina contains approximately **120 million rods** (sensitive exclusively to lightness/luminance and motion) and only **6 million cones** (sensitive to red, green, and blue wavelengths).

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        RETINAL PHOTORECEPTOR BALANCE                        │
│                                                                             │
│   Rods (Luminance, Edges, Depth, Motion):   ████████████████████  (95%)     │
│   Cones (Hue, Color Saturation):             █                     (5%)     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### The First Invariant: Luminance First, Color Second
* The human brain resolves spatial boundaries, reading shapes, and optical hierarchy through **lightness contrast** before it processes hue.
* **The Grayscale Test**: Convert your interface entirely to grayscale. If the hierarchy, primary button, status differences, or text legibility collapse without color, the design is broken. Fix the luminance contrast and weight first; add color last.

---

## 2. The 60-30-10 Chromatic Rule

Never distribute colors equally across an interface. Use the classic architectural ratio to maintain visual calm:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            THE 60-30-10 DISTRIBUTION                        │
│                                                                             │
│   ██████████████████████████████████████   60% Dominant Neutral Ground      │
│   ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒                       30% Structural Medium            │
│   ░░░░░░                                   10% Intentional Accent           │
└─────────────────────────────────────────────────────────────────────────────┘
```

1. **60% Dominant Ground (Canvas & Backgrounds)**:
   - Sets the ambient environment. Neutral whites, soft creams, or deep charcoals.
   - Provides the peaceful surface on which all other elements exist.
2. **30% Structural Medium (Typography, Panels, Dividers)**:
   - Primary and secondary text, subtle card surfaces, sidebar chrome, header navigation.
   - High enough contrast to be effortlessly readable, neutral enough not to compete with interactive triggers.
3. **10% Intentional Accent (Interactive Affordances & State)**:
   - Primary action buttons, active tab indicators, focus rings, and critical telemetry badges.
   - If everything is colorful, nothing has prominence. Restraint gives the accent its electrical charge.

---

## 3. The "Blue Means Trust" Fallacy

Amateur design guides repeat naive pop-psychology tropes ("blue means trust, green means health, red means passion"). In real software engineering:
* **Color is Contextual & Domain-Specific**:
  - In Chinese and East Asian financial markets, **Red means price rise / financial gain**, and **Green means price drop / loss** (the exact opposite of Western markets).
  - In aviation and medical software, color is governed by strict functional standards (e.g. Amber is advisory caution, Red is immediate life-threatening alert).
  - In clinical software, Blue represents venous blood and Cyan represents pulse oximetry.
* **The Rule**: Anchor color meanings to domain conventions and user mental models, not universal pop-psychology.

---

## 4. Hue Reservation for Semantic State

Never contaminate semantic state colors with decorative accents:

| Semantic State | Reserved Hue Band | Permitted Usage | Strictly Forbidden Usage |
| :--- | :--- | :--- | :--- |
| **Destructive / Error** | Crimson / Red ($0^\circ - 15^\circ$) | Form validation failures, critical system alerts, irreversible deletion buttons. | Marketing banners, decorative icons, random card headers. |
| **Warning / Caution** | Amber / Gold ($35^\circ - 45^\circ$) | Impending rate limits, non-blocking deprecation notices, unsaved changes. | Default badge borders, general information callouts. |
| **Success / Valid** | Emerald / Green ($140^\circ - 160^\circ$) | Completed transactions, active server status, verified security keys. | General primary buttons (unless "Proceed" is the core verb). |
| **Informational / Link**| Cobalt / Sky ($200^\circ - 220^\circ$) | Interactive hyperlinks, documentation callouts, active system states. | Passive decorative card fills. |

*Rule of State Sole-Conveyor*: Color must **never** be the single conveyor of state. Always pair color with an explicit icon (e.g. checkmark, exclamation triangle), text label, or spatial shift for the 8% of men and 0.5% of women with color-vision deficiencies.

---

## 5. Contrast Science: APCA & WCAG Standards

Modern interfaces should adhere to the **Accessible Perceptual Contrast Algorithm (APCA)**, which models how the human eye actually perceives text against backgrounds:

* **Body Prose ($14 - 18\text{px}$)**: Requires an APCA Lightness Contrast $L^c \ge 60$ (equivalent to WCAG AA $4.5:1$).
* **Large Headings ($> 24\text{px}$ or bold)**: Requires an APCA Lightness Contrast $L^c \ge 45$ (equivalent to WCAG AA $3.0:1$).
* **Secondary / Muted Text**: Must never drop below $L^c \ge 45$.
* **Disabled States**: Permitted to drop to $L^c \approx 30$, but must be paired with disabled cursor/touch affordance.

---

## 6. Dark Mode Luminance Physics

Dark mode is **not** simply inverting light mode hex codes:

### 6.1 Avoid Pure `#000000` Against Pure `#FFFFFF`
High contrast on OLED screens causes **halation / text irradiation** (bright white text bleeds into pure black ground, causing optical blurring and eye fatigue).
* Use deep neutral charcoal or zinc for the base ground (`#0F1115` or `#121212`).
* Step down primary text lightness slightly from pure white (e.g. `#E6E8EB` or $90\%$ opacity white).

### 6.2 Elevation Means Increasing Lightness (Not Darker Shadows)
In real physical space, objects closer to a light source receive more illumination:
* In Light Mode: Elevated cards cast darker shadows downward.
* In Dark Mode: Shadows are invisible on black. Instead, **elevate surfaces by stepping up surface lightness**:
  - Ground: `#0D0E11`
  - Layer 1 (Card / Sidebar): `#16181D`
  - Layer 2 (Modal / Popover): `#20232B`
  - Layer 3 (Active Item / Focus): `#2A2E38`
