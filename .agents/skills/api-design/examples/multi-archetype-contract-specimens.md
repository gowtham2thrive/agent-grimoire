# Multi-Archetype Contract Specimens (Rosetta Stone)

> **Mandate**: *An interface contract is paradigm-independent.* To demonstrate that the 8 Universal API Design Invariants apply universally across any technology or runtime, this specimen models the exact same domain operation—**Issuing a Customer Refund**—across 5 distinct software archetypes:
> 1. **REST / HTTP Web API**
> 2. **gRPC / Protocol Buffers**
> 3. **Asynchronous Event-Driven (CloudEvents)**
> 4. **Command-Line Interface (CLI)**
> 5. **AI Agent Tool / MCP (Model Context Protocol)**

---

## The Common Domain Operation Specification
* **Domain Operation**: `IssueRefund`
* **Preconditions**: Payment exists, status is `SETTLED`, unrefunded balance $\ge$ requested refund amount.
* **Idempotency**: Tokenized execution; duplicate attempts with identical key replay outcome; mismatched payloads return `INVALID_ARGUMENT`.
* **Resource Bounds**: Refund amount must be $> 0$ and $\le \$100,000.00$. Reason string bounded between 3 and 500 characters.

---

## Archetype 1: REST / HTTP Web API

### Request
```http
POST /v1/payments/pay_01HZX87K/actions/refund HTTP/1.1
Host: api.payments.example.com
Authorization: Bearer sec_tok_99128...
Idempotency-Key: 7b34da6a-3ce9-49d0-e0e4-7364bf92f357
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
Content-Type: application/json

{
  "amount_cents": 4500,
  "currency": "USD",
  "reason": "Customer returned undamaged merchandise."
}
```

### Success Response (`201 Created`)
```http
HTTP/1.1 201 Created
Content-Type: application/json
X-Correlation-ID: 4bf92f3577b34da6a3ce929d0e0e4736

{
  "id": "ref_01HZX99M",
  "payment_id": "pay_01HZX87K",
  "amount_cents": 4500,
  "currency": "USD",
  "status": "COMPLETED",
  "created_at": "2026-09-25T14:32:00Z"
}
```

---

## Archetype 2: gRPC / Protocol Buffers (proto3)

### Service & Message Contract (`refund_service.proto`)
```protobuf
syntax = "proto3";

package payments.v1;

import "google/protobuf/timestamp.proto";
import "google/rpc/status.proto";

service PaymentService {
  // Issues a partial or full refund against a settled payment.
  // Idempotent when request_id is reused with identical parameters.
  rpc IssueRefund(IssueRefundRequest) returns (IssueRefundResponse);
}

message IssueRefundRequest {
  string payment_id = 1;      // e.g. "pay_01HZX87K"
  int64 amount_cents = 2;     // Bounded: 1 <= amount <= 10000000
  string currency = 3;        // ISO-4217 3-letter code
  string reason = 4;          // Length: 3..500 characters
  string idempotency_key = 5; // Client-generated UUIDv4/UUIDv7
}

message IssueRefundResponse {
  string refund_id = 1;
  string payment_id = 2;
  int64 amount_cents = 3;
  string currency = 4;
  RefundStatus status = 5;
  google.protobuf.Timestamp created_at = 6;
}

enum RefundStatus {
  REFUND_STATUS_UNSPECIFIED = 0;
  REFUND_STATUS_PENDING = 1;
  REFUND_STATUS_COMPLETED = 2;
  REFUND_STATUS_FAILED = 3;
}
```

---

## Archetype 3: Asynchronous Event-Driven (CloudEvents v1.0)

### Event Schema (`payment.refund.requested.v1`)
```json
{
  "specversion": "1.0",
  "type": "com.payments.refund.requested.v1",
  "source": "https://service.payments.example.com",
  "id": "evt_7b34da6a3ce94da6",
  "time": "2026-09-25T14:32:00Z",
  "datacontenttype": "application/json",
  "traceparent": "00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01",
  "data": {
    "payment_id": "pay_01HZX87K",
    "amount_cents": 4500,
    "currency": "USD",
    "reason": "Customer returned undamaged merchandise.",
    "idempotency_key": "7b34da6a-3ce9-49d0-e0e4-7364bf92f357"
  }
}
```

---

## Archetype 4: Command-Line Interface (CLI / POSIX)

### Command Invocation
```bash
corp-pay refund \
  --payment-id pay_01HZX87K \
  --amount 45.00 \
  --currency USD \
  --reason "Customer returned undamaged merchandise." \
  --idempotency-key 7b34da6a-3ce9-49d0-e0e4-7364bf92f357 \
  --json
```

### Exit Codes & Structured Standard Streams
* **Success Exit Code**: `0`
* **Validation Error Code**: `2` (`INVALID_ARGUMENT`)
* **State Conflict Error Code**: `3` (`FAILED_PRECONDITION`)
* **Stdout (when `--json` flag passed)**:
```json
{
  "refund_id": "ref_01HZX99M",
  "status": "COMPLETED",
  "amount_cents": 4500
}
```
* **Stderr (on error)**:
```
Error: [FAILED_PRECONDITION] Payment 'pay_01HZX87K' has already been fully refunded.
Correlation ID: 4bf92f3577b34da6a3ce929d0e0e4736
```

---

## Archetype 5: AI Agent Tool / MCP (Model Context Protocol)

### Tool Schema Definition
```json
{
  "name": "payments_issue_refund",
  "description": "Issues a partial or full refund against a previously settled payment. Fully idempotent when request_id is reused.",
  "inputSchema": {
    "type": "object",
    "required": ["payment_id", "amount_cents", "reason", "idempotency_key"],
    "properties": {
      "payment_id": {
        "type": "string",
        "pattern": "^pay_[0-9a-zA-Z]{8,16}$",
        "description": "The unique identifier of the settled payment to refund."
      },
      "amount_cents": {
        "type": "integer",
        "minimum": 1,
        "maximum": 10000000,
        "description": "Refund amount in smallest currency units (e.g. 4500 for $45.00)."
      },
      "currency": {
        "type": "string",
        "minLength": 3,
        "maxLength": 3,
        "default": "USD"
      },
      "reason": {
        "type": "string",
        "minLength": 3,
        "maxLength": 500,
        "description": "Audit justification for why the refund is being issued."
      },
      "idempotency_key": {
        "type": "string",
        "description": "Unique client token to prevent duplicate charges on retry."
      }
    },
    "additionalProperties": false
  }
}
```
