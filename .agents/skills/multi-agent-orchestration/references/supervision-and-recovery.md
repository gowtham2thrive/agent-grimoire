# Supervision & Recovery

Deep reference for Section 6 of the main [`SKILL.md`](../SKILL.md). Covers liveness monitoring, progress detection, the escalation ladder, heartbeat protocols, and triage runbooks for parked agents.

---

## 1. Two Dimensions of Worker Health

| Dimension | What It Measures | Signal Source |
| :--- | :--- | :--- |
| **Liveness** | Is the worker process alive and producing output? | Process status, log file modification time. |
| **Progress** | Is the worker making meaningful forward movement? | New git commits, file diffs, test executions. |

A worker can be **live but not progressing** (stuck in a loop, retrying the same compilation error indefinitely). Both signals must be tracked independently.

---

## 2. Timeout Configuration

| Parameter | Purpose | Recommended Default |
| :--- | :--- | :--- |
| `timeout_mins` | Maximum wall-clock time with no log output before declaring the worker dead. | 10–15 minutes (scale with task complexity). |
| `progress_timeout_mins` | Maximum time with log activity but no commits, unstaged diffs, or test runs. | 15–20 minutes. |
| `max_restarts` | Total restart budget per agent across all escalation levels. | 3 restarts. |

**Tuning heuristic**: Small, focused tasks (fix a bug, write a test) warrant short timeouts (5–10 min). Large implementation tasks (build a service, migrate a module) warrant longer timeouts (15–30 min). Never set unlimited timeouts — they convert stalled agents into silent cost sinks.

---

## 3. Escalation Ladder

### Level 1 — Soft Restart

**Trigger**: First progress timeout, first validation failure, or first boundary violation.

**Procedure**:
1. Kill the worker process.
2. Preserve uncommitted work: `git add -A && git commit -m "WIP: <agent-name> — soft restart"`.
3. Compose corrective instructions:
   - Include the specific error output (test failure, type error, lint violation).
   - Include the validation command and its stderr/stdout.
   - Narrow the focus: "The auth middleware fails because X. Fix X specifically."
4. Respawn the worker in the same worktree with the corrective prompt.
5. Increment `restart_count`.

**Key principle**: Soft restarts preserve accumulated work. The worker continues from where it stopped, with targeted guidance about what went wrong.

### Level 2 — Hard Restart

**Trigger**: Second failure (after a soft restart failed to resolve the issue), or when the worker's approach is fundamentally wrong (e.g., implementing the wrong architecture, using a deprecated API).

**Procedure**:
1. Kill the worker process.
2. Create a recovery tag preserving the full state:
   ```bash
   git add -A
   git commit -m "recovery: <agent-name> pre-hard-restart"
   git tag recovery/<agent-name>/<timestamp>
   ```
3. Reset the worktree to the foundation baseline:
   ```bash
   git reset --hard <baseline-commit>
   ```
4. Compose a restructured prompt:
   - Explain why the previous approach failed.
   - Provide narrower scope or an alternative approach.
   - Reference the recovery tag if the worker needs to inspect prior work.
5. Respawn the worker with the restructured prompt.
6. Increment `restart_count`.

**Key principle**: Hard restarts wipe the slate but preserve evidence. The recovery tag ensures nothing is permanently lost.

### Level 3 — Park (`needs_attention`)

**Trigger**: Third failure, or `restart_count` exceeds `max_restarts`.

**Procedure**:
1. Kill the worker process.
2. Preserve the worktree as-is (do not reset or delete).
3. Mark the agent as `needs_attention` in coordination state.
4. **Stop all further automatic recovery.** No more restarts.
5. Report to the orchestrator/user:
   ```
   ⚠ Worker [auth-api] parked after 3 failed restarts.
   Worktree preserved at: .agents/worktrees/auth-api
   Last error: [summary of final failure]
   Recovery tags: recovery/auth-api/20260924T1200, recovery/auth-api/20260924T1215
   Action required: Human inspection and decision.
   ```

**Key principle**: Parking prevents runaway cost. Three failures with no progress indicates the decomposition, scope, or approach is wrong — not that the fourth attempt will succeed.

---

## 4. Progress Heartbeat Protocol (Optional)

Workers may write a lightweight heartbeat file to signal progress in non-editing phases:

```json
// coord/progress/<agent-name>.json
{
  "agent": "auth-api",
  "phase": "reading",
  "timestamp": "2026-09-24T12:15:00Z",
  "detail": "Reading src/auth/middleware.ts to understand existing JWT logic"
}
```

**Valid phases**: `reading`, `planning`, `coding`, `testing`, `building`, `debugging`, `installing`.

**How it's used**: The supervision loop checks the file's modification time (filesystem `mtime`). A recent heartbeat in a non-editing phase (`reading`, `planning`) can grant one bounded grace period before triggering a progress timeout. This prevents false positives when a worker legitimately spends time reading and planning before writing code.

**Caution**: Heartbeats are advisory, not authoritative. A worker writing heartbeats but never producing code diffs is still stalled — the grace period is bounded, not infinite.

---

## 5. Triage Runbook for Parked Agents

When a worker is parked in `needs_attention`, the orchestrator or user should:

1. **Inspect the worktree**: `git -C .agents/worktrees/<agent> status` and `git -C .agents/worktrees/<agent> log --oneline -10`.
2. **Read the last error**: Check the worker's log output for the final failure.
3. **Compare recovery tags**: `git diff recovery/<agent>/<first-tag>..recovery/<agent>/<last-tag>` to understand what changed across restart attempts.
4. **Diagnose the root cause**:
   - **Wrong decomposition**: The task boundaries are incorrect. Re-decompose and re-dispatch.
   - **Missing context**: The worker prompt lacked critical information. Augment the contract and re-dispatch.
   - **Environmental issue**: Missing dependency, wrong Node version, flaky test. Fix the environment and resume.
   - **Task is too hard for delegation**: Absorb the task back into the orchestrator session.
5. **Resume or abandon**:
   - **Resume**: Fix the root cause, then re-dispatch with `--resume` to reuse the existing worktree.
   - **Abandon**: Clean up the worktree (`git worktree remove`) and either re-decompose or handle the task directly.
