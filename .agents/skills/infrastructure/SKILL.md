---
name: infrastructure
description: >-
  Universal infrastructure engineering, declarative resource topology, and state convergence protocol.
  Use when provisioning, modifying, adopting, or decommissioning compute, network, storage, IAM, or backing service resources
  across any platform (Cloud, On-Premises, Kubernetes, Serverless, Bare-Metal, or AI/GPU Compute Swarms).
  Do NOT activate for logical software architecture and module contracts (use system-architecture),
  or application configuration schemas and environment variables (use configuration-management).
  Enforces the 8 Universal Infrastructure Invariants, 7-stage closed-loop lifecycle, 6 cognitive sizing modes,
  the State Triad reconciliation model, acyclic dependency DAGs, partitioned blast-radius containment,
  non-destructive brownfield adoption, and zero ambient authority.
---

# Infrastructure Engineering: Universal Resource Topology & State Convergence Protocol

> **Mandate**: *Infrastructure is the receptive computational substrate, resource topology, and security perimeter upon which software systems, data pipelines, and autonomous agent swarms execute.* Infrastructure engineering does not prescribe tools or cloud vendors; it is an invariant discipline governing declarative desired state, acyclic dependency graph execution, partitioned state blast-radius containment, two-phase plan-apply verification, zero-trust network boundaries, brownfield unmanaged resource adoption, and continuous drift reconciliation. True infrastructure engineering preserves agent creativity, rejects platform dogma, and guarantees that every computational foundation is reproducible, auditable, secure, and recoverable across any runtime environment.

---

## 1 · The 7-Stage Universal Infrastructure Lifecycle

Every infrastructure task—from spinning up a local test container to orchestrating a global multi-region cloud topology or provisioning an autonomous GPU cluster—traverses this closed-loop lifecycle:

```mermaid
flowchart LR
    S1["1. Discover & Model<br/>(Workload Envelope, DAG, Stacks)"] --> S2["2. Specify & Guard<br/>(Declarative Code, IAM, Policies)"]
    S2 --> S3["3. Lock & Plan<br/>(Remote Lock, Plan Diff, Blast Calc)"]
    S3 --> S4["4. Gate & Authorize<br/>(Policy Audit, Cost, Sign-off)"]
    S4 --> S5["5. Converge & Mutate<br/>(Topological Apply, Idempotence)"]
    S5 --> S6["6. Verify & Baseline<br/>(Health Probes, Telemetry, Drift Baseline)"]
    S6 --> S7["7. Govern & Reconcile<br/>(Drift Detection, FinOps, Tombstoning)"]
```

1. **Stage 1 — Discover & Model (Workload & Topology Discovery)**: Model capacity constraints (CPU, RAM, IOPS, latency, throughput). Decompose the environment into decoupled stack tiers (Network $\to$ Data $\to$ Compute $\to$ Ingress) to isolate failure blast radius. In brownfield environments, discover and inventory existing physical or cloud assets (see [`references/stack-composition-and-dependency-dags.md`](references/stack-composition-and-dependency-dags.md) and [`references/brownfield-discovery-and-resource-import.md`](references/brownfield-discovery-and-resource-import.md)).
2. **Stage 2 — Specify & Guard (Declarative Specification & Shift-Left Policy)**: Declare target resources in version-controlled manifests. Enforce resource attribution tags, default-deny network microsegmentation, and zero-trust IAM matrices. Execute pre-commit syntax linting and policy-as-code evaluations (see [`references/iac-security-and-policy-guardrails.md`](references/iac-security-and-policy-guardrails.md) and [`references/network-topology-and-perimeter-defense.md`](references/network-topology-and-perimeter-defense.md)).
3. **Stage 3 — Lock & Plan (Atomic State Locking & Plan Diff Generation)**: Acquire an atomic distributed state lock with lease TTL. Refresh recorded state against live reality ($S_{\text{observed}}$). Compute the exact plan diff $\mathcal{P} = S_{\text{desired}} \ominus S_{\text{observed}}$ and calculate the blast-radius risk metric $R_{\text{blast}}$ (see [`references/desired-state-and-drift-engine.md`](references/desired-state-and-drift-engine.md)).
4. **Stage 4 — Gate & Authorize (Reviewability & Approval Gate)**: Inspect planned resource creations, in-place updates, and destructions. Validate cost impact against FinOps thresholds. For high blast-radius changes or stateful data store recreations, obtain explicit human authorization before mutating physical reality.
5. **Stage 5 — Converge & Mutate (Idempotent Topological Apply)**: Apply mutations strictly along the topological ordering of the dependency graph ($G = (V, E)$). Handle provider eventual consistency with jittered exponential backoffs. Commit updated physical bindings to the remote encrypted state ledger ($S_{\text{recorded}}$).
6. **Stage 6 — Verify & Baseline (Mechanical Verification & Drift Baseline)**: Execute mechanical connectivity and health probes (socket bindings, DNS resolution, IAM assumption). Verify telemetry scaffolding (flow logs, audit logs, saturation metrics). Seal the post-apply state checksum as the active drift baseline.
7. **Stage 7 — Govern & Reconcile (Continuous Drift, FinOps & Decommissioning)**: Run scheduled drift detection loops to catch out-of-band mutations. Reclaim orphaned and unattached resources. Execute reverse-topological decommissioning with mandatory pre-destroy snapshots for stateful storage (see [`references/finops-and-resource-lifecycle.md`](references/finops-and-resource-lifecycle.md)).

