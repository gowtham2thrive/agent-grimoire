# Full-Stack API Load Testing & Profiling Walkthrough

> **Purpose**: Demonstrates a complete brownfield performance optimization campaign on a production API endpoint using load testing, flame graphs, the solution hierarchy, and regression shielding.

---

## 1 · Scenario Context

A high-traffic e-commerce order checkout API endpoint (`POST /api/v1/orders`) is experiencing severe latency spikes under peak load. 
- **Requirement**: Endpoint must maintain $P_{99} \le 120\text{ms}$ at $2,500\text{ req/sec}$ with $< 0.1\%$ error rate.

---

## 2 · Step-by-Step Execution Trace

### Phase 1: Baseline Load Test Capture
The agent executes an open-arrival load test using an open-system rate generator:
```javascript
// k6 script: constant-arrival-rate
export const options = {
  scenarios: {
    constant_request_rate: {
      executor: 'constant-arrival-rate',
      rate: 2500,
      timeUnit: '1s',
      duration: '10m',
      preAllocatedVUs: 100,
      maxVUs: 500,
    },
  },
  thresholds: {
    http_req_duration: ['p(99)<120'], // SLA requirement
    http_req_failed: ['rate<0.001'],
  },
};
```

**Pre-Mutation Baseline Metrics ($M_{\text{pre}}$)**:
- $P_{50}$: $38\text{ms}$
- $P_{95}$: $185\text{ms}$
- $P_{99}$: $940\text{ms}$ (❌ Fails SLA threshold of $120\text{ms}$)
- Throughput achieved: $2,180\text{ req/sec}$ (Target was $2,500$)
- Error Rate: $1.4\%$ (`HTTP 504 Gateway Timeout`)

---

### Phase 2: Profiler Routing & Bottleneck Isolation
The agent notes:
- Application server CPU is only at $22\%$ (ruling out pure on-CPU computational bottlenecks).
- The agent routes to an **Off-CPU / Wait-State profiler** and database query execution plan.

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM inventory_reservations 
WHERE warehouse_id = 42 AND status = 'AVAILABLE' 
ORDER BY priority DESC LIMIT 10;
```
*Profiler Output*:
- `Seq Scan on inventory_reservations (cost=0.00..48200.00 rows=1250000 width=72)`
- `Buffers: shared read=38200, hit=1200`
- `Execution Time: 214.32ms`
- Database lock wait time: Connection pool was set to 250 connections, causing PostgreSQL lock manager contention.

---

### Phase 3: Solution Hierarchy Traversal
1. **Tier 1 (Configuration)**: Reduce database connection pool from 250 to 33 connections using the universal pool sizing formula: $(\text{CPU Cores} \times 2) + \text{Spindle} = (16 \times 2) + 1 = 33$.
2. **Tier 2 (Storage Indexing)**: Add a composite covering index:
   ```sql
   CREATE INDEX CONCURRENTLY idx_inventory_warehouse_status_prio
   ON inventory_reservations (warehouse_id, status, priority DESC)
   INCLUDE (id, sku);
   ```

---

### Phase 4 & 5: Targeted Mutation & Re-measurement
The agent updates the connection pool configuration in the service environment and executes the database migration.

The identical 10-minute load test is re-run:
- `EXPLAIN (ANALYZE)` confirms: `Index Only Scan using idx_inventory_warehouse_status_prio` ($0.18\text{ms}$ vs $214.32\text{ms}$).

**Post-Mutation Metrics ($M_{\text{post}}$)**:
- $P_{50}$: $12\text{ms}$ ($-68.4\%$)
- $P_{95}$: $34\text{ms}$ ($-81.6\%$)
- $P_{99}$: $68\text{ms}$ ($-92.7\%$) (✅ Passes SLA threshold of $120\text{ms}$)
- Throughput: $2,500\text{ req/sec}$ (100% target achieved)
- Error Rate: $0.00\%$ (Zero timeouts)

---

### Phase 6 & 7: Regression Shielding & Stopping Certification
The agent locks the fix by adding an automated query plan assertion in the integration test suite:
```python
def test_inventory_query_plan_uses_index(db_session):
    plan = db_session.execute("EXPLAIN SELECT ...").fetchall()
    assert "Seq Scan" not in str(plan)
    assert "Index Only Scan" in str(plan)
```
The stopping contract is certified with documented evidence.
