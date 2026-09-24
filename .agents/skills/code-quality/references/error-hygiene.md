# Error Hygiene & Failure Transparency

> **Mandate**: Errors are first-class domain citizens. Never swallow exceptions, never discard root-cause stack traces, and maintain an absolute distinction between expected operational failures and catastrophic programmer bugs.

---

## 1. The Dual Taxonomy of Errors

Every error falls into one of two fundamentally distinct categories:

```text
┌─────────────────────────────────┬─────────────────────────────────┐
│     OPERATIONAL FAILURES        │        PROGRAMMER BUGS          │
├─────────────────────────────────┼─────────────────────────────────┤
│ • Network timeout or disconnect │ • Null pointer / undefined read │
│ • Invalid user input / payload  │ • Violated precondition/assert  │
│ • Database connection saturated │ • Out-of-bounds array access    │
│ • File not found / permission   │ • Unreachable code reached      │
├─────────────────────────────────┼─────────────────────────────────┤
│ ACTION: Handle, retry, wrap     │ ACTION: Crash / fail-fast loudly│
│ with context, report to caller. │ to prevent state corruption.    │
└─────────────────────────────────┴─────────────────────────────────┘
```

---

## 2. The Golden Rules of Error Hygiene

### 1. Never Swallow Errors
Catching an exception without logging, re-throwing, or executing an explicit fallback is a critical defect.

```typescript
// ❌ CATASTROPHIC: Silently destroys debugging information and leaves state broken
try {
  await database.persist(record);
} catch (e) {
  // Silent fail
}

// ✅ CLEAN: Structured fallback or contextual rethrow
try {
  await database.persist(record);
} catch (err) {
  throw new DatabaseWriteError(`Failed to persist record ${record.id}`, { cause: err });
}
```

### 2. Preserve Causal Chains
When transforming a low-level error into a domain-specific error, always retain the root cause so debugging tools and telemetry can inspect the full causal trace.

* **TypeScript / Modern Node.js**: Use `Error.cause`:
  ```typescript
  export class ServiceUnavailableError extends Error {
    constructor(message: string, options?: { cause?: unknown }) {
      super(message, options);
      this.name = "ServiceUnavailableError";
    }
  }
  // Usage:
  throw new ServiceUnavailableError("Payment service down", { cause: originalHttpError });
  ```
* **Go**: Use `%w` verb with `fmt.Errorf`:
  ```go
  if err := db.Ping(); err != nil {
      return fmt.Errorf("pinging primary db cluster: %w", err)
  }
  ```
* **Python**: Use explicit exception chaining (`raise ... from ...`):
  ```python
  try:
      raw = fetch_remote_config()
  except TimeoutError as exc:
      raise ConfigurationUnavailableError("Could not load cluster config") from exc
  ```
* **Rust**: Use `anyhow::Context` or `thiserror`:
  ```rust
  let config = fs::read_to_string("config.toml")
      .with_context(|| "Failed to read configuration file at config.toml")?;
  ```

### 3. Avoid Generic Catch-Alls
Avoid catching the top-level base exception (`catch (e)`, `except Exception:`, `catch (...)`) unless you are at the outermost system boundary (e.g., an HTTP global exception middleware or an event loop supervisor).

### 4. Provide Actionable Telemetry Context
An error message should tell the engineer **what** was being attempted, **with what key identifiers**, and **why** it failed:
* ❌ `"Failed to update database"`
* ✅ `"Failed to update order status to 'SHIPPED' for order_id=ord_9981: connection timed out after 3000ms"`

---

## 3. Graceful Degradation vs Fast Failure

| Situation | Correct Failure Strategy | Rationale |
| :--- | :--- | :--- |
| **Optional enrichment** (e.g. avatar fetch, non-critical metrics) | **Degrade Gracefully**: Catch local error, log warning, return default/fallback. | Main user transaction must not abort due to secondary failure. |
| **Transactional state** (e.g. billing, order placement, auth) | **Fail Fast & Rollback**: Abort operation immediately, trigger rollback, surface error. | Partial execution results in corrupted or out-of-sync financial/auth state. |
| **Internal invariant violation** (e.g. impossible branch in state machine) | **Panic / Throw Uncaught**: Terminate current process/request immediately. | Continuing under corrupted internal memory leads to unpredictable security holes. |
