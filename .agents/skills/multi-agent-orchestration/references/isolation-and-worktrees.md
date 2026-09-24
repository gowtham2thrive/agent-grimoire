# Isolation & Git Worktrees

Deep reference for Section 4.2 of the main [`SKILL.md`](../SKILL.md). Covers the mechanics of worker isolation — Git worktree lifecycle, branch conventions, path boundary enforcement, and cross-platform considerations.

---

## 1. Why Physical Isolation

Concurrent agents sharing a single Git working tree will corrupt each other's work:

- **Index Conflicts**: Git's staging area (`.git/index`) is a single file. Two processes running `git add` simultaneously produce undefined behavior.
- **Dirty Tree Ambiguity**: When multiple agents modify files, `git status` and `git diff` reflect a merged view that no single agent intended.
- **Merge Hell**: Without isolation, changes interleave at the file level, making conflict resolution nearly impossible.

**Git worktrees** solve this by giving each worker a separate working directory with its own index, while sharing the same repository objects. Each worktree is checked out to a dedicated branch, enabling clean merges after completion.

---

## 2. Worktree Lifecycle

### 2.1 Creation

Create one worktree per worker, branching from the foundation baseline:

```bash
# From the main repository root
git worktree add -b <branch-name> <worktree-path> <baseline>
```

**Example** (two parallel workers after a phased foundation commit):

```bash
# Foundation already committed on main
git worktree add -b agent/auth-api .agents/worktrees/auth-api main
git worktree add -b agent/profile-ui .agents/worktrees/profile-ui main
```

**Conventions**:
- **Branch naming**: `agent/<worker-name>` — clear provenance in `git log --all`.
- **Worktree path**: `.agents/worktrees/<worker-name>` — grouped under `.agents/` alongside skills and configuration.
- **Baseline**: Always branch from the latest committed foundation state. Never from a dirty or uncommitted tree.

### 2.2 Worker Execution

Each worker operates exclusively within its worktree directory. The worker's prompt specifies:
- The absolute path to its worktree as the working directory.
- `allowed_paths` relative to the worktree root.
- `forbidden_paths` relative to the worktree root.

### 2.3 Completion & Merge

After a worker reports completion and its local validation passes:

```bash
# From the main repository root
git checkout main
git merge --no-ff agent/auth-api -m "Merge agent/auth-api: JWT middleware"
git merge --no-ff agent/profile-ui -m "Merge agent/profile-ui: Avatar upload component"
```

Use `--no-ff` to preserve the branch topology in history — each worker's contribution is visible as a distinct merge commit.

### 2.4 Cleanup

After successful integration and verification:

```bash
git worktree remove .agents/worktrees/auth-api
git worktree remove .agents/worktrees/profile-ui
git branch -d agent/auth-api
git branch -d agent/profile-ui
```

**Never clean up before integration verification passes.** If integration fails, the worktrees contain the evidence needed for diagnosis.

---

## 3. Path Boundary Enforcement

For lightweight orchestration (single tree, no worktrees), enforce non-overlapping path boundaries:

### 3.1 Soft Boundaries

The orchestrator assigns `allowed_paths` and `forbidden_paths` in the task contract. Workers are *instructed* to respect these boundaries. Violations are detected during the merge/review phase.

**Detection**:
```bash
# Check if a worker modified files outside its allowed paths
git diff --name-only <baseline>..<worker-branch> | grep -v -E "<allowed_path_regex>"
```

**Response to violations**:
1. First violation → soft restart with corrective instructions.
2. Repeated violation → hard restart with narrowed scope.
3. Persistent violation → park agent; decomposition is likely wrong.

### 3.2 Hard Boundaries (Git Worktrees)

With worktrees, boundaries are physically enforced — each worker simply cannot see other workers' files unless they exist in the shared baseline. This is the preferred approach for `parallel` and `phased` topologies.

---

## 4. Cross-Platform Considerations

| Concern | macOS / Linux | Windows |
| :--- | :--- | :--- |
| **Symlinks** | Git worktrees use symlinks for `.git` files. Work reliably. | Symlinks require elevated permissions or Developer Mode. May fail silently. |
| **File Locking** | Files are not locked by editors/processes by default. | Editors and processes can hold exclusive locks, preventing Git operations. |
| **Path Length** | No practical limit. | 260-character `MAX_PATH` limit can break deeply nested worktree paths. Use short worktree directory names. |
| **Process Signaling** | POSIX `kill` and process groups work reliably for stopping workers. | `taskkill` is the equivalent but lacks process-group semantics. Workers may leave orphan child processes. |

**Recommendation**: For production multi-agent orchestration, prefer macOS or Linux. On Windows, use short worktree paths (e.g., `.agents/wt/<name>`) and verify worktree creation succeeds before dispatching workers.

---

## 5. Worktree State Preservation

### On Worker Failure

- **Soft restart**: Preserve uncommitted work via `git add -A && git commit -m "WIP: <agent-name>"` before respawning.
- **Hard restart**: Create a recovery tag (`git tag recovery/<agent>/<timestamp>`) capturing the full state, then reset the worktree to baseline.

### On Orchestration Abort

- **Do not `git reset --hard`** worktrees on abort. In-flight work is valuable diagnostic evidence.
- **Do not delete `coord/`** or coordination files. They contain the audit trail.
- Worktrees and branches persist until the user explicitly cleans up or a new `--resume` launch reuses them.
