# Project Accessibility Contract (`ACCESSIBILITY.md`)

> **Living Baseline**: This document establishes the binding accessibility commitments, verification gates, and conformance baselines for this repository. All contributors and AI coding agents must uphold these standards.

---

## 1 · Conformance Target & Scope

- **Target Standard**: [WCAG 2.2 AA / EN 301 549 / Section 508 / Platform Native]
- **Target Platforms**: [Modern Web (Desktop & Mobile) / iOS / Android / Desktop / TUI]
- **Supported Assistive Technologies**:
  - Screen Readers: [e.g., NVDA on Windows, VoiceOver on macOS/iOS, TalkBack on Android, Orca on Linux]
  - Alternate Inputs: [Keyboard-only, Switch devices, Voice Control, Head-tracking]
- **Contrast Standard**: WCAG 2.2 AA ($4.5:1$ text / $3:1$ UI components) + APCA $L^c \ge 60$ for body reading.

---

## 2 · Automated & Manual Verification Commands

Run these automated verification checks locally before submitting any PR:

```bash
# 1. Static AST / Linter Accessibility Scan (Tier 1)
npm run lint:a11y

# 2. Automated Headless Browser & Engine Audits (Tier 2/3)
npm run test:a11y

# 3. Component Keyboard Navigation Tests
npm run test:unit -- --testPathPattern="a11y"
```

---

## 3 · Manual Verification Protocol (Pre-Release Gate)

Before deploying major feature releases or new UI modules, the following manual walkthrough must be executed and recorded:
- [ ] **Pass 1 (Keyboard Operability)**: Unplug mouse. Complete entire primary user flow using only `Tab`, `Shift+Tab`, `Enter`, `Space`, and `Escape`. Verify focus outline is never obscured.
- [ ] **Pass 2 (Screen Reader Verification)**: Verify flow with at least one primary screen reader (e.g. VoiceOver, NVDA). Verify all buttons and inputs have concise, unambiguous names.
- [ ] **Pass 3 (Zoom & Reflow)**: Zoom browser to $400\%$ ($320\text{px}$ width). Verify content stacks vertically without horizontal scrollbars.
- [ ] **Pass 4 (Forced Colors / High Contrast)**: Enable OS High Contrast. Verify all button borders and input outlines remain distinct.
- [ ] **Pass 5 (Reduced Motion)**: Enable `prefers-reduced-motion`. Verify all decorative slides and parallax animations are muted.

---

## 4 · Known Exceptions & Technical Debt

Document all formal exceptions granted under the Accessibility Exception Protocol:

| Exception ID | Target Component | Reason / Constraint | Compensating Pathway | Sunset Target |
| :--- | :--- | :--- | :--- | :--- |
| `A11Y-EXP-01` | `src/legacy/chart.js` | Legacy canvas chart without virtual tree | Dedicated tabular data toggle button (`Alt+T`) | Sprint 32 |

---

## 5 · Contact & Escalation

- **Accessibility Lead**: [Name / Email / Slack Channel]
- **Escalation SLA**: Severity 3 (Major) and Severity 4 (Blocker) defects must be triaged within 24 hours.
