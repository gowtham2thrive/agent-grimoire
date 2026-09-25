---
name: github-pro
description: >-
  Universal Version Control and Platform Forge Operations protocol.
  Enforces pre-flight inspection, intent disambiguation, meaningful commit construction,
  automatic release milestone detection with authorization-aware publishing, safety guardrails,
  contextual secret hygiene, and post-action verification across Git and modern forge environments
  (GitHub, GitLab, Bitbucket, or local repositories).
  Do not activate for routine code editing or refactoring (use code-quality or refactoring),
  release governance and readiness scorecards (use release-management), or CI pipeline definition (use ci-cd).
---

# Version Control & Forge Operations: Autonomous VCS Protocol

> **Mandate**: No blind Git or forge actions. Disambiguate user intent, dynamically inspect repository state and conventions, execute the minimal safe professional workflow, automatically detect release-worthy milestones while requiring authorization to publish, verify state changes, and report proportionally.

---

## 1 · Adaptive Decision Lifecycle

Never map user phrases to hard-coded command sequences (e.g., `"push this"`, `"sync"`, `"ship it"`, `"release this"`, `"merge it"`, `"clean this"`, `"publish this"` require contextual analysis before execution). Execute every request through this adaptive pipeline:

1. **User Intent**: Decode the actual objective rather than matching keywords.
2. **Repository State**: Inspect working tree status (`git status --short`), diffs, and clean/dirty state.
3. **Branch & Policy**: Detect active branch, upstream tracking, configured remote(s), default trunk branch (`main`, `master`, `trunk`, `develop`), and branch protection rulesets.
4. **Conventions & Tooling**: Inspect history patterns (`git log -n 5 --oneline`), project test/lint scripts, and host tool capabilities/auth (`git --version`, forge CLI status).
5. **Minimal Safe Action**: Execute only the operations necessary to achieve the goal safely. Engage PRs/MRs, reviews, checks, or issues only when required by branch policy or explicitly requested. Never introduce unsolicited ceremony.
6. **Automatic Milestone & Release Triage**: Assess whether accumulated changes constitute a milestone (see Section 3).
7. **Post-Execution Verification**: Confirm actual state changes (`git log`, remote refs, forge PR/MR/release view). Never rely solely on process exit codes.
8. **Proportional Reporting**: 1–2 high-signal lines for routine tasks; structured breakdown for milestones and release candidates.

---

## 2 · Meaningful Commits

Whenever creating commits, preserve clear, reviewable repository history:

- **Logical Grouping**: Stage and commit only logically related changes together. Never combine unrelated modifications into a single commit or create artificial commits merely to satisfy a workflow.
- **Understand Actual Changes**: Inspect diffs (`git diff`) before composing messages to understand what actually changed. Never modify user code or edits solely to make a commit look cleaner.
- **Convention & Specificity**: Follow the repository's prevailing commit-message style (`git log -n 5 --oneline`). Write concise, specific messages describing the real change so future developers can understand the history.
- **No Vague Summaries**: Strictly avoid ambiguous or placeholder messages such as `update`, `changes`, `fix`, `stuff`, or `misc`.

---

## 3 · Automatic Release Detection & Versioning

Continuously evaluate pushes, merges, and change batches for milestone significance without requiring explicit "release" prompts:

- **Assessment Criteria**:
  - Semantic impact (breaking change, feature addition, bug fix, or routine chore).
  - Coherence and completeness of changes across affected components.
  - Test suite and CI check health.
  - Commits and diffs accumulated since previous tag/release (`git describe`, `git tag -l`, forge release queries).
  - Repository release cadence, changelog conventions, and automated workflows.
  - Overall project stability and releasability (a large diff alone is **not** a release candidate).
- **Triage Classification**:
  - *Routine*: Internal refactors, minor fixes, isolated chores, or ongoing feature work. Execute minimal safe action without release overhead.
  - *Significant*: Meaningful functional addition or milestone advancement. Complete requested workflow and note milestone progress in report.
  - *Release Candidate*: Coherent, stable, passing all tests/checks, and representing a logical release boundary.
- **Authorization-Aware Publishing**:
  - If the repository has an explicit automated release workflow triggered by the action (e.g., CI on trunk push), proceed according to that workflow.
  - Otherwise, **never publish tags, releases, or version bumps autonomously**. Report:
    > **Release candidate detected.**
    Concisely detail detected version impact, rationale, and proposed release actions, awaiting explicit user authorization.
