---
name: system-architecture
description: >-
  Universal system architecture, structural design, and trade-off protocol.
  Use when designing new systems, decomposing modules, defining state authority,
  scaling distributed workloads, or conducting architecture reviews across any language
  or paradigm. Do NOT activate for physical resource provisioning (use infrastructure),
  fine-grained API endpoint design (use api-design), or local code construction (use code-quality).
  Enforces the 7 Universal Architecture Invariants, constraint-first workload envelopes,
  deterministic state authority, acyclic module DAGs, failure blast-radius containment,
  and load-bearing ADR discipline without limiting architectural creativity.
---

# System Architecture: Universal Structural Design & Trade-Off Protocol

> **Mandate**: *Architecture is the art of bound trade-offs under physical and operational constraints.* Requirements and constraints strictly precede structure. Every design must declare what it optimizes for, what it sacrifices, how it isolates failure, and where state authority resides. Never build speculative enterprise complexity for today's prototype; never permit cyclic coupling across architectural boundaries. Structural simplicity is earned through rigorous constraint analysis, not assumed by default.

---

## 1 · The 5-Phase Architectural Lifecycle

Every structural design, modular decomposition, or architectural audit task traverses this 5-phase protocol:

```mermaid
flowchart LR
    P1["1. Constraint & Workload Framing<br/>(Envelope, SLIs/SLOs, Non-goals)"] --> P2["2. Boundary & Topology Cartography<br/>(Domain seams, State authority, DAG)"]
    P2 --> P3["3. Resilient Failure & Concurrency<br/>(Fault domains, Blast radius, Degraded modes)"]
    P3 --> P4["4. Trade-Off Discipline & ADR<br/>(Load-bearing choices, Simplest viable topology)"]
    P4 --> P5["5. Adversarial Review & Drift Gate<br/>(Cycle audit, Layer leaks, Integrity score)"]
```

1. **Phase 1 — Constraint & Workload Framing (Requirements Before Structure)**: Formulate the quantitative **Workload Envelope** (read/write volume, payload distributions, latency ceilings, concurrency peaks, storage growth). Define hard correctness criteria (consistency, ordering, durability) and explicit **Non-Goals** to eliminate speculative scope creep before drawing a single box.
2. **Phase 2 — Boundary & Topology Cartography**: Partition the problem into **Bounded Contexts** along domain seams. Enforce **Deterministic State Authority** (single-writer or mathematically deterministic convergence). Guarantee an **Architectural Boundary DAG** where dependencies flow acyclically from delivery mechanisms to stable domain policies.
3. **Phase 3 — Resilient Failure & Concurrency Modeling**: Partition the system into isolated **Fault Domains** so that failure in one subsystem cannot cause an unconstrained cascade. Establish backpressure, circuit breaking, timeouts, jittered exponential backoff, and explicit **Degraded Operating Modes**.
4. **Phase 4 — Trade-Off Discipline & ADR Synthesis**: Apply the **Simplest Viable Topology Axiom** (never distribute what can run efficiently in-process). Formalize every load-bearing trade-off (*"We choose A over B because of C, explicitly accepting drawback D"*) and emit an immutable **Architecture Decision Record (ADR)**.
5. **Phase 5 — Adversarial Review & Drift Gate**: Subject the design to the **6 Adversarial Critic Lenses** (boundaries, cycles, state authority, blast radius, premature abstraction, agent safety). Quantify structural confidence. If unresolved trade-offs or cycle violations are detected, backtrack to Phase 2 to resolve coupling before code implementation.

---

## 2 · Progressive Disclosure (Reference Routing)

To preserve context bandwidth and prevent cognitive overload, consult reference manuals strictly on demand based on task context:

| Context Trigger | Mandatory Reference Manual | Purpose |
| :--- | :--- | :--- |
| **QPS, capacity, latency, memory limits, SLOs, Little's Law** | [`references/constraint-first-design.md`](references/constraint-first-design.md) | Workload envelope math, Little's Law ($L = \lambda W$), Amdahl's Law, Universal Scalability Law, SLO/SLI error budgets. |
| **Module seams, DB schemas, state mutation, CRDTs, cycles** | [`references/boundaries-coupling-and-data-ownership.md`](references/boundaries-coupling-and-data-ownership.md) | Domain seams, Deterministic State Authority, Acyclic Module DAGs, coupling metrics ($C_a, C_e, I, D$). |
| **Timeouts, retries, network partitions, cascading errors, backpressure** | [`references/failure-modes-and-resilience.md`](references/failure-modes-and-resilience.md) | Fault domains, circuit breakers, backpressure, idempotency keys, CAP/PACELC partition handling, graceful degradation. |
| **Architectural choices, ADRs, trade-offs, edge-case exceptions** | [`references/trade-off-discipline-and-adrs.md`](references/trade-off-discipline-and-adrs.md) | Load-bearing trade-offs, immutable ADR schema, Invariant Exception Protocol for extreme edge cases. |
| **LLMs, tools, MCP servers, subagents, prompt/token budgets** | [`references/agentic-system-architecture.md`](references/agentic-system-architecture.md) | Agentic topology matrix (Code vs Skill vs MCP vs Subagent), token economics, tool contract schemas, Zero Ambient Authority. |
| **Pre-implementation review, PR audit, refactoring gate, anti-patterns** | [`references/architectural-critique-and-anti-patterns.md`](references/architectural-critique-and-anti-patterns.md) | 6 Adversarial review lenses, cycle/leak detection, integrity scoring, anti-pattern catalog. |

