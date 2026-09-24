# Archetype Quality Patterns

> **Mandate**: Software engineering rules must adapt to the concrete runtime environment. Apply the specific defensive patterns tailored to the 7 major software archetypes.

---

## 1. Web APIs & Microservices (FastAPI, Go Gin, Express, Spring Boot)
* **Ingress Validation**: Bind request bodies directly to strict DTO/schema validators. Reject unknown or unexpected fields to prevent mass-assignment vulnerabilities.
* **Context Cancellation**: Ensure every database query, Redis lookup, and external HTTP call accepts the incoming request's cancellation context (`req.context` in Go, `AbortSignal` in Node, `request.is_disconnected()` in FastAPI).
* **Idempotency**: External mutations (e.g. charge payment, create order) must support an `Idempotency-Key` header with database-backed deduplication.
* **Graceful Teardown**: Listen for `SIGINT` / `SIGTERM` signals, stop accepting new connections, drain ongoing in-flight requests within a grace period (e.g. 15s), and cleanly disconnect DB pools.

---

## 2. Frontend & Mobile (React, Vue, Flutter, SwiftUI)
* **Defensive Rendering**: Never assume nested API models exist. Guard against `undefined` / `null` fields in UI components.
* **Effect Cleanup**: Every `useEffect`, `StreamSubscription`, or event listener must return a cleanup function to prevent memory leaks on unmount.
* **Optimistic Rollback**: When performing optimistic UI updates, always capture a snapshot of previous state and revert cleanly if the network call fails.
* **Locality of State**: Keep state as close as possible to the component that renders it. Do not push local form state into global stores (Redux/Zustand) unless other components require it.

---

## 3. CLI Tools & System Daemons (Rust, Go, C++, Python)
* **POSIX Exit Codes**: `0` for success; non-zero (`1`, `2`, etc.) for errors. Never exit `0` when an unhandled error or validation failure occurred.
* **Standard Streams**: Send normal structured output to `STDOUT`; send error logs, warnings, and progress indicators to `STDERR` so CLI pipelines (`grep`, `jq`, pipe redirection) remain clean.
* **Atomic File Updates**: Never overwrite an active configuration or data file in place. Write the new content to a unique temporary file (`config.tmp.1234`), flush to disk (`fsync`), and perform an atomic filesystem rename over the target.

---

## 4. Libraries & Public SDKs (npm, Crates.io, PyPI)
* **Zero Global Mutation**: Never mutate global prototypes (`String.prototype`), register global polyfills, or set global environment variables inside a library.
* **Semantic Versioning Integrity**: Changing a public function signature, removing an export, or altering an error type is a breaking change requiring a major version bump.
* **Minimal Dependency Tree**: Avoid adding external dependencies for trivial utilities (e.g. left-pad, is-number). Keep the supply-chain footprint light.

---

## 5. Data & ML Pipelines (PySpark, dbt, PyTorch, Pandas)
* **Schema Evolution Guards**: Validate column names, nullability, and data types before executing expensive multi-stage DAGs.
* **Stochastic Determinism**: Fix random seeds (`torch.manual_seed(42)`, `np.random.seed(42)`) in training and preprocessing scripts to enable reproducible debugging.
* **Data Leakage Prevention**: Enforce strict architectural boundaries between training sets, validation sets, and test sets. Preprocessing transforms must be fit strictly on train sets.

---

## 6. Infrastructure as Code & GitOps (Terraform, Kubernetes, Ansible)
* **Immutable State**: Treat cloud resources as immutable; prefer replacement over in-place configuration mutation.
* **Secret Isolation**: Never commit raw API keys, passwords, or private certificates. Pull secrets dynamically from cloud secret managers or encrypted vaults.
* **Drift & Dry-Run**: Ensure every resource definition can be planned/simulated (`terraform plan`, `kubectl diff`) without side effects.

---

## 7. Monorepos & Polyglot Workspaces (Turborepo, Bazel, Cargo)
* **Package Seam Encapsulation**: A package in a monorepo must only expose its public API via its top-level entrypoint (`index.ts` / `lib.rs` / `package.go`). Deep imports into internal files (`@pkg/internal/utils/secret.ts`) are forbidden.
* **No Circular Dependencies**: Cross-package dependencies must form a Directed Acyclic Graph (DAG).
