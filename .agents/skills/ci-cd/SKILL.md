---
name: ci-cd
description: >-
  Universal continuous integration, continuous delivery, and automated verification pipeline protocol.
  Use when designing, creating, optimizing, or debugging automated build, test, and verification workflows
  across any platform (GitHub Actions, GitLab CI, Tekton, Argo, Jenkins, CircleCI, local runners, Makefiles).
  Do NOT activate for live runtime progressive delivery or canary traffic routing (use deployment),
  or release versioning governance (use release-management). Enforces the 8 Universal CI/CD Invariants,
  6-phase closed-loop lifecycle, 6 cognitive sizing modes, build-once artifact immutability, safe caching,
  honest quality gates, controlled concurrency, and least privilege security.
---

# CI/CD: Universal Automated Pipeline & Continuous Verification Protocol

> **Mandate**: *Continuous Integration and Continuous Delivery (CI/CD) is the automated, repeatable, and tamper-evident conversion of source state mutations ($\Delta S$) into verified computational artifacts ($\mathcal{A}$), orchestrating their progressive promotion across quality and operational boundaries.*  
> A pipeline is not a vendor YAML dialect, nor is it coupled to any container runtime, orchestrator, or cloud provider. It is a declarative Directed Acyclic Graph (DAG) of transformations, gates, and promotion criteria. True CI/CD engineering preserves agent problem-solving creativity, rejects platform dogma, enforces immutable quality gates, and guarantees that every state transition is reproducible, observable, and recoverable across any software archetype.

---

## 1 · The 6-Phase Universal CI/CD Lifecycle

Every automated pipeline—from a local pre-commit check to an enterprise monorepo matrix or an autonomous AI agent evaluation fleet—traverses this closed-loop lifecycle:

```mermaid
flowchart LR
    P1["1. Ingest & Trigger Triage<br/>(Event filter, concurrency cancel)"] --> P2["2. Static Fast Feedback<br/>(Format, lint, types, SAST)"]
    P2 --> P3["3. Hermetic Build & Digest<br/>(Compile, package, hash digest)"]
    P3 --> P4["4. Dynamic Verification Matrix<br/>(Unit, integration, e2e, shard)"]
    P4 --> P5["5. Seal & Attestation<br/>(SBOM, provenance, security audit)"]
    P5 --> P6["6. Promotion & Delivery<br/>(Env rings, approval, handoff)"]
```

1. **Phase 1 — Ingest & Trigger Triage**: Evaluate incoming triggers (push, pull request, tag, schedule, manual dispatch). Apply change-detection filters to prune unaffected dependency sub-graphs. Enforce **Controlled Concurrency** by atomically cancelling obsolete in-progress runs on superseded branch commits.
2. **Phase 2 — Static Fast Feedback (Pre-Flight Gatekeeping)**: Execute lightweight, deterministic static gates (syntax, formatting, linting, typechecking, and secret scanning) targeting fast execution before initiating heavy compute resources (see [`references/pipeline-topology-and-stage-ordering.md`](references/pipeline-topology-and-stage-ordering.md)).
3. **Phase 3 — Hermetic Build & Artifact Synthesis**: Compile and package deployable artifacts in an isolated, reproducible environment. Calculate and pin the immutable cryptographic digest ($\text{Digest}(\mathcal{A})$). Ensure build outputs are completely environment-agnostic (see [`references/hermetic-builds-and-artifact-provenance.md`](references/hermetic-builds-and-artifact-provenance.md)).
4. **Phase 4 — Dynamic Verification Matrix**: Execute test suites across configured runtime matrices or shards. Enforce honest gating with strict flakiness quarantine, zero ignored exit codes, and statistical confidence thresholds for stochastic/AI workflows (see [`references/honest-quality-gates-and-signal-integrity.md`](references/honest-quality-gates-and-signal-integrity.md)).
5. **Phase 5 — Package Sealing & Attestation**: Synthesize the Software Bill of Materials (SBOM), inject build provenance metadata, scan packaged artifacts for critical vulnerabilities, and publish sealed artifacts to an immutable registry (see [`references/hermetic-builds-and-artifact-provenance.md`](references/hermetic-builds-and-artifact-provenance.md) and [`references/pipeline-security-and-least-privilege.md`](references/pipeline-security-and-least-privilege.md)).
6. **Phase 6 — Promotion Gating & Delivery Handoff**: Validate environment readiness, evaluate promotion criteria across progressive rings (Ephemeral $\to$ Staging $\to$ Prod), verify human approval gates where required, and hand off the certified artifact tuple $\langle \mathcal{A}, \mathcal{C}, \mathcal{S} \rangle$ to [`deployment`](../deployment/SKILL.md) (see [`references/environment-promotion-and-approval-gates.md`](references/environment-promotion-and-approval-gates.md)).

