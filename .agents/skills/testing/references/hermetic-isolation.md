# Hermetic Test Isolation & Flakiness Elimination

> **Mandate**: Flaky tests destroy trust in automated engineering. A test must produce the exact same deterministic outcome whether run once in isolation or 10,000 times in parallel. Eliminate real time waits, enforce ephemeral test boundaries, and guarantee zero leaked state.

---

## 1. The Causes of Test Flakiness

```text
┌─────────────────────────────────┬─────────────────────────────────┐
│       FLAKINESS VECTOR          │       HERMETIC REMEDIATION      │
├─────────────────────────────────┼─────────────────────────────────┤
│ • Real `sleep()` / wall-clock   │ • Fake clocks & promise polling │
│ • Hardcoded network ports       │ • Ephemeral OS port binding (:0)│
│ • Shared database state / leaks │ • Fresh DB / transaction per test│
│ • Unseeded random data (Faker)  │ • Deterministic PRNG seeds      │
│ • Order-dependent execution     │ • Strict per-test setup/teardown│
│ • Unclosed background tasks     │ • Cancellation on test teardown │
└─────────────────────────────────┴─────────────────────────────────┘
```

---

## 2. Eliminating Time-Based Flakiness

### Anti-Pattern: Wall-Clock Sleep
```typescript
// ❌ FLAKY: Fails on slow CI runners, wastes developer time on fast machines
await triggerBackgroundTask();
await new Promise((r) => setTimeout(r, 1500)); // Hope it finished
expect(await checkResult()).toBe("DONE");
```

### Clean Alternative A: Fake Timers (Synchronous Time Travel)
```typescript
// ✅ FAST & DETERMINISTIC: Advances virtual clock instantaneously
vi.useFakeTimers();
try {
  triggerPeriodicSync();
  vi.advanceTimersByTime(60_000); // Fast forward 1 minute
  expect(syncCount).toBe(1);
} finally {
  vi.useRealTimers();
}
```

### Clean Alternative B: Event-Driven Polling with Hard Cap
```typescript
// ✅ ROBUST: Resolves the exact millisecond the condition becomes true
export async function waitForCondition(
  predicate: () => Promise<boolean> | boolean,
  timeoutMs: number = 2000,
  intervalMs: number = 25
): Promise<void> {
  const start = Date.now();
  while (Date.now() - start < timeoutMs) {
    if (await predicate()) return;
    await new Promise((r) => setTimeout(r, intervalMs));
  }
  throw new Error(`Condition not met within ${timeoutMs}ms`);
}
```

---

## 3. Hermetic State Isolation

### 1. Ephemeral Port Binding for Network Tests
Never hardcode ports like `3000` or `8080`. Hardcoded ports collide during parallel test runs.
```javascript
// ✅ Binds to an unused ephemeral port assigned by the OS kernel
const server = app.listen(0, () => {
  const port = server.address().port;
  console.log(`Running on dynamic port ${port}`);
});
```

### 2. Filesystem Isolation (Unique Temp Directories)
Never write test artifacts to fixed paths like `./tmp/test.json`.
```typescript
import os from "node:os";
import path from "node:path";
import fs from "node:fs/promises";
import crypto from "node:crypto";

export async function withIsolatedDirectory(testFn: (tmpDir: string) => Promise<void>) {
  const uniqueId = crypto.randomUUID();
  const tmpDir = path.join(os.tmpdir(), `grimoire-test-${uniqueId}`);
  await fs.mkdir(tmpDir, { recursive: true });

  try {
    await testFn(tmpDir);
  } finally {
    await fs.rm(tmpDir, { recursive: true, force: true });
  }
}
```

### 3. Database Isolation (Transactions or Ephemeral DBs)
* **Strategy A (Transaction Rollback)**: Begin a transaction before each test, run the test, and unconditionally rollback in `afterEach`.
* **Strategy B (Ephemeral SQLite/Schema)**: Create a unique database file or SQLite `:memory:` connection per test suite.
