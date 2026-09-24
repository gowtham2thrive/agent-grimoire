# Feature Pre-Flight: Add Webhook Signature Verification
> Target Objective: Enforce HMAC SHA-256 signature verification on inbound payment webhooks from Stripe before job dispatch.

## 1. Mutation Surface
| Target File | Action | Purpose | Existing Tests | Churn Risk |
| :--- | :--- | :--- | :--- | :--- |
| `apps/api/src/middleware/verify-stripe.ts` | **Create** | Dedicated Fastify pre-handler plugin for HMAC signature checking. | N/A (New File) | Low |
| `apps/api/src/routes/webhooks.ts` | **Modify** | Attach `verifyStripeSignature` hook to `POST /webhooks/stripe`. | 2 unit tests in `apps/api/tests/webhooks.test.ts` | Low (4 commits in 3mo) |
| `apps/api/src/config/env.ts` | **Modify** | Add `STRIPE_WEBHOOK_SECRET` to Zod environment schema. | High (Tested in `env.test.ts`) | Low |
| `packages/types/src/webhook.ts` | **Modify** | Export typed webhook payload interfaces. | High | Low |

## 2. Ingress & Egress Impact
- **Upstream Callers Affected**: External Stripe webhook dispatchers. Requests without valid `stripe-signature` header will be rejected before hitting route logic.
- **Downstream Services Touched**: Fastify raw body parser (must retain raw unparsed buffer for cryptographic signature validation).
- **Invariants at Risk**:
  - `[INV-03]` (Webhook Idempotency): Signature verification must happen *before* idempotency key lookup in Redis to prevent replay attacks with fake signatures.

## 3. Pre-Flight Test Plan
- **Baseline Verification**:
  ```powershell
  pnpm --filter @repo/api test -- webhooks.test.ts
  ```
  *(Status: 2 passed in 1.4s)*
- **Post-Mutation Verification**:
  1. Valid signature header $\rightarrow$ HTTP 200 + BullMQ job dispatched.
  2. Invalid signature header $\rightarrow$ HTTP 400 + zero Redis writes.
  3. Missing signature header $\rightarrow$ HTTP 401 + security audit log emitted.

## 4. Blast-Radius Risk Score: MODERATE
- **Risk Rationale**: Fastify JSON body parser automatically parses bodies into objects. HMAC signature verification requires the exact raw unparsed request buffer. Misconfiguring the raw body parser plugin could corrupt body parsing for other POST routes in `apps/api`.
- **Precaution**: Scope raw body retention strictly to the `/webhooks/*` route prefix.