---

## 2 · Progressive Disclosure (Reference Routing)

To prevent cognitive overload and preserve LLM context bandwidth, consult reference manuals strictly on demand based on task context:

| Context Trigger | Mandatory Reference Manual | Purpose |
| :--- | :--- | :--- |
| **Pipeline DAG design, fail-fast ladders, stage ordering, change detection** | [`references/pipeline-topology-and-stage-ordering.md`](references/pipeline-topology-and-stage-ordering.md) | DAG formulation, topological sorting, fail-fast mechanics, affected-graph filtering for monorepos. |
| **Reproducible builds, digest pinning, SBOM, provenance, container sealing** | [`references/hermetic-builds-and-artifact-provenance.md`](references/hermetic-builds-and-artifact-provenance.md) | Deterministic build environments, SHA-256 digest pinning, SBOMs, build attestations. |
| **Cache key hashing, cross-branch poisoning, concurrency cancellation, cost** | [`references/caching-concurrency-and-resource-efficiency.md`](references/caching-concurrency-and-resource-efficiency.md) | Hierarchical cache lattice, lockfile hashing, auto-cancellation, worker quotas. |
| **OIDC tokens, secret scoping, fork PR security ("Pwn Requests"), least privilege** | [`references/pipeline-security-and-least-privilege.md`](references/pipeline-security-and-least-privilege.md) | Two-tier PR security model, short-lived OIDC federation, token permission sandboxing. |
| **Flaky test quarantine, test sharding, stochastic AI evaluation, coverage gates** | [`references/honest-quality-gates-and-signal-integrity.md`](references/honest-quality-gates-and-signal-integrity.md) | Honest merge signals, flakiness quarantine, statistical gating, merge queues. |
| **Multi-ring promotion, approval checkpoints, GitOps handoffs, rollback readiness** | [`references/environment-promotion-and-approval-gates.md`](references/environment-promotion-and-approval-gates.md) | Promotion rings, human governance gates, handoff to `deployment` & `release-management`. |

---

## 3 · Adaptive Cognitive Sizing (The 6 Execution Modes)

Size your CI/CD engineering effort strictly to the task's scope, operational risk, and repository archetype. Never apply heavy enterprise bureaucracy to a single-line script or local prototype, and never execute speculative, unverified mutations across high-blast-radius production systems:

| Mode | Trigger & Scope | Engineering Discipline | Required Output & Protocol |
| :--- | :--- | :--- | :--- |
| **`micro-check`** | Local commit hooks, typo fixes, doc updates, single-file scripts. | Run fast static check $\to$ unit smoke test. **Zero boilerplate**. | **3-Line CI Intent Block** directly preceding execution. |
| **`standard-pr`** | Feature branch pull request, bug fix, component update across interacting files. | Full CI verification: static checks $\to$ build $\to$ unit/integration tests $\to$ PR status gate. | **PR Verification Status Report**: Stage timings, gate outcomes, failure diagnostics. |
| **`matrix-library`** | Open-source libraries, client SDKs, cross-platform CLI utilities. | Multi-OS and multi-runtime matrix verification with concurrency limits. | **Matrix Compatibility Grid**: Status per platform/version combination. |
| **`monorepo-cascading`** | Polyglot or multi-package monorepos with internal dependency DAGs. | Impact-aware change detection; topological build/test execution; shared remote caching. | **Monorepo Execution Graph**: Affected packages, skipped packages, cache hit rates. |
| **`delivery-promotion`** | Production services, multi-environment applications (Dev $\to$ Staging $\to$ Prod). | Build once $\to$ deploy to staging $\to$ automated integration/smoke verify $\to$ approval gate $\to$ production rollout handoff. | **Environment Promotion Audit**: Artifact hash attestation, stage approvals, deployment handoff tuple. |
| **`agentic-eval-pipeline`** | AI agent prompts, MCP servers, system instructions, model fine-tunes. | Golden dataset benchmark execution, model evaluation scoring ($\ge \tau$), token/latency budget gate, semantic drift check. | **Agentic Verification Certificate**: Benchmark score delta, tool permission audit, cost summary. |

### The 3-Line CI Intent Protocol (For `micro-check` Mode)
When operating in `micro-check` mode, emit this concise block directly preceding pipeline triggers or local validation commands:
```markdown
> **Pipeline Target**: [Ref / Branch / Target Path]
> **Active Gates**: [Static Lint: PASS | Tests (N/N): PASS]
> **Artifact State**: [Ephemeral / Verified Digest: SHA256:abcd...]
```