---

## 2 · Progressive Disclosure (Reference Routing)

To prevent cognitive overload and preserve LLM context bandwidth, consult reference manuals strictly on demand based on task context:

| Context Trigger | Mandatory Reference Manual | Purpose |
| :--- | :--- | :--- |
| **State backends, remote locking, state corruption, drift detection, GitOps** | [`references/desired-state-and-drift-engine.md`](references/desired-state-and-drift-engine.md) | State Triad ($S_{\text{desired}} \iff S_{\text{recorded}} \iff S_{\text{observed}}$), distributed locking, lease TTLs, drift distance metric, clean NoOp reconciliation. |
| **Multi-stack design, monolithic state refactoring, DAGs, cross-stack wiring** | [`references/stack-composition-and-dependency-dags.md`](references/stack-composition-and-dependency-dags.md) | Stack hierarchy, topological ordering, state partitioning, output contracts, resolving circular deadlocks. |
| **Existing cloud resources, unmanaged infrastructure, clickops remediation** | [`references/brownfield-discovery-and-resource-import.md`](references/brownfield-discovery-and-resource-import.md) | Non-destructive adoption, live discovery scans, declarative code synthesis, zero-churn import invariance. |
| **VPC, CIDR, subnets, routing, VPN, Transit Gateway, firewalls, private endpoints** | [`references/network-topology-and-perimeter-defense.md`](references/network-topology-and-perimeter-defense.md) | Non-overlapping CIDR math, 3-tier subnet topology, transit hubs, security group microsegmentation, flow logs. |
| **IAM roles, least privilege, security scanning, CIS compliance, secrets** | [`references/iac-security-and-policy-guardrails.md`](references/iac-security-and-policy-guardrails.md) | Shift-left security pipeline, OIDC federation, zero static keys, policy-as-code, KMS envelope encryption. |
| **Cloud costs, instance right-sizing, idle resources, deprovisioning, snapshots** | [`references/finops-and-resource-lifecycle.md`](references/finops-and-resource-lifecycle.md) | Workload envelope capacity planning, mandatory FinOps tagging, orphan asset reaping, Pre-Destroy Snapshot Gate. |
| **AI agent hosts, GPU provisioning, Slurm, Ray, MCP infrastructure, vector DBs** | [`references/agentic-infrastructure-and-compute-fabrics.md`](references/agentic-infrastructure-and-compute-fabrics.md) | GPU interconnect topologies, NUMA node affinity, cluster manifests, sandboxed MCP server hosting, isolated execution microVMs. |

---

## 3 · Adaptive Cognitive Sizing (The 6 Execution Modes)

