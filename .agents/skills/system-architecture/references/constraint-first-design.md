# Constraint-First System Design: Workload Envelopes & Capacity Physics

> **Mandate**: Architecture begins with physical and operational constraints, never with diagrams or technology preferences. A design without a quantified workload envelope is an unverified hypothesis. Calculate capacity, latency ceilings, concurrency limits, and error budgets before proposing a single structural component.

---

## 1 · The Workload Envelope Formula

A complete workload envelope quantifies both average operational throughput and hostile peak conditions across six orthogonal dimensions:

```markdown
Workload Envelope = {
    Throughput: [QPS_read, QPS_write, Peak_Multiplier],
    Payload:    [P50_size, P99_size, Max_ingress_bytes],
    Latency:    [P50_target_ms, P99_ceiling_ms, Max_timeout_ms],
    Memory:     [Working_set_mb, Cache_budget_mb, Leak_threshold],
    Storage:    [Initial_volume_gb, Growth_rate_gb_per_month, TTL_retention],
    Durability: [RPO_seconds, RTO_seconds, Availability_target]
}
```

### 1.1 Quantifying Peak Multipliers
Systems rarely fail under average load; they fail during peak surges and sudden concurrency spikes.
$$\text{Peak QPS} = \text{Average QPS} \times \text{Surge Factor } (\sigma)$$
* Standard web applications: $\sigma \approx 3 \text{ to } 5\times$
* E-commerce flash sales / breaking news: $\sigma \approx 10 \text{ to } 50\times$
* Autonomous agent tool bursts: $\sigma \approx 20\times$ (parallel subagent tool calls)

---

## 2 · Mathematical Laws of Capacity & Throughput

### 2.1 Little's Law (Queueing Physics)
Little’s Law establishes the fundamental invariant between concurrency, arrival rate, and latency:
$$L = \lambda \cdot W$$
Where:
* $L$ = Number of concurrent requests inside the system (in-flight concurrency).
* $\lambda$ = Arrival rate (requests per second / $QPS$).
* $W$ = Average residence time / latency per request (seconds).

> **Architectural Corollary**: If your system handles $\lambda = 5,000 \text{ req/s}$ with an average latency of $W = 200\text{ms} = 0.2\text{s}$, your system **must** support $L = 5,000 \times 0.2 = 1,000$ concurrent in-flight connections/threads without memory exhaustion. If latency degrades to $1\text{s}$, required concurrency explodes to $5,000$, risking immediate thread-pool starvation.

### 2.2 Amdahl's Law (Concurrency Bottlenecks)
Amdahl's Law calculates the theoretical speedup limit when parallelizing workloads:
$$S(s) = \frac{1}{(1 - p) + \frac{p}{s}}$$
Where:
* $p$ = Fraction of the workload that is parallelizable.
* $s$ = Speedup factor (e.g. number of CPU cores or worker threads).
* $(1 - p)$ = The strictly sequential / serial bottleneck (e.g. locks, database writes, global state).

> **Architectural Corollary**: If just $5\%$ of your task requires sequential synchronization ($(1 - p) = 0.05$), no matter how many cores, machines, or worker agents you throw at it ($s \to \infty$), maximum theoretical speedup can **never exceed $20\times$**. Eliminate serial coordination barriers first before adding horizontal workers.

### 2.3 Universal Scalability Law (Gunther's USL)
The Universal Scalability Law accounts for both queueing contention and cross-node cache-coherency overhead:
$$C(N) = \frac{N}{1 + \alpha(N - 1) + \beta N(N - 1)}$$
Where:
* $N$ = Concurrency or worker count.
* $\alpha$ = Contention coefficient (waiting for shared resources / locks).
* $\beta$ = Coherency coefficient (cross-talk delay to maintain consistent state).

When $\beta > 0$, adding nodes beyond a critical threshold **reduces** total throughput (retrograde scaling). Distributed consensus, two-phase commits, and shared databases hit USL retrogrades rapidly.

---

## 3 · SLIs, SLOs, and Error Budgets

Never specify fuzzy targets like "high performance" or "reliable". Formulate formal operational objectives:

| Metric | Definition | Mathematical Example |
| :--- | :--- | :--- |
| **SLI** (Service Level Indicator) | Quantified real-time measurement of service behavior. | $\frac{\text{Successful Requests (latency } \le 200\text{ms})}{\text{Total Valid Requests}} \times 100\%$ |
| **SLO** (Service Level Objective) | Target reliability committed to internally. | $99.9\%$ over a rolling 30-day window. |
| **Error Budget** | The allowed margin of failure ($1 - \text{SLO}$). | $0.1\% \implies 43.8 \text{ minutes of total outage/unavailability per month}$. |
| **RPO** (Recovery Point Objective) | Maximum acceptable data loss duration. | $\le 0 \text{s}$ for financial ledgers; $\le 5\text{m}$ for analytical logs. |
| **RTO** (Recovery Time Objective) | Maximum acceptable duration to restore service. | $\le 60\text{s}$ via automated failover; $\le 1\text{hr}$ for disaster recovery. |

---

## 4 · Explicit Non-Goals Formulation

The fastest way to derail a system architecture is to design for theoretical requirements that do not exist. Every architecture specification **must** include an explicit "Non-Goals" section:

```markdown
### Explicit Non-Goals (Scope Defenses)
- **Non-Goal 1**: Sub-millisecond latency. (This is a batch analytics tool; 500ms P99 is entirely acceptable).
- **Non-Goal 2**: Multi-region active-active database replication. (The business operates strictly in a single geographic zone; single-region primary + read replicas avoids distributed consensus overhead).
- **Non-Goal 3**: Generalized plugin framework. (YAGNI; we have 2 known integrations, not a public third-party marketplace).
```

By explicitly codifying what the system is **not** attempting to solve, the agent defends against architecture astronautics and premature complexity.