---

## 4 · The 8 Universal CI/CD Invariants

Regardless of programming language, build engine, CI provider, or cloud runtime, every valid automated pipeline upholds these 8 mathematical laws:

### 4.1 Invariant 1: Build-Once Artifact Immutability (The Promotion Axiom)
A deployable computational artifact $\mathcal{A}$ must be compiled or packaged exactly once and promoted across target environments without re-compilation or repackaging:
$$\text{Digest}(\mathcal{A}_{\text{staging}}) = \text{Digest}(\mathcal{A}_{\text{prod}})$$
* **Prohibition**: Compiling distinct binaries for staging vs. production or baking environment-specific configurations into build layers.

### 4.2 Invariant 2: Topological Fast Feedback (The Fail-Fast Axiom)
Pipeline stages must be topologically sorted such that execution cost and latency increase with stage depth:
$$\text{Cost}(S_1) \le \text{Cost}(S_2) \le \dots \le \text{Cost}(S_n)$$
$$\text{GateFail}(S_k) \implies \text{AbortAll}(S_{k+1 \dots n})$$
* **Prohibition**: Running long-running integration or browser test suites before cheap static linting and typechecking gates pass.

### 4.3 Invariant 3: Hermetic Build Determinism (The Reproducibility Axiom)
Given an immutable source tree at commit $C$, locked dependency tree $\mathcal{L}$, and declared toolchain version $\mathcal{T}$, pipeline outputs must be functionally identical across any host:
$$f(C, \mathcal{L}, \mathcal{T})_{\text{runner}_A} \equiv f(C, \mathcal{L}, \mathcal{T})_{\text{runner}_B}$$
* **Prohibition**: Unpinned floating dependencies (`latest`, `^`, `*`), unversioned compiler environments, or dependency on ambient runner state.

### 4.4 Invariant 4: Honest Gating & Statistical Signal Integrity (The Proof Axiom)
A pipeline gate is valid if and only if its exit signal is strictly deterministic and non-deceptive:
$$\text{DeterministicGate}(\text{PASS}) \iff \text{RequirementsSatisfied} \land \text{ExitCode} = 0$$
* **Prohibition**: Retrying failing tests in a blind loop to mask flakiness; suppressing non-zero exit codes with `|| true`; ignoring broken test suites on critical branches.

### 4.5 Invariant 5: Safe Caching & Hierarchical Scoping (The Isolation Axiom)
Build caches must be deterministic, isolated by branch scope, and invalidable by input dependency changes:
$$\text{CacheKey} = Hash(\text{ToolchainVersion} + \text{LockfileContent} + \text{EnvironmentKey})$$
$$\text{Scope}(\text{CacheWrite}) = \text{CurrentBranch}; \quad \text{Scope}(\text{CacheRead}) = \{\text{CurrentBranch}, \text{Mainline}\}$$
* **Prohibition**: Allowing unprivileged feature branches to write to the shared mainline cache (cache poisoning), or relying on untracked global cache directories.

### 4.6 Invariant 6: Least Privilege & Zero Ambient Authority (The Security Axiom)
Pipeline jobs must execute with the minimal permissions required for their specific phase, utilizing short-lived ephemeral tokens and zero ambient long-lived secrets:
$$\text{Perms}(\text{Job}) = \bigcap \text{RequiredCapabilities}(\text{Job})$$
* **Two-Tier PR Rule**: Untrusted external PRs run in zero-secret sandboxes. Elevated tokens are restricted strictly to maintainer-approved or post-merge contexts.

### 4.7 Invariant 7: Controlled Concurrency & Obsolete Run Cancellation (The Conservation Axiom)
Pushing a new commit to a development branch must atomically cancel obsolete in-progress pipeline runs on that ref:
$$\text{Push}(C_{t+1}, \text{Ref}) \implies \text{Cancel}(\text{Run}(C_t, \text{Ref}))$$

### 4.8 Invariant 8: Traceability & Actionable Failure Diagnostics (The Observability Axiom)
Every pipeline execution must produce an audit trail connecting source commit, pipeline run, generated artifact digest, and test reports:
$$\mathcal{T}_{\text{run}} = \langle \text{CommitSHA}, \text{RunID}, \text{Digest}(\mathcal{A}), \text{LogsURI}, \text{Attestations} \rangle$$

---

## 5 · The CI/CD Invariant Exception Protocol (Emergency Hotfixes)

