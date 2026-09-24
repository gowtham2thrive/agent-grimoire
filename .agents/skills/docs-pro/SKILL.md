---
name: docs-pro
description: >-
  Professional technical documentation and architecture knowledge protocol.
  Use when authoring, auditing, or refactoring READMEs, architecture maps,
  ADRs, API references, code contracts, guides, changelogs, release notes,
  runbooks, or agent instructions (AGENTS.md/SKILL.md). Enforces convention
  discovery, evidence-backed grounding, Diátaxis mode discipline, anti-drift
  synchronization, and progressive disclosure. For isolated single-line
  docstring or comment fixes, apply ecosystem conventions directly without
  loading this full skill.
---

# Docs Pro: Autonomous Technical Documentation Protocol

> **Mandate**: Ground every document in verifiable code evidence, discover and preserve repository documentation conventions, classify intent via Diátaxis, design for dual human/agent readability, prevent documentation drift, and never generate speculative, rotting, or boilerplate content.

---

## 1 · Adaptive Documentation Lifecycle

Never draft or update documentation from assumptions. Execute every documentation task through this adaptive pipeline:

1. **User Intent**: Decode the actual objective—creating, auditing, restructuring, or synchronizing documentation—rather than matching keywords.
2. **Convention Discovery**: Inspect existing documentation patterns before imposing structure:
   - Detect the documentation format (`*.md`, `*.rst`, `*.adoc`, `*.mdx`, wiki pages) and authoring conventions already in use.
   - Inspect existing docs directory structure (`docs/`, `documentation/`, `wiki/`, inline `README.md` files in subdirectories).
   - Review recent documentation commits (`git log --diff-filter=M -- '*.md' -n 5 --oneline`) to understand the project's prevailing voice, heading style, and update cadence.
   - Detect project-specific tooling and commands from manifests (`package.json`, `Cargo.toml`, `pyproject.toml`, `Makefile`, `Taskfile.yml`, `justfile`, `CMakeLists.txt`, `build.gradle`, `pom.xml`, `.csproj`) rather than guessing generic commands.
   - **Match, don't override**: If the project uses reStructuredText, write reStructuredText. If it uses AsciiDoc, write AsciiDoc. If it has an established tone or heading convention, preserve it.
3. **Codebase Pre-Flight**: Inspect target source files, exports, public interfaces, schemas, dependencies, and test suites before writing a single word.
4. **Mode Classification**: Classify the document into exactly one Diátaxis mode (see Section 2).
5. **Minimal High-Signal Writing**: Write with maximum information density. Cut every word that does no work.
6. **Post-Draft Verification**: Run the quality gate (see Section 7) before declaring work complete.

---

## 2 · Mode Selection (Diátaxis)

Before drafting, classify the document into exactly one primary mode. Two questions decide it: does the content serve *action* or *understanding*, and does it serve *learning* or *work*?

| Mode | Reader Need | Tone | Key Discipline |
| :--- | :--- | :--- | :--- |
| **Tutorial** | Learning by doing | Patient, sequential, encouraging | Every step produces visible feedback. Cut theory to links. Never ask the learner to make architectural choices. |
| **How-To Guide** | Solving a real problem | Goal-driven, direct, recipe-style | Assume competence. Action-only. Allow conditional forks. Skip background and general theory. |
| **Reference** | Information lookup | Dry, authoritative, complete | Mirror actual code structure. State facts, types, defaults, limits, and errors without persuasion or opinion. |
| **Explanation** | Understanding & mental model | Analytical, evaluative, context-rich | Explain the *why*: design trade-offs, invariants, failure modes, rejected alternatives, and ADR history. Opinion is permitted here and nowhere else. |

**Never mix modes in one section.** Do not embed reference tables inside tutorials or tutorial hand-holding inside reference docs. Split and link instead. *(Deep-dive: [`references/diataxis-guide.md`](references/diataxis-guide.md).)*

---

## 3 · Evidence-Backed Grounding (Code as Source of Truth)

Documentation must reflect actual system reality, not aspirations or obsolete assumptions:

