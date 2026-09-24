# Characterization Testing (Pinning Legacy Behavior)

> **Mandate**: Never refactor code without an active test harness. When working with legacy, brownfield, or undocumented code that lacks tests, you must write characterization tests (Golden Master snapshots) to pin existing behavior before altering a single line of production code.

---

## 1. What is a Characterization Test?

A **Characterization Test** (introduced by Michael Feathers) does not assert what the code *should* do according to an ideal specification; it asserts what the code *actually does right now* (including its quirks, edge case outputs, and error states).

```text
[ Unverified Legacy Function ] ──(Inject sample inputs A, B, C)──► [ Capture Exact Outputs ]
                                                                             │
                                                                   [ Save as Snapshot / Assertion ]
                                                                             │
                                                          [ REFACTOR INTERNAL IMPLEMENTATION ]
                                                                             │
                                              (Run Characterization Test: Must remain 100% green!)
```

---

## 2. The Pinning Protocol

Execute these four steps before touching production code:

### Step 1: Write an Intentionally Failing Assertion
Invoke the legacy function with representative inputs, asserting an arbitrary placeholder value to force the test to fail:
```typescript
test("characterize calculateTax output", () => {
  const result = legacyTaxCalculator({ subtotal: 100, state: "CA", isExempt: false });
  expect(result).toBe("TEMP_PLACEHOLDER");
});
```

### Step 2: Record the Real-World Output
Run the test and observe the failure message:
```text
Expected: "TEMP_PLACEHOLDER"
Received: { taxCents: 825, appliedRate: 0.0825, jurisdiction: "CA-STATE" }
```

### Step 3: Pin the Real-World Output
Update the assertion to match the actual output of the legacy system:
```typescript
test("characterize calculateTax output for standard CA taxable order", () => {
  const result = legacyTaxCalculator({ subtotal: 100, state: "CA", isExempt: false });
  expect(result).toEqual({
    taxCents: 825,
    appliedRate: 0.0825,
    jurisdiction: "CA-STATE",
  });
});
```

### Step 4: Expand the Input Space
Repeat for boundary inputs:
- Zero subtotal (`subtotal: 0`)
- Negative subtotal (does it throw? return 0?)
- Exempt status (`isExempt: true`)
- Unknown state code (`state: "ZZ"`)

Once you have 4–8 characterization tests passing green, **you have pinned the behavior**. You may now refactor the internals of `legacyTaxCalculator` with 100% confidence. If any characterization test turns red, your refactor broke an invariant.
