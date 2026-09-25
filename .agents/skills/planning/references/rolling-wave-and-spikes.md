# Rolling Wave Planning & Investigation Spikes

> **Mandate**: *Do not map the distant fog with false precision.* Detailing twenty tasks when task two depends on an unverified third-party API or an unproven algorithm is a recipe for hallucination and wasted effort. Plan in **rolling waves**: elaborate the immediate horizon in crisp, falsifiable detail, while holding distant horizons as coarse milestones until prerequisite spikes land.

---

## 1 · The Rolling Wave Horizon Concept

Planning operates across two distinct epistemic horizons:

```mermaid
flowchart LR
    subgraph Immediate Wave ["Immediate Wave (High Fidelity)"]
        T1["Task 1: Spike or Foundation<br/>(Exact files & test commands)"]
        T2["Task 2: Core Vertical Slice<br/>(Exact files & test commands)"]
    end
    subgraph Distant Wave ["Distant Wave (Milestones & Boundaries)"]
        M1["Milestone 3: Consumer Integration<br/>(Boundary goals & acceptance specs)"]
        M2["Milestone 4: Deployment & Telemetry<br/>(Integration assertions)"]
    end

    Immediate Wave ==>|Wave Elaboration Trigger| Distant Wave
```

### Horizon 1: The Immediate Wave (Current Phase)
* **Granularity**: Full vertical slices.
* **Metadata**: Exact `Target Files`, specific AST symbol names, explicit Pre-check and Post-verification commands.
* **Scope**: Tasks that can be executed and verified immediately without speculative assumptions.

### Horizon 2: The Distant Wave (Downstream Phases)
* **Granularity**: Coarse-grained milestones.
* **Metadata**: Intended functional outcomes, boundary constraints, and acceptance criteria.
* **Elaboration Trigger**: Downstream tasks are converted into detailed Immediate Wave tasks **only when the current wave completes and verifies green**.

---

## 2 · Investigation Spikes (Quarantining Unknowns)

When an essential technical premise is uncertain (e.g. an unverified third-party API, an unfamiliar library behavior, or a complex performance bottleneck):

### When to Schedule a Spike
Apply the **Clarification & Risk Gate**:
$$\text{If } C_{\text{reversal}} > 2\text{ hours} \quad \text{and uncertainty is technical} \implies \text{Schedule Investigation Spike}$$

Do not guess or draft downstream implementation tasks on an unverified assumption.

### Rules of Engagement for Spikes
1. **Timeboxed & Disposable**: Spikes must be tightly timeboxed and produce disposable throwaway prototypes, benchmarks, or inspection traces—never permanent production code.
2. **Explicit Hypothesis**: Every spike must state: *"We hypothesize that Library X can perform Operation Y in $< 100\text{ms}$ with configuration Z."*
3. **Falsifiable Finding**: The spike outputs a structured `SPIKE_REPORT.md` answering the hypothesis with empirical data.
4. **Downstream Unblocking**: Once the spike delivers evidence, the agent elaborates the downstream tasks using the proven approach.

---

## 3 · The Autonomous Defaulting Axiom

When uncertainty arises during planning, do not stall or interrogate the user over low-impact choices:

$$\text{If } C_{\text{reversal}} \le 2\text{ hours} \implies \text{Adopt Standard Architectural Default \& Document in Assumptions}$$

* If naming conventions, internal directory layouts, or standard library helper patterns are unspecified, adopt the ecosystem standard default.
* Log the decision in the plan's `Assumptions Register`.
* Only query the user if a requirement is contradictory, user-facing behavior is ambiguous, or security/data integrity is at stake.
