# Invariance Verification

> **Mandate**: Refactoring is defined by what does *not* change. Prove that the refactored code preserves external observable behavior, public interfaces, error semantics, and runtime performance.

---

## 1. The Invariance Checklist

Before declaring a refactor complete, verify these four dimensions of behavioral invariance:

| Invariance Dimension | What Must Remain Identical | Verification Method |
| :--- | :--- | :--- |
| **1. Public Interface Contract** | Export names, function signatures, parameter order, return types. | Static analysis / TypeScript compiler (`tsc --noEmit`). |
| **2. Return Value Semantics** | Object schemas, numeric values, string formats, collection ordering. | Existing regression tests and characterization tests. |
| **3. Error Handling Semantics** | Specific exception types thrown, status codes, causal error details. | Hostile path test assertions. |
| **4. Side Effect Invariants** | Number and content of DB writes, emitted events, file system modifications. | Integration tests / DB transaction audits. |

---

## 2. Git Diff Hygiene for Refactoring

A clean refactoring diff tells a clear story to code reviewers:

1. **Zero New Logic**: If the git diff contains an added `if` statement handling a brand-new business case, you violated the **Separation Law**. Extract that feature logic and commit it in a separate PR.
2. **Zero Formatting Noise**: Never run an aggressive global formatter across 50 untouched files in the same commit as a structural refactor. Keep formatting changes isolated.
3. **Preserve Comments & Docstrings**: Never strip existing architectural comments or explanatory docstrings during restructuring unless they are rendered completely obsolete by the refactoring move.

---

## 3. Rollback Safety & Checkpointing

During a multi-step refactor, treat Git as your checkpoint harness:

* **Commit on Every Green Step**:
  ```bash
  git commit -m "refactor: extract calculateTax helper (tests green)"
  git commit -m "refactor: introduce TaxOptions parameter object (tests green)"
  ```
* **Instant Revert on Failure**:
  If a micro-step fails and you cannot identify the exact typo within 2 minutes:
  ```bash
  git checkout -- .
  ```
  *Never spend 30 minutes trying to patch a broken refactor. Revert instantly to the last green checkpoint and take a smaller, safer micro-step.*