Size your infrastructure engineering effort to the task's scope and risk profile. Never apply heavy enterprise bureaucracy to a local container sandbox, and never execute speculative, unverified mutations across high-blast-radius production environments:

| Mode | Trigger & Scope | Engineering Discipline | Required Output & Protocol |
| :--- | :--- | :--- | :--- |
| **`ephemeral-local`** | Local development harnesses, Docker Compose, Kind/K3s clusters, disposable test sandboxes. | Rapid spin-up/teardown, zero remote state locking boilerplate, local port mapping. **Zero red tape**. | **3-Line Infrastructure Intent Block** directly preceding execution. |
| **`module-contract`** | Creating or editing reusable infrastructure modules (Terraform, Pulumi components, Helm charts). | Strict input validation, explicit output contracts, default-secure parameters, SemVer tag. | Module Contract Specification: Typed inputs/outputs table + lint/validation status. |
| **`stack-converge`** | Standard cloud infrastructure changes (adding subnets, provisioning clusters, scaling pools). | State locking, plan diff generation, blast-radius calculation, topological apply. | Plan Diff Summary & Blast-Radius Score: Detailed breakdown of Create/Modify/Destroy. |
| **`brownfield-import`** | Adopting unmanaged, legacy, or manually created cloud/hardware assets into declared code. | Live discovery, non-destructive state import, zero-churn code generation, drift verification. | Adoption Proof & Parity Diff: Confirmation that $S_{\text{desired}} \equiv S_{\text{observed}}$ with zero recreation. |
| **`perimeter-harden`** | Network topology updates, firewall rules, security groups, transit gateways, IAM roles. | Zero-trust microsegmentation, default-deny verification, egress route audit, IAM simulation. | Perimeter Security Attestation: Ingress/egress matrix + IAM least-privilege proof. |
| **`disaster-recovery`** | Cross-region failover, state ledger reconstruction, snapshot restoration, emergency failover. | Blast-radius containment, state ledger backup restoration, DNS weight shifting, post-recovery audit. | DR Execution Runbook & Verification: Health status of promoted standby resources. |

### The 3-Line Infrastructure Intent Protocol (For `ephemeral-local` mode)
When operating in `ephemeral-local` mode, summarize intent in exactly 3 lines before emitting commands or mutations:
```markdown
> **Target**: [Local / Kind / Docker Compose / Ephemeral Sandbox]
> **Action**: [Up / Down / Provision] (Zero remote state lock required)
> **Verification**: [Port / Socket / Process reachability verified: PASS]
```

> **Boundary**: This skill owns *physical resource provisioning and state convergence* — IaC, networking, compute, drift reconciliation. For *logical system design* (module decomposition, API contracts, state authority), activate `system-architecture` instead.

---

## 4 · The 8 Universal Infrastructure Invariants

Regardless of language, framework, cloud provider, or runtime environment, every robust computational system upholds these 8 timeless invariants:

### 4.1 Invariant 1: Declarative Desired State & State Triad Equivalence (The Convergence Axiom)
Infrastructure must be expressed declaratively as the target state. The infrastructure engine continuously reconciles the State Triad:
$$\Delta_{\text{drift}} = (S_{\text{desired}} \ominus S_{\text{recorded}}) \cup (S_{\text{recorded}} \ominus S_{\text{observed}})$$
- $S_{\text{desired}}$: The version-controlled specification.
- $S_{\text{recorded}}$: The persisted state ledger containing physical resource identifiers and metadata.
- $S_{\text{observed}}$: The actual live physical reality queried from provider APIs.
- Mutation is valid if and only if it brings $S_{\text{observed}} \to S_{\text{desired}}$ while maintaining state consistency.

### 4.2 Invariant 2: Acyclic Topological Dependency DAG (The Ordering Axiom)
Infrastructure resources and modules form a Directed Acyclic Graph $G = (V, E)$:
$$\forall (u, v) \in E, \quad u \prec v \implies \text{Ready}(u) \text{ strictly precedes } \text{Create}(v)$$
- A network subnet must exist before an IP can be bound; an IAM role must exist before a compute node can assume it; an encryption key must be enabled before an encrypted volume can mount.
- Circular dependencies are strictly illegal and must be resolved via decoupling.

