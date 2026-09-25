# Evidence Grounding & Epistemic Verification

> **Mandate**: *A plan constructed on unverified assumptions is an hallucinated fiction.* Every task in an implementation plan must be anchored in verified repository facts—actual file paths, resolved AST symbols, declared dependency versions, and active test harness commands. Never guess what the codebase can answer deterministically.

---

## 1 · The 4-Tier Evidence Verification Matrix

Before scheduling any task that modifies or integrates with existing code, the agent must certify the task's premises against the 4 tiers of empirical evidence:

| Evidence Tier | Verification Target | Permitted Discovery Tool | Prohibited Speculation |
| :--- | :--- | :--- | :--- |
| **Tier 1: Path & File Existence** | Verify whether target files, directories, and entrypoints actually exist on disk. | `view_file`, directory listing, filesystem search (`Get-ChildItem`, `find`, `ls`). | Guessing file paths based on standard conventions without verifying repository reality. |
| **Tier 2: Symbol & AST Integrity** | Verify exact class names, function signatures, exported types, and module imports. | Code exploration tools, language server queries, regex / AST symbol inspection. | Assuming function parameters, return types, or method names without reading declaration. |
| **Tier 3: Dependency & Toolchain Baseline** | Verify installed packages, framework versions, language toolchains, and build scripts. | `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, `build.gradle`, CLI version checks. | Assuming a library, CLI tool, or compiler feature is available without manifest confirmation. |
| **Tier 4: Operational Test Baseline** | Verify that existing tests pass green *before* any plan mutations begin. | Test runner execution (`npm test`, `cargo test`, `pytest`, `go test`). | Assuming the codebase is healthy; planning new features on top of a broken test suite. |

---

## 2 · The Intent-Plan Separation Law: "What" vs "How"

A common planning pathology is conflating the settled requirement with the implementation plan:

```mermaid
flowchart TD
    subgraph Requirements Space ["Requirements Space (Settled 'WHAT' & 'WHY')"]
        R1["User Goal: 'Allow users to authenticate via OAuth2'"]
        R2["Acceptance: 'Given valid Google token, return JWT session'"]
        R3["NFR: 'Auth handshake latency < 250ms p95'"]
    end
    subgraph Planning Space ["Planning Space (Verifiable 'HOW' & 'IN WHAT ORDER')"]
        P1["Task 1: Install & lock oauth2 client library in package.json"]
        P2["Task 2: Define OAuthUser DTO and TokenSession interface"]
        P3["Task 3: Implement GoogleAuthProvider adapter in src/auth/providers/"]
        P4["Task 4: Add /api/auth/google/callback endpoint with state validation"]
    end
    Requirements Space ==> Planning Space
```

### The Clean-Room Separation Rules
1. **Requirements are Immutable Inputs**: The plan does not reinvent, renegotiate, or alter settled requirements from `requirements-analysis`. If a requirement is discovered to be physically impossible during planning, trigger a `REQ-DELTA` feedback loop.
2. **Plans are Mutable Execution Strategies**: The plan represents the most efficient, verifiable path through the codebase to fulfill the settled requirements.
3. **No Implementation Code in the Plan**: Plans define contracts, file targets, dependencies, and verification commands. Wholesale code implementations belong in the working tree, not inside the planning markdown.

---

## 3 · Cartography Pre-Flight Checklist

Before generating the task DAG, execute this checklist:

- [ ] **Working Directory & Git Status Verified**: Active branch is confirmed, working tree is clean or has known uncommitted state.
- [ ] **Target Paths Confirmed**: Every existing file scheduled for modification has been viewed (`view_file`) or its existence verified.
- [ ] **Parent Directories Mapped**: For new files, the destination directory structure exists or is scheduled for explicit creation.
- [ ] **Test Harness Confirmed**: The exact test command that will verify each task has been run or verified against configuration manifests.
- [ ] **Baseline Health Recorded**: The current test suite passes green or known failing tests are documented in the plan's pre-existing conditions register.

---

## 4 · Evidence Citation Syntax

To ensure auditable grounding, tasks in `standard` and `epic` plans must cite their supporting evidence:

```markdown
### Task 002: Implement Token Validation Middleware
- **Evidence**:
  - Target file: `src/middleware/auth.ts` (verified via `view_file` lines 1–45)
  - Interface: `SessionToken` exported in `src/types/auth.ts:14`
  - Existing middleware pattern: `src/middleware/logger.ts`
- **Pre-check**: `npm test -- src/middleware/auth.test.ts` (verifies existing baseline)
- **Mutation Scope**: Add token extraction and signature verification in `src/middleware/auth.ts`.
- **Post-verification**: `npm test -- src/middleware/auth.test.ts` (falsifiable assertion)
```
