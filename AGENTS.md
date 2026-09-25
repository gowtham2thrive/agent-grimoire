# Agent Grimoire: Operational Guidelines & Protocol Router

This file governs agent behavior within this workspace. It acts as the primary rulebook and router for loading skills from `.agents/skills/`.

Skills are activated by relevance, not by availability.

The presence of a skill does not imply that it should be used.

The agent must determine:
1. whether the skill is relevant,
2. whether another skill already covers the need,
3. whether activating it adds meaningful value,
4. whether its context cost is justified,
5. whether the task can safely proceed without it.

Use the minimum sufficient set of skills required for the task.

Never activate every skill by default.
Never invoke a skill solely because another skill mentioned its name.
Never use a technology-specific skill when a universal principle is sufficient.
---

## 1. Skill Discovery & Invocation Mandate

Whenever a task relates to a registered skill in `.agents/skills/`, you **MUST** read the corresponding `SKILL.md` before executing actions:

| Skill | Activate When |
| :--- | :--- |
| [`requirements-analysis`](.agents/skills/requirements-analysis/SKILL.md) | Receiving raw requirements, feature requests, bug reports, or spec clarification before coding. |
| [`code-quality`](.agents/skills/code-quality/SKILL.md) | Writing, modifying, or reviewing code for defensive correctness. |
| [`code-review`](.agents/skills/code-review/SKILL.md) | Reviewing PRs, diffs, commits, or conducting pre-merge audits. |
| [`testing`](.agents/skills/testing/SKILL.md) | Writing tests, verifying bug fixes, validating features, or designing QA strategy. |
| [`refactoring`](.agents/skills/refactoring/SKILL.md) | Behavior-preserving code restructuring, module decoupling, or debt cleanup. |
| [`agent-evaluation`](.agents/skills/agent-evaluation/SKILL.md) | Final gate before declaring any task complete or submitting a PR. |
| [`project-analysis`](.agents/skills/project-analysis/SKILL.md) | Exploring unfamiliar codebases, onboarding, or mapping architecture before changes. |
| [`security-engineering`](.agents/skills/security-engineering/SKILL.md) | Designing, auditing, or hardening security posture across any system. |
| [`github-pro`](.agents/skills/github-pro/SKILL.md) | Running Git/GitHub commands: commits, branches, PRs, tags, releases. |
| [`docs-pro`](.agents/skills/docs-pro/SKILL.md) | Authoring or updating READMEs, API docs, ADRs, changelogs, or guides. |
| [`multi-agent-orchestration`](.agents/skills/multi-agent-orchestration/SKILL.md) | Splitting work across parallel agents, fan-out dispatch, or worktree isolation. |
| [`design-philosophy`](.agents/skills/design-philosophy/SKILL.md) | Visual structure, typography, color, spacing, layout aesthetics. |
| [`ux-engineering`](.agents/skills/ux-engineering/SKILL.md) | User task flows, information architecture, cognitive load, interaction design. |
| [`accessibility`](.agents/skills/accessibility/SKILL.md) | Inclusive access: screen readers, ARIA, keyboard navigation, motor/visual impairment. |
| [`system-architecture`](.agents/skills/system-architecture/SKILL.md) | Designing systems, decomposing modules, defining contracts, or architecture reviews. |
| [`api-design`](.agents/skills/api-design/SKILL.md) | Designing or evolving APIs: REST, gRPC, GraphQL, events, SDKs, CLI, MCP tools. |
| [`data-management`](.agents/skills/data-management/SKILL.md) | Schema design, data modeling, migrations, pipeline idempotency, or data quality. |
| [`planning`](.agents/skills/planning/SKILL.md) | Moving from requirements to executable implementation plans with task DAGs. |
| [`dependency-management`](.agents/skills/dependency-management/SKILL.md) | Auditing dependencies, evaluating libraries, upgrading packages, or supply-chain security. |
| [`solution-discovery`](.agents/skills/solution-discovery/SKILL.md) | Evaluating build-vs-adopt decisions before adding dependencies or custom code. |
| [`infrastructure`](.agents/skills/infrastructure/SKILL.md) | Provisioning environments, IaC, state convergence, or drift reconciliation. |
| [`configuration-management`](.agents/skills/configuration-management/SKILL.md) | Config schemas, precedence, secrets separation, environment parity. |
| [`failure-recovery`](.agents/skills/failure-recovery/SKILL.md) | Agent-side failures: test failures, tool crashes, wrong assumptions, rollbacks. |
| [`ci-cd`](.agents/skills/ci-cd/SKILL.md) | Build pipelines, automated verification, artifact packaging, or quality gates. |
| [`release-management`](.agents/skills/release-management/SKILL.md) | What ships, when, versioning, changelogs, readiness scorecards. |
| [`deployment`](.agents/skills/deployment/SKILL.md) | How artifacts reach environments: rollout, health checks, progressive delivery. |
| [`observability`](.agents/skills/observability/SKILL.md) | Telemetry design, SLIs/SLOs, distributed tracing, alerting, or diagnostics. |
| [`performance-engineering`](.agents/skills/performance-engineering/SKILL.md) | Profiling, load testing, latency budgets, capacity optimization. |
| [`maintenance`](.agents/skills/maintenance/SKILL.md) | Dead code audits, tech debt valuation, system vitality, deprecation governance. |
| [`incident-response`](.agents/skills/incident-response/SKILL.md) | Live/production failures affecting users: outages, degradations, crisis management. |

