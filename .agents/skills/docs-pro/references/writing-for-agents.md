# Writing for Autonomous AI Agents

Guidelines for authoring documents consumed by AI coding agents — such as `AGENTS.md`, `CLAUDE.md`, `COPILOT.md`, skills (`SKILL.md`), and modular project guidelines. The packaging differs; the writing principles do not.

---

## 1. Context Pointers & Triggers

A **context pointer** is a reference held in the agent's context that identifies out-of-context material and encodes the conditions for reaching it:
- A skill's YAML `description` field is a pointer.
- A bullet point in `AGENTS.md` routing a domain to a skill file is a pointer.
- A comment in code pointing to a design doc is a pointer.

### Crafting High-Precision Pointers
- **Front-Load Trigger Keywords**: The pointer fires based on its opening words. Place decisive nouns and verbs at the start.
- **One Trigger per Execution Branch**: A branch is a distinct case the document handles. Synonyms that rename a single branch are one branch written twice — collapse them.
- **Sharpen Rather than Inline**: If an agent fails to reach a document, the pointer's wording is the defect. Sharpen it before bloating root files with inlined content.
- **Cut Identity the Body Carries**: Do not repeat in the pointer what the body already says clearly on read.

---

## 2. Managing the Two Budgets

Every document and pointer spends one of two budgets:

1. **Context Load (Tokens & Attention)**:
   - The cost of always-loaded material on the agent's context window: `AGENTS.md` lines, skill descriptions, anything present every turn.
   - Every unnecessary word degrades model attention and wastes budget whether or not it fires.
   - Ruthlessly prune root instruction files. Material reached through a pointer escapes context load at the price of one pointer line.

2. **Cognitive Load (Human Navigation)**:
   - The human's ability to understand what documents exist and when to reach for each.
   - This is not a cost to minimize — it is the price of human agency. Spend it where human judgment matters; remove it where it does not.
   - Group files cleanly with self-explanatory filenames and directory structure.

---

## 3. Information Hierarchy & Progressive Disclosure

Organize content into three rungs:

1. **In-File Steps (Rung 1)**: The core sequential actions the agent performs. Keep these immediate and visible — this is the primary tier.
2. **In-File Reference (Rung 2)**: Essential guardrails, rules, and definitions consulted on demand during execution. Often a legitimately flat peer-set (every rule at one rung), which is fine.
3. **Disclosed Reference (Rung 3)**: In-depth specifications, verbose tables, full templates, and edge-case catalogs. Pushed into `references/`, `examples/`, or `docs/` and reached via a relative link only when the task demands it.

**Progressive disclosure** is the deliberate move down the ladder — not primarily a token optimization, but how the hierarchy is protected. Inline what every branch needs; push behind a pointer what only some branches reach.

---

## 4. Co-Location vs. Fragmentation

- **Co-Location**: Keep a concept's definition, rules, constraints, failure modes, and caveats grouped under a single heading. When the agent reads that section, it gains complete localized context.
- **Anti-Pattern (Fragmentation)**: Scattering a concept's definition in one section, its security warnings in another, and its parameter table in a third forces readers to assemble jigsaw pieces — multiplying hallucination risk for agents and confusion for humans.

The test: the document should read like documentation written *for* the agent. Grouped material reads that way; scattered material does not.
