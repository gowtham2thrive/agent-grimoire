<div align="center">

  <img src="logo.svg" alt="Agent Grimoire Logo" width="200" />
  <h1 align="center" style="border-bottom: none">Agent Grimoire</h1>

  <p>
    <strong>An evolving library of reusable AI agent skills, workflows, and protocols for building and evolving software autonomously.</strong>
  </p>

  <p>
    <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square" alt="License: MIT" /></a>
    <a href="#core-tenets"><img src="https://img.shields.io/badge/Agents-Agent--Agnostic-38bdf8.svg?style=flat-square" alt="Agents: Agent-Agnostic" /></a>
  </p>

  <p>
    <a href="#overview"><strong>Overview</strong></a> •
    <a href="#built-in-skills-catalog"><strong>Skill Catalog</strong></a> •
    <a href="#skill-architecture"><strong>Skill Architecture</strong></a> •
    <a href="#skill-archetype-patterns"><strong>Archetype Patterns</strong></a> •
    <a href="#installation--usage"><strong>Installation & Usage</strong></a> •
    <a href="#verification-did-it-work"><strong>Verification</strong></a> •
    <a href="#acknowledgments--references"><strong>Acknowledgments</strong></a>
  </p>

</div>

## Overview

**Agent Grimoire** provides a standardized, battle-tested collection of deterministic skills, operational guardrails, and decision frameworks for autonomous AI coding agents.

Rather than relying on unguided prompts or rigid scripts, Grimoire equips agents with **adaptive execution lifecycles**—enforcing pre-flight discovery, intent disambiguation, safety verifications, and high-signal communication.

### Core Tenets

| Tenet | Focus | Operational Principle |
| :--- | :---: | :--- |
| **Intent Disambiguation** | **Context First** | Prevents blind execution by analyzing workspace state, environment conditions, and user goals before running actions. |
| **Zero Data Loss & Hygiene** | **Defensive Execution** | Enforces strict guardrails against credential leaks, irreversible destructive operations, and unverified bulk mutations. |
| **Framework Agnostic** | **Universal Portability** | Native compatibility with Google Antigravity, Claude Code, Cursor, Copilot Workspace, Windsurf, and custom agent harnesses. |
| **Autonomous Evolution** | **Continuous Alignment** | Engineered to adapt dynamically alongside agent runtimes, tool-calling APIs, and modern development stacks. |

---

## Built-in Skills Catalog

Agent Grimoire provides an integrated suite of **30 battle-tested, deterministic skills** covering the entire software development lifecycle. Each skill is packaged as an independent capability adhering to the universal protocol contract:

