# Accessibility Audit Report

**Project**: [Project Name / URL / App Bundle]  
**Auditor**: [Agent / Engineer Name]  
**Date**: [YYYY-MM-DD]  
**Standard Evaluated**: WCAG 2.2 AA / EN 301 549  
**Highest Epistemic Evidence Tier Attained**: [Tier 1 (AST) / Tier 2 (DOM) / Tier 3 (Rendered Surface) / Tier 4 (Task Completion)]  

---

## 1 · Executive Summary

| Total Findings | Severity 4 (Blocker) | Severity 3 (Major) | Severity 2 (Moderate) | Severity 1 (Minor) | Severity 0 (Cosmetic) |
| :---: | :---: | :---: | :---: | :---: | :---: |
| [Count] | [Count] | [Count] | [Count] | [Count] | [Count] |

**Task Completion Conformance Status**:  
[CONFORMANT / CONDITIONALLY CONFORMANT / NON-CONFORMANT]

---

## 2 · Verified In-Scope User Tasks

| Task ID | User Task Description | Primary Modalities Verified | Outcome Equivalence ($\mathcal{A}_1$) |
| :--- | :--- | :--- | :---: |
| `TASK-01` | Account Registration & Email Verification | Keyboard-only, VoiceOver, 400% Zoom | PASS |
| `TASK-02` | Search & Filter Inventory Data Table | Keyboard-only, TalkBack, High Contrast | BLOCKED (Finding A11Y-01) |

---

## 3 · Detailed Mathematical Findings

### [FINDING-A11Y-01] [Brief Title Describing Barrier]
- **Component / File Anchor**: `[path/to/file.tsx:L123-L145]`
- **User Task & Affected Barrier**: [Describe who is blocked and what task fails]
- **Violated Invariant / Standard**: Invariant [A1-A8] · WCAG 2.2 SC [X.X.X] (Level A/AA)
- **Severity**: [0 - Cosmetic / 1 - Minor / 2 - Moderate / 3 - Major / 4 - Catastrophic]
- **Evidence Tier**: [Tier 1 - AST / Tier 2 - Headless DOM / Tier 3 - Rendered Surface / Tier 4 - AT Task Completion]
- **Empirical Observation**: [Exact behavior observed during keyboard/AT traversal]
- **Concrete Code Remediation**:
```tsx
// Drop-in code fix demonstrating compliant implementation
```

---

## 4 · Evidence Scope & Limitations

*State clearly what was NOT tested to prevent false confidence claims:*
- Testing Environment: [Browser & OS versions, viewport resolutions]
- Unverified Surfaces: [e.g. Third-party payment iframe, legacy PDF exports]
- Screen Readers Tested: [e.g. VoiceOver macOS 15, NVDA 2024.1 on Windows 11]
