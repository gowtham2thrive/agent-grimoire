# Concurrency Safety & Resource Lifetimes

> **Mandate**: Every acquired resource must have an unambiguous, deterministic release scope. Every asynchronous or concurrent operation must respect cancellation signals and explicit deadlines. Never permit shared mutable state without synchronization guarantees.

---

## 1. Deterministic Resource Management

Resource leaks (unclosed sockets, orphaned file descriptors, memory leaks from event listeners, dangling database transactions) degrade systems over time and cause unpredictable outages.

### The Scope-Bound Acquisition Principle
Every resource acquisition must be immediately paired with its deterministic release construct:

| Runtime / Language | Recommended Mechanism | Code Idiom |
| :--- | :--- | :--- |
| **Go** | `defer` | `file, err := os.Open(path); if err != nil { return err }; defer file.Close()` |
| **Rust / C++** | RAII (Drop trait) | `let _guard = mutex.lock().unwrap(); // Released when _guard goes out of scope` |
| **Python** | Context Manager (`with`) | `with open(path, "r", encoding="utf-8") as f: data = f.read()` |
| **TypeScript / Node.js** | `try...finally` or `using` | `const client = await pool.connect(); try { ... } finally { client.release(); }` |
| **C# / .NET** | `using` statement | `using var stream = File.OpenRead(path);` |

---

## 2. Cancellation & Timeout Propagation

Never initiate unbounded asynchronous operations. Network requests, subprocess spawns, and worker tasks must accept timeouts and cancellation signals.

### TypeScript / JavaScript: `AbortSignal`
```typescript
export async function fetchWithDeadline<T>(
  url: string,
  timeoutMs: number,
  parentSignal?: AbortSignal
): Promise<T> {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(new Error("Operation timed out")), timeoutMs);

  // Link parent signal if provided
  if (parentSignal) {
    parentSignal.addEventListener("abort", () => controller.abort(parentSignal.reason), { once: true });
  }

  try {
    const response = await fetch(url, { signal: controller.signal });
    if (!response.ok) throw new Error(`HTTP error ${response.status}`);
    return (await response.json()) as T;
  } finally {
    clearTimeout(timeoutId);
  }
}
```

### Go: `context.Context`
```go
func QueryWithTimeout(ctx context.Context, db *sql.DB, query string, timeout time.Duration) (*Result, error) {
    queryCtx, cancel := context.WithTimeout(ctx, timeout)
    defer cancel()

    rows, err := db.QueryContext(queryCtx, query)
    if err != nil {
        if errors.Is(queryCtx.Err(), context.DeadlineExceeded) {
            return nil, fmt.Errorf("query exceeded deadline of %v: %w", timeout, err)
        }
        return nil, err
    }
    defer rows.Close()

    return processRows(rows)
}
```

---

## 3. Eliminating Concurrency Hazards

### 1. The Check-Then-Act Race Condition
* **Flaw**: Reading state, determining an action, and then writing state without locking or atomic guarantees.
* **Remediation**:
  - Use database row-level locking (`SELECT ... FOR UPDATE`), optimistic concurrency checks (`UPDATE ... WHERE version = expected_version`), or atomic memory primitives (`atomic.CompareAndSwap`).

### 2. Lock Inversion & Deadlocks
* When multiple locks must be acquired, **always acquire them in a globally consistent order** (e.g. sorted by resource ID).
* Keep the duration of lock holding as short as possible; never perform network I/O or disk operations while holding an in-memory lock.

### 3. Reentrancy & Double-Invocation
* Ensure that event handlers, webhooks, or background workers are **idempotent**.
* Use idempotency keys, unique database constraints, or deduplication windows to safely handle duplicate events.