- **Convention-First Versioning**:
  - Detect and match repository versioning schemes (SemVer, CalVer, date-based, or build numbers) from package manifests (`package.json`, `Cargo.toml`, `pyproject.toml`) or tag history.
  - Never invent version schemes or assume SemVer if another pattern is established.
  - Match existing tag conventions (lightweight vs annotated); do not force annotated tags if the repository uses lightweight or custom tags.
  - Never generate unnecessary tags, releases, changelogs, version bumps, PRs, or issues.

---

## 4 · Safety Guardrails & Destructive Action Policy

- **Blocked Without Explicit Confirmation**:
  - `git reset --hard` (prefer `git stash` or soft resets).
  - `git clean -f` / `-fd` (always run dry-run `git clean -nd` first).
  - `git branch -D` (always attempt safe `git branch -d` first).
  - Blind or unqualified `git push --force`.
- **Force-Push Constraints**:
  - Strictly forbidden on trunk, protected, or shared branches.
  - Permitted only via `--force-with-lease` on private topic branches following an intentional rebase.
- **Conflict & Error Recovery**:
  - *Non-fast-forward push rejection*: Fetch and rebase against detected upstream remote (`git pull --rebase <remote> <branch>`); resolve conflicts carefully; never force-push over remote changes.
  - *Protected branch push rejection*: Isolate changes onto a new topic branch, push upstream (`git push -u <remote> <branch>`), and create a PR/MR.
  - *Interrupted rebase or merge*: Cleanly abort (`git rebase --abort` or `git merge --abort`) if unresolvable, preventing corrupted intermediate states.

---

## 5 · Contextual Security, Secrets & CI Hygiene

- **Pre-Staging Secret Inspection**:
  - Inspect `git diff` and `git diff --staged` before committing for private keys (`*.pem`, `*.key`), tokens, credentials, and environment files (`.env*`).
  - **No Blanket Exemptions**: Never treat test fixtures, sample configs, or mock files (`.env.example`) as inherently safe. Inspect actual contents to prevent accidental live credential leaks.
  - Unstage detected sensitive files immediately and ensure `.gitignore` covers local secrets and build artifacts.
- **CI & Pipeline Integrity**:
  - Enforce least-privilege permissions in CI workflow configurations (e.g. `contents: read`).
  - In high-assurance workflows, pin external actions/dependencies to immutable hashes or specific tags to mitigate supply-chain risks.
  - Never log or echo sensitive variables in workflow steps or scripts. Respect repository secret scanning alerts.

---

## 6 · Toolchain, Version Awareness & Forge Adaptation

VCS operations adapt gracefully to the tools and hosting forge available in the environment:

| Forge / Environment | Detection | PR / MR Creation | Release Queries | Fallback When CLI Absent |
| :--- | :--- | :--- | :--- | :--- |
| **GitHub** | `git remote -v` contains `github.com` | `gh pr create` / `gh pr view` | `gh release list` | Standard git push + emit branch compare URL |
| **GitLab** | `git remote -v` contains `gitlab.com` or self-hosted | `glab mr create` / `glab mr view` | `glab release list` | Standard git push + emit GitLab MR URL |
| **Bitbucket / Other** | Remote URL inspection | Forge CLI or web interface | Forge API / git tags | Standard git push + branch URL |
| **Local Git Only** | No remote or air-gapped | N/A (local branch merge) | `git tag -l` | Standard local Git workflows |

- **Graceful CLI Degradation**:
  - Verify forge CLI availability (`gh --version`, `glab --version`) before calling forge-specific subcommands.
  - If a forge CLI is missing or unauthenticated, execute standard Git operations (`git checkout -b`, `git add`, `git commit`, `git push -u origin <branch>`) and provide the user with the direct web URL to open the pull/merge request.
  - Avoid unsolicited package installations (never run `brew`, `apt`, `choco`, `winget` to install forge CLIs without permission).

---

## 7 · The Clean VCS Stopping Contract

A version control operation is strictly **COMPLETE** only when:
1. **Working Tree Clean**: The working directory is in the intended state with zero leftover untracked debris or partial rebase/merge states.
2. **Commit Hygiene Verified**: All commits are logically atomic, accurately named according to repository convention, and contain zero unintended file touches.
3. **Secret Hygiene Proven**: Diffs have been inspected and confirmed free of plaintext secrets, tokens, or private credentials.
4. **Remote Synchronization Confirmed**: Changes are cleanly pushed to the designated remote branch with tracking set, or explicitly kept local per user instruction.
5. **Proportional Status Reported**: The user receives a concise summary of the branch, commit SHA, and remote status (plus PR/MR link if created).
