# Risk-Based Testing & Effort Allocation

> **Mandate**: Software risk is heterogeneous. High-consequence code requires exhaustive verification across both nominal and hostile paths; low-consequence code warrants lightweight checks. Allocate testing effort where defects cause financial, security, or data loss.

---

## 1. The Risk Scoring Matrix

Classify the target module or feature into one of three risk tiers based on **Impact of Failure** $\times$ **Likelihood of Regression**:

```text
               High Impact (Data Loss, Security Breach, Billing Failure)
                       ▲
                       │     [ TIER 0: Critical Invariants ]
                       │     • Full Unit + Integration + Hostile Paths
                       │     • Property-Based Invariant Verification
                       │     • Mutation Testing Check
                       │
                       │     [ TIER 1: Core Domain ]
                       │     • Unit + Integration
                       │     • Golden Path + Common Error Paths
                       │
                       │     [ TIER 2: Peripheral / UI ]
                       │     • Smoke Test / Contract Check
                       ▼
               Low Impact (Cosmetic Glitch, Minor Typo, Transient Cache Miss)
```

---

## 2. Tier Breakdown & Required Test Strategies

### Tier 0: Critical Invariants
* **Target Domains**:
  - Authentication, authorization, token verification, permission gates.
  - Financial calculations, billing transactions, ledger accounting.
  - Data persistence, schema migrations, transactional rollback logic.
  - Concurrency locks, rate limiters, distributed state coordination.
* **Mandated Verification Depth**:
  - 100% path coverage of error and rollback branches.
  - Hostile testing: Malformed tokens, expired sessions, network drop injection, race condition tests.
  - Property-based testing for serialization, cryptography, or calculation invariants.
  - Inline mutation check: Verify that altering any condition turns at least one test RED.

### Tier 1: Core Domain Workflows
* **Target Domains**:
  - Business workflows, order processing, user management, CRUD services.
  - REST, gRPC, and GraphQL endpoint handlers.
  - Data transformation pipelines and domain service logic.
* **Mandated Verification Depth**:
  - Golden path verification (valid inputs $\rightarrow$ expected state transition $\rightarrow$ output).
  - Standard edge cases: empty lists, maximum allowed limits, duplicate records.
  - Input validation error paths (400 Bad Request on invalid payloads).

### Tier 2: Peripheral, Glue & Presentational Code
* **Target Domains**:
  - CLI help text formatting, banner rendering.
  - Presentational UI components without business logic.
  - Logging formatters, configuration wrappers.
* **Mandated Verification Depth**:
  - Smoke test: Does the component instantiate without crashing?
  - Contract check: Does it render the expected text or return the expected status?
  - **Avoid**: Testing exact CSS pixel positions, internal HTML tag hierarchies, or private component state.