### 4.3 Invariant 3: Blast-Radius Minimization & State Partitioning (The Isolation Axiom)
State ledgers and resource collections must be partitioned by lifecycle, fault domain, and access tier:
$$\mathcal{S} = \bigcup_{i=1}^k S_i \quad \text{where} \quad S_i \cap S_j = \emptyset \quad (i \neq j)$$
- Monolithic state files (e.g. single global state holding networking, databases, and ephemeral applications) are strictly prohibited.

### 4.4 Invariant 4: Two-Phase Plan-Before-Mutate Gate (The Reviewability Axiom)
Every state-altering operation must be preceded by an attributable, reviewable diff plan:
$$\mathcal{P} = S_{\text{desired}} \ominus S_{\text{observed}} = \langle \mathcal{R}_{\text{create}}, \mathcal{R}_{\text{update}}, \mathcal{R}_{\text{destroy}}, \mathcal{R}_{\text{noop}} \rangle$$
- Execution must never occur without prior computation and inspection of the plan diff $\mathcal{P}$.
- Compute blast radius based on resource statefulness, recreation necessity, and dependency depth. Any plan scheduling destructive changes to persistent data stores, encryption keys, or critical network backbones mandates explicit human authorization.

### 4.5 Invariant 5: Idempotent Convergence (The Determinism Axiom)
Re-applying an identical infrastructure specification against an already-converged environment must produce zero mutations and zero unmanaged side-effects:
$$I(I(E)) = I(E) \implies \mathcal{P}_{\text{subsequent}} = \emptyset$$

### 4.6 Invariant 6: Zero Ambient Authority & Perimeter Defense (The Least-Privilege Axiom)
Infrastructure resources and agent provisioning credentials must possess minimal necessary permissions:
$$\text{Perm}(R) = \min(\text{RequiredCap}(R)), \quad \text{Perimeter}(N) = \text{DefaultDeny}(\text{Ingress}) \land \text{Restricted}(\text{Egress})$$
- Wildcard IAM policies and open ingress on administrative ports are prohibited.

### 4.7 Invariant 7: Immutable Provenance & Attributable Ownership (The Governance Axiom)
Every provisioned physical resource must possess unambiguous attribution (Owner, Environment, Service, CostCenter). Untagged or unowned resources are treated as anomalous drift and flagged for quarantine or decommissioning.

### 4.8 Invariant 8: Non-Destructive Continuity & Snapshot Gate (The Safety Axiom)
A destructive mutation against any stateful infrastructure element (databases, object buckets, persistent block volumes, DNS zones) is invalid without a verified pre-mutation snapshot. In brownfield operations, adopting an unmanaged resource into state must never recreate or restart the live asset.

---

## 5 · The Infrastructure Invariant Exception Protocol (Extreme Edge Cases)

When exceptional operational constraints (active zero-day production hotfixes, air-gapped enclaves, legacy single-instance databases requiring locked maintenance windows, or physical hardware failures) conflict with standard infrastructure invariants:

> [!CAUTION] INFRASTRUCTURE INVARIANT EXCEPTION PROTOCOL
> An agent may deliberately bypass an infrastructure invariant **IF AND ONLY IF**:
> 1. **Operational Blocker Citation**: Explicitly cites the physical or operational necessity.
> 2. **Blast-Radius Quarantining**: Confines the bypass to the minimal necessary blast radius.
> 3. **Micro-ADR & Convergence Commitment**: Records the decision in an immutable record with a committed post-incident reconciliation date.

---

## 6 · Universal Archetype Adaptation

The 8 invariants adapt dynamically across every computational archetype:

* **Public Cloud**: Declarative IaC engines; cloud object store + remote state locking; virtual networks, subnets, and security groups; cloud VMs, container clusters; cloud IAM roles and KMS encryption.
* **Cloud-Native / Kubernetes**: Declarative cluster resource manifests; etcd distributed state; network policies and service meshes; pods and replica sets; persistent volume claims and CSI drivers; RBAC service accounts.
* **On-Premises & Bare Metal**: Host configuration management; remote cluster or file locks; VLANs, BGP routers, and hardware firewalls; physical blades and hypervisors; SAN/NAS storage arrays; LDAP/SSH keyrings.
* **Serverless & Edge**: Serverless application models; managed platform state; edge routing and zero trust access; edge workers and serverless functions; managed object and database storage; ephemeral JWTs and edge tokens.
* **AI Swarms & Compute Fabrics**: Cluster batch manifests; orchestrator DB / shared mounts; private high-speed interconnects; GPU nodes and distributed workers; shared high-throughput NVMe scratch arrays; mutual TLS certs and capability tokens.

---

## 7 · Guardrails & Strictly Disallowed Anti-Patterns

* ❌ **No Blind Applies without Plan Review**: Never execute an infrastructure mutation without generating, inspecting, and logging the reviewable plan diff.
* ❌ **No Monolithic Shared State Files**: Never lump networking, data stores, and ephemeral applications into a single shared state file. Stacks must be partitioned by failure domain and lifecycle.
* ❌ **No Plaintext Secrets in Manifests or State**: Never hardcode API keys, database passwords, or private certificates into IaC code. Use secret manager data lookups or KMS envelope encryption.
* ❌ **No Wildcard Open Network Ingress (`0.0.0.0/0`)**: Never open management ports to the public internet. Enforce bastion hosts, VPNs, or private endpoints.
* ❌ **No Destructive Recreates Without Verified Snapshots**: Never destroy or replace a stateful resource without first verifying an independent snapshot.
* ❌ **No Circular Stack Dependencies**: Never create architecture where Stack A requires outputs of Stack B while Stack B requires outputs of Stack A. Wire acyclic hierarchies via contract extraction.
* ❌ **No Untracked Out-of-Band Mutations**: Never manually modify resources in web consoles without immediately reconciling them back into declared code or state.
* ❌ **No Untagged Orphan Infrastructure**: Never provision resources without mandatory attribution metadata.

---

## 8 · Ecosystem Boundary Routing

* **[`system-architecture`](../system-architecture/SKILL.md)**: Owns the logical structural design (module DAGs, interface contracts, state authority). `infrastructure` converts those logical modules into physical compute, network, and storage topology.
* **[`configuration-management`](../configuration-management/SKILL.md)**: Owns application-level configuration keys, environment variables, and runtime secret injection. `infrastructure` provisions the physical secret vaults, KMS keys, and host environments where configs live.
* **[`deployment`](../deployment/SKILL.md)**: Owns orchestrating software artifact transitions onto receptive infrastructure. `infrastructure` ensures that target environments possess capacity, network reachability, and health probe endpoints.

---

## 9 · The Clean Infrastructure Stopping Contract

An infrastructure engineering task is strictly **COMPLETE** only when all 6 exit criteria are certified with verifiable evidence:

1. **State Triad Convergence Certified**: The declared code ($S_{\text{desired}}$), state ledger ($S_{\text{recorded}}$), and observed live environment ($S_{\text{observed}}$) are identical. A post-apply refresh yields $\mathcal{P} = \emptyset$ (Zero drift / Clean NoOp).
2. **Topological Health Verified**: All provisioned resources in the DAG have passed mechanical health checks (process running, sockets bound, endpoints answering, routes verified).
3. **Perimeter Defense Attested**: Ingress/egress rules, security groups, and IAM policies have been audited and conform to the zero-trust, least-privilege specification.
4. **State Ledger Protection Confirmed**: State files are securely locked, encrypted at rest, and stored in a remote, versioned backend with atomic locking enabled.
5. **Attribution & FinOps Tagging Validated**: Every modified or created resource possesses mandatory ownership, service, environment, and cost tags.
6. **Recovery & Drift Baseline Established**: Stateful resources possess pre-mutation recovery checkpoints (snapshots/backups), and the post-convergence state hash is recorded as the active drift baseline.