| Skill | Category | Mandate & Purpose | Entrypoint |
| :--- | :--- | :--- | :--- |
| **`accessibility`** | Inclusive Access | Task primacy, 8 Universal Accessibility Invariants, 4-tier Epistemic Evidence Ladder, LIFO focus stack machine, and WCAG compliance across all medium interfaces. | [`.agents/skills/accessibility/SKILL.md`](.agents/skills/accessibility/SKILL.md) |
| **`agent-evaluation`** | Certification | Universal 5-Point Verification Gate (traceability, static types, test suite, diff sanity, negative boundary validation) before declaring completion. | [`.agents/skills/agent-evaluation/SKILL.md`](.agents/skills/agent-evaluation/SKILL.md) |
| **`api-design`** | Interface Design | Durable contracts first, 8 Universal API Invariants, mathematical idempotency, Hyrum's Law shielding, canonical failure domains, and monotonic compatibility. | [`.agents/skills/api-design/SKILL.md`](.agents/skills/api-design/SKILL.md) |
| **`ci-cd`** | Automation | Declarative Directed Acyclic Graph (DAG) pipelines, 8 Universal CI/CD Invariants, build-once artifact immutability, safe caching, and tamper-evident gates. | [`.agents/skills/ci-cd/SKILL.md`](.agents/skills/ci-cd/SKILL.md) |
| **`code-quality`** | Construction | Boundary validation, algebraic state modeling, structured causal error hygiene, deterministic resource lifetimes (RAII), and concurrency safety. | [`.agents/skills/code-quality/SKILL.md`](.agents/skills/code-quality/SKILL.md) |
| **`code-review`** | Audit | Two-stage review gate (spec compliance first, then code quality), 4 specialized audit lenses, 0–100 confidence scoring, and noise suppression. | [`.agents/skills/code-review/SKILL.md`](.agents/skills/code-review/SKILL.md) |
| **`configuration-management`** | Operations | 7 Universal Configuration Invariants, deterministic precedence lattices, schema validation, environment parity, and secrets vs. config separation. | [`.agents/skills/configuration-management/SKILL.md`](.agents/skills/configuration-management/SKILL.md) |
| **`data-management`** | Data Architecture | 8 Universal Data Invariants, non-destructive schema evolution (Expand/Contract), idempotent pipelines ($f(f(x))=f(x)$), and data quality gates. | [`.agents/skills/data-management/SKILL.md`](.agents/skills/data-management/SKILL.md) |
| **`dependency-management`** | Supply Chain | 7 Universal Dependency Invariants, transitive tree cartography, pre-adoption gating, script sandboxing, and reachability-based triage. | [`.agents/skills/dependency-management/SKILL.md`](.agents/skills/dependency-management/SKILL.md) |
| **`deployment`** | Delivery | 7 Universal Deployment Invariants, 7-stage closed-loop lifecycle, risk-proportional blast radius, dual-horizon health, and deterministic rollback. | [`.agents/skills/deployment/SKILL.md`](.agents/skills/deployment/SKILL.md) |
| **`design-philosophy`** | Visual Design | Values $\to$ Principles $\to$ Moves, perceptual physics, Gestalt grouping, typographic rhythm, 60-30-10 chromatic restraint, and scaled state completeness. | [`.agents/skills/design-philosophy/SKILL.md`](.agents/skills/design-philosophy/SKILL.md) |
| **`docs-pro`** | Knowledge | Architecture playbooks, ADRs, READMEs, Diátaxis framework, anti-drift synchronization, and progressive disclosure. | [`.agents/skills/docs-pro/SKILL.md`](.agents/skills/docs-pro/SKILL.md) |
| **`failure-recovery`** | Dev Resilience | Decision-centric self-healing, 7 Recovery Invariants, 3-tier fault attribution, assumption backtracking, and convergence verification. | [`.agents/skills/failure-recovery/SKILL.md`](.agents/skills/failure-recovery/SKILL.md) |
| **`github-pro`** | Mechanics | Git and GitHub workflow automation, intent disambiguation, pre-flight safety checks, release milestone management, and conflict recovery. | [`.agents/skills/github-pro/SKILL.md`](.agents/skills/github-pro/SKILL.md) |
| **`incident-response`** | Live Resilience | 12-phase operational crisis lifecycle, reversible containment over curiosity, blast radius containment (P1–P4), and blameless post-incident reviews (PIR). | [`.agents/skills/incident-response/SKILL.md`](.agents/skills/incident-response/SKILL.md) |
| **`infrastructure`** | Substrate | 8 Universal Infrastructure Invariants, declarative desired state, State Triad reconciliation, partitioned blast-radius containment, and brownfield adoption. | [`.agents/skills/infrastructure/SKILL.md`](.agents/skills/infrastructure/SKILL.md) |
| **`maintenance`** | Vitality | Continuous deterioration detection, 8 Universal Maintenance Invariants, dead-code excision ($C_{\text{dead}} \ge 0.95$), and technical-debt quantification ($I_{\text{debt}}$). | [`.agents/skills/maintenance/SKILL.md`](.agents/skills/maintenance/SKILL.md) |
| **`multi-agent-orchestration`** | Coordination | Parallel and phased sub-agent decomposition, isolated worktrees/branches, durable task contracts, and holistic merge verification. | [`.agents/skills/multi-agent-orchestration/SKILL.md`](.agents/skills/multi-agent-orchestration/SKILL.md) |
| **`observability`** | Telemetry | Question-first telemetry design, 8 Universal Observability Invariants, continuous causal context propagation, and trace-based RCA. | [`.agents/skills/observability/SKILL.md`](.agents/skills/observability/SKILL.md) |
| **`performance-engineering`** | Optimization | Measurement-first profiling, 8 Universal Performance Invariants, Amdahl's Law alignment, distribution percentiles, and CI regression shielding. | [`.agents/skills/performance-engineering/SKILL.md`](.agents/skills/performance-engineering/SKILL.md) |
| **`planning`** | Execution | Universal implementation planning, 7 Universal Planning Invariants, vertical slices, and deterministic verification triads $\langle \text{Pre}, \text{Mutation}, \text{Post} \rangle$. | [`.agents/skills/planning/SKILL.md`](.agents/skills/planning/SKILL.md) |
| **`project-analysis`** | Discovery | Understand before modifying; bifurcated flow tracing, boundary cartography, 5-tier evidence grounding, and blast radius reports. | [`.agents/skills/project-analysis/SKILL.md`](.agents/skills/project-analysis/SKILL.md) |
| **`refactoring`** | Evolution | Separation Law (never mix refactoring with behavioral changes), characterization pinning, and atomic micro-steps under green tests. | [`.agents/skills/refactoring/SKILL.md`](.agents/skills/refactoring/SKILL.md) |
| **`release-management`** | Release Governance | 8 Universal Release Invariants, semantic delta classification (SemVer/CalVer), multi-lens readiness evidence, changelogs, and launch approvals. | [`.agents/skills/release-management/SKILL.md`](.agents/skills/release-management/SKILL.md) |
| **`requirements-analysis`** | Specification | Intent-mechanism separation, 7 Universal Requirements Invariants, EARS syntax, and BDD acceptance criteria. | [`.agents/skills/requirements-analysis/SKILL.md`](.agents/skills/requirements-analysis/SKILL.md) |
| **`security-engineering`** | Defense | Universal threat modeling, 7 security invariants, archetype-aware reachability, zero ambient authority, and 0–100 confidence gating. | [`.agents/skills/security-engineering/SKILL.md`](.agents/skills/security-engineering/SKILL.md) |
| **`solution-discovery`** | Strategy | Triviality Threshold, Framework-First inspection, 5-Tier Solution Spectrum, 10-axis candidate evaluation, and anti-corruption adapters. | [`.agents/skills/solution-discovery/SKILL.md`](.agents/skills/solution-discovery/SKILL.md) |
| **`system-architecture`** | System Design | 7 Universal Architecture Invariants, constraint-first envelopes, deterministic state authority, acyclic module DAGs, and ADR discipline. | [`.agents/skills/system-architecture/SKILL.md`](.agents/skills/system-architecture/SKILL.md) |
| **`testing`** | Verification | Empirical testing over implementation, runner auto-discovery, risk-based allocation, hostile failure paths, and mutation mindset. | [`.agents/skills/testing/SKILL.md`](.agents/skills/testing/SKILL.md) |
| **`ux-engineering`** | Ergonomics | User goal defense, 8 Universal UX Invariants, HCI empirical laws, information architecture, interaction mechanics, and usability auditing. | [`.agents/skills/ux-engineering/SKILL.md`](.agents/skills/ux-engineering/SKILL.md) |

