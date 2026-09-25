# Agent Grimoire: Operational Guidelines & Protocol Router

This file governs agent behavior within this workspace. It acts as the primary rulebook and router for loading skills from `.agents/skills/`.

Skills are activated by relevance, not by availability.

The presence of a skill does not imply that it should be used.

The agent must determine:
1. whether the skill is relevant to the explicit user goal,
2. whether another skill already covers the primary responsibility,
3. whether activating it adds meaningful decision-making value,
4. whether its context cost is justified,
5. whether the task can safely proceed without loading it.

Use the minimum sufficient set of skills required for the task.

Never activate every skill by default.
Never invoke a skill solely because another skill mentioned its name.
Never use a technology-specific procedure when a universal principle is sufficient.

---

## 1. Skill Discovery & Invocation Mandate

Whenever a task relates to a registered skill in `.agents/skills/`, verify both the **Activate When** and **Do Not Activate When** conditions before loading `SKILL.md`:

| Skill | Activate When | Do Not Activate When / Defer To |
| :--- | :--- | :--- |
| [`requirements-analysis`](.agents/skills/requirements-analysis/SKILL.md) | Clarifying underspecified prompts, ambiguous feature requests, or missing acceptance criteria before implementation. | Implementation is already clear and well-scoped; for code construction defer to `code-quality`. |
| [`planning`](.agents/skills/planning/SKILL.md) | Decomposing multi-step, multi-file features or complex migrations into an ordered task DAG with verification triads. | Single-step, localized edits; for multi-agent parallel fan-out defer to `multi-agent-orchestration`. |
| [`code-quality`](.agents/skills/code-quality/SKILL.md) | Writing or modifying code for defensive boundaries, type representations, resource cleanup, and error hygiene. | Pure behavior-preserving restructuring (defer to `refactoring`) or reviewing existing diffs (defer to `code-review`). |
| [`code-review`](.agents/skills/code-review/SKILL.md) | Conducting adversarial pre-merge audits, PR inspections, diff evaluations, or sanity-checking uncommitted changes. | Active code authoring (defer to `code-quality`) or final completion gate certification (defer to `agent-evaluation`). |
| [`testing`](.agents/skills/testing/SKILL.md) | Designing test strategies, authoring test suites, reproducing bugs via negative tests, or verifying regression shields. | General code construction without test authoring; for gate certification defer to `agent-evaluation`. |
| [`refactoring`](.agents/skills/refactoring/SKILL.md) | Restructuring code, decoupling modules, or simplifying logic while strictly preserving observable behavior. | Adding new features or modifying public contracts (defer to `code-quality` or `api-design`). |
| [`agent-evaluation`](.agents/skills/agent-evaluation/SKILL.md) | Final verification gate before declaring any task complete, opening a PR, or presenting results to the user. | Mid-flight development, exploratory research, or read-only codebase navigation. |
| [`project-analysis`](.agents/skills/project-analysis/SKILL.md) | Onboarding to unfamiliar repositories, mapping architectural topology, tracing execution flows, or assessing blast radius. | Familiar repositories with well-understood scope, or localized single-file edits. |
| [`security-engineering`](.agents/skills/security-engineering/SKILL.md) | Threat modeling, auditing trust boundaries, sanitizing tainted inputs, handling credentials, or hardening security posture. | Routine bug fixes or code construction without security, auth, or boundary implications. |
| [`github-pro`](.agents/skills/github-pro/SKILL.md) | Executing version control operations, branch management, safe commits, merge conflict resolution, and forge interactions. | Computing semantic versions or evaluating release readiness criteria (defer to `release-management`). |
| [`docs-pro`](.agents/skills/docs-pro/SKILL.md) | Authoring or updating READMEs, architectural docs, ADRs, runbooks, or API reference guides. | Single-line docstring edits, inline code comments, or automated code formatting. |
| [`multi-agent-orchestration`](.agents/skills/multi-agent-orchestration/SKILL.md) | Coordinating parallel sub-agents across isolated worktrees/branches with durable contracts and holistic integration. | Single-agent sequential tasks (defer to `planning`) or tightly coupled changes sharing mutable state. |
| [`design-philosophy`](.agents/skills/design-philosophy/SKILL.md) | Establishing visual hierarchy, typography, spatial rhythm, color palettes, and component aesthetics. | Interaction task flows (defer to `ux-engineering`) or screen reader/assistive tech (defer to `accessibility`). |
| [`ux-engineering`](.agents/skills/ux-engineering/SKILL.md) | Designing user task journeys, form workflows, information architecture, cognitive ergonomics, and error recovery. | Pure visual styling (defer to `design-philosophy`) or assistive compliance (defer to `accessibility`). |
| [`accessibility`](.agents/skills/accessibility/SKILL.md) | Ensuring inclusive access: screen readers, keyboard navigation, focus management, ARIA contracts, and WCAG compliance. | Headless services, backend APIs, CLI daemons without user interaction, or purely visual palette tuning. |
| [`system-architecture`](.agents/skills/system-architecture/SKILL.md) | Designing module boundaries, state authority, distributed topology, capacity envelopes, or structural trade-offs. | Concrete physical infrastructure provisioning (defer to `infrastructure`) or local code construction. |
| [`api-design`](.agents/skills/api-design/SKILL.md) | Specifying or evolving interface contracts (HTTP, RPC, GraphQL, IPC, SDKs, CLI, MCP tools), idempotency, and schemas. | Internal module implementation details that do not cross component boundaries (defer to `code-quality`). |
| [`data-management`](.agents/skills/data-management/SKILL.md) | Designing storage schemas, state persistence, expand/contract migrations, data pipelines, or lifecycle retention. | In-memory data structures, UI state modeling, or purely ephemeral caching. |
| [`dependency-management`](.agents/skills/dependency-management/SKILL.md) | Auditing package manifests, upgrading dependencies, hardening supply chains, or resolving transitive conflicts. | Evaluating whether to build vs adopt new capabilities (defer to `solution-discovery`). |
| [`solution-discovery`](.agents/skills/solution-discovery/SKILL.md) | Evaluating build-vs-adopt decisions, selecting libraries/tools, and creating anti-corruption adapter boundaries. | Routine dependency updates for already-adopted libraries (defer to `dependency-management`). |
| [`infrastructure`](.agents/skills/infrastructure/SKILL.md) | Provisioning environments, cloud resources, IaC state convergence, and infrastructure lifecycle operations. | Logical module topology (defer to `system-architecture`) or application config values (defer to `configuration-management`). |
| [`configuration-management`](.agents/skills/configuration-management/SKILL.md) | Managing configuration schemas, environment variable precedence, secret isolation, and environment parity. | Provisioning physical secret vaults (defer to `infrastructure`) or runtime traffic shifting (defer to `deployment`). |
| [`failure-recovery`](.agents/skills/failure-recovery/SKILL.md) | Recovering from agent development-loop failures: test breaks, compilation errors, tool crashes, or wrong assumptions. | Production system outages affecting live users or deployed services (defer to `incident-response`). |
| [`ci-cd`](.agents/skills/ci-cd/SKILL.md) | Designing, authoring, or troubleshooting automated build, lint, and test verification pipelines. | Live runtime progressive delivery or canary traffic routing in environments (defer to `deployment`). |
| [`release-management`](.agents/skills/release-management/SKILL.md) | Governing release readiness scorecards, version semantics (SemVer/CalVer), changelogs, and launch approvals. | Progressive deployment mechanics (defer to `deployment`) or executing Git branch/tag commands (defer to `github-pro`). |
| [`deployment`](.agents/skills/deployment/SKILL.md) | Orchestrating runtime transitions, progressive rollouts (canary, blue-green), health verification, and rollback. | Deciding what ships or version numbers (defer to `release-management`) or building CI pipelines (defer to `ci-cd`). |
| [`observability`](.agents/skills/observability/SKILL.md) | Designing telemetry models, metrics/SLIs, distributed trace propagation, logging hygiene, and alert specs. | Live crisis management during outages (defer to `incident-response`) or standard unit test assertions. |
| [`performance-engineering`](.agents/skills/performance-engineering/SKILL.md) | Profiling bottlenecks, measuring latency percentiles, load testing, query plan optimization, and regression shielding. | Routine unprofiled micro-optimizations, cosmetic restructuring, or styling adjustments. |
| [`maintenance`](.agents/skills/maintenance/SKILL.md) | Auditing technical debt, identifying dead code reachability, evaluating technology aging, and governance. | Executing restructuring (defer to `refactoring`) or updating packages (defer to `dependency-management`). |
| [`incident-response`](.agents/skills/incident-response/SKILL.md) | Managing operational incidents, live outages, service degradations, emergency containment, and blameless postmortems. | Failures confined to the agent's local development or test environment (defer to `failure-recovery`). |

