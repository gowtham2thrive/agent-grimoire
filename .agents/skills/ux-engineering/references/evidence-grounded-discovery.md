# Evidence-Grounded UX Discovery & JTBD Formulation

This reference defines the protocol for discovering user needs, diagnosing friction, and formulating product requirements without hallucinating fictional users or synthetic research.

---

## 1 · The Epistemic Evidence Ladder for UX

When agents design or audit user experiences, they often make the mistake of inventing fictional personas (e.g. *"Sarah, 34, busy marketing director who wants quick reports"*). **Fabricated personas are anti-patterns**—they lead to speculative, ungrounded features that solve non-existent problems.

All UX reasoning must be grounded in the **Epistemic Evidence Ladder**:

```
▲ TIER 1: EMPIRICAL REPOSITORY EVIDENCE (Highest Confidence: 90–100)
│ • Closed/Open GitHub issues reporting confusion, bug reports, feature requests
│ • Production crash logs, telemetry metrics, drop-off rates, support tickets
│ • User discussions, Discord/Slack feedback excerpts checked into the repo
│
▲ TIER 2: CODEBASE INTERACTION FLOW TRACING (High Confidence: 75–89)
│ • Number of clicks, keystrokes, or screen transitions required to complete a task
│ • Unhandled error branches, missing loading states, synchronous network blocking
│ • Brittle schema validations that reject valid human input formats
│
▲ TIER 3: COGNITIVE HEURISTIC INFERENCE (Deductive Confidence: 60–74)
│ • Application of empirical HCI laws (Fitts, Hick-Hyman, Miller, Jakob)
│ • Clearly labeled as "Agent Deductive Inference" rather than observed fact
│
▼ STRICTLY FORBIDDEN: SYNTHETIC FICTION (Confidence: 0)
  • Fabricated user interview quotes, imagined demographics, or fake survey numbers
```

---

## 2 · Mining Repository Artifacts for Real User Friction

To discover genuine user pain points in any software project, inspect these concrete repository locations before proposing UX changes:

### 2.1 Issue Trackers & PR Comments
* Search open and closed issues for friction keywords: `confusing`, `slow`, `lost data`, `where is`, `how do I`, `cannot find`, `accidental click`.
* Look for issues labeled `ux`, `ui`, `enhancement`, `good first issue`, `documentation`, or `bug`.
* Inspect PR review discussions where reviewers questioned user flows, confusing error messages, or missing empty states.

### 2.2 Execution Path & Friction Forensics
Trace the primary user journeys directly through code:
1. **Count the Critical Interaction Path**:
   * *Web/Mobile*: How many screens, form fields, and clicks does it take from landing to core value delivery?
   * *CLI*: How many distinct command invocations and flag lookups are needed to initialize and run a workflow?
2. **Identify Cognitive Roadblocks**:
   * Look for un-chunked inputs (forms with 15 flat text fields).
   * Look for modal interruptions that block reading background context.
   * Look for brittle parsers that crash on unformatted user inputs.
   * Look for synchronous blocking calls without active feedback.

---

## 3 · The Jobs-to-be-Done (JTBD) Framework

Rather than describing who a user *is* (demographics), JTBD focuses on what a user is trying to *accomplish* when they hire software.

### 3.1 The Canonical Job Story Formula
Every feature or flow must be grounded in an explicit Job Story:

$$\text{When } [\text{Trigger / Context}], \quad \text{I want to } [\text{Action / Motivation}], \quad \text{so I can } [\text{Expected Outcome}].$$

#### Concrete Multi-Platform Examples

* **Web SaaS Example**:
  > *"**When** a client disputes a line item on an invoice, **I want to** adjust the item amount and reissue the PDF directly from the invoice view, **so I can** get paid today without recalculating tax and discounts manually."*
  > *(Reveals need for: inline line-item editing, automatic tax recalculation, instant 1-click re-send with audit history).*

* **CLI Developer Tool Example**:
  > *"**When** a migration fails halfway through a production deploy, **I want to** inspect the exact failing SQL statement and roll back to the clean snapshot with a single command, **so I can** restore service immediately without corrupting production tables."*
  > *(Reveals need for: clear error to `stderr`, exact statement printout, dedicated `rollback` command with confirmation).*

* **AI Agent Conversational Example**:
  > *"**When** an autonomous agent proposes a complex multi-file refactor, **I want to** preview the diff summary and selectively approve or reject individual files, **so I can** maintain control over code quality without rejecting the entire generation."*
  > *(Reveals need for: file-by-file approval affordance, side-by-side diff preview, incremental commit).*

### 3.2 The 3 Dimensions of a Job
Every complete Job Story encompasses three dimensions:
1. **Functional Job**: The practical, core task to be completed (e.g. transfer funds, compile code, generate report).
2. **Emotional Job**: How the user wants to feel while doing it (e.g. confident, in control, relieved, secure).
3. **Social Job**: How the user wants to be perceived by their peers or stakeholders (e.g. competent, professional, reliable).

---

## 4 · Problem Statement Validation Checklist

Before writing code or designing screens, validate the problem statement against these 4 tests:

1. **The Evidence Test**: Can I point to an existing issue, code trace, or user complaint that proves this friction exists? (If no, classify as *Tier 3 Hypothesis* and seek validation).
2. **The Outcome Test**: Does solving this problem save the user time, prevent data loss, or eliminate cognitive confusion?
3. **The Simplicity Test**: Is the proposed solution the shortest path between user intent and outcome, or does it add new layers of configuration?
4. **The Non-Disruption Test**: Does this fix respect Jakob's Law by aligning with established user mental models?