---

## Skill Architecture

Every Grimoire capability is packaged as a self-contained skill folder centered on a `SKILL.md` specification, engineered for **progressive disclosure** (keeping context tokens low until activated):

```text
agent-grimoire/
├── .agents/
│   └── skills/
│       └── <skill-name>/
│           ├── SKILL.md          # Primary entrypoint, frontmatter & core instructions (Required)
│           ├── scripts/          # Executable helper scripts and CLI automation (Optional)
│           ├── references/       # In-depth reference docs, tables & specs (Optional)
│           ├── examples/         # Reference implementations & templates (Optional)
│           └── resources/        # Static assets, prompts, or configuration files (Optional)
└── AGENTS.md                     # Workspace rulebook & protocol router
```

### Universal Anatomy of `SKILL.md`

Every `SKILL.md` implements a cohesive, standardized structure that provides high actionable signal while preserving agent context bandwidth:

```markdown
---
name: <skill-name>
description: >-
  Trigger specification: defines WHAT the capability is, precisely WHEN
  the agent must activate it, and explicit boundaries for WHEN NOT to activate (deferrals).
---

# <Skill Title>: <Core Mandate>

> **Mandate**: High-level declaration of the skill's purpose, philosophical axioms, and non-negotiables.

---

## 1 · Closed-Loop Lifecycle
Mermaid flowchart depicting the sequential or cyclic phases of the engineering process.

## 2 · Adaptive Cognitive Sizing (Mode Selection)
Explicit sizing modes (e.g., triage / standard / critical) to scale effort proportionally to risk and blast radius.

## 3 · Universal Engineering Invariants
Non-negotiable foundational invariants (e.g., boundary validation, algebraic state, or immutability).

## 4 · Progressive Disclosure (Reference Routing)
On-demand routing table pointing to deep-dive files in `references/` and `examples/`.

## 5 · Domain Engineering Protocols
Actionable guidance, checklists, decision lattices, anti-patterns, and concrete recipes.

## 6 · Verification Gate & Stopping Contract
Falsifiable pass/fail criteria and evidence required before the agent can declare completion.
```

