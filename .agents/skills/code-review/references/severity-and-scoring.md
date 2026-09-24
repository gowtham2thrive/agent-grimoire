# Severity Rubric & Confidence Scoring

> **Mandate**: Code review comments must provide maximum signal with minimum noise. Enforce strict objective severity definitions, calculate confidence scores before reporting, and automatically suppress any findings that do not meet the minimum evidence bar.

---

## 1. Severity Tiers

Every finding must be assigned exactly one severity tier based on business and operational impact:

| Tier | Name | Criteria | Merge Action |
| :---: | :--- | :--- | :--- |
| **P0** | **Blocker** | • Active security vulnerability (injection, auth bypass, secret leak)<br/>• Data loss or corruption risk<br/>• Build, compiler, or critical test suite breakage<br/>• Severe regression in core user journey | **Must block merge**. Code cannot ship until resolved. |
| **P1** | **Critical** | • High-probability logic defect in main workflow<br/>• Resource leak (unclosed DB connection, memory leak)<br/>• Breaking change to public API contract without deprecation<br/>• Unhandled error path in transactional code | Must be fixed before release unless explicitly waived by lead. |
| **P2** | **Moderate** | • Missing test coverage for non-trivial edge case<br/>• Sub-optimal query ($N+1$) that degrades performance under load<br/>• Code smell that increases ongoing maintenance risk | Should be addressed; non-blocking for emergency hotfixes. |
| **P3** | **Advisory** | • Meaningful architectural simplification (with code snippet)<br/>• Documentation omission or outdated comment<br/>• Better standard library alternative available | Optional suggestion; author may adopt or decline. |

---

## 2. Confidence Scoring (0–100)

Before a finding is approved for the final report, compute its **Confidence Score**:

$$\text{Confidence} = \text{Grounding (up to 40)} + \text{Reproducibility (up to 40)} + \text{Impact Clarity (up to 20)} - \text{Speculation Penalty}$$

* **Grounding (0–40 points)**:
  - 40: Exact line citations with verified AST/runtime behavior.
  - 20: Grounded in diff, but caller context not fully inspected.
  - 0: Vague mention of a file or pattern without line citations.
* **Reproducibility (0–40 points)**:
  - 40: Concrete inputs and execution sequence provided that triggers the bug.
  - 20: Plausible theoretical failure scenario without exact inputs.
  - 0: Speculative opinion ("this might cause issues in the future").
* **Impact Clarity (0–20 points)**:
  - 20: Direct damage identified (e.g. "causes 500 error on checkout for users with no middle name").
  - 10: General performance or code health concern.
  - 0: Pure aesthetic preference.
* **Speculation Penalty (-30 points)**:
  - Deduct 30 points if the finding relies on unverified assumptions about frameworks or upstream guarantees.

---

## 3. The Strict Suppression Rules

A finding is **automatically dropped** if it matches any of the following:

1. **Low Confidence Filter**: Any finding with a total confidence score $< 70$ is pruned.
2. **Cosmetic & Style Filter**: Comments on formatting, whitespace, single vs double quotes, trailing commas, or variable naming preferences are strictly prohibited. These belong to automated formatters (`prettier`, `black`, `gofmt`).
3. **Pre-Existing Code Filter**: Issues present in unchanged legacy code that the author did not touch are suppressed (unless directly broken by the new diff).
4. **Pedantic Trivia Filter**: Theoretical micro-optimizations (e.g. `for` vs `forEach` when processing 10 items) that have zero measurable latency impact.
