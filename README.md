<div align="center">

  <img src="logo.svg" alt="Agent Grimoire Logo" width="200" />
  <h1 align="center" style="border-bottom: none">Agent Grimoire</h1>

  <p>
    <strong>An evolving library of reusable AI agent skills, workflows, and protocols for building and evolving software autonomously.</strong>
  </p>

  <p>
    <a href="https://github.com/gowtham2thrive/skills"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square" alt="License: MIT" /></a>
    <a href="#core-tenets"><img src="https://img.shields.io/badge/Agents-Agent--Agnostic-38bdf8.svg?style=flat-square" alt="Agents: Agent-Agnostic" /></a>
  </p>

  <p>
    <a href="#overview"><strong>Overview</strong></a> •
    <a href="#built-in-skills-catalog"><strong>Skill Catalog</strong></a> •
    <a href="#skill-architecture"><strong>Skill Architecture</strong></a> •
    <a href="#installation--usage"><strong>Installation & Usage</strong></a> •
    <a href="#verification-did-it-work"><strong>Verification</strong></a>
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

Agent Grimoire provides an integrated suite of battle-tested capabilities covering the entire software development lifecycle:

| Skill | Category | Mandate & Purpose | Entrypoint |
| :--- | :--- | :--- | :--- |
| **`code-quality`** | Construction | Boundary validation, algebraic state modeling, structured causal error hygiene, and concurrency safety without limiting creativity. | [`.agents/skills/code-quality/SKILL.md`](.agents/skills/code-quality/SKILL.md) |
| **`code-review`** | Audit | Two-stage review gate (spec compliance first, then code quality), 4 specialized lenses, 0–100 confidence scoring, and noise suppression. | [`.agents/skills/code-review/SKILL.md`](.agents/skills/code-review/SKILL.md) |
| **`testing`** | Verification | Empirical testing over implementation, runner auto-discovery, risk-based allocation, hostile failure paths, and mutation mindset. | [`.agents/skills/testing/SKILL.md`](.agents/skills/testing/SKILL.md) |
| **`refactoring`** | Evolution | Separation Law (never mix refactoring with behavioral changes), characterization pinning, and atomic micro-steps. | [`.agents/skills/refactoring/SKILL.md`](.agents/skills/refactoring/SKILL.md) |
| **`agent-evaluation`** | Certification | Universal 5-Point Verification Gate (traceability, static types, test suite, diff sanity, negative validation) before declaring completion. | [`.agents/skills/agent-evaluation/SKILL.md`](.agents/skills/agent-evaluation/SKILL.md) |
| **`project-analysis`** | Discovery | Understand before modifying; bifurcated flow tracing, boundary cartography, 5-tier evidence grounding, and blast radius reports. | [`.agents/skills/project-analysis/SKILL.md`](.agents/skills/project-analysis/SKILL.md) |
| **`multi-agent-orchestration`** | Coordination | Parallel and phased sub-agent decomposition, isolated worktrees, durable task contracts, and holistic merge verification. | [`.agents/skills/multi-agent-orchestration/SKILL.md`](.agents/skills/multi-agent-orchestration/SKILL.md) |
| **`github-pro`** | Mechanics | Git and GitHub workflow automation, intent disambiguation, pre-flight safety checks, and release milestone management. | [`.agents/skills/github-pro/SKILL.md`](.agents/skills/github-pro/SKILL.md) |
| **`docs-pro`** | Knowledge | Architecture playbooks, ADRs, READMEs, Diátaxis framework, and anti-drift synchronization. | [`.agents/skills/docs-pro/SKILL.md`](.agents/skills/docs-pro/SKILL.md) |

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

Every `SKILL.md` starts with standard YAML frontmatter followed by a modular structure that fits **any skill archetype**—from operational workflows to API reference guides and diagnostic audits:

```markdown
---
name: <skill-name>
description: >-
  Trigger specification: defines WHAT the capability is and precisely WHEN
  the agent must load it (key phrases, tasks, file extensions, or tool triggers).
---

# <Skill Title>: <Core Mandate>

> **Mandate / Intent**: High-level declaration of the skill's purpose, boundaries, and non-negotiables.

---

## 1 · Context & Activation Trigger
Scope, prerequisites, environment conditions, or target files that dictate activation.

## 2 · Core Instructions & Knowledge
The actionable content tailored to the skill type (execution steps, API patterns, or audit rules).

## 3 · Guardrails & Anti-Patterns
Critical constraints, edge cases, disallowed actions, or safety verifications.

## 4 · Verification & Output Standards
How to validate correctness (tests, diff inspection, lint checks) and format proportional responses.
```

### Skill Archetype Patterns

Depending on whether a skill drives **actions**, imparts **domain knowledge**, or conducts **audits**, the internal sections adapt naturally:

| Archetype | Primary Purpose | Example Skills | Recommended Section Structure |
| :--- | :--- | :---: | :--- |
| **Workflow / Protocol** | Executes multi-step, mutating tasks safely | `github-pro`, `db-migrate`, `deploy` | 1. Pre-Flight Inspection<br/>2. Execution Flow / Decision Matrix<br/>3. Safety Guardrails<br/>4. Post-Verification |
| **Knowledge / Reference** | Guides syntax, APIs, design patterns & rules | `docs-pro`, `gemini-api`, `modern-web` | 1. Overview & Setup<br/>2. Recommended Patterns & Syntax<br/>3. Code Examples<br/>4. Anti-Patterns & Gotchas |
| **Audit / Diagnostic** | Evaluates code quality, security, or performance | `a11y-debugging`, `security-audit` | 1. Inspection Scope<br/>2. Diagnostic Checklist & Heuristics<br/>3. Severity Matrix<br/>4. Remediation Recipes |

> [!TIP]
> **Progressive Disclosure**: Keep `SKILL.md` lean and actionable (under 200–300 lines). Offload large data tables, voluminous API specs, or lengthy command examples into `references/` or `examples/`. The agent will only traverse into secondary files when the task demands it, saving precious context window tokens.

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
git clone --depth 1 https://github.com/gowtham2thrive/skills.git temp-grimoire

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
git clone --depth 1 https://github.com/gowtham2thrive/skills.git temp-grimoire

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
git submodule add https://github.com/gowtham2thrive/skills.git .agents/grimoire
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
git clone https://github.com/gowtham2thrive/skills.git ~/.agent-grimoire

# Link desired skills into your global Antigravity / Gemini skills registry
mkdir -p ~/.gemini/config/skills
ln -s ~/.agent-grimoire/.agents/skills/github-pro ~/.gemini/config/skills/github-pro
```

##### Windows (PowerShell)
```powershell
# Clone to a permanent local directory
git clone https://github.com/gowtham2thrive/skills.git "$HOME\.agent-grimoire"

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
