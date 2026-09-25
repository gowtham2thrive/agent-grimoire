---
name: configuration-management
description: >-
  Universal, timeless configuration engineering and operational governance protocol.
  Use when defining environment variables, managing secrets, establishing precedence
  hierarchies, enforcing schema validation, maintaining environment parity, orchestrating
  dynamic rollouts, or synchronizing multi-agent system configurations across any stack.
  Enforces the 7 Universal Configuration Invariants, 5-phase lifecycle, 6 cognitive sizing
  modes, mathematical precedence lattices, drift forensics, and the Invariant Exception Protocol
  without limiting agent creativity or restricting to any specific technology.
---

# Configuration Management: Universal Configuration Engineering & Operational Governance Protocol

> **Mandate**: *Configuration is the parameterized boundary between invariant logic and variable operational environments.* Invariant computation (code, binaries, container images, agent system prompts) must remain immutable across deployment lifecycles; all operational dimensions (network endpoints, resource quotas, secret bindings, feature toggles, agent tool policies) are externalized, strictly validated, and deterministically resolved. Unchecked configuration drift, ambiguous precedence cascades, plaintext secret leakage, and unvalidated boot sequences represent catastrophic systemic vulnerabilities. Separation of code and context, deterministic precedence lattices, strict schema contracts, idempotent convergence, and zero ambient secret exposure strictly precede mutation. Never bake variable context into deployable binaries; never commit plaintext secrets to version control; and never apply dynamic runtime mutations without bounded blast-radius controls and instant rollback readiness.

---

## 1 · The 5-Phase Configuration Lifecycle

Every configuration task—whether declaring local `.env` variables, provisioning multi-environment cloud templates, configuring runtime feature flags, or syncing autonomous agent tool policies—traverses this 5-phase protocol:

```mermaid
flowchart LR
    P1["1. Cartography & Hierarchy<br/>(SSOT, Precedence DAG, Schema)"] --> P2["2. Secrets & Boundary Defense<br/>(Zero-Trust, KMS/Vault, Redaction)"]
    P2 --> P3["3. Validation & Pre-Flight Gate<br/>(Strict typing, dry-run, constraints)"]
    P3 --> P4["4. Idempotent Rollout & Convergence<br/>(Canary dial, zero-downtime reload)"]
    P4 --> P5["5. Drift Forensics & Audit<br/>(Reconciliation, distance metric, rollback)"]
```

1. **Phase 1 — Cartography & Hierarchy Definition**: Discover all declared configuration sources, establish the authoritative Single Source of Truth (SSOT), define the typed schema contract, and resolve the acyclic precedence DAG (see [`references/precedence-scoping-and-hierarchy.md`](references/precedence-scoping-and-hierarchy.md)).
2. **Phase 2 — Secrets Isolation & Boundary Defense**: Enforce strict ontological separation between public configuration parameters and sensitive secrets. Validate storage mechanisms (KMS, vaults, ephemeral injected environment variables) and verify zero plaintext in logs, traces, and VCS (see [`references/secrets-management-and-zero-trust.md`](references/secrets-management-and-zero-trust.md)).
3. **Phase 3 — Validation & Pre-Flight Gatekeeping**: Execute static type checking, range validation, semantic cross-field assertions, and dry-run execution against the target environment. Block any mutation with missing, ambiguous, or invalid parameters (see [`references/validation-schema-and-contract-enforcement.md`](references/validation-schema-and-contract-enforcement.md)).
4. **Phase 4 — Idempotent Rollout & Convergence**: Apply desired state idempotently ($f(f(x)) = f(x)$). For dynamic configuration or infrastructure changes, execute ring-based phased rollout (canary $\to$ blast-radius monitoring $\to$ full fleet) with zero-downtime hot-reloads (see [`references/dynamic-config-and-safe-rollout.md`](references/dynamic-config-and-safe-rollout.md)).
5. **Phase 5 — Continuous Reconciliation, Drift Forensics & Rollback**: Monitor runtime divergence against declared state using the formal Drift Distance Metric. Maintain an immutable, cryptographically verifiable change log and maintain immediate rollback readiness across applications, infrastructure, and agent systems (see [`references/environment-parity-and-drift-forensics.md`](references/environment-parity-and-drift-forensics.md) and [`references/agentic-governance-and-cross-tool-sync.md`](references/agentic-governance-and-cross-tool-sync.md)).

---

## 2 · Progressive Disclosure (Reference Routing)

