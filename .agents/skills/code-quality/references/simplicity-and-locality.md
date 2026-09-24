# Simplicity, Locality of Behavior & Cognitive Ergonomics

> **Mandate**: Write software that human engineers and AI agents can understand at a glance. Prioritize locality of behavior over fragmented abstraction, avoid speculative over-engineering, and keep cyclomatic complexity low.

---

## 1. Locality of Behavior (LoB)

> *"The behaviour of a unit of code should be as obvious as possible by looking only at that unit of code."* — Carson Gross

When code is split across dozens of files, abstract base classes, and indirection layers, engineers suffer from **cognitive thrashing**—having to open 8 tabs just to trace how a single button click saves a record.

### Anti-Pattern: The Speculative Indirection Web
```text
OrderController
  └── OrderService (interface)
        └── OrderServiceImpl
              └── OrderWorkflowManager
                    └── OrderStateValidatorFactory
                          └── StandardOrderStateValidator
```
For a simple CRUD update, this 6-layer architecture adds zero business value, bloats the token window, and increases bug surface.

### Clean Approach: Direct Cohesion
```typescript
export async function cancelOrder(orderId: string, reason: string, db: DatabaseClient): Promise<OrderResult> {
  const order = await db.orders.findById(orderId);
  if (!order) {
    return { success: false, error: "ORDER_NOT_FOUND" };
  }
  if (order.status !== "PENDING" && order.status !== "PROCESSING") {
    return { success: false, error: `CANNOT_CANCEL_STATUS_${order.status}` };
  }

  await db.transaction(async (tx) => {
    await tx.orders.updateStatus(orderId, "CANCELLED");
    await tx.auditLogs.insert({ entityId: orderId, action: "CANCEL", reason });
  });

  return { success: true };
}
```
*The entire business logic, validation, state transition, and transaction boundary are visible in one screen.*

---

## 2. The Anti-Overengineering Rulebook

### 1. The Rule of Three for Abstraction
Never create an interface, generic parameter, or abstract class for a single concrete implementation. Write the concrete implementation directly. When a second implementation arrives, evaluate similarities. When the **third** arrives, extract the shared abstraction.

### 2. Single Level of Abstraction (SLA)
A function should do one thing at one conceptual level:
```typescript
// ❌ BAD: Mixing high-level orchestration with low-level string slicing
export async function processUserData(rawInput: string) {
  // Low level
  const clean = rawInput.trim().toLowerCase().replace(/[^a-z0-9]/g, "");
  // High level
  const user = await fetchUserProfile(clean);
  // Low level byte manipulation
  const hash = crypto.createHash("sha256").update(user.secret).digest("hex");
  // High level
  await notifyUser(user.email, hash);
}

// ✅ GOOD: Decomposed to consistent abstraction level
export async function processUserData(rawInput: string) {
  const cleanId = sanitizeIdentifier(rawInput);
  const user = await fetchUserProfile(cleanId);
  const authHash = computeUserHash(user.secret);
  await notifyUser(user.email, authHash);
}
```

### 3. Eliminate Flag Arguments
Passing boolean flags to functions (`processOrder(order, true, false, true)`) forces the caller to know internal conditional branches and leads to temporal coupling.
* Prefer two distinct functions (`processExpeditedOrder(order)` vs `processStandardOrder(order)`) or a structured options object.

### 4. Guard Clauses over Deep Nesting
Never nest `if` statements 4 levels deep. Use early return guard clauses:
```typescript
// ❌ BAD: Deep arrow anti-pattern
function handle(req) {
  if (req.auth) {
    if (req.body) {
      if (req.body.id) {
        return doWork();
      }
    }
  }
  return null;
}

// ✅ GOOD: Flat guard clauses
function handle(req) {
  if (!req.auth) return null;
  if (!req.body) return null;
  if (!req.body.id) return null;

  return doWork();
}
```