1. **The Codebase is the Word List**: Use exact identifiers, file paths, configuration keys, CLI flags, and error names from the actual source. Never invent synonyms or descriptions when the real symbol exists.
2. **Distinguish Current vs. Target State**: Label existing implementations vs. proposed architectures explicitly. Never document planned behavior as working code.
3. **Trace Execution Paths**: Follow at least one end-to-end flow through the codebase to verify architectural claims before documenting them.
4. **Identify Change Locations**: For architecture and module docs, specify *where to look* and *what files to touch* when modifying behavior (e.g., *"To add a new API route: create handler in `src/handlers/`, register in `src/router.ts`"*).
5. **Document Invariants & Failure Modes**: Code explains *how*; documentation must capture *invariants* (what must never break), *failure behaviors* (what happens when a dependency is unavailable), and *trust/persistence boundaries*. *(System mapping playbook: [`references/architecture-playbook.md`](references/architecture-playbook.md).)*

---

## 4 · Universal Technical Writing Craft

These principles apply regardless of language, framework, or documentation format:

- **Signal-Over-Slop**:
  - Cut every word that does no work (*"in order to"* → *"to"*; delete *"it is worth noting that"*, *"seamlessly"*, *"leverage"*).
  - Use the short, everyday word: *"use"* over *"utilize"*, *"help"* over *"facilitate"*, *"run"* over *"execute"*.
  - When a rule makes a sentence worse, fix the sentence another way. The rules serve the reader.
- **Vary the Rhythm**:
  - Mix short sentences that land a point with longer ones that carry a fact alongside its condition or consequence.
  - Be specific over sterile: *"renaming this column deadlocks the migration"*, not *"schema modifications may introduce operational challenges"*.
- **Evidence-Backed Snippets**:
  - Every command, configuration block, and code sample must be syntax-highlighted, verifiable against the project's actual toolchain, and runnable in the target environment.
- **Visuals & Diagrams**:
  - Use diagrams for multi-component topologies, state machines, data flows, and sequence lifecycles. Quote node labels containing special characters to prevent parser errors.
- **Secret & PII Hygiene**: Never include live tokens, private keys, passwords, or personal data. Use RFC placeholders (`example.com`, `<YOUR_API_KEY>`).

---

## 5 · Dual-Audience Design (Humans & AI Agents)

Modern documentation is consumed by human developers and autonomous AI agents alike. Structure content for both:

- **Context Pointers**: A pointer's *wording* decides when an agent reaches the material. Front-load trigger keywords. If an agent fails to find a document, sharpen the pointer before inlining content. *(Guide: [`references/writing-for-agents.md`](references/writing-for-agents.md).)*
- **Information Hierarchy**:
  1. *In-file steps*: Core sequential actions the agent performs.
  2. *In-file reference*: Essential guardrails, rules, and definitions consulted on demand.
  3. *Disclosed reference*: Deep templates, verbose tables, and exhaustive specifications pushed into `references/` behind context pointers to protect context budgets.
- **Co-Location**: Keep a concept's definition, rules, constraints, and failure caveats grouped under a single heading. Scattering fragments across multiple sections forces readers (human or agent) to assemble jigsaw pieces, multiplying error risk.

---

## 6 · Documentation Drift & Synchronization

Documentation rots the moment code changes without doc updates. Treat docs as first-class build artifacts:

- **Same-Changeset Updates**: When modifying code, review and update affected docstrings, README sections, and API specs within the same commit or pull request.
- **Deprecation & Supersession**: When replacing architecture patterns or tools, mark legacy ADRs and docs as `Superseded by ADR-XXX` or `Deprecated`. Never silently delete historical context unless instructed.
- **Contract Parity**: Public API documentation, typed schemas, and code docstrings must stay 100% in sync with runtime signatures and behavior. *(Contract patterns: [`references/contracts-and-docstrings.md`](references/contracts-and-docstrings.md).)*

---

## 7 · Verification & Quality Gate

Before declaring documentation work complete:

- [ ] **Executable Verification**: All CLI commands, build scripts, and code snippets work as written in the target environment.
- [ ] **Link & Path Integrity**: Every relative link, code symbol reference, and anchor target resolves to an existing file or section.
- [ ] **Code-Grounding Check**: No hallucinated parameters, phantom endpoints, or unsupported options.
- [ ] **Convention Alignment**: Output format, heading style, and tone match the repository's established documentation patterns.
- [ ] **Fluff Audit**: Zero marketing filler, conversational padding, or self-evident docstrings.
- [ ] **Context Load Optimization**: Main files remain lean; detailed tables and reference catalogs are safely disclosed into companion files.
