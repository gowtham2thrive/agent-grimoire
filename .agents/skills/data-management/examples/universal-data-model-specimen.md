# Universal Data Model Specimen: Subscription Billing & Entitlements

> **Purpose**: *This document serves as the canonical reference specimen and reusable template for formalizing data models across Conceptual, Logical, and Physical boundaries within the Arsenal ecosystem.*

---

## 1. Domain Metadata & Boundary Profile

* **Bounded Context**: `Billing & Customer Entitlements`
* **Authoritative Service Owner**: `SubscriptionService` (Single-Writer)
* **Storage Paradigm**: Relational OLTP (Authoritative) + Analytical Event Stream (Audit)
* **Consistency Profile**: Strong ACID for Ledger & Invoicing; Read-Committed with OCC for Subscriptions
* **Data Classification**: Financial & PII (Requires Field-Level Encryption & Cryptographic Shredding)

---

## 2. Conceptual Domain Model

```
┌─────────────────┐       1:N       ┌──────────────────────┐       1:N       ┌─────────────────────┐
│  Organization   │ ─────────────── │     Subscription     │ ─────────────── │  SubscriptionItem   │
│ (Tenant Root)   │                 │ (State: Active/Past) │                 │  (Plan / Meter SKU) │
└─────────────────┘                 └──────────────────────┘                 └─────────────────────┘
         │                                     │                                        │
         │ 1:N                                 │ 1:N                                    │
         ▼                                     ▼                                        ▼
┌─────────────────┐                 ┌──────────────────────┐                 ┌─────────────────────┐
│ CustomerInvoice │                 │    BillingSchedule   │                 │    EntitlementGrant │
│  (Final Ledger) │                 │   (Proration Rules)  │                 │    (Feature Flags)  │
└─────────────────┘                 └──────────────────────┘                 └─────────────────────┘
```

### Business Invariants
1. An `Organization` must have exactly one active `Subscription` at any given instant.
2. A `Subscription` cannot transition from `CANCELLED` to `ACTIVE` (state transitions are strictly one-way; reactivation requires creating a new subscription entity).
3. The total amount on a finalized `CustomerInvoice` must equal the exact sum of its invoice line items ($\sum \text{LineItems} = \text{Total}$).
4. Invoices once finalized are **immutable financial records** and cannot be deleted or updated.

---

## 3. Logical Data Model

### 3.1 Grain Declarations ($\mathcal{G}$)
* `subscriptions`: One record represents the continuous lifecycle of a specific customer's billing relationship.
* `subscription_items`: One record represents an individual SKU/Plan entitlement attached to a subscription.
* `customer_invoices`: One record represents an immutable, finalized billing charge event.

### 3.2 Relational Logical Schema (DDL Specimen)

```sql
-- Enforces Invariant 1: Multi-Modal Grain & Identity (UUIDv7)
CREATE TABLE subscriptions (
    id UUID PRIMARY KEY, -- UUIDv7 (Timestamp-ordered, 128-bit)
    organization_id UUID NOT NULL,
    plan_tier VARCHAR(64) NOT NULL,
    status VARCHAR(32) NOT NULL,
    current_period_start TIMESTAMPTZ NOT NULL,
    current_period_end TIMESTAMPTZ NOT NULL,
    version INT NOT NULL DEFAULT 1, -- For Optimistic Concurrency Control
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- Enforces Invariant 3: Domain Range Invariants at Storage Layer
    CONSTRAINT check_subscription_status CHECK (
        status IN ('TRIALING', 'ACTIVE', 'PAST_DUE', 'CANCELLED', 'EXPIRED')
    ),
    CONSTRAINT check_valid_period CHECK (
        current_period_end > current_period_start
    )
);

-- Active subscription exclusivity invariant per tenant (Exclusion / Partial Unique)
CREATE UNIQUE INDEX idx_subscriptions_one_active_per_org 
ON subscriptions (organization_id) 
WHERE (status IN ('TRIALING', 'ACTIVE', 'PAST_DUE'));

CREATE TABLE subscription_items (
    id UUID PRIMARY KEY,
    subscription_id UUID NOT NULL REFERENCES subscriptions(id) ON DELETE RESTRICT,
    feature_code VARCHAR(64) NOT NULL,
    allocated_quantity INT NOT NULL DEFAULT 1,
    unit_price_cents BIGINT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT check_positive_quantity CHECK (allocated_quantity >= 0),
    CONSTRAINT check_non_negative_price CHECK (unit_price_cents >= 0),
    CONSTRAINT unique_feature_per_subscription UNIQUE (subscription_id, feature_code)
);

-- Immutable Finalized Ledger
CREATE TABLE customer_invoices (
    id UUID PRIMARY KEY,
    organization_id UUID NOT NULL,
    subscription_id UUID NOT NULL REFERENCES subscriptions(id) ON DELETE RESTRICT,
    invoice_number VARCHAR(64) NOT NULL UNIQUE,
    total_amount_cents BIGINT NOT NULL,
    currency CHAR(3) NOT NULL,
    finalized_at TIMESTAMPTZ NOT NULL,
    idempotency_key VARCHAR(128) NOT NULL UNIQUE, -- Invariant 5: Idempotent emission

    CONSTRAINT check_valid_currency CHECK (currency IN ('USD', 'EUR', 'GBP'))
);
```

---

## 4. Physical Storage Architecture

* **Database Engine**: PostgreSQL 16 (B-Tree storage engine with MVCC)
* **Primary Key Mechanics**: UUIDv7 generated in application layer via monotonic timestamp prefix, guaranteeing sequential B-Tree page insertion without random fragmentation.
* **Secondary Indexing Strategy**:
  * `idx_subscriptions_org_lookup`: B-Tree index on `(organization_id, status)` for fast tenant lookups.
  * `idx_invoices_finalized_date`: B-Tree index on `(finalized_at DESC)` for billing reconciliation scans.
* **Isolation Level**:
  * Routine queries: `READ COMMITTED` with Optimistic Concurrency Control (`version = version + 1`).
  * Invoice Finalization: Explicit `SERIALIZABLE` transaction to prevent race conditions during billing runs.

---

## 5. Lifecycle, Privacy & Governance Policy

* **Retention Tiering**:
  * `subscriptions`: Retained indefinitely in hot OLTP storage for customer history.
  * `customer_invoices`: Retained 7 years in hot/warm storage to comply with statutory financial audits; archived to cold WORM object storage (S3 Glacier Vault) with immutable compliance locks.
* **Privacy & GDPR Compliance**:
  * Customer billing address and billing contact names are encrypted with the customer's tenant key ($K_{\text{tenant}}$).
  * Upon tenant account closure and expiration of the statutory audit period, $K_{\text{tenant}}$ is **Cryptographically Shredded**, permanently rendering customer PII unreadable.