When exceptional operational constraints (active Sev-0 production hotfix, catastrophic network partition preventing remote cache retrieval, or air-gapped forensic recovery) conflict with standard pipeline invariants:

> [!CAUTION] CI/CD INVARIANT EXCEPTION PROTOCOL
> An agent or engineer may deliberately bypass non-critical pipeline gates **IF AND ONLY IF**:
> 1. **Operational Blocker Citation**: Explicitly cites the Sev-0/Sev-1 incident ticket and explains why specific stages are bypassed.
> 2. **Non-Negotiable Safety Core**: Mandatory static typecheck, secret scanning, and fundamental smoke sanity can **NEVER** be bypassed.
> 3. **Micro-ADR & Reconciliation Commitment**: Records the bypass decision in the pipeline log with a committed reconciliation plan.

---

## 6 · Universal Archetype Adaptation

The 8 invariants adapt dynamically across every software archetype:

* **Backend Services & APIs**: Static analysis $\to$ Unit tests $\to$ DB migration dry-run $\to$ Integration tests $\to$ OCI image pinned by SHA-256 digest + SBOM.
* **Polyglot Monorepos**: Affected graph discovery $\to$ Topological sub-DAG execution with shared remote caching.
* **Libraries & Public SDKs**: Multi-OS and multi-runtime matrix verification $\to$ Signed package archive published via authenticated token.
* **Mobile Applications**: Lint $\to$ Unit tests $\to$ Headless emulator UI tests $\to$ Signed binary with distribution keys.
* **Embedded & Firmware**: Static analysis $\to$ Simulation / Hardware-in-the-Loop bench test $\to$ Signed binary firmware image.
* **Data & ML Pipelines**: Code lint $\to$ Data contract / schema validation $\to$ Model metric evaluation.
* **AI Agents & MCP Systems**: Prompt schema validation $\to$ Golden benchmark eval suite ($\ge \tau$) $\to$ Token budget audit $\to$ Immutable agent bundle.

---

## 7 · Boundary Routing with Arsenal Skills

* **`ci-cd`**: The automated execution engine for change flows. Orchestrates stages, enforces dependencies, builds artifacts, executes test matrices, and gates promotion.
* **`release-management`**: Release semantics and governance. Decides *WHAT* is being released (versioning, SemVer/CalVer, changelogs, readiness scorecards).
* **`deployment`**: Operational environment delivery. Decides *HOW* a release reaches an environment (canaries, blue-green, traffic shifting, health probes).
* **`configuration-management`**: Manages environment variables and secret stores injected into pipelines.
* **`testing`**: Designs tests, test doubles, and behavioral assertions executed by CI/CD.
* **`security-engineering`**: Defines threat models, vulnerability policies, and SAST/DAST rules.
* **`failure-recovery`**: Responds to pipeline failures, broken builds, or rollback requirements.

---

## 8 · Antipatterns & Failure Modes

* ❌ **Rebuilding Per Environment**: Building separate binaries for dev, staging, and prod.
* ❌ **Vanity Green Gates**: Using `continue-on-error: true` or `|| true` to suppress test failures and achieve fake green builds.
* ❌ **Blind Retry Loops**: Auto-retrying failed test suites multiple times to bypass flakiness instead of fixing root causes.
* ❌ **Fork Secret Leakage**: Running untrusted pull request code with access to production secrets or write tokens.
* ❌ **Shared Cache Poisoning**: Allowing feature branches to overwrite mainline cache entries.
* ❌ **Zombie Concurrency Waste**: Allowing superseded commits to continue running full CI suites.

---

## 9 · The Clean CI/CD Stopping Contract

An automated pipeline or CI/CD task is strictly **COMPLETE** only when:
1. **Pipeline DAG Proven**: The pipeline workflow is valid declarative syntax, passes linting, and represents an acyclic dependency graph.
2. **Deterministic Build Verified**: Artifacts build reproducibly from pinned sources and dependencies with an immutable cryptographic digest.
3. **Honest Quality Gates Cleared**: All static and dynamic gates execute without suppressed exit codes; flaky tests are quarantined rather than retried in blind loops.
4. **Least-Privilege Security Verified**: Token permissions are restricted to minimal required scopes; untrusted PR paths have zero access to production secrets.
5. **Caching & Concurrency Configured**: Cache keys are derived from content hashes, and obsolete runs are automatically cancelled on new commits.
6. **Agent Loop Limit**: When troubleshooting a failing pipeline, the agent must limit automated repair attempts to a maximum of 3 iterations before presenting causal diagnostic logs and escalating to the user.
