# Merge & Integration Verification

Deep reference for Section 7 of the main [`SKILL.md`](../SKILL.md). Covers the merge protocol, conflict resolution strategies, holistic integration testing, and the synthesis report template.

---

## 1. Merge Protocol

### Step 1 — Diff Inspection

Before merging any worker branch, inspect its changes against the baseline:

```bash
# List files changed by the worker
git diff --name-only <baseline>..<worker-branch>

# Full diff for review
git diff <baseline>..<worker-branch>
```

**Verify**:
- The worker only modified files within its `allowed_paths`.
- No unauthorized changes to `forbidden_paths` files (shared configs, other workers' modules, coordination files).
- No unintended deletions or renames.

### Step 2 — Cross-Worker Conflict Detection

Before merging multiple workers, check for semantic and textual conflicts:

```bash
# Dry-run merge to detect textual conflicts
git merge --no-commit --no-ff agent/worker-a
git merge --abort  # inspect, then abort

# Compare two workers' changes for overlap
git diff agent/worker-a..agent/worker-b -- <shared-interface-file>
```

**Semantic conflicts to watch for** (not caught by Git's textual merge):
- Two workers imported different versions of a dependency.
- Two workers defined the same function name or type with incompatible signatures.
- Two workers created test fixtures that use the same port number, database name, or file path.
- Two workers made contradictory assumptions about shared API behavior.

### Step 3 — Deliberate Reconciliation

When conflicts are detected:

1. **Textual conflicts**: Resolve manually in the merge, preserving the intent of both workers.
2. **Semantic conflicts**: The orchestrator decides which worker's approach wins, then:
   - Applies the fix to the merged tree, or
   - Re-dispatches one worker with corrective instructions and the other worker's output as context.
3. **Unauthorized file mutations**: Revert the unauthorized changes and soft-restart the offending worker with narrower scope.

### Step 4 — Branch Merge

Once conflicts are resolved:

```bash
git checkout main  # or integration branch

# Merge each worker with --no-ff to preserve branch topology
git merge --no-ff agent/auth-api -m "Merge agent/auth-api: JWT middleware + unit tests"
git merge --no-ff agent/profile-ui -m "Merge agent/profile-ui: Avatar upload component"
```

**Alternative — Squash Merge**: If the worker's intermediate commits are noisy (WIP, soft restart debris), squash to a clean single commit:

```bash
git merge --squash agent/auth-api
git commit -m "feat(auth): JWT middleware with RS256 signing + 12 unit tests"
```

Match the repository's prevailing merge strategy (`git log --oneline -10` to detect conventions).

---

## 2. Integration Verification Gate

**Individual worker validation is necessary but insufficient.** After merging all workers, run holistic checks against the combined result:

### 2.1 Test Suite

Run the **full project test suite**, not just scoped worker tests:

```bash
# Node.js / JavaScript
npm test

# Python
pytest

# Rust
cargo test

# Go
go test ./...
```

Workers may individually pass their scoped tests while the combined result fails due to:
- Import collisions or circular dependencies.
- Shared state pollution in test fixtures.
- Interface contract mismatches (function signature changes one worker didn't account for).

### 2.2 Type Checking

Run the project's type checker across the entire codebase:

```bash
# TypeScript
npx tsc --noEmit

# Python (mypy)
mypy src/

# Rust (happens during cargo test, but explicit check)
cargo check
```

### 2.3 Build Verification

Confirm the project compiles and bundles cleanly:

```bash
# Node.js / Vite / Next.js
npm run build

# Rust
cargo build --release

# Go
go build ./...
```

### 2.4 Lint & Format (if applicable)

```bash
# ESLint + Prettier
npx eslint . && npx prettier --check .

# Python
ruff check . && ruff format --check .

# Rust
cargo clippy && cargo fmt --check
```

### 2.5 Gate Decision

| Outcome | Action |
| :--- | :--- |
| **All checks pass** | Orchestration complete. Emit synthesis report. Clean up worktrees. |
| **Tests fail** | Identify which worker's changes caused the failure. Re-dispatch that worker with the test failure output. |
| **Type errors** | Identify the interface mismatch. Update `DECISIONS.md` with the corrected contract. Re-dispatch affected workers. |
| **Build fails** | Diagnose the build error. Likely an import issue or missing dependency. Fix in the merged tree or re-dispatch. |

---

## 3. Synthesis Report Template

After successful integration, emit a structured report to the user:

```markdown
## Orchestration Synthesis

### Objective
[Original user request — one sentence]

### Topology
[parallel / phased / single_worker] — [brief rationale]

### Foundation
[What was implemented and committed before fan-out, or "Not required"]

### Worker Results

| Worker | Scope | Status | Key Deliverable | Restarts |
|--------|-------|--------|-----------------|----------|
| auth-api | `src/auth/**` | ✅ Complete | JWT middleware + 12 unit tests | 0 |
| profile-ui | `src/components/profile/**` | ✅ Complete | Avatar upload + Storybook story | 1 (soft) |

### Integration Verification
- **Tests**: ✅ 247/247 passing
- **Types**: ✅ No errors
- **Build**: ✅ Clean production bundle
- **Lint**: ✅ No violations

### Conflicts Resolved
- [Description of any merge conflicts and how they were resolved]
- [Or "None — clean merge"]

### Notable Decisions
- [Any scope adjustments, architectural choices, or contract revisions made during orchestration]
```

### Report Proportionality

- **Simple orchestration** (2 workers, no conflicts, all green): 5–10 line summary.
- **Complex orchestration** (4+ workers, conflicts resolved, restarts occurred): Full structured report with conflict details and decision rationale.
- **Failed orchestration** (parked workers, integration gate failure): Detailed diagnostic with root cause analysis, recovery tag references, and recommended next steps.
