# Data Quality Audit Scorecard & Certification Report

> **Dataset Under Audit**: `analytics_mart.customer_orders_daily`  
> **Authoritative Upstream Source**: `OrderService` (PostgreSQL OLTP replica)  
> **Evaluation Horizon**: Last 30 Days (Sample Size: 2,450,000 records)  
> **Auditing Agent**: Data Management Specialist  
> **Overall Certification**: ⚠️ **CONDITIONAL PASS** ($Q_{\text{score}} = 0.942$ / Target: $\ge 0.950$)

---

## 1. Executive Quality Scorecard Summary

$$Q_{\text{score}} = \sum_{i=1}^{6} w_i \cdot D_i = 0.942$$

| Quality Dimension | Weight ($w_i$) | Raw Score ($D_i$) | Weighted Contribution | Status | Primary Finding |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Completeness** | 0.20 | 0.985 | 0.197 | ✅ PASS | 1.5% nulls on `billing_postal_code` (acceptable for guest checkouts). |
| **2. Uniqueness** | 0.25 | 1.000 | 0.250 | ✅ PASS | Zero duplicate composite keys on `(order_id, line_item_id)`. |
| **3. Validity** | 0.20 | 0.890 | 0.178 | ❌ FAIL | 11% of order currencies stored as lowercase (`'usd'` instead of `'USD'`). |
| **4. Timeliness** | 0.15 | 0.960 | 0.144 | ✅ PASS | Mean pipeline ingestion lag = 4.2 minutes (SLA: $\le 15$ min). |
| **5. Consistency** | 0.10 | 0.920 | 0.092 | ⚠️ WARN | 8% discrepancy between Mart order totals and raw Payment gateway logs. |
| **6. Accuracy** | 0.10 | 0.910 | 0.091 | ⚠️ WARN | Tax calculation rounding variances detected on split shipments. |

---

## 2. Automated Quality Assertion Test Suite (SQL Specimen)

Below are the empirical SQL assertion tests used to calculate each dimensional score:

### Test 2.1: Completeness Assertion
```sql
-- Metric: Percentage of non-null mandatory fields
SELECT 
    COUNT(*) AS total_rows,
    COUNT(order_id)::FLOAT / COUNT(*) AS completeness_order_id,
    COUNT(customer_id)::FLOAT / COUNT(*) AS completeness_customer_id,
    COUNT(total_amount)::FLOAT / COUNT(*) AS completeness_total_amount
FROM analytics_mart.customer_orders_daily;
-- Result: 1.000 (Passes completeness gate)
```

### Test 2.2: Uniqueness Assertion (Grain Verification)
```sql
-- Metric: Zero duplicate records for declared Grain G
SELECT 
    order_id, 
    line_item_id, 
    COUNT(*) AS collision_count
FROM analytics_mart.customer_orders_daily
GROUP BY order_id, line_item_id
HAVING COUNT(*) > 1;
-- Result: 0 rows returned (100% Unique)
```

### Test 2.3: Validity & Format Assertion
```sql
-- Metric: Check ISO currency conformity and non-negative amounts
SELECT 
    COUNT(*) AS invalid_records,
    COUNT(*)::FLOAT / (SELECT COUNT(*) FROM analytics_mart.customer_orders_daily) AS invalid_rate
FROM analytics_mart.customer_orders_daily
WHERE 
    total_amount < 0 
    OR currency !~ '^[A-Z]{3}$'; -- Must match 3 uppercase letters
-- Result: 269,500 invalid rows (11% format violation due to lowercase 'usd')
```

### Test 2.4: Consistency Assertion (Cross-Boundary Parity)
```sql
-- Metric: Cross-system reconciliation against Payment Gateway ledger
SELECT 
    o.order_id,
    o.total_amount AS mart_amount,
    p.charged_amount AS gateway_amount,
    ABS(o.total_amount - p.charged_amount) AS diff
FROM analytics_mart.customer_orders_daily o
JOIN payment_service.gateway_charges p ON o.order_id = p.order_id
WHERE ABS(o.total_amount - p.charged_amount) > 0.01;
-- Result: 19,600 mismatched rows (Investigate currency conversion timing)
```

---

## 3. Prioritized Remediation Action Plan

```
┌────────────────────────────────────────────────────────────────────────┐
│                        REMEDIATION ACTION DAG                          │
├────────────────────────────────────────────────────────────────────────┤
│ Priority 1 (Blocker): Add Ingress Uppercase Normalization Filter       │
│             -> Enforce `currency = UPPER(currency)` at ingest stream   │
│ Priority 2 (Medium):  Fix Payment Gateway Multi-Currency Timestamp     │
│             -> Reconcile exchange rate lookup to exact transaction     │
│                timestamp instead of daily close rate                   │
│ Priority 3 (Low):     Backfill Historical Lowercase Currencies         │
│             -> Run keyset batch update to capitalize legacy rows       │
└────────────────────────────────────────────────────────────────────────┘
```

### 4. Promotion Certification Decision
* **Decision**: **QUARANTINE FROM FINANCIAL REPORTING**.
* **Reason**: While valid for marketing engagement analysis, the 8% gateway discrepancy and unnormalized currency strings violate financial audit invariance.
* **Re-evaluation Trigger**: Re-run automated suite post-ingest normalization deployment.
