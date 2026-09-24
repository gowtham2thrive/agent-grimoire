# Agent Refactoring Catalogue

> **Mandate**: Standardized, atomic refactoring moves adapted for autonomous AI agents. Every recipe defines preconditions, execution steps, and verification invariants to guarantee zero accidental behavioral mutations.

---

## 1. Extract Function / Method

### Intent
Transform a cohesive fragment of code inside a long function into its own named function at the same or helper module scope.

### Preconditions
- The code fragment performs a single logical operation (e.g. data validation, tax calculation, string formatting).
- Local variables read by the fragment are identified as parameters; local variables modified by the fragment are identified as return values.

### Execution Steps
1. Create a new empty function with an intention-revealing name.
2. Copy the code fragment from the source function into the target function body.
3. Pass all read-only variables as strongly-typed parameters.
4. If the fragment modifies a local variable, return that value from the new function.
5. Replace the original code fragment in the source function with a call to the extracted function.
6. **Run tests immediately**.

---

## 2. Replace Conditional with Map / Strategy

### Intent
Eliminate sprawling `switch` statements or nested `if/else` ladders that grow with every new business type.

### Preconditions
- A `switch` or `if/else` statement inspects a single discriminator property (e.g. `type: "CREDIT_CARD" | "PAYPAL" | "CRYPTO"`).

### Execution Steps
1. Define a common interface or function signature for the operation:
   ```typescript
   type PaymentHandler = (order: Order) => Promise<PaymentResult>;
   ```
2. Extract each case branch into an independent handler function.
3. Create a lookup dictionary or map mapping discriminators to handlers:
   ```typescript
   const PAYMENT_HANDLERS: Record<PaymentMethod, PaymentHandler> = {
     CREDIT_CARD: processCreditCard,
     PAYPAL: processPayPal,
     CRYPTO: processCrypto,
   };
   ```
4. Replace the `switch` statement with a map lookup and a fallback guard:
   ```typescript
   const handler = PAYMENT_HANDLERS[order.paymentMethod];
   if (!handler) throw new UnsupportedPaymentMethodError(order.paymentMethod);
   return await handler(order);
   ```
5. **Run tests immediately**.

---

## 3. Introduce Parameter Object

### Intent
Replace a function that accepts 4 or more individual parameters with a structured, strongly-typed configuration/options object.

### Preconditions
- Multiple functions pass the same cluster of parameters together across call chains.

### Execution Steps
1. Declare a new interface or dataclass representing the parameter group:
   ```typescript
   export interface SearchFilterOptions {
     query: string;
     category?: string;
     minPrice?: number;
     maxPrice?: number;
     limit?: number;
   }
   ```
2. Update the target function signature to accept the parameter object.
3. Update call sites one by one (or using default parameter destructuring to maintain backward compatibility during migration).
4. **Run tests immediately**.

---

## 4. Invert Dependency (Introduce Surgical Seam)

### Intent
Decouple a function or class from a hardcoded global instance, static singleton, or external network client to enable independent testing and modularity.

### Preconditions
- The target unit directly instantiates an external client (e.g. `new S3Client()` or `DatabasePool.getInstance()`).

### Execution Steps
1. Define an interface representing the minimal subset of methods the target unit actually consumes.
2. Add an optional parameter (or constructor argument) for the interface, defaulting to the current production implementation:
   ```typescript
   export class ReportGenerator {
     private s3: S3StorageClient;

     constructor(options?: { s3Client?: S3StorageClient }) {
       this.s3 = options?.s3Client ?? defaultS3Client;
     }
   }
   ```
3. Existing production call sites remain 100% unaffected.
4. Tests can now inject a lightweight in-memory fake client.
5. **Run tests immediately**.
