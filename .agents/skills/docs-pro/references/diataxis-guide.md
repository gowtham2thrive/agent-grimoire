# Diátaxis Documentation Mode Guide

Two questions classify any document: does the content serve **action** (doing) or **understanding** (thinking), and does it serve **learning** (acquisition) or **work** (application)?

---

## 1. Tutorial (Action + Learning)

**You are the instructor.** The learner's success is your responsibility, not theirs.

- Open with what the learner will **build**, not what they will "learn".
- Provide a minimal, linear path. Never ask the learner to make architectural choices (don't ask them to choose between databases or frameworks).
- Every step must produce **immediate visible output**: a terminal message, a rendered page, a passing test, a log line.
- Tell them what they should see: the expected output, the prompt change, the confirmation message.
- Cut explanation to one clause and a link. Teaching pauses break the lesson.
- Write as "we", in commands: *"First, create the config file. Now, run the server."*

---

## 2. How-To Guide (Action + Work)

**You are answering a focused question for a competent practitioner.**

- Solve a problem a person has, not an operation the machine can perform.
- Assume general competence. Skip elementary setup and teaching pauses.
- Structure as sequential steps directly advancing toward the goal.
- Allow conditional forks and judgment: *"If you need TLS, set `ssl_mode=require`; otherwise omit the flag."*
- Name the guide by the task: *"How to configure read replicas"*, not *"Read Replica Configuration"*.
- No digressions, no background, no completeness for its own sake. Link those instead.

---

## 3. Reference (Understanding + Work)

**You are a dictionary.** Describe reality with exact precision.

- Mirror the structure of the thing being documented (API routes match route files, CLI flags match the command tree, config keys match the schema).
- State facts, options, types, default values, boundaries, and error codes without opinion, hedging, or persuasion.
- Keep sentences concise, declarative, and free of narrative fluff.
- Put material where readers expect it. Generate from code where possible, so it stays true.
- Reference is the mode most likely to drift. Keep synchronized with code definitions and typed interfaces.

---

## 4. Explanation (Understanding + Learning)

**You are a senior engineer explaining the system over coffee.**

- One bounded topic, readable away from the product. Each title should tolerate an implicit "About..." in front.
- Anchor on a real *why* question: why is this designed this way, why was this alternative rejected, why does this constraint exist?
- Give context: design decisions, history, constraints, alternatives considered.
- Opinion is allowed here and nowhere else.
- Weigh trade-offs. Connect disparate subsystems into a cohesive mental model.
- Architectural Decision Records (ADRs) are a specialized form of Explanation.

---

## Mode Mixing: The Cardinal Sin

Do not mix modes. No reference tables inside a tutorial. No tutorial hand-holding inside reference. No arguing inside a how-to. When you find yourself shifting mode mid-section, split and link instead. A document that tries to be everything teaches nothing and references nothing reliably.
