# Agent Grimoire: Operational Guidelines & Protocol Router

This file governs agent behavior within this workspace. It acts as the primary rulebook and router for loading skills from `.agents/skills/`.

---

## 1. Skill Discovery & Invocation Mandate

Whenever a task relates to a registered skill in `.agents/skills/`, you **MUST** read the corresponding `SKILL.md` before executing actions:

- **Code Quality & Defensive Engineering**: Load [`.agents/skills/code-quality/SKILL.md`](.agents/skills/code-quality/SKILL.md). Enforces boundary validation, making illegal states unrepresentable, structured causal error hygiene, deterministic resource management, concurrency safety, and anti-overengineering locality of behavior without limiting architectural creativity.
- **Independent Code Review**: Load [`.agents/skills/code-review/SKILL.md`](.agents/skills/code-review/SKILL.md). Enforces a two-stage review gate (specification compliance first, then code quality), specialized multi-lens auditing (correctness, security, performance, regression), 0–100 confidence scoring, and ruthless filtering of cosmetic noise.
- **Empirical Testing & Verification**: Load [`.agents/skills/testing/SKILL.md`](.agents/skills/testing/SKILL.md). Enforces testing behavior over implementation, risk-based allocation, strong semantic assertions, hostile failure paths, mutation mindset, runner auto-discovery, and hermetic flakiness elimination across any runtime or brownfield project.
- **Behavior-Preserving Refactoring**: Load [`.agents/skills/refactoring/SKILL.md`](.agents/skills/refactoring/SKILL.md). Enforces the Separation Law (never mix refactoring with behavioral changes), characterization pinning before mutation, atomic micro-steps under an active test harness, and strict behavioral invariance verification.
- **Agent Evaluation & Task Verification Gate**: Load [`.agents/skills/agent-evaluation/SKILL.md`](.agents/skills/agent-evaluation/SKILL.md). Enforces the 5-Point Verification Gate (requirement traceability, static types, test suite execution, diff sanity, and negative validation) before declaring any task complete.
- **Project Analysis & Codebase Exploration**: Load [`.agents/skills/project-analysis/SKILL.md`](.agents/skills/project-analysis/SKILL.md). Enforces understand-before-modifying discipline, bifurcated vertical flow tracing, 5-tier epistemic evidence grounding, boundary cartography, risk hotspot forensics, and reusable context distillation across any language, framework, or tooling.
- **Git & GitHub Workflows**: Load [`.agents/skills/github-pro/SKILL.md`](.agents/skills/github-pro/SKILL.md). Enforces intent disambiguation, pre-flight checks, release detection, and hygiene before running any Git/GitHub commands.
- **Documentation & Technical Writing**: Load [`.agents/skills/docs-pro/SKILL.md`](.agents/skills/docs-pro/SKILL.md). Enforces convention discovery, evidence-backed grounding, Diátaxis mode discipline, anti-drift synchronization, and progressive disclosure for any project documentation.
- **Multi-Agent Orchestration**: Load [`.agents/skills/multi-agent-orchestration/SKILL.md`](.agents/skills/multi-agent-orchestration/SKILL.md). Enforces topology selection, foundation-first discipline, isolated worktree dispatch, bounded supervision with recovery ladders, and holistic integration verification.
- **Data Safety**: Always verify before running commands that could result in irreversible data loss.
- **Reporting**: Keep routine responses concise; provide detailed structured breakdowns for milestones or architectural changes.

---

## 2. General Principles

1. **Pre-flight Inspection First**: Inspect environment, working directory, git state, and configuration before taking action.
2. **Minimal Safe Action**: Never execute speculative, destructive, or bulk changes without verification.
3. **Preserve Integrity**: Do not remove existing comments, docstrings, or tests unless specifically instructed.
4. **Post-Action Verification**: Always verify that operations succeeded via status inspection or test runs rather than assuming success.
