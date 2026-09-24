# TEST_SPEC: Verification Matrix

```markdown
# Test Specification: [Feature or Module Name]

> **Objective**: [1-sentence summary of the behavior being verified]
> **Risk Tier**: [Tier 0: Critical Invariant | Tier 1: Core Domain | Tier 2: Peripheral]
> **Target Harness**: [e.g. Vitest 2.0 / Pytest 8.2 / go test]

---

## 1. Test Matrix Overview

| Test Case ID | Path Category | Description / Inputs | Expected Observable Behavior | Invariant Verified |
| :--- | :--- | :--- | :--- | :--- |
| `TC-01` | **Golden Path** | Canonical valid request payload. | Status 200/201, valid response schema, record saved to DB. | Core functional contract |
| `TC-02` | **Hostile Path** | Missing required auth token. | Status 401 Unauthorized, structured error cause. | Auth security boundary |
| `TC-03` | **Hostile Path** | Negative amount or boundary overflow. | Status 400 Bad Request, validation message with field pointer. | Input boundary guard |
| `TC-04` | **Hostile Path** | Simulated downstream timeout (3000ms). | Status 504 Gateway Timeout, transaction cleanly rolled back. | Resource & timeout cleanup |
| `TC-05` | **Property/Invariant** | 1000 generated inputs across parser. | `deserialize(serialize(x)) === x`. | Algebraic round-trip |

---

## 2. Execution & Scoped Commands

- **Scoped Test Command**:
  ```bash
  [Command to run ONLY this test file, e.g. npx vitest run tests/unit/auth.test.ts]
  ```
- **Full Suite Regression Command**:
  ```bash
  [Command to run full project test suite, e.g. npm test]
  ```

---

## 3. Mutation Testing Self-Check

- [ ] **Boundary Condition**: Inverting `>=` to `>` causes `TC-03` to FAIL.
- [ ] **Error Path**: Deleting auth check causes `TC-02` to FAIL.
- [ ] **Side Effect**: Commenting out database insert causes `TC-01` to FAIL.
```
