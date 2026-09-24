# Deliverable Artifact Schemas & Downstream Contracts

> **Core Axiom**: An analysis deliverable must be compact, dense, and structured for dual human/agent readability. It must fit comfortably within downstream context budgets ($\le 2,000$ tokens) while conveying complete architectural reality.

---

## 1. Schema: `PROJECT_BRIEF.md` (The 2-Page Repository Digest)

Use this schema when running in `system_onboarding` or broad architectural mapping mode. Store in `.context/PROJECT_BRIEF.md` or conversation artifacts.

```markdown
# Project Brief: {Project Name}
> Generated: {YYYY-MM-DD} | Mode: system_onboarding | Target Revision: {git-commit-hash}

## 1. System Identity & Workspace Structure
- **Archetype**: {Web API | Frontend | CLI Daemon | Library/SDK | Data/ML | IaC | Agentic}
- **Workspace Layout**: {Single Package | Polyglot Monorepo | Multi-Crate Workspace}
- **Core Stack**:
  - Language & Runtime: {e.g. TypeScript / Node 20, Rust 1.78, Python 3.12}
  - Primary Framework: {e.g. Fastify, Actix-Web, Django, React}
  - State & Storage: {e.g. PostgreSQL + Prisma, SQLite, Redis, S3}
  - Build & Package Tool: {e.g. pnpm, cargo, poetry, make}

## 2. Execution Harness & Baseline Health
| Operation | Command | Grounding Source | Baseline Status |
| :--- | :--- | :--- | :--- |
| **Build** | `{build-command}` | `[CONFIG: {file}#L{line}]` | `{Verified Passing \| Pre-existing Error}` |
| **Test (Full)** | `{test-all-command}` | `[CONFIG: {file}#L{line}]` | `{Passing \| N failing tests noted}` |
| **Test (Single)**| `{test-single-command}` | `[CONFIG: {file}#L{line}]` | `{Verified Passing}` |
| **Dev / Run** | `{dev-command}` | `[CONFIG: {file}#L{line}]` | `{Configured}` |

## 3. Directory Layout Semantics
- `{path-1}/`: {Core domain logic and business rules}
- `{path-2}/`: {Ingress routing, controllers, CLI parsers}
- `{path-3}/`: {Persistence adapters, ORM schemas, external API clients}

## 4. Key Invariants & Non-Negotiables
1. `[INV-01]` {Invariant description with line-anchored grounding, e.g., All mutations must pass schema validation at the ingress boundary}. `[CODE: {file}#L{line}]`
2. `[INV-02]` {Invariant description, e.g., Database transactions must be explicit and roll back on error}. `[CODE: {file}#L{line}]`

## 5. Canonical Vertical Flows
- **Golden Path ({Journey Name})**: {Stimulus} $\rightarrow$ `{Ingress Handler}` $\rightarrow$ `{Domain Service}` $\rightarrow$ `{Persistence Store}` $\rightarrow$ `{Egress Response}`. `[VERIFIED: {test}#L{line}]`
- **Failure Path**: {Fault Stimulus} $\rightarrow$ `{Error Middleware / Guard}` $\rightarrow$ `{Rollback Action}` $\rightarrow$ `{Error Egress}`. `[CODE: {file}#L{line}]`

## 6. Change Location Index (Where to Look)
- **Add new ingress route/command**: `{path}` $\rightarrow$ register in `{router-file}`
- **Modify database schema/migration**: `{migration-dir}`
- **Add external integration**: `{adapter-dir}`
- **Add/update unit tests**: `{test-dir}`
```

---

## 2. Schema: `FEATURE_BLAST_RADIUS.md` (Pre-Flight Feature Contract)

Use this schema in `feature_preflight` mode before modifying code for a new feature, bug fix, or refactor.

```markdown
# Feature Pre-Flight: {Feature / Bug Fix Name}
> Target Objective: {One sentence description of the proposed change}

## 1. Mutation Surface
| Target File | Action | Purpose | Existing Tests | Churn Risk |
| :--- | :--- | :--- | :--- | :--- |
| `{path-1}` | `{Modify \| Create}` | {Reason for mutation} | `{High (N tests) \| Desert (0 tests)}` | `{Low \| Moderate \| High}` |
| `{path-2}` | `{Modify \| Create}` | {Reason for mutation} | `{Coverage status}` | `{Risk level}` |

## 2. Dependency Ingress & Egress Impact
- **Upstream Callers Affected**: `{list modules / functions calling mutated files}`
- **Downstream Services Affected**: `{list DB tables, caches, or external APIs touched}`
- **State Invariants at Risk**: `{list any [INV-XX] invariants that this change could violate}`

## 3. Pre-Flight Test Plan
- Pre-mutation verification: `{command to run before touching code}`
- Post-mutation verification: `{command to run to prove change succeeded}`

## 4. Blast-Radius Risk Score: [LOW | MODERATE | HIGH]
- **Rationale**: {Why this score was assigned based on churn, test coverage, and coupling}
```

---

## 3. Schema: `RISK_MATRIX.md` (Forensic Audit Deliverable)

Use this schema in `forensic_audit` mode when evaluating technical debt or legacy health.

```markdown
# Forensic Risk Matrix: {Project Name}
> Target Revision: {git-commit-hash} | Audit Date: {YYYY-MM-DD}

## 1. Critical Hotspot Register
| Rank | Component / File | Git Churn (3mo) | Test Coverage | Coupling | Risk Level | Prescribed Action |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `{file-path}` | `{N} commits` | `{None \| Low \| High}` | `{In: X, Out: Y}` | **CRITICAL** | {Isolate with tests before refactoring} |

## 2. Testing Deserts (Dark Corners)
- `{directory-or-module}`: High complexity, core business logic, zero tests.

## 3. Architectural Drift
- `[DRIFT]` `{Doc file}` states `{X}`, but `{Code file}` implements `{Y}`.
```
