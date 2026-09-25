# Maintenance Routing & Backlog Synthesis

> **Mandate**: *An audit that produces no durable work items is pure waste; a maintenance task that attempts to fix everything in one unbounded PR is pure chaos.* An agent must strictly bound its immediate blast radius, enforce clear boundary contracts with specialized sibling skills, and synthesize all remaining findings into structured, trackable technical-debt items.

---

## 1 · The Arsenal Skill Boundary Routing Matrix

Maintenance is the **diagnostic sensor and dispatch router** of the software lifecycle. When deterioration is discovered during an audit, the agent determines the appropriate handling protocol using this deterministic decision matrix:

| Finding Type | Condition / Trigger | Target Arsenal Skill | Handoff Protocol |
| :--- | :--- | :--- | :--- |
| **Dead Code Excision** | Symbol is provably unreachable ($C_{\text{dead}} \ge 0.95$), bounded to leaf nodes, zero external users. | **`maintenance`** (Internal) | Execute via reverse topological deletion, verify tests, commit cleanly. |
| **Structural Restructuring** | Code is active and reachable, but has high cyclomatic complexity, God-class bloat, or tangled coupling. | [`refactoring`](../../refactoring/SKILL.md) | Pin with characterization tests first; execute atomic structural micro-steps with zero behavioral changes. |
| **Outdated / Vulnerable Packages** | Manifest packages are aging, have active CVEs, or lockfiles are desynchronized. | [`dependency-management`](../../dependency-management/SKILL.md) | Audit changelogs, verify call-graph reachability, execute staged upgrade batches with clean install tests. |
| **Defensive Code Hygiene** | Missing boundary validation, illegal state representation, unhandled errors, or resource leaks. | [`code-quality`](../../code-quality/SKILL.md) | Apply boundary guards, strict domain typing, and deterministic cleanup handlers (`defer`, `using`, `try-finally`). |
| **Stale or Drifted Docs** | READMEs, ADRs, architecture diagrams, or code comments contradict actual implementation. | [`docs-pro`](../../docs-pro/SKILL.md) | Apply Diátaxis framework; update documentation to reflect ground-truth implementation. |
| **Security Surface Decay** | Deprecated cryptographic algorithms, plain-text secret exposure, or authentication bypass risks. | [`security-engineering`](../../security-engineering/SKILL.md) | Apply threat modeling, zero ambient authority, and evidence-grounded vulnerability remediation. |
| **Obsolete Wheels Requiring Adoption** | Custom in-house utility can be replaced by modern standard library features or high-vitality libraries. | [`solution-discovery`](../../solution-discovery/SKILL.md) | Evaluate alternatives across 10-axis matrix; synthesize Solution Adoption Record (SAR). |
| **High Blast-Radius Debt** | Finding requires multi-step migration touching $> K_{\text{mode}}$ files or crossing service boundaries. | [`planning`](../../planning/SKILL.md) + Backlog | Synthesize into structured `TECH_DEBT.md` item; author execution plan with verification triads. |

---

## 2 · The Bounded Blast Radius Gate (Preventing Rabbit Holes)

To prevent the agent from descending into an infinite maintenance spiral during a scoped task, the agent enforces the **Blast Radius Bound**:

$$\text{Files Mutated} \le K_{\text{mode}}$$

| Execution Mode | Maximum Files Allowed to Mutate ($K_{\text{mode}}$) | Policy on Exceeding Bound |
| :--- | :--- | :--- |
| **`micro-hygiene`** | $\le 2$ files | If cleanup touches a 3rd file, **abort immediately**; emit finding to scratchpad. |
| **`dead-code-prune`** | $\le 10$ files (leaf cluster) | If unreachability cascade exceeds 10 files, stop at leaf layer; register remainder in `TECH_DEBT.md`. |
| **`tech-debt-audit`** | **0 files (Read-Only)** | Strictly forbidden to mutate code during an audit. Output scorecard and backlog items only. |
| **`tech-radar-review`**| $\le 2$ files (Radar/ADR only) | Do not execute migrations; update radar documentation and roadmaps only. |
| **`agent-hygiene`** | $\le 3$ files (`AGENTS.md` / `SKILL.md`) | Scope strictly to identified rule consolidations; verify with Invariant Shield. |
| **`sdlc-sprint`** | Defined by Sprint Plan | Governed strictly by the pre-approved sprint plan and active test harness. |

---

## 3 · Technical Debt Register Specification (`TECH_DEBT.md`)

When an audit reveals technical debt that cannot be remediated within the active task boundary, the agent records it in the repository's **Technical Debt Register** (`TECH_DEBT.md`):

```markdown
### [DEBT-ID]: <Concise Title>
* **Category**: [Structural | Dead Code | Dependency | Security | Documentation | Agent Guidance]
* **Interest Index ($I_{\text{debt}}$)**: [P1 - Critical ($I \ge 25$) | P2 - High ($10 \le I < 25$) | P3 - Medium ($5 \le I < 10$) | P4 - Low ($I < 5$)]
* **Location & Scope**: `<file_path>` (Lines: X–Y) | Blast Radius: [Local | Module | Systemic]
* **Root Structural Cause**: [Description of complexity, coupling, or obsolescence]
* **Evidence & Metrics**: [Cyclomatic: N, Churn: N commits/90d, Downstream Callers: N]
* **Recommended Skill**: [refactoring | dependency-management | security-engineering | solution-discovery]
* **Remediation Specification**: [What concrete actions should be taken when addressed]
* **Review Horizon**: [Review date or trigger milestone]
```

---

## 4 · Issue Tracker & Backlog Integration

If the repository utilizes an external issue tracker (GitHub Issues, GitLab, Jira), the agent can synthesize findings directly into issue payloads:

```markdown
## Title: `[TECH-DEBT: P1] Decompose monolithic checkout state machine in src/core/checkout.ts`

### Problem Statement
Static analysis and churn forensics identified `src/core/checkout.ts` as a P1 technical-debt hotspot ($I_{\text{debt}} = 78.1$):
- Cyclomatic complexity: 48 (deeply nested switch/case and if/else cascades).
- Churn: 18 commits in last 90 days (6 defect-fix commits).
- Fan-in: 12 downstream callers.

### Proposed Remediation
1. Route to `refactoring` skill.
2. Pin existing state transitions using Characterization Tests (Golden Master snapshots).
3. Apply State Pattern refactoring to extract checkout transitions into independent strategy classes.
4. Verify behavioral invariance against existing checkout test suites.
```
