---
name: github-pro
description: Professional Git and GitHub workflow automation with adaptive decision-making. Enforces pre-flight inspection, intent disambiguation, automatic release milestone detection with authorization-aware publishing, safety guardrails, contextual secret hygiene, and post-action verification.
---

# GitHub Pro: Autonomous Workflow Protocol

> **Mandate**: No blind Git or GitHub actions. Disambiguate user intent, dynamically inspect repository state and conventions, execute the minimal safe professional workflow, automatically detect release-worthy milestones while requiring authorization to publish, verify state changes, and report proportionally.

---

## 1 · Adaptive Decision Lifecycle

Never map user phrases to hard-coded command sequences (e.g., `"push this"`, `"sync"`, `"ship it"`, `"release this"`, `"merge it"`, `"clean this"`, `"publish this"` require contextual analysis before execution). Execute every request through this adaptive pipeline:

1. **User Intent**: Decode the actual objective rather than matching keywords.
2. **Repository State**: Inspect working tree status (`git status --short`), diffs, and clean/dirty state.
3. **Branch & Policy**: Detect active branch, upstream tracking, configured remote(s), default trunk branch (`main`, `master`, `trunk`, `develop`), and branch protection rulesets.
4. **Conventions & Tooling**: Inspect history patterns (`git log -n 5 --oneline`), project test/lint scripts, and host tool capabilities/auth (`git --version`, `gh auth status`).
5. **Minimal Safe Action**: Execute only the operations necessary to achieve the goal safely. Engage PRs, reviews, checks, or issues only when required by branch policy or explicitly requested. Never introduce unsolicited ceremony.
6. **Automatic Milestone & Release Triage**: Assess whether accumulated changes constitute a milestone (see Section 3).
7. **Post-Execution Verification**: Confirm actual state changes (`git log`, remote refs, `gh pr view`, `gh release view`). Never rely solely on process exit codes.
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
  - Commits and diffs accumulated since previous tag/release (`git describe`, `git tag -l`, `gh release list`).
  - Repository release cadence, changelog conventions, and automated workflows in `.github/workflows/`.
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
  - *Protected branch push rejection*: Isolate changes onto a new topic branch, push upstream (`git push -u <remote> <branch>`), and create a PR.
  - *Interrupted rebase or merge*: Cleanly abort (`git rebase --abort` or `git merge --abort`) if unresolvable, preventing corrupted intermediate states.

---

## 5 · Contextual Security, Secrets & CI Hygiene

- **Pre-Staging Secret Inspection**:
  - Inspect `git diff` and `git diff --staged` before committing for private keys (`*.pem`, `*.key`), tokens, credentials, and environment files (`.env*`).
  - **No Blanket Exemptions**: Never treat test fixtures, sample configs, or mock files (`.env.example`) as inherently safe. Inspect actual contents to prevent accidental live credential leaks.
  - Unstage detected sensitive files immediately and ensure `.gitignore` covers local secrets and build artifacts.
- **GitHub Actions & CI Integrity**:
  - Enforce least-privilege `permissions:` blocks for `GITHUB_TOKEN` (e.g., `contents: read`).
  - In high-assurance workflows, pin third-party Actions to full commit SHAs with version comments to mitigate supply-chain risks.
  - Never log or echo sensitive variables in workflow steps or scripts. Respect repository secret scanning and Dependabot alerts.

---

## 6 · Toolchain, Version Awareness & Platform Coverage

- **Capability & Version Detection**:
  - Check `git --version` and `gh --version` / `gh auth status` when flags or subcommands vary by environment.
  - Use syntax supported by installed tool versions; gracefully degrade to compatible alternatives or standard Git remotes if `gh` is unavailable or unauthenticated.
  - Avoid unsolicited tool upgrades (never run `brew`, `apt`, `choco`, `winget`).
- **Intelligent Platform Coverage**:
  - Recognize when to leverage GitHub platform features: repositories, branches, commits, PRs (`gh pr view/create/checks`), code reviews, merge methods (squash, rebase, or merge commit matching repository policy), CI checks, Actions workflows, branch protection rulesets, issues (`gh issue`), releases (`gh release`), tags, and GitHub API queries (`gh api`).
  - For version-sensitive GitHub platform capabilities, refer to current official documentation or dynamic CLI help rather than static assumptions.