- **Data Safety**: Always verify before running commands that could result in irreversible data loss.
- **Reporting**: Keep routine responses concise; provide detailed structured breakdowns for milestones or architectural changes.

---

## 2. Skill Composition & Safety Protocol

### 2.1 Low-Risk & Localized Change Bypass
If a task is a localized, low-risk change with obvious correctness (such as a typo fix, single import correction, comment edit, or localized variable rename) that does not alter public API boundaries, exported symbols, schema definitions, or security posture, apply core engineering invariants directly from memory without loading any `SKILL.md`. This preserves context bandwidth for complex reasoning.

**Exception**: Changes to public interfaces, authentication/authorization paths, data persistence schemas, or core configuration are never trivial regardless of line count—always consult the relevant skill.

### 2.2 Conflict Resolution & Priority Ordering
When two or more skills appear relevant, apply these unambiguous routing rules:

| Ambiguous Scenario | Primary Skill | Routing Rule |
| :--- | :--- | :--- |
| Failure in agent's local dev loop (test failure, tool crash, wrong assumption) | `failure-recovery` | Contained to agent's own process $\to$ `failure-recovery` |
| Failure affecting live users, production systems, or operational deployments | `incident-response` | User-facing operational impact $\to$ `incident-response` |
| Visual aesthetics, typography, color harmony, and spatial rhythm | `design-philosophy` | Perceptual physics & styling $\to$ `design-philosophy` |
| User task flows, information architecture, mental models, cognitive load | `ux-engineering` | Interaction flows & ergonomics $\to$ `ux-engineering` |
| Inclusive access, screen readers, keyboard navigation, focus management | `accessibility` | Assistive technology compliance $\to$ `accessibility` |
| What ships, version identification, changelog governance, readiness scoring | `release-management` | Release boundary & consumer contract $\to$ `release-management` |
| How artifacts reach target environments (canary, rolling, health soak) | `deployment` | Progressive delivery & runtime rollback $\to$ `deployment` |
| Deciding whether to adopt an external solution vs build custom capability | `solution-discovery` | Build-vs-adopt evaluation & adapter contracts $\to$ `solution-discovery` |
| Managing, upgrading, auditing, or pruning existing project dependencies | `dependency-management` | Supply-chain & dependency tree hygiene $\to$ `dependency-management` |
| Structural behavior-preserving code restructuring | `refactoring` | Restructuring under green tests $\to$ `refactoring` |
| Auditing dead code, measuring technical debt, and system vitality | `maintenance` | Diagnostic audit & routing $\to$ `maintenance` |
| Logical system architecture (module DAGs, state authority, boundary trade-offs) | `system-architecture` | Conceptual structure & contracts $\to$ `system-architecture` |
| Physical resource provisioning (VMs, cloud substrates, IaC convergence) | `infrastructure` | Concrete environment provisioning $\to$ `infrastructure` |
| Single-agent sequential task decomposition into an ordered plan | `planning` | Sequential task DAG for one agent $\to$ `planning` |
| Parallel multi-agent fan-out across physically decoupled modules | `multi-agent-orchestration` | Worker isolation & supervision $\to$ `multi-agent-orchestration` |
| Automated verification workflow definition (pipeline triggers, matrix builds) | `ci-cd` | Automated pipeline automation $\to$ `ci-cd` |
| Version control operations (commits, branch hygiene, merge conflict recovery) | `github-pro` | VCS mechanics & forge interaction $\to$ `github-pro` |

