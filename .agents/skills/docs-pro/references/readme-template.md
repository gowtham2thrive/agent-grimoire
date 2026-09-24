# Project README Layout Guide

A README is the project's front door. It must deliver immediate time-to-value — a reader should understand what the project does, how to run it, and where to look within 2 minutes.

This guide provides a structural skeleton. **Adapt it to the project's established conventions** — if the project already has a README format, heading style, or badge convention, match that rather than imposing this template wholesale.

---

## Structural Skeleton

The following sections appear in order of reader priority. Include what applies; skip what doesn't:

### 1. Hero Block
- Project name, one-sentence value statement (what it does and the primary problem it solves).
- Relevant badges (CI status, license, version, coverage) — only if the project uses them.

### 2. Overview
- 2–3 paragraphs maximum. Describe key capabilities, design philosophy, and differentiating features.
- Use bullet points for feature highlights with tangible benefits.

### 3. Quickstart
- **Prerequisites**: List runtimes, tools, and minimum versions.
- **Installation**: Clone, install dependencies, and configure — using the project's actual package manager and commands.
- **Basic Usage**: A minimal, runnable command or snippet that demonstrates the core capability.

### 4. Architecture & Workflow (if applicable)
- A high-level diagram showing major components, data flow, or module boundaries.
- Brief annotation of each component's responsibility.

### 5. Configuration
- Table of environment variables, CLI flags, or configuration keys with types, defaults, and descriptions.

### 6. Development & Testing
- Commands to run tests, linting, and builds — using the project's actual tooling (detected from manifests, not assumed).

### 7. Contributing & License
- Link to `CONTRIBUTING.md` if it exists.
- License type and link.
