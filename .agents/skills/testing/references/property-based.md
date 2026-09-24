# Property-Based Testing & Invariant Design

> **Mandate**: Example-based tests (`assert f(2) == 4`) test your assumptions, not your invariants. For parsers, serializers, schemas, encoders, and algorithmic logic, verify universal mathematical properties across thousands of randomly generated inputs.

---

## 1. When to Use Property-Based Testing

Do not use property-based testing for high-level business flows (e.g. "user clicks checkout"). Use it where input spaces are infinite and boundary edge cases are subtle:

| Domain | Universal Invariant to Assert |
| :--- | :--- |
| **Serialization / Parsing** | **Round-Trip**: `deserialize(serialize(x)) === x` |
| **Text Normalization / Formatting** | **Idempotency**: `format(format(x)) === format(x)` |
| **Collection Operations (Sort/Filter)** | **Length & Membership**: `sort(list).length === list.length` |
| **Financial / Math Calculations** | **Commutativity / Identity**: `add(a, b) === add(b, a)` |
| **State Machine Transitions** | **Valid Reachability**: System state never transitions into an undefined enum. |

---

## 2. Universal Properties in Code

### Property 1: The Round-Trip Invariant (Fast-Check / TypeScript)
```typescript
import fc from "fast-check";
import { encodeBase64Url, decodeBase64Url } from "./codec";

test("Round-trip encoding and decoding preserves arbitrary byte arrays", () => {
  fc.assert(
    fc.property(fc.uint8Array({ minLength: 0, maxLength: 4096 }), (original) => {
      const encoded = encodeBase64Url(original);
      const decoded = decodeBase64Url(encoded);
      expect(decoded).toEqual(original);
    }),
    { numRuns: 1000 }
  );
});
```

### Property 2: Idempotency (Hypothesis / Python)
```python
from hypothesis import given, strategies as st
from my_sanitizer import sanitize_phone_number

@given(st.text(min_size=1, max_size=50))
def test_sanitization_is_idempotent(raw_input: str):
    first_pass = sanitize_phone_number(raw_input)
    second_pass = sanitize_phone_number(first_pass)
    assert first_pass == second_pass
```

### Property 3: Sorting Invariance (Go Standard Fuzzing)
```go
func FuzzSortInts(f *testing.F) {
    // Seed corpus
    f.Add([]byte{5, 2, 6, 3, 1, 4})
    
    f.Fuzz(func(t *testing.T, data []byte) {
        ints := make([]int, len(data))
        for i, b := range data {
            ints[i] = int(b)
        }
        
        origLen := len(ints)
        CustomSort(ints)
        
        // Invariant 1: Length preserved
        if len(ints) != origLen {
            t.Fatalf("length mutated from %d to %d", origLen, len(ints))
        }
        // Invariant 2: Strictly monotonic ordering
        for i := 0; i < len(ints)-1; i++ {
            if ints[i] > ints[i+1] {
                t.Fatalf("unsorted elements at index %d: %d > %d", i, ints[i], ints[i+1])
            }
        }
    })
}
```

---

## 3. Shrinking & Reproducibility
* Property-based frameworks automatically **shrink** failing inputs to the minimal reproducible case (e.g. if an array of 500 items fails, it shrinks it down to finding that `[0, -1]` causes the bug).
* Always log the **falsifying seed** so that edge cases discovered by fuzzing/property testing can be permanently added to the regression test suite as a standard unit test.
