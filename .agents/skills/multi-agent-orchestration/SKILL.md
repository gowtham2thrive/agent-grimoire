---
name: multi-agent-orchestration
description: >-
  Coordinate parallel or phased sub-agents with isolated worktrees, durable
  task contracts, and holistic integration verification. Use when the user asks
  to "split work across agents", "parallelize", "fan out", "spawn workers",
  "orchestrate sub-agents", or when a task has genuinely independent modules
  that bottleneck a single agent's context window. Also use when evaluating
  whether a complex task warrants multi-agent decomposition at all.
  Do not activate for tightly coupled or sequential single-agent tasks (use planning
  or code-quality), or for root-cause diagnosis within a single failing agent (use failure-recovery).
---

# Multi-Agent Orchestration: Autonomous Coordination Protocol

> **Mandate**: Decompose only when independence is real, settle shared foundations before spawning, isolate every worker's mutation surface, supervise with bounded recovery, and verify the integrated result holistically — never trust individual "all green" reports as proof of system correctness.

---

## 1 · Adaptive Orchestration Lifecycle

Never spawn agents speculatively. Execute every orchestration task through this pipeline:

1. **User Intent**: Decode the actual objective — a feature, refactor, audit, or research task — rather than pattern-matching keywords to a fixed topology.
2. **Independence Analysis**: Map the task's internal dependency graph. Identify which subtasks share mutable state (files, schemas, configs) and which are genuinely decoupled.
3. **Topology Selection**: Classify the task into exactly one execution mode (see Section 2).
4. **Foundation Settlement**: If shared primitives exist, implement and commit them before spawning parallel workers (see Section 3).
5. **Task Contract Formulation**: Write the durable coordination contract — boundaries, validation commands, and deliverable definitions — before dispatch (see Section 4).
6. **Isolated Dispatch**: Create physically separated workspaces (Git worktrees or non-overlapping path boundaries) and launch workers with self-contained prompts (see Section 5).
7. **Supervision & Liveness**: Monitor progress, detect stalls, and apply bounded recovery — soft restarts, then hard restarts, then park (see Section 6).
8. **Reconciliation & Holistic Verification**: Merge worker outputs, resolve conflicts, and run end-to-end integration checks against the full project (see Section 7).
9. **Proportional Reporting**: Concise structured synthesis for the user — not raw worker logs.

---

## 2 · Topology Selection

Before decomposing, classify the task into exactly one mode. Two questions decide it: *can subtasks mutate independently?* and *do they share foundational contracts?*

| Mode | When to Use | Orchestration Cost |
| :--- | :--- | :--- |
| **`direct`** | Tightly coupled, single-file, sequential, or debugging work. One agent suffices. | None — **bypass this skill entirely**. |
| **`single_worker`** | Substantial bounded work benefiting from background delegation, but no parallelism. | Minimal — one worktree, one contract, one merge. |
| **`parallel`** | Genuinely independent boundaries with disjoint file ownership. No subtask depends on another's mid-flight output. | Moderate — N worktrees, N contracts, one merge pass. |
| **`phased`** | Shared foundations (schemas, types, manifests) must land first, then independent leaves fan out. | High — foundation commit + parallel fan-out + merge. |
| **`review_gate`** | High-risk architectural changes requiring independent critique before any implementation. | Variable — design review loop before entering another mode. |

**Decision heuristic**: If two subtasks must touch the same file, or one needs another's output mid-flight, either merge them into one unit or sequence them — do not parallelize. One focused agent beats five stepping on each other.

*(Deep topologies, trade-off analysis, and visual diagrams: [`references/topologies.md`](references/topologies.md).)*

> **Boundary**: This skill owns *parallel multi-agent coordination* — topology selection, worker isolation, bounded supervision, and holistic integration. For *single-agent sequential task decomposition* into ordered steps with verification triads, activate `planning` instead.

---

## 3 · Foundation-First Discipline

When the topology is `phased`, identify all shared primitives that parallel workers will depend on:

- Package manifests (`package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`).
- Shared type definitions, interfaces, or schema files.
- Database migrations and seed data.
- Router registrations, middleware pipelines, or dependency injection containers.
- Test configuration and fixture scaffolding.

**The rule is absolute**: implement, verify (tests pass, types check), and commit these foundation changes to the primary working tree *before* spawning any worker. Workers spawn from this committed baseline. Spawning before foundation settlement guarantees architectural divergence and un-mergeable conflicts.