### Skill Archetype Patterns

Skills in Agent Grimoire are organized into **7 distinct Skill Archetypes**. Rather than using generic templates, each archetype shapes the agent's cognitive posture, section hierarchy, and verification mechanics for that domain:

| Archetype | Primary Focus & Cognitive Role | Representative Grimoire Skills | Recommended Section Structure & Key Deliverable |
| :--- | :--- | :--- | :--- |
| **Workflow & Execution** | Orchestrates multi-step, mutating tasks deterministically with safety circuit breakers and rollback protection | [`planning`](.agents/skills/planning/SKILL.md), [`github-pro`](.agents/skills/github-pro/SKILL.md), [`deployment`](.agents/skills/deployment/SKILL.md), [`multi-agent-orchestration`](.agents/skills/multi-agent-orchestration/SKILL.md), [`ci-cd`](.agents/skills/ci-cd/SKILL.md), [`release-management`](.agents/skills/release-management/SKILL.md), [`infrastructure`](.agents/skills/infrastructure/SKILL.md) | 1. Pre-Flight Inspection & Discovery<br/>2. Execution DAG & Sizing Modes<br/>3. Blast Radius & Guardrails<br/>4. Verification Triads $\langle \text{Pre}, \text{Mutation}, \text{Post} \rangle$<br/>*Deliverable: Verified State Transition* |
| **Defensive Construction** | Directs invariant-driven code authoring, type-safe boundaries, resource cleanup, and schema evolution | [`code-quality`](.agents/skills/code-quality/SKILL.md), [`api-design`](.agents/skills/api-design/SKILL.md), [`data-management`](.agents/skills/data-management/SKILL.md), [`configuration-management`](.agents/skills/configuration-management/SKILL.md), [`refactoring`](.agents/skills/refactoring/SKILL.md) | 1. Boundary Defense & Ingress Validation<br/>2. Algebraic Modeling (Illegal States Unrepresentable)<br/>3. Locality of Behavior & Single Abstraction<br/>4. Causal Error Hygiene & RAII Resource Scoping<br/>*Deliverable: Clean Type-Safe Code Diff* |
| **Adversarial Audit & Certification** | Executes independent verification, multi-lens inspection, threat modeling, and completion gating | [`agent-evaluation`](.agents/skills/agent-evaluation/SKILL.md), [`code-review`](.agents/skills/code-review/SKILL.md), [`security-engineering`](.agents/skills/security-engineering/SKILL.md), [`testing`](.agents/skills/testing/SKILL.md), [`observability`](.agents/skills/observability/SKILL.md), [`performance-engineering`](.agents/skills/performance-engineering/SKILL.md), [`maintenance`](.agents/skills/maintenance/SKILL.md) | 1. Epistemic Evidence Gathering (Code as Truth)<br/>2. Multi-Lens Audit Matrix & Invariant Checklists<br/>3. Quantitative Scoring (0–100 Confidence)<br/>4. Falsifiable Verification Gates & Stopping Contract<br/>*Deliverable: Audit Ledger & Gate Certification* |
| **Strategic Discovery & Architecture** | Resolves ambiguity, maps architectural topology, evaluates build-vs-adopt decisions, and bounds systems | [`requirements-analysis`](.agents/skills/requirements-analysis/SKILL.md), [`system-architecture`](.agents/skills/system-architecture/SKILL.md), [`project-analysis`](.agents/skills/project-analysis/SKILL.md), [`solution-discovery`](.agents/skills/solution-discovery/SKILL.md), [`dependency-management`](.agents/skills/dependency-management/SKILL.md) | 1. Intent-Mechanism Separation (EARS / BDD)<br/>2. Bifurcated Flow Tracing & Cartography<br/>3. Multi-Axis Solution Evaluation Spectra<br/>4. Topological Module DAGs & State Authority<br/>*Deliverable: Living Spec, Architecture DAG, or ADR* |
| **Human Interface & Experience** | Engineers inclusive accessibility, intuitive ergonomics, and coherent visual design systems | [`accessibility`](.agents/skills/accessibility/SKILL.md), [`ux-engineering`](.agents/skills/ux-engineering/SKILL.md), [`design-philosophy`](.agents/skills/design-philosophy/SKILL.md) | 1. User Task Primacy & Sensorimotor Defense<br/>2. Deterministic Semantic Trees & LIFO Focus Stacks<br/>3. Empirical HCI Laws & Interaction Friction Budgets<br/>4. Chromatic / Typographic Rhythm & Scaled States<br/>*Deliverable: WCAG-Compliant Ergonomic UI Spec/Code* |
| **Systemic Resilience & Recovery** | Manages fault detection, causal attribution, reversible containment, and post-incident hardening | [`failure-recovery`](.agents/skills/failure-recovery/SKILL.md), [`incident-response`](.agents/skills/incident-response/SKILL.md) | 1. Fault Attribution & Scope Triage (Dev-Loop vs Live)<br/>2. Reversible Containment & State Freezing<br/>3. Causal Hypothesis Backtracking<br/>4. Convergence Proof & Blameless Post-Mortem<br/>*Deliverable: Restored State & Resilient Runbook* |
| **Knowledge Governance** | Prevents documentation rot, grounds technical writing in code evidence, and maintains living playbooks | [`docs-pro`](.agents/skills/docs-pro/SKILL.md) | 1. Diátaxis Mode Classification (Tutorial/How-To/Ref/Exp)<br/>2. Codebase Pre-Flight & Evidence Grounding<br/>3. Minimal High-Signal Authoring (Zero Drift)<br/>4. Anti-Drift Synchronization & Automated Quality Gates<br/>*Deliverable: Grounded Technical Artifact / Playbook* |

