# Defensive Boundaries & Type Invariants

> **Mandate**: Validate and sanitize data at untrusted system boundaries; once inside the boundary, represent data in types that make invalid states unrepresentable. Never scatter redundant defensive checks throughout internal domain logic when strong types can eliminate the defect category entirely.

---

## 1. The Boundary Cartography Principle

Software architectures have distinct trust zones:

```text
[ Untrusted World: HTTP, CLI, DB, File, Env ]
                     │
         === System Boundary === (VALIDATE & NORMALIZE HERE)
                     │
[ Trusted Core Domain: Strongly Typed Entities, Invariants Guaranteed ]
```

1. **At the Boundary**:
   - Treat every byte as hostile or malformed until proven otherwise.
   - Enforce schema validation (lengths, regex, bounds, enum inclusion, structural integrity).
   - Reject early with descriptive, user- or caller-friendly error messages.
2. **Inside the Trusted Core**:
   - Do not pass raw dictionaries, stringly-typed primitives, or unchecked generic objects.
   - Rely on domain types and compiler invariants. If a function accepts a `UserId`, it must be impossible to pass an arbitrary unvalidated string.

---

## 2. Making Illegal States Unrepresentable

### Anti-Pattern: Primitive Obsession & Boolean Flag Combinatorics
```typescript
// ❌ BAD: Allows impossible states (e.g. status === "success" but error is defined, or data is missing)
interface AsyncResult {
  status: "idle" | "loading" | "success" | "error";
  data?: UserData;
  error?: string;
  statusCode?: number;
}
```

### Clean Invariant: Tagged Unions / Algebraic Data Types
```typescript
// ✅ GOOD: The type system guarantees that data ONLY exists when status is "success"
type AsyncResult =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: UserData }
  | { status: "error"; error: DomainError; statusCode: number };
```

---

## 3. Language-Specific Boundary & Invariant Idioms

### TypeScript / JavaScript
* **Boundary Validation**: Use runtime validation libraries (such as Zod, ArkType, or TypeBox) or strict type guards at ingress:
  ```typescript
  import { z } from "zod";

  export const CreateUserSchema = z.object({
    email: z.string().email(),
    age: z.number().int().min(18).max(120),
    role: z.enum(["admin", "member"]),
  });

  export type CreateUserInput = z.infer<typeof CreateUserSchema>;
  ```
* **Branded / Nominal Types**: Prevent passing arbitrary strings where a specific ID is required:
  ```typescript
  export type UserId = string & { readonly __brand: unique symbol };
  export function parseUserId(raw: string): UserId {
    if (!/^[a-f0-9-]{36}$/.test(raw)) throw new ValidationError("Invalid UUID");
    return raw as UserId;
  }
  ```

### Python
* **Ingress Validation**: Leverage Pydantic or `@dataclass(frozen=True)` with explicit post-init checks:
  ```python
  from dataclasses import dataclass

  @dataclass(frozen=True)
  class EmailAddress:
      value: str

      def __post_init__(self):
          if "@" not in self.value or "." not in self.value.split("@")[-1]:
              raise ValueError(f"Malformed email address: {self.value}")

      def __str__(self) -> str:
          return self.value
  ```
* **Algebraic State Machines**: Use `Union` and pattern matching:
  ```python
  from typing import Union, Literal
  from dataclasses import dataclass

  @dataclass(frozen=True)
  class Pending: status: Literal["pending"] = "pending"
  @dataclass(frozen=True)
  class Active: token: str; status: Literal["active"] = "active"
  @dataclass(frozen=True)
  class Terminated: reason: str; status: Literal["terminated"] = "terminated"

  SessionState = Union[Pending, Active, Terminated]
  ```

### Go
* **Constructor Invariants**: Make struct fields private and force creation through validating constructors:
  ```go
  package domain

  import "errors"

  type NonEmptyString struct {
      value string
  }

  func NewNonEmptyString(raw string) (NonEmptyString, error) {
      if len(raw) == 0 {
          return NonEmptyString{}, errors.New("string cannot be empty")
      }
      return NonEmptyString{value: raw}, nil
  }

  func (s NonEmptyString) Value() string {
      return s.value
  }
  ```

### Rust
* **Parse, Don't Validate**: Leverage Rust's `enum` and `TryFrom` to parse untrusted data into strong types:
  ```rust
  #[derive(Debug, Clone, PartialEq, Eq)]
  pub struct PositiveInt(u64);

  impl TryFrom<i64> for PositiveInt {
      type Error = &'static str;

      fn try_from(value: i64) -> Result<Self, Self::Error> {
          if value > 0 {
              Ok(PositiveInt(value as u64))
          } else {
              Err("Value must be strictly positive")
          }
      }
  }
  ```

---

## 4. Boundary Checklist
Before completing boundary-adjacent code:
- [ ] Are all external inputs (JSON payloads, query parameters, CLI flags, env vars) validated against an explicit schema?
- [ ] Are numeric values validated for bounds (min/max, off-by-one, overflow risks)?
- [ ] Are string values bounded in length to prevent memory exhaustion / DoS?
- [ ] Are illegal combinations of parameters rejected at creation time?
