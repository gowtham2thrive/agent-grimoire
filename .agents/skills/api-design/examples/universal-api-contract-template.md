# Universal API Contract Specification Template

Use this template when authoring formal interface specifications in `endpoint-spec` or `subsystem-contract` modes.

---

# [Service / Interface Name] Contract Specification

* **Contract Version**: `v1.2.0` (SemVer)
* **Archetype**: [REST | gRPC | GraphQL | Async Event | CLI | SDK | MCP Tool]
* **Maturity / Lifecycle**: [Draft | Active | Deprecated | Sunset]
* **Target Audience**: [Public External | Partner | Internal Subsystem]

---

## 1 · Consumer Intent & Problem Statement
* **Job-to-be-Done**: [What consumer problem does this interface solve?]
* **Consumer Mental Model**: [Key abstractions, domain concepts, and relationship to consumer workflow]
* **Non-Goals / Scope Boundaries**: [What this interface explicitly does NOT do]

---

## 2 · Operation Catalog & Semantics

### Operation: `[OperationName]` (e.g. `CancelOrder`)
* **Semantic Type**: [Read-Only Query | Mutating Transition | Upsert | Bulk / Batch]
* **Protocol Signature**:
  * *REST*: `POST /orders/{id}/actions/cancel`
  * *gRPC*: `rpc CancelOrder(CancelOrderRequest) returns (CancelOrderResponse)`
  * *CLI*: `corp order cancel <id> --reason <text>`
* **Idempotency Guarantee**:
  * [None (Read) | Natural Idempotent | Tokenized Deduplication via `Idempotency-Key` or Request UUID]
  * Concurrency Lock Window: `30 seconds`
  * Payload Mismatch Behavior: `422 Unprocessable (INVALID_ARGUMENT)`
* **Authorization & Scopes**:
  * Required Scope: `orders:write`
  * Multi-Tenancy Boundary: `tenant_id` resolved cryptographically from auth context.

---

## 3 · Data Schemas & Invariant Bounds

### 3.1 Input / Request Schema
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["cancellation_reason"],
  "properties": {
    "cancellation_reason": {
      "type": "string",
      "minLength": 3,
      "maxLength": 500,
      "description": "Human-readable justification for cancellation."
    },
    "refund_preference": {
      "type": "string",
      "enum": ["ORIGINAL_PAYMENT", "STORE_CREDIT"],
      "default": "ORIGINAL_PAYMENT"
    }
  },
  "additionalProperties": false
}
```

### 3.2 Output / Response Schema
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["id", "status", "cancelled_at", "refund_amount"],
  "properties": {
    "id": { "type": "string", "pattern": "^ord_[0-9a-zA-Z]{16}$" },
    "status": { "type": "string", "enum": ["CANCELLED"] },
    "cancelled_at": { "type": "string", "format": "date-time" },
    "refund_amount": {
      "type": "object",
      "required": ["currency", "units"],
      "properties": {
        "currency": { "type": "string", "minLength": 3, "maxLength": 3 },
        "units": { "type": "integer", "minimum": 0 }
      }
    }
  }
}
```

---

## 4 · State Transition & Precondition Invariants

* **Valid Source States**: `["DRAFT", "PENDING_PAYMENT", "PROCESSING"]`
* **Target State on Success**: `CANCELLED`
* **Precondition Guards**:
  * If current state is `SHIPPED` $\to$ Reject with `FAILED_PRECONDITION` (`409 Conflict`).
  * If current state is `CANCELLED` $\to$ Replay previous cancellation outcome deterministically.

---

## 5 · Failure Domain & Canonical Errors

| Error Code | HTTP / Transport Code | Cause / Trigger | Safe Message | Retryable? |
| :--- | :--- | :--- | :--- | :--- |
| `INVALID_ARGUMENT` | `400 Bad Request` | Reason too short or exceeds 500 characters. | "Reason must be between 3 and 500 characters." | ❌ No |
| `NOT_FOUND` | `404 Not Found` | Order ID does not exist in tenant domain. | "Order 'ord_xxx' could not be found." | ❌ No |
| `FAILED_PRECONDITION` | `409 Conflict` | Order is in SHIPPED status. | "Order cannot be cancelled after shipping." | ❌ No |
| `RESOURCE_EXHAUSTED` | `429 Too Many Req` | Rate limit of 60 req/min exceeded. | "Rate limit exceeded. Retry after indicated period." | ⚠️ Yes |

---

## 6 · Resource Bounds & Traffic Envelope

* **Rate Limit Policy**: `Token Bucket` — 100 requests / minute per client token. Burst capacity: 15.
* **Pagination Envelope** (if collection):
  * Default Limit: `20`
  * Max Limit Ceiling: `100` (server clamped)
  * Cursor Type: Keyset cursor (`ORDER BY created_at DESC, id DESC`)
* **Timeout & SLO**:
  * Server Processing Deadline: `1,500 ms`
  * P99 Latency Target: `< 250 ms`

---

## 7 · Verification & Contract Test Criteria

1. **Happy Path Assertions**: Successful transition from `PROCESSING` to `CANCELLED` produces expected schema diff.
2. **Idempotency Replay Test**: 10 parallel identical requests with key `test_key_1` return identical responses and trigger exactly one refund execution.
3. **Mismatched Payload Replay Test**: Request with key `test_key_1` and altered reason returns `422 Unprocessable`.
4. **Boundary Fuzzing**: Negative inputs (SQL characters, 10,000-char strings, negative values) rejected at schema perimeter.