To prevent cognitive overload and preserve LLM context bandwidth, **never load all reference manuals simultaneously**. Consult reference manuals strictly on demand based on task context:

| Context Trigger | Mandatory Reference Manual | Purpose |
| :--- | :--- | :--- |
| **Config precedence, multiple sources, cascading overrides, SSOT** | [`references/precedence-scoping-and-hierarchy.md`](references/precedence-scoping-and-hierarchy.md) | Semi-lattice precedence order, cascading merges, variable expansion, SSOT authority, anti-shadowing. |
| **Config schemas, type safety, boundary validation, pre-flight checks** | [`references/validation-schema-and-contract-enforcement.md`](references/validation-schema-and-contract-enforcement.md) | Typed schemas (JSON Schema, Zod, Pydantic, CUE), type coercion rules, semantic constraints, fail-fast gates. |
| **API keys, passwords, certificates, encryption, .env leakage** | [`references/secrets-management-and-zero-trust.md`](references/secrets-management-and-zero-trust.md) | Secrets vs config ontology, vaults, KMS envelope encryption, rotation lifecycles, log redaction. |
| **Dev vs Staging vs Prod drift, .env.example, preview envs** | [`references/environment-parity-and-drift-forensics.md`](references/environment-parity-and-drift-forensics.md) | Scaling parity, ephemeral previews, drift distance metric $\Delta$, bidirectional state reconciliation. |
| **Runtime toggles, feature flags, zero-downtime reloads, rollbacks** | [`references/dynamic-config-and-safe-rollout.md`](references/dynamic-config-and-safe-rollout.md) | Dynamic reconfig without restarts, canary rings, blast-radius mitigation, circuit breakers, 1-step rollback. |
| **AI agent rules, prompt parameters, tool permissions, MCP sync** | [`references/agentic-governance-and-cross-tool-sync.md`](references/agentic-governance-and-cross-tool-sync.md) | AI agent rulesets, Claude/Cursor/Codex cross-tool sync, SAFE/APPROVAL/LOCKED policy tiers, mutation audits. |

---

## 3 · Adaptive Cognitive Sizing (The 6 Execution Modes)

Size your configuration engineering effort to the task's scope and risk profile. Never apply heavy enterprise bureaucracy to a single-line patch, and never execute speculative, unverified mutations across high-blast-radius production systems:

| Mode | Trigger & Scope | Engineering Discipline | Required Output & Protocol |
| :--- | :--- | :--- | :--- |
| **`schema-lint`** | Adding or modifying config keys, syntax linting, type validation. | Evaluate format, type safety, default values, and schema definitions. | **Schema Validation Verdict**: Pass/Fail with typed schema diff. |
| **`secret-audit`** | Security reviews, pre-commit checks, credential audits. | Scan for plaintext keys, verify `.gitignore`/`.env` isolation, audit redaction. | **Zero-Trust Secret Certification**: Proof of zero plaintext in VCS/logs. |
| **`env-parity-sync`** | Environment drift, staging/prod desynchronization, template updates. | Reconcile `.env.example`, unify cross-environment keys, verify scale-only delta. | **Environment Parity Matrix**: Comprehensive key diff across tiers. |
| **`drift-reconcile`** | Out-of-band mutations, infrastructure audit, config convergence. | Measure live vs declared state divergence, calculate drift metric $\Delta$, plan convergence. | **Drift Forensics Report & Remediation**: Plan to restore desired state. |
| **`dynamic-rollout`** | Feature flagging, runtime tuning, percentage dials, traffic shifting. | Establish canary rings, define health metric gates, configure circuit breakers. | **Safe Rollout Plan & Rollback Trigger**: Ring progression schedule. |
| **`agent-govern`** | AI agent instruction changes, MCP server configs, tool authorization. | Classify by policy tier (`SAFE`, `APPROVAL`, `LOCKED`), audit provenance, sync across runtimes. | **Agent Policy & Cross-Tool Sync Proof**: Multi-agent coherence log. |

---

## 4 · The 7 Universal Configuration Invariants

Regardless of language, framework, cloud provider, or agent harness, every robust computational system upholds these 7 timeless invariants:

### 4.1 Invariant 1: Invariant Logic & Operational Context Separation (The Invariance Axiom)
A deployable computational artifact (binary, container, script, or agent system prompt) must be immutable and compile once. 
- Any parameter that varies across deployment targets, environments, regions, or tenants must be injected at the execution boundary, never hardcoded into logic.
- Building separate binaries for different environments is strictly prohibited.