- **Data Safety**: Always verify before running commands that could result in irreversible data loss.
- **Reporting**: Keep routine responses concise; provide detailed structured breakdowns for milestones or architectural changes.


---

## 2. Skill Composition & Safety Protocol

### 2.1 Triviality Bypass
If the task is a single-file edit under ~30 lines with obvious correctness (typo fix, import addition, comment edit, local variable rename), apply `code-quality` principles directly from memory without loading any SKILL.md. This avoids burning context on trivial changes. **Exception**: Changes to public APIs, exported symbols, or interface contracts are never trivial regardless of line count — always load the relevant skill.

### 2.2 Conflict Resolution & Priority Ordering
When two skills could both activate for the same task, apply these routing rules:

| Ambiguous Scenario | Primary Skill | Routing Rule |
| :--- | :--- | :--- |
| Failure in agent's dev loop (test failure, tool crash, wrong assumption) | `failure-recovery` | Contained to agent's own process → failure-recovery |
| Failure affecting live users or production services | `incident-response` | User-facing operational impact → incident-response |
| Visual structure & aesthetics (typography, color, spacing) | `design-philosophy` | Perceptual physics & layout → design-philosophy |
| User task flows & cognitive load (forms, navigation, JTBD) | `ux-engineering` | Interaction flows & mental models → ux-engineering |
| Inclusive access (screen readers, ARIA, motor impairment) | `accessibility` | Assistive technology compliance → accessibility |
| What ships & when (versioning, changelog, readiness) | `release-management` | Version identity & consumer protection → release-management |
| How artifacts reach environments (rollout, health checks) | `deployment` | Progressive delivery & rollback → deployment |
| Structural code restructuring | `refactoring` | Behavior-preserving code changes → refactoring |
| Dead code, tech debt auditing, system vitality assessment | `maintenance` | Assessment & routing (routes to specialized skills for execution) |
| Logical system design (modules, contracts, state authority) | `system-architecture` | Abstract structure & trade-offs → system-architecture |
| Physical resource provisioning (VMs, IaC, networking, drift) | `infrastructure` | Concrete environment convergence → infrastructure |
| Single-agent task decomposition into ordered steps | `planning` | Sequential task DAG for one agent → planning |
| Parallel multi-agent fan-out across independent modules | `multi-agent-orchestration` | Worker isolation & supervision → multi-agent-orchestration |

### 2.3 Global Safety Circuit Breaker
Across ALL active skills in a session, the agent maintains a **global failure budget**:
- **Maximum 5 total failed recovery attempts** across all skills before mandatory user escalation.
- A "failed attempt" is any code mutation that fails post-verification (test failure, type error, runtime crash).
- When the global budget expires: stop all mutations, preserve current state, and escalate to the user with a structured summary of all attempts and failures.
- Individual skill budgets (e.g., failure-recovery's $N_{\max} \le 3$) still apply and may trigger earlier.

### 2.4 Multi-Skill Activation Discipline
When multiple skills activate for a single task:
1. Load the **primary** skill's SKILL.md fully.
2. For **secondary** skills, use the selective loading protocol:
   - **Read**: YAML frontmatter, Invariants, Boundary notes, Stopping Contract.
   - **Skip**: Lifecycle flowchart, Progressive Disclosure table, Cognitive Sizing table, Invariant Exception Protocol, Archetype Adaptation section, generic Guardrails (these follow the same meta-pattern as the primary skill).
   - **Exception**: If the secondary skill's domain is unfamiliar, load it fully.
3. If you have already read a skill in this session, apply its invariants and stopping contracts from memory. Only re-read if uncertain.
4. When two skills define the same constant (e.g., hit target sizes, contrast thresholds), the **domain-specific skill's value takes precedence** (accessibility > design-philosophy for contrast; performance-engineering > system-architecture for latency budgets).
5. Stopping contracts from ALL active skills must be satisfied before declaring completion. If they conflict, satisfy the stricter constraint.

---

## 3. General Principles

1. **Pre-flight Inspection First**: Inspect environment, working directory, git state, and configuration before taking action.
2. **Minimal Safe Action**: Never execute speculative, destructive, or bulk changes without verification.
3. **Preserve Integrity**: Do not remove existing comments, docstrings, or tests unless specifically instructed.
4. **Post-Action Verification**: Always verify that operations succeeded via status inspection or test runs rather than assuming success.
