# Hostile Path Testing & The Mutation Mindset

> **Mandate**: Systems rarely fail on the happy path. Software must be verified against adversarial inputs, simulated infrastructure failures, corrupted data, and concurrent race conditions. Apply the mutation testing mindset to guarantee tests actually catch defects.

---

## 1. The Hostile Test Matrix

Every non-trivial capability must be subjected to the **Four Hostile Vectors**:

```text
┌──────────────────────────────┬──────────────────────────────┐
│ 1. MALFORMED / BOUNDARY DATA │ 2. INFRASTRUCTURE & NETWORK  │
├──────────────────────────────┼──────────────────────────────┤
│ • Empty strings, whitespace  │ • Network timeout simulation │
│ • Negative numbers, max int  │ • Dropped DB connections     │
│ • Deeply nested JSON / arrays│ • Disk full / read-only FS   │
│ • Unexpected Unicode / emojis│ • DNS resolution failure     │
├──────────────────────────────┼──────────────────────────────┤
│ 3. CONCURRENCY & RACE HAZARDS│ 4. AUTH & SECURITY BYPASS    │
├──────────────────────────────┼──────────────────────────────┤
│ • Simultaneous double-spend  │ • Expired / forged JWT       │
│ • Reentrant event handlers   │ • Missing role / permissions │
│ • Out-of-order message delivery│ • SQL / NoSQL injection payload│
└──────────────────────────────┴──────────────────────────────┘
```

---

## 2. The Mutation Testing Mindset

**Mutation Testing** evaluates the quality of your test suite by injecting intentional defects (mutants) into the code and checking whether at least one test fails ("kills the mutant").

### The 3 Inline Mutation Checks (Self-Verification)
Before committing a test, mentally or temporarily apply these three mutations to the code under test:

1. **Boundary Mutation**:
   - Change `if (x >= limit)` to `if (x > limit)`.
   - *Result*: Does your boundary test fail? If not, you are missing a test for `x === limit`.
2. **Boolean / Logical Inversion**:
   - Invert an error check condition: `if (err != null)` $\rightarrow$ `if (err == null)`.
   - *Result*: Does your hostile path test fail?
3. **Statement Deletion / Return Mutation**:
   - Comment out the state-saving line (`await db.save(record)`) or return a default value (`return null`).
   - *Result*: Does your integration test catch the missing side effect?

> [!IMPORTANT]
> If a mutant survives (i.e. all tests remain green despite intentional code corruption), your test assertions are too weak or incomplete. Strengthen the assertions before proceeding.

---

## 3. Simulating Hostile Infrastructure Failures

### Testing Timeouts & Aborts (TypeScript)
```typescript
test("aborts and releases resources when gateway times out", async () => {
  const slowServer = http.createServer((req, res) => {
    // Hangs indefinitely without responding
  });
  await new Promise((r) => slowServer.listen(0, r));
  const port = (slowServer.address() as net.AddressInfo).port;

  try {
    const client = new PaymentClient(`http://localhost:${port}`, { timeoutMs: 50 });
    await expect(client.processCharge("tok_123", 500)).rejects.toThrow(
      new GatewayTimeoutError("Gateway did not respond within 50ms")
    );
  } finally {
    slowServer.close();
  }
});
```

### Testing Concurrency & Double-Execution (Go)
```go
func TestConcurrentTransferNoDoubleSpend(t *testing.T) {
    db := setupTestDB(t)
    account := createAccount(t, db, 100) // $100 initial balance

    var wg sync.WaitGroup
    for i := 0; i < 2; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            // Try to spend $100 twice simultaneously
            _ = accountService.Transfer(db, account.ID, targetID, 100)
        }()
    }
    wg.Wait()

    finalBalance := getBalance(t, db, account.ID)
    if finalBalance < 0 {
        t.Fatalf("account overdrawn: balance is %d", finalBalance)
    }
    if finalBalance != 0 {
        t.Fatalf("expected balance 0, got %d", finalBalance)
    }
}
```
