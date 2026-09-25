# Technical Debt Register Template (`TECH_DEBT.md`)

Use this standardized template to track, price, and prioritize technical debt discovered during audits. Store in the repository root as `TECH_DEBT.md` or sync directly to your issue tracking system.

---

# Technical Debt Register

> **Policy**: Technical debt recorded here represents explicit liabilities against system maintainability and engineering velocity. Items are reviewed during sprint planning and addressed through prioritized maintenance allocations or when touching related functional areas.

## Summary Metric Dashboard

* **Total Tracked Items**: `3`
* **Critical Priority ($I_{\text{debt}} \ge 25$)**: `1`
* **High Priority ($10 \le I_{\text{debt}} < 25$)**: `1`
* **Medium/Low Priority ($I_{\text{debt}} < 10$)**: `1`

---

## Active Technical Debt Items

### [DEBT-001]: Monolithic checkout state machine in `src/core/checkout.ts`
* **Category**: Structural Complexity
* **Interest Index ($I_{\text{debt}}$)**: **$78.1$** (P1 - Critical)
* **Location & Scope**: `src/core/checkout.ts#L45-L380` | Blast Radius: Systemic (12 downstream modules)
* **Root Structural Cause**: God-function mixing database persistence, payment gateway calls, and discount validation into a single 335-line switch block.
* **Evidence & Metrics**: Cyclomatic Complexity: 48 | Churn: 18 commits in last 90 days (6 defect fixes) | Fan-in: 12 callers.
* **Recommended Skill**: [`refactoring`](.agents/skills/refactoring/SKILL.md)
* **Remediation Specification**: Pin transition matrix with characterization tests; extract state machine transitions into polymorphic strategy objects.
* **Review Horizon**: Q3 Release Milestone / Sprint 14.

---

### [DEBT-002]: Quarantined legacy auth fallback in `src/auth/token_verify.py`
* **Category**: Dead / Deprecated Code
* **Interest Index ($I_{\text{debt}}$)**: **$14.2$** (P2 - High)
* **Location & Scope**: `src/auth/token_verify.py#L110-L165` | Blast Radius: Module
* **Root Structural Cause**: Deprecated v1 HMAC token verification logic retained after v2 RSA migration.
* **Evidence & Metrics**: Telemetry shows zero v1 token requests in 90 days. Static callers: 1 legacy test fixture.
* **Recommended Skill**: [`maintenance`](.agents/skills/maintenance/SKILL.md) (Mode: `dead-code-prune`)
* **Remediation Specification**: Delete legacy HMAC verification function and obsolete mock test fixture in a reverse topological pass.
* **Review Horizon**: Next maintenance sprint.

---

### [DEBT-003]: Undocumented magic numbers in `src/rendering/canvas_layout.cpp`
* **Category**: Maintainability
* **Interest Index ($I_{\text{debt}}$)**: **$3.4$** (P4 - Low)
* **Location & Scope**: `src/rendering/canvas_layout.cpp#L82-L115` | Blast Radius: Local
* **Root Structural Cause**: Hardcoded screen DPI offsets and padding factors without named constants.
* **Evidence & Metrics**: Cyclomatic: 4 | Churn: 1 commit in last 180 days.
* **Recommended Skill**: [`code-quality`](.agents/skills/code-quality/SKILL.md)
* **Remediation Specification**: Extract numeric literals into typed `constexpr` struct.
* **Review Horizon**: When canvas rendering is next updated.
