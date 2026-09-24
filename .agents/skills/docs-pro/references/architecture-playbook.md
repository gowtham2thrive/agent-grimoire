# System Architecture Documentation Playbook

Evidence-backed architecture documentation enables developers and autonomous coding agents to locate responsibilities, trace flows, and modify code without violating system invariants.

---

## 1. System Map: Seven Dimensions

When documenting repository architecture, systematically identify and map these dimensions. Not every project has all seven — document what exists and explicitly note what is absent or not applicable:

1. **System Context & Boundaries**:
   - Who or what uses the system (human actors, client applications, upstream services, scheduled jobs)?
   - What downstream dependencies exist (databases, queues, caches, third-party APIs, file systems)?
   - What trust boundaries, authentication walls, or network perimeters separate components?

2. **Component Responsibilities**:
   - What are the major modules, packages, or services and their single responsibilities?
   - What is the dependency direction (top-down layered, onion/clean, hexagonal, event-driven, microservices)?

3. **Critical Data & Control Flows**:
   - Trace at least one primary read path and one primary write path end to end through the codebase.
   - Note serialization formats, message envelopes, and transformation points along each flow.

4. **System Invariants (Non-Negotiables)**:
   - What conditions must *always* remain true?
   - Examples: *"All mutations pass through the audit logger before commit"*, *"User tokens are verified at the edge gateway before reaching any downstream worker."*

5. **Failure Behaviors & Recovery**:
   - What happens when a downstream dependency is unavailable?
   - Document circuit breakers, retry policies, dead-letter queues, graceful degradation, and backpressure mechanisms.

6. **Change Locations (Where to Look)**:
   - Provide an explicit lookup guide for common modifications:
     - *"To add a new API endpoint → create handler in `src/handlers/`, register route in `src/router`."*
     - *"To add a database migration → create file in `migrations/`, update types in `src/types/db`."*
   - This section prevents the most common onboarding bottleneck: knowing *where* to make a change.

7. **Domain Glossary**:
   - Define ubiquitous domain terms explicitly so all engineers and agents use consistent nomenclature.
   - Distinguish terms that share names but differ in meaning across bounded contexts.

---

## 2. Architecture Decision Records (ADRs)

ADRs capture architectural choices, trade-offs, and design pivots to preserve engineering context across team changes and time.

- Store ADRs sequentially in a dedicated directory (e.g., `docs/adr/`) with zero-padded prefixes: `0001-use-sqlite-for-local-cache.md`.
- If the project already has an established ADR location or format, match it.
- Full template: [`adr-template.md`](adr-template.md).
