# REVIEW_FINDINGS Template

```markdown
# Code Review Findings: [PR Title or Target Branch]

> **Review Verdict**: [APPROVED | REQUEST_CHANGES | BLOCKED]
> **Scope Audited**: [git diff reference, e.g. main...feature-auth (4 files, +120 -15 lines)]
> **Stage 1 (Spec Compliance)**: [PASSED | DEFICIT DETECTED]

---

## Executive Summary

| Severity | Count | Status |
| :--- | :---: | :--- |
| **P0 - Blocker** | 0 | None detected |
| **P1 - Critical** | 1 | Action required before merge |
| **P2 - Moderate** | 1 | Recommended improvement |
| **P3 - Advisory** | 0 | Suppressed (noise filtering) |

---

## Detailed Findings

### [P1] Missing Transaction Rollback on Payment Failure
* **File & Line**: `[CODE: src/services/billing.ts#L84-L92]`
* **Lens**: Contract & Correctness / Resource Safety
* **Confidence**: 95/100 (Verified line citation and execution sequence)

#### 1. Failure Scenario
When `gateway.charge()` rejects due to an insufficient funds error, the database transaction is not aborted. The subsequent catch block logs the error, but the open connection is released back to the pool with an active uncommitted transaction, leaving pending rows locked.

#### 2. Blast Radius & Impact
High. Subsequent database operations borrowing this pooled connection inherit an open transaction, leading to deadlocks and data inconsistency under concurrent traffic.

#### 3. Recommended Remediation
```diff
--- a/src/services/billing.ts
+++ b/src/services/billing.ts
@@ -84,7 +84,8 @@ export async function processBilling(userId: string, amount: number) {
     await db.orders.updateStatus(orderId, "COMPLETED");
   } catch (err) {
+    await tx.rollback();
     logger.error("Billing failed", { userId, err });
     throw new BillingError("Charge failed", { cause: err });
   } finally {
```

---

### [P2] Unbounded Database Query in Order History
* **File & Line**: `[CODE: src/controllers/orders.ts#L35]`
* **Lens**: Performance & Resource Safety
* **Confidence**: 85/100

#### 1. Failure Scenario
`db.orders.findMany({ where: { userId } })` does not enforce pagination (`take` or `limit`). For accounts with thousands of historic orders, this loads the entire dataset into memory.

#### 2. Recommended Remediation
```diff
--- a/src/controllers/orders.ts
+++ b/src/controllers/orders.ts
@@ -35,2 +35,4 @@ export async function listOrders(req: Request, res: Response) {
-  const orders = await db.orders.findMany({ where: { userId: req.user.id } });
+  const limit = Math.min(parseInt(req.query.limit as string) || 20, 100);
+  const orders = await db.orders.findMany({ where: { userId: req.user.id }, take: limit });
```

---

## Stage 1 Spec Verification Matrix

- [x] Implemented token refresh logic as requested in prompt.
- [x] Preserved backward compatibility with legacy `/api/v1/login` callers.
- [x] Handled expired token edge case with HTTP 401 response.
```
