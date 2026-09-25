# Trade-Off Discipline, ADRs, and the Invariant Exception Protocol

> **Mandate**: There are no solutions in software architecture; there are only trade-offs. An architectural proposal that lists only advantages is marketing, not engineering. Every structural choice buys specific capabilities at the direct expense of complexity, latency, operational burden, or financial cost. Formalize decisions in durable Architecture Decision Records (ADRs).

---

## 1 · The Load-Bearing Trade-Off Theorem

Complexity is a conserved quantity (Tesler’s Law). When an architecture introduces an abstraction, framework, or distributed layer, it does not destroy complexity—it transforms it into operational, cognitive, or networking overhead.

```markdown
Architectural Balance Sheet:
    + Capabilities Gained:   [What the new structure enables]
    - Liabilities Incurred:  [Operational burden, network hops, cognitive load, failure modes]
    = Net Strategic Value:   [Must strictly exceed the status quo]
```

### 1.1 The Classic Trade-Off Triangles
1. **Consistency vs Availability vs Latency (CAP / PACELC)**: Strong consistency requires network synchronization; eventual consistency accepts transient read anomalies.
2. **Normalization vs Query Performance**: Normalized schemas eliminate write anomalies but require expensive joins; denormalized schemas speed up reads but require multi-table write synchronization.
3. **Microservices vs Monolith**: Microservices buy organizational autonomy and isolated deployment at the cost of distributed networking, network latency, distributed tracing, and eventual consistency.
4. **Generalization vs Velocity (YAGNI)**: Highly generalized plugin architectures support hypothetical future requirements at the cost of immediate development friction and cognitive overhead.

---

## 2 · The Immutable Architecture Decision Record (ADR) Schema

Store ADRs in the repository under `docs/adr/` or `.agents/adrs/` as numbered, append-only markdown documents (`ADR-001-title.md`).

```markdown
# ADR-001: [Concise Decision Title]

## Status
[PROPOSED | ACCEPTED | DEPRECATED | SUPERSEDED by ADR-XXX]

## Context & Problem Statement
What is the business or technical context? What problem are we solving?
What are the hard constraints and quantified workload envelopes ($QPS$, latency, memory)?

## Decision Drivers (Forces)
* Driver 1: [e.g. Must support 5,000 writes/sec with P99 < 50ms]
* Driver 2: [e.g. Small 3-person engineering team with minimal DevOps bandwidth]
* Driver 3: [e.g. Zero tolerance for financial transaction loss (RPO = 0)]

## Considered Options
* **Option 1 (Simplest Viable Alternative)**: [e.g. Modular Monolith with Single PostgreSQL Primary]
* **Option 2**: [e.g. Event-Driven Microservices with Apache Kafka]
* **Option 3**: [e.g. Serverless Functions with DynamoDB]

## Decision Outcome
**Chosen Option**: Option 1: Modular Monolith with PostgreSQL.

### Justification & Load-Bearing Rationale
Why did this option win? Connect the choice directly to the decision drivers and workload envelope:
*"PostgreSQL easily handles 5,000 writes/sec on modern SSD hardware with WAL tuning. Choosing Kafka/Microservices would introduce distributed partition management that our 3-person team cannot operationally sustain."*

## Consequences & Accepted Liabilities
Every honest ADR must declare both sides of the coin:
* **Positive Consequences (Gains)**:
  - Single atomic database transactions eliminate distributed two-phase commit overhead.
  - Zero network hop latency between domain modules.
  - Simple local developer setup and hermetic testing.
* **Negative Consequences (Accepted Liabilities)**:
  - All domain modules share the same database CPU/memory pool (requires query isolation and resource quotas).
  - A bug causing process memory exhaustion crashes the entire application instance.
  - Vertical scaling ceiling at ~50,000 writes/sec (acceptable based on 3-year growth projections).
```

---

## 3 · The Invariant Exception Protocol (Extreme Edge Cases)

Standard architectural invariants (such as abstracting I/O behind ports, decoupling modules, or enforcing strong consistency) sometimes conflict directly with physical constraints.

### 3.1 Legitimate Exception Drivers
1. **High-Frequency Trading (HFT) & Real-Time DSP**: Microsecond or nanosecond latency ceilings make virtual function dispatch, heap allocation, or interface abstraction layers prohibitive. Data must be aligned directly to CPU L1/L2 cache lines.
2. **Hard Real-Time Embedded Systems**: Microcontrollers with 32KB RAM or avionics where Worst-Case Execution Time (WCET) must be mathematically bounded; dynamic memory allocation is strictly banned.
3. **Partition-Tolerant Edge Meshes**: Offline-first mobile devices or submarines where physical network links are severed for hours; strong consistency is physically impossible.

### 3.2 Micro-ADR Schema for Invariant Exceptions
When an agent or engineer deliberately violates an invariant, it must emit a Micro-ADR inline or in the code header:

```markdown
> [!CAUTION] ARCHITECTURAL INVARIANT EXCEPTION: [VIOLATED INVARIANT]
> * **Violated Invariant**: Invariant 3 (Architectural Boundary DAG) / Invariant 1 (Ports & Adapters)
> * **Physical Driver**: Sub-microsecond latency ceiling ($P_{99} \le 800\text{ns}$) precludes dynamic interface dispatch.
> * **Quarantine Seam**: Confined strictly inside `src/engine/fast_path/`. No public API leaks this structure.
> * **Accepted Liability**: Hard coupling between fast path and concrete memory layout; requires manual cache-line invalidation tests.
```

---

## 4 · Anti-Drift Verification (Keeping ADRs Alive)

Documentation decays when code evolves while ADRs remain static.

### 4.1 Drift Detection Protocol
During Phase 5 (Adversarial Review) or during PR audits:
1. **Compare Implementation to Stated ADRs**: Does the new module introduce an external HTTP dependency that violates `ADR-004: In-Process Monolithic Boundaries`?
2. **Trigger Supercession**: When requirements change significantly (e.g. workload envelope exceeds original 3-year projection by $10\times$), do **not** mutate the historical ADR. Author a new ADR that explicitly marks the prior one as `SUPERSEDED`.
3. **Automated Boundary Linting**: In languages supporting architecture tests (e.g. ArchUnit, `go-critic`, Rust module visibility, or custom dependency AST checks), encode ADR boundary rules into CI test assertions to fail builds on illegal cross-module imports.
