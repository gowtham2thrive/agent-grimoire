# Assertion Strength & False-Confidence Elimination

> **Mandate**: A test with weak assertions is worse than no test—it creates a dangerous illusion of safety (False Confidence). Assert the exact semantic state, schema shape, and domain invariants of the system; eliminate tautological assertions and mock drift.

---

## 1. Catalog of Dangerous Weak Assertions

### Weak Assertion 1: Existence Without Value or Schema
```typescript
// ❌ DANGEROUS: Passes even if data is an empty object, an error response, or corrupted!
const res = await api.getUser("123");
expect(res).toBeDefined();
expect(res.data).not.toBeNull();

// ✅ STRONG: Asserts exact schema shape and identity invariant
expect(res.status).toBe(200);
expect(res.data).toMatchObject({
  id: "123",
  email: "user@example.com",
  role: "MEMBER",
});
```

### Weak Assertion 2: HTTP Status Without Body Inspection
```typescript
// ❌ DANGEROUS: Passes if endpoint returns 200 with { error: "Database offline" }
const res = await request(app).post("/api/orders").send(payload);
expect(res.status).toBe(201);

// ✅ STRONG: Asserts status AND database record persistence
expect(res.status).toBe(201);
expect(res.body.orderId).toMatch(/^ord_[a-z0-9]+$/);
const savedOrder = await db.orders.findById(res.body.orderId);
expect(savedOrder.totalCents).toBe(payload.amount);
expect(savedOrder.status).toBe("PENDING");
```

### Weak Assertion 3: Tautological Mock Verification
```typescript
// ❌ DANGEROUS: You just passed { id: 1 }, asserting the mock received { id: 1 } tests nothing!
userService.getUser = vi.fn().mockResolvedValue({ id: 1 });
await controller.handle({ id: 1 });
expect(userService.getUser).toHaveBeenCalledWith({ id: 1 });

// ✅ STRONG: Assert that the controller correctly transformed the result or propagated error
const result = await controller.handle({ id: 1 });
expect(result.viewPayload).toEqual({ userId: 1, formattedDate: "2026-09-24" });
```

---

## 2. Preventing Mock Drift

**Mock Drift** occurs when tests pass against mocked dependencies that no longer match the real-world behavior of the downstream service (e.g. API returns a new schema or different error code in production).

### Rules to Prevent Mock Drift:
1. **Prefer Real In-Memory Adapters**: Use SQLite in-memory, fake filesystem trees, or mock HTTP servers with real schema validation rather than arbitrary method mocks.
2. **Contract-Backed Mocks**: If mocks are necessary, derive mock return types directly from TypeScript interfaces or OpenAPI schemas.
3. **Verify Error Signatures**: When mocking an error, mock the exact error class and causal properties thrown by the real implementation, not generic `new Error()`.

---

## 3. Negative Assertion Standards

When testing failure paths, never just assert that an error was thrown; assert the **exact error type** and **causal message**:

```typescript
// ❌ WEAK: Passes if code throws a TypeError because of a typo in the test!
await expect(service.transferFunds(-100)).rejects.toThrow();

// ✅ STRONG: Asserts exact domain exception and rejection reason
await expect(service.transferFunds(-100)).rejects.toThrow(
  new InvalidTransferAmountError("Amount must be strictly positive, received: -100")
);
```