---

## 4 · Task Contracts & Worker Isolation

### 4.1 The Coordination Contract

Before dispatching any worker, formulate a structured contract defining:

- **Objective**: What the worker builds, in one sentence.
- **Allowed Paths**: Files and directories the worker *may* create or modify (glob patterns).
- **Forbidden Paths**: Files the worker *must not* touch (shared configs, other workers' modules, coordination files).
- **Read-First Files**: Interfaces, types, or architecture docs the worker should read before writing code.
- **Validation Command**: A deterministic, scoped test command the worker runs to verify its own output (e.g., `["npm", "test", "--", "src/auth"]`).
- **Definition of Done**: Exact behavioral assertions and test expectations.

*(Full schema definitions and examples: [`references/task-contract-schema.md`](references/task-contract-schema.md).)*

### 4.2 Isolation Mechanisms

Workers must be physically prevented from corrupting each other's state:

- **Git Worktrees** (preferred for Git repositories): Each worker operates in a separate worktree checked out to a dedicated branch from the foundation baseline. This eliminates shared git index corruption entirely.
- **Isolated Workspace Directories / Sandboxes** (for non-Git or multi-repo setups): Each worker receives an isolated copy or containerized sandbox of the necessary files, returning diffs upon completion.
- **Branch / Worktree Isolation**: Workers develop on isolated feature branches, avoiding shared staging areas.
- **Path-Glob Boundaries** (lightweight alternative): Workers share a single tree but are assigned non-overlapping `allowed_paths`. Violations are detected at review time and trigger corrective restarts.

*(Worktree lifecycle commands, branch conventions, and cross-platform caveats: [`references/isolation-and-worktrees.md`](references/isolation-and-worktrees.md).)*

### 4.3 Context Hygiene

Workers do not inherit the orchestrator's chat history. Every worker prompt must be **fully self-contained**: objective, constraints, file paths, interfaces, validation command, and done criteria. If context cannot be serialized into the prompt, it cannot be delegated.

Workers absorb all execution noise (compiler logs, test output, search results) in their own context. They return **only structured deliverables** — a diff summary, a status report, or a produced artifact. The orchestrator never ingests raw worker noise.

*(Worker prompt template with substitution variables: [`examples/worker-prompt-template.md`](examples/worker-prompt-template.md).)*

---

## 5 · Dispatch Protocol

### 5.1 Roles

- **Orchestrator**: Decomposes, writes contracts, dispatches, supervises, merges, and verifies. **Writes zero feature code during the orchestration phase.**
- **Workers**: Execute exactly one bounded unit. Report structured results. Have no authority beyond their assigned scope.
- **Reviewer** (optional): An independent agent that inspects the merged result against the original spec. Never reviews its own work.

### 5.2 Launching

- Launch all independent workers in a **single batch** so they run concurrently. Do not spawn-await-spawn sequentially — that eliminates the parallelism benefit.
- Cap concurrency at the true independence width. More workers than genuinely independent units adds merge cost with zero speedup.
- Each worker receives its rendered prompt via a dedicated file or stdin — never via shared conversation state.

---

## 6 · Supervision, Liveness & Bounded Recovery

Once workers are dispatched, the orchestrator monitors progress. The supervision loop must be **bounded** — never burn tokens in infinite retry cycles.

### 6.1 Liveness & Progress Detection

- **Liveness**: Is the worker process still running and producing log output?
- **Progress**: Is the worker producing meaningful state changes — new commits, file diffs, or test runs?
- A worker can be alive (producing logs) but not progressing (stuck in a loop). Both signals matter.

### 6.2 Escalation Ladder

| Escalation Level | Trigger | Action |
| :--- | :--- | :--- |
| **Level 1 — Soft Restart** | First progress timeout or validation failure. | Kill process, preserve uncommitted work (WIP commit), respawn with focused error context and test output. (Invoke `failure-recovery` for localized hypothesis diagnosis). |
| **Level 2 — Hard Restart** | Second failure or fundamental approach mismatch. | Kill process, tag current state for recovery (`recovery/<agent>/<timestamp>`), reset worktree to baseline, respawn with narrowed scope and corrective instructions. |
| **Level 3 — Park** | Third failure or restart budget exhausted. | Mark agent as `needs_attention`. Preserve worktree for human inspection. **Do not continue retrying.** |

**Restart cap**: Every agent has a maximum restart budget (default: 3). Past this limit, further respawns are blocked. This prevents runaway API credit consumption and context window pollution.
> **Boundary Note**: `failure-recovery` governs the root-cause analysis, error classification, and localized fix hypothesis for an individual agent's failure. `multi-agent-orchestration` governs supervisor-level decisions: restarting, re-assigning, or parking the worker, and reclaiming the isolated workspace.

*(Heartbeat protocols, timeout configuration, and triage runbooks: [`references/supervision-and-recovery.md`](references/supervision-and-recovery.md).)*

---

## 7 · Reconciliation & Holistic Verification

### 7.1 Merge Protocol

1. **Diff Inspection**: Review each worker's branch diff against the foundation baseline. Verify adherence to `allowed_paths` — flag any unauthorized file mutations.
2. **Conflict Detection**: Identify overlapping edits, contradictory assumptions, or drifted interface contracts across workers.
3. **Deliberate Reconciliation**: The orchestrator resolves conflicts — re-dispatch a worker with corrective instructions if its output missed scope or violated contracts.
4. **Branch Merge**: Fast-forward or merge worker branches into the target integration branch.

### 7.2 Integration Gate

**Individual worker success is necessary but not sufficient.** After merging all workers:

- Run the **full project test suite** (not scoped worker tests).
- Run **type-checking** across the entire codebase.
- Run the **build pipeline** to verify compilation.
- If the project has linting or formatting standards, verify those too.

Only after the integrated result passes all checks is the orchestration considered complete.

### 7.3 Synthesis Report

Emit a structured report for the user:

```markdown
## Orchestration Synthesis

### Objective
[Original user request, one line]

### Topology Used
[parallel / phased / single_worker]

### Worker Results
| Worker | Scope | Status | Key Deliverable |
|--------|-------|--------|-----------------|
| auth-api | src/auth/** | ✅ Complete | JWT middleware + 12 unit tests |
| profile-ui | src/components/profile/** | ✅ Complete | Avatar upload component + Storybook |

### Integration Verification
- Tests: ✅ 247/247 passing
- Types: ✅ No errors
- Build: ✅ Clean

### Notable Decisions
[Any conflicts resolved, scope adjustments, or architectural choices made during merge]
```

*(Detailed merge workflow and conflict resolution playbooks: [`references/merge-and-verification.md`](references/merge-and-verification.md).)*

---

## 8 · Anti-Patterns

These mistakes consistently destroy the value of multi-agent orchestration:

- **Premature Parallelization**: Spawning agents before confirming independence. If subtasks share mutable state, parallelizing them creates race conditions and lost work.
- **Missing Foundation**: Launching parallel workers without committing shared schemas, types, or configs first. Workers diverge on core primitives, producing un-mergeable branches.
- **Ambient Context Assumption**: Writing worker prompts that assume conversation history the worker cannot see. Workers operate in isolated contexts — all constraints must be explicit.
- **Sequential Dispatch Labeled Parallel**: Spawn → await → spawn → await is serial execution with extra overhead, not parallelism.
- **Trusting Individual Reports**: Accepting "all tests pass" from each worker without running an integration suite across the combined result. Units green individually can break together.
- **Unbounded Retries**: Restarting a failing worker indefinitely without a cap. Three failures with no progress means the decomposition or approach is wrong — not that the fourth attempt will succeed.
- **Over-Orchestration**: Using 5 agents for 2 independent units. Coordination overhead exceeds execution benefit. Match agent count to true independence width.

---

## 9 · The Clean Orchestration Stopping Contract

An orchestration engagement is strictly **COMPLETE** only when:
1. **Durable Task Completion**: All assigned worker deliverables are integrated and accounted for.
2. **Holistic Integration Passed**: Full repository test suite, static type analysis, and build pipeline execute with exit code `0` on the integrated branch.
3. **Workspace Decontamination**: All temporary worker branches, worktrees, and ephemeral scratchpad files are cleaned up or parked cleanly.
4. **Scope Integrity Verified**: Zero unauthorized file mutations occurred outside defined `allowed_paths`.
5. **Synthesis Emitted**: A clear orchestration synthesis report is provided to the user detailing results, integration status, and key decisions.
