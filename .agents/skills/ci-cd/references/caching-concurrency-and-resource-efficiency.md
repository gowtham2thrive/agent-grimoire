# Caching, Concurrency Control & Resource Efficiency

> **Mandate**: *An un-cached pipeline is an economic failure; a wrongly-cached pipeline is an integrity failure. CI/CD engineering balances blistering speed with mathematical isolation and resource conservation.*

---

## 1 · Cache Key Mathematics & Deterministic Hashing

Build caching avoids re-downloading dependencies and re-compiling unmodified translation units.  
A valid cache key is an injective hash of all inputs that determine the cached output:

$$\text{CacheKey} = Hash(\text{OS} + \text{Arch} + \text{ToolchainVersion} + \text{LockfileHash} + \text{ConfigHash})$$

### The Exact Match vs Prefix Fallback Strategy
Pipelines utilize a tiered lookup strategy:
1. **Primary Exact Key**: Matches the exact current lockfile hash.
   `v1-deps-linux-x64-node20-b49f8a...`
2. **Secondary Prefix Fallback (Restore-Key)**: If exact lockfile hash misses (e.g. 1 package added), restore the nearest prior cache prefix to benefit from partial compilation or dependency reuse:
   `v1-deps-linux-x64-node20-`

```mermaid
flowchart TD
    Start["Job Starts: Fetch Cache"] --> Lookup{"Exact Key Exists?"}
    Lookup -->|Yes (Hit)| RestoreFull["Restore Exact Cache<br/>(Zero Download Time)"]
    Lookup -->|No (Miss)| PrefixLookup{"Prefix Fallback Exists?"}
    PrefixLookup -->|Yes (Partial)| RestorePartial["Restore Nearest Cache<br/>(Delta Download Only)"]
    PrefixLookup -->|No (Cold)| Fresh["Full Clean Download & Compile"]
    
    RestoreFull --> Execute["Execute Build / Test"]
    RestorePartial --> Execute
    Fresh --> Execute
    
    Execute --> Modified{"Lockfile or Build Changed?"}
    Modified -->|Yes| Save["Save New Cache Key"]
    Modified -->|No| Skip["Skip Cache Write"]
```

---

## 2 · The Hierarchical Cache Scoping Lattice (Cache Poisoning Defense)

Allowing untrusted feature branches or pull requests to overwrite mainline cache entries enables **Cache Poisoning**—where malicious or corrupted binaries infiltrate subsequent builds.

### The Scope Isolation Rules
1. **Mainline Authority**: The `main` or `master` branch creates the authoritative base cache.
2. **Branch Read-Only Inheritance**: Feature branches (`feat/*`, `fix/*`) and PRs can **read** from the mainline cache to accelerate their initial run.
3. **Branch Write Isolation**: Feature branches may only **write** to their own branch-scoped cache namespace. They are mathematically prohibited from mutating or overwriting mainline cache entries.
4. **Fork Complete Read/Write Isolation**: Untrusted fork pull requests cannot read or write private repository caches.

$$\text{CanRead}(\text{Branch}, \text{CacheEntry}) \iff \text{Owner}(\text{CacheEntry}) \in \{\text{Branch}, \text{Mainline}\}$$
$$\text{CanWrite}(\text{Branch}, \text{CacheEntry}) \iff \text{Owner}(\text{CacheEntry}) = \text{Branch}$$

---

## 3 · Controlled Concurrency & Auto-Cancellation

When a developer pushes 3 successive commits in 2 minutes, running 3 full CI suites concurrently burns compute and delays feedback.

### 3.1 Branch-Level Preemption (`cancel-in-progress`)
Any new push to a feature branch or pull request must atomically preempt and terminate obsolete running pipelines on that ref:
$$\text{Push}(C_{t+1}, \text{Ref}) \implies \text{Cancel}(\text{Run}(C_t, \text{Ref}))$$

### 3.2 Production Deployment Serialization
Conversely, deployments to staging or production environments must **NEVER** cancel in-progress runs; they must execute with **FIFO Queueing** or atomic concurrency locking:
$$\text{ConcurrencyGroup}(\text{Production}) \implies \text{QueueMax} = 1 \land \text{CancelInProgress} = \text{false}$$

| Environment / Ref | Concurrency Policy | Rationale |
| :--- | :--- | :--- |
| **Pull Request / Feature Branch** | `cancel-in-progress: true` | Only the latest commit matters for code review and merge gating. |
| **Mainline Integration** | `cancel-in-progress: false` (Queued) | Every integrated commit must verify build provenance and artifact generation. |
| **Staging / Production Deploy** | `cancel-in-progress: false` (Serialized Lock) | Concurrent mutations against the same target environment cause race conditions and state corruption. |

---

## 4 · Resource Sizing, Cost Awareness & Timeout Safeguards

Runners must be sized according to workload characteristics to eliminate compute waste and prevent runaway costs:

### 4.1 Memory & CPU Right-Sizing
- **Static Lint / Types**: Low compute (1–2 vCPUs, 2–4 GB RAM). Sizing a 16-core runner for linting is pure waste.
- **Large Compilations (Rust / C++ / Go)**: Compute-bound (8–16 vCPUs, fast NVMe, high RAM for parallel translation units).
- **End-to-End Browser Suites**: Memory-bound (Headless browsers require $\ge 2\text{ GB}$ per browser worker).

### 4.2 Explicit Job Timeouts
Every single job in a pipeline must specify an explicit, bounded timeout:
$$\forall v \in \mathcal{V}, \quad \text{Timeout}(v) \le 2 \times P_{99}(\text{Duration}(v))$$
* **Prohibition**: Running jobs with default unlimited timeouts (e.g. 6 hours). Any hung test process or network deadlock must be forcibly terminated within a bounded window (typically 10–20 minutes).
