# Micro-Sizing & Edge-Case Execution Traces

> **Purpose**: Demonstrate how the `requirements-analysis` protocol operates under extreme conditions: frictionless `micro` mode for tiny edits, `ambiguity-rescue` mode for contradictory prompts, and `REQ-DELTA` baseline re-negotiation for brownfield architectural collisions.

---

## Case 1 · Micro-Mode Execution (Zero Bureaucracy)

### User Prompt
> *"The CLI tool crashes with an unhandled exception if the `--port` flag is set to a negative number. Fix it."*

### Analysis & Sizing
* **Task Blast Radius**: Single argument validation check ($< 10$ lines of code).
* **Mode Selected**: **`micro`**.
* **Protocol Applied**: Skip all PRDs, diagrams, RTMs, and interviews. Emit the **3-Line Requirement Intent** block directly before code execution:

```markdown
> **Intent**: Reject negative `--port` arguments with an informative error message to prevent unhandled crash exceptions.
> **Boundary**: Valid port domain is $[1, 65535]$; values $\le 0$ or $> 65535$ must emit an error to stderr and exit with status code 2.
> **Acceptance**: Deterministic CLI invocation `mycli --port -1` exits with code 2 and outputs `Error: Invalid port '-1'. Port must be between 1 and 65535.`
```

*Result*: Instant execution, zero friction, perfect verifiability.

---

## Case 2 · Ambiguity-Rescue Mode (Contradictory Requirements)

### User Prompt
> *"We want a 100% decentralized peer-to-peer messaging app with zero servers and zero cloud storage, but users must be able to instantly search across all messages ever sent in any chat room globally in under 100 milliseconds without downloading the chat history to their phones."*

### Analysis & Sizing
* **Defect Identified**: Physical/mathematical impossibility (Direct Contradiction).
* **Root Conflict**:
  - Premise 1: Zero servers and zero cloud storage (no centralized index).
  - Premise 2: Zero download of chat history to client phone (no local index).
  - Premise 3: Global search across all messages in $< 100\text{ms}$.
  - *Conflict*: To search an index in $< 100\text{ms}$, the index must physically reside either on a remote server/cloud service OR locally on the client device. Information theory forbids searching unindexed, unheld data without downloading or querying an index host.

### Protocol Applied: Triad Trade-off Matrix & Single-Turn Structured Clarification
The agent **never** hallucinates a impossible solution. It triggers the **Clarification Gate** via `ask_question`:

```markdown
Question: "The requirement for global instant search without servers and without downloading data conflicts with physical information storage laws. How would you like to resolve this architectural trade-off?"
Options:
- "(Recommended) Client-Side Encrypted Cache: Messages are stored peer-to-peer, but each user builds a local encrypted SQLite search index of rooms they participate in"
- "Federated Blind Index: Introduce self-hosted community relay nodes that maintain an encrypted search index without reading message plaintext"
- "Scoped Ephemeral Search: Restrict search strictly to currently loaded in-memory messages of active peer sessions (no historical global search)"
```

*Outcome*: The user selects Option 1. The requirement is formalized cleanly without infinite loops or impossible promises.

---

## Case 3 · Brownfield Collision & Living Baseline Re-negotiation (`REQ-DELTA`)

### Background
During initial requirements analysis, the user agreed to:
* `REQ-004`: *"The sync agent shall stream database change events using PostgreSQL Logical Replication via the `pgoutput` plugin."*

### Downstream Roadblock Discovered in `project-analysis`
While inspecting the existing host repository, `project-analysis` discovers that the production database is Amazon Aurora PostgreSQL configured with a read-replica cluster where the master database does NOT have `rds.logical_replication = 1` enabled, and company compliance strictly forbids altering master database parameters without a 30-day change review board.

### Protocol Applied: The `REQ-DELTA` State Machine
The agent halts premature mutation and emits an attributable requirement delta:

```markdown
### REQ-DELTA: REQ-004 (Revision from v1.0 to v1.1)

- **Requirement Affected**: [REQ-004: PostgreSQL Logical Replication streaming]
- **Invalidated Assumption**: [Assumed logical replication slot creation was permitted on the production database cluster]
- **Evidence Anchor**: `[CODE: config/database.yml#L18]` & `[CODE: terraform/aurora.tf#L42]` (Parameter group has `rds.logical_replication = 0`; compliance policy forbids parameter group mutation)
- **Impact**: Continuing with logical replication will fail immediately in integration testing with `ERROR: must be superuser to create replication slot`.
- **Proposed Alternatives**:
  - *Option A (Audit-Log Polling)*: Transition sync mechanism to poll the existing immutable `audit_events` ledger table with timestamp watermarking.
  - *Option B (Application Event Hook)*: Publish domain events directly from the application service bus at transaction commit time.
- **Decision Adopted**: Option A (Maintains zero modifications to application code while respecting database compliance locks).
- **Updated Requirement Statement (EARS)**:
  `The sync agent SHALL poll the audit_events ledger table every 1000ms using monotonic sequence watermarks to detect newly committed transactions.`
```

*Result*: The baseline is versioned to v1.1 with clean evidence traceability, preventing days of wasted engineering on an impossible replication slot.
