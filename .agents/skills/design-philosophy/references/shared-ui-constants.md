# Shared UI Constants

These constants are shared across `design-philosophy`, `ux-engineering`, and `accessibility`. They are defined once here and referenced by all three skills.

## Interactive Target Sizing
- Minimum interactive target: ≥ 44 × 44 pt / 9mm
- Minimum spacing between adjacent interactive controls: ≥ 8 pt
- Fitts's Law: High-frequency triggers → large hit areas in natural reach zones; dangerous actions → deliberate friction

## Feedback Timing
- Visual acknowledgment of any user gesture: ≤ 100 ms (Doherty threshold)
- Complete interaction loop closure: ≤ 400 ms

## Contrast & Readability
- APCA body text: Lc ≥ 60
- APCA large headers: Lc ≥ 45
- WCAG 2.2 AA: 4.5:1 normal text, 3:1 large text

## Responsive Resilience
- Layout must survive 3× string length expansion
- Layout must survive 30% text expansion (translation)
- Layout must survive 200% font zoom without clipping or horizontal scroll
