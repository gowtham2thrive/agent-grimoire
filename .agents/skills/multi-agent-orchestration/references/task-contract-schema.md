# Task Contract Schema

Deep reference for Section 4.1 of the main [`SKILL.md`](../SKILL.md). Defines the structure of coordination contracts used to brief workers and track execution state.

---

## 1. Coordination Context (`context.json`)

The machine-readable run configuration. Created by the orchestrator before dispatch; consumed by workers (via rendered prompts) and the supervision loop (for validation and restart decisions).

### Schema

```json
{
  "project": "<one-line description of the user's task>",
  "created_at": "<ISO 8601 timestamp>",

  "execution_topology": {
    "mode": "<direct | single_worker | parallel | phased | review_gate>",
    "reason": "<why this topology is the right amount of orchestration>",
    "dependency_notes": ["<shared foundations, sequencing constraints, or fan-out dependencies>"]
  },

  "foundation": {
    "status": "<not_required | completed_committed | owned_by_worker>",
    "paths": ["<shared foundation file paths or globs; empty when not_required>"],
    "commit": "<git commit SHA when status is completed_committed>",
    "owner": "<task name when status is owned_by_worker>"
  },

  "chat_context": {
    "preferences": ["<user coding style preferences, e.g., 'Use explicit typing'>"],
    "architecture": ["<architectural patterns in use, e.g., 'MVVM with repository pattern'>"],
    "naming_conventions": ["<naming rules, e.g., 'camelCase for variables, PascalCase for types'>"],
    "gotchas": ["<environment-specific warnings, e.g., 'Using Node 18, not 22'>"]
  },

  "requirements": ["<compact requirement summaries>"],
  "constraints": ["<compact constraint summaries>"],

  "tasks": {
    "<agent-name>": {
      "description": "<what this worker builds, in one sentence>",
      "read_first": ["<paths to interface definitions and architecture docs>"],
      "allowed_paths": ["<file paths and globs the worker MAY create or modify>"],
      "forbidden_paths": ["<file paths and globs the worker MUST NOT touch>"],
      "validation_command": ["<JSON array of command segments, e.g., ['npm', 'test', '--', 'src/auth']>"],
      "timeout_mins": "<max wall-clock minutes before liveness kill>",
      "progress_timeout_mins": "<max minutes without commits or diffs before progress timeout>",
      "definition_of_done": "<exact behavioral assertions and test expectations>"
    }
  }
}
```

### Field Rules

| Field | Required | Notes |
| :--- | :---: | :--- |
| `project` | ✅ | One sentence. Appears in worker prompts and synthesis reports. |
| `execution_topology.mode` | ✅ | Must match a mode from Section 2 of SKILL.md. |
| `execution_topology.reason` | ✅ | Justifies the topology choice. Reviewed during optional plan review. |
| `foundation.status` | ✅ | `not_required` for pure parallel; `completed_committed` for phased (with commit SHA). |
| `tasks.<name>.allowed_paths` | ✅ | Glob patterns. Must be disjoint across all workers in `parallel` and `phased` modes. |
| `tasks.<name>.forbidden_paths` | ✅ | Must include `coord/`, other workers' `allowed_paths`, and shared configs (unless foundation). |
| `tasks.<name>.validation_command` | Recommended | JSON array form preferred (no shell expansion). `null` disables local validation. |
| `tasks.<name>.definition_of_done` | ✅ | Concrete success criteria the integration gate uses to verify the deliverable. |

---

## 2. Decisions Document (`DECISIONS.md`)

A human-readable Markdown file serving as the durable source of truth for:

- Architectural decisions made during decomposition.
- Shared API contracts and data models.
- File ownership boundaries.
- Interface agreements between workers.

**Ownership**: The orchestrator writes and updates `DECISIONS.md`. Workers read it but never modify it. If a worker discovers that a decision needs revision, it reports the issue in its deliverable — the orchestrator decides whether to update.

### Template

```markdown
# Orchestration Decisions

## Project
[One-line project description]

## Architectural Decisions

### AD-1: [Decision Title]
- **Context**: [Why this decision was needed]
- **Decision**: [What was decided]
- **Consequences**: [Trade-offs accepted]

## Shared Contracts

### API: [Endpoint/Interface Name]
- **Path**: `src/types/user.ts`
- **Consumed by**: worker-auth, worker-profile
- **Contract**: [Interface definition or link to file]

## File Ownership

| Worker | Owns | Must Not Touch |
|--------|------|---------------|
| auth-api | `src/auth/**`, `test/auth/**` | `src/profile/**`, `package.json` |
| profile-ui | `src/components/profile/**` | `src/auth/**`, `package.json` |
```

---

## 3. Worker Brief (Prompt Payload)

The rendered prompt delivered to each worker. Must be **fully self-contained** — workers have zero access to the orchestrator's chat history.

### Required Sections

1. **Objective**: What to build.
2. **Context**: Architecture, naming conventions, and gotchas from `chat_context`.
3. **Scope**: `allowed_paths` (what to touch) and `forbidden_paths` (what to avoid).
4. **Read First**: Files to read before writing code.
5. **Shared Contracts**: Relevant entries from `DECISIONS.md`.
6. **Validation**: The command to run and the expected outcome.
7. **Definition of Done**: Exact deliverable expectations.
8. **Deliverable Format**: How to report results (structured Markdown summary + diff).

*(Ready-to-use prompt template: [`../examples/worker-prompt-template.md`](../examples/worker-prompt-template.md).)*