> [!TIP]
> **Progressive Disclosure Discipline**: Core `SKILL.md` files are strictly capped (typically under 200–300 lines). Deep-dive data tables, complete API schemas, toolchain matrices, and step-by-step walkthroughs are stored in `references/` or `examples/`. An agent only loads secondary files when explicitly needed, preserving precious context tokens for active problem-solving.

---

## Installation & Usage

Grimoire works at two levels: **per-project (recommended)** so skills and rules travel with your repository, or **globally** across all projects on your machine.

### Option A: Project-Level Integration (Recommended)

Copy the [`.agents/skills/`](.agents/skills/) directory and [`AGENTS.md`](AGENTS.md) into your target project's root:

```text
your-project/
├── .agents/
│   └── skills/
│       └── github-pro/
│           └── SKILL.md
├── AGENTS.md
└── ... (your project files)
```

#### Quick Setup Commands

##### macOS & Linux (Bash / Zsh)
```bash
# 1. Clone into a temporary folder
git clone --depth 1 https://github.com/gowtham2thrive/agent-grimoire.git temp-grimoire

# 2. Copy skills catalog and router into your workspace root
mkdir -p .agents/skills
cp -r temp-grimoire/.agents/skills/* .agents/skills/
cp temp-grimoire/AGENTS.md ./AGENTS.md

# 3. Clean up temporary files
rm -rf temp-grimoire
```

