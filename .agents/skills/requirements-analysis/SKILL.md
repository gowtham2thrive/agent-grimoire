---
name: requirements-analysis
description: >-
  Universal, timeless requirements engineering, intent formalization, and specification protocol.
  Use when receiving raw prompts, user requests, feature ideas, bug reports, or architectural goals
  before touching code or designing architecture. Enforces intent-mechanism separation, the 7 Universal
  Requirements Invariants, EARS syntax, operational (M, T, C) NFR triads, BDD acceptance criteria,
  the Autonomous Defaulting Axiom, and living baseline traceability across any software archetype
  without limiting agent creativity or prescribing implementation mechanisms.
---

# Requirements Analysis: Universal Intent Engineering Protocol

> **Mandate**: *Requirements engineering is the disciplined bridge between human intent and computational reality.* Define what problem must be solved and why, establish empirical boundaries of satisfaction, surface implicit assumptions, and guard against premature implementation without constraining architectural creativity or micromanaging agent cognition.

---

## 1 · The 5-Phase Requirements Lifecycle

Every user request, feature prompt, or architectural initiative traverses this 5-phase protocol:

```mermaid
flowchart LR
    P1["1. Intent Extraction & Triage<br/>(Goal decoding, mode sizing, de-solutioning)"] --> P2["2. Orthogonal Decomposition<br/>(Functional, NFRs, Constraints, Non-Goals)"]
    P2 --> P3["3. Conflict & Boundary Forensics<br/>(Triad conflicts, EARS unwanted behavior, 1st-degree edge matrix)"]
    P3 --> P4["4. Formal Specification<br/>(EARS syntax, BDD Given-When-Then, (M, T, C) NFRs)"]
    P4 --> P5["5. Baseline & Traceability Handoff<br/>(RTM synthesis, quality gate 0-100, handoff to project-analysis)"]
```

1. **Phase 1 — Intent Extraction & Triage**: Decode the raw request to extract the root Job-to-be-Done. Apply the **Intent De-Solutioning Filter** to strip premature implementation mechanisms into candidate preferences. Size the cognitive mode (`micro`, `standard`, `architectural`, `ambiguity-rescue`). Apply the **Clarification Gate** ($C_{\text{reversal}} > 2\text{ hours}$); for low-cost forks, execute the **Autonomous Defaulting Axiom**.
2. **Phase 2 — Orthogonal Decomposition**: Disentangle the request into Functional Requirements (FR), Quality Attributes (NFR), Constraints, and Assumptions. Define the **Mandatory Scope Ceiling & Non-Goals** to eliminate scope creep.
3. **Phase 3 — Conflict & Boundary Forensics**: Audit language against the **Ambiguity Filter** (strip unmeasurable adjectives). Explore the **1st-Degree Operational Neighborhood** (input bounds, state counts, immediate environment failures). Resolve competing constraints via the **Triad Trade-off Matrix**.
4. **Phase 4 — Formal Specification**: Express functional behavior using **EARS** (Ubiquitous, Event, State, Optional, Unwanted). Define verifiable acceptance criteria via **BDD Given-When-Then** scenarios. Bind every NFR to the **Operational Measurement Triad $\langle M, T, C \rangle$**.
5. **Phase 5 — Baseline & Traceability Handoff**: Audit specification against the **0–100 Specification Quality Rubric**. Emit the durable requirement baseline (`REQUIREMENTS.md` or scoped intent block). Initialize the **Requirements Traceability Matrix (RTM)** and hand off to `project-analysis`.

---

## 2 · Progressive Disclosure (Reference Routing)

To preserve context bandwidth and prevent cognitive overload, consult reference manuals strictly on demand:

