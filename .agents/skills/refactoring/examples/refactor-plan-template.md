# REFACTOR_PLAN: Micro-Step Execution Matrix

```markdown
# Refactoring Plan: [Target Module / Subsystem]

> **Objective**: [1-sentence summary of architectural debt being resolved]
> **Mode**: [opportunistic | strategic]
> **Baseline Test Status**: [VERIFIED GREEN (14 passing tests)]

---

## 1. Smells Identified & Target Architecture

- **Code Smell**: [e.g. 450-line order checkout function with 4 levels of nested if-else ladders]
- **Target Architecture**: [e.g. Decomposed into 3 cohesive stages: IngressValidation, TaxCalculationStrategy, and OrderPersistence]
- **Invariants to Preserve**: [Public checkout API signature, database schema, payment gateway webhook contracts]

---

## 2. Micro-Step Sequence

| Step # | Refactoring Move | Target File / Lines | Scoped Verification Command | Status |
| :---: | :--- | :--- | :--- | :---: |
| **0** | Pin legacy behavior with Characterization Tests | `tests/characterization/checkout.test.ts` | `npx vitest run tests/characterization/` | ✅ Done |
| **1** | Extract `validateCheckoutPayload` helper | `src/checkout.ts#L45-L95` | `npx vitest run tests/characterization/` | ✅ Done |
| **2** | Introduce `PaymentStrategy` map | `src/payments/strategy.ts` | `npx vitest run tests/characterization/` | ⏳ Pending |
| **3** | Replace conditional with strategy lookup | `src/checkout.ts#L120-L180` | `npx vitest run tests/characterization/` | ⏳ Pending |
| **4** | Clean up unused intermediate variables | `src/checkout.ts` | `npm test` (full suite) | ⏳ Pending |

---

## 3. Invariance Verification Sign-off

- [ ] All characterization tests pass 100% green.
- [ ] Full regression suite passes with exit code 0.
- [ ] Static type check (`tsc --noEmit` / `mypy` / `cargo check`) passes without errors.
- [ ] `git diff` confirms zero changes to public exports or error signatures.
```
