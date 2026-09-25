# API Evolution & Deprecation Walkthrough

> **Scenario**: A high-volume SaaS subscription platform needs to replace its legacy flat plan system (`plan_id: "plan_basic"`) with a multi-tiered, seat-based pricing model (`tier: "PRO", seat_count: 5`). Millions of active API consumers call this endpoint daily. A hard breaking change would cause widespread outages.
>
> This walkthrough demonstrates the end-to-end execution of the **Expand-Contract Protocol** under the Monotonic Compatibility Axiom.

---

## 1 · Initial Baseline Contract (Version 1.0)

### Legacy Request Schema
```json
{
  "customer_id": "cust_01HZX87K",
  "plan_id": "plan_enterprise_flat",
  "payment_method_id": "pm_99213"
}
```

---

## 2 · Phase 1: Expand (Additive Evolution)

We introduce the new billing model while maintaining 100% backward compatibility for existing consumers using `plan_id`.

### 2.1 The Additive Contract (Version 1.1)
```json
{
  "customer_id": "cust_01HZX87K",
  "plan_id": {
    "type": "string",
    "deprecated": true,
    "description": "DEPRECATED: Use tier and seat_count instead. Sunset: 2027-03-31."
  },
  "tier": {
    "type": "string",
    "enum": ["STARTER", "PRO", "ENTERPRISE"]
  },
  "seat_count": {
    "type": "integer",
    "minimum": 1,
    "default": 1
  },
  "payment_method_id": "pm_99213"
}
```

### 2.2 Server-Side Anti-Corruption Translation Adapter
```typescript
interface CreateSubscriptionInput {
  customerId: string;
  paymentMethodId: string;
  planId?: string;       // Legacy parameter
  tier?: SubscriptionTier; // Modern parameter
  seatCount?: number;     // Modern parameter
}

export function normalizeSubscriptionRequest(input: CreateSubscriptionInput): ValidatedSubscriptionOrder {
  // 1. Defend against ambiguous input conflicts
  if (input.planId && input.tier) {
    throw new CanonicalApiError({
      code: "INVALID_ARGUMENT",
      httpStatus: 400,
      message: "Cannot specify both legacy 'plan_id' and modern 'tier' simultaneously."
    });
  }

  // 2. Backward compatibility fallback: Map legacy plan to modern tier
  if (input.planId) {
    const legacyMapping = LEGACY_PLAN_MAP[input.planId];
    if (!legacyMapping) {
      throw new CanonicalApiError({
        code: "INVALID_ARGUMENT",
        httpStatus: 400,
        message: `Unknown legacy plan_id '${input.planId}'.`
      });
    }
    return {
      customerId: input.customerId,
      paymentMethodId: input.paymentMethodId,
      tier: legacyMapping.tier,
      seatCount: legacyMapping.defaultSeats,
      isLegacyCaller: true
    };
  }

  // 3. Modern validation path
  if (!input.tier) {
    throw new CanonicalApiError({
      code: "INVALID_ARGUMENT",
      httpStatus: 400,
      message: "Field 'tier' is required when 'plan_id' is omitted."
    });
  }

  return {
    customerId: input.customerId,
    paymentMethodId: input.paymentMethodId,
    tier: input.tier,
    seatCount: input.seatCount ?? 1,
    isLegacyCaller: false
  };
}
```

### 2.3 Deprecation Signaling in Outgoing HTTP Response
When `isLegacyCaller === true`, the server attaches standard IETF deprecation headers:

```http
HTTP/1.1 201 Created
Content-Type: application/json
Deprecation: @1743379200
Sunset: Wed, 31 Mar 2027 00:00:00 GMT
Link: <https://api.corp.com/docs/migrations/seat-based-pricing>; rel="sunset"; type="text/html"

{
  "id": "sub_01HZX87K",
  "status": "ACTIVE",
  "tier": "ENTERPRISE",
  "seat_count": 10,
  "warnings": [
    {
      "code": "DEPRECATED_PARAMETER",
      "message": "Parameter 'plan_id' is deprecated. Migrate to 'tier' and 'seat_count' before 2027-03-31."
    }
  ]
}
```

---

## 3 · Phase 2: Migrate (Telemetry & Consumer Tracking)

1. **Telemetry Counter**: Instrument metrics emitting tag `contract_version: legacy` vs `contract_version: modern`.
2. **Traffic Audit**: Generate automated weekly reports of active API keys still sending `plan_id`.
3. **Consumer Notification**: Send targeted developer notifications to account holders whose traffic is classified as legacy.

---

## 4 · Phase 3: Contract (Safe Tombstoning)

After the sunset date passes and traffic from `plan_id` drops to zero (or reaches the governance hard-cutoff threshold):

### 4.1 The Tombstone Response
Any remaining requests with `plan_id` receive an explicit terminal response rather than a silent failure:

```http
HTTP/1.1 410 Gone
Content-Type: application/problem+json

{
  "type": "https://api.corp.com/errors/parameter-sunset",
  "title": "Parameter Sunset",
  "status": 410,
  "code": "PARAMETER_SUNSET",
  "detail": "Parameter 'plan_id' was sunset on 2027-03-31. Please update your integration to pass 'tier' and 'seat_count'.",
  "instance": "/subscriptions",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736"
}
```

### 4.2 Excision Diff
1. Remove `LEGACY_PLAN_MAP` lookup tables.
2. Remove translation shims.
3. Mark `plan_id` in database migration as retired.
4. Baseline test suite passes with zero legacy branches.