---

## 3 · Adaptive Cognitive Sizing (The 6 Execution Modes)

Size your architectural effort to the scope, operational risk, and lifecycle phase of the task. Do not write essays for simple internal modules, and never skip structural scaffolding on distributed or agentic platforms:

| Mode | Trigger & Scope | Architectural Discipline | Required Output & Protocol |
| :--- | :--- | :--- | :--- |
| **`spike`** | Quick proof-of-concept, unproven algorithm exploration, disposable prototype. | Bypasses formal ADRs; verifies basic memory/process safety; isolates temporary debt behind a seam. | Code header: `[SPIKE: exploratory - target cleanup YYYY-MM-DD]`. |
| **`micro-arch`** | Internal module seam, interface contract, or single component boundary change. | Verify single responsibility, acyclic dependency direction, explicit input/output boundaries, error bubbling. | **Zero boilerplate**. 3-Line Structural Intent block directly preceding code. |
| **`system-design`** | New service, monolithic application, desktop/CLI tool, or domain module overhaul. | Full 5-phase lifecycle; domain boundaries, deterministic state authority, sequence flows, failure handling. | System Design Document (or inline architectural RFC) + targeted ADR. |
| **`distributed-scale`** | Multi-service mesh, event streaming, distributed caching, partition tolerance, high throughput. | Rigorous workload envelope math, CAP/PACELC trade-offs, network partition handling, idempotency, observability. | Distributed Architecture Spec + Workload Envelope Matrix + Failure Playbook. |
| **`agentic-system`** | Autonomous agentic swarms, MCP servers, tool ecosystems, skill hierarchies, LLM workflows. | Agentic Topology Selection (Skill vs Subagent vs MCP vs Workflow), Context/Token Budgets, Tool Contracts, Least Privilege. | Agent System Spec + Tool Contract Schemas. |
| **`critic-audit`** | Pre-implementation design review, PR architectural audit, legacy debt audit, boundary verification. | Adversarial stress-testing across 6 lenses (Coupling, Cycles, Cohesion, Leaks, Blast Radius, Premature Abstraction). | Architecture Audit Report with findings and concrete remedies. |

### The 3-Line Structural Intent Protocol (For `micro-arch` mode)
When operating in `micro-arch` mode, summarize structural intent in exactly 3 lines before emitting code:
```markdown
> **Seam**: [Component A] ➔ [Component B] via [Interface/Contract] (Direction: Acyclic DAG)
> **Authority**: [Component A] holds mutable authority; [Component B] receives immutable view / event
> **Failure Boundary**: [Local catch / Bubbled error / Degraded fallback]
```

> **Boundary**: This skill owns *logical system design* — module decomposition, state authority, and trade-off analysis. For *physical resource provisioning* (VMs, IaC, networking, drift reconciliation), activate `infrastructure` instead. For *interface schemas and endpoint protocols*, activate `api-design`.

---

## 4 · The 7 Universal Architecture Invariants

Regardless of language, framework, or runtime, every robust computational system upholds these 7 foundational invariants:

### 4.1 Invariant 1: Workload & Constraint Primacy (Anti-Astronautics)
Structure strictly follows quantified constraints, never aesthetic dogma or technology hype. A system design is invalid until its workload envelope (throughput, payload distribution, memory bounds, latency ceilings, and durability requirements) is explicitly formulated.

### 4.2 Invariant 2: Deterministic State Authority (Convergence Law)
Every piece of mutable state must have unambiguous write authority:
* **Centralized Systems**: Exactly one authoritative owner module or service. Shared mutable databases, cross-boundary direct table writes, and uncoordinated state stores are forbidden.
* **Distributed / Concurrent Systems**: If multiple nodes or threads mutate state concurrently (e.g. CRDTs, consensus protocols, lock-free queues), the system **must** provide a mathematically proven, conflict-free deterministic convergence law.

### 4.3 Invariant 3: Architectural Boundary DAG (Acyclic Dependency Law)
The dependency graph between architectural boundaries (packages, modules, namespaces, services) must form a strict Directed Acyclic Graph (DAG). Stable domain policies must never depend on volatile I/O mechanisms or external delivery details.  
*(Clarification: Encapsulated internal in-memory object graphs within a single boundary are fully permitted).*

