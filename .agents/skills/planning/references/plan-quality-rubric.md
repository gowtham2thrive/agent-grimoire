# Plan Quality Rubric & Verification Gate

> **Mandate**: *Never advance to code execution on an unvetted plan.* Before a plan in `standard` or `epic` mode is released for implementation, it must be evaluated against this 0–100 quality scoring rubric. A plan with weak verification, overlapping boundaries, or hallucinated file paths must be revised before execution begins.

---

## 1 · The 0–100 Plan Quality Rubric

The rubric evaluates the implementation plan across 5 orthogonal axes (20 points each):

| Axis | Max Points | Evaluation Focus | 15–20 Pts (Exemplary) | 10–14 Pts (Adequate) | 0–9 Pts (Failing) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Evidence Grounding** | 20 | Empirical grounding in actual repository files, symbols, and tooling. | Every path, symbol, and tool verified via cartography. Zero guesswork. | Most paths verified; minor naming conventions inferred without risk. | References hypothetical files or assumed libraries without checking. |
| **2. Topological Rigor** | 20 | DAG ordering, foundation-first discipline, critical path management. | Strict acyclic DAG; shared schemas/types precede consumers; zero deadlocks. | Acyclic order maintained, but foundation tasks loosely coupled. | Cyclic dependencies present ($A \to B \to A$); consumers precede schemas. |
| **3. Boundary Isolation** | 20 | Disjoint mutation surfaces, blast-radius containment, concurrency safety. | Every task has exclusive `Target Files`; parallel tasks are strictly disjoint. | Target files declared, but shared registry file touched across multiple tasks. | Ambiguous or overlapping file boundaries across concurrent tasks. |
| **4. Verification Determinism** | 20 | Falsifiable Verification Triads $\langle \text{Pre}, \text{Mutation}, \text{Post} \rangle$. | Automated test command per task; boundary/negative checks included; no self-grading. | Test commands present, but rely on coarse full-suite runs. | Subjective self-review (*"verify code looks correct"*); missing test commands. |
| **5. Thermo-Nuclear Simplicity** | 20 | Minimum viable moving parts; absence of speculative abstractions or bloat. | Plan pruned aggressively; zero premature wrappers; every task traces to `REQ-xxx`. | Mostly lean, but contains 1–2 minor speculative helper tasks. | Over-engineered; 10 layers for a simple task; enterprise boilerplate. |

### Minimum Passing Threshold
$$\text{Total Score} \ge 85 / 100 \quad \text{AND} \quad \text{Score}_{\text{axis}} \ge 15 / 20 \quad \forall \; \text{axes}$$

Any plan scoring $< 85$, or $< 15$ on any single axis, is **rejected** and must be refactored before execution.

---

## 2 · Hard Rejection Triggers (Immediate Vetoes)

The presence of any single condition below triggers an **immediate hard rejection**, overriding any numerical score:

- ⛔ **Hallucinated Path**: Any task targets a file path that does not exist and is not explicitly scheduled for creation.
- ⛔ **Cyclic Dependency**: Any circular loop in the task DAG ($T_1 \to T_2 \to T_1$).
- ⛔ **Shared Mutation Collision**: Two parallel or concurrent tasks declare overlapping `Target Files`.
- ⛔ **Subjective Verification Gate**: Any task uses self-attestation (*"ensure code is clean"*) as its sole post-verification.
- ⛔ **Code Smuggling**: Implementation bodies pasted wholesale into `PLAN.md` instead of being scoped for code editing.
- ⛔ **Missing Foundation Settlement**: Leaf components scheduled before database migrations or type definitions have landed.

---

## 3 · Pre-Flight Execution Release Checklist

Before running the first implementation tool:

- [ ] Plan scored $\ge 85/100$ on the Plan Quality Rubric.
- [ ] Zero Hard Rejection Triggers tripped.
- [ ] Working directory verified clean via `git status`.
- [ ] Initial `PLAN.md` committed or persisted to disk.
- [ ] Execution mode confirmed (`standard`, `epic`, or `micro`).