| Context Trigger | Mandatory Reference Manual | Purpose |
| :--- | :--- | :--- |
| **Vague prompts, clarification gates, defaults, EARS patterns** | [`references/elicitation-and-ambiguity.md`](references/elicitation-and-ambiguity.md) | Clarification Gate formula, Autonomous Defaulting Axiom, Ambiguity filter words, full EARS grammar. |
| **Decomposing requests, NFRs, ISO 25010, non-goals, constraints** | [`references/decomposition-and-nfrs.md`](references/decomposition-and-nfrs.md) | The 2-Tier Ontological Sieve, ISO 25010 NFR catalog, $(M, T, C)$ measurement triads, scope ceilings. |
| **BDD acceptance criteria, edge-case bounds, domain invariants** | [`references/formal-spec-and-bdd.md`](references/formal-spec-and-bdd.md) | Given-When-Then semantics, 1st-degree operational neighborhood, Meyer contract invariants, state tables. |
| **Traceability matrix, downstream handoff, living baseline deltas** | [`references/traceability-and-baseline.md`](references/traceability-and-baseline.md) | RTM schema, living baseline state machine, `REQ-DELTA` feedback protocol, skill handoff contracts. |
| **Scoring requirements, hard vetoes, quality audit gates** | [`references/specification-quality-rubric.md`](references/specification-quality-rubric.md) | 0–100 quality scoring gate (5 axes $\times$ 20 pts), hard rejection triggers, pre-flight checklist. |
| **End-to-end case walkthroughs & concrete execution traces** | [`examples/requirements-walkthrough.md`](examples/requirements-walkthrough.md) | Complete practical trace: raw ambiguous request $\to$ EARS $\to$ BDD $\to$ Baseline $\to$ RTM. |
| **Micro-mode, conflict resolution, & edge-case demonstrations** | [`examples/micro-sizing-and-edge-cases.md`](examples/micro-sizing-and-edge-cases.md) | Fast 3-Line Intent blocks, CAP theorem conflict resolution, brownfield architectural collisions. |

---

## 3 · Adaptive Cognitive Sizing (The 4 Modes)

Size requirements engineering to task risk, scope, and blast radius. Never apply heavy paperwork to trivial edits; never skip rigorous specifications on core systems:

| Mode | Trigger & Scope | Analysis Protocol | Target Output |
| :--- | :--- | :--- | :--- |
| **`micro`** | Quick bug fix, localized utility, small refactor ($< 30$ lines of change). | Instant intent extraction $\to$ verify testability $\to$ edge check. **Zero boilerplate**. | **3-Line Requirement Intent** block directly preceding code edit. |
| **`standard`** | Typical feature, API endpoint, service integration, DB schema change, UI screen. | Functional EARS decomposition $\to$ NFR triad $\to$ BDD Given-When-Then acceptance criteria $\to$ edge cases. | Feature Requirements Spec (`REQUIREMENTS.md` or ticket description). |
| **`architectural`** | Greenfield project, major subsystem, cross-cutting auth/crypto engine, multi-agent protocol. | Full 5-phase lifecycle: Actor mapping $\to$ EARS taxonomy $\to$ ISO 25010 NFRs $\to$ Non-Goals $\to$ RTM. | Formal Product Requirements Document (`PRD.md`) / Software Requirements Specification (`SRS.md`). |
| **`ambiguity-rescue`** | Contradictory, underspecified, or chaotic user prompt. | Proactive ambiguity triage $\to$ Trade-off matrix generation $\to$ Targeted clarification interview via `ask_question`. | Disambiguated Requirement Baseline confirmed with user. |

### The 3-Line Requirement Intent Protocol (For `micro` Mode)
When operating in `micro` mode, emit this block directly preceding implementation:
```markdown
> **Intent**: [Precise problem being solved and why]
> **Boundary**: [Input/state domain and negative edge condition handled]
> **Acceptance**: [Deterministic, falsifiable test assertion confirming success]
```

---

## 4 · The 7 Universal Requirements Invariants

Regardless of language, framework, or computational paradigm, every valid requirement upholds these 7 timeless laws:

### 4.1 Invariant 1: Intent-Mechanism Separation (Problem Space Primacy)
Specify **what** the system must accomplish and **why**, never **how** it is constructed. Requirements must be completely free of implementation mechanisms (languages, frameworks, internal classes, database tables, or design patterns) unless explicitly mandated as an unchangeable external constraint.

### 4.2 Invariant 2: Verifiability & Falsifiability Mandate (The Testability Law)
Every requirement must be deterministically verifiable or falsifiable through an empirical test, proof, or measurement triad $\langle M, T, C \rangle$. Unfalsifiable statements are strictly forbidden.

### 4.3 Invariant 3: Epistemic Explication & Bounded Defaults (The Assumption Law)
Every unstated environmental, behavioral, or context premise must be brought to the surface. If uncertainty is below the Clarification Gate ($C_{\text{reversal}} \le 2\text{ hours}$), the agent must bind it with an explicit sensible default in the Assumptions Register.