### 4.4 Invariant 4: Fault Domain Isolation & Blast-Radius Bounding
A failure in one subsystem must never cause an unconstrained catastrophic cascade across the entire system. Every external integration, remote call, child process, or tool invocation must execute within an isolated fault domain bounded by timeouts, circuit breakers, backpressure, and explicit fallback/degradation behavior.

### 4.5 Invariant 5: Control Plane & Data Plane Decoupling
Control operations (routing, policy enforcement, orchestration, schema evolution, agent reasoning) must be decoupled from high-throughput data operations (payload processing, serialization, streaming, batch crunching). A spike in data traffic must never starve or stall control coordination.

### 4.6 Invariant 6: Load-Bearing Trade-Off Conservation
Complexity cannot be destroyed; it can only be moved, consolidated, or traded. Every architectural decision buys specific properties (latency, availability, isolation, scalability) at the direct expense of others (consistency, simplicity, operational cost). Any design claiming "zero drawbacks" is an unverified hypothesis.

### 4.7 Invariant 7: Simplest Viable Topology & Proportionality
Match structural complexity to problem reality. Do not introduce distributed services, message brokers, or multi-agent swarms when an in-process modular monolith, a local embedded database, or a deterministic script satisfies the workload envelope. Complexity must earn its keep.

---

## 5 · The Invariant Exception Protocol (Extreme Edge Cases)

When physical realities (sub-microsecond latency requirements, extreme embedded memory constraints, hard real-time worst-case execution time, or network partitions) conflict with standard architectural invariants, the system invokes this protocol:

> [!CAUTION] INVARIANT EXCEPTION PROTOCOL
> An agent may deliberately bypass an architectural invariant (e.g., bypassing an abstraction boundary for CPU cache alignment, or violating strong consistency during a network partition) **IF AND ONLY IF**:
> 1. **Physical Constraint Citation**: Explicitly cites the hardware/runtime constraint.
> 2. **Boundary Containment**: Isolates the violation inside a strictly quarantined module behind an opaque boundary.
> 3. **Micro-ADR**: Records the trade-off and accepted technical liability in an immutable record.

---

## 6 · Universal Archetype Adaptation

The 7 invariants adapt dynamically across every software archetype:

* **Modular Monoliths & Monorepos**: Enforce boundary encapsulation via private package visibility or internal interfaces; strictly forbid circular package imports; execute transactions within single process boundaries.
* **Distributed Cloud & Microservices**: Use explicit network contracts (Protobuf/gRPC/OpenAPI); enforce single-writer data ownership; manage partial failure via circuit breakers and idempotency keys.
* **Embedded, Systems & CLIs**: Model workload envelopes against hardware limits (RAM, stack depth, CPU cycles); decouple flag parsing from core business engines; handle signals and process exit codes deterministically.
* **Frontend & Client Applications**: Enforce unidirectional data flow; decouple UI rendering from async network synchronization; provide optimistic updates with deterministic rollback.
* **Autonomous AI Agents & Swarms**: Apply the Agentic Topology Matrix (Code vs Skill vs MCP vs Subagent); enforce strict context token budgeting; scope tool capability tokens with Zero Ambient Authority.
* **Data, Stream & Batch Pipelines**: Enforce schema validation at ingress; decouple stream coordination from worker processing; ensure all transform stages are idempotent to tolerate retry without duplication.

---

## 7 · Guardrails & Anti-Patterns

- ❌ **No Premature Microservices**: Never decompose an application into distributed network services before hitting concrete organizational or workload scaling limits.
- ❌ **No Cyclic Module Coupling**: Never allow package A to import package B while package B imports package A across architectural boundaries.
- ❌ **No Shared Mutable Database Integration**: Never allow two distinct services to write directly to the same database tables. Data sharing across services occurs via contracts or events.
- ❌ **No Unstated Trade-Offs**: Never propose an architecture claiming "pure upside". State the latency, complexity, or operational penalty explicitly.
- ❌ **No Architecture Astronautics**: Never introduce enterprise event brokers, multi-region replication, or multi-agent swarms for simple CRUD or localized utilities.
- ❌ **No Unbounded Agent Tool Capabilities**: Never give an AI agent ambient root authority or unrestricted mutating access without confirmation gates.

---

## 8 · The Clean Architecture Stopping Contract

An architectural task is strictly **COMPLETE** only when:
1. **Workload Envelope Quantified**: Throughput, latency target, and capacity constraints are explicitly defined.
2. **Boundary Cartography Verified**: Every module seam has a clear acyclic dependency direction, and state authority is unambiguously assigned.
3. **Failure Playbook Documented**: Blast radius is contained with timeouts, circuit breakers, and degraded fallbacks.
4. **ADR Synthesized**: Load-bearing trade-offs are committed to an immutable record, including any Invariant Exceptions.
5. **Adversarial Gate Passed**: Structural integrity verified across review lenses with zero unresolved circular dependencies. If structural constraints prevent a passing review, backtrack to Phase 2 to refactor boundaries before code mutation.
