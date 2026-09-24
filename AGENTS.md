# Agent Grimoire: Operational Guidelines & Protocol Router

This file governs agent behavior within this workspace. It acts as the primary rulebook and router for loading skills from `.agents/skills/`.

---

## 1. Skill Discovery & Invocation Mandate

Whenever a task relates to a registered skill in `.agents/skills/`, you **MUST** read the corresponding `SKILL.md` before executing actions:

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