### 4.4 Invariant 4: Boundary & Unwanted-Path Completeness (The Negative Space Law)
Every functional capability must formally specify its behavior under invalid inputs, system limits, network partitions, and resource exhaustion using EARS *Unwanted Behavior* clauses.

### 4.5 Invariant 5: Multi-Axis Constraint Orthogonality (The ISO 25010 Law)
Functional capabilities are bounded by orthogonal quality attributes: **Performance/Efficiency**, **Security/Privacy**, **Reliability/Fault-Tolerance**, **Usability/Accessibility**, and **Maintainability/Operability**, each bounded by $\langle M, T, C \rangle$.

### 4.6 Invariant 6: Global Coherence & Triad Non-Contradiction (The Consistency Law)
No requirement within a baseline may logically contradict another. Conflicting constraints (e.g., Performance vs Security, Consistency vs Availability) must be surfaced via the **Triad Trade-off Matrix** and resolved before baseline confirmation.

### 4.7 Invariant 7: Living Baseline & Provenance Traceability (The RTM Law)
Every requirement must trace to an authentic user intent or business goal, and every downstream engineering artifact (architecture, test case, code commit) must trace back to a specific requirement ID (`REQ-xxx`). Requirements updates must be tracked as attributable `REQ-DELTA` items.

---

## 5 · Universal Archetype Adaptation

The protocol adapts to any technical domain without modifying its invariants:

* **Cloud & Distributed Microservices**: Latency percentiles ($M=\text{p99}$), network partition boundaries, eventual consistency models, and multi-tenant data isolation.
* **Embedded, Real-Time & Systems**: Deterministic execution cycle budgets, zero-allocation memory constraints, hardware interrupt response times, and watchdog failure states.
* **Compilers, Parsers & CLI Tools**: Formal grammar specifications, POSIX exit codes, standard stream behaviors (`stdin`/`stdout`/`stderr`), and deterministic memory consumption.
* **Data, ML & Pipelines**: Data drift thresholds, schema evolution rules, backpressure handling, idempotency guarantees, and throughput volume envelopes.
* **Mobile & Desktop Applications**: Offline-first synchronization, battery/thermal budgets, OS lifecycle suspend/resume transitions, and platform permission states.
* **Autonomous AI Agents & Swarms**: Model stochasticity boundaries, tool schema validation contracts, task completion verification gates, and finite loop halting guarantees.

---

## 6 · Guardrails & Anti-Patterns

- ❌ **No Clarification Interrogation**: Never ask open-ended questions about decisions that can be reasonably defaulted. Only query the user if $C_{\text{reversal}} > 2\text{ hours}$, contradictions exist, or security/data loss is at stake.
- ❌ **No Smuggled Implementation**: Never allow technology choices to masquerade as functional requirements. Strip them into candidate preferences for `solution-discovery`.
- ❌ **No Unbounded Edge Hallucination**: Never invent esoteric multi-disaster edge cases beyond the 1st-degree operational neighborhood. Push them to Non-Goals.
- ❌ **No Unfalsifiable Adjectives**: Never accept "fast", "intuitive", "scalable", or "secure" without an operational measurement triad $\langle M, T, C \rangle$.
- ❌ **No Happy-Path-Only Specifications**: Never specify positive behavior without specifying the corresponding EARS unwanted behavior mitigation.
- ❌ **No Static Stone Tablets**: Never treat a baseline as immutable when downstream discoveries reveal physical or architectural roadblocks; execute a `REQ-DELTA`.

---

## 7 · Verification & Decision Gate

Before declaring requirements analysis complete and handing off to `project-analysis`:

1. **De-Solutioning Verified**: Confirm zero implementation mechanisms are disguised as requirements.
2. **Quality Gate Cleared**: Ensure specification scores $\ge 85/100$ on the Specification Quality Rubric with no single axis $< 15/20$.
3. **EARS Dual-Path Checked**: Confirm every functional requirement contains an unwanted behavior clause.
4. **NFRs Quantified**: Confirm all quality attributes have explicit $\langle M, T, C \rangle$ triads.
5. **Non-Goals Defined**: Confirm explicit out-of-scope boundaries are documented.
6. **RTM Initialized**: Emit the confirmed baseline artifact with unique `REQ-xxx` identifiers.