##### Windows (PowerShell)
```powershell
# 1. Clone into a temporary folder
git clone --depth 1 https://github.com/gowtham2thrive/agent-grimoire.git temp-grimoire

# 2. Copy skills catalog and router into your workspace root
New-Item -ItemType Directory -Force -Path ".agents\skills"
Copy-Item -Recurse -Force "temp-grimoire\.agents\skills\*" ".agents\skills\"
Copy-Item -Force "temp-grimoire\AGENTS.md" ".\AGENTS.md"

# 3. Clean up temporary files
Remove-Item -Recurse -Force temp-grimoire
```

##### Alternative: Git Submodule
If you prefer tracking Grimoire updates cleanly via Git:
```bash
git submodule add https://github.com/gowtham2thrive/agent-grimoire.git .agents/grimoire
cp .agents/grimoire/AGENTS.md ./AGENTS.md
```

> [!NOTE]
> **Why `AGENTS.md`?**
> [`AGENTS.md`](AGENTS.md) acts as the operational rulebook and protocol router. When your agent boots up, it reads `AGENTS.md`, discovering available skills and adhering to execution guardrails.

---

### Option B: Global Agent Configuration

To make Grimoire skills available across all projects without copying files into every repository:

#### Quick Setup Commands

##### macOS & Linux (Bash / Zsh)
```bash
# Clone to a permanent local directory
git clone https://github.com/gowtham2thrive/agent-grimoire.git ~/.agent-grimoire

# Link desired skills into your global Antigravity / Gemini skills registry
mkdir -p ~/.gemini/config/skills
ln -s ~/.agent-grimoire/.agents/skills/github-pro ~/.gemini/config/skills/github-pro
```

##### Windows (PowerShell)
```powershell
# Clone to a permanent local directory
git clone https://github.com/gowtham2thrive/agent-grimoire.git "$HOME\.agent-grimoire"

# Copy desired skills to your global registry
New-Item -ItemType Directory -Force -Path "$HOME\.gemini\config\skills"
Copy-Item -Recurse -Force "$HOME\.agent-grimoire\.agents\skills\github-pro" "$HOME\.gemini\config\skills\github-pro"
```

---

## Verification: Did It Work?

To confirm that your agent recognizes Grimoire skills, open your project in your agent environment and ask:

```text
"What skills and operational guidelines are active in this workspace?"
```

**Expected Result**: The agent should cite `AGENTS.md` and report active skills (e.g., `github-pro`, `docs-pro`) along with its pre-flight and safety mandates.

---

## Acknowledgments & References

Agent Grimoire draws inspiration from foundational engineering standards, software architecture patterns, and active open-source agent research across the AI community (including EARS, Diátaxis, WCAG, and ecosystem best practices).

We are committed to proper attribution and community respect. If you believe any reference, pattern, or asset requires updated attribution or clarification, please [open an issue](https://github.com/gowtham2thrive/agent-grimoire/issues) or reach out directly—we will gladly review and address it promptly.