### 2.3 Global Safety Circuit Breaker
Across ALL active skills in a session, the agent maintains a **global failure budget**:
- **Maximum 5 total failed recovery attempts** across all skills before mandatory user escalation.
- A "failed attempt" is any code or configuration mutation that fails post-verification (test failure, type error, runtime crash, lint violation).
- When the global budget expires: stop all mutations immediately, preserve the current state, and escalate to the user with a structured summary of all attempts, hypotheses, and failure signatures.
- Individual skill budgets (e.g., `failure-recovery`'s $N_{\max} \le 3$) still apply and may trigger earlier.

### 2.4 Multi-Skill Activation Discipline
When multiple skills activate for a single task:
1. Load the **primary** skill's `SKILL.md` fully.
2. For **secondary** skills, use the selective loading protocol:
   - **Read**: YAML frontmatter, Invariants, Boundary notes, and Stopping Contract.
   - **Skip**: Detailed reference manuals, lifecycle diagrams, and boilerplate sections already familiar from memory.
   - **Exception**: If the secondary skill's domain is unfamiliar or safety-critical (e.g., `security-engineering`), load it fully.
3. If you have already read a skill in this session, apply its invariants and stopping contracts from memory. Only re-read if uncertain.
4. When two skills define domain constraints, the **domain-specific skill's value takes precedence** (e.g., `accessibility` takes precedence over `design-philosophy` for contrast thresholds; `performance-engineering` takes precedence over `system-architecture` for latency budgets).
5. Stopping contracts from ALL active skills must be satisfied before declaring completion. If they conflict, satisfy the stricter constraint.

### 2.5 Repository Context Adaptation Axiom
Every reusable skill must adapt to the target repository's established conventions, tools, and idioms:
- **Respect Established Patterns**: Use the project's existing package managers, build systems, test runners, linters, directory structures, and code style.
- **No Unsolicited Re-platforming**: Never introduce new tools, libraries, architectural layers, or config systems unless explicitly requested or required to fix a verified defect.
- **Graceful Tool Degradation**: When a preferred tool (e.g., a specific CLI or forge helper) is unavailable, gracefully fall back to native standard tools and notify the user proportionally.

---

## 3. General Principles

1. **Pre-flight Inspection First**: Inspect environment, working directory, version control status, and configuration before taking mutating action.
2. **Minimal Safe Action**: Never execute speculative, destructive, or bulk changes without verification.
3. **Preserve Integrity**: Do not remove existing comments, docstrings, architectural structures, or tests unless specifically instructed.
4. **Post-Action Verification**: Always verify that operations succeeded via concrete status inspection, compile checks, or test runs rather than assuming success from exit codes alone.
