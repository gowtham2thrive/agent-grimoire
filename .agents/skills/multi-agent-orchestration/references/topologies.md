# Execution Topologies: Decision Heuristics & Trade-Offs

Deep reference for Section 2 of the main [`SKILL.md`](../SKILL.md). Consult when the topology choice is ambiguous or when evaluating whether coordination overhead exceeds execution benefit.

---

## Topology Catalog

### 1. Direct (No Orchestration)

```
User → Single Agent → Result
```

**Use when**: The task is tightly coupled, touches few files, requires sequential reasoning, or involves interactive debugging. One agent holding the full context produces better results than fragmented coordination.

**Indicators**:
- Fewer than ~5 files involved.
- Subtasks depend on each other's intermediate output.
- The task is exploratory or diagnostic (debugging, profiling, root-cause analysis).
- Coordination overhead (contract writing, worktree setup, merge) would exceed the task itself.

**Action**: Bypass `multi-agent-orchestration` entirely. Handle in the caller session.

---

### 2. Single Worker (Delegated Background)

```
Orchestrator → [writes contract] → Worker → [returns deliverable] → Orchestrator merges
```

**Use when**: The task is substantial and bounded but has no parallelism opportunity. Delegation frees the orchestrator's context for other work or prevents context window pollution from long-running compilation, testing, or refactoring.

**Indicators**:
- One large, self-contained unit of work (e.g., migrate 40 test files from Jest to Vitest).
- The work is mechanical or repetitive, benefiting from dedicated focus.
- No independent subtasks exist — forcing parallelism would create artificial, coupled boundaries.

**Cost**: One worktree, one contract, one merge. Minimal coordination overhead.

---

### 3. Parallel (Concurrent Fan-Out)

```
Orchestrator → [writes N contracts]
    ├── Worker A (worktree-a) ──→ Deliverable A
    ├── Worker B (worktree-b) ──→ Deliverable B
    └── Worker C (worktree-c) ──→ Deliverable C
Orchestrator ← [merges all] ← Integration Gate
```

**Use when**: The task decomposes into genuinely independent units with non-overlapping file ownership. No worker needs another worker's mid-flight output.

**Indicators**:
- Each subtask touches distinct files/directories.
- No shared mutable state between subtasks.
- Independent validation commands exist for each subtask.
- Serial execution would take N× longer with no benefit from the sequential ordering.

**Cost**: N worktrees, N contracts, one merge pass. Moderate coordination overhead.

**Critical test**: If you cannot assign each worker a `forbidden_paths` list that includes every other worker's `allowed_paths`, the tasks are not truly independent. Merge them or sequence them.

---

### 4. Phased (Foundation → Fan-Out)

```
Phase 1:  Orchestrator → [implements shared foundation] → Commit baseline
Phase 2:  Orchestrator → [writes N contracts from baseline]
              ├── Worker A ──→ Deliverable A
              └── Worker B ──→ Deliverable B
Phase 3:  Orchestrator ← [merges all] ← Integration Gate
```

**Use when**: Subtasks are independent *after* shared primitives are settled, but not before. The foundation includes shared schemas, core types, package manifests, router stubs, or database migrations.

**Indicators**:
- Multiple workers would need to modify `package.json`, `tsconfig.json`, a shared `types.ts`, or a database migration.
- The architecture requires a contract (API interface, database schema) that all workers consume.
- Without the foundation, workers would independently invent incompatible versions of the shared primitive.

**Cost**: Foundation implementation + commit + N worktrees + N contracts + merge. Highest coordination overhead. Only justified when the parallel phase delivers meaningful time savings.

**Foundation examples**:
- Database schema migration + TypeScript entity types.
- Shared API client interface + mock factory.
- Package dependency installation + build config.
- Router registration stubs + middleware pipeline.

---

### 5. Review Gate (Design Critique Loop)

```
Orchestrator → [drafts design] → Reviewer Agent(s)
Reviewer(s) → [critique: blockers, risks, alternatives] → Orchestrator
Orchestrator → [reconciles feedback] → Revised design
(repeat if needed, bounded by max_iterations)
Orchestrator → [enters parallel/phased/single_worker mode]
```

**Use when**: The architectural decision carries high risk — irreversible schema changes, public API surface modifications, security-critical design choices — and benefits from independent critique before implementation begins.

**Indicators**:
- The change is difficult or expensive to reverse.
- Multiple valid architectural approaches exist with non-obvious trade-offs.
- The design affects external consumers (public APIs, database schemas used by other services).

**Cost**: One or more review iterations before entering an execution topology. Adds latency but prevents costly implementation rework.

---

## Decision Flowchart

```
Is the task tightly coupled or sequential?
├── Yes → direct (bypass this skill)
└── No →
    Can it be split into independent, non-overlapping units?
    ├── No → single_worker
    └── Yes →
        Do the units share foundational contracts (schemas, types, configs)?
        ├── Yes → phased (foundation first, then parallel)
        └── No → parallel
        
Is the architectural risk high enough to warrant independent critique?
├── Yes → review_gate (then re-enter the tree above)
└── No → proceed with selected topology
```

---

## When NOT to Orchestrate

Multi-agent orchestration has real costs. Avoid it when:

- **Coordination overhead exceeds execution time**: Writing contracts, setting up worktrees, and merging results takes longer than just doing the work.
- **The task is exploratory**: Debugging, profiling, and root-cause analysis require iterative reasoning that cannot be pre-decomposed into independent units.
- **Context is the bottleneck, not parallelism**: If the task is slow because it requires deep understanding of a single complex module, splitting it across agents fragments that understanding.
- **Fewer than 2 genuinely independent units exist**: Forcing parallelism on coupled work creates artificial boundaries and merge conflicts.