### 4.2 Invariant 2: Deterministic Precedence Lattice & Attribution (The Cascade Axiom)
Configuration resolution must follow an unambiguous, acyclic Upper Semi-Lattice $(\mathcal{S}, \prec)$:
$$V_{\text{resolved}}(k) = \max_{\prec} \{ v_i \in \text{Sources}(k) \}$$
- When multiple sources supply a key, resolution order must be mathematically deterministic.
- Every resolved value must possess traceable provenance: the system can always answer *which source* provided the active value and *why* it superseded lower tiers.

### 4.3 Invariant 3: Ontological Distinction of Secrets (The Zero-Trust Secrecy Axiom)
Secrets (private keys, tokens, passwords, certificates) are mathematically distinct from configuration.
- Configuration is public operational metadata; secrets are high-liability access credentials.
- Secrets must never be stored in plain text, never committed to version control, never exposed to client browsers, and never output in application logs, traces, or agent conversation context.

### 4.4 Invariant 4: Strict Schema & Pre-Flight Validation (The Fail-Fast Boundary Axiom)
Configuration input constitutes an untrusted boundary.
- Systems must validate configuration values against a rigid, typed schema (checking types, formats, ranges, and cross-field constraints) before bootstrapping runtime services.
- A misconfigured system must fail fast during pre-flight startup or dry-run evaluation, never crashing midway through serving production traffic.

### 4.5 Invariant 5: Declarative Desired State & Idempotency (The Convergence Axiom)
Configuration changes must be expressed as the desired end-state, not imperative action scripts.
- Applying a configuration specification must be strictly idempotent:
  $$f(f(x)) = f(x)$$
- Re-applying the identical configuration against a converged system must produce zero net mutation and zero side-effects.

### 4.6 Invariant 6: Scaled Environment Parity (The Equivalence Axiom)
Development, staging, preview, and production environments are fundamentally the same system operating at different sizes.
- Any divergence between environments must be strictly confined to resource capacity, performance tiering, and external endpoint bindings.
- Divergent architectural flows or environment-specific conditional branching inside application code violate parity.

### 4.7 Invariant 7: Bounded Blast-Radius & Instant Rollback (The Containment Axiom)
Dynamic runtime reconfigurations and feature rollouts must be guarded by bounded exposure envelopes (canary cohorts, ring progression, percentage gating).
- Dynamic changes must be continuously monitored for error budget degradation:
  $$R_{\text{blast}}(M) = \left( \frac{N_{\text{exposed}}}{N_{\text{total}}} \right) \times C_{\text{impact}}(M)$$
- The system must maintain an immutable rollback posture: any configuration mutation must be instantly revertible to the previous known-good state with zero downtime.

---

## 5 · The Configuration Invariant Exception Protocol (Extreme Edge Cases)

No single static set of rules can accommodate 100% of real-world operational anomalies without breaking. When exceptional operational constraints (e.g. legacy brownfield monoliths with hardcoded settings, air-gapped enclaves without external vaults, emergency zero-day production hotfixes, or firmware hardware constraints) conflict with standard configuration invariants, the agent invokes this protocol:

> [!CAUTION] CONFIGURATION INVARIANT EXCEPTION PROTOCOL
> An agent may deliberately bypass a configuration invariant (e.g., using an emergency local override, applying an unversioned runtime hot-patch, or temporarily hardcoding a fallback endpoint) **IF AND ONLY IF**:
> 1. **Operational Constraint Citation**: Explicitly cites the physical or organizational blocker (e.g., *"Upstream KMS unreachable during network partition in emergency hotfix"* or *"Legacy vendor binary only accepts static embedded config"*).
> 2. **Quarantined Boundary Containment**: Confines the exception inside an isolated boundary (e.g. a dedicated shim module, an encrypted local envelope, or a quarantined staging branch).
> 3. **Micro-ADR**: Records the trade-off, rationale, and technical debt retirement ticket in an immutable record (`[CONFIG-EXCEPTION: emergency manual override for outage incident INC-8492 pending post-mortem convergence]`).

---

## 6 · Universal Archetype Adaptation

The 7 invariants adapt dynamically across every software ecosystem by abstracting tools to universal configuration roles:

| Dimension | Application & Microservices | Infrastructure as Code & Orchestration | Dynamic Runtime & Edge | AI Agent Systems & MCP | OS & System Engineering |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Declared Desired State (SSOT)** | Version-controlled config files (`config.yaml`, `settings.json`) | Declarative templates (Terraform HCL, K8s manifests, Ansible playbooks) | Centralized dynamic key-value stores (Consul, etcd, Redis, Feature flag SaaS) | Canonical workspace rulebooks (`AGENTS.md`, `SKILL.md`, MCP JSON manifests) | System configuration tree (`/etc/*`, systemd units, dotfiles, registry) |
| **Boundary Injection** | Process environment variables (`process.env`, `os.environ`) | Host parameters, cloud-init, metadata service injection | Polling or WebSocket subscription streams, local cache fallbacks | Prompt context injection, tool argument schemas, MCP server env blocks | Kernel parameters (`sysctl`), init system environment files |
| **Secrets Management** | KMS envelope encryption, secret manager sidecars, in-memory vault mounts | Provider-managed secrets, sealed secrets, ephemeral IAM tokens | Encrypted payloads with zero-knowledge runtime decryption keys | Ephemeral session tokens, mediated tool execution, secret proxying | Hardware security modules (HSM), TPM chips, secure enclaves |
| **Precedence Resolution** | Defaults $\to$ File $\to$ Env Var $\to$ CLI Flag | Base module $\to$ Environment tfvars $\to$ CLI variable overrides | Global default $\to$ Environment toggle $\to$ User cohort $\to$ Request header | Global default $\to$ Workspace rule $\to$ Subagent prompt $\to$ Tool parameter | Compile default $\to$ `/usr/lib` $\to$ `/etc` $\to$ `/run` $\to$ `$HOME/.config` |
| **Validation Gate** | Startup pre-flight parser (Zod, Pydantic, struct unmarshaler) | Static linter / plan stage (`terraform plan`, `kubeconform`, dry-run) | Schema enforcement on change publish, client fallback sanity checks | Agent evaluation verification gate, tool schema conformance check | Kernel parameter range check, daemon configuration syntax test (`nginx -t`) |
| **Drift Forensics** | Continuous reconciliation against orchestrator desired replica state | Continuous state reconciliation (`terraform plan -detailed-exitcode`) | Heartbeat telemetry vs active configuration version hashes | Agent rulebook hash verification across tools and sub-agent worktrees | Configuration integrity monitors (`aide`, `tripwire`, file hash checks) |

---

## 7 · Guardrails & Anti-Patterns

### Strictly Disallowed Actions
- ❌ **No Plaintext Secrets in Repositories or Logs**: Never commit passwords, tokens, private keys, or `.env` files containing credentials to version control. Never print credentials in logs or agent conversational outputs.
- ❌ **No Ambiguous Precedence (Shadowing)**: Never establish a configuration topology where two conflicting sources can override a parameter without an explicit, deterministic precedence ordering.
- ❌ **No Silent Failure / Unvalidated Bootstrapping**: Never allow a service or agent to boot with invalid, unparsed, or malformed configuration using silent default fallbacks that mask critical misconfigurations.
- ❌ **No Client-Side Secret Leakage**: Never bundle server-side credentials, database connection strings, or master API tokens into frontend web client code, mobile client bundles, or public edge scripts.
- ❌ **No Out-of-Band Production Drift**: Never apply direct manual mutations ("hot-fixing" via cloud console or SSH edits) without immediately committing the change to the declarative version-controlled source of truth.
- ❌ **No Ungoverned High-Risk Agent Mutations**: Never permit an autonomous agent to alter `LOCKED` security boundaries, tool execution scopes, or authentication secrets without explicit approval gates.
- ❌ **No Un-revertible Dynamic Rollouts**: Never push a dynamic configuration or feature flag without a tested, zero-downtime 1-step rollback path and active telemetry monitoring.

---

## 8 · The Clean Configuration Stopping Contract

A configuration management task is strictly **COMPLETE** only when:
1. **Schema & Typings Verified**: All configuration parameters conform to a strict, typed schema; invalid inputs are demonstrably rejected with clear causal error diagnostics.
2. **Secrets Hermetically Isolated**: Zero plaintext secrets exist in version control, diffs, or client-facing artifacts. All credentials utilize approved vault/KMS injection channels.
3. **Deterministic Precedence Proven**: All overrides, environment profiles, and cascading values resolve deterministically with traceable source provenance.
4. **Environment Parity & Convergence Demonstrated**: Configuration templates (`.env.example`, base profiles) reflect all active keys. Live target state converges idempotently ($f(f(x)) = f(x)$) with zero unmanaged drift ($\Delta = 0$).
5. **Rollback & Audit Trail Certified**: Every change is recorded in an attributable version-controlled commit or immutable audit log, with a verified instant rollback mechanism.
